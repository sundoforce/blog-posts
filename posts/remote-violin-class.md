---
layout: post
title:  "(초안)비대면 강의를 위한 서버 구축기"
subtitle: "언텍트 교육 프로그램"
author: "sdk(sun-do, kim)"
catalog: true
header-mask:  0.3
tags:
- Python
- Go
- React
- WebRTC
- WebSocket
- Mediasoup
- coturn
- HLS
- DASH

date: 2021-11-07
multilingual: flase
---

> 2021년 4월에 잠시 시도해본 Toy Project 입니다.  
> 2021년 지란지교패밀리데이 `드림 미래 플랫폼` 이라는 주제에 아이디어 입상한 작품입니다.

### 영상을 먼저 소개할게요
<div>
    <iframe width="560" height="315" src="https://www.youtube.com/embed/POBUwZJXXFM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

> 이 프로젝트를 위해서 생성한 MongoDB Cloud 사용료가 $461.3 청구 결제되었습니다.  
> 본선 진출시 상금이 있어서, 비용을 마련하고자 4시간 정도 시간을 들여 만든 영상입니다.  
> MongoDB 에서 탕감 받았지만, 과금 과정과 용서?받은 내용은 별도 글을 작성 예정입니다.

## 발단
아부다비 지인과 카톡을 하다가, 코로나로 인해 아들의 바이올린 수업을 비대면으로 한다는 말을 들었습니다.    
그러면서 한국과 아부다비와 비대면 원격 렛슨을 구상하였습니다.

오프라인에서 알지도 못하는 두 사람이 온라인을 통해 첫 만남을 가졌습니다.
```aidl
{ 
  선생: 아내 (대한민국 거주),  
  제자: 지인 아들(중동 아부다비 거주),
  수업: 바이올린, 
  시차: 한국과 5시간 차이
}
```

### Time Line
* 2020년 5월 6일 첫 비대면 렛슨 시작
* 2021년 7월 11일 첫 만남 
* 2021년 7월 이후 제자의 출국 전까지 3회정도 오프라인 렛슨 
* 2021년 8월 부터 현재(2021년 11월 07일)까지 매주 렛슨 

### 시작은 거창 하였으나
첫 수업은 요란스러웠습니다.   
당시 초등학교 바이올린 수업을 위해서 고급 음향엔지니어 (저, 제자의 아버지)가  
컴퓨터에 온갖 영상, 음향장비를 연결해 약속된 시간 전에 준비를 하고,  
렛슨 동안에도 생중계 행사 오퍼레이팅 하듯이 서로 화면을 동시에 보면서

수업을 돕기 위해 기술지원 2명이 더 투입되었습니다.   
(아부다비 학부형, 한국 글쓴이)  

