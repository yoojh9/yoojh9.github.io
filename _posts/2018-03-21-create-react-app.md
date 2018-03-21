---
layout: post
title: "create-react-app을 이용하여 Redux 프로젝트 생성"
tags: [react]
comments: true
---

## 1. create-react-app 사용
- 페이스북에서 만든 React 프로젝트 생성 도구

<br/>

- create-react-app 전역 설치

```
$ npm install -g create-react-app
```

<br/>

- app 생성

```
 $ create-react-app hello-world
```

<br/>

- start dev-server

```
$ npm start
```

<br/>

- building for production

```
$ npm run build
```

<br/>

## 2. redux 프로젝트를 위한 모듈 설치
- react-redix: 뷰 레이어 바인딩. 컴포넌트에서 redux에 쉽게 연결 가능

```
$ npm install --save redux react-redux
```
