# 온라인 프로그래밍 과제 평가 시스템 설계 문서

## 문서 정보
- **프로젝트명**: Online Programming Assignment Evaluation System
- **작성일**: 2026-01-01
- **버전**: 1.0
- **대상**: 컴퓨터 공학 전공 대학생

---

## 1. 프로젝트 개요

### 1.1 목적
컴퓨터 공학 전공 대학생을 위한 프로그래밍 과제 자동 평가 시스템 구축
- 프로그래밍 과제 출제 및 관리
- 학생 코드 제출 및 자동 평가
- 테스트 케이스 기반 채점
- 코드 품질 분석 및 피드백 제공

### 1.2 주요 요구사항
- **대상 사용자**: 컴퓨터 공학 전공 대학생
- **총 사용자 수**: 약 400명
- **동시 접속자**: 최대 100명
- **지원 언어**: C++, Python, JavaScript, Rust
- **평가 방식**: 테스트 케이스 기반 자동 채점 + 코드 품질 분석
- **배포 환경**: 클라우드 환경 (Kubernetes + Docker)

### 1.3 개발 방식
- **접근법**: Top-Down 방식
- **시작점**: 전체 구조 검증 가능한 Skeleton 구축
- **진행**: 점진적 기능 추가
- **대시보드**: 프로젝트 초기부터 구축 및 개발 진행 상황 추적

---

## 2. 시스템 아키텍처

### 2.1 전체 아키텍처 개요

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (React)                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │Dashboard │  │ Problem  │  │  Code    │  │  Result  │   │
│  │          │  │  List    │  │  Editor  │  │  View    │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ↕ REST API
┌─────────────────────────────────────────────────────────────┐
│                   Backend (Node.js/Express)                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Auth   │  │ Problem  │  │Submission│  │Evaluation│   │
│  │  Service │  │ Service  │  │ Service  │  │  Service │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│              Code Execution Engine (Sandbox)                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Docker  │  │  Test    │  │  Code    │  │  Result  │   │
│  │Container │  │  Runner  │  │ Quality  │  │ Collector│   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                    Database (MongoDB)                        │
│     Users | Problems | Submissions | TestCases | Results    │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 클라우드 네이티브 아키텍처 (Kubernetes)

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Ingress Controller (nginx-ingress)                    │ │
│  │  - 외부 트래픽 라우팅                                   │ │
│  │  - SSL/TLS 종료                                         │ │
│  └────────────────────────────────────────────────────────┘ │
│                            ↓                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Frontend    │  │   Backend    │  │ Judge Engine │     │
│  │  Deployment  │  │  Deployment  │  │  Deployment  │     │
│  │              │  │              │  │              │     │
│  │  Replicas: 2 │  │  Replicas: 3 │  │  Replicas: 5 │     │
│  │  Port: 80    │  │  Port: 3000  │  │  Port: 8080  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│         ↓                  ↓                  ↓             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Service    │  │   Service    │  │   Service    │     │
│  │  (ClusterIP) │  │  (ClusterIP) │  │  (ClusterIP) │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                            ↓                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Persistent Storage                                     │ │
│  │  ┌──────────────┐  ┌──────────────┐                   │ │
│  │  │   MongoDB    │  │    Redis     │                   │ │
│  │  │ StatefulSet  │  │  Deployment  │                   │ │
│  │  │   + PVC      │  │              │                   │ │
│  │  └──────────────┘  └──────────────┘                   │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 컴포넌트 설명

#### 2.3.1 Frontend (React)
- **역할**: 사용자 인터페이스 제공
- **주요 기능**:
  - Dashboard: 통계 및 현황 표시
  - Problem List: 문제 목록 및 검색
  - Code Editor: 코드 작성 및 제출
  - Result View: 채점 결과 및 피드백 표시
- **배포**: Nginx 기반 정적 파일 서빙

#### 2.3.2 Backend (Node.js/Express)
- **역할**: 비즈니스 로직 처리 및 API 제공
- **주요 서비스**:
  - Auth Service: 사용자 인증/권한 관리
  - Problem Service: 문제 CRUD 및 관리
  - Submission Service: 코드 제출 관리
  - Evaluation Service: 채점 결과 처리
- **API**: RESTful API

