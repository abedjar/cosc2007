---
theme: default
title: 'AVL Trees'
class: text-left
fonts: { mono: 'Space Mono', sans: 'Space Mono' }
colorSchema: dark
transition: slide-left
mdc: true
---

# AVL Trees
A self-balancing binary search tree · after Carrano, *Data Structures and Abstractions with Java™* (5th ed.)

<div class="opacity-60 mt-8">COSC2006 · Data Structures I</div>

<style>
:root { --slidev-theme-primary: #60a5fa }
h1 { color: #60a5fa !important } h2 { color: #fb923c !important }
.ok { color: #4ade80 } .hl { color: #fb923c } .bl { color: #60a5fa } .bd { color: #f87171 }
</style>

---

# Roadmap

1. Recall: the binary search tree (BST)
2. The problem: a BST can degrade into a list
3. The idea: **keep the tree balanced**
4. What "AVL" means — height and balance factor
5. Insertion, then the **four imbalance cases**
6. Rotations (single, then double)
7. A full worked trace, deletion, and why height stays O(log n)
8. *Supplement:* the Java code

<div class="mt-4 text-sm">Diagrams: <span class="bl">■ normal</span> <span class="hl">■ in focus</span> <span class="ok">■ fixed</span> <span class="bd">■ unbalanced</span> · small number beside a node = its balance factor</div>

---

# Recall: binary search tree

For every node: **left subtree < node < right subtree.**
Searching = one comparison per level, going left or right.

<TreeSteps :bf="false" :frames="[
{t:[40,[20,[10],[30]],[60,[50],[70]]],hl:[40],note:`search 30: 30 is less than 40, go left`},
{t:[40,[20,[10],[30]],[60,[50],[70]]],hl:[20],note:`30 is greater than 20, go right`},
{t:[40,[20,[10],[30]],[60,[50],[70]]],ok:[30],note:`found 30 after 3 comparisons`}]" />

---

# Cost of search = height of the tree

Each step moves down one level, so **search, add and remove cost O(height)**.

- Full tree with n nodes: height ≈ log₂ n → **fast**
- 1,000,000 nodes → about 20 comparisons

So the question becomes: *is the height always about log₂ n?*

<div class="mt-6 opacity-70">Spoiler: not with a plain BST. Next slide →</div>

---

# The problem: order of insertion matters

Insert the **already sorted** keys 10, 20, 30, 40 into a plain BST:

<TreeSteps :bf="false" :h="260" :frames="[
{t:[10],note:`insert 10`},
{t:[10,null,[20]],note:`20 greater than 10: goes right`},
{t:[10,null,[20,null,[30]]],note:`30 goes right of 20`},
{t:[10,null,[20,null,[30,null,[40]]]],bad:[10,20,30,40],note:`A linked list in disguise. Height = n, so search is O(n)`}]" />

---

# The fix: same keys, better shape

The same four keys, shaped so the height is 3 instead of 4 — and with more keys the gap explodes.

<div class="text-center"><AvlTree :t="[20,[10],[30,null,[40]]]" :ok="[20]" :bf="false" :w="320" /></div>

- n = 1,000,000: **worst case ≈ 1,000,000** steps vs **≈ 20**
- Goal: *automatically* keep the tree close to balanced as keys arrive

---

# Height of a subtree

<v-clicks>

- Height of an **empty** tree = **0**
- Height of a **leaf** = **1**
- Height of any node = **1 + max(height of left, height of right)**

</v-clicks>

<div class="text-center mt-4"><AvlTree :t="[40,[20,[10],null],[60]]" :bf="false" :w="320" /></div>

<div class="text-sm opacity-70 text-center">height(10) = 1 · height(20) = 2 · height(60) = 1 · height(40) = 3</div>

---

# Balance factor

For a node: **bf = height(left) − height(right)**

<div class="text-center"><AvlTree :t="[40,[20,[10],null],[60]]" :w="320" /></div>

| node | left h | right h | bf |
|---|---|---|---|
| 10, 60 | 0 | 0 | 0 |
| 20 | 1 | 0 | +1 |
| 40 | 2 | 1 | +1 |

**+** = left-heavy · **−** = right-heavy · **0** = perfectly balanced

---

# Definition: AVL tree

An **AVL tree** is a BST in which **every node has balance factor −1, 0, or +1**.

- Named after its inventors: **A**delson-**V**elsky and **L**andis (1962)
- The condition applies to **every** node, not only the root
- Is the tree on the previous slide an AVL tree? Yes: all bf values are in {−1, 0, +1}

<div class="text-center mt-2"><AvlTree :t="[40,[20,[10],null],[60]]" :ok="[10,20,40,60]" :w="300" /></div>

---

# Not an AVL tree

Delete 60 from that tree: the root now has left height 2 and right height 0.

<div class="text-center"><AvlTree :t="[40,[20,[10]]]" :bad="[40]" :w="300" /></div>

- Still a valid **BST**, but bf(40) = **+2** → violates the AVL rule
- Our job: detect this and **repair** it without breaking the BST order

---

# Plan for insertion

<v-clicks>

1. Insert the key exactly like a normal BST (a new leaf)
2. Walk **back up** toward the root, recomputing heights
3. At the **first** node (lowest) with |bf| = 2, apply a **rotation**
4. After that single fix, the whole tree is balanced again — stop

</v-clicks>

<div class="mt-6 opacity-70">Only nodes on the path from the new leaf to the root can change balance, so at most O(log n) nodes are checked.</div>

---

# Four ways to become unbalanced

Let **z** be the lowest unbalanced node. Where did the new key go?

| bf(z) | heavy child | new key went into… | case | fix |
|---|---|---|---|---|
| +2 | left | left child's **left** | Left-Left | rotate **right** at z |
| −2 | right | right child's **right** | Right-Right | rotate **left** at z |
| +2 | left | left child's **right** | Left-Right | rotate **left** at child, then **right** at z |
| −2 | right | right child's **left** | Right-Left | rotate **right** at child, then **left** at z |

We take them one at a time, simplest first.

---

# Case 1 — Left-Left: the picture

Insert 30, 20, 10. A left-leaning "line" of three nodes.

<TreeSteps :frames="[
{t:[30,[20]],note:`insert 30, then 20: fine (bf +1 at 30)`},
{t:[30,[20,[10]]],bad:[30],note:`insert 10: bf(30) = +2. Left-Left case`},
{t:[30,[20,[10]]],hl:[30,20],note:`pivot = 30's left child (20). Lift it up…`},
{t:[20,[10],[30]],ok:[20,10,30],note:`rotate RIGHT at 30: 20 becomes the root, 30 its right child. All bf = 0`}]" />

