---
title : "[Spring] webSocket으로 채팅서버 구현하기 (1)"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
# 서론
새로 개발할 앱을 플러터 + 스프링부트로 개발하기로 마음먹고나서, 이번 앱에 가장 중요한 부분이 채팅기능이기 때문에 가장먼저 스프링부트로 채팅서버를 구현해보려고한다.


# WebSocket
만약 우리가 채팅이라던가 주식정보를 가져오는등의 실시간으로 서버에 계속 통신을 해야하는 경우를 생각해보자. 2초마다 GET 요청을 보내 http 통신을 요청 할수도 있겠지만,
이렇게 하면 매번 연결을 맺고 끊는 연결과정의 비용이 들어갈 뿐만아니라, 단방향 통신으로 반드시 클라이언트가 요청을 해야만 서버가 정보를 주기 때문에, 양방향 통신이 필요한 기능을 만들기에는 HTTP 통신은 불편한 부분이 있다. 그래서 이러한 비연결성과 단방향 통신의 불편함을 해소하기 위해 만든 통신규약이 바로 webSocket으로, webSocket은 한번 연결을 맺으면 연결을 해제하기 전까지 유지 되기 떄문에, 연결을 맺고 끊는 연결과정을 없앨 수 있을 뿐만아니라, 양방향 통신이 가능하기 때문에 클라이언트가 요청을 하지않아도 서버가 주어야할 데이터를 보내줄 수 있다. 뿐만아니라, HTTP요청에 비해 webSocket은 한번에 주고받는 데이터의 양이 HTTP에 비해 확연히 적기 때문에, 계속 데이터를 주고받아야 하는 서비스에서는 적절하지 않다.

그럼 스프링 부트에서 webSocket을 어떻게 사용해야하는지 알아보고, 이를 이용해서 간단한 채팅 서비스를 만들어보자.

## 의존성 추가
build.gradle 파일에 아래와 같이 스프링부트에서 webSocket을 사용하기 위한 의존성을 추가해준다.
```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-websocket'
}

```

## WebSocket 기본 설정
스프링 부트에서 webSocket을 활성화 하려면, WebSocketConfig 클래스를 만들어 클라이언트가 보내는 정보를 받아서 처리할 Handler를 만들어주고, 이를 연결할 웹소켓 주소를 설정해 주어야한다.

### WebSocketConfig
```java
@RequiredArgsConstructor
@Configuration
@EnableWebSocket
public class WebSockConfig implements WebSocketConfigurer {
    private final WebSocketHandler webSocketHandler;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(webSocketHandler, "ws/chat").setAllowedOrigins("*");
    }
}
```
@EnableWebSocket을 통해 webSocket을 활성화 해주고, WebSocektConfigurer 인터페이스를 적용하여 WebSocketConfig라는 클래스를 만들어 이를 설정해준다. 
우리가 정보를 처리할 Handler와 webSocket 주소를 WebSocketHandlerRegistry에 추가해주면, 해당 주소로 접근하면 웹소켓 연결을 할 수가 있다. 

아직 Handler를 만들지 않아 에러가 뜨는데, 그럼 여기서 정보를 받아 처리할 Handler를 만들어보자.

### WebSocketHandler
```java
@Slf4j
@RequiredArgsConstructor
@Component
public class WebSocketHandler extends TextWebSocketHandler {
    private final ObjectMapper objectMapper;
    private final ChatService chatService;

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        log.info("{}", payload);
        ChatMessage chatMessage = objectMapper.readValue(payload, ChatMessage.class);
        
        ChatRoom chatRoom = chatService.findRoomById(chatMessage.getRoomId());
        chatRoom.handlerActions(session, chatMessage, chatService);
    }
}
``` 
스프링은 Text 타입과 Binary타입의 핸들러를 지원하는데, 우리는 채팅 서비스를 만들 것 이므로 TextWebSocketHandler를 상속하여 WebSocketHandler를 만들어줄것이다.
우리가 메세지를 json형식을 통해서 웹소켓을 통해 서버로 보내면, Handler는 이를받아 ObjectMapper를 통해서 해당 json 데이터를 chatMessage.class에 맞게 파싱하여 ChatMessage 객체로 변환하고,
이 json 데이터에 들어있는 roomId를 이용해서 해당 채팅방을 찾아 handlerAction() 이라는 메서드를 실행시킬 것이다. 그러면 handlerAction() 메서드는 이 참여자가 현재 이미 채팅방에 접속된 상태인지, 아니면 이미 채팅에 참여해있는 상태인지를 판별하여, 만약 채팅방에 처음 참여하는거라면 session을 연결해줌과 동시에 메시지를 보낼것이고 아니라면 메시지를 해당 채팅방으로 보내게 될 것이다.

