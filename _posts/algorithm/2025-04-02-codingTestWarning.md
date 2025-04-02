---
published: true
title:  "코딩 테스트 전에 보고가면 좋을 것들"
categories:
  - algorithm
---

## 비트마스킹

- 비트마스킹 할 때 bit의 개수, 즉 범위에 대해서 조심해야 합니다.

아래의 두 코드는 천지차이

~~~
var bitMask = 0L
var bitMask = 0
~~~

- java.lang.Long.bitCount()

Int는 아래처럼 작성하면 됩니다.

~~~
val count = Integer.bitCount()
~~~

- 테크닉

(추가 예정)