# NestJS의 인터페이스

## 인터페이스

### 컨트롤러

☑️ Nest의 컨트롤러는 **엔드포인트 라우팅 메커니즘**을 통해 각 컨트롤러가 받을 수 있는 요청을 분류한다. 

☑️ 컨트롤러는 요청을 받고 처리된 결과를 응답으로 돌려주는 인터페이스 역할이다.

![image](https://user-images.githubusercontent.com/73332608/186041045-5002895b-e834-4df6-9eb9-5f46e88abb7a.png)

> 컨트롤러를 사용 목적에 따라 구분하면 구조적이고 모듈화된 소프트웨어를 작성할 수 있다.
> 

### 컨트롤러 CLI 생성

✔︎ **컨트롤러 생성**

*$ nest g controller [name]*

**✔︎ 만들고자 하는 리소스의 CRUD 보일러 플레이트 코드 한 번에 생성**

*$ nest g resource [name]*

→ 위 명령어 실행시 module, controller, service, entity, dto 코드와 테스트 코드를 자동으로 생성해준다.

### 라우팅 (routing)

이미 app.controller.ts에서 localhost의 루트 경로로 요청 처리를 해준 상태이다.

```tsx
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

 

☑️ 서버가 수행해야하는 많은 귀찮은 작업을 데코레이터를 통해 어플리케이션이 가지는 핵심 로직에 집중할 수 있도록 도와준다. 

☑️ `@Controller` 데코레이터를 클래스에 선언하는 것으로 해당 클래스는 컨트롤러의 역할을 하게 된다.

☑️ getHello 함수는 `@Get` 데코레이터를 가지고 있다. 루트 경로(`’/’` 생략)로 들어오는 요청을 처리할 수 있게 되어있다. 

☑️ 라우팅 경로를 `@Get` 데코레이터의 인자로 관리할 수 있다. 

경로를 루트 경로가 아니라 /hello로 변경해보자.

```tsx
@Get("/hello")
getHello2(): string {
	return this.appService.getHello();
}
```

→ 그리고 다시 루트 경로로 요청을 보내면 404 Not found 에러가 난다.

→ [http://localhost:3000/hello](http://localhost:3000/hello) 로 요청을 보내면 정상동작하게 된다.

☑️ `@Controller` 데코레이터에도 인자를 전달할 수 있다. 이를 통해 라우팅 경로의 prefix를 지정한다.

→ `@Controller(”app”)` 지정시, [http://localhost:3000/app/hello](http://localhost:3000/app/hello) 라는 경로로 접근 가능

(prefix는 보통 컨트롤러가 맡은 리소스의 이름을 지정함) 

### 와일드카드 사용

라우팅 패스는 와일드카드를 이용하여 작성할 수 있다.

> 별포(`*`)문자를 사용하면 문자열 가운데 어떤 문자가 와도 상관없이 라우팅 패스를 구성하겠다는 의미이다.
> 

```tsx
@Get("he*lo")
getHello(): string {
	return this.appService.getHello();
}
```

☑️ 위 경로는 `helo`, `hello`, `he__lo` 와 같은 경로로 요청을 받을 수 있다.

`*`외에 `?`, `+`, `()` 문자 역시 정규 표현식에서의 와일드카드와 동일하게 동작한다. (`-`, `.` 제외)

### 요청 객체 (Request Object)

클라이언트는 어떤 요청을 보내면서 종종 서버가 원하는 정보를 함께 전송하는데, Nest는 요청과 함께 전달되는 데이터를 핸들러가 다룰 수 있는 객체로 변환한다.

→ 이렇게 변환된 객체는 `@Req()` 데코레이터를 이용하여 다룰 수 있다.

```tsx
import { Request } from "express";
import { Controller, Get, Req } from "@nestjs/common";
import { AppService } from "./app.service";

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(@Req() req: Request): string {
    console.log(req);
    return this.appService.getHello();
  }
} 
```

☑️ 요청 객체는 HTTP 요청을 나타낸다.

☑️ 요청 객체 (req)가 어떻게 구성되어 있는지 콘솔에 출력하면 다음과 같이 나온다.

![image](https://user-images.githubusercontent.com/73332608/186041086-51678bb9-26c2-4fcc-9d0b-c81cf0528c7e.png)

☑️ 다행히 우리가 API 작성시 요청 객체를 직접 다루는 경우는 거의 없다. 

☑️ Nest는 `@Query`, `@Param(key?: string)`, `@Body` 데코레이터를 이용해서 요청에 포함된 쿼리 파라미터, 패스 파라미터, 본문을 쉽게 받을 수 있도록 해준다.

### 응답

> Nest는 응답을 어떤 방식으로 처리할지 미리 정의해 두었다.
> 

☑️ 각 요청의 성공 응답 코드는 POST일 경우에만 201이고 나머지는 200이다.

☑️ 응답 본문은 string 값을 가지는데 이는 컨트롤러의 각 메서드가 리턴하는 값이다.

☑️ string, number, boolean과 같이 자바스크립트 원시 타입을 리턴할 경우 직렬화 없이 보낸다.

☑️ 객체를 리턴할 경우, 직렬화를 통해 JSON으로 자동 변환한다. 

위 경우는 CLI로 자동 생성된 경우이고 (권장), 라이브러리별 응답 객체를 직접 다루는 방법도 있다. ⬇️

Express 사용시 Express response object를 `@Res` 데코레이터를 이용해서 다룰 수 있다.

```tsx
@Get()
findAll(@Res() res) {
	const users = this.usersService.findAll();

	return res.status(200).send(users);
}
```

만약 정해진 상태코드를 다른 값으로 변경하길 원한다면 `@HttpCode`를 통해 손쉽게 바꿀 수 있다.

```tsx
import { HttpCode } from "@nestjs/common";

