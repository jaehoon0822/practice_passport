# Passport.js Documentation 

## Overview
---

passport 는 Node.js 의 인증 middleware 이다.

Passport 는 어플리케이션에 data access 같은 연관되지 않은 세부사항에 대한 위임을 깔끔고 기능적으로 캡슐화 하였다.

당신이 새로운 어플리케이션을 만들든, 기존 어플리케이션에서 작업하든 관심사 분리를 통해 Passport 는 매우 쉽게 통합할 수 있다. 

현대의 웹 어플리케이션의 인증은 여러 방법을 통해 수행할 수 있다.
전통적으로, username 그리고 password 을 제공하여 로그인한다.

Social network 에 가입한 수백만의 사람들과 함께, Facebook or Google 을 사용하는 SSO(Single Sign-On)이 대중적인 옵션이 되었다.

Web Authentication 을 둘러썬 최신 혁신은 얼굴 인식 및 지문 인식을 사용한 로그인을 허용한다.

Application architectures 역시 어떻게 인증이 달성될지에 대한 영향을 받았다.
desktop 그리고 native mobile 과 더불어 web application 에서 지원하기 위해, server-side logic 을 API 로 노출할 수 있다.

이 API 는 desktop, mobile device, Javascript 로 실행되는 browser 내에서 호출된다.
접근되는 APIs 는 일반적으로 OAuth 를 통해 발급된 token 기반 자격 증명에 의해 보호된다.

Passport 는 유연한 framework 를 제공한다.
이 framework 는 어플리케이션이 인증 mechanisms 중 어떠한 것을 사용하게 만든다.

Passport 는 인증 요청의 복잡도를 간단한 문장으로 줄여준다.

```javascript

app.post("/login/password", passport.authenticate('local'));

```

단순한 문장뒤에 숨은 세가지 기본개념은 다음과 같다.

1. Middleware
2. Strategies
3. Sessions

이 guide 는 저 컨셉들의 개념을 제공한다.
위의 개념들이 Passport 안에서 어떻게 함께 맞추어지는지 설명한다.

대부분의 일반적으로 사용되는 authentication mechanisms 은 어떻게 통합되는지 illustrate 를 통해 자세하게 설명할 것이다.

이 가이드를 읽은 이후에, 당신은 Passport 가 어플리케이션 요청에 대한 인증을 할때 어떻게 작업하는지 이해하게 될것이다.

---

## Middleware

Passport 는 인증 요청을 위해 웹 어플리케이션 내에 미들웨어로써 사용된다.
Middleware 는 Express 및 더 미니멀한 형제격인 Connect 에 의해 Node.js 에서 대중화 되었다.

이러한 인기를 감안할때, 미들웨어는 다른 웹 어플리케이션을 배우는데 더 쉽게 적응할수 있다.

다음의 코드는 username 그리고 password 와 함께 사용자 인증 경로에 대한 예제이다.

```javascript

app.post('/login/password',
    passport.authenticate('local', {
        failureRedirect: '/login',
        failureMessage: true,
    },
    function(req, res) {
        res.redirect('/~' + req.user.username);
    });

```

passport.authenticate() 는 middleware 이다.
이 미들웨어는 인증을 요청할 것이다

#### when authentication succeeds

기본적으로, 인증 성공이면 req.user property 는 인증된 user 로 설정되며, 로그인 session 은 수립되고, stack 안의 다음 function 이 호출된다.

이 다음 function 은 일반적으로 application-specific logic(어플리케이션 별 logic) 이다.
이 application-specific logic 은 user 를 대신해서 요청을 처리할 것이다.

#### when authentication fails

인증이 fail 될 때, HTTP 는 401 Unauthorized response 를 보내게 될것이고, 요청-응답 사이클은 끝나게 된다.

stack 안의 어떤 추가된 함수는 호출되지 않을 것이다.

이것은 REST(representational state transfer) 제약 준수 API 에 의해 맞추어진 기본 동작이다.
그리고 option 들을 사용하여 수정할 수 있다.

전통적인 웹 어플리케이션은 HTML page, forms를 통해 상호작용하며, <span style="background-colo: gray; color: white;">failureRedirect</span> 옵션을 일반적으로 사용한다.

<span style="background-colo: gray; color: white;">401 Unauthorized</span> 에 대한 응답 대신, 브라우저는<span style="background-colo: gray; color: white;">302 Found</span> 응답에 대해 주어진 위치로 리다이렉트 될 것이다.

이 위치는 일반적으로 login page 이며, 이 page 에는 인증실패 이후 다른 로그인 시도를 제공한다.
이것은 종종 <span style="background-colo: gray; color: white;">failureMessage</span> 옵션과 함께 쌍을 이룬다.

이는 사용자에게 표시될 수 있는 왜 인증이 실패 이유에 대한 session 으로 정보 메시지를 추가 할 수 있다.

인증요청에 사용된 mechanism 은 strategy 에 의해 구현된다. 

username, passowrd로 사용자를 인증하는 것은 OpenID Connect 를 통한 사용자 인증하는 것보다 다른 작업 모음을 수반한다.

예를들어, 2가지 다른 strategies 에 의해 2개의 mechanism 을 구현해야 된다.
위 경로에서, local strategy 는 username 그리고 password 를 검증에 사용된다.

---

## Strategies

