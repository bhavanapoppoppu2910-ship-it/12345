#include <stdio.h>
#include <stdlib.h>
#define MAX 100

int V, E;
int adj[MAX][MAX];
int disc[MAX], low[MAX], parent[MAX];
int time_dfs, bccCount;

struct Edge { int u, v; } stackEdges[MAX * MAX];
int top = -1;

void pushEdge(int u, int v) {
    stackEdges[++top] = (struct Edge){u, v};
}

void popUntil(int u, int v) {
    while (top >= 0) {
        struct Edge e = stackEdges[top--];
        if ((e.u == u && e.v == v) || (e.u == v && e.v == u)) break;
    }
}

void BCCUtil(int u) {
    int children = 0;
    disc[u] = low[u] = ++time_dfs;

    for (int v = 0; v < V; v++) {
        if (adj[u][v]) {
            if (disc[v] == -1) {
                children++;
                parent[v] = u;
                pushEdge(u, v);
                BCCUtil(v);

                if (low[v] >= disc[u]) {  // found a BCC
                    bccCount++;
                    popUntil(u, v);
                }

                if (low[v] < low[u]) low[u] = low[v];
            }
            else if (v != parent[u] && disc[v] < disc[u]) {
                pushEdge(u, v);
                if (disc[v] < low[u]) low[u] = disc[v];
            }
        }
    }
}

void findBCCs() {
    for (int i = 0; i < V; i++) {
        disc[i] = -1;
        parent[i] = -1;
    }

    time_dfs = 0;
    bccCount = 0;
    top = -1;

    for (int i = 0; i < V; i++) {
        if (disc[i] == -1) {
            BCCUtil(i);

            if (top >= 0) {  // leftover edges form a BCC
                bccCount++;
                top = -1;
            }
        }
    }
}

int main() {
    scanf("%d %d", &V, &E);

    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            adj[i][j] = 0;

    for (int i = 0; i < E; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        adj[u][v] = adj[v][u] = 1;
    }

    findBCCs();
    printf("%d\n", bccCount);

    return 0;
}
end
#shortet path
#include <stdio.h>
#include <limits.h>
#define MAX 100
#define INF INT_MAX
int n, m;
int graph[MAX][MAX]; // adjacency matrix
// Function to find vertex with minimum distance not yet visited
int minDistance(int dist[], int visited[]) {
    int min = INF, min_index = -1;
    for (int v = 0; v < n; v++) {
        if (!visited[v] && dist[v] <= min) {
            min = dist[v];
            min_index = v;
        }
    }
    return min_index;
}
void dijkstra(int src) {
    int dist[MAX];
    int visited[MAX];
    // Initialize distances
    for (int i = 0; i < n; i++) {
        dist[i] = INF;
        visited[i] = 0;
    }
    dist[src] = 0;
    // Process all vertices
    for (int count = 0; count < n - 1; count++) {
        int u = minDistance(dist, visited);
        if (u == -1) break; // no more reachable vertices
        visited[u] = 1;
        // Update neighbors
        for (int v = 0; v < n; v++) {
            if (!visited[v] && graph[u][v] != INF &&
                dist[u] != INF &&
                dist[u] + graph[u][v] < dist[v]) {
                dist[v] = dist[u] + graph[u][v];
            }
        }
    }
    // Print results
    for (int i = 0; i < n; i++) {
        if (dist[i] == INF)
            printf("%d -1\n", i); // unreachable
        else
            printf("%d %d\n", i, dist[i]);
    }
}
int main() {
    scanf("%d %d", &n, &m);
    // Initialize adjacency matrix
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            graph[i][j] = INF;
    // Input edges
    for (int i = 0; i < m; i++) {
        int u, v, w;
        scanf("%d %d %d", &u, &v, &w);
        graph[u][v] = w; // directed graph
    }
    int src;
    scanf("%d", &src);
    dijkstra(src);
    return 0;
}
end
job sequencing 
#include <stdio.h>
#include <stdlib.h>
// Structure for a Job
typedef struct {
    int id;
    int deadline;
    int profit;
} Job;

// Compare function for sorting jobs in descending order of profit
int compare(const void *a, const void *b) {
    Job *j1 = (Job *)a;
    Job *j2 = (Job *)b;
    return j2->profit - j1->profit;
}

