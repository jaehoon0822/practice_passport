# Passport.js Documentation 

## Overview

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

### when authentication succeeds

기본적으로, 인증 성공이면 req.user property 는 인증된 user 로 설정되며, 로그인 session 은 수립되고, stack 안의 다음 function 이 호출된다.

이 다음 function 은 일반적으로 application-specific logic(어플리케이션 별 logic) 이다.
이 application-specific logic 은 user 를 대신해서 요청을 처리할 것이다.

### when authentication fails

인증이 fail 될 때, HTTP 는 401 Unauthorized response 를 보내게 될것이고, 요청-응답 사이클은 끝나게 된다.

stack 안의 어떤 추가된 함수는 호출되지 않을 것이다.

이것은 REST(representational state transfer) 제약 준수 API 에 의해 맞추어진 기본 동작이다.
그리고 option 들을 사용하여 수정할 수 있다.

전통적인 웹 어플리케이션은 HTML page, forms를 통해 상호작용하며, <span style="background-colo: gray; color: white;">failureRedirect</span> 옵션을 일반적으로 사용한다.

<span style="background-colo: gray; color: white;">401 Unauthorized</span> 에 대한 응답 대신, 브라우저는<span style="background-colo: gray; color: white;">302 Found</span> 응답에 대해 주어진 위치로 리다이렉트 될 것이다.

location는 일반적으로 login page 이며, 이 page 에는 인증실패 이후 다른 로그인 시도를 제공한다.
이것은 종종 <span style="background-colo: gray; color: white;">failureMessage</span> 옵션과 함께 쌍을 이룬다.

이는 사용자에게 표시될 수 있는 왜 인증이 실패 이유와 함께, 해당 내용을 session 으로 정보 메시지를 추가 할 수 있다.

인증요청에 사용된 mechanism 은 strategy 에 의해 구현된다. 

username, passowrd로 사용자를 인증하는 것은 OpenID Connect 를 통한 사용자 인증하는 것보다 다른 작업 모음을 수반한다.

예를들어, 2가지 다른 strategies 에 의해 2개의 mechanism 을 구현해야 된다.
위 경로에서, local strategy 는 username 그리고 password 를 검증에 사용된다.

## Strategies

Strategies 는 인증 요청에 대한 책임을 가지며, authentication mechanism 을 구현하여 달성한다.
Authentication mechanisms 은 요청(request)에서 password 또는 identity provider(idP) 로 부터의 표명(assertion) 같은 자격증명을 어떻게 부호화 하는지 정한다.

또한, 자격증명에 대한 필요한 검증 절차를 지정한다.
만약, 성공적으로 검증된 자격증명이라면 요청은 인증된다. 

Strategies 는 많은 종류의 authentication mechanisms 그리고 이에 상응하는 갖가지 strategies 가 있다.
Strategies 는 구분된 package 로 배포되며 반드시 설치, 등록, 설정 해야만 한다.

### Install

passport-local 패키지는 username 과 password 에 대한 인증 strategy 를 제공한다.
npm 을 통해 install 한다.


```sh
$ npm i passport-local; npm i passport-openidconnect;
```

여기서 passport-openidconnect 는 OpenID Connect 에 대한 지원을 구현한 package 이다.
Developer 는 필요한 페키지를 설치하고, 어플리케이션에 의해 요청된 인증 메커니증을 제공한다.
그 패키지는 passport 에 연결된다.

이는 불필요한 의존을 방지되며 어플리케이션 사이즈를 줄여준다.

### Configure

패키지가 설치되면, strategy 는 설정할 필요가 있다.
그 설정은 각 인증 메커니즘과 함께 다양하다. 그래서 서술된 documentation 을 찾아야만 한다. 
하지만, 거기에는 공통된 패턴들이 있다.

다음 코드는 LocalStrategy 설정에 대한 예제이다.

```javascript

const localStrategy = require("passport-local");

const strategy = new LocalStrategy(function verify(username, password, cb) {
    db.get("select * from users where username = ?", [ username ], function(err, user) {
        if (err) { return cb(err} }
        if (!user} { return cb(null, false, { message: "Incorrect user or passowrd" } }
        // cb 는 첫번째 파라미터로 err 를 받으며,
        //       두번째는 value,
        //       세번째는 전달한 message 객체를 받는듯하다.

        // pbkdf2 는 crypto module 의 method 이다.
        // 이는 password 를 hashed 시켜, 암호화시키는데 많이 사용된다.
        crypto.pbkdf2(password, user.salt, 310000, 32, "sha256", function (err, hashedPassword) {
            if (err) { return cb(err); }
            if (!crypto.timingSafeEqual(user.hashedPassword, hashedPassword)) {
                return cb(null, false, { message: "Incorrect username or password" } );
            }

            return cb(nul, user);
        });
    });
}); 

```

### Verfify Function

