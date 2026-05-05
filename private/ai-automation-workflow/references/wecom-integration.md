# 企业微信 (WeCom) 集成参考

> 节点包：`n8n-nodes-wecom`
> GitHub：https://github.com/funcodingdev/n8n-nodes-wecom
> NPM：https://www.npmjs.com/package/n8n-nodes-wecom
> ncnodes：https://ncnodes.com/package/n8n-nodes-wecom
> 最新版本：0.4.10 | MIT License

---

## 一、节点类型（共7个）

| 节点名称 | 类型标识 | 用途 |
|---------|---------|------|
| 企业微信-基础 | `n8n-nodes-wecom.weComBase` | 通讯录、消息发送、群聊、素材、企业互联等 |
| 企业微信-办公 | `n8n-nodes-wecom.weComOffice` | 日程、会议、直播、邮件、文档、微盘、打卡、审批等 |
| 企业微信-连接微信 | `n8n-nodes-wecom.weComWeChat` | 客户联系、微信客服、家校应用 |
| 企业微信消息接收触发器 | `n8n-nodes-wecom.weComTrigger` | 接收应用消息和事件推送 |
| 企业微信消息接收(被动回复)触发器 | `n8n-nodes-wecom.weComTriggerReply` | 接收消息并支持被动回复 |
| 企业微信第三方应用指令回调触发器 | `n8n-nodes-wecom.weComSuiteTrigger` | 接收第三方应用回调事件 |
| 企业微信智能机器人消息接收触发器 | `n8n-nodes-wecom.weComBotTrigger` | 接收智能机器人消息 |

## 二、凭证配置

### 凭证一：企业微信请求凭证 (`weComApi`)

**用途**：消息发送、通讯录、素材管理等 API 调用

**配置步骤**：
1. 登录企业微信管理后台 https://work.weixin.qq.com/
2. "我的企业" > "企业信息" → 复制企业ID (CorpID)
3. "应用管理" > 选择或创建应用 → 复制 AgentId 和 Secret

| 参数 | 说明 |
|------|------|
| CorpID | 企业ID |
| AgentId | 应用ID |
| Secret | 应用密钥 |
| API Base URL | 可选，自定义 API 代理地址（默认直连 qyapi.weixin.qq.com） |

### 凭证二：企业微信消息接收凭证 (`weComReceiveApi`)

**用途**：接收企业微信消息和事件回调

**配置步骤**：
1. 登录企业微信管理后台
2. 获取企业ID (CorpID)
3. "应用管理" → 选择应用 → 启用 "API接收消息" → 设置 Token、EncodingAESKey
4. 在 n8n 中创建触发器节点，配置凭证和 Path
5. 将生成的 Webhook URL 填入企业微信后台

| 参数 | 说明 |
|------|------|
| CorpID | 企业ID |
| Token | 消息校验 Token |
| EncodingAESKey | 消息加解密密钥 |
| Path | Webhook URL 路径（建议使用应用 AgentId） |

### 凭证三：群机器人 Webhook 凭证 (`weComBotWebhook`)

**用途**：通过群机器人 Webhook 发送消息到群聊

**配置步骤**：
1. 在企业微信群聊中，点击右上角 "..." → "群机器人" > "添加机器人"
2. 创建机器人并复制 Webhook 地址
3. 在 n8n 中配置凭证，填入 Webhook 地址

### 凭证四：第三方应用指令回调凭证 (`weComSuiteApi`)

| 参数 | 说明 |
|------|------|
| SuiteID | 第三方应用ID（以ww或wx开头） |
| Token | 校验 Token |
| EncodingAESKey | 加解密密钥 |
| Path | Webhook 路径 |

> 注意：回调使用 SuiteID 作为 receiveid（不是 CorpID），收到推送后必须返回 "success"

## 三、常用开发模式

### 模式一：接收消息 → 处理 → 发送消息

```
[weComTrigger] → [Set/Code 处理] → [weComBase(message send)]
```

