---
theme: default
title: 'Ch. 27 — A Heap Implementation'
class: text-left
fonts: { mono: 'Space Mono', sans: 'Space Mono' }
colorSchema: dark
transition: slide-left
mdc: true
routerMode: hash
---

# A Heap Implementation
Chapter 27 · Data Structures and Abstractions with Java™ (5th ed.)

<div class="opacity-60 mt-8">COSC2006 · Data Structures I</div>

<style>
:root { --slidev-theme-primary: #60a5fa }
h1 { color: #60a5fa !important } h2 { color: #fb923c !important }
.ok { color: #4ade80 } .hl { color: #fb923c } .bl { color: #60a5fa }
</style>

---

# Roadmap

1. What a heap is (and what it is *not*)
2. Storing a tree **inside an array** — the index trick
3. `add` — bubble a value **up**
4. `removeMax` — sink a value **down** (`reheap`)
5. Building a heap fast
6. Heap sort

<div class="mt-6">Colour code used in every diagram:</div>
<span class="bl">■ normal</span> &nbsp; <span class="hl">■ being compared / moved</span> &nbsp; <span class="ok">■ settled / sorted</span> &nbsp; ┅ empty hole

---
layout: center
class: text-center
---

# 1 · What a heap is (and what it is *not*)

<div class="opacity-60 mt-4">complete binary trees, heaps and maxheaps</div>

---

# Quick recall: complete binary tree

A binary tree is **complete** when every level is full except possibly the last, and the last level is filled **left to right** with no gaps.

<div class="grid grid-cols-2 gap-6 mt-4 text-center">
<div><HeapViz :a="[1,2,3,4,5,6]" :arr="false" :idx="false" :w="300" /><div class="ok">✔ complete</div></div>
<div><HeapViz :a="[1,2,3,null,5,6]" :arr="false" :idx="false" :w="300" /><div class="text-red-400">✘ gap at position 4</div></div>
</div>

Completeness is what will let us use a plain array later — no gaps means no wasted slots.

---

# Heap and maxheap

- **Heap**: a complete binary tree whose nodes contain `Comparable` objects
- **Maxheap**: each node's object is **≥ the objects in all its descendants**
  - So the **largest value is always at the root**
  - (A *minheap* is the mirror image: smallest at the root)

<div class="grid grid-cols-2 gap-6 mt-2 text-center">
<div><HeapViz :a="[90,80,60,70,30,20]" :arr="false" :idx="false" :w="300" /><div class="ok">✔ maxheap</div></div>
<div><HeapViz :a="[90,80,60,70,85,20]" :hl="[4]" :arr="false" :idx="false" :w="300" /><div class="text-red-400">✘ 85 &gt; its parent 80</div></div>
</div>

<div class="text-sm opacity-70">Note: siblings are <b>not</b> ordered (80 vs 60) — a heap is only partially ordered.</div>

---

# Extra: Complete vs non-complete binary trees (1/4)

<div class="text-center"><HeapViz :a="[1,2,3,4,5,6,7,8,9,10,11]" :ok="[0,1,2,3,4,5,6,7,8,9,10]" :arr="false" :w="520" /></div>

<div class="mt-2">✔ **Complete.** Levels 1–3 are completely full; the last level holds 4 nodes packed from the **left**, with no gap. Positions 1…11 are all used.</div>

---

# Extra: Complete vs non-complete binary trees (2/4)

<div class="text-center"><HeapViz :a="[1,2,3,4,5,6,7,8,null,10,11]" :hl="[9,10]" :arr="false" :w="520" /></div>

<div class="mt-2">✘ **Not complete.** Position 9 is empty (dashed) but positions 10 and 11 are used. The last level may not have a gap.</div>

---

# Extra: Complete vs non-complete binary trees (3/4)

<div class="text-center"><HeapViz :a="[1,2,3,4,5,null,7,8,9]" :hl="[7,8]" :arr="false" :w="520" /></div>

<div class="mt-2">✘ **Not complete.** Position 6 is missing, so level 3 is not full, yet level 4 already has nodes (8 and 9). A level may only start once the one above is full.</div>

---

# Extra: Complete vs non-complete binary trees (4/4)

<div class="text-center"><HeapViz :a="[1,2,3,4,5,6,7,8,9]" :ok="[0,1,2,3,4,5,6,7,8]" :arr="false" :w="520" /></div>

<div class="mt-2">**Quick test:** number the positions 1, 2, 3, … level by level, left to right. A tree is complete exactly when positions 1…n are all used. This tree fills the gaps of the previous example: same 9 nodes, no holes.</div>

---
layout: center
class: text-center
---

# 2 · Storing a tree inside an array

<div class="opacity-60 mt-4">the ADT, the index trick, the class</div>

---

# The ADT maxheap: interface

```java {all|3|4-5|6-8}
/** An interface for the ADT maxheap. */
public interface MaxHeapInterface<T extends Comparable<? super T>>
{
   public void add(T newEntry);     // insert, keep the heap property
   public T removeMax();            // remove and return the largest
   public T getMax();               // look at the largest, no removal
   public boolean isEmpty();
   public int getSize();
   public void clear();
} // end MaxHeapInterface
```

`T extends Comparable<? super T>` is what allows `compareTo` between entries.

---

# An array can represent a complete tree

Number the nodes in **level order** (top→bottom, left→right). That number is the **array index**.

<div class="text-center mt-2"><HeapViz :a="[90,80,60,70,30,20,50,10,40]" :w="480" /></div>

- Read the tree row by row → you get the array
- We **skip index 0** on purpose (next slide shows why)

---

# The index trick

For a node at index <b class="hl">i</b>:

| | formula | example i = 5 (value 30) |
|---|---|---|
| left child | `2i` | 10 |
| right child | `2i + 1` | 11 |
| parent | `i / 2` (integer division) | 2 (value 80) |

<div class="text-center"><HeapViz :a="[90,80,60,70,30,20,50,10,40]" :hl="[1,4,8]" :w="420" /></div>

Starting at 1 makes the formulas this clean. With 0-based arrays they become `2i+1`, `2i+2`, `(i-1)/2` — we'll need that for heap sort.

---

# Class MaxHeap — fields and constructor

```java {all|4-5|6-8|10-20}
public final class MaxHeap<T extends Comparable<? super T>>
             implements MaxHeapInterface<T>
{
   private T[] heap;      // Array of heap entries; ignore heap[0]
   private int lastIndex; // Index of last entry AND number of entries
   private boolean integrityOK = false;
   private static final int DEFAULT_CAPACITY = 25;
   private static final int MAX_CAPACITY = 10000;

   public MaxHeap(int initialCapacity)
   {
      if (initialCapacity < DEFAULT_CAPACITY)
         initialCapacity = DEFAULT_CAPACITY;
      else
         checkCapacity(initialCapacity);
      @SuppressWarnings("unchecked")   // safe: array holds only nulls
      T[] tempHeap = (T[])new Comparable[initialCapacity + 1]; // +1: slot 0 unused
      heap = tempHeap;
      lastIndex = 0;                   // empty heap
      integrityOK = true;
   } // end constructor
```

---

# Class MaxHeap — the easy methods

```java {all|1-4|6-12|14-17|19-22}
   public MaxHeap()
   {
      this(DEFAULT_CAPACITY); // reuse the other constructor
   }

   public T getMax()                 // the max is always the root
   {
      checkIntegrity();
      T root = null;
      if (!isEmpty())
         root = heap[1];
      return root;
   }

   public boolean isEmpty() { return lastIndex < 1; }

   public int getSize()     { return lastIndex; }  // index of last = count
```

Because index 0 is unused, `lastIndex` doubles as the **size**.

---

# Class MaxHeap — what's left

```java {all|1-5|7-9|11-17}
   public void add(T newEntry)       { /* next: slides 13–17 */ }

   public T removeMax()              { /* then: slides 18–24 */ }

   public void clear()
   {
      checkIntegrity();
      while (lastIndex > -1)         // null out every slot
      {
         heap[lastIndex] = null;
         lastIndex--;
      }
      lastIndex = 0;                 // size back to 0
   }

   // private: checkIntegrity(), checkCapacity(), ensureCapacity(), reheap()
```

<div class="text-sm opacity-70">Helpers: <code>checkIntegrity</code> throws if the constructor didn't finish; <code>checkCapacity</code> rejects sizes above MAX_CAPACITY; <code>ensureCapacity</code> doubles the array when it is full.</div>

---
layout: center
class: text-center
---

# 3 · `add` — bubble a value **up**

<div class="opacity-60 mt-4">with swaps · without swaps</div>

---

# Adding an entry — the big idea

Add **85** to this maxheap.

<div class="grid grid-cols-2 gap-4">
<div class="text-center"><HeapViz :a="[90,80,60,70,30,20,50,10,40]" :arr="false" :idx="false" :w="320" /></div>
<div>

1. **Keep it complete**: the new value *must* go in the next free spot — the leftmost open slot on the last level
2. **Restore the order**: while it is bigger than its parent, move it **up**

Step 2 touches at most one node per level → **O(log n)**

</div></div>

---

# Adding 85 — with swaps

<div class="grid grid-cols-3 gap-2 text-center text-sm">
<div>① put 85 in the next leaf<br><HeapViz :a="[90,80,60,70,30,20,50,10,40,85]" :hl="[9]" :arr="false" :idx="false" :w="260" /></div>
<div>② 85 &gt; parent 30 → swap, then 85 &gt; 80 → swap<br><HeapViz :a="[90,85,60,70,80,20,50,10,40,30]" :hl="[1]" :arr="false" :idx="false" :w="260" /></div>
<div>③ 85 &lt; 90 → stop. Maxheap again ✔<br><HeapViz :a="[90,85,60,70,80,20,50,10,40,30]" :ok="[0,1]" :arr="false" :idx="false" :w="260" /></div>
</div>

Each swap writes **two** values. Can we do better? →

---

# Adding 85 — without swaps (the trick)

Keep the new value **in your hand** and only move parents *down* into the hole. Write once at the end.

<div class="grid grid-cols-4 gap-1 text-center text-xs">
<div>a) open a hole at the new leaf; 30 &lt; 85 → slide 30 down<br><HeapViz :a="[90,80,60,70,30,20,50,10,40,null]" :hl="[4]" :arr="false" :idx="false" :w="200" /></div>
<div>b) hole now at 5; 80 &lt; 85 → slide 80 down<br><HeapViz :a="[90,80,60,70,null,20,50,10,40,30]" :hl="[1]" :arr="false" :idx="false" :w="200" /></div>
<div>c) hole at 2; parent 90 &gt; 85 → stop<br><HeapViz :a="[90,null,60,70,80,20,50,10,40,30]" :hl="[0]" :arr="false" :idx="false" :w="200" /></div>
<div>d) drop 85 into the hole ✔<br><HeapViz :a="[90,85,60,70,80,20,50,10,40,30]" :ok="[1]" :arr="false" :idx="false" :w="200" /></div>
</div>

