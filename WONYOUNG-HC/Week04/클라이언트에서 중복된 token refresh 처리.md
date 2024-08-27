프로젝트를 진행하면서 인증 방식으로 JWT를 사용했다. JWT에서 access token이 만료된 경우 refresh token을 사용해 새로운 access token을 발급받는 전략을 사용하고 있다.

https://youthing.tistory.com/215
(위의 글의 작성해 주신 분이 만들어준 token을 사용한다 ㅎ)

axios interceptor를 사용해서 access token이 만료된 에러 코드가 발생한 새로운 access token을 발급하는 요청을 전송한다. 하지만 문제가 발생한 상황은 한 번에 여러 요청을 보내는 경우 새로운 access token 발급 요청이 중복되어 전송이 되었다.

이번에 작성하는 글은 JWT, axios에 대해 정리하고, 중복된 요청에 대해 해결한 방법에 대한 내용이다.

## JWT (Json Web Token)

### 인증과 인가

JWT은 웹에서 인증 방식이다. 여기서 인증과 인가의 개념이 등장하는데, 간단히 정리하면 **인증은 사용자의 신원을 확인**하는 과정이고, **인가는 인증된 사용자에게 권한을 허락**하는 행위이다.

티스토리에 본인이 작성한 글을 수정해야 한다면, 글을 수정한 사람이 본인이 맞는지 확인하는 과정이 필요하다. 여기서 **로그인을 통해 해당 글을 본인이 작성한것이지 확인하는 것이 인증**이고, **티스토리에서 글의 수정을 허용하는것이 인가**이다.

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDA1Az%2FbtsI1SY51tl%2FnaZAZgWMHKHL4mFCSbY4l0%2Fimg.png">

로그인을 하면 서버에서는 access token을 발행하여 클라이언트에게 전달하고, 클라이언트는 access token을 사용해 서버로부터 인증을 받는다.

### reresh token

하지만 JWT 방식에서 access token이 해커에게 탈취당하면, 해커가 탈취한 access token을 통해 사용자 행세를 할 수 있다. 그래서 access token에는 **만료 시간**을 설정한다. 지금 필자가 하고있는 프로젝트에서는 access token을 발급하고 30분 이후에는 token을 사용할 수 없다. 그러면 access token이 만료된 30분 이후에는 로그아웃이 되고, 사용자는 30분마다 로그인을 해야 하는 번거로움이 있다.

이런 문제를 해결하기 위해 **refresh token**이 등장한다.

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FIHdCU%2FbtsI2qOB2US%2FcyNPUnVRnm664YCmm5Um6K%2Fimg.png">

이 방식은 사용자가 로그인하면 서버에서 클라이언트에서 **access token**과 **refresh token** 두 개를 전달한다. 클라이언트는 서버로부터 받은 access token을 통해서 인가 요청을 보낸다. 그리고 기존 access token이 만료되면 서버에서는 기존의 access token이 만료되었다는 응답을 준다. 그러면 클라이언트는 처음 서버로부터 받은 **refresh token을 통해서 새로운 access token 발급을 요청**한다.

이렇게 refresh token을 통해 access token의 짧은 만료 시간에서 발생할 수 있는 문제점을 보완해준다.

## axios

### fetch API와 axios 라이브러리

클라이언트에서 서버에게 HTTP 요청을 보내는 가장 기본적인 방법은 fetch API를 사용하는 것이다. 하지만 fetch API를 사용하기에는 불편한 점이 몇몇 존재한다. 우선 body가 JSON 형식이기에 항상 string으로 변환이 필요하다. 하지만 axios 라이브러리를 사용하면 body를 객체로 접근할 수 있어 편리하다. 이뿐만 아니라 사용 방법 자체도 axios가 fetch보다 간단하다.

또한 fetch에서 요청 성공이란 서버로부터 응답이 도착하면 성공으로 간주한다. 즉 서버가 status code를 4xx, 5xx 번대로 보내도 응답이 왔다고 판단하여 요청 성공으로 간주한다. 그래서 예외 처리를 하기 위해서는 status code에 대해 직접 문기분을 작성해야 가능하다. 하지만 axios에서는 2xx 번대 응답만 성공으로 간주하고 나머지는 모두 에러처리 하여 예외 처리가 fetch보다 용이해진다.

그리고 이번 글에서 가장 중요한 **interceptors**에 대한 기능 유무이다.

### axios interceptors

axios 공식 문서에서 interceptors에 대한 설명은 **"then 또는 catch\*\***로 처리되기 전에 요청과 응답을 가로챌 수 있습니다."\*\*라고 나와 있다. 아래는 공식 문서에 나와있는 예시이다.

