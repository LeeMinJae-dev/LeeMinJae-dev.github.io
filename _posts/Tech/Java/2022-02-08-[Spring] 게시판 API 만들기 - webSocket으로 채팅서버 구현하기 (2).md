---
title : "[Spring] webSocket으로 채팅서버 구현하기 (2)"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
# 서론
저번 게시물에서는 WebSocekt을 통해서 간단한 서버/클라이언트 통신을 만들어보았다. 하지만 WebSocket만을 사용해서 통신을 하게되면, 통신구조가 무척 단순하기 때문에 WebSocket만을 이용하여 구현하면 해당 메시지가 어떤 요청이고, 어떻게 처리해야 하는지에 따라 채팅룸과 세션을 일일히 구현하고 메시지 발송을 처리하는 코드를 구현해주어야한다.

따라서 이러한 문제를 해결하기 위해 STOMP를 사용하여 조금 더 쉽게 이러한 과정들을 처리 할 수 있도록 할 것이다. STOMP는 Simple Text Oriented Messaging Protocol의 줄임말로, 메시지 브로커를 활용하여 쉽게 메시지를 주고 받을 수 있는 프로토콜이다. 

STOMP는 pub과 sub을 갖는데, pub을 통해 발행한 메시지를 sub을 통해 구독하면, 수신자는 발신자가 발행하는 메시지를 수신한다.

따라서 STOMP는 WebSocket위에 얹어 사용할 수 있는 하위 프로토콜로, 우리가 구현했던 WebSocket을 사용한 서버를 훨씬 편하게 구현할 수 있도록 도와준다.



