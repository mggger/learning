# 拓扑排序

拓扑排序要解决的问题是给一个图的所有节点排序。


**Kahn算法**

1. 假设```L```是存放结果的列表,  先找到哪些入度为零的节点, 把这些节点放到```L```中, 然后把与这些节点相连的边从图中去掉， 再寻找图中
的入度为零的节点。  

2. 对于新找到的这些入度为零的节点， 他们的父节点已经都在```L```中， 所以也可以放入L。 

3. 重复上述操作， 直到找不到入度为零的节点。如果此时L中的元素个数和节点总数相同，说明排序完成；如果L中的元素个数和节点总数不同，说明原图中存在环，无法进行拓扑排序


实现算法:

```python
from collections import defaultdict


class Graph:

    def __init__(self, directed=False):

        self.graph = defaultdict(list)
        self.directed = directed

    def addEdge(self, frm, to):

        self.graph[frm].append(to)

        if self.directed is False:
            self.graph[to].append(frm)

        else:
            self.graph[to] = self.graph[to]

    def topoSortVisit(self, s, visited, sortList):

        visited[s] = True

        for i in self.graph[s]:
            if not visited[i]:
                self.topoSortVisit(i, visited, sortList)

        sortList.insert(0, s)

    def topoSort(self):

        visited = {i: False for i in self.graph}

        sortList = []

        for v in self.graph:
            if not visited[v]:
                self.topoSortVisit(v, visited, sortList)

        print(sortList)


if __name__ == '__main__':
    g = Graph(directed=True)
    g.addEdge(1, 3)
    g.addEdge(3, 5)
    g.addEdge(5, 6)
    g.addEdge(2, 4)
    g.addEdge(4, 6)
    g.addEdge(6, 7)
    g.addEdge(7, 8)
    g.addEdge(8, 9)

    print("Topological Sort:")
    g.topoSort()

##Topological Sort:
##[2, 4, 1, 3, 5, 6, 7, 8, 9]
```