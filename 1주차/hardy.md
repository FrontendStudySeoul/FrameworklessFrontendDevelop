## 프레임워크란?

캠브리지 사전에서 프레임워크의 정의는 다음과 같다.

> 무언가를 만들 수 있는 지지 구조
> 

이 정의는 소프트웨어 프레임워크의 일반적인 개념과 일치한다. 

실제 애플리케이션에서 스택은  lodash, moment 등 다른 요소들을 포함해 개발한다. 이를 라이브러리라고 부른다. 그럼 라이브러리와 프레임워크의 차이는 무엇일까?

> 프레임워크는 코드를 호출한다. 코드는 라이브러리를 호출한다.
> 

프레임워크는 내부적으로 하나 이상의 라이브러리를 사용할 수 있지만, 개발자가 모듈식 프레임워크를 선택하면 프레임워크를 단일 단위나 여러 모듈로 보는 개발자에게는 이러한 사실이 숨겨진다.

![image](https://github.com/FrontendStudySeoul/FrontEndDevelopNoFramework/assets/59603529/710eae00-ed81-4d72-86c5-879b71a69979)


### 프레임워크와 라이브러리 비교

예제를 통해 프레임워크와 라이브러리의 차이점을 알아보자

```tsx
import { Injectable }from "@angular/core";
import { HttpClient }from "@angular/common/http";

const URL = "http://example.api.com/";

@Injectable({
  providedIn: "root",
})
exportclass peopleService {
  constructor(private http: HttpClient) {}
  list() {
returnthis.http.get(URL);
  }
}

```

```tsx
import { Component, OnInit }from "@angular/core";
import { PeopleService }from "../people.service";

@Component({
  selector: "people-list",
  templateUrl: "./people-list.component.html",
})
exportclass PeopleListComponentimplements OnInit {
  constructor(private peopleService: PeopleService) {}

  ngOnInit() {
this.loadList();
  }

  loadList():void {
this.peopleService
      .getHeroes()
      .subscribe((people) => (this.people = people));
  }
}

```

```tsx
import moment from "moment";

const DATE_FORMAT = "DD/MM/YYY";

export const formatDate = (date) => {
  return moment(date).format(DATE_FORMAT);
};
```

앵귤러에서 PeopleListComponent가 PeopleService와 상호작용하게 하려면 @Injectable 어노테이션을 사용하고 생성자에 인스턴스를 넣어야 한다. 하지만 moment는 코드를 어떻게 구성해야 하는지에 대해 특별한 형식을 요구하지 않는다. 그저 가져와 사용하기만 하면 된다.

moment는 개발자가 어떻게 코드를 통합하는지 강요하지 않는다. 이에 반해 앵귤러는 몇 가지 제약 조건을 지켜며 개발해야 한다. 

## 자바스크립트 프레임워크 연혁

### 제이쿼리

제이쿼리는 모든 자바스크립트 프레임워크의 모체가 됐다. 제이쿼리의 가장 중요한 기능은 선택자 구문이다. 제이쿼리는 브라우저 간 공통어를 만들었으며, 이는 커뮤니티가 공통의 토양에서 성장하도록 도와줫다. 오늘날 프론트엔드 개발자들이 제이쿼리를 우습게 여기는 경향이 있지만 제이쿼리는 현대 웹 개발의 초석 역할을 했다.

### 앵귤러JS

앵귤러는 단일 페이지 애플리케이션을 주류로 만드는 데 큰 역할을 했다. 주목할 만한 기능은 양방향 데이터 바인딩이다. 양방향 데이터 바인딩 덕분에 개발자는 웹 애플리케이션을 빠르게 작성할 수 있게 됐다. 하지만 양방향 데이터 바인딩이 대규모 애플리케이션에는 적합하지 않기 때문에 시간이 지나자 많은 개발자가 앵귤러는 떠났다.

### 리액트

현재 가장 있기 있는 프론트엔드 라이브러리(프레임워크)다. 기술적으로 보면 리액트는 프레임워크가 아니라 렌더링 라이브러리다. 리액트는 선언적 패러다임으로 동작한다. 일반적으로 DOM을 직접 수정하는 대신 setState 메서드로 상태를 변경하면 리액트가 나머지 작업을 수행한다.

### 앵귤러

앵귤러는 원래 앵귤러JS의 새로운 버전으로 시작됐기 때문에 이전에는 앵귤러2로 불렸다. 프로젝트의 팀은 시맨틱 버전 관리를 매우 진지하게 여겼기 때문에 앵귤러2는 완전히 다른 프레임워크가 됐고 두 버전 사이의 완전히 다른 접근 방식으로 인해 프로젝트는 중단됐다.

## 기술 부채

모든 프레임워크는 기술 부채를 갖고 잇다. 미래에 코드 변경이 어렵다는 측면에서 보면 모든 프레임워크에는 비용이 발생한다. 프레임워크는 아키텍처 자체에 이미 비용을 포함하고 있다. 시간이 지남에 따라 소프트웨어의 변경이 필요하며, 아키텍처 역시 변경돼야 한다. 대부분의 경우 프레임워크는 이런 변경이 필요한 상황에서 장애물이 된다.