#### 2.3.3 Code Execution Engine (Judge)
- **역할**: 안전한 코드 실행 및 평가
- **주요 기능**:
  - Docker Container: 샌드박스 환경
  - Test Runner: 테스트 케이스 실행
  - Code Quality Analyzer: 코드 품질 분석
  - Result Collector: 결과 수집 및 반환
- **언어 지원**: C++, Python, JavaScript, Rust

#### 2.3.4 Database (MongoDB)
- **역할**: 데이터 영구 저장
- **주요 컬렉션**:
  - users: 사용자 정보
  - problems: 문제 정보
  - submissions: 제출 이력
  - testcases: 테스트 케이스
  - results: 채점 결과

#### 2.3.5 Message Queue (Redis + Bull)
- **역할**: 비동기 작업 처리
- **사용 목적**:
  - 코드 실행 작업 큐잉
  - 부하 분산
  - 작업 재시도 관리

---

## 3. 기술 스택

### 3.1 Frontend
```
- React 18 + TypeScript
- React Router v6 (페이지 라우팅)
- Monaco Editor (코드 에디터)
- Recharts (대시보드 차트)
- Axios (API 통신)
- TailwindCSS (스타일링)
- Vite (빌드 도구)
```

### 3.2 Backend
```
- Node.js 20 LTS
- Express.js 4.x
- TypeScript
- MongoDB + Mongoose
- JWT (인증)
- Bull (작업 큐)
- Docker SDK for Node.js
- Jest (테스트)
```

### 3.3 Code Execution
```
- Docker Engine
- Docker Compose (개발 환경)
- 언어별 공식 이미지:
  - C++: gcc:latest
  - Python: python:3.11-slim
  - JavaScript: node:20-alpine
  - Rust: rust:latest
```

### 3.4 Infrastructure
```
- Kubernetes 1.28+
- Docker 24+
- Nginx Ingress Controller
- Helm (패키지 관리)
- Prometheus + Grafana (모니터링)
```

### 3.5 Development Tools
```
- Git (버전 관리)
- ESLint + Prettier (코드 포맷팅)
- Docker Compose (로컬 개발)
- Postman (API 테스트)
```

---

## 4. 데이터베이스 스키마 설계

### 4.1 Users Collection
```javascript
{
  _id: ObjectId,
  email: String (unique, required),
  password: String (hashed, required),
  name: String (required),
  role: String (enum: ['student', 'instructor', 'admin']),
  studentId: String (optional),
  createdAt: Date,
  updatedAt: Date
}
```

### 4.2 Problems Collection
```javascript
{
  _id: ObjectId,
  title: String (required),
  description: String (required),
  difficulty: String (enum: ['easy', 'medium', 'hard']),
  timeLimit: Number (milliseconds),
  memoryLimit: Number (MB),
  allowedLanguages: [String], // ['cpp', 'python', 'javascript', 'rust']
  tags: [String],
  createdBy: ObjectId (ref: 'Users'),
  testCases: [ObjectId] (ref: 'TestCases'),
  isPublic: Boolean,
  createdAt: Date,
  updatedAt: Date
}
```

### 4.3 TestCases Collection
```javascript
{
  _id: ObjectId,
  problemId: ObjectId (ref: 'Problems'),
  input: String,
  expectedOutput: String,
  isPublic: Boolean, // 공개 여부 (학생에게 보여줄지)
  points: Number, // 이 테스트 케이스의 배점
  createdAt: Date
}
```

### 4.4 Submissions Collection
```javascript
{
  _id: ObjectId,
  problemId: ObjectId (ref: 'Problems'),
  userId: ObjectId (ref: 'Users'),
  language: String (enum: ['cpp', 'python', 'javascript', 'rust']),
  code: String (required),
  status: String (enum: ['pending', 'running', 'completed', 'error']),
  result: {
    verdict: String (enum: ['AC', 'WA', 'TLE', 'MLE', 'RE', 'CE']),
    // AC: Accepted, WA: Wrong Answer, TLE: Time Limit Exceeded
    // MLE: Memory Limit Exceeded, RE: Runtime Error, CE: Compile Error
    score: Number,
    executionTime: Number (milliseconds),
    memoryUsed: Number (KB),
    testCaseResults: [{
      testCaseId: ObjectId,
      passed: Boolean,
      executionTime: Number,
      memoryUsed: Number,
      output: String,
      error: String
    }],
    codeQuality: {
      score: Number,
      issues: [{
        line: Number,
        message: String,
        severity: String
      }]
    }
  },
  createdAt: Date,
  completedAt: Date
}
```

