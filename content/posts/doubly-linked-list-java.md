---
author: "Naadiyaar"
title: "Find the way back home"
date: 2024-01-30
draft: false
ShowToc: true
---
## Intro
In times of using *arrays*, where inserting or deleting elements in the middle often requires shifting subsequent elements, when lists grow in size this tasks exponentially become expensive. And traversing over these huge lists make CPUs scream, [top](https://www.man7.org/linux/man-pages/man1/top.1.html) proves.

But no worries, it's also possible to traverse from end of the list to the very first node, thus accessing nodes placed in end parts of the list becomes much cheaper. For this we should remember the path to the previous node and all the way back home; In more technical terms we need to create a **doubly linked list** and we'll see how, implemented in **Java**.

## Linked list
In general a linked list is dynamic a arrangement which all of its elements/nodes has a reference to the next element.
This way every element is accessible through the previous element which has a pointer attached to next one and this way each element can be stored at any block of heap memory and allocating a specific block of storage isn't necessary anymore.
### Being upon such structure, has some benefits:
- **Dynamic data structure:** there is no need to give a linked list an initialized size because it can shrink and grow during run time freely. 
- **No memory wastage:** since there's no need to pre-allocate memory all the memory consumed is used.
- **Easier to manipulate:** in addition and deletion rest of elements doesn't shift and stay unchanged therefor it's easier work with a linked list.

But there are some limitations to a linked list, like traversing from head to the tail of the list can be troublesome especially when dealing with huge chunks of data and this pushes us to *doubly linked lists*.

## Doubly linked lists
Linked lists and doubly linked lists have more similarities than differences. As the name suggests each element in *doubly* once have two links instead of one. One link is pointing to next element same as single linked lists and one is pointing to the previous element and this enable us to traverse backward and saves so much time and energy when working on huge lists.


![Diagram of a doubly linked list](/doubly-link-list.png)


Though there are downsides to related to having an extra link, e.g. More memory and storage consumption, complexity and insertion/deletion overhead.
In these article we'll explore how to create a DLL and implement some necessary methods in Java:
- **addHead():** inserting an element to the beginning of the list.
- **addTail():** inserting an element to the end of the list.
- **rmHead():** removing first element of the list.
- **rmTail():** removing last element of the list.
- **addIndex():** inserting an element to a specific index.
- **rmIndex():** removing an element from the list by its index.
- **remove():** remove an element from list by its data.

## Constructors
Each element in a DLL is made of three components, a variable to hold the actual data, a variable to hold a reference to the next element and a variable pointing at the previous element.
To create doubly linked list in Java, we need to define the *Element* class which implements these three variables.
```java
package DoublyLinkedList;

public class Element<T> {

    // actual data
    final T data;
    // reference to the previous node
    Element<T> prev;
    // reference to the next node
    Element<T> next;

    // Constructor
    Element(T data, Element<T> prev, Element<T> next) {
        this.data = data;
        this.prev = prev;
        this.next = next;
    }
}
```
Now we can use this class to easily generate DLL elements of any type.
Two *head* and *tail* variables are references to the first and last element of the list respectively, to initialize them with also required method we create the *DoublyLinkedList* class; with a constructor to set *tail* and *head* to null until we add some data to the list. 
```java
package DoublyLinkedList;

public class DoublyLinkedList<T> {

    // First element
    private Element<T> head;
    // Last element
    private Element<T> tail;

    public DoublyLinkedList() {
        this.head = null;
        this.tail = null;
    }
}
```

## Implementing methods
Now it's time to add methods to the `DoublyLinkedList` class; this methods will be non-static and related to each instance of the class.
### addHead()
This method will simply get *data* as parameter and adds it to the beginning of the list, so it becomes head, or first element of list.
```java
public void addHead(T data){

    // Create a temporary instance of data to keep a reference
    Element<T> tmp = new Element<>(data, null, null);
    if (head == null)
        // Generate first and last element 
        head = tail = tmp;
    else {
        // updating pointers 
        tmp.next = head;
        head.prev = tmp;
        head = tmp;
    }
}
```
This method checks if the list is empty it will create the first element of the list and updates 'head' and 'tail' references; if not, it will update references for `head` and put the previous head as second element.
### addTail()
This method will simply get *data* as parameter and adds it to the end of the list, so it becomes tail, or last element of list.
```java
public void addTail(T data){

    // Create a temporary instance of data to keep a reference
    Element<T> tmp = new Element<>(data, null, null);
    if (tail == null)
        // Generate first and last element 
        head = tail = tmp;
    else {
        // updating pointers 
        tmp.prev = tail;
        tail.next = tmp;
        tail = tmp;
    }
}
```
### rmHead()
This method will first check if the method isn't empty then checks if list has only one element, if true it will remove it and set *head* and *tail* to null and if it had more than one element removes all pointers to first element (head) and set the second element as new *head*.
```java
public void rmHead() {

    // check if list is null
    if (head == null) return;

    // check if list has only one element
    if (head == tail) {
        head = tail = null;
        return;
    }

    // update head
    head = head.next;
    // remove pointer to previous element
    head.prev = null;

}
```
### rmTail()
This method follows steps similar to the previous method but instead removes the last element of the list and updates *tail*.
```java
public void rmTail() {

    // check if list is null
    if (tail == null) return;

    // check if list has only one element
    if (head == tail) {
        head = tail = null;
        return;
    }

    // update tail
    tail = tail.prev;
    // remove pointer to next element
    tail.next = null;

}
```
### addIndex()

![Diagram of a adding an element to a doubly linked list](/doubly-link-list-new-element.png)

Using this method you can insert a new element to any position in the list (starting from 1). In this case we remove pointers among two elements and point them to new element; this way new node will be inserted between them.
```java
public void addIndex(T data, int index) throws IllegalArgumentException {

    // counting starts from 1
    if (index == 0) throw new IllegalArgumentException("Counting elements starts from 1");

    // check if data should be added to the beginning of the list
    if (index == 1) addHead(data);
    else {
        // create a trackers to traverse the list
        Element<T> current = head;
        int currPosition = 1;

        // find the position that data must be inserted in
        while (currPosition < index && current != null) {
            current = current.next;
            currPosition++;
        }

        // if position is out of scope
        if (current == null) addTail(data);
        else { // updating pointers to insert data into the list
            Element<T> tmp = new Element<>(data, null, null);
            tmp.prev = current.prev;
            tmp.next = current;
            current.prev.next = tmp;
            current.prev = tmp;
        }
    }
}
```
What is happening in the last **else** block, is it creates a new element from *data* calls is it *tmp*, then update pointers around it to finally insert it completely in the right position.
First it creates pointers to previous and next elements for *tmp*. Then it updates pointers for surrounding elements and point them to *tmp* object.
### rmIndex()
This is much similar to `addIndex()`; So we only nuke pointers to an element and attach them to its previous/next element. Also this method will throw an exception if user tries to remove an index bigger than size of the list.
```java
public void rmIndex(int index) throws IllegalArgumentException {

    // counting starts from 1
    if (index == 0) throw new IllegalArgumentException("Counting elements starts from 1");

    // check if list has no element
    if (head == null) return;

    // check if head must be removed
    if (index == 1) {
        rmHead();
        return;
    }

    // trackers for traversing the list
    Element<T> current = head;
    int currPosition = 1;

    // find position of data must be deleted
    while (currPosition < index && current != null) {
        current = current.next;
        currPosition++;
    }

    if (current == null) throw new IllegalArgumentException("Index doesn't exist in the range of list");

    if (current == tail) {
        rmTail();
        return;
    }

    current.prev.next = current.next;
    current.next.prev = current.prev;
    current.prev = current.next = null;
}
```
### remove()
This is probably the most practical method among all of them, it gets data as an argument and search the list to remove the first element containing the *data*.
```java
public void remove(T data) throws NoSuchElementException {

    // check if list contain anything
    if (head == null) throw new NoSuchElementException("List is empty");

    // removing first element
    if (head.data.equals(data)) {
        head = head.next;
        return;
    }

    Element<T> current = head;

    // looping to find data
    while (!current.data.equals(data) && current != null) current = current.next;

    // if data doesn't exist in the list
    if (current == null) throw new NoSuchElementException("element " data.toString() " not found");

    // check if it's not last element
    if (current.next != null) current.next.prev = current.prev;

    current.prev.next = current.next;
}
```
## Outro
So far we implemented useful method for generating and manipulating doubly-linked-lists. Despite their issues about memory consumption and relative complexity there are many use-cases to them and can be used to traverse back in list and find the way back home safely.


This article made possible by getting insights from other resources, thanks to:
- [Linked Lists](https://www.youtube.com/watch?v=_jQhALI4ujg) video by Computerphile.
- [Introduction to Doubly Linked Lists in Java](https://www.geeksforgeeks.org/introduction-to-doubly-linked-lists-in-java/) from GeeksforGeeks.
- [Doubly Linked List in Java](https://dzone.com/articles/doubly-linked-list-in-java) from Dzone.


**Thanks for reading; please let me know what do you think; good luck.**
