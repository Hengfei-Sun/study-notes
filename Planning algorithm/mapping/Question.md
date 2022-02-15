# Question

请自主分辨正确性，如有不同见解，欢迎讨论

**1.为什么路径规划常用栅格地图？**


答:路径规划常用的图搜索算法（Dijkstar、A*等）适用于加权有向图，而栅格图可以视为图的特殊情况：栅格中心为节点，中心连线为边。栅格及邻居栅格可以构成由边连接的节点集，这些边具有附加的数字权重（栅格移动成本）。

（与其他图对比）常见的图形算法主要有以下几种：[栅格法，拓扑法，自由空间法和可视法](https://www.cnblogs.com/DissertationSubmitted/p/14975782.html)。

有关图的更多理论请查看：

[Grid and Graphs](https://www.redblobgames.com/pathfinding/grids/graphs.html)

[Map representations](http://theory.stanford.edu/~amitp/GameProgramming/MapRepresentations.html)

[Grid pathfinding optimizations](https://www.redblobgames.com/pathfinding/grids/algorithms.html)

[Wikipedia: Graph(abstract data type)](https://en.wikipedia.org/wiki/Graph_(abstract_data_type))