---
title: JS에서 안전하게 널값 처리하기 Optional chaining(?.) & Nullish coalescing operator (??)
categories: [JS]
tags: [JS, Javascript, Nullish]
---


## JS에서 널값 안전하게 유연하게 처리하기 

| Uncaught TypeError: Cannot read properties of undefined (reading 'xxx')

위의 에러는 undefined인 객체의 xxx 속성에 접근하려고 할 때 쉽게 만날 수 있는 에러이다. 이런 에러를 만나게 되면 대부분 이전 단계에서 조건문을 통해 undefined인지 체크를 하고 undefined 가 아님을 보장한 상태에서 내부의 속성값을 접근하도록 해주었다. 그러나 `?.`, `??` 연산자를 사용하면 내가 직접 체크를할 필요 없이 좀 더 편하고 유연하게 처리할 수 있다. 

얼마 전에 프로젝트 소스코드를 보다가 처음 보는 자바스크립트 코드를 보게되었다. 삼항연산자 같기도 하고, `.`을 이어 붙여서 속성에 접근하는 것 같기도 하고. 

```javascript
return xxxList?.ooProperty?.xxProperty ?? defaultValue
```

찾아보니 `?.`연산자는 Optional Chaining, `??` 연산자는 Nullish coalescing operator 라고 한다. ES2020 표준으로 나름 최신 문법이다. 

이 두 가지 연산자는 모두 객체의 널값과 관련이 있다. 접근하고자 하는 값이 nullish한 경우 undefined 에러를 뱉지 않고 분기처리할 수 있도록 도와준다. 따라서 해당 속성값을 참조하기 이전에 조건문 등으로 해당 값이 존재하는지 여부를 판단하는 코드를 추가할 필요 없이 속성에 접근하는 연산자만 사용하면 된다. 
## Optional Chaining < ?. >

직독직해하면 선택적 체이닝. 선택적의 대상은 접근하고자하는 값이 nullish한지 여부다. ( nullish = null 또는 undefined.)
nullish 여부에 따라 체이닝을 이어나가기도, 멈추기도할 수 있다.
문법은 `.` 접근 연산자 앞에 `?` 을 붙이면된다. `?.` 연산자를 적용할 수 있는 대상은 객체의 속성값, 표현식, 함수호출 3가지이다. 

```javascript
obj.val?.prop
obj.val?.[expr]
obj.func?.(args)

``` 
`?` **앞의 속성에 대해서 nullish 여부를 체크한다.** (즉, obj.val에 대해서)
  - nullish 하지 않다면 체이닝을 이어나간다. 
  - nullish 하다면 undefined를 리턴하고 체이닝을 이어가지 않고 종료한다. 

```javascript
let car = { 
    power: 240, 
    engine: {
        aaa: 100, 
        bbb: 200, 
        ccc: {
            ...
        }
    }
}

car.engine.ddd.prop // #1. Uncaught TypeError: Cannot read properties of undefined (reading 'prop') 
car.engine.ddd?.prop // #2. return undefined
```

그리고 무엇보다도 nullish한 경우 **type error를 내뱉지 않고** 널값 처리를 할 수 있어 안전하게 널값을 처리할 수 있다는 장점이 있다. 

1. `ddd` 속성이 nullish 하기 때문에 prop에 접근할 수 없어서 TypeError 
2. `ddd?.` 옵셔널 체이닝 연산자를 이용하여 `ddd`가 nullish인지 체크하고 undefined 반환하고 에러 없이 종료. 

`?`가 붙어있는 속성값에 대해서만 체크를 하기 때문에,
`car.engine?.ddd.prop` 으로 접근해도 #1과 같은 에러가 난다. 따라서 `?`을 잘 붙여서 사용해야 한다. 

코드를 작성하는 사람은 데이터 구조를 알고 있기 때문에 당연히 값이 있다고 생각하고 또는 그럴거라고 짐작하고 `.` 연산자로 이어붙여 해당 값에 접근하게 된다. 막상 런타임에서 수많은 값의 변경이 연쇄적으로, 동시다발적으로 일어나게 되면 어느 순간은 null 이나 undefined로 참조되는 경우가 많다. 그렇게 되면 내가 작성한 코드는 TypeError를 내뿜게 되는 것이다. 

보통 중첩된 정도가 깊지 않은 경우에는 앞서 해당 속성값이 존재하는지 여부를 조건문을 통해 별도로 체크해주게 된다. 그러나 optional chaining 연산자는 별도로 체크해주는 과정을 대신 해준다. 

## (실전 적용) Vue 컴포넌트에서 사용해보기 

