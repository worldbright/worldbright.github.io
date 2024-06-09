## 문제

Given an integer array `nums` and an integer `val`, remove all occurrences of `val` in `nums` [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm). The order of the elements may be changed. Then return _the number of elements in_ `nums` _which are not equal to_ `val`.

Consider the number of elements in `nums` which are not equal to `val` be `k`, to get accepted, you need to do the following things:

- Change the array `nums` such that the first `k` elements of `nums` contain the elements which are not equal to `val`. The remaining elements of `nums` are not important as well as the size of `nums`.
- Return `k`.

**Custom Judge:**

The judge will test your solution with the following code:

``` c++
int[] nums = [...]; // Input array
int val = ...; // Value to remove
int[] expectedNums = [...]; // The expected answer with correct length.
                            // It is sorted with no values equaling val.

int k = removeElement(nums, val); // Calls your implementation

assert k == expectedNums.length;
sort(nums, 0, k); // Sort the first k elements of nums
for (int i = 0; i < actualLength; i++) {
    assert nums[i] == expectedNums[i];
}
```


If all assertions pass, then your solution will be **accepted**.

### 요약

N개의 정수가 들어있는 배열이 있다.
1. val와 똑같지 않은 정수의 개수 K를 return 하고,
2. 배열의 요소들은 다음과 같이 수정되어야 한다.
	1. 배열의 앞 K개는 기존 배열에서 val를 제외한 모든 요소들이 들어가야 한다. 순서는 기존 배열과 동일하게 보장되어야 한다.
	2. 그 뒤 N-K개는 어떠한 숫자가 들어와도 상관없다.

## 풀이

### 접근 1

``` c++
class Solution {  
public:  
    int removeElement(vector<int>& nums, int val) {  
        int answer = nums.size();  
        for (int i = 0; i < nums.size(); i++) {  
            if (nums[i] == val) {  
                answer--;  
                nums[i] = -1;  
            }  
        }  
  
        for (int cur = 0, resultCur = 0; cur < nums.size(); cur++) {  
            if (nums[cur] != -1) {  
                nums[resultCur++] = nums[cur];  
            }  
        }  
        return answer;  
    }  
};
```

문제의 해결과정을 단순하게 1단계, 2단계로 나누어서 빠르게 코드를 작성했어요.

1. val 원소들을 -1로 초기화 한다. (val는 0보다 크다는 범위 명시가 있었기 때문)
2. 이후 -1을 제외한 숫자들을 몽땅 앞으로 당긴다.

문제는 간단하고 빠르게 해결했지만, 다시 생각해보니 -1로 모두 초기화 하는 과정이 필요 없었던 것 같아요. **1단계, 2단계로 나눌 필요 없이, 하나의 단계로 줄일 수 있을 것 같아서 코드를 다시 작성했어요.**
### 접근 2

``` c++
class Solution {  
public:  
    int removeElement(vector<int>& nums, int val) {  
        int answer = 0;  
        for (int cur = 0, resultCur = 0; cur < nums.size(); cur++) {  
            if (nums[cur] != val) {  
                answer++;  
                nums[resultCur++] = nums[cur];  
            }  
        }  
        return answer;  
    }  
};
```

Approach 1에서 1단계, 2단계가 의미하는 바는 단순하게 한 문장으로 줄일 수 있어요. val을 제외한 원소들을 배열의 앞으로 당긴다. 그리고 단순하게 한 문장으로 줄일 수 있다는 뜻은, 1단계로 줄일 수 있다는 뜻도 되죠.

1. val 을 제외한 숫자들을 몽땅 앞으로 당긴다.

단계가 하나로 줄어듬에 따라서 for문도 하나로 줄어들었어요.

## 소감

너무나도 쉬운 5분 만에 해결이 가능한 문제였어요. 다만 어떠한 방법이 더 나은가, 어떻게 하면 더 간단하고 쉬운 단계로 풀 수 있는가, 간단하고 쉬운 풀이와, 단계가 조금 있지만 가독성이 좋은 코드 중 어떠한 코드를 짜야 하는가... 등등 여러가지 생각이 들었어요. 제 생각엔 후자의 코드를 짜야할 것 같은데, [접근 1](#접근-1) 의 코드가.. 더 가독성이 좋은지는 사실 잘 모르겠네요. 사실 코드를 짠 사람의 입장에서는 코드를 짠 직후의 시점에서는 어느 코드가 더 가독성이 좋은가에 대한 생각이 정확하지가 않을 것 같아요. 조금 지난 시점에서, 혹은 다음날 정도에 다시 보면 모를까.. 아무튼 갑자기 뭔가 생각할 거리가 많았네요.

첫 리트코드를 4월 27일에 풀고.. 한 달 반 정도가 순식간에 지나갔네요.. ㅠㅠ.. 너무 시간이 빨리가고 자기개발을 하기 위한 체력은 더없이 부족한 것 같아요. 최근에는 주에 3회 아침 수영을 시작했어요. 체력을 많이 키울 수 있겠죠? .. 리트코드는 업로드하지 못했지만 저번 주에는 쿠버네티스 관련해서 궁금한 점을 조사하고 업로드 했어요. 그래도 주에 하나 씩은 꼭 업로드 하자는 목표를 세워야겠어요.

지금은 아무도 보지 않는 블로그이지만, 미래의 내가 많이 볼 것 같아요. 아, 이때는 이런 마음으로 살았구나.. 이런 생각을 했구나 하는. 그 미래의 나는 지금보다는 조금 열심히 살겠지요??? 화이팅..