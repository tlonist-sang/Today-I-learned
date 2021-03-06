---
layout: post
title:  "ReScript의 파워풀한 패턴 매칭(Pattern-Matching)"
date:   2020-09-04 17:53:57 +0900
categories: jekyll update
comments: true
---

이번 시간에는 ReScript의
- Destructuring
- 데이터 모형에 기반한 switch문
- Exhaustive check 
에 대해서 다뤄보겠습니다. **매우매우 중요한 부분입니다!!**

## Destructuring
- JavaScript에서도 많이 사용하는 기능인 destructuring은, 객체 안의 데이터를 객체를 통한 접근이 아닌 직접 접근할 수 있도록 해줍니다. Flattening(평면화)라고도 하는데요, ReScript에서도 그 기능을 매우 잘 지원하고 있습니다.

```javascript
let coordinates = (10, 29, 23);
let (x, _, _) = coordinates;
Js.log(x) // 10
```
- 이런 단순한 데이터뿐만 아니라 대부분의 데이터 타입에서 destructuring 기능이 제공되는데요, 아래는 그 예시입니다.
```javascript
type student = {name: string, age: int}
let student1 = {name: "Harry", age: 14}
let {name} = student1 // (1) 

// Variant
type result =
  | Success(string)
let myResult = Success("You did it!")
let Success(message) = myResult // (2)"You did it!" assigned to `message`

// Array
let myArray = [1, 2, 3]
let [item1, item2] = myArray // (3) 1 assigned to `item1`, 2 assigned to `item2`

// List
let myList = list{1, 2, 3}
let list{head, ...tail} = myList // (4) 1 assigned to `head`, `list{2, 3}` assigned to tail
```
(1) "Harry" 가 `name` 에 할당되었습니다.
(2) Variant 타입인 Success 안의 "You did it"이 message에 매칭되었습니다.
(3) myArray 배열이 앞부터 차례로 1, 2에 매칭되었습니다. 
(4) list의 처음은 1에, 나머지 {2,3}은 tail에 매칭되었습니다.

- 레코드를 쓸 때는, destructure 된 값에 다른 필드를 매핑할 수도 있습니다.
```javascript
type student = {name: string, age: int}
let student1 = {name: "Harry", age: 14}
let {name: n} = student1 // Harry는 'n 이라는 타입에 매핑되었습니다.
```

## Switch 기반의 패턴 매칭
```javascript
type payload =
  | BadResult(int)
  | GoodResult(string)
  | NoResult
```
- 위와 같이 Variant가 정의되어 있고, 각각의 경우마다 다른 호출 결과를 보이고 싶다고 합시다. 잠깐 Variant에 대해서 설명을 하자면, payload는 BadResult, GoodResult, NoResult 세 개중 하나의 타입을 가질 수 있으며, 각각의 타입은 괄호 안의 값을 가질 수 있습니다. GoodResult 일 때는 축하메시지를, BadResult일때는 에러 코드를 알려주고 아무 결과가 없을때는 아무말이나 출력해봅시다. 


```javascript
let input = GoodResult("Harry");
switch(input){
    | GoodResult(name) => {
        Js.log("Congratulations!" ++ name)
    }
    | BadResult(errorCode) => {
        Js.log("Error occured with code: " ++ Js.Int.toString(errorCode))
    }
    | NoResult => {
        Js.log("bibi~")
    }
}
```
- 위의 코드는 **Congratulations!Harry** 를 출력합니다. 다음은 좀 더 어려운 예제를 보겠습니다.

```javascript
type status = Delivered | OnTheWay(int) | Cancelled
type crop = {name: string, quantity: int}
type person = 
    | Farmer({
        name: string,
        age: int,
        crop: crop
    })
    | Purchaser({
        name: string,
        status: status
    })
```
- 다음의 요구사항을 구현해봅시다. 
1. farmer의 이름이 John이거나 Matt 이면 나이를 물어봅니다. 
2. farmer의 crop이 100이면 수고했다고 말해줍니다.
3. 다른 farmer 들은 그냥 인사를 합니다.
4. 만약 purchaser의 상태가 OnTheWay면 며칠 남았는지를 표시합니다(int는 days를 가르킵니다.)
5. 다른 모든 경우에는 "Good!"을 출력합니다.

