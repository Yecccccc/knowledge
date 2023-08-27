1. 滑动窗口

   ```cpp
   int left = 0, right = 0;
   while (right < s.size()) {
       window.add(s[right]);
       right++;
       while (window needs shrink) {
           window.remove(s[left]);
           left++;
       }
   }
   
   ```

2. 二分搜索

   ```c++
   // 普通二分
   int binary_search(vector<int> nums, int target) {
       int left = 0, right = nums.size() - 1;
       while (left <= right) {
           int mid = left + (right - left)/ 2;
           if (nums[mid] < target) {
               left = mid + 1;
           }
           else if (nums[mid] > target) {
               right = mid - 1;
           }
           else if (nums[mid] == target) {
               return mid;
           }
       }
       return -1;
   }
   
   // 搜索左侧，右侧收缩
   int left_bound(vector<int> nums, int target) {
       int left = 0, right = nums.size() - 1;
       while (left <= right) {
           int mid = left + (right - left)/ 2;
           if (nums[mid] < target)
               left = mid + 1;
          	else
               right = mid - 1;
       }
       if (left < 0 || left >= nums.size())
           return -1;
       return nums[left] == target? left : -1;
   }
   
   // 搜索右侧，左侧收缩
   int right_bound(vector<int> nums, int target) {
       int left = 0, int right = nums.size() - 1;
       while (left <= right) {
           int mid = left + (right - left)/ 2;
           if (nums[mid] > target)
               right = mid -1;
           else
               left = mid + 1;
       }
   	if (right < 0 || right >= nums.size())
           return -1;
       return nums[right] == target? right: -1;
   }
   ```

3. 归并——后序遍历

   ```C++
   void sort(vector<int>& nums, int lo, int hi){
       if (lo == hi)
           return;
       int mid = lo + (hi - lo) / 2;
       sort(nums, lo, mid);
       sort(nums, mid, hi);
       merge(nums, lo, mid, hi);
   }
   void merge(vector<int>&nums, int lo, int mid, int hi) {
       vector<int> temp(nums.size());
       for (int i = lo; i <= hi; i++)
           temp[i] = nums[i];
       int i = 0, j = mid + 1;
       for (int p = lo; p <= hi; p++) {
           if (i == mid + 1)
               nums[p] = temp[j++];
           else if (j == hi + 1)
               nums[p] = temp[i++];
          	else if (temp[i] > temp[j])
               nums[p] = temp[j++];
           else
               nums[p] = temp[i++];
       }
   }
   ```

4. 快排——先序遍历

   ```C++
   void shuffle (vector<int>& nums) {
       int n = nums.size();
       for (int i = 0; i < n; i++) {
           int r = i + rand() % (n - i);
           swap(nums[i], nums[r]);
       }
   }
   int partition(vector<int>&nums, int lo, int hi) {
       int pivot = nums[lo];
       int i = lo + 1, j = hi;
       while (i <= j) {
           while (i < hi && nums[i] <= pivot)
               i++;
           while (j > lo && nums[j] > pivot)
               j--;
           if (i >= j)
               break;
           swap(nums[i], nums[j]);
       }
       swap(nums[lo], nums[j]);
       return j;
   }
   void sort (vector<int> & nums, int lo, int hi) {
       if (lo >= hi)
           return;
       int p = partition(nums, lo, hi);
       sort(nums, lo, p - 1);
       sort(nums, p + 1, hi);
   }
   vector<int> sortArray(vector<int>& nums) {
       shuffle(nums);
       sort(nums, 0, nums.size() - 1);
       return nums;
   }
   ```

5. 并查集

   ```C++
   class UF {
   private:
       int count;
       // 节点 x 的父节点是 parent[x]
       vector<int> parent;
   public:
       UF(int n) {
           this->count = n;
           parent.resize(n);
           for (int i = 0; i < n; i++)
               parent[i] = i;
       }
       void myunion(int p, int q) {
           int rootP = find(p);
           int rootQ = find(q);
           if (rootP == rootQ)
               return;
           parent[rootQ] = rootP;
           // 两个连通分量合并成一个连通分量
           count--;
       }
       int find(int x) {
           if (parent[x] != x) {
               parent[x] = find(parent[x]);
           }
           return parent[x];
       }
       bool connected(int p, int q) {
           int rootP = find(p);
           int rootQ = find(q);
           return rootP == rootQ;
       }
       int count() {
           return count;
       }
   }
   
   ```

