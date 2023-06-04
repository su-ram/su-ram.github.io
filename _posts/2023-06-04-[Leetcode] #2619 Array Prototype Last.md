---
title: Leetcode | #2619 Array Prototype Last, #2620 Counter  
categories: [알고리즘]
tags: [Leetcode, Problem Solving, 알고리즘, JS]
---

자바스크립트 easy 2문제를 풀었다. 

## [2619. Array Prototype Last](https://leetcode.com/problems/array-prototype-last/)

자바스크립트 Array 객체의 프로토타입에 `last` 라는 새로운 메소드를 추가하는 것이 문제. 
`last` 함수는 배열의 마지막 값을 리턴한다. 빈 배열이면 -1.

간단하게는 삼항 연산자로 해결할 수 있지만, 나는 Nullish coalescing operator ?? 를 사용했다. 
```javascript
Array.prototype.last = function() {
    return this.at(-1) ?? -1
};

```
배열에 직접 인덱스로 접근할 때는 음수 인덱싱이 불가능하다. `at()` 메소드 사용할 때만 가능하다. 
여기서 자바스크립트 개념과 연결해볼만 한 부분은 `this` 객체. 
Array 프로토타입 내의 메소드들의 this는 실제 데이터를 담고 있는 배열을 가리킨다. 


## [2620. Counter](https://leetcode.com/problems/counter/)

`n`이 주어졌을 때, 호출할 때마다 n의 값이 1씩 증가하는 함수를 만들 것. 예시코드를 보면 함수를 리턴하는 함수를 작성해야 한다. 처음 createCounter의 파라미터로 `n` 을 받고 1씩 증가하는 함수를 리턴한다. 

호이스팅의 개념을 적용하여 리턴하는 함수에서 바로 상위 함수에서 파라미터로 받은 `n`을 증가연산하여 리턴한다. 호이스팅은 함수가 선언될 시의 주변 환경값을 기억하는 특징이 있기 때문에 내부 함수의 파라미터로는 아무것도 넘겨주지 않아도 되는 것이다. 

createCounter(10) 을 호출한 순간 10이 함수 내부적으로 저장되어, counter()로 아무런 파라미터값 없이 호출해도 내부에서는 10을 기억하고 있다. 




```javascript
/**
 * @param {number} n
 * @return {Function} counter
 */
var createCounter = function(n) {
    return function() {
        return n++
    };
};

/** 
 * const counter = createCounter(10)
 * counter() // 10
 * counter() // 11
 * counter() // 12
 */

```

