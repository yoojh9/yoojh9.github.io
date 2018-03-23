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

<br/>

## 3. react-redux
- 뷰 레이어 바인딩 도구.
- 스토어를 뷰에 연결하기 위해서 Redux는 약간의 도움이 필요데 이 둘을 연결해 주는 것이 바로 뷰 레이어 바인딩이다. React를 쓴다면 react-redux가 이 역할을 한다
- 뷰 레이어 바인딩은 모든 컴포넌트를 스토어에 연결하는 역할을 한다.
- 뷰 레이어 바인딩은 세 개의 컨셉을 가지고 있다.
  - 공급 컴포넌트(Provider Component): 컴포넌트 트리를 감싸는 컴포넌트이다. connect()를 이용해 루트 컴포넌트 밑의 컴포넌트들이 스토어에 연결되기 쉽게 만들어준다
  - connect(): react-redux가 제공하는 함수로 컴포넌트가 애플리케이션 상태 업데이트를 받고 싶으면 connect()를 이용해서 컴포넌트를 감싸주면 된다. 그러면 connect()가 selector를 이용해서 필요한 모든 연결을 만들어준다
  - selector: 직접 만들어야 되는 함수이다. 애플리케이션 상태 안의 어느 부분이 컴포넌트에 props로써 필요한 것인지 지정한다.
<br/>

#### 3-1. Provider
- 컴포넌트에서 redux를 사용하도록 서비스를 제공. Provider는 하나의 컴포넌트이다. 컴포넌트를 ReactDOM으로 페이지에 렌더링 할 때 해당 컴포넌트를 Provider 컴포넌트 안에 감싸면 Provider가 복잡한 작업들을 알아서 해줌.

```
<Provider store={store}>
  <App/>
</Provider>
```  

<br/>

#### 3-2. connect([...options])  
- 컴포넌트를 REDUX에 연결하는 함수를 반환한다. 예를 들어 connect()(Counter) 와 같이 Counter를 인수로 전달해 주게 되면 Counter가 stor에 연결된 새로운 컴포넌트 클래스가 반환된다. 그렇다고 기존에 있던 Counter 컴포넌트가 변경되는 것은 아님. 만약 connect에 옵션을 전달하지 않았다면 컴포넌트 내부에서 this.props.store로 접근 가능.  

```
connect(
  [mapStateToProps],
  [mapDispatchToProps],
  [mergeProps],
  [options]
)
```  

- 첫번째 3개는 함수 형태의 파라미터로,
  - mapStateToProps: state를 파라미터로 갖는 함수로 state를 해당 컴포넌트의 props로 연결해줌
  - mapDispatchToProps: dispatch를 파라미터로 갖는 함수로 dispatch한 함수는 props로 연결해줌
  - mergeProps: state와 dispatch를 파라미터로 가져서 컴포넌트에 연결해야 할 props가 state와 dispatch 동시에 사용해야 할 때 사용(잘 사용 안함)
  - options <- {[pure = true], [withRef = false]}

<br/>

---
#### 참고
[create-react-app 예제 코드](https://github.com/yoojh9/react-example/tree/master/redux-example) <br/>
