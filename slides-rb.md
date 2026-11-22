---
theme: default
title: 'Red-Black Trees'
class: text-left
fonts: { mono: 'Space Mono', sans: 'Space Mono' }
colorSchema: dark
transition: slide-left
mdc: true
---

# Red-Black Trees
A balanced BST that is really a **2-4 tree in disguise** · after Carrano, *Data Structures and Abstractions with Java™* (5th ed.)

<div class="opacity-60 mt-8">COSC2006 · Data Structures I</div>

<style>
:root { --slidev-theme-primary: #60a5fa }
h1 { color: #60a5fa !important } h2 { color: #fb923c !important }
.rd { color: #f87171; font-weight: bold } .bk { color: #e2e8f0; background: #020617; padding: 0 6px; border: 1px solid #cbd5e1; border-radius: 4px; font-weight: bold }
.hl { color: #fbbf24 } .ok { color: #4ade80 } .bd { color: #e879f9 }
</style>

---

# Roadmap

1. Recall 2-4 trees and their one drawback
2. The idea: **encode multi-key nodes with colored binary nodes**
3. The five red-black rules
4. Why the height stays O(log n)
5. Insertion: color red, then fix with **recolor** or **rotate**
6. Worked trace, deletion (idea), comparison
7. *Supplement:* Java code

<div class="mt-4 text-sm">Diagrams: <span class="rd">● red</span> / <span class="bk">● black</span> nodes · <span class="hl">◯ yellow ring = in focus</span> · <span class="bd">◯ dashed ring = rule violated</span> · <span class="ok">◯ green ring = just fixed</span></div>

---

# Recall: 2-4 trees

- Balanced by **node size**: every internal node has 2–4 children, all leaves at one depth
- Guaranteed O(log n) search, add, remove
- Drawback: nodes are **variable size** (1, 2 or 3 keys); the code must shuffle keys and children inside nodes

<v-clicks>

- Could we get the same shape and the same guarantees using **ordinary binary nodes**?
- Yes: use a **color bit** to say which binary nodes belong together

</v-clicks>

---

# Encoding a 2-node

A 2-node is just one **black** node.

<div class="grid grid-cols-2 gap-6 text-center mt-2">
<div><MwTree :t="[[20],[[10]],[[30]]]" :w="220" /><div class="text-sm mt-2">2-4 tree: 2-node</div></div>
<div><RbTree :t="[20,'B',[10,'B'],[30,'B']]" :w="220" /><div class="text-sm mt-2">red-black: one black node</div></div>
</div>

---

# Encoding a 3-node

A 3-node becomes a black node plus **one red child**. The red node is "glued" to its black parent. Either orientation works.

<div class="grid grid-cols-2 gap-6 text-center mt-2">
<div><MwTree :t="[[20,40],[[10]],[[30]],[[50]]]" :w="260" /><div class="text-sm mt-2">2-4 tree: 3-node [20, 40]</div></div>
<div><RbSteps :h="190" :w="260" :frames="[
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],hl:[20,40],note:`red 40 leans right`},
{t:[40,'B',[20,'R',[10,'B'],[30,'B']],[50,'B']],hl:[20,40],note:`or red 20 leans left (mirror)`}]" /></div>
</div>

---

# Encoding a 4-node

A 4-node becomes a black node with **two red children**, one on each side.

<div class="grid grid-cols-2 gap-6 text-center mt-2">
<div><MwTree :t="[[20,40,60],[[10]],[[30]],[[50]],[[70]]]" :w="280" /><div class="text-sm mt-2">2-4 tree: 4-node [20, 40, 60]</div></div>
<div><RbTree :t="[40,'B',[20,'R',[10,'B'],[30,'B']],[60,'R',[50,'B'],[70,'B']]]" :w="280" /><div class="text-sm mt-2">red-black: black 40, red 20 and 60</div></div>
</div>

Summary: **black node = start of a 2-4 node · red child = extra key in the same node**

---

# The five rules (read off the encoding)

| # | rule | why (2-4 view) |
|---|---|---|
| 1 | every node is <span class="rd">red</span> or <span class="bk">black</span> | |
| 2 | the root is black | root is the top of a 2-4 node |
| 3 | every NIL (empty child) counts as black | |
| 4 | a red node has **no red child** | at most 3 keys per node |
| 5 | every path from a node to its NIL leaves has the **same number of black nodes** | all leaves of a 2-4 tree share one depth |

Next: rules 4 and 5, one at a time.

---

# Rule 4: no red-red

Red nodes can never touch each other vertically.

<RbSteps :frames="[
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],ok:[40],note:`valid: red 40 has black children`},
{t:[20,'B',[10,'B'],[40,'R',[30,'R'],[50,'B']]],bad:[40,30],note:`violation: red 40 with red child 30. In 2-4 terms that would be a node with 4 keys`}]" />