### 4.5 Dashboard Statistics (임베디드)
```javascript
// 실시간 계산 또는 별도 컬렉션
{
  totalUsers: Number,
  totalProblems: Number,
  totalSubmissions: Number,
  activeUsers: Number (last 24h),
  submissionsByLanguage: {
    cpp: Number,
    python: Number,
    javascript: Number,
    rust: Number
  },
  problemsSolvedRate: Number,
  averageExecutionTime: Number
}
```

---

## 5. API 설계

### 5.1 인증 API
```
POST   /api/auth/register          # 회원가입
POST   /api/auth/login             # 로그인
POST   /api/auth/logout            # 로그아웃
GET    /api/auth/me                # 현재 사용자 정보
```

### 5.2 문제 API
```
GET    /api/problems               # 문제 목록 조회
GET    /api/problems/:id           # 문제 상세 조회
POST   /api/problems               # 문제 생성 (관리자)
PUT    /api/problems/:id           # 문제 수정 (관리자)
DELETE /api/problems/:id           # 문제 삭제 (관리자)
GET    /api/problems/:id/testcases # 공개 테스트 케이스 조회
```

### 5.3 제출 API
```
POST   /api/submissions            # 코드 제출
GET    /api/submissions/:id        # 제출 결과 조회
GET    /api/submissions/my         # 내 제출 목록
GET    /api/submissions/problem/:problemId  # 특정 문제의 제출 목록
```

### 5.4 대시보드 API
```
GET    /api/dashboard/stats        # 전체 통계
GET    /api/dashboard/user-stats   # 사용자별 통계
GET    /api/dashboard/problem-stats # 문제별 통계
GET    /api/dashboard/recent-submissions # 최근 제출 목록
```

### 5.5 사용자 API
```
GET    /api/users                  # 사용자 목록 (관리자)
GET    /api/users/:id              # 사용자 정보 조회
PUT    /api/users/:id              # 사용자 정보 수정
GET    /api/users/:id/submissions  # 사용자 제출 이력
```

---

## 6. 코드 실행 및 평가 프로세스

### 6.1 실행 흐름
```
1. 사용자 코드 제출
   ↓
2. Backend: 제출 정보 저장 (status: 'pending')
   ↓
3. Backend: Bull Queue에 작업 추가
   ↓
4. Judge Worker: 큐에서 작업 가져오기
   ↓
5. Judge: Docker 컨테이너 생성
   ↓
6. Judge: 코드 컴파일 (필요시)
   ↓
7. Judge: 각 테스트 케이스 실행
   ↓
8. Judge: 결과 수집 및 채점
   ↓
9. Judge: 코드 품질 분석 실행
   ↓
10. Judge: 컨테이너 정리
    ↓
11. Judge: 결과를 Backend로 전송
    ↓
12. Backend: 제출 정보 업데이트 (status: 'completed')
    ↓
13. Frontend: 실시간 결과 표시 (WebSocket/Polling)
```

### 6.2 샌드박스 보안
```
- Docker 컨테이너 격리
- 리소스 제한:
  - CPU: 1 core
  - Memory: 256MB
  - Disk: 100MB (tmpfs)
  - Network: 차단
  - 실행 시간: 문제별 설정 (기본 5초)
- 읽기 전용 파일 시스템
- 권한 제한 (non-root 사용자)
- 프로세스 수 제한
```

### 6.3 언어별 실행 설정

#### C++
```bash
# 컴파일
g++ -std=c++17 -O2 -Wall solution.cpp -o solution

# 실행
timeout 5s ./solution < input.txt > output.txt
```

#### Python
```bash
# 실행
timeout 5s python3 solution.py < input.txt > output.txt
```

#### JavaScript
```bash
# 실행
timeout 5s node solution.js < input.txt > output.txt
```

