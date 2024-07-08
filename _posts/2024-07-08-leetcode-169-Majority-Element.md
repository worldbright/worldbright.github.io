---
title: "LeetCode - 169. Majority Element"
categories: [Leet Code, Top Interview 150]
tags: [LeetCode, TopInterview150, MedianOfMedians, MooreMajorityVote]
---

## 문제

Given an array `nums` of size `n`, return _the majority element_.

The majority element is the element that appears more than `⌊n / 2⌋` times. You may assume that the majority element always exists in the array.

**Example 1:**

> **Input:** nums = [3,2,3]
> **Output:** 3

**Example 2:**

**Input:** nums = [2,2,1,1,1,2,2]
**Output:** 2

**Constraints:**

- `n == nums.length`
- `1 <= n <= 5 * 104`
- `-109 <= nums[i] <= 109`

**Follow-up:** Could you solve the problem in linear time and in `O(1)` space?

### 요약

n개의 정수값이 들어있는 배열에, [n / 2]개 보다 많이 들어있는 값이 있다. 해당 값을 찾아라.

## 풀이

``` c++
class Solution {
public:
    // get Kth biggest value by bubble sort
    int getKthByBubbleSort(vector%3Cint%3E& nums, int startIndex, int endIndex, int k) {
        // simple bubble sort
        for (int i = 0; i < endIndex - startIndex - 1; i++) {
            for (int j = startIndex; j < endIndex - 1 - i; j++) {
                if (nums[j] > nums[j+1]) {
                    swap(nums[j], nums[j+1]);
                }
            }
        }
        return nums[endIndex - k];
    }

    // partition by pivotvalue (smaller values to left, bigger values to right of the pivot)
    pair<int, int> partitionByPivot(vector<int>& nums, int startIndex, int endIndex, int pivotValue) {
        int cursor = startIndex;
        for (int i = startIndex; i < endIndex; i++) {
            if (nums[i] > pivotValue) {
                swap(nums[i], nums[endIndex - 1]);
                endIndex--;
                i--;
            } else if (nums[i] < pivotValue) {
                swap(nums[cursor], nums[i]);
                cursor++;
            }
        }

        int cursorStart = cursor;
        while(cursor < endIndex && nums[cursor] == pivotValue) cursor++;
        int cursorEnd = cursor - 1;
        return make_pair(cursorStart, cursorEnd);
    }

    // get Kth biggest value in O(n)
    int getKthByMedianOfMedians(vector<int>& nums, int startIndex, int endIndex, int k) {
        if (endIndex - startIndex <= 5) {
            return getKthByBubbleSort(nums, startIndex, endIndex, k);
        }

        vector<int> medians;
        for(int i = startIndex; i < endIndex; i += 5) {
            medians.push_back(
                getKthByBubbleSort(nums, 
                                   i,
                                   min(i + 5, endIndex), 
                                   min(3    , ((endIndex - i) / 2) + 1)
                                  )
                             );
        }

        int medianOfMedians = getKthByMedianOfMedians(medians, 0, medians.size(), (medians.size() / 2) + 1);
        // pivot is medianOfMedians
        pair<int, int> indexOfPivot = partitionByPivot(nums, startIndex, endIndex, medianOfMedians); 

        if (indexOfPivot.first <= endIndex - k && endIndex - k <= indexOfPivot.second) {
            return medianOfMedians;
        } else if (indexOfPivot.second < endIndex - k) {
            return getKthByMedianOfMedians(nums, indexOfPivot.second + 1, endIndex, k);
        } else {
            return getKthByMedianOfMedians(nums, startIndex, indexOfPivot.first, k - (endIndex - indexOfPivot.first));
        }
    }
    int majorityElement(vector<int>& nums) {
        return getKthByMedianOfMedians(nums, 0, nums.size(), (nums.size() / 2) + 1);
    }
}
```

![image](/assets/img/2024-07-08-leetcode-169-Majority-Element/Pasted-image-20240708231036.png){:width="500"}

### 접근

