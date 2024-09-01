---
title: "[Medium] LeetCode - 209. Minimum Size Subarray Sum"
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
  - LeetCodeMedium
---
## 문제

Given an array of positive integers `nums` and a positive integer `target`, return _the **minimal length** of a_ 

_subarray_

 _whose sum is greater than or equal to_ `target`. If there is no such subarray, return `0` instead.

**Example 1:**

> **Input:** target = 7, nums = [2,3,1,2,4,3]  
> **Output:** 2  
> **Explanation:** The subarray [4,3] has the minimal length under the problem constraint.

**Example 2:**

> **Input:** target = 4, nums = [1,4,4]  
> **Output:** 1

**Example 3:**

> **Input:** target = 11, nums = [1,1,1,1,1,1,1,1]  
> **Output:** 0

**Constraints:**

- `1 <= target <= 109`
- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 104`

**Follow up:** If you have figured out the `O(n)` solution, try coding another solution of which the time complexity is `O(n log(n))`.

## 문제 한 줄 요약

> 주어진 정수 배열에서, 요소들의 합이 target과 같거나 더 큰 가장 짧은 SubArray의 길이를 구하시오.

## 풀이

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int result = 0;
        int sum = 0;
        for (int i = 0, j = 0; i < nums.size(); i++) {
            sum += nums[i];
            if (sum >= target) {
                while(j < nums.size() && sum - nums[j] >= target) {
                    sum -= nums[j];
                    j++;
                }
                result = result == 0 ? i - j + 1 : std::min(result, i - j + 1);
            }
        }
        return result;
    }
}
```

1. 커서A를 요소 처음부터 끝까지 이동시키면서 값을 계속 더해 nums[0:A] 요소들의 합 SUM을 구합니다.
2. SUM이 target보다 커지면, 커서B를 요소 처음부터 끝까지 이동시키면서, nums[B:A] 요소들의 합 SUM이 target보다 큰 가장 큰 B를 찾습니다.
3. 이 과정을 요소 끝까지 진행시키면서, B-A의 최솟값을 저장하고 반환합니다.

전형적으로 두 개의 커서를 활용하는 문제였습니다.

문제에 O(nlogn)의 해답도 고민해보라고 해서, 예전에 한창 알고리즘 공부할 때 가끔 써먹었던 lazy propagation 트리 관련 알고리즘이 떠올랐지만.. 확실한 건 코딩해봐야 알겠지오.

우선은 O(n)의 해답을 찾았으니 패스하도록 합니다!