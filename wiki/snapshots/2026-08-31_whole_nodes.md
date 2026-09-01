# 快照：whole_nodes — 村落生成器（面向 Unreal）

- 网络：`/obj/trynew_copy_1_1/trynew_copy1`（SOP subnet，182 节点），Houdini 21.0.671，2026-08-31 导出。
- 做什么：给一块地皮（地块线 + 地形 + 城墙线 + 道路线 + 河道线，从子网输入进来）生成一个完整村落——建筑无重叠摆放、城墙、路面、河面、沿河水车——并同时输出**几何版**和 **UE 实例点版**。
- HDA 暴露的参数（网络里 `ch("../xxx")` 引用到的）：`npts` 建筑数量、`interval` 建筑间距、`rotation` 全局朝向、`rotationrange` 随机朝向幅度、`road_width`、`RiverWidth`、`Wall_Height`、`Watermill_width`、`Watermill_density`、`input` 输出切换、`objpath1` UE DataTable 点云路径。

## 数据流总览

```
子网输入 ──► 地块线─skin1─transform2(0.8)─► null4 地块面
            地形──► LandScape    城墙线──► Wall 链    道路线──► Road 链    河道线──► River 链

subnet1（建筑块预处理 [[fbx-import-cleanup]]）
  ├─ 出口0 Block：4 种完整建筑（name=BP_Buildings_0X）
  └─ 出口1 BlockPrimitive：4 种占地代理面 [[bbox-footprint-proxy]]

撒点：null4 → scatter1(npts) → attribrandomize6（随机 name）→ Scatter_points

摆放核心 [[uvlayout-placement-solver]]：
  建筑源+撒点 → copytopoints4 → uvlayout1(padding=interval, target=地块面)
  → foreach1 每面片取中心点(center_point/set_name/Tag/delete3) → 落点(带 name)

落点过滤：polyextrude 选区体+group1 圈地块内 → ray4 落地形
  → ray5 向上打 河+路 → delete5/6 删碰撞点 [[ray-projection-filtering]]

实体输出：foreach2 [[foreach-metadata-random]] → copytopoints5(useidattrib name)
  [[name-variant-copy]] → transform3 逐栋随机旋转 → +城墙 → switch1 → output0

UE 实例输出 [[unreal-instance-export]]：object_merge2(DataTable) → attribute1 字段改名
  → @P=0 → copytopoints6 到落点 → attribwrangle7 N 四元数旋转 → wall_bpbuilding 合流
```

## 分区块

| 区块 | 节点 | 说明 |
|---|---|---|
| 建筑块预处理 | `subnet1/*` | 4 个 FBX：清 fbx_* 属性 → 0.01 缩放+rx90 → 居中 → bbox 落地 → 打 name；另出一套 bound+delete 的占地面片 |
| 地块皮 | attribdelete2→skin1→transform2→null4 | 地块线蒙皮、缩 0.8 留边；是 uvlayout 的排布目标 |
| 撒点 | attribcreate1→scatter1→attribrandomize6→Scatter_points | 每点 discrete 随机抽一种建筑 name |
| 摆放求解 | copytopoints4→uvlayout1→foreach1→delete3 | 见 [[uvlayout-placement-solver]]，全网络最有含金量的部分 |
| 落点过滤 | group1/delete4、ray4、ray5、delete5/6 | 圈地块范围、落地形、避河避路 |
| 建筑实体 | foreach2 + copytopoints5 + transform3 | 按 name 分发变体、逐栋随机旋转 |
| 城墙 | resample1→ray2→copytopoints2(box2)→Wall | 墙线投地形，box 墙段沿线复制，Wall_Height 控高 |
| 道路 | Road_1→resample2→copytopoints3(line1)→skin2 | [[copy-skin-sweep]]，road_width 控宽 |
| 河流 | null1→resample3→copytopoints1(line2)→skin3→UV.x/UV.y→unreal_material→River | 扫掠 + [[manual-strip-uv]] + M_River_Inst 材质 |
| 水车 | WaterMill 子网 + Wattermill_all | 河向蒙皮取 3 列点（ptnum%3 拆列），Watermill_L/R 分列对位复制 |
| UE 实例链 | attribute1→copytopoints6→attribwrangle7→wall_bpbuilding | 输出 unreal_instance 点；unreal_material/uv 各三处 attribrandomize 做外观变体 |
| 调试出口 | switch1(input) | 0建筑 1+墙 2墙 3UE合流 4实例foreach 5河 |

## 观察 / 可改进

- 三组 unreal_material/uv 随机化链（attribcreate5/6/9 + randomize 8/11、13/12、14/15）是复制粘贴的重复，可收进一个 foreach 或 HDA。
- attribcreate7、attribwrangle8、attribrandomize7 是悬空备用链，未接入主流程。
- 种子几乎全是 5986，说明"整体可复现"，但想每链独立变化时要改成不同种子。
- subnet1 里 4 条 FBX 链也是四份复制粘贴，天然适合 foreach + 文件名参数化。