---

# Right rotation: the general rule

If z has left child y, rotate right: **y goes up, z goes down to the right.**
The only subtree that changes parent is y's **right** subtree T2: it becomes z's **left** subtree.

<TreeSteps :bf="false" :h="200" :frames="[
{t:['z',['y',['T1'],['T2']],['T3']],hl:['z','y'],note:`before: T1 less than y less than T2 less than z less than T3`},
{t:['z',['y',['T1'],['T2']],['T3']],hl:['T2'],note:`T2 is the odd one out: it lies between y and z`},
{t:['y',['T1'],['z',['T2'],['T3']]],hl:['T2'],ok:['y'],note:`after: T2 is now z's left subtree. Same in-order sequence, so still a BST`}]" />

---

# Case 2 — Right-Right (mirror image)

Insert 10, 20, 30: a right-leaning line.

<TreeSteps :frames="[
{t:[10,null,[20]],note:`insert 10, then 20`},
{t:[10,null,[20,null,[30]]],bad:[10],note:`insert 30: bf(10) = −2. Right-Right case`},
{t:[10,null,[20,null,[30]]],hl:[10,20],note:`pivot = right child (20)`},
{t:[20,[10],[30]],ok:[10,20,30],note:`rotate LEFT at 10: exactly the mirror of a right rotation`}]" />

---

# Why one rotation is not always enough

Insert 30, 10, 20 — the new key is in the **inner** (zig-zag) position.

