---
title:  关于Java 正则表达式的介绍
description: 我们将讨论Java Regex API以及如何在Java编程语言中使用正则表达式。
...


在本文中，我们将讨论Java Regex API以及如何在Java编程语言中使用正则表达式。 在正则表达式的世界中，有许多不同的风格可供选择，例如grep，Perl，Python，PHP，awk等等。 这意味着在一种编程语言中工作的正则表达式可能在另一种编程语言中不起作用。 Java中的正则表达式语法与Perl中的语法最相似。

> 这篇文章大部分内容只是针对： https://www.baeldung.com/regular-expressions-java 进行了翻译； Chrome的插件翻译的近乎完美，大部分的翻译都是直接来源于此。

要在Java中使用正则表达式，我们不需要任何特殊设置。 JDK包含一个特殊的包java.util.regex，完全专用于正则表达式操作。我们只需要将它导入我们的代码中。 此外，java.lang.String类还具有内置的正则表达式支持，我们通常在代码中使用它们

## Java 正则表达式包
java.util.regex包由三个类组成：Pattern，Matcher和PatternSyntaxException： 
- Pattern对象是一个已编译的正则表达式。 Pattern类不提供公共构造函数。要创建模式，我们必须首先调用其公共静态编译方法之一，然后返回Pattern对象。这些方法接受正则表达式作为第一个参数。 
- Matcher对象解释模式并对输入String执行匹配操作。它还定义了没有公共构造函数。我们通过在Pattern对象上调用matcher方法来获取Matcher对象。 
- PatternSyntaxException对象是未经检查的异常，表示正则表达式模式中的语法错误。

## 简单示例来理解如何应用
```
@Test
public void givenText_whenSimpleRegexMatches_thenCorrect() {
    Pattern pattern = Pattern.compile("foo");
    Matcher matcher = pattern.matcher("foo");
  
    assertTrue(matcher.find());
}
```
我们首先通过调用静态编译方法创建一个Pattern对象，并将它传递给我们想要使用的模式。 然后我们创建一个Matcher对象，调用Pattern对象的matcher方法并将它传递给我们要检查匹配的文本。 之后，我们在Matcher对象中调用方法find。 

find方法不断推进输入文本并为每个匹配返回true，因此我们也可以使用它来查找匹配计数：
```
@Test
public void givenText_whenSimpleRegexMatchesTwice_thenCorrect() {
    Pattern pattern = Pattern.compile("foo");
    Matcher matcher = pattern.matcher("foofoo");
    int matches = 0;
    while (matcher.find()) {
        matches++;
    }
  
    assertEquals(matches, 2);
}
```
由于我们将运行更多测试，因此我们可以在名为runTest的方法中抽象查找匹配数的逻辑：
```
public static int runTest(String regex, String text) {
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(text);
    int matches = 0;
    while (matcher.find()) {
        matches++;
    }
    return matches;
}
```

## 元字符
元字符会影响模式的匹配方式，从而为搜索模式添加逻辑。 Java API支持几个元字符，最直接的是匹配任何字符的点“.”：
```
@Test
public void givenText_whenMatchesWithDotMetach_thenCorrect() {
    int matches = runTest(".", "foo");
     
    assertTrue(matches > 0);
}
```
考虑前面的例子，其中正则表达式foo与文本foo以及foofoo匹配两次。如果我们在正则表达式中使用点元字符，我们在第二种情况下不会得到两个匹配：
```
@Test
public void givenRepeatedText_whenMatchesOnceWithDotMetach_thenCorrect() {
    int matches= runTest("foo.", "foofoo");
  
    assertEquals(matches, 1);
}
```
注意正则表达式中foo之后的点。匹配器匹配前面有foo的每个文本，因为最后一个点部分表示后面的任何字符。所以在找到第一个foo之后，其余部分被视为任何角色。这就是为什么只有一个匹配。 API支持其他几个元字符`<（[{\ ^ - = $！|]}）？* +。>`，我们将在本文中进一步研究。

## 字符类
浏览官方Pattern类规范，我们将发现支持的正则表达式结构的摘要。在字符类下，我们有大约6个构造。

