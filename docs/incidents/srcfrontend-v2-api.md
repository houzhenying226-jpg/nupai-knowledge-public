# src/与frontend-v2/ API客户端导出方式不兼容

> 类型：故障复盘 | 项目：nupai-store | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

在 src/api/ 下新建适配文件 wecom.api.ts，用 src/ 的 default import 重新封装：
```PLAIN_TEXT
// src/api/wecom.api.tsimport client from './client'
export const wecomApi = {  getJsapiTicket: (url: string) =>    client.g

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
从 frontend-v2/ 复制组件到 src/ 后，API 调用报错，因为两个目录的 API 客户端导出方式不同。

## 内容
- frontend-v2/src/api/client.ts：export \{ apiClient \} \(named export\)
```PLAIN_TEXT
import { apiClient } from '@/api/client'apiClient.get('/...')
```
- src/api/client.ts：export default client \(default export\)
```PLAIN_TEXT
import client from './client'client.get('/...')
```
从 frontend-v2/ 复制来的组件直接用 import \{ apiClient \} 会报 TS 错误 / 运行时 undefined。

## 结论
在 src/api/ 下新建适配文件 wecom.api.ts，用 src/ 的 default import 重新封装：
```PLAIN_TEXT
// src/api/wecom.api.tsimport client from './client'
export const wecomApi = {  getJsapiTicket: (url: string) =>    client.get<JsapiTicketResponse>('/wecom/jsapi-ticket', { params: { url } }).then(r => r.data),  suggest: (data: WecomSuggestRequest) =>    client.post<WecomSuggestResponse>('/ai/wecom-suggest', data, { timeout: 15000 }).then(r => r.data),}
```
组件里改成 import \{ wecomApi \} from '@/api/wecom.api'。

## 影响
- src/api/wecom.api.ts（已创建）
- src/pages/store/WecomSidebar.tsx（已修正 import）

## 下次注意
从 frontend-v2/ 搬代码到 src/ 时，必须把所有 import \{ apiClient \} 改成 import client from './client'，或新建 xxx.api.ts 封装层。这是两个目录最常见的不兼容点。

