# Project 6: Routing with Dijkstra's

Internal Gateway Protocols share information with one another such that
every router has a complete make of the network they're a part of. This
way, each router can make autonomous routing decisions given an IP
address. No matter where it's going, the router can always forward the
packet in the right direction.

IGPs like Open Shortest Path First (OSPF) use Dijkstra's Algorithm to
find the shortest path along a weighted graph.

In this project, we're going to simulate that routing. Were going to
implement Dijkstra's Algorithm to print out the shortest path from one
IP to another IP, showing the IPs of all the routers in between.

We won't be using a real network for this. Rather, your program will
read in a JSON file that contains the network description and then
compute the route from that.

## Graphs Refresher

Graphs are made of _vertices_ and _edges_. Sometimes vertices are called
"vertexes" or "verts" or "nodes". Edges are connections from one node to
another.

Any node on the graph can be connected to any number of other nodes,
even zero. A node can be connected to every single other node. It could
even connect to itself.

An edge can have a _weight_ which generally means the cost of traversing
that edge when you're walking a path through the graph.

For example, imagine a highway map that shows cities and highways
between the cities. And each highway is labeled with its length. In this
example, the cities would be vertices, the highways would be edges, and
the highway length would be the edge weight.

When traversing a graph, the goal is to minimize the total of all the
edge waits that you encounter along the way. On our map, the goal would
be to choose edges from our starting city through all the intermediate
cities to our destination city that had us driving the minimum total
distance.

## Dijkstra's Algorithm Overview

Edsger Dijkstra (_DIKE-struh_) was a famous computer scientist who came
up with a lot of things, but one of them was so influential it came to
be known by only his name: Dijkstra's Algorithm.

> Protip: The secret to spelling "Dijkstra" is remembering that "ijk"
> appears in order.

If you want to find the shortest path between nodes in an unweighted
graph, you merely need to perform a breadth-first traversal until you
find what you're looking for. Distances are only measured in "hops".

But if you add weights to the edges between the nodes, BFT can't help
you differentiate them. Maybe some paths are very desirable, and others
are very undesirable.

Dijkstra's _can_ differentiate. It's a BFT with a twist.

The gist of the algorithm is this: explore outward from the starting
point, pursuing only the path with the shortest total length so far.

Each path's total weight is the sum of the weights of all its edges.

In a well-connected graph, there will be a _lot_ of potential paths from
the start to the destination. But since we only pursue the shortest
known path so far, we'll never pursue one that takes us a million miles
out of the way, assuming we know of a path that is shorter than a
million miles.

## Dijkstra's Implementation

Dijkstra's builds a tree structure on top of a graph. When you find the
shortest path from any node back toward the start, that node records the
prior node in its path as its _parent_. 

If another shorter path comes to be found later, the parent is switched
to the new shorter path's node.