6. 迪杰斯特拉(BFS)

   ```c++
   using pi = pair<int, int>;
   // 输入一个起点 start，计算从 start 到其他节点的最短距离
   vector<int> dijkstra(int start, vector<vector<pi>>& graph) {
       // 定义：distTo[i] 的值就是起点 start 到达节点 i 的最短路径权重
       vector<int> distTo(graph.size(), INT_MAX);
       // base case，start 到 start 的最短距离就是 0
       distTo[start] = 0;
       // 优先级队列，distFromStart 较小的排在前面
       priority_queue<pi> pq;
       // 从起点 start 开始进行 BFS
       pq.push(pi(start, 0));
       while (!pq.empty()) {
           pi curState = pq.top();
           pq.pop();
           // first代表id, second代表距离出发点的距离
           int curNodeID = curState.first;
           int curDistFromStart = curState.second;
   		// 如果这个点离start距离比目前保存的还大，则退出
           if (curDistFromStart > distTo[curNodeID]) {
               continue;
           }
           // 将 curNode 的相邻节点装入队列
           for (pair<int, int>& neighbor : graph[curNodeID]) {
               int nextNodeID = neighbor.first;
               int distToNextNode = distTo[curNodeID] + neighbor.second;
               // 更新 dp table
               if (distTo[nextNodeID] > distToNextNode) {
                   distTo[nextNodeID] = distToNextNode;
                   pq.push(State(nextNodeID, distToNextNode));
               }
           }
       }
       return distTo;
   }
   ```

7. 拓扑排序

   将后序遍历的结果进行反转，就是拓扑排序的结果

   ```C++
   class Solution {
   public:
       // 记录后序遍历结果
       vector<int> postorder;
       // 记录是否存在环
       bool hasCycle = false;
       vector<bool> visited, onPath;
       // 主函数
       vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
           vector<vector<int>> graph = buildGraph(numCourses, prerequisites);
           visited = vector<bool>(numCourses, false);
           onPath = vector<bool>(numCourses, false);
           // 遍历图
           for (int i = 0; i < numCourses; i++) {
               traverse(graph, i);
           }
           // 有环图无法进行拓扑排序
           if (hasCycle) {
               return {};
           }
           // 逆后序遍历结果即为拓扑排序结果
           reverse(postorder.begin(), postorder.end());
   		return postorder;
       }
   
       // 图遍历函数
       void traverse(vector<vector<int>>& graph, int s) {
           if (onPath[s]) {
               // 发现环
               hasCycle = true;
           }
           if (visited[s] || hasCycle) {
               return;
           }
           // 前序遍历位置
           onPath[s] = true;
           visited[s] = true;
           for (int t : graph[s]) {
               traverse(graph, t);
           }
           // 后序遍历位置
           postorder.push_back(s);
           onPath[s] = false;
       }
   
       // 建图函数
       vector<vector<int>> buildGraph(int numCourses, vector<vector<int>>& prerequisites) 		{
           // 图中共有 numCourses 个节点
           auto* graph = new vector<int>[numCourses];
           for (int i = 0; i < numCourses; i++) {
               graph[i] = vector<int>();
           }
           for (auto& edge : prerequisites) {
               int from = edge[1], to = edge[0];
               // 添加一条从 from 指向 to 的有向边
               // 边的方向是「被依赖」关系，即修完课程 from 才能修课程 to
               grap[from].push_back(to);
           }
           return graph;
       }
   
   ```
   
