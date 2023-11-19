# 7장: 상태관리

## MVC 패턴

<img width='50%' src='https://yhancsx.github.io/assets/images/frameworkless_frontend/mvc_pattern.png'/>

1. Controller는 Model에서 초기 상태를 가져온다.
2. Controller는 View를 호출해 초기 상태를 렌더링한다.
3. 사용자가 어떤 동작을 수행한다.
4. Controller는 올바른 메서드(model.addItem)와 사용자의 동작을 매핑한다.
5. Model이 상태를 업데이트 한다.
6. Controller는 Model에서 새로운 상태를 얻는다.
7. Controller는 View를 호출해 새로운 상태를 렌더링한다.

### Controller

```js
const model = modelFactory();

const events = {
  addItem: (text) => {
    model.addItem(text);
    render(model.getState());
  },
  // 생략
};

const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector("#root");

    const newMain = registry.renderRoot(main, state, events);

    applyDiff(document.body, main, newMain);
  });
};

render(model.getState());
```

### Model

```js
const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)

  const getState = () => {
    return Object.freeze(cloneDeep(state))
  }

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos.push({
      text,
      completed: false
    })
  }

  // 생략

  return {
    addItem,
    ...
  }
}
```

### 특징

- 관심사의 분리: 도메인 비즈니스 로직이 Modal 객체에 포함되어 있다.
- 뷰와 컨트롤러의 차이점이 명확하지 않아 프레임워크마다 다른 버전의 MVC가 구현될 수 있다. 그러므로 확장될 수록 일관성을 지키려는 노력이 필요하다.

### 단점

- 수동으로 render 메서드를 호출해야 하기 때문에 오류가 쉽게 발생할 수 있다.
- 동작이 상태를 변경하지 않을 때도 render 메서드가 호출된다.
- 이를 관찰자 패턴(Observable Model)으로 해결 가능하다.

<br />

## Observable Model

### Controller

```js
const model = modelFactory();

const { addChangeListener, ...events } = model;

const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector("#root");

    const newMain = registry.renderRoot(main, state, events);

    applyDiff(document.body, main, newMain);
  });
};

addChangeListener(render);
```

### Model

```js
const cloneDeep = (x) => {
  return JSON.parse(JSON.stringify(x));
};

const freeze = (x) => Object.freeze(cloneDeep(x));

const INITIAL_STATE = {
  todos: [],
  currentFilter: "All",
};

export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState);
  let listeners = [];

  const addChangeListener = (listener) => {
    listeners.push(listener);

    listener(freeze(state));

    return () => {
      listeners = listeners.filter((l) => l !== listener);
    };
  };

  const invokeListeners = () => {
    const data = freeze(state);
    listeners.forEach((l) => l(data));
  };

  const addItem = (text) => {
    if (!text) {
      return;
    }

    state.todos.push({
      text,
      completed: false,
    });

    invokeListeners();
  };

  return {
    addItem,
    ...
  };
};
```

### Observable Model 단위 테스트

```js
describe("observable model", () => {
  beforeEach(() => {
    model = modelFactory();
  });

  test("listeners should be invoked when changing data", () => {
    let counter = 0;
    model.addChangeListener((data) => {
      counter++;
    });
    model.addItem("dummy");
    expect(counter).toBe(2);
  });

  //...
});
```

### 특징

- model 객체로부터 상태를 얻으려면 리스터 콜백을 추가해야 한다.
- 관찰자 패턴은 공개 인터페이스를 수정하지 않고 컨트롤러에 새로운 기능을 추가하는데 유용하다.
- 컨트롤러가 모델과 밀접하게 결합된다면 이 패턴을 고려하는 것이 좋다.

<br />

## 반응형 모델

> 반응형 패러다임의 구현은, 애플리케이션이 모델 변경, HTTP 요청, 사용자 동작, 탐색 등과 같은 이벤트를 방출할 수 있는 옵저버블로 동작하도록 구현하는 것을 의미한다.

- 도메인 로직에만 집중하고 아키텍처 부분은 별도의 옵저버블 팩토리를 기반으로 하는 새 모델을 구축
- Model 객체의 프록시를 생성하면 원본 모델의 모든 메서드는 원본 메서드를 래핑하고 리스너를 호출하는 동일한 이름의 새 메서드를 생성한다.

### Model

```js
export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState);

  const addItem = (text) => {
    if (!text) {
      return;
    }

    state.todos.push({
      text,
      completed: false,
    });
  };

  const model = {
    addItem,
    ...
  };

  return observableFactory(model, () => state);
};
```

### Observable Factory

```js
export default (model, stateGetter) => {
  let listeners = [];

  const addChangeListener = (cb) => {
    listeners.push(cb);
    cb(freeze(stateGetter()));
    return () => {
      listeners = listeners.filter((element) => element !== cb);
    };
  };

  const invokeListeners = () => {
    const data = freeze(stateGetter());
    listeners.forEach((l) => l(data));
  };

  const wrapAction = (originalAction) => {
    return (...args) => {
      const value = originalAction(...args);
      invokeListeners();
      return value;
    };
  };

  const baseProxy = {
    addChangeListener,
  };

  return Object.keys(model)
    .filter((key) => {
      return typeof model[key] === "function";
    })
    .reduce((proxy, key) => {
      const action = model[key];
      return {
        ...proxy,
        [key]: wrapAction(action),
      };
    }, baseProxy);
};
```

## 이벤트 버스 패턴

> 이벤트 주도 아키텍처를 구현하는 방법

<img src='https://yhancsx.github.io/assets/images/frameworkless_frontend/event_bus_pattern.png' />

- 이벤트는 이벤트 이름과 이벤트 처리를 위한 정보를 담는 페이로드로 정의된다.
- 이벤트 버스 단일 객체가 모든 이벤트를 처리하고, 처리되면 결과가 연결된 모든 노드로 전송된다.

1. view는 초기 상태를 렌더링한다.
2. 사용자가 폼을 작성하고 엔터키를 누른다.
3. DOM 이벤트가 View에 의해 캡처된다.
4. View는 ITEM_ADDED 이벤트를 생성하고 버스로 보낸다.
5. Bus는 새로운 상태를 생성하는 이벤트를 처리한다.
6. 새로운 상태가 Controller로 전송된다.
7. Controller가 View를 호출해 새로운 상태를 렌더링한다.
8. 시스템이 사용자 입력을 받을 준비가 됐다.

### Event Bus

```js
export default (model) => {
  let listeners = [];
  let state = model();

  const subscribe = (listener) => {
    listeners.push(listener);

    return () => {
      listeners = listeners.filter((l) => l !== listener);
    };
  };

  const invokeSubscribers = () => {
    const data = freeze(state);
    listeners.forEach((l) => l(data));
  };

  const dispatch = (event) => {
    const newState = model(state, event);

    if (!newState) {
      throw new Error("modifiers should always return a value");
    }

    if (newState === state) {
      return;
    }

    state = newState;

    invokeSubscribers();
  };
  return {
    subscribe,
    dispatch,
    getState: () => freeze(state),
  };
};
```

### 이벤트 버스 vs Redux

- 이밴트 버스 -> 스토어
- 이벤트 -> 액션
- 모델 -> 리듀서

## 정리

|             | 단순성 | 일관성 | 확장성 |
| :---------- | :----: | :----: | :----: |
| MVC         |   O    |   X    |   X    |
| 반응형      |   X    |   O    |   -    |
| 이벤트 버스 |   -    |   X    |   O    |