---

# Rule 5: equal black count (black-height)

Count the **black** nodes on every path from the root down to a NIL. They must all agree.

<RbSteps :nil="true" :h="250" :w="440" :frames="[
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],hl:[20,10],note:`path 20 → 10 → NIL: 2 black nodes`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],hl:[20,40,30],note:`path 20 → 40 (red, not counted) → 30 → NIL: 2 black`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],hl:[20,40,50],note:`path 20 → 40 → 50 → NIL: 2 black. Every path agrees, so rule 5 holds`}]" />

---

# Reading a red-black tree as a 2-4 tree

Glue every red node to its black parent:

<div class="grid grid-cols-2 gap-6 text-center">
<div><RbTree :t="[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]]" :hl="[20,40]" :w="240" /></div>
<div><MwTree :t="[[20,40],[[10]],[[30]],[[50]]]" :hl="[20,40]" :w="240" /></div>
</div>

<div class="text-center text-sm mt-2">Rule 5 is exactly "all leaves of the 2-4 tree are on one level".</div>

---

# Why the height is O(log n)

Two facts:

<v-clicks>

- **Rule 5:** the shortest root-to-NIL path is all black nodes, say *b* of them
- **Rule 4:** red nodes can't be adjacent, so a path can hold at most *b* red ones: longest ≤ 2 × shortest

</v-clicks>

Hence **height ≤ 2 · log₂(n + 1)**. The tree may be uneven, but never badly.

---

# Shortest versus longest path

<RbSteps :h="250" :w="440" :frames="[
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B',null,[60,'R']]]],hl:[20,10],note:`shortest path: 20 → 10: 2 nodes, 2 black`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B',null,[60,'R']]]],hl:[20,40,50,60],note:`longest path: 20 → 40 → 50 → 60: 4 nodes. It alternates black / red / black / red, and is exactly 2× the shortest`}]" />

---

# Searching

Ignore the colors: it is an ordinary BST search. Search for 30.

<RbSteps :frames="[
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],hl:[20],note:`30 greater than 20: go right`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],hl:[40],note:`30 less than 40: go left`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B']]],ok:[30],note:`found. At most 2·log n comparisons`}]" />

---

# Insertion: the strategy

1. Insert like a BST (new leaf)
2. Color the new node **red**
3. Fix any rule that broke
4. Make sure the root ends up black

**Why red?** Adding a black node would change the black count of one path, breaking rule 5 everywhere. A red node breaks **at most rule 4** (red parent), which is much easier to repair.

---

# Which rule can break?

New node **z** is red. Look at its parent.

| parent | result |
|---|---|
| <span class="bk">black</span> | nothing broken. Done |
| <span class="rd">red</span> | rule 4 broken (red-red). Then the grandparent is black; look at the **uncle** (the parent's sibling) |

| uncle | z is… | fix |
|---|---|---|
| <span class="rd">red</span> | any | **recolor**, continue upward |
| <span class="bk">black</span> (or NIL) | inner (zig-zag) | **rotate** to make it outer, then below |
| <span class="bk">black</span> (or NIL) | outer (straight line) | **rotate** grandparent + swap colors |

---

# Case 0: black parent

Insert 25.

<RbSteps :frames="[
{t:[20,'B',[10,'B'],[30,'B']],note:`start`},
{t:[20,'B',[10,'B'],[30,'B',[25,'R']]],hl:[25],note:`25 goes under black 30, colored red. No rule is violated`}]" />

---

# Case 1: red uncle, recolor

Insert 5.

<RbSteps :h="250" :ms="2800" :frames="[
{t:[20,'B',[10,'R'],[30,'R']],note:`start: 10 and 30 are red`},
{t:[20,'B',[10,'R',[5,'R']],[30,'R']],bad:[5,10],note:`insert 5 as red: red-red with its red parent 10`},
{t:[20,'B',[10,'R',[5,'R']],[30,'R']],hl:[10,30,20],note:`parent 10 and uncle 30 are both red; grandparent 20 is black`},
{t:[20,'R',[10,'B',[5,'R']],[30,'B']],ok:[10,30],hl:[20],note:`recolor: parent and uncle become black, grandparent becomes red`},
{t:[20,'B',[10,'B',[5,'R']],[30,'B']],ok:[20],note:`20 is the root: rule 2 turns it black. Black count of every path rose by 1 at once`}]" />

---

# Case 1 in 2-4 terms

