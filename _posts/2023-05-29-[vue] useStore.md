---
title: Vue | useStore()를 사용하는 이유  
categories: [Vue]
tags: [Vue, Vuex, 상태관리]
---

## Vue에서 store 접근하기 

보통 프론트앱에서 데이터의 변경을 감지하고 추적하는 상태관리가 필요한 데이터들은 store에 보관하여 사용한다. store 객체를 생성하고 이를 vue app에 전역적으로 등록한다. 그러면 컴포넌트에서 해당 store에 접근하여 데이터를 get, mutation 등등의 작업을 할 수 있다. 

보통은 store에 접근하는 방법은 다음과 같다. composition api 기준

- vue app에 전역적으로 등록한 store를 this를 통해 접근하는 경우 
  
```javascript 
setup() {
    const some = computed(() => this.$store.getters.something)
}
```

- store 객체를 직접 다이렉트로 import 하는 경우 
```javascript
import {store} from '@/../../store/index.js'

setup() {
    const some = computed(() => store.getters.something)
}
```

기존의 vuex2 또는 vuex3에서는 이렇게 두 가지 방법을 많이 사용한다. vuex4부터는 store에 접근할 수 있는 새로운 api를 지원한다. 

## useStore() 

store 객체에 접근할 수 있는 Vuex function 이다. vuex에서 임포트해서 사용할수 있다. composition api에서 setup hook에 사용할 수 있다.


```javascript 
import { useStore } from 'vuex'

setup() {
    const store = useStore()
    ...
}
```

위의 코드는 기존의 `this.$store` 와 똑같은 기능을 제공한다. 그러나 composition api의 setup 훅에서의 `this` 는 현재의 인스턴스를 참조하지 않는다. 컴포넌트내의 옵션들이 결정되기 전에 setup hook이 호출되기 때문에 this는 아무런 것도 참조할 수 없게 된다. (undefined를 참조함) 따라서 setup hook에서 store를 접근할 수 있도록 새로 나온 함수가 useStore이다. option api를 사용할 경우는 기존과 같이 `this.$store`를 사용하면 된다. 

## 참고 

- [공식문서에서 소개하는 store에 접근하는 방법](https://vuex.vuejs.org/guide/composition-api.html#accessing-state-and-getters)
