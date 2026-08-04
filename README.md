# graph

Adjacency lists, traversal, and the questions they answer: what can reach what,
by which route, and in what order.

Built for the small explicit graphs that turn up in application code -
permission and delegation chains, dependency orders, category trees. Every edge
counts as one hop. Weights, shortest-path-by-cost and flow are a separate
package.

Pure computation: no capabilities, no I/O.

## Install

```bash
ecko get github.com/ecko-lang/graph
```

```ecko
import graph
```

## Usage

```ecko
import graph

# Who can approve a spend alice raised, directly or by delegation?
approvals = graph.from_edges([
    ["alice", "bob"],
    ["alice", "carol"],
    ["bob", "dave"],
    ["dave", "erin"],
])

graph.reachable(approvals, "alice")      # ["bob", "carol", "dave", "erin"]
graph.path(approvals, "alice", "erin")   # ["alice", "bob", "dave", "erin"]
graph.distance(approvals, "alice", "erin")   # 3
graph.connected(approvals, "erin", "alice")  # false - delegation has a direction
graph.has_cycle(approvals)               # false - nobody can approve their own request

# The same structure answers "what order do I build these in?"
deps = graph.from_edges([["lex", "parse"], ["parse", "compile"]])
graph.topo_sort(deps)                    # ["lex", "parse", "compile"]
```

Builders return a new graph rather than mutating, so they compose:

```ecko
friends = graph.new({ directed: false })
    |> graph.add_edge("ann", "ben")
    |> graph.add_edge("dan", "eve")
    |> graph.add_node("frank")

graph.components(friends)   # [["ann", "ben"], ["dan", "eve"], ["frank"]]
```

## API

### Building

| Function | Result |
|---|---|
| `new(opts = {})` | An empty graph. `{ directed: false }` for an undirected one; the default is directed. |
| `from_edges(pairs, opts = {})` | A graph built from a list of `[src, dst]` pairs. |
| `add_node(g, node)` | `g` with `node` in it. Adding one that exists changes nothing. |
| `add_edge(g, src, dst)` | `g` with the edge, adding either endpoint that is missing. Undirected graphs get the reverse edge too. |
| `remove_edge(g, src, dst)` | `g` without that edge. Both nodes stay. |
| `remove_node(g, node)` | `g` without that node or any edge touching it. |
| `link(g, src, dst)` / `unlink(g, src, dst)` | One direction only. `add_edge`/`remove_edge` are built on these. |

### Inspecting

| Function | Result |
|---|---|
| `nodes(g)` | Every node, sorted. |
| `edges(g)` | Every edge as sorted `[src, dst]` pairs. An undirected edge appears once. |
| `neighbors(g, node)` | The nodes `node` points at, sorted. Raises if the node is unknown. |
| `has_node(g, node)` / `has_edge(g, src, dst)` | Booleans. |
| `order(g)` / `size(g)` | How many nodes / how many edges. |
| `out_degree(g, node)` / `in_degree(g, node)` / `degree(g, node)` | Edge counts at a node. |

### Traversing

| Function | Result |
|---|---|
| `bfs(g, start)` | Breadth-first order, `start` first. |
| `dfs(g, start)` | Depth-first order, `start` first. |
| `reachable(g, start)` | Everything reachable from `start`, sorted, excluding `start`. |
| `connected(g, src, dst)` | Can `src` reach `dst`? A node always reaches itself. |
| `path(g, src, dst)` | The fewest-hops route as a list of nodes, or `null` if there is none. |
| `distance(g, src, dst)` | The number of hops on that route, or `null`. |
| `components(g)` | Connected groups, each sorted, the list sorted by first member. |
| `has_cycle(g)` | Is there a directed cycle? A diamond is not a cycle. |
| `topo_sort(g)` | Nodes ordered so every edge points forwards. Raises on a cyclic graph. |

## Notes

**Everything that returns a list is sorted.** A graph is stored as a map, and
Ecko maps are unordered, so an unsorted `nodes` would come back differently on a
different run and every traversal built on it would inherit that. Sorting is
what makes these results testable, diffable and reproducible.

**Traversal is iterative, not recursive.** A long chain would otherwise hit the
recursion cap, and a graph big enough to matter is exactly the one that has long
chains. `has_cycle` is the exception: it recurses, so it is bounded by the depth
of the graph rather than its size.

**Unknown nodes raise rather than returning empty.** `neighbors(g, "typo")`
raises `{ kind: "graph", message: ... }`. Silently returning `[]` is how a
traversal quietly explores nothing and reports success.

**`components` is for undirected graphs.** On a directed one it follows edges
forwards only, so it reports what is reachable rather than true weak components.

**Lookups are linear.** `has_edge` scans a neighbour list and `in_degree` scans
every node. That is the right trade for the sizes this package targets - tens or
hundreds of nodes, not millions.

## Testing

```bash
ecko test
```

Offline and deterministic: no `ai`, no filesystem, no network.

## License

MIT. See [LICENSE](LICENSE).
