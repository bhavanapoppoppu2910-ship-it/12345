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