```js
// 요청 인터셉터 추가하기
axios.interceptors.request.use(
  function (config) {
    // 요청이 전달되기 전에 작업 수행
    return config;
  },
  function (error) {
    // 요청 오류가 있는 작업 수행
    return Promise.reject(error);
  }
);

// 응답 인터셉터 추가하기
axios.interceptors.response.use(
  function (response) {
    // 2xx 범위에 있는 상태 코드는 이 함수를 트리거 합니다.
    // 응답 데이터가 있는 작업 수행
    return response;
  },
  function (error) {
    // 2xx 외의 범위에 있는 상태 코드는 이 함수를 트리거 합니다.
    // 응답 오류가 있는 작업 수행
    return Promise.reject(error);
  }
);
```

axios에서 요청과 응답에 대한 interceptors를 제공한다. interceptors.request.use 함수는 콜백 함수 두 개를 인자로 받고, 첫 번째 콜백 함수는 **정상적인 요청 또는 응답**에 대한 처리, 두 번째 콜백 함수는 **에러가 발생한 경우에 대한 처리**이다.

request에 대한 interceptors는 사용자가 요청을 보낸 이후 **서버로 전송되기 직전에 실행**되고, response에 대한 interceptros는 서버로부터 응답이 도착하고, **axios 함수를 반환하기 직전에 실행**된다.

프로젝트에서 request interceptor는 헤더에 access token을 포함시킬 때 사용한다.

```js
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

인가가 필요한 요청을 보낼 때마다 일일이 헤더에 access token을 저장하지 않고, request interceptros를 사용해 모든 요청에 대해 access token을 헤더에 첨부하여 보낼 수 있다.

그러면 **response interceptors**는 언제 사용할 수 있을까? 바로 **access token이 만료되어 refresh token을 통해 새로운 access token을 요청**할 때 사용할 수 있다.

```js
axios.interceptors.response.use(
  (res) => res,
  async (error) => {
    const { config, response } = error;

	// access token이 만료되어 에러가 발생한 경우
    if (response.data.code === 'T-001') {
        try {
          // 새로운 access token을 발급 받는 요청을 전송
          const response = await api.post('/v1/api/auth/reissue');

          // 새롭게 발급 받은 access token을 사용
          config.headers.Authorization = `Bearer ${response.data.accessToken}`;
          localStorage.setItem('token', response.data.accessToken);

          // 실패한 요청을 다시 전송
          return axios(config);
        } catch (err) {
          // 새로운 access token을 발급받다 실패한 경우 access token을 지우고 메인 페이지로 이동
          localStorage.clear();
          window.location.replace('/');
        }
      }
    }

    // access token 만료로 인한 실패가 아닌 경우 에러로 처리
    return Promise.reject(error);
  },
);
```

response에서 에러가 발생한 경우 interceptor를 실행한다. config는 HTTP 요청에 대한 정보들이 담긴 객체이고, response는 실패한 요청에 대한 응답이다.

프로젝트에서 우리 서버는 access token이 만료된 경우 'T-001'을 response의 data에 담아 전송해 준다. 이를 통해 interceptor에서 어떤 에러가 발생했는지 탐지가 가능하고, **access token이 만료된 에러에 대한 분기를 처리해준다.**

클라이언트에서 access code는 local stroage에 저장하지만, refresh token은 cookie에 저장되어 있기 때문에 refresh token이 필요한 요청은 별도로 처리하지 않고 전송한다.

성공적으로 새로운 access token을 발행받으면 local storage에 새로운 aceess token을 저장하고, 실패한 요청을 새로운 access token으로 다시 실행한다.

새로운 access token을 발급받다가 실패한 경우 local sotrage를 clear 함으로써 access token을 삭제하고, 메인 페이지로 이동시킨다.

만약 에러가 'T-001'이 아니어서 access token만료로 인한 에러가 아닌 경우 에러를 그대로 반환한다.

이렇게 response interceptors를 사용해서 access token이 만료된 경우 refresh token을 사용하여 새로운 access token을 발급받고, 기존의 요청을 다시 실행시켜 **axios를 사용하는 개발자는 access token만료를 신경 쓰지 않고 개발을 할 수 있다.**

## 중복된 access token 발급 요청

페이지 접속 시 단 하나의 요청만 전송하는 경우도 있지만, 여러 개의 요청을 보내야 하는 경우도 더러 있다. 여러 요청을 동시에 보내는 상황에서 access token이 만료된 경우 위의 interceptors를 사용하면 어떤 일이 발생할까?

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBk6RE%2FbtsI1f1ABqy%2F9exKbYXP2s98npKQm3VX9k%2Fimg.png">

**interceptor가 이렇게 여러 번 실행된다.**

위에 정의한 interceptor는 단순히 access token만 새로 발급하기만 하지 않고, 기존 요청을 새로운 access token으로 다시 실행시킨다. 서버에서 refresh token을 사용해 새로운 access token을 발급받은 경우 기존의 access token을 만료 처리를 한다면, 그림에서 **access token 1은 발급을 받자마자 만료**가 된다.

A 요청과 B 요청을 동시에 전송하는 상황에서 access token이 만료되면 아래와 같은 흐름이 발생할 수도 있다.  
A 요청 실패로 인한 access token 발급 요청 -> B 요청 실패로 인한 access token 발급 요청 -> A 요청에 대해 new access token 1을 발급하고 기존 access token을 만료 처리 -> B 요청에 대해 new access token 2를 발급하고 new access token 1을 만료 처리 ->  실패한 A 요청이 new access token1을 통해 인가 요청

이렇게 새로운 access token을 발급받아 요청을 보내려 해도, 이미 만료가 되어 버려 새로운 access token을 또 발급받아야 하고, 이는 무한 재귀로까지 빠질 수도 있다.

즉 interceptor가 여러 번 실행되더라도, **새로운 access token을 받는 요청은 단 한 번만 실행되어야 한다.**

어떻게 이 문제를 해결을 할 수 있을까 고민하고, 검색을 해본 결과 아래 medium 블로그에 개시된 글을 통해 방법을 찾을 수 있었다.

https://medium.com/@sina.alizadeh120/repeating-failed-requests-after-token-refresh-in-axios-interceptors-for-react-js-apps-50feb54ddcbc

해당 블로그에서는 queue를 사용해서 이 문제를 해결했다.

```ts
interface RetryQueueItem {
  resolve: (value?: any) => void;
  reject: (error?: any) => void;
  config: AxiosRequestConfig;
}

