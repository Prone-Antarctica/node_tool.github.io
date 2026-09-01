# foreach + metadata 逐实例随机

**是什么**：For-Each（按 piece）循环里再挂一个 **metadata 类型的 Block Begin**，从它身上读 `iteration` 当随机种子，给每次迭代算一份独立随机量（这里是随机旋转角）。

**节点链**（whole_nodes，foreach2 为例）：

```
foreach_begin2 (method=piece)          foreach_begin2_metadata1 (method=metadata)
        │                                        │
  copytopoints5                        attribwrangle4 (class=detail):
        │                                f@randRot = (rand(iteration)-0.5)*2*ch("../rotationrange")
  transform3  ry = ch("../rotation") + detail(-1,"randRot",0)
        │       └ spare_input0 = ../attribwrangle4   ← 用 spare input 引 metadata 结果
foreach_end2 (itermethod=pieces, method=merge)
```

**要点**
- VEX 原文：`f@randRot=0; int tempNum=detailattrib(0,"iteration",0,0); @randRot=(rand(tempNum)-0.5)*2*ch("../rotationrange");`——rand(种子) 移到 [-1,1] 再乘幅度，是标准的"有符号随机"写法。
- transform3 的 `spare_input0` 指向 wrangle，表达式里用 `detail(-1,…)` 读第 -1 号输入 = spare input——**参数表达式引别的节点数据就靠 spare input**。
- transform3 的 pivot 用 `centroid(0,D_X/Y/Z)`：绕自身中心转，不跑位。
- 同样的结构在实例链复制了一份（foreach3 + metadata2 + attribwrangle6/7），几何用 transform 转、实例点用四元数转 N（`qrotate(quaternion(angle,{0,1,0}), @N)`）。

**出现位置**：2026-08-31_whole_nodes（foreach2 几何链、foreach3 实例链）。