### OR类
构造为[abc]。集合中的任何元素都匹配：
```
@Test
public void givenORSet_whenMatchesAny_thenCorrect() {
    int matches = runTest("[abc]", "b");
  
    assertEquals(matches, 1);
}
```
如果它们都出现在文本中，则每个都是单独匹配而不考虑顺序：
```
@Test
public void givenORSet_whenMatchesAnyAndAll_thenCorrect() {
    int matches = runTest("[abc]", "cab");
  
    assertEquals(matches, 3);
}
```
它们也可以作为String的一部分进行交替。在下面的示例中，当我们通过将第一个字母与集合中的每个元素交替来创建不同的单词时，它们都匹配：
```
@Test
public void givenORSet_whenMatchesAllCombinations_thenCorrect() {
    int matches = runTest("[bcr]at", "bat cat rat");
  
    assertEquals(matches, 3);
}
```
### NOR 类
通过添加插入符号`^`作为第一个元素来取消上面的设置：
```
@Test
public void givenNORSet_whenMatchesNon_thenCorrect() {
    int matches = runTest("[^abc]", "g");
  
    assertTrue(matches > 0);
}
```
Another case:
```
@Test
public void givenNORSet_whenMatchesAllExceptElements_thenCorrect() {
    int matches = runTest("[^bcr]at", "sat mat eat");
  
    assertTrue(matches > 0);
}
```
### Range 类
我们可以定义一个类，使用连字符（ - ）指定匹配文本应该落在的范围内，同样，我们也可以否定范围。

匹配大写的英文字母:
```
@Test
public void givenUpperCaseRange_whenMatchesUpperCase_
  thenCorrect() {
    int matches = runTest(
      "[A-Z]", "Two Uppercase alphabets 34 overall");
  
    assertEquals(matches, 2);
}
```
匹配小写的英文字母:
```
@Test
public void givenLowerCaseRange_whenMatchesLowerCase_
  thenCorrect() {
    int matches = runTest(
      "[a-z]", "Two Uppercase alphabets 34 overall");
  
    assertEquals(matches, 26);
}
```
匹配大小写的英文字母:
```
@Test
public void givenBothLowerAndUpperCaseRange_
  whenMatchesAllLetters_thenCorrect() {
    int matches = runTest(
      "[a-zA-Z]", "Two Uppercase alphabets 34 overall");
  
    assertEquals(matches, 28);
}
```
匹配给定范围的数字：
```
@Test
public void givenNumberRange_whenMatchesAccurately_
  thenCorrect() {
    int matches = runTest(
      "[1-5]", "Two Uppercase alphabets 34 overall");
  
    assertEquals(matches, 2);
}
```
匹配另一个数字范围：

```
@Test
public void givenNumberRange_whenMatchesAccurately_
  thenCorrect2(){
    int matches = runTest(
      "[30-35]", "Two Uppercase alphabets 34 overall");
  
    assertEquals(matches, 1);
}
```
### Union 类

联合字符类是组合两个或多个字符类的结果：
```
@Test
public void givenTwoSets_whenMatchesUnion_thenCorrect() {
    int matches = runTest("[1-3[7-9]]", "123456789");
  
    assertEquals(matches, 6);
}
```
上面的测试只匹配9个整数中的6个，因为联合集跳过了3,4和5。

### Intersection 类
与union类相似，此类是从两个或多个集合之间选择公共元素得到的。要应用交集，我们使用&&：
```
@Test
public void givenTwoSets_whenMatchesIntersection_thenCorrect() {
    int matches = runTest("[1-6&&[3-9]]", "123456789");
  
    assertEquals(matches, 4);
}
```
我们得到4个匹配，因为两个集合的交集只有4个元素。

### Subtraction Class
我们可以使用减法来否定一个或多个字符类，例如匹配一组奇数十进制数：

```
@Test
public void givenSetWithSubtraction_whenMatchesAccurately_thenCorrect() {
    int matches = runTest("[0-9&&[^2468]]", "123456789");
  
    assertEquals(matches, 5);
}
```
只有1,3,5,7,9匹配。

