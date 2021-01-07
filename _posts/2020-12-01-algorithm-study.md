---
title: "알고리즘 리뷰"
categories:
  - algorithm
tags:
  - 알고리즘
---

코딩테스트를 준비하면서, 무작정 문제를 많이 풀고 넘어가는것에 그치지 않고 사용했던 기술이나 느낀점을 기록해놓기로 하였다.


# 리뷰

### 카카오 다트 게임 [`commit 07c31`](https://github.com/maenguin/Algorithm/commit/07c314561c29136f284fb457ea3d9340cade6f1d)
문자열의 문자를 하나하나씩 처리하는걸 생각했지만 조건이 여러개라 구현하는데 어려움을 겪었다.
* 점수가 1 ~ 10 이기 때문에 10 나오면 문자 하나로 숫자를 판별할 수 없다.
* 옵션은 있을 수 도 있고 없을 수 도 있다.
* 그래서 다트 게임 한세트의 정보 길이는 2 ~ 4로 유동적이다.
* 옵션으로 * 이 나오면 이전 세트에 변화를 주어야 했다.

문제를 처음 풀었을때는 우격다짐식으로 토큰화를 했다. 덕분에 코드가 길어지고 if else가 난무했다.
카카오 리뷰를 보고 정규식으로 토큰화를 하면 괜찮겠다고 생각했다. 

