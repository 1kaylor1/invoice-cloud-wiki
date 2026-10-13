---
name: yunzhijia-knowledge
description: 回答云之家（Kingdee云之家）产品功能、开发集成、实施交付相关的知识问题。使用当用户询问云之家产品使用、API开发、开放平台对接、实施部署、功能配置、常见问题等话题时。触发词包括：云之家、yunzhijia、开放接口、API、开发文档、产品手册、实施指南、审批、考勤、门户、通讯、轻云、融合中心。
---

# 云之家知识问答

> **知识库截至时间：2026年5月**。2026年5月之后的产品更新、API变更、新功能等内容不在本技能覆盖范围内，遇到时请提示用户查阅线上最新资料。
>
> **环境依赖**：检索脚本需要 Python 3（仅使用标准库，无需额外安装包）。

## 触发规则

当用户提出以下类型问题时，自动应用本 Skill：
- 云之家产品功能使用咨询（如审批、考勤、门户、IM等） — **CSM常用**
- 开放平台开发对接（API调用、服务端开发、客户端开发、OAuth鉴权、回调配置） — **技术支持常用**
- 实施交付相关问题（部署、配置、数据迁移）
- 云之家与其他系统的集成方案（如ERP对接审批、HR系统同步等） — **技术支持常用**
- 功能更新或版本变更咨询
- 常见问题的排查与解决（接口报错、审批异常、集成故障等） — **技术支持常用**

## 知识域划分

本 Skill 采用**三域划分**组织知识，匹配用户角色和问题类型：

| 知识域 | 目录 | 适用角色 | 内容类型 |
|-------|------|---------|---------|
| **product** | references/product/ | CSM、客户 | 功能使用指南、FAQ、版本更新 |
| **development** | references/development/ | **技术支持**、开发人员 | API文档、SDK、客户端开发、OAuth授权、集成方案 |
| **implementation** | references/implementation/ | 实施顾问、CSM | 配置指南、部署文档、数据迁移 |

> **技术支持岗位使用提示**：development 域是技术支持的核心参考资料，覆盖OAuth鉴权、所有服务端API、流程中心集成、IM消息推送、组织人员同步等高频技术场景。遇到客户集成对接问题时，优先检索 development 域。

## 检索策略

**本技能完全使用本地检索，不访问任何外部网站。**

### 检索方法

执行检索脚本，按知识域精准搜索：

```bash
# 搜索产品功能（CSM常用）
python3 {skill_dir}/scripts/search_reference.py "审批" --domain product

# 搜索开发API（技术支持常用）
python3 {skill_dir}/scripts/search_reference.py "accessToken" --domain development

# 搜索实施配置（实施顾问常用）
python3 {skill_dir}/scripts/search_reference.py "考勤规则" --domain implementation

# 全域搜索（不确定问题类型时）
python3 {skill_dir}/scripts/search_reference.py "审批" --domain all

# 多关键词AND搜索（所有关键词都匹配）
python3 {skill_dir}/scripts/search_reference.py "审批 回调" --domain development

# 多关键词OR搜索（任一关键词匹配）
python3 {skill_dir}/scripts/search_reference.py "审批 审批流" --mode or

# 按业务模块精准过滤
python3 {skill_dir}/scripts/search_reference.py "token" --module OAuth授权
python3 {skill_dir}/scripts/search_reference.py "部门" --module 组织人员

# 排除噪音关键词（JSAPI文档中大量callback参数会干扰"回调"搜索）
python3 {skill_dir}/scripts/search_reference.py "回调" --domain development --exclude JSAPI qing.call
```

参数说明：
- `-d/--domain` — 知识域（product/development/implementation/all）
- `-m/--module` — 业务模块过滤（OAuth授权、组织人员、智能审批、IM消息...）
- `--mode` — 多关键词模式：and=全部匹配 or=任一匹配（默认and）
- `-e/--exclude` — 排除关键词（文件包含这些词则跳过，如 `--exclude JSAPI qing.call`）
- `-n/--max` — 最大返回结果数（默认10）
- `-c/--context` — 上下文行数（默认5）

智能提示：
- 搜索时自动根据关键词推荐可能匹配的模块（如搜"回调"会提示"流程引擎"）
- 无结果时自动推荐可尝试的 `--module` 参数
- 无需手动指定 `--module` 也能看到模块建议

### 检索流程

1. **判断问题所属域**
   - 产品功能使用问题 → `--domain product`
   - API/开发/集成/鉴权问题 → `--domain development`（**技术支持首选**）
   - 配置/部署问题 → `--domain implementation`
   - 不确定 → `--domain all`