Only **one write per level** instead of two. The same idea returns in `reheap`.

---

# The same steps, as an array

<div class="grid grid-cols-1 gap-4 text-center text-sm">
<div>new leaf at 10 (hole)<br><HeapViz :a="[90,80,60,70,30,20,50,10,40,null]" :hl="[9]" :tree="false" :w="400" /></div>
<div>30 moved from 5 → 10<br><HeapViz :a="[90,80,60,70,null,20,50,10,40,30]" :hl="[4]" :tree="false" :w="400" /></div>
<div>80 moved from 2 → 5<br><HeapViz :a="[90,null,60,70,80,20,50,10,40,30]" :hl="[1]" :tree="false" :w="400" /></div>
<div>85 placed at 2 ✔<br><HeapViz :a="[90,85,60,70,80,20,50,10,40,30]" :ok="[1]" :tree="false" :w="400" /></div>
</div>

The hole travels along the path 10 → 5 → 2 using `parent = i / 2`.

---

# Algorithm: add

```text {all|2-3|4-9|10-11}
Algorithm add(newEntry)
// Precondition: the array heap has room for another entry
newIndex    = index of next available array location
parentIndex = newIndex / 2
while (parentIndex > 0 and newEntry > heap[parentIndex])
{
   heap[newIndex] = heap[parentIndex]     // move parent DOWN into the hole
   newIndex       = parentIndex           // the hole moves UP
   parentIndex    = newIndex / 2
}
heap[newIndex] = newEntry                 // place the new entry
if (the array heap is full) double its size
```

