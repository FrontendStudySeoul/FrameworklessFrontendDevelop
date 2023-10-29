
# 렌더링
모든 웹 프레임워크에서 제일 중요한 기능 중 하나는 데이터의 표시다.
### 이 장에서 설명하는것
- 프레임우크 없이 DOM을 효과적으로 조작하는 방법을 배우는 법.

### 문서 객체 모델(DOM)
DOM은 웹 어플리케이션을 구성하는 요소를 조작할 수 있는 API다.
<br>
기술적 관점에서 보면 모든 HTML 페이지는 트리로 구성된다. 그리고 DOM이 HTML 요소로 정의된 트리를 관리하는 방법임을 알 수 있다.
<br>
예를들어 리액트 셀의 배경색을 변경하려면 다음과 같이 작성하면 된다.
```tsx
const SELECTOR = "tr:nth-child(3) > td"
const cell = document.quertSelector(SELECTOR)
cell.stylee.backgroundColor = "red"
```

### 렌더링 성능 모니터링
웹용 렌더링 엔진을 설계할 떄는 가독성과 유지 관리성을 염두에 둬야 한다. 그리고 렌더링 엔진에서 중요한것은 성능이다.
<br>
크롬 개발자 도구를 사용해 ctrl+shift+p 를 누르고 ``Show frame per seconds (FPS) meter``를 검색하면 다음과 같이 프레임을 알 수 있다.
<img width="257" alt="image" src="https://github.com/FrontendStudySeoul/FrontEndDevelopNoFramework/assets/103626175/2039db42-8583-488e-9249-d5d1b93ba0e6">
<br>
혹은 stats.js라이브러리를 활용하여 표시할 수 있다.
```
사람이 편안하게 느끼는 프레임 속도는 주로 30fps(프레임/초)에서 60fps(프레임/초) 사이입니다.

일반적으로 30fps는 대부분의 상황에서 자연스럽게 보이는 프레임 속도입니다. 이는 대부분의 영화, TV 프로그램, 비디오 게임에서 사용되는 표준 프레임 속도입니다.

그러나 더 높은 프레임 속도인 60fps는 더 부드럽고 정교한 움직임을 제공합니다. 이는 고화질 비디오, 고속 액션 게임, 스포츠 방송 등에서 많이 사용됩니다.
```

### 사용자 정의 성능 위젯


### 렌더링 함수
순수하게 함수를 사용해 요소를 DOM에 렌더링하는 다양한 방법이 있다. 순수 함수로 요소를 렌더링한다는 것은 DOM요소가 애플리케이션의 상태에만 의존한다는 것을 의미한다.
<br>
```
//index.html
<html>

<head>
    <link rel="shortcut icon" href="../favicon.ico" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/todomvc-common@1.0.5/base.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/todomvc-app-css@2.1.2/index.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Faker/3.1.0/faker.js"></script>
    <title>
        Frameworkless Frontend Development: Rendering
    </title>
</head>

<body>
    <section class="todoapp">
        <header class="header">
            <h1>todos</h1>
            <input class="new-todo" placeholder="What needs to be done?" autofocus>
        </header>
        <section class="main">
            <input id="toggle-all" class="toggle-all" type="checkbox">
            <label for="toggle-all">Mark all as complete</label>
            <ul class="todo-list">
            </ul>
        </section>
        <footer class="footer">
            <span class="todo-count">1 Item Left</span>
            <ul class="filters">
                <li>
                    <a href="#/">All</a>
                </li>
                <li>
                    <a href="#/active">Active</a>
                </li>
                <li>
                    <a href="#/completed">Completed</a>
                </li>
            </ul>
            <button class="clear-completed">Clear completed</button>
        </footer>
    </section>
    <footer class="info">
        <p>Double-click to edit a todo</p>
        <p>Created by <a href="http://twitter.com/thestrazz86">Francesco Strazzullo</a></p>
        <p>Thanks to <a href="http://todomvc.com">TodoMVC</a></p>
    </footer>
    <script type="module" src="index.js"></script>
</body>

</html>
```