1. 처음에는 easy 난이도라서, 쉬운 문제겠거니 하고 그냥 정석적인 방법으로 풀려고 했어요. 배열의 길이가 50000개로 한정되어 있었고. 간단하게 Map같은 곳에 카운팅을 저장하면 되려나...? 했어요.
2. 그런데 이 문장에 완전히 꽂혀버렸어요.  
   `Follow-up: Could you solve the problem in linear time and in `O(1)` space?`  
   O(n) 시간 안에 fix된 메모리를 써서 풀 수 있다고...?
	1. 그 순간부터 O(n)안에 해결할 수 있는 방법을 생각하기 시작했어요.. 뭐가 있지...? 뭐가 있을까..? 그 동안 밥도 차리고,, 밥 먹고 설거지도 하고.. 거의 1시간은 고민했던 것 같아요. 거시적으로 바라보지 않고 머릿속에 for문과 if문을 그리면서 너무 미시적이게 코딩수준에서 고민해서 오래걸렸던 것 같아요.
	2. 조건은 어떠한 값이 (n/2)+1 개보다 무조건 많이 있다. 였으니.. 배열을 정렬한다면 !? 무조건 배열의 중앙에 해당 값이 자리할 수 밖에 없어요. 1시간 만에 깨달은 방법이죠.
	3. 하지만.. 정렬은 O(nlogn)이에요... 선형 시간이 아니죠.
3. 1시간만에 생각해낸 해법다운 해법같았던... 것이... 아니었어요. 하지만 곧바로 한가지 사실을 더 깨달았죠.  
   ***전부 다 정렬할 필요는 없다.***
	1. 해법에는 사실 정렬이 필요 없어요. 값들을 순서대로 나열했을 때, 가장 중앙에 있는 값을 원해요. 즉, 정렬할 필요 없이  (n / 2) + 1번째로 큰 값을 찾기만 하면 된다는 뜻이죠.
	2. 요 알고리즘의 이름은 제가 정말 잘 기억해요. [**median of medians**](https://en.wikipedia.org/wiki/Median_of_medians){:target="blank"}. 대학생 2학년 때 **문제해결기법**이라는, 젊으신 신임교수님이 진행하던 너무너무 재미있는 수업에서 과제로 나왔었어요. 매 주 항상 새로운 과제와 새로운 알고리즘을 알아가는 재미가 엄청나던 수업이었어요. 아무튼.. 어떤 과제의 가장 최적의 정답이 median of medians 라는 알고리즘 이었어요. 과제가 정확히 어떤 문제였는지 잘 기억은 안나지만,, 뭐.. 꼬으고 꼬여있던 해법이 배열에서 k번째로 큰 수를 찾는 거 였던 것 같아요. *근데 신기했던게, 이 알고리즘을 알아갔더니 교수님은 정작 모르고 계시던 알고리즘이었어요 ㅋㅋㅋㅋ..*
	3. 알고리즘의 로직도 대충 기억하고 있어서, 위키를 잠깐 둘러보고 바로 코딩하기 시작했어요. 다만... 너무 오랜만에 머리를 굴려서 그런지 무한루프도 발생하고, index stackoverflow도 발생하고.... 버그 잡느라 정말 정말 고생고생을 하면서 **2시간은 걸린 것 같아요.**
	4. 몇 번의 재시도 끝에 결국에 **Accepted**가 떴을 때 그 쾌감이란.... 찌릿짜릿
4. 다만..... 요것도 사실 메모리를 O(N)만큼은 소모해요. medians를 저장해둘 공간이 필요하기 때문이죠.
	1. 정말 정말 아무리 생각해봐도 모르겠어서 다른사람들의 Solutions를 봤더니... [Moore majority vote algorithm](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm){:target="blank"}이라는 해법이 존재했어요. for문 하나에... if문 하나.. 선형적이고 되게 간단한 해답.
	2. 몇 시간에 걸쳐서 median of medians를 구현한게 허탈해졌어요. 다만, 새삼 알고리즘의 위대함을 느끼네요. 위대한 알고리즘 하나로 70줄 가량의 코드가 하는 일을 단 10줄로 그것도 엄청 빠르게 해내니깐요. 그 70줄도 위대한 알고리즘인데 말이죠.

## 결론

median of medians 알고리즘을 직접 구현해보면서 느낀 건데, 구현 정확도나 속도가 정말 이전에 비해 많이 떨어진 것 같아요. 구현하는 문제를 많이 풀어야겠어요... 후아후아.

그래도 구현이... 성취감은... 엄청나네요 헿

오늘도 한 건 했다. 꾸준히 할게요 **화이팅**