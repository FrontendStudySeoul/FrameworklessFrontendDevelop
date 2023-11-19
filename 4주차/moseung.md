# 6장 라우팅

## SPA
단일페이지어플리케이션(SPA)는 하나의 HTML 페이지로 실행되는 웹 애플리케이션이다. 사용자가 다른 뷰로 이동할떄 어플리케이션은 뷰를 동적으로 다시 그린다.
<br>
<img width="633" alt="image" src="https://github.com/FrontendStudySeoul/FrameworklessFrontendDevelop/assets/103626175/b779ae3c-afb6-4fe5-8867-62f8867d26d9">
이 SPA방식은 앵귤러와 앰버같은 프레임워크 덕분에 현재 주류가 됐다.
### 코드 예제
5장과 마찬가지로 라우팅을 세가지 버전으로 작성한다. 우선 프레임워크 없는 방식으로 프래그먼트 식별자, 히스토리를 기반으로 한다.
```tsx
//index.js
import createRouter from './router.js'
import createPages from './pages.js'

const container = document.querySelector('main')

const pages = createPages(container)

const router = createRouter()

router
  .addRoute('#/', pages.home)
  .addRoute('#/list', pages.list)
  .setNotFound(pages.notFound)
  .start()

//pages.js
export default container => {
  const home = () => {
    container
      .textContent = 'This is Home page'
  }

  const list = () => {
    container
      .textContent = 'This is List Page'
  }

  const notFound = () => {
    container
      .textContent = 'Page Not Found!'
  }

  return {
    home,
    list,
    notFound
  }
}

//router.js
export default () => {
  const routes = []
  let notFound = () => {}

  const router = {}

  const checkRoutes = () => {
    const currentRoute = routes.find(route => {
      return route.fragment === window.location.hash
    })

    if (!currentRoute) {
      notFound()
      return
    }

    currentRoute.component()
  }

  router.addRoute = (fragment, component) => {
    routes.push({
      fragment,
      component
    })

    return router
  }

  router.setNotFound = cb => {
    notFound = cb
    return router
  }

  router.start = () => {
    window
      .addEventListener('hashchange', checkRoutes)

    if (!window.location.hash) {
      window.location.hash = '#/'
    }

    checkRoutes()
  }

  return router
}
```
라우터는 세가지 공개 메스드를 가지며 addRoute를 통해 추가하고 setNotFound를 통해 없는 페이지에 대한 정리(404) React-router-dom으로 생각하면 주소에 /*을 넣는것 start에는 hashchange이벤트를 등록하여 새로운 페이지를 렌더링하는것.
<br>
checkRoute가 핵심 함수인데 해당 함수는 현재 프래그먼트와 일치하는 경로를 찾아서 있다면 해당 페이지를 보여주고 없으면 404페이지를 렌더링한다.
<img width="530" alt="image" src="https://github.com/FrontendStudySeoul/FrameworklessFrontendDevelop/assets/103626175/2bd8a708-12a1-41f2-a4fe-787ed955dddd">

### 프로그래밍 방식으로 탐색
만약 특정 상황에서 로그인이 돼있으면 마이페이지로 이동시키는 프로그래밍 규칙이 추가된다고 생각해보자
```tsx
//html
 <header>
        <button data-navigate="/">
            Go To Index
        </button>
        <button data-navigate="/list">
            Go To List
        </button>
        <button data-navigate="/dummy">
            Dummy Page
        </button>
    </header>

//비즈니스 로직
const NAV_BTN_SELECTOR = 'button[data-navigate]'

document
  .body
  .addEventListener('click', e => {
    const { target } = e
    if (target.matches(NAV_BTN_SELECTOR)) {
      const { navigate } = target.dataset
      router.navigate(navigate)
    }
  })


```
위에서 taget.dataset을 console찍어보면 다음 사진과 같다
<br>
<img width="564" alt="image" src="https://github.com/FrontendStudySeoul/FrameworklessFrontendDevelop/assets/103626175/74107efe-02c4-412a-ac8a-8a7938b191d6">
<br>
### 번외 
만약에 reactRouterDom에서 프로토콜을 입력하지않은 URI를 입력하면 어떻게 될까요?
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>


```
React Router는 웹 애플리케이션 내부에서의 라우팅을 관리하는 라이브러리입니다. 이 라이브러리는 내부적으로 'history'라는 JavaScript 라이브러리를 사용하여 브라우저의 히스토리 스택, URL 변경 등을 관리합니다.
React Router에서 URL을 다룰 때, URL이 '/'로 시작하면 그것을 절대 경로로 해석하고, 그렇지 않으면 상대 경로로 해석합니다. 따라서 'www.jungnang.go.kr/'처럼 입력하면, React Router는 이를 상대 경로로 인식하고 현재 URL에 추가됩니다.
이와 달리 'http://' 또는 'https://' 등의 프로토콜을 포함한 URL을 사용하면 브라우저는 이를 외부 링크로 인식하고, 이 경우 React Router는 이를 처리하지 않습니다. 따라서 외부 링크를 열려면 'window.location'이나 'window.open()'과 같은 JavaScript의 내장 메서드를 사용해야 합니다.
즉, React Router는 애플리케이션 내부의 라우팅을 관리하는 데 사용되며, 외부 링크를 열기 위해서는 프로토콜을 명시적으로 포함한 URL을 사용해야 합니다.
```


### 경로 매개변수
예를들어 http://localhost:8080#/order/1과 같은 주문페이지에 대한 동적으로 라우팅을 적용할떄는 어떻게 해야할까? 이를 책에서 경로 매개변수라고 부른다.
<br>
다음 코드처럼 detail페이지에서 params가 추가된다.
```tsx
export default container => {
  const home = () => {
    container
      .textContent = 'This is Home page'
  }

  const list = () => {
    container
      .textContent = 'This is List Page'
  }

  const detail = (params) => {
    const { id } = params
    container
      .textContent = `This is Detail Page with Id ${id}`
  }

  const anotherDetail = (params) => {
    const { id, anotherId } = params
    container
      .textContent = `
        This is Detail Page with Id ${id} 
        and AnotherId ${anotherId}
      `
  }

  const notFound = () => {
    container
      .textContent = 'Page Not Found!'
  }

  return {
    home,
    list,
    detail,
    anotherDetail,
    notFound
  }
}

```

관련 로직을 추가했으니 실제로 id를 포함한 값이 해시에 추가됐을떄 페이지로 라우팅하는 비즈니스 로직을 정리한다.

```tsx
//router.js
const ROUTE_PARAMETER_REGEXP = /:(\w+)/g
const URL_FRAGMENT_REGEXP = '([^\\/]+)'

router.addRoute = (fragment, component) => {
    const params = []

    const parsedFragment = fragment
      .replace(
        ROUTE_PARAMETER_REGEXP,
        (match, paramName) => {
          params.push(paramName)
          return URL_FRAGMENT_REGEXP
        })
      .replace(/\//g, '\\/')

    console.log(`^${parsedFragment}$`)

    routes.push({
      testRegExp: new RegExp(`^${parsedFragment}$`),
      component,
      params
    })

    return router
  }
```
### 히스토리 API
히스토리 API를 통해 개발자는 사용자 탐색 히스토리를 조작할 수 있다<br>
![KakaoTalk_Photo_2023-11-19-14-24-59](https://github.com/FrontendStudySeoul/FrameworklessFrontendDevelop/assets/103626175/7ad775a1-8671-43ad-83e7-435ffe7dff66)<br>
그래서 아까전 해시와 달라진점은 히스토리 API를 사용한다는점이다.
```tsx
//hash
 const currentRoute = routes.find(route => {
      return route.fragment === window.location.hash
    })

router.navigate = fragment => {
    window.location.hash = fragment
  }
//history
const { pathname } = window.location
    if (lastPathname === pathname) {
      return
    }

router.navigate = path => {
    window
      .history
      .pushState(null, null, path)
  }
```

### 라이브러리 사용하기
[Navigo](https://github.com/krasimir/navigo)라는 라이브러리를 사용하여 중요 비즈니스 로직을 라이브러리에 위임할 수 있다.

```tsx
export default () => {
  const navigoRouter = new window.Navigo()
  const router = {}

  router.addRoute = (path, callback) => {
    navigoRouter.on(path, callback)
    return router
  }

  router.setNotFound = cb => {
    navigoRouter.notFound(cb)
    return router
  }

  router.navigate = path => {
    navigoRouter.navigate(path)
  }

  router.start = () => {
    navigoRouter.resolve()
    return router
  }

  return router
}

```

### 요약
- 두개의 방식으로 Vanialla JS에서 라우팅을 구현 가능하다. 첫번쨰는 해시값 두번쨰는 History API를 이용하는것.

### 일어보면 좋을것들
- [SPA와 MPA의 차이](https://it-techtree.tistory.com/entry/Web-Advantage-And-Disadvantage-SPA-MPA)
- [프론트엔드개발의 역사와 미래](https://yozm.wishket.com/magazine/detail/1289/)