위와 같은 html을 동적으로 만들려면 to-do 리스트 데이터를 가져와 다음을 업데이트한다.
- 필터링된 todo 리스트를 가진 ul
- 완료되지 않은 todo 수를 가진 span
- selected 클래스를 오른쪽에 추가한 필터 유형을 가진 링크

```tsx
//view.js
const getTodoElement = (todo) => {
  const { text, completed } = todo;

  return `
  <li ${completed ? 'class="completed"' : ""}>
    <div class="view">
      <input 
        ${completed ? "checked" : ""}
        class="toggle" 
        type="checkbox">
      <label>${text}</label>
      <button class="destroy"></button>
    </div>
    <input class="edit" value="${text}">
  </li>`;
};

const getTodoCount = (todos) => {
  const notCompleted = todos.filter((todo) => !todo.completed);

  const { length } = notCompleted;
  if (length === 1) {
    return "1 Item left";
  }

  return `${length} Items left`;
};

export default (targetElement, state) => {
  const { currentFilter, todos } = state;

  const element = targetElement.cloneNode(true);

  const list = element.querySelector(".todo-list");
  const counter = element.querySelector(".todo-count");
  const filters = element.querySelector(".filters");

  list.innerHTML = todos.map(getTodoElement).join("");
  counter.textContent = getTodoCount(todos);

  Array.from(filters.querySelectorAll("li a")).forEach((a) => {
    if (a.textContent === currentFilter) {
      a.classList.add("selected");
    } else {
      a.classList.remove("selected");
    }
  });

  return element;
};

```

이 뷰 함수는 기본으로 사용되는 타깃 DOM을 받는다. 그런 다음 원래 노드를 복제하고 state 매개변수를 사용해 업데이트한다.

### 코드 리뷰
하지만 이 코드느 ㄴ두가지의 중요한 문제를 갖고 있다.
- **하나의 거대한 함수** 여러 DOM 요소를 조작하는 함수가 단 하나뿐이다. 이는 상황을 아주 쉽게 복잡하게 만들 수 있다.
- **동일한 작업을 수행하는 여러 방법** 문자열을 통해 리스트 항목을 생성한다. todo count 요소의 경우 단순히 기존 요소에 테스트를 추가하기만 하면 된다.

그래서 위 코드의 View를 새로 리팩토링 한다.

```tsx
//app.js
import todosView from "./todos.js";
import counterView from "./counter.js";
import filtersView from "./filters.js";

export default (targetElement, state) => {
  const element = targetElement.cloneNode(true);

  const list = element.querySelector(".todo-list");
  const counter = element.querySelector(".todo-count");
  const filters = element.querySelector(".filters");

  list.replaceWith(todosView(list, state));
  counter.replaceWith(counterView(counter, state));
  filters.replaceWith(filtersView(filters, state));

  return element;
};

```

```
//counter.js
const getTodoCount = (todos) => {
  const notCompleted = todos.filter((todo) => !todo.completed);

  const { length } = notCompleted;
  if (length === 1) {
    return "1 Item left";
  }

  return `${length} Items left`;
};

export default (targetElement, { todos }) => {
  const newCounter = targetElement.cloneNode(true);
  newCounter.textContent = getTodoCount(todos);
  return newCounter;
};

```

```
//filter.js
export default (targetElement, { currentFilter }) => {
  const newCounter = targetElement.cloneNode(true);
  Array.from(newCounter.querySelectorAll("li a")).forEach((a) => {
    if (a.textContent === currentFilter) {
      a.classList.add("selected");
    } else {
      a.classList.remove("selected");
    }
  });
  return newCounter;
};

```

```
//todos.js
const getTodoElement = (todo) => {
  const { text, completed } = todo;

  return `
      <li ${completed ? 'class="completed"' : ""}>
        <div class="view">
          <input 
            ${completed ? "checked" : ""}
            class="toggle" 
            type="checkbox">
          <label>${text}</label>
          <button class="destroy"></button>
        </div>
        <input class="edit" value="${text}">
      </li>`;
};

export default (targetElement, { todos }) => {
  const newTodoList = targetElement.cloneNode(true);
  const todosElements = todos.map(getTodoElement).join("");
  newTodoList.innerHTML = todosElements;
  return newTodoList;
};

```