@HttpCode(202)
@Patch(":id")
update(@Param("id") id: string, @Body() updateUserDto: UpdateUserDto) {
	return this.usersService.update(+id, updateUserDto);
}
```

🚨 **예외 처리**

요청 도중 에러가 발생하거나 예외를 던져야 한다면 다음과 같이 처리하면 된다.

```tsx
@Get(":id")
findOne(@Param("id") id: string) {
	if (+id < 1) {
		throw new BadRequetException("id는 0보다 큰 값이어야 한다");
	}

	return this.usersService.findOne(+id);
}
```

### 헤더

> Nest는 응답 헤더를 자동 구성해준다.
> 

![image](https://user-images.githubusercontent.com/73332608/186043586-6de97644-f4ad-46a8-9117-a9a8d87d0890.png)

☑️ 응답에 커스텀 헤더를 추가하려면 `@Header` 데코레이터를 사용하면 된다.

☑️ 인자로 **헤더 이름**과 **값**을 받는다. 
(→ 라이브러리에서 제공받은 응답 객체를 사용해서 res.header() 메서드로 직접 설정도 가능하다.)

```tsx
import { Header } from "@nestjs/common";

@Header("Custom", "Test Header")
@Get(":id")
findOneWithHeader(@Param("id") id: string) {
	return this.usersService.findOne(+id);	
}
```

![image](https://user-images.githubusercontent.com/73332608/186043619-178daaf2-7dbb-49c2-8d50-6abfcdf671da.png)

### ➕➕➕ 추가

<aside>
💡 curl 명령어 사용시 자세한 정보를 얻으러면 -v(verbose) 옵션을 주면 된다.
-X 옵션을 생략하면 GET으로 동작한다.

</aside>

![image](https://user-images.githubusercontent.com/73332608/186043653-6def2b5e-e5e7-49d2-9887-9189550526b5.png)

### 리디렉션 (Redirection)

> 종종 서버는 요청 처리 후 요청을 보낸 클라이언트를 다르페이지로 이동시키고 싶은 경우가 있다.
> 

☑️ 응답 본문에 redirectUrl을 포함시켜 클라이언트가 스스로 페이지를 이동해도 되지만, `@Redirect` 데코레이터를 사용하면 쉽게 구현이 가능하다

☑️ 데코레이터의 두 번째 인자는 상태 코드이다.
→ **301 Moved Permanently**는 요청한 리소스가 헤더에 주어진 리소스로 완전히 이동됐다는 의미이다.
→ 이 상태코드를 200과 같이 다른 것으로 바꾸어 응답할 수 있다. 

⚠️ 301, 307, 308과 같이 Redirect로 정해진 응답 코드가 아닐 경우 브라우저가 제대로 반응하지 않을 수 있다. 

```tsx
import { Redirect } from "@nestjs/common";

@Redirect("https://nestjs.com", 301)
@Get(":id")
findOne(@Param("id") id: string) {
  return this.usersService.findOnoe(+id);
}
```

위 코드를 curl을 통해 응답을 받아보면 301 Moved Permanently가 나온다. 

이를 동적으로 리다이렉트해보자.

아래는 쿼리 파라미터로 버전 숫자를 전달받아 해당 버전의 페이지로 이동하는 경우이다.

```tsx
@Get("redirect/docs")
@Redirect("https://docs.nestjs.com", 302)
getDocs(@Query("version") version) {
	if (version && version === "5") {
		return { url: "https://docs.nestjs.com/v5/" };		
	}
}
```

👉 위 코드를 실행하고 브라우저에서 [http://localhost:3000/redirect/docs?version=5](http://localhost:3000/redirect/docs?version=5)를 입력하면 [https://docs.nestjs.com/v5/](https://docs.nestjs.com/v5/) 페이지로 이동하게 된다.

### 라우트 파라미터

<aside>
💡 라우트 파라미터를 전달받는 방법은 2가지가 있다.

</aside>

1. 파라미터가 여러 개 전달될 경우 객체로 한번에 받는 방법 (params 타입이 any가 되어 권장 ❎)
    
    라우트 파라미터 타입은 항상 string이므로 명시적으로 { [key: string]: string } 타입을 지정해주어도 된다.
    
    ```tsx
    @Delete(":userId/memo/:memoId")
    deleteUserMemo(@Param() params: { [key: string]: string }) {
    	return `userId: ${params.userId}, memoId: ${params.memoId}`
    }
    ```
    

1. 일반적인 방법은 아래 코드처럼 라우팅 파라미터를 따로 받는 것이다. 
→ REST API를 구성할 때 라우팅 파라미터의 개수가 너무 많아지지 않게 설계하는 것이 좋으므로 따로 받아도 코드가 많이 길어지지 않는다.
    
    ```tsx
    @Delete(":userId/memo/:memoId")
    deleteUserMemo(
    	@Param("userId") userId: string,
    	@Param("memoId") memoId: string,
    ) {
    	return `userId: ${userId}, memoId: ${memoId}`
    }
    ```
    

### 하위 도메인(Sub-Domain) 라우팅

서버에서 제공하는 기능을 API로 외부에 공개하기로 가정하자.

현재 회사에서 사용하고 있는 도메인은 example.com이고, API 요청은 api.example.com으로 받기로 했다.

즉, `http://example.com`, `http://api.example.com`로 들어온 요청을 서로 다르게 처리하고 싶다

