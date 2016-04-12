---
layout: post
title: 01背包及扩展
subtitle: 求解最大价值及取得最大价值的物品组合
---

## Problem
给定n个物品，大小为size[i]，价值为val[i]，要装入一个容量为m的背包，问能够得到的最大价值为多大？      
Follow up 1: 输出组成最大价值的物品组合，如果有多种组合，输出一种即可。      
Follow up 2: 输出组成最大价值的物品组合，如果有多种组合，输出编号字典序最小的组合。      
Follow up 3: 输出组成最大价值的所有组合。

## Solution
**Original Problem  求解最大价值**     
Solution 1: 二维DP      
状态：dp[i][j] -- 从物品0到物品i，恰好达到容量j时，能够得到的最大价值  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（dp[i][j] == －1表示无法恰好构成容量j）      
递推关系（物品i不选或者选）：dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - size[i]] + val[i])      
初始条件：dp[i][0] = 0，if (m >= size[0]) dp[0][size[0]] = val[0]，other d[i][j] = -1    
所求结果：max{dp[n - 1][j], j = 0, ..., n - 1}  
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<vector<int> > dp(n, vector<int>(m + 1, -1));
  for (int i = 0; i < n; ++i)
    dp[i][0] = 0;
  if (m >= size[0]) dp[0][size[0]] = val[0];
  for (int i = 1; i < n; ++i) {
    for (int j = 1; j <= m; ++j) {
      dp[i][j] = dp[i - 1][j];
      if (j >= size[i] && dp[i - 1][j - size[i]] != -1 &&
          dp[i - 1][j - size[i]] + val[i] > dp[i - 1][j])
        dp[i][j] = dp[i - 1][j - size[i]] + val[i];
    }
  }
  int max_val = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[n - 1][j] > max_val) max_val = dp[n - 1][j];
  }
  cout << max_val << endl;
}
{% endhighlight %}

Solution 2: 空间优化（推荐）     
因为计算行i时只依赖于行i - 1的两个元素，可以将空间优化到一维，且从后向前计算      
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<int> dp(m + 1, -1);
  dp[0] = 0;
  if (m >= size[0]) dp[size[0]] = val[0];
  for (int i = 1; i < n; ++i) {
    for (int j = m; j >= size[i]; --j) {
      if (dp[j - size[i]] != -1 && dp[j - size[i]] + val[i] > dp[j])
        dp[j] = dp[j - size[i]] + val[i];
    }
  }
  int max_val = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[j] > max_val) max_val = dp[j];
  }
  cout << max_val << endl;
}
{% endhighlight %}


