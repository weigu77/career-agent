# 大学生职业规划智能体 - API 接口文档

## 版本信息
- **版本**: 1.0.0
- **基础路径**: `/api/v1`
- **协议**: HTTP + WebSocket
- **认证**: JWT Bearer Token（开发环境可暂缓，预留 `Authorization: Bearer <token>` 头）

---

## 1. 聊天对话接口

### 1.1 普通对话 (HTTP)

**端点**: `POST /chat`

**描述**: 发送用户消息，获取完整回复（非流式）。适用于短文本或不需要流式展示的场景。

**请求头**:
| 名称 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `Content-Type` | string | 是 | `application/json` |
| `Authorization` | string | 否 | Bearer token |

**请求体** (JSON):
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `message` | string | 是 | 用户输入消息 |
| `conversation_id` | string | 否 | 会话ID，不传则自动生成新会话 |
| `user_id` | string | 是 | 用户唯一标识 |
| `stream` | boolean | 否 | 固定 `false` |

**响应体** (JSON):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "conversation_id": "conv_123abc",
    "answer": "根据您的计算机专业背景... 推荐前端实习岗位：...",
    "matched_jobs": [
      {
        "title": "Web前端开发实习生",
        "company": "东莞市恒亚罗斯计算机科技有限公司",
        "url": "https://...",
        "skills_required": ["React", "前端架构", "性能优化"],
        "gap": []
      }
    ],
    "plan": "建议学习计划：... (Markdown)",
    "timestamp": "2025-04-26T10:30:00Z"
  }
}
```

**示例**:
```bash
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user001",
    "message": "我是计算机专业大二，熟悉Python，想找数据分析实习"
  }'
```

---

### 1.2 流式对话 (WebSocket)

**端点**: `ws/chat`

**描述**: 建立 WebSocket 连接，实时推送 Agent 的思考过程、工具调用和最终回答。

**连接参数** (URL Query):
| 参数 | 必填 | 说明 |
|------|------|------|
| `user_id` | 是 | 用户标识 |
| `conversation_id` | 否 | 不传则服务端生成新会话 |

**客户端发送消息格式** (JSON):
```json
{
  "type": "user_message",
  "content": "我想了解产品经理岗位所需技能"
}
```

**服务端推送事件格式** (JSON 流，每行一个事件):
```json
// 思考中
{"type": "thought", "content": "用户询问产品经理技能，需要先匹配岗位..."}

// 工具调用开始
{"type": "tool_start", "tool": "VectorSearch", "input": "产品经理 技能"}

// 工具调用结果
{"type": "tool_result", "tool": "VectorSearch", "output": "找到12条相关岗位..."}

// 最终答案（可能分多次）
{"type": "answer_chunk", "content": "产品经理通常需要..."}

// 附加岗位信息
{"type": "matched_jobs", "data": [{"title": "...", "company": "..."}]}

// 结束
{"type": "end"}
```

**示例流程**:
1. 客户端连接: `ws://localhost:8000/api/v1/ws/chat?user_id=user001`
2. 客户端发送: `{"type":"user_message","content":"推荐几个南京的Java实习"}`
3. 服务端依次推送上述事件。

---

## 2. 简历上传与解析

**端点**: `POST /resume/upload`

**描述**: 上传简历文件（PDF/Word/纯文本），返回解析后的结构化信息，并自动更新用户画像。

**请求头**:
| 名称 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `Content-Type` | string | 是 | `multipart/form-data` |

**请求体** (form-data):
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `resume` | file | 是 | 简历文件（扩展名 .pdf/.docx/.txt） |
| `user_id` | string | 是 | 用户ID |

**响应体**:
```json
{
  "code": 200,
  "data": {
    "skills": ["Python", "SQL", "Tableau"],
    "major": "信息管理与信息系统",
    "degree": "本科",
    "projects": ["电商用户行为分析", "智慧校园App"],
    "education": "XX大学 2022-2026",
    "weak_skills": []
  }
}
```

**错误响应**:
```json
{
  "code": 400,
  "message": "不支持的文件格式，仅支持 PDF/DOCX/TXT"
}
```

**注意**: 服务器不保留原始文件，解析后立即删除。

---

## 3. 岗位匹配 (快速查询)

**端点**: `GET /jobs/match`

**描述**: 不通过对话流程，直接根据技能和目标岗位检索匹配的职位。适合前端自动补全或推荐列表。

**请求参数** (Query String):
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `skills` | string | 是 | 技能列表，逗号分隔，例如 `Python,SQL,React` |
| `goal` | string | 否 | 目标岗位关键词，例如 `数据分析` |
| `industry` | string | 否 | 行业过滤，例如 `互联网`、`生物科技` |
| `city` | string | 否 | 城市过滤，例如 `深圳` |
| `top_k` | integer | 否 | 返回数量，默认5，最大20 |
| `user_id` | string | 是 | 用于个性化排序（后续扩展） |