`parentIndex > 0` stops us at the root (index 1 has parent 0 = unused).

---

# Method add — in Java

```java {all|3-4|5-10|12-13}
public void add(T newEntry)
{
   checkIntegrity();
   int newIndex = lastIndex + 1;
   int parentIndex = newIndex / 2;
   while ((parentIndex > 0) && newEntry.compareTo(heap[parentIndex]) > 0)
   {
      heap[newIndex] = heap[parentIndex];
      newIndex = parentIndex;
      parentIndex = newIndex / 2;
   } // end while

   heap[newIndex] = newEntry;
   lastIndex++;
   ensureCapacity();
} // end add
```

`compareTo(...) > 0` reads as "newEntry is greater than parent".

---

# Extra: Adding to a min heap, with swaps (1/5)

<div class="text-center"><HeapViz :a="[10,20,15,30,25,18,40,35,32,28]" :arr="false" :w="520" /></div>

<div class="mt-2">A **min heap**: every parent ≤ its children, so the **smallest** value is at the root. Goal: add **12**.</div>

---

# Extra: Adding to a min heap, with swaps (2/5)

<div class="text-center"><HeapViz :a="[10,20,15,30,25,18,40,35,32,28,12]" :hl="[10]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 1.** Put 12 in the next free leaf: position 11, the child of position 5 (value 25).</div>

---

# Extra: Adding to a min heap, with swaps (3/5)

<div class="text-center"><HeapViz :a="[10,20,15,30,12,18,40,35,32,28,25]" :hl="[4,10]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 2.** 12 &lt; parent 25, so **swap**. 12 is now at position 5; its parent is position 2 (value 20).</div>

---

# Extra: Adding to a min heap, with swaps (4/5)

<div class="text-center"><HeapViz :a="[10,12,15,30,20,18,40,35,32,28,25]" :hl="[1,4]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 3.** 12 &lt; parent 20, so **swap**. 12 is now at position 2; its parent is the root (10).</div>

---

# Extra: Adding to a min heap, with swaps (5/5)

<div class="text-center"><HeapViz :a="[10,12,15,30,20,18,40,35,32,28,25]" :ok="[1]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 4.** 12 ≥ parent 10, so **stop**. It is a min heap again. 2 swaps along one root-to-leaf path: O(log n).</div>

---

# Extra: Adding to a max heap, without swaps (1/5)

<div class="text-center"><HeapViz :a="[95,80,90,60,70,85,50,40,55,65]" :arr="false" :w="520" /></div>

<div class="mt-2">A **max heap**. Goal: add **92**. We hold 92 "in our hand" and only move parents **down** into a hole, writing 92 once at the end.</div>

---

# Extra: Adding to a max heap, without swaps (2/5)

<div class="text-center"><HeapViz :a="[95,80,90,60,70,85,50,40,55,65,null]" :hl="[4]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 1.** Open a hole in the next free leaf (position 11). Its parent is position 5 (value 70). 70 &lt; 92, so move 70 **down** into the hole.</div>

---

# Extra: Adding to a max heap, without swaps (3/5)

<div class="text-center"><HeapViz :a="[95,80,90,60,null,85,50,40,55,65,70]" :hl="[1]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 2.** 70 moved down; the hole is now at position 5. Its parent is position 2 (value 80). 80 &lt; 92, so move 80 down.</div>

---

# Extra: Adding to a max heap, without swaps (4/5)

<div class="text-center"><HeapViz :a="[95,null,90,60,80,85,50,40,55,65,70]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 3.** 80 moved down; the hole is at position 2. Its parent is the root (95). 95 ≥ 92, so **stop**.</div>

---

# Extra: Adding to a max heap, without swaps (5/5)

<div class="text-center"><HeapViz :a="[95,92,90,60,80,85,50,40,55,65,70]" :ok="[1]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 4.** Drop 92 into the hole. One write per level instead of two. The result is a max heap.</div>

---
layout: center
class: text-center
---

