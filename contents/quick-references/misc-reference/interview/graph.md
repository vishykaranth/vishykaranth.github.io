#Graph 

### Usecases
- **Maps** 
    - A person who is planning a trip may need to answer questions such as "What is the least expensive way to get from Princeton to San Jose?" 
    - A person more interested in time than in money may need to know the answer to the question
    - "What is the fastest way to get from Princeton to San Jose?" 
    - To answer such questions, we process information about connections (travel routes) between items (towns and cities).
- **Hypertexts** 
    - When we browse the Web, we encounter documents that contain references (links) to other documents and we move from document to document by clicking on the links. 
    - The entire Web is a graph, where the items are documents and the connections are links. 
    - Graph-processing algorithms are essential components of the search engines that help us locate information on the Web.
- **Circuits** 
    - An electric circuit comprises elements such as transistors, resistors, and capacitors that are intricately wired together. 
    - We use computers to control machines that make circuits and to check that the circuits perform desired functions. 
    - We need to answer simple questions such as "Is a short-circuit present?" as well as complicated questions such as 
    - "Can we lay out this circuit on a chip without making any wires cross?" 
    - In this case, the answer to the first question depends on only the properties of the connections (wires), 
    - whereas the answer to the second question requires detailed information about the wires, the items that those wires connect, and the physical constraints of the chip.
- **Schedules** 
    - A manufacturing process requires a variety of tasks to be performed, under a set of constraints that specifies that certain tasks cannot be started until certain other tasks have been completed. 
    - We represent the constraints as connections between the tasks (items), and we are faced with a classical scheduling problem: 
    - How do we schedule the tasks such that we both respect the given constraints and complete the whole process in the least amount of time?
- **Transactions** 
    - A telephone company maintains a database of telephone-call traffic.
    - Here the connections represent telephone calls. 
    - We are interested in knowing about the nature of the interconnection structure because we want to lay wires and build switches that can handle the traffic efficiently. 
    - As another example, a financial institution tracks buy/sell orders in a market. 
    - A connection in this situation represents the transfer of cash between two customers. 
    - Knowledge of the nature of the connection structure in this instance may enhance our understanding of the nature of the market.
- **Matching** 
    - Students apply for positions in selective institutions such as social clubs, universities, or medical schools. 
    - Items correspond to the students and the institutions; connections correspond to the applications. 
    - We want to discover methods for matching interested students with available positions.
    - Networks A computer network consists of interconnected sites that send, forward, and receive messages of various types. 
    - We are interested not just in knowing that it is possible to get a message from every site to every other site but also in maintaining this connectivity for all pairs of sites as the network changes. 
    - For example, we might wish to check a given network to be sure that no small set of sites or connections is so critical that losing it would disconnect any remaining pair of sites.
- **Program structure** 
    - A compiler builds graphs to represent relationships among modules in a large software system. 
    - The items are the various classes or modules that comprise the system; 
    - connections are associated either with the possibility that a method in one class might invoke another (static analysis) 
    - or with actual invocations while the system is in operation (dynamic analysis). 
    - We need to analyze the graph to determine how best to allocate resources to the program most efficiently. 

## Fundamental 
- A graph is a set of vertices and a set of edges that connect pairs of distinct vertices 
    - (with at most one edge connecting any pair of vertices).
    - A graph with V vertices has at most V (V – 1)/2 edges.
        - Proof: The total of V2 possible pairs of vertices includes V self-loops and accounts twice for each edge between distinct vertices, 
        - so the number of edges is at most (V2 – V)/2 = V(V – 1)/2.   
- A subgraph is a subset of a graph's edges (and associated vertices) that constitutes a graph.

### 
- A path in a graph is a sequence of vertices in which each successive vertex (after the first) is adjacent to its predecessor in the path. 
    - In a simple path, the vertices and edges are distinct. 
    - A cycle is a path that is simple except that the first and final vertices are the same.
- A graph is a connected graph if there is a path from every vertex to every other vertex in the graph. 
    - A graph that is not connected consists of a set of connected components, which are maximal connected subgraphs. 
- An acyclic connected graph is called a tree. 
    - A set of trees is called a forest. A spanning tree of a connected graph is a subgraph that contains all of that graph's vertices and is a single tree. 
    - A spanning forest of a graph is a subgraph that contains all of that graph's vertices and is a forest.            