#### Rust
```bash
# 컴파일
rustc -O solution.rs -o solution

# 실행
timeout 5s ./solution < input.txt > output.txt
```

---

## 7. 개발 단계 (Phase)

### Phase 1: Skeleton 구축 (2-3주)

**목표**: 전체 시스템 구조 검증 및 기본 기능 동작

**구현 내용**:
1. 프로젝트 구조 설정
   - Frontend: React + TypeScript 프로젝트 초기화
   - Backend: Node.js + Express 프로젝트 초기화
   - Docker Compose로 로컬 개발 환경 구성

2. 기본 대시보드
   - 전체 통계 표시 (더미 데이터)
   - 최근 제출 목록
   - 시스템 상태 모니터링

3. 간단한 인증
   - 회원가입/로그인
   - JWT 기반 인증
   - Role 기반 접근 제어 (학생/교수)

4. 문제 목록
   - 문제 목록 조회
   - 문제 상세 보기
   - 간단한 문제 생성 (관리자)

5. Hello World 제출
   - Monaco Editor 통합
   - 단순 코드 제출
   - "Hello, World!" 출력 검증
   - 결과 표시

**산출물**:
- 동작하는 프로토타입
- Docker Compose 설정
- 기본 Kubernetes 매니페스트
- README 및 설치 가이드

### Phase 2: Core Features (4-5주)

**목표**: 실제 사용 가능한 채점 시스템

**구현 내용**:
1. 다중 언어 지원
   - C++, Python, JavaScript, Rust 컴파일/실행
   - 언어별 Docker 이미지 구성
   - 언어별 시간/메모리 제한 설정

2. 테스트 케이스 시스템
   - 테스트 케이스 생성/관리
   - 공개/비공개 테스트 케이스
   - 다중 테스트 케이스 실행
   - 부분 점수 시스템

3. 채점 엔진 고도화
   - Bull Queue 통합
   - 병렬 실행
   - 재시도 메커니즘
   - 에러 처리

4. 제출 이력 관리
   - 개인별 제출 이력
   - 문제별 제출 통계
   - 최고 점수 추적

5. 문제 관리 강화
   - 문제 편집기
   - 테스트 케이스 일괄 업로드
   - 문제 태그 시스템

**산출물**:
- 완전한 채점 시스템
- 다중 언어 지원
- 성능 테스트 결과

### Phase 3: Advanced Features (3-4주)

**목표**: 고급 기능 및 사용자 경험 향상

**구현 내용**:
1. 코드 품질 분석
   - ESLint (JavaScript)
   - Pylint (Python)
   - Clippy (Rust)
   - Cppcheck (C++)
   - 코드 스타일 점수화

2. 실시간 기능
   - WebSocket 통합
   - 실시간 제출 결과 업데이트
   - 실시간 대시보드 통계

3. 성능 측정
   - 정확한 실행 시간 측정
   - 메모리 사용량 추적
   - 성능 비교 그래프

4. 순위표 (Leaderboard)
   - 문제별 순위
   - 전체 순위
   - 언어별 순위

5. 사용자 프로필
   - 해결한 문제 목록
   - 통계 그래프
   - 배지 시스템 (선택)

**산출물**:
- 완성된 프로덕션 시스템
- 성능 최적화 보고서
- 사용자 매뉴얼

---

## 8. Kubernetes 배포 구성

### 8.1 네임스페이스
```yaml
# 환경별 네임스페이스
- dev (개발)
- staging (스테이징)
- production (운영)
```

### 8.2 주요 리소스

#### Frontend Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    spec:
      containers:
      - name: frontend
        image: your-registry/frontend:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

#### Backend Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    spec:
      containers:
      - name: backend
        image: your-registry/backend:latest
        ports:
        - containerPort: 3000
        env:
        - name: MONGODB_URI
          valueFrom:
            secretKeyRef:
              name: db-secrets
              key: mongodb-uri
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
```

#### Judge Engine Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: judge-engine
spec:
  replicas: 5
  selector:
    matchLabels:
      app: judge-engine
  template:
    spec:
      containers:
      - name: judge
        image: your-registry/judge:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: docker-sock
          mountPath: /var/run/docker.sock
        securityContext:
          privileged: true  # Docker-in-Docker 필요
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
      volumes:
      - name: docker-sock
        hostPath:
          path: /var/run/docker.sock
```

