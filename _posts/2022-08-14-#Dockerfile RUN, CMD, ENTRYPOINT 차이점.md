---
title: Dockerfile | RUN, CMD, ENTRYPOINT 명령어 차이점
categories: [docker]
tags: [docker, devops ]
---

# Dockerfile 명령어 속성
:whale: :whale: :whale:

- `RUN`
- `CMD`
- `ENTRYPOINT`
  
도커파일을 작성할 때 명령어를 지시할 수 있는 3가지 속성이 있다. 
3가지의 실행 시점과 차이점을 잘 알아야 도커파일을 잘 사용할 수 있다. 
(나처럼 삽질 안 하고...)

### 0. Intro

``` Dockerfile
FROM node:16-alpine as builder
WORKDIR /app
COPY package.json .
COPY yarn.lock .
RUN yarn
COPY . .
EXPOSE 8080
CMD [ "yarn", "build" ]

FROM nginx:stable-alpine as production
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80 
CMD ["nginx", "-g", "daemon off;"]

```

> Q. 이 도커파일에서 틀린 곳을 찾아보시오. 

> A. ??? 


### 1. 내가 의도한 도커파일은 

1. yarn으로 의존성들을 설치하고 이를 /app/dist 폴더 밑에 빌드한다. 
2. 빌드된 산출물들을 nginx로 서빙하기 위해 하위 파일들을 복사 붙여넣기 한다. 
3. nginx 실행 

으로 아주 간단한 nodejs 앱 기반의 빌드 - 배포 시나리오이다. 
기존에는 프론트 앱의 publicPath = `~/repository_name/` 이렇게 설정을 한 상태였다. 굳이 그럴 필요가 없을 것 같아서 해당 설정을 지우고 (vue.config.js파일)
 다시 동일한 도커 파일을 이용해 도커 이미지를 빌드해주었다. 
빌드한 이미지를 실행하고 해당 url을 확인해보니 여전히 default로 모든 요청 파일에 `~/repository_name/`이 붙어 있었다. 즉, 수정 사항이 반영 안 되었다는 얘기다.

정확한 에러는 다음과 같다. 
``` html
<noscript>
      <strong>We're sorry but #myappname@ doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
</noscript>
```
빌드가 정상적으로 되었으나, default 호출 url로 prefix에 `~/repository_name/`이 붙어서 나는 안내 메세지였다. 즉 호출을 잘못하고 있다는 뜻이다. 

이를 해결하기 위해 해본 삽질

- publicPath 설정 바꿔줌 : 효과 x
- vue 인스턴스를 초기화하는 방식 변경 : 효과 x
- 콘솔에 나오는 경고 메세지 관련한 코드 찾아서 수정 : 효과 x 
- 포트 변경 : 효과 x 
- nginx의 html 하위 파일 모두 삭제 후 다시 도커 빌드 : 효과 x 

사실 핵심은 첫번째인데. publicPath 설정을 바꿔준 수정 사항이 당최 도커 이미지에 반영이 안되는 것으로 생각하고 도커파일 관련해서 다시 찾아보던 중....
**CMD는 도커 파일 내에서 한 번만 실행된다는 것을 알았다. 중복해서 쓰였을 경우 가장 마지막이 실제로 호출된다는 것이었다.**

저 위에 도커 파일에서 잘못된 점은 
```Dockerfile
CMD [ "yarn", "build" ]
CMD ["nginx", "-g", "daemon off;"]
```
이 부분으로 CMD를 두 번 작성하여 아래의 CMD문이 반영된 것이다. 
즉 코드를 백만번 수정해도 실제로는 빌드를 하지 않은 셈이다. 
그래서 반영이 안되었던 것이다. 

### 2. 차이점 
RUN과 CMD 모두 명령어겠지~ 라고 생각하고 두 개의 차이점을 인지하지 못하고 작성했던 것이 문제였다. 
찾아보니 관련된 속성으로 ENTRYPOINT라는 녀석까지 하나 더 있다고 한다. 

핵심은 3가지 구문 모두 **실행시점**이 각자 다르다는 것!

1. RUN 
- 실행시점 : docker image가 빌드되는 동안 

