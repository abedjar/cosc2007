---
theme: default
title: '2-3 and 2-4 Trees'
class: text-left
fonts: { mono: 'Space Mono', sans: 'Space Mono' }
colorSchema: dark
transition: slide-left
mdc: true
---

# 2-3 and 2-4 Trees
Balanced search trees that grow **upward** · after Carrano, *Data Structures and Abstractions with Java™* (5th ed.)

<div class="opacity-60 mt-8">COSC2006 · Data Structures I</div>

<style>
:root { --slidev-theme-primary: #60a5fa }
h1 { color: #60a5fa !important } h2 { color: #fb923c !important }
.ok { color: #4ade80 } .hl { color: #fb923c } .bl { color: #60a5fa } .bd { color: #f87171 }
</style>

---

# Roadmap

1. The problem, and a new way to attack it
2. **2-3 trees:** nodes, definition, search
3. 2-3 insertion: the *split*, then cascading splits
4. 2-3 deletion (idea)
5. **2-4 trees:** why allow bigger nodes, top-down insertion
6. Comparison and summary
7. *Supplement:* Java code

<div class="mt-4 text-sm">Diagrams: <span class="bl">■ normal</span> <span class="hl">■ in focus</span> <span class="ok">■ just fixed</span> <span class="bd">■ too many keys</span> ┅ empty node</div>

---

# Recall: why we balance

- A BST costs O(height); height can reach n (sorted input).
- AVL trees fix this by **rotating** nodes after the fact.

<v-clicks>

- A different idea: **never let the shape become unbalanced** in the first place
- Instead of changing the shape, let each node hold **more than one key**
- Keep **all leaves at the same depth**, always

</v-clicks>

---

# Idea: a node with two keys

A BST node splits the number line into **2** ranges. Add a second key and it splits into **3**.

<div class="grid grid-cols-2 gap-6 text-center mt-2">
<div><MwTree :t="[[20],[[10]],[[30]]]" :w="220" /><div class="text-sm mt-2">1 key, 2 children<br>less than 20 · greater than 20</div></div>
<div><MwTree :t="[[20,40],[[10]],[[30]],[[50]]]" :w="260" /><div class="text-sm mt-2">2 keys, 3 children<br>less than 20 · between · greater than 40</div></div>
</div>

---

# 2-node and 3-node

| node | keys | children | child ranges |
|---|---|---|---|
| **2-node** | 1 (`a`) | 2 | `< a`, `> a` |
| **3-node** | 2 (`a < b`) | 3 | `< a`, between `a` and `b`, `> b` |

A leaf has the keys but **no** children.

Note the naming: it counts **children**, not keys.

---

# Definition: 2-3 tree

A **2-3 tree** is a search tree where

1. every internal node is a **2-node** or a **3-node**
2. keys inside a node are sorted; subtrees obey the ranges
3. **all leaves are on the same level**

<div class="text-center mt-2"><MwTree :t="[[40],[[20],[[10]],[[30]]],[[60,80],[[50]],[[70]],[[90]]]]" :ok="[40]" :w="420" /></div>

Every leaf is two steps from the root. Rule 3 is what we must protect.

---

# Searching a 2-3 tree

Same idea as a BST: compare, then pick **one** child. Search for 70.

<MwSteps :frames="[
{t:[[40],[[20],[[10]],[[30]]],[[60,80],[[50]],[[70]],[[90]]]],hl:[40],note:`70 greater than 40: go to the right child`},
{t:[[40],[[20],[[10]],[[30]]],[[60,80],[[50]],[[70]],[[90]]]],hl:[60,80],note:`70 is greater than 60 and less than 80: take the MIDDLE child`},
{t:[[40],[[20],[[10]],[[30]]],[[60,80],[[50]],[[70]],[[90]]]],ok:[70],note:`found 70. At most 2 comparisons per node, one node per level`}]" />

---

# Why it is always balanced

Insertion will **never** add a new leaf at the bottom of one branch only. Trees grow in just two ways:

<v-clicks>

- a key joins an existing leaf (no new nodes at all), or
- the **root splits**, adding one level to *every* path at once

</v-clicks>

<div class="mt-4">So all root-to-leaf paths stay the same length.</div>

---

# Insertion: the strategy

1. **Search** down to the leaf where the key belongs (never insert into an internal node)
2. If the leaf is a **2-node** → it becomes a 3-node. **Done.**
3. If the leaf is a **3-node** → it temporarily holds 3 keys: **split** it
4. A split sends one key **up** into the parent; repeat if the parent overflows

We start with the easy case.

---

# Case A: the leaf has room

Insert 25 into this tree.

<MwSteps :frames="[
{t:[[20],[[10]],[[30]]],note:`start`},
{t:[[20],[[10]],[[30]]],hl:[20],note:`25 greater than 20: go right`},
{t:[[20],[[10]],[[30]]],hl:[30],note:`leaf [30] is a 2-node: there is room`},
{t:[[20],[[10]],[[25,30]]],ok:[25,30],note:`25 slots in. The leaf is now a 3-node. Nothing else changes`}]" />

---

# Case B: the leaf is full: split

A leaf with 3 keys is too big (a 3-node can hold only 2). **Split it:**

<MwSteps :frames="[
{t:[[10,20]],note:`a single 3-node leaf. Insert 30`},
{t:[[10,20,30]],bad:[10,20,30],note:`temporarily 3 keys: too many`},
{t:[[20],[[10]],[[30]]],ok:[20],note:`split: smallest and largest become two separate nodes; the MIDDLE key (20) moves up and becomes the new root`}]" />