int main() {
    int n;
    scanf("%d", &n);

    Job jobs[n];
    int i;

    for (i = 0; i < n; i++) {
        scanf("%d %d %d", &jobs[i].id, &jobs[i].deadline, &jobs[i].profit);
    }

    // Sort jobs by profit in descending order
    qsort(jobs, n, sizeof(Job), compare);

    // Find maximum deadline
    int maxDeadline = 0;
    for (i = 0; i < n; i++) {
        if (jobs[i].deadline > maxDeadline) {
            maxDeadline = jobs[i].deadline;
        }
    }

    // Create slots (all initially free)
    int slot[maxDeadline + 1];
    for (i = 0; i <= maxDeadline; i++) {
        slot[i] = -1;
    }

    int countJobs = 0, totalProfit = 0;

    // Schedule jobs
    for (i = 0; i < n; i++) {
        // Try to schedule in the latest free slot <= deadline
        for (int j = jobs[i].deadline; j > 0; j--) {
            if (slot[j] == -1) {
                slot[j] = jobs[i].id;
                countJobs++;
                totalProfit += jobs[i].profit;
                break;
            }
        }
    }

    printf("%d %d\n", countJobs, totalProfit);
    return 0;
}
enddd
#jknap sack
int knapsack(int weights[], int profits[], int n, int m) {
    int M[n + 1][m + 1];
    // Build table dp[][] in bottom up manner
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            if (i == 0 || j == 0)
                M[i][j] = 0;
            else if (weights[i-1] <= j)
                M[i][j]=(profits[i-1]+M[i-1][j-weights[i-1]]>M[i-1][j])?(profits[i-1]+M[i-1][j-weights[i-1]]):M[i-1][j];
            else
                M[i][j] = M[i - 1][j];
        }
    }
    return M[n][m];
}
int main() {
    int n;
    scanf("%d", &n);
    int weights[n], profits[n];
    // Input: weights
    for (int i = 0; i < n; i++) {
        scanf("%d", &weights[i]);
    }
    // Input: values
    for (int i = 0; i < n; i++) {
        scanf("%d", &profits[i]);
    }
    int capacity;
    scanf("%d", &capacity);
    // Solve and print result
    int maxProfit = knapsack(weights, profits, n, capacity);
    printf("%d\n", maxProfit);
    return 0;
} end 
#n queens
#include <stdio.h>
#include <stdbool.h>

#define MAX 20

int board[MAX][MAX], N;

void printBoard() {
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++)
            printf("%d ", board[i][j]);
        printf("\n");
    }
}

bool isSafe(int row, int col) {
    for (int i = 0; i < row; i++)
        if (board[i][col]) return false;

    for (int i = row, j = col; i >= 0 && j >= 0; i--, j--)
        if (board[i][j]) return false;

    for (int i = row, j = col; i >= 0 && j < N; i--, j++)
        if (board[i][j]) return false;

    return true;
}

bool solveNQueens(int row) {
    if (row >= N) return true;  // Base case

    for (int col = 0; col < N; col++) {  // try all columns
        if (isSafe(row, col)) {
            board[row][col] = 1;  // place queen

            if (solveNQueens(row + 1))  // Recursive call
                return true;

            board[row][col] = 0;  // backtrack
        }
    }

    return false; // no valid column
}

int main() {
    scanf("%d", &N);

    if (solveNQueens(0))
        printBoard();
    else
        printf("No solution exists for N = %d\n", N);

    return 0;
}
#back tracking

#include <stdio.h>

int n;
int values[100], weights[100];
int capacity;
int maxProfit = 0;

// Recursive backtracking function
void knapsack(int i, int currWeight, int currValue) {
    // Base case: all items considered
    if (i == n) {
        if (currValue > maxProfit)
            maxProfit = currValue;
        return;
    }

    // Case 1: Exclude current item
    knapsack(i + 1, currWeight, currValue);

    // Case 2: Include current item (if within capacity)
    if (currWeight + weights[i] <= capacity) {
        knapsack(i + 1,
                 currWeight + weights[i],
                 currValue + values[i]);
    }
}

int main() {
    scanf("%d", &n);

    for (int i = 0; i < n; i++) {
        scanf("%d %d", &values[i], &weights[i]);
    }

    scanf("%d", &capacity);

    // Start recursion from first item
    knapsack(0, 0, 0);

    printf("%d\n", maxProfit);

    return 0;
}
#travelling sales man
#include <stdio.h>
#include <limits.h>

#define MAX 20
#define INF 999999

int n;
int cost[MAX][MAX];
int visited[MAX];
int minCost = INT_MAX;

