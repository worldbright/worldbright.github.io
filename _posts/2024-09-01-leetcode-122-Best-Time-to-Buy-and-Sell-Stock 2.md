---
title: "[Medium] LeetCode - 122. Best Time to Buy and Sell Stock 2"
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
  - LeetCodeMedium
---
## 문제

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the `ith` day.

On each day, you may decide to buy and/or sell the stock. You can only hold **at most one** share of the stock at any time. However, you can buy it then immediately sell it on the **same day**.

Find and return _the **maximum** profit you can achieve_.

**Example 1:**

> **Input:** prices = [7,1,5,3,6,4]  
> **Output:** 7  
> **Explanation:** Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = 5-1 = 4.  
> Then buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = 6-3 = 3.  
> Total profit is 4 + 3 = 7.

**Example 2:**

> **Input:** prices = [1,2,3,4,5]  
> **Output:** 4  
> **Explanation:** Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.  
> Total profit is 4.

**Example 3:**

> **Input:** prices = [7,6,4,3,1]  
> **Output:** 0  
> **Explanation:** There is no way to make a positive profit, so we never buy the stock to achieve the maximum profit of 0.

**Constraints:**

- `1 <= prices.length <= 3 * 104`
- `0 <= prices[i] <= 104`
## 풀이

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int profit = 0;
        int prevPrice = prices[0];
        for (int price : prices) {
            if (price - prevPrice > 0) {
                profit += price - prevPrice;
            }
            prevPrice = price;
        }
        return profit;
    }
};
```

Medium 난이도길래 조금 어려울 줄 알았는데, 1분 만에 풀어버린 너무 간단한 해답이었습니다.

주식시장에서 무조건 이득을 얻는 방법은.. 상승 곡선을 그리자 마자 팔아치우면 되죠.

이전 문제로 단순하게 생각하자고 결심했던게 도움이 됐던 걸까요...! 생각보다 빠르게 효과가 찾아온듯 하네요...!?

ㅎㅎ.. 진지한 제 생각으로는 난이도 측정이 잘못된 것 같습니다. 이 문제는 Easy로 가야 맞는 것 같아요.