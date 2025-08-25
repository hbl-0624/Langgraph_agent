基于 LangGraph、FastAPI 和 Streamlit 构建的 AI 代理服务工具包，支持多类型代理（聊天、研究、RAG 等）、流式对话、多轮会话管理与 LangSmith 反馈跟踪，可快速搭建企业级 AI 代理应用。
一、项目架构
层级	核心组件与能力
后端层	FastAPI 提供 RESTful API 与 SSE 流式接口；
LangGraph 实现代理逻辑编排；
支持工具调用（网页搜索、计算器、数据库查询）。
前端层	Streamlit 构建轻量交互式界面；
支持代理切换、模型选择、会话分享与反馈提交。
依赖层	Python 虚拟环境隔离依赖；
.env 文件统一管理配置；
适配多类 LLM（OpenAI、Anthropic、Azure 等）。
二、环境要求
依赖项	版本要求	说明
Python	3.11+（推荐 3.11.9）	低版本不兼容 langchain-core 0.3.x 等核心依赖
网络环境	无 VPN / 代理干扰	关键！ 需关闭科学上网，确保可访问 PyPI 源与 LLM 接口（如 OpenAI）
依赖管理工具	pip 23.0+	用于安装项目依赖，避免版本兼容问题
三、快速部署步骤
1. 克隆项目代码
bash
# 克隆 GitHub 仓库（替换为实际仓库地址）
git clone https://github.com/hbl-0624/Langgraph_agent.git

# 进入项目根目录
cd Langgraph_agent
2. 安装 Python 3.11+
若系统未安装 Python 3.11，按以下步骤操作：

访问 Python 官网下载页，选择 Python 3.11.x（推荐 3.11.9）。
安装时务必勾选 "Add Python to PATH"（关键！确保命令行可调用 Python）。
验证安装：打开新终端执行命令，显示 Python 3.11.x 即成功：
bash
python --version

3. 创建并激活虚拟环境
虚拟环境可隔离项目依赖，避免与系统 Python 冲突：

bash
# 1. 删除旧环境（若存在，Windows PowerShell 命令）
rm -Recurse -Force .venv 2>$null

# 2. 用 Python 3.11 创建新虚拟环境
py -3.11 -m venv .venv

# 3. 激活虚拟环境（激活后命令行前缀显示 .venv）
.venv\Scripts\activate

# 4. 验证环境：确认 Python 版本为 3.11.x
python --version
4. 配置环境变量（.env 文件）
在项目根目录创建 / 修改 .env 文件，填入核心配置（可选配置已注释，按需启用）：

env
# ===================== 1. LLM 配置（必选）=====================
# OpenAI 及兼容接口配置（示例：第三方兼容接口）
OPENAI_API_KEY=sk-xxx  # 替换为你的有效 API 密钥
OPENAI_API_BASE=https://api.chatanywhere.tech/v1  # 接口地址（官方/第三方均可）

# 其他 LLM 配置（按需启用，删除 # 生效）
# AZURE_OPENAI_API_KEY=
# ANTHROPIC_API_KEY=
# GOOGLE_API_KEY=

# ===================== 2. 服务配置（必选）=====================
# 后端绑定地址（0.0.0.0 支持局域网访问，前端需用 localhost）
HOST=0.0.0.0
PORT=8081  # 避免 8080 端口冲突，可修改为 8082/8888 等

# 前端连接后端地址（关键！必须用 localhost，不可用 0.0.0.0）
AGENT_URL=http://localhost:8081

# ===================== 3. 数据库配置（可选）=====================
# 推荐 SQLite（无需额外安装，轻量易用）
DATABASE_TYPE=sqlite
SQLITE_DB_PATH=./agent.db  # 数据库文件自动生成

# PostgreSQL 配置（按需启用，需提前部署数据库）
# POSTGRES_USER=your_username
# POSTGRES_PASSWORD=your_password
# POSTGRES_HOST=localhost
# POSTGRES_PORT=5432
# POSTGRES_DB=agent_db

# ===================== 4. 调试配置（可选）=====================
# 启用假模型（跳过真实 LLM，用于测试服务连通性）
# USE_FAKE_MODEL=true

