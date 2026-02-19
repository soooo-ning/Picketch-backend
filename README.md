# Picketch

웹에서 즐길 수 있는 실시간 캐치마인드(드로잉 추리 게임) <br>
백엔드 서버 코드 리포지토리입니다.

<br>

![Picketch](https://github.com/user-attachments/assets/868799c6-b0c8-4db6-80d5-a5ebc23d6549)

👉 <a href="https://github.com/1roo/picketch-frontend">프론트 코드 리포 주소

> **Notice**
> 이 레포지토리는 팀프로젝트 소스코드 아카이브입니다.
> AWS EC2 배포 종료 및 DB 환경 미포함으로 현재 실행되지 않습니다.

<br>

## 🗓️ Project Period

2025.02.10 ~ 2025.02.28 (3주) | 백엔드 3명, 프론트 3명

<br>

## 💻 Tech Stack

<img src="https://img.shields.io/badge/javascript-F7DF1E?style=flat-square&logo=javascript&logoColor=white"/> &nbsp;
<img src="https://img.shields.io/badge/Node.js-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white"/> &nbsp;
<img src="https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white"/> &nbsp;
<img src="https://img.shields.io/badge/Socket.io-010101?style=flat-square&logo=socketdotio&logoColor=white"/> &nbsp;
<img src="https://img.shields.io/badge/mysql-4479A1?style=flat-square&logo=mysql&logoColor=white"/> &nbsp;
<img src="https://img.shields.io/badge/Redis-FF4438?style=flat-square&logo=redis&logoColor=white"/> &nbsp;
<img src="https://img.shields.io/badge/Sequelize-52B0E7?style=flat-square&logo=sequelize&logoColor=white"/>

<br>

| 분류             | 기술                                             |
| ---------------- | ---------------------------------------------- |
| **Backend**      | Node.js, Express, Sequelize ORM, JavaScript    |
| **Realtime**     | Socket.io (게임, 알림, DM, 친구 상태)              |
| **Auth**         | JWT, OAuth 2.0 (Kakao, Google, Naver)          |
| **Database**     | MySQL (AWS RDS), Redis (ioredis)               |
| **Deploy**       | AWS EC2, S3, PM2                               |
| **Docs**         | Swagger (swagger-ui-express)                   |
| **Etc**          | Axios, CORS, dotenv, Prettier                  |

<br>

## 📁 Project Structure

```
📂 Picketch-backend
├── app.js              # 서버 초기화 및 API 라우트 연결
├── config              # 설정 파일 (DB, Redis)
├── controllers         # 요청 처리 및 비즈니스 로직
│   ├── authController      # 인증 (OAuth, JWT)
│   ├── userController      # 사용자 프로필 관리
│   ├── FriendController    # 친구 관리
│   ├── GameRoomController  # 게임 방 관리
│   ├── gameController      # 게임 참가
│   └── RankingController   # 랭킹 조회
├── middleware          # 인증 미들웨어 (HTTP, Socket)
├── models              # Sequelize ORM 모델 정의
├── routes              # 라우터 정의
├── socket              # 소켓 통신 처리
│   ├── notificationHandler   # 실시간 알림
│   ├── friendStatusHandler   # 친구 상태 관리
│   ├── dmChat                # DM 채팅
│   └── game                  # 게임 소켓 핸들러
│       ├── gameSocket        # 게임 소켓 메인
│       ├── gameCanvas        # 드로잉 캔버스 동기화
│       ├── gameChat          # 게임 채팅 (정답 추리)
│       └── gameStore         # 게임 상태 저장소
├── utils               # 유틸 함수
│   ├── common              # 공통 응답 함수
│   ├── redisManager        # Redis 연결 매니저 (싱글톤)
│   └── redisUtils          # Redis 유틸 (압축, 캐싱, 정리)
├── docs                # 문서 (Swagger, Sequence Diagram)
└── init.sql            # DDL 및 초기 데이터
```

<br>

## 📃 Output

<details>
  <summary>ERD</summary>
  <img src="https://github.com/user-attachments/assets/ce591236-54d8-4db4-9aa8-6580d8781cd7" width="1500px;" alt="erd"/>
</details>

<details>
  <summary>Rest API</summary>
  <img src="https://github.com/user-attachments/assets/f03ea7a4-6029-473f-8642-5a43dd4ca27a" width="1500px;" alt="erd"/>
</details>

<details>
  <summary>Socket.IO API</summary>
  
  > **범례**
  > - `C → S` : Client → Server (클라이언트가 emit)
  > - `S → C` : Server → Client (서버가 emit)
  > - 담당 이벤트만 작성하였습니다.  
  
  ---
  
  ## 1. /notifications
  
  알림 네임스페이스 - 실시간 알림 수신 및 응답 처리
  
  ### 연결
  
  소켓 연결 시 `socket.userInfo.user_id`로 사용자를 식별하며, Redis에 소켓 정보를 저장합니다.
  
  ---
  
  ### C → S: `GET_NOTIFICATIONS`
  
  사용자의 알림 목록을 조회합니다.
  Redis 캐시가 유효하면 캐시 데이터를 반환하고, 그렇지 않으면 DB에서 최대 20개를 조회합니다.
  
  #### Payload
  
  | 필드     | 타입    | 필수 | 설명                         |
  | -------- | ------- | ---- | ---------------------------- |
  | `userId` | integer | O    | 조회할 사용자 ID (본인 확인용) |
  
  ---
  
  ### C → S: `NOTIFICATION_RES`
  
  알림에 대한 수락/거절 응답을 전송합니다.
  
  - 친구 요청 수락 → 친구 관계 API 호출, 요청자에게 수락 알림 전송
  - 친구 요청 거절 → 친구 관계 거절 API 호출
  - 게임 초대 수락 → `GAME_INVITE_ACCEPTED` 이벤트 발생
  
  #### Payload
  
  | 필드             | 타입    | 필수 | 설명                                       |
  | ---------------- | ------- | ---- | ------------------------------------------ |
  | `notificationId` | integer | O    | 응답할 알림 ID                              |
  | `response`       | string  | O    | `ACCEPTED` 또는 `REJECTED`                  |
  
  ---
  
  ### S → C: `NOTIFICATIONS_LIST`
  
  알림 목록 조회 응답입니다.
  
  #### Payload
  
  ```json
  {
    "type": "NOTIFICATIONS_LIST",
    "data": {
      "notifications": [Notification],
      "unreadCount": 5
    }
  }
  ```
  
  #### Notification 객체
  
  | 필드               | 타입      | 설명                                                                    |
  | ------------------ | --------- | ----------------------------------------------------------------------- |
  | `notificationId`   | string    | 알림 고유 ID                                                            |
  | `notificationType` | string    | `FRIEND_REQUEST` / `FRIEND_ACCEPT` / `GAME_INVITE` / `DM_RECEIVED`     |
  | `responseStatus`   | string?   | `PENDING` / `ACCEPTED` / `REJECTED`                                     |
  | `from`             | object    | `{ userId: integer, nickname: string }`                                 |
  | `content`          | string    | 알림 내용                                                               |
  | `roomId`           | integer?  | 관련 게임방 ID                                                          |
  | `linkUrl`          | string?   | 관련 링크 URL                                                           |
  | `requiresResponse` | boolean   | 응답 필요 여부                                                          |
  | `isRead`           | boolean   | 읽음 여부                                                               |
  | `createdAt`        | datetime  | 생성 시각                                                               |
  | `expiresAt`        | datetime? | 만료 시각                                                               |
  
  ---
  
  ### S → C: `NOTIFICATION`
  
  새 알림을 실시간으로 수신합니다. 소켓 미연결 시 다음 연결 시 목록에서 확인 가능합니다.
  
  #### Payload
  
  ```json
  {
    "type": "NOTIFICATION",
    "data": { ...Notification }
  }
  ```
  
  ---
  
  ### S → C: `NOTIFICATION_RES_SUCCESS`
  
  알림 응답 처리가 성공적으로 완료되었음을 알립니다.
  
  #### Payload
  
  ```json
  {
    "type": "NOTIFICATION_RES_SUCCESS",
    "data": {
      "notificationId": 42,
      "status": "ACCEPTED"
    }
  }
  ```
  
  ---
  
  ### S → C: `GAME_INVITE_ACCEPTED`
  
  게임 초대를 수락한 경우 게임방 정보를 반환합니다.
  
  #### Payload
  
  ```json
  {
    "type": "GAME_INVITE_ACCEPTED",
    "data": {
      "roomId": 10,
      "linkUrl": "/game/10"
    }
  }
  ```
  
  ---
  
  ### S → C: `ERROR`
  
  알림 처리 중 에러가 발생했을 때 전송됩니다.
  
  #### Payload
  
  ```json
  {
    "type": "ERROR",
    "error": {
      "code": "UNAUTHORIZED",
      "message": "알림을 볼 수 있는 권한이 없습니다",
      "notificationId": 42
    }
  }
  ```
  
  #### 에러 코드
  
  | 코드                     | 설명                     |
  | ------------------------ | ------------------------ |
  | `UNAUTHORIZED`           | 알림을 볼 수 있는 권한이 없음 |
  | `NOTIFICATION_NOT_FOUND` | 알림을 찾을 수 없음         |
  | `ALREADY_RESPONDED`      | 이미 응답한 알림            |
  | `EXPIRED_NOTIFICATION`   | 만료된 알림                |
  | `FRIEND_API_ERROR`       | 친구 요청 처리 중 오류      |
  | `SERVER_ERROR`           | 서버 내부 오류              |
  
  ---
  
  ## 2. /friendStatus
  
  친구 상태 네임스페이스 - 온라인/오프라인/인게임 상태 실시간 동기화
  
  ### 연결
  
  소켓 연결 시 자동으로 `ONLINE` 상태로 변경되며, 모든 친구의 상태가 개별 `friend_status` 이벤트로 전송됩니다.
  연결 종료 시 자동으로 `OFFLINE` 상태로 변경되어 친구들에게 브로드캐스트됩니다.
  
  ---
  
  ### C → S: `GAME_STATUS_CHANGE`
  
  사용자의 게임 참여 상태를 변경합니다. 변경된 상태는 자동으로 모든 친구에게 브로드캐스트됩니다.
  
  #### Payload
  
  | 필드         | 타입    | 필수 | 설명                                                       |
  | ------------ | ------- | ---- | ---------------------------------------------------------- |
  | `inGame`     | boolean | O    | 게임 참여 중 여부 (`true` → IN_GAME, `false` → ONLINE)      |
  | `gameRoomId` | integer | △    | 참여 중인 게임방 ID (`inGame`이 `true`인 경우)               |
  
  ---
  
  ### S → C: `friend_status`
  
  친구의 현재 상태 정보를 수신합니다.
  브로드캐스트는 `friend_status_for_{userId}` 이벤트명으로 전달됩니다.
  
  #### Payload
  
  | 필드         | 타입      | 설명                                    |
  | ------------ | --------- | --------------------------------------- |
  | `userId`     | integer   | 친구의 사용자 ID                         |
  | `status`     | string    | `ONLINE` / `OFFLINE` / `IN_GAME`        |
  | `isOnline`   | boolean   | 온라인 여부                              |
  | `lastSeen`   | datetime? | 마지막 접속 시각 (OFFLINE인 경우)         |
  | `gameRoomId` | integer?  | 참여 중인 게임방 ID (IN_GAME인 경우)      |
  
  ---
  
  ### S → C: `ERROR`
  
  친구 상태 관련 처리 중 에러가 발생했을 때 전송됩니다.
  
  #### Payload
  
  ```json
  {
    "type": "ERROR",
    "error": {
      "code": "SERVER_ERROR",
      "message": "서버 오류가 발생했습니다"
    }
  }
  ```
</details>

<br>

## 👨‍💻 My Role

### 담당 기능

<table border="1" cellspacing="0" cellpadding="8">
  <thead>
    <tr>
      <th>구분</th>
      <th>기능</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">인증 시스템</td>
      <td>JWT 기반 Access / Refresh Token 발급 및 검증</td>
    </tr>
    <tr>
      <td>OAuth 2.0 소셜 로그인 (Kakao, Google, Naver)</td>
    </tr>
    <tr>
      <td>Soft Delete 패턴을 활용한 회원 탈퇴 처리</td>
    </tr>
    <tr>
      <td rowspan="4">실시간 알림</td>
      <td>Socket.io 기반 실시간 알림 전송 시스템</td>
    </tr>
    <tr>
      <td>Redis 캐싱을 활용한 알림 목록 조회 최적화</td>
    </tr>
    <tr>
      <td>읽지 않은 알림 관리 (Redis Set 구조)</td>
    </tr>
    <tr>
      <td>최대 알림 초과 시 오래된 알림 자동 읽음 처리</td>
    </tr>
    <tr>
      <td rowspan="3">친구 상태 관리</td>
      <td>소켓 연결 기반 온라인/오프라인/게임 중 상태 동기화</td>
    </tr>
    <tr>
      <td>친구 목록 기반 선택적 상태 브로드캐스트</td>
    </tr>
    <tr>
      <td>Redis + DB 이중 기록으로 상태 정합성 유지</td>
    </tr>
    <tr>
      <td rowspan="3">Redis 최적화</td>
      <td>Redis 싱글톤 매니저 및 연결 관리 구현</td>
    </tr>
    <tr>
      <td>gzip 압축 기반 캐시 데이터 메모리 최적화</td>
    </tr>
    <tr>
      <td>메모리 임계값(50MB) 기반 자동 정리 스케줄러</td>
    </tr>
    <tr>
      <td rowspan="1">인프라</td>
      <td>AWS EC2 배포 및 서버 모니터링 (PM2)</td>
    </tr>
  </tbody>
</table>

<br>

### 주요 구현

<br>

<img src="https://github.com/user-attachments/assets/3fd46c63-8c6f-4248-986d-afe8490a3d68" width="1500px;" alt="flow chart"/>

### 1. JWT 인증 및 OAuth 2.0 소셜 로그인

> [authController.js](controllers/authController.js) · [authMiddleware.js](middleware/authMiddleware.js)

- Access Token(1h) / Refresh Token(14d) 이중 토큰 사용
- Kakao, Google, Naver 3개 OAuth 프로바이더별 인증 플로우 구현
- 신규 유저 자동 가입 및 임시 닉네임(`T_${timestamp}`) 발급으로 프로필 생성 분리
- `is_deleted` 변수 기반 탈퇴 계정 로그인 차단 처리

<details>
  <summary>Sequence Diagram - 로그인</summary>
  <img src="https://github.com/user-attachments/assets/421cb217-471f-44b4-a57a-f673920c2017" width="1500px;" alt="login-sequence"/>
</details>

<br>

### 2. 실시간 알림 시스템

> [notificationHandler.js](socket/notificationHandler.js) · [redisUtils.js](utils/redisUtils.js) · [소켓 이벤트 명세](docs/socket-api.md)

- Socket.io 네임스페이스 기반 알림 전송/조회/응답 처리
- Redis 캐싱(3분 TTL)으로 반복 DB 조회 최소화
- Redis Set 구조를 활용한 읽지 않은 알림 관리
- 최대 알림 개수(30개) 초과 시 오래된 알림 자동 읽음 처리로 메모리 최적화
- Redis Pipeline을 활용한 다중 명령 일괄 처리

<details>
  <summary>Sequence Diagram - 실시간 알림</summary>
  <img src="https://github.com/user-attachments/assets/46b8475d-0c0c-4a68-9f0d-00e733137c5c" width="1500px;" alt="notification-sequence"/>
</details>

<br>

### 3. 친구 상태 관리 시스템

> [friendStatusHandler.js](socket/friendStatusHandler.js) · [소켓 이벤트 명세](docs/socket-api.md)

- 소켓 연결/해제 이벤트 기반 ONLINE/OFFLINE/IN_GAME 상태 자동 전환
- Redis + DB 이중 기록(Dual-Write)으로 상태 데이터 정합성 유지
- 친구 목록 기반 선택적 브로드캐스트로 불필요한 네트워크 트래픽 제거

<details>
  <summary>Sequence Diagram - 친구 상태</summary>
  <img src="https://github.com/user-attachments/assets/571b9d5d-611a-4149-9e47-f03535ebef84" width="1500px;" alt="friend-status-sequence"/>
</details>

<br>

### 4. Redis 메모리 관리 시스템

> [redisManager.js](utils/redisManager.js) · [redisUtils.js](utils/redisUtils.js) · [redis.js](config/redis.js)

- 싱글톤 패턴 Redis 매니저로 연결 관리 및 지수 백오프 재연결 전략
- 512바이트 이상 데이터 자동 gzip 압축/해제 유틸리티
- 15분 주기 메모리 모니터링 및 50MB 임계값 초과 시 자동 정리
- 키 프리픽스 체계(`n:`, `s:`, `h:`, `u:`)로 데이터 분류 관리

<br>

## 🔍 Trouble Shooting

### 1. Redis 대용량 캐시 데이터 - 메모리 사용량 급증 문제

실시간 알림 시스템 운영 중 Redis 메모리 사용량이 지속적으로 증가하는 문제가 발생했습니다.

| 문제                | 원인                        | 상세                                                         |
| ------------------- | ---------------------- | ----------------------------------------------------------- |
| 메모리 사용량 급증  | 캐시 데이터 무제한 적재     | 알림, 소켓 정보 등 캐시 데이터가 TTL 없이 누적되어 메모리 초과                 |
| 응답 속도 저하      | 비압축 대용량 데이터 저장   | JSON 직렬화된 알림 데이터가 원본 크기 그대로 저장되어 I/O 병목 발생            |

<br>

**Solution**

> [redisUtils.js](utils/redisUtils.js) - `compressData`, `performRedisCleanup`

512바이트 이상 데이터에 **gzip 자동 압축**을 적용하고, 15분 주기 **메모리 정리 스케줄러**를 도입:

```javascript
// gzip 압축: 512바이트 이상 데이터 자동 압축
async function compressData(data, threshold = 512) {
  const stringData = typeof data === "string" ? data : JSON.stringify(data);

  if (stringData.length < threshold) {
    return { data: stringData, compressed: false };
  }

  const compressed = await gzipAsync(Buffer.from(stringData), { level: 6 });
  return { data: compressed.toString("base64"), compressed: true };
}

// Redis 저장: 압축 + 메타데이터 + TTL을 Pipeline으로 일괄 처리
async function saveToRedis(redis, key, data, expireTime = 180) {
  const { data: compressedData, compressed } = await compressData(data);
  const pipeline = redis.pipeline();

  pipeline.set(key, compressedData);
  if (compressed) {
    pipeline.set(`${key}:meta`, "compressed");
    pipeline.expire(`${key}:meta`, expireTime);
  }
  pipeline.expire(key, expireTime);

  await pipeline.exec();
}
```

<br>

### 2. OAuth 소셜 로그인 - 프로바이더별 인증 플로우 불일치

3개 소셜 로그인(Kakao, Google, Naver) 구현 시 프로바이더마다 인증 방식이 달라 통합 처리에 어려움이 있었습니다.

| 문제                   | 원인                           | 상세                                                         |
| ---------------------- | ------------------------------ | -------------------------------------------------------------- |
| 인증 플로우 불일치     | 프로바이더별 상이한 OAuth 스펙 | Kakao/Naver는 Authorization Code, Google은 Access Token 직접 전달 방식        |
| 응답 데이터 구조 차이  | 사용자 정보 API 응답 형식 상이 | Kakao는 `id`, Google은 `sub`, Naver는 `response.id`로 사용자 식별값 위치 상이 |

<br>

**Solution**

> [authController.js](controllers/authController.js) - `kakaoLogin`, `googleLogin`, `naverLogin`

프로바이더별 **개별 컨트롤러 메서드**로 분리하되, 이후 로직(유저 조회/생성, 토큰 발급, 응답)은 **동일한 패턴**으로 통일:

```javascript
// Kakao: Authorization Code → Token 교환 → 사용자 정보 조회
const tokenResponse = await axios.post("https://kauth.kakao.com/oauth/token", null, {
  params: { code, client_id, client_secret, redirect_uri, grant_type: "authorization_code" },
});
const kakaoId = (await axios.get("https://kapi.kakao.com/v2/user/me", {
  headers: { Authorization: `Bearer ${tokenResponse.data.access_token}` },
})).data.id.toString();

// Google: 클라이언트에서 Access Token 직접 전달 → 사용자 정보 조회
const googleId = (await axios.get("https://www.googleapis.com/oauth2/v3/userinfo", {
  headers: { Authorization: `Bearer ${accessToken}` },
})).data.sub;

// Naver: Authorization Code → Token 교환 → 사용자 정보 조회
const naverId = (await axios.get("https://openapi.naver.com/v1/nid/me", {
  headers: { Authorization: `Bearer ${tokenResponse.data.access_token}` },
})).data.response.id;

// 이후 공통 로직: 유저 조회/생성 → 토큰 발급 → 응답
let user = await User.findOne({ where: { social_id, social_type } });
if (!user) {
  user = await User.create({ social_id, social_type, nickname: `T_${Date.now()}` });
}
const tokens = generateTokens(user);
```

<br>

### 3. 실시간 친구 상태 동기화 - Redis와 DB 데이터 불일치

사용자의 접속 상태가 Redis와 DB 간에 불일치하는 문제가 발생했습니다.

| 문제              | 원인                           | 상세                                                          |
| ----------------- | ------------------------------ | --------------------------------------------------------- |
| 상태 불일치       | 단일 저장소만 업데이트         | Redis만 업데이트하고 DB 반영이 누락되어 서버 재시작 시 상태 초기화               |
| 오프라인 미반영   | disconnect 이벤트 미처리       | 소켓 연결 해제 시 상태 업데이트가 누락되어 영구 ONLINE 상태로 남음               |

<br>

**Solution**

> [friendStatusHandler.js](socket/friendStatusHandler.js) - `updateUserStatus`

Redis와 DB **동시 업데이트**를 적용하고, disconnect 이벤트에서 OFFLINE 전환 및 `last_seen` 기록을 보장:

```javascript
async updateUserStatus(userId, status, gameRoomId = null, isDisconnect = false) {
  // Redis 상태 업데이트
  const redisData = {
    status: status,
    ...(gameRoomId && { gameRoomId }),
    ...(isDisconnect && { lastSeen: new Date().toISOString() }),
  };
  await this.redis.hset(`status:${userId}`, redisData);

  // DB 상태 업데이트 (동기화)
  const dbData = {
    status: status,
    ...(isDisconnect && { last_seen: new Date() }),
  };
  await User.update(dbData, { where: { user_id: userId } });
}

// disconnect 이벤트에서 OFFLINE 전환 보장
socket.on("disconnect", async () => {
  await this.updateUserStatus(userId, "OFFLINE", null, true);
  await this.broadcastStatusToAllFriends(userId, "OFFLINE");
});
```

<br>

### 4. 실시간 알림 - 소켓 재연결 시 알림 중복 응답

사용자가 친구 요청이나 게임 초대 알림에 응답할 때, 소켓 재연결 등의 상황에서 동일한 알림에 대해 여러 번 응답이 전송되는 문제가 발생했습니다.

| 문제              | 원인                           | 상세                                                                   |
| ----------------- | ------------------------------ | ------------------------------------------------------------------ |
| 알림 중복 응답    | 응답 상태 확인 로직 미흡       | 알림의 현재 `response_status`를 확인하지 않아 이미 처리된 알림에 재응답 가능     |
| 재연결 시 재전송  | 소켓 재연결 시 상태 미동기화   | 연결이 끊기고 재연결될 때 클라이언트에서 동일한 응답을 다시 전송                 |

<br>

**Solution**

> [notificationHandler.js](socket/notificationHandler.js) - `NOTIFICATION_RES` 이벤트 핸들러

알림 응답 처리 전 **상태를 명확히 검증**하고, 이미 처리된 알림은 에러로 반환하도록 개선. Redis 캐시 무효화를 함께 적용하여 재연결 시에도 최신 상태를 DB에서 조회하도록 보장:

```javascript
// 이미 응답된 알림 차단
if (notification.response_status && notification.response_status !== "PENDING") {
  return this.emitError(socket, "ALREADY_RESPONDED", "이미 응답한 알림입니다", {
    notificationId: data.notificationId,
  });
}
// 만료된 알림 차단
if (notification.expires_at && new Date() > notification.expires_at) {
  return this.emitError(socket, "EXPIRED_NOTIFICATION", "만료된 알림입니다", {
    notificationId: data.notificationId,
  });
}
```

<br>

### 5. 친구 상태 브로드캐스트 - 비동기 처리 순서 문제

사용자의 온라인/오프라인 상태 변경이 친구 목록에 실시간으로 반영되지 않아 잘못된 상태 정보가 표시되는 문제가 발생했습니다.

| 문제                    | 원인                           | 상세                                                                             |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------- |
| 상태 브로드캐스트 누락  | 비동기 작업 순서 부적절        | 상태 업데이트 완료 전에 브로드캐스트가 실행되어 이전 상태가 전파됨               |
| 전파 실패 시 전체 중단  | 오류 격리 미적용               | 개별 친구에게 상태 전파 실패 시 전체 브로드캐스트 프로세스가 중단됨              |

<br>

**Solution**

> [friendStatusHandler.js](socket/friendStatusHandler.js) - `disconnect` 이벤트, `broadcastStatusToAllFriends`

상태 업데이트 → 브로드캐스트 순서를 **`await`로 명확히 보장**하고, 친구별 오류를 격리하여 일부 실패가 전체에 영향을 미치지 않도록 개선:

```javascript
socket.on("disconnect", async () => {
  // 1. 상태 업데이트 완료를 보장한 후
  await this.updateUserStatus(userId, "OFFLINE", null, true);
  // 2. 브로드캐스트 실행 (개별 친구별 오류 격리)
  await this.broadcastStatusToAllFriends(userId, "OFFLINE");
});
```

<br>

## 💡 What I Learned

소켓을 처음 써본 프로젝트였는데, REST API와는 결이 다른 부분이 많았습니다.

특히 **비동기 처리 순서**가 실시간 시스템에서 얼마나 중요한지 직접 겪으면서 배웠습니다. 단순히 로직이 동작하는 것과, 정확한 타이밍에 동작하는 것은 완전히 다른 문제였습니다. 상태 업데이트가 끝나기 전에 브로드캐스트가 나가면서 친구 목록에 잘못된 상태가 표시되는 버그를 잡으면서 이 점을 체감했습니다.

Redis도 단순 캐시 용도로만 생각했는데, 알림 시스템을 만들면서 Pipeline, Set 자료구조, TTL 전략, 압축까지 다양하게 활용해볼 수 있었습니다. 메모리가 계속 쌓이는 문제를 겪고 나서야 캐시에도 관리가 필요하다는 걸 알게 됐습니다.

아쉬운 점은 3주라는 짧은 기간 때문에 구조적인 부분(서비스 레이어 분리, 에러 핸들링 통일 등)을 충분히 챙기지 못한 것인데, 이 부분은 아래 Points to Improve에 정리했습니다.

| 영역                          | 배운 점                                                           |
| ----------------------------- | ---------------------------------------------------------------- |
| **실시간 통신 설계**           | Socket.io 네임스페이스 분리를 통한 도메인별 이벤트 관리 체계 구축            |
| **Redis 캐싱 전략**           | TTL 기반 캐시 무효화, gzip 압축, Pipeline 일괄 처리로 성능 최적화            |
| **메모리 관리**               | 메모리 임계값 모니터링 및 자동 정리 스케줄러를 통한 Redis 메모리 최적화       |
| **OAuth 2.0 통합**            | 다중 프로바이더 OAuth 인증 플로우 설계 및 JWT 이중 토큰 전략 적용            |
| **데이터 정합성**             | Redis-DB Dual-Write 패턴으로 분산 환경에서 상태 데이터 일관성 유지           |
| **공통 유틸 설계**            | 표준화된 API 응답 함수(`common.js`) 설계를 통한 일관된 응답 구조 구축        |

<br>

## 🛠️ Points to Improve

아쉬웠던 점 → 이후 개선 방향

| 문제점                              | 원인                             | 개선 방향                                       |
| ----------------------------------- | -------------------------------- | --------------------------------------------- |
| OAuth 로직 중복                     | 프로바이더별 컨트롤러 개별 구현  | 공통 OAuth 서비스 추상화로 코드 중복 제거                      |
| Redis-DB 동기화 실패 시 미처리      | 트랜잭션 미적용                  | 동기화 실패 시 보상 로직 또는 이벤트 소싱 패턴 도입        |
| 소켓 미들웨어 인증 방식 불일치      | HTTP는 JWT, 소켓은 query 파라미터 | 소켓 handshake에서도 JWT 토큰 기반 인증으로 통일               |
| 에러 핸들링 분산                    | 글로벌 에러 처리 미적용          | Express 글로벌 에러 핸들러 미들웨어 도입                       |
| Redis `KEYS` 명령 사용             | 정리 작업에서 패턴 스캔 필요     | O(N) 블로킹 명령인 `KEYS` 대신 커서 기반 `SCAN`으로 전환      |

