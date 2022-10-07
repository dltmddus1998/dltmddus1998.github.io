# 당근 마켓 백엔드 클론코딩 Dev-log (6) - 웹소캣에 대해 알아보고 실시간 채팅 구현하기

## ⚒ 데이터베이스 구성

### chat.entity.ts

```tsx
import { Field, ObjectType } from '@nestjs/graphql';
import { BaseEntity, Column, CreateDateColumn, Entity, JoinColumn, ManyToOne, OneToMany, PrimaryGeneratedColumn } from 'typeorm';
import { User } from 'src/users/entities/user.entity';
import { ChatRoom } from './chatRoom.entity';
import { ChatComplaints } from './chatComplaints.entity';

@Entity()
@ObjectType()
export class Chat extends BaseEntity {
  @Field(() => Number)
  @PrimaryGeneratedColumn({ type: 'int' })
  chatId!: number;

  @Field(() => ChatRoom)
  @JoinColumn({ name: 'chatRoomId' })
  @ManyToOne(type => ChatRoom, chatRoom => chatRoom.chat, { eager: true })
  chatRoom!: ChatRoom;

  @Field(() => User)
  @JoinColumn({ name: 'writer' })
  @ManyToOne(type => User, user => user.chat, { eager: true })
  user!: User;

  @Field()
  @Column({ type: 'text' })
  chatting!: string;

  @Field()
  @CreateDateColumn({ type: 'datetime' })
  createdAt!: Date;

  @Field()
  @Column({ default: false })
  isConfirmed!: Boolean;

  @Field()
  @Column({ default: false })
  reportHandling!: boolean;

  @OneToMany(type => ChatComplaints, chatComplaints => chatComplaints.chat, { eager: false })
  chatComplaints!: ChatComplaints[];
}
```

### chatRoom.entity.ts

```tsx
import { Field, ObjectType } from '@nestjs/graphql';
import { BaseEntity, Column, CreateDateColumn, Entity, JoinColumn, ManyToOne, OneToMany, PrimaryGeneratedColumn } from 'typeorm';
import { Post } from 'src/posts/entities/post.entity';
import { User } from 'src/users/entities/user.entity';
import { Chat } from './chat.entity';

@Entity()
@ObjectType()
export class ChatRoom extends BaseEntity {
  @Field(() => Number)
  @PrimaryGeneratedColumn({ type: 'int' })
  chatRoomId!: number;

  @Field()
  @CreateDateColumn({ type: 'datetime' })
  createdAt!: Date;

  @Field()
  @Column({ default: false })
  sellerLeft!: boolean;

  @Field()
  @Column({ default: false })
  senderLeft!: boolean;

  @Field(() => Post)
  @JoinColumn({ name: 'postId' })
  @ManyToOne(type => Post, post => post.chatRoom, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  post!: Post;

  @Field(() => User)
  @JoinColumn({ name: 'userPhoneNumber' })
  @ManyToOne(type => User, user => user.chatRoom, { eager: true, onUpdate: 'CASCADE', onDelete: 'CASCADE' })
  user!: User;

  @OneToMany(type => Chat, chat => chat.chatRoom, { eager: false })
  chat!: Chat[];
}
```

## 👩🏻‍💻 구현 방법

> Nest에서 `Socket`을 이용해 채팅 서버를 구현할 예정이다.
> 

🧨 **찾아보니 Websocket을 이용하면 cors 에러가 자주 나고 시도때도 없이 에러가 난다고는 하는데 실시간 채팅이 가능한 방법은 소켓을 사용하는 것 밖에 없는 것 같아서 우선 사용해보기로 했다.** 

### WebSocket이란?

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

#### 설치

```tsx
**$ npm i --save @nestjs/websockets @nestjs/platform-socket.io**
```

#### Gateway

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

#### Exception Filter

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

### WebSocket과 Socket.io

<aside>
✅ WebSocket의 양방향 통신을 위한 프로토콜이다.

</aside>

<aside>
✅ Socket.io는 양방향 통신을 하기 위해 웹소캣 기술을 활용하는 라이브러리이다.

</aside>

#### WebSocket과 Socket.io

**WebSocket**

- HTML5 웹 표준 기술
- 매우 빠르게 동작
- 통신시 매우 적은 데이터 이용
- 이벤트를 단순히 듣고, 보내는 것만 가능

**Socket.io**

- 표준 기술 x (라이브러리)
- 소켓 연결 실패시 `fallback`을 통해 다른 방식으로 알아서 해당 클라이언트와 연결 시도
    - fallback: 시스템에 문제가 발생하였을 때 장애를 겪는 시스템을 특정 컴퓨터나 수동적인 기능으로 대치하여 고장 원인을 제거한 후 시스템을 완전히 가동할 수 있는 상태로 회복시키는 일
