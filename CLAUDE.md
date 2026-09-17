# CLAUDE.md - WeWorkHere

> AI 개발 도구(Claude Code)를 위한 프로젝트 가이드.

## 빠른 참조

### 필수 명령어
```bash
# 서비스 시작
docker-compose up -d --build

# 로그 확인
docker-compose logs -f backend
docker-compose logs -f frontend

# Backend 개발 (가상환경 필수!)
cd backend && source venv/bin/activate
python3 -m uvicorn app.main:app --host 0.0.0.0 --port 25050 --reload

# Frontend 개발
cd frontend && npm run dev

# Database 마이그레이션
cd backend && source venv/bin/activate
alembic upgrade head
alembic revision --autogenerate -m "description"

# API 문서
# http://localhost:25050/docs
```

### 핵심 아키텍처
- **Frontend**: Page → Organisms → Hooks → Services (API)
- **Backend**: Router → Service → Repository → Database
- **중요**: 중간 계층 건너뛰기 절대 금지!

### 포트
- Frontend: 24050
- Backend: 25050
- Database: 5443 (외부) / 5432 (컨테이너 내부)

---

## 프로젝트 개요

**WeWorkHere**는 한국에서 근무하는 외국인 노동자를 위한 익명 커뮤니티 플랫폼입니다.

**주요 특징**:
- 익명 닉네임 기반 인증 (전화번호 없음)
- 다국어 지원 (한국어, 영어, 베트남어, 네팔어)
- 모바일 최적화 (Mobile First)
- 게시글/댓글 시스템
- 카테고리별 필터링

**기술 스택**:
- Frontend: Next.js 14, TypeScript, Tailwind CSS, Axios
- Backend: FastAPI 0.109, Python 3.11, SQLAlchemy 2.0 (Async with asyncpg)
- Database: PostgreSQL 15-alpine
- Deployment: Docker Compose on Mac Mini
- Ports: Frontend 24050, Backend 25050, DB 5443 (external) / 5432 (internal)
- Network: Custom bridge network (weworkhere_network)

---

## 아키텍처

### 디렉토리 구조

```
WeWorkHere/
├── .env                         # 환경 변수
├── docker-compose.yml           # Docker 설정
├── README.md                    # 프로젝트 개요
├── CLAUDE.md                    # 이 파일
│
├── frontend/src/
│   ├── app/                     # Next.js file-based routing
│   │   ├── [lang]/              # 다국어 동적 라우팅
│   │   ├── sitemap.ts           # SEO
│   │   └── robots.ts            # SEO
│   ├── components/              # Atomic Design
│   │   ├── atoms/               # 10-30줄
│   │   ├── molecules/           # 30-80줄
│   │   └── organisms/           # 80-150줄
│   ├── hooks/                   # 비즈니스 로직 (50-150줄)
│   ├── services/                # API 호출
│   ├── types/                   # TypeScript 타입
│   ├── constants/               # 상수
│   ├── dictionaries/            # i18n 번역
│   └── utils/                   # 헬퍼 함수
│
└── backend/app/
    ├── main.py                  # FastAPI 진입점
    ├── core/                    # 설정, DB
    ├── models/                  # SQLAlchemy 모델
    ├── schemas/                 # Pydantic 스키마
    ├── repositories/            # DB 접근
    ├── services/                # 비즈니스 로직
    ├── routers/                 # HTTP 레이어
    └── utils/                   # 헬퍼 함수
```

### 계층별 책임

**Frontend:**
- **atoms** (10-30줄): 단일 UI 요소, props만 사용, API 호출 금지, 복잡한 로직 금지
- **molecules** (30-80줄): atoms 조합, 간단한 로직 허용, API 호출 금지, 전역 상태 금지
- **organisms** (80-150줄): 복잡한 UI 섹션, hooks 사용, 직접 API 호출 금지
- **hooks** (50-150줄): 비즈니스 로직 및 상태 관리, services 호출
- **services**: API 호출만, 비즈니스 로직 금지

**Backend:**
- **Router**: HTTP 요청/응답, Pydantic 검증, service 호출 (비즈니스 로직 금지, DB 직접 접근 금지)
- **Service**: 비즈니스 로직, repositories 조합, 트랜잭션 관리 (HTTP 코드 금지, 직접 SQL 금지)
- **Repository**: SQLAlchemy 쿼리로 DB 접근 (비즈니스 로직 금지)

