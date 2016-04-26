---
layout: post
title: 最长回文
subtitle: 最长回文子串 & 最长回文子序列
---

## Problem 1
输入一个字符串，找出其中的最长回文`子串`

## Solution 1  
* 子串（substring）意味着所求结果必须是原字符串中连续的部分  
* 以字符串的每一个字符为中心，向两边扩展，得出以每一个字符为中心时的最长回文子串  
* 遍历到某字符时，若其后连续的若干字符与该字符相等，则扩展中心为这连续相等的若干字符，再向两边扩展  
* 本方法无需区分对称轴是奇数个还是偶数个字符  
* 时间复杂度O(n^2)

{% highlight javascript %}
void LongestPalindromicSubstring(string& str) {
  int center = 0, begin = 0, next = 0, max_length = 0;
  int n = str.size();
  while (2 * (n - center) - 1 > max_length) { // 可以提前结束循环
    while (next < n && str[center] == str[next]) ++next;
    int i = center - 1, j = next;
    while (i >= 0 && j < n && str[i] == str[j]) {
      --i;
      ++j;
    }
    if (j - i - 1 > max_length) {
      max_length = j - i - 1;
      begin = i + 1;
    }
    center = next;
    next = center + 1;
  }
  cout << "longest palindromic substring: " 
       << str.substr(begin, max_length) << endl;
}
{% endhighlight %}

**测试程序与结果**

{% highlight javascript %}
#include <iostream>
#include <string>
using namespace std;

int main() {
  string str("abcddcbefgfejk");
  LongestPalindromicSubstring(str);
  return 0;
}
{% endhighlight %}

~~~
longest palindromic substring: bcddcb
~~~


## Problem 2
输入一个字符串，找出其中的最长回文`子序列`  

## Solution 2  
* 子序列（subsequence）意味着所求结果可以是不连续的  
* f(i, j)表示从str[i]到str[j]所能形成的最长回文子序列的长度，则`f(i, j) = str[i] == str[j] ? f(i + 1, j - 1) + 2 : max(f(i, j - 1), f(i + 1, j))`，最终的结果就是f(0, n - 1)  
* 因为有重复的子问题，所以用DP来求解，区间的长度由小到大计算 

{% highlight javascript %}
void LongestPalindromicSubsequence(string& str) {
  int n = str.size();
  vector<vector<int>> dp(n, vector<int>(n, 0)); // dp[i][j] = 0, i > j
  for (int i = 0; i < n; ++i)
    dp[i][i] = 1;
  for (int d = 1; d < n; ++d) {
    for (int i = 0; i + d < n; ++i) {
      int j = i + d;
      dp[i][j] = str[i] == str[j] ? dp[i + 1][j - 1] + 2 :
                 max(dp[i][j - 1], dp[i + 1][j]);
    }
  }
  cout << "longest length: " << dp[0][n - 1] << endl;
  cout << "longest palindromic subsequence: ";
  vector<bool> used(n, false);
  int i = 0, j = n - 1;
  while (i <= j) {
    if (str[i] == str[j]) used[i++] = used[j--] = true;
    else if (dp[i][j - 1] > dp[i + 1][j]) --j;
    else ++i;
  }
  for (int i = 0; i < n; ++i) {
    if (used[i]) cout << str[i] << " ";
  }
  cout << endl;
}
{% endhighlight %}


**测试程序与结果**  
{% highlight javascript %}
#include <iostream>
#include <string>
using namespace std;

int main() {
  string str("abcfccdcdbaef");
  LongestPalindromicSubsequence(str);
  return 0;
}
{% endhighlight %}

~~~
longest length: 8
longest palindromic subsequence: a b c c c c b a 
~~~




