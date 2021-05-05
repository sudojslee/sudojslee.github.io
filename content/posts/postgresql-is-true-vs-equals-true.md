+++ 
draft = false
date = 2021-05-05T22:47:40+09:00
title = "PostgreSQL IS TRUE vs = 'true'"
description = ""
slug = "postgresql-is-true-vs-equals-true"
authors = []
tags = ["Database"]
categories = []
externalLink = ""
series = []
+++

```postgres
SELECT * FROM table WHERE idx IS TRUE;
SELECT * FROM table WHERE idx = 'true';
```

최근에 PostgreSQL 을 사용하면서 삽질 했던 내용을 공유하기 위해 작성한다. 

```postgres
CREATE INDEX idx ON example_table WHERE a = 'true'


EXPLAIN ANALYZE SELECT * FROM example_table WHERE a IS TRUE;
// sequential scan
```

엥..? 분명히 partial index 를 만들었는데 select query 를 했을 때는 왜 index scan 이 아닌 sequential scan 을 하지?
는 인덱스와 쿼리를 유심히 살펴보면 정답이 나온다. 
인덱스는 `= 'true'` 로 만들었고, 쿼리는 `IS TRUE` 로 했기 때문이다. 


[Comparison Functions and Operators](https://www.postgresql.org/docs/13/functions-comparison.html)

`= 'true'` 는 Comparison Operator 이다.

<i>datatype</i> = <i>datatype</i> -> boolean

즉, 왼쪽과 오른쪽이 같은지를 비교하는 연산자이다.

a = 'true' 의 경우, a 라는 값과, 'true' 라는 값(value) 가 서로 같은지를 비교하고, boolean 값을 리턴한다.


`IS TRUE` 는 Comparison Predicate 이다. 

boolean IS TRUE -> boolean

a IS TRUE 는 a 라는 expression 을 계산하고, 그 결과가 <b>참인지 거짓인지</b> 을 판별한다.


사실 풀어서 설명하면 당연하다.

그렇다면 왜 이런 사소한 실수를 저질렀을까? 곰곰히 생각해봤는데, 그것은 우리의 한국인 뇌 때문이다. 한국인의 뇌를 대표하는 파파고에게 물어보자. 

1 equals null

1 is null
 
둘 다 `1은 null입니다.` 로 번역된다. 

`1 is equal to null` 은 `1은 null과 같습니다.` 로 번역 된다.

물론, 1 equals true와, 1 is true 는 서로 다른 번역을 준다.