// Recursive function for TSP
void tsp(int city, int count, int currCost, int start) {
    // If all cities are visited and we can return to start
    if (count == n && cost[city][start] != INF) {
        int totalCost = currCost + cost[city][start];
        if (totalCost < minCost)
            minCost = totalCost;
        return;
    }

    // Try visiting unvisited cities
    for (int next = 0; next < n; next++) {
        if (!visited[next] && cost[city][next] != INF) {
            visited[next] = 1;
            tsp(next, count + 1, currCost + cost[city][next], start);
            visited[next] = 0; // backtrack
        }
    }
}

int main() {
    scanf("%d", &n);

    // Read cost matrix
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            int val;
            scanf("%d", &val);
            if (val == -1) {
                cost[i][j] = INF; // no direct connection
            } else {
                cost[i][j] = val;
            }
        }
    }

    // Initialize visited array
    for (int i = 0; i < n; i++)
        visited[i] = 0;

    // Start from city 0
    visited[0] = 1;
    tsp(0, 1, 0, 0);

    printf("%d\n", minCost);

    return 0;
}
bfs
#include <stdio.h>
#include <stdlib.h>
#define MAX 10
int adj[MAX][MAX];   // Adjacency matrix
int visited[MAX];    // Visited array
int queue[MAX];      // Queue for BFS
int front = 0, rear = 0;
int n;               // Number of vertices
// Enqueue function
void enqueue(int vertex) {
    queue[rear++] = vertex;
}
// Dequeue function
int dequeue() {
    return queue[front++];
}
// Check if queue is empty
int isEmpty() {
    return front == rear;
}
// Breadth First Search function
void bfs(int start) {
    visited[start] = 1;
    enqueue(start);
    while (!isEmpty()) {
        int current = dequeue();
        printf("%d ", current);
        // Traverse neighbors in increasing order
        for (int i = 0; i < n; i++) {
            if (adj[current][i] == 1 && !visited[i]) {
                visited[i] = 1;
                enqueue(i);
            }
        }
    }
}
int main() {
    int edges, u, v, start;
    // Input number of vertices
    scanf("%d", &n);
    // Input number of edges
    scanf("%d", &edges);
    // Initialize adjacency matrix and visited array
    for (int i = 0; i < n; i++) {
        visited[i] = 0;
        for (int j = 0; j < n; j++) {
            adj[i][j] = 0;
        }
    }
    // Read edges and update matrix
    for (int i = 0; i < edges; i++) {
        scanf("%d %d", &u, &v);
        adj[u][v] = 1;
        adj[v][u] = 1; // Undirected graph
    }
    // Input starting vertex
    scanf("%d", &start);
    // Perform BFS
    bfs(start);
    return 0;
}
Write a C program to perform BFS on a directed graph using adjacency lists. The graph has  vertices and  edges. After building the graph, sort each vertex’s adjacency list in ascending order. Then perform BFS from a given starting node and print all reachable nodes in the order they are visited as space-separated integers. When multiple adjacent nodes are available, always visit the smallest one first.
#include <stdio.h>
#include <stdlib.h>
#define MAX 100
int adj[MAX][MAX];     // Adjacency list
int adjCount[MAX];     // Number of neighbors for each node
int visited[MAX];      // Visited array
int queue[MAX];        // Queue for BFS
int front = 0, rear = 0;
// Comparison function for sorting neighbors
int compare(const void *a, const void *b) {
    return (*(int *)a - *(int *)b);
}
// Enqueue operation
void enqueue(int v) {
    queue[rear++] = v;
}
// Dequeue operation
int dequeue() {
    return queue[front++];
}
// Check if queue is empty
int isEmpty() {
    return front == rear;
}
// BFS function
void bfs(int start) {
    visited[start] = 1;
    enqueue(start);
    while (!isEmpty()) {
        int current = dequeue();
        printf("%d ", current);
        for (int i = 0; i < adjCount[current]; i++) {
            int neighbor = adj[current][i];
            if (!visited[neighbor]) {
                visited[neighbor] = 1;
                enqueue(neighbor);
            }
        }
    }
}
int main() {
    int vertices, edges;
    scanf("%d %d", &vertices, &edges);
    // Initialize structures
    for (int i = 0; i < vertices; i++) {
        adjCount[i] = 0;
        visited[i] = 0;
    }
    // Input edges
    for (int i = 0; i < edges; i++) {
        int u, v;
        scanf("%d %d", &u, &v); // directed edge u → v
        adj[u][adjCount[u]++] = v;
    }
    // Sort neighbors for each vertex
    for (int i = 0; i < vertices; i++) {
        qsort(adj[i], adjCount[i], sizeof(int), compare);
    }
    int start;
    scanf("%d", &start);
    // Perform BFS
    bfs(start);
    return 0;
}