도커 엔진이 도커 파일을 한 줄 한 줄 읽으면서 이미지를 빌드할 때 실행된다. 

2. CMD 
- 실행시점 : 빌드가 완료되고 해당 이미지가 실행될 때 (시작)

빌드되는 동안에 실행되는 것이 아니라, 실제로 이미지가 실행되기 시작할때 CMD 구문이 실행된다. 
그리고 docker run 명령에 인자값으로 넘겨서 사용할 수도 있다. 
```shell
docker run {image} {cmd_command}
```
이런 경우에는 이미 dockerfile에 정의된 cmd 구문은 실행되지 않고 docker run에 인자값으로 넘긴 cmd 구문이 실행된다. 

> [도커 공식 문서 중 CMD 관련 부분](https://docs.docker.com/engine/reference/builder/#cmd)
>  
> Do not confuse RUN with CMD. RUN actually runs a command and commits the result; CMD does not execute anything at build time, but specifies the intended command for the image.
> => Run과 CDM를 헷갈리지말 것. Run은 명령구문을 실행하고 결과물을 저장한다. (= 이미지를 빌드한다는 뜻) CMD는 빌드 타임에는 아무것도 실행되지 않는다. 그러나 특정 커맨드를 의도대로 지정할 수 있다. (=동일 이미지에 가변적으로 명령어를 지정할 수 있다는 뜻)

1. ENTRYPOINT
- 실행시점 : CMD랑 같다 

그럼 CMD와의 차이점은 docker run으로 CMD의 내용을 상황에 따라 바꿀 수 있는 것과 달리, 항상 실행된다는 점이다. 즉 dockerfile 내에서 정의한 ENTRYPOINT는 해당 컨테이너 실행시에 변경할 수 없다. 

> [도커 공식 문서 중 CMD와 ENTRYPOINT 차이점 부분](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)
> 1. 도커파일은 최소한 한 개의 CMD 혹은 ENTRYPOINT가 있어야 한다. 
> 2. ENTRYPOINT는 컨테이너를 실행파일로 사용할 때 정의해야 한다. 
> 3. CMD는 ENTRYPOINT의 default 값으로 지정하는 방식으로 사용해야 한다. 
> 4. CMD는 컨테이너를 실행할 때 다시 정의할 수 있다. 

### 3. 요약
- 이미지가 빌드 되는 동안에는 RUN 
- 이미지가 실행될 때, 가변적으로 실행되어야 할 때는 CMD
- 이미지가 실행될 때, 항상 똑같이 실행되어야 할 때는 ENTRYPOINT

### 4. 그래서

> Q. 이 도커파일에서 틀린 곳을 찾아보시오. 

> A. 답은

``` Dockerfile
FROM node:16-alpine as builder
WORKDIR /app
COPY package.json .
COPY yarn.lock .
RUN yarn
COPY . .
EXPOSE 8080
# CMD [ "yarn", "build" ] CMD가 아니라, RUN으로 바꿔야 한다.
RUN yarn build 

FROM nginx:stable-alpine as production
RUN rm -rf /usr/share/nginx/html/*
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80 
CMD ["nginx", "-g", "daemon off;"]

```
CMD가 두 번 사용되었기 때문에 빌드가 안 되고 있었던 상황이다.
빌드라는 것은 이미지가 실행되기 전에 이미 완료되어야하는 작업이므로 빌드 중일 때 실행되는 명령어인 RUN을 사용하는 것이 맞다. 


### 5. Shell form, Exec form 
run, cmd, entrypoint 모두 2가지 form을 지원하고 있다. 
배열로 전달하는 여부에 따라 달라진다. 

``` 
RUN yarn build // shell form 
RUN ["yarn", "build"] // exec form 
```
배열로 사용할 것을 권장하고 있다. 작성한 구문을 도커 엔진이 읽을 때 과정을 한 단계 거치지 않아서 더 깔끔하다는 것...이라고 이해했다.

또한 exec form 으로 사용할 시에는 double quote `""` 을 사용해야 한다. JSON형식으로 명령어들을 파싱하기 때문이라고 한다. 


### 6. 나머지 공부 
정리하면 도커파일 문법을 잘 몰라서 생긴 문제.
https://docs.docker.com/engine/reference/builder/



