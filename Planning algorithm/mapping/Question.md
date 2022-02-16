# Question

请自主分辨正确性，如有不同见解，欢迎讨论

**1.为什么可以用栅格地图做路径规划？**


答:路径规划常用的图搜索算法（Dijkstar、A*等）适用于加权有向图，而栅格图可以视为图的特殊情况：栅格中心为节点，中心连线为边。栅格及邻居栅格可以构成由边连接的节点集，这些边具有附加的数字权重（栅格移动成本）。

常见的地图表示方法请查看：

[Map representations](http://theory.stanford.edu/~amitp/GameProgramming/MapRepresentations.html)

优缺点对比：[Grid maps;Polygonal maps;Navigation Meshes;Hierarchical maps](http://theory.stanford.edu/~amitp/GameProgramming/MapRepresentations.html#graph-format-recommendations)。