Rule: **small | middle ↑ | large**

---

# Split inside a bigger tree

If the parent is a 2-node, it simply **absorbs** the promoted key.

<MwSteps :frames="[
{t:[[20],[[10]],[[30,40]]],note:`insert 50`},
{t:[[20],[[10]],[[30,40,50]]],bad:[30,40,50],note:`the leaf overflows: 30, 40, 50`},
{t:[[20,40],[[10]],[[30]],[[50]]],ok:[40],note:`split: 40 goes up into the parent (a 2-node, so it becomes a 3-node). Done`}]" />

---

# Cascading splits

What if the **parent** is also full? Then it splits too, and the key moves up again.

<MwSteps :h="240" :frames="[
{t:[[20,40],[[10]],[[30]],[[50,60]]],note:`insert 70. The leaf [50,60] and the parent [20,40] are both 3-nodes`},
{t:[[20,40],[[10]],[[30]],[[50,60,70]]],bad:[50,60,70],note:`leaf overflows`},
{t:[[20,40,60],[[10]],[[30]],[[50]],[[70]]],bad:[20,40,60],note:`split the leaf: 60 goes up. Now the parent overflows`},
{t:[[40],[[20],[[10]],[[30]]],[[60],[[50]],[[70]]]],ok:[40],note:`split the parent: 40 goes up and becomes the new root. Only now does the tree grow, and every leaf stays on the same level`}]" />

---

# Worked example: insert 10, 20, 30, 40, 50, 60, 70

Sorted input: the worst case for a plain BST. Step through.

<MwSteps :h="240" :ms="2800" :frames="[
{t:[[10]],note:`insert 10`},
{t:[[10,20]],note:`insert 20: leaf becomes a 3-node`},
{t:[[10,20,30]],bad:[10,20,30],note:`insert 30: overflow`},
{t:[[20],[[10]],[[30]]],ok:[20],note:`split: 20 up`},
{t:[[20],[[10]],[[30,40]]],note:`insert 40: room in leaf`},
{t:[[20],[[10]],[[30,40,50]]],bad:[30,40,50],note:`insert 50: overflow`},
{t:[[20,40],[[10]],[[30]],[[50]]],ok:[40],note:`split: 40 up, parent absorbs`},
{t:[[20,40],[[10]],[[30]],[[50,60]]],note:`insert 60: room`},
{t:[[40],[[20],[[10]],[[30]]],[[60],[[50]],[[70]]]],ok:[40,20,60],note:`insert 70: leaf splits, parent splits, new root. A perfectly balanced tree`}]" />

---

# Insertion summary (2-3)