그리고 하위 도메인에서 처리하지 못하는 요청은 원래의 도메인에서 처리하고 싶다. 

이런 경우, 하위 도메인 라우팅 기법을 사용할 수 있다.

☑️ ApiController라는 새로운 컨트롤러를 생성한다.

*$ nest g co ApiController*

☑️ app.conotroller.ts에 이미 루트 라우팅 경로를 가진 엔드포인트가 존재한다. 

☑️ ApiController에도 같은 엔드포인트를 받을 수 있도록 할 것인데 이를 위해 ApiController가 먼저 처리될 수 있도록 순서를 수정한다.

```tsx
@Module({
	controllers: [ApiController, AppController],
	...
})
export class AppModule { }
```

`@Controller` 데코레이터는 ControllerOptions 객체를 인자로 받는데 host 속성에 하위 도메인을 기술하면 된다.

```tsx
@Controller({ host: "api.example.com" }) // 하위 도메인 요청 처리 설정
export class ApiController {
  @Get() // 같은 루트 경로
  index(): string {
    return "Hello, API"; // 같은 응답
  }
}
```

**⚠️ 로컬에서 테스트를 하기 위해 하위 도메인을 api.localhost로 지정하면 curl 명령어가 제대로 동작하지 않는다. 이를 해결하기 위해서는 /etc/hosts 파일의 마지막에 `127.0.0.1 api.localhost` 을 추가하고 서버를 다시 구동하면 된다.**

```tsx
@Controller({ host: "api.localhost" }) // 하위 도메인 요청 처리 설정
export class ApiController {
  @Get() // 같은 루트 경로
  index(): string {
    return "Hello, API"; // 같은 응답
  }
}
```

```
$ curl http://localhost:3000
Hello World!

$ curl http://api.localhost:3000
Hello, API
```

요청 패스를 `@Param` 데코레이터로 받아 동적으로 처리한 것과 같이, `@HostParam` 데코레이터를 이용하면 서브 도메인을 변수로 받아올 수 있다.

API 버저닝을 하는 방법이 여러가지 있지만 하위 도메인을 이용하는 방법을 많이 사용한다.

아래와 같이 하위 도메인 라우팅으로 쉽게 API를 버전별로 분리할 수 있다.

```tsx
@Controller({ host: ":version.api.localhost" })
export class ApiController {
	@Get()
	index(@HostParam("version") version: string): string {
		return `Hello, API ${version}`
	}
}
```

```
$ curl http://v1.api.localhost:3000
Hello, API v1

$ curl http://api.localhost:3000
Hello World!
```

### 페이로드 다루기

☑️ POST, PUT, PATCH 요청은 보통 처리에 필요한 데이터를 함께 실어 보낸다.

→ 이 페이로드를 body라고 한다.

☑️ NestJS는 body를 DTO(Data Transfer Object)를 정의하여 쉽게 다룰 수 있다.

회원가입을 하기 위해 이름과 이메일을 추가해보자

```tsx
export class CreateUserDto {
	name: string;
	email: string;
}
```

```tsx
@Post()
create(@Body() createUserDto: CreateUserDto) {
	const { name, email } = createUserDto;
	
	return `유저를 생성했다. 이름: ${name}, 이메일: ${email}`;
}
```

```
$ curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d '{"name": "Dexter", "email": "dexter.haan@gmail.com"}'
유저를 생성했습니다. 이름: Dexter, 이메일: dexter.haan@gmail.com
```

☑️ GET 요청에서 서버에게 전달할 데이터를 포함할 때는 일반적으로 요청 주소에 포함시킨다. 

→ 예를 들어 유저 목록을 가져오는 요청은 `GET /users?offset=0&limit=10` 과 같이 페이징 옵션이 포함되도록 구성할 수 있다.

이를 `@Query` DTO로 처리할 수 있다.

```tsx
export class GetUsersDto {
	offset: number;
	limit: number;
}
```