2. **执行检索**
   - 返回匹配片段，不加载全文
   - 根据匹配数排序，优先返回高匹配结果

3. **精准加载**
   - 根据检索结果，定向加载相关文件
   - 避免加载大文件消耗大量 token

4. **本地无结果时**
   - 明确告知用户本地资料未覆盖此问题
   - 提示用户按需访问以下网站自行搜索：

| 网站 | URL | 适用场景 |
|-----|-----|---------|
| 开放平台文档 | https://open.yunzhijia.com/opendocs/#/ | API最新文档 |
| 产品手册（语雀） | https://yunzhijia.yuque.com/qhigp7/sat4a4/znsp | 功能使用指南 |
| 问题合集（语雀） | https://yunzhijia.yuque.com/qhigp7/kfcpqh/znspqa | 常见问题排查 |
| 金蝶社区 | https://vip.kingdee.com/?productLineId=12 | 社区问答 |

## 问题分类路由

完整的问题分类与知识域路由表见 [categories.md](categories.md)。以下为高频场景速查：

| 用户问题示例 | 推荐知识域 | 关键词示例 |
|-------------|-----------|-----------|
| "审批怎么用？" | product | 审批、功能使用 |
| "审批API怎么调用？" | development | API、accessToken、回调 |
| "考勤怎么配置？" | implementation | 配置、规则、设置 |
| "审批流程报错了" | product + development | 报错、失败、排查 |
| "客户ERP要对接审批回调" | development | 回调、集成、流程引擎 |
| "accessToken获取报403" | development | accessToken、OAuth、鉴权 |

跨域复杂问题（如"审批回调ERP"）应分步检索：先 `--domain development --module 流程引擎` 找 API，再 `--domain implementation` 找配置指南，最后 `--domain development --module OAuth授权` 找鉴权方式。不要指望一次全域搜索解决。

## 内置资料结构

完整资料清单见 [sources.md](sources.md)。以下为常用文件速查：

### product 域（207个文件）
- `功能-智能审批-汇总.md` / `功能-智能审批-*.md` — 审批功能指南
- `功能-轻云-汇总.md` — 轻云功能指南
- `更新-2026年产品更新-*.md` — 最新版本更新
- `FAQ-智能审批QA合集.md` — 审批FAQ 334条

### development 域（41个文件）
- `API-获取accessToken.md` — OAuth2授权
- `API-智能审批API.md` — 审批服务端API
- `API-流程中心.md` — 统一流程中心SDK
- `开发-客户端文档.md` — JSAPI+客户端开发
- `开发-入门指南.md` — 开发入门+轻应用教程

### implementation 域（41个文件）
- `配置-智能审批.md` — 审批配置
- `配置-智能考勤.md` — 考勤配置
- `配置-融合中心.md` — 融合中心配置+交付流程
- `部署-实施交付.md` — 实施交付文档

## 回答格式

```
**简要结论**
[用1-2句话直接回答用户问题]

**详细说明**
[分点展开，引用具体资料内容]

**相关参考**
- 知识域：[product/development/implementation]
- 文件：[文件名]
```

注意事项：
- 标注知识域和文件名，便于溯源
- 版本敏感问题优先参考最新更新文档
- 无直接答案时明确告知，并提供线上查阅入口

## 维护陷阱

1. **Metadata module 不能用内容关键词推断**：accessToken 出现在几乎所有 API 文件中，会导致 90%+ 文件被错误归类到同一模块。必须基于文件名前缀精确映射（如 `API-获取accessToken` → OAuth授权，`API-人员` → 组织人员）。
2. **空壳文件合并**：语雀抓取会产生大量 <1KB 的标题文件（只有标题没有正文），应按模块合并为汇总文件，否则文件碎片化严重。
3. **frontmatter 与 body 分离**：检索时必须跳过 YAML frontmatter 区域再匹配关键词，否则 metadata 中的 keywords 会干扰搜索相关性。
4. **JSAPI噪音污染**：搜"回调/callback"时，客户端JSAPI文档(`开发-客户端文档.md`)中包含大量 `qing.call` 和 `callback` 参数，匹配数远超目标文件。必须用 `--exclude JSAPI qing.call` 或 `--module 流程引擎` 排除。类似地，搜"token"会被几乎所有API文件命中，需 `--module OAuth授权` 精准定位。
5. **跨域复杂问题检索策略**：涉及"审批回调ERP"这类跨域问题（产品配置+开发API+集成方案），应分步检索：先 `--domain development --module 流程引擎` 找API，再 `--domain implementation` 找配置指南，最后 `--domain development --module OAuth授权` 找鉴权方式。不要指望一次全域搜索解决。