[flw[Wikipedia has some great diagrams that show it in
action|Dijkstra's_algorithm]].

Now how do make it work?

Dijkstra's itself only builds the tree representing the shortest paths
back to the start. We'll follow that shortest path tree later to find a
particular path.

* Dijkstra's Algorithm to compute all shortest paths over a graph from a
  source point:
  * Initialization:
    * Create an empty `to_visit` set. This is the set of all nodes we
      still need to visit.
    * Create a `distance` dictionary. For any given node (as a key), it
      will hold the distance from that node to the starting node
    * Create a `parent` dictionary. For any given node (as a key), it
      lists the key for the that leads back to the starting node (along
      the shortest path).
    * For every node:
      * Set its `parent` to `None`.
      * Set its `distance` to infinity. (Python has infinity in
        `math.inf`, but you could also use just a very large number, e.g.
        4 billion.)
      * Add the node to the `to_visit` set.

  * Running:
    * While `to_visit` isn't empty:
      * Find the node in `to_visit` with the smallest `distance`. Call
        this the "current node".
      * Remove the current node from the `to_visit` set.
      * For each one of the current node's neighbors still in `to_visit`:
        * Compute the distance from the starting node to the neighbor.
          This is the distance of the current node plus the edge weight to
          the neighbor.
        * If the computed distance is less than the neighbor's current
          value in `distance`:
          * Set the neighbor's value in `distance` to the computed
            distance.
          * Set the neighbor's `parent` to the current node.
        * [This process is called "relaxing". The node distances start
          at infinity and "relax" down to their shortest distances.]
 
Wikipedia offers this pseudocode, if that's easier to digest:

``` {.py}
 1  function Dijkstra(Graph, source):
 2
 3      for each vertex v in Graph.Vertices:
 4          dist[v] ← INFINITY
 5          prev[v] ← UNDEFINED
 6          add v to Q
 7      dist[source] ← 0
 8
 9      while Q is not empty:
10          u ← vertex in Q with min dist[u]
11          remove u from Q
12
13          for each neighbor v of u still in Q:
14              alt ← dist[u] + Graph.Edges(u, v)
15              if alt < dist[v]:
16                  dist[v] ← alt
17                  prev[v] ← u
18
19      return dist[], prev[]
```

At this point, we have constructed our tree made up of all the `parent`
pointers.

To find the shortest path from one point back to the start (at the root
of the tree), you need to just follow the `parent` pointers from that
point back up the tree.

* Get Shortest Path from source to destination:
  * Set our current node to the **destination** node.
  * Set our `path` to be an empty array.
  * While current node is not starting node:
    * Append current node to `path`.
    * current node = the `parent` of current node
  * Append the starting node to the path.

Of course, this will build the path in reverse order. It has to, since
the parent pointers all point back to the starting node at the root of
the tree.  Either reverse it at the end, or run the main Dijkstra's
algorithm passing the destination in for the source.

### Getting the Minimum Distance

Part of the algorithm is to find the node with the minimum `distance`
that is still in the `to_visit` set.

For this project, you can just do a `O(n)` linear search to find the
node with the shortest distance so far.

In real life, this is too expensive--`O(n²)` performance over the number
of vertices. So implementations will use a [min
heap](https://en.wikipedia.org/wiki/Binary_heap) which will effectively
get us the minimum in far-superior `O(log n)` time. This gets us to `O(n
log n)` over the number of verts.

If you wish the additional challenge, use a min heap.

## What About Our Project?

[_All IP addresses in this project are IPv4 addresses._]

[fls[Download the skeleton code ZIP here|dijkstra/dijkstra.zip]].

OK, so that was a lot of general stuff.

So what does that correspond to in the project?

### The Function, Inputs, and Outputs

You have to implement this function:

``` {.py}
def dijkstras_shortest_path(routers, src_ip, dest_ip):
```

The function inputs are:

* `routers`: A dictionary representing the graph.
* `src_ip`: A source IP address as a dots-and-numbers string.
* `dest_ip`: A destination IP address as a dots-and-numbers string.

The function output is:

* An array of strings showing all the router IP addresses along the
  route.
  * **Note: If the source IP and destination IP are on the same subnet
    as one another, return an empty array.** No routers would be
    involved in this case.

Code to drive your function is already included in the skeleton code
above. It will output to the console lines like this showing the source,
destination, and all routers in between:

``` {.default}
10.34.52.158 -> 10.34.166.1 ['10.34.52.1', '10.34.250.1', '10.34.166.1']
```

### The Graph Representation

The graph dictionary in `routers` looks like this excerpt:

``` {.json}
{
    "10.34.98.1": {
        "connections": {
            "10.34.166.1": {
                "netmask": "/24",
                "interface": "en0",
                "ad": 70
            },
            "10.34.194.1": {
                "netmask": "/24",
                "interface": "en1",
                "ad": 93
            },
            "10.34.46.1": {
                "netmask": "/24",
                "interface": "en2",
                "ad": 64
            }
        },
        "netmask": "/24",
        "if_count": 3,
        "if_prefix": "en"
    },
     
    # ... etc. ...
```

The top-level keys (e.g. `"10.34.98.1"`) are the router IPs. These are
the vertices of the graph.

For each of those, you have a list of `"connections"` which are the
edges of the graph.

In each connection, you have a field `"ad"` which is the edge weight.

> "AD" is short for _Administrative Distance_. This is a weight set
> manually or automatically (or a mix of both) that defines how
> expensive a particular segment of the route is.
> The default value is 110. Higher numbers are more expensive.
>
> The metric encompasses a number of ideas about the route, including
> how much bandwidth it provides, how congested it is, how much the
> administrators want it used (or not), and so on.

The netmask for the router IP is in the `"netmask"` field, and there are
additional `"netmask"` fields for all the connection routers, as well.

The `"interface"` says which network device on the router is used to
reach a neighboring router. It is unused in this project.

`"if_count"` and `"if_prefix"` are also unused in this project.

## Input File and Example Output

The skeleton archive includes an example input file (`example1.json`)
and expected output for that file (`example1_output.json`).

## Hints

* Rely heavily on the network functions you wrote in the previous project!
* Fully understand this project description before coming up with a
  plan!
* Come up with a plan as much as possible before writing any code!

<!--
Rubric:

12
Starting router IP is correctly determined from source IP address.

12
Ending router IP is correctly determined from destination IP address.

8
Shortest path array does not include start IP

8
Shortest path array does not include end IP

20
Shortest path array includes all router IPs from source to destination in order source-to-destination.

20
Shortest path array contains the actual shortest path.

12
Empty shortest path is returned when source and destination are on the same subnet.

5
Code requested to be unmodified is unmodified
-->