**响应体**:
```json
{
  "code": 200,
  "data": [
    {
      "job_id": "job_001",
      "title": "数据分析实习生",
      "company": "腾讯科技",
      "url": "https://...",
      "update_date": "2025-04-20",
      "skills_required": ["Python", "SQL", "数据可视化"],
      "match_score": 0.85,
      "gap": ["数据可视化"]
    },
    {
      "job_id": "job_002",
      "title": "BI工程师",
      "company": "字节跳动",
      "url": "https://...",
      "update_date": "2025-04-18",
      "skills_required": ["Python", "Tableau", "数据仓库"],
      "match_score": 0.72,
      "gap": ["Tableau", "数据仓库"]
    }
  ]
}
```

**示例**:
```bash
GET http://localhost:8000/api/v1/jobs/match?skills=Python,SQL&goal=数据分析&top_k=3&user_id=user001
```

---

## 4. 反馈提交

**端点**: `POST /feedback`

**描述**: 用户对某条智能体回答或推荐结果进行评价，用于后续模型微调和检索排序。

**请求体**:
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `user_id` | string | 是 | 用户ID |
| `message_id` | string | 是 | 对应回答的消息ID（由对话接口返回） |
| `rating` | integer | 是 | 1（有用）或 0（无用） |
| `comment` | string | 否 | 可选理由 |
| `conversation_id` | string | 是 | 所属会话ID |
| `job_ids` | array | 否 | 如果反馈针对具体的岗位推荐，可提供岗位ID列表 |

**响应体**:
```json
{
  "code": 200,
  "message": "feedback recorded"
}
```

---

## 5. 健康检查

**端点**: `GET /health`

**描述**: 返回服务运行状态。

**响应体**:
```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

---

## 6. 错误码定义

| 错误码 | 含义 | 典型场景 |
|--------|------|----------|
| 200 | 成功 | - |
| 400 | 请求参数错误 | 缺少必填字段、文件格式不支持 |
| 401 | 未授权 | Token 无效或过期 |
| 404 | 资源不存在 | 会话ID不存在 |
| 429 | 请求过于频繁 | 触发限流 |
| 500 | 服务器内部错误 | LLM调用失败、数据库连接出错 |
| 503 | 服务不可用 | 依赖服务（如Milvus）不可用 |

**统一错误响应格式**:
```json
{
  "code": 400,
  "message": "技能参数不能为空",
  "details": null
}
```

---

## 7. WebSocket 完整交互示例

```text
Client -> Server (连接)
ws://localhost:8000/api/v1/ws/chat?user_id=u1&conversation_id=conv_101

Client -> Server:
{"type":"user_message","content":"我是生物技术专业，熟悉PCR和测序，想找研究助理工作"}

Server -> Client (逐条):
{"type":"thought","content":"用户希望岗位匹配，调用画像Agent提取专业与技能"}
{"type":"tool_start","tool":"ProfileAgent","input":"生物技术, PCR, 测序, 研究助理"}
{"type":"tool_result","tool":"ProfileAgent","output":"{\"major\":\"生物技术\",\"skills\":[\"PCR\",\"测序\"],\"goal\":\"研究助理\"}"}
{"type":"thought","content":"现在检索1万条岗位数据"}
{"type":"tool_start","tool":"VectorSearch","input":"研究助理 生物技术 PCR 测序"}
{"type":"tool_result","tool":"VectorSearch","output":"找到香港中文大学深圳研究院等3个岗位"}
{"type":"matched_jobs","data":[{"title":"研究助理（生物技术）","company":"香港中文大学深圳研究院","url":"...","gap":["高通量测序(NGS)","长读长测序"]}]}
{"type":"answer_chunk","content":"为您匹配到3个研究助理岗位，其中与您最匹配的是：香港中文大学深圳研究院"}
{"type":"answer_chunk","content":"您缺少NGS和长读长测序经验。是否需要我为您制定学习计划？"}
{"type":"end"}
```

---

## 8. 注意事项

- **认证**: 生产环境建议启用 JWT；开发环境可用 `X-User-Id` 头模拟。
- **限流**: 每用户每分钟最多 30 次 HTTP 请求，5 次 WebSocket 连接。
- **数据保留**: 用户画像保留 180 天，超过则清理。
- **跨域**: 所有接口支持 CORS，允许前端 React 应用访问。

---

## 9. 更新日志

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 1.0.0 | 2025-04-26 | 初始版本，定义所有核心接口 |

---