# 4 · `removeMax` — sink a value **down**

<div class="opacity-60 mt-4">`reheap`, with swaps · without swaps</div>

---

# Removing the max — the big idea

The max is the root. Deleting it leaves a hole at the top, but a complete tree can only lose its **last** leaf.

1. Save the root (that is the answer)
2. Move the **last leaf's value** into the root → shape is complete again
3. The root may now be too small → this is a **semiheap**: both subtrees are heaps, only the root may be wrong
4. Sink it **down** until it fits → `reheap`

<div class="text-center mt-2"><span class="ok">semiheap</span> = a tree whose left and right subtrees are heaps, but whose root may violate the order</div>

---

# Removing the root — with swaps

Heap `[90,85,60,70,80,20,50,10,40,30]` → remove 90.

<div class=" gap-1 text-center text-xs">
<div>replace root with last leaf (30)<br><HeapViz :a="[90,85,60,70,80,20,50,10,40,30]" :hl="[0,9]" :arr="false" :idx="false" :w="400" /></div>
</div>

⚠ Always swap with the **larger** child — otherwise the smaller child would end up above the larger one.

---

# Removing the root — with swaps

Heap `[90,85,60,70,80,20,50,10,40,30]` → remove 90.

<div class="grid grid-cols-4 gap-1 text-center text-xs">
<div>replace root with last leaf (30)<br><HeapViz :a="[30,85,60,70,80,20,50,10,40]" :hl="[0]" :arr="false" :idx="false" :w="200" /></div>
<div>larger child of 30 is 85 → swap<br><HeapViz :a="[85,30,60,70,80,20,50,10,40]" :hl="[1]" :arr="false" :idx="false" :w="200" /></div>
<div>larger child is 80 → swap<br><HeapViz :a="[85,80,60,70,30,20,50,10,40]" :hl="[4]" :arr="false" :idx="false" :w="200" /></div>
<div>30 is a leaf → done ✔<br><HeapViz :a="[85,80,60,70,30,20,50,10,40]" :ok="[4]" :arr="false" :idx="false" :w="200" /></div>
</div>

⚠ Always swap with the **larger** child — otherwise the smaller child would end up above the larger one.

---

# Removing the root — without swaps

Hold the "orphan" (30) in your hand; slide the larger child **up** into the hole.

<div class="grid grid-cols-4 gap-1 text-center text-xs">
<div>copy 30 aside; hole at root<br><HeapViz :a="[null,85,60,70,80,20,50,10,40]" :hl="[1,2]" :arr="false" :idx="false" :w="200" /></div>
<div>85 &gt; 60 → 85 moves up<br><HeapViz :a="[85,null,60,70,80,20,50,10,40]" :hl="[3,4]" :arr="false" :idx="false" :w="200" /></div>
<div>80 &gt; 70 and 80 &gt; 30 → 80 moves up<br><HeapViz :a="[85,80,60,70,null,20,50,10,40]" :hl="[7,8]" :arr="false" :idx="false" :w="200" /></div>
<div>hole has no children → drop 30 in ✔<br><HeapViz :a="[85,80,60,70,30,20,50,10,40]" :ok="[4]" :arr="false" :idx="false" :w="250" /></div>
</div>

Stops early if the orphan is already ≥ the larger child.

---

# Algorithm: reheap

```text {all|3-4|5-12|13}
Algorithm reheap(rootIndex)
// Transforms the semiheap rooted at rootIndex into a heap
done   = false
orphan = heap[rootIndex]
while (!done and heap[rootIndex] has a child)
{
   largerChildIndex = index of the larger child of heap[rootIndex]
   if (orphan < heap[largerChildIndex])
   {
      heap[rootIndex] = heap[largerChildIndex]   // child moves UP
      rootIndex       = largerChildIndex         // the hole moves DOWN
   }
   else done = true                              // orphan fits here
}
heap[rootIndex] = orphan
```

Mirror image of `add`: **add** moves a hole *up*, **reheap** moves it *down*.

---

# reheap — in Java

```java {all|3-5|7-14|16-22|24}
private void reheap(int rootIndex)
{
   boolean done = false;
   T orphan = heap[rootIndex];
   int leftChildIndex = 2 * rootIndex;

   while (!done && (leftChildIndex <= lastIndex))     // has a child?
   {
      int largerChildIndex = leftChildIndex;           // assume left is larger
      int rightChildIndex = leftChildIndex + 1;

      if ((rightChildIndex <= lastIndex) &&            // right child exists AND
           heap[rightChildIndex].compareTo(heap[largerChildIndex]) > 0)
         largerChildIndex = rightChildIndex;           // ...is larger

      if (orphan.compareTo(heap[largerChildIndex]) < 0)
      {
         heap[rootIndex] = heap[largerChildIndex];
         rootIndex = largerChildIndex;
         leftChildIndex = 2 * rootIndex;
      }
      else
         done = true;
   }
   heap[rootIndex] = orphan;
}
```

---

# removeMax — putting it together

```java {all|5|6-8|9|11}
public T removeMax()
{
   checkIntegrity();
   T root = null;
   if (!isEmpty())
   {
      root = heap[1];             // save the answer
      heap[1] = heap[lastIndex];  // last leaf → root: a semiheap
      lastIndex--;                // shrink BEFORE reheap
      reheap(1);                  // sink it down
   }
   return root;
}
```

**Cost:** `reheap` follows one root-to-leaf path → **O(log n)**. `add` is also **O(log n)**.

---

# Extra: Removing the root from a min heap, with swaps (1/5)