뷰 컴포넌트의 template 코드 안에서도 이 문법이 적용될까?
사실 이 부분이 제일 궁금했다. 중첩된 객체를 특정 컴포넌트의 props로 넘겨주는 경우가 그렇다. 

```javascript
<template>
    <MyComponent
        :data="myData.files.itemList">
    </MyComponent>
</template>
```
위의 코드에서 만약 files가 undefined라면 콘솔에 TypeError가 날 것이다. (실제로 개발하면서 엄청 많이 겪었다.) 이런 경우에 대비해서 나는 주로 v-if / v-show 디렉티브를 사용했다. 
중첩된 속성 접근 전에 항상 이런 방어적인 코드를 추가하다보니 번거롭다는 생각을 하기도 했고, 실수로 누락되는 부분이 생기기 마련이었다. 그래서 옵셔널 체이닝 문법을 사용하면 사전의 v-if / v-show 를 쓰지 않아도 될 것 같아서 실험을 해보았는데 아직 template 내에서 옵셔널 체이닝 문법을 사용하지는 못한다. 물론 vue2 기준이다.

```javascript
<template>
    <MyComponent
        :data="myData.files?.itemList"> // error 뷰가 이해하지 못한다.
    </MyComponent>
</template>
```
물론, script 태그 내에서는 옵셔널 체이닝 문법 사용이 가능하다.



## Nullish coalescing operator < ?? > 

```javascript
return xxxList?.ooProperty?.xxProperty ?? defaultValue
```
아까 위에 코드를 다시 보면, `??` 연산자도 보인다. 딱봐도 삼항연산자 비스무리하게 생겨서 짐작하기 쉬웠다. 

```javascript
//문법
leftExpr ?? rightExpr

//예시 
const aaa = 0 ?? 'default value' // 'default value'
const bbb = isEmpty() ?? false
```

`??` 연산자는 **논리 연산자**다. `??` 기준으로 앞선 왼쪽 논리식이 null 또는 undefined 라면 오른쪽 값을 반환하고 true하다면 왼쪽 값을 반환하는 연산자이다. 여기서 true로 판단하는 기준은 해당 값이 falsy한지 여부다. (false, '', "", 0, NaN 모두 falsy한 값이므로 true가 된다.) 

MDN 문서에서는 OR 논리 연산자의 특별한 케이스라고 설명하고 있다. `??` 와 `||` 연산자의 차이점은 false 판단 기준이다.

### 거짓 판단 기준

`??` 
   - 오직 null, undefined 값만 false로 판단한다. 빈 문자열, 0, NaN도 값으로 인정하여 true로 판단한다.

`||`
   - null, undefined, '', "", 0, Nan 등 자바스크립트에서 **falsy** 하다고 여겨지는 것들을 모두 false로 처리한다. 

빈 문자열, 0 도 자바스크립트 엔진이 거짓값스럽다?라고 해석하는 걸 볼 수 있다. 이걸 영어로 falsy 하다고 한다. 나도 이번에 처음 알았다. 실제로 코드로 실험해보면 다른 결과가 나오는 걸 볼 수 있다.

```javascript
숫자 0 

0 ?? 'default value' // return 0
0 || 'default value' //return 'default value'

빈 문자열

'' ?? 'default value' // return ''
'' || 'default value' // return 'default value'

```
falsy한 값들도 고유한 값으로 인정하여 true하도록 하고 싶다면 `??` 연산자를 사용하면 된다. 주로 변수의 디폴트 값을 할당할 때 `??`, `||` 연산자를 많이 사용하게 될텐데 변수에 의미를 주고 싶을 때에 따라 적절하게 같이 사용하면 될 것 같다. 

또한 다른 논리 연산자들처럼 왼쪽식이 false이면 오른쪽 식은 검증하지 않는다.


## 안전하게 객체 속성에 접근하기 
MDN 문서에서는 `?.` 연산자와 `??` 연산자를 같이 사용하여 안전하게 속성값에 접근할 것을 권장하고 있다. 여기서 안전하게라는 말은 nullish한 객체의 속성을 접근하고자 하는 시도일 것이다. 
```javascript
foo = { first : { prop: 100}}
foo.first?.prop ?? 'default value'
```
`?.` 연산자로 값을 연속해서 접근해나가다 최종 접근 속성값이 undefined 또는 null 이라면 ?? 연산자 뒤의 값을 할당하는 방식으로 가장 많이 조합되어 쓰일 것 같다. 


## 참고 
- Nullish 하다는 것 = 값이 Null 또는 undefined 
- falsy 하다 = 논리식에서 false로 판단되는 값들 
  - false, 0, -0, ``, '', "", NaN, Null, undefined
- [MDN 문서 optioncal chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
- [MDN 문서 nullish coalescing operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