#### MongoDB StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    spec:
      containers:
      - name: mongodb
        image: mongo:7
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
  volumeClaimTemplates:
  - metadata:
      name: mongodb-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
```

### 8.3 Ingress 설정
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - yourdomain.com
    secretName: tls-secret
  rules:
  - host: yourdomain.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 3000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

---

## 9. 보안 고려사항

### 9.1 인증 및 권한
- JWT 기반 토큰 인증
- Access Token (15분) + Refresh Token (7일)
- Role-based Access Control (RBAC)
- API Rate Limiting

### 9.2 코드 실행 보안
- Docker 컨테이너 격리
- 네트워크 차단
- 리소스 제한 (CPU, 메모리, 디스크)
- 실행 시간 제한
- 읽기 전용 파일 시스템
- Non-root 사용자로 실행

### 9.3 데이터 보안
- 비밀번호 해싱 (bcrypt)
- MongoDB 연결 암호화
- 환경 변수로 민감 정보 관리
- Kubernetes Secrets 사용
- HTTPS 강제 (TLS 1.3)

### 9.4 입력 검증
- 코드 크기 제한 (최대 10KB)
- 파일명 검증
- SQL Injection 방지 (Mongoose ODM)
- XSS 방지 (입력 sanitization)

---

## 10. 모니터링 및 로깅

### 10.1 모니터링
```
- Prometheus: 메트릭 수집
- Grafana: 시각화 대시보드
- 주요 메트릭:
  - API 응답 시간
  - 제출 처리 시간
  - 에러 발생률
  - 리소스 사용률 (CPU, 메모리)
  - 동시 접속자 수
```

### 10.2 로깅
```
- ELK Stack (Elasticsearch, Logstash, Kibana)
- 로그 레벨: DEBUG, INFO, WARN, ERROR
- 구조화된 로그 (JSON)
- 요청/응답 로그
- 에러 스택 트레이스
```

### 10.3 알람
```
- 높은 에러율 (>5%)
- 응답 시간 지연 (>1초)
- 디스크 사용률 (>80%)
- Pod 재시작
```

---

## 11. 성능 목표

### 11.1 응답 시간
- API 응답: < 200ms (p95)
- 코드 실행: < 5초 (언어별 설정)
- 페이지 로딩: < 2초

### 11.2 처리량
- 동시 제출 처리: 최소 100개
- API 요청: 1000 req/s

### 11.3 가용성
- Uptime: 99.5% (월 3.6시간 다운타임 허용)
- 데이터 백업: 일 1회

---

## 12. 개발 환경 설정

### 12.1 로컬 개발
```bash
# Docker Compose로 전체 스택 실행
docker-compose up -d

# 포트 매핑
- Frontend: http://localhost:3000
- Backend: http://localhost:5000
- MongoDB: localhost:27017
- Redis: localhost:6379
```

### 12.2 필요한 도구
```
- Node.js 20 LTS
- Docker Desktop
- kubectl
- helm
- Git
- VS Code (권장)
```

---

## 13. 배포 전략

### 13.1 CI/CD 파이프라인
```
1. Git Push (main branch)
   ↓
2. GitHub Actions 트리거
   ↓
3. 테스트 실행 (Jest)
   ↓
4. Docker 이미지 빌드
   ↓
5. 이미지 레지스트리 푸시
   ↓
6. Kubernetes 배포 (Rolling Update)
   ↓
7. Health Check
   ↓
8. 배포 완료 / 롤백
```

### 13.2 환경별 배포
- **Development**: 자동 배포 (모든 커밋)
- **Staging**: 수동 승인 후 배포
- **Production**: 수동 승인 + 배포 시간 제한

---

## 14. 향후 확장 계획

### 14.1 단기 (3-6개월)
- 실시간 협업 기능
- 문제 난이도 자동 분류
- AI 기반 힌트 시스템
- 모바일 앱

### 14.2 장기 (6-12개월)
- 대회/콘테스트 모드
- 팀 프로젝트 지원
- 코드 리뷰 시스템
- 통합 개발 환경 (Web IDE)
- 다국어 지원

---

## 15. 예상 비용 (클라우드)

### 15.1 AWS EKS 기준 (월)
```
- EKS Cluster: $73
- EC2 인스턴스 (t3.medium x 3): $100
- MongoDB (db.t3.medium): $60
- Load Balancer: $20
- Storage (100GB): $10
- 데이터 전송: $20