<div class="text-center"><HeapViz :a="[10,12,15,30,20,18,40,35,32,28,25]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="mt-2">Remove the root, **10** (the minimum). It is the value we return. The heap has 11 entries.</div>

---

# Extra: Removing the root from a min heap, with swaps (2/5)

<div class="text-center"><HeapViz :a="[25,12,15,30,20,18,40,35,32,28]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 1.** Move the **last leaf** (25) to the root and drop that leaf. The shape is complete again, but 25 is too big here: a *semiheap*. Children: 12 and 15. The **smaller** is 12.</div>

---

# Extra: Removing the root from a min heap, with swaps (3/5)

<div class="text-center"><HeapViz :a="[12,25,15,30,20,18,40,35,32,28]" :hl="[1]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 2.** Swap with the **smaller** child: 25 ↔ 12. Now 25 sits at position 2; its children are 30 and 20. The smaller is 20.</div>

---

# Extra: Removing the root from a min heap, with swaps (4/5)

<div class="text-center"><HeapViz :a="[12,20,15,30,25,18,40,35,32,28]" :hl="[4]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 3.** Swap 25 ↔ 20. Now 25 sits at position 5; its only child is 28 (position 10).</div>

---

# Extra: Removing the root from a min heap, with swaps (5/5)

<div class="text-center"><HeapViz :a="[12,20,15,30,25,18,40,35,32,28]" :ok="[4]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 4.** 25 ≤ 28, so **stop**. A valid min heap with 10 entries; the new minimum is 12.</div>

---

# Extra: Removing the root from a max heap, without swaps (1/5)

<div class="text-center"><HeapViz :a="[95,92,90,60,80,85,50,40,55,65,70]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="mt-2">Remove the root, **95** (the maximum): we return it. The last leaf, **70**, will be the *orphan* that we place later.</div>

---

# Extra: Removing the root from a max heap, without swaps (2/5)

<div class="text-center"><HeapViz :a="[null,92,90,60,80,85,50,40,55,65]" :hl="[1,2]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 1.** Take 70 out of the last leaf and hold it. The root is now a hole. Its children: 92 and 90. The **larger** is 92, and 92 &gt; 70.</div>

---

# Extra: Removing the root from a max heap, without swaps (3/5)

<div class="text-center"><HeapViz :a="[92,null,90,60,80,85,50,40,55,65]" :hl="[3,4]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 2.** 92 moves **up** into the hole. The hole is now at position 2; its children are 60 and 80. The larger is 80, and 80 &gt; 70.</div>

---

# Extra: Removing the root from a max heap, without swaps (4/5)

<div class="text-center"><HeapViz :a="[92,80,90,60,null,85,50,40,55,65]" :hl="[9]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 3.** 80 moves up. The hole is at position 5; its only child is 65, and 65 &lt; 70.</div>

---

# Extra: Removing the root from a max heap, without swaps (5/5)

<div class="text-center"><HeapViz :a="[92,80,90,60,70,85,50,40,55,65]" :ok="[4]" :arr="false" :w="520" /></div>

<div class="mt-2">**Step 4.** 65 &lt; 70, so stop and drop 70 into the hole. One write per level. A valid max heap with 10 entries.</div>

---
layout: center
class: text-center
---

# 5 · Building a heap fast

<div class="opacity-60 mt-4">repeated adds vs `reheap`</div>

---

# Creating a heap: two approaches

Build from `20, 40, 30, 10, 90, 70`.

**A. Repeated `add`** — n adds, each O(log n) → **O(n log n)**

<div class="grid grid-cols-3 gap-2 text-center text-xs mt-2">
<div>add 20, 40 → 40 rises<br><HeapViz :a="[40,20]" :arr="false" :idx="false" :w="200" /></div>
<div>add 30, 10<br><HeapViz :a="[40,20,30,10]" :arr="false" :idx="false" :w="200" /></div>
<div>add 90, 70 → 90 rises to root<br><HeapViz :a="[90,40,70,10,20,30]" :arr="false" :idx="false" :w="200" /></div>
</div>

**B. `reheap` bottom-up** — faster; next slide ↓

---

# Creating a heap with reheap

Copy the array in, then call `reheap` on every **non-leaf**, from the last one back to the root.

<div class="grid grid-cols-4 gap-1 text-center text-xs">
<div>copy array (as is)<br><HeapViz :a="[20,40,30,10,90,70]" :w="210" /></div>
<div>reheap(3): 30 vs 70 → swap<br><HeapViz :a="[20,40,70,10,90,30]" :hl="[2]" :w="210" /></div>
<div>reheap(2): 40 vs 90 → swap<br><HeapViz :a="[20,90,70,10,40,30]" :hl="[1]" :w="210" /></div>
<div>reheap(1): 20 sinks to 5 ✔<br><HeapViz :a="[90,40,70,10,20,30]" :ok="[0,1,2,3,4,5]" :w="210" /></div>
</div>

**Why start at `n/2`?** Indices above `n/2` are leaves — a leaf is already a one-node heap. Each non-leaf is a semiheap by the time we reach it, because everything below was already fixed.

---

# Constructor that builds a heap

```java {all|3-5|7-9|11-13}
public MaxHeap(T[] entries)
{
   this(entries.length);        // allocate
   lastIndex = entries.length;

   // copy into slots 1..n
   for (int index = 0; index < entries.length; index++)
      heap[index + 1] = entries[index];

   // fix non-leaves, last one first
   for (int rootIndex = lastIndex / 2; rootIndex > 0; rootIndex--)
      reheap(rootIndex);
}
```