```javascript
let farmer1 = Farmer({fname: "Bibi", age: 47, crop: {cname: "potato", quantity: 100}});

let message = switch farmer1 {
    | Farmer({fname: "Matt" | "John", age}) => {
        "Hi~!" ++ ", are you " ++ Js.Int.toString(age) ++ " years old?";
        //Js.log(fname)
    }
    | Farmer({fname, crop: {cname, quantity: 100}}) => {
       "Hi, " ++ fname ++ " !, whoa you've raised " ++ Js.Int.toString(quantity) ++ " of "++ cname ++ "!"
    }
    | Farmer({fname}) => {
        "Hello~" ++ fname ++ "!";
    }
    | Purchaser({name, status: OnTheWay(daysLeft)}) => {
        "You have " ++ Js.Int.toString(daysLeft) ++ " days left";
    }
    | _ => {
        "Good!";
    }
}
```

- 위의 코드의 결과는 
```javascript
Hi, Bibi !, whoa you've raised 100 of potato!
```
입니다. 왜 그런지 천천히 직접 음미해보시죠!

- Farmer(payload) 와 같은 variant는 _로 안의 payload를 완전히 무시할 수 있습니다. 아래와 같이요
```javascript
switch person1 {
| Farmer(_) => Js.log("Hi Person")
| Person(_) => Js.log("Hey farmer")
}
```

위의 농부가 나온 예시에서, 작물이 100kg이 넘으면 축하한다고 얘기를 해봅시다.

```javascript
| Farmer({fname, crop: {quantity}}) => {
       if quantity > 100 {
           Js.log("Congratz!!")
       }
    }
```
- 이런식의 연산도 가능합니다.

- 또한 중요한 사실은 패턴으로 넘겨 받는 값은 무조건 literal(실값) 만 가능하다는 것입니다. let binding 된 변수는 받을 수 없습니다.
```javascript
let coordinates = (10, 20, 30)
let centerY = 20
switch coordinates {
| (x, centerY, _) => Js.log(x)
} //centerY가 안됩니다!
```

## Exhaustiveness check
- if-else 문이나 다른 조건문들로 범벅된 코드에 구멍이 하나도 없다고 정말 단언할 수 있을까요? 매우 어려울 겁니다. 놀랍게도 ReScript는 컴파일 타임에 이 논리적 구멍을 검증해주는 일도 해줍니다!!

```javascript
let myNullableValue = Some(5)

switch myNullableValue {
| Some(v) => Js.log("value is present")
| None => Js.log("value is absent")
}
```
- 여기서 None을 지우면 컴파일러가 경고를 ~~한다고 하는데 왜 제꺼에선 안할까요~~ 합니다. 이렇듯 논리의 구멍을 찾아서 경고해주는 훌륭한 역할도 한다고 하네요 ^^;

- 끝으로 절차피클에 찌든 JS 코드로 시작해서 간결하고 아름다운 ReScript 코드로 끝맺는 변천사를 보여드리겠습니다. 

```javascript
let optionBoolToBool = opt => {
  if opt == None {
    false
  } else if opt === Some(true) {
    true
  } else {
    false
  }
}
```
- opt에 값이 없거나 false면 false를 리턴하고 아니면 true를 리턴하는 매우 간단한 함수입니다.

```javascript
let optionBoolToBool = opt => {
  switch opt {
  | None => false
  | Some(a) => a ? true : false
  }
}
```
- 3항 연산자와 패턴매칭 적용

```javascript
let optionBoolToBool = opt => {
  switch opt {
  | None => false
  | Some(true) => true
  | Some(false) => false
  }
}
```
- 좀 더 풀어서 적어보았다가,

```javascript
let optionBoolToBool = opt => {
  switch opt {
  | Some(true) => true
  | _ => false
  }
}
```
- exhaustiveness를 _로 적용도 해보았다가

```javascript
let optionBoolToBool = opt => {
  switch opt {
  | Some(trueOrFalse) => trueOrFalse
  | None => false
  }
}
```
- 간결하게 정리!

오늘은 이렇게 pattern matching에 대해 알아보았습니다. 갈 길이 멀지만, 그래도 조금 더 알게되었다는 사실을 새기며 나아갑시다!