```text
add(x):
   leaf = search down to the leaf for x
   put x into leaf
   while (node has 3 keys):
        split node into [small] and [large]; push [middle] up to the parent
        if node was the root: create a new root holding [middle]
        node = parent
```

**Bottom-up**: we go down to find the spot, then walk **back up** fixing overflows. Cost O(log n).

---

# Deletion: removing from an internal node

Keys in internal nodes are the problem, so **swap** the key with its **in-order successor** (always in a leaf), then delete there.

<MwSteps :frames="[
{t:[[20,40],[[10]],[[30]],[[50,60]]],hl:[40],note:`delete 40 (internal)`},
{t:[[20,40],[[10]],[[30]],[[50,60]]],hl:[50],note:`successor = smallest key in the subtree to its right: 50, in a leaf`},
{t:[[20,50],[[10]],[[30]],[[60]]],ok:[50],note:`50 replaces 40; the leaf [50,60] loses 50. It was a 3-node, so no further work`}]" />

---

# Deletion: the leaf becomes empty

When the leaf was a **2-node**, removing its key leaves it **empty** (underflow). Fix 1: **borrow** from a 3-node sibling, rotating through the parent.

<MwSteps :frames="[
{t:[[20],[[10]],[[30,40]]],hl:[10],note:`delete 10`},
{t:[[20],[[]],[[30,40]]],note:`the left leaf is empty. The sibling [30,40] has a spare key`},
{t:[[30],[[20]],[[40]]],ok:[30],note:`redistribute: parent key 20 comes DOWN, sibling key 30 goes UP. Done`}]" />

---

# Deletion: merge (the tree can shrink)

If the sibling is a 2-node it has nothing to spare. Fix 2: **merge**: pull the parent key down next to it.

<MwSteps :frames="[
{t:[[20],[[10]],[[30]]],hl:[10],note:`delete 10`},
{t:[[20],[[]],[[30]]],note:`left leaf is empty and the sibling [30] has only one key: cannot borrow`},
{t:[[20,30]],ok:[20,30],note:`merge: 20 comes down and joins 30. The root emptied, so the tree is one level shorter. If the parent underflowed instead, we would repeat the fix upward`}]" />

---

# 2-3 tree: height and cost

With n keys and height h:

| tree full of… | nodes at most | |
|---|---|---|
| only 2-nodes | 2ʰ − 1 | smallest n for height h |
| only 3-nodes | 3ʰ − 1 | largest n for height h |

So **log₃(n+1) ≤ h ≤ log₂(n+1)**.

**search / add / remove: O(log n)**, guaranteed, with no rotations and no height bookkeeping.

---

# The cost of 2-3 trees

- Insertion needs to handle a **temporary 3-key node** (a 4-node) on the way back up
- Splitting can **cascade** to the root
- Two node types, plus every borrow/merge case

*Could we make insertion cleaner by allowing 4-nodes to exist permanently?*

---
layout: section
---

# 2-4 Trees

---

# Node types in a 2-4 tree

Allow one more kind of node: the **4-node** (3 keys, 4 children).

<div class="grid grid-cols-3 gap-4 text-center mt-2 text-sm">
<div><MwTree :t="[[40],[[20]],[[60]]]" :w="180" /><div>2-node</div></div>
<div><MwTree :t="[[30,60],[[10]],[[45]],[[80]]]" :w="220" /><div>3-node</div></div>
<div><MwTree :t="[[30,60,90],[[10]],[[45]],[[75]],[[99]]]" :w="260" /><div>4-node</div></div>
</div>

**2-4 tree:** every internal node is a 2-, 3-, or 4-node, and all leaves are on the same level.

---

# A 2-4 tree

<div class="text-center"><MwTree :t="[[30,60],[[10,20]],[[40,50]],[[70,80,90]]]" :ok="[30,60]" :w="440" /></div>

Wider nodes mean a **shorter** tree: 8 keys here need only 2 levels. Height is between log₄(n+1) and log₂(n+1).

---

# Searching a 2-4 tree

Identical idea: find the gap between keys, follow that child. Search for 80.

