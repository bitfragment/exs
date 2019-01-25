---
title: "Doubly linked list"
date: 2018-11-15
tags: c
published: true
---

Task is to implement two functions for use in manipulating
a doubly linked list.

* TOC
{:toc}


listNode
--------

```c
typedef struct listNode {
  int value;
  struct listNode *next;
  struct listNode *prev;
} listNode;
```


doublyLinkedList
----------------

```c
typedef struct doublyLinkedList {
  listNode *head;
  listNode *tail;
} doublyLinkedList;
```


insertHead()
------------
Given a pointer to a doubly linked list and a value, insert
a new node, with that value, at the head of the list.

```c
void insertHead(doublyLinkedList * listPtr, int value) {

  // Create the new head node.
  listNode * newNodePtr = (listNode *) malloc(sizeof(listNode));

  if (newNodePtr != NULL) { // If we successfully allocated memory...

    // Deal with the new node first.
    //
    // First, add the value.
    newNodePtr->value = value;

    // Within the head node of a doubly linked list, the `prev` pointer
    // is NULL and the `next` pointer is the head of the list.
    newNodePtr->next = listPtr->head;
    newNodePtr->prev = NULL;

    // Finally, if this list already contained at least one node, then
    // the new second node in the list must point back to the new 
    // first node in the list.
    if (newNodePtr->next != NULL) newNodePtr->next->prev = newNodePtr;

    // Now update the list itself.
    //
    // If a node is inserted into an empty list, that list's head pointer
    // and its tail pointer both point to the same node.
    if (listPtr->head == NULL) listPtr->tail = newNodePtr;

    // Update the list's head pointer.
    listPtr->head = newNodePtr;
  }

  return;
}
```


removeTail()
------------
Given a pointer to a doubly linked list, remove the tail node
of the list and return its value.

```c
int removeTail(doublyLinkedList * listPtr) {
  int val;

  // Get the first node in the list.
  listNode * curr = listPtr->head;

  // Handle case in which the list contains only one node.
  if (curr->next == NULL) {
    val = curr->value;
    free(curr);
    listPtr->head = NULL;
    listPtr->tail = NULL;
    return val;
  }

  // Traverse the list, preserving the previous node.
  listNode * prev;
  while (curr->next != NULL) {

    // Debugging code.
    // printf("Traversing list... prev: %d, curr: %d\n", 
    //                            prev->value, curr->value);

    prev = curr;
    curr = curr->next;
  }

  // Get the value of the tail node, then free its memory.
  val = curr->value;
  free(curr);

  // Update the previous node's next pointer.
  prev->next = NULL;

  // Update the list's tail pointer.
  listPtr->tail = prev;

  return val;
} 
```
