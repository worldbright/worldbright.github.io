---
title: "[Medium] LeetCode - 80. Remove Duplicates from Sorted Array 2"
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
  - LeetCodeMedium
---

## 문제

Given an integer array `nums` sorted in **non-decreasing order**, remove some duplicates [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm) such that each unique element appears **at most twice**. The **relative order** of the elements should be kept the **same**.

Since it is impossible to change the length of the array in some languages, you must instead have the result be placed in the **first part** of the array `nums`. More formally, if there are `k` elements after removing the duplicates, then the first `k` elements of `nums` should hold the final result. It does not matter what you leave beyond the first `k` elements.

Return `k` _after placing the final result in the first_ `k` _slots of_ `nums`.

Do **not** allocate extra space for another array. You must do this by **modifying the input array [in-place](https://en.wikipedia.org/wiki/In-place_algorithm)** with O(1) extra memory.

**Custom Judge:**

The judge will test your solution with the following code:

``` c++
int[] nums = [...]; // Input array
int[] expectedNums = [...]; // The expected answer with correct length

int k = removeDuplicates(nums); // Calls your implementation

assert k == expectedNums.length;
for (int i = 0; i < k; i++) {
    assert nums[i] == expectedNums[i];
}
```

If all assertions pass, then your solution will be **accepted**.

**Example 1:**

> **Input:** nums = [1,1,1,2,2,3]
> **Output:** 5, nums = [1,1,2,2,3,_]
> **Explanation:** Your function should return k = 5, with the first five elements of nums being 1, 1, 2, 2 and 3 respectively.
> It does not matter what you leave beyond the returned k (hence they are underscores).

**Example 2:**

> **Input:** nums = [0,0,1,1,1,1,2,3,3]
> **Output:** 7, nums = [0,0,1,1,2,3,3,_,_]
> **Explanation:** Your function should return k = 7, with the first seven elements of nums being 0, 0, 1, 1, 2, 3 and 3 respectively.
> It does not matter what you leave beyond the returned k (hence they are underscores).

**Constraints:**

- `1 <= nums.length <= 3 * 104`
- `-104 <= nums[i] <= 104`
- `nums` is sorted in **non-decreasing** order.

### 요약

최대 3 * 10^4 개의 정수가 들어있는 배열을 중복된 값 최대 2개를 가질 수 있는 순서를 유지한 배열로 만들고, 해당 배열의 값 개수를 리턴하세요.

## 풀이

``` c++
class Solution {
public:
    int removeDuplicates(vector%3Cint%3E& nums) {
        int actualIdx = 2;

        int before2;
        int before1 = nums[0];

        for (int cursorIdx = 2; cursorIdx < nums.size(); cursorIdx++) {
            before2 = before1;
            before1 = nums[cursorIdx-1];
            
            nums[actualIdx] = nums[cursorIdx];

            if (!(nums[cursorIdx] == before1 && before1 == before2)) {
                actualIdx++;
            }
        }

        return min(actualIdx, (int)nums.size());
    }
}
```

이전 문제에서 단 하나의 조건 (중복 최대 2개)만 생겼을 뿐인데 생각할 거리가 많아졌어요.

### 접근

1. 이전과 같은 방식으로 두 커서를 이용한 해결방법을 생각했어요.
	- 두 커서를 이동하면서 커서 A에 커서 B를 복사하다가, 3개의 중복 값이 나오면
	- 해당 3번째 중복 값이 존재하는 커서A를 잠시 멈추고 커서 B만 옮기는 방식으로
	- 3번째 중복 값이 새로운 값으로 덮어씌워지면서 최대 2개의 중복값만 남도록 했어요.
2. 다만 3개의 중복값이 나오는 것을 어떻게 체크하느냐? 때문에 제 풀이가 조금 복잡해졌어요.
	- 커서 두 개를 이용해 **같은 배열**에서 뒤의 값(커서B)을 앞의 순서로 덮어씌우고 있기 때문에, 언제든 nums[i-n]의 값이 nums[i]값이 될 수 있었고, nums[i-2] == nums[i] 와 같은 조건으로는 중복 3개가 있는지 체크가 불가능했어요.
	- 따라서 nums[i-2], nums[i-1], nums[i] 의 값을 직접 조회하지 않고 미리 저장해두는 방식을 사용했어요.
3. 위의 두 개의 로직을 유지하면서 최대한 가독성이 좋게 코드를 작성해봤어요

## 결론

고등학교, 대학교,,, 어린 시절에 비해 확실히 머리가 많이 굳은 것이 느껴지네요. 머리가 이전처럼 잘 돌아가도록 꾸준히 풀어야겠어요 ㅠㅠ