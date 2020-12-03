---
title: "알고리즘 리뷰"
categories:
  - algorithm
tags:
  - 알고리즘
---

코딩테스트를 준비하면서, 무작정 문제를 많이 풀고 넘어가는것에 그치지 않고 사용했던 기술이나 느낀점을 기록해놓기로 하였다.


## 형변환

### int array로 반환해야 할때

1. 스트림이용(속도가 느림)
```java
collection.stream()
         .mapToInt(Integer::intValue)
         .toArray()
```


2. iterator 이용
```java
int[] answer = new int[size()];
int index = 0;
Iterator itor = collection.iterator();
while(itor.hasNext()){
    answer[index++] = (int)itor.next();
}
```

## 깔끔한 코드

* switch말고 배열과 % 사용하기 (programmers '2016년' 문제) [`commit 4d2df`](https://github.com/maenguin/Algorithm/commit/4d2dfd3ccb792f3af604602d4d1b62a6af8b25d4)
```java
String[] dayofWeek = {"THU","FRI","SAT","SUN","MON","TUE","WED"};
String answer = dayofWeek[sumOfDays%7];
```

## 헷갈리는 문법

### substring
C#을 주로 사용했던터라 Java의 substring을 사용할때 실수를 하는 경우가 종종 있다.
차이를 정리해보자

#### Java - substring(int beginIndex, int endIndex)
두번쨰 파라미터로 endIndex를 받는다. beginIndex부터 endIndex 이전 index까지의 데이터가 선택된다. 
{: .notice}

```java
String text = "ABCDE";
text.substring(1,4) // BCD
```
#### C# - Substring(int beginIndex, int length)
두번째 파라미터로 endIndex가 아니라 length를 받는다. beginIndex를 포함하여 length까지의 데이터가 선택된다.
{: .notice--primary}

```c#
string text = "ABCDE";
text.Substring(1,4) // BCDE
```