## 预定义字符类
Java正则表达式API也接受预定义的字符类。上述某些字符类可以用较短的形式表示，但使代码不太直观。这个正则表达式的Java版本的一个特殊方面是转义字符。 正如我们将看到的，大多数字符将以反斜杠开头，这在Java中具有特殊含义。要由Pattern类编译这些 - 必须转义前导反斜杠，即`\ d`变为`\\ d`。 匹配数字，相当于[0-9]：

匹配数字，相当于[0-9]：

```
@Test
public void givenDigits_whenMatches_thenCorrect() {
    int matches = runTest("\\d", "123");
  
    assertEquals(matches, 3);
}
```
匹配非数字，相当于[^ 0-9]：

```
@Test
public void givenNonDigits_whenMatches_thenCorrect() {
    int mathces = runTest("\\D", "a6c");
  
    assertEquals(matches, 2);
}
```
匹配空白区域：

```
@Test
public void givenWhiteSpace_whenMatches_thenCorrect() {
    int matches = runTest("\\s", "a c");
  
    assertEquals(matches, 1);
}
```
匹配非白色空间：
```
@Test
public void givenNonWhiteSpace_whenMatches_thenCorrect() {
    int matches = runTest("\\S", "a c");
  
    assertEquals(matches, 2);
}
```
匹配单词字符，相当于[a-zA-Z_0-9]：
```
@Test
public void givenWordCharacter_whenMatches_thenCorrect() {
    int matches = runTest("\\w", "hi!");
  
    assertEquals(matches, 2);
}
```
匹配非单词字符

```
@Test
public void givenNonWordCharacter_whenMatches_thenCorrect() {
    int matches = runTest("\\W", "hi!");
  
    assertEquals(matches, 1);
}
```

## 量词
Java正则表达式API还允许我们使用量词。这使我们能够通过指定要匹配的出现次数来进一步调整匹配的行为。

为了匹配文本零或一次，我们使用？量词：
```
@Test
public void givenZeroOrOneQuantifier_whenMatches_thenCorrect() {
    int matches = runTest("\\a?", "hi");
  
    assertEquals(matches, 3);
}
```
或者，我们可以使用Java正则表达式API支持的大括号语法：

```
@Test
public void givenZeroOrOneQuantifier_whenMatches_thenCorrect2() {
    int matches = runTest("\\a{0,1}", "hi");
  
    assertEquals(matches, 3);
}
```
此示例介绍了零长度匹配的概念。碰巧的是，如果量词的匹配阈值为零，它总是匹配文本中的所有内容，包括每个输入末尾的空字符串。这意味着即使输入为空，它也将返回一个零长度匹配。 

这解释了为什么我们在上面的例子中得到3个匹配，尽管有一个长度为2的字符串。第三个匹配是零长度的空字符串。 

为了匹配文本零或无限次，我们使用`*`量词，它类似于`？`：

```
@Test
public void givenZeroOrManyQuantifier_whenMatches_thenCorrect() {
     int matches = runTest("\\a*", "hi");
  
     assertEquals(matches, 3);
}
```
同等效果的另外一种写法：

```
@Test
public void givenZeroOrManyQuantifier_whenMatches_thenCorrect2() {
    int matches = runTest("\\a{0,}", "hi");
  
    assertEquals(matches, 3);
}
```
具有差异的量词是+，它具有匹配阈值1.如果根本不发生所需的字符串，则不会匹配，甚至不是零长度字符串：

```
@Test
public void givenOneOrManyQuantifier_whenMatches_thenCorrect() {
    int matches = runTest("\\a+", "hi");
  
    assertFalse(matches);
}
```
同等效果的另外一种写法：

```
@Test
public void givenOneOrManyQuantifier_whenMatches_thenCorrect2() {
    int matches = runTest("\\a{1,}", "hi");
  
    assertFalse(matches);
}
```
与Perl和其他语言一样，大括号语法可用于多次匹配给定文本：

