# Ray 投射与避让过滤

**是什么**：用"方向属性 + Ray SOP"做两件事：① 把点贴到某个面上（投射）；② 探测点上方/下方有没有某类几何，命中的删掉（避让）。

**节点链**（whole_nodes）：

```
① 落地形：attribwrangle2  v@Dir={0,-1,0}
          ray4  dirmethod=attribute, dirattrib=Dir, 输入1=地形 → 点贴到地形表面

② 避河避路：attribwrangle3  v@Dire={0,1,0}
            ray5  dirattrib=Dire, entity=point, newgrp=on → 命中进 rayHitGroup
                  输入1 = merge2(河面+路面)
            delete5 / delete6  删 rayHitGroup → 建筑不长在河和路上

③ 城墙贴地(反向用法)：attribcreate2 点 N=(0,-1,0) → ray2(输入1=地形, newgrp)
                      delete2 negate=keep → 只留"打得中地形"的墙点
```

**要点**
- Ray 默认沿点法线 N 投；要自定义方向就 `dirmethod=attribute` + 一个向量属性。
- `newgrp=on` 生成 rayHitGroup 后，**删命中**（避让）和**只留命中**（裁掉悬空段）是同一开关的两种用法（delete 的 negate）。
- 圈范围的兄弟手法：地块面 bound → polyextrude(pointnormal) 挤成选区体 → groupcreate `groupbounding + usebconvex` 圈出体内点（group1 → delete4）。

**出现位置**：2026-08-31_whole_nodes（ray2/ray4/ray5 + group1 选区）。
