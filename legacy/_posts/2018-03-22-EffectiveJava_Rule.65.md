---
layout: post
title:  Effective Java Rule.65        
tags: Effective Study 
categories: Effective
---   

이번 룰은 간단합니다. 

#### 예외를 무시하지 마라 

Java에서는 간단한 기능을 구현할 때도 try-catch를 강요합니다. (Java 공부를 시작할 때,) 간단하게 DB와 연동하는 예제를 작성했엇는데, 컴파일러가 try-catch를 쓰라고 강요했고, 저는 이를 throw를 통해 무시했습니다.  

하지만, 이번 rule에서는 무시하지는 말라고하네요 .. 

사실 조금만 생각하면 결론이 나올 정도로 뻔한 이유입니다.

> 여기서 무시하지 마라는 의미는 catch문에 아무것도 작성하지 않은 상태를 의미합니다.

예외처리를 무시할 경우, 해당 코드가 아닌 다른 부분에서 문제가 발생할 수 있으며 이를 찾기 어렵다.

맞는 말이고, 당연한 말입니다.

책에서는 적어도 로그만이라도 남겨두어, 추후 문제가 발생했을 때 이유를 찾을 수 있도록 방패를 만들어 두라고 합니다.   

> 시간이 남으면 토론해보고 싶은 부분이긴 하네요. 

끝 
