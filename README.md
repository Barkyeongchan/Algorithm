# 알고리즘(Algorithm)
어떤 문제를 해결하기 위한 절차나 방법을 명확하게 정의한 단계적인 명령어들의 집합

## 교재 목차 지도
<img width="1152" height="768" alt="image" src="https://github.com/user-attachments/assets/4393ccf3-4568-4afc-ba38-38085a782576" />


## 1. 정렬(Sort)
데이터를 일정한 기준에 따라 순서대로(오름차순 또는 내림차순) 나열하는 작업

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2a99c23a-d34f-4dd8-914d-8c90090b27a2" />

## 2. 슬라이딩 윈도우(Sliding Window)
데이터의 특정 범위를 정해놓고 정해진 크기만큼 이동시켜 값을 구하는 알고리즘

<img width="1983" height="1216" alt="image" src="https://github.com/user-attachments/assets/bf1f6383-c375-41bd-a3d7-6c1e4cdb002d" />

## 3. 그리디 알고리즘(Greedy Algorithm)
현재 상황에서 최적의 선택을 반복하여 최종 결과값을 구하는 알고리즘

<img width="759" height="509" alt="image" src="https://github.com/user-attachments/assets/43592d5c-c025-48cf-9fba-c5c41fa8edad" />

## 4. Python3 Turtle 로봇 알고리즘
python3의 turtle을 활용하여 로봇 제어 알고리즘을 익힌다.

<img width="428" height="372" alt="image" src="https://github.com/user-attachments/assets/58513359-4048-4622-b02c-9015ad6e9b0c" />


🚀 FastAPI 서버 구축 및 ROS → 로컬 → EC2 → 웹 클라이언트 실시간 연결

기존 Spring Boot WMS 서버를 FastAPI 기반으로 완전히 이식한 버전입니다.
AWS EC2에 배포하여 ROS 데이터를 실시간으로 웹 브라우저에 표시하는 구조로 설계되었습니다.

📌 개요
🧩 전체 흐름
ROS → 로컬(roslibpy) → EC2(FastAPI /ws) → WebClient(JavaScript)

🧱 FastAPI 주요 특징

🧠 Python 기반의 비동기 웹 프레임워크

⚡ Spring Boot보다 가볍고 빠른 WebSocket 지원

🧾 Swagger를 통한 API 자동 문서화

🗄️ SQLAlchemy ORM 기반의 DB 연동

📖 개발 단계 요약
단계	설명
1️⃣	환경 세팅 및 기본 구조 생성
2️⃣	DB 모델 (엔티티) 작성
3️⃣	CRUD & 서비스 계층 구성
4️⃣	API 라우터 구축
5️⃣	템플릿 및 정적 리소스 설정
6️⃣	WebSocket 서버 구축
7️⃣	웹 클라이언트에서 실시간 데이터 표시
1️⃣ 환경 세팅

목표: FastAPI 서버의 기본 구조를 잡고, 환경변수를 분리하여 RDS(MySQL)와 연결 가능한 상태로 세팅한다.

📁 디렉토리 구조
WMS_FastApi/
├── .env
├── .gitignore
├── requirements.txt
└── app/
    ├── main.py
    ├── core/
    │   ├── config.py
    │   └── database.py
    ├── models/
    ├── routers/
    ├── crud/
    ├── services/
    ├── schemas/
    ├── templates/
    └── static/

⚙️ 가상환경 및 패키지 설치
python3 -m venv wms
source wms/bin/activate
pip install fastapi uvicorn[standard] sqlalchemy pydantic==1.10.24 python-dotenv mysqlclient jinja2


✅ pydantic==1.10.24: FastAPI 2.x 버전과의 호환성 유지용
✅ python-dotenv: .env 환경변수 로드
✅ mysqlclient: RDS(MySQL) 연결용 드라이버

