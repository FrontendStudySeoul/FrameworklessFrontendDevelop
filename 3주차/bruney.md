# 웹 컴포넌트

## 들어가기
오늘날 개발자들이 사용하는 주요 FE 프레임워크의 공통점: UI 구성을 위한 기본 블록으로 컴포넌트를 사용함. -> 하지만 거의 모든 최신 브라우저에서 **웹 컴포넌트**라고 하는 네이티브 API 셋을 사용해 컴포넌트를 작성할 수 있음.

## API
웹 컴포넌트의 세 가지 중요 기술
1. HTML 템플릿
- template 태그: 콘텐츠가 렌더링X, JS 코드에서 동적인 콘텐츠를 생성하는 **스탬프**로 사용되도록 하려는 경우에 유용
2. 사용자 정의 컴포넌트
- 개발자가 완전한 기능을 갖춘 자신만의 DOM 요소를 작성할 수 있음.
3. Shadow DOM
- 웹 컴포넌트가 컴포넌트 외부의 DOM에 영향을 받지 않아야 하는 경우에 유용
- 다른 사람들과 공유할 수 있도록 컴포넌트 라이브러리나 위젯을 작성하려는 경우 매우 유용함.

브라우저 호환성을 맞추기 위해 폴리필을 사용할 수 있음. but IE를 지원해야 하는 경우 많은 폴리필이 추가돼야 하기 때문에 비추천.

> Shadow DOM vs 가상 DOM<br/>Shadow DOM: 캡슐화와 관련됨.<br/>가상 DOM: 성능과 관련됨.

## 사용자 정의 컴포넌트
웹 컴포넌트의 핵심 요소. `<app-calendar />`

사용자 정의 컴포넌트 API를 사용해 사용자 정의 태그를 작성할 대는 **대시로 구분된 두 단어 이상의 태그**를 사용해야 한다. **한 단어 태그는 W3C에서만 단독으로 사용할 수 있다.**

> 사용자 정의 컴포넌트는 HTML 요소를 확장하는 JS 클래스일 뿐.

```js
export default class HelloWord extends HTMLElement {
  connectedCallback() {
    widnow.requestAnimationFrame(() => {
      this.innerHTML = `<div>Hello World!</div>`
    })
  }
}
```

connectedCallback은 사용자 정의 컴포넌트의 라이프사이클 메서드 중 하나. 컴포넌트가 DOM에 연결될 때 호출됨.

React의 componentDidMount와 매우 유사. 예제처럼 컴포넌트의 콘텐츠 렌더링 or 타이머 시작 or Data Fetching하기 좋은 장소임.

마찬가지로 DOM에서 삭제될 때 disconnectedCallback이 호출되는데, 정리 작업에서 유용한 메서드.

새로 생성된 이 컴포넌트를 사용하려면 브라우저 구성 요소 레지스트리에 추가해야 함. -> 태그 이름을 사용자 컴포넌트 클래스에 연결하는 것을 의미함. 이제 `<hello-world />` 처럼 사용할 수 있음.

```js
import HelloWorld from '...'

window.customElements.define('hello-world', HelloWorld);
```

## 속성 관리
가장 중요한 기능: 개발자가 어떤 프레임워크와도 호환되는 새로운 컴포넌트를 만들 수 있다는 것. 달성하려면 컴포넌트에 다른 표준 HTML 요소와 동일한 공용 API가 있어야 한다. 즉, HTML 요소의 특성을 잘 기억하고 있어야 한다.

```js
export default class HelloWord extends HTMLElement {
  get color() {
    return this.getAttribute('color')
  }

  set color(value) {
    this.setAttribute('color', value)
  }

  connectedCallback() {
    widnow.requestAnimationFrame(() => {
      const div = document.createElement('div')
      div.textContent = 'Hello World!'
      div.style.color = this.color
      this.appendChild(div)
    })
  }
}

<hello-world color='red' />
```

장점
- 다른 개발자가 컴포넌트를 쉽게 릴리즈하는 데 도움이 됨. -> 모든 사람이 사용할 수 있음.

단점
- HTML 속성은 문자열인데 문자열이 아닌 속성이 필요한 경우 먼저 속성을 변환해야 한다.

하지만 이 제약 조건은 다른 개발자에게 컴포넌트를 제시해야 하는 경우에만 유효. 실제 앱에서 대부분의 컴포넌트는 게시될 필요가 없음.

## attributeChangedCallback
초기 렌더링 후 속성을 클릭 이벤트 핸들러로 변경하면 어떻게 될까?