Recoloring = **splitting a 4-node**: black parent 20 with two red children is a 4-node [10, 20, 30]. After the recolor the middle key moves up (20 turns red and joins its parent) and 10, 30 become two separate 2-nodes.

If the grandparent's own parent is red, we repeat the same checks one level higher, exactly like a cascading split.

---

# Case 3: black uncle, outer node

Insert 10. Uncle is NIL, which counts as black.

<RbSteps :h="250" :ms="2800" :frames="[
{t:[30,'B',[20,'R']],note:`start`},
{t:[30,'B',[20,'R',[10,'R']]],bad:[20,10],note:`insert 10: red-red, and 10 is on the outer (left-left) side`},
{t:[30,'B',[20,'R',[10,'R']]],hl:[30],note:`uncle (30's right) is NIL = black: recoloring cannot help, we must rotate`},
{t:[20,'R',[10,'R'],[30,'B']],hl:[20,30],note:`step 1: rotate right at the grandparent 30 (20 moves up)`},
{t:[20,'B',[10,'R'],[30,'R']],ok:[20],note:`step 2: swap colors of 20 and 30. Valid, and no change above`}]" />

<div class="text-sm opacity-70">The mirror image (right-right) uses a left rotation.</div>

---

# Case 2: black uncle, inner node

Insert 20. Now the new node is in the **zig-zag** position.

<RbSteps :h="250" :ms="2800" :frames="[
{t:[30,'B',[10,'R']],note:`start`},
{t:[30,'B',[10,'R',null,[20,'R']]],bad:[10,20],note:`insert 20: red-red, 20 is the INNER child of 10`},
{t:[30,'B',[20,'R',[10,'R']]],hl:[10,20],note:`step 1: rotate left at the parent 10. Now the red pair is a straight line`},
{t:[20,'B',[10,'R'],[30,'R']],ok:[20],note:`step 2: it is case 3: rotate right at 30 and swap colors`}]" />

---

# Cases at a glance

| uncle | shape | action | in 2-4 terms |
|---|---|---|---|
| <span class="rd">red</span> | any | recolor 3 nodes, move up | split a 4-node |
| <span class="bk">black</span> | inner | rotate parent, then case 3 | choose the other 3-node orientation |
| <span class="bk">black</span> | outer | rotate grandparent, swap colors | re-orient the 3-node |

**Rotations per insertion: at most 2.** Recoloring may climb up to the root: O(log n).

---

# Worked example: insert 10, 20, 30, 40, 50, 60, 70

The same sorted input again. Step through.

<RbSteps :h="260" :w="440" :ms="2900" :frames="[
{t:[10,'R'],hl:[10],note:`insert 10: new nodes are red`},
{t:[10,'B'],ok:[10],note:`rule 2: the root becomes black`},
{t:[10,'B',null,[20,'R']],note:`insert 20: red under black. Fine`},
{t:[10,'B',null,[20,'R',null,[30,'R']]],bad:[20,30],note:`insert 30: red-red, uncle is NIL, outer: needs a rotation`},
{t:[20,'B',[10,'R'],[30,'R']],ok:[20],note:`rotate left at 10, swap colors`},
{t:[20,'B',[10,'R'],[30,'R',null,[40,'R']]],bad:[30,40],note:`insert 40: parent 30 and uncle 10 are both red`},
{t:[20,'B',[10,'B'],[30,'B',null,[40,'R']]],ok:[10,30],note:`recolor; the root is made black again`},
{t:[20,'B',[10,'B'],[30,'B',null,[40,'R',null,[50,'R']]]],bad:[40,50],note:`insert 50: uncle NIL, outer`},
{t:[20,'B',[10,'B'],[40,'B',[30,'R'],[50,'R']]],ok:[40],note:`rotate left at 30, swap colors`},
{t:[20,'B',[10,'B'],[40,'B',[30,'R'],[50,'R',null,[60,'R']]]],bad:[50,60],note:`insert 60: parent 50 and uncle 30 both red`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B',null,[60,'R']]]],ok:[30,50],note:`recolor: 40 becomes red, its parent 20 is black: stop`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[50,'B',null,[60,'R',null,[70,'R']]]]],bad:[60,70],note:`insert 70: uncle NIL, outer`},
{t:[20,'B',[10,'B'],[40,'R',[30,'B'],[60,'B',[50,'R'],[70,'R']]]],ok:[60],note:`rotate left at 50, swap colors. Height 4 for 7 keys (a plain BST would be 7)`}]" />

---

# Deletion (idea)

1. Delete like a BST (swap with the in-order successor if the node has two children)
2. **Red node removed** → nothing to fix
3. **Black node with a red child** → the child takes its place, colored black
4. **Black leaf removed** → that path is now short by one black: a "double black" problem