<MwSteps :frames="[
{t:[[30,60],[[10,20]],[[40,50]],[[70,80,90]]],hl:[30,60],note:`80 greater than 60: rightmost child`},
{t:[[30,60],[[10,20]],[[40,50]],[[70,80,90]]],ok:[80],note:`found 80 inside a 4-node. At most 3 comparisons per node`}]" />

---

# Insertion: split on the way DOWN

New rule: **while searching, split every 4-node you meet** (middle key goes up to the parent). Then insert into a leaf that is never full.

Why? The parent was already visited, so it is **not** a 4-node; it always has room for the key we push up. Splits **never cascade**, and we never walk back up.

<MwSteps :frames="[
{t:[[10,20,30]],hl:[10,20,30],note:`insert 40: the root is a 4-node, so split it before going in`},
{t:[[20],[[10]],[[30]]],ok:[20],note:`split: 20 up as the new root`},
{t:[[20],[[10]],[[30,40]]],ok:[40],note:`now descend and insert 40 into the leaf`}]" />

---

# Top-down split in a deeper tree

Insert 60.

<MwSteps :frames="[
{t:[[20],[[10]],[[30,40,50]]],note:`start. Root [20] is fine. Moving to the child…`},
{t:[[20],[[10]],[[30,40,50]]],hl:[30,40,50],note:`…the child is a 4-node: split it now`},
{t:[[20,40],[[10]],[[30]],[[50]]],ok:[40],note:`40 moved up into the parent (room guaranteed)`},
{t:[[20,40],[[10]],[[30]],[[50,60]]],ok:[60],note:`60 > 40: go right and insert into [50]`}]" />

---

# 2-3 versus 2-4 insertion

| | 2-3 tree | 2-4 tree |
|---|---|---|
| direction | search down, **fix upward** | **fix while descending** |
| overflow | temporary 4-node | a 4-node is legal |
| cascading splits | yes, up to the root | **never** |
| code | recursion returning a promoted key | simple loop |
| cost | O(log n) | O(log n) |

---

# Worked example: insert 10 … 70 in a 2-4 tree

Same sorted input as the 2-3 example. Compare the shapes.

<MwSteps :h="220" :ms="2800" :frames="[
{t:[[10,20,30]],note:`10, 20, 30 fit in one 4-node`},
{t:[[20],[[10]],[[30,40]]],ok:[20],note:`insert 40: root is a 4-node, split first, then insert`},
{t:[[20],[[10]],[[30,40,50]]],note:`insert 50: fits`},
{t:[[20,40],[[10]],[[30]],[[50,60]]],ok:[40],note:`insert 60: leaf [30,40,50] was a 4-node, split on the way down`},
{t:[[20,40],[[10]],[[30]],[[50,60,70]]],note:`insert 70: fits. Done: still 2 levels, where the 2-3 tree needed 3`}]" />

---

# 2-4 deletion (idea)

- Same swap-with-successor step for internal keys
- **Top-down version:** while descending, make sure every node you enter has **at least 2 keys** (borrow from a sibling or merge). Then the deletion in the leaf never underflows
- **Bottom-up version:** same borrow / merge fixes as in 2-3 trees

<div class="mt-4 opacity-70">2-4 trees are also the conceptual model behind red-black trees (not covered here).</div>

---

# Summary

- Both trees stay balanced by **node size**, not by rotations: all leaves share one depth
- **2-3:** keys 1–2, children 2–3; insert **bottom-up**, split overflow, may cascade
- **2-4:** keys 1–3, children 2–4; insert **top-down**, split 4-nodes on the way
- Deletion: swap with successor, then **borrow** or **merge**
- All operations **O(log n)** worst case

| | AVL | 2-3 | 2-4 |
|---|---|---|---|
| balance by | rotations | splits | splits |
| nodes | 1 key | 1–2 keys | 1–3 keys |
| height | ≤ 1.44 log₂ n | ≤ log₂ n | ≤ log₂ n |

---
layout: section
---

# Supplement
Java implementation

---

# Supplement: 2-3 node and helpers

