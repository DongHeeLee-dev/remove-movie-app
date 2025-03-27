# Vanilla JS SPA 프레임워크 클론

- 목표: React의 핵심 구조(Component, Router, Store)를 Vanilla JS로 직접 구현

1.  Component 클래스: 최상위 요소 생성, props/state 관리, render()

2.  Router: 해시 기반 SPA 라우팅 처리 및 쿼리 파싱

3.  Store: subscribe 기반 상태 변경 감지 & UI 자동 반영 구조

# ✨ Vanilla JS SPA Framework Clone

React의 핵심 개념인 **Component**, **Router**, **Store** 구조를 바닐라 자바스크립트로 직접 구현한 개인 학습 프로젝트입니다.  
프레임워크 없이도 SPA(싱글 페이지 애플리케이션)가 동작하는 원리를 이해하고 재현하는 데 중점을 두었습니다.

---

## 📌 주요 기능

| 기능               | 설명                                                        |
| ------------------ | ----------------------------------------------------------- |
| 🧩 컴포넌트 시스템 | `Component` 클래스를 통해 props, state, render 구조 구현    |
| 🔄 상태 관리       | `Store` 클래스에서 옵저버 패턴 기반의 상태 구독 시스템 설계 |
| 🔀 라우팅 처리     | `Router` 함수로 해시 기반의 SPA 라우팅 및 페이지 전환 구현  |
| 💡 쿼리 파싱       | URL 쿼리스트링을 객체로 파싱하여 `history` 상태에 저장      |
| ⚡ 자동 렌더링     | 상태 변경 시 구독된 콜백을 실행하여 UI를 자동으로 업데이트  |
| 🧼 구조적 설계     | 컴포넌트, 라우터, 스토어를 분리하여 모듈화 구조 구현        |

---

## 🗂️ 프로젝트 구조

### 💡 설명

- `core/`: React 구조를 참고해 만든 **프레임워크 핵심 로직**들이 위치합니다.
- `components/`: 각 화면별 컴포넌트를 구성하며, **UI 재사용성과 분리된 책임**을 고려해 설계했습니다.
- `App.js`: 루트 컴포넌트로, 전체 앱의 구성과 라우팅을 조립합니다.
- `main.js`: 진입점 파일로, 초기 라우팅 설정 및 App 렌더링을 담당합니다.

---

## 💻 주요 코드 요약

### 🔹 Component

```js
export class Component {
  constructor({ tagName = "div", props = {}, state = {} } = {}) {
    this.el = document.createElement(tagName)
    this.props = props
    this.state = state
    this.render()
  }
  render() {}
}
```

### 🔹 Store

```js
export class Store {
  constructor(state) {
    this.state = {}
    this.observers = {}
    for (const key in state) {
      Object.defineProperty(this.state, key, {
        get: () => state[key],
        set: (val) => {
          state[key] = val
          if (Array.isArray(this.observers[key])) {
            this.observers[key].forEach((observer) => observer(val))
          }
        },
      })
    }
  }

  subscribe(key, cb) {
    this.observers[key] = this.observers[key] || []
    this.observers[key].push(cb)
  }
}
```

### 🔹 Router

```js
function routeRender(routes) {
  const routerView = document.querySelector("router-view")
  const [hash] = location.hash.split("?")
  const currentRoute = routes.find((route) =>
    new RegExp(`${route.path}/?$`).test(hash)
  )
  routerView.innerHTML = ""
  routerView.append(new currentRoute.component().el)
}
```

1️⃣ Component.js – 컴포넌트 베이스 클래스

📍 역할:

1. 각 UI 컴포넌트를 생성하고 렌더링하는 기본 클래스

2. props와 state를 관리하여 데이터 기반 UI 업데이트 가능

3. React의 Component 개념을 순수 JS로 구현

주요 개념:

1. this.el → HTML 요소를 생성

2. this.props → 부모 컴포넌트에서 받은 데이터

3. this.state → 컴포넌트 내부에서 관리하는 데이터

4. render() → 실제 HTML을 화면에 렌더링

```js
export class Component {
  constructor(payload = {}) {
    const { tagName = "div", props = {}, state = {} } = payload
    this.el = document.createElement(tagName)
    this.props = props
    this.state = state
    this.render()
  }
  render() {
    // 각 컴포넌트에서 이 메서드를 오버라이딩하여 구현!
  }
}
```

Router.js – SPA 라우터  
📍 역할:

1. 싱글 페이지 애플리케이션(SPA) 라우팅을 담당

2. 브라우저의 location.hash를 활용하여 현재 URL을 감지

3. URL 변경 시 해당하는 페이지를 렌더링

📌 주요 개념:

1. location.hash → 현재 해시(경로)를 가져와서 라우팅 처리

2. history.replaceState() → URL 상태 변경

3. queryString을 객체 형태로 변환하여 전달

```js
function routeRender(routes) {
  if (!location.hash) {
    history.replaceState(null, "", "/#/")
  }
  const routerView = document.querySelector("router-view")
  const [hash, queryString = ""] = location.hash.split("?")

  const query = queryString.split("&").reduce((acc, cur) => {
    const [key, value] = cur.split("=")
    acc[key] = value
    return acc
  }, {})
  history.replaceState(query, "")

  const currentRoute = routes.find((route) =>
    new RegExp(`${route.path}/?$`).test(hash)
  )
  routerView.innerHTML = ""
  routerView.append(new currentRoute.component().el)
  window.scrollTo(0, 0)
}

export function createRouter(routes) {
  return function () {
    window.addEventListener("popstate", () => {
      routeRender(routes)
    })
    routeRender(routes)
  }
}
```

3️⃣ Store.js – 상태 관리 시스템  
📍 역할:

1. 전역 상태(데이터)를 관리하여 여러 컴포넌트에서 공유 가능

2. subscribe()를 활용하여 특정 상태 변경 시 UI를 자동 업데이트

3. 옵저버 패턴을 적용하여 React의 상태 관리 방식과 유사

📌 주요 개념:

1. this.state → 전역 상태 데이터

2. this.observers → 상태 변경을 감지하는 콜백 함수 저장

3. subscribe() → 특정 상태 값이 변경되면 자동으로 UI 업데이트

```js
export class Store {
  constructor(state) {
    this.state = {}
    this.observers = {}

    for (const key in state) {
      Object.defineProperty(this.state, key, {
        get: () => state[key],
        set: (val) => {
          state[key] = val
          if (Array.isArray(this.observers[key])) {
            this.observers[key].forEach((observer) => observer(val))
          }
        },
      })
    }
  }

  subscribe(key, cb) {
    Array.isArray(this.observers[key])
      ? this.observers[key].push(cb)
      : (this.observers[key] = [cb])
  }
}
```
