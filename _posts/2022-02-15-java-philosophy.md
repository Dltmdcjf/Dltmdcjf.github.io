---
date: 2021-02-01
title: "자바의 철학 - 자바의 스택은 왜 register를 사용하지 않고 operand stack을 이용했을까?"
categories: Java
---

## 1.1. 자바 바이트 코드

아래 자바 코드를 바이트 코드로 변환해보자
```java
for (int i = 2; i < 1000; i++) {
    for (int j = 2; j < i; j++) {
        if (i % j == 0)
            continue outer;
    }
    System.out.println (i);
}
```

자바 컴파일러는 위의 자바 소스 파일을 아래와 같은 자바 바이트 코드로 변환해 준다.
대략 store,load 같은 단어들이 보인다. 자바는 해당 바이트 코드를 한줄씩 읽어내려간다.
이때, 자바는 operand stack을 사용해 해당 명령어에 해당하는 value들을 push하거나 pop후 연산의 결과를
다시 operand stack에다가 집어 넣는다. 

여기서 왜 Java는 귀찮게 stack을 사용했을까? 보통 연산은 레지스트리 3개 정도를 사용해서 빠르게 연산하는데 왜 굳이
빠른 registry를 사용하는대신 operand stack을 이용했을까?


이해하기 위해서는 자바의 JVM의 철학을 이해해야 한다. 자바는 초창기 JVM을 디자인 할 때, 네트워크에 연결된 모든 디바이스에
서 작동하는 것이 목표였다. 근데 과연 모든 디바이스가 동일한 레지스트리를 가지고 있다고 보장할 수 있을까?
아마 아니었을거다. 이를 일반화 하기 위해 조금은 느리더라도 operand stack을 사용해 바이트 코드의 명령어들을 처리하는 기법을 선택했으리라
생각한다.

```bash
0:   iconst_2
1:   istore_1
2:   iload_1
3:   sipush  1000
6:   if_icmpge       44
9:   iconst_2
10:  istore_2
11:  iload_2
12:  iload_1
13:  if_icmpge       31
16:  iload_1
17:  iload_2
18:  irem
19:  ifne    25
22:  goto    38
25:  iinc    2, 1
28:  goto    11
31:  getstatic       #84; // Field java/lang/System.out:Ljava/io/PrintStream;
34:  iload_1
35:  invokevirtual   #85; // Method java/io/PrintStream.println:(I)V
38:  iinc    1, 1
41:  goto    2
44:  return
```


***** 
## References