```java {all|1-6|8-9}
class Node23<T extends Comparable<? super T>> {
   T k1, k2;                         // k2 == null  →  2-node
   Node23<T> c1, c2, c3;             // children (all null in a leaf)
   Node23(T k, Node23<T> l, Node23<T> r) { k1 = k; c1 = l; c2 = r; }
   boolean isLeaf() { return c1 == null; }
   boolean is3()    { return k2 != null; }
}
class Split<T extends Comparable<? super T>> {  // result of an overflow
   T up;  Node23<T> left, right;
}
```

---

# Supplement: 2-3 add — descent

```java {all|2|4-8|9-10|12-16}
private Split<T> insert(Node23<T> n, T x) {
   if (n.isLeaf()) return absorb(n, x, null, null);

   Split<T> s;
   if (x.compareTo(n.k1) < 0)                   s = insert(n.c1, x);
   else if (!n.is3() || x.compareTo(n.k2) < 0)  s = insert(n.c2, x);
   else                                         s = insert(n.c3, x);

   if (s == null) return null;                  // child absorbed it: done
   return absorb(n, s.up, s.left, s.right);     // take the promoted key
}

public void add(T x) {
   if (root == null) { root = new Node23<>(x, null, null); return; }
   Split<T> s = insert(root, x);
   if (s != null) root = new Node23<>(s.up, s.left, s.right);   // root split
}
```

---

# Supplement: 2-3 add — absorb or split

```java {all|2-8|9-22}
private Split<T> absorb(Node23<T> n, T key, Node23<T> l, Node23<T> r) {
   if (!n.is3()) {                                   // 2-node → 3-node
      if (key.compareTo(n.k1) < 0) { n.k2 = n.k1; n.k1 = key; n.c3 = n.c2; n.c2 = r; n.c1 = l; }
      else                         { n.k2 = key; n.c2 = l; n.c3 = r; }
      return null;
   }
   T a, b, c;  Node23<T> w, x, y, z;                 // 3 keys, 4 children, sorted
   if (key.compareTo(n.k1) < 0)      { a = key;  b = n.k1; c = n.k2; w = l;    x = r;    y = n.c2; z = n.c3; }
   else if (key.compareTo(n.k2) < 0) { a = n.k1; b = key;  c = n.k2; w = n.c1; x = l;    y = r;    z = n.c3; }
   else                              { a = n.k1; b = n.k2; c = key;  w = n.c1; x = n.c2; y = l;    z = r;    }
   Split<T> s = new Split<>();
   s.up = b;                                         // middle key goes up
   s.left  = new Node23<>(a, w, x);
   s.right = new Node23<>(c, y, z);
   return s;
}
```

---

# Supplement: 2-4 node and split

```java {all|1-4|6-13}
class Node24<T extends Comparable<? super T>> {
   List<T> keys = new ArrayList<>();           // 1..3 keys, sorted
   List<Node24<T>> kids = new ArrayList<>();   // empty for a leaf, else keys.size()+1
   boolean full() { return keys.size() == 3; }
}
private void splitChild(Node24<T> p, int i) {  // child i of p is a 4-node
   Node24<T> c = p.kids.get(i), r = new Node24<>();
   r.keys.add(c.keys.remove(2));               // largest key → new right node
   if (!c.kids.isEmpty()) { r.kids.add(c.kids.remove(2)); r.kids.add(c.kids.remove(2)); }
   p.keys.add(i, c.keys.remove(1));            // middle key → parent
   p.kids.add(i + 1, r);
}
```

---

# Supplement: 2-4 add (top-down)

```java {all|2-5|7-14|15-16}
public void add(T x) {                           // root starts as an empty leaf
   if (root.full()) {                            // split a full root first
      Node24<T> r = new Node24<>();  r.kids.add(root);
      splitChild(r, 0);  root = r;
   }
   Node24<T> n = root;
   while (!n.kids.isEmpty()) {
      int i = 0;  while (i < n.keys.size() && x.compareTo(n.keys.get(i)) > 0) i++;
      if (n.kids.get(i).full()) {                // split BEFORE descending
         splitChild(n, i);
         if (x.compareTo(n.keys.get(i)) > 0) i++;
      }
      n = n.kids.get(i);
   }
   int j = 0;  while (j < n.keys.size() && x.compareTo(n.keys.get(j)) > 0) j++;
   n.keys.add(j, x);
}
```

---
layout: end
---

# End
