addition of cuda
#include <iostream>
#include <vector>
#include <omp.h>

using namespace std;

int main() {
    const int size = 1000000;
    vector<int> vec1(size, 1);
    vector<int> vec2(size, 2);
    vector<int> result(size);
    
    #pragma omp parallel for
    for(int i = 0; i < size; ++i) {
        result[i] = vec1[i] + vec2[i];
    }
    
    cout << "RESULT VECTOR:" << endl;
    for(int i = 0; i < 10; ++i) {
        cout << result[i] << " ";
    }
    cout << endl;
    
    return 0;
}


matrix multi 

#include <iostream>
#include <vector>
#include <omp.h>

using namespace std;

int main() {
    const int rows = 1000;
    const int cols = 1000;
    vector<vector<int>> matrix1(rows, vector<int>(cols, 1));
    vector<vector<int>> matrix2(rows, vector<int>(cols, 2));
    vector<vector<int>> result(rows, vector<int>(cols, 0));

    #pragma omp parallel for collapse(2)
    for(int i = 0; i < rows; ++i) {
        for(int j = 0; j < cols; ++j) {
            for(int k = 0; k < cols; ++k) {
                result[i][j] += matrix1[i][k] * matrix2[k][j];
            }
        }
    }

    cout << "Result matrix: " << endl;
    for(int i = 0; i < 3; ++i) {
        for(int j = 0; j < 3; ++j) {
            cout << result[i][j] << " ";
        }
        cout << endl;
    }

    return 0;
}

min max

#include <iostream>
#include <omp.h>

using namespace std;

int main() {
    const int size = 10;
    int arr[size] = {43, 60, 78, 62, 76, 56, 64, 9, 2, 8};
    int min_val = arr[0];
    int max_val = arr[0];
    int sum = 0;
    
    #pragma omp parallel for reduction(min:min_val) reduction(max:max_val) reduction(+:sum)
    for(int i = 0; i < size; ++i) {
        if(arr[i] < min_val) {
            min_val = arr[i];
        }
        if(arr[i] > max_val) {
            max_val = arr[i];
        }
        sum += arr[i];
    }
    
    double average = static_cast<double>(sum) / size;
    
    cout << "Minimum Value: " << min_val << endl;
    cout << "Maximum Value: " << max_val << endl;
    cout << "Sum: " << sum << endl;
    cout << "Average: " << average << endl;
    
    return 0;
}

merge
#include <iostream>
#include <omp.h>

using namespace std;

void mergesort(int a[], int i, int j);
void merge(int a[], int i1, int j1, int i2, int j2);

void mergesort(int a[], int i, int j) {
    int mid;
    if (i < j) {
        mid = (i + j) / 2;

        #pragma omp parallel sections
        {
            #pragma omp section
            {
                mergesort(a, i, mid);
            }

            #pragma omp section
            {
                mergesort(a, mid + 1, j);
            }
        }

        merge(a, i, mid, mid + 1, j);
    }
}

void merge(int a[], int i1, int j1, int i2, int j2) {
    int temp[1000];
    int i, j, k;
    i = i1;
    j = i2;
    k = 0;

    while (i <= j1 && j <= j2) {
        if (a[i] < a[j]) {
            temp[k++] = a[i++];
        } else {
            temp[k++] = a[j++];
        }
    }

    while (i <= j1) {
        temp[k++] = a[i++];
    }

    while (j <= j2) {
        temp[k++] = a[j++];
    }

    for (i = i1, j = 0; i <= j2; i++, j++) {
        a[i] = temp[j];
    }
}

int main() {
    int *a, n, i;
    cout << "\n enter total no of elements=>";
    cin >> n;
    a = new int[n];

    cout << "\n enter elements=>";
    for (i = 0; i < n; i++) {
        cin >> a[i];
    }

    double start_time = omp_get_wtime(); // Start measuring time
    mergesort(a, 0, n - 1);
    double end_time = omp_get_wtime(); // Stop measuring time

    cout << "\n sorted array is=>";
    for (i = 0; i < n; i++) {
        cout << "\n" << a[i];
    }

    cout << "\nTime taken: " << end_time - start_time << " seconds" << endl;

    delete[] a; // Free dynamically allocated memory
    return 0;
}

bubble