- 방(room)개념을 이용해 일부 클라이언트에게만 데이터를 전송하는 브로드캐스팅 가능

> WebSocket을 이용하면 브라우저와 서버간 연결을 유지한채로 양방향 통신을 구현할 수 있다.
> 

이에 따라, 채팅 기능을 ws 프로토콜을 기반으로 구축된 [socket.io](http://socket.io) 라이브러리를 이용하여 구현할 예정이다.

### 📝 기능 로직 및 수도 코드 작성

#### 💡 상세 기능 정리

1. 작성자 외 모든 사용자들은 구매희망자로서 채팅 시작 가능 (with 채팅방 생성)
2. 서로에게 실시간 채팅 가능 
3. 채팅방 내 대화들 조회 가능
4. 특정 게시글에 해당하는 채팅 목록 조회 가능

#### 🚩 DB

1️⃣ **채팅방 테이블**

채팅방 ID[PK], 게시글 ID[FK], 메시지 발송자 (구매희망자)[FK], 생성일자, 판매자 삭제, 발송자 삭제

2️⃣ **채팅 테이블**

채팅방 ID[PK], 생성일자, 작성자[FK], 채팅글, 읽음여부, 신고처리 

#### 🖐🏻 참고 사항

> WebSocketGateway 데코레이터의 옵션에서 namespace라는 옵션을 통해
> 

### 🚀 기능 구현

<aside>
✅ 백엔드 부분만 구현하다보니 소켓으로 채팅을 구현하는 데에는 한계가 있어 우선 소켓을 제외하고 graphQL만을 이용해 구현해봤다.

</aside>

#### 채팅방 만들기

> 당근 마켓 특성상 특정 게시글에 대해서 채팅방을 만들 수 있기 때문에 ChatRoom이라는 엔티티를 생성할 때 Post 엔티티를 참조하도록 구현했다.
> 

```tsx
async createChatRoom(user: User, createChatRoomDto: CreateChatRoomDto) {
  const { post } = createChatRoomDto;
  const query = await getRepository(ChatRoom).createQueryBuilder('ChatRoom').insert().into(ChatRoom).values({ post, user }).execute();
  return query.raw.insertId;
}
```

```tsx
async createChatRoom(user: User, createChatRoomDto: CreateChatRoomDto) {
  const { post } = createChatRoomDto;
  const foundPost = await Post.findOne(post);
  if (foundPost.user.userName === user.userName) {
    throw new BadRequestException('본인이 작성한 게시글에서 채팅방을 만들 수 없습니다.');
  }
  const chatRoom = await ChatRoom.findOne({
    where: {
      post,
      user,
    },
  });
  if (!chatRoom) {
    const insertId = await this.chatRepository.createChatRoom(user, createChatRoomDto);
    return await this.getChatRoomById(insertId);
  } else {
    throw new InternalServerErrorException('해당 채팅방은 이미 존재합니다.');
  }
}
```

#### 채팅 생성

> 채팅 신고 기능 구현한 내용을 후에 기록을 계획인데, 그 때 나오는 BlockUser 엔티티, 즉 특정 유저 차단 목록에 포함되는 유저는 채팅이 불가능도록 구현했다.
> 

```tsx
async createChat(user: User, createChatDto: CreateChatDto) {
  const { chatRoom, chatting } = createChatDto;
  const query = await getRepository(Chat).createQueryBuilder('Chat').insert().into(Chat).values({ chatRoom, user, chatting }).execute();
  return query.raw.insertId;
}
```

```tsx
async createChat(user: User, createChatDto: CreateChatDto): Promise<Chat> {
  // 현재 로그인한 유저 - 채팅하려는 상대방 유저 조합이 blockUser에서 targetUser - user에서 발견되면 채팅 불가
  const blockUser = await getRepository(BlockUser).findOne({
    where: {
      user,
    },
  });
  const { chatRoom } = createChatDto;
  const foundChatRoom = await ChatRoom.findOne(chatRoom);
  if (foundChatRoom.user.userName === user.userName || user.userName === foundChatRoom.post.user.userName) {
    const insertId = await this.chatRepository.createChat(user, createChatDto);
    return await Chat.findOne(insertId);
  } else if (foundChatRoom.user.userName === blockUser.user.userName && user.userName === blockUser.targetUser.userName) {
    throw new BadRequestException('로그인한 유저를 차단한 유저와는 채팅할 수 없습니다.');
  } else if (foundChatRoom.user.userName === blockUser.targetUser.userName && user.userName === blockUser.user.userName) {
    throw new BadRequestException('차단한 유저와는 채팅할 수 없습니다.');
  } else {
    throw new BadRequestException('해당 게시글 작성자와 구매희망자 외에는 채팅에 참여할 수 없습니다.');
  }
}
```