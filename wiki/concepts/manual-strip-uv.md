# 手算条状 UV（河面）

**是什么**：对 skin 出来的条状面（河面这种"一串横条 prim"），用 wrangle 按 prim/点序直接算 uv，比展 UV 稳定且方向可控。

**节点链**（whole_nodes，River 链）：

```
skin3 → uvtexture1 → UV.x → UV.y → skin4 → uvtransform1(sy=10) → unreal_material
```

**VEX 原文**：

```c
// UV.x：横向 = 第几条 prim，归一化
@uv.x = float(@primnum) / float(@numprim - 1);
@Cd = @uv.x;                       // 顺手可视化

// UV.y：纵向 = 点序归一化（沿河向）
float pointUv = (@ptnum / (@numprim)) * (@numprim);
@uv.y = pointUv / float(npoints(0));
```

**要点**
- 前提是 skin 产出的 prim/点编号是规则的（cols 蒙皮天然满足）。
- `uvtransform sy=10`：uv.y 拉 10 倍 = 贴图沿河平铺 10 次，流水材质密度就靠它调。
- `@Cd=@uv.x` 是调 UV 时的可视化小技巧，最后记得删 Cd（这里由下游 attribdelete 收拾）。
- 相关：[[copy-skin-sweep]]（这条面就是它蒙出来的）、[[unreal-instance-export]]（uv 随机偏移做变体）。

**出现位置**：2026-08-31_whole_nodes（UV.x / UV.y 两个 wrangle）。