이제 코드가 더 가독성이 좋아졌다.

#### replaceWith
```tsx
const div = document.createElement("div");
const p = document.createElement("p");
div.appendChild(p);
const span = document.createElement("span");

p.replaceWith(span);

console.log(div.outerHTML);
// "<div><span></span></div>"
```

### 구성 요소 함수
앱뷰의 코드를 확인해보면 올바른 함수를 수동으로 호출해야 한다.구성 요소 기반의 애플리케이션을 작성하려면 구성 요소간의 상호작용에 선언적 방식을 사용해야 한다.
[using data attributes](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes)<br>
이번 예제에서는  todos, counters, filters 세가지 구성 요소를 가진다.

```
<html>

<head>
    <link rel="shortcut icon" href="../favicon.ico" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/todomvc-common@1.0.5/base.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/todomvc-app-css@2.1.2/index.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Faker/3.1.0/faker.js"></script>
    <title>
        Frameworkless Frontend Development: Rendering
    </title>
</head>

<body>
    <section class="todoapp">
        <header class="header">
            <h1>todos</h1>
            <input 
                class="new-todo" 
                placeholder="What needs to be done?" 
                autofocus>
        </header>
        <section class="main">
            <input 
                id="toggle-all" 
                class="toggle-all" 
                type="checkbox">
            <label for="toggle-all">
                Mark all as complete
            </label>
            <ul class="todo-list" data-component="todos">
            </ul>
        </section>
        <footer class="footer">
            <span 
                class="todo-count" 
                data-component="counter">
                    1 Item Left
            </span>
            <ul class="filters" data-component="filters">
                <li>
                    <a href="#/">All</a>
                </li>
                <li>
                    <a href="#/active">Active</a>
                </li>
                <li>
                    <a href="#/completed">Completed</a>
                </li>
            </ul>
            <button class="clear-completed">
                Clear completed
            </button>
        </footer>
    </section>
    <footer class="info">
        <p>Double-click to edit a todo</p>
        <p>Created by <a href="http://twitter.com/thestrazz86">Francesco Strazzullo</a></p>
        <p>Thanks to <a href="http://todomvc.com">TodoMVC</a></p>
    </footer>
    <script type="module" src="index.js"></script>
</body>

</html>
```

```tsx
//registry.js
const registry = {};

const renderWrapper = (component) => {
  return (targetElement, state) => {
    const element = component(targetElement, state);

    const childComponents = element.querySelectorAll("[data-component]");

    Array.from(childComponents).forEach((target) => {
      const name = target.dataset.component;

      const child = registry[name];
      if (!child) {
        return;
      }

      target.replaceWith(child(target, state));
    });

    return element;
  };
};

const add = (name, component) => {
  registry[name] = renderWrapper(component);
};

const renderRoot = (root, state) => {
  const cloneComponent = (root) => {
    return root.cloneNode(true);
  };

  console.log(registry);

  return renderWrapper(cloneComponent)(root, state);
};

export default {
  add,
  renderRoot,
};

```

<img width="403" alt="image" src="https://github.com/FrontendStudySeoul/FrontEndDevelopNoFramework/assets/103626175/9ebb3762-303a-4b4f-b748-bbbc5369b28b">
위 사진처럼 데이터  레지스트리의 키는 data-component속성 값과 일치한다.<br>
이것이 구성 요소 기반 렌더링 엔진의 핵심 메커니즘이다. 이렇게하면 모든 구성 요소가 다른 구성 요소 안에서도 사용될 수 있다. 그리고 이런 재사용성은 구성 요소 기반 애플리케이션에서 필수적이다.

#### 구성 요소 기반 애플리케이션
```
CBP(컴포넌트 기반 프로그래밍)는 더 작고 독립적인 컴포넌트의 구성을 통해 복잡한 시스템을 구축함으로써 모듈성, 재사용 및 관심사 분리를 강조하는 고급 소프트웨어 개발 패러다임입니다. 일반적으로 "모듈"이라고 하는 개별 단위로 캡슐화되는 이러한 구성 요소는 독립적이고 느슨하게 결합되어 있으며 시스템 내에서 특정 작업을 수행하거나 특정 기능을 수행하도록 설계된 재사용 가능성이 높은 엔터티입니다. CBP는 견고성, 유지 관리 용이성 및 소프트웨어 개발 프로세스의 설계, 구현, 테스트 및 배포 단계를 간소화하여 애플리케이션 개발을 가속화하는 능력으로 인해 다양한 산업 및 부문에서 널리 채택되었습니다.
```

