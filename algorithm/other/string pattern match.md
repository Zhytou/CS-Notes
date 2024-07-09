# 字符串匹配问题

- [字符串匹配问题](#字符串匹配问题)
  - [KMP算法](#kmp算法)
  - [Boyer–Moore算法](#boyermoore算法)
  - [参考](#参考)

字符串匹配问题，又称模式匹配(Pattern Matching)。它可以概括为给定字符串S和T，在主串S中寻找子串T。其中，字符串T称为模式串。

这种问题最简单的做法就是暴力算法。其基本思想是从主串S的第一个字符开始和模式串T的第一个字符进行比较，若相等，则继续比较二者的后续字符；否则，模式串T回退到第一个字符，重新和主串S的第二个字符进行比较。如此往复，直到S或T中所有字符比较完毕。

## KMP算法

然而，暴力算法的时间复杂度较高。为了更高效地解决字符串匹配问题，Knuth、Morris和Pratt在1977年提出了KMP算法。KMP算法通过构建模式串的前缀表，在匹配过程中可以跳过一些不可能匹配的位置，从而提高匹配效率。

具体来说，暴力匹配的问题在于它一次只能将子串T整个后移一位，再从头逐个比较。这样做虽然可行，但是效率很差，因为你要把"搜索位置"移到已经比较过的位置，重比一遍。而KMP算法改进了这一点，它通过提前建立子串T的部分匹配表(Partial Match Table)来跳过一些不可能匹配的位置。比如，主串为"BBC ABCDAB ABCDABCDABDE"，而子串为"ABCDABD"。

首先，建立它的部分匹配表如下图。

![子串部分匹配表](https://www.ruanyifeng.com/blogimg/asset/201305/bg2013050109.png)

接着，移动搜索位置依次匹配子串。下图表示搜索位置为4时，子串匹配上了6位。

![子串部分匹配](https://www.ruanyifeng.com/blogimg/asset/201305/bg2013050107.png)

已知空格与D不匹配时，前面六个字符"ABCDAB"是匹配的。查表可知，最后一个匹配字符B对应的"部分匹配值"为2，因此按照下面的公式算出向后移动的位数：6-2，即将搜索位置移到8。

![子串部分匹配](https://www.ruanyifeng.com/blogimg/asset/201305/bg2013050110.png)

部分匹配值其实是前缀和后缀的最长的共有元素的长度。以"ABCDABD"为例，

- "A"的前缀和后缀都为空集，共有元素的长度为0；
- "AB"的前缀为[A]，后缀为[B]，共有元素的长度为0；
- "ABC"的前缀为[A, AB]，后缀为[BC, C]，共有元素的长度0；
- "ABCD"的前缀为[A, AB, ABC]，后缀为[BCD, CD, D]，共有元素的长度为0；
- "ABCDA"的前缀为[A, AB, ABC, ABCD]，后缀为[BCDA, CDA, DA, A]，共有元素为"A"，长度为1；
- "ABCDAB"的前缀为[A, AB, ABC, ABCD, ABCDA]，后缀为[BCDAB, CDAB, DAB, AB, B]，共有元素为"AB"，长度为2；
- "ABCDABD"的前缀为[A, AB, ABC, ABCD, ABCDA, ABCDAB]，后缀为[BCDABD, CDABD, DABD, ABD, BD, D]，共有元素的长度为0。

## Boyer–Moore算法

## 参考

[字符串匹配](https://oi-wiki.org/string/match/)