```
@Test
public void givenBraceQuantifier_whenMatches_thenCorrect() {
    int matches = runTest("a{3}", "aaaaaa");
  
    assertEquals(matches, 2);
}
```
在上面的示例中，我们得到两个匹配项，因为匹配仅在连续出现三次时才会发生。但是，在下一个测试中，我们不会得到匹配，因为文本只连续出现两次：

```
@Test
public void givenBraceQuantifier_whenFailsToMatch_thenCorrect() {
    int matches = runTest("a{3}", "aa");
  
    assertFalse(matches > 0);
}
```
当我们在大括号中使用一个范围时，匹配将是贪婪的，从范围的较高端匹配：
```
@Test
public void givenBraceQuantifierWithRange_whenMatches_thenCorrect() {
    int matches = runTest("a{2,3}", "aaaa");
  
    assertEquals(matches, 1);
}
```
我们已经指定了至少两次但不超过三次，所以我们得到一个匹配，而匹配器看到一个aaa和一个无法匹配的孤独aa。

但是，API允许我们指定一个懒惰或不情愿的方法，以便匹配器可以从范围的下端开始，在这种情况下匹配两次出现为aa和aa：

```
@Test
public void givenBraceQuantifierWithRange_whenMatchesLazily_thenCorrect() {
    int matches = runTest("a{2,3}?", "aaaa");
  
    assertEquals(matches, 2);
}
```

## 捕获组
API还允许我们通过捕获组将多个字符视为一个单元。 它会将数字附加到捕获组，并允许使用这些数字进行反向引用。 在本节中，我们将看到一些关于如何在Java regex API中使用捕获组的示例。 让我们使用仅当输入文本包含彼此相邻的两个数字时才匹配的捕获组：

```
@Test
public void givenCapturingGroup_whenMatches_thenCorrect() {
    int maches = runTest("(\\d\\d)", "12");
  
    assertEquals(matches, 1);
}
```
附加到上面匹配的数字是1，使用后向引用告诉匹配器我们要匹配文本的匹配部分的另一个出现。

```
@Test
public void givenCapturingGroup_whenMatches_thenCorrect2() {
    int matches = runTest("(\\d\\d)", "1212");
  
    assertEquals(matches, 2);
}
```
对于输入有两个单独的匹配，我们可以有一个匹配但传播相同的正则表达式匹配以使用反向引用跨越输入的整个长度：

```
@Test
public void givenCapturingGroup_whenMatchesWithBackReference_
  thenCorrect() {
    int matches = runTest("(\\d\\d)\\1", "1212");
  
    assertEquals(matches, 1);
}
```
我们必须在没有反向引用的情况下重复正则表达式以获得相同的结果：

```
@Test
public void givenCapturingGroup_whenMatches_thenCorrect3() {
    int matches = runTest("(\\d\\d)(\\d\\d)", "1212");
  
    assertEquals(matches, 1);
}
```
类似地，对于任何其他重复次数，反向引用可以使匹配器将输入视为单个匹配：

```
@Test
public void givenCapturingGroup_whenMatchesWithBackReference_
  thenCorrect2() {
    int matches = runTest("(\\d\\d)\\1\\1\\1", "12121212");
  
    assertEquals(matches, 1);
}
```
但如果你改变了哪怕最后一位数，那么匹配就会失败：
```
@Test
public void givenCapturingGroupAndWrongInput_whenMatchFailsWithBackReference_thenCorrect() {
    int matches = runTest("(\\d\\d)\\1", "1213");
  
    assertFalse(matches > 0);
}
```
重要的是不要忘记转义反斜杠，这在Java语法中至关重要。
> 捕获组更有价值的地方在于通过Matcher 的 group()方法来后去捕获到的字符串；详情可以参考：  http://www.runoob.com/w3cnote/java-capture-group.html

