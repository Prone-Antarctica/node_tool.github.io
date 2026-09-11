# Houdini 节点归档（HNA）— 给 Claude 的说明

这是用户的 Houdini 节点快照归档仓库，由「HNA 同步」（houdini-node-archive 仓库的导出器）推送，GitHub Pages 服务根目录 `index.html`（在线查看器）。

## 仓库结构

- `<日期>_<名字>/graph.json` — 一个快照：Houdini 网络的完整导出（schema v1）。节点含 `type`（真实节点类型）、`parms`（键的顺序 = Houdini 参数面板顺序；raw=表达式原文 + eval=求值结果 + label/folder；2026-09-10 之后的快照连默认值参数也导出并标 `default: true`，**读的时候先过滤掉 default 的**，老快照只有非默认参数）、`code`（VEX/Python 源码）、`user_data`、连线、网络框、便签。
- `<日期>_<名字>/meta.json` — 标题、日期、节点数。
- `<日期>_<名字>/nodes.hipnc` — 对应的 Houdini 工程文件（二进制，别读）。
- `manifest.json` — 快照清单（同步时自动重建）。
- `wiki/` — 知识库（见下）。

## user_data 里的键（写在每个节点上）

- `hna.concept` / `hna.confidence` — **用户自己**标的概念和掌握度（1-5），只有用户改，Claude 不要动。
- `hna.ai_note` — **Claude 写的**节点注释（这个节点在整个网络里干什么）。可以更新。

## wiki 规则

- `wiki/INDEX.md` — 入口索引。回答用户问题、或要生成新网络时，**先读这个**再按需下钻，不要上来就读全部 graph.json。
- `wiki/concepts/<slug>.md` — 一个可复用的手法/模式一篇：是什么、节点链怎么搭、在哪些快照的哪些节点出现过。互相用 `[[slug]]` 引用。
- `wiki/snapshots/<目录名>.md` — 一个快照一篇：整体在做什么、分区块的数据流讲解。
- 新快照进来时的维护流程：读 graph.json → 给节点写 `hna.ai_note` → 写/更新 snapshots 页 → 把新出现的手法沉淀成 concepts 页（已有概念就在"出现位置"里追加）→ 更新 INDEX.md。
- 注释和 wiki 一律写中文；节点类型、属性名、参数名保留英文原文。

## 注意

- graph.json 也会被 Houdini 侧的「HNA 同步」和查看器的标注写回功能改动，改前 `git pull`。
- 修改 graph.json 只允许动 `user_data`，其他字段是导出器的输出，改了会和下次导出冲突。
- 生成新网络给用户时：先查 INDEX → 相关 concepts → 需要精确参数再去 graph.json 里查对应节点。