```js
const changeColorTo = color => {
  document.querySelectorAll('hello-world').forEach(helloWorld => helloWorld.color = color)
}

document.querySelector('button').addEventListener('click', () => changedColorTo('blue'))
```

- 예상: 버튼을 클릭하면 모든 hello-world 컴포넌트의 color 속성이 파란색으로 변경됨.
- 실제: 아무 일도 일어나지 않음.
- 해결책: 세터 자체에 일종의 DOM 조작을 추가(빠르지만 지저분함.)
```js
set color(value) {
  this.setAttribute('color', value)
  // 새로운 색상으로 DOM 업데이트
}
```

but 세터 대신 setAttribute 메서드를 사용하면 DOM도 업데이트되지 않았기 때문에 매우 취약함. -> attributeChangedCallback을 사용할 것.

```js
export default class HelloWord extends HTMLElement {
  // 위와 동일

  attributeChangedCallback(name, oldValue, newValue) {
    if(!this.div) return
    if(name === 'color') this.div.style.color = newValue
  }
}
```

변경된 속성의 이름, 속성의 이전 값, 새로운 값을 매개변수로 받음.

> 모든 속성이 attributeChangedCallback을 트리거하지는 않으며 observedAttributes 배열에 나열된 속성만 트리거함.

## 가상 DOM과 통합
```js
const createDomElement = color => {
  const div = document.createElement('div')
  div.textContent = 'Hello World!'
  div.style.color = this.color
  return div
}

export default class HelloWord extends HTMLElement {
  static get observedAttributes() {
    return ['color']
  }

  get color() {
    return this.getAttribute('color')
  }

  set color(value) {
    this.setAttribute('color', value)
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if(!this.hasChildNodes()) return
    applyDiff(
      this,
      this.firstElementChild,
      createDomElement(newValue)
    )
  }

  connectedCallback() {
    widnow.requestAnimationFrame(() => {
      this.appendChild(createDomElement(this.color))
    })
  }
}
```

## 사용자 정의 이벤트
<details>
<summary>data fetching 코드</summary>

```js
const ERROR_IMAGE = 'https://files-82ee7vgzc.now.sh'
const LOADING_IMAGE = 'https://files-8bga2nnt0.now.sh'
const AVATAR_LOAD_COMPLETE = 'AVATAR_LOAD_COMPLETE'
const AVATAR_LOAD_ERROR = 'AVATAR_LOAD_ERROR'

export const EVENTS = {
  AVATAR_LOAD_COMPLETE,
  AVATAR_LOAD_ERROR
}

const getGitHubAvatarUrl = async user => {
  if (!user) {
    return
  }

  const url = `https://api.github.com/users/${user}`

  const response = await fetch(url)
  if (!response.ok) {
    throw new Error(response.statusText)
  }
  const data = await response.json()
  return data.avatar_url
}

export default class GitHubAvatar extends HTMLElement {
  constructor () {
    super()
    this.url = LOADING_IMAGE
  }

  get user () {
    return this.getAttribute('user')
  }

  set user (value) {
    this.setAttribute('user', value)
  }

  render () {
    window.requestAnimationFrame(() => {
      this.innerHTML = ''
      const img = document.createElement('img')
      img.src = this.url
      this.appendChild(img)
    })
  }

  onLoadAvatarComplete () {
    const event = new CustomEvent(AVATAR_LOAD_COMPLETE, {
      detail: {
        avatar: this.url
      }
    })

    this.dispatchEvent(event)
  }

  onLoadAvatarError (error) {
    const event = new CustomEvent(AVATAR_LOAD_ERROR, {
      detail: {
        error
      }
    })

    this.dispatchEvent(event)
  }

  async loadNewAvatar () {
    const { user } = this
    if (!user) {
      return
    }
    try {
      this.url = await getGitHubAvatarUrl(user)
      this.onLoadAvatarComplete()
    } catch (e) {
      this.url = ERROR_IMAGE
      this.onLoadAvatarError(e)
    }

    this.render()
  }

  connectedCallback () {
    this.render()
    this.loadNewAvatar()
  }
}
```

</div>
</details>

```js
window.customElements.define('github-avatar', GitHubAvatar)

document
  .querySelectorAll('github-avatar')
  .forEach(avatar => {
    avatar
      .addEventListener(
        EVENTS.AVATAR_LOAD_COMPLETE,
        e => {
          console.log(
            'Avatar Loaded',
            e.detail.avatar
          )
        })

    avatar
      .addEventListener(
        EVENTS.AVATAR_LOAD_ERROR,
        e => {
          console.log(
            'Avatar Loading error',
            e.detail.error
          )
        })
  })