<TreeSteps :frames="[
{t:[30,[10]],note:`insert 30, then 10`},
{t:[30,[10,null,[20]]],bad:[30],note:`insert 20: bf(30) = +2, but the left child 10 leans RIGHT (−1). Left-Right case`},
{t:[10,null,[30,[20]]],bad:[10],note:`Try a plain right rotation at 30… the imbalance just moves to the other side (bf(10) = −2)`}]" />

---

# Case 3 — Left-Right: double rotation

Fix the zig-zag first by turning it into a straight line.

<TreeSteps :frames="[
{t:[30,[10,null,[20]]],bad:[30],note:`start: 30 is +2, its left child 10 is −1`},
{t:[30,[10,null,[20]]],hl:[10,20],note:`step 1: rotate LEFT at the child (10)`},
{t:[30,[20,[10]]],hl:[30,20],note:`now a Left-Left line, case 1`},
{t:[20,[10],[30]],ok:[10,20,30],note:`step 2: rotate RIGHT at 30. Balanced`}]" />

---

# Case 4 — Right-Left: double rotation (mirror)

Insert 10, 30, 20.

<TreeSteps :frames="[
{t:[10,null,[30,[20]]],bad:[10],note:`10 is −2, its right child 30 leans left (+1)`},
{t:[10,null,[30,[20]]],hl:[30,20],note:`step 1: rotate RIGHT at the child (30)`},
{t:[10,null,[20,null,[30]]],hl:[10,20],note:`now a Right-Right line, case 2`},
{t:[20,[10],[30]],ok:[10,20,30],note:`step 2: rotate LEFT at 10. Balanced`}]" />

---

# Which rotation? Decision rule

Look at **z** (bf = ±2) and its **heavy child c**:

| bf(z) | bf(c) | action |
|---|---|---|
| +2 | +1 (or 0) | right rotation at z |
| +2 | −1 | left at c, then right at z |
| −2 | −1 (or 0) | left rotation at z |
| −2 | +1 | right at c, then left at z |

*Rule of thumb:* if the heavy child leans **the same way** as z → single rotation; if it leans the **opposite** way → double rotation.
(bf(c) = 0 only happens during deletion.)

---

# Worked example: insert 10, 20, 30, 40, 50, 25

Step through — each "fix" is one of the four cases.

<TreeSteps :h="300" :ms="2800" :frames="[
{t:[10],note:`insert 10`},
{t:[10,null,[20]],note:`insert 20`},
{t:[10,null,[20,null,[30]]],bad:[10],note:`insert 30: Right-Right at 10`},
{t:[20,[10],[30]],ok:[20],note:`left rotation at 10`},
{t:[20,[10],[30,null,[40]]],hl:[40],note:`insert 40: all bf within range`},
{t:[20,[10],[30,null,[40,null,[50]]]],bad:[30],note:`insert 50: lowest unbalanced node is 30 (Right-Right). Fixing it also repairs 20`},
{t:[20,[10],[40,[30],[50]]],ok:[40],note:`left rotation at 30`},
{t:[20,[10],[40,[30,[25]],[50]]],bad:[20],note:`insert 25: 20 is −2, right child 40 leans left. Right-Left case`},
{t:[20,[10],[30,[25],[40,null,[50]]]],hl:[20,30],note:`step 1: rotate right at 40`},
{t:[30,[20,[10],[25]],[40,null,[50]]],ok:[30],note:`step 2: rotate left at 20. Balanced`}]" />

---

# Deletion (idea only)

1. Delete like a normal BST (leaf, one child, or in-order successor)
2. Walk back up, recomputing heights; fix **any** node with |bf| = 2
3. Unlike insertion, fixing one node may unbalance an ancestor → up to O(log n) rotations

<TreeSteps :frames="[
{t:[20,[10],[40,[30],[50]]],hl:[10],note:`delete 10`},
{t:[20,null,[40,[30],[50]]],bad:[20],note:`bf(20) = −2, but child 40 is balanced (bf 0): a single left rotation suffices`},
{t:[40,[20,null,[30]],[50]],ok:[40],note:`left rotation at 20: balanced`}]" />

---

# Why the height stays O(log n)

Smallest AVL tree of height h: N(h) = N(h−1) + N(h−2) + 1 — one subtree one level shorter, the other **two** levels shorter.