localStrategy 생성자는 argument 로 함수를 가진다.
이 함수는 많은 strategies 에서 일반적인 패턴으로 verify 함수로써 알려져있다. 

strategy 는 요청 인증을 할때, request 안에 포함된 자격증명을 구문 분석한다.
verify 함수는 그때 호출된다. 이 함수는 사용자가 자격증명에 속하는지 결정할 책임이 있다.

이를 통해 data acess 를 application 에 위임할 수 있게 된다.

연습한 예제안, verify 함수는 data base로 부터 user record 를 얻는 SQL query를 실행하고,
password 를 검증한 이후 strategy 에 record 를 반환하여 사용자 인증 그리고 로그인 세션을 수립한다.

verify 함수는 어플리케이션 자체에서 제공되기 때문에, 영구 저장소의 access는 어떤 식으로든 제약을 받지 않는다.

이 어플리케이션은 relational database, graph databases 또는 document stores 를 포함한 데이터 저장 시스템을 자유롭게 사용할 수 있으며, 모든 스키마에 따라 해당 데이터베이스 내의 데이터를 구조화 한다. 

verify 함수는 strategy 별로 다르며, arguemnts 를 받고 생성되는 parameter 는 인증 매커니즘에 따라 다르다. 

인증 메커니즘을 위해 password 같은 공유 screts 를 포함하며, verify 함수는 자격증명을 검증을 위한 책임 그리고 사용자를 생성한다.

암호화 인증을 제공하기 위한 매커니즘을 위해 verify 함수는 user 그리고 key 를 산출하며, key는 strategy 에서 암호화된 자격증명을 검증하는데 사용된다.

verfy 함수는 success, failuer, error 세가지 조건 중 하나에 양보한다.

verify 함수가 자격증명에 속한 user 를 찾고 자격증명이 유효하다면, 인증 user 와 함께 callback 함수를 호출한다. 

``` javascript

return cb(null, user);

```

만약 알고있는 사용자가 자격증명에 속하지 않거나, 유효하지 않다면, verify function 은 인증 실패를 나타내는 false 와 함께 callback 함수를 호출한다.

```javascript

return cb(null, false);

```

만약 database 를 사용할 수 없음같은 error 가 발생한다면, callback 은 error 와 함께 호출된다.
이때, error callback 은 관용적인 Node.js 스타일을 사용한다.

```javascript

return cb(err);

```

이것은 발생할 수 있는 2가지 failure case 사이를 구별하는것이 중요하다.

authentication failures 는 비록 사용자로부터 유효하지 않은 자격증명을 받았더라도 server 는 정상작동 되는 조건을 예상한다.

서버가 비정상적으로 작동한다면 err 를 설정하며, 이는 internal error 를 나타낸다. 

## Register

strategy 이 구성되면, 그때 .use() 호출하여 등록된다.

```javascript

const passport = require("passport");

passort.use(strategy);

```

모든 strategies 는 관습적으로, package 이름에 따른 passport-{name} 패턴을 통해 일치된다.
위의 LocalStrategy 경우, 배포된 passport-local 패키지에서의 local 로 이름으로 구성된다.

한번 등록되면, strategy 는 passport.authenticate 미들웨어의 첫번째 argument 로써 strategy 의 이름 전달하여 요청을 인증하는데 사용할 수 있다.

```javascript

app.post("/login/password", 
    passport.authenticate('local', {
        failureRedirect: "/login",
        failureMessage: true,
    }),
    function(req, res) {
        res.redirect("/~" | req.user.username);
    });

```

간결함을 위해, 종종 strategies 는 단일문으로 등록되고 구성한다.

```javascript

const passport = require("passport");

passport.use(new LocalStrategy(function verify('local', user, password, cb) {
    // ...
}));

```

## Sessions

web application 은 page 에서 page 로 이동할때 page 마다 사용자를 식별할 능력이 필요하다.
각 동일한 사용자와 연관된 응답 및 요청의 나열을 session 이라 한다.

HTTP 는 stateless protocol 이다.
이는 어플리케이션의 각 요청을 앞전의 요청에 의한 context 없이, 개별적으로 이해할 수 있는것을 의미한다.

인증된 사용자가 어플리케이션을 탐색할때 후속 요청에서 기억해야 하므로 로그인한 사용자가 있는 웹 어플리케이션에서는 문제가 된다.

이 과제를 해결하기 위해 web application 은 session 을 만들었다.
session 은 사용자 브라우저 그리고 application 서버사이에 상태를 유지할 수 있도록 허용한다.

session 은 browser 의 HTTP cookie 를 설정하여 수립한다. 
그런 다음 browser 는 서버로 매번 요청할때 cookie 를 전달한다.

서버는 cookie 의 값을 사용하여 여러 요청에서 정보를 검색한다. 
이것은 HTTP 위에 stateful protocol 을 생성한다.