## 边界匹配
Java正则表达式API还支持边界匹配。如果我们关心匹配应该在输入文本中的确切位置，那么这就是我们正在寻找的。在前面的例子中，我们关心的是是否找到匹配。 要仅在文本开头所需的正则表达式为真时匹配，我们使用插入符号^。 此测试将成功，因为可以在开头找到文本狗：
```
@Test
public void givenText_whenMatchesAtBeginning_thenCorrect() {
    int matches = runTest("^dog", "dogs are friendly");
  
    assertTrue(matches > 0);
}
```
下面的将失败：
```
@Test
public void givenTextAndWrongInput_whenMatchFailsAtBeginning_
  thenCorrect() {
    int matches = runTest("^dog", "are dogs are friendly?");
  
    assertFalse(matches > 0);
}
```
要仅在文本末尾所需的正则表达式为真时匹配，我们使用美元字符$。在以下情况中将找到匹配：
```
@Test
public void givenTextAndWrongInput_whenMatchFailsAtEnd_thenCorrect() {
    int matches = runTest("dog$", "is a dog man's best friend?");
  
    assertFalse(matches > 0);
}
```
如果我们只想在单词边界找到所需的文本时匹配，我们在正则表达式的开头和结尾使用\\ b正则表达式：
```
@Test
public void givenText_whenMatchesAtWordBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\b", "a dog is friendly");
  
    assertTrue(matches > 0);
}
```
空格是一个词边界：
```
@Test
public void givenText_whenMatchesAtWordBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\b", "a dog is friendly");
  
    assertTrue(matches > 0);
}
```
一行开头的空字符串也是一个单词边界：
```
@Test
public void givenText_whenMatchesAtWordBoundary_thenCorrect2() {
    int matches = runTest("\\bdog\\b", "dog is man's best friend");
  
    assertTrue(matches > 0);
}
```
这些测试通过，因为String的开头以及一个文本和另一个文本之间的空格标记了单词边界，但是，以下测试显示相反的结果：
```
@Test
public void givenWrongText_whenMatchFailsAtWordBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\b", "snoop dogg is a rapper");
  
    assertFalse(matches > 0);
}
```

连续出现的双字符不标记单词边界，但我们可以通过更改正则表达式的结尾来查找非单词边界：
```
@Test
public void givenText_whenMatchesAtWordAndNonBoundary_thenCorrect() {
    int matches = runTest("\\bdog\\B", "snoop dogg is a rapper");
    assertTrue(matches > 0);
}
```

## Pattern Class Methods
之前，我们只以基本方式创建了Pattern对象。但是，此类具有另一种编译方法，它接受一组标志以及影响模式匹配方式的正则表达式参数。 这些标志只是抽象的整数值。让我们重载测试类中的runTest方法，以便它可以将标志作为第三个参数：

```
public static int runTest(String regex, String text, int flags) {
    pattern = Pattern.compile(regex, flags);
    matcher = pattern.matcher(text);
    int matches = 0;
    while (matcher.find()){
        matches++;
    }
    return matches;
}
```
在本节中，我们将查看不同的受支持标志及其使用方式。

### Pattern.CANON_EQ

此标志启用规范等效。如果指定，当且仅当它们的完整规范分解匹配时，才会认为两个字符匹配。

考虑重音字符é。它的复合Unicode代码是u00E9。但是，Unicode还为其组件字符`e`(u0065)和急性重音u0301分别设置了一个代码点。在这种情况下，复合字符u00E9与两个字符序列u0065 u0301无法区分。在http://tool.chinaz.com/tools/unicode.aspx 输入Unicode: `\u00E9`和`\u0065\u0301`转化后是同一个字符

默认情况下，匹配不会将规范等效考虑在内

```
@Test
public void givenRegexWithoutCanonEq_whenMatchFailsOnEquivalentUnicode_thenCorrect() {
    int matches = runTest("\u00E9", "\u0065\u0301");
  
    assertFalse(matches > 0); // 注意这里是assertFalse
}
```
但是如果我们添加标志，那么测试将通过:

```
@Test
public void givenRegexWithCanonEq_whenMatchesOnEquivalentUnicode_thenCorrect() {
    int matches = runTest("\u00E9", "\u0065\u0301", Pattern.CANON_EQ);
  
    assertTrue(matches > 0);
}
```
### Pattern.CASE_INSENSITIVE

该标志无论大小写都能够进行匹配。默认情况下，匹配将案例考虑在内：