Most nodes are near the bottom and sink only a step or two, so total work is **O(n)** — beating n calls to `add`.

---

# Extra: Creating a max heap with repeated adds (1/6)

<div class="text-center"><HeapViz :a="[50,30]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="mt-2">Input order: **30, 50, 20, 70, 40, 60, 10, 80, 65, 35**. Add 30 (it becomes the root), then 50: it is bigger than its parent, so it swaps up.</div>

---

# Extra: Creating a max heap with repeated adds (2/6)

<div class="text-center"><HeapViz :a="[70,50,20,30]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="mt-2">Add 20 (fits under 50). Add 70: it swaps with 30, then with 50, and reaches the root.</div>

---

# Extra: Creating a max heap with repeated adds (3/6)

<div class="text-center"><HeapViz :a="[70,50,60,30,40,20,10]" :hl="[2]" :arr="false" :w="520" /></div>

<div class="mt-2">Add 40: stays (parent 50 is bigger). Add 60: swaps with 20 and stops below 70. Add 10: stays. The first three levels are full.</div>

---

# Extra: Creating a max heap with repeated adds (4/6)

<div class="text-center"><HeapViz :a="[70,50,60,30,40,20,10,80]" :hl="[7]" :arr="false" :w="520" /></div>

<div class="mt-2">Add **80**: first it goes into the next free leaf, position 8 (below 30).</div>

---

# Extra: Creating a max heap with repeated adds (5/6)

<div class="text-center"><HeapViz :a="[80,70,60,50,40,20,10,30]" :hl="[0,1,3]" :arr="false" :w="520" /></div>

<div class="mt-2">80 bubbles up: swap with 30, then with 50, then with 70. It is the new root. 3 swaps = the height of the tree.</div>

---

# Extra: Creating a max heap with repeated adds (6/6)

<div class="text-center"><HeapViz :a="[80,70,60,65,40,20,10,30,50,35]" :hl="[3]" :arr="false" :w="520" /></div>

<div class="mt-2">Add 65: swaps with 50 and stops below 70. Add 35: stays under 40. **Final max heap.** n adds × O(log n) = O(n log n).</div>

---

# Extra: Creating a max heap with reheap (1/8)

<div class="text-center"><HeapViz :a="[35,15,60,20,45,10,80,25,50,70]" :dim="[5,6,7,8,9]" :arr="false" :w="520" /></div>

<div class="mt-2">Copy the 10 values into a complete tree. It is **not** a heap yet. The leaves (grey, positions 6–10) are already one-node heaps. Call **reheap** on positions 5, 4, 3, 2, 1.</div>

---

# Extra: Creating a max heap with reheap (2/8)

<div class="text-center"><HeapViz :a="[35,15,60,20,70,10,80,25,50,45]" :hl="[4,9]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(5):** 45 has one child, 70. 70 &gt; 45, so swap.</div>

---

# Extra: Creating a max heap with reheap (3/8)

<div class="text-center"><HeapViz :a="[35,15,60,50,70,10,80,25,20,45]" :hl="[3,8]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(4):** children 25 and 50. The larger, 50, is &gt; 20, so swap.</div>

---

# Extra: Creating a max heap with reheap (4/8)

<div class="text-center"><HeapViz :a="[35,15,80,50,70,10,60,25,20,45]" :hl="[2,6]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(3):** children 10 and 80. Swap 60 with 80.</div>

---

# Extra: Creating a max heap with reheap (5/8)

<div class="text-center"><HeapViz :a="[35,70,80,50,15,10,60,25,20,45]" :hl="[1,4]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(2), part 1:** children 50 and 70. Swap 15 with 70.</div>

---

# Extra: Creating a max heap with reheap (6/8)

<div class="text-center"><HeapViz :a="[35,70,80,50,45,10,60,25,20,15]" :hl="[4,9]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(2), part 2:** 15 keeps sinking. Its only child, 45, is bigger, so swap. Position 2 is now a heap.</div>

---

# Extra: Creating a max heap with reheap (7/8)

<div class="text-center"><HeapViz :a="[80,70,35,50,45,10,60,25,20,15]" :hl="[0,2]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(1), part 1:** children 70 and 80. Swap 35 with 80.</div>

---

# Extra: Creating a max heap with reheap (8/8)

<div class="text-center"><HeapViz :a="[80,70,60,50,45,10,35,25,20,15]" :ok="[0,1,2,3,4,5,6,7,8,9]" :arr="false" :w="520" /></div>

<div class="mt-2">**reheap(1), part 2:** 35 sinks again (children 10 and 60): swap with 60. **Done: a max heap**, built with only 7 swaps in total (O(n)).</div>

---
layout: center
class: text-center
---

# 6 · Heap sort

<div class="opacity-60 mt-4">build a max heap, then swap and re-heap</div>

---

# Heap sort — the idea

Use the heap's own array as the workspace. Repeat: **the max goes to the end**.

1. Turn the array into a maxheap (reheap bottom-up)
2. **Swap** root ↔ last unsorted element → the largest is now in its final place
3. Shrink the heap by one, `reheap(0)` to repair it
4. Repeat until one element is left

<div class="grid grid-cols-3 gap-2 text-center text-xs mt-2">
<div>original<br><HeapViz :a="[20,40,30,10,90,70]" :zero="true" :tree="true" :w="260" /></div>
<div>after heap creation<br><HeapViz :a="[90,40,70,10,20,30]" :zero="true" :tree="true" :w="260" /></div>
<div>swap 90 ↔ 30<br><HeapViz :a="[30,40,70,10,20,90]" :ok="[5]" :hl="[0]" :zero="true" :tree="true" :w="260" /></div>
</div>