### 의존성 흐름

```
Frontend: organisms → hooks → services
          molecules → atoms
          hooks → services

Backend: router → service → repository
```

**중요**: 중간 계층 건너뛰기 절대 금지

---

## Docker 네트워크 아키텍처

### 컨테이너 간 통신

```yaml
# docker-compose.yml 네트워크 구조
services:
  frontend:
    - 외부 접근: http://localhost:24050
    - 백엔드 호출: http://backend:25050 (컨테이너명 사용)

  backend:
    - 외부 접근: http://localhost:25050
    - DB 접근: postgresql://db:5432 (내부 포트 5432)

  db:
    - 외부 접근: localhost:5443
    - 내부 포트: 5432 (컨테이너 내부)
    - 네트워크: weworkhere_network (격리된 브리지 네트워크)
```

### 환경 변수 패턴

```bash
# Frontend → Backend (컨테이너 내부)
NEXT_PUBLIC_API_URL=/api/v1                    # 브라우저에서 사용 (상대 경로)
BACKEND_URL=http://backend:25050               # SSR에서 사용 (컨테이너명)

# Backend → Database
DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/dbname  # 내부 포트 5432
```

### 완전한 .env 템플릿

```bash
# === Database Configuration ===
POSTGRES_USER=weworkhere_user
POSTGRES_PASSWORD=your_secure_password_here
POSTGRES_DB=weworkhere_db
POSTGRES_PORT=5443

# === Application Ports ===
BACKEND_PORT=25050
FRONTEND_PORT=24050
HOST=0.0.0.0

# === Frontend Environment Variables ===
NEXT_PUBLIC_API_URL=/api/v1
NEXT_PUBLIC_BASE_URL=http://localhost:24050

# === Backend Environment Variables ===
BACKEND_URL=http://backend:25050
DATABASE_URL=postgresql+asyncpg://weworkhere_user:your_secure_password_here@db:5432/weworkhere_db

# === CORS Configuration ===
ALLOWED_ORIGINS=http://localhost:24050

# === Security Configuration ===
JWT_SECRET_KEY=your_jwt_secret_key_minimum_32_characters_long
SESSION_EXPIRE_HOURS=720

# === Optional: Debug Mode ===
DEBUG=false
```

**중요 사항**:
- `POSTGRES_PORT=5443`: 호스트 머신에서 접근하는 포트
- `db:5432`: Docker 네트워크 내부에서 사용하는 포트
- `DATABASE_URL`은 항상 `db:5432` 사용 (컨테이너 내부 통신)
- `NEXT_PUBLIC_*` 변수는 브라우저에 노출됨 (민감 정보 금지)

---

## 개발 순서

### Backend
1. **models** (DB 스키마)
2. **schemas** (Pydantic DTOs)
3. **repositories** (CRUD 함수)
4. **services** (비즈니스 로직)
5. **routers** (API 엔드포인트)
6. **alembic** 마이그레이션

### Frontend
1. **types** (인터페이스)
2. **services** (API 호출)
3. **hooks** (비즈니스 로직)
4. **atoms** → **molecules** → **organisms**
5. **pages** (라우팅)

---

## 기존 컴포넌트 목록

### Frontend Components

**Atoms** (10-30줄):
- `Button.tsx` - 기본 버튼 컴포넌트
- `Input.tsx` - 텍스트 입력 필드
- `Card.tsx` - 카드 컨테이너
- `Badge.tsx` - 배지/라벨
- `Alert.tsx` - 알림 메시지
- `Select.tsx` - 드롭다운 선택
- `Textarea.tsx` - 텍스트 영역
- `LoadingSpinner.tsx` - 로딩 표시

**Molecules** (30-80줄):
- `PostCard.tsx` - 게시글 카드
- `CommentItem.tsx` - 댓글 아이템
- `CategoryFilter.tsx` - 카테고리 필터
- `LanguageSwitcher.tsx` - 언어 전환
- `UserDisplay.tsx` - 사용자 정보 표시

**Organisms** (80-150줄):
- `PostList.tsx` - 게시글 목록
- `CommentSection.tsx` - 댓글 섹션
- `PostForm.tsx` - 게시글 작성/수정 폼
- `Header.tsx` - 헤더 네비게이션