#### CDD(Component Driven develop)
```
"구성 요소 기반 애플리케이션"과 "Component-Driven Development (CDD)"는 서로 밀접한 관련이 있습니다.

"구성 요소 기반 애플리케이션"은 애플리케이션을 독립적이고 재사용 가능한 컴포넌트들로 구성하는 방식을 가리킵니다. 이러한 컴포넌트는 각각 특정 기능을 수행하며, 이들을 조합하여 전체 애플리케이션을 만듭니다. React, Vue, Angular 등의 프레임워크/라이브러리는 이러한 접근 방식을 사용합니다.

"Component-Driven Development (CDD)"는 이러한 컴포넌트 중심의 접근 방식을 개발 프로세스에 적용하는 방법론입니다. CDD에서는 개별 컴포넌트를 독립적으로 개발하고 테스트하며, 이러한 컴포넌트들을 함께 사용하여 전체 애플리케이션을 구축합니다. 이 방식은 컴포넌트의 재사용성을 증가시키고, 유지보수를 쉽게 만들며, 테스트를 간소화하는 등의 이점을 제공합니다.

따라서, "구성 요소 기반 애플리케이션"은 컴포넌트를 사용하여 애플리케이션을 구성하는 개념이며, "Component-Driven Development"는 이러한 컴포넌트 중심의 접근을 개발 프로세스에 적용하는 방법론이라고 할 수 있습니다. 이 둘은 서로 밀접한 관련이 있으며, 종종 함께 사용됩니다.
```

```tsx
//index.js
import getTodos from './getTodos.js'
import todosView from './view/todos.js'
import counterView from './view/counter.js'
import filtersView from './view/filters.js'

import registry from './registry.js'

registry.add('todos', todosView)
registry.add('counter', counterView)
registry.add('filters', filtersView)

const state = {
  todos: getTodos(),
  currentFilter: 'All'
}

window.requestAnimationFrame(() => {
  const main = document.querySelector('.todoapp')
  const newMain = registry.renderRoot(main, state)
  main.replaceWith(newMain)
})

```

### 동적 데이터 렌더링
이전까지는 정적 데이터를 사용했찌만 실제 애플리케이션에서는 사용잔 시스템의 이벤트에 의해 데이터가 변경된다.
<br>
그래서 임의로 5초마다 상태를 무작위로 변경해보자.
```tsx
import getTodos from './getTodos.js'
import todosView from './view/todos.js'
import counterView from './view/counter.js'
import filtersView from './view/filters.js'

import registry from './registry.js'

registry.add('todos', todosView)
registry.add('counter', counterView)
registry.add('filters', filtersView)

const state = {
  todos: getTodos(),
  currentFilter: 'All'
}

const render = ()=>{
  window.requestAnimationFrame(() => {
    const main = document.querySelector('.todoapp')
    const newMain = registry.renderRoot(main, state)
    main.replaceWith(newMain)
  })
}

window.setInterval(()=>{
  state.todos = getTodos()
  render()
},5000)

render()
```

새 데이터가 있을 때마다 가상 루트 요소를 만든 다음 실제 요소를 새로 생성된 요소로 바꾼다.

