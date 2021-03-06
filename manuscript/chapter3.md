# API 실제 사용 Getting Real with an API

이제 API로 데이터를 요청하고 받아봅시다.

API에 잘 모른다면, 저자가 쓴 [아무도 나에게 API를 알려주지 않았다](https://www.robinwieruch.de/what-is-an-api-javascript/) 글을 읽어보길 바랍니다. 

[해커 뉴스(Hacker News)](https://news.ycombinator.com/) 플랫폼은 기술 주제 관련 우수 기사를 큐레이션하는 플랫폼입니다. 이 책에서는 해커 뉴스 API를 사용해 플랫폼에서 인기 급상승 중인 글 리스트를 보여줄 것입니다. 해커 뉴스는 [데이터](https://github.com/HackerNews/API) 조회 API와 [검색](https://hn.algolia.com/api) API를 제공합니다. 사이트에서 API 명세서를 읽어본다면 데이터 구조를 이해할 수 있을 겁니다.

## 생명주기 메서드 Lifecycle Methods

API를 사용하기 전 컴포넌트 생명주기 메서드에 대해 알아봅시다. 생명주기 메서드는 ES6 클래스 컴포넌트에서 사용할 수 있지만 비상태 컴포넌트에서는 사용할 수 없습니다. 

이전 장에서 E6 클래스와 리액트에서 사용법을 배웠습니다. `render()`외에, 리액트 ES6 클래스 컴포넌트에서 오버라이드가 가능한 메서드가 있습니다. 바로 생명주기 메서드입니다.

우리는 이미 ES6 클래스 컴포넌트에서 사용하는 두 생명주기 메서드 `constructor()`와  `render()`를 알고 있습니다.

컴포넌트 인스턴스가 만들어져 DOM에 삽입 될 때 생성자가 호출됩니다. 컴포넌트가 인스턴스화됩니다. 이 프로세스를 컴포넌트 탑재됐다(mounting)고 말합니다.

`render()`는 마운트 프로세스 중에도 호출되지만 컴포넌트가 업데이트 될 때도 호출됩니다. 컴포넌트의 state와 props가 변경될 때마다 `render()`이 호출됩니다.

아마 이전 장에서 두 메서드를 사용해봤을 겁니다. 그 외 더 많은 생명주기 메서드가 있습니다
 
컴포넌트 마운트에는` componentWillMount()`와`componentDidMount()` 두 메서드가 있습니다. `constructor()`가 호출되면, `componentWillMount()`가 호출되고, 그 후 `render()`이 호출되며, 마지막에 `componentDidMount()`가 호출됩니다.

일반적인 마운팅 프로세스에서 생명주기 메서드는 아래 순서대로 호출됩니다.

* constructor()
* componentWillMount()
* render()
* componentDidMount()


state나 props가 변경 시, 업데이트 생명주기는 어떨까요? 아래와 같은 순서로 5개의 생명주기 메서드가 호출됩니다.

* componentWillReceiveProps()
* shouldComponentUpdate()
* componentWillUpdate()
* render()
* componentDidUpdate()

마지막으로 마운트 해제 메서드 componentWillUnmount()가 호출됩니다.

* componentWillUnmount()

처음부터 모든 생명주기 메서드를 사용하지 않아도 됩니다. 아직 많이 다뤄보지 않았기 때문에 익숙해지지 않았을 뿐입니다. 대규모 애플리케이션에도 `constructor()`과 `render()`만 사용할 경우가 많습니다. 앞서 알아본 생명주기 메서드를 언제 사용해야 하는지 정리해봅시다.

* **constructor(props)** - 컴포넌트 초기화 시 호출됩니다. 초기 컴포넌트 상태 및 클래스 메서드를 정의합니다.

* **componentWillMount()** - `render()` 전에 호출됩니다. 컴포넌트의 두 번째 렌더링을 트리거하지 않아 컴포넌트 상태를 설정하기 적합합니다. `constructor()`을 사용해 초기 상태를 설정하는 것이 좋습니다.

* **render()** - 컴포넌트의 출력을 반환하기 때문에 필수로 사용합 반환합 그것은 소품 및 상태로 입력을 가져오고 요소를 반환합니다.

* **componentDidMount()** - 컴포넌트가 마운트 될 때 한 번만 호출됩니다. 이 메서드에서 API에서 데이터를 가져 오기 위해 비동기 요청을 수행할 수 있습니다. 가져온 데이터는 내부 컴포넌트 상태에 저장되어 `render()`로 컴포넌트가 업데이트 됩니다.

* **componentWillReceiveProps(nextProps)** - 업데이트 생명주기 동안 호출됩니다. `nextProps`는 다음 props를, `this.props`로 이전 props를 조회해 서로 차이를 알 수 있습니다. nextProps를 컴포넌트의 state로 사용할 수 있습니다.
 
* **shouldComponentUpdate(nextProps, nextState)** - props 또는 state의 변경으로 컴포넌트가 업데이트 될 때 호출됩니다. 성능 최적화를 위해 고도화된 리액트 애플리케이션에서 사용됩니다. 생명주기 메서드에서 반환하는 부울 값(boolean)에 컴포넌트와 모든 자식이 업데이트 주기에 렌더링되거나 반대로 그렇지 않을 수 있습니다. 문제가 되는 특정 컴포넌트의 렌더링 생명주기 렌더링을 막을 수 있습니다.명주기 메서드를 방지 할 수 있습니다.

* **componentWillUpdate(nextProps, nextState)** - `render()` 메서드 전에 바로 호출됩니다. `nextProps`는 다음 props를, `nextState`로 다음 state를 조회할 수 있습니다. `render()` 메서드가 실행되기 전에 마지막으로 상태를 업데이트 할 수 있는 기회입니다. 이후에는 `setState()`를 더 이상 사용할 수 없습니다. nextProps를 state 값으로 사용하려면 `componentWillReceiveProps()`를 사용합니다.

* **componentDidUpdate(prevProps, prevState)** - `render()` 후에 즉시 호출됩니다. DOM조작을 하거나 추가 비동기 요청을 수행할 수 있습니다.

* **componentWillUnmount()** - 컴포넌트를 해체하기 전에 호출 됩니다. 테스크를 초기화하는 작업을 수행할 수 있습니다.


`constructor()`와 `render()` 메서드는 이미 사용해봤습니다. ES6 클래스 컴포넌트에서 사용되는 생명주기 입니다. `render()`은 컴포넌트 인스턴스를 반환하기 때문에 반드시 필요한 메서드입니다.

그 외 생명주기 메서드로 `componentDidCatch(error, info)`가 있습니다. 이 메서드는 [React 16](https://www.robinwieruch.de/what-is-new-in-react-16/)에서 도입되었으며 컴포넌트 에러를 캐치합니다. 예를 들어 목록을 표시하는 애플리케이션이 있다고 해봅시다. 외부 API 호출이 실패하여 state가 `null`로 표시되었습니다. 리스트가 비어있지 않고 `null` 상태이기 때문에 filter과 map을 사용할 수 없습니다. 이 경우 에러가 발생하여 컴포넌트가 깨지고 전체 애플리케이션이 작동되지 않습니다. 이 때, `componentDidCatch()`로 에러를 포착하고 내부 상태에 저장하여 사용자에게 에러 메세지를 표시해줄 수 있습니다.

### 읽어보기

* [[리액트 공식문서] 리액트 생명주기](https://facebook.github.io/react/docs/react-component.html)
* [[리액트 공식문서] 리액트 생명주기 메서드와 상태 관리](https://facebook.github.io/react/docs/state-and-lifecycle.html)
* [[reactjs.org] 컴포넌트 에러 핸들링](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

## 데이터 가져오기 Fetching Data

이제 해커 뉴스 API로 데이터를 가져올 준비가 끝났습니다. `componentDidMount()` 생명주기 메서드 안에 `fetch()` 자바스크립트 네이티브 API를 사용해 요청을 수행할 수 있습니다.

그 전에 URL과 API 요청 분리하여 기본 매개 변수로 설정합시다.

{title="src/App.js",lang=javascript}
~~~~~~~~
import React, { Component } from 'react';
import './App.css';

# leanpub-start-insert
const DEFAULT_QUERY = 'redux';

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
# leanpub-end-insert

...
~~~~~~~~

ES6에서는 [템플릿 문자열](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals)로 문자열을 연결합니다. 이를 사용해 URL을 API 엔드 포인트에 연결합시다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// ES6
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${DEFAULT_QUERY}`;

// ES5
var url = PATH_BASE + PATH_SEARCH + '?' + PARAM_SEARCH + DEFAULT_QUERY;

console.log(url);
// output: https://hn.algolia.com/api/v1/search?query=redux
~~~~~~~~

이렇게 하면 나중에 URL 구성을 좀더 편하게 관리할 수 있습니다.

API 요청을 시작합시다. 전체 데이터가 한 번에 가져올 것인데, 각 단계는 이후에 설명하겠습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
...

class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
# leanpub-start-insert
      result: null,
      searchTerm: DEFAULT_QUERY,
# leanpub-end-insert
    };

# leanpub-start-insert
    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
# leanpub-end-insert
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

# leanpub-start-insert
  setSearchTopStories(result) {
    this.setState({ result });
  }

  fetchSearchTopStories(searchTerm) {
    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}`)
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
      .catch(e => e);
  }

  componentDidMount() {
    const { searchTerm } = this.state;
    this.fetchSearchTopStories(searchTerm);
  }
# leanpub-end-insert

  ...
}
~~~~~~~~

이 코드 속에 정말 많은 일들이 일어났습니다. 더 작은 조각으로 나눌까 싶었지만 그렇다면 다시 각 조각의 관계를 파악하기 어렵다고 판단했습니다. 각 단계를 자세히 설명하겠습니다.

첫째, 해커 뉴스 API의 데이터가 사용됨으로 이전의 샘플 데이터는 더 이상 필요없기 때문에 제거합니다. 컴포넌트 초기 상태는 결과 `result`가 비어있고, 검색어 `searchTerm`는 디폴트 값(`DEFAULT_QUERY`)으로 설정되어 있습니다. searchTerm은 Search 컴포넌트의 입력필드와 첫 번째 요청 시 사용됩니다.
                                                                                    
둘째, 컴포넌트가 마운트된 후 데이터를 가져오기 위해 `componentDidMount()` 생명주기 메서드를 사용합니다. 첫 번째 요청 시 로컬 상태의 기본 검색어가 사용됩니다. 기본 매개변수(`DEFAULT_QUERY`)값이 "redux"이기 때문에 "redux" 기사를 가져올 것입니다.

세 번째, 자바스크립트 네이티브 fetch API가 사용됩니다. ES6 템플릿 문자열로  `searchTerm`과 함께 URL를 만듭니다. 이 URL는 fetch API로 인자로 사용합니다. 응답은 JSON 데이터 구조로 변환하고 컴포넌트 내부 `result` 상태 값에 저장됩니다. 요청 중에 오류가 발생하면 함수는 `then()` 아닌 catch 블록으로 실행됩니다. 다음 장에서 오류 처리에 대해 설명합니다.

마지막으로 생성자에 컴포넌트 메서드를 바인딩 합니다.

이제 기존 데이터대신 페치한 데이터를 사용할 수 있습니다. 그러나 조심히 사용해야 합니다. result는 데이터 목록으로 [메타 정보와 인기 리스트가 같이 담겨 객체가 복잡합니다.](https://hn.algolia.com/api). `render()` 메서드에서 `console.log(this.state);`를 사용하여 상태를 출력해 개발자 도구에서 확인할 수 있습니다.

다음으로 result를 사용해 렌더링을 처리합니다. 그러나 처음에 result 값이 없는 경우 null을 반환합니다. API 요청이 성공하면 결과값이 내부 상태값에 저장되고 업데이트된 App 컴포넌트를 다시 렌더링됩니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const { searchTerm, result } = this.state;

    if (!result) { return null; }

# leanpub-end-insert
    return (
      <div className="page">
        ...
        <Table
# leanpub-start-insert
          list={result.hits}
# leanpub-end-insert
          pattern={searchTerm}
          onDismiss={this.onDismiss}
        />
      </div>
    );
  }
}
~~~~~~~~

컴포넌트 생명주기동안 일어나는 일을 요약하면 다음과 같습니다. 컴포넌트는 생성자로 초기화된 후 렌더링됩니다. 내부 상태  result 값이 null이므로 이 컴포넌트는 아무것도 보이지 않습니다. 그 다음 `componentDidMount()` 메서드가 실행됩니다. 이 메서드에서 해커 뉴스 API를 통해 비동기로 데이터를 가져옵니다. 데이터가 도착하면 `setSearchTopStories()`에서 내부 컴포넌트 상태를 업데이트합니다. 이후 업데이트 생명주기가 시작됩니다. 컴포넌트는 `render()`메소드를 다시 실행하지만 이번에는 상태 result 값이 있기 때문에 나타납니다. 따라서 컴포넌트와 Table 컴포넌트가 렌더링됩니다.

대부분 브라우저가 지원하는 `fetch()`로 비동기 요청을 수행했습니다.*create-react-app*은 모든 브라우저를 지원하고 있습니다. `fetch()` 대신 노드패키지인 [superagent](https://github.com/visionmedia/superagent) 또는 [axios](https://github.com/mzabriskie/axios)를 사용할 수 있습니다.

다시 애플리케이션으로 돌아갑시다. 인기 목록이 보일 겁니다. 두 가지 버그가 생겼습니다. 첫째, "닫기" 버튼입니다. 이 버튼은 새로 받은 객체를 식별하지 못해 작동하지 않습니다. 둘째, 초기에 서버에서 검색한 기사 목록을 가져왔지만, 이후 다른 검색어로 요청하면 클라이언트에서 리스트를 필터링합니다. 가장 좋은 방법은 Search 컴포넌트에서 API로 result 객체를 가져오는 것입니다. 버그는 다음 장에서 해결하겠습니다.


### 읽어보기

* [[MDN] ES6 템플릿 문자열](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals)
* [[MDN] the native fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API)
* [[저자 블로그] 리액트에서 데이터 호출하기](https://www.robinwieruch.de/react-fetching-data/)

## ES6 전개 연산자 Spread Operators

`onDismiss()` 메서드는 요청받은 데이터 객체를 인식하지 못하기 때문에 "취소"버튼이 작동하지 않습니다. 지금은 전체 목록만 알고 있을 뿐입니다. 목록 내 각 객체를 식별할 수 있게 변경해보겠습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
# leanpub-start-insert
  const updatedHits = this.state.result.hits.filter(isNotId);
  this.setState({
    ...
  });
# leanpub-end-insert
}
~~~~~~~~

지금 `setState()`에서 무슨 일이 일어날까요? result는 객체 안에 hits 프로퍼티를 조회해야 합니다. hits는 여러 프로퍼티 중 하나입니다. 그러나 항목이 result 개체에서 제거되면 목록만 업데이트되고 다른 프로퍼티는 그대로 유지됩니다.

한 가지 방법은 result 객체안에 있는 hits 프로퍼티를 변경할 수 있겠지만, 이렇게 구현하지 않을 겁니다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// 이렇게 하지마세요. 
this.state.result.hits = updatedHits;
~~~~~~~~

리액트는 불변 데이터 구조가 원칙입니다. 객체를 변경하거나, 상태를 직접 변경해서는 안됩니다. 다른 방법은 새로운 객체를 생성해 어떤 객체도 변경하지 않아 불변 데이터 구조를 유지합니다. 항상 새 객체를 반환하고 객체를 변경하지 않도록 합니다.

이를 위해 ES6 `Object.assign()` 메서드를 사용합니다. 첫 번째 인자에 타겟 객체이고, 나머지 인수는 소스 객체입니다. 이 객체를 병함해 타겟 객체에 병합합니다. 타겟 객체는 빈 객체가 될 수 있습니다. 소스 객체는 변경되지 않기 때문에 불변성 원칙을 고수합니다. 아래 코드를 작성해봅시다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const updatedHits = { hits: updatedHits };
const updatedResult = Object.assign({}, this.state.result, updatedHits);
~~~~~~~~

동일한 프로퍼티 명을 공유할 때, 후자 객체는 전자의 병합 객체를 덮어 씁니다. 이제 `onDismiss()` 메서드에서 해보겠습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
  const updatedHits = this.state.result.hits.filter(isNotId);
  this.setState({
# leanpub-start-insert
    result: Object.assign({}, this.state.result, { hits: updatedHits })
# leanpub-end-insert
  });
}
~~~~~~~~

해결되었지만, 앞에서 배운 전개 연산자로 더 간단하게 작성할 수 있습니다. `...`은 배열 또는 객체의 모든 값을 복사합니다.

물론 ES6 **배열** 스프레드 연산자가 필요없다고 생각할 수 있겠지만, 아래와 같이 작성할 수 있습니다.


{title="Code Playground",lang="javascript"}
~~~~~~~~
const userList = ['Robin', 'Andrew', 'Dan'];
const additionalUser = 'Jordan';
const allUsers = [ ...userList, additionalUser ];

console.log(allUsers);
// output: ['Robin', 'Andrew', 'Dan', 'Jordan']
~~~~~~~~

변수 `allUsers`는 완전히 새로운 배열입니다. 변수 `userList`와 `additionalUser`는 그래로 유지됩니다. 이런 식으로 두 배열을 새 배열로 병합 할 수도 있습니다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const oldUsers = ['Robin', 'Andrew'];
const newUsers = ['Dan', 'Jordan'];
const allUsers = [ ...oldUsers, ...newUsers ];

console.log(allUsers);
// output: ['Robin', 'Andrew', 'Dan', 'Jordan']
~~~~~~~~

이제 객체 스프레드 연산자(object spread operator)를 살펴보겠습니다. ES6이 아닌, 리액트 커뮤니티에서 이미 사용하고있는 [다음 자바 스크립트 버전 제안](https://github.com/sebmarkbage/ecmascript-rest-spread)을 말합니다. *create-react-app*에서는 이 기능을 도입했습니다.

ES6 배열 전개 연산자와 같지만 객체가 있습니다. 각 키 값 쌍을 새 객체에 복사합니다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
const age = 28;
const user = { ...userNames, age };

console.log(user);
// output: { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
~~~~~~~~

배열 전개 예제와 같이 여러 객체를 펼칠 수 있습니다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const userNames = { firstname: 'Robin', lastname: 'Wieruch' };
const userAge = { age: 28 };
const user = { ...userNames, ...userAge };

console.log(user);
// output: { firstname: 'Robin', lastname: 'Wieruch', age: 28 }
~~~~~~~~

결국 `Object.assign()` 메서드를 대체할 수 있습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
onDismiss(id) {
  const isNotId = item => item.objectID !== id;
  const updatedHits = this.state.result.hits.filter(isNotId);
  this.setState({
# leanpub-start-insert
    result: { ...this.state.result, hits: updatedHits }
# leanpub-end-insert
  });
}
~~~~~~~~

`onDismiss()`메서드가 잘 작동하는지 확인합시다. `onDismiss()`메서드는 result 목록에서 각 객체를 식별하고 목록에서 취소된 항목을 업데이트합니다.

### 읽어보기

* [[MDN] ES6 Object.assign()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
* [[MDN]ES6 배열 전개 연산자](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator)

## 조건부 렌더링 Conditional Rendering

앞에서 조건부 렌더링에 대해 일부 설명했습니다. 조건부 렌더링은 하나 또는 다른 요소를 렌더링을 결정합니다. 요소를 렌더링하거나 아무것도 렌더링하지 않는 것을 말합니다. 우리가 잘 알고 있는 if-else 문으로 간단하게 작성할 수 있습니다.

컴포넌트 내부 상태 `result` 객체 초기값은 `null`입니다. App 컴포넌트에 API에서 `result`가 도착하지 않으면 요소를 반환하지 않습니다. 이 것이 바로 조건부 렌더링입니다. 특정 조건에 대해 이전에`render()` 생명주기 메서드에서  반환하기 때문에 이미 조건부 렌더링에 해당합니다. App 컴포넌트는 아무것도 렌더링하지 않거나 요소를 렌더링하게 됩니다.

한 단계를 더 나아가봅시다. 독립적인 조건부 렌더링에서 `result`에 의존하는 컴포넌트인 Table도 함께 포함시킵니다. `result`가 없더라도 모든 내용이 표시되어야 합니다. 이를 위해 JSX에서 삼항 조건 연산자(ternary operator)를 사용할 수 있습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const { searchTerm, result } = this.state;
# leanpub-end-insert
    return (
      <div className="page">
        <div className="interactions">
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
          >
            Search
          </Search>
        </div>
# leanpub-start-insert
        { result
          ? <Table
            list={result.hits}
            pattern={searchTerm}
            onDismiss={this.onDismiss}
          />
          : null
        }
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

마지막으로 논리 연산자 `&&`를 사용합시다. 자바스크립트에서 `true &&'Hello World'` 경우 'Hello World'로 평가되고, `false && 'Hello World'`경우 항상 거짓으로 평가됩니다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const result = true && 'Hello World';
console.log(result);
// output: Hello World

const result = false && 'Hello World';
console.log(result);
// output: false
~~~~~~~~

리액트에서도 논리 연산자를 사용할 수 있습니다. 조건이 참이면, `&&` 논리 연산자 뒷 부분 내용이 출력됩니다. 조건이 거짓이면 리액트는 표현식을 무시하고 건너 뜁니다. Table을 반환거나 아무것도 반환하지 않도록 조건부 렌더링에 만들어봅시다.

{title="src/App.js",lang=javascript}
~~~~~~~~
{ result &&
  <Table
    list={result.hits}
    pattern={searchTerm}
    onDismiss={this.onDismiss}
  />
}
~~~~~~~~

지금까지 조건부 렌더링 사용 방법에 대해 알아보았습니다. [저자의 블로그]( (https://www.robinwieruch.de/conditional-rendering-react/))에서 조건부 렌더링에 대한 더 많은 예제를 소개했습니다. 

마침내 애플리케이션에서 페치한 데이터를 모두 볼 수 있습니다. 데이터 페치가 보류 중일 때를 제외하고 모든 항목이 표시됩니다. 요청이 결과를 해석하여 로컬 상태로 저장하면 `render()` 메서드가 다시 실행되고, 조건부 렌더링의 조건에 따라 Table 컴포넌트가 표시됩니다.


### 읽어보기

* [[리액트 공식문서] 리액트 조건 렌더링](https://facebook.github.io/react/docs/conditional-rendering.html)
* [[저자 블로그] 조건문 렌더링의 다양한 방법](https://www.robinwieruch.de/conditional-rendering-react/)

## Search 컴포넌트의 클라이언트와 서버 처리  Client-/Server-side Search

클라이언트에서 검색어에 따라 목록 필터링을 구현해보겠습니다. 검색어 기본 매개변수로 해커 뉴스 API에 요청하여 서버에서 검색한 후, 이에 대한 응답을 `componentDidMount()`에서 처리할 것입니다.                                     
App 컴포넌트에서 `onSearchSubmit()` 메서드를 정의하겠습니다. 이 메서드는 Search 컴포넌트에서 검색을 실행할 때 해커 뉴스 API 요청 결과를 가져옵니다. `componentDidMount()` 생명주기 메소드에서 했던 것과 같습니다. 그러나 이번에는 검색어 기본 매개변수가 아닌 내부 상태 값을 사용합니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
      result: null,
      searchTerm: DEFAULT_QUERY,
    };

    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
# leanpub-start-insert
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
# leanpub-end-insert
    this.onDismiss = this.onDismiss.bind(this);
  }

  ...

# leanpub-start-insert
  onSearchSubmit() {
    const { searchTerm } = this.state;
    this.fetchSearchTopStories(searchTerm);
  }
# leanpub-end-insert

  ...
}
~~~~~~~~

Search 컴포넌트에 '검색하기' 버튼을 추가해야 합니다. 이 버튼은 검색 요청을 보내고, 해커 뉴스 API에서 데이터를 가져옵니다. 

먼저 Search 컴포넌트에 `onSearchSubmit()` 메서드를 전달합시다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, result } = this.state;
    return (
      <div className="page">
        <div className="interactions">
          <Search
            value={searchTerm}
            onChange={this.onSearchChange}
# leanpub-start-insert
            onSubmit={this.onSearchSubmit}
# leanpub-end-insert
          >
            Search
          </Search>
        </div>
        { result &&
          <Table
            list={result.hits}
            pattern={searchTerm}
            onDismiss={this.onDismiss}
          />
        }
      </div>
    );
  }
}
~~~~~~~~

두 번째로 Search 컴포넌트에 버튼을 추가하겠습니다. 버튼은 `type="submit"`을 가지고, 폼은 `onSubmit()` 메서드를 전달하기 위해 `onSubmit()` 속성을 사용합니다. 자식 프로퍼티는 버튼 내용으로 사용하겠습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
const Search = ({
  value,
  onChange,
  onSubmit,
  children
}) =>
  <form onSubmit={onSubmit}>
    <input
      type="text"
      value={value}
      onChange={onChange}
    />
    <button type="submit">
      {children}
    </button>
  </form>
# leanpub-end-insert
~~~~~~~~

Table 컴포넌트에서 클라이언트의 검색이 필요없기 때문에 필터 기능을 제거합니다. `isSearched()` 함수도 더 이상 사용하지 않기 때문에 제거합니다. 비로소 검색 버튼을 클릭하면 해커 뉴스 API를 통해 결과값을 가져옵니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, result } = this.state;
    return (
      <div className="page">
        ...
        { result &&
          <Table
# leanpub-start-insert
            list={result.hits}
            onDismiss={this.onDismiss}
# leanpub-end-insert
          />
        }
      </div>
    );
  }
}

