<img width="100%" alt="미리보기" src="https://github.com/user-attachments/assets/ffce2d21-a05c-4ea9-9bc8-e1070dbb7e05" />


<br/>

# RatBox

## 1. 프로젝트 소개
- **한 줄 소개**: 냉장고 속 재료로 레시피와 대체재를 추천하고, 요리 중 음성 질문에도 실시간으로 답해주는 AI 요리 비서
- **주요 사용자**: 자취생·1인 가구, 그중에서도 "이 재료 없어도 되나?"를 스스로 판단하기 어려운 초보 자취러. 
- **프로젝트를 만들게 된 배경**: 자취생·1인 가구는 재료를 소량·비정형으로 보유해, 완전일치 검색이나 정형화된 필터 위주의 기존 레시피 서비스로는 "지금 가진 재료로 뭘 만들 수 있는지" 확인이 어려움. 요리 중에는 손에 물이나 재료가 묻어 텍스트 입력도 번거로워, 음성 인터페이스(STT) 기반 서비스가 적합하다고 판단해 기획함
- **최종 결과물의 형태**: React 기반 모바일 웹앱 + FastAPI/LangGraph 기반 백엔드로 구성된 AI 서비스.
- **접속 링크**: https://ratbox.cloud/

<br/>
<br/>

## 2. 문제 정의 

- **사용자가 겪는 불편함**: 냉장고에 재료가 애매하게 남아있을 때(대파 반 단, 계란 몇 개 등) 뭘 만들 수 있는지 매번 검색하기 번거로워, 결국 배달을 시키거나 재료를 버리게 됨. 요리 중 재료가 없다는 걸 알게 돼도, 손에 물이나 재료가 묻어 있어 검색이 애매해 생략하거나 조리를 중단해버림
- **기존 방식의 한계**: 기존 레시피 서비스는 재료 매칭까지만 제공하고, 부족 재료의 생략·대체 판단은 전적으로 사용자 몫으로 남김. 이 판단은 조리 전 한 번이 아니라 조리 중 상황이 바뀔 때도 반복 필요하지만, 기존 서비스는 조리 시작 이후의 대응이 전혀 없음
- **왜 이 문제가 중요한지**: 재료 매칭 이후의 생략·대체 판단을 대신해주는 것이 AI가 사용자를 대신할 수 있는 핵심 가치이자, 단순 검색과 AI 에이전트를 구분 짓는 지점임. 이 판단을 해결하면 재료가 부족해도, 조리 중 문제가 생겨도 요리를 끝까지 완성할 수 있음

## 3. 문제 해결
- **핵심 아이디어**: 요리비서 Agent '뚜이'가 선택된 재료 ID 목록으로 레시피 DB에서 매칭 후보를 검색하고, 부족 재료가 적은 순으로 후보 3개를 추천함. 사용자가 레시피를 선택하면 그 레시피에 한해 부족 재료의 생략 가능 여부와 대체재를 판단해 이유와 함께 설명하고, 조리 시작 후에는 음성 질의(STT)로 현재 조리 중인 레시피를 인식한 상태에서 실시간으로 대체재·생략 가능 여부를 안내함
- **AI/소프트웨어가 문제 해결에 사용된 방식**: LangGraph 기반으로 판단→도구 선택→재판단 루프를 그래프 노드로 명시적으로 관리함. 후보 추천 단계에서는 Text-to-SQL Tool(Upstage Solar Pro가 SQL 직접 생성, 화이트리스트 검증 후 실행)만 호출하고, 레시피 선택 이후에만 부족재료 분류·대체재 검색 Tool을 호출함. 최종 결과는 알레르기 하드 필터링을 거쳐 자연어로 변환돼 SSE로 스트리밍됨
- **전체 동작 흐름**:
  1. 재료 목록에서 보유 재료 선택 → 실시간 처리 상태 확인(SSE 스트리밍)
  2. 레시피 후보 3개 확인(요리명, 부족재료 목록) → 후보 중 하나 선택
  3. 상세보기 진입(전체 조리법 + 대체재 설명) → 조리 모드 시작
  4. 조리 진행 중 마이크 버튼으로 음성 질의(예: "간장 없는데 어떡하지?")
  5. 질문받은 재료의 대체안·생략 가능 여부를 짧은 텍스트로 즉시 확인, 요리 계속
  6. 조리 종료

