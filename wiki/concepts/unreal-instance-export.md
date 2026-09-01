# UE 实例输出（unreal_instance / unreal_material）

**是什么**：给 Unreal（Houdini Engine）输出**实例点**而不是几何——点上带 `unreal_instance`（资产路径）就会在 UE 里生成实例；`unreal_material` 指定材质。资产清单从 UE 的 DataTable 点云读进来。

**节点链**（whole_nodes）：

```
object_merge2  objpath1 = chsop("../objpath1")   ← HDA 参数指到 DataTable 点云
→ attribute1   字段改名：
    unreal_data_table_1_Name  → name
    unreal_data_table_2_Group → Group
    unreal_data_table_3_Actor → unreal_instance
    （顺手删 unreal_data_table_rowname/rowstruct）
→ attribwrangle5  @P=0（实例定义点归零，copy 只带属性）
→ copytopoints6   useidattrib(name)：按落点 name 分发 → 每个落点一个 unreal_instance 点
→ attribwrangle7  N 四元数旋转（朝向，见 [[foreach-metadata-random]]）
→ attribdelete6   negate=on，只留 unreal_instance P N
→ wall_bpbuilding merge 合流 → switch1 input=3 输出
```

**要点**
- DataTable 改名是关键一跳：UE 表字段 → Houdini 惯用属性，之后整个 [[name-variant-copy]] 体系原样适用。
- 实例点只需要 `P / N(朝向) / unreal_instance`，其余属性全删干净。
- 材质变体：`attribcreate(unreal_material)` + 两级 attribrandomize（material 一次、uv 一次）——同一 mesh 在 UE 里随机换材质/贴图偏移。材质路径写法见 River 链：`/Script/Engine.MaterialInstanceConstant'/Game/Meshes/M_River_Inst.M_River_Inst'`。
- 按 Group 筛条目：`delete keep @Group==WaterMill` 把水车从表里挑出来单独走链。

**出现位置**：2026-08-31_whole_nodes（实例链 + River 材质 + WaterMill 筛选）。
