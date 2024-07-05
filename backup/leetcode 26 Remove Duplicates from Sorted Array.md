---
title: LeetCode - 26. Remove Duplicates from Sorted Array
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
---

## 문제

Given an integer array `nums` sorted in **non-decreasing order**, remove the duplicates [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm) such that each unique element appears only **once**. The **relative order** of the elements should be kept the **same**. Then return _the number of unique elements in_ `nums`.

Consider the number of unique elements of `nums` to be `k`, to get accepted, you need to do the following things:

- Change the array `nums` such that the first `k` elements of `nums` contain the unique elements in the order they were present in `nums` initially. The remaining elements of `nums` are not important as well as the size of `nums`.
- Return `k`.

**Custom Judge:**

The judge will test your solution with the following code:

int[] nums = [...]; // Input array
int[] expectedNums = [...]; // The expected answer with correct length

int k = removeDuplicates(nums); // Calls your implementation

assert k == expectedNums.length;
for (int i = 0; i < k; i++) {
    assert nums[i] == expectedNums[i];
}

If all assertions pass, then your solution will be **accepted**.

**Example 1:**

> **Input:** nums = [1,1,2]
> **Output:** 2, nums = [1,2,_]
> **Explanation:** Your function should return k = 2, with the first two elements of nums being 1 and 2 respectively.
> It does not matter what you leave beyond the returned k (hence they are underscores).

**Example 2:**

> **Input:** nums = [0,0,1,1,1,2,2,3,3,4]
> **Output:** 5, nums = [0,1,2,3,4,_,_,_,_,_]
> **Explanation:** Your function should return k = 5, with the first five elements of nums being 0, 1, 2, 3, and 4 respectively.
> It does not matter what you leave beyond the returned k (hence they are underscores).

**Constraints:**

- `1 <= nums.length <= 3 * 104`
- `-100 <= nums[i] <= 100`
- `nums` is sorted in **non-decreasing** order.

### 요약

최대 3 * 104 개의 정수가 들어있는 배열을 순서를 유지한 unique한 값 배열로 만든 후에 unique한 값의 개수를 return하십시다.

## 풀이

``` c++
class Solution {
public:
    int removeDuplicates(vector%3Cint%3E& nums) {
        int resultIdx = 0;
        for(int cur = 1; cur < nums.size(); cur++) {
            if (nums[resultIdx] != nums[cur]) resultIdx++;
            nums[resultIdx] = nums[cur];
        }
        return resultIdx + 1;
    }
};
```

생각보다 별 거 아닌 문제인데 조금 고민을 해보았어요.

1. 가장 첫번째로 떠오른 풀이는 O(n^2) 이나오는, 중복이 발생할때 마다 그 뒤 모든 원소들을 한칸 앞당겨 오는 방식.
2. 하지만 분명 O(n)으로 해결할 수 있을 것 같았어요. 가장 간단한 방식으로는 unique한 값들을 다른 배열에 기록해놓았다가 원래 배열에 순차적으로 입력하는 방식, **하지만** 이것도 당연히 탐탁치 않았고..

결국 5분 정도 계속 고민을 하다가 위와 같은 코드로 두 개의 커서를 사용하면 쉽게 풀린다는 사실을 알았어요.

사실 요러한 코드는 읽기에는 어려울 것 같은데. 좀 더 가독성이 좋은 풀이가 궁금하네요.