인증 상태를 유지하기 위해 session 이 사용되는 동안, 어플리케이션에서 인증과 관련되지 않은 다른 상태를 유지하는데 사용될 수 있다.

Passport 는 session 안에 저장된 다른 상태로 부터 login session 이라고 하는 인증 상태를 격리하도록 신중하게 설계되었다.

Application 은 login session 을 사용하도록 만들기 위해 반드시 초기화된 session 지원을 초기화한다. 
Express app 은 session 지원은 express-session 미들웨어를 사용하여 추가된다.

```javascript

const session = require("express-session");

app.use(session({
    secret: "keyboard cat",
    resave: false,
    saveUninitialized: false,
    cookie: { secure: true },
}));


```

Passport 는 login session 을 유지하기 위해, session 으로 부터 사용자 정보를 serializes 그리고 deserializes 한다.

저장되는 정보는 serializeUser 그리고 deserializeUsr 함수를 제공제공하는 어플리케이션에 의해 결정된다.

```javascript

passport.serializeUser(function(user, cb) {
    process.nextTick(function () {
        return cb(null, {
            id: user.id,
            username: user.username,
            picture: user.picture,
        });
    });
});

passport.deserializeUser(function(user, cb) {
    process.nextTick(function() {
        return cb(null, user);
    });
});


```

이 로그인 세션은 사용자가 자격증명을 사용하는 인증을 성공하면 수립된다.
다음경로는 사용자가 사용하는 username, password 인증일 것이다.

만약 성공적으로 검증되었다면, Passport 는 serializeUser 함수를 호출한다.
호출된 함수는 위 예제안에서 사용자 ID, username, picture 를 저장한다.

address 또는 birthday 같은 사용자의 다른 모든 프로퍼티는 저장되지 않는다.

```javascript

app.post("/login/password"
    passport.authenticate('local', {
        failureRedirect: "/login",
        failureMessage: true,
    },
    function(req, res) {
        res.redirect("/~" + req.user.username);
    }),
);

```

사용자가 page 에서 page 로 이동할때, built-in session strategy 를 사용하여 session 자신을 인증이 수 있다.

왜냐하면, 인증된 session 은 어플리케이션의 대부분의 경로에 일반적으로 필요하기 때문에
이후 session 미들웨어에서 어플리케이션 레벨 미들웨어로써 공동적으로 사용한다.

```javascript

app.use(session(/* ... */));
app.use(passport.authenticate("session");

```

이 또한 달성할수 있으며, 좀더 간결하게 passport.session() 별칭을 사용한다.

```javascript

app.use(session(/* ... */));
app.use(passport.session());

```

session 이 인증되었을때, Passport 는 deserializeUser 함수를 호출한다.
deserializeUser 함수는 앞전에 저장된 userID, username, picture 를 생성한다.  
그런다음 req.user 프로퍼티에 생성된 정보로 설정된다.

거기에는 session 안의 저장된 데이터의 양과 인증할때 발생하는 데이터베이스 로드 사이에는 내장된 장단점이 있다.

이 장장점은 cookie-session 같은 패키지를 사용하여 session 데이터를 저장할때 server 보다는 client 에 특히 적절하다.

session 안에 더 적은 데이터 정보를 얻는데 대한 db 로 부터의 더 많은 query 가 요구된다.
반대로 session 에 더 많은 데이터를 저장하면 데이터베이스 쿼리가 줄어들고 잠재적으로 쿠키에 저장할 수 있는 최대 데이터양을 초과할 수 있다.

이 장단점은 제공되는 deserializeUser 그리고 serializeUsre, 어플리케이션에 의해 컨트롤된다.

위 예제와 달리, 다음의 예제는 session 인증된 모든 요청에 대해 db  query 를 하는 대신 session 에 저장된 데이터를 최소화한다.

```javascript

passport.serializeUser(function(user, cb) {
    process.nextTick(function() {
        return cb(null, user.id);
    });
});

passport.deserializeUser(function(id, cb) {
    db.get("select * from user where id = ?", [ id ], function() {
        if (err) return cb(err);
        return cb(null, user);
     });
});

```

이 절충에 균형을 맞추기 위해, 어플리케이션에 대한 모든 요청에 필요한 모든 사용자 정보를 session 에 저장하는것이 좋다.

예를 들어, 만약 어플리케이션에 user element 에 user's name, email , address, photo 를 모든 페이지에 표시해야 한다면 session 에 저장되어 빈번한 db query 를 제거해야 한다.


배송 주소같은 추가 정보가 필요한 체크아웃 페이지와 같은 특정 경로는 해당 데이터를 데이터베이스에 쿼리할 수 있다.


이렇게 passport 의 3가지 개념에 대해서 알아보았다.
다음은 강좌를 보며 직접 구현해볼 생각이다.
