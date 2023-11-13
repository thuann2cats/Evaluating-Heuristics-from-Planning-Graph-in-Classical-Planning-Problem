# Experiment: Planning Graph in the Air Cargo problem

In this experiment, which was an assignment from Udacity on classical planning and Planning Domain Definition Language (PDDL), I implemented the missing parts of `my_planning_graph` and the heuristic functions `maxlevel`, `levelsum` and `setlevel`. 

The planning graph and the heuristics are used by various search algorithms in a Python automated planning program, which reads in 4 instances of the Air Cargo planning problem and runs 11 search algorithms to compare runtime. 

The results, in terms of actual run time or the number of nodes expanded or added to frontier, for each type of heuristic, as well as my analysis, are displayed in the tables below.

The Air Cargo problems are as described in "_Artificial Intelligence: A Modern Approach_" 3rd edition by Stuart Russell and Peter Norvig, Chapter 10.

The experiment can be re-run manually by running:
```
$ python run_search.py -m
```
Original instructions from the Udacity assignment can be found here: [instructions.md](instructions.md)

## Analysis

Here are my main observations:

+ Across the 4 air cargo planning problems at all levels of complexity, the `greedy_best_first_search` algorithm with `h_unmet_goals` heuristic (the number of literals in the goal that still haven't been met) seems to be the best.


+ `greedy_best_first_search` algorithm `h_unmet_goals` heuristic runs much faster than the same algorithm with the remaining three heuristics, which rely on planning graph (`levelsum`, `maxlevel` and `setlevel`). That is because planning graph needs to be constructed each time a heuristic needs to be determined.


+ In these problem instances, `greey_best_first_search` consistently outperforms `a_star` (if using the same heuristic) by x10-100 times. This is probably due to the fact that in order to improve runtime, A* heuristics need to closely approximate the true cost to goal. Maxlavel, levelsum and setlevel all seem to be quite imperfect (p382 Artificial Intelligence: A Modern Approach 3rd edition by Stuart Russell and Peter Norvig). 
    + For example, when I look into the `maxlevel` heuristic value during the A* run, that heuristic only takes the values of 2 or 3 for most of the time - those values makes sense as the answer to "what's the maximum number of steps to satisfy any literal in the goal right now?" - but a heuristic that has roughly the same values for every node is hardly useful for A* to direct the search.


+ Among the heuristics derived from planning graph, `levelsum` consistently outperform others. This is consistent with Stuart Russell and Peter Norvig's observation that this heuristic can be inadmissible, but is more accurate and works well in practice. 
    + My guess is that in these instances of cargo problems, goals seems to be quite independent. Moving cargo C1 to airport A1 as desired is unlikely to satisfy other goals such as C2 at A2, C3 at A3, and so the number of remaining steps to final victory seems to be close to the number of steps to finish each goal one by one.


+ If `levelsum` can be a good approximate of true cost remaining to goal, why does `a_star` with `levelsum` heuristic is slower than `greedy_best_first_search` with the same heuristic? I think that is because `a_star` looks at both the heuristic as well as the "cost so far" from initial state when choosing which node to expand next (which node has minimum f(n) + h(n)), and thus expand far more nodes.
    + In other words, while `greedy_best_first_search` only cares "how far I am from the final goal," and tries to "get to work" immediately to jump to nodes that are closer to goal - "just get to the goals and get it over with!", then `a_star` seems to be overthinking - "need to find the best plan"!

## Results (by search algorithms and heuristics)

### Air Cargo Problem 1 

Initial state: [At(C1, SFO), At(C2, JFK), At(P1, SFO), At(P2, JFK)]

Goal: [At(C1, JFK), At(C2, SFO)]

| **Air Cargo Problem 1**                                  | # actions | # nodes expanded | # nodes in frontier | Runtime (sec) | Plan length |
|----------------------------------------------------------|-----------|------------------|---------------------|---------------|-------------|
| breadth_first_search                                     | 20        | 43               | 178                 | 0.0168        | 6           |
| depth_first_graph_search                                 | 20        | 21               | 84                  | 0.0083        | 20          |
| uniform_cost_search                                      | 20        | 60               | 240                 | 0.0387        | 6           |
| greedy_best_first_graph_search (Heuristic: h_unmet_goals) | 20        | 7                | 29                  | 0.0026        | 6           |
| greedy_best_first_graph_search (Heuristic: h_pg_levelsum) | 20        | 6                | 28                  | 0.4880        | 6           |
| greedy_best_first_graph_search (Heuristic: h_pg_maxlevel) | 20        | 6                | 24                  | 0.3317        | 6           |
| greedy_best_first_graph_search (Heuristic: h_pg_setlevel) | 20        | 6                | 28                  | 0.5122        | 6           |
| astar_search (Heuristic: h_unmet_goals)                  | 20        | 50               | 206                 | 0.0534        | 6           |
| astar_search (Heuristic: h_pg_levelsum)                  | 20        | 28               | 122                 | 0.4184        | 6           |
| astar_search (Heuristic: h_pg_maxlevel)                  | 20        | 43               | 180                 | 0.8913        | 6           |
| astar_search (Heuristic: h_pg_setlevel)                  | 20        | 33               | 138                 | 2.6833        | 6           |


### Air Cargo Problem 2 

Initial state: [At(C1, SFO), At(C2, JFK), At(C3, ATL), At(P1, SFO), At(P2, JFK), At(P3, ATL)]

Goal: [At(C1, JFK), At(C2, SFO), At(C3, SFO)]

| **Air Cargo Problem 2**                                   | # actions | # nodes expanded | # nodes in frontier | Runtime (sec) | Plan length |
|-----------------------------------------------------------|-----------|------------------|---------------------|---------------|-------------|
| breadth_first_search                                      | 72        | 3343             | 30503               | 4.0495        | 9           |
| depth_first_graph_search                                  | 72        | 624              | 5602                | 5.7078        | 619         |
| uniform_cost_search                                       | 72        | 5154             | 46618               | 4.7294        | 9           |
| greedy_best_first_graph_search (Heuristic: h_unmet_goals) | 72        | 17               | 170                 | 0.0406        | 9           |
| greedy_best_first_graph_search (Heuristic: h_pg_levelsum) | 72        | 9                | 86                  | 4.5977        | 9           |
| greedy_best_first_graph_search (Heuristic: h_pg_maxlevel) | 72        | 27               | 249                 | 7.3950        | 9           |
| greedy_best_first_graph_search (Heuristic: h_pg_setlevel) | 72        | 9                | 84                  | 21.7078       | 9           |
| astar_search (Heuristic: h_unmet_goals)                   | 72        | 2467             | 22522               | 3.1290        | 9           |
| astar_search (Heuristic: h_pg_levelsum)                   | 72        | 357              | 3426                | 155.5152      | 9           |
| astar_search (Heuristic: h_pg_maxlevel)                   | 72        | 2887             | 26594               | 1070.9344     | 9           |
| astar_search (Heuristic: h_pg_setlevel)                   | 72        | 1037             | 9605                | 1739.8779     | 9           |

### Air Cargo Problem 3

Initial state: [At(C1, SFO), At(C2, JFK), At(C3, ATL), At(C4, ORD), At(P1, SFO), At(P2, JFK)]

Goal: [At(C1, JFK), At(C2, SFO), At(C3, JFK), At(C4, SFO)]

| **Air Cargo Problem 3**                                   | # actions | # nodes expanded | # nodes in frontier | Runtime (sec) | Plan length |
|-----------------------------------------------------------|-----------|------------------|---------------------|---------------|-------------|
| breadth_first_search                                      | 88        | 14663            | 129625              | 15.1333       | 12          |
| depth_first_graph_search                                  | 88        | 408              | 3364                | 1.5849        | 392         |
| uniform_cost_search                                       | 88        | 18510            | 161936              | 20.8790       | 12          |
| greedy_best_first_graph_search (Heuristic: h_unmet_goals) | 88        | 25               | 230                 | 0.0686        | 15          |
| greedy_best_first_graph_search (Heuristic: h_pg_levelsum) | 88        | 14               | 126                 | 13.3530       | 14          |
| greedy_best_first_graph_search (Heuristic: h_pg_maxlevel) | 88        | 21               | 195                 | 12.1255       | 13          |
| greedy_best_first_graph_search (Heuristic: h_pg_setlevel) | 88        | 35               | 345                 | 132.1739      | 17          |
| astar_search (Heuristic: h_unmet_goals)                   | 88        | 7388             | 65711               | 18.8679       | 12          |
| astar_search (Heuristic: h_pg_levelsum)                   | 88        | 369              | 3403                | 465.4644      | 12          |
| astar_search (Heuristic: h_pg_maxlevel)                   | 88        | 9580             | 86312               | 4705.1904     | 12          |
| astar_search (Heuristic: h_pg_setlevel)                   | 88        | didn't terminate | didn't terminate    | n/a           | n/a         |


### Air Cargo Problem 4

Initial state: [At(C1, SFO), At(C2, JFK), At(C3, ATL), At(C4, ORD), At(C5, ORD), At(P1, SFO), At(P2, JFK)]

Goal: [At(C1, JFK), At(C2, SFO), At(C3, JFK), At(C4, SFO), At(C5, JFK)]

| **Air Cargo Problem 4**                                   | # actions | # nodes expanded | # nodes in frontier | Runtime (sec) | Plan length |
|-----------------------------------------------------------|-----------|------------------|---------------------|---------------|-------------|
| breadth_first_search                                      | 104       | 99736            | 944130              | 135.2811      | 14          |
| depth_first_graph_search                                  | 104       | 25174            | 228849              | 4781.5620     | 24132       |
| uniform_cost_search                                       | 104       | 113339           | 1066413             | 134.9041      | 14          |
| greedy_best_first_graph_search (Heuristic: h_unmet_goals) | 104       | 29               | 280                 | 0.0951        | 18          |
| greedy_best_first_graph_search (Heuristic: h_pg_levelsum) | 104       | 17               | 165                 | 18.3278       | 17          |
| greedy_best_first_graph_search (Heuristic: h_pg_maxlevel) | 104       | 56               | 580                 | 29.3829       | 17          |
| greedy_best_first_graph_search (Heuristic: h_pg_setlevel) | 104       | 107              | 1164                | 491.0615      | 23          |
| astar_search (Heuristic: h_unmet_goals)                   | 104       | 34330            | 328509              | 58.2571       | 14          |
| astar_search (Heuristic: h_pg_levelsum)                   | 104       | 1208             | 12210               | 1164.0671     | 15          |
| astar_search (Heuristic: h_pg_maxlevel)                   | 104       | didn't terminate | didn't terminate    | n/a           | n/a         |
| astar_search (Heuristic: h_pg_setlevel)                   | 104       | didn't terminate | didn't terminate    | n/a           | n/a         |