```
@Test
public void givenRegexWithDefaultMatcher_whenMatchFailsOnDifferentCases_thenCorrect() {
    int matches = runTest("dog", "This is a Dog");
  
    assertFalse(matches > 0);
}
```
因此，使用此标志，我们可以更改默认行为：

```
@Test
public void givenRegexWithCaseInsensitiveMatcher_whenMatchesOnDifferentCases_thenCorrect() {
    int matches = runTest(
      "dog", "This is a Dog", Pattern.CASE_INSENSITIVE);
  
    assertTrue(matches > 0);
}
```
我们还可以使用等效的嵌入式标志表达式来实现相同的结果：
```
@Test
public void givenRegexWithEmbeddedCaseInsensitiveMatcher_whenMatchesOnDifferentCases_thenCorrect() {
    int matches = runTest("(?i)dog", "This is a Dog");
  
    assertTrue(matches > 0);
}
```
### Pattern.COMMENTS

TJava API允许在正则表达式中使用＃包含注释。这有助于记录复杂的正则表达式，这对于另一个程序员来说可能并不是很明显。 comments标志使匹配器忽略正则表达式中的任何空格或注释，只考虑模式。在默认匹配模式下，以下测试将失败： 

```
@Test
public void givenRegexWithComments_whenMatchFailsWithoutFlag_thenCorrect() {
    int matches = runTest(
      "dog$  #check for word dog at end of text", "This is a dog");
  
    assertFalse(matches > 0);
}
```
这是因为匹配器将在输入文本中查找整个正则表达式，包括空格和＃字符。但是当我们使用该标志时，它将忽略额外的空格，并且以＃开头的每个文本将被视为每行忽略的注释：

```
@Test
public void givenRegexWithComments_whenMatchesWithFlag_thenCorrect() {
    int matches = runTest(
      "dog$  #check end of text","This is a dog", Pattern.COMMENTS);
  
    assertTrue(matches > 0);
}
```
还有一个替代的嵌入式标志表达式：

```
@Test
public void givenRegexWithComments_whenMatchesWithEmbeddedFlag_thenCorrect() {
    int matches = runTest(
      "(?x)dog$  #check end of text", "This is a dog");
  
    assertTrue(matches > 0);
}
```
### Pattern.DOTALL
默认情况下，当我们在regex中使用点“.”表达式时，我们匹配输入String中的每个字符，直到遇到新的行字符。 使用此标志，匹配也将包括行终止符。

我们将通过以下示例更好地理解。这些例子会有所不同。由于我们对匹配的String断言感兴趣，因此我们将使用matcher的group方法返回上一个匹配项。 首先，我们将看到默认行为：

```
@Test
public void givenRegexWithLineTerminator_whenMatchFails_thenCorrect() {
    Pattern pattern = Pattern.compile("(.*)");
    Matcher matcher = pattern.matcher(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line");
    matcher.find();
  
    assertEquals("this is a text", matcher.group(1));
}
```
我们可以看到，只有行终止符之前输入的第一部分匹配。 

下面的示例处于dotall模式，包括行终止符的整个文本将匹配：

```
@Test
public void givenRegexWithLineTerminator_whenMatchesWithDotall_thenCorrect() {
    Pattern pattern = Pattern.compile("(.*)", Pattern.DOTALL);
    Matcher matcher = pattern.matcher(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line");
    matcher.find();
    assertEquals(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line", matcher.group(1));
}
```
我们还可以使用嵌入式标志表达式来启用dotall模式：

```
@Test
public void givenRegexWithLineTerminator_whenMatchesWithEmbeddedDotall_thenCorrect() {
     
    Pattern pattern = Pattern.compile("(?s)(.*)");
    Matcher matcher = pattern.matcher(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line");
    matcher.find();
  
    assertEquals(
      "this is a text" + System.getProperty("line.separator") 
        + " continued on another line", matcher.group(1));
}
```
### Pattern.LITERAL

在此模式下，matcher对任何元字符，转义字符或正则表达式语法没有特殊含义。如果没有此标志，匹配器将对任何输入字符串匹配以下正则表达式：

