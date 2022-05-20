---
title:  "비대면 바이올린 강의를 위한 서비스 구축기"
subtitle: "언텍트 교육 프로그램"
tags:
- Python
- Go
- React
- WebRTC
- WebSocket
- Media-soup
- coturn
- HLS
- DASH
- violin
- zoom
date: 2021-11-07
image: https://user-images.githubusercontent.com/16316626/140653960-16ff206c-bb15-4bf7-b328-0b8453508ae3.png](https://user-images.githubusercontent.com/16316626/140653960-16ff206c-bb15-4bf7-b328-0b8453508ae3.png
draft: false
---
<div>
    <iframe width="560" height="315" src="https://www.youtube.com/embed/POBUwZJXXFM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

> 2021년 지란지교 패밀리데이의 `드림 미래 플랫폼` 주제에 아이디어 공모한 영상입니다.

# 0.목차
[1.발단](#1.발단)  
[2.목표설정](#2.목표설정)    
[3.진행과정](#3.진행과정)  
[4.서비스구조](#4.서비스구조)  
[5.서비스구조설명](#5.서비스구조설명)  
[6.구현](#6.구현)  
[7.문제점](#7.문제점)  
[8.회고](#8.회고)  
[9.앞으로의 계획](#9.앞으로의-계획)


# 1.발단
코로나 시대를 맞아 아부다비에 살고 있는 지인의 자녀 초등학생의 모든 수업이 온라인으로 전환되었다는 소식을 들었습니다.  
바이올린 수업도 온라인으로 시도하게 되었고, 좀더 좋은 선생님이 필요하다는 연락을 받게 되었습니다.

한국에서 바이올린 선생인 아내와 5시간 시차가 있는 아부다비에서 학생이 온라인에서 처음 만나게 되었습니다.

`Zoom`을 이용하였지만 `WebRTC`를 이용해 직접 구현해 보고 싶었습니다.

그 중간에 오픈소스인 `Jitsi`를 서버에 설치하였는데 서비스가 원활하지 않았습니다.

그래서 직접구현해 보고 싶은 마음이 생겼습니다.

온라인에서 `WebRTC Media Server`를 구현과 서비스를 하고 계신 분께 아래와 같은 키워드를 얻게 되었습니다.  
`WebRTC`, `mediasoup`,  `Coturn`, `HLS`,  `DASH`

# 2.목표설정

* 온라인에서 영상, 음성을 통한 비대면 렛슨이 가능한 서비스
* 자동 녹화기능 및 복습기능
* 예약된 시간에 자동으로 생성된 화상수업 ID와 링크로 생성
* 예습을 할 수 있게 미리 준비된 동영상 제공

# 3.진행과정
* 2020년 5월 6일: 첫 비대면 렛슨 시작
* 2021년 7월 11일: 첫 만남
* 2021년 7~8월: 한국에서 오프라인 3~4회 렛슨
* 2021년 8월 이후: 출국 부터 현재 (2021.11월) 온라인 수업은 진행 형

  ## 1) 2020년 5월 6일
  ![https://user-images.githubusercontent.com/16316626/140648140-1a684caa-1d18-4238-99a9-3298c5ecd660.png](https://user-images.githubusercontent.com/16316626/140648140-1a684caa-1d18-4238-99a9-3298c5ecd660.png)
  ### 이게 가능할까? (첫 비대면 음악 수업)
  비디오 카메라, 컨덴서 마이크, 비디오 스위쳐, 42인치 TV, 방송용 조명등 많은 장비를 셋팅했습니다.
  당분간은 학생 부모님과 저는 원활한 수업을 위해 수업 참관을 하면서 보조역할을 하기도 하였습니다.

  한동안 온라인 수업에 4명의 스케줄을 맞추어야 했는데
  시간이 지나자 선생님과 학생이 편한 시간에 보면대에 아이패드를 거치하는 것으로 거창한 셋팅이 간소화 되었습니다.

  ## 2) 2021년 7월 11일
  온라인으로만 매주 만났던 사제간의 만남이 실제 이루어졌습니다.
  1년만에 초등학생이였던 아내 제자가 중학생이 되었습니다.
  전체적인 자세와 보잉을 직접 만나서 수업이 진행되니 온라인보다 오프라인이 훨씬 양쪽간의 만족도가 높았습니다.

  하지만, 저희는 온라인에서 비대면으로 계속 렛슨을 진행해야 합니다.

  ## 3) 오프라인 이후 온라인으로
  2022년 여름을 기약하며, 지금까지 아내와 제자는 매주 `Zoom` 수업을 진행하고 있습니다.

# 4. 서비스구조 계획  
  ![https://user-images.githubusercontent.com/16316626/140653960-16ff206c-bb15-4bf7-b328-0b8453508ae3.png](https://user-images.githubusercontent.com/16316626/140653960-16ff206c-bb15-4bf7-b328-0b8453508ae3.png)
  ## 4-1.Front-End
  ### React
  최근에 사용해 보고 싶었던 `React.js`를 선택했습니다.    
  디바이스 카메라권한을 받고, websocket 프로토콜을 이용해서 클라이언트들이 소켓서버에 접속해서   
  영상과 음성을 주고 받는 방식 같습니다. (signaling(시그널링))  
  
  집 내부 망에서 테스트 할 때는 `Node.js`로 구현된것을 그대로 실행하면 소켓서버가 실행이되고 거기에 연결 해주는 작업을 해주었습니다.  
  ```javascript
    webSocketRef.current = new WebSocket(
                `ws://192.168.55.131:8000/join?roomID=${props.match.params.roomID}`
    );
  ```

  ### 메뉴  
  `Tutors`: 는  1:1 수업에 필요한 도구들을 제공   
  `Ensemble`: `Create Room` 버튼을 누르면 방이 생성되고 대시보드가 나오는 것을 그리고 싶었습니다.   
  `Commnity`: 커뮤니티 사이트로 이동합니다.  
  

  ## 4-2.Back-End
  ### Go + Fiber
  > [https://github.com/gofiber/fiber/blob/master/.github/README_ko.md](https://github.com/gofiber/fiber/blob/master/.github/README_ko.md)   
  > Fiber는 Express에서 영감을 받고, Go를 위한 가장 빠른 HTTP 엔진인 Fasthttp를 토대로 만들어진 웹 프레임워크 입니다.   
  > 비 메모리 할당과 성능을 고려한 빠른 개발을 위해 손쉽게 사용되도록 설계되었습니다.

  Go 언어와, Fiber 프레임워크를 선택했습니다. Gin과 고민을 하였는데,     
  최근 활발히 한국어와 함께 업데이트되고 있는 문서를 보고 `Fiber`로 결정했습니다.

  ## 4-3 Deploy
  ![스크린샷 2021-11-08 오후 4 58 34](https://user-images.githubusercontent.com/16316626/140704664-ec23e3bd-952e-4a51-9b32-c4539674769a.png)    
  **구성**: [https://sdk.xyz/2020/04/05/install-kubernetes/](https://sdk.xyz/2020/04/05/install-kubernetes/)
  홈서버에 구동중인 Kubernetes 에 올려보려 합니다.  
  현재 Node 5개로 구성되어있습니다.

  현재 홈서버에는 Go RestAPI만 docker로 실행중입니다.

  > Go 실행 방법
  > 1. nohup
  > 2. Pm go [https://github.com/struCoder/pmgo](https://github.com/struCoder/pmgo)
  > 3. docker
  > 4. K8S
  > 5. go Air

  * JWT 인증 : `go get -u github.com/form3tech-oss/jwt-go`
  * cors: `https://github.com/gofiber/fiber/blob/master/middleware/cors/README.md`
  * WASM(Web Assembly) 공부 시작 (현재 중단)
  > `C++`로 개발된 영상 채팅 프로그램을 Web에서 실행 하는 시도를 해보고 싶었음.  
  >  한빛 출판사에서 `Web Assembly in action` 도서를 지원 받았습니다.

  ## 4-4.기타
  ### WebSocket
  최종적으로 사용한것은 `Node.js`으로 구현되어있는것을 그대로 받아 헤로쿠에 실행 중입니다. 
  Go 언어일 경우 `Gorilla` (https://github.com/gorilla/websocket) 을 사용할 수 있었습니다.    
  인터넷 레퍼런스가 Go로 구현된것 보다 JavaScript 로 구현된 것이 훨씬 더 많았습니다.  

# 5.구현
![https://user-images.githubusercontent.com/16316626/140649989-72ff4399-b16f-416d-bcaf-7c7003c28187.png](https://user-images.githubusercontent.com/16316626/140649989-72ff4399-b16f-416d-bcaf-7c7003c28187.png)

  ## 5-1. JavaScript
  많은 레퍼런스들이 인터넷에 존재했습니다.
  공개된 소스와 유튜브강좌를 보고 작동되는 소스코드를 활용했습니다.

  ```aidl
  import React, { useEffect, useRef } from "react";
  import Header from "../common/Header";

  const Room = (props) => {
      const userVideo = useRef();
      const userStream = useRef();
      const partnerVideo = useRef();
      const peerRef = useRef();
      const webSocketRef = useRef();

      const openCamera = async () => {
          const allDevices = await navigator.mediaDevices.enumerateDevices();
          const cameras = allDevices.filter(
              (device) => device.kind === "videoinput"
          );
          console.log(cameras);

          const constraints = {
              audio: true,
              video: {
                  deviceId: cameras[1].deviceId,
              },
          };

          try {
              return await navigator.mediaDevices.getUserMedia(constraints);
          } catch (err) {
              console.log(err);
          }
      };

      useEffect(() => {
          openCamera().then((stream) => {
              userVideo.current.srcObject = stream;
              userStream.current = stream;

              webSocketRef.current = new WebSocket(
                  `ws://192.168.55.131:8000/join?roomID=${props.match.params.roomID}`
              );

              webSocketRef.current.addEventListener("open", () => {
                  webSocketRef.current.send(JSON.stringify({ join: true }));
              });

              webSocketRef.current.addEventListener("message", async (e) => {
                  const message = JSON.parse(e.data);

                  if (message.join) {
                      callUser();
                  }

                  if (message.offer) {
                      handleOffer(message.offer);
                  }

                  if (message.answer) {
                      console.log("Receiving Answer");
                      peerRef.current.setRemoteDescription(
                          new RTCSessionDescription(message.answer)
                      );
                  }

                  if (message.iceCandidate) {
                      console.log("Receiving and Adding ICE Candidate");
                      try {
                          await peerRef.current.addIceCandidate(
                              message.iceCandidate
                          );
                      } catch (err) {
                          console.log("Error Receiving ICE Candidate", err);
                      }
                  }
              });
          });
      });

      const handleOffer = async (offer) => {
          console.log("Received Offer, Creating Answer");
          peerRef.current = createPeer();

          await peerRef.current.setRemoteDescription(
              new RTCSessionDescription(offer)
          );

          userStream.current.getTracks().forEach((track) => {
              peerRef.current.addTrack(track, userStream.current);
          });

          const answer = await peerRef.current.createAnswer();
          await peerRef.current.setLocalDescription(answer);

          webSocketRef.current.send(
              JSON.stringify({ answer: peerRef.current.localDescription })
          );
      };

      const callUser = () => {
          console.log("Calling Other User");
          peerRef.current = createPeer();

          userStream.current.getTracks().forEach((track) => {
              peerRef.current.addTrack(track, userStream.current);
          });
      };

      const createPeer = () => {
          console.log("Creating Peer Connection");
          const peer = new RTCPeerConnection({
              iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
          });

          peer.onnegotiationneeded = handleNegotiationNeeded;
          peer.onicecandidate = handleIceCandidateEvent;
          peer.ontrack = handleTrackEvent;

          return peer;
      };

      const handleNegotiationNeeded = async () => {
          console.log("Creating Offer");

          try {
              const myOffer = await peerRef.current.createOffer();
              await peerRef.current.setLocalDescription(myOffer);

              webSocketRef.current.send(
                  JSON.stringify({ offer: peerRef.current.localDescription })
              );
          } catch (err) {}
      };

      const handleIceCandidateEvent = (e) => {
          console.log("Found Ice Candidate");
          if (e.candidate) {
              console.log(e.candidate);
              webSocketRef.current.send(
                  JSON.stringify({ iceCandidate: e.candidate })
              );
          }
      };

      const handleTrackEvent = (e) => {
          console.log("Received Tracks");
          partnerVideo.current.srcObject = e.streams[0];
      };

      return (

          <div>
              <Header/>
              <video autoPlay controls={true} ref={userVideo}></video>
              <video autoPlay controls={true} ref={partnerVideo}></video>
          </div>
      );
  };

  export default Room;

  ```
  ## 5-2. Go + Fiber
  RestFul API를 염두하고 코딩을 시작했습니다.  
  생성된 방의 정보를 MongoDB에서 관리 하려고 생각했습니다.  
  
  ![https://user-images.githubusercontent.com/16316626/140649451-09db7049-eefa-4beb-a84f-b409646f8e12.png](https://user-images.githubusercontent.com/16316626/140649451-09db7049-eefa-4beb-a84f-b409646f8e12.png)  
  Fiber를 사용하면 controller / modules / routes / config 로 분리하고 구조적으로 설계가 가능해 보였습니다.  
  
  

  ```aidl
  package controllers

  import (
      "encoding/json"
      "github.com/gofiber/fiber/v2"
      "github.com/gorilla/websocket"
      //"github.com/valyala/fasthttp"
      "log"
      "net/http"
  )

  // AllRooms is the global hashmap for the server
  var AllRooms RoomMap

  // CreateRoomRequest Create a Room and return roomID
  func CreateRoomRequest(c *fiber.Ctx) error {

      roomID := AllRooms.CreateRoom()

      type resp struct {
          RoomID string `json:"room_id"`
      }

      json.NewEncoder(c).Encode(resp{RoomID: roomID})
      return c.Status(fiber.StatusOK).JSON(fiber.Map{
          "success": true,
          "data": fiber.Map{
              "RoomID": roomID,
          },
      })
  }

  var upgrader = websocket.Upgrader{
      CheckOrigin: func(r *http.Request) bool {
          return true
      },
  }

  type broadcastMsg struct {
      Message map[string]interface{}
      RoomID  string
      Client  *websocket.Conn
  }

  var broadcast = make(chan broadcastMsg)

  func broadcaster() {
      for {
          msg := <-broadcast

          for _, client := range AllRooms.Map[msg.RoomID] {
              if client.Conn != msg.Client {
                  err := client.Conn.WriteJSON(msg.Message)

                  if err != nil {
                      log.Fatal(err)
                      client.Conn.Close()
                  }
              }
          }
      }
  }

  // https://github.com/gofiber/websocket
  // JoinRoomRequest will join the client in a particular room

  func JoinRoomRequest(c *fiber.Ctx) error {

      roomID := c.Params("roomID")
      log.Print(roomID)

      //roomID, ok := r.URL.Query()["roomID"]

      //if !ok {
      //	log.Println("roomID missing in URL Parameters")
      //	return
      //}
      //
      //ws, err := upgrader.Upgrade(w, r, nil)
      //if err !=  {
      //	log.Fatal("Web Socket Upgrade Error", err)
      //}
      //
      //AllRooms.InsertIntoRoom(roomID, false, ws)

      go broadcaster()

      for {
          var msg broadcastMsg

          //err := ws.ReadJSON(&msg.Message)
          //if err != nil {
          //	log.Fatal("Read Error: ", err)
          //}
          //
          //msg.Client = ws
          msg.RoomID = roomID

          log.Println(msg.Message)

          broadcast <- msg
      }
  }

  ```

# 6.결과
**뮤즈아카데미** : [https://muse.sdk.xyz](https://muse.sdk.xyz) 에 서비스? 중입니다.
![https://user-images.githubusercontent.com/16316626/140645971-96ba3e1d-9b59-4e7e-8eec-579549db8823.png](https://user-images.githubusercontent.com/16316626/140645971-96ba3e1d-9b59-4e7e-8eec-579549db8823.png)

## Test 방법
1. Account Info 에 이름을 입력하고 COPY YOUR ID를 누르면 랜덤 아이디가 생성됩니다.
2. 다른 사람에게 랜덤아이디를 알려주면 `Make a call`로 요청을 할 수 있습니다.

![screencapture-muse-ac-2021-06-01-20_57_03 복사본](https://user-images.githubusercontent.com/16316626/140646254-0f20544f-2a90-4de4-ad42-da64e31ec012.png)


# 7.문제점
* 아부다비에서는 한국에서 영상과 음성이 나오지 않았습니다
* 서울과 대전에선 작동했으나, 물리적 거리 + 소리나 영상의 레이턴시가 너무 생기고 (동시에 불가) 소통 힘들었습니다.  
* WebSocket 프로토콜을 이미 개발된것을 이용 할 뿐이지, Low Level 또는 Core 단 개발은 꿈도 꾸지 못합니다.  
  ## 해결 방안
  * Web Assembly 로 이미 잘 짜여있는 C++ 프로그램을 포팅하는 방법이 있을 것입니다.  
  * WebRTC 프로토콜을 잘 활용하는 방법이 있을 것입니다.  
  * `Google Meet`, `Zoom`같은 상용서비스를 이용하는 방법이 있을 것입니다.  

# 8.회고
**한 것이 없다라고 생각했는데,,뭐라도 시도해본 흔적이 있어서...다행**  
머릿속으론 하고 싶은것이 있지만, 얼토 당토 않은 지식과 구현 자체가 안되는 프로젝트 였습니다.  
설령 기능을 만든다고 하더라도, 홈서버에서 실시간 영상처리 및 음성 서비스는 불가능 처럼 보였습니다.  

이 토이프로젝트는 `코로나 시대에 실제 진행되고 있는 아내의 수업`에서 모티브를 얻는         
아내를 사랑하는 마음에서 시작된 프로젝트이고, 
중동에서 초등학생이였던 제자는 어느덧 중학생이 되었고,    
제가 보기에도 많은 실력 향상이 있었습니다.  

비대면 교육 프로그램 구축을 제외 하면,
지금도 진행하고 있는 렛슨은 남들이 하지 못한 특별한 수업입니다.

지금도 아내의 수업은 진행되고 있으며,  
여러가지를 시도한 `뮤즈아카데미`는 실패하지 않았고 내가 알지 못하는 많은 것이 많다는 `배움`을 얻었습니다.

# 9.앞으로의 계획

Google MeetUp 혹을 Zoom 같은 완성도가 높은 것들을 활용하되  
공개된 API가 있다면 이를 연동하고 멘토와 멘티를 중간에서 연결해주는 서비스가 가능할 것으로 생각됩니다.     
제 부족한 기술로나마 도움이 될 수 있다면   
`따뜻한 마음` 을 가지고 `사회적 약자` 혹은 `아동`들에게 교육의 기회를 제공해 줄수 있도록  
개발에 대해서 좀더 공부하겠습니다. 

