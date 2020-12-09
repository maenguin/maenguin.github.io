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

### int[] to List<Integer>

```java
List<Integer> integerList = Arrays.stream(intArray).boxed().collect(Collectors.toList());
```

### Integer[] to List<Integer>

```java
List<Integer> integerList = Arrays.stream(integerArray).collect(Collectors.toList());
List<Integer> integerList2 = Arrays.asList(integerArray);
```

### List<Integer> to int[]

```java
int[] intArray = integerList.stream().mapToInt(Integer::intValue).toArray();
```

### List<Integer> to Integer[]

```java
Integer[] integerArray = integerList.toArray(Integer[]::new);
```

### char[] to String

```java
String s = String.valueOf(charArray);
String s1 = new String(charArray);
```

## 정렬

### Sort Desending String [`commit c1a6c`](https://github.com/maenguin/Algorithm/commit/c1a6c022e2537804d97749cb9fe73a095820d644)
```java
Arrays.sort(charArray);
new StringBuilder(String.valueOf(chars)).reverse().toString();
///
String[] split = s.split("");
Arrays.sort(split, Comparator.reverseOrder());
return String.join("",split);
///
Arrays.stream(s.split("")).sorted(Comparator.reverseOrder()).collect(Collectors.joining());
///
s.chars().boxed().sorted(Collections.reverseOrder()).collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append).toString();
```


## 깔끔한 코드

* switch말고 배열과 % 사용하기 (programmers '2016년' 문제) [`commit 4d2df`](https://github.com/maenguin/Algorithm/commit/4d2dfd3ccb792f3af604602d4d1b62a6af8b25d4)

```java
String[] dayofWeek = {"THU","FRI","SAT","SUN","MON","TUE","WED"};
String answer = dayofWeek[sumOfDays%7];
```

* 두 정수 사이의 합 (programmers '두 정수 사이의 합' 문제)[`commit b88cd`](https://github.com/maenguin/Algorithm/commit/b88cd1c2e14f595610e540c6f8d77a6c80501796)

수학의 등차수열의 합을 응용했다. 
```java
private long sumAtoB(long min, long max) {
    return (max - min + 1) * (min + max) / 2;
}

//sumAtoB(3,5) : 12
//sumAtoB(-5,-3) : -12
```

## 헷갈리는 문법

### Substring
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

## 실수

### 명시적 변환이 필요한 경우
다음과 같이 int 정수를 받아서 연산을 한 뒤에 long으로 값을 반환하는 함수가 있다.
반환형식이 long이라 곱셉의 결과가 int를 넘어도 문제없을것 같지만 n이 int이기 때문에 곱셈 결과가 int를 넘어서면 잘못된 값이 반환된다.
```java
private long sum(int n){
    return n*(n+1)/2;
}
```
다음과 같이 long형으로 입력값을 받거나 코드 내부에서 명시적 변환을 해야한다.
```java
private long sum(long n){
    return n*(n+1)/2;
}
```