---

### Heap sort — trace

<div class="grid grid-cols-3 gap-2 text-center text-xs">
<div>reheap(0) on first 5<br><HeapViz :a="[70,40,30,10,20,90]" :ok="[5]" :size="5" :zero="true" :w="230" /></div>
<div>swap 70 ↔ 20<br><HeapViz :a="[20,40,30,10,70,90]" :ok="[4,5]" :size="4" :zero="true" :w="230" /></div>
<div>reheap → swap 40 ↔ 10<br><HeapViz :a="[10,20,30,40,70,90]" :ok="[3,4,5]" :size="3" :zero="true" :w="230" /></div>
<div>reheap → swap 30 ↔ 10<br><HeapViz :a="[10,20,30,40,70,90]" :ok="[2,3,4,5]" :size="2" :zero="true" :w="230" /></div>
<div>swap 20 ↔ 10<br><HeapViz :a="[10,20,30,40,70,90]" :ok="[1,2,3,4,5]" :size="1" :zero="true" :w="230" /></div>
<div>sorted ✔<br><HeapViz :a="[10,20,30,40,70,90]" :ok="[0,1,2,3,4,5]" :zero="true" :tree="true" :w="230" /></div>
</div>

The green region grows from the right; the heap (blue) shrinks.

---

# Heap sort needs a 0-based reheap

We are sorting a **plain array** — there is no spare slot 0. So the index formulas change:

| | MaxHeap class (1-based) | Heap sort (0-based) |
|---|---|---|
| left child | `2i` | `2i + 1` |
| right child | `2i + 1` | `2i + 2` |
| last non-leaf | `n / 2` | `n / 2 - 1` |

---

# Heap sort needs a 0-based reheap

```java {all|1-2|7|15,18}
private static <T extends Comparable<? super T>>
        void reheap(T[] heap, int rootIndex, int lastIndex)
{
   boolean done = false;
   T orphan = heap[rootIndex];
   int leftChildIndex = 2 * rootIndex + 1;          // ← 0-based
   while (!done && (leftChildIndex <= lastIndex))
   {
      int largerChildIndex = leftChildIndex;
      int rightChildIndex = leftChildIndex + 1;
      if ((rightChildIndex <= lastIndex) &&
           heap[rightChildIndex].compareTo(heap[largerChildIndex]) > 0)
         largerChildIndex = rightChildIndex;
      if (orphan.compareTo(heap[largerChildIndex]) < 0)
      {  heap[rootIndex] = heap[largerChildIndex];
         rootIndex = largerChildIndex;
         leftChildIndex = 2 * rootIndex + 1; }       // ← 0-based
      else done = true;
   }
   heap[rootIndex] = orphan;
}
```

It also takes `lastIndex` as a parameter, because the heap part of the array keeps shrinking.

---

# The heapSort method

```java {all|4-6|8|10-14}
public static <T extends Comparable<? super T>>
       void heapSort(T[] array, int n)
{
   // 1. create the first heap (last non-leaf down to the root)
   for (int rootIndex = n / 2 - 1; rootIndex >= 0; rootIndex--)
      reheap(array, rootIndex, n - 1);

   swap(array, 0, n - 1);                    // 2. max to its final place

   for (int lastIndex = n - 2; lastIndex > 0; lastIndex--)
   {
      reheap(array, 0, lastIndex);           // 3. repair the smaller heap
      swap(array, 0, lastIndex);             // 4. next max to its place
   }
}
```

**Time: O(n log n)**, in place, no extra array. (`swap` simply exchanges two array slots.)

---

# Extra: Heap sort with numbers (1/10)

<div class="text-center"><HeapViz :a="[40,15,60,25,50,10,35,70]" :dim="[4,5,6,7]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Heap sort** = (1) turn the array into a max heap, (2) repeatedly swap the root with the last unsorted entry and re-heap the rest. First, build the heap: the non-leaves are positions 4, 3, 2, 1 (leaves are grey).</div>

---

# Extra: Heap sort with numbers (2/10)

<div class="text-center"><HeapViz :a="[40,15,60,70,50,10,35,25]" :hl="[3,7]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Build, part 1.** reheap(4): 25 swaps with its child 70. reheap(3): 60 is already larger than 10 and 35, so nothing moves.</div>

---

# Extra: Heap sort with numbers (3/10)

<div class="text-center"><HeapViz :a="[40,70,60,25,50,10,35,15]" :hl="[1,3,7]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Build, part 2.** reheap(2): 15 swaps with 70, then sinks again, swapping with 25.</div>

---

# Extra: Heap sort with numbers (4/10)

<div class="text-center"><HeapViz :a="[70,50,60,25,40,10,35,15]" :hl="[0,1,4]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Build, part 3.** reheap(1): 40 swaps with 70, then with 50. Max heap built; the largest value, 70, is at the root.</div>

---

# Extra: Heap sort with numbers (5/10)

<div class="text-center"><HeapViz :a="[60,50,35,25,40,10,15]" :hl="[0,2,6]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">70</b></div>

<div class="mt-2">**Round 1.** Swap the root 70 with the last entry (15). 70 is now in its final place, outside the tree. Re-heap the 7 remaining: 15 sinks via 60, then 35.</div>

---

# Extra: Heap sort with numbers (6/10)

<div class="text-center"><HeapViz :a="[50,40,35,25,15,10]" :hl="[0,1,4]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">60 70</b></div>

