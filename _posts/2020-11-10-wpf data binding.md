---
title: "WPF Data Binding"
date: 2020-11-10 12:12:00 -0400
categories: Study WPF
---
###WPF을 사용해 프로그램을 개발할때, mvvm 모델을 사용하는걸 권장합니다.
###mvvm 모델의 핵심은 Data Binding인데 이 포스트에서는 Microsoft docs를 보면서 WPF의 Data Binding에 대해 배워보고자 합니다.
[마이크로소프트 WPF Data Binding docs][md]


#Data binding overview in WPF
----

#What is data binding?
###데이터 바인딩이란 app UI와 UI가 표시하는 데이터 사이의 연결을 설정하는 일종의 프로세스 입니다.
데이터가 UI에 바인딩이 올바르게 되어있다면, 데이터의 변경이 생긴경우 변경사항을 UI에 자동으로 반영합니다.
이는 반대로도 가능하며 UI에 변경이 생긴 경우 변경값을 데이터에 자동 반영합니다.

일반적으로 데이터 바인딩의 용도는 서버나 로컬 configration 데이터를 UI에 배치하는 것 이라면,
WPF에서 데이터 바인딩은 좀 더 광범위한 속성 바인딩을 포함하도록 확장되었습니다.
WPF에서는 .NET objects나 XML data를 바인딩 할 수 있습니다.

[md]: https://docs.microsoft.com/en-us/dotnet/desktop/wpf/data/data-binding-overview?view=netdesktop-5.0