- 触发器：`n8n-nodes-wecom.weComTrigger`，typeVersion: 1
- 凭证：`weComReceiveApi`
- 发送节点：`n8n-nodes-wecom.weComBase`，typeVersion: 1
- 凭证：`weComApi`

发送节点参数示例：
```json
{
  "resource": "message",
  "touser": ["用户ID"],
  "content": "={{ $json.Content }}"
}
```

### 模式二：接收消息 → 被动回复

```
[weComTriggerReply] → [处理(可选)] → [weComBase(被动回复)]
```

- 被动回复节点必须是工作流最后一个节点
- 必须在 5 秒内返回响应

### 模式三：群机器人推送

```
[任意触发器] → [处理] → [weComBase(群机器人推送)]
```

## 四、Webhook 路径配置

**Path 参数说明**：
- Path 表示 Webhook URL 的路径部分，建议使用应用的 AgentId
- 完整 URL 格式：`https://your-n8n.com/webhook/{Path}`

**配置流程**：
1. 登录企业微信管理后台
2. "我的企业" → "企业信息"，复制企业ID (CorpID)
3. "应用管理" → 选择或创建应用
4. 启用 API接收消息，设置 Token、EncodingAESKey
5. 在 n8n 中创建触发器节点，配置凭证和 Path
6. 保存节点后获取 Webhook URL
7. 将 Webhook URL 填入企业微信后台的接收消息服务器配置

**重要规则**：
- 每个企业微信应用只能配置一个接收消息 URL
- 多个工作流可共享同一凭证（同一应用），共用同一个 Webhook URL
- 不同应用需创建不同凭证，使用不同的 Path

## 五、消息数据字段

### 接收消息触发器输出

| 字段 | 说明 |
|------|------|
| `Content` | 文本消息内容 |
| `FromUserName` | 发送者 UserID |
| `CreateTimestamp` | 消息创建时间戳 |
| `MsgType` | 消息类型（text/image/voice/video/location/link） |
| `PicUrl` | 图片消息 URL |
| `MediaId` | 媒体文件 ID |

### 发送消息节点参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `resource` | 资源类型 | `"message"` |
| `touser` | 接收用户ID列表 | `["SuJianHan"]` |
| `content` | 消息内容（支持表达式） | `"={{ $json.Content }}"` |
| `msgtype` | 消息类型 | `"text"`（默认） |

## 六、完整功能清单

### 基础功能（weComBase）

#### 消息接收（触发器）
- 文本/图片/语音/视频/位置/链接消息
- 事件推送（成员变更、部门变更等）、接口许可失效通知
- URL 验证、消息加解密、签名验证

#### 被动回复消息
- 文本/图片/语音/视频/图文/模板卡片更新
- 自动加密和签名，必须在 5 秒内返回

#### 消息推送（群机器人）
文本/Markdown/Markdown V2/图片/图文/文件/语音/模板卡片

#### 应用消息发送
文本/Markdown/图片/语音/视频/文件/文本卡片/图文/小程序通知/任务卡片/模板卡片/学校通知/撤回/更新模板卡片

#### 群聊会话
创建/获取/修改群聊、发送消息到群聊（文本/图片/文件/Markdown/图文，支持@群成员/@所有人）

#### 通讯录管理
- **成员管理（14项）**：创建/读取/更新/删除/批量删除/部门成员列表/详情/ID列表/userid↔openid/二次验证/邀请/二维码/手机号邮箱查userid/临时外部联系人ID转换
- **部门管理（6项）**：创建/更新/删除/列表/子部门ID列表/详情
- **标签管理（7项）**：创建/更新/删除/获取/增删成员/列表
- **账号ID转换（12项）**：用户ID↔OpenID/corpid/userid/external_userid/unionid转换/客户标签ID/微信客服ID转换/ID迁移
- **异步导入（4项）**：增量更新成员/全量覆盖成员/部门/获取结果
- **异步导出（5项）**：导出成员/详情/部门/标签成员/获取结果

#### 素材管理
上传临时素材/上传图片/异步上传/获取临时素材/获取高清语音素材