Case 4 is the same as an **underflow in a 2-4 tree**: borrow or merge.

---

# Fixing the double-black

Look at the sibling **s** of the short side:

| sibling | its children | fix | 2-4 view |
|---|---|---|---|
| <span class="rd">red</span> | (both black) | rotate at the parent so the sibling becomes black, then continue | change orientation |
| <span class="bk">black</span> | has a red child | **rotate**, recolor. Done | borrow a key |
| <span class="bk">black</span> | both black | recolor s red, **push the problem up** | merge nodes |

---

# Example: sibling with a red child

Delete 10 (a black leaf).

<RbSteps :h="250" :ms="2800" :frames="[
{t:[20,'B',[10,'B'],[30,'B',null,[40,'R']]],hl:[10],note:`delete the black leaf 10`},
{t:[20,'B',null,[30,'B',null,[40,'R']]],bad:[20],note:`the left side is now one black short. The sibling 30 is black with a red child 40`},
{t:[30,'B',[20,'B'],[40,'B']],ok:[30],note:`rotate left at 20; 40 becomes black. All black counts agree again`}]" />

---

# Cost and comparison

- Height ≤ 2 log₂(n+1) → **search, add, remove: O(log n)** worst case
- At most **2 rotations** per insert, **3** per delete
- One extra bit (the color) per node, no heights stored

| | AVL | 2-3 / 2-4 | Red-black |
|---|---|---|---|
| balance by | rotations | splits | rotations + recolor |
| per node | height | 1–3 keys | color bit |
| height | ≤ 1.44 log₂ n | ≤ log₂ n | ≤ 2 log₂ n |
| rotations on insert | ≤ 2 | none | ≤ 2 |
| lookup | slightly faster | | slightly slower |

AVL is more tightly balanced; red-black does less work on updates.

---

# Summary

- A red-black tree is a **BST encoding of a 2-4 tree**: black node = a 2-4 node, red child = an extra key in it
- Five rules: colors, black root, black NIL, **no red-red**, **equal black count**
- Rules 4 + 5 give **longest path ≤ 2 × shortest** → O(log n)
- Insert red, then: **black parent** done · **red uncle** recolor · **black uncle** rotate (once or twice)
- Delete: red is easy; double-black = borrow (rotate) or merge (recolor, move up)

---
layout: section
---

# Supplement
Java implementation

---

# Supplement: node and rotation

```java {all|1-4|6-15}
class RbNode<T extends Comparable<? super T>> {
   T data;  boolean red = true;                 // new nodes are red
   RbNode<T> left, right, parent;
}
private void rotateLeft(RbNode<T> x) {
   RbNode<T> y = x.right;
   x.right = y.left;  if (y.left != null) y.left.parent = x;
   y.parent = x.parent;                         // reattach y where x was
   if (x.parent == null)         root = y;
   else if (x == x.parent.left)  x.parent.left = y;
   else                          x.parent.right = y;
   y.left = x;  x.parent = y;
}                                               // rotateRight is the mirror image
```

---

# Supplement: add

```java {all|2-7|8-9|11}
public void add(T x) {
   RbNode<T> p = null, cur = root;
   while (cur != null) {                        // ordinary BST descent
      p = cur;  int c = x.compareTo(cur.data);
      if (c == 0) return;                       // ignore duplicates
      cur = (c < 0) ? cur.left : cur.right;
   }
   RbNode<T> n = new RbNode<>(); n.data = x; n.parent = p;
   if (p == null) root = n; else if (x.compareTo(p.data) < 0) p.left = n; else p.right = n;
   fixAfterInsert(n);
}
private boolean isRed(RbNode<T> n) { return n != null && n.red; }   // NIL is black
```

---

# Supplement: fix after insert

```java {all|2|3-5|6-8|9-15|17}
private void fixAfterInsert(RbNode<T> z) {
   while (z != root && isRed(z.parent)) {            // red-red violation
      RbNode<T> p = z.parent, g = p.parent;
      boolean parentIsLeft = (p == g.left);
      RbNode<T> uncle = parentIsLeft ? g.right : g.left;
      if (isRed(uncle)) {                            // case 1: recolor
         p.red = false;  uncle.red = false;  g.red = true;  z = g;
      } else {
         if (parentIsLeft && z == p.right)      { z = p; rotateLeft(z); }    // case 2
         else if (!parentIsLeft && z == p.left) { z = p; rotateRight(z); }
         p = z.parent;  g = p.parent;
         p.red = false;  g.red = true;               // case 3: swap colors
         if (parentIsLeft) rotateRight(g); else rotateLeft(g);
      }
   }
   root.red = false;                                 // rule 2
}
```

---
layout: end
---

# End
