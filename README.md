# [HW1.P3] Skip List Tutorial
<!-- 文字敘述如何實作 -->
Keywords: `skip list`.

## sample code
<!-- 註解函數、幾個重要變數在幹嘛 -->
```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

typedef struct SkipNode {
    long long key;
    struct SkipNode **next;
} SkipNode;

typedef struct SkipList {
    int max_level;
    int level;
    SkipNode *header;
} SkipList;

SkipNode* createNode(long long key, int level) {
    SkipNode *new_node = (SkipNode*)malloc(sizeof(SkipNode));
    new_node->key = key;
    new_node->next = (SkipNode**)malloc(sizeof(SkipNode*) * (level + 1));
    return new_node;
}

SkipList* createSkipList(int max_level) {
    SkipList *list = (SkipList*)malloc(sizeof(SkipList));
    list->max_level = max_level;
    list->level = 0;
    list->header = createNode(LLONG_MAX, max_level);
    for (int i = 0; i <= max_level; i++) {
        list->header->next[i] = NULL;
    }
    return list;
}

int GetLevel(long long key) { // calculate the layer of the node
    int level = 0;
    while (key % 2 == 1) {
        level++;
        key = key / 2;
    }
    return level;
}

void insertNode(SkipList *list, long long key) {
    SkipNode *update[list->max_level + 1]; // store the previous node in each layer
    SkipNode *current = list->header;
    for (int i = list->level; i >= 0; i--) {
        while (current->next[i] != NULL && key < current->next[i]->key) {
            current = current->next[i];
        }
        update[i] = current;
    }
    current = current->next[0];
    if (current == NULL || current->key != key) {
        int new_level = GetLevel(key);
        if (new_level > list->level) {
            for (int i = list->level + 1; i <= new_level; i++) {
                update[i] = list->header;
            }
            list->level = new_level;
        }
        SkipNode *new_node = createNode(key, new_level);
        for (int i = 0; i <= new_level; i++) {
            new_node->next[i] = update[i]->next[i];
            update[i]->next[i] = new_node;
        }
    }
}

void deleteNode(SkipList *list, long long key) {
    SkipNode *update[list->max_level + 1];
    SkipNode *current = list->header;
    for (int i = list->level; i >= 0; i--) {
        while (current->next[i] != NULL && key < current->next[i]->key) {
            current = current->next[i];
        }
        update[i] = current;
    }
    current = current->next[0];
    if (current != NULL && current->key == key) {
        for (int i = 0; i <= list->level; i++) {
            if (update[i]->next[i] != current) {
                break;
            }
            update[i]->next[i] = current->next[i];
        }
        free(current->next);
        free(current);
        while (list->level > 0 && list->header->next[list->level] == NULL) {
            list->level--;
        }
    }
}

void slow_get(SkipList *list, long long k){ // slow get the first element >= k
    SkipNode *current = list->header->next[0];
    int found = 0;
    while (current != NULL && current->key >= k) {
        printf("%lld ", current->key);
        found = 1;
        current = current->next[0];
    }
    if(found == 0) printf("-1");
    printf("\n");
}

void fast_get(SkipList *list, long long k){ // fast get the first element >= k
    SkipNode *current = list->header;
    int found = 0;
    for (int i = list->level; i >= 0; i--){
        if(current->key != LLONG_MAX) printf("%lld ", current->key);
        while (current->next[i] != NULL && k <= current->next[i]->key) {
            current = current->next[i];
            found = 1;
            printf("%lld ", current->key);
        }
    }
    if(found == 0) printf("-1");
    printf("\n");
}

void freeSkipList(SkipList *list) { // not necessary
    SkipNode *current = list->header->next[0];
    while (current != NULL) {
        SkipNode *temp = current;
        current = current->next[0];
        free(temp->next);
        free(temp);
    }
    free(list->header->next);
    free(list->header);
    free(list);
}

int main() {
    SkipList *list = createSkipList(64); // the max layer is < 64
    int M, t;
    long long k;
    scanf("%d", &M);
    for(int i = 0; i < M; i++){
        scanf("%d%lld", &t, &k);
        if(t == 1) slow_get(list, k);
        else if(t == 2) fast_get(list, k);
        else if(t == 3) insertNode(list, k);
        else deleteNode(list, k);
    }
    freeSkipList(list);
    return 0;
}
```

## common mistakes
<!-- 寫幾個常見錯誤 -->

1. Since the range of $k$ is $[0, 10^{18}]$, it needs to be declared using `long long`. Additionally, if the value of `head` is $\infty$, it should also be changed to `LLONG_MAX` instead of the original `INT_MAX`.
2. When implementing `INSERT`, you should start searching $k$ from the top layer of the skip list, instead of the layer that $k$ appear first. 
3. If you opt to set the head value of each layer as the maximum number within that layer, you'll need to update the head when the maximum number of that layer is deleted.
4. If you are deleting the node recursively from top to bottom, you need to pay attention to the termination condition. 

## coding tips
<!-- 一些簡化程式複雜程度的技巧 -->
1. When implementing `CoinFlip`, it's better to directly calculate the required level for a certain node. If recalculating $2^{i-1}$ for each level ascent, it's more likely to exceed the time limit. (Using bit operations can also prevent timeouts).
2. The value of the head node for each layer can be the maximum value of that layer or positive infinity. It's generally easier to implement the latter.