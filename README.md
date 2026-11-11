# graph

Adjacency-list graphs for [Ecko](https://ecko.sh) and the questions they
answer: what can reach what, by which route, and in what order. Pure
computation - no capabilities.

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

print(graph.reachable(approvals, "alice"))  # [bob, carol, dave, erin]
print(graph.path(approvals, "alice", "erin"))  # [alice, bob, dave, erin]
print(graph.distance(approvals, "alice", "erin"))  # 3
print(graph.connected(approvals, "erin", "alice"))  # false - delegation has a direction
print(graph.has_cycle(approvals))  # false - nobody can approve their own request

# The same structure answers "what order do I build these in?"
deps = graph.from_edges([["lex", "parse"], ["parse", "compile"]])
print(graph.topo_sort(deps))  # [lex, parse, compile]
```

### Undirected graphs

Builders return a new graph rather than changing the one they were given, so
they compose with `|>`:

```ecko
import graph

friends = graph.new(directed: false)
    |> graph.add_edge("ann", "ben")
    |> graph.add_edge("dan", "eve")
    |> graph.add_node("frank")

print(graph.components(friends))  # [[ann, ben], [dan, eve], [frank]]
print(graph.has_cycle(friends))  # false - one edge is not a cycle
```

## API

### Building

| function | returns |
|---|---|
| `new(directed: true)` | an empty graph, directed unless `directed: false` |
| `from_edges(pairs, directed: true)` | a graph built from a list of `[src, dst]` pairs |
| `add_node(g, node)` | `g` with `node` in it; adding one that exists changes nothing |
| `add_edge(g, src, dst)` | `g` with the edge, adding either endpoint that is missing; an undirected graph gets the reverse edge too |
| `remove_edge(g, src, dst)` | `g` without that edge; both nodes stay |
| `remove_node(g, node)` | `g` without that node or any edge touching it |

### Inspecting

| function | returns |
|---|---|
| `nodes(g)` | every node, sorted |
| `edges(g)` | every edge as sorted `[src, dst]` pairs; an undirected edge appears once |
| `neighbors(g, node)` | the nodes `node` points at, sorted |
| `has_node(g, node)` | `true` when `node` is in the graph |
| `has_edge(g, src, dst)` | `true` when there is an edge from `src` to `dst` |
| `order(g)` | how many nodes |
| `size(g)` | how many edges |
| `out_degree(g, node)` | how many edges leave `node` |
| `in_degree(g, node)` | how many edges arrive at `node` |
| `degree(g, node)` | out-degree plus in-degree |

### Traversing

| function | returns |
|---|---|
| `bfs(g, start)` | the nodes in breadth-first order, `start` first |
| `dfs(g, start)` | the nodes in depth-first order, `start` first |
| `reachable(g, start)` | everything reachable from `start`, sorted, excluding `start` |
| `connected(g, src, dst)` | `true` when `src` can reach `dst`; a node always reaches itself |
| `path(g, src, dst)` | the fewest-hops route as a list of nodes, or `null` if there is none |
| `distance(g, src, dst)` | the number of hops on that route, or `null` |
| `components(g)` | the connected groups (weak components on a directed graph), each sorted, the list sorted by first member |
| `has_cycle(g)` | `true` when the graph has a cycle; a diamond is not one, and on an undirected graph neither is a single edge |
| `topo_sort(g)` | the nodes ordered so every edge points forwards, smallest ready node first |

## Notes

### Errors

| kind | reason | when |
|---|---|---|
| `graph` | `no_such_node` | a traversal or degree names a node that is not in the graph |
| `graph` | `cycle` | `topo_sort` on a graph with a cycle, a self-loop included |
| `bug` | `bad_node` | a node is not a string |
| `bug` | `bad_argument` | `directed` is not a Bool, `from_edges` got something other than a list of pairs, or `topo_sort` got an undirected graph |

### Behaviour

**Nodes are strings.** They are the keys of the graph's adjacency map; convert
ids with `string(id)` first.

**Everything that returns a list is sorted.** A graph is stored as a map, and
Ecko maps are unordered, so an unsorted `nodes` would come back differently on a
different run and every traversal built on it would inherit that. Sorting is
what makes these results testable, diffable and reproducible.

**Nothing recurses.** Every traversal, `has_cycle` included, keeps its own
queue or stack, so a long chain cannot reach the recursion limit.

**Unknown nodes raise rather than returning empty.** `neighbors(g, "typo")`
raises `graph`/`no_such_node`. Silently returning `[]` is how a traversal
quietly explores nothing and reports success.

**`components` on a directed graph follows edges both ways**, giving weakly
connected components: every node is in exactly one group.

**A self-loop is a cycle.** `has_cycle` reports it and `topo_sort` refuses it.

**Every edge is one hop.** There are no weights.

**Lookups are linear.** `has_edge` scans a neighbour list and `in_degree` scans
every node. That is the right trade for the sizes this package targets - tens or
hundreds of nodes, not millions.

## Testing

```bash
ecko test
```

## License

[MIT](LICENSE)