<br/>
<br/>

# 4 .핵심 기능

- 재료·알레르기 ID 기반 레시피 후보 추천 (Text-to-SQL)
- 부족 재료 필수/생략 분류 및 대체재 제안
- 결과 검증 루프 (알레르기 등 제약 위반 여부 재확인)
- SSE 기반 실시간 스트리밍 응답
- 조리 중 음성 질의 처리 (Google Cloud STT + 실시간 재료 대체·생략 판단)
- 알레르기 유발 재료 자동 필터링 가드레일

<br/>
<br/>

# 5. 데모 UI

<table align="center" style="border-collapse: collapse; width: 100%; max-width: 1200px; margin: 20px auto; table-layout: fixed;">
  <tr>
    <td align="center" style="width: 50%; padding: 10px;">
      <img src="https://github.com/user-attachments/assets/1dec3c1a-8592-4e27-8c02-b2158f4cc6b3" alt="랜딩 페이지" style="width: 560px; max-width: 100%; height: 347px; object-fit: cover;">
      <p><strong>랜딩 페이지</strong></p>
    </td>
    <td align="center" style="width: 50%; padding: 10px;">
      <img src="https://github.com/user-attachments/assets/7d14284c-d1de-4f68-b267-237832beb570" alt="홈화면" style="width: 560px; max-width: 100%; height: 347px; object-fit: cover;"/>
      <p><strong>홈화면</strong></p>
    </td>
  </tr>
  <tr>
    <td align="center" style="width: 50%; padding: 10px;">
      <img src="https://github.com/user-attachments/assets/7f1ec5bc-732b-44fe-9644-e5af89f7f4db" alt="레시피 후보 추천" style="width: 560px; max-width: 100%; height: 510px; object-fit: cover;">
      <p><strong>재료선택</strong></p>
    </td>
    <td align="center" style="width: 50%; padding: 10px;">
      <img src="https://github.com/user-attachments/assets/420beff0-319f-4934-abbb-816090191fc2" alt="레시피 상세" style="width: 560px; max-width: 100%; height: 510px; object-fit: cover;"/>
      <p><strong>레시피 후보 추천</strong></p>
    </td>
  </tr>
  <tr>
    <td align="center" style="width: 50%; padding: 10px;">
      <img src="https://github.com/user-attachments/assets/1187bcd3-5c1c-44b1-884a-9baf1f9317d4" alt="조리 모드" style="width: 560px; max-width: 100%; height: 535px; object-fit: cover;"/>
      <p><strong>레시피 상세</strong></p>
    </td>
    <td align="center" style="width: 50%; padding: 10px;">
      <img src="https://github.com/user-attachments/assets/22804004-0944-402d-a512-3813af69760b" alt="음성 질의" style="width: 560px; max-width: 100%; height: 535px; object-fit: cover;"/>
      <p><strong>조리 모드</strong></p>
    </td>
  </tr>
</table>

<br/>
<br/>

## 6. 팀원 소개

| 이름 | 역할 | GitHub |
|---|---|---|
| 김다인 | Backend, AI | @kallin1 |
| 최서윤 | Frontend | @seoyunch |

<br/>
<br/>

## 7. Getting Started (실행 방법)

### Requirements
- Node.js 18+ / npm
- Python 3.12+
- Redis (로컬 설치 또는 Docker)
- 본인 명의로 발급받은 Supabase(PostgreSQL) 프로젝트, Upstage API Key, Google Cloud STT API Key
  (`.env`는 `.gitignore`에 포함돼 저장소에 올라가지 않으므로, 각자 발급받아 채워야 함)

