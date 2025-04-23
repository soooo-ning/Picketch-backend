# 🎨 Picketch

웹에서 즐길 수 있는 실시간 캐치마인드(드로잉 추리 게임) <br>
백엔드 서버 코드 리포지토리입니다.

> 👉 <a href="https://github.com/1roo/picketch-frontend">프론트 코드 리포 주소
> <br>
> 👉 http://43.200.254.250:8080/ ✨배포종료✨
> <br>

![main](https://github.com/user-attachments/assets/868799c6-b0c8-4db6-80d5-a5ebc23d6549)
<br><br>

## 🗓️ Project Period

2025.02.10 ~ 2025.02.28 (3주)
<br><br><br>


## 👨‍💻 Role & Contribution

#### - Backend -

- 사용자 인증 및 게임 상태 관리를 위한 JWT 기반 인증 시스템 구현
- OAuth2.0 API를 활용한 소셜 로그인 기능 구현
- 대용량 트래픽 처리를 위한 Redis 서버 구현 및 최적화
- Socket.io를 사용한 실시간 알림 시스템 및 친구 상태 관리 기능 개발
- AWS 기반 인프라 구축 및 배포 환경 설정
  <br><br>

## 🔧 Stack

<img src="https://img.shields.io/badge/Javascript-F7DF1E?style=flat-square&logo=javascript&logoColor=white"/><img src="https://img.shields.io/badge/Node.js-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white"/><img src="https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white"/><img src="https://img.shields.io/badge/Socket.io-010101?style=flat-square&logo=socketdotio&logoColor=white"/><img src="https://img.shields.io/badge/Mysql-4479A1?style=flat-square&logo=mysql&logoColor=white"/><img src="https://img.shields.io/badge/Redis-FF4438?style=flat-square&logo=redis&logoColor=white"/><img src="https://img.shields.io/badge/Sequelize-52B0E7?style=flat-square&logo=sequelize&logoColor=white"/>
<br>

- **Language** : JavaScript
- **Library & Framework** : Node.js express Socket.io
- **Database** : MySQL, Redis, AWS RDS
- **ORM** : Sequelize
- **Deploy** : AWS EC2, S3
  <br><br>

## 💻 Software Design

### API 명세서

<!-- <img src="https://github.com/user-attachments/assets/84988fc9-1079-4c65-97d4-3f3db161bcaf" width="800px;" alt=""/> -->
<br>

### ERD

<img src="https://github.com/user-attachments/assets/ce591236-54d8-4db4-9aa8-6580d8781cd7" width="1500px;" alt=""/>
<br><br>

### Sequence Diagram

- 로그인
  <img src="https://github.com/user-attachments/assets/421cb217-471f-44b4-a57a-f673920c2017" width="1500px;" alt=""/>
  <br>
- 실시간 알림
  <img src="https://github.com/user-attachments/assets/46b8475d-0c0c-4a68-9f0d-00e733137c5c" width="1500px;" alt=""/>
  <br>
- 친구 상태
  <img src="https://github.com/user-attachments/assets/571b9d5d-611a-4149-9e47-f03535ebef84" width="1500px;" alt=""/>
  <br>

## ⭐ Main Feature

### JWT 기반 인증 시스템 및 사용자 관리

- JWT 토큰 발급 및 검증
- Oauth2를 활용한 kakao, goole, naver 로그인
- 닉네임 중복 확인, 사용자 프로필 생성 및 수정 기능
- soft delete 패턴을 활용한 회원 탈퇴 처리
  <br><br>

<!-- <img src="https://github.com/user-attachments/assets/d98f6146-0512-454a-bb32-ce70d2569e66" width="1500px;"/><br><br> -->

### 실시간 알림

- 실시간 게임 초대 / DM 수신 / 친구 추가 알림 기능
- 알림 목록 캐싱을 통한 반복 DB 조회 최소화
- Redis Set 구조를 활용한 읽지 않은 알림 효율적 관리
- 최대 알림 개수 초과 시 오래된 알림 자동 읽음 처리로 메모리 최적화
  <br><br>

<!-- <img src="https://github.com/user-attachments/assets/f3485493-ffb1-41a9-a2ae-6e20a680aa5f" width="1500px;"/><br><br>
<img src="https://github.com/user-attachments/assets/6391c13e-b60d-4e71-89de-e185f2541cb9" width="1500px;"/><br><br> -->

### 사용자 및 친구 상태 관리

- 사용자별 소켓 ID 및 연결 상태 Redis에 저장 및 실시간 업데이트
- 연결 해제 시 오프라인 상태 기록 및 마지막 접속 시간 추적
- 친구 접속 상태 관리 및 동기화
- 친구 목록 기반 선택적 상태 브로드캐스트
  <br><br>

<!-- <img src="https://github.com/user-attachments/assets/d6c64334-ac4c-4d7f-ae83-06c15de0600c" width="1500px;"/><br><br> -->

## 💻 Getting Started

> 해당 프로젝트 설치 및 실행 방법
> <br>

### Installation

```
npm install
```

### Develop Mode

```
npm run dev
```

### Production

```
npm run start
```

<br><br>

## :open_file_folder: Project Structure

```markdown
📂 Picketch-backend
├── config # 설정 파일
├── controllers # 로직 및 요청 처리
├── docs # 문서 (swagger, diagram)
│   ├── diagram
│   ├── component
│   └── paths
├── middlewares # 미들웨어
├── models # 데이터베이스 모델
├── routes # 라우터 정의
├── socket # 소켓 통신 처리
├── utils # 유틸 함수 (응답 함수)
└── app.js # 서버 초기화 및 API 라우트 연결
```
