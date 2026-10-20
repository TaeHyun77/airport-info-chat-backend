### 프로젝트
---
공공데이터 API를 통해 항공편 상태와 출국장 혼잡도 등이 정보를 수집하고, 저장 및 갱신하는 데이터 파이프라인 설계에 중점을 두었습니다. 이를 바탕으로 최신 공항 정보를 제공하고, 사용자 간 실시간 채팅과 공항 데이터를 활용한 AI 챗봇 기능을 구현했습니다.<br><br>

### 기술 스택
---
Backend : Java, Spring Boot, Spring Data JPA, Spring Security, Spring Mail

Frontend : React, JavaScript

Database : MySQL (AWS RDS)

Search : Elasticsearch

Infrastructure : Docker, AWS EC2

ECT : Claude API, WebSocket/STOMP<br><br>

### 아키텍처
---
<p align="center">
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/ceb79139-81cc-40b9-8332-5f984c89cfcd" /></p><br><br>

### 구현
---
- 공항 데이터 수집 파이프라인
    
    API를 주기적으로 호출하여 데이터를 수집하고 DB에 저장, 갱신하는 파이프라인을 구축하여, 사용자 요청 시 최신 수집 데이터를 기반으로 응답하도록 구현했습니다.
    
- 전용 스레드 풀 기반 비동기 알림 발송
    
    출국장 혼잡도와 관심 항공편의 상태 변경을 감지해 이메일로 알립니다. 수집 과정에서 메일을 동기로 보내면 SMTP 응답을 기다리는 동안 다음 수집이 늦어져 데이터의 최신성이 떨어지므로 알림 발송을 비동기로 분리했고, 다른 비동기 작업과 스레드를 나눠 쓰며 서로 영향을 주지 않도록 전용 스레드 풀에서 처리했습니다. 그 결과 메일 100건 기준 알림 완료 시간을 `30.6초에서 7.6초로 약 75%` 단축했습니다.
    
- WebSocket과 STOMP 기반 실시간 채팅
    
    채팅은 사용자가 메시지를 보내고 받는 양방향 통신이기에 WebSocket을 사용했습니다. 순수 WebSocket은 메시지를 어느 채팅방으로 보낼지 직접 구현해야 하므로, 목적지 기반 구독을 제공하는 STOMP로 채팅방별 메시지 라우팅을 처리했습니다. 
    
    또한, 사용자가 퇴장 버튼 없이 탭을 닫거나 네트워크가 끊기는 경우도 감지할 수 있도록, 세션 연결, 종료 이벤트로 접속자 수를 집계하여 실시간으로 브로드캐스트했습니다.
    
- Claude API 기반 공항 정보 AI 챗봇
    
    항공편, 혼잡도, 주차장 정보가 분산되어 있기 때문에 사용자가 필요한 정보를 한눈에 파악하기 어려웠습니다. 따라서, 질문을 통해 답을 얻을 수 있도록 AI 챗봇을 구현했습니다. 이때 수집한 최신 데이터를 함께 전달하여, 모델이 추측하지 않고 실제 데이터를 근거로 답하도록 했습니다.
    
- Elasticsearch 기반 항공편 검색
    
    DB의 `LIKE` 검색은 인덱스를 활용하지 못하고 오타나 형태소 단위 검색을 지원하지 않습니다. 사용자는 도시명이나 항공사명을 틀리거나 편명을 일부만 입력하는 경우가 많기 때문에, Elasticsearch에 Nori 형태소 분석기를 적용하고 Fuzzy 검색과 자동완성으로 오타와 부분 입력에도 결과를 찾을 수 있도록 했습니다.
    
- OAuth 2.0 소셜 로그인을 적용하고, Google OAuth는 localhost가 아닌 환경에서 IP 주소나 HTTP로 된 리다이렉트 URI를 허용하지 않아, 도메인을 연결하고 HTTPS 환경을 구성했습니다.<br><br>

### 트러블슈팅
---