🧾 .env — 환경변수 파일
DB_USER=root
DB_PASSWORD=password
DB_HOST=my-rds-host.amazonaws.com
DB_PORT=3306
DB_NAME=wms
DEBUG=True


✅ .env를 사용하면 DB 계정 정보가 코드에 직접 노출되지 않음

🧩 config.py — 환경 변수 로딩
# app/core/config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    DB_USER: str
    DB_PASSWORD: str
    DB_HOST: str
    DB_PORT: int
    DB_NAME: str
    DEBUG: bool = True

    class Config:
        env_file = ".env"  # .env 파일 자동 로드

settings = Settings()


✅ .env에 있는 값을 자동으로 불러와 settings.DB_USER 식으로 접근 가능
✅ Spring Boot의 application.properties 역할

🧠 database.py — DB 연결 구성
# app/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base
from app.core.config import settings

DB_URL = (
    f"mysql+mysqlclient://{settings.DB_USER}:{settings.DB_PASSWORD}"
    f"@{settings.DB_HOST}:{settings.DB_PORT}/{settings.DB_NAME}"
)

engine = create_engine(DB_URL, pool_pre_ping=True)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


✅ pool_pre_ping=True → DB 연결 끊김 시 자동 복구
✅ Base → 모든 ORM 모델의 부모 클래스 (Spring의 @Entity 역할)

🖥️ main.py — FastAPI 서버 기본 실행
from fastapi import FastAPI
from app.core.config import settings

app = FastAPI(title="WMS FastAPI Server", debug=settings.DEBUG)

@app.get("/")
def root():
    return {"message": "FastAPI 서버 실행 중"}


실행 명령어:

uvicorn app.main:app --reload


✅ 브라우저에서 http://127.0.0.1:8000
 접속 시
{"message": "FastAPI 서버 실행 중"} 출력 → 정상 실행 확인

2️⃣ DB 모델 작성

목표: Spring Boot의 @Entity를 FastAPI의 SQLAlchemy 모델로 변환

🧩 예시: app/models/log.py
from sqlalchemy import Column, Integer, String, BigInteger, DateTime
from app.core.database import Base

class Log(Base):
    __tablename__ = "log"

    id = Column(BigInteger, primary_key=True, autoincrement=True)
    robot_name = Column(String, nullable=False)
    pin_name = Column(String, nullable=False)
    product_name = Column(String, nullable=False)
    quantity = Column(Integer, nullable=False)
    action = Column(String, nullable=False)
    timestamp = Column(DateTime, nullable=False)


✅ Spring의 @Entity + @Column 문법을 Python 방식으로 변환
✅ nullable=False → 필수 입력 필드

🔍 DB 생성 확인
from app.core.database import Base, engine
from app.models.log import Log

Base.metadata.create_all(bind=engine)


✅ RDS(MySQL)에 log 테이블 자동 생성됨

3️⃣ CRUD & 서비스 계층

목표: ORM 기반으로 데이터 추가, 수정, 삭제, 조회 기능 구현

🧩 app/crud/log_crud.py
from sqlalchemy.orm import Session
from app.models.log import Log

def create_log(db: Session, data: dict):
    log = Log(**data)
    db.add(log)
    db.commit()
    db.refresh(log)
    return log

def get_logs(db: Session):
    return db.query(Log).all()


✅ commit() → 실제 DB 반영
✅ refresh() → DB 반영 후 최신 상태로 갱신

⚙️ app/services/log_service.py
from sqlalchemy.orm import Session
from app.crud import log_crud

class LogService:
    def __init__(self, db: Session):
        self.db = db

    def list_logs(self):
        return log_crud.get_logs(self.db)


✅ Spring의 @Service 역할 — CRUD 호출 로직 분리

4️⃣ API 라우터 구축

목표: CRUD 기능을 RESTful API 형태로 외부에 제공

🧩 app/routers/log_router.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from app.core.database import SessionLocal
from app.crud import log_crud

