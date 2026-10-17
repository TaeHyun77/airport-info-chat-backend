### 프로젝트
---
출국에 필요한 항공편 상태와 출국장 혼잡도 등의 공항 정보를 실시간으로 제공하고, 항공편 검색 및 항공편 상태 변동 알림 구독 기능과 사용자 간 소통을 위한 채팅 기능을 제공하는 통합 공항 정보 서비스입니다.<br><br>

### 기술 스택
---
- Backend : Spring Boot, Spring Data JPA, Java
  
- Frontend : React, JavaScript
  
- Database : MySQL (RDS)
  
- Messaging : Kafka
  
- Search : Elasticsearch
  
- Infrastructure : Docker, AWS EC2<br><br>

### 아키텍처
---
<p align="center">
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/29148d54-02d0-41c8-a443-93c3bf9a7ad0" /></p><br><br>

### 구현 및 기능
---
- 공항 관련 API를 주기적으로 호출해 정보를 수집하고 DB에 저장/갱신하는 데이터 수집 파이프라인을 구축했습니다.
  
- 출국장 혼잡도나 관심 항공편의 상태 변경을 감지해 조건을 만족하는 회원에게만 알림을 비동기로 발송하고, 별도 스레드에서 처리하여 정합성과 성능을 확보했습니다.
  
- OAuth 2.0 기반 소셜 로그인을 구현하고, Google OAuth 연동을 위해 도메인과 HTTPS 환경을 구성했습니다.
- Elasticsearch에 Nori 한글 형태소 분석기를 적용하고 Fuzzy 검색과 자동완성을 구현해, 오타와 부분 입력에도 대응하는 항공편 검색 기능을 구축했습니다.
- WebSocket과 STOMP를 활용해 실시간 채팅을 구현하고, 세션 이벤트로 채팅방의 접속/이탈을 추적하여 접속자 수를 실시간으로 브로드캐스트했습니다.<br><br>

### 트러블 슈팅 및 개선
---
- 대량 데이터 저장 시 성능 개선을 위해 Batch Insert/Update를 적용하였으며, 현재 프로젝트의 데이터 규모에서는 성능 향상이 크지 않았지만 향후 확장성을 고려하여 도입하였습니다.

- JPA의 Fetch Join을 적용하여 N+1 문제를 해결하고, 연관 관계 조회 성능을 약 63% 개선하였습니다.

- 제한된 EC2 환경(1 Core, 1GB Memory)에서 Docker 컨테이너 실행 시 React 서버의 메모리 사용량으로 인해 컨테이너가 정상적으로 실행되지 않는 문제 발생하였습니다.

  스왑 메모리(2GB)를 추가했으나 메모리 부족 문제가 지속되어, React 애플리케이션을 빌드하여 정적 파일로 변환하고 Nginx에서 서빙하도록 구조를 변경하여, 서버 메모리 사용량을 줄이고 안정적으로 서비스를 운영할 수 있도록 개선하였습니다.<br><br>

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
