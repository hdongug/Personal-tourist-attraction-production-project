# 관광지 정보 시스템

관광지 정보를 조회하고 검색할 수 있는 콘솔 애플리케이션입니다.

## 기능

1. 전체 관광지 목록 조회 (페이지)
2. 권역별 관광지 조회
3. 관광지 검색

## 프로젝트 구조

```
travel-app/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── travel/
│       │           ├── app/
│       │           │   └── TravelApp.java
│       │           ├── dao/
│       │           │   └── TravelDao.java
│       │           ├── service/
│       │           │   └── TravelService.java
│       │           └── vo/
│       │               └── TravelVO.java
│       └── resources/
│           ├── application.properties
│           └── data/
│               └── travel.csv
├── lib/
│   └── mysql-connector-j-8.0.xx.jar
├── pom.xml
└── README.md
```

# TravelApp 시스템 플로우 아키텍처

## 📋 목차
1. [시스템 개요](#시스템-개요)
2. [아키텍처 레이어](#아키텍처-레이어)
3. [데이터 플로우](#데이터-플로우)
4. [API 엔드포인트 플로우](#api-엔드포인트-플로우)
5. [데이터베이스 스키마](#데이터베이스-스키마)

---

## 시스템 개요

**TravelApp**은 Spring Boot 기반의 관광지 정보 관리 시스템입니다.
- **프레임워크**: Spring Boot
- **데이터베이스**: MySQL
- **아키텍처 패턴**: MVC (Model-View-Controller) + Service Layer
- **포트**: 8080

---

## 아키텍처 레이어

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                         │
│                    (Web Browser / API Client)               │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP Request
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                       │
│  ┌───────────────────────────────────────────────────────┐  │
│  │         TravelController (@RestController)            │  │
│  │  - @GetMapping("/api/travels/all")                    │  │
│  │  - @GetMapping("/api/travels/district/{district}")    │  │
│  │  - @GetMapping("/api/travels/search")                 │  │
│  │  - @GetMapping("/api/travels/random")                 │  │
│  └───────────────────────┬───────────────────────────────┘  │
└────────────────────────────┼────────────────────────────────┘
                           │ Method Call
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     BUSINESS LAYER                          │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              TravelService                            │  │
│  │  - getTotalPages(pageSize)                            │  │
│  │  - insertTravel(TravelVO)                             │  │
│  │  - getAllTravels(page, pageSize)                      │  │
│  │  - getTravelsByDistrict(district)                     │  │
│  │  - searchTravels(keyword)                             │  │
│  │  - getRandomTravels(limit)                            │  │
│  │                                                       │  │
│  │  [Exception Handling & Business Logic]                │  │
│  └───────────────────────┬───────────────────────────────┘  │
└────────────────────────────┼────────────────────────────────┘
                           │ DAO Call
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    PERSISTENCE LAYER                        │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                  TravelDao                            │  │
│  │  - getConnection()                                    │  │
│  │  - getTotalPages(pageSize)                            │  │
│  │  - insertTravel(TravelVO)                             │  │
│  │  - getAllTravels(page, pageSize)                      │  │
│  │  - getTravelsByDistrict(district)                     │  │
│  │  - searchTravels(keyword)                             │  │
│  │  - getRandomTravels(limit)                            │  │
│  │  - mapResultSetToTravel(ResultSet)                    │  │
│  └───────────────────────┬───────────────────────────────┘  │
└────────────────────────────┼────────────────────────────────┘
                           │ JDBC
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      DATABASE LAYER                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           MySQL Database (travel_db)                  │  │
│  │                                                       │  │
│  │  Table: travel                                        │  │
│  │  - no (INT, PK, AUTO_INCREMENT)                       │  │
│  │  - district (VARCHAR)                                 │  │
│  │  - title (VARCHAR)                                    │  │
│  │  - description (TEXT)                                 │  │
│  │  - address (VARCHAR)                                  │  │
│  │  - phone (VARCHAR)                                    │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                        DATA MODEL                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                   TravelVO                            │  │
│  │  - int no                                             │  │
│  │  - String district                                    │  │
│  │  - String title                                       │  │
│  │  - String description                                 │  │
│  │  - String address                                     │  │
│  │  - String phone                                       │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 데이터 플로우

### 1️⃣ 전체 관광지 조회 (페이징)

```
[Client]
   │
   │ GET /api/travels/all?page=1
   ▼
[TravelController.getAllTravels()]
   │
   │ 1. travelService.getTotalPages(PAGE_SIZE)
   │ 2. travelService.getAllTravels(page, PAGE_SIZE)
   ▼
[TravelService]
   │
   │ 1. travelDao.getTotalPages(pageSize)
   │ 2. travelDao.getAllTravels(page, pageSize)
   ▼
[TravelDao]
   │
   │ 1. SQL: SELECT COUNT(*) FROM travel
   │ 2. SQL: SELECT * FROM travel LIMIT ? OFFSET ?
   │ 3. mapResultSetToTravel(rs) → TravelVO
   ▼
[MySQL Database]
   │
   │ Execute Query & Return ResultSet
   ▼
[Response Flow]
   │
   │ TravelVO List → TravelService → TravelController
   ▼
[Client]
   │
   │ JSON Response:
   │ {
   │   "success": true,
   │   "travels": [...],
   │   "currentPage": 1,
   │   "totalPages": 10
   │ }
```

### 2️⃣ 지역별 관광지 조회

```
[Client]
   │
   │ GET /api/travels/district/서울
   ▼
[TravelController.getTravelsByDistrict("서울")]
   │
   │ travelService.getTravelsByDistrict("서울")
   ▼
[TravelService]
   │
   │ travelDao.getTravelsByDistrict("서울")
   ▼
[TravelDao]
   │
   │ SQL: SELECT * FROM travel WHERE district = ?
   │ mapResultSetToTravel(rs) → TravelVO List
   ▼
[MySQL Database]
   │
   │ Execute Query & Return ResultSet
   ▼
[Response Flow]
   │
   │ TravelVO List → TravelService → TravelController
   ▼
[Client]
   │
   │ JSON Response:
   │ {
   │   "success": true,
   │   "travels": [...]
   │ }
```

### 3️⃣ 키워드 검색

```
[Client]
   │
   │ GET /api/travels/search?keyword=박물관
   ▼
[TravelController.searchTravels("박물관")]
   │
   │ travelService.searchTravels("박물관")
   ▼
[TravelService]
   │
   │ travelDao.searchTravels("박물관")
   ▼
[TravelDao]
   │
   │ SQL: SELECT * FROM travel 
   │      WHERE title LIKE ? OR description LIKE ? OR address LIKE ?
   │ Parameters: "%박물관%", "%박물관%", "%박물관%"
   │ mapResultSetToTravel(rs) → TravelVO List
   ▼
[MySQL Database]
   │
   │ Execute Query & Return ResultSet
   ▼
[Response Flow]
   │
   │ TravelVO List → TravelService → TravelController
   ▼
[Client]
   │
   │ JSON Response:
   │ {
   │   "success": true,
   │   "travels": [...]
   │ }
```

### 4️⃣ 랜덤 관광지 조회

```
[Client]
   │
   │ GET /api/travels/random?limit=10
   ▼
[TravelController.getRandomTravels(10)]
   │
   │ travelService.getRandomTravels(10)
   ▼
[TravelService]
   │
   │ travelDao.getRandomTravels(10)
   ▼
[TravelDao]
   │
   │ SQL: SELECT * FROM travel ORDER BY RAND() LIMIT ?
   │ mapResultSetToTravel(rs) → TravelVO List
   ▼
[MySQL Database]
   │
   │ Execute Query & Return ResultSet
   ▼
[Response Flow]
   │
   │ TravelVO List → TravelService → TravelController
   ▼
[Client]
   │
   │ JSON Response:
   │ {
   │   "success": true,
   │   "travels": [...]
   │ }
```

### 5️⃣ 관광지 등록

```
[Client]
   │
   │ POST /api/travels (TravelVO JSON)
   ▼
[TravelController.insertTravel(TravelVO)]
   │
   │ travelService.insertTravel(travel)
   ▼
[TravelService]
   │
   │ travelDao.insertTravel(travel)
   ▼
[TravelDao]
   │
   │ SQL: INSERT INTO travel 
   │      (district, title, description, address, phone) 
   │      VALUES (?, ?, ?, ?, ?)
   ▼
[MySQL Database]
   │
   │ Execute Insert & Commit
   ▼
[Response Flow]
   │
   │ Success/Exception → TravelService → TravelController
   ▼
[Client]
   │
   │ JSON Response:
   │ {
   │   "success": true/false,
   │   "message": "..."
   │ }
```

---

## API 엔드포인트 플로우

### 📍 API 엔드포인트 맵

| HTTP Method | Endpoint | Description | Parameters |
|------------|----------|-------------|------------|
| GET | `/api/travels/all` | 전체 관광지 조회 (페이징) | `page` (default: 1) |
| GET | `/api/travels/district/{district}` | 지역별 관광지 조회 | `district` (path variable) |
| GET | `/api/travels/search` | 키워드 검색 | `keyword` (query param) |
| GET | `/api/travels/random` | 랜덤 관광지 조회 | `limit` (default: 10) |

### 🔄 요청/응답 플로우

```
┌──────────┐         ┌──────────────┐         ┌──────────────┐
│          │  HTTP   │              │  Java   │              │
│  Client  │ ──────► │  Controller  │ ──────► │   Service    │
│          │         │              │         │              │
└──────────┘         └──────────────┘         └──────────────┘
     ▲                                                │
     │                                                │
     │                                                ▼
     │                                         ┌──────────────┐
     │                                         │              │
     │                                         │     DAO      │
     │                                         │              │
     │                                         └──────────────┘
     │                                                │
     │                                                │
     │                                                ▼
     │                                         ┌──────────────┐
     │                                         │              │
     │                                         │    MySQL     │
     │                                         │   Database   │
     │                                         │              │
     │                                         └──────────────┘
     │                                                │
     │                JSON Response                   │
     └────────────────────────────────────────────────┘
```

---

## 데이터베이스 스키마

### 📊 travel 테이블

```sql
CREATE TABLE travel (
    no INT PRIMARY KEY AUTO_INCREMENT,
    district VARCHAR(50) NOT NULL,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    address VARCHAR(300),
    phone VARCHAR(20)
);
```

### 🔗 데이터베이스 연결 정보

```
URL: jdbc:mysql://localhost:3306/travel_db
User: root
Password: qwer1234!
Driver: MySQL JDBC Driver
```

---

## 예외 처리 플로우

```
[Any Layer]
   │
   │ SQLException or RuntimeException
   ▼
[TravelService]
   │
   │ Catch SQLException
   │ Wrap in RuntimeException with message
   ▼
[TravelController]
   │
   │ Catch Exception
   │ Build error response
   ▼
[Client]
   │
   │ JSON Response:
   │ {
   │   "success": false,
   │   "message": "Error description"
   │ }
```

---

## 애플리케이션 시작 플로우

```
[TravelApp.main()]
   │
   │ SpringApplication.run(TravelApp.class, args)
   ▼
[Spring Boot]
   │
   │ 1. Component Scan (com.travel.*)
   │ 2. Initialize TravelController (@RestController)
   │ 3. Start Embedded Tomcat Server
   │ 4. Listen on port 8080
   ▼
[Server Ready]
   │
   │ Console Output:
   │ "🌐 관광지 정보 시스템 웹 서버 시작!"
   │ "📍 브라우저에서 http://localhost:8080 을 열어주세요"
```

---

## 주요 컴포넌트 의존성

```
TravelApp (Entry Point)
   │
   └─► Spring Boot Framework
         │
         └─► TravelController
               │
               ├─► TravelService
               │     │
               │     └─► TravelDao
               │           │
               │           └─► MySQL Database
               │
               └─► TravelVO (Data Transfer)
```

---

## 설계 패턴

1. **MVC Pattern**: Controller-Service-DAO 구조
2. **DTO Pattern**: TravelVO를 통한 데이터 전송
3. **Singleton Pattern**: Service와 DAO는 단일 인스턴스
4. **Repository Pattern**: DAO가 데이터 접근 추상화
5. **Exception Handling**: Service 레이어에서 예외 처리 및 변환

---

## 기술 스택

- **Backend Framework**: Spring Boot
- **Web Layer**: Spring Web MVC (@RestController)
- **Database**: MySQL 8.x
- **JDBC**: Java Database Connectivity
- **Build Tool**: Maven/Gradle (추정)
- **Java Version**: Java 8+

---


