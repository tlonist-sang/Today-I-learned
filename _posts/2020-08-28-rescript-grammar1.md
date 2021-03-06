---
layout: post
title:  "ReScript의 여러 문법 알아보기 - 1"
date:   2020-08-28 17:53:57 +0900
categories: jekyll update
comments: true
---

킹 갓 디바인 셀레스쳘 언어, ReScript의 여러 문법을 알아보는 시간을 갖겠습니다. 대부분의 내용은 [공식문서](https://rescript-lang.org/docs/manual/latest/primitive-types)를 그대로 번역했습니다. 

- 그 전에, ReScript에 대해 간략한 설명을 하고 넘어갈게요. 설명은 ReScript 포럼의 [FAQ](https://forum.rescript-lang.org/t/rescript-frequently-asked-questions/59)를 참고했습니다. 

- ReScript는 JavaScript 플랫폼에서 최고의 typed language 경험을 제공하고자 하는 목적으로 만들어졌습니다. 
- ReScript는 bs-platform 이라는 BuckleScript 라이브러리로 JavaScript로 트랜스파일 됩니다. 
- 원래는 ReasonML 에서 리브랜딩 된 것이며, 따라서 대표적인 함수형 프로그래밍 언어인 OCaml의 타입 시스템에 기반을 두고 있습니다. OCaml의 버전이 올라감에 따라 싱크를 맞추고 있으며, ReScript 고유의 Js interop, 부가 언어 기능들을 제공하고 있습니다. 
- 2020년 8월 현재 Bob, Cheng, Cristiano, Maxim, Patrick, Ricky 이 여섯명에 의해 개발 및 유지보수 되고 있습니다.


### Primitives

#### String
- String은 "" (double quote)를 쓰도록 되어있습니다. ''(single quote)는 charactor 타입이 사용합니다. multi-line 도 허용합니다. Js 로 변환되면 '/n'이 붙습니다. String을 합차기 위해선 ++ 연산자가 사용됩니다. 

```ReScript
    let greeting = "Hello world!"
    let multigreeting = "Hello
    HowAreYou!"

    let introduction = "My name is " ++ "Harry."
```

- Js에서처럼 String interpolation(삽입)하고 싶다면, 해당 타입을 String으로 바꿔줘야 합니다. 만약 implicit 하게 진행하고 싶다면 j를 붙여줍니다.

```ReScript
let age = 10
let message = j`Today I am ${age} years young.`
```

- String에 대한 API 문서는 [여기](https://rescript-lang.org/docs/manual/latest/api/js/string)에 있습니다. 심심할 때 지하철에서 읽어 볼 예정입니다. 

#### Boolean
- Boolean 의 타입은 bool 입니다. true || false의 값을 가지며 연산자는 다른 언어들과 다르지 않습니다. == 은 구조적 동등을, === 은 참조적 동등을 비교합니다. 예를 들면,

```ReScript
(1,2) == (1,2) // true. 1은 1에 대응되고 2는 2에 대응되므로 구조적으로 일치합니다.
(1,2) === (1,2) // false. 두 값이 다른 주소를 바라보고 있습니다.
```

#### Integer과 Float
- Integer 타입은 int로 표현하며 32bit로 양/음을 표현합니다 (-2147483648~2147483647). 
```ReScrip
    Js.log(Js.Int.max) //2147483647
    Js.log(Js.Int.min) //-2147483648
```

- Float을 연산할 때는 +-*/ 대신 +. -. *. /. 를 씁니다. int는 계산 한계가 있지만 놀랍게도 float은 없습니다. 대신 precison이 존재하며 Js.Float.toExponentialWithPrecison 으로 원하는 정밀도만큼 계산이 가능합니다 (실험결과 ~digits의 한계는 100입니다.)

```ReScript
let result: float = 984098219048120948120984091283902183902184902810948210923123123123.1 *. 121233231.23123
Js.log(result) //1.1930540694410248e+74

\"@@"(Js.log, Js.Float.toExponentialWithPrecision(984098219048120948120984091283902183902184902810948210923123123123.1 *. 121233231.23123, ~digits=100)) //1.1930540694410247820811163947439585469104842873348658926353747093134875033600000000000000000000000000e+74
```

#### Unit
- unit 타입은 **()** 이라는 단 하나의 값만 갖습니다. ()는 JavaScript의 undefined로 변환됩니다. 주로 void function을 나타낼 때 많이 사용합니다.

```ReScript
type foo = unit => unit
type bar = unit => string
```
