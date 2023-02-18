---
layout: single
title:  "[C++ / Algorithm] 순열, 조합 구하기"
categories: algorithm
tag: [algorithm]
# search: false 검색 기능에 나오지 않게 할려면 false
---

알고리즘 문제를 풀다보면 여러 숫자가 주어졌을 때 순열이나 조합, 때로는 조합할 수 있는 모든 수를 확인해야할 때가 있습니다.<br>
그럴 때 C++의 next_permutation함수를 사용하면 쉽게 순열이나 조합을 구할 수 있습니다!! <br>
따라서 오늘 포스팅은 next_permutation함수 입니다! <br><br><br>

# next_permutation 함수 소개

[https://cplusplus.com/reference/algorithm/](https://cplusplus.com/reference/algorithm/) <br>
![](../../assets/images/2023-02-18-Permutation.md/2023-02-18-10-07-14.png) <br>

해당 링크 제일 밑에 가시면 함수에 대한 정확한 정보를 얻으실 수 있습니다. <br>

기본적 문법입니다.
> bool next_permutation (BidirectionalIterator first, BidirectionalIterator last);

파라미터로 오름차순으로 정렬된 컨테이너의 시작과 끝을 넣어줍니다. <br>
3번째 파라미터로 비교함수를 넘기는 것도 가능합니다. <br><br>

반환값 설명입니다.
> true if the function could rearrange the object as a lexicographicaly greater permutation.
Otherwise, the function returns false to indicate that the arrangement is not greater than the previous, but the lowest possible (sorted in ascending order).

만약 사전순으로 더 큰 순열이 있을 경우 재배치하고 true를 반환합니다. <br>
더 큰 순열이 없다면 false를 반환하고 오름차순으로 정렬시킵니다. <br><br><br>


# 사용예제

next_permutation 함수를 사용하기 위해서는 algorithm 라이브러리를 include해야 합니다. <br>

## 순열 구하기

``` c++
#include <iostream>     // std::cout
#include <algorithm>    // std::next_permutation, std::sort

int main () {
  int myints[] = {1,2,3};

  std::sort (myints,myints+3);

  do {
    std::cout << myints[0] << ' ' << myints[1] << ' ' << myints[2] << '\n';
  } while ( std::next_permutation(myints,myints+3) );

  std::cout << "After loop: " << myints[0] << ' ' << myints[1] << ' ' << myints[2] << '\n';

  return 0;
}
```
<br>
Output:
```
1 2 3
1 3 2
2 1 3
2 3 1
3 1 2
3 2 1
After loop: 1 2 3
```

next_permutation하기 전 오름차순으로 정렬해야 합니다. 기본적으로 현재보다 더 큰 순열을 찾기 때문입니다. <br>
do while 문으로 현재 순열을 출력하고, next_permutation 함수를 실행하여 myints 배열을 다음으로 큰 순열로 재배치합니다. <br>
더 큰 순열이 없다면 false를 반환하여 반복문이 종료됩니다. <br>
다 끝나면 오름차순으로 정렬됩니다. <br><br><br>


## 조합할 수 있는 모든 수 구하기
<br>
완전 탐색문제중 모든 조합을 구해서 확인하는 일이 종종 있습니다. 그럴때 사용하면 좋습니다. <br>

``` c++
#include <iostream>
#include<vector>
#include<string>     
#include <algorithm>    
#include <set>

using namespace std;

int main () {
  vector<int> v{1, 2, 3};
  set<int> s;

  do {
        string num = "";
        for (int i = 0; i < v.size(); i++)
        {
            num += to_string(v[i]);
            s.insert(stoi(num));
        }
    } while (next_permutation(v.begin(), v.end()));

  for (auto iter = s.begin(); iter != s.end(); iter++)
        cout << *iter << ' ';

  return 0;
}
```
<br>
Output
```
1 2 3 12 13 21 23 31 32 123 132 213 231 312 321
```
<br><br><br>


## 조합 구하기

{1, 2, 3, 4} 배열이 있을 때 이 중 2개 뽑는 조합을 구해봅시다.
``` c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
 
int main() {
    vector<int> s{ 1, 2, 3, 4 };
    vector<int> temp{ 0, 0, 1, 1 };
 
    do {
        for (int i = 0; i < s.size(); ++i) {
            if (temp[i] == 0)
                cout << s[i] << ' ';
        }
        cout << endl;
    } while (next_permutation(temp.begin(), temp.end()));
}

```
Output
```
1 2
1 3
1 4
2 3
2 4
3 4
```
<br>

n개의 배열이 있고 이 중 r개를 조합한다고 하면 0이 r개 1이 n-r개인 배열을 만듭니다. <br>
그리고 그 배열을 next_permutation을 합니다. <br>
만약 4개중 2개를 조합한다면 모든 순열은 다음과 같습니다. 
```
0 0 1 1
0 1 0 1
0 1 1 0
1 0 0 1
1 0 1 0
1 1 0 0
```
여기서 {1, 2, 3, 4}의 배열을 0과 1로 매핑시킵니다. <br>
0이면 숫자를 포함하고, 1이면 숫자를 포함하지 않는거죠. <br>
그럼 위 코드의 결과처럼 나오게 됩니다. <br><br>

0이 포함인게 마음에 들지 않는다면 prev_permutation을 사용하면 됩니다. <br>
prev_permutation은 next_permutation와 반대입니다. <br>
내림차순인 컨테이너의 처음과 끝을 인지로 넣고, 더 작은 순열이 있으면 true아니면 false를 반환하고 내림차순으로 재배치해줍니다.<br>

그럼 prev_permutation을 사용해서 조합을 구해보면 다음과 같습니다.
``` c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
 
int main() {
    vector<int> s{ 1, 2, 3, 4 };
    vector<int> temp{ 1, 1, 0, 0 };
 
    do {
        for (int i = 0; i < s.size(); ++i) {
            if (temp[i] == 1)
                cout << s[i] << ' ';
        }
        cout << endl;
    } while (prev_permutation(temp.begin(), temp.end()));
}
```
결과는 위와 같습니다. <br><br><br>


# 느낀점
항상 조합을 구할 땐 무식하게 매번 재귀함수를 만들어서 구했었는데 이 함수를 알고서는 정말 편해졌습니다. <br>
모든 조합을 구할 때도 소개한 위 코드말고 반복문을 돌면서 조합구하는 코드를 사용해도 좋겠죠. <br>
어떻게 사용하든 자기 마음입니다!! <br>