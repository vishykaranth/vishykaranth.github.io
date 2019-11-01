---
layout: page
title: Dynamic Programming  
permalink: /google-ds-dp/
---


## Dynamic programming
- Dynamic programming, like the divide-and-conquer method, solves problems by combining the solutions to subproblems. 
    - “Programming” in this context refers to a tabular method, not to writing computer code. 
- Divide-and-conquer algorithms partition the problem into disjoint subproblems, solve the subproblems recursively, and then combine their solutions to solve the original problem. 
In contrast, dynamic programming applies when the subproblems overlap—that is, when subproblems share subsubproblems. 
- In this context, a divide-and-conquer algorithm does more work than necessary, repeatedly solving the common subsubproblems. 
- A dynamic-programming algorithm 
    - solves each subsubproblem just once 
    - and then saves its answer in a table, 
    - thereby avoiding the work of recomputing the answer every time it solves each subsubproblem.
- We typically apply dynamic programming to optimization problems. 
- Such problems can have many possible solutions. Each solution has a value, 
    - and we wish to find a solution with the optimal (minimum or maximum) value. 
    - We call such a solution an optimal solution to the problem, 
    - as opposed to the optimal solution, since there may be several solutions that achieve the optimal value.
- When developing a dynamic-programming algorithm, we follow a sequence of four steps:
    - Characterize the structure of an optimal solution.
    - Recursively define the value of an optimal solution.
    - Compute the value of an optimal solution, typically in a bottom-up fashion.
    - Construct an optimal solution from computed information.

### Optimal substructure 
- A problem is said to have optimal substructure if an optimal solution can be constructed efficiently from optimal solutions of its subproblems. 
- This property is used to determine the usefulness of dynamic programming and greedy algorithms for a problem.
- Typically, a greedy algorithm is used to solve a problem with optimal substructure. 
- Otherwise, provided the problem exhibits overlapping subproblems as well, dynamic programming is used.
- Example : 
    - Consider finding a **shortest path** for travelling between two cities by car. 
    - Such an example is likely to exhibit optimal substructure. 
    - That is, if the shortest route from Seattle to Los Angeles passes through Portland and then Sacramento, 
        - then the shortest route from Portland to Los Angeles must pass through Sacramento too. 
        - That is, the problem of how to get from Portland to Los Angeles is nested inside the problem of how to get from Seattle to Los Angeles.

### Overlapping subproblems
- A problem is said to have overlapping subproblems if the problem can be broken down into subproblems which are reused several times or a recursive algorithm for the problem solves the same subproblem over and over rather than always generating new subproblems.
- For example, 
    - the problem of computing the **Fibonacci** sequence exhibits overlapping subproblems. 
    - The problem of computing the nth Fibonacci number F(n), 
        - can be broken down into the subproblems of computing F(n − 1) and F(n − 2), and then adding the two. 
        - The subproblem of computing F(n − 1) can itself be broken down into a subproblem that involves computing F(n − 2). 
        - Therefore the computation of F(n − 2) is reused, and the Fibonacci sequence thus exhibits overlapping subproblems.