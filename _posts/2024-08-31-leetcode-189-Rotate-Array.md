---
title: "[Medium] LeetCode - 189. Rotate Array"
categories:
  - Leet Code
  - Top Interview 150
tags:
  - LeetCode
  - TopInterview150
---
## 문제

Given an integer array `nums`, rotate the array to the right by `k` steps, where `k` is non-negative.

**Example 1:**

> **Input:** nums = [1,2,3,4,5,6,7], k = 3  
> **Output:** [5,6,7,1,2,3,4]
> **Explanation:**  
> rotate 1 steps to the right: [7,1,2,3,4,5,6]  
> rotate 2 steps to the right: [6,7,1,2,3,4,5]  
> rotate 3 steps to the right: [5,6,7,1,2,3,4]

**Example 2:**

> **Input:** nums = [-1,-100,3,99], k = 2
> **Output:** [3,99,-1,-100]
> **Explanation:**    
> rotate 1 steps to the right: [99,-1,-100,3]  
> rotate 2 steps to the right: [3,99,-1,-100]

**Constraints:**

- `1 <= nums.length <= 105`
- `-231 <= nums[i] <= 231 - 1`
- `0 <= k <= 105`

**Follow up:**

- Try to come up with as many solutions as you can. There are at least **three** different ways to solve this problem.
- Could you do it in-place with `O(1)` extra space?

## 풀이

가장 쉬운 접근 방법은 당연히... 배열 하나를 더 만들어서 몽땅 집어넣고 복사하는 것. 하지만 우리의 리트코드는 그걸 원한게 아니겠지요..

씨름 씨름 하다가, 재귀적으로 swap 시켜 몽땅 같은 순서를 유지시키는 아래 코드를 완성해냈습니다. 본능적으로 완성한 코드라.. 자세한 설명에는 고통이 따를 것 같은데.. 그래도 설명해보도록 하겠습니다. ㅠㅠ

### 코드

``` cpp
class Solution {
public:
    int process(vector%3Cint%3E& nums, int length, int k) {
        int cnt = 0;
        for (int i = (length - k) - 1; i >= 0; i--) {
            cnt = ++cnt % k;
            std::swap(nums[i], nums[i + k]);
        }
        return cnt;
    }
    void rotate(vector<int>& nums, int k) {
        k %= nums.size();
        if (k == 0) return;

        int cnt, length = nums.size();
        do {
            cnt = process(nums, length, k);
            length = k;
            k = k - cnt;
        } while (cnt != 0);
    }
}
```

### 전제

- n : nums 의 개수

### 접근 과정

1. 가장 처음 접근은 단순하게 첫 **(n-k)개의 요소를 배열의 맨 끝으로** 이동시키는 것이었습니다.
	- 예시 (k = 3)
		- [1,2,3,4,5,6,7]
		- [?,?,?,1,2,3,4]
2. 추가적인 공간을 쓰지 않고 이동시키기 위해서 무작정 배열의 끝 부터 desc 순서로 swap 했습니다.
	- 예시 (k = 3)
		- 1,2,3,4,5,6,7
		- 1,2,3,**7**,5,6,**4**
		- 1,2,**6**,7,5,**3**,4
		- 1,**5**,6,7,**2**,3,4
		- **7**,5,6,**1**,2,3,4
3. 다만, 이렇게 하면 이론상 배열의 첫 k개 요소의 순서가 올바르지 않게 되어 또 새로운 문제를 생성하게 됩니다.
	- 즉, n이 k로 나누어 떨어지면 너무 좋은 해답이 되겠지만, 아니라면 **나머지 개수** 만큼 더 처리해줘야 합니다. 이유를 예시로 들어 설명하자면...
		- 예시1 (k = 3)
			- 1,2,3,4,5,6,7,8,9 를 과정을 거치게 되면
			- 1,2,3,4,5,**9**,7,8,**6**
			- 1,2,3,4,**8,9**,7,**5,6**
			- 1,2,3,**7,8,9**,**4,5,6**
			- 1,2,**9,7,8,3,4,5,6**
			- 1,**8,9,7,2,3,4,5,6**
			- **7,8,9,1,2,3,4,5,6**
			- 아주 딱 알맞게 떨어지게 됩니다. 9는 3으로 나누어 떨어지기 때문이죠.
		- 예시2 (k = 3)
			- 1,2,3,4,5,6,7 을 k번(=3번) 만큼 과정을 거치면
			- 1 / **5,6,7** / **2,3,4** 가 되어서 순서가 유지되는데, 마지막 한 번 더 과정을 거치는 순간
			- **7,5,6,1,2,3,4** 가 되어 7,5,6의 순서를 망쳐버립니다.
	- 위 예시 2에서 7,5,6은 5,6,7을 k = 1 (n % k) 처리한 것과 같은 결과이고, 다시 5,6,7을 만드려면 k = 2 (k - (n%k)) 번 더 처리해야 합니다.
		- 위 코드에서 cnt가 k = 1 (몇 번 swap 처리 했는지) 저장하는 역할을 하는 것이고, 재귀적으로 다시 수행할 때에 k = k - cnt를 하는 것이 k = k - (n%k) 와 같은 식입니다.
4. 즉, 재귀적으로 표현한 식은 다음과 같습니다.
	- T(n, k) = O(n - k) + T(k, k - (n%k))
	- T(n, k) = O(n - k) + O(n % k) + T(k - (n%k), ...)
	- T(n, k) = O(n - k) + O(k) + ... = O(n)

## 결론

풀고나서 보니.. Solutions에는 비교적 간단한 해법이 있었습니다.

- 1,2,3,4,5,6,7
- 7,6,5,4,3,2,1 로 reverse 시킨 후
- 7,6,5 / 4,3,2,1 로 k번째에서 집합을 나누어
- 5,6,7 / 1,2,3,4 다시 reverse 시킵니다.

이 해법은 T(n) =  (n개 reverse)O(n / 2) + (k개 reverse)O(k / 2) + (n-k개 reverse)O((n-k)/2) = O(n) 입니다.

허탈하네요,,, 가장 직관적이고 이해가 빠른 해답이네요.

제 해답이 부끄러워지는 순간이지만.. 특별한.. 유니크하다고 생각하고 넘어가겠습니다.