총 예상 비용: 약 $283/월 (약 37만원)
```

### 15.2 비용 최적화
- Spot 인스턴스 활용
- Auto Scaling 설정
- 리소스 사용 모니터링
- 개발 환경은 자동 종료

---

## 16. 리스크 및 대응

### 16.1 주요 리스크

| 리스크 | 영향 | 대응 방안 |
|--------|------|-----------|
| 악의적 코드 실행 | 높음 | 샌드박스 강화, 네트워크 차단 |
| 과부하 공격 | 중간 | Rate Limiting, Auto Scaling |
| 데이터 유실 | 높음 | 정기 백업, 복제 설정 |
| 부정 행위 (치팅) | 중간 | 코드 유사도 검사, 제출 시간 추적 |

### 16.2 대응 전략
- 정기적인 보안 감사
- 침투 테스트
- 재해 복구 계획 (DR)
- 정기 백업 및 복원 테스트

---

## 17. 프로젝트 일정

### 17.1 전체 타임라인 (10-12주)

```
Week 1-3:   Phase 1 - Skeleton 구축
Week 4-8:   Phase 2 - Core Features
Week 9-12:  Phase 3 - Advanced Features
Week 13+:   운영 및 유지보수
```

### 17.2 마일스톤
- Week 3: MVP 데모
- Week 8: 베타 테스트 시작
- Week 12: 프로덕션 배포

---

## 18. 팀 역할 (참고)

### 18.1 권장 팀 구성
- **Frontend Developer**: 1-2명
- **Backend Developer**: 2명
- **DevOps Engineer**: 1명
- **QA/Tester**: 1명

### 18.2 현재 상황
- 현재 개발자: 1명 (Full Stack)
- 우선순위: Skeleton 구축 및 검증

---

## 19. 참고 자료

### 19.1 유사 시스템
- LeetCode
- HackerRank
- Codeforces
- BOJ (Baekjoon Online Judge)

### 19.2 기술 문서
- Docker Documentation
- Kubernetes Documentation
- React Documentation
- Node.js Best Practices

---

## 20. 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 1.0 | 2026-01-01 | 초기 설계 문서 작성 | Peter |

---

## 부록 A: 프로젝트 디렉토리 구조

```
online-judge/
├── frontend/                 # React Frontend
│   ├── src/
│   │   ├── components/      # 재사용 컴포넌트
│   │   ├── pages/           # 페이지 컴포넌트
│   │   ├── services/        # API 서비스
│   │   ├── hooks/           # Custom Hooks
│   │   ├── utils/           # 유틸리티 함수
│   │   └── App.tsx
│   ├── public/
│   ├── Dockerfile
│   └── package.json
│
├── backend/                  # Node.js Backend
│   ├── src/
│   │   ├── controllers/     # 라우트 컨트롤러
│   │   ├── models/          # Mongoose 모델
│   │   ├── routes/          # API 라우트
│   │   ├── middlewares/     # 미들웨어
│   │   ├── services/        # 비즈니스 로직
│   │   ├── utils/           # 유틸리티
│   │   └── server.ts
│   ├── Dockerfile
│   └── package.json
│
├── judge-engine/             # Code Execution Engine
│   ├── src/
│   │   ├── executors/       # 언어별 실행기
│   │   ├── sandbox/         # Docker 컨테이너 관리
│   │   ├── queue/           # Bull Queue Worker
│   │   └── analyzer/        # 코드 품질 분석
│   ├── Dockerfile
│   └── package.json
│
├── k8s/                      # Kubernetes 매니페스트
│   ├── base/                # 공통 설정
│   ├── overlays/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── helm/                # Helm Charts
│
├── docker-compose.yml        # 로컬 개발 환경
├── .github/
│   └── workflows/           # CI/CD 파이프라인
├── docs/                     # 문서
└── README.md
```

---

**문서 종료**

이 설계 문서는 프로젝트 진행에 따라 지속적으로 업데이트됩니다.