const refreshAndRetryQueue: RetryQueueItem[] = [];
```

refreshAndRetryQueue는 access token이 만료된 요청들을 저장하는 리스트이다. 리스트 원소의 타입은 resolve, resject 함수와 config를 가지고 있다. refreshAndRetryQueue를 이용해서 access token이 만료된 응답을 받을 때마다 **새로운 access token을 발급하는 요청을 보내지 않고, 요청을 대기**시켜 놓을 수 있게 된다.

```ts
const refreshAndRetryQueue: RetryQueueItem[] = [];
let isRefreshing = false;

axios.interceptors.response.use(
  (res) => res,
  async (error) => {
    const { config, response } = error;

    if (response.data.code === 'T-001') {
      // 아직 아무 요청도 새로운 access token을 발급받지 않았다면
      if (!isRefreshing) {
        isRefreshing = true;

        try {
          const response = await api.post('/v1/api/auth/reissue');
          config.headers.Authorization = `Bearer ${response.data.accessToken}`;
          localStorage.setItem('token', response.data.accessToken);
        }
      }
    }

    return Promise.reject(error);
  },
);
```

코드를 부분적으로 살펴보겠다. isRereshing 변수는 여러 요청 중 어떤 요청이 access token이 만료된 응답을 받고 **새로운 access token을 발급받고 있는 중임을 표시하는 변수**이다. isRereshing이 true라면 다른 요청이 이미 새로운 access token을 발급 받고 있는 중이라는 의미이고, false이면 아직 아무 요청도 새로운 access token을 발급받지 않았다는 의미이다.

isRefresing은 초기 값으로 false로 설정되어 있고, access token이 만료된 첫 응답을 받은 요청은 위의 코드대로 실행이 될 것이다. 그러면 해당 요청은 새로운 access token을 발급받으니 **isRefresing 변수를 true로 설정하고 새로운 access token을 발급받는다.**

```ts
axios.interceptors.response.use(
  (res) => res,
  async (error) => {
    const { config, response } = error;

    if (response.data.code === 'T-001') {
      if (!isRefreshing) {
        isRefreshing = true;

        try {
          const response = await api.post('/v1/api/auth/reissue');
          config.headers.Authorization = `Bearer ${response.data.accessToken}`;
          localStorage.setItem('token', response.data.accessToken);
        }
      }

      // 어떤 요청이 이미 새로운 access token발급을 하고 있는 경우
      // queue에 요처을 대기
      return new Promise<void>((resolve, reject) => {
        refreshAndRetryQueue.push({ config, resolve, reject });
      });
    }

    return Promise.reject(error);
  },
);
```

다른 요청이 이미 새로운 access token을 발급받고 있는 중이면, **rereshAndRetryQueue에 요청을 대기**시킨다. 이를 통해 새로운 동시에 여러 요청이 새로운 access token을 발급받는 요청을 보내지 않아 위에서 언급한 무한 재귀 발생 문제는 방지를 할 수가 있다.

```ts
axios.interceptors.response.use(
  (res) => res,
  async (error) => {
    const { config, response } = error;

    if (response.data.code === "T-001") {
      if (!isRefreshing) {
        isRefreshing = true;

        try {
          const response = await api.post("/v1/api/auth/reissue");
          config.headers.Authorization = `Bearer ${response.data.accessToken}`;
          localStorage.setItem("token", response.data.accessToken);

          // 새로운 access token을 발급 받으면 queue에서 대기하던 요청들을 실행
          refreshAndRetryQueue.forEach(({ config, resolve, reject }) => {
            axios
              .request(config)
              .then((response) => resolve(response))
              .catch((err) => reject(err));
          });
          refreshAndRetryQueue.length = 0;

          // 새로운 access token을 발급 받은 요청을 다시 실행
          return axios(config);
        } catch (err) {
          localStorage.clear();
          window.location.replace("/");
        } finally {
          isRefreshing = false;
        }
      }

      return new Promise<void>((resolve, reject) => {
        refreshAndRetryQueue.push({ config, resolve, reject });
      });
    }

    return Promise.reject(error);
  }
);
```

처음에 만료된 access token 응답을 받은 요청이 새로운 access token을 발급받으면 **isRefreshAndRetryQueue에서 대기 중인 요청들을 처리힌다.** 그리고 새로운 access token을 발급받은 요청은 queue에 들어있지 않았으니 이에 대한 요청도 실행시킨다. 그리고 finally에서 isRefreshing을 false로 설정해서 다음에 refresh 로직이 가능하도록 준비를 해둔다.

전체 코드는 다음과 같다. 실제 코드는 axios에 interceptors를 설정하는 대신에, api이름으로 axios 인스턴스를 생성하여 사용한다.

```ts
import axios, { AxiosRequestConfig } from "axios";

