# 숙련 주차 프로젝트
### 기술 스택
- Java 21
- Spring Boot 4.1.0
- Spring Data JPA (Hibernate)
- Spring WebSocket
- MySQL 8
- Redis 7
- Gradle

## 실행 방법

### 1. MySQL, Redis 컨테이너 실행

```bash
docker run -d --name craft-db -e MYSQL_ROOT_PASSWORD=12345678 -e MYSQL_DATABASE=craft_db -p 3307:3306 mysql:8
docker run -d --name game-redis -p 6379:6379 redis:7
```

컨테이너 상태 확인:
```bash
docker ps
```

### 2. application.properties 설정

`src/main/resources/application.properties`에 아래 환경 변수가 설정되어 있습니다.

```properties
spring.application.name=game-expert

spring.datasource.url=jdbc:mysql://localhost:3307/craft_db
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=12345678

spring.data.redis.host=localhost
spring.data.redis.port=6379

spring.jpa.hibernate.ddl-auto=update
```

### 3. 애플리케이션 실행

IntelliJ에서 `GameExpertApplication`을 실행하거나, 터미널에서:

```bash
./gradlew bootRun
```

서버가 정상적으로 뜨면 `http://localhost:8080`에서 게임 화면이 열립니다.

## 구현 내용

### 인프라 & 데이터베이스
- **Lv 1** — Docker로 MySQL/Redis 실행 및 Spring 환경 변수 설정
- **Lv 2** — `ChatMessage` 엔티티에 `@Table`/`@Index`로 복합 인덱스(`world_id`, `created_at`) 선언

### REST API
- **Lv 3** — 플레이어 등록 API (`POST /players`), 닉네임 검증(2~12자, 영문/숫자/밑줄) 및 중복 처리
- **Lv 4** — 월드 생성 API (`POST /worlds`), 동시 생성 제어 및 최대 월드 수(3개) 제한
- **Lv 5** — 채팅 저장 및 최근 채팅 조회 로직 (`ChatService`)
- **Lv 6** — 최근 채팅 조회 API (`GET /worlds/{worldId}/chats`)

### WebSocket 연결 & 세션 관리
- **Lv 7** — WebSocket 핸드셰이크 시 닉네임/월드 검증 및 세션 속성 저장 (`NicknameHandshakeInterceptor`)
- **Lv 8** — 핸드셰이크 인터셉터를 `WebSocketConfig`에 등록
- **Lv 9** — 월드별 WebSocket 세션 레지스트리 (`WorldSessionRegistry`)
- **Lv 10** — Redis Sorted Set을 이용한 접속 상태 관리 (`PresenceService`)

### 실시간 메시지 처리
- **Lv 11** — 메시지 라우팅 및 ping/pong 하트비트 처리 (`MessageRouter`, `PingWsHandler`)
- **Lv 12** — 플레이어 이동 요청 처리 (`MoveWsHandler`)
- **Lv 13** — 채팅 요청 처리 및 응답 구성 (`ChatWsHandler`, `ChatResponse`)
- **Lv 14** — 같은 월드 참여자에게 채팅 브로드캐스트 (`LocalChatSender`)
- **Lv 15** — 접속자 목록 조회 (`OnlineUsersWsHandler`)

### 동시성 & 페이지네이션 (도전 과제)
- **Lv 16** — JPA 낙관적 락(Optimistic Lock)을 이용한 동시 수정 충돌 방지 (`WorldTrialSite`)
- **Lv 17** — 커서 기반 채팅 과거 내역 페이지 조회 (`ChatHistoryService`, `GET /worlds/{worldId}/chats/history`)

## API 요약

| 메서드 | 경로 | 성공 코드 | 설명 |
|---|---|---|---|
| POST | `/players` | 201 | 플레이어 등록 |
| GET | `/worlds` | 200 | 월드 목록 조회 |
| POST | `/worlds` | 201 | 월드 생성 |
| GET | `/worlds/{worldId}/chats` | 200 | 최근 채팅 조회 |
| GET | `/worlds/{worldId}/chats/history` | 200 | 과거 채팅 커서 조회 |
| WS | `/ws/worlds/{worldId}` | 101 | WebSocket 연결 |