### 1) 클론
```bash
git clone <repo-url>
cd project
```

### 2) Backend (RatBox-BE) 준비
```bash
cd RatBox-BE
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# .env에 SUPABASE_URL, SUPABASE_KEY, DATABASE_URL_READONLY, UPSTAGE_API_KEY, JWT_SECRET_KEY 등 입력
```

### 3) DB 스키마 적용
Supabase 프로젝트의 SQL Editor(또는 `psql "$DATABASE_URL_READONLY"`)에서 아래 마이그레이션을 **순서대로** 실행함
(`db/schema.sql` 상단 주석 기준 실제 운영 스키마에 필요한 파일만 나열):
```bash
db/migrations/0004_schema_sync.sql
db/migrations/0005_readonly_role.sql
db/migrations/0007_readonly_rls_policy.sql
db/migrations/0008_ingredients_category.sql
```
또는 `psql`이 설치돼 있다면 `db/schema.sql`을 그대로 실행해도 됨(내부에서 위 4개 파일을 `\i`로 순서대로 불러옴):
```bash
psql "$DATABASE_URL_READONLY" -f db/schema.sql
```

### 4) 초기 데이터 적재
스키마 적용 후, 레시피/재료/카테고리/알레르기 데이터를 순서대로 적재함:
```bash
# 1. 레시피·재료 마스터 데이터 적재 (allergen_master, ingredients_master, recipes, recipe_ingredients)
python scripts/load_csv.py data/TB_RECIPE_SEARCH_251231.csv

# 2. 재료 카테고리 적재 (0008 마이그레이션 적용 후 실행)
python scripts/load_ingredient_categories.py --apply

# 3. 재료-알레르기 매핑 백필
python scripts/backfill_ingredient_allergens.py --apply
```
> `--apply` 없이 실행하면 dry-run(미리보기)만 하고 실제로 반영하지 않음

### 5) Redis 실행
```bash
# 로컬에 설치된 경우
redis-server

# 또는 Docker로 Redis만 띄우는 경우
docker run -p 6379:6379 redis:7-alpine
```

### 6) Backend 실행
```bash
cd RatBox-BE
uvicorn app.main:app --reload
```
- 기본 주소: `http://localhost:8000`
- Health check: `GET /health` → `{"status": "ok"}`

### 7) Frontend (RatBox-FE)
```bash
cd RatBox-FE
npm install

cp .env.example .env
# .env에 VITE_API_URL(백엔드 주소), VITE_GOOGLE_STT_API_KEY 등 입력

npm run dev
```
- 기본 주소: `http://localhost:5173`

### 8) Docker로 백엔드+Redis 한 번에 실행 (선택, 2~4단계 완료 후)
```bash
cd RatBox-BE
docker compose up --build
```
- FastAPI(8000)와 Redis(6379) 컨테이너가 함께 실행됨 (DB 스키마·데이터 적재는 별도로 미리 해둬야 함)

<br/>
<br/>

## 8. Technology Stack (기술 스택)

### Frontend
![React](https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=TypeScript&logoColor=fff)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat&logo=vite&logoColor=white)

### Backend
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat&logo=langchain&logoColor=white)

### AI Integration
![Upstage](https://img.shields.io/badge/Upstage%20Solar%20Pro-6236FF?style=flat&logoColor=white)
![Google Cloud STT](https://img.shields.io/badge/Google%20Cloud%20STT-4285F4?style=flat&logo=googlecloud&logoColor=white)
![Langfuse](https://img.shields.io/badge/Langfuse-000000?style=flat&logoColor=white)

### Infra
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![GCP](https://img.shields.io/badge/Google%20Cloud-4285F4?style=flat&logo=googlecloud&logoColor=white)

### DB
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=flat&logo=supabase&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat&logo=redis&logoColor=white)