**Hooks**:
- `useAuth.ts` - 인증 관리 (login, logout, user state)
- `usePost.ts` - 게시글 CRUD 및 좋아요
- `useComments.ts` - 댓글 CRUD
- `useDictionary.ts` - i18n 번역 관리

**Services**:
- `apiClient.ts` - Axios 인스턴스 (interceptors 포함)
- `authService.ts` - 인증 API
- `postService.ts` - 게시글 API
- `commentService.ts` - 댓글 API
- `categoryService.ts` - 카테고리 API

### Backend 계층

**Models** (SQLAlchemy ORM):
- `user.py` - User 모델
- `post.py` - Post 모델
- `comment.py` - Comment 모델
- `category.py` - Category 모델
- `reaction.py` - Reaction 모델 (향후 사용)

**Repositories** (DB 접근):
- `base.py` - BaseRepository[ModelType] 제네릭 클래스
- `user_repository.py` - UserRepository
- `post_repository.py` - PostRepository
- `comment_repository.py` - CommentRepository
- `category_repository.py` - CategoryRepository

**Services** (비즈니스 로직):
- `auth_service.py` - AuthService
- `post_service.py` - PostService
- `comment_service.py` - CommentService

**Routers** (API 엔드포인트):
- `health_router.py` - 헬스체크
- `auth_router.py` - 인증 엔드포인트
- `post_router.py` - 게시글 엔드포인트
- `comment_router.py` - 댓글 엔드포인트
- `category_router.py` - 카테고리 엔드포인트

---

## 네이밍 규칙

| 대상 | 규칙 | 예시 |
|------|------|------|
| Python 변수/함수 | snake_case | `user_name`, `get_user()` |
| Python 클래스 | PascalCase | `UserService`, `UserRepository` |
| Python 파일 | snake_case.py | `user_service.py` |
| TypeScript 변수/함수 | camelCase | `userName`, `getUser()` |
| TypeScript 컴포넌트 | PascalCase | `UserCard`, `PostList` |
| 컴포넌트 파일 | PascalCase.tsx | `UserCard.tsx` |
| Hook 파일 | camelCase.ts | `useAuth.ts`, `usePosts.ts` |
| 상수 | UPPER_SNAKE_CASE | `MAX_RETRY`, `API_URL` |
| DB 테이블 | snake_case (복수) | `users`, `posts`, `comments` |
| DB 컬럼 | snake_case | `created_at`, `user_id` |
| API 엔드포인트 | kebab-case | `/user-profiles`, `/post-categories` |

---

## 타입 힌트 요구사항

모든 함수는 타입 힌트 필수

**Python:**
```python
from typing import Optional, List

async def get_user(user_id: int) -> Optional[User]:
    pass

async def get_posts(skip: int = 0, limit: int = 20) -> List[Post]:
    pass
```

**TypeScript:**
```typescript
interface User {
  id: number;
  nickname: string;
}

function getUser(id: number): User | null {
  return null;
}

async function getPosts(params: PaginationParams): Promise<Post[]> {
  return [];
}
```

---

## 코드 표준

### 하드코딩 규칙
- **허용**: 0, 1, -1, true, false
- **금지**: 반복되는 숫자, URL, 에러 메시지 (상수 사용)

### 함수 크기
- 일반적으로 ≤50줄
- 이 제한을 초과하면 리팩토링 고려

### 환경 변수
- 절대 하드코딩하지 않음
- `.env` 파일에서 관리
- 프로덕션과 개발 환경 분리

---

## 인증 시스템

### 익명 세션 기반 인증

```typescript
// Frontend - localStorage에 세션 토큰 저장
const { user, login, logout } = useAuth();

// 로그인
await login('MyNickname');
// localStorage.setItem('session_token', token)

// 모든 API 요청에 헤더 추가
headers: {
  'X-Session-Token': localStorage.getItem('session_token')
}
```

```python
# Backend - 세션 토큰 검증
@router.get("/me")
async def get_current_user(
    session_token: str = Header(None, alias="X-Session-Token")
):
    user = await auth_service.validate_session(session_token)
    if not user:
        raise HTTPException(401, "Invalid session")
    return user
```

---

## 다국어 (i18n)

