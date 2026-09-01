# 包围盒占地代理（footprint proxy)

**是什么**：重的建筑模型不直接参与摆放计算——先做一张"占地面片"（包围盒的顶面）当代理，摆放/求解都用代理，最后才把真模型 copy 到解出的落点上。

**节点链**（subnet1，四条同构）：

```
清理后的建筑 → bound（包围盒）
→ delete  stdswitcher=2（按法线删）, dir=(0,1,0), angle=90, affectnormal=on
          → 只留法线朝上的面 = 顶面一张四边形
→ name    Building_0X
→ merge6 → BlockPrimitive 出口（outputidx=1）
```

**要点**
- subnet1 因此有**两个出口**：出口0 完整模型（Block）、出口1 代理面（BlockPrimitive）——同一份处理、两种精度，输出口用 Output SOP 的 `outputidx` 区分。
- 代理面积 = 真实占地，所以 [[uvlayout-placement-solver]] 用它排间距是准的。
- "按法线删面"取包围盒某一面，比 blast 组名稳（不依赖 prim 编号）。

**出现位置**：2026-08-31_test / 2026-08-31_whole_nodes 的 `subnet1`（bound1-4 + delete1-4）。