8. 最小生成树

   ```c++
   bool cmp(const vector<int>& a, const vector<int>& b) {
       return a[2] > b[2];
   }
   // kruskal 所有边按照权重排序，从最小的开始，如果加入不形成环，则成为最小生成树的一部分
   int kruskal(int n, vector<vector<int>> connections) {
       UF uf(n + 1);
       sort(connections.begin(), connections.end(), cmp);
       int mst = 0;
       for (auto edge: connections) {
           int u = edge[0];
           int v = edge[1];
           int weigt = edge[2];
           // 有环
           if (uf.connected(u, v))
               continue;
           mst += weight;
           uf.myunion(u, f);
       }
       return uf.count() == 2 ? mst : -1;
   }
   ```

   ```C++
   // prim算法
   class Prin{
   private:
        // 核心数据结构，存储「横切边」的优先级队列
       // 三元组 {from, to, weight} 表示一条边
       priority_queue<vector<int>, vector<vector<int>>, greater<vector<int>>> pq;
       // 类似 visited 数组的作用，记录哪些节点已经成为最小生成树的一部分
       vector<bool> inMST;
       // 记录最小生成树的权重和
       int weightSum = 0;
       // graph 是用邻接表表示的一幅图，
       // graph[s] 记录节点 s 所有相邻的边
       vector<vector<int>> graph;
   public:
       Prim(vector<vector<int>> graph) {
           // 图中有 n 个节点
           int n = graph->size();
           this->inMST.resize(n);
           // 随便从一个点开始切分都可以，我们不妨从节点 0 开始
           inMST[0] = true;
           cut(0);
           // 不断进行切分，向最小生成树中添加边
           while(!pq.empty()) {
               vector<int> edge = pq.top();
               pq.pop();
               int to = edge[1];
               int weight = edge[2];
               if (inMST[to]) {
                   // 节点 to 已经在最小生成树中，跳过
                   // 否则这条边会产生环
                   continue;
               }
               // 将边 edge 加入最小生成树
               weightSum += weight;
               inMST[to] = true;
               // 节点 to 加入后，进行新一轮切分，会产生更多横切边
               cut(to);
           }
       }
   
       // 将 s 的横切边加入优先队列
       void cut(int s) {
           // 遍历 s 的邻边
           for (auto edge : graph[s]) {
               int to = edge[1];
               if (inMST[to]) {
                   // 相邻接点 to 已经在最小生成树中，跳过
                   // 否则这条边会产生环
                   continue;
               }
               // 加入横切边队列
               pq.push(edge);
           }
       }
   
       // 最小生成树的权重和
       int weightSum() {
           return weightSum;
       }
   
       // 判断最小生成树是否包含图中的所有节点
       bool allConnected() {
           for (bool connected : inMST) {
               if (!connected) {
                   return false;
               }
           }
           return true;
       }
   };
   ```

9. BFS

   ```C++
   int BFS(Node start, Node target) {
       queue<Node> q;
       unordered_set<Node> visited;
       q.push(start);
       int step = 0;
       while(!q.empty()) {
           int sz = q.size();
           for (int i = 0; i < sz; i++) {
               Node cur = q.front();
               q.pop();
               if (cur == target)
                   return step;
               for (Node x: cur.adj()){
                   if (!visited.count(x)) {
                       q.push(x);
                       visited.insert(x);
                   }
               }
           }
           step++;
       }
   }
   ```

10. LRU

    ```c++
    class LRUcahce {
    public:
        list<pair<int, int>> cache;
        unordered_map<int, list<pair<int, int>>::iterator> map;
        int get(int key) { 
            if(!map.count(key))
                return -1;
            else {
                // 需要把它放到list头
                auto key_value = *map[key];
                cache.erase(map[key]);
                cache.push_front(key_value);
                map[key] = cache.begin();
                return (*map[key]).second;
            }
        }
        void put(int key, int value) {
            if (map.count(key)) {
                cache.erase(map[key]);
            }
            else {
                // 需要插入
                if(cache.size() == cap) {
                    // 删除尾巴
                    map.erase(cache.back().first);
                    cache.pop_back();
                }
            }
            cache.push_front({key, value});
            map[key] = cache.begin();
        }
    }
    ```

11. 树状数组

    ```C++
    // 树状数组模板
    class BIT {
        vector<int> tree;
    public:
        BIT(int n) : tree(n) {}
        // 将下标 i 上的数加一
        void inc(int i) {
            while (i < tree.size()) {
                ++tree[i];
                i += i & -i;
            }
        }
    
        // 返回闭区间 [1, i] 的元素和
        int sum(int x) {
            int res = 0;
            while (x > 0) {
                res += tree[x];
                x &= x - 1;
            }
            return res;
        }
    
        // 返回闭区间 [left, right] 的元素和
        int query(int left, int right) {
            return sum(right) - sum(left - 1);
        }
    };
    ```

    