### 지원 언어
- 🇰🇷 한국어 (ko)
- 🇬🇧 영어 (en)
- 🇻🇳 베트남어 (vi)
- 🇳🇵 네팔어 (ne)

### 구현

```typescript
// 1. Dictionary Context 사용
const dict = useDictionary();

// 2. 번역 접근
<h1>{dict.home.title}</h1>
<p>{dict.home.description}</p>

// 3. 라우팅
// /ko/posts
// /en/posts
// /vi/posts
// /ne/posts
```

### 번역 파일 구조

```json
// src/dictionaries/ko.json
{
  "common": {
    "login": "로그인",
    "logout": "로그아웃"
  },
  "home": {
    "title": "환영합니다",
    "description": "외국인 노동자 커뮤니티"
  },
  "post": {
    "create": "게시글 작성",
    "edit": "수정",
    "delete": "삭제"
  }
}
```

---

## API 엔드포인트

모든 엔드포인트는 `/api/v1` prefix 사용

### 인증
- `POST /api/v1/auth/anonymous` - 익명 사용자 생성
- `GET /api/v1/auth/me` - 현재 사용자 정보

### 카테고리
- `GET /api/v1/categories` - 카테고리 목록

### 게시글
- `GET /api/v1/posts` - 목록 (pagination + category filter)
- `GET /api/v1/posts/{id}` - 상세
- `POST /api/v1/posts` - 작성
- `PUT /api/v1/posts/{id}` - 수정
- `DELETE /api/v1/posts/{id}` - 삭제
- `POST /api/v1/posts/{id}/like` - 좋아요

### 댓글
- `GET /api/v1/posts/{post_id}/comments` - 목록
- `POST /api/v1/posts/{post_id}/comments` - 작성
- `DELETE /api/v1/comments/{id}` - 삭제

---

## 데이터베이스

### 주요 모델

**User** (사용자)
- `id`: Integer (PK)
- `nickname`: String(50) (unique, not null)
- `session_token`: String(255) (unique, indexed, not null)
- `created_at`: DateTime (default: now)
- `updated_at`: DateTime (default: now, onupdate: now)
- **Relationships**: posts (1:N), comments (1:N), reactions (1:N)

**Category** (카테고리)
- `id`: Integer (PK)
- `name_ko`: String(50) (not null) - 한국어 이름
- `name_en`: String(50) (not null) - 영어 이름
- `name_vi`: String(50) (not null) - 베트남어 이름
- `name_ne`: String(50) (not null) - 네팔어 이름
- `slug`: String(50) (unique, indexed, not null)
- **Relationships**: posts (1:N)

**Post** (게시글)
- `id`: Integer (PK)
- `user_id`: Integer (FK → users.id, indexed, not null)
- `category_id`: Integer (FK → categories.id, nullable, indexed)
- `title`: String(200) (not null)
- `content`: Text (not null)
- `image_url`: String(500) (nullable)
- `view_count`: Integer (default: 0)
- `like_count`: Integer (default: 0)
- `created_at`: DateTime (default: now)
- `updated_at`: DateTime (default: now, onupdate: now)
- **Relationships**: user (N:1), category (N:1), comments (1:N), reactions (1:N)
- **Indexes**: user_id, category_id, created_at

**Comment** (댓글)
- `id`: Integer (PK)
- `post_id`: Integer (FK → posts.id, indexed, not null)
- `user_id`: Integer (FK → users.id, indexed, not null)
- `content`: Text (not null)
- `created_at`: DateTime (default: now)
- `updated_at`: DateTime (default: now, onupdate: now)
- **Relationships**: post (N:1), user (N:1)
- **Indexes**: post_id, user_id, created_at

**Reaction** (반응) - 준비됨
- `id`: Integer (PK)
- `post_id`: Integer (FK → posts.id, not null)
- `user_id`: Integer (FK → users.id, not null)
- `type`: String(20) (not null) - 'like', 'heart', etc.
- `created_at`: DateTime (default: now)
- **Unique Constraint**: (post_id, user_id, type)
- **Relationships**: post (N:1), user (N:1)

### 마이그레이션

```bash
# Backend 디렉토리에서
cd backend
source venv/bin/activate

# 새 마이그레이션 생성
alembic revision --autogenerate -m "add user table"

# 마이그레이션 적용
alembic upgrade head

# 롤백
alembic downgrade -1
```