`String.split(String regex)`을 먼저 시도 했지만 원하는 결과가 나오지 않았다.
```java
String[] tokens = dartResult.split("(?=[0-9]|10)([SDT])([*#])?");
//10이 들어가면 1과 0으로 분리되는 현상이 있었다.
```
그래서 관련 내용을 찾아보다가 [`Pattern`과 `Matcher` 이용해 토큰화를 성공할 수 있었다.](#토큰화-하기)


### 문자열 압축 [`commit b4be3`](https://github.com/maenguin/Algorithm/commit/b4be32543a8b5ecbfe514fb33f6c1b0886c10d82)
주어진 문자열을 n개 별로 잘라서 압축했을때 최소 길이를 구하는 [문제였다.](#문자열-n개씩-자르기)



### 핸드폰 번호 가리기 [`commit 71924`](https://github.com/maenguin/Algorithm/commit/71924947b8564a1f7074f2d7ee0d6bf478014702)
핸드폰 번호의 뒤 4자리를 \*로 가리는 문제였다. [java의 replaceAll과 정규 표현식을 사용했다.](#replaceAll)


### 소수 찾기 [`commit 33a71`](https://github.com/maenguin/Algorithm/commit/33a71adc3d0eb58d20051cc76f362f88683e8bd1)
1과 주어진 숫자 n사이에 있는 소수의 개수를 구하는 문제였다. 
처음에는 단순히 학교에서 소수를 찾을때 썻던 알고리즘을 사용해 풀었지만 효율성 테스트에서 막히고 말았다.
날로 먹을려던 행동을 반성하고 `에라토스테네스의 체` 알고리즘을 사용해 다시 풀었다.
하지만 여전히 효율성 테스트를 통과하지 못했다.
무엇을 더 효율좋게 바꾸어야 될까 고민하고 고민하다가 나랑 같은 문제를 직면하신분이 내놓은 해결책을 보게 되었다.
에라토스테네스의 체로서 사용하던 Array<Boolean>을 `Boolean`에서 `Integer`로 바꾸라는것이었다.
반신반의하면서 바꾸어보았는데 테스트를 말끔히 통과하였다.
왜 이런 퍼포먼스적인 차이가 발생했는지 알아봐야겠다...

### 문자열 정수로 바꾸기 [`commit 2fd76`](https://github.com/maenguin/Algorithm/commit/2fd76694a2adc8247594402d5097e10c351f2ed0)
말그대로 주어진 문자열을 정수로 바꾸는 문제였다.
`Integer.parseInt`를 약식으로 구현하였는데 실제 함수를 살펴보니 조금 특이한점을 발견했다.
나는 문자열을 숫자로 변환해 결과를 담는 변수에 양수로 값을 쌓았지만 실제 함수 `음수`로 값을 쌓았다.
이유인 즉슨, 자료형의 값의 범위를 보았을떄 `MIN_VALUE`의 절대적 크기가 `MAX_VALUE`보다 1크기 때문에
양수로 값을 쌓게되면 표시하지 못하는 값이 생길 수 있기 떄문이었다.

### 제일 작은 수 제거하기 [`commit 8354b`](https://github.com/maenguin/Algorithm/commit/8354bb29be9684aa6b62664902634d050070b85e)
솔루션을 두개 작성했다. 하나는 스트림을 이용해서 풀었고 나머지는 고전적인 방법을 이용해 풀었다.
스트림 케이스를 박싱과 언박싱이 없도록 사용하였지만 고전적인 풀이에 비해 속도가 5배이상 차이가 났다.
스트림을 사용해서 문제를 풀때면 항상 성능적인 부분에 대해 고민하게 된다.
실무에서는 어떻게 사용할지 궁금하다.

### 멀쩡한 사각형 [`commit 2d8a7`](https://github.com/maenguin/Algorithm/commit/2d8a74e35e2abdbc3add297462ff77cf3ba9498d)
대각선이 지나는 사각형 격자가 있을때 대각선을 포함하지 않는 격자칸의 개수를 구하는 문제였다.
수학적인 규칙이 있을것이라 예상했지만 발견하는데 오래걸렸기 때문에 처음에는 기울기를 이용해서 풀었다.

문제를 풀고 나서야 격자칸의 개수는 가로길이 + 세로길이 - 가로와 세로의 최대공약수 인것을 알게되었다.
문제를 처음보았을때 당황하지 않고 그림을 ppt등 이용해 그려서 규칙을 도출해내었다면 쉽게 풀 수 있었을것 같다.  
  
***

# 기술

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
Arrays.sort(s.toCharArray());
return new StringBuilder(String.valueOf(chars)).reverse().toString();
///
String[] split = s.split("");
Arrays.sort(split, Comparator.reverseOrder());
return String.join("",split);
///
return Arrays.stream(s.split("")).sorted(Comparator.reverseOrder()).collect(Collectors.joining());
///
return s.chars().boxed().sorted(Collections.reverseOrder())
                        .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
                        .toString();
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

## 기억해두면 좋은 코드

### 문자열 처리 (String Manipulation)
#### 토큰화 하기
```java
String dartResult = "10S*4D1T#"
Pattern pattern = Pattern.compile("([0-9]|10)([SDT])([*#])?");
Matcher matcher = pattern.matcher(dartResult);

while (matcher.find()) {
    String score = matcher.group(1);
    String bonus = matcher.group(2);
    String option = matcher.group(3); //없으면 null
}
//10S* 4D 1T#
```

#### 정규식과 replaceAll로 문자 치환하기
```java
String phone_number = "01012345678"
phone_number.replaceAll(".(?=.{4})","*"); //0101234****

//번외 후방탐색 앞 3자리 남기고 * 치환
phone_number.replaceAll("(?<=.{3}).","*"); //010********
//번외 가운데 번호 * 치환
phone_number.replaceAll("(?<=.2|.{3}).(?=.{4})","*"); //010****5678
```
#### 문자열 n개씩 자르기
* substring으로 문자열 i개씩 자르기
```java
for (int i = 1; i <= length/2; i++) {
    for (int j = 0; j < length; j += i) {
        String sub = s.substring(j, Math.min(j + i, length));
    }
}
```
* 정규식으로 문자열 i개씩 자르기
```java
for (int i = 1; i <= length/2; i++) {
    String[] split = s.split("(?<=\\G.{" + i + "})");
}
```


### 완전 탐색  
가능한 모든 경우의 수를 탐색해 문제를 해결하는 방법이다.  
효율과는 거리가 먼 방법이지만 직관적으로 빠르게 문제를 해결할 수 있다는 장점이 있다.  
완전 탐색을 구현하는 방법에도 여러가지가 있는데, 예를 들면 다음과 같다. 
  
  
#### Brute Force  
for문과 if를 이용해 탐색  
```java
//예시 1 : 리그제 처럼 각 원소들 끼리 한번씩 비교하는 코드
for (int i = 0; i < arr.length - 1; i++) {
    for (int j = i; j < arr.length; j++) {
        var a = arr[i];
        var b = arr[j];
        //... a b 비교
    }
}
```
#### BFS/DFS
#### Back Tracking
#### Permutation
경우의 수를 구하는것은 대표적으로 순열을 예로 들 수 있다.  
  
swap과 backtracking을 이용해 구현한 순열 코드  
순서를 보장받지 못한다.
```java
void unstablePermutation(int[] arr, int n, int r, int depth) {
    if (depth == r) {
        print(arr, r); // 0 부터 r개 출력
        return;
    }

    for (int i = depth; i < n; i++) {
        swap(arr, depth, i);
        perm(arr, n, r, depth + 1);
        swap(arr, depth, i);
    }
}
```
#### Bit Mask
#### Recursive Function



### 그외

#### switch말고 배열과 % 사용하기 [`commit 4d2df`](https://github.com/maenguin/Algorithm/commit/4d2dfd3ccb792f3af604602d4d1b62a6af8b25d4)

```java
String[] dayofWeek = {"THU","FRI","SAT","SUN","MON","TUE","WED"};
String answer = dayofWeek[sumOfDays%7];
```

#### 두 정수 사이의 합 [`commit b88cd`](https://github.com/maenguin/Algorithm/commit/b88cd1c2e14f595610e540c6f8d77a6c80501796)

수학의 등차수열의 합을 응용했다. 
```java
private long sumAtoB(long min, long max) {
    return (max - min + 1) * (min + max) / 2;
}

//sumAtoB(3,5) : 12
//sumAtoB(-5,-3) : -12
```


