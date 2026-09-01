# name 变体分发（copytopoints useidattrib）

**是什么**：一次 Copy to Points 把多种变体各就各位——源几何每种变体打 string `name`，目标点也随机分配一个 `name`，copytopoints 开 `useidattrib`（id 属性默认就是 `name`）后只把同名的源 copy 到同名的点上。

**节点链**（whole_nodes）：

```
源：subnet1 出口（4 种建筑分别 name=BP_Buildings_01..04 后 merge）
点：scatter1 → attribrandomize6(distribution=discrete, valuetype=string,
      strvalue0..3 = BP_Buildings_01..04) → 每点随机抽一种
copytopoints5: useidattrib=on → 按 name 匹配分发
```

**要点**
- attribrandomize 的 discrete + string 模式就是"按权重抽签"，`weight0..3` 可调各变体比例。
- 源侧的 name 在 subnet1 里用 Name SOP 打（`name1='BP_Buildings_0X'`）。
- 配合 [[uvlayout-placement-solver]]：落点的 name 一路从撒点透传到落点（set_name wrangle），实体化时不会张冠李戴。

**出现位置**：2026-08-31_whole_nodes（copytopoints5、copytopoints6 两处），2026-08-31_test（name 打标部分）。
