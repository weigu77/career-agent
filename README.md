# 🧭 大学生职业规划智能体

> 基于大语言模型和真实招聘数据的个性化职业规划助手

[![Python Version](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.136+-green.svg)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-19.2+-cyan.svg)](https://react.dev/)
[![LangChain](https://img.shields.io/badge/LangChain-1.2+-orange.svg)](https://www.langchain.com/)
[![Milvus](https://img.shields.io/badge/Milvus-2.6+-purple.svg)](https://milvus.io/)

---

## 📖 项目简介

**大学生职业规划智能体** 是一个结合 LangGraph、RAG 和真实招聘数据的智能对话系统。它能够：

- 🔍 **岗位匹配**：基于 1 万+ 真实招聘信息，为用户推荐最匹配的岗位
- 📊 **技能差距分析**：对比用户现有技能与岗位要求，清晰指出提升方向
- 📅 **个性化学习规划**：根据用户时间约束自动生成周度学习计划（含课程推荐）
- 💬 **多轮对话 & 记忆**：长期保存用户画像，支持追问和迭代优化
- 📎 **简历解析**：上传 PDF/Word 简历，自动提取技能、专业、项目经历

本项目采用 **FastAPI + LangGraph + React + Milvus** 技术栈，完全满足课程设计、创新创业项目或毕业设计的技术深度和完整性要求。

---

## 🛠️ 技术栈

| 类别        | 技术                  | 版本             |
| ----------- | --------------------- | ---------------- |
| 后端语言    | Python                | 3.11.9           |
| Web 框架    | FastAPI               | 0.136.1+         |
| Agent 框架  | LangChain + LangGraph | 1.2.14+ / 1.1.3+ |
| 向量数据库  | Milvus (Lite)         | 2.6.11+          |
| 关系数据库  | PostgreSQL + pgvector | 17+              |
| 缓存 & 记忆 | Redis                 | 8.4.1+           |
| 前端框架    | React                 | 19.2.5           |
| 构建工具    | Vite                  | 8.0.8+           |
| 状态管理    | Zustand               | 5.0.11+          |
| 文档解析    | pypdf / python-docx   | 6.6.0+ / 1.2.0+  |
| 容器编排    | Docker Compose        | 5.1.1+           |

详细版本说明和选型理由见 [技术架构文档](./docs/ARCHITECTURE.md)。

---

## 🚀 快速开始（5 分钟跑通 Demo）

### 前置要求
- Python 3.11
- Node.js 18+
- Docker & Docker Compose（可选，用于一键启动依赖服务）

### 1. 克隆仓库
```bash
git clone https://github.com/weigu77/career-agent.git
cd career-agent