![https://user-images.githubusercontent.com/16316626/140648140-1a684caa-1d18-4238-99a9-3298c5ecd660.png](https://user-images.githubusercontent.com/16316626/140648140-1a684caa-1d18-4238-99a9-3298c5ecd660.png)

여러가지 시행 착오를 거쳐서, 현재는 일상이 된 듯 렛슨 하고  
어느 순간 부터 자연스럽게 덕지 덕지 붙었던 장비들과, 기술지원이 빠지고,   
보면대에 아이패드를 거치해서 Zoom으로 수업이 이루어졌습니다. 

## 뮤즈 아카데미 기획 
> 코로나시대 전부터 비대면 합주, 비대면 콘서트 , 비대면 연습등이 가능한 플랫폼을 만들고 싶었습니다.  
> 지금으로 부터 5년 전쯤 음악인 몇명에게 사용할 의향을 물었는데 아무도 사용하지 않다고 했는데
> 이미 방송국과 예술의 전당에선 비대면 음악회, 합주등이 시작되고 있었습니다. 

### 실행에 옮기자...
* 뮤즈아카데미 : [https://muse.ac](https://muse.ac)
`개발 임시 중단`  
  (사이트는 살아 있습니다.) 

**목표**
* 프로토콜(WebRTC):  `화상, 음성 채팅`을 이용한 `1:1 렛슨`
* `1:N 합주` 연습 
* 콘서트 
* 실시간 송출 및 녹화 

**2021년 5월 무작정 개발에 들어갔습니다.**
![https://user-images.githubusercontent.com/16316626/140645971-96ba3e1d-9b59-4e7e-8eec-579549db8823.png](https://user-images.githubusercontent.com/16316626/140645971-96ba3e1d-9b59-4e7e-8eec-579549db8823.png)
![screencapture-muse-ac-2021-06-01-20_57_03 복사본](https://user-images.githubusercontent.com/16316626/140646254-0f20544f-2a90-4de4-ad42-da64e31ec012.png)

### 홈서버 구성도 
![https://user-images.githubusercontent.com/16316626/140652847-4894dead-7948-46e5-a16e-415fb8cede23.png](https://user-images.githubusercontent.com/16316626/140652847-4894dead-7948-46e5-a16e-415fb8cede23.png)  
Zoom 으로 진행 하는 것보다 자체 서버가 있으면 좋겠다고 생각했습니다.    
* 무작정 홈서버에 Kubernetes 설치 했습니다. [https://sdk.xyz/2020/04/05/install-kubernetes/](https://sdk.xyz/2020/04/05/install-kubernetes/)
* 도메인 (https://muse.ac)을 확보 했습니다.
* 메모장에 구상을 하기 시작했습니다.  
* 관련된 기술들과 키워드를 조합으로 검색했습니다.
* Jitsi (설치형 화상 회의 프로그램)을 서버에 설치했습니다.  
  (설치기록: [https://sdk.xyz/2020/07/14/install-jitsi/](https://sdk.xyz/2020/07/14/install-jitsi/))
  
### 단순한 그림 
![https://user-images.githubusercontent.com/16316626/140653960-16ff206c-bb15-4bf7-b328-0b8453508ae3.png](https://user-images.githubusercontent.com/16316626/140653960-16ff206c-bb15-4bf7-b328-0b8453508ae3.png)
* WebRTC는 Go에 적용하긴 러닝커브가 있었습니다. -> JavaScript 사용 하였습니다. 
* `Room`, `Ensemble` , `Toutor` 메뉴를 선택하면 uuid 같은 방 ID가 생성되고, 그것을 DB에 저장합니다. 
  * 구현은 mongoDB connection 정도와, 방이 생성되었을때 방 아이디를 GET으로 넘겨주는 것까지 했던것 같습니다 
* 1:1 로 전화 걸로 받고 하는 것은 Youtube 20분짜리 영상을 보고 응용했습니다. 
* 헤로쿠에 ws에 돌고 있다 



### 기술 스텍 
* Front-End: `React.js`
* Back-End: `Go` (Fiber Framework)
* Web Socket: `Gorilla` (https://github.com/gorilla/websocket)
* WebRTC 
* MongoDB 

#### **Front-End**   

![x9791160508796](https://user-images.githubusercontent.com/16316626/140648843-a7663257-145e-4dbe-889f-a2f6c88dd7e1.jpeg)
  
2020년에 `리엑트를 다루는 기술`에 나오는 소스코드를 무작정 clone coding 한적이 있어서 소스코드가 github에 그대로 있었습니다.
React.js에 대해서 기초가 없어서 새로 만들기 보단 `있는것을 활용` 하는것이 빠른 진행이 될 것이라고 생각했습니다.   
하지만 예상대로 `React.js`를 잘 모르는 상태에서 사소한 것을 수정하는데도 많은 시간이 걸렸습니다.  
  
Front-end 와 Back-end로 분리하기 위해서 평소 궁금했던 Go를 API로 활용하고 싶어서 시도했습니다. 
![https://user-images.githubusercontent.com/16316626/140649989-72ff4399-b16f-416d-bcaf-7c7003c28187.png](https://user-images.githubusercontent.com/16316626/140649989-72ff4399-b16f-416d-bcaf-7c7003c28187.png) 
  
WebSocket의 예제는 JavaScript가 많이 있어서 그대로만 따라해도 구현이 됩니다.    
Front에서 ws 프로토콜로 통신하고, 새로 생성한 방 ID를 저장하기 위해서 mongoDB를 시도했습니다.


### WebRTC 구현 부분
**어디서 배껴온 소스코드**
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

* **Socket**     
WebSocket 프로토콜 서버가 한대 필요해서 `heroku` 에서 실행중입니다. 


* **Back-End**
    * API: https://musia.io (도메인 확부)
    * Go : Fiber   
        Go 프레임워크 중 후보가 Gin , Fiber로 줄여졌고    
        최종적으로 `Fiber`로 결정하였습니다 
    * MongoDB를 사용하고 싶어서 사용했습니다. 
  ![https://user-images.githubusercontent.com/16316626/140649451-09db7049-eefa-4beb-a84f-b409646f8e12.png](https://user-images.githubusercontent.com/16316626/140649451-09db7049-eefa-4beb-a84f-b409646f8e12.png)  
  
   Fiber 프레임웤에선 controller / modules / routes / config 로 분리하고 구조적으로 설계가 가능해 보였습니다.  
   한글 문서도 잘되어있고, 당시에 활발해 보였습니다. 

> [https://github.com/gofiber/fiber/blob/master/.github/README_ko.md](https://github.com/gofiber/fiber/blob/master/.github/README_ko.md)   
> Fiber는 Express에서 영감을 받고, Go를 위한 가장 빠른 HTTP 엔진인 Fasthttp를 토대로 만들어진 웹 프레임워크 입니다.   
> 비 메모리 할당과 성능을 고려한 빠른 개발을 위해 손쉽게 사용되도록 설계되었습니다.

**Go Lang 시도**
gorilla - websocket 
방아이디를 받아서 입장하게 처리하려고 시도했으나, 실패한 흔적이 보이지만,  
절대 공개하기 싫은 부끄러운 소스코드입니다.  
결국 JavaScript 단에서 처리 했던것으로 기억합니다. 
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
 
 
### 이외 작업들  
Back-end Go API `배포`는 `Docker`로 배포, react 는 `PM2` 로 Background 에서 실행되고 있습니다. 

홈서버에 설치된 Kubernetes 에 배포하기엔 러닝커브가 조금 있었습니다. 
`Ingress`를 이용해서 도메인도 연결해보고 로드벨런싱도 해보고 싶었지만, 
될듯 하면서 되지 않고, 속성으로 말고 나중에 제대로 공부를 해야 겠다고 생각했습니다. 

### Go Background 실행방법 
1. nohup
2. Pm go [https://github.com/struCoder/pmgo](https://github.com/struCoder/pmgo)
3. docker
4. K8S
5. go Air 

* JWT 인증 : `go get -u github.com/form3tech-oss/jwt-go`
* cors: `https://github.com/gofiber/fiber/blob/master/middleware/cors/README.md`
* WASM(Web Assembly) 공부 시작 (현재 중단)  
> `C++`로 개발된 영상 채팅 프로그램을 Web에서 실행 하는 시도를 해보고 싶었음.  
>  한빛 출판사에서 `Web Assembly in action` 도서를 지원 받았습니다.  

## 문제점
* 아부다비에서는 한국에서 영상과 음성이 나오지 않았습니다 
* 서울과 대전에선 작동했으나, 물리적 거리 + 소리나 영상의 레이턴시가 너무 생기고 (동시에 불가) 소통 힘들었습니다.  
* WebSocket 프로토콜을 이미 개발된것을 이용 할 뿐이지, Low Level 또는 Core단 개발은 꿈도 꾸지 못합니다.  

## 혼자 생각 한 점 
* 영상과 , 음성을 분리해서 나중에 합치는 방안을 생각.
* 개발 Skill 이 부족한 상태에서, 모든것을 하다보니 시간만가고 이도 저도 안됨.

## 회고 
머릿속으론 하고 싶은것이 있지만, 얼토 당토 않은 지식과 구현 자체가 안되는 프로젝트 였습니다.
설령 기능을 만든다고 하더라도, 홈서버에서 실시간 영상처리 및 음성 서비스는 불가능 처럼 보였습니다. 

그래서  
Google MeetUp 혹을 Zoom 같은 완성도가 높은 것들을 활용하되,    
시공간에 제약을 받지 않고, 원하는 선생님과 함께 수업을 하고,  

사회적 약자 혹은 아동에게 교육의 기회를 제공해 줄수 있는 플랫폼을 만들고 싶습니다. 

이 토이프로젝트는 `코로나 시대에 실제 진행되고 있는 아내의 수업`에서 모티브를 얻는       
아내를 사랑하는 마음에서 시작된 프로젝트이고, 
중동에서 초등학생이였던 제자는 어느덧 중학생이 되었고,  
제가 보기에도 많은 실력 향상이 있었습니다. 

비대면 교육 프로그램 구축을 제외 하면, 
지금도 진행하고 있는 렛슨은 남들이 하지 못한 특별한 수업입니다.  
  
지금도 아내의 수업은 진행되고 있으며, 
여러가지를 시도한  `뮤즈아카데미`는 실패하지 않았고, 
내가 개발자로 한걸을 더 `성장`했다고 믿고 싶은 프로젝트라고 생각합니다.  

## fork 했던 Repository
* https://github.com/versatica/mediasoup
* https://github.com/li-yanhao/janus-webrtc-react-native
* https://github.com/pion/webrtc