```

## TodoMVC에 웹 컴포넌트 적용


<details>
<summary>List.js</summary>

```js
const TEMPLATE = '<ul class="todo-list"></ul>'

export const EVENTS = {
  DELETE_ITEM: 'DELETE_ITEM'
}

export default class List extends HTMLElement {
  static get observedAttributes () {
    return [
      'todos'
    ]
  }

  get todos () {
    if (!this.hasAttribute('todos')) {
      return []
    }

    return JSON.parse(this.getAttribute('todos'))
  }

  set todos (value) {
    this.setAttribute('todos', JSON.stringify(value))
  }

  onDeleteClick (index) {
    const event = new CustomEvent(
      EVENTS.DELETE_ITEM,
      {
        detail: {
          index
        }
      }
    )

    this.dispatchEvent(event)
  }

  createNewTodoNode () {
    return this.itemTemplate
      .content
      .firstElementChild
      .cloneNode(true)
  }

  getTodoElement (todo, index) {
    const {
      text,
      completed
    } = todo

    const element = this.createNewTodoNode()

    element.querySelector('input.edit').value = text
    element.querySelector('label').textContent = text

    if (completed) {
      element.classList.add('completed')
      element
        .querySelector('input.toggle')
        .checked = true
    }

    element
      .querySelector('button.destroy')
      .dataset
      .index = index

    return element
  }

  updateList () {
    this.list.innerHTML = ''

    this.todos
      .map(this.getTodoElement)
      .forEach(element => {
        this.list.appendChild(element)
      })
  }

  connectedCallback () {
    this.innerHTML = TEMPLATE
    this.itemTemplate = document
      .getElementById('todo-item')

    this.list = this.querySelector('ul')

    this.list.addEventListener('click', e => {
      if (e.target.matches('button.destroy')) {
        this.onDeleteClick(e.target.dataset.index)
      }
    })

    this.updateList()
  }

  attributeChangedCallback () {
    this.updateList()
  }
}
```
</details>

<details>
<summary>App.js</summary>

```js
import { EVENTS } from './List.js'

export default class App extends HTMLElement {
  constructor () {
    super()
    this.state = {
      todos: [],
      filter: 'All'
    }

    this.template = document
      .getElementById('todo-app')
  }

  deleteItem (index) {
    this.state.todos.splice(index, 1)
    this.syncAttributes()
  }

  addItem (text) {
    this.state.todos.push({
      text,
      completed: false
    })
    this.syncAttributes()
  }

  syncAttributes () {
    this.list.todos = this.state.todos
    this.footer.todos = this.state.todos
    this.footer.filter = this.state.filter
  }

  connectedCallback () {
    window.requestAnimationFrame(() => {
      const content = this.template
        .content
        .firstElementChild
        .cloneNode(true)

      this.appendChild(content)

      this
        .querySelector('.new-todo')
        .addEventListener('keypress', e => {
          if (e.key === 'Enter') {
            this.addItem(e.target.value)
            e.target.value = ''
          }
        })

      this.footer = this
        .querySelector('todomvc-footer')

      this.list = this.querySelector('todomvc-list')
      this.list.addEventListener(
        EVENTS.DELETE_ITEM,
        e => {
          this.deleteItem(e.detail.index)
        }
      )

      this.syncAttributes()
    })
  }
}
```

</details>

이 컴포넌트는 속성이 없는 대신 내부 상태를 가진다. DOM의 이벤트는 이 상태를 변경한 다음 컴포넌트가 `syncAttributes` 메서드에서 해당 상태를 하위 속성과 동기화한다.

## 웹 컴포넌트와 렌더링 함수
### 코드 스타일
HTML 요소를 확장해야 하므로 클래스 작업이 필요함. FP를 선호한다면 이 방식이 불편할 수 있음.

### 테스트 가능성
JSDOM이 현재 사용자 정의 컴포넌트를 지원하지 않는다. 사용자 정의 컴포넌트를 테스트하려면 실제 브라우저를 puppeteer같은 도구와 함께 사용해야 하지만 테스트 속도가 느리고 복잡함.

### 휴대성
웹 컴포넌트는 휴대성이 좋아야 한다. 다른 DOM 요소와 동일하게 동작한다는 사실은 다른 애플리케이션 간에 동일한 컴포넌트를 사용해야 하는 경우 핵심 기능이 된다.

## 사라지는 프레임워크
사라지는 프레임워크: 웹 컴포넌트의 등장으로 생긴 흥미로운 부작용

제품 번들을 제작할 때 출력은 표준 웹 컴포넌트가 된다. 즉, 컴파일 시간에 프레임워크는 사라진다.