<div class="mt-2">**Round 2.** Swap 60 with the last entry (15). Re-heap 6: 15 sinks via 50, then 40.</div>

---

# Extra: Heap sort with numbers (7/10)

<div class="text-center"><HeapViz :a="[40,25,35,10,15]" :hl="[0,1,3]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">50 60 70</b></div>

<div class="mt-2">**Round 3.** Swap 50 with the last entry (10). Re-heap 5: 10 sinks via 40, then 25.</div>

---

# Extra: Heap sort with numbers (8/10)

<div class="text-center"><HeapViz :a="[35,25,15,10]" :hl="[0,2]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">40 50 60 70</b></div>

<div class="mt-2">**Round 4.** Swap 40 with the last entry (15). Re-heap 4: 15 swaps with its larger child 35 and has no more children.</div>

---

# Extra: Heap sort with numbers (9/10)

<div class="text-center"><HeapViz :a="[15,10]" :hl="[0]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">25 35 40 50 60 70</b></div>

<div class="mt-2">**Rounds 5 and 6.** Swap 35 with 10; 10 sinks below 25 → [25, 10, 15]. Swap 25 with 15; 15 ≥ 10, so no sink → [15, 10].</div>

---

# Extra: Heap sort with numbers (10/10)

<div class="text-center"><HeapViz :a="[10]" :ok="[0]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">10 15 25 35 40 50 60 70</b></div>

<div class="mt-2">**Round 7.** Swap 15 with 10. One entry is left and it is already in place. Sorted ascending: **10 15 25 35 40 50 60 70**. Total O(n log n).</div>

---

# Extra: Heap sort with letters (1/10)

<div class="text-center"><HeapViz :a="['H','E','A','P','S','O','R','T']" :dim="[4,5,6,7]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Heap sort** = (1) turn the array into a max heap, (2) repeatedly swap the root with the last unsorted entry and re-heap the rest. First, build the heap: the non-leaves are positions 4, 3, 2, 1 (leaves are grey).</div>

---

# Extra: Heap sort with letters (2/10)

<div class="text-center"><HeapViz :a="['H','E','R','T','S','O','A','P']" :hl="[2,3,6,7]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Build, part 1.** reheap(4): P swaps with its child T. reheap(3): A swaps with R, the larger of O and R.</div>

---

# Extra: Heap sort with letters (3/10)

<div class="text-center"><HeapViz :a="['H','T','R','P','S','O','A','E']" :hl="[1,3,7]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Build, part 2.** reheap(2): E swaps with T, then sinks again and swaps with P.</div>

---

# Extra: Heap sort with letters (4/10)

<div class="text-center"><HeapViz :a="['T','S','R','P','H','O','A','E']" :hl="[0,1,4]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">—</b></div>

<div class="mt-2">**Build, part 3.** reheap(1): H swaps with T, then with S. Max heap built; the alphabetically largest letter, T, is at the root.</div>

---

# Extra: Heap sort with letters (5/10)

<div class="text-center"><HeapViz :a="['S','P','R','E','H','O','A']" :hl="[0,1,3]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">T</b></div>

<div class="mt-2">**Round 1.** Swap the root T with the last entry (E). T is final. Re-heap 7: E sinks via S, then P.</div>

---

# Extra: Heap sort with letters (6/10)

<div class="text-center"><HeapViz :a="['R','P','O','E','H','A']" :hl="[0,2,5]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">S T</b></div>

<div class="mt-2">**Round 2.** Swap S with the last entry (A). Re-heap 6: A sinks via R, then O.</div>

---

# Extra: Heap sort with letters (7/10)

<div class="text-center"><HeapViz :a="['P','H','O','E','A']" :hl="[0,1,4]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">R S T</b></div>

<div class="mt-2">**Round 3.** Swap R with A. Re-heap 5: A sinks via P, then H.</div>

---

# Extra: Heap sort with letters (8/10)

<div class="text-center"><HeapViz :a="['O','H','A','E']" :hl="[0,2]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">P R S T</b></div>

<div class="mt-2">**Round 4.** Swap P with A. Re-heap 4: A swaps with its larger child O and stops.</div>

---

# Extra: Heap sort with letters (9/10)

<div class="text-center"><HeapViz :a="['E','A']" :hl="[0]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">H O P R S T</b></div>

<div class="mt-2">**Rounds 5 and 6.** Swap O with E; E sinks below H → [H, E, A]. Swap H with A; A swaps with E → [E, A].</div>

---

# Extra: Heap sort with letters (10/10)

<div class="text-center"><HeapViz :a="['A']" :ok="[0]" :arr="false" :w="520" /></div>

<div class="text-center text-sm mt-1">Sorted (final places): <b class="ok">A E H O P R S T</b></div>

<div class="mt-2">**Round 7.** Swap E with A. Sorted alphabetically: **A E H O P R S T**.</div>

---

# Summary

| Operation | How | Time |
|---|---|---|
| `getMax` | read `heap[1]` | O(1) |
| `add` | hole in next leaf, move **up** | O(log n) |
| `removeMax` | last leaf → root, `reheap` **down** | O(log n) |
| build from array | `reheap` from `n/2` to 1 | O(n) |
| heap sort | build + n × (swap, reheap) | O(n log n) |

- Complete tree ⇒ array with `2i`, `2i+1`, `i/2`
- No-swap versions: carry the value in your hand, write once
- Heaps are the standard way to implement a **priority queue**

---
layout: end
---

# End of Chapter 27