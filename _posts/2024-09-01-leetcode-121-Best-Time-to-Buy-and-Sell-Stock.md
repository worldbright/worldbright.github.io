---
title: "[Easy] LeetCode - 121. Best Time to Buy and Sell Stock"
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
---
## 문제

You are given an array `prices` where `prices[i]` is the price of a given stock on the `ith` day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return _the maximum profit you can achieve from this transaction_. If you cannot achieve any profit, return `0`.

**Example 1:**

> **Input:** prices = [7,1,5,3,6,4]
> **Output:** 5
> **Explanation:** Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
> Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.

**Example 2:**

> **Input:** prices = [7,6,4,3,1]
> **Output:** 0
> **Explanation:** In this case, no transactions are done and the max profit = 0.

**Constraints:**

- `1 <= prices.length <= 105`
- `0 <= prices[i] <= 104`

## 풀이

### 첫 접근

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        vector<int> mins(n, prices[0]);
        vector<int> maxs(n, prices[n-1]);
        for (int i = 1; i < n; i++) {
            mins[i] = (mins[i-1] > prices[i]) ? prices[i] : mins[i-1];
            maxs[n-i-1] = (maxs[n-i] < prices[n-i-1]) ? prices[n-i-1] : maxs[n-i];
        }

        int max = 0;
        for (int i = 0; i < n; i++) {
            max = std::max(max, maxs[i] - mins[i]);
        }
        return max;
    }
};
```

첫 접근은... 너무 꼬아서 복잡하게 생각했던 것 같습니다. 처음에 DP로 해결을 하려다가 너무 복잡한 것 같아 이런 방향으로 틀었는데, 지금보면 너무 복잡하게 생각했네요.

**풀이 요약**
- mins, maxs 배열을 생성한다.
	- mins[i] = prices[0 ~ i] 의 최솟값
		- 지금까지의 최솟값
	- maxs[i] = prices[i ~ n] 의 최대값
		- 지금부터 끝까지의 최댓값
- maxs[i] - mins[i] 의 최댓값을 구한다.
	- 지금까지의 최소 주식 가격과, 이 이후의 최대 주식 가격. 즉 차익의 최댓값

### 개선한 접근

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int max = 0;
        int min = prices[0];
        for (int price : prices) {
            if (price < min) {
                min = price;
            }
            max = std::max(max, price - min);
        }
        return max;
    }
}
```

모든 원소를 순차적으로 순회하며 **지금까지의 원소 중 최솟값**과 **순회 중인 한 원소**의 차이가 가장 큰 값을 도출해내면 됩니다.

첫번째 접근에서 굳이 최솟값과 최댓값을 미리 구해놓을 필요가 없었던 것이죠.

ㅎㅎ,,, 문제를 꼬아서 생각하지 않고 조금 더 쉽게 생각하는 연습을 할 필요가 있겠습니다.