...

# leanpub-start-insert
const Table = ({ list, onDismiss }) =>
# leanpub-end-insert
  <div className="table">
# leanpub-start-insert
    {list.map(item =>
# leanpub-end-insert
      ...
    )}
  </div>
~~~~~~~~

검색 버튼을 클릭하면 브라우저가 새로고침되는데 이는 HTML 폼 전송 콜백에 대한 네이티브 브라우저 동작입니다. 리액트에서는 네이티브 브라우저 동작을 방지하기 위해 `preventDefault()` 이벤트 메서드를 자주 사용합니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
# leanpub-start-insert
onSearchSubmit(event) {
# leanpub-end-insert
  const { searchTerm } = this.state;
  this.fetchSearchTopStories(searchTerm);
# leanpub-start-insert
  event.preventDefault();
# leanpub-end-insert
}
~~~~~~~~

해커 뉴스 기사를 검색할 수 있어야 합니다. API가 완벽하게 작동합니다. 더 이상 클라이언트에 검색을 처리하지 않도록 해야합니다.

### 읽어보기

* [[리액트 공식문서] synthetic events in React](https://facebook.github.io/react/docs/events.html)

### 실습하기
* [해커 뉴스 API](https://hn.algolia.com/api)를 이것저것 실험해봅니다.

## 페이지 매김 데이터 가져오기 Paginated Fetch

반환되는 데이터 구조를 자세히 살펴보았나요? [해커 뉴스 API](https://hn.algolia.com/api)는 첫 번째   `hits` 목록만 반환합니다. 처음 응답시, page 프로퍼티 값은 `0`으로 페이지 번호를 말합니다. 이 프로퍼티 값으로 목록을 가져옵니다. 검색어는 유지하고 다음 페이지번호를 API에 전달하면 됩니다.

페이지 매김 데이터를 처리할 수 있게 요청할 URL를 쪼개어 각 상수로 만들겠습니다. 이렇게 하면 URL를 구성 가능한 형태로 만들 수 있습니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
const DEFAULT_QUERY = 'redux';

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
# leanpub-start-insert
const PARAM_PAGE = 'page=';
# leanpub-end-insert
~~~~~~~~

이제 상수를 매개변수로 추가합시다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const url = `${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}`;

console.log(url);
// output: https://hn.algolia.com/api/v1/search?query=redux&page=
~~~~~~~~

`fetchSearchTopStories()` 메서드는 두 번째 인자가 있는데, 이 두 번째 인자가 없으면 기본으로 `0`페이지를 전달합니다. 따라서 `componentDidMount()`와 `onSearchSubmit()`는 첫 번째 요청 시, 첫 번째 페이지를 가져옵니다. 추가 요청이 있을 때마다 두 번째 인자를 통해 다음 페이지를 가져와야 합니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

# leanpub-start-insert
  fetchSearchTopStories(searchTerm, page = 0) {
    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}`)
# leanpub-end-insert
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
      .catch(e => e);
  }

  ...

}
~~~~~~~~

이제 `fetchSearchTopStories()`에서 API 응답을 받아 현재 페이지를 사용할 수 있습니다. 버튼에서 이 메서드를 사용해 `onClick` 버튼 핸들러로 더 많은 목록을 가져올 수 있습니다. 다음으로 버튼을 클릭하면 그 다음 페이지의 데이터를 가져오게 만들어봅시다. `onClick()` 핸들러에서 현재 검색어와 다음 페이지 번호(현재 페이지 + 1)만 정의하면 됩니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const { searchTerm, result } = this.state;
# leanpub-start-insert
    const page = (result && result.page) || 0;
# leanpub-end-insert
    return (
      <div className="page">
        <div className="interactions">
        ...
        { result &&
          <Table
            list={result.hits}
            onDismiss={this.onDismiss}
          />
        }
# leanpub-start-insert
        <div className="interactions">
          <Button onClick={() => this.fetchSearchTopStories(searchTerm, page + 1)}>
            More
          </Button>
        </div>
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

`render()` 메서드에서 `result`가 없을 때 페이지 번호 기본값은 `0`입니다. `render()` 메서드는  `componentDidMount()` 생명주기 메서드에서 비동기로 데이터를 가져오기 전에 실행됨을 잊지 맙시다.

아직 한 가지 더 할일이 남아있습니다. 다음 페이지의 데이터를 가져오지만, 이전 데이터에 덮어쓰게되는 문제가 있습니다. 새 데이터를 정의하기보다 기존 상태값에 새 결과 객체 목록을 추가하는 것이 바람직합니다.

{title="src/App.js",lang=javascript}
~~~~~~~~
setSearchTopStories(result) {
# leanpub-start-insert
  const { hits, page } = result;

  const oldHits = page !== 0
    ? this.state.result.hits
    : [];

  const updatedHits = [
    ...oldHits,
    ...hits
  ];

  this.setState({
    result: { hits: updatedHits, page }
  });
# leanpub-end-insert
}
~~~~~~~~

`setSearchTopStories()`메서드에서 일어나는 일을 살펴봅시다. 

첫째, 먼저 `result`에서 `hits`와 `page`를 얻습니다.

둘째, 이미 `hits`가 있는지 확인합니다. `page` 번호가 0일 때는 `componentDidMount()`또는 `onSearchSubmit()`에서 새로운 검색 요청이 전달됩니다. `hits` 목록은 비어있지만, "More" 버튼을 클릭하면 0이 아닌 다음 페이지를 요청합니다. 이전 `hits`는 이미 저장되어있으므로 사용가능 합니다.

셋째, 이전 `hits`에 새로운 `hits`이 덮어쓰기되면 안됩니다. 따라서 두 목록을 병합해야 합니다. 이를 위해 ES6 배열 전개 연산자를 사용합니다.

넷째, `hits`와  `hits`를 컴포넌트 내부 상태로 업데이트합니다.

마지막으로 "More" 버튼 클릭할 때 일부 목록만 가져오게 해봅시다. 각 요청에 따라 목록을 가져올 수 있게 API URL를 구성가능한 형태로 만들겠습니다. 각 경로를 상수로 만들어 추가하면 됩니다.   

{title="src/App.js",lang=javascript}
~~~~~~~~
const DEFAULT_QUERY = 'redux';
# leanpub-start-insert
const DEFAULT_HPP = '100';
# leanpub-end-insert

const PATH_BASE = 'https://hn.algolia.com/api/v1';
const PATH_SEARCH = '/search';
const PARAM_SEARCH = 'query=';
const PARAM_PAGE = 'page=';
# leanpub-start-insert
const PARAM_HPP = 'hitsPerPage=';
# leanpub-end-insert
~~~~~~~~

이제 상수를 사용해 API URL를 조합합시다.

{title="src/App.js",lang=javascript}
~~~~~~~~
fetchSearchTopStories(searchTerm, page = 0) {
# leanpub-start-insert
  fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
# leanpub-end-insert
    .then(response => response.json())
    .then(result => this.setSearchTopStories(result))
    .catch(e => e);
}
~~~~~~~~

이전보다 더 많은 목록을 가져옵니다. 이번 장에서는 해커 뉴스 API를 통해 실제 데이터를 사용해봤습니다. 새 프로그래밍 언어나 라이브러리를 학습할 때, 외부 API를 사용하면 더 재미있게 배울 수 있을 겁니다. 저도 이렇게 배웠으니까요.

### 실습하기

* [해커 뉴스 API](https://hn.algolia.com/api)의 매개변수를 바꿔 API를 요청해봅니다.

## 클라이언트 캐시 Client Cache

Each search submit makes a request to the Hacker News API. You might search for "redux", followed by "react" and eventually "redux" again. In total it makes 3 requests. But you searched for "redux" twice and both times it took a whole asynchronous roundtrip to fetch the data. In a client-sided cache you would store each result. When a request to the API is made, it checks if a result is already there. If it is there, the cache is used. Otherwise an API request is made to fetch the data.

In order to have a client cache for each result, you have to store multiple `results` rather than one `result` in your internal component state. The results object will be a map with the search term as key and the result as value. Each result from the API will be saved by search term (key).

현재 로컬 상태는 아래 코드와 비슷할 겁니다.

{title="Code Playground",lang="javascript"}
~~~~~~~~
result: {
  hits: [ ... ],
  page: 2,
}
~~~~~~~~

Imagine you have made two API requests. One for the search term "redux" and another one for "react". The results object should look like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
results: {
  redux: {
    hits: [ ... ],
    page: 2,
  },
  react: {
    hits: [ ... ],
    page: 1,
  },
  ...
}
~~~~~~~~

Let's implement a client-side cache with React `setState()`. First, rename the `result` object to `results` in the initial component state. Second, define a temporary `searchKey` which is used to store each `result`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {
    super(props);

    this.state = {
# leanpub-start-insert
      results: null,
      searchKey: '',
# leanpub-end-insert
      searchTerm: DEFAULT_QUERY,
    };

    ...

  }

  ...

}
~~~~~~~~

The `searchKey` has to be set before each request is made. It reflects the `searchTerm`. You might wonder: Why don't we use the `searchTerm` in the first place? That's a crucial part to understand before continuing with the implementation. The `searchTerm` is a fluctuant variable, because it gets changed every time you type into the Search input field. However, in the end you will need a non fluctuant variable. It determines the recent submitted search term to the API and can be used to retrieve the correct result from the map of results. It is a pointer to your current result in the cache and thus can be used to display the current result in your `render()` method.

{title="src/App.js",lang=javascript}
~~~~~~~~
componentDidMount() {
  const { searchTerm } = this.state;
# leanpub-start-insert
  this.setState({ searchKey: searchTerm });
# leanpub-end-insert
  this.fetchSearchTopStories(searchTerm);
}

onSearchSubmit(event) {
  const { searchTerm } = this.state;
# leanpub-start-insert
  this.setState({ searchKey: searchTerm });
# leanpub-end-insert
  this.fetchSearchTopStories(searchTerm);
  event.preventDefault();
}
~~~~~~~~

Now you have to adjust the functionality where the result is stored to the internal component state. It should store each result by `searchKey`.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  setSearchTopStories(result) {
    const { hits, page } = result;
# leanpub-start-insert
    const { searchKey, results } = this.state;

    const oldHits = results && results[searchKey]
      ? results[searchKey].hits
      : [];
# leanpub-end-insert

    const updatedHits = [
      ...oldHits,
      ...hits
    ];

    this.setState({
# leanpub-start-insert
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      }
# leanpub-end-insert
    });
  }

  ...

}
~~~~~~~~

The `searchKey` will be used as the key to save the updated hits and page in a `results` map.

First, you have to retrieve the `searchKey` from the component state. Remember that the `searchKey` gets set on `componentDidMount()` and `onSearchSubmit()`.

Second, the old hits have to get merged with the new hits as before. But this time the old hits get retrieved from the `results` map with the `searchKey` as key.

Third, a new result can be set in the `results` map in the state. Let's examine the `results` object in `setState()`.

{title="src/App.js",lang=javascript}
~~~~~~~~
results: {
  ...results,
  [searchKey]: { hits: updatedHits, page }
}
~~~~~~~~

The bottom part makes sure to store the updated result by `searchKey` in the results map. The value is an object with a hits and page property. The `searchKey` is the search term. You already learned the `[searchKey]: ...` syntax. It is an ES6 computed property name. It helps you to allocate values dynamically in an object.

The upper part needs to spread all other results by `searchKey` in the state by using the object spread operator. Otherwise you would lose all results that you have stored before.

Now you store all results by search term. That's the first step to enable your cache. In the next step, you can retrieve the result depending on the non fluctuant `searchKey` from your map of results. That's why you had to introduce the `searchKey` in the first place as non fluctuant variable. Otherwise the retrieval would be broken when you would use the fluctuant `searchTerm` to retrieve the current result, because this value might change when you would use the Search component.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
# leanpub-start-insert
    const {
      searchTerm,
      results,
      searchKey
    } = this.state;

    const page = (
      results &&
      results[searchKey] &&
      results[searchKey].page
    ) || 0;

    const list = (
      results &&
      results[searchKey] &&
      results[searchKey].hits
    ) || [];

# leanpub-end-insert
    return (
      <div className="page">
        <div className="interactions">
          ...
        </div>
# leanpub-start-insert
        <Table
          list={list}
          onDismiss={this.onDismiss}
        />
# leanpub-end-insert
        <div className="interactions">
# leanpub-start-insert
          <Button onClick={() => this.fetchSearchTopStories(searchKey, page + 1)}>
# leanpub-end-insert
            More
          </Button>
        </div>
      </div>
    );
  }
}
~~~~~~~~

Since you default to an empty list when there is no result by `searchKey`, you can spare the conditional rendering for the Table component now. Additionally you will need to pass the `searchKey` rather than the `searchTerm` to the "More" button. Otherwise your paginated fetch depends on the `searchTerm` value which is fluctuant. Moreover make sure to keep the fluctuant `searchTerm` property for the input field in the "Search" component.

The search functionality should work again. It stores all results from the Hacker News API.

Additionally the `onDismiss()` method needs to get improved. It still deals with the `result` object. Now it has to deal with multiple `results`.

{title="src/App.js",lang=javascript}
~~~~~~~~
  onDismiss(id) {
# leanpub-start-insert
    const { searchKey, results } = this.state;
    const { hits, page } = results[searchKey];
# leanpub-end-insert

    const isNotId = item => item.objectID !== id;
# leanpub-start-insert
    const updatedHits = hits.filter(isNotId);

    this.setState({
      results: {
        ...results,
        [searchKey]: { hits: updatedHits, page }
      }
    });
# leanpub-end-insert
  }
~~~~~~~~

The "Dismiss" button should work again.

However, nothing stops the application from sending an API request on each search submit. Even though there might be already a result, there is no check that prevents the request. Thus the cache functionality is not complete yet. It caches the results, but it doesn't make use of them. The last step would be to prevent the API request when a result is available in the cache.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  constructor(props) {

    ...

# leanpub-start-insert
    this.needsToSearchTopStories = this.needsToSearchTopStories.bind(this);
# leanpub-end-insert
    this.setSearchTopStories = this.setSearchTopStories.bind(this);
    this.fetchSearchTopStories = this.fetchSearchTopStories.bind(this);
    this.onSearchChange = this.onSearchChange.bind(this);
    this.onSearchSubmit = this.onSearchSubmit.bind(this);
    this.onDismiss = this.onDismiss.bind(this);
  }

# leanpub-start-insert
  needsToSearchTopStories(searchTerm) {
    return !this.state.results[searchTerm];
  }
# leanpub-end-insert

  ...

  onSearchSubmit(event) {
    const { searchTerm } = this.state;
    this.setState({ searchKey: searchTerm });
# leanpub-start-insert

    if (this.needsToSearchTopStories(searchTerm)) {
      this.fetchSearchTopStories(searchTerm);
    }

# leanpub-end-insert
    event.preventDefault();
  }

  ...

}
~~~~~~~~

Now your client makes a request to the API only once although you search for a search term twice. Even paginated data with several pages gets cached that way, because you always save the last page for each result in the `results` map. Isn't that a powerful approach to introduce caching to your application? The Hacker News API provides you with everything you need to even cache paginated data effectively.

## 에러 핸들링 Error Handling

Everything is in place for your interactions with the Hacker News API. You even have introduced an elegant way to cache your results from the API and make use of its paginated list functionality to fetch an endless list of sublists of stories from the API. But there is one piece missing. Unfortunately it is often missed when developing applications nowadays: error handling. It is too easy to implement the happy path without worrying about the errors that can happen along the way.

In this chapter, you will introduce an efficient solution to add error handling for your application in case of an erroneous API request. You have already learned about the necessary building blocks in React to introduce error handling: local state and conditional rendering. Basically, the error is only another state in React. When an error occurs, you will store it in the local state and display it with a conditional rendering in your component. That's it. Let's implement it in the App component, because it's the component that is used to fetch the data from the Hacker News API in the first place. First, you have to introduce the error in the local state. It is initialized as null, but will be set to the error object in case of an error.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      results: null,
      searchKey: '',
      searchTerm: DEFAULT_QUERY,
# leanpub-start-insert
      error: null,
# leanpub-end-insert
    };

    ...
  }

...

}
~~~~~~~~

Second, you can use the catch block in your native fetch to store the error object in the local state by using `setState()`. Every time the API request isn't successful, the catch block would be executed.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  fetchSearchTopStories(searchTerm, page = 0) {
    fetch(`${PATH_BASE}${PATH_SEARCH}?${PARAM_SEARCH}${searchTerm}&${PARAM_PAGE}${page}&${PARAM_HPP}${DEFAULT_HPP}`)
      .then(response => response.json())
      .then(result => this.setSearchTopStories(result))
# leanpub-start-insert
      .catch(e => this.setState({ error: e }));
# leanpub-end-insert
  }

  ...

}
~~~~~~~~

Third, you can retrieve the error object from your local state in the `render()` method and display a message in case of an error by using React's conditional rendering.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
# leanpub-start-insert
      error
# leanpub-end-insert
    } = this.state;

    ...

# leanpub-start-insert
    if (error) {
      return <p>Something went wrong.</p>;
    }
# leanpub-end-insert

    return (
      <div className="page">
        ...
      </div>
    );
  }
}
~~~~~~~~

That's it. If you want to test that your error handling is working, you can change the API URL to something else that is non existent.

{title="src/App.js",lang=javascript}
~~~~~~~~
const PATH_BASE = 'https://hn.foo.bar.com/api/v1';
~~~~~~~~

Afterward, you should get the error message instead of your application. It is up to you where you want to place the conditional rendering for the error message. In this case, the whole app isn't displayed anymore. That wouldn't be the best user experience. So what about displaying either the Table component or the error message? The remaining application would still be visible in case of an error.

{title="src/App.js",lang=javascript}
~~~~~~~~
class App extends Component {

  ...

  render() {
    const {
      searchTerm,
      results,
      searchKey,
      error
    } = this.state;

    const page = (
      results &&
      results[searchKey] &&
      results[searchKey].page
    ) || 0;

    const list = (
      results &&
      results[searchKey] &&
      results[searchKey].hits
    ) || [];

    return (
      <div className="page">
        <div className="interactions">
          ...
        </div>
# leanpub-start-insert
        { error
          ? <div className="interactions">
            <p>Something went wrong.</p>
          </div>
          : <Table
            list={list}
            onDismiss={this.onDismiss}
          />
        }
# leanpub-end-insert
        ...
      </div>
    );
  }
}
~~~~~~~~

In the end, don't forget to revert the URL for the API to the existent one.

{title="src/App.js",lang=javascript}
~~~~~~~~
const PATH_BASE = 'https://hn.algolia.com/api/v1';
~~~~~~~~

Your application should still work, but this time with error handling in case the API request fails.

### 읽어보기

* [[reactjs.org] 리액트 컴포넌트 오류 처리](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)

{pagebreak}

여러분은 리액트에서 API를 사용할 수 있습니다! 이번 장에서 배운 내용을 정리해봅시다.

* React
  * ES6 class component lifecycle methods for different use cases
  * componentDidMount() for API interactions
  * conditional renderings
  * synthetic events on forms
  * error handling
* ES6
  * template strings to compose strings
  * spread operator for immutable data structures
  * computed property names
* 일반
  * 해커 뉴스 API 인터렉션
  * native fetch browser API
  * client- and server-side search
  * pagination of data
  * client-side caching

실습 코드는 [깃허브 리퍼지토리](https://github.com/rwieruch/hackernews-client/tree/4.2)에서 확인할 수 있습니다.