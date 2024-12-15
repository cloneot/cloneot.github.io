---
title: "NestJS에서 Passport 사용하여 OAuth 2.0 구현하기"
layout: post
categories: development
tags:

- web
- nestjs
- oauth
- auth
---

## Goal

- NestJS에서 Passport를 활용하여 OAuth 2.0 구현하기 (세션 기반 인증)
- passport, passport-oauth2, passport-google-oauth20, nestjs/passport 코드 살펴보기
- 키워드: NestJS, TypeScript, Passport, Session, OAuth 2.0, OIDC



## Non-Goal

- Youtube Data API 사용법 설명하기
- JWT 기반 인증
- passport 기본 동작 자세히 설명하기
- 결정사항들에 대한 근거 자세히 설명하기



## Background

- [NestJS](https://nestjs.com/)를 이용해 API 서버를 구현한 내 프로젝트(pl-maker)에서 [Youtube Data API](https://developers.google.com/youtube/v3)(이하 Youtube API)를 사용하고 싶다
  - 동영상 조회
  - 재생목록 조회 및 수정
- Google의 [OAuth 2.0](https://developers.google.com/identity/protocols/oauth2) 및 [OpenID Connect](https://developers.google.com/identity/openid-connect/openid-connect)(이하 OIDC)를 이용하여, 인가 및 인증을 처리하고 싶다
  - 인가(Authorization): 특정 리소스나 기능에 대한 접근 권한을 부여하는 과정
    - 표준화된 OAuth 2.0 절차를 거치면 access token 및 refresh token을 얻을 수 있음
    - access token: 유튜브 API를 호출할 때 헤더에 넣어 전송
    - refresh token: 만료된 access token을 재발급할 때 사용
  - 인증(Authentication): 사용자의 신원을 확인하는 과정
    - 표준화된 OIDC 절차를 거치면 id token을 얻을 수 있음 (OAuth 2.0과 유사)
    - id token: 사용자 식별 정보가 담긴 JSON Web Token (만료시점, 고유한 유저 식별자, 이메일, ... )
- [Passport](https://www.passportjs.org/)를 사용하여 OAuth 2.0, OIDC 절차를 수행하고 싶다
  - 보안이 중요하다 (Youtube 권한 일부를 사용하므로)
  - Passport is most popular node.js authentication library - https://docs.nestjs.com/recipes/passport
  - 토큰이 아닌 세션 기반 인증을 사용하고 싶다



## Overview

- 내 프로젝트는 (1) NestJS API 서버, (2) 리액트로 구성되고, 유튜브 기능을 이용할 때 다음 절차를 거친다:
  1. 유저가 브라우저(리액트)에서 NestJS API 서버로 요청을 보낸다 (fetch API 사용)
  2. NestJS API 서버에서 적절한 Youtube API를 선택해, 유저의 access token을 포함하여 Youtube API 요청을 보낸다
  3. NestJS API 서버에서 Youtube API 응답을 받아 적절한 가공 및 후작업 후에, 리액트에 데이터를 반환한다
  4. 리액트에서 데이터를 가지고 렌더링한다
- 따라서, 
  - 프론트 단에는 NestJS API 서버로의 로그인 여부를 관리하는 세션 ID(및 유저 ID)만 저장하고, (후에 나올 serializeUser의 반환값)
  - 백엔드에서 대부분의 인가 및 인증 정보를 관리한다 (OAuth 2.0 토큰 등을 포함하여)


- 구현하고자 하는 OAuth 2.0 프로토콜(authorization code grant)에서 NestJS API 서버가 해야 하는 일은 다음과 같다:
  1. 유저가 구글 로그인할 수 있는 authorization url 생성하기
     - 1의 auth url에 유저가 접속하여 권한 부여에 동의하고 구글 계정으로 로그인 → 구글 auth server에서 로그인 정보 인증 후, 미리 지정해놓은 NesJS API 서버 측 redirect uri로 authorization code를 쿼리 스트링에 담아 전송
  2. redirect uri에 auth code가 오면, 구글 auth server에 auth code를 전송하여 OAuth 2.0과 OIDC에 필요한 토큰 발급받기
     - access token, refresh token을 적절한 위치에 저장하기
     - 구글 계정마다 고유한 값을 가지는, [id token의 sub 필드](https://developers.google.com/identity/openid-connect/openid-connect#an-id-tokens-payload)를 토대로 인증 진행하기
       - 권한 요구 목록인 scope에 'openid'를 포함할 경우, id token도 발급됨
       - 처음 로그인한 유저일 경우, 회원가입 절차를 진행하거나 자동으로 회원가입
  3. 유저가 Youtube API가 필요한 요청을 할 때마다, 해당 유저의 access token을 이용해 Youtube API 호출



## Passport - Express

껍데기 자체는 간단하여 코드를 직접 보는 게 이해하기 수월하다. 

- https://github.com/jaredhanson/passport
- https://github.com/jaredhanson/passport-oauth2
- https://github.com/jaredhanson/passport-google-oauth2



최초에 express를 대상으로 만든 라이브러리인 만큼 express 예제를 보는 것이 편하다. 

- [Passport strategy for Google OAuth 2.0](https://www.passportjs.org/packages/passport-google-oauth2/) 
- [[Node.js] passport Google OAuth 2.0 로그인 사용하기](https://millo-l.github.io/Nodejs-passport-Google-OAuth-2-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/#7-googlejs)

간단히 말해 OAuth 2.0 인증을 위해서, 다음 내용만 구현하면 된다: 

- OAuth 2.0 과정에 필요한 정보 지정 (client id, client secret, redirect uri, scope, token url, access type, ...)
- 로그인 성공시 수행할 verify 콜백 함수: `function(accessToken, refreshToken, profile, done)`
- 세션에 저장할 유저 관련 정보 지정(로그인 성공시 호출): `serializeUser`
- 세션의 정보를 가지고 유저 정보 복원하여 req.user에 저장(매 요청마다 호출): `deserializeUser`
- 다음 두 컨트롤러에 passport.authenticate 미들웨어 붙이기
  1. 소셜 로그인 화면으로 리다이렉트할 컨트롤러: `GET /auth/login` (or `POST /auth/login`, fetch API 이슈 때문)
  
  2. auth code가 전달될 redirect uri에 대한 컨트롤러: `GET /auth/callback`



로그인, 로그아웃, 로그인 여부 판단 등은 passport에서 제공하는 메소드를 활용하면 된다. (logIn, logOut, isAuthenticated)

다음 OAuth 2.0 과정은 passport-oauth2 및 passport-google-oauth20에 구현되어 있다:

- passport-oauth2

  - OAuth 2.0의 authorization code grant 프로토콜 (csrf token, pkce 등도 사용 가능)

    - lib/strategy.js의 `OAuth2Strategy.prototype.authenticate`

    - request의 query string에 'code'가 있으면, auth code를 이용해 토큰을 발급받는다

      ```javascript
      if ((req.query && req.query.code) || (req.body && req.body.code)) {
        function loaded(err, ok, state) {
          self._oauth2.getOAuthAccessToken(code, params,
            function(err, accessToken, refreshToken, params) {
              // ...
            }
          );
        }
        // ...
            this._stateStore.verify(req, state, meta, loaded);
        // ...
      }
      ```

    - 포함되어 있지 않다면, authorization url로 리다이렉트하여 소셜 로그인 창을 띄운다

      ```javascript
      else {
        var params = this.authorizationParams(options);
        params.response_type = 'code';
        if (callbackURL) { params.redirect_uri = callbackURL; }
        // ...
          var parsed = url.parse(self._oauth2._authorizeUrl, true);
          utils.merge(parsed.query, params);
          parsed.query['client_id'] = self._oauth2._clientId;
          delete parsed.search;
          var location = url.format(parsed);
          self.redirect(location);
        // ...
      }
      ```

- passport-google-oauth20 (패키지 이름은 passport-google-oauth2)

  - passport-oauth2를 상속받는다: `util.inherits(Strategy, OAuth2Strategy);`

  - lib/strategy.js의 `function Strategy`

    - Google OAuth와 관련된 실질적인 값이 저장된 걸 확인할 수 있다

    ```javascript
      options.authorizationURL = options.authorizationURL || 'https://accounts.google.com/o/oauth2/v2/auth';
      options.tokenURL = options.tokenURL || 'https://www.googleapis.com/oauth2/v4/token'; 
    ```

  - lib/strategy.js의 `Strategy.prototype.userProfile = function(accessToken, done) { ... }`

    - accessToken을 이용해 id token 정보 얻어옴



## Passport - NestJS

- https://github.com/nestjs/passport

NestJS에서는 구현해야 하는 위치가 흩뿌려져 있어, Express보다는 조금 복잡하다. 

앞에서 언급한 구현해야 하는 내용들을, NestJS에서는 다음 위치에 구현해야 한다. 

- OAuth 2.0 정보 및 verify 콜백 함수: `PassportStrategy`를 상속받은 클래스의 생성자 및 validate 메소드
- serializeUser / deserializeUser: `PassportSerializer`를 상속받은 클래스의 serializeUser, deserializeUser 메소드
- 2개 컨트롤러에 미들웨어 붙이기: `AuthGuard`를 상속받은 클래스의 canActivate 메소드를 구현하고, 해당 컨트롤러에 가드 붙이기



다음과 같이 구현하는 게 일반적인 듯하다. (TODO: details 태그로 바꾸기)


```typescript
// google.strategy.ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { Profile, Strategy, VerifyCallback } from 'passport-google-oauth20';
import { ConfigService } from '@nestjs/config';
import { UsersService } from 'src/users/users.service';

@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor(
    private readonly configService: ConfigService,
    private readonly usersService: UsersService,
  ) {
    super({
      clientID: configService.get('oauth.google.clientID'),
      clientSecret: configService.get('oauth.google.clientSecret'),
      callbackURL: configService.get('oauth.google.callbackURL'),
      scope: configService.get('oauth.google.scope'),
      tokenURL: 'https://oauth2.googleapis.com/token',
      accessType: 'offline',
    });
  }

  // optional
  authorizationParams(): { [key: string]: string } {
    return {
      access_type: 'offline',
      prompt: 'consent',
    };
  }

  async validate(
    accessToken: string,
    refreshToken: string,
    profile: Profile,
    done: VerifyCallback,
  ) {
    const { id, displayName: username } = profile;

    const user = this.usersService.findOrCreate({
      username: username,
      googleSub: id,
      accessToken: accessToken,
      refreshToken: refreshToken,
    });
    done(null, user);
  }
}
```

```typescript
// auth.serializer.ts
import { Injectable } from '@nestjs/common';
import { PassportSerializer } from '@nestjs/passport';
import { UserEntity } from 'src/users/user.entity';
import { UsersService } from 'src/users/users.service';

@Injectable()
export class AuthSerializer extends PassportSerializer {
  constructor(private readonly usersService: UsersService) {
    super();
  }

  serializeUser(user: UserEntity, done: CallableFunction) {
    const payload = { userId: user.userId, username: user.username };
    done(null, payload);
  }

  async deserializeUser(
    payload: { userId: number; username: string },
    done: CallableFunction,
  ) {
    const user = await this.usersService.findOne(payload.userId);
    done(null, user);
  }
}
```

```typescript
// google-auth.guard.ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class GoogleAuthGuard extends AuthGuard('google') {
  constructor() {
    super();
  }

  async canActivate(context: ExecutionContext) {
    const activate = (await super.canActivate(context)) as boolean;
    const request = context.switchToHttp().getRequest();
    await super.logIn(request);
    return activate;
  }
}
```



편의를 위해 다음을 추가할 수 있다:

- LoggedIn 가드: 로그인되어 있을 때만 접근을 허용

  ```typescript
  // logged-in.guard.ts
  
  import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
  
  @Injectable()
  export class LoggedInGuard implements CanActivate {
    canActivate(context: ExecutionContext) {
      return context.switchToHttp().getRequest().isAuthenticated();
    }
  }
  ```

- User 데코레이터: `@User user` 꼴로 req.user를 편하게 사용

  ```typescript
  // user.decorator.ts
  
  import { ExecutionContext, createParamDecorator } from '@nestjs/common';
  
  export const User = createParamDecorator<unknown, ExecutionContext>(
    (data: unknown, ctx: ExecutionContext) => {
      const request = ctx.switchToHttp().getRequest();
      return request.user;
    },
  );
  ```

- 활용 예

  ```typescript
  // users.controller.ts
  
  @Controller('users')
  export class UsersController {
    @UseGuards(LoggedInGuard)
    @Get('me')
    findMe(@User() user: UserEntity) {
      return user;
    }
  }
  ```



그리고 NestJS에서는 다음 부분을 신경써줘야 한다:

- PassportModule를 imports에 포함할 때, `PassportModule.register({ session: true })`로 import해야 한다

  - [defaultOptions](https://github.com/nestjs/passport/blob/master/lib/options.ts)이 `export const defaultOptions = { session: false, property: 'user' };`이다

- **await** req.logOut를 쓰고 싶으면 promisify 미들웨어를 추가해야 한다 (req.logout은 식별자만 다른 동일 함수)

  - req.logIn / req.logOut 구현체: https://github.com/jaredhanson/passport/blob/master/lib/http/request.js

  - 이들은 session manager의 logIn/logOut을 호출 (session의 키 'passport'는 옵션으로 변경가능)

    - logIn: req.session['passport'].user에 serializeUser의 결과 저장
    - logOut: `delete req.session['passport'].user`

  - 비동기 처리

    - [SessionManager.prototype.logOut](https://github.com/jaredhanson/passport/blob/217018dbc46dcd4118dd6f2c60c8d97010c587f8/lib/sessionmanager.js#L57) 는 콜백 기반임: `function(req, options, cb)`
    - async, await를 사용하기 위해서는 promisify가 필요
    - req.logIn/req.logOut을 promisify하는 미들웨어를 추가하여 해결 가능
    - callback 그대로 쓸거면 직접 new Promise로 api 서버에서 처리해줘야 됨
    - 다음 방법을 쓸 수도 있다고 한다: `await promisify(req.session.destroy.bind(req.session))();`
    
    ```typescript
    // main.ts
    
    function promisifyPassport(req: any, res: Response, next: NextFunction) {
      req.logIn = promisify(passportHttpRequest.logIn);
      req.logOut = promisify(passportHttpRequest.logOut);
      next();
    }
    
    // ...
    
    async function bootstrap() {
      // ...
      app.use(session({ /* ... */ }));
      
      app.use(promisifyPassport);
      app.use(passport.initialize());
      app.use(passport.session());
      
      await app.listen(port);
    }
    bootstrap()
    ```



## Troubleshooting

- google oauth20 strategy이 무엇을 구현한 것인가?
  - oauth 2.0 과정에서 필요한 루틴만 (authorization code grant protocol)
  - 로그인 여부를 판단하는 guard는 별도로 생성해야 함 (logged-in.guard.ts)
  - Q) req.user는 자동으로 붙여주나? A) deserializeUser에서 붙여줌
- req.session에 passport 정보가 없음
  - SPA에서 fetch API로 api 서버와 통신한다면, cookie의 origin은 SPA의 것으로 설정됨
  - 둘의 origin이 다르면, CORS 문제가 발생. cookie는 origin 별로 관리됨
    - SPA: `pl-maker.netlify.app`
    - API: `pl-maker.cloneot.dev`
  - cross origin으로 cookie를 보내기 위해, SPA의 fetch API 옵션과, api 서버의 enable cors 설정에서 credentials: true 세팅
- req.logOut 호출해도 세션이 삭제되지 않음
  - req.logIn, req.logOut이 callback 구조로 작성되어 있어서 그럼 (promise 버전 없음)
  - app.use(passport.initialize()) 미들웨어에서 req.logIn, req.logOut 함수를 attach함
    - 이때 이미 해당 프로퍼티가 있으면 건너뜀
    - 솔루션: req.logIn, req.logOut 프로퍼티에 promisify()하여 미리 넣어둠
- fetch API redirect CORS 이슈
  - https://stackoverflow.com/questions/21451172/how-to-integrate-oauth-with-a-single-page-application
  - SPA에서 fetch로 /auth/login 요청 보내면, API 측의 guard가 알아서 auth url로 redirect함
  - 근데 이게 url이 바뀌는 판정이 아니라, 새로운 url로 SPA에서 fetch가 날아가는 판정인듯 (추측)
  - 구글 로그인 페이지는 우리에게 CORS 허용을 안해줬으므로 망함 (프론트, 백 둘 다 마찬가지)
  - 솔루션: (1) form을 쓴다, (2) api 서버에서 redirectUri를 반환하면 SPA에서 window.location을 변경한다
    - form에서는 cross orign이라도 디폴트가 알아서 cookie를 보내는 듯..?



## Review

- 다음에 NestJS에서 OAuth를 구현해야 되면 다른 라이브러리를 알아볼 듯하다
  - NestJS에서는 Express에서만큼 간결하지 않다
  - Typescript랑 잘 맞는지 모르겠다
    - ctrl+click으로 따라가면 아무 도움 안되는 타입 시그니쳐만 뜬다
    - passport 깃헙에서 js 코드를 직접 보면 이해는 되지만, 오래됐단 느낌이 팍팍 든다 (하위호환성을 지키려는 노력이 보인다...)
  - logout promisify 미들웨어를 직접 추가해야 한다
- OAuth 2.0의 authorization code grant 프로토콜은 기억한 것 같다
  - passport 라이브러리를 사용하기 전에 직접 구현해본 게 도움이 되었다
  - 배포할 때는 csrf token, pkce을 추가해야 한다
- SPA와 API 서버를 사용할 때, 리다이렉트를 어떤 식으로 구현하는 게 이상적인지 모르겠다
  - 새로고침 없이 fetch API만으로 모든 걸 처리하고 싶었는데 CORS 에러 때문에 막혔다
- 테스트를 어떻게 진행해야 할지 잘 모르겠다
  - 자동 로그인이 잘 되는지, 특정 시간이 지나고 풀리는지, 풀린 후 재로그인할 때 권한 요청 창이 다시 안 뜨는지
  - postman을 사용할 때의 e2e 테스트 같은 걸 하고 싶은데, 유저 → SPA → API 서버 순으로 접근해야 하고, 로그인 세션 정보는 SPA 단에 붙어서 간단하게는 불가능하다



## References

- OAuth 2.0
  - OAuth 2.0 동작 방식의 이해: https://blog.naver.com/mds_datasecurity/222182943542
  - Using OAuth 2.0 to Access Google APIs: https://developers.google.com/identity/protocols/oauth2
- Passport
  - Nest.js에서 멀티 프로세스 용 세션 로그인 구현하기: https://blog.naver.com/biud436/222841223210
  - Passport (authentication): https://docs.nestjs.com/recipes/passport
  - Passport strategy for Google OAuth 2.0: https://www.passportjs.org/packages/passport-google-oauth2/
  - Understanding passport serialize deserialize: https://stackoverflow.com/questions/27637609/understanding-passport-serialize-deserialize
  - [Node.js] passport Google OAuth 2.0 로그인 사용하기: [https://millo-l.github.io/Nodejs-passport-Google-OAuth-2-로그인-사용하기/](https://millo-l.github.io/Nodejs-passport-Google-OAuth-2-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/)
