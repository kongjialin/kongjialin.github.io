---
layout: post
title: 最长等差数列
subtitle: Longest Arithmetic Progression
---

## Problem 1
给定一个无序数组，求其中的最长的等差数列（子序列）的长度，并输出该等差数列。  
要求子序列中各元素的相对位置与原数组相同，如{1, 7, 3, 9, 11, 5, 13}，最长等差数列长度为4，数列为7, 9, 11, 13

## Solution 1  
* 最先应该想到暴力的方法，枚举所有的元素对［O(n^2)］，并以各元素对作为等差数列的前两个元素，然后依次尝试后面的每个元素［O(n)］，使该等差数列尽可能的长。最终得出最长的数列长度，总的复杂度为O(n^3)。  
* 从暴力的解法中可以发现，有许多不必要的计算，如数组{1, x, 3, x, 5, x, 7}，如果在处理数对{3, 5}时，已经找到数列{3, 5, 7}，那么在访问数对{1, 3}时，就不需要再对后面的数字进行O(n)的重新扫描，而是直接得到长度为3 + 1 = 4，从而降低复杂度。那么就需要额外的空间对已求得的结果进行缓存，f(i, d)表示以arr[i]作为开头，等差值为d的序列的最大长度，则`f(i, d) = f(j, d) + 1, arr[j] == arr[i] + d`，在枚举到数对{arr[i], arr[j]}时，递推关系可以写为`f(i, arr[j] - arr[i]) = f(j, arr[j] - arr[i]) + 1`。  
* 遍历数对的起点i是从后往前，这样可以利用已求得的结果。对于每一个i，遍历数对的另一个值j，也需要从后往前，因为有可能出现覆盖的情况：例如j1 < j2而arr[j1] == arr[j2]，那么j1和j2之间的几个数，有可能会导致f(j1, arr[j1] - arr[i]) > f(j2, arr[j1] - arr[i])，因此先访问{i, j2}，后访问{i, j1}，可以保证求得的f(i, d)是更大的那一个，如果访问的顺序倒过来，就会出现小的值把大的值覆盖的情况。   
* 此解法其实也是DP，即dp[i][d]，但是由于d = arr[j] - arr[i]可能是任何值，可能非常大，因此使用二维数组来表示dp[i][d]可能会用到非常大的空间（除非题目对d有明确的大小限制）。于是可以采用hash map，hash map的key为{i, d}组成的pair，value为最大长度，即f(i, d)。这样，求f(i, d)时，从hash map中查询{j, d}，如果不存在，则f(i, d) = 2，如果存在，则f(i, d) = f(j, d) + 1。枚举数对{arr[i], arr[j]}的复杂度为O(n^2)，插入{i, d}和查询{j, d}是O(1)的，总的时间复杂度为O(n^2)。    
* 本题中使用的hash map的key是pair，c++中pair已经重载了==运算符，可以解决hash冲突时寻找目标元素的问题，但是并没有关于pair的默认hash，所以需要自定义一个hash类。
* 要输出最长的等差数列，除了记录最长的长度，还需记录相对应的差值，以及该数列的第一个元素。  
* 注：之前有尝试过枚举arr[i], arr[j]时以arr[j] - arr[i]作为hash map的key，value是一个pair，但是无论value是arr[i]和arr[j]的pair还是i和j的pair，都无法有效地对建好的hash map进行处理找出最长等差序列，因此放弃这种做法。

{% highlight javascript %}
struct PairHash {
  size_t operator()(const pair<int, int>& p) const {
    size_t h1 = hash<int>()(p.first);
    size_t h2 = hash<int>()(p.second);
    return h1 ^ (h2 << 1);
  }
};

void LongestArithmeticProgression(vector<int>& arr) {
  int n = arr.size();
  if (n <= 2) return;
  unordered_map<pair<int, int>, int, PairHash> dp;
  int max_length = 2, diff = arr[1] - arr[0], first_num = arr[0];
  for (int i = n - 2; i >= 0; --i) {
    for (int j = n - 1; j > i; --j) {
      pair<int, int> current(i, arr[j] - arr[i]);
      pair<int, int> previous(j, arr[j] - arr[i]);
      if (dp.find(previous) == dp.end())
        dp[current] = 2;
      else
        dp[current] = dp[previous] + 1;
      if (dp[current] > max_length) {
        max_length = dp[current];
        diff = arr[j] - arr[i];
        first_num = arr[i];
      }
    }
  }
  cout << "max length: " << max_length << endl;
  cout << "longest arithmetic progression is: ";
  for (int i = 0; i < max_length; ++i) {
    cout << first_num << " ";
    first_num += diff;
  }
  cout << endl;
}
{% endhighlight %}