# LangSmith 跟踪配置（按需启用，用于调试代理流程）
# LANGSMITH_TRACING=true
# LANGSMITH_API_KEY=your_langsmith_key
# LANGSMITH_PROJECT=agent-service-toolkit
5. 安装项目依赖
bash
# 1. 升级 pip 到最新版本（避免依赖安装失败）
python -m pip install --upgrade pip -i https://pypi.org/simple/

# 2. 安装核心依赖（从 pyproject.toml 读取）
pip install . -i https://pypi.org/simple/

# 3. 手动安装缺失依赖（若上述命令报错，示例）
# pip install langchain-core==0.3.74 langgraph==0.6.5 duckduckgo-search==7.3.0 -i https://pypi.org/simple/
6. 启动服务（分两步：后端 + 前端）
需打开 两个独立终端窗口，均保持虚拟环境激活，分别启动后端与前端。
步骤 6.1 启动 FastAPI 后端
bash
# 在项目根目录执行
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
验证后端健康：浏览器访问 http://localhost:8081/info，应返回包含 agents 和 models 的 JSON 数据，示例：
json
{
  "agents": [
    {"key":"chatbot","description":"A simple chatbot."},
    {"key":"research-assistant","description":"A research assistant with web search and calculator."}
  ],
  "models": ["gpt-4o","gpt-4o-mini"],
  "default_agent": "research-assistant",
  "default_model": "gpt-4o-mini"
}

步骤 6.2 启动 Streamlit 前端
打开新终端窗口，执行以下命令：

bash
# 1. 进入项目根目录（若终端已关闭）
cd Langgraph_agent

# 2. 激活虚拟环境（若已关闭）
.venv\Scripts\activate

# 3. 启动前端
streamlit run src/streamlit_app.py

启动成功标识：

终端显示：
plaintext
You can now view your Streamlit app in your browser.
Local URL: http://localhost:8501
Network URL: http://192.168.1.2:8501

浏览器自动打开前端界面（未自动打开则手动复制 Local URL 访问）。
四、核心功能使用指南
1. 界面导航（左侧边栏）
功能模块	操作说明
New Chat	新建对话（清空当前会话历史，生成新 thread_id）
Settings	配置中心：
- 选择代理类型（如 research-assistant 带搜索功能）
- 选择 LLM 模型（如 gpt-4o-mini）
- 启用 / 禁用流式输出
Architecture	查看项目架构图（含技术栈与服务交互流程）
Share/resume chat	生成对话分享链接（含 thread_id，支持跨设备续聊）
Privacy	查看隐私政策（对话数据存储与 LangSmith 跟踪说明）
View the source code	跳转 GitHub 仓库查看源码
2. 常用代理功能对比
代理类型	核心能力	适用场景	依赖工具 / 服务
chatbot	纯文本对话（无工具调用）	简单问答、闲聊、基础知识科普	无（仅依赖 LLM）
research-assistant	网页搜索 + 计算器 + 信息整理	实时信息查询、数据计算、趋势分析	DuckDuckGo Search、Python 计算器
rag-assistant	私有数据库检索（RAG）	基于内部文档的问答（如公司手册、产品文档）	SQLite/PostgreSQL 数据库
command-agent	执行预设命令（自动化操作）	批量处理、文件生成、第三方服务调用	需提前配置命令列表
五、常见问题排查
1. 端口冲突（后端启动报错）
错误信息：[Errno 10048] 通常每个套接字地址只允许使用一次
解决方案：修改 .env 中的 PORT 为未占用端口（如 8082），重启后端：python src/run_service.py。
2. 前端 502 Bad Gateway 错误
错误信息：Error connecting to agent service at http://localhost:8081
排查步骤：
关闭 VPN / 代理，确保网络正常；
浏览器访问 http://localhost:8081/info，确认后端是否返回 JSON；
检查 .env 中 AGENT_URL 是否为 http://localhost:8081（不可用 0.0.0.0）。
3. LLM 调用失败
错误信息：Could not connect to OpenAI API 或 Invalid API key
解决方案：
验证 OPENAI_API_KEY 是否有效（替换为新密钥）；
检查 OPENAI_API_BASE 是否可访问；
临时启用 USE_FAKE_MODEL=true（.env 中），测试后端是否正常。
4. 依赖缺失（ModuleNotFoundError）
错误信息：No module named 'xxx'
解决方案：执行 pip install xxx -i https://pypi.org/simple/ 安装缺失包，重启服务。