아직 ChatMessage와 ChatRoom, ChatService 클래스를 만들지 않아 에러가 뜨지만, 대략적인 웹소켓 구성을 다음과 같이 구상하고 하나씩 만들어보도록 하자.

## DTO 만들기
위의 WebSocketHandler에서 사용했던 ChatMessage와 ChatRoom 클래스를 만들어 json데이터로 받아온 정보를 통해 해당 채팅방으로 채팅 메시지를 보낼 수 있도록 해당 json에 맞는 DTO들을 만들어주자.
 
### ChatMessage
```java
@Getter
@Setter
public class ChatMessage {
    public enum MessageType{
        ENTER, TALK
    }

    private MessageType type;
    private String roomId;
    private String sender;
    private String message;
}
```
간단하게 사용자가 ENTER, 처음 채팅방에 들어오는 상태인지 TALK, 이미 session에 연결되어있고 채팅중인 상태인지를 파악하기 위해 ENUM 타입으로 Message 타입을 선언해주고, 이를 type으로 갖게 하도록한다. 그 이외에는 간단하게, 이 메시지를 보낼 채팅방 id인 roomId와, 보내는 사람의 닉네임인 sender, 메시지를 담는 변수 message를 만들어, json 데이터로 채팅 데이터가 들어오면 이를 ChatMessage로 변환해 줄 수 있도록하자.

### ChatRoom 
```java
@Getter
public class ChatRoom {
    private String roomId;
    private String name;
    private Set<WebSocketSession> sessions = new HashSet<>();

    @Builder
    public ChatRoom(String roomId, String name) {
        this.roomId = roomId;
        this.name = name;
    }

    public void handlerActions(WebSocketSession session, ChatMessage chatMessage, ChatService chatService) {
        if (chatMessage.getType().equals(ChatMessage.MessageType.ENTER)) {
            sessions.add(session);
            chatMessage.setMessage(chatMessage.getSender() + "님이 입장했습니다.");
        }
        sendMessage(chatMessage, chatService);

    }

    private <T> void sendMessage(T message, ChatService chatService) {
        sessions.parallelStream()
                .forEach(session -> chatService.sendMessage(session, message));
    }
}

```
ChatRoom은 roomId와 name, 그리고 session을 관리할 sessions를 갖는다. json 데이터를 받아 WebSocketHandler에서 해당 데이터에 담긴 roomId를 chatService를 통해서 조회 하여 해당 id의 채팅방을 찾아 json데이터에 담긴 메시지를 해당 채팅방으로 보내게된다. 이 기능을 담당하는 곳이 handlerActions 메서드로, 해당 roomId의 채팅방을 찾아 handlerActions로 메시지와 세션을 보내면 이 메시지의 상태가 ENTER 상태인지 TALK 상태인지 판별하여 만약 ENTER 상태라면 session을 연결한뒤에 해당 sender가 입장했다는 메시지를 해당 채팅방에 보내고, 만약 이미 연결된 TALK 상태라면 해당 메시지를 해당 채팅방에 입장해있는 모든 클라이언트들 (Websocket session)에게 메시지를 보낸다. 

그럼 이제 이러한 기능들을 처리할 ChatService와 ChatController를 만들어 본격적으로 클라이언트가 서버에 웹소켓을 열고 이를 통해 메시지를 주고받게 해줄수 있는 로직을 작성해보자.

## Service, Controller

### ChatService 
```java
@Slf4j
@RequiredArgsConstructor
@Service
public class ChatService {
    private final ObjectMapper objectMapper;
    private Map<String, ChatRoom> chatRooms;

    @PostConstruct
    private void init() {
        chatRooms = new LinkedHashMap<>();
    }

    public List<ChatRoom> findAllRoom() {
        return new ArrayList<>(chatRooms.values());
    }

    public ChatRoom findRoomById(String roomId) {
        return chatRooms.get(roomId);
    }

    public ChatRoom createRoom(String name) {
        String randomId = UUID.randomUUID().toString();
        ChatRoom chatRoom = ChatRoom.builder()
                .roomId(randomId)
                .name(name)
                .build();
        chatRooms.put(randomId, chatRoom);
        return chatRoom;
    }

    public <T> void sendMessage(WebSocketSession session, T message) {
        try{
            session.sendMessage(new TextMessage(objectMapper.writeValueAsString(message)));
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        }
    }
}
```
ChatService는 chatRooms를 클래스 변수로 갖는데, chatRooms는 RoomId를 key로 갖고 chatRoom을 value로 갖는 Map으로, createRoom() 메서드가 실행되면 새로 방이 만들어지고 chatRooms에 chatRoom이 추가된다. 방의 아이디는 UUID로 랜덤으로 생성하여 지정되며, roomId를 사용해 findRoomById() 메서드를 통해서 해당 채팅방을 불러올 수 있다. 

