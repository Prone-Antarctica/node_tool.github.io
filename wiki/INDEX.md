# HNA 知识库索引

## 快照

| 快照 | 内容 | 详情 |
|---|---|---|
| 2026-08-31_whole_nodes | 村落生成器全网络（182 节点）：建筑摆放 + 城墙/道路/河流/水车 + UE 实例输出 | [[2026-08-31_whole_nodes]] |
| 2026-08-31_test | 上者的核心子集（53 节点）：建筑块预处理 + UVLayout 摆放求解器 | [[2026-08-31_test]] |

## 概念 / 手法

| 概念 | 一句话 | 文件 |
|---|---|---|
| UVLayout 摆放求解器 | 把 UV Layout 当 2D 装箱器用，解建筑无重叠摆放（本库最核心的创意） | [[uvlayout-placement-solver]] |
| name 变体分发 | 点上随机 string name + copytopoints useidattrib：一次 copy 分发多种变体 | [[name-variant-copy]] |
| foreach + metadata 随机 | 循环里用 iteration 当种子做逐实例随机（旋转等） | [[foreach-metadata-random]] |
| Ray 投射与避让 | 方向属性 + Ray：落地形、检测河/路碰撞后删点 | [[ray-projection-filtering]] |
| copy+skin 手工扫掠 | 横截线 copy 到路径点再 skin：路面/河面/水车网格全用它 | [[copy-skin-sweep]] |
| FBX 导入清理 | fbx_* 属性清除 + 0.01 缩放 + rx90 + bbox 落地的标准四件套 | [[fbx-import-cleanup]] |
| UE 实例输出 | unreal_instance / unreal_material / DataTable 字段改名，输出实例点而非几何 | [[unreal-instance-export]] |
| 手算条状 UV | wrangle 里 primnum/ptnum 归一化算 uv，配 uvtransform 平铺 | [[manual-strip-uv]] |
| 包围盒占地代理 | bound + 按法线删面取顶面 = 建筑 footprint，重活交给代理 | [[bbox-footprint-proxy]] |

## 怎么用（给未来的 Claude）

1. 用户问"XX 怎么做/帮我搭 XX"→ 在上表找相关概念 → 读 concepts 页拿节点链和参数要点。
2. 需要精确参数或 VEX 原文 → 按 concepts 页里标的节点路径去对应 `graph.json` 查（`parms[].raw`、`code.text`）。
3. 节点级注释在 graph.json 每个节点的 `user_data["hna.ai_note"]`。