### 가상 DOM
리액트에 의해 유명해진 가상 DOM 개념은 선언적 렌더링 엔진의 성능을 개선시키는 방법이다. UI 표현은 메모리에 의해 유지되고 실제 DOM과 동기화된다. 실제 DOM은 가능한 적은 작업을 수행하며 이는 [https://ko.legacy.reactjs.org/docs/reconciliation.html](조정(reconcilation))이라고 불린다.
<br>
```tsx
<ul>
<li>first Item</li>
</ul>
```
위와 같은 간단한 코드에서 second Item을 가진 li를 추가하려고 할떄 이전 알고리즘에서는 ul 전체를 교체했다. 하지만 가상 DOM을 사용하면 마지막 li가 실제 DOM에 필요한 유일한 작업임을 동적으로 이해한다.
![image](https://github.com/FrontendStudySeoul/FrontEndDevelopNoFramework/assets/103626175/bd68e741-3855-4bdf-a9b2-c22292ec5b4e)

```
diff 알고리즘
React의 diff 알고리즘은 가상 DOM을 사용하여 실제 DOM에 대한 변경 사항을 효율적으로 계산하는 방법입니다. 이 알고리즘을 통해 React는 애플리케이션의 상태가 변경될 때마다 전체 UI를 새로 그리지 않고, 변경된 부분만 업데이트하여 성능을 최적화합니다.

React diff 알고리즘의 핵심 원칙은 다음과 같습니다:

두 개의 다른 타입의 요소는 서로 다른 트리를 만듭니다. React는 요소의 타입이 변경되면 그 요소와 그의 모든 자식을 새로운 트리로 간주하고 새로 그립니다.
개발자가 key prop을 제공할 수 있습니다. 만약 두 요소가 같은 key를 가지고 있다면, React는 그들이 같은 요소로 간주합니다. 따라서 배열의 항목을 반복적으로 렌더링할 때 key를 지정하는 것이 중요합니다. 이를 통해 React는 배열의 항목을 적절하게 재사용하거나 재정렬할 수 있습니다.
이러한 원칙들은 React가 트리의 변경 사항을 효율적으로 계산하고, 필요한 경우에만 실제 DOM을 업데이트하도록 도와줍니다. 이는 React의 가장 중요한 성능 최적화 기법 중 하나입니다.
```

### 간단한 가상 DOM 구현

```tsx
const isNodeChanged = (node1, node2) => {
  const n1Attributes = node1.attributes
  const n2Attributes = node2.attributes
  if (n1Attributes.length !== n2Attributes.length) {
    return true
  }

  const differentAttribute = Array
    .from(n1Attributes)
    .find(attribute => {
      const { name } = attribute
      const attribute1 = node1
        .getAttribute(name)
      const attribute2 = node2
        .getAttribute(name)

      return attribute1 !== attribute2
    })

  if (differentAttribute) {
    return true
  }

  if (node1.children.length === 0 &&
    node2.children.length === 0 &&
    node1.textContent !== node2.textContent) {
    return true
  }

  return false
}

const applyDiff = (
  parentNode,
  realNode,
  virtualNode) => {
  if (realNode && !virtualNode) {
    realNode.remove()
    return
  }

  if (!realNode && virtualNode) {
    parentNode.appendChild(virtualNode)
    return
  }

  if (isNodeChanged(virtualNode, realNode)) {
    realNode.replaceWith(virtualNode)
    return
  }

  const realChildren = Array.from(realNode.children)
  const virtualChildren = Array.from(virtualNode.children)

  const max = Math.max(
    realChildren.length,
    virtualChildren.length
  )
  for (let i = 0; i < max; i++) {
    applyDiff(
      realNode,
      realChildren[i],
      virtualChildren[i]
    )
  }
}

export default applyDiff

```

위 코드에서 먼저 새 노드가 정의되지 않은 경우 실제 노드를 삭제하고, 실제 노드가 정의되지 않았찌만 가상 노드가 존재할 경우 부모 노드에 추가한다. 그리고 두 노드가 모두 정의된 경우 두 노드간에 차이가 있는지 확인한다.
<br>
그리고 모든 하위 노드에 대해서 diff알고리즘을 적용한다. 

위의 diff 알고리즘 구현에서는 노드를 다른 노드와 비교해 노드가 변공됐는지 확인한다.
- 속성 수가 다르다.
- 하나 이상의 속성이 변경됐다.
- 노드에는 자식이 없으며, textContent가 다르다.

### 요약
2장에서는 프레임워크 없이 애플리케이션의 렌더링 엔진을 만드는 방법을 배웠다. 또한 간단한 구성 요소 레지스트리 작성 방법과 가상 DOM 알고리즘을 사용해(diff) 엔진 성능을 향상시키는 방법도 살펴봤다.
