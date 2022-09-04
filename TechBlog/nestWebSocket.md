# NestJS에서 사용하는 WebSocket에 대해 알아보자

## WebSocket이란?

💡 **채팅 기능 구현을 시작하기에 앞서 NestJS의 WebSocket에 대해 알아보자. ([NestJS 공식 문서 참고](https://docs.nestjs.com/websockets/gateways))**

✔︎ 종속성 주입,  데코레이터, 예외 필터, 파이프, 가드 및 인터셉트와 같은 Nest에서 사용되는 대부분의 개념은 게이트웨이에 동일하게 적용된다.

✔︎ 가능한 경우 Nest는 동일한 구성 요소가 HTTP 기반 플랫폼, WebSocket 및 마이크로서비스에서 실행될 수 있도록 구현 세부 정보를 추상화한다.

✔︎ Nest에서 게이트웨이는 단순히 `@WebSocketGateway()` 데코레이터로 주석이 달린 클래스이다.  

✔︎ 기술적으로, 게이트웨이는 플랫폼에 구애받지 않으므로 (platform-agnostic) 접속 소켓(adapter)가 생성되면 모든 WebSockets 라이브러리와 호환된다.

✔︎ 기본적으로 지원되는 두 가지 플랫폼으로는 **socket.io**와 **ws**가 있다.

<aside>
💡 Gateway는 providers로 다뤄진다.
→ 클래스 생성자를 통해 종속성을 주입할 수 있다.
→ Gateway는 다른 클래스 (providers & controllers)에서도 주입할 수 있다.

</aside>

### 설치

```tsx
**$ npm i --save @nestjs/websockets @nestjs/platform-socket.io**
```

### Gateway

✔︎ 일반적으로 앱이 웹 애플리케이션이 아니거나 포트를 수동으로 변경하지 않는 한 각 게이트웨이는 **HTTP server**와 동일한 포트에서 수신 대기한다.

✔︎ 이 기본 동작은 80이 선택된 포트 번호인 `@WebSocketGateway(80)` 데코레이터에 인수를 전달함으로써 수정할 수 있다.

**다음과 같이 namespace를 설정할 수 있다.**

```tsx
@WebSocketGateway(80, { namespace: 'events' })
```

**다음과 같이 `@WebSocketGateway()` 데코레이터의 두 번째 인수를 통해 소켓 생성자에 해당하는 옵션을 전달할 수 있다.**

```tsx
@WebSocketGateway(81, { transports: ['websocket'] })
```

✔︎ 현재, 게이트웨이가 현재 수신 대기 중이지만 아직 어느 수신 메시지도 구독하지 않았다.

`**event` 메시지들을 구독하고 정확한 동일 데이터로 유저에게 응답하는 핸들러를 만들어보자.**

```tsx
// event.gateway.ts
import { SubscribeMessage, MessageBody } from '@nestjs/websockets';

@SubscribeMessage('events')
handleEvent(@MessageBody() data: string): string {
	return data;
}
```

**생성된 게이트웨이를 다음과 같이 모듈에 등록한다.**

```tsx
// event.module.ts

@Module({
	providers: [EventsGateway]
})
export class EventsModule {}
```

**들어오는 메시지 본문을 추출하기 위해 속성키를 데코레이터에 넣을 수 있다.**

```tsx
// events.gateway.ts

@SubscribeMessage('events')
handleEvent(@MessageBody('id') id: numebr): number {
	// id === messageBody.id
	return id;
}
```

**데코레이터 없이 다음과 같이 쓸 수 있다. (권장 x)**

```tsx
// events.gateway.ts

@SubscribeMessage('events')
handleEvent(client: Socket, data: string): string {
	return data;
}
```

✔︎ 위에서 사용된 `handleEvent()` 함수는 두 개의 인수를 받는다. 

1. 플랫폼별 소켓 인스턴스
2. 클라이언트에서 받은 데이터

**⚠️ 이 방식은 각 단위 테스트에서 소켓을 가짜 값으로 만들어야 해서 권장되는 방식은 아니다.**

✔︎ `event` 메시지 수신시 핸들러는 네트워크를 통해 전송된 것과 동일한 데이터로 승인을 보낸다.

✔︎ `client.emit()` 메서드를 이용하는 것처럼 라이브러리별 접근 방식을 사용하여 메시지를 내보낼 수 있다.

**연결된 소켓 인스턴스에 접근하려면 `@ConnectedSocket()` 데코레이터를 사용하면 된다.**

```tsx
// events.gateway.ts
import { ..., ConnectedSocket } from '@nestjs/websockets';

@SubscribeMessage('events')
handleEvent(
	@MessageBody() data: string,
	@ConnectedSocket() client: Socket,
): string {
	return data;
}
```

✔︎ 위의 경우 인터셉터를 활용할 수 없다.

✔︎ 유저에게 응답하지 않으려면 return 명령문을 건너뛰면 된다. (또는, `undefined`과 같은 falsy한 값을 명시적으로 리턴하면 된다.)

**이제 클라이언트가 메시지를 내보낼 때 다음과 같이 작성하면 된다.**

```tsx
socket.emit('events', { name: 'Nest' });
```

✔︎ handleEvent() 메소드가 실행될 것이다.

**위의 핸들러 내에서 보내진 메시지들을 수신하려면, 클라이언트는 해당 리스너에 접근해야 한다.**

```tsx
socket.emit('events', { name: 'Nest' }, data => console.log(data));
```

**다중 응답 (Multiple responses)**

✔︎ 승인(acknowledgment)은 한번만 발송되며 기본 WebSocket 구현에서는 지원되지 않는다.

✔︎ 이 제한을 해결하려면, 두 개의 속성으로 이루어진 객체를 리턴하면된다. 

**event는 보내진 이벤트의 이름이고 데이터는 클라이언트에게 전달되어야 한다.**

```tsx
// events.gateway.ts

@SubscribeMessage('events')
handleEvent(@MessageBody() data: unknown): WsResponse<unknown> {
	const event = 'events';
	return { event, data };
}
```

**들어오는 응답을 수신하려면 클라이언트가 또 다른 이벤트 리스너를 적용해야 한다.**

```tsx
socket.on('events', data => console.log(data));
```

**비동기 응답 (Asynchronous responses)**

✔︎ 메시지 핸들러는 동기 또는 비동기로 응답할 수 있다. ( → `async` 메서드 지원)

✔︎ 메시지 핸들러는 `Observable` 스트림이 완료될 때까지 결과 값을 내보내는 경우 반환활 수도 있다.

**3번 응답하는 메시지 핸들러**

```tsx
// events.gateway.ts

@Bind(MessageBody())
@SubscribeMessage('events')
onEvent(data) {
	const event = 'events';
	const response = [1, 2, 3];

	return from(response).pipe(
		map(data => ({ event, data})),
	);
}
```

**Lifecycle hooks**

3가지 유용한 Lifecycle hooks가 있다.

- `**OnGatewayInit**`
    
    ✔︎ `afterInit()` 메서드를 구현하도록 강제한다.
    
    ✔︎ **라이브버리별 서버 인스턴스를 인수**로 사용하고 필요시 나머지를 퍼뜨린다.
    
- `**OnGatewayConnection**`
    
    ✔︎ `handleConnection()` 메서드를 구현하도록 강제한다.
    
    ✔︎ **라이브러리별 클라이언트 소켓 인스턴스를 인수**로 사용한다.
    
- `**OnGatewayDisconnect**`
    
    ✔︎ `handleDisconnect()` 메서드를 구현하도록 강제한다.
    
    ✔︎ **라이브러리별 클라이언트 소켓 인스턴스를 인수**로 사용한다.
    

**서버 (Server)**

✔︎ 경우에 따라 플랫폼별 서버 인스턴스에 직접 접근할 수 있다.

✔︎ 이 객체애 대한 참조는 `afterIntit()` 메서드 (`OnGateWayInit` 인터페이스)에 대한 인수로 전달된다.

✔︎ 다른 옵션으로는 `@webSocketServer()`를 사용하는 것이 있다.

```tsx
@webSocketServer()
server: Server;
```

### Exception Filter

```tsx
throw new WsException('Invalid credentials.');
```

**WsException 구조**

```tsx
{
  status: 'error',
  message: 'Invalid credentials.'
}
```

**사용 예시**

```tsx
@UseFilters(new WsExceptionFilter())
@SubscribeMessage('events')
onEvent(client, data: any): WsResponse<any> {
  const event = 'events';
  return { event, data };
}

```

**핵심 예외 필터를 확장하고 특정 요인에 따라 동작을 재정의 하려는 경우 다음과 같이 상속을 사용할 수 있다.**

```tsx
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseWsExceptionFilter } from '@nestjs/websockets';@Catch()
export class AllExceptionsFilter extends BaseWsExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
```