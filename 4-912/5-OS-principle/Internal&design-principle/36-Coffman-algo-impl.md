## 代码框架  
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int n; // Number of processes
int m; // Number of resource types

std::vector<bool> marked; // To keep track of marked processes
std::vector<bool> finished; // To keep track of finished processes

std::vector<std::vector<int>> allocation; // Allocation matrix
std::vector<int> available; // Available vector
std::vector<std::vector<int>> request; // Request matrix

void deadlockDetection() {
    // Initialize marked and finished vectors
    marked.resize(n, false);
    finished.resize(n, false);

    // Step 1: Mark processes with no allocation
    for (int i = 0; i < n; ++i) {
        bool hasAllocation = std::any_of(allocation[i].begin(), allocation[i].end(), [](int val) { return val > 0; });
        if (!hasAllocation) {
            marked[i] = true;
        }
    }

    while (true) {
        // Step 2: Initialize temporary vector W
        std::vector<int> W(available);

        bool found = false;
        int processIndex = -1;

        // Step 3: Find an unmarked process that can be satisfied
        for (int i = 0; i < n; ++i) {
            if (!marked[i] && !finished[i]) {
                bool canBeSatisfied = true;
                for (int j = 0; j < m; ++j) {
                    if (request[i][j] > W[j]) {
                        canBeSatisfied = false;
                        break;
                    }
                }
                if (canBeSatisfied) {
                    found = true;
                    processIndex = i;
                    break;
                }
            }
        }

        if (!found) {
            // No unmarked process can be satisfied, deadlock check ends
            break;
        }

        // Step 4: Mark the process and update W
        marked[processIndex] = true;
        for (int j = 0; j < m; ++j) {
            W[j] += allocation[processIndex][j];
        }
    }

    // Print processes in deadlock
    for (int i = 0; i < n; ++i) {
        if (marked[i] && !finished[i]) {
            std::cout << "Process " << i << " is in deadlock." << std::endl;
        }
    }
}

int main() {
    // Initialize allocation, available, request, n, and m
    // ...

    deadlockDetection();

    return 0;
}

```

## 实例化实现
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int n = 5; // Number of processes
int m = 4; // Number of resource types

std::vector<bool> marked; // To keep track of marked processes
std::vector<bool> finished; // To keep track of finished processes

std::vector<std::vector<int>> allocation = {
    {0, 1, 0, 0},
    {2, 0, 0, 1},
    {0, 0, 0, 0},
    {1, 0, 3, 0},
    {0, 0, 0, 2}
}; // Allocation matrix

std::vector<int> available = {1, 5, 2, 0}; // Available vector

std::vector<std::vector<int>> request = {
    {0, 0, 0, 0},
    {2, 0, 2, 0},
    {0, 0, 0, 0},
    {1, 0, 0, 0},
    {0, 0, 2, 0}
}; // Request matrix

void printMatrix(const std::vector<std::vector<int>>& matrix, const std::string& name) {
    std::cout << name << " matrix:" << std::endl;
    for (int i = 0; i < matrix.size(); ++i) {
        for (int j = 0; j < matrix[i].size(); ++j) {
            std::cout << matrix[i][j] << " ";
        }
        std::cout << std::endl;
    }
    std::cout << std::endl;
}

void deadlockDetection() {
    // Initialize marked and finished vectors
    marked.resize(n, false);
    finished.resize(n, false);

    // Step 1: Mark processes with no allocation
    for (int i = 0; i < n; ++i) {
        bool hasAllocation = std::any_of(allocation[i].begin(), allocation[i].end(), [](int val) { return val > 0; });
        if (!hasAllocation) {
            marked[i] = true;
        }
    }

    int round = 0;
    while (true) {
        ++round;
        std::cout << "Round " << round << ":" << std::endl;

        // Print the current state
        printMatrix(allocation, "Allocation");
        printMatrix(request, "Request");
        printMatrix(std::vector<std::vector<int>>{available}, "Available");

        // Step 2: Initialize temporary vector W
        std::vector<int> W(available);

        bool found = false;
        int processIndex = -1;

        // Step 3: Find an unmarked process that can be satisfied
        for (int i = 0; i < n; ++i) {
            if (!marked[i] && !finished[i]) {
                bool canBeSatisfied = true;
                for (int j = 0; j < m; ++j) {
                    if (request[i][j] > W[j]) {
                        canBeSatisfied = false;
                        break;
                    }
                }
                if (canBeSatisfied) {
                    found = true;
                    processIndex = i;
                    break;
                }
            }
        }

        if (!found) {
            // No unmarked process can be satisfied, deadlock check ends
            break;
        }

        // Step 4: Mark the process and update W
        marked[processIndex] = true;
        for (int j = 0; j < m; ++j) {
            W[j] += allocation[processIndex][j];
        }

        // Print the marked process and updated W
        std::cout << "Marked process: " << processIndex << std::endl;
        std::cout << "Updated Available after marking process " << processIndex << ":" << std::endl;
        for (int i = 0; i < m; ++i) {
            std::cout << W[i] << " ";
        }
        std::cout << std::endl << std::endl;
    }

    // Print processes in deadlock
    for (int i = 0; i < n; ++i) {
        if (marked[i] && !finished[i]) {
            std::cout << "Process " << i << " is in deadlock." << std::endl;
        }
    }
}

int main() {
    deadlockDetection();

    return 0;
}

```