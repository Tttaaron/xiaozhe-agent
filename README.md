# 小哲电商客服 Agent · 终版提炼项目

以课程 **第 41 课（final-rehearsal）** 为核心的独立可运行项目。已去掉其余 45 课快照，只保留跑通全链路所需内容。

## 目录结构

```
xiaozhe-final-project/
├── agent/                      # Agent 后端（原 lesson-41-final-rehearsal/backend）
│   ├── main.py                 # 启动入口，:8000
│   ├── api/ agents/ rag/ tools/ workflows/ ...
│   ├── knowledge/              # RAG 知识库（Markdown 政策）
│   └── cases.yml               # 回归评测用例
├── ecommerce-backend/          # Spring Boot 电商业务后端 :8081（Docker 构建）
├── frontend/                   # Agent 教学调试台 :5173
├── doc/                        # 原课程文档（运行手册等，路径以本项目为准）
├── requirements.txt            # Agent Python 依赖
├── docker-compose.infra.yml    # MySQL + ecommerce-service
├── .env                        # Compose 用的演示变量
├── course.env                  # 模型配置（含 API Key，请勿外传）
└── course.env.example          # 模型配置模板
```

## 环境要求

- Docker Desktop
- Python 3.13+（建议在本项目根目录建 `.venv`）
- Node.js 20.19+ / 22.12+（仅调试台需要）

## 快速启动

### 1. 模型 Key

编辑本目录 `course.env`，确认 `AGENT_OPENAI_API_KEY` 为真实 Key（不要提交到 Git / 截图分享）。

### 2. Python 环境（首次）

```powershell
cd <本项目根目录>
py -3.13 -m venv .venv
.venv\Scripts\python -m pip install -r requirements.txt
```

### 3. 电商后端（业务事实源）

本机若无 `docker compose` 子命令，请用 `docker-compose`：

```powershell
cd <本项目根目录>
docker-compose -f docker-compose.infra.yml up -d --build mysql ecommerce-service
```

容器名：`xz-final-mysql`、`xz-final-ecommerce`（避免与旧课程仓 `demo-mysql` 冲突）。

健康检查：`http://localhost:8081/actuator/health`  
商城：`http://localhost:8081/` · 管理后台：`http://localhost:8081/admin/`

### 4. Agent 后端

```powershell
cd agent
..\\.venv\\Scripts\\python.exe main.py
```

健康检查：`http://localhost:8000/health` · API 文档：`http://localhost:8000/docs`

### 5. 调试台（可选）

```powershell
cd frontend
npm install
npm run dev
```

打开：`http://localhost:5173`  
若 Agent 不在 8000，在 `frontend/.env.local` 设置 `VITE_AGENT_BASE_URL`。

## 主要端点

| 端点 | 说明 |
|---|---|
| `GET /health` | 健康检查 |
| `GET /capabilities` | 能力声明（调试台点亮 UI） |
| `POST /chat` | 对话主入口 |
| `POST /chat/resume` | HITL 审批恢复 |
| `GET /sessions/{id}/trace` | 公开 Trace |
| `POST /eval/run` | 回归评测 |
| `POST /feedback/submit` | 失败反馈/归因 |

## 演示账号

| 账号 | 密码 | 角色 |
|---|---|---|
| zhangsan | 123456 | 用户 |
| lisi | 123456 | 用户 |
| wangwu | 123456 | 用户 |
| admin | admin123456 | 管理员 |

## 与原课程仓的差异

- 删除 `agent-course-versions/` 下除第 41 课外的课次快照
- `agent/` 即原 `lesson-41-final-rehearsal/backend/`
- `course.env` 从 `agent-course-versions/` 移到项目根；`agent/config/settings.py` 的默认路径已改为 `parents[2]`（项目根）
- 其余业务后端、调试台、Compose、文档与发布包一致

## 说明

- 高风险退款只走资格判断 + 人工审批，不执行真实资金退款
- 完整 system prompt、hidden CoT 不会通过 Trace 对外暴露
- 详细排错见 `doc/运行手册.md`
