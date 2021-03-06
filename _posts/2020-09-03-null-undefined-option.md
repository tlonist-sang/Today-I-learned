---
layout: post
title:  "Null, Undefined and Option"
date:   2020-09-03 17:53:57 +0900
categories: jekyll update
comments: true
---

오늘은 ReScript에서 Null, Undefined를 다루는 방법과 option syntax에 대해서 알아보겠습니다.
```javascript
```
- ReScript 자체에는 null 혹은 undefined 라는 개념이 없습니다. 이건 **굉장히 좋은** 것으로, JS에는 허다하게 나오는 ~가 undefined 되었다 혹은 null 값을 참조하고 있다 등의 오류가 **아예 나오지 않는** 것을 의미합니다. 

- 이런 엄격한 null / undefined 체킹을 하기 위해, ReScript에서는 Option 이라는 키워드를 사용합니다. 자바의 Optional과 일맥상통하는 개념입니다. ~~하지만 거의 쓰는걸 못봤습니다.~~ 

```javascript
type option<'a> = None | Some('a)
```
- 살펴보면, Option이란 특별한 값이 아니라 None 혹은 Some 값을 갖는 Variant의 한 종류임을 알 수 있습니다. 예시를 보겠습니다.

```javascript
let ReScriptExpert = "yes";

//ReScriptExpert라는 값의 'null'일 수도 있는 가능성을 어떻게 표현할 수 있을까요?

let ReScriptExpert = 
    if knowsOptional {
        Some("yes")
    }else{
        None
    }
```
- 이런식으로 Some()과 None으로 wrapping 하는것이 가능합니다. 나중에 이 ReScriptExpert라는 값을 꺼내기 위해서는 Switch 문 안의 조작이 필요합니다.

```javascript
switch ReScriptExpert {
| None =>
  Js.log("The person is not an expert")
| Some(answer) =>
  Js.log(answer ++ "The person is an expert!")
}
```
- ReScript에선 null이 될 수 있는 요소는 optional로 감싸지기 때문에, null 에러가 없습니다. 다시 말해, null 인경우에 대한 처리가 항상 컴파일 시점에 이뤄지므로 예상치 못한 null 에러가 발생하지 않는 것입니다.

- 이 None은 그럼 무엇일까요? Null 일까요 undefined 일까요?
```javascript
let x = None //ReScript
let x //JavaScript
```
- None은 undefined 였습니다. 꽤나 [내부적 논의](https://github.com/rescript-lang/rescript-compiler/issues/4016)가 있었던 것 같은데요, 결론은 'undefined로 정했고 두개를 동시에 지원하기엔 구현적 어려움이 있다' 정도 인 것 같습니다. 

Option을 사용할 때는 유의해야 할 점이 몇가지 있습니다.
```javascript
let x = Some(Some(Some(10)));
let x = Some(None) 
```

각각이 JS로 트랜스파일 된 결과입니다.
```javascript
let x = 10;
var x = Caml_option.some(undefined);
```

- 이런 중첩된 Option은 굉장히 굉장히 피곤할 수 있습니다. 
1. Option wrapping이 언제 끝날 지 미리 알 수 없으며,
2. undefined로 define 하는게 의미 있는 경우가 없기 때문입니다.

- 따라서 [공식문서](https://rescript-lang.org/docs/manual/latest/null-undefined-option)에서는 절대로
1. Option을 중첩해서 사용하지 말 것과
2. JS에서 오는 값들을 option으로 받지 말고, 비 다형적(non-polymorphic)이고 구체적인 타입으로 명시할 것을 당부하고 있습니다.

- 많은 경우에, JS 값은 null 과 undefined로 정의될 때가 있습니다. (value 와 undefined가 아니라요). 이 경우는 어떻게 하면 될까요? Option은 Some과 None 값만 가지기에 null에 대한 체크를 할 수 없습니다. 이럴 때 사용하는 것이 Js.Nullable 입니다. 예시를 보겠습니다.

```javascript
@bs.module("MyConstant") external myId: Js.Nullable.t<string> = "myId"
```

```javascript
@bs.module("MyIdValidator") external validate: Js.Nullable.t<string> => bool = "validate"
let personId: Js.Nullable.t<string> = Js.Nullable.return("abc123")

let result = validate(personId)
```
- 이렇게 하면 return 파트에서 스트링 값을 nullable string으로 감쌉니다. 즉, 받는 측에서는 이 string의 값이 존재할수도, null일수도, undefined 일 수도 있게 되는 것이죠!