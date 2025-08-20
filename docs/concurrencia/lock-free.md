---
title: "Algoritmos Lock-Free"
description: "Estructuras de datos y algoritmos sin bloqueos para alta concurrencia"
nav_order: 1
---

# Algoritmos Lock-Free

## 🎯 Resumen Ejecutivo

Los **algoritmos lock-free** permiten que múltiples threads accedan a estructuras de datos compartidas sin usar locks tradicionales, eliminando contención y mejorando el rendimiento en sistemas altamente concurrentes.

### Conceptos Clave
- **Compare-And-Swap (CAS)**: Operación atómica fundamental
- **ABA Problem**: Challenge clásico de lock-free programming
- **Memory Ordering**: Consistencia en arquitecturas modernas
- **Hazard Pointers**: Gestión segura de memoria

---

## 🔄 Compare-And-Swap (CAS)

### Operación Fundamental

`java
public class AtomicReference<V> {
    /**
     * Atomically sets the value to newValue if current value == expectedValue
     */
    public final boolean compareAndSet(V expectedValue, V newValue) {
        // Implementado a nivel hardware
        return unsafe.compareAndSwapObject(this, valueOffset, expectedValue, newValue);
    }
}
`

### 🚀 Stack Lock-Free

`java
public class LockFreeStack<T> {
    private final AtomicReference<Node<T>> top = new AtomicReference<>();
    
    private static class Node<T> {
        final T item;
        final Node<T> next;
        
        Node(T item, Node<T> next) {
            this.item = item;
            this.next = next;
        }
    }
    
    public void push(T item) {
        Node<T> newNode = new Node<>(item, null);
        Node<T> currentTop;
        
        do {
            currentTop = top.get();
            newNode.next = currentTop;
        } while (!top.compareAndSet(currentTop, newNode));
    }
    
    public T pop() {
        Node<T> currentTop;
        Node<T> newTop;
        
        do {
            currentTop = top.get();
            if (currentTop == null) {
                return null;
            }
            newTop = currentTop.next;
        } while (!top.compareAndSet(currentTop, newTop));
        
        return currentTop.item;
    }
}
`

---

## 🔄 Queue Lock-Free (Michael & Scott)

`java
public class LockFreeQueue<T> {
    private final AtomicReference<Node<T>> head;
    private final AtomicReference<Node<T>> tail;
    
    private static class Node<T> {
        volatile T item;
        volatile Node<T> next;
        
        Node(T item) {
            this.item = item;
        }
    }
    
    public LockFreeQueue() {
        Node<T> dummy = new Node<>(null);
        head = new AtomicReference<>(dummy);
        tail = new AtomicReference<>(dummy);
    }
    
    public void enqueue(T item) {
        Node<T> newNode = new Node<>(item);
        
        while (true) {
            Node<T> last = tail.get();
            Node<T> next = last.next;
            
            if (last == tail.get()) { // Verificar consistencia
                if (next == null) {
                    // Intentar agregar el nuevo nodo
                    if (casNext(last, null, newNode)) {
                        // Avanzar tail
                        tail.compareAndSet(last, newNode);
                        break;
                    }
                } else {
                    // Ayudar a avanzar tail
                    tail.compareAndSet(last, next);
                }
            }
        }
    }
    
    public T dequeue() {
        while (true) {
            Node<T> first = head.get();
            Node<T> last = tail.get();
            Node<T> next = first.next;
            
            if (first == head.get()) { // Verificar consistencia
                if (first == last) {
                    if (next == null) {
                        return null; // Queue vacío
                    }
                    // Ayudar a avanzar tail
                    tail.compareAndSet(last, next);
                } else {
                    if (next == null) {
                        continue; // Estado inconsistente
                    }
                    
                    T item = next.item;
                    if (head.compareAndSet(first, next)) {
                        return item;
                    }
                }
            }
        }
    }
    
    private boolean casNext(Node<T> node, Node<T> expected, Node<T> update) {
        return UNSAFE.compareAndSwapObject(node, NEXT_OFFSET, expected, update);
    }
}
`

---

!!! info "Contenido Completo"
    
    Esta es una vista previa de la sección de algoritmos lock-free. El documento completo incluiría:
    
    - Más estructuras de datos (Hash Tables, Binary Trees)
    - Memory ordering models (Sequential Consistency, Release/Acquire)
    - ABA problem y soluciones (Hazard Pointers, Epochs)
    - Benchmarks de performance vs locks
    - Testing con jcstress

---

## 📚 Referencias

- [**Michael & Scott**](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf) - Simple, Fast, and Practical Non-Blocking Queues
- [**Herlihy & Shavit**](https://www.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) - The Art of Multiprocessor Programming