---

## 일반적인 명령어

### Docker Compose

```bash
# 서비스 시작
docker-compose up -d --build

# 로그 확인
docker-compose logs -f

# 서비스 중지
docker-compose down

# 특정 서비스 재시작
docker-compose restart frontend
docker-compose restart backend
```

### Backend 개발

```bash
cd backend

# 가상환경 활성화
source venv/bin/activate

# 개발 서버 실행
python3 -m uvicorn app.main:app --host 0.0.0.0 --port 25050 --reload

# 테스트
pytest

# API 문서
# http://localhost:25050/docs
```

### Frontend 개발

```bash
cd frontend

# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# 프로덕션 빌드
npm run build

# 프로덕션 서버
npm start
```

---

## 보안 고려사항

### 익명성 보장
- 최소한의 개인정보만 수집 (닉네임만)
- 위치 정보 수집 없음
- IP 로깅 최소화

### 세션 관리
- 세션 토큰은 256비트 랜덤 생성
- localStorage에 저장 (프론트엔드)
- 만료 시간: 720시간 (30일)

### CORS 설정
```python
# 개발 환경
origins = ["http://localhost:24050"]

# 프로덕션 환경
origins = ["https://weworkhere.alldatabox.com"]
```

---

## 에러 처리

### Frontend
```typescript
try {
  const result = await postService.createPost(data);
  setSuccess(true);
} catch (err) {
  const message = err instanceof Error ? err.message : 'Unknown error';
  setError(message);
}
```

### Backend
```python
try:
    result = service.method(request)
    return result
except ValueError as e:
    raise HTTPException(status_code=400, detail=str(e))
except Exception as e:
    raise HTTPException(status_code=500, detail=f"Error: {str(e)}")
```

---

## 테스트

### 테스트 현황
- **Backend**: 테스트 환경 준비됨, pytest 사용 가능
- **Frontend**: 테스트 환경 준비됨, Jest 사용 가능
- **현재 상태**: 자동화된 테스트 아직 작성되지 않음

### Backend 테스트 (준비됨)
```bash
cd backend
source venv/bin/activate

# 테스트 실행
pytest tests/

# 커버리지와 함께 실행
pytest --cov=app tests/

# 특정 테스트 파일만 실행
pytest tests/test_auth.py

# 특정 테스트 함수만 실행
pytest tests/test_auth.py::test_create_anonymous_user
```

### Frontend 테스트 (준비됨)
```bash
cd frontend

# 테스트 실행
npm test

# Watch 모드로 실행
npm test -- --watch

# 커버리지와 함께 실행
npm test -- --coverage
```

---

## 디버깅 및 문제 해결

### Docker 문제 해결

```bash
# 모든 컨테이너 상태 확인
docker-compose ps

# 특정 서비스 로그 확인
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f db

# 컨테이너 내부 접속
docker-compose exec backend bash
docker-compose exec frontend sh
docker-compose exec db psql -U weworkhere_user -d weworkhere_db

# 네트워크 확인
docker network inspect weworkhere_weworkhere_network

# 완전히 초기화 (데이터 삭제됨!)
docker-compose down -v
docker-compose up -d --build
```

### Database 직접 접속

```bash
# 호스트에서 접속
psql -h localhost -p 5443 -U weworkhere_user -d weworkhere_db

# 또는 Docker 컨테이너를 통해
docker-compose exec db psql -U weworkhere_user -d weworkhere_db

# 유용한 SQL 명령어
\dt                    # 테이블 목록
\d users               # users 테이블 구조
SELECT * FROM users;   # 사용자 목록
```

### Backend API 문서 접근

```bash
# FastAPI 자동 문서
http://localhost:25050/docs        # Swagger UI
http://localhost:25050/redoc       # ReDoc

# Health Check
http://localhost:25050/health
```

### 일반적인 문제

**1. 포트 충돌**
```bash
# 포트 사용 확인 (macOS)
lsof -i :24050
lsof -i :25050
lsof -i :5443

# 프로세스 종료
kill -9 <PID>
```

**2. Database 연결 실패**
- DATABASE_URL의 포트가 `db:5432`인지 확인 (5443이 아님!)
- 컨테이너 이름이 `db`인지 확인
- 컨테이너가 같은 네트워크에 있는지 확인

