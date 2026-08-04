---
name: wiki-loop
description: 发票云Wiki知识库书写+初审调度器。遍历README中所有文章，自动从飞书知识库找原始材料、起草、初审，输出结论报告给用户。FAQ终审由wiki-faq-loop单独跑。
---

# wiki-loop：Wiki 书写+初审调度器

## 触发方式

用户输入 `/wiki-loop` 启动。

一次 loop = 遍历知识库所有文章，能写的写完并通过 lint，不能写的列出原因。全部处理完后输出报告给用户。

---

## 执行流程

### 第一步：读取文章列表

读取 `0导航与索引/README.md`，提取所有文章的路径和描述，建立待处理列表。

跳过以下文章（不在处理范围）：
- 已标注"待完善"且无 description 的空文件
- README.md 本身

---

### 第二步：逐篇查找原始材料

对每篇文章，按以下优先级在飞书知识库查找原始材料：

**查找步骤：**
1. 先读对应知识库的目录树（不读全文），根据文章标题和 description 定位最相关节点
2. 找到相关节点后，读取具体文章内容
3. 判断可写程度：

| 可写程度 | 判断标准 | 处理方式 |
|---|---|---|
| ✅ 可写 | 找到完整原始文档，内容可信 | 起草文章 |
| ⚠️ 部分可写 | 找到片段信息，有框架但细节不全 | 起草框架，关键细节标【待确认】 |
| ❌ 不可写 | 完全找不到原始材料 | 跳过，记录在报告里 |

**飞书知识库查找优先级：**

主要参考（优先找这两个）：
- 知识与运营部：https://icn1dae2f6c3.feishu.cn/wiki/RHxgwuPLEiyJQQkGqFYcj3nBnUf
- 运营管理：https://icn1dae2f6c3.feishu.cn/wiki/TjmQwa4Eui65wakW1DqcsgEQnvh

补充参考（主要参考找不到时再找）：
- 交付服务（对外）：https://icn1dae2f6c3.feishu.cn/wiki/NgZ3wbivFixNUckxyrlcfU7Wndh
- 营销管理（对内）：https://icn1dae2f6c3.feishu.cn/wiki/HYBmwUn27iQj87k0Kamc6PbWngf
- 营销管理（对外）：https://icn1dae2f6c3.feishu.cn/wiki/DA74wuEK3iED4wkfm1xcFKUInjb
- 交付管理（对内）：https://icn1dae2f6c3.feishu.cn/wiki/I8UWwVvLGiimIHkc0xhcuAmgnQb
- 产研工程：https://icn1dae2f6c3.feishu.cn/wiki/UeXnw7f8jivAcykqINUcdtZWnVf
- 伙伴管理（对外）：https://icn1dae2f6c3.feishu.cn/wiki/W3IzwdlAWisu3fkJ2V7c3HVGnhd
- 国际区产品：https://icn1dae2f6c3.feishu.cn/wiki/MZT3wW6NyiGpfzk6btBc2haCnLb
- 中国区产品（对外）：https://icn1dae2f6c3.feishu.cn/wiki/LWiYwnuC5iJYFLkj8fwcPXcWnIb
- 国际产品部（内部）：https://icn1dae2f6c3.feishu.cn/wiki/VR3CwloD9iCPEMkoIOMc3BtSnfb
- 中国区产品部门（对内）：https://icn1dae2f6c3.feishu.cn/wiki/ZllswsVphi0TItkv5v3cO3jbnLb

会议纪要参考（辅助补充，语音识别可能有文字错误，仅作参考，不作主要来源）：
- EOP Wiki 本地文档：E:\EOP-wiki（0430）
- 会议纪要1：https://icn1dae2f6c3.feishu.cn/docx/TQyAdtHkKomjv5xRMXRcbtP0nfg
- 会议纪要2：https://icn1dae2f6c3.feishu.cn/docx/KHoddBG06oj9BCxr1q6cIkjCnyh
- 会议纪要3：https://icn1dae2f6c3.feishu.cn/docx/Nbi1dvTDSoWH7RxQ2xicA0wdnpf
- 会议纪要4：https://icn1dae2f6c3.feishu.cn/docx/Ooj3dM8t1oHsTwxrLk5c6l7VnR5
- 会议纪要5：https://icn1dae2f6c3.feishu.cn/docx/A05TdO7azo6ESqxawfnc9vVxnic
- 会议纪要6：https://icn1dae2f6c3.feishu.cn/docx/Rjq2dd7Ico9hvoxKPQWcD14Znqh

---

### 第三步：批量起草

对所有 ✅ 和 ⚠️ 文章，按 wiki-write skill 规范起草。

起草完成后，**一次性输出所有草稿给用户确认**，等用户确认内容准确后进入第四步。

---

### 第四步：初审（wiki-lint）

用户确认草稿后，对每篇文章**各自启动一个独立无记忆子 agent** 执行 wiki-lint：

- 每篇文章独立子 agent，不共享书写阶段上下文
- 自动修复所有阻断项，直到该篇阻断项清零
- 所有文章阻断项全部清零后，进入第五步

---

### 第五步：Git 存档

输出报告前，在知识库目录下执行 git commit，标记本次 loop：

```bash
cd "E:/运营管理Wiki"
git add -A
git commit -m "wiki-loop #<N>：完成 <完成数> 篇，跳过 <跳过数> 篇，待确认 <待确认数> 项"
git push
```

commit message 格式固定，`#<N>` 为本次 loop 的序号（第一次跑是 #1，第二次是 #2，依此类推）。序号从 git log 里统计已有的 wiki-loop commit 数量 +1 得出。

---

### 第六步：输出结论报告

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Wiki 书写+初审报告
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 本次处理文章总数：<N> 篇
✅ 完成（通过lint）：<N> 篇
⚠️ 部分完成（含【待确认】）：<N> 篇
❌ 跳过（缺原始材料）：<N> 篇

❌ 跳过的文章（需你补充原始材料后下次再跑）：
- <文章路径>：<找不到的原因>

⚠️ 需要你决策的【待确认】项：
- <文章路径> → <待确认内容>

📋 下一步：
- 补充以上缺失材料后，重新执行 /wiki-loop
- FAQ 文档准备好后，执行 /wiki-faq-loop 做终审
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 重要原则

- **一次 loop 不一定能写完所有文章**，缺材料的文章下次补充材料后再跑
- **loop 是迭代的**，每次跑完报告告诉你缺什么，你补充后继续跑，逐步完善
- **lint 必须职责隔离**：每篇文章独立无记忆子 agent，不能用书写时的同一个 agent
- **宁可不写，不可乱写**：找不到原始材料宁可标 ❌ 跳过，不自行脑补
