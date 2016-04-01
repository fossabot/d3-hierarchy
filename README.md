# d3-hierarchy

…

## Installing

If you use NPM, `npm install d3-hierarchy`. Otherwise, download the [latest release](https://github.com/d3/d3-hierarchy/releases/latest). You can also load directly from [d3js.org](https://d3js.org), either as a [standalone library](https://d3js.org/d3-hierarchy.v0.1.min.js) or as part of [D3 4.0 alpha](https://github.com/mbostock/d3/tree/4). AMD, CommonJS, and vanilla environments are supported. In vanilla, a `d3_hierarchy` global is exported:

```html
<script src="https://d3js.org/d3-hierarchy.v0.1.min.js"></script>
<script>

var treemap = d3_hierarchy.treemap();

</script>
```

[Try d3-hierarchy in your browser.](https://tonicdev.com/npm/d3-hierarchy)

## API Reference

* [Hierarchy](#hierarchy) ([Hierarchy Node](#hierarchy-node), [Stratify](#stratify))
* [Cluster](#cluster)
* [Tree](#tree)
* [Treemap](#treemap) ([Treemap Tiling](#treemap-tiling))
* [Partition](#partition)
* [Pack](#pack)

### Hierarchy

Before you can compute a hierarchical layout, you need a [root node](#hierarchy-node). If your data is already in a hierarchical format, such as JSON, you can pass it directly to [d3.hierarchy](#hierarchy); otherwise, you can rearrange tabular data, such as comma-separated values (CSV), into a hierarchy using [d3.stratify](#stratify).

<a name="hierarchy" href="#hierarchy">#</a> d3.<b>hierarchy</b>(<i>data</i>[, <i>children</i>])

Constructs a root node from the specified hierarchical *data*. The specified *data* must be an object representing the root node. For example:

```json
{
  "name": "Eve",
  "children": [
    {
      "name": "Cain"
    },
    {
      "name": "Seth",
      "children": [
        {
          "name": "Enos"
        },
        {
          "name": "Noam"
        }
      ]
    },
    {
      "name": "Abel"
    },
    {
      "name": "Awan",
      "children": [
        {
          "name": "Enoch"
        }
      ]
    },
    {
      "name": "Azura"
    }
  ]
}
```

The specified *children* accessor function is invoked for each datum, starting with the root *data*, and must return an array of data representing the children, or null if the current datum has no children. If *children* is not specified, it defaults to:

```js
function children(d) {
  return d.children;
}
```

The [*node*.data](#node_data) of each node in the returned root is a reference to the corresponding value in the input data. (The data is not copied.) See also [Stratify](#stratify) for how to convert tabular data into a hierarchy.

#### Hierarchy Node

<a name="node_value" href="#node_value">#</a> <i>node</i>.<b>value</b>

The combined value of this node and all its descendants; computed by [*node*.sum](#node_sum).

<a name="node_data" href="#node_data">#</a> <i>node</i>.<b>data</b>

A reference to the data associated with this node, as specified to the [constructor](#hierarchy).

<a name="node_depth" href="#node_depth">#</a> <i>node</i>.<b>depth</b>

The depth of the node: zero for the root node, and increasing by one for each descendant generation.

<a name="node_height" href="#node_height">#</a> <i>node</i>.<b>height</b>

The height of the node: zero for leaf nodes, and the greatest distance from any descendant leaf for internal nodes.

<a name="node_parent" href="#node_parent">#</a> <i>node</i>.<b>parent</b>

A reference to the parent node; null for the root node.

<a name="node_children" href="#node_children">#</a> <i>node</i>.<b>children</b>

An array of child nodes, if any; undefined for leaf nodes.

<a name="node_ancestors" href="#node_ancestors">#</a> <i>node</i>.<b>ancestors</b>()

Returns the array of ancestors nodes, starting with this node, then followed by each parent up to the root.

<a name="node_descendants" href="#node_descendants">#</a> <i>node</i>.<b>descendants</b>()

Returns the array of descendant nodes, starting with this node, then followed by each child in topological order.

<a name="node_leaves" href="#node_leaves">#</a> <i>node</i>.<b>leaves</b>()

Returns the array of leaf nodes in traversal order; leaves are nodes with no children.

<a name="node_path" href="#node_path">#</a> <i>node</i>.<b>path</b>(<i>target</i>)

Returns the shortest path through the hierarchy from this *node* to the specified *target* node. The path starts at this *node*, ascends to the least common ancestor of this *node* and the *target* node, and then descends to the *target* node. This is particularly useful for [hierarchical edge bundling](https://bl.ocks.org/mbostock/7607999).

<a name="node_sum" href="#node_sum">#</a> <i>node</i>.<b>sum</b>(<i>value</i>)

Evaluates the specified *value* function for this *node* and each descendant in [post-order traversal](#node_eachAfter), and returns this *node*. The [value](#node_value) of each node is set to the numeric value returned by the specified function plus the combined value of all descendants. The function is passed the node’s [data](#node_data), and must return a non-negative number.

You must call *node*.sum before invoking a hierarchical layout that requires [*node*.value](#node_value), such as [d3.treemap](#treemap). Since the API supports [method chaining](https://en.wikipedia.org/wiki/Method_chaining), you can invoke *node*.sum and [*node*.sort](#node_sort) before computing the layout, and then subsequently generate an array of all [descendant nodes](#node_descendants) like so:

```js
var treemap = d3.treemap()
    .size([width, height])
    .padding(2);

var nodes = treemap(root
    .sum(function(d) { return d.value; })
    .sort(function(a, b) { return b.height - a.height || b.value - a.value; }))
  .descendants();
```

This example assumes that the [node data](#node_data) has a value field.

<a name="node_sort" href="#node_sort">#</a> <i>node</i>.<b>sort</b>(<i>compare</i>)

Sorts the children of this *node*, if any, and each of this *node*’s descendants’ children, in [pre-order traversal](#node_eachBefore) using the specified *compare* function, and returns this *node*. The specified function is passed two nodes *a* and *b* to compare. If *a* should be before *b*, the function must return a value less than zero; if *b* should be before *a*, the function must return a value greater than zero; otherwise, the relative order of *a* and *b* are not specified. See [*array*.sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) for more.

Unlike [*node*.sum](#node_sum), the *compare* function is passed two [nodes](#hierarchy-node) rather than two nodes’ [data](#node_data). For example, if the data has a value property, this sorts nodes by the descending aggregate [value](#node_value) of the node and all its descendants:

```js
root
    .sum(function(d) { return d.value; })
    .sort(function(a, b) { return b.value - a.value; });
``````

Similarly, to sort nodes by descending [height](#node_height) (greatest distance from any descendant leaf) and then descending value, as is recommended for [treemaps](#treemap):

```js
root
    .sum(function(d) { return d.value; })
    .sort(function(a, b) { return b.height - a.height || b.value - a.value; });
```

To sort nodes by descending height and then ascending id, as is recommended for [trees](#tree) and [dendrograms](#cluster):

```js
root
    .sum(function(d) { return d.value; })
    .sort(function(a, b) { return b.height - a.height || a.id.localeCompare(b.id); });
```

You must call *node*.sort before invoking a hierarchical layout if you want the new sort order to affect the layout; see [*node*.sum](#node_sum) for an example.

<a name="node_each" href="#node_each">#</a> <i>node</i>.<b>each</b>(<i>function</i>)

Invokes the specified *function* for *node* and each descendent in [breadth-first order](https://en.wikipedia.org/wiki/Breadth-first_search), such that a given *node* is only visited if all nodes of lesser [depth](#node_depth) have already been visited, as well as all preceeding nodes of the same depth. The specified function is passed the current *node*.

<a name="node_eachAfter" href="#node_eachAfter">#</a> <i>node</i>.<b>eachAfter</b>(<i>function</i>)

Invokes the specified *function* for *node* and each descendent in [post-order traversal](https://en.wikipedia.org/wiki/Tree_traversal#Post-order), such that a given *node* is only visited after all of its descendants have already been visited. The specified function is passed the current *node*.

<a name="node_eachBefore" href="#node_eachBefore">#</a> <i>node</i>.<b>eachBefore</b>(<i>function</i>)

Invokes the specified *function* for *node* and each descendent in [pre-order traversal](https://en.wikipedia.org/wiki/Tree_traversal#Pre-order), such that a given *node* is only visited after all of its ancestors have already been visited. The specified function is passed the current *node*.

<a name="node_copy" href="#node_copy">#</a> <i>node</i>.<b>copy</b>()

Return a deep copy of the tree starting at this root *node*. (The returned deep copy shares the same [data](#node_data), however.)

#### Stratify

Consider the following table of relationships:

Name  | Parent
------|--------
Eve   |
Cain  | Eve
Seth  | Eve
Enos  | Seth
Noam  | Seth
Abel  | Eve
Awan  | Eve
Enoch | Awan
Azura | Eve

These names are conveniently unique, so we can unambiguously represent the hierarchy as a CSV file:

```
name,parent
Eve,
Cain,Eve
Seth,Eve
Enos,Seth
Noam,Seth
Abel,Eve
Awan,Eve
Enoch,Awan
Azura,Eve
```

To parse the CSV using [d3.csvParse](https://github.com/d3/d3-dsv#csvParse):

```js
var table = d3.csvParse(text);
```

This returns:

```json
[
  {"name": "Eve",   "parent": ""},
  {"name": "Cain",  "parent": "Eve"},
  {"name": "Seth",  "parent": "Eve"},
  {"name": "Enos",  "parent": "Seth"},
  {"name": "Noam",  "parent": "Seth"},
  {"name": "Abel",  "parent": "Eve"},
  {"name": "Awan",  "parent": "Eve"},
  {"name": "Enoch", "parent": "Awan"},
  {"name": "Azura", "parent": "Eve"}
]
```

To convert to a hierarchy:

```js
var root = d3.stratify()
    .id(function(d) { return d.name; })
    .parentId(function(d) { return d.parent; })
    (table);
```

This returns:

[<img alt="Stratify" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/stratify.png">](https://tonicdev.com/mbostock/56fed33d8630b01300f72daa)

This hierarchy can now be passed to a hierarchical layout, such as [d3.tree](#_tree), for visualization.

<a name="stratify" href="#stratify">#</a> d3.<b>stratify</b>()

Constructs a new stratify operator with the default settings.

<a name="_stratify" href="#_stratify">#</a> <i>stratify</i>(<i>data</i>)

Generates a new hierarchy from the specified tabular *data*. Each node in the returned object has a shallow copy of the properties from the corresponding data object, excluding the following reserved properties: id, parentId, children.

<a name="stratify_id" href="#stratify_id">#</a> <i>stratify</i>.<b>id</b>([<i>id</i>])

If *id* is specified, sets the id accessor to the given function and returns this stratify operator. Otherwise, returns the current id accessor, which defaults to:

```js
function id(d) {
  return d.id;
}
```

The id accessor is invoked for each element in the input data passed to the [stratify operator](#_stratify), being passed the current datum (*d*) and the current index (*i*). The returned string is then used to identify the node’s relationships in conjunction with the [parent id](#stratify_parentId). For leaf nodes, the id may be undefined; otherwise, the id must be unique. (Null and the empty string are equivalent to undefined.)

<a name="stratify_parentId" href="#stratify_parentId">#</a> <i>stratify</i>.<b>parentId</b>([<i>parentId</i>])

If *parentId* is specified, sets the parent id accessor to the given function and returns this stratify operator. Otherwise, returns the current parent id accessor, which defaults to:

```js
function parentId(d) {
  return d.parentId;
}
```

The parent id accessor is invoked for each element in the input data passed to the [stratify operator](#_stratify), being passed the current datum (*d*) and the current index (*i*). The returned string is then used to identify the node’s relationships in conjunction with the [id](#stratify_id). For the root node, the parent id should be undefined. (Null and the empty string are equivalent to undefined.) There must be exactly one root node in the input data, and no circular relationships.

### Cluster

[<img alt="Dendrogram" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/cluster.png">](http://bl.ocks.org/mbostock/ff91c1558bc570b08539547ccc90050b)

The **cluster layout** produces [dendrograms](http://en.wikipedia.org/wiki/Dendrogram): node-link diagrams that place leaf nodes of the tree at the same depth. Dendograms are typically less compact than [tidy trees](#tree), but are useful when all the leaves should be at the same level, such as for hierarchical clustering or [phylogenetic tree diagrams](http://bl.ocks.org/mbostock/c034d66572fd6bd6815a).

<a name="cluster" href="#cluster">#</a> d3.<b>cluster</b>()

Creates a new cluster layout with default settings.

<a name="_cluster" href="#_cluster">#</a> <i>cluster</i>(<i>root</i>)

Lays out the specified *root* [hierarchy](#hierarchy), assigning the following properties on *root* and its descendants:

* *node*.x - the *x*-coordinate of the node
* *node*.y - the *y*-coordinate of the node

The coordinates *x* and *y* represent an arbitrary coordinate system; for example, you can treat *x* as a radius and *y* as an angle to produce a [radial layout](http://bl.ocks.org/mbostock/4739610f6d96aaad2fb1e78a72b385ab). You may want to call [*root*.sort](#node_sort) before passing the hierarchy to the cluster layout.

<a name="cluster_size" href="#cluster_size">#</a> <i>cluster</i>.<b>size</b>([<i>size</i>])

If *size* is specified, sets this cluster layout’s size to the specified two-element array of numbers [*width*, *height*] and returns this cluster layout. If *size* is not specified, returns the current layout size, which defaults to [1, 1]. A layout size of null indicates that a [node size](#node_size) will be used instead. The coordinates *x* and *y* represent an arbitrary coordinate system; for example, to produce a [radial layout](http://bl.ocks.org/mbostock/4739610f6d96aaad2fb1e78a72b385ab), a size of [360, *radius*] corresponds to a breadth of 360° and a depth of *radius*.

<a name="cluster_nodeSize" href="#cluster_nodeSize">#</a> <i>cluster</i>.<b>nodeSize</b>([<i>size</i>])

If *size* is specified, sets this cluster layout’s node size to the specified two-element array of numbers [*width*, *height*] and returns this cluster layout. If *size* is not specified, returns the current node size, which defaults to null. A node size of null indicates that a [layout size](#cluster_size) will be used instead. When a node size is specified, the root node is always positioned at ⟨0, 0⟩.

<a name="cluster_separation" href="#cluster_separation">#</a> <i>cluster</i>.<b>separation</b>([<i>separation</i>])

If *separation* is specified, sets the separation accessor to the specified function and returns this cluster layout. If *separation* is not specified, returns the current separation accessor, which defaults to:

```js
function separation(a, b) {
  return a.parent == b.parent ? 1 : 2;
}
```

The separation accessor is used to separate neighboring leaves. The separation function is passed two leaves *a* and *b*, and must return the desired separation. The nodes are typically siblings, though the nodes may be more distantly related if the layout decides to place such nodes adjacent.

### Tree

[<img alt="Tidy Tree" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/tree.png">](http://bl.ocks.org/mbostock/9d0899acb5d3b8d839d9d613a9e1fe04)

The **tree** layout produces tidy node-link diagrams of trees using the [Reingold–Tilford “tidy” algorithm](http://emr.cs.iit.edu/~reingold/tidier-drawings.pdf), improved to run in linear time by [Buchheim *et al.*](http://dirk.jivas.de/papers/buchheim02improving.pdf) Tidy trees are typically more compact than [dendograms](#cluster).

<a name="tree" href="#tree">#</a> d3.<b>tree</b>()

Creates a new tree layout with default settings.

<a name="_tree" href="#_tree">#</a> <i>tree</i>(<i>root</i>)

Lays out the specified *root* [hierarchy](#hierarchy), assigning the following properties on *root* and its descendants:

* *node*.x - the *x*-coordinate of the node
* *node*.y - the *y*-coordinate of the node

The coordinates *x* and *y* represent an arbitrary coordinate system; for example, you can treat *x* as a radius and *y* as an angle to produce a [radial layout](http://bl.ocks.org/mbostock/2e12b0bd732e7fe4000e2d11ecab0268). You may want to call [*root*.sort](#node_sort) before passing the hierarchy to the tree layout.

<a name="tree_size" href="#tree_size">#</a> <i>tree</i>.<b>size</b>([<i>size</i>])

If *size* is specified, sets this tree layout’s size to the specified two-element array of numbers [*width*, *height*] and returns this tree layout. If *size* is not specified, returns the current layout size, which defaults to [1, 1]. A layout size of null indicates that a [node size](#node_size) will be used instead. The coordinates *x* and *y* represent an arbitrary coordinate system; for example, to produce a [radial layout](http://bl.ocks.org/mbostock/2e12b0bd732e7fe4000e2d11ecab0268), a size of [360, *radius*] corresponds to a breadth of 360° and a depth of *radius*.

<a name="tree_nodeSize" href="#tree_nodeSize">#</a> <i>tree</i>.<b>nodeSize</b>([<i>size</i>])

If *size* is specified, sets this tree layout’s node size to the specified two-element array of numbers [*width*, *height*] and returns this tree layout. If *size* is not specified, returns the current node size, which defaults to null. A node size of null indicates that a [layout size](#tree_size) will be used instead. When a node size is specified, the root node is always positioned at ⟨0, 0⟩.

<a name="tree_separation" href="#tree_separation">#</a> <i>tree</i>.<b>separation</b>([<i>separation</i>])

If *separation* is specified, sets the separation accessor to the specified function and returns this tree layout. If *separation* is not specified, returns the current separation accessor, which defaults to:

```js
function separation(a, b) {
  return a.parent == b.parent ? 1 : 2;
}
```

A variation that is more appropriate for radial layouts reduces the separation gap proportionally to the radius:

```js
function separation(a, b) {
  return (a.parent == b.parent ? 1 : 2) / a.depth;
}
```

The separation accessor is used to separate neighboring nodes. The separation function is passed two nodes *a* and *b*, and must return the desired separation. The nodes are typically siblings, though the nodes may be more distantly related if the layout decides to place such nodes adjacent.

### Treemap

[<img alt="Treemap" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/treemap.png">](http://bl.ocks.org/mbostock/6bbb0a7ff7686b124d80)

Introduced by [Ben Shneiderman](http://www.cs.umd.edu/hcil/treemap-history/) in 1991, a **treemap** recursively subdivides area into rectangles according to each node’s associated value. D3’s treemap implementation supports an extensible [tiling method](#treemap_tile): the default [squarified](#treemapSquarified) method seeks to generate rectangles with a [golden](https://en.wikipedia.org/wiki/Golden_ratio) aspect ratio; this offers better readability and size estimation than [slice-and-dice](#treemapSliceDice), which simply alternates between horizontal and vertical subdivision by depth.

<a name="treemap" href="#treemap">#</a> d3.<b>treemap</b>()

Creates a new treemap layout with default settings.

<a name="_treemap" href="#_treemap">#</a> <i>treemap</i>(<i>root</i>)

Lays out the specified *root* [hierarchy](#hierarchy), assigning the following properties on *root* and its descendants:

* *node*.x0 - the left edge of the rectangle
* *node*.y0 - the top edge of the rectangle
* *node*.x1 - the right edge of the rectangle
* *node*.y1 - the bottom edge of the rectangle

You must call [*root*.sum](#node_sum) before passing the hierarchy to the treemap layout. You probably also want to call [*root*.sort](#node_sort) to order the hierarchy before computing the layout.

<a name="treemap_tile" href="#treemap_tile">#</a> <i>treemap</i>.<b>tile</b>([<i>tile</i>])

If *tile* is specified, sets the [tiling method](#treemap-tiling) to the specified function and returns this treemap layout. If *tile* is not specified, returns the current tiling method, which defaults to [d3.treemapSquarify](#treemapSquarify) with the golden ratio.

<a name="treemap_size" href="#treemap_size">#</a> <i>treemap</i>.<b>size</b>([<i>size</i>])

If *size* is specified, sets this treemap layout’s size to the specified two-element array of numbers [*width*, *height*] and returns this treemap layout. If *size* is not specified, returns the current size, which defaults to [1, 1].

<a name="treemap_round" href="#treemap_round">#</a> <i>treemap</i>.<b>round</b>([<i>round</i>])

If *round* is specified, enables or disables rounding according to the given boolean and returns this treemap layout. If *round* is not specified, returns the current rounding state, which defaults to false.

<a name="treemap_padding" href="#treemap_padding">#</a> <i>treemap</i>.<b>padding</b>([<i>padding</i>])

If *padding* is specified, sets the [inner](#treemap_paddingInner) and [outer](#treemap_paddingOuter) padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current inner padding function.

<a name="treemap_paddingInner" href="#treemap_paddingInner">#</a> <i>treemap</i>.<b>paddingInner</b>([<i>padding</i>])

If *padding* is specified, sets the inner padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current inner padding function, which defaults to the constant zero. If *padding* is a function, it is invoked for each node with children, being passed the current node. The inner padding is used to separate a node’s adjacent children.

<a name="treemap_paddingOuter" href="#treemap_paddingOuter">#</a> <i>treemap</i>.<b>paddingOuter</b>([<i>padding</i>])

If *padding* is specified, sets the [top](#treemap_paddingTop), [right](#treemap_paddingRight), [bottom](#treemap_paddingBottom) and [left](#treemap_paddingLeft) padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current top padding function.

<a name="treemap_paddingTop" href="#treemap_paddingTop">#</a> <i>treemap</i>.<b>paddingTop</b>([<i>padding</i>])

If *padding* is specified, sets the top padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current top padding function, which defaults to the constant zero. If *padding* is a function, it is invoked for each node with children, being passed the current node. The top padding is used to separate the top edge of a node from its children.

<a name="treemap_paddingRight" href="#treemap_paddingRight">#</a> <i>treemap</i>.<b>paddingRight</b>([<i>padding</i>])

If *padding* is specified, sets the right padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current right padding function, which defaults to the constant zero. If *padding* is a function, it is invoked for each node with children, being passed the current node. The right padding is used to separate the right edge of a node from its children.

<a name="treemap_paddingBottom" href="#treemap_paddingBottom">#</a> <i>treemap</i>.<b>paddingBottom</b>([<i>padding</i>])

If *padding* is specified, sets the bottom padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current bottom padding function, which defaults to the constant zero. If *padding* is a function, it is invoked for each node with children, being passed the current node. The bottom padding is used to separate the bottom edge of a node from its children.

<a name="treemap_paddingLeft" href="#treemap_paddingLeft">#</a> <i>treemap</i>.<b>paddingLeft</b>([<i>padding</i>])

If *padding* is specified, sets the left padding to the specified number or function and returns this treemap layout. If *padding* is not specified, returns the current left padding function, which defaults to the constant zero. If *padding* is a function, it is invoked for each node with children, being passed the current node. The left padding is used to separate the left edge of a node from its children.

#### Treemap Tiling

Several built-in tiling methods are provided for use with [*treemap*.tile](#treemap_tile).

<a name="treemapBinary" href="#treemapBinary">#</a> d3.<b>treemapBinary</b>(<i>node</i>, <i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

Recursively partitions the specified *nodes* into an approximately-balanced binary tree, choosing horizontal partitioning for wide rectangles and vertical partitioning for tall rectangles.

<a name="treemapDice" href="#treemapDice">#</a> d3.<b>treemapDice</b>(<i>node</i>, <i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

Divides the rectangular area specified by *x0*, *y0*, *x1*, *y1* horizontally according the value of each of the specified *node*’s children. The children are positioned in order, starting with the left edge (*x0*) of the given rectangle. If the sum of the children’s values is less than the specified *node*’s value (*i.e.*, if the specified *node* has a non-zero internal value), the remaining empty space will be positioned on the right edge (*x1*) of the given rectangle.

<a name="treemapSlice" href="#treemapSlice">#</a> d3.<b>treemapSlice</b>(<i>node</i>, <i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

Divides the rectangular area specified by *x0*, *y0*, *x1*, *y1* vertically according the value of each of the specified *node*’s children. The children are positioned in order, starting with the top edge (*y0*) of the given rectangle. If the sum of the children’s values is less than the specified *node*’s value (*i.e.*, if the specified *node* has a non-zero internal value), the remaining empty space will be positioned on the bottom edge (*y1*) of the given rectangle.

<a name="treemapSliceDice" href="#treemapSliceDice">#</a> d3.<b>treemapSliceDice</b>(<i>node</i>, <i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

If the specified *node* has odd depth, delegates to [treemapSlice](#treemapSlice); otherwise delegates to [treemapDice](#treemapDice).

<a name="treemapSquarify" href="#treemapSquarify">#</a> d3.<b>treemapSquarify</b>(<i>node</i>, <i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

Implements the [squarified treemap](https://www.win.tue.nl/~vanwijk/stm.pdf) algorithm by Bruls *et al.*, which seeks to produce rectangles of a given [aspect ratio](#squarify_ratio).

<a name="squarify_ratio" href="#squarify_ratio">#</a> <i>squarify</i>.<b>ratio</b>(<i>ratio</i>)

Specifies the desired aspect ratio of the generated rectangles. The specified *ratio* is merely a hint; the rectangles are not guaranteed to have the specified aspect ratio. If not specified, the aspect ratio defaults to the golden ratio, φ = (1 + sqrt(5)) / 2, per [Kong *et al.*](http://vis.stanford.edu/papers/perception-treemaps)

### Partition

[<img alt="Partition" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/partition.png">](http://bl.ocks.org/mbostock/2e73ec84221cb9773f4c)

The **partition layout** produces adjacency diagrams: a space-filling variant of a node-link tree diagram. Rather than drawing a link between parent and child in the hierarchy, nodes are drawn as solid areas (either arcs or rectangles), and their placement relative to other nodes reveals their position in the hierarchy. The size of the nodes encodes a quantitative dimension that would be difficult to show in a node-link diagram.

<a name="partition" href="#partition">#</a> d3.<b>partition</b>()

Creates a new partition layout with the default settings.

<a name="_partition" href="#_partition">#</a> <i>partition</i>(<i>root</i>)

Lays out the specified *root* [hierarchy](#hierarchy), assigning the following properties on *root* and its descendants:

* *node*.x0 - the left edge of the rectangle
* *node*.y0 - the top edge of the rectangle
* *node*.x1 - the right edge of the rectangle
* *node*.y1 - the bottom edge of the rectangle

You must call [*root*.sum](#node_sum) before passing the hierarchy to the partition layout. You probably also want to call [*root*.sort](#node_sort) to order the hierarchy before computing the layout.

<a name="partition_size" href="#partition_size">#</a> <i>partition</i>.<b>size</b>([<i>size</i>])

If *size* is specified, sets this partition layout’s size to the specified two-element array of numbers [*width*, *height*] and returns this partition layout. If *size* is not specified, returns the current size, which defaults to [1, 1].

<a name="partition_round" href="#partition_round">#</a> <i>partition</i>.<b>round</b>([<i>round</i>])

If *round* is specified, enables or disables rounding according to the given boolean and returns this partition layout. If *round* is not specified, returns the current rounding state, which defaults to false.

<a name="partition_padding" href="#partition_padding">#</a> <i>partition</i>.<b>padding</b>([<i>padding</i>])

If *padding* is specified, sets the padding to the specified number and returns this partition layout. If *padding* is not specified, returns the current padding, which defaults to zero. The padding is used to separate a node’s adjacent children.

### Pack

[<img alt="Circle-Packing" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/pack.png">](http://bl.ocks.org/mbostock/ca5b03a33affa4160321)

Enclosure diagrams use containment (nesting) to represent a hierarchy. The size of the leaf circles encodes a quantitative dimension of the data. The enclosing circles show the approximate cumulative size of each subtree, but due to wasted space there is some distortion; only the leaf nodes can be compared accurately. Although [circle packing](http://en.wikipedia.org/wiki/Circle_packing) does not use space as efficiently as a [treemap](#treemap), the “wasted” space more prominently reveals the hierarchical structure.

<a name="pack" href="#pack">#</a> d3.<b>pack</b>()

Creates a new pack layout with the default settings.

<a name="_pack" href="#_pack">#</a> <i>pack</i>(<i>root</i>)

Lays out the specified *root* [hierarchy](#hierarchy), assigning the following properties on *root* and its descendants:

* *node*.x - the *x*-coordinate of the circle’s center
* *node*.y - the *y*-coordinate of the circle’s center
* *node*.r - the radius of the circle

You must call [*root*.sum](#node_sum) before passing the hierarchy to the pack layout. You probably also want to call [*root*.sort](#node_sort) to order the hierarchy before computing the layout.

<a name="pack_radius" href="#pack_radius">#</a> <i>pack</i>.<b>radius</b>([<i>radius</i>])

If *radius* is specified, sets the pack layout’s radius accessor to the specified function and returns this pack layout. If *radius* is not specified, returns the current radius accessor, which defaults to null. If the radius accessor is null, the radius of each leaf circle is derived from the leaf [*node*.value](#node_value) (computed by [*node*.sum](#node_sum)); the radii are then scaled proportionally to fit the [layout size](#pack_size). If the radius accessor is not null, the radius of each leaf circle is specified exactly by the function.

<a name="pack_size" href="#pack_size">#</a> <i>pack</i>.<b>size</b>([<i>size</i>])

If *size* is specified, sets this pack layout’s size to the specified two-element array of numbers [*width*, *height*] and returns this pack layout. If *size* is not specified, returns the current size, which defaults to [1, 1].

<a name="pack_padding" href="#pack_padding">#</a> <i>pack</i>.<b>padding</b>([<i>padding</i>])

If *padding* is specified, sets this pack layout’s padding accessor to the specified number or function or returns this pack layout. If *padding* is not specified, returns the current padding accessor, which defaults to the constant zero. When siblings are packed, tangent siblings will be separated by approximately the specified padding; the enclosing parent circle will also be separated from its children by approximately the specified padding. If an [explicit radius](#pack_radius) is not specified, the padding is approximate because a two-pass algorithm is needed to fit within the [layout size](#pack_size): the circles are first packed without padding; a scaling factor is computed and applied to the specified padding; and lastly the circles are re-packed with padding.

<a name="packSiblings" href="#packSiblings">#</a> d3.<b>packSiblings</b>(<i>circles</i>)

Packs the specified array of *circles*, each of which must have a *circle*.r property specifying the circle’s radius. Assigns the following properties to each circle:

* *circle*.x - the *x*-coordinate of the circle’s center
* *circle*.y - the *y*-coordinate of the circle’s center

The circles are positioned according to the front-chain packing algorithm by [Wang *et al.*](https://dl.acm.org/citation.cfm?id=1124851)

<a name="packEnclose" href="#packEnclose">#</a> d3.<b>packEnclose</b>(<i>circles</i>)

Computes the [smallest circle](https://en.wikipedia.org/wiki/Smallest-circle_problem) that encloses the specified array of *circles*, each of which must have a *circle*.r property specifying the circle’s radius, and *circle*.x and *circle*.y properties specifying the circle’s center. The enclosing circle is computed using [Welzl’s algorithm](http://link.springer.com/chapter/10.1007/BFb0038202) adapted to enclose circles rather than points. (See also [Apollonius’ Problem](https://bl.ocks.org/mbostock/751fdd637f4bc2e3f08b).)