**Follow-up 1： 输出最大价值对应的任一组合**      
Solution 1: 在二维DP中直接判断      
根据当j >= size[i]时dp[i - 1][j - size[i]] + val[i]与dp[i - 1][j]的关系来判断是否选择物品i，采用回溯的方式从物品n-1推到物品0。     
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<vector<int> > dp(n, vector<int>(m + 1, -1));
  for (int i = 0; i < n; ++i)
    dp[i][0] = 0;
  if (m >= size[0]) dp[0][size[0]] = val[0];
  for (int i = 1; i < n; ++i) {
    for (int j = 1; j <= m; ++j) {
      dp[i][j] = dp[i - 1][j];
      if (j >= size[i] && dp[i - 1][j - size[i]] != -1 &&
          dp[i - 1][j - size[i]] + val[i] > dp[i - 1][j])
        dp[i][j] = dp[i - 1][j - size[i]] + val[i];
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[n - 1][j] > max_val) {
      max_val = dp[n - 1][j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  vector<int> selected;
  for (int i = n - 1; i > 0; --i) {
    if (index >= size[i] && dp[i - 1][index - size[i]] != -1 &&
        dp[i - 1][index - size[i]] + val[i] > dp[i - 1][index]) {
      selected.push_back(i);
      index -= size[i];
    }
  }
  if (index) selected.push_back(0);
  cout << "selected items indexes: ";
  for (int i = selected.size() - 1; i >= 0; --i)
    cout << selected[i] << " ";
  cout << endl;
}
{% endhighlight %}

Solution 2: 在一维DP基础上      
在一维DP的基础上，需要一个二维bool数组choice，choice[i][j]表示物品0到物品i达到容量为j的最大价值时，物品i是否被选择。然后采用回溯的方式，得出从物品n－1 一直到物品0是否被选择。  
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<int> dp(m + 1, -1);
  vector<vector<bool> > choice(n, vector<bool>(m + 1, false));
  dp[0] = 0;
  if (m >= size[0]) {
    dp[size[0]] = val[0];
    choice[0][size[0]] = true;
  }
  for (int i = 1; i < n; ++i) {
    for (int j = m; j >= size[i]; --j) {
      if (dp[j - size[i]] != -1 && dp[j - size[i]] + val[i] > dp[j]) {
        dp[j] = dp[j - size[i]] + val[i];
        choice[i][j] = true;
      }
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[j] > max_val) {
      max_val = dp[j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  vector<int> selected;
  for (int i = n - 1; i >= 0; --i) {
    if (choice[i][index]) {
      selected.push_back(i);
      index -= size[i];
    }
  }
  cout << "selected items indexes: ";
  for (int i = selected.size() - 1; i >= 0; --i)
    cout << selected[i] << " ";
  cout << endl;
}
{% endhighlight %}

**Follow-up 2： 输出最大价值对应的最小字典序编号组合**   
最小字典序意味着`编号1，2，9`排在`编号1，4，6`的前面。  
Solution 1: 更改状态定义以及递推关系  
上文的状态定义及求解方法不能保证选择的组合符合最小字典序（在`dp[i - 1][j - size[i]] + val[i] == dp[i - 1][j]`时，选或者不选物品i都没有办法保证最小字典序，选出的组合没有什么规律），可以更改状态的定义以及递推关系，并从后向前计算。  
状态：dp[i][j]表示物品i到物品n-1，恰好达到容量j时，得到的最大价值  
递推关系：dp[i][j] = max(dp[i + 1][j], dp[i + 1][j - size[i]] + val[i])，特别地，当`dp[i + 1][j] == dp[i + 1][j - size[i]] + val[i]`时，选取物品i，这样的话，i从n-1推到0，能够保证选择的物品组合是最小字典序。  
结果表示：max{dp[0][j], j = 0, ..., m}

1.1 二维DP  
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<vector<int> > dp(n, vector<int>(m + 1, -1));
  for (int i = 0; i < n; ++i)
    dp[i][0] = 0;
  if (m >= size[n - 1]) dp[n - 1][size[n - 1]] = val[n - 1];
  for (int i = n - 2; i >= 0; --i) {
    for (int j = 1; j <= m; ++j) {
      dp[i][j] = dp[i + 1][j];
      if (j >= size[i] && dp[i + 1][j - size[i]] != -1 &&
          dp[i + 1][j - size[i]] + val[i] > dp[i + 1][j])
        dp[i][j] = dp[i + 1][j - size[i]] + val[i];
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[0][j] > max_val) {
      max_val = dp[0][j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  cout << "selected items indexes: ";
  vector<int> selected;
  for (int i = 0; i < n - 1; ++i) {
    if (index >= size[i] && dp[i + 1][index - size[i]] != -1 &&
        dp[i + 1][index - size[i]] + val[i] >= dp[i + 1][index]) {
      selected.push_back(i);
      index -= size[i];
    }
  }
  if (index) selected.push_back(n - 1);
  for (int i = 0; i < selected.size(); ++i)
    cout << selected[i] << " ";
  cout << endl;
}
{% endhighlight %}

1.2 在一维DP的基础上  
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<int> dp(m + 1, -1);
  vector<vector<bool> > choice(n, vector<bool>(m + 1, false));
  dp[0] = 0;
  if (m >= size[n - 1]) {
    dp[size[n - 1]] = val[n - 1];
    choice[n - 1][size[n - 1]] = true;
  }
  for (int i = n - 2; i >= 0; --i) {
    for (int j = m; j >= size[i]; --j) {
      if (dp[j - size[i]] != -1 && dp[j - size[i]] + val[i] >= dp[j]) {
        dp[j] = dp[j - size[i]] + val[i];
        choice[i][j] = true;
      }
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[j] > max_val) {
      max_val = dp[j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  cout << "selected items indexes: ";
  vector<int> selected;
  for (int i = 0; i < n; ++i) {
    if (choice[i][index]) {
      selected.push_back(i);
      index -= size[i];
    }
  }
  for (int i = 0; i < selected.size(); ++i)
    cout << selected[i] << " ";
  cout << endl;
}
{% endhighlight %}

Solution 2: 数组翻转处理  
其实，用最开始的解法，如果在`dp[i - 1][j - size[i]] + val[i] == dp[i - 1][j]`时，选择物品i，那么就能保证可以选到物品编号较大的那个（正向来看跟字典序没什么关系）。但是可以将原物品的数组翻转，然后尽量选择翻转后的数组编号较大的那个，其实就是选出了原数组中编号字典序最小的组合，最后由新数组的编号得到原数组的编号。

2.1: 二维DP
{% highlight javascript %}
#include <algorithm> // std::reverse
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<int> r_size(size);
  vector<int> r_val(val);
  reverse(r_size.begin(), r_size.end());
  reverse(r_val.begin(), r_val.end());
  vector<vector<int> > dp(n, vector<int>(m + 1, -1));
  for (int i = 0; i < n; ++i)
    dp[i][0] = 0;
  if (m >= r_size[0]) dp[0][r_size[0]] = r_val[0];
  for (int i = 1; i < n; ++i) {
    for (int j = 1; j <= m; ++j) {
      dp[i][j] = dp[i - 1][j];
      if (j >= r_size[i] && dp[i - 1][j - r_size[i]] != -1 &&
          dp[i - 1][j - r_size[i]] + r_val[i] > dp[i - 1][j])
        dp[i][j] = dp[i - 1][j - r_size[i]] + r_val[i];
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[n - 1][j] > max_val) {
      max_val = dp[n - 1][j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  vector<int> selected;
  for (int i = n - 1; i > 0; --i) {
    if (index >= r_size[i] && dp[i - 1][index - r_size[i]] != -1 &&
        dp[i - 1][index - r_size[i]] + r_val[i] >= dp[i - 1][index]) {
      selected.push_back(i);
      index -= size[i];
    }
  }
  if (index) selected.push_back(0);
  cout << "selected items indexes: ";
  for (int i = 0; i < selected.size(); ++i)
    cout << n - 1 - selected[i] << " ";
  cout << endl;
}
{% endhighlight %}

2.2 一维DP基础上  
{% highlight javascript %}
void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<int> r_size(size);
  vector<int> r_val(val);
  reverse(r_size.begin(), r_size.end());
  reverse(r_val.begin(), r_val.end());
  vector<int> dp(m + 1, -1);
  vector<vector<bool> > choice(n, vector<bool>(m + 1, false));
  dp[0] = 0;
  if (m >= r_size[0]) {
    dp[r_size[0]] = r_val[0];
    choice[0][r_size[0]] = true;
  }
  for (int i = 1; i < n; ++i) {
    for (int j = m; j >= r_size[i]; --j) {
      if (dp[j - r_size[i]] != -1 && dp[j - r_size[i]] + r_val[i] >= dp[j]) {
        dp[j] = dp[j - r_size[i]] + r_val[i];
        choice[i][j] = true;
      }
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[j] > max_val) {
      max_val = dp[j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  vector<int> selected;
  for (int i = n - 1; i >= 0; --i) {
    if (choice[i][index]) {
      selected.push_back(i);
      index -= size[i];
    }
  }
  cout << "selected items indexes: ";
  for (int i = 0; i < selected.size(); ++i)
    cout << n - 1 - selected[i] << " ";
  cout << endl;
}
{% endhighlight %}

**Follow-up 3： 输出最大价值对应的所有组合**   
在`dp[i - 1][j - size[i]] + val[i] == dp[i - 1][j]`时，物品i可选可不选，因此产生了多种组合。根据DP得到的最大价值所对应的容量，dfs（backtrack）各种可能的组合。

Solution 1: 二维DP

{% highlight javascript %}
void dfs(vector<vector<int> >& dp, vector<int>& size, vector<int>& val, int i,
         int volume, vector<int>& element, vector<vector<int> >& selected) {
  if (volume == 0) {
    selected.push_back(element);
    return;
  }
  if (i == 0) {
    selected.push_back(element);
    selected.back().push_back(0);
    return;
  }
  if (volume >= size[i] && dp[i - 1][volume - size[i]] != -1 &&
      dp[i - 1][volume - size[i]] + val[i] >= dp[i - 1][volume]) {
    element.push_back(i);
    dfs(dp, size, val, i - 1, volume - size[i], element, selected);
    element.pop_back();
  }
  if (volume < size[i] || dp[i - 1][volume - size[i]] == - 1 ||
      dp[i - 1][volume - size[i]] + val[i] <= dp[i - 1][volume]) {
    dfs(dp, size, val, i - 1, volume, element, selected);
  }
}

void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<vector<int> > dp(n, vector<int>(m + 1, -1));
  for (int i = 0; i < n; ++i)
    dp[i][0] = 0;
  if (m >= size[0]) dp[0][size[0]] = val[0];
  for (int i = 1; i < n; ++i) {
    for (int j = 1; j <= m; ++j) {
      dp[i][j] = dp[i - 1][j];
      if (j >= size[i] && dp[i - 1][j - size[i]] != -1 &&
          dp[i - 1][j - size[i]] + val[i] > dp[i - 1][j])
        dp[i][j] = dp[i - 1][j - size[i]] + val[i];
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[n - 1][j] > max_val) {
      max_val = dp[n - 1][j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  vector<vector<int> > selected;
  vector<int> element;
  dfs(dp, size, val, n - 1, index, element, selected);
  cout << "all the selected items indexes: " << endl;
  for (int i = 0; i < selected.size(); ++i) {
    for (int j = selected[i].size() - 1; j >= 0; --j)
      cout << selected[i][j] << " ";
    cout << endl;
  }
}
{% endhighlight %}

Solution 2: 在一维DP基础上  
此时的choice[i][j]可以为0（不选物品i），1（选择物品i）和2（选或者不选物品i都可以），据此可以用dfs得出各种组合。 

{% highlight javascript %}
void dfs(vector<vector<int> >& choice, vector<int>& size, vector<int>& val,
    int i, int volume, vector<int>& element, vector<vector<int> >& selected) {
  if (volume == 0) {
    selected.push_back(element);
    return;
  }
  if (choice[i][volume] == 1 || choice[i][volume] == 2) {
    element.push_back(i);
    dfs(choice, size, val, i - 1, volume - size[i], element, selected);
    element.pop_back();
  }
  if (choice[i][volume] == 0 || choice[i][volume] == 2) {
    dfs(choice, size, val, i - 1, volume, element, selected);
  }
}

void Backpack(int m, vector<int>& size, vector<int>& val) {
  int n = size.size();
  vector<int> dp(m + 1, -1);
  vector<vector<int> > choice(n, vector<int>(m + 1, 0));
  dp[0] = 0;
  if (m >= size[0]) {
    dp[size[0]] = val[0];
    choice[0][size[0]] = 1;
  }
  for (int i = 1; i < n; ++i) {
    for (int j = m; j >= size[i]; --j) {
      if (dp[j - size[i]] != -1) {
        if (dp[j - size[i]] + val[i] == dp[j]) {
          choice[i][j] = 2;
        } else if (dp[j - size[i]] + val[i] > dp[j]) {
          choice[i][j] = 1;
          dp[j] = dp[j - size[i]] + val[i];
        }
      }
    }
  }
  int max_val = 0, index = 0;
  for (int j = 1; j <= m; ++j) {
    if (dp[j] > max_val) {
      max_val = dp[j];
      index = j;
    }
  }
  cout << "max value: " << max_val << endl;
  cout << "all the selected items indexes: " << endl;
  vector<vector<int> > selected;
  vector<int> element;
  dfs(choice, size, val, n - 1, index, element, selected);
  for (int i = 0; i < selected.size(); ++i) {
    for (int j = selected.size() - 1; j >= 0; --j)
      cout << selected[i][j] << " ";
    cout << endl;
  }
}
{% endhighlight %}

## 测试结果
使用如下样例进行测试：

{% highlight javascript %}
#include <iostream>
#include <vector>
#include <algorithm> // std::reverse
using namespace std;

int main() {
  vector<int> size({2,1,1,3,2});
  vector<int> val({30, 1, 20, 50, 40});
  int m = 4;
  Backpack(m, size, val);
  return 0;
}
{% endhighlight %}

结果如下： 
 
~~~
// 输出对应于最优解的任一组合
max value: 70
selected items indexes: 2 3

// 输出对应于最优解的最小字典序编号组合
max value: 70
selected items indexes: 0 4

// 输出对应于最优解的全部组合
max value: 70
all the selected items indexes:
0 4
2 3
~~~




