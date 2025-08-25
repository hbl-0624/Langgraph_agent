Agent Service Toolkit - 部署与使用指南
基于 LangGraph、FastAPI 和 Streamlit 构建的 AI 代理服务工具包，支持多类型代理（聊天、研究、RAG 等）、流式对话、多轮会话管理与 LangSmith 反馈跟踪，可快速搭建企业级 AI 代理应用。

项目架构如下：
后端层：FastAPI 提供 RESTful API 与 SSE 流式接口，LangGraph 实现代理逻辑编排，支持工具调用（搜索、计算、数据库查询）。
前端层：Streamlit 构建轻量交互式界面，支持代理切换、模型选择、会话分享与反馈提交。
依赖层：Python 虚拟环境隔离依赖，.env 统一管理配置，适配多类 LLM（OpenAI、Anthropic、Azure 等）。
环境要求
依赖项	版本要求	说明
Python	3.11+（推荐 3.11.9）	低版本不兼容 langchain-core 0.3.x 等依赖
网络环境	无 VPN / 代理干扰	确保可访问 PyPI 源与 LLM 接口（如 OpenAI）！！！一点别忘了关闭科学上网！！！
依赖管理工具	pip 23.0+	用于安装项目依赖

快速部署步骤
1. 克隆项目代码
bash
git clone https://github.com:hbl-0624/Langgraph_agent.git
cd Langgraph_agent

2. 安装 Python 3.11+
若系统未安装 Python 3.11，需先完成安装：
访问 Python 官网下载页，选择 Python 3.11.x（如 3.11.9）。
安装时务必勾选 "Add Python to PATH"（关键！确保后续可通过命令行调用 Python）。
验证安装：打开新终端执行以下命令，显示 Python 3.11.x 即成功：
bash
python --version

3. 创建并激活虚拟环境
虚拟环境可隔离项目依赖，避免与系统 Python 环境冲突：

bash
# 1. 删除旧环境（若存在，避免版本残留）
rm -Recurse -Force .venv 2>$null

# 2. 用 Python 3.11 创建新虚拟环境
py -3.11 -m venv .venv

# 3. 激活虚拟环境（Windows PowerShell 命令）
.venv\Scripts\activate

# 4. 验证环境：命令行前缀显示 ".venv"，且 Python 版本为 3.11.x
python --version

4. 配置环境变量（.env 文件）
在项目根目录创建 / 修改 .env 文件，填入核心配置如下（其他可选配置可参考模板注释）：

env
# ===================== 1. LLM 配置（必选）=====================
# OpenAI 及兼容接口配置（示例：第三方兼容接口）
OPENAI_API_KEY=sk-xxx  # 替换为你的 API 密钥
OPENAI_API_BASE=  # 接口地址（官方/第三方均可）

# 其他 LLM 配置（按需启用）
# AZURE_OPENAI_API_KEY=
# ANTHROPIC_API_KEY=
# GOOGLE_API_KEY=

# ===================== 2. 服务配置（必选）=====================
# 后端服务绑定地址（0.0.0.0 支持局域网访问，前端需用 localhost）
HOST=0.0.0.0
PORT=8081  # 避免 8080 端口冲突，可修改为 8082/8888 等

# 前端连接后端的地址（关键！必须用 localhost，不可用 0.0.0.0）
AGENT_URL=http://localhost:8081

# ===================== 3. 数据库配置（可选）=====================
# 推荐使用 SQLite（无需额外安装，轻量易用）
DATABASE_TYPE=sqlite
SQLITE_DB_PATH=./agent.db  # 数据库文件路径（自动生成）

# PostgreSQL 配置（按需启用，需提前部署数据库）
# POSTGRES_USER=
# POSTGRES_PASSWORD=
# POSTGRES_HOST=
# POSTGRES_PORT=
# POSTGRES_DB=

# ===================== 4. 调试配置（可选）=====================
# 启用假模型（跳过真实 LLM 调用，用于测试服务连通性）
# USE_FAKE_MODEL=true

# LangSmith 跟踪配置（按需启用，用于调试代理流程）
# LANGSMITH_TRACING=true
# LANGSMITH_API_KEY=
# LANGSMITH_PROJECT=agent-service-toolkit

5. 安装项目依赖
bash
# 1. 升级 pip 到最新版本（避免依赖安装失败）
python -m pip install --upgrade pip -i https://pypi.org/simple/

# 2. 安装核心依赖（从 pyproject.toml 读取，包含 FastAPI/Streamlit/LangGraph 等）
pip install . -i https://pypi.org/simple/

# 3. 手动安装缺失依赖（若上述命令报错，示例）
# pip install langchain-core==0.3.74 langgraph==0.6.5 duckduckgo-search==7.3.0 -i https://pypi.org/simple/

6. 启动服务（分两步：后端 + 前端）
需打开 两个终端窗口，分别启动后端与前端（均需保持虚拟环境激活）。
步骤 6.1 启动 FastAPI 后端
bash
# 在项目根目录执行（确保虚拟环境已激活）
python src/run_service.py

启动成功标识：终端显示以下日志（确认端口与 .env 配置一致）：

plaintext
INFO:     Started server process [12345]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8081 (Press CTRL+C to quit)

后端 API 文档（调试用）：
交互式文档（可直接测试接口）：http://localhost:8081/docs
静态文档（接口详情）：http://localhost:8081/redoc
验证后端健康：浏览器访问 http://localhost:8081/info，应返回包含 agents（代理列表）和 models（模型列表）的 JSON 数据，示例：
json
{"agents":[{"key":"chatbot","description":"A simple chatbot."},{"key":"research-assistant","description":"A research assistant with web search and calculator."}],"models":["gpt-4o","gpt-4o-mini"],"default_agent":"research-assistant","default_model":"gpt-4o-mini"}

步骤 6.2 启动 Streamlit 前端
打开新的终端窗口，执行以下命令：

bash
# 1. 进入项目根目录
cd agent-service-toolkit

# 2. 激活虚拟环境（若已关闭）
.venv\Scripts\activate

# 3. 启动前端
streamlit run src/streamlit_app.py

启动成功标识：

终端显示 You can now view your Streamlit app in your browser，并给出访问地址（默认 http://localhost:8501）。
浏览器自动打开前端界面（若未自动打开，手动复制 Local URL 到浏览器）。
