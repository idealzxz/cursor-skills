---
name: yuque-workload-evaluator
description: 从语雀(Yuque)文档读取PRD需求并评估前端开发工时。当用户提供语雀链接要求评估工时、估时、排期，或提到PRD评审、工时评估时使用。
---

# 语雀 PRD 前端工时评估

从语雀文档读取 PRD 需求内容，结合当前项目代码现状，输出前端开发工时评估。

## 工作流程

### Step 1: 解析语雀 URL

从用户提供的 URL 中提取 `namespace`（用户/团队）、`book`（知识库）、`slug`（文档标识）：

| URL 格式 | 示例 |
|---------|------|
| `https://{org}.yuque.com/{namespace}/{book}/{slug}` | `https://duiba.yuque.com/slswz7/yz4g7v/kwqlgo3rqu6q2qam` |

### Step 2: 获取语雀文档内容

语雀企业版需要登录认证，**必须使用 Shell curl + session cookie** 访问。

#### 2.1 获取 session cookie

请用户从浏览器获取 `_yuque_session` cookie：
- 开发者工具 → Application → Cookies → 对应域名 → `_yuque_session`

#### 2.2 通过内部 API 获取文档内容

使用 `sourcecode` 字段获取 markdown 格式的文档内容：

```bash
curl -s --connect-timeout 15 -L \
  -H "Cookie: _yuque_session={cookie}" \
  "https://{org}.yuque.com/api/docs/{doc_id}?book_id={book_id}&merge_dynamic_data=false&mode=markdown"
```

#### 2.3 若不知道 doc_id，先从页面 HTML 提取

```bash
curl -s --connect-timeout 15 -L \
  -H "Cookie: _yuque_session={cookie}" \
  "https://{org}.yuque.com/{namespace}/{book}/{slug}" | python3 -c "
import sys, urllib.parse, json, re
html = sys.stdin.read()
match = re.search(r'decodeURIComponent\(\"(.+?)\"\)', html)
if match:
    decoded = urllib.parse.unquote(match.group(1))
    data = json.loads(decoded)
    doc = data.get('doc', {})
    print(f'doc_id: {doc.get(\"id\")}')
    print(f'book_id: {doc.get(\"book_id\")}')
    print(f'title: {doc.get(\"title\")}')
"
```

获取到 `doc_id` 和 `book_id` 后，再通过内部 API 获取完整内容。若内部 API 返回 404，可直接从页面 HTML 的 `appData.doc.sourcecode` 字段提取文档内容：

```bash
curl -s --connect-timeout 15 -L \
  -H "Cookie: _yuque_session={cookie}" \
  "https://{org}.yuque.com/{namespace}/{book}/{slug}" | python3 -c "
import sys, urllib.parse, json, re
html = sys.stdin.read()
match = re.search(r'decodeURIComponent\(\"(.+?)\"\)', html)
if match:
    decoded = urllib.parse.unquote(match.group(1))
    data = json.loads(decoded)
    doc = data.get('doc', {})
    print(doc.get('sourcecode', ''))
"
```

### Step 3: 分析项目现状

探索当前项目代码库，了解已完成的开发工作：

1. **页面**：`src/pages/` 下已有哪些页面
2. **组件**：`src/components/` 下已有哪些组件
3. **API 接口**：`src/api/index.js` 中已定义的接口
4. **Mock 数据**：`mock/` 下已有的 mock
5. **Store**：`src/store/` 中的状态管理
6. **路由配置**：页面注册情况

### Step 4: 评估工时

#### 评估维度

将 PRD 中的需求拆解为以下类别：

| 类别 | 评估要点 |
|------|---------|
| **页面开发** | 切图 + 交互 + 业务逻辑，考虑复杂度和可复用性 |
| **弹窗开发** | 弹窗 UI + 交互逻辑 |
| **通用功能** | 登录授权、分享、埋点、时间判断等 |
| **联调 & 自测** | 接口联调 + 功能自测 + Bug 修复 |

#### 工时参考基准（H5活动页）

| 页面复杂度 | 参考工时 | 特征 |
|-----------|---------|------|
| 简单页面 | 2-4h | 纯展示，少量交互，无表单 |
| 中等页面 | 4-8h | 有表单/列表，中等交互逻辑 |
| 复杂页面 | 8-12h | 多状态切换、复杂表单、分页/轮播 |
| 弹窗 | 2-4h | 含按钮交互和状态判断 |
| 通用功能 | 2-4h/项 | 登录/分享/埋点等 |
| 接口联调 | 8-12h | 视接口数量而定 |
| 自测 | 4-8h | 功能走查 + 兼容性 |

#### 调整因素

- 已有代码可复用 → 减少 30-50%
- 多角色/多入口 → 增加复用组件的适配时间
- 第三方对接（小程序API、SDK）→ 额外预留时间
- 埋点方案未确认 → 标注待定，预留 buffer

### Step 5: 输出格式

以序号列表形式输出，单位为小时：

```markdown
# {项目名} - 前端工时评估

1. {页面/功能名称}（简要说明）：Xh
2. {页面/功能名称}（简要说明）：Xh
...
N. 接口联调：Xh
N+1. 自测 & Bug修复：Xh

合计：XXh
```

## 注意事项

- 语雀文档 `sourcecode` 字段为 HTML 格式，需从中提取需求信息
- WebFetch 无法访问需要登录的语雀页面，必须用 Shell curl
- session cookie 有时效性，过期需用户重新提供
- 工时评估应结合项目已有代码进行调整，避免重复计算已完成部分
