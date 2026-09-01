# UVLayout 摆放求解器

**是什么**：把 UV Layout SOP 当 2D 装箱求解器用——它本职是把 UV 岛无重叠地排进 0-1 空间，但让它直接操作 `P` 而不是 `uv`，就成了"把一堆建筑底面无重叠地排进一块地皮"的摆放器。这是本归档目前最核心的创意手法。

**节点链**（whole_nodes / test 顶层）：

```
建筑代理面(带 name) ─┐
                     ├─ copytopoints4 ─ uvlayout1 ─ foreach_begin1 ┐
撒点(带随机 name) ───┘        地块面 ─ 输入1(target)              │ 每 piece：
                                                                  │ center_point  addpoint(@P) 加中心点
                                                                  │ set_name      角点 name/N 抄给中心点
                                                                  │ Tag           中心点 delete=1
                                                     foreach_end1 ┘
                                                     delete3  只留 delete==1 → 落点集合
```

**uvlayout1 关键参数**（原文见 graph.json）：
- `uvattrib = P`、`targetuvattrib = P` —— 排的是位置不是 uv
- `projplane = zx` —— 在水平面上排布（配合上游 `@N={0,0,1}` 统一法线）
- `padding = ch("../interval")` —— **建筑间距**就是 UV padding
- `targettype = geo` + 输入1 接地块面 —— 排布范围就是这块地
- `scaling = custom`、`iterations = 5`、`randseed = 50`

**要点**
- copytopoints4 的 `applyattribs1` 排除了 `N/up/orient/rot/…` 全部变换属性：copy 只负责"每个点出一份带 name 的面片"，位置完全交给 uvlayout 重排。
- 排的是 [[bbox-footprint-proxy]] 代理面而不是完整建筑——快，而且间距按占地算。
- 落点带回 name，后面 [[name-variant-copy]] 按名实体化。

**出现位置**：2026-08-31_test（最小可复用版本）、2026-08-31_whole_nodes。
