WeWorkHere
외국인 노동자를 위한 익명 커뮤니티 플랫폼

📖 프로젝트 개요
WeWorkHere는 한국에서 근무하는 외국인 노동자들이 안전하게 경험과 정보를 공유할 수 있는 익명 커뮤니티 플랫폼입니다!

🎯 목적
외국인 노동자들이 안전하게 경험·정보를 공유할 수 있는 공간 제공
브로커·업체·관리자 중심이 아닌 당사자 중심 커뮤니티 구축
언어·국적·사업장 차이를 넘는 실질적인 생존 정보 교환
👥 타겟 사용자
한국에서 근무 중인 외국인 노동자 (제조업, 농축산, 건설, 서비스업)
모바일 중심 사용자 (Android 스마트폰 비중 높음)
익명성이 중요한 사용자
🛠️ 기술 스택
Frontend
Framework: Next.js 14 (App Router)
Language: TypeScript (strict mode)
Styling: Tailwind CSS 3.4
HTTP Client: Axios 1.6
i18n: Custom dictionary context
Backend
Framework: FastAPI 0.109
Language: Python 3.11
ORM: SQLAlchemy 2.0 (Async)
Database: PostgreSQL 15-alpine
Server: Uvicorn
Migration: Alembic
Validation: Pydantic 2.x
Deployment
Container: Docker Compose
Server: Mac Mini (macOS)
Ports:
Frontend: 24050
Backend: 25050
Database: 5443
🌍 다국어 지원
🇰🇷 한국어 (Korean)
🇬🇧 영어 (English)
🇻🇳 베트남어 (Vietnamese)
🇳🇵 네팔어 (Nepali)
✨ 핵심 기능 (MVP)
커뮤니티 기능
✅ 게시글 작성/수정/삭제 (텍스트 + 사진)
✅ 댓글 작성 및 삭제
✅ 좋아요/공감 버튼
✅ 카테고리별 필터링
임금/급여
숙소
사업장 문제
계약/비자
자유 이야기
인증 시스템
✅ 익명 닉네임 기반 로그인 (회원가입 선택적)
✅ 세션 토큰 기반 인증 (localStorage)
✅ 전화번호 인증 없음
✅ 익명성 보장
안전 기능
✅ 신고 기능 (준비됨)
✅ 관리자 개입 최소화
✅ 위치 정보 수집 없음
