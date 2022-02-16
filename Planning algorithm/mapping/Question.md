# Question

请自主分辨正确性，如有不同见解，欢迎讨论

**1.为什么可以用栅格地图做路径规划？**


答:路径规划常用的图搜索算法（Dijkstar、A*等）适用于加权有向图，而栅格图可以视为图的特殊情况：栅格中心为节点，中心连线为边。栅格及邻居栅格可以构成由边连接的节点集，这些边具有附加的数字权重（栅格移动成本）。

常见的地图表示方法请查看：

[Map representations](http://theory.stanford.edu/~amitp/GameProgramming/MapRepresentations.html)

优缺点对比：

[Grid maps;Polygonal maps;Navigation Meshes;Hierarchical maps](http://theory.stanford.edu/~amitp/GameProgramming/MapRepresentations.html#graph-format-recommendations)。

**2.如何提高在栅格地图上的搜索效率?**


答：一般来说，图搜索算法必须处理的节点越少，它运行的速度就越快。

① 可以通过合理缩减地图大小（指节点和边的数量）来提升效率：常用方法有航点法、导航网格法、分层法、四叉树、八叉树（3D）等，另外还可以使用分层级地图。

② 利用节点的额外信息提升搜索速度：例如往一个方向搜索往往会是更好的选择，A*是一个一个格点搜索，而Jump Point Search可以跳转搜索。

③ 更简单的优先级队列：使用堆实现优先级队列、使用分区优先级队列、批处理优先级队列中的节点等。

④ 改进启发式：启发式的目的是估计最短路径的长度。启发式越接近实际最短路径长度，A*运行得越快。另外，在一些网格地图中，有许多相同长度的路径，使用Tie break可以大大降低扩展结点。

更多搜索优化方法请查看：

[Grid pathfinding optimizations](https://www.redblobgames.com/pathfinding/grids/algorithms.html)