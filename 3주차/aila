# HTTP 요청

- 5장에서 알아볼 것 : 프레임워크 없는 방식으로 HTTP 클라이언트 구축 방법을 알아본다.

## AJAX

- 예전에는 서버에서 데이터를 가져올 때마다 전체 페이지를 모두 다시 로드했다.
- 필요한 데이터만 서버에서 로드하는 AJAX가 나왔다.
- AJAX 애플리케이션의 핵심은 XMLHttpRequest

  - 해당 객체를 사용하면 HTTP 요청으로 서버에서 데이터를 가져올 수 있다.

- AJAX가 등장했을 때는 웹 애플리케이션은 서버 데이터를 XML 형태로 수신했다.
- 지금은 JSON 형식을 사용한다.

![](https://velog.velcdn.com/images/nsunny0908/post/86b2a79b-6800-485c-abfb-32808bbcf501/image.png)

### XML vs JSON

- XML은 트리 패턴
- JSON은 키, 값 페어

```json
{
  "guests": [
    { "firstName": "John", "lastName": "Doe" },

    { "firstName": "María", "lastName": "García" },

    { "firstName": "Nikki", "lastName": "Wolf" }
  ]
}
```

```xml
<guests>
  <guest>
    <firstName>John</firstName> <lastName>Doe</lastName>
  </guest>

  <guest>
    <firstName>María</firstName> <lastName>García</lastName>
  </guest>

  <guest>
    <firstName>Nikki</firstName> <lastName>Wolf</lastName>
  </guest>

</guests>

```

![](https://velog.velcdn.com/images/nsunny0908/post/18272148-2a1c-4893-a1ca-3cbd5375871b/image.png)

- 인터넷 유물 박물관 : https://neal.fun/internet-artifacts/

## todo 리스트 REST 서버

```js
const express = require("express");
const bodyParser = require("body-parser");
const uuidv4 = require("uuid/v4");
const findIndex = require("lodash.findindex");

// 포트 설정
const PORT = 8080;

const app = express();
let todos = [];

//  "public" 에 있는 파일들을 static으로 제공하기 위한 미들웨어 설정
app.use(express.static("public"));

// request body를 json 형식으로 파싱하기 위한 미들웨어를 설정
app.use(bodyParser.json());

// CRUD api 작성
app.get("/api/todos", (req, res) => {
  res.send(todos);
});

app.post("/api/todos", (req, res) => {
  const newTodo = {
    completed: false,
    ...req.body,
    id: uuidv4(),
  };

  todos.push(newTodo);

  res.status(201);
  res.send(newTodo);
});

app.patch("/api/todos/:id", (req, res) => {
  const updateIndex = findIndex(todos, (t) => t.id === req.params.id);
  const oldTodo = todos[updateIndex];

  const newTodo = {
    ...oldTodo,
    ...req.body,
  };

  todos[updateIndex] = newTodo;

  res.send(newTodo);
});

app.delete("/api/todos/:id", (req, res) => {
  todos = todos.filter((t) => t.id !== req.params.id);

  res.status(204);
  res.send();
});

app.listen(PORT);
```

## REST

- 웹서비스를 디자인하고 개발하는 방법
- REST API의 추상화는 `리소스`에 있다.
- 도메인을 리소스로 분할해야하며 리소스는 특정 URI로 접근해 읽거나 조작할 수 있어야 한다.
- PUT : 새로운 사용자의 모든 데이터를 전달
- PATCH : 이전 상태와의 차이만 포함

## 코드 예제

- XMLHttpRequest, Fetch, axios를 사용해 각기 다른 세 가지 HTTP 클라이언트 버전을 작성해본다.

### index.html

![](https://velog.velcdn.com/images/nsunny0908/post/6e866e35-31f6-4e45-a09a-6e9a16d9b501/image.png)

```html
<html>
  <head>
    <link rel="shortcut icon" href="../favicon.ico" />
    <title>Frameworkless Frontend Development: HTTP Requests</title>
  </head>

  <body>
    <button data-list>Read Todos list</button>
    <button data-add>Add Todo</button>
    <button data-update>Update todo</button>
    <button data-delete>Delete Todo</button>
    <div></div>
    <script type="module" src="index.js"></script>
  </body>
</html>
```

### index.js

- 메인 컨트롤러

```js
import todos from "./todos.js";

const printResult = (action, result) => {
  const time = new Date().toTimeString();
  const node = document.createElement("p");
  node.textContent = `${action.toUpperCase()}: ${JSON.stringify(
    result
  )} (${time})`;

  document.querySelector("div").appendChild(node);
};

const onListClick = async () => {
  const result = await todos.list();
  printResult("list todos", result);
};

const onAddClick = async () => {
  const result = await todos.create("A simple todo Element");
  printResult("add todo", result);
};

const onUpdateClick = async () => {
  const list = await todos.list();

  const { id } = list[0];
  const newTodo = {
    id,
    completed: true,
  };

  const result = await todos.update(newTodo);
  printResult("update todo", result);
};

const onDeleteClick = async () => {
  const list = await todos.list();
  const { id } = list[0];

  const result = await todos.delete(id);
  printResult("delete todo", result);
};

// DOM에 접근해서 이벤트 리스너 달아줌.
document
  .querySelector("button[data-list]")
  .addEventListener("click", onListClick);

document
  .querySelector("button[data-add]")
  .addEventListener("click", onAddClick);

document
  .querySelector("button[data-update]")
  .addEventListener("click", onUpdateClick);

document
  .querySelector("button[data-delete]")
  .addEventListener("click", onDeleteClick);
```

- HTTP 요청을 todos 모델 객체에 래핑함으로써 캡슐화의 이점을 가져간다.
  - `onListClick`, `onAddClick`, `onUpdateClick`, `onDeleteClick` 함수에서 'todos' 모델 객체를 사용하여 서버와의 통신을 통해 CRUD 를 구현
  - 서버와의 통신을 추상화하고 클라이언트단에서 함수 호출을 통해 기능이 동작하도록 한다.
    - 클라이언트단 코드와 서버단의 코드를 분리

### todo.js

- todo list 관련 CRUD 구현
- 서버와 통신

```js
import http from "./http.js";

// request 헤더를 정의
const HEADERS = {
  "Content-Type": "application/json",
};

const BASE_URL = "/api/todos";

const list = () => http.get(BASE_URL);

const create = (text) => {
  const todo = {
    text,
    completed: false,
  };

  return http.post(BASE_URL, todo, HEADERS);
};

const update = (newTodo) => {
  const url = `${BASE_URL}/${newTodo.id}`;
  return http.patch(url, newTodo, HEADERS);
};

const deleteTodo = (id) => {
  const url = `${BASE_URL}/${id}`;
  return http.delete(url);
};

export default {
  list,
  create,
  update,
  delete: deleteTodo,
};
```

### 방법1: XMLHttpPRequest를 이용해 작성한 https.js

- xhr 객체
  ![](https://velog.velcdn.com/images/nsunny0908/post/1c16def3-6ea0-46d2-90db-b41a82e7c570/image.png)

```js
// const HEADERS = {
//   "Content-Type": "application/json",
// };

const setHeaders = (xhr, headers) => {
  Object.entries(headers).forEach((entry) => {
    const [name, value] = entry;

    xhr.setRequestHeader(name, value);
  });
};

const parseResponse = (xhr) => {
  const { status, responseText } = xhr;

  let data;

  // 204(No Content)
  if (status !== 204) {
    data = JSON.parse(responseText);
  }

  return {
    status,
    data,
  };
};

const request = (params) => {
  // Promise를 반환
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

    const { method = "GET", url, headers = {}, body } = params;

    // url로 요청 초기화
    xhr.open(method, url);
    // header 설정
    setHeaders(xhr, headers);

    // requestbody를 설정하고 HTTP 요청을 보냄
    xhr.send(JSON.stringify(body));

    // 실패, 타임아웃의 경우 reject를 호출하여 오류를 반환
    xhr.onerror = () => {
      reject(new Error("HTTP Error"));
    };

    xhr.ontimeout = () => {
      reject(new Error("Timeout Error"));
    };

    // 성공하면 resolve를 호출하여 응답을 반환
    xhr.onload = () => resolve(parseResponse(xhr));
  });
};

const get = async (url, headers) => {
  const response = await request({
    url,
    headers,
    method: "GET",
  });

  return response.data;
};

const post = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: "POST",
    body,
  });
  return response.data;
};

const put = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: "PUT",
    body,
  });
  return response.data;
};

const patch = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: "PATCH",
    body,
  });
  return response.data;
};

const deleteRequest = async (url, headers) => {
  const response = await request({
    url,
    headers,
    method: "DELETE",
  });
  return response.data;
};

export default {
  get,
  post,
  put,
  patch,
  delete: deleteRequest,
};
```

- 핵심은 request 메서드
- XMLHttpRequest를 사용한 HTTP 요청의 흐름

1. `const xhr = new XMLHttpRequest()`

   - XMLHttpRequest 객체 생성

2. `xhr.open(method, url)`

   - 특정 URL로 요청 초기화

3. `setHeaders(xhr, headers)`

   - request 구성

4. `xhr.send(JSON.stringify(body))`

   - 요청 전송

5. 요청이 끝날 때까지 대기

   - onload, onerror, ontimeout

![](https://velog.velcdn.com/images/nsunny0908/post/87e57142-3cca-4a40-a1e8-5478cd369947/image.png)

### 방법2: Fetch를 이용해 작성한 https.js

- window.fetch 사용

```js
// response 파싱
// fetch가 promise를 리턴하고 비동기 request를 보내기 때문에 비동기 작업을 다루기 위해 await와 async 키워드 사용
const parseResponse = async (response) => {
  const { status } = response;
  let data;
  if (status !== 204) {
    // 비동기 함수
    data = await response.json();
  }

  return {
    status,
    data,
  };
};

const request = async (params) => {
  const { method = "GET", url, headers = {}, body } = params;

  const config = {
    method,
    headers: new window.Headers(headers),
  };

  if (body) {
    config.body = JSON.stringify(body);
  }

  // fetch API를 사용해 서버에 요청 보내기
  const response = await window.fetch(url, config);

  return parseResponse(response);
};
```

- 콜백 기반의 XMLHttpRequest 에서 최신의 promise 기반
- window.fetch에서 반환된 promise는 response객체를 resolve 한다.
  - 이 객체를 사용해 서버가 보낸 response body를 추출할 수 있다.
- 수신된 데이터의 형식에 따라 text(), blob(), json() 같은 다른 메서드 사용

### 방법3: axios를 이용해 작성한 https.js

```js
const request = async (params) => {
  const { method = "GET", url, headers = {}, body } = params;

  const config = {
    url,
    method,
    headers,
    data: body,
  };

  return axios(config);
};
```

- 세가지 방법 중에 가장 읽기 쉽다.
- request 메서드는 axios 서명을 공공 계약과 일치하도록 매개변수를 재배열한다.
  - axios는 요청을 보낼 때 config 객체를 사용하며, 이 객체에 요청과 관련된 정보를 포함시킨다.
  - `params`에서 추출한 정보를 `config` 객체의 필드에 대응시키는 방식으로 axios 요청을 설정한다.

## 아키텍처 검토

> 구현이 아닌 인터페이스로 프로그래밍하라

- 세 버전의 클라이언트는 동일한 공용 API를 가진다.
- 모든 클라이언트는 HTTP 클라이언트 인터페이스를 구현한다.

## 적절한 HTTP API를 선택하는 방법

- 호환성
- 휴대성
- 발전성
- 보안
- 학습곡선

### XHR

- 호환성

### fetch

- 발전성
- 학습곡선

### axios

- 호환성 (IE 11 이전의 경우는 해당하지 않음)
- 휴대성 (Node.js나 RN같은 다른 js 환경에서 코드를 실행해야하는 경우)
- 보안
- 학습곡선
