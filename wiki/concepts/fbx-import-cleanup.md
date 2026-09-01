# FBX 导入清理四件套

**是什么**：外部 FBX（UE/DCC 资产）进 Houdini 的标准预处理，让模型"干净、米制、Y-up、原点在脚下"。

**节点链**（subnet1，四条同构链）：

```
file (BP_Buildings_0X.fbx)
→ attribdelete   ptdel: fbx_rotation fbx_scale fbx_translation   vtxdel: N uv
→ transform      scale=0.01（厘米→米） rx=90（Z-up→Y-up）
→ matchsize      justifytarget=origin（居中到原点）
→ transform      ty = -bbox(0,D_YMIN)（底面落到 y=0）
→ name           name=BP_Buildings_0X
```

**要点**
- `fbx_*` 属性是 FBX 导入器塞的变换记录，不删会在下游 copy/instance 时造成意外偏移。
- 顺手删 `N/uv`（后面要么重算要么不需要），减少属性拖累。
- `-bbox(0,D_YMIN)` 是"落地"的惯用表达式；居中(matchsize)在先、落地在后，顺序不能反。
- 每条链最后打 name，为 [[name-variant-copy]] 做准备；同一批模型还会走 [[bbox-footprint-proxy]] 出占地面。
- 四条链是复制粘贴的，多资产时值得改成 foreach + 文件名参数。

**出现位置**：2026-08-31_test / 2026-08-31_whole_nodes 的 `subnet1`。