<TreeSteps :bf="false" :h="260" :frames="[
{t:[1],note:`h = 1: 1 node`},
{t:[2,[1]],note:`h = 2: 2 nodes`},
{t:[3,[2,[1]],[4]],note:`h = 3: 4 nodes = 2 + 1 + 1`},
{t:[5,[3,[2,[1]],[4]],[7,[6]]],note:`h = 4: 7 nodes = 4 + 2 + 1`}]" />

---

# Height bound and cost

| h | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| min nodes N(h) | 1 | 2 | 4 | 7 | 12 | 20 |

N(h) grows exponentially (Fibonacci-like), so **h ≤ ≈ 1.44 · log₂(n + 2)**.

| operation | plain BST (worst) | AVL (worst) |
|---|---|---|
| search | O(n) | **O(log n)** |
| add | O(n) | **O(log n)** |
| remove | O(n) | **O(log n)** |

Rotations are O(1) each, so balancing never changes the big-O.

---

# Summary

- AVL = BST + **every node has |bf| ≤ 1**
- Insert like a BST, walk up, fix the **lowest** unbalanced node
- Four cases → **two single** + **two double** rotations
- Rotations keep the in-order order, so the BST property survives
- Result: guaranteed **O(log n)** search, add, remove

<div class="mt-4 opacity-70">Next: the Java code (supplement).</div>

---
layout: section
---

# Supplement
Java implementation

---

# Supplement: node and helpers

```java {all|1-5|7-9|10}
class AvlNode<T extends Comparable<? super T>> {
   T data;  AvlNode<T> left, right;
   int height = 1;                        // a new leaf has height 1
   AvlNode(T d) { data = d; }
}

private int height(AvlNode<T> n)  { return n == null ? 0 : n.height; }
private void update(AvlNode<T> n) { n.height = 1 + Math.max(height(n.left), height(n.right)); }
private int balance(AvlNode<T> n) { return height(n.left) - height(n.right); }
```

---

# Supplement: rotations

```java {all|1-6|8-13}
private AvlNode<T> rotateRight(AvlNode<T> z) {
   AvlNode<T> y = z.left;
   z.left = y.right;        // T2 moves across
   y.right = z;             // z goes down
   update(z); update(y);    // lower node first
   return y;                // new subtree root
}

private AvlNode<T> rotateLeft(AvlNode<T> z) {
   AvlNode<T> y = z.right;
   z.right = y.left;
   y.left = z;
   update(z); update(y);
   return y;
}
```

---

# Supplement: rebalance and add

```java {all|1-12|14-22}
private AvlNode<T> rebalance(AvlNode<T> n) {
   update(n);
   int b = balance(n);
   if (b > 1) {                                   // left-heavy
      if (balance(n.left) < 0) n.left = rotateLeft(n.left);   // Left-Right
      return rotateRight(n);
   }
   if (b < -1) {                                  // right-heavy
      if (balance(n.right) > 0) n.right = rotateRight(n.right); // Right-Left
      return rotateLeft(n);
   }
   return n;
}

private AvlNode<T> add(AvlNode<T> n, T x) {
   if (n == null) return new AvlNode<>(x);
   int c = x.compareTo(n.data);
   if (c < 0)      n.left  = add(n.left, x);
   else if (c > 0) n.right = add(n.right, x);
   else return n;                                // duplicate: ignore
   return rebalance(n);                          // runs on the way back up
}
public void add(T x) { root = add(root, x); }
```

---

# Supplement: remove

```java {all|4-8|10}
private AvlNode<T> remove(AvlNode<T> n, T x) {
   if (n == null) return null;
   int c = x.compareTo(n.data);
   if (c < 0)      n.left  = remove(n.left, x);
   else if (c > 0) n.right = remove(n.right, x);
   else {
      if (n.left == null)  return n.right;
      if (n.right == null) return n.left;
      AvlNode<T> s = n.right;  while (s.left != null) s = s.left;   // in-order successor
      n.data = s.data;  n.right = remove(n.right, s.data);
   }
   return rebalance(n);
}
```

---
layout: end
---

# End