const api = axios.create({
  baseURL: process.env.REACT_APP_BASE_URL,
  withCredentials: true,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

interface RetryQueueItem {
  resolve: (value?: any) => void;
  reject: (error?: any) => void;
  config: AxiosRequestConfig;
}

const refreshAndRetryQueue: RetryQueueItem[] = [];
let isRefreshing = false;

api.interceptors.response.use(
  (res) => res,
  async (error) => {
    const { config, response } = error;

    if (response.data.code === "T-001") {
      if (!isRefreshing) {
        isRefreshing = true;

        try {
          const response = await api.post("/v1/api/auth/reissue");
          config.headers.Authorization = `Bearer ${response.data.accessToken}`;
          localStorage.setItem("token", response.data.accessToken);

          refreshAndRetryQueue.forEach(({ config, resolve, reject }) => {
            api
              .request(config)
              .then((response) => resolve(response))
              .catch((err) => reject(err));
          });
          refreshAndRetryQueue.length = 0;

          return api(config);
        } catch (err) {
          localStorage.clear();
          window.location.replace("/");
        } finally {
          isRefreshing = false;
        }
      }

      return new Promise<void>((resolve, reject) => {
        refreshAndRetryQueue.push({ config, resolve, reject });
      });
    }

    return Promise.reject(error);
  }
);

export default api;
```

JWT 인증 방식을 채택하면서 refresh token을 사용해 새로운 access token을 발급받는 로직은 널리 알려진 방법인 거 같다. 처음에는 단순히 interceptors에서 refresh 로직을 설정했지만, 한 페이지에서 여러 요청이 동시에 발생하는 경우 해결 방법은 쉽게 찾지는 못했었다. 발견한 문제점은 해결했지만, 아직 이 방법이 맞는 방법일까?라는 의문이 아직 해결하지 못했다. (만약 더 좋은 방법이 있다면 알려 주세요!!)

하지만 해당 방식대로 직접 구현하면서 axios interceptors의 사용 방법을 알게 되었다. 그리고 글이 너무 길어질까 봐 언급하지는 않았지만 Promise 객체를 처음으로 직접 생성하며 사용해 보았다.

비록 해당 방법이 최선의 방법인지는 모르겠지만, 이를 직접 구현하면서 **axios와 Promise 객체에 대해 이해**를 할 수 있는 기회가 되었다.