router = APIRouter(prefix="/logs", tags=["Logs"])

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@router.get("/")
def read_logs(db: Session = Depends(get_db)):
    return log_crud.get_logs(db)


✅ @router.get() → Spring의 @GetMapping 대응
✅ Depends(get_db) → DB 세션 자동 주입

🧠 main.py 라우터 등록
from app.routers import log_router
app.include_router(log_router.router)


✅ 라우터를 통해 엔드포인트를 모듈 단위로 관리
✅ Swagger 자동 문서화 활성화 → http://127.0.0.1:8000/docs

5️⃣ 정적 리소스 & 템플릿

목표: HTML + CSS + JS 기반 대시보드 구성

📁 구조
app/
├── templates/
│   └── index.html
└── static/
    ├── css/style.css
    ├── js/main.js
    └── images/

🧩 index.html (요약)
<body>
  <main>
    <div class="log">
      <p class="log_t">작업 내역</p>
      <p class="log_text">작업 내역 나오는 곳</p>
    </div>
  </main>
  <script src="/static/js/main.js"></script>
</body>

⚙️ main.py 수정
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles
from fastapi.responses import HTMLResponse

app.mount("/static", StaticFiles(directory="app/static"), name="static")
templates = Jinja2Templates(directory="app/templates")

@app.get("/", response_class=HTMLResponse)
async def root(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})


✅ StaticFiles: /static 경로로 CSS/JS 제공
✅ TemplateResponse: HTML 템플릿 렌더링

6️⃣ WebSocket 구축 (로컬 ↔ EC2)

목표: ROS 데이터를 실시간으로 수신하고 클라이언트에 브로드캐스트

🧩 main.py (WebSocket 추가)
from fastapi import WebSocket, WebSocketDisconnect
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

active_connections = []

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    active_connections.append(websocket)
    print("[EC2] WebSocket connected")

    try:
        while True:
            data = await websocket.receive_text()
            for conn in list(active_connections):
                try:
                    await conn.send_text(data)
                except:
                    active_connections.remove(conn)
    except WebSocketDisconnect:
        active_connections.remove(websocket)


✅ 신규 기능

active_connections: 연결된 클라이언트 관리

WebSocketDisconnect: 연결 끊김 자동 정리

7️⃣ 웹 클라이언트 실시간 데이터 표시

목표: WebSocket을 통해 받은 ROS 데이터를 실시간으로 HTML에 표시

🧩 static/js/main.js
const ws = new WebSocket("ws://" + window.location.host + "/ws");

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    document.querySelector(".log_text").innerText =
        `X: ${data.x}, Y: ${data.y}, θ: ${data.theta}`;
};


✅ window.location.host → 현재 접속 서버 주소 자동 인식
✅ onmessage → 데이터 수신 시마다 화면 업데이트

✅ 실행 순서
단계	명령어	설명
1	uvicorn app.main:app --host 0.0.0.0 --port 8000	EC2 서버 실행
2	python ws_client.py	로컬 ROS 데이터 전송
3	브라우저 접속: http://<EC2_IP>:8000	대시보드 확인
4	WebSocket 로그	[CLIENT] Received: 메시지 출력
⚡ 주요 변경 요약
위치	추가된 기능	설명
config.py	.env 로드	Spring application.properties 대체
database.py	SessionLocal, Base 구성	ORM 기반 DB 연결
main.py	CORS + WebSocket 추가	외부 접속 및 실시간 통신 가능
/routers	APIRouter 관리	컨트롤러 역할
/static/js/main.js	실시간 데이터 표시	ROS → 웹 브라우저 출력
🎯 완성 결과

✅ EC2 서버에서 FastAPI 정상 구동
✅ RDS 연동 및 CRUD API 작동
✅ ROS → 로컬 → EC2 → 웹 실시간 데이터 송수신
✅ Swagger UI 자동 생성 (/docs)
✅ HTML + JS 기반 대시보드에서 실시간 표시
