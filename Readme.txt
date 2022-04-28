1. Ford-Fulkerson Algorithm

Ford-Fulkerson 알고리즘은 flow network의 최대 흐름을 계산하는 그리디 알고리즘이다.

최초의 네트워크 유량 알고리즘.
네트워크의 모든 간선의 유량을 0으로 시작, Source에서 Sink로 보낼 수 있는 경로로 유량을 보내기를 반복하면 된다.

잔여 그래프(residual graph)의 augmenting path를 찾는 접근법이 완전히 명시되지 않거나 실행 시간이 
다른 여러 구현에서 지정되기 때문에 "algorithm" 대신 "method"라고 부르기도 한다.

네트워크 입력 G=(V,E) 흐름 용량 c, 소스 노드 s 및 싱크 노드 t
출력 최대값의 s에서 t까지의 f 흐름 계산
1. f(u,v) <<  0어느 모서리에서 보아도 (u,v)
2. s에서 t까지 경로 p가 있는 동안 G_f, c_f(u,v)>0어느 모서리에서 보아도 (u,v) in p
          1. c_f(p) = min{c_f(u,v):(u,v) in p} 찾기
          2. 각 모서리마다(u,v) in p
                    1. f(u,v) << f(u,v) + c_f(p) (경로를 따라 흐름 전송)
                    2. f(v,u) << f(v,u) - c_f(p) (나중에 흐름이 반환될 수 있음)

2. Code

import collections

class Graph:
    """This class represents a directed graph using adjacency matrix representation."""

    def __init__(self, graph):
        self.graph = graph  # residual graph
        self.row = len(graph)

    def bfs(self, s, t, parent):
        """Returns true if there is a path from source 's' to sink 't' in
        residual graph. Also fills parent[] to store the path."""

        # Mark all the vertices as not visited
        visited = [False] * self.row

        # Create a queue for BFS
        queue = collections.deque()

        # Mark the source node as visited and enqueue it
        queue.append(s)
        visited[s] = True

        # Standard BFS loop
        while queue:
            u = queue.popleft()

            # Get all adjacent vertices of the dequeued vertex u
            # If an adjacent has not been visited, then mark it
            # visited and enqueue it
            for ind, val in enumerate(self.graph[u]):
                if (visited[ind] == False) and (val > 0):
                    queue.append(ind)
                    visited[ind] = True
                    parent[ind] = u

        # If we reached sink in BFS starting from source, then return
        # true, else false
        return visited[t]

    # Returns the maximum flow from s to t in the given graph
    def edmonds_karp(self, source, sink):

        # This array is filled by BFS and to store path
        parent = [-1] * self.row

        max_flow = 0  # There is no flow initially

        # Augment the flow while there is path from source to sink
        while self.bfs(source, sink, parent):

            # Find minimum residual capacity of the edges along the
            # path filled by BFS. Or we can say find the maximum flow
            # through the path found.
            path_flow = float("Inf")
            s = sink
            while s != source:
                path_flow = min(path_flow, self.graph[parent[s]][s])
                s = parent[s]

            # Add path flow to overall flow
            max_flow += path_flow

            # update residual capacities of the edges and reverse edges
            # along the path
            v = sink
            while v != source:
                u = parent[v]
                self.graph[u][v] -= path_flow
                self.graph[v][u] += path_flow
                v = parent[v]

        return max_flow

3. Result

graph = [[0, 8, 0, 0, 3, 0],
            [0, 0, 9, 0, 1, 0],
            [0, 1, 0, 16, 7, 2],
            [10, 0, 0, 8, 0, 5],
            [0, 2, 7, 4, 0, 0],
            [9, 0, 0, 12, 0, 0]]

Max Flow: 7