#### 企业互联
企业互联基础（3项）/ 上下游基础（5项）/ 通讯录管理（6项）/ 规则管理（5项）

#### 其他
- 系统：获取接口IP段/回调IP段
- 电子发票：查询/更新/批量更新/批量查询
- 第三方应用授权（13项）
- 接口调用许可（8项）
- 收银台（8项）
- 推广二维码（15项）

### 办公功能（weComOffice）

#### 邮件管理
发送（普通/日程/会议邮件）/ 接收（收件箱/邮件内容）/ 账号管理 / 邮件群组（5项）/ 公共邮箱（5项）/ 客户端密码 / 高级功能 / 功能属性 / 未读数

#### 文档管理
管理（新建/重命名/删除/信息/分享）/ 编辑（文档/表格）/ 智能表格（子表/视图/字段/记录增删改查 + 接收外部数据）/ 数据获取 / 权限管理（5项）/ 收集表（5项）

#### 日程管理
日历（创建/更新/详情/删除）/ 日程（创建/更新/更新重复/参与者管理/列表/详情/取消）

#### 会议管理
基础（5项）/ 统计 / 高级管理（6项）/ 会中控制（3项）/ 录制（2项）/ 高级功能账号（3项）

#### 直播管理（9项）
创建/修改/取消预约/删除回放/微信观看/ID列表/详情/观看明细/观众信息

#### 微盘管理
空间管理（8项）/ 文件管理（8项）/ 文件权限管理（6项）

#### 打卡管理（12项）
规则/记录/日报/月报/排班/补卡/添加记录/录入人脸/设备数据/规则管理

#### 审批管理（9项）
模板创建/获取/更新/提交审批/批量单号/详情/假期管理/余额查询修改

#### 其他
汇报管理（4项）/ 人事助手（3项）/ 会议室管理（5项）/ 紧急通知（2项）

### 连接微信功能（weComWeChat）

#### 客户联系
企业服务人员 / 客户管理（4项）/ 标签管理（5项）/ 在职继承（3项）/ 离职继承（4项）/ 客户群管理（3项）/ 联系我与入群方式（8项）/ 朋友圈（4项）/ 消息推送（8项）/ 统计（2项）/ 其他（5项）

#### 微信客服
客服账号管理（5项）/ 接待人员管理（3项）/ 会话与消息（6项）/ 统计（2项）/ 机器人管理（2项）

#### 家校应用
健康上报（4项）/ 上课直播（7项）/ 班级收款（2项）/ 家校沟通（6项）/ 网页授权（4项）/ 学生家长管理（16项）/ 部门管理（6项）/ 通讯录变更（2项）

#### 政民沟通
网格配置（5项）/ 事件类别（4项）/ 巡查上报（7项）/ 居民上报（7项）

## 七、隐私与安全

| 特性 | 说明 |
|------|------|
| 数据直连 | 默认直连企业微信官方服务器 (qyapi.weixin.qq.com) |
| API代理 | 支持自定义 API Base URL |
| 无数据缓存 | 不存储、不缓存任何企业数据 |
| 无第三方依赖 | 不依赖第三方数据/分析服务 |
| 开源透明 | 源代码完全开源可审计 |

## 八、安装方式

**n8n 界面安装**：社区节点管理界面搜索 `n8n-nodes-wecom`

**命令行安装**：`npm install n8n-nodes-wecom`

## 九、官方参考

- 企业微信开发文档：https://developer.work.weixin.qq.com/document/
- API 错误码：https://developer.work.weixin.qq.com/document/path/90313
- 常见问题：https://developer.work.weixin.qq.com/document/path/90315

## 十、注意事项

1. 被动回复必须在 5 秒内返回，且必须是工作流最后一个节点
2. 第三方应用回调使用 SuiteID（不是 CorpID）作为 receiveid
3. 第三方应用授权事件必须在 1000ms 内响应
4. 每个应用只能配置一个接收消息 URL
5. 多个工作流可共享同一凭证和 Webhook URL
6. 不同应用需创建不同凭证和 Path