**3. Frontend에서 Backend 호출 실패**
- 브라우저 콘솔에서 네트워크 탭 확인
- CORS 에러인 경우 `ALLOWED_ORIGINS` 확인
- API URL이 `/api/v1`로 시작하는지 확인

**4. Session Token 문제**
- localStorage에 `session_token` 저장되었는지 확인
- Backend에서 `X-Session-Token` 헤더 받는지 확인
- 토큰 만료 시간 확인 (기본 720시간)

---

## 핵심 패턴 및 예제

### 1. Generic Repository Pattern (Backend)

**BaseRepository 구조**:
```python
from typing import TypeVar, Generic, Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

ModelType = TypeVar("ModelType")

class BaseRepository(Generic[ModelType]):
    def __init__(self, model: type[ModelType], db: AsyncSession):
        self.model = model
        self.db = db

    async def get_by_id(self, id: int) -> Optional[ModelType]:
        result = await self.db.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()

    async def get_all(
        self, skip: int = 0, limit: int = 100
    ) -> List[ModelType]:
        result = await self.db.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return result.scalars().all()

    async def create(self, obj: ModelType) -> ModelType:
        self.db.add(obj)
        await self.db.commit()
        await self.db.refresh(obj)
        return obj

    async def update(self, obj: ModelType) -> ModelType:
        await self.db.commit()
        await self.db.refresh(obj)
        return obj

    async def delete(self, id: int) -> bool:
        obj = await self.get_by_id(id)
        if obj:
            await self.db.delete(obj)
            await self.db.commit()
            return True
        return False
```

**사용 예제**:
```python
# repositories/post_repository.py
from app.repositories.base import BaseRepository
from app.models.post import Post

class PostRepository(BaseRepository[Post]):
    async def get_by_category(
        self, category_id: int, skip: int = 0, limit: int = 20
    ) -> List[Post]:
        result = await self.db.execute(
            select(Post)
            .where(Post.category_id == category_id)
            .offset(skip)
            .limit(limit)
        )
        return result.scalars().all()
```

### 2. API Client with Interceptors (Frontend)

**apiClient 설정**:
```typescript
// services/apiClient.ts
import axios, { AxiosInstance } from 'axios';
import { getSessionToken, clearSessionToken } from '@/utils/storage';

const API_URL = process.env.NEXT_PUBLIC_API_URL || '/api/v1';

const apiClient: AxiosInstance = axios.create({
  baseURL: API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor - 모든 요청에 세션 토큰 추가
apiClient.interceptors.request.use(
  (config) => {
    const token = getSessionToken();
    if (token) {
      config.headers['X-Session-Token'] = token;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor - 401 에러 시 로그아웃 처리
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      clearSessionToken();
      window.location.href = '/ko/login';
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

### 3. Hook + Service Pattern (Frontend)

**useAuth 구현**:
```typescript
// hooks/useAuth.ts
import { useState, useEffect } from 'react';
import authService from '@/services/authService';
import { User, AnonymousLoginRequest } from '@/types';
import { getSessionToken, setSessionToken, clearSessionToken } from '@/utils/storage';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    const token = getSessionToken();
    if (token) {
      try {
        const userData = await authService.getCurrentUser();
        setUser(userData);
      } catch (error) {
        clearSessionToken();
      }
    }
    setLoading(false);
  };

  const login = async (data: AnonymousLoginRequest) => {
    const response = await authService.createAnonymousUser(data);
    setSessionToken(response.session_token);
    setUser(response);
  };

  const logout = () => {
    clearSessionToken();
    setUser(null);
  };

  return { user, loading, login, logout };
}
```

**authService 구현**:
```typescript
// services/authService.ts
import apiClient from './apiClient';
import { User, AnonymousLoginRequest } from '@/types';

const authService = {
  async createAnonymousUser(data: AnonymousLoginRequest): Promise<User> {
    const response = await apiClient.post('/auth/anonymous', data);
    return response.data;
  },

  async getCurrentUser(): Promise<User> {
    const response = await apiClient.get('/auth/me');
    return response.data;
  },
};

export default authService;
```

### 4. Service Layer Pattern (Backend)

**PostService 구조**:
```python
# services/post_service.py
from typing import Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from app.repositories.post_repository import PostRepository
from app.models.post import Post
from app.schemas.post_schema import PostCreate, PostUpdate

