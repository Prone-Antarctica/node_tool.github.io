# copy + skin 手工扫掠

**是什么**：不用 Sweep SOP，自己拼扫掠：横截面 Line copy 到重采样后的路径点上，再 Skin 蒙皮成面。宽度、密度全是明确的参数，行为完全可控。

**节点链**（三处同构）：

```
路径线 → resample(length=密度) ─┐
                                ├ copytopoints ─ skin
横截 line(dist=宽, origin=-宽/2)┘
```

| 用途 | 路径重采样 | 横截线 | skin | 备注 |
|---|---|---|---|---|
| 路面 | resample2 (1.282) | line1 `dist=ch('../road_width')` points=9 | skin2 | |
| 河面 | resample3 (3.29) | line2 `dist=ch('../RiverWidth')` points=3 | skin3 (cols,force) | 后接 [[manual-strip-uv]] |
| 水车点阵 | resample4 `ch('../../Watermill_density')` | line3 `ch('../../Watermill_width')` points=3 | skin5 (cols) | 蒙完不要面，要 3 列点 |

**要点**
- 横截线 `originz = -宽*0.5` 让路径线居中。
- skin `surftype=cols` 只沿列连——水车链靠它得到规则 3 列点阵，再用 wrangle `ptnum%3` 拆左/中/右列（1R/2MIDDLE/2L），把 Watermill_L/R 模型分列 copy——**skin 蒙皮点阵当放置网格**是这条链的彩蛋。
- 宽度类参数全部 `ch()` 到 HDA 上，横截 points 数决定面的分段。

**出现位置**：2026-08-31_whole_nodes（Road、River、WaterMill 三链）。
