# Changelog

## 0.38.0

### Fixed

- `components` on a directed graph no longer drops nodes
  (`from_edges([["a", "b"], ["c", "b"]])` gave `[[a, b]]`). It now returns the
  weakly connected components, with every node in exactly one group.
- `has_cycle` on an undirected graph no longer reports a cycle for a single
  edge; it needs a real cycle (or a self-loop).
- `has_cycle` no longer fails with "recursion limit exceeded" on a long chain
  (it failed at 2,500 nodes).
- `from_edges` is no longer quadratic in the number of edges: 5,000 edges took
  over a second and now take milliseconds. `bfs`, `path`, `distance`,
  `reachable` and `connected` are faster on large graphs too.
- `connected` raises `graph`/`no_such_node` for an unknown node on either side;
  before, an unknown destination answered `false` while an unknown source
  raised.
- A node that is not a string now fails with `bug`/`bad_node` naming the
  function, instead of "insert needs a string, got int".
- `from_edges` with an edge that is not a pair, or with a non-list, fails with
  `bug`/`bad_argument` instead of an index error.
- Errors carry a `reason` (`no_such_node`, `cycle`) and name the function.

### Breaking

- `new` and `from_edges` take `directed:` as a named argument instead of an
  options map. Migration: `graph.new({ directed: false })` becomes
  `graph.new(directed: false)`, and `graph.from_edges(pairs, { directed: false })`
  becomes `graph.from_edges(pairs, directed: false)`.
- `link` and `unlink` are no longer exported. Used directly they could leave a
  graph with an edge to a node that does not exist. Migration: use `add_edge`
  and `remove_edge`.
- `topo_sort` refuses a self-loop with `graph`/`cycle`, matching `has_cycle`,
  which reports one. Migration: remove self-loops before sorting.
- `topo_sort` on an undirected graph fails with `bug`/`bad_argument` instead of
  kind `graph`. Migration: build the graph directed.