**测试程序与结果**

{% highlight javascript %}
#include <iostream>
#include <vector>
#include <unordered_map>
#include <utility> // std::pair
#include <functional> // std::hash
using namespace std;

int main() {
  vector<int> arr({1, 7, 3, 9, 11, 5, 13});
  LongestArithmeticProgression(arr);
  return 0;
}
{% endhighlight %}

~~~
// {0, 3, 5, 8, 10, 15, 20}
max length: 5
longest arithmetic progression is: 0 5 10 15 20 

// {1, 1, 1, 1, 1, 1, 1, 1}
max length: 8
longest arithmetic progression is: 1 1 1 1 1 1 1 1 

// {1, 2, 3, 4, 5, 2, 3}
max length: 5
longest arithmetic progression is: 1 2 3 4 5 

// {1, 2, 3, 2, 3, 4, 5}
max length: 5
longest arithmetic progression is: 1 2 3 4 5 
~~~


## Problem 2
* 给定的数组是有序的，求其中的最长的等差数列（子序列）的长度，并输出该等差数列。如{0, 1, 2, 5, 5, 6, 10, 15}，最长等差数列为0, 5, 10, 15。  
* 或者是给定一个无序的**集合**，求可以组成的最长等差子序列，序列中元素的位置可以与输入的集合不同，如{1, 7, 3, 9, 11, 5, 13}，最长等差数列长度为7，数列为1, 3, 5, 7, 9, 11, 13，此情况下可以先对输入的集合进行排序，再进行计算。  

## Solution 2  
* 对于有序数组的求解，除了可以用前文所述的通用方法计算，还有其他的方法，可以利用有序的性质。  
* f(i, j)表示以arr[i]和arr[j]作为数列的前两个元素的最长等差序列的长度，则`f(i, j) = f(j, k) + 1, arr[k] - arr[j] == arr[j] - arr[i]`，并且不需要枚举{i, j} 然后扫描k，做法是：由大到小枚举j，然后令i = j - 1，k = j + 1，根据有序的性质，i和k向两边扫描，当arr[k] - arr[j] == arr[j] - arr[i]时，求解f(i, j)。  
* 时间复杂度为O(n^2)

{% highlight javascript %}
// arr is sorted
void LongestArithmeticProgression(vector<int>& arr) {
  int n = arr.size();
  if (n <= 2) return;
  int max_length = 2, diff = arr[1] - arr[0], first_num = arr[0];
  vector<vector<int>> dp(n - 1, vector<int>(n, 2));
  for (int j = n - 2; j > 0; --j) {
    int i = j - 1, k = j + 1;
    while (i >= 0 && k < n) {
      if (arr[j] - arr[i] < arr[k] - arr[j]) {
        --i;
      } else if (arr[j] - arr[i] > arr[k] - arr[j]) {
        ++k;
      } else {
        dp[i][j] = dp[j][k] + 1;
        if (dp[i][j] > max_length) {
          max_length = dp[i][j];
          diff = arr[j] - arr[i];
          first_num = arr[i];
        }
        --i;
      }
    }
  }
  cout << "max length: " << max_length << endl;
  cout << "longest arithmetic progression: ";
  for (int i = 0; i < max_length; ++i) {
    cout << first_num << " ";
    first_num += diff;
  }
  cout << endl;
}
{% endhighlight %}


**测试程序与结果**  
{% highlight javascript %}
#include <iostream>
#include <vector>
using namespace std;

int main() {
  vector<int> arr({0, 1, 2, 5, 6, 6, 6, 7, 10, 15, 20});
  LongestArithmeticProgression(arr);
  return 0;
}
{% endhighlight %}

 
~~~
// {0, 1, 2, 5, 6, 6, 6, 7, 10, 15, 20}
max length: 5
longest arithmetic progression: 0 5 10 15 20 

// {1, 1, 1, 1, 1, 1}
max length: 6
longest arithmetic progression: 1 1 1 1 1 1
~~~