sendMessage() 메서드는 TALK 상태일 경우 실행되는 메서드로, 메시지를 해당 채팅방의 webSocket 세션에 보내는 메서드이다.

이제 이러한 로직들을 ChatController를 통해 Http요청을 받으면 채팅방을 만들고, roomId를 반환하게 만들어 해당 roomId를 통해서 채팅방에 메시지를 날려보자.

### ChatController
```java
@RequiredArgsConstructor
@RestController
@RequestMapping("/chat")
public class ChatController {
    private final ChatService chatService;

    @PostMapping
    public ChatRoom createRoom(@RequestBody String name) {
        return chatService.createRoom(name);
    }

    @GetMapping
    public List<ChatRoom> findAllRoom() {
        return chatService.findAllRoom();
    }
}
```
@RequestMapping 어노테이션을 통해서 "/chat" 주소로 Post요청이 들어오면 json 데이터에서 name값을 받아 해당 이름으로 된 채팅방을 생성하고, Get요청이 들어오면 현재 열려있는 모든 채팅방을 모두 조회 할 수 있게 해주었다. 그럼 이제 모두 완성 되었으니, 한번 "/chat" 주소로 Post 요청을 보내 방을 생성하고 webSocket 접속을 도와주는 테스트 클라이언트를 열어 채팅을 실행해보자.

## 테스트
먼저, Post요청을 보내면, 

![image1](/assets/images/tech/Java/chat1/image1.png)

다음과 같이 정상적으로 방을 생성하고 생성된 방의 roomId를 리턴해주는 걸 볼 수 있다.

그럼 이제 이 방으로 접속해서 웹소켓을 이용한 채팅을 진행해보자. 구글 크롬 확장 프로그램인 WebSocketTestClient를 검색해서 추가한뒤 열어준다.

우리가 webSocket을 사용하기 위한 주소로 지정했던 "ws/chat"을 붙여주고 Open 버튼을 눌러 webSocket에 연결한다. "ws://localhost:8080/ws/chat" webSocket 요청이므로 http가 아닌 ws가 들어가야 한다.

![image2](/assets/images/tech/Java/chat1/image2.png)

이제 그럼 Request 부분에 아래와 같은 json 데이터를 보내보자.

```
{
  "type":"ENTER",
  "roomId":"e4f57c65-0b0f-4972-b20c-4a024a0a4f81",
  "sender":"사용자1",
  "message":"asd"
}
```
여기서 roomId는 아까 Post 요청을 통해 반환받은, 방금 생성한 채팅방의 roomId를 넣어주면 된다. send를 눌러 해당 데이터를 보내면,

![image2](/assets/images/tech/Java/chat1/image3.png)

다음과 같이 올바르게 session이 연결되고 ENTER 요청이므로 sender가 입장했다는 message가 전송되는 것을 확인 할 수 있다. 그럼 올바르게 채팅 내용이 채팅방 참여자 모두에게 전해지고 있는 지 확인 하기 위해서 웹페이지를 하나 더 연뒤 ENTER 하여 세션을 연결하고 TALK 속성의 메시지를 보내보도록 하자. 동일하게 페이지 하나를 더 열어 사용자2 라는 sender를 채팅방에 참여시킨다.

![image2](/assets/images/tech/Java/chat1/image4.png)

다음과 같이 사용자2도 올바르게 채팅방에 입장한 것을 확인할 수 있으며, 

![image2](/assets/images/tech/Java/chat1/image5.png)

사용자 1도 같은 세션에 접속해 있으므로 동일하게 참가했다는 메시지를 확인할 수 있다.

그럼 이제 사용자1이 TALK 메시지를 보내보면,

![image2](/assets/images/tech/Java/chat1/image6.png)

다음과 같이 올바르게 메시지가 송신되고,

![image2](/assets/images/tech/Java/chat1/image6.png)

마찬가지로 같은 세션에 연결되어있는 사용자2에게도 동일한 메시지가 보이는 것을 확인 할 수 있다.