#include<iostream>
#include<omp.h>

using namespace std;

void bubble(int *, int);
void swap(int &, int &);

void bubble(int *a, int n) {
    for (int i = 0; i < n; i++) {
        int first = i % 2;

        #pragma omp parallel for shared(a, first)
        for (int j = first; j < n - 1; j += 2) {
            if (a[j] > a[j + 1]) {
                swap(a[j], a[j + 1]);
            }
        }
    }
}

void swap(int &a, int &b) {
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int *a, n;
    cout << "\n enter total no of elements=>";
    cin >> n;
    a = new int[n];
    cout << "\n enter elements=>";
    for (int i = 0; i < n; i++) {
        cin >> a[i];
    }

    double start_time = omp_get_wtime(); // Start measuring time
    bubble(a, n);
    double end_time = omp_get_wtime(); // Stop measuring time

    cout << "\n sorted array is=>";
    for (int i = 0; i < n; i++) {
        cout << a[i] << endl;
    }

    cout << "\nTime taken: " << end_time - start_time << " seconds" << endl;

    delete[] a; // Free dynamically allocated memory
    return 0;
}


dfs

#include <iostream>
#include <vector>
#include <stack>
#include <omp.h>

using namespace std;

const int MAX = 100000;
vector<int> graph[MAX];
bool visited[MAX];

void dfs(int node) {
    stack<int> s;
    s.push(node);

    while (!s.empty()) {
        int curr_node = s.top();
        s.pop();

        if (!visited[curr_node]) {
            visited[curr_node] = true;

            cout << curr_node << " ";

            #pragma omp parallel for
            for (int i = 0; i < graph[curr_node].size(); i++) {
                int adj_node = graph[curr_node][i];
                if (!visited[adj_node]) {
                    s.push(adj_node);
                }
            }
        }
    }
}

int main() {
    int n, m, start_node;
    cout << "Enter No of Node, Edges, and start node: ";
    cin >> n >> m >> start_node;
    // n: node, m: edges

    cout << "Enter Pair of edges: ";
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        // u and v: Pair of edges
        graph[u].push_back(v);
        graph[v].push_back(u);
    }

    #pragma omp parallel for
    for (int i = 0; i < n; i++) {
        visited[i] = false;
    }

    dfs(start_node);

    return 0;
}
bfs

#include<iostream>
#include<stdlib.h>
#include<queue>
#include<omp.h>

using namespace std;

class node {
public:
    node *left, *right;
    int data;
};

class Breadthfs {
public:
    node *insert(node *, int);
    void bfs(node *);
};

node *Breadthfs::insert(node *root, int data) {
    if (!root) {
        root = new node;
        root->left = NULL;
        root->right = NULL;
        root->data = data;
        return root;
    }

    queue<node *> q;
    q.push(root);

    while (!q.empty()) {
        node *temp = q.front();
        q.pop();

        if (temp->left == NULL) {
            temp->left = new node;
            temp->left->left = NULL;
            temp->left->right = NULL;
            temp->left->data = data;
            return root;
        } else {
            q.push(temp->left);
        }

        if (temp->right == NULL) {
            temp->right = new node;
            temp->right->left = NULL;
            temp->right->right = NULL;
            temp->right->data = data;
            return root;
        } else {
            q.push(temp->right);
        }
    }
    return root;
}

void Breadthfs::bfs(node *head) {
    queue<node*> q;
    q.push(head);
    int qSize;
    
    while (!q.empty()) {
        qSize = q.size();
        
        #pragma omp parallel for
        for (int i = 0; i < qSize; i++) {
            node* currNode;
            #pragma omp critical
            {
                currNode = q.front();
                q.pop();
                cout << "\t" << currNode->data;
            }
            #pragma omp critical
            {
                if (currNode->left)
                    q.push(currNode->left);
                if (currNode->right)
                    q.push(currNode->right);
            }
        }
    }
}

int main() {
    node *root = NULL;
    int data;
    char ans;

    do {
        cout << "\n enter data=>";
        cin >> data;
        Breadthfs obj;
        root = obj.insert(root, data);
        cout << "do you want insert one more node?";
        cin >> ans;
    } while (ans == 'y' || ans == 'Y');

    Breadthfs obj;
    obj.bfs(root);

    return 0;
}
