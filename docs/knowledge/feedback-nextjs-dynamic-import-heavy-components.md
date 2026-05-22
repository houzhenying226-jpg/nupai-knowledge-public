# L16 · Next.js 重组件（D3/图谱）必须用 dynamic import 懒加载

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

判断标准：组件在 Tab 下（非首屏必见）且 bundle >50KB → 必须 dynamic()。ssr: false + loading Skeleton。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
D3 关系图同步引入导致首屏多 ~300KB JS bundle，影响首页加载速度。

## 内容
const RelationshipTopology = dynamic(() => import('@/components/ai/RelationshipTopology'), { ssr: false, loading: () => <Skeleton active paragraph={{ rows: 8 }} /> });

## 结论
判断标准：组件在 Tab 下（非首屏必见）且 bundle >50KB → 必须 dynamic()。ssr: false + loading Skeleton。

## 影响
首页加载体积减少 ~300KB，首屏速度明显提升。

## 下次注意
任何 Tab 下组件（图表/D3/图谱）bundle >50KB → 必须 dynamic import 懒加载。