class PostService:
    def __init__(self, db: AsyncSession):
        self.post_repo = PostRepository(Post, db)

    async def create_post(
        self, user_id: int, post_data: PostCreate
    ) -> Post:
        """게시글 생성"""
        post = Post(
            user_id=user_id,
            category_id=post_data.category_id,
            title=post_data.title,
            content=post_data.content,
            image_url=post_data.image_url,
        )
        return await self.post_repo.create(post)

    async def get_posts_paginated(
        self,
        page: int = 1,
        page_size: int = 20,
        category_id: Optional[int] = None
    ):
        """페이지네이션된 게시글 목록"""
        skip = (page - 1) * page_size

        if category_id:
            posts = await self.post_repo.get_by_category(
                category_id, skip, page_size
            )
        else:
            posts = await self.post_repo.get_all(skip, page_size)

        total = await self.post_repo.count()

        return {
            "posts": posts,
            "total": total,
            "page": page,
            "page_size": page_size,
            "total_pages": (total + page_size - 1) // page_size
        }
```

### 5. Dependency Injection Pattern (Backend)

**dependencies.py 구조**:
```python
# core/dependencies.py
from fastapi import Header, HTTPException, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import get_db
from app.models.user import User
from app.repositories.user_repository import UserRepository

async def get_current_user(
    x_session_token: str = Header(None, alias="X-Session-Token"),
    db: AsyncSession = Depends(get_db)
) -> User:
    """세션 토큰으로 현재 사용자 확인"""
    if not x_session_token:
        raise HTTPException(status_code=401, detail="Not authenticated")

    user_repo = UserRepository(User, db)
    user = await user_repo.get_by_session_token(x_session_token)

    if not user:
        raise HTTPException(status_code=401, detail="Invalid session token")

    return user
```

**Router에서 사용**:
```python
# routers/post_router.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import get_db
from app.core.dependencies import get_current_user
from app.models.user import User
from app.services.post_service import PostService
from app.schemas.post_schema import PostCreate, PostResponse

router = APIRouter(prefix="/posts", tags=["posts"])

@router.post("", response_model=PostResponse)
async def create_post(
    post_data: PostCreate,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    """게시글 작성"""
    post_service = PostService(db)
    post = await post_service.create_post(current_user.id, post_data)
    return post
```

---

## 중요 규칙

### 절대 하지 말아야 할 것 ❌
- 계층 건너뛰기 (예: Router에서 직접 Repository 호출)
- 하드코딩 (URL, 포트, 에러 메시지)
- 타입 힌트 누락
- 직접 SQL 작성 (ORM 사용)
- 환경 변수를 코드에 하드코딩

### 반드시 해야 할 것 ✅
- 모든 함수에 타입 힌트
- 계층 분리 엄격히 준수
- 환경 변수 사용
- 에러 처리
- 코드 재사용 최대화

---

## 관련 문서

- `README.md` - 프로젝트 개요
- `PLATFORM_REUSE_CATALOG.md` - 재사용 가능한 코드 카탈로그
- `backend/README.md` - Backend 상세 가이드
- `frontend/README.md` - Frontend 상세 가이드
- `backend/IMPLEMENTATION_SUMMARY.md` - Backend 구현 요약
- `frontend/IMPLEMENTATION_SUMMARY.md` - Frontend 구현 요약

---

## 변경 이력

### Version 1.1.0 (2026-01-08)
**추가됨**:
- Docker 네트워크 아키텍처 섹션 (컨테이너 간 통신)
- 완전한 .env 템플릿 및 환경 변수 설명
- 기존 컴포넌트 목록 (Frontend & Backend)
- 핵심 패턴 및 예제 섹션:
  - Generic Repository Pattern
  - API Client with Interceptors
  - Hook + Service Pattern
  - Service Layer Pattern
  - Dependency Injection Pattern
- 디버깅 및 문제 해결 가이드
- 테스트 명령어 상세 설명
- 데이터베이스 스키마에 인덱스 및 Relationships 추가

**개선됨**:
- 기술 스택에 버전 정보 추가
- 데이터베이스 모델에 상세 타입 및 제약조건 명시

### Version 1.0.0 (2026-01-08)
- 초기 버전

---

**Last Updated**: 2026-01-08
**Version**: 1.1.0