```
@Test
public void givenRegex_whenMatchFailsWithLiteralFlag_thenCorrect() {
    int matches = runTest("(.*)", "text", Pattern.LITERAL);
  
    assertFalse(matches > 0);
}
```
现在，如果我们添加所需的字符串，测试将通过：

```
@Test
public void givenRegex_whenMatchesWithLiteralFlag_thenCorrect() {
    int matches = runTest("(.*)", "text(.*)", Pattern.LITERAL);
  
    assertTrue(matches > 0);
}
```
没有用于启用文字解析的嵌入标志字符。

### Pattern.MULTILINE

默认情况下，^和$ metacharacters分别在整个输入String的开头和结尾处匹配。匹配器忽略任何行终止符：
```
@Test
public void givenRegex_whenMatchFailsWithoutMultilineFlag_thenCorrect() {
    int matches = runTest(
      "dog$", "This is a dog" + System.getProperty("line.separator") 
      + "this is a fox");
  
    assertFalse(matches > 0);
}
```
上面匹配失败，因为匹配器在整个String的末尾搜索dog，但是狗出现在字符串第一行的末尾。 

但是，使用该标志，相同的测试将通过，因为匹配器现在考虑了行终止符。所以String line就在行终止之前找到，因此成功：
```
@Test
public void givenRegex_whenMatchesWithMultilineFlag_thenCorrect() {
    int matches = runTest(
      "dog$", "This is a dog" + System.getProperty("line.separator") 
      + "this is a fox", Pattern.MULTILINE);
  
    assertTrue(matches > 0);
}
```
这是嵌入式标志版本：

```
@Test
public void givenRegex_whenMatchesWithEmbeddedMultilineFlag_
  thenCorrect() {
    int matches = runTest(
      "(?m)dog$", "This is a dog" + System.getProperty("line.separator") 
      + "this is a fox");
  
    assertTrue(matches > 0);
}
```
## Matcher Class Methods
在本节中，我们将介绍Matcher类的一些有用方法。我们将根据功能对它们进行分组以便清晰。
   
###  Index Methods
索引方法提供有用的索引值，可以精确显示在输入String中找到匹配的位置。在下面的测试中，我们将在输入String中确认dog匹配的开始和结束索引：

```
@Test
public void givenMatch_whenGetsIndices_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher("This dog is mine");
    matcher.find();
  
    assertEquals(5, matcher.start());
    assertEquals(8, matcher.end());
}
```
###  Methods match and lookingAt  
研究方法遍历输入String并返回一个布尔值，指示是否找到该模式。常用的是match和lookingAt方法。 matches和lookingAt方法都尝试将输入序列与模式匹配。不同之处在于，匹配需要匹配整个输入序列，而查找则不需要。

Both methods start at the beginning of the input String :

```
@Test
public void whenStudyMethodsWork_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher("dogs are friendly");
  
    assertTrue(matcher.lookingAt());
    assertFalse(matcher.matches());
}
```
两种方法都从输入String的开头开始：

```
@Test
public void whenMatchesStudyMethodWorks_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher("dog");
  
    assertTrue(matcher.matches());
}
```

###  Replacement Methods
替换方法对于替换输入字符串中的文本很有用。常见的是replaceFirst和replaceAll。 replaceFirst和replaceAll方法替换匹配给定正则表达式的文本。正如其名称所示，replaceFirst替换第一个匹配项，replaceAll替换所有匹配项：

```
@Test
public void whenReplaceFirstWorks_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher(
      "dogs are domestic animals, dogs are friendly");
    String newStr = matcher.replaceFirst("cat");
  
    assertEquals(
      "cats are domestic animals, dogs are friendly", newStr);
}
```
替换所有匹配项：
```
@Test
public void whenReplaceAllWorks_thenCorrect() {
    Pattern pattern = Pattern.compile("dog");
    Matcher matcher = pattern.matcher(
      "dogs are domestic animals, dogs are friendly");
    String newStr = matcher.replaceAll("cat");
  
    assertEquals("cats are domestic animals, cats are friendly", newStr);
}
```