- OSIV 비활성화에 따른 `LazyInitializationException` 문제 및 N+1 문제 해결
    
    [ 상황 ]
    
    OSIV가 켜져 있으면 영속성 컨텍스트와 DB 커넥션이 응답을 반환할 때까지 유지됩니다. 채팅처럼 요청이 잦은 기능에서 커넥션을 오래 점유하지 않도록 OSIV를 비활성화했습니다.
    
    [ 문제 ]
    
    채팅 목록 조회 시 트랜잭션이 끝난 뒤 발신자 정보에 접근하면서 `LazyInitializationException`이 발생했습니다. 이를 해결하기 위해 트랜잭션 안에서 발신자를 조회하도록 바꾸자, 메시지마다 발신자 조회 쿼리가 추가로 실행되는 N+1 문제가 발생했습니다.
    
    [ 해결 ]
    
    한 번의 쿼리로 연관 데이터를 조회하면서 조회 관계를 명확하게 드러낼 수 있는 Fetch Join을 사용하여 문제를 해결하고 1,000건 조회 시간을 `356ms에서 3ms로 단축`했습니다.
    
- 대량 항공편 데이터 INSERT 성능 개선
    
    [ 상황 ]
    
    10,000건의 항공편 데이터를 개별 INSERT로 등록하고 있었습니다.
    
    [ 문제 ]
    
    INSERT마다 DB와 한 번씩 통신하여 총 10,000회의 왕복이 발생했고, 등록에 약 1.1초가 걸렸습니다. 소요 시간의 대부분은 쿼리 실행보다 네트워크 왕복 비용이었습니다.
    
    [ 해결 ]
    
    DB 왕복 횟수를 줄이기 위해 JDBC Batch Insert를 적용했습니다. 하지만 IDENTITY 전략은 INSERT 실행 후에야 ID를 알 수 있어 배치 처리가 어렵습니다.
    
    이를 해결하기 위해 애플리케이션에서 ID를 미리 생성하도록 변경했습니다. UUID 대신 시간순으로 증가하는 64비트 정수인 Snowflake를 사용하여 인덱스 효율을 고려했습니다. 그 결과 100건씩 일괄 등록하여 DB 왕복 횟수를 10,000회에서 100회로 줄이고, 등록 시간을 `1,108ms에서 146ms로 약 87% 단축`했습니다.
    
- 제한된 서버 환경에서 React 서버 메모리 문제 개선
    
    [ 상황 ]
    
    1 Core, 1GB Memory의 제한된 EC2 환경에서 React 애플리케이션과 백엔드를 함께 운영하고 있었습니다.
    
    [ 문제 ]
    
    React 개발 서버의 상시 실행으로 메모리 부담이 커져 컨테이너 운영에 문제가 발생했으며, 스왑 메모리 추가만으로는 해결되지 않았습니다.
    
    [ 해결 ]
    
    React 애플리케이션을 정적 파일로 빌드하고 Nginx에서 서빙하도록 변경했습니다. 별도의 React 서버 실행을 제거해 메모리 사용 부담을 줄이고, 제한된 EC2 환경에서 안정적으로 서비스를 운영할 수 있도록 개선했습니다.<br><br>

### 페이지
---
[ 홈 페이지 ]

<img width="806" height="771" alt="image" src="https://github.com/user-attachments/assets/263edc54-954c-46d3-94c7-1e711fe4e3f8" /><br>

<img width="806" height="650" alt="image" src="https://github.com/user-attachments/assets/c3cbc364-c14c-4f62-a440-59df9d0a177a" /><br><br>

[ 마이페이지 ]

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/d2bfb7a7-c1a1-47b8-a3ae-7a00402c96ff" /><br>

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/e607485a-413b-435a-8508-f883e4bf1af3" /><br><br>

[ 부가 정보 ] - 체크인 카운터 ~ 공항 철도, 주차장 도보 거리

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/22b90656-add7-4b62-b693-9be9ceb10f89" /><br>

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/106e9471-cd00-4639-8d3c-db4e2ac989b3" /><br><br>

[ 실시간 주차장 정보 ]

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/28da129c-bcff-42a2-b731-df1522cd2f8d" /><br><br>

[ 채팅 ]

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/d6e586ed-e10c-49e7-8948-f19387e14d05" /><br>

<img width="806" height="888" alt="image" src="https://github.com/user-attachments/assets/a4ef3ad6-b48c-4f6a-9abf-9730a9ccba63" />
