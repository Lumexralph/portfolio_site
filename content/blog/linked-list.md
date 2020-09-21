---
title: "Data Structures: Singly-Linked List with Go"
date: 2020-08-04
image: "images/blog/singly-linked-list.png"
description: "review of a linked list using Go"
author: "Olumide Ogundele"
type: "post"
---

From learning about Operating Systems, diving into some source code of Linux, seeing the data structures for p_threads, mutex, sockets etc. I have grown to appreciate data structures and algorithms, they are combined to build powerful computer systems and tools. So, I want to share my learning as I resume my study on data structures and algorithms again after a while.

Majority of my learnings so far are attributed to this [book](https://www.amazon.com/Algorithm-Design-Manual-Steven-Skiena/dp/1849967202) and there are some excerpts from it too.

Data structures can be classified as contiguous or linked depending on if it is based on **arrays** or **pointers**. I think it is safe to see data structures as a way data or values are laid into the memory.

**Contiguously-allocated Structures** are composed of single slabs or blocks of memory, examples of data structures that are based on this are **arrays**, **heaps**, **hash tables** etc.

**Linked Data Structures** are composed of distinct chunks of memory bound together by pointers. They include **lists**, **trees**, **graphs** etc.

#### Pointers and Linked Structures

* Pointers are the connections that hold the pieces of linked structures together.

* Pointers represents the address of a location in memory. So a variable storing a pointer to a data item can provide more freedom than having a copy of the given data stored in the variable. This is what happens in Go when you pass value to a function by reference instead of passing a copy of the value.

* A cell phone number can be thought of as a pointer to the owner of the phone as they move about on planet earth.

* Much of the spaces used in linked-data structures has to be devoted to the pointers than the data

* Finally, we need a pointer to the head of the structure to know where to access it.

There is a small singly-linked list data structure created in Go and the basic operations supported are searching, insertions and deletions as shown below:

The complete code with the tests can be found on [github](https://github.com/Lumexralph/go-datastructures-algorithms/blob/master/dataStructures/linkedList/linked_list_b.go)

I will be creating a linked data structure that stores an item but in this case a name, the elements of the linked-list will be encapsulated in a node where we can hold the item and also the pointer to the next node that is linked to it, see the code below;

```go
    type nameNode struct {
        name string    // item
        next *nameNode // pointer to the next node it is linked to
    }
```

We need to have the main list that houses all the nodes and it will have just one attribute, `head`, this will help us to know the start of the list.

```go
    type nameList struct {
	head *nameNode // the starting point in the list
    }
```

#### Searching a List

It can be done iteratively or recursively, we take the item being searched for and iteratively or recursively move through the pointer till we get to the end which will be `nil` or empty in whichever way you structure your element or node. This is illustrated below:

```go
    func (l *nameList) searchList(name string) *nameNode {
        // starting from the head of the list
        // searchNode is a recursive function
        return searchNode(l.head, name)
    }
```

#### Insertion into a List

* It is easily done at the beginning of the list, no need to traverse the list.
* What is just needed is to have a variable to store the head of the pointer.
* When a new insertion is needed, swap the head for the new insertion and point the next node of the new insertion to the previous head.
* Update the head of the list to the new node.

This is illustrated below:

```go
    func (l *nameList) insertList(name string) {
        newName := &nameNode{name, nil}
        if l.head == nil {
            l.head = newName
            return
        }
        previousHeader := l.head
        // link with the previous head and insert at the head
        newName.next = previousHeader
        l.head = newName
    }
```

#### Deletion from a List

It is a little tricky if it is a singly-linked-list compared to doubly-linked-list.

* Search for the node or element.

* Create another operation to return the pointer to the predecessor node before the item to be deleted.
* Link the predecessor node with the node after the found node i.e make the predecessor pointer, point to the node after the found node which was gotten through search. That way, we detach the node we want to delete from the linked-list and during house-keeping by the system or garbage collection, it will be removed from the memory and whatever memory address it was allocated will be freed.

This is shown below:

```go
    func (l *nameList) deleteList(name string) {
        // search if the node exists
        foundNode := l.searchList(name)

        if foundNode != nil {
            // get the nodes before name
            nodeBefore := nodesBeforeItem(l.head, name)
            if nodeBefore != nil {
                nodeBefore.next = foundNode.next
            } else { // it means the foundNode is the head node
                l.head = foundNode.next
            }
        }
    }
```

#### Final Thoughts

* Insertions and deletions are easier in a linked list compared to array lists.
When you have large records or data, moving pointers is easier and faster than moving the items themselves which might be huge, a good point to this is when you pass values to function in Go by reference and not by copying the value.

* Linked structures require extra space for storing pointers, which means that apart from the data we want, pointers take up memory space too, so it might increase the data size.
* The convenience of random access to items we get from array lists is not in linked-list, we have to do a traversal.

* Also due to the way the memory is laid out in the computer, some optimizations have also being implemented by the operating system which allows fast data access and reduce CPU cycles, memory cache was created for better performance, array allows better memory locality which the memory cache takes advantage of because the block of memory used are allocated in a continuous way, compared to pointer structure that is composed of small memory chunks linked together which will result in jumping around in the memory and also lead to losing the memory locality. In other words, if you’ll be doing a lot of searching, compared to insertion and deletion, maybe you should think of the benefit of an array list.

A final thing to think about is that, Array and Lists are recursive objects, you can always remove the first element of a list and you have smaller elements, likewise you can remove some constant number of elements from an array till you get to an empty array, which is a good base index when doing stuff the recursive way.

 I will be glad to hear from you if you have question(s) or feedback, have fun!
