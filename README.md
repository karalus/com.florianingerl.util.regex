![com.github.florianingerl.util.regex](media/logo.png)

### Introduction
This is a Java Regular Expressions library. Compared to the Regular Expression library shipped with the Java JDK, it provides support for Recursive and Conditional Regular Expressions, gives detailed results for a successfull match via a so-called Group Tree and allows the user to install plugins into the regex engine.

In the following screenshot, all the new features are summarized.
![com.github.florianingerl.util.regex.newfeatures](media/newfeatures.png)


### What's new :star:

### Version 1.1.1
- Group Trees
- Recursive Regular Expressions now deal with Capturing Groups, Backreferences and Backtracking similar as Perl 5.10 did (see explanations and examples below)
### Version 1.0.3
- `(?(DEFINE)never-executed-pattern)`
- Plugins for the Regex engine

### Version 1.0.2
- Recursive Regular Expressions
- Conditional Regular Expressions
- Captures

### Usage
The API is exactly the same as in java.util.regex. The only difference is that the required import statement is `import com.florianingerl.util.regex.\*;` instead of `import java.util.regex.\*;`

To illustrate the functionality, we will use the following utility functions
```java
import com.florianingerl.util.regex.*;

private static void check(String p, String s, boolean expected) 
{
	Matcher matcher = Pattern.compile(p).matcher(s);
	if (matcher.find() != expected)
		failCount++;
}
static void check(String regex, String input, String[] expected) 
{
	List<String> result = new ArrayList<String>();
	Pattern p = Pattern.compile(regex);
	Matcher m = p.matcher(input);
	while (m.find()) {
		result.add(m.group());
	}
	if (!Arrays.asList(expected).equals(result))
		failCount++;
}
```

### Recursive Regular Expressions
The following tests illustrate what you can do with Recursive Regular Expressions. Be aware that the syntax `(?R)` or `(?0)` as in Perl is not supported, only `(?n)` where `n` is greater than `0` or `(?'groupName')` is supported.
```java
//Matching Java types
String pattern = "^(?<javaType>[a-zA-Z]\\w*(?:\\<(?'javaType')(,(?'javaType'))*\\>)?)$";
check(pattern, "List<Integer>", true);
check(pattern, "HashMap<Integer,String>", true);
check(pattern, "Map<Integer,List<String>>", true);

//Matching constructs with an equal number of open and closing braces
check("(?<brace>\\((?:[^()]+|(?'brace'))*+\\))", "(go away (here (everything) is fine) afterwards",
				new String[] { "(here (everything) is fine)" });
				
//Matching anagrams
pattern = "\\b(([a-zA-Z])(?1)?\\2|[a-zA-Z])\\b";
check(pattern, "anna is an anagram, so is lagerregal and otto and radar and every single letter like z",
new String[] { "anna", "lagerregal", "otto", "radar", "z" });				
```

Different Regular Expression libraries handle Recursion, Group Capturing and Backreferences differently, so this topic deserves some comment here.

com.florianingerl.util.regex isolates capturing groups between each level of recursion. When the regex engine enters recursion, all capturing groups appear as they have not participated in the match yet. Initially, all backreferences will fail. During the recursion, capturing groups capture as normal. Backreferences match text captured during the same recursion as normal. When the regex engine exits from the recursion, all capturing groups revert to the state they were in prior to the recursion, except for the capturing group that was recursed to and has just been freshly captured.
These tests will illustrate:

```java
String pattern = "(?(DEFINE)(?<letter>[a-zA-Z]))\\b(?<anagram>(?'letter')(?'anagram')?\\k<letter>|(?'letter'))\\b";
check(pattern, "anna is an anagram, so is lagerregal and otto and radar and every single letter like z",
new String[] { "anna", "lagerregal", "otto", "radar", "z" });
pattern = "(?(DEFINE)(?<wrapper>(?<letter>[a-zA-Z])))\\b(?<anagram>(?'wrapper')(?'anagram')?\\k<letter>|(?'letter'))\\b";
check(pattern, "otto", false);

pattern = "(?(DEFINE)(?<second>\\k<first>))(?<first>[a-z])(?'second')";
check(pattern, "bb", false);
```

### Plugins for the Regex engine
Since version 1.0.3, you can install plugins for the Regex engine. The method of the Pattern class seen in the screenshot below is used for that purpose.
![com.florianingerl.util.regex.plugins](media/plugins.png)

Good examples for plugins are given in the PluginTest class, see [PluginTest.java](regex/src/test/java/com/florianingerl/util/regex/tests/PluginTest.java). You might also want to read the [JavaDoc](https://florianingerl.github.io/com.florianingerl.util.regex/).

### Group Trees
This concept is best illustrated by an example. The following regex (which is stored in a file) should parse mathematical terms such as (6*[6+7+8]+9)\*78\*[4*(6+5)+4] .

```
//term.regex
(?x) #turned on the comment mode
(?(DEFINE)
(?<term>(?'number')|(?'sum')|(?'product'))
(?<sum> (?= (?: [^()+]+ | (?<brace> \( (?: [^()]+| (?'brace') )+ \) ) )+ \+ )#lookahead, that detects the start of a sum 
(?'summand')(?:\+(?'summand'))+
) # end of sum
(?<summand> (?'number') | (?'product') | (?: (?'round') | \[ )(?: (?'sum') | (?'product') ) (?(round)\)|\]) ) # end of summand
(?<product> (?= (?: [^()*+]+ | (?'brace') )+ \* ) # lookahead, that detects the start of a product
(?'factor')(?:\*(?'factor'))+
) # end of product
(?<factor>(?'number')| (?: (?'round') | \[ )(?: (?'sum') | (?'product') ) (?(round)\)|\]) ) # end of factor
(?<number>\d+)
(?<round>\()
)# end of DEFINE
(?'term')
```

After having parsed a term, you can inspect the so-called Group Tree of the match, which reflects the hierarchical nature of the groups. E.g. in this case, the term (6*[6+7+8]+9)\*78\*[4*(6+5)+4] is a product which consists of three factors. The first one of these factors is a sum and so on...

The following code

```java
String regex = IOUtils.toString(
				new FileInputStream("term.regex"), "UTF-8");
Pattern p = Pattern.compile(regex);

String term = "(6*[6+7+8]+9)*78*[4*(6+5)+4]";

System.out.println("You see the term tree for: " + term);
Matcher m = p.matcher(term);
assertTrue(m.matches());
System.out.println(m.groupTree());
```

produces the output seen in the following screenshot:
![a term tree](media/termtree.png)

### Maven Dependency
In order to use this library, add the following dependency to your pom.xml.
```
<dependency>
	<groupId>com.github.florianingerl.util</groupId>
	<artifactId>regex</artifactId>
	<version>1.1.1</version>
</dependency>
```

### Known Issues
Unfortunately this library needs more stacks than java.util.regex which can lead to a StackOverflowException more quickly in rare cases.
E.g. suppose you wanted to match Java strings with the regex 
```
"(\\.|[^"])*"
```
then this would only work for string lengths up to 3890, whereas with java.util.regex it would work with string lengths up to 6930. The problem is the
*-repetition that has to keep track of group captures and the backtracking options of the alternation. However a simple character class can be repeated nearly an unlimited number of times,
so the regex above could be improved to
```
"(?:\\.|[^"\\]+)*"
```
