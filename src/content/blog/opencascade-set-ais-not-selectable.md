---
title: "OpenCASCADE 显示对象设置不可选中"
description: "OpenCASCADE 中把 AIS 显示对象设置为不可选中。"
pubDate: 2024-01-01
tags: ['Tech']
---

原有的选中模式代码：

```
static Standard_Integer SelectionMode (const TopAbs_ShapeEnum theShapeType)
{
    switch (theShapeType)
    {
    case TopAbs_VERTEX:    return 1;
    case TopAbs_EDGE:      return 2;
    case TopAbs_WIRE:      return 3;
    case TopAbs_FACE:      return 4;
    case TopAbs_SHELL:     return 5;
    case TopAbs_SOLID:     return 6;
    case TopAbs_COMPSOLID: return 7;
    case TopAbs_COMPOUND:  return 8;
    case TopAbs_SHAPE:     return 0;
    }
    return 0;
}
```

例如： `AIS_Shape::SelectionMode(TopAbs_EDGE)` 选中模式为边线

调用 AIS\_InteractiveContext::Display 时设置 -1 ，则对象无法选中

```
myContext->Display(theObjects,
                    AIS_Shaded,
                    selectable ? AIS_Shape::SelectionMode(TopAbs_EDGE) : -1,
                    Standard_True);
```
