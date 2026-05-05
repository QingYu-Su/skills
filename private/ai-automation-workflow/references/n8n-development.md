# n8n 工作流开发参考

## n8n 核心概念

- **Workflow（工作流）**：由多个节点和连接组成的自动化流程
- **Node（节点）**：工作流中的单个操作单元，分为触发器节点和功能节点
- **Connection（连接）**：节点之间的数据流向
- **Trigger（触发器）**：工作流的起始节点，负责接收外部事件
- **Credential（凭证）**：存储 API 密钥等敏感信息的配置
- **Expression（表达式）**：n8n 的动态数据语法，使用 `={{ $json.fieldName }}` 格式

## MCP 连接配置教程

### 检查连通性

调用 `n8n_health_check` 工具验证 n8n MCP 连接。如果失败，按以下步骤配置。

### 配置步骤

**1. 获取 n8n 连接凭证**
- 访问 n8n 实例并登录
- 点击左下角 Settings → n8n API
- 生成一个新的 API Key（记下 N8N_API_KEY）
- 确认 N8N_API_URL（本地 Docker 通常为 `http://host.docker.internal:5678`，宿主机直接访问用 `http://localhost:5678`）

**2. 配置 CodeBuddy 的 MCP**

配置文件路径：`~/.codebuddy.json`（Mac/Linux）或 `%USERPROFILE%\.codebuddy.json`（Windows）

编辑该文件，添加 n8n-mcp 配置（替换实际的 URL 和 Key）：

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "http://host.docker.internal:5678",
        "N8N_API_KEY": "你的_API_Key_在这里"
      }
    }
  }
}
```

## 常用内置节点

| 节点 | 类型标识 | 用途 |
|------|---------|------|
| Webhook | `n8n-nodes-base.webhook` | 接收 HTTP 请求触发工作流 |
| Set | `n8n-nodes-base.set` | 设置/转换数据 |
| IF | `n8n-nodes-base.if` | 条件分支 |
| Switch | `n8n-nodes-base.switch` | 多路分支 |
| HTTP Request | `n8n-nodes-base.httpRequest` | 发送 HTTP 请求 |
| Code | `n8n-nodes-base.code` | 执行自定义 JS/Python 代码 |
| Merge | `n8n-nodes-base.merge` | 合并多个数据流 |
| Respond to Webhook | `n8n-nodes-base.respondToWebhook` | 响应 Webhook 请求 |
| Schedule Trigger | `n8n-nodes-base.scheduleTrigger` | 定时触发 |
| No Operation | `n8n-nodes-base.noOp` | 空操作（用于分支合并点） |

## n8n MCP 工具清单

使用 MCP 工具前必须先调用 `mcp_get_tool_description` 获取参数说明。

| 工具 | 用途 |
|------|------|
| `n8n_health_check` | 检查 n8n 实例连通性 |
| `search_nodes` | 搜索可用节点 |
| `get_node` | 获取节点详细参数 |
| `validate_node` | 验证节点配置 |
| `search_templates` | 搜索工作流模板 |
| `get_template` | 获取模板详情 |
| `n8n_create_workflow` | 创建新工作流 |
| `n8n_get_workflow` | 获取工作流详情 |
| `n8n_update_full_workflow` | 全量更新工作流 |
| `n8n_update_partial_workflow` | 增量更新（推荐），支持 addNode/removeNode/updateNode/addConnection/removeConnection/activateWorkflow/patchNodeField 等操作 |
| `n8n_delete_workflow` | 删除工作流 |
| `n8n_list_workflows` | 列出所有工作流 |
| `n8n_validate_workflow` | 验证工作流配置 |
| `n8n_test_workflow` | 测试执行工作流 |
| `n8n_executions` | 查看执行记录 |
| `n8n_workflow_versions` | 查看工作流版本历史 |

## 工作流开发流程

1. **规划工作流**：明确触发源、处理逻辑、输出目标
2. **搜索可用节点**：使用 `search_nodes` 查找合适的节点
3. **获取节点详情**：使用 `get_node` 查看节点参数和版本
4. **创建/更新工作流**：使用 `n8n_create_workflow` 或 `n8n_update_full_workflow`
5. **验证工作流**：使用 `n8n_validate_workflow` 检查配置
6. **测试工作流**：使用 `n8n_test_workflow` 执行测试
7. **激活工作流**：使用 `n8n_update_partial_workflow` 的 `activateWorkflow` 操作

## 重要注意事项

- 节点的 `type` 和 `typeVersion` 必须准确，使用 `get_node` 确认
- 凭证（credentials）需要引用已存在的凭证 ID 和名称，从用户已有工作流中获取
- 表达式语法：`={{ $json.fieldName }}`
- 节点位置（position）使用 `[x, y]` 坐标，建议相邻节点间隔 200-260
- `n8n_update_partial_workflow` 比 `n8n_update_full_workflow` 更轻量，适合小幅修改
