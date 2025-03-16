---
title: Constrained Multi-Vehicle Routing Algorithm
lang: en
layout: single
author_profile: true
permalink: /en/projects/vrp-algo/
classes: wide
github:
---

# Constrained Multi-Vehicle Routing Algorithm

A heuristic algorithm for computing multi-vehicle package delivery routes. {% if page.github %} <a href="{{ page.github }}">View on GitHub</a> {% endif %}

## Algorithm Visualization

{% include youtube_video.html id="Uj8KTY3-qD4" %}

**Input:** 3 depots, 2205 packages, 37 vehicles with a total capacity of 1805.

**Output:** 37 routes, a total of 1805 packages delivered.

## Key Highlights

- Won 2nd place (out of 103 teams) in the 2022 CJ Logistics Future Technology Challenge.
- Achieved the maximum possible delivery count on the competition dataset (37 vehicles, 1,805 packages) using only 73.5% of the vehicle resources and satisfying constraints on vehicle capacity, travel distances, and per-package delivery time windows.
- Decreased runtime by 20% using multi-threading, while maintaining deterministic results.

## Objective

The goal of the algorithm is to produce package delivery routes for all vehicles while satisfying the constraints below.

### Vehicle Constraints

- Fixed start and end depot, where start depot $ \neq $ end depot.
- Vehicle-specific capacity (given as an integer, e.g. 50).
- Vehicle-specific working hours (e.g. 6/20/2022 00:00 ~ 6/20/2022 05:00).
- Maximum daily travel distance (110km).
- Fixed traveling speed (40km/h).

### Package Constraints

- Package-specific volume (same unit as vehicle capacity, given as an integer).
- Package-specific drop-off time window (e.g. 6/20/2022 01:00 ~ 6/20/2022 02:00).
- Package-specific unloading-from-vehicle time (e.g. 2 min.).
- Some packages require a specific vehicle (e.g. package A00012 can only be delivered by vehicle #11).
- Some packages can only be delivered by 'small' trucks. Each vehicle is classified as either 'big' or 'small'.

## Methodology

The algorithm consists of two stages: *insertion heuristic* and *local search*.

First, the insertion heuristic constructs tentative routes that satisfy all vehicle and package constraints. Then, local search iteratively improves the routes by making small intra/inter-route modifications.

### Insertion Heuristic

The insertion heuristic consists of three steps: *seed selection*, *seed insertion*, and *iterative insertion*.

<div class="side-by-side">
  <div style="width: 65%" class="responsive-width">
    <p>
      <b>Seed selection:</b> The first packages inserted into each route are called 'seeds'. Seed selection is an important step because the insertion heuristic's results can vary greatly by how these seeds are selected.
    </p>
    <p>
      This algorithm follows Savelsbergh's<a href="#references">[1]</a> method to select seeds based on the distribution density of the packages' drop-off locations.
    </p>
  </div>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/seed_selection.png"
      alt="Seed selection.">
    <figcaption>Seeds selected based on distribution density.</figcaption>
  </figure>
</div>

<div class="side-by-side">
  <div style="width: 65%" class="responsive-width">
    <p>
      <b>Seed insertion:</b> Seeds are inserted into each route using the "look-ahead heuristic" (a.k.a. 'regret')<a href="#references">[1]</a>. Regret stands for the cost difference between the best and second-best choices. Prioritizing the insertion of packages with the biggest regret can be interpreted as trying to minimize the potential future loss when the package's best insertion choice is no longer available.
    </p>
    <p>
      The cost function used to calculate the best choice and second-best choice is simply <em>(distance from start depot to package drop-off site)</em> + <em>(distance from package drop-off site to end depot)</em>.
    </p>
  </div>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/seed_insertion.png"
      alt="Seed insertion.">
    <figcaption>Seeds inserted into each route.</figcaption>
  </figure>
</div>

**Iterative insertion:** The rest of the drop-off sites are inserted into the routes using a modified version of Solomon’s[[2]](#references) time-window-based insertion heuristic. The modified formulas are as below.

$$ c_1 = b_i' - b_i $$

$$ c_2 = (t_{start} + t_{unload} + t_{end} - c_1 - 2t_{centroid}) \times (p / q) $$

<details>
  <summary>Explanations on each symbol. (Click Me)</summary>
  <p> Given that a new package $u$ gets inserted into route $r$ right before package $i$ </p>
  <ul>
    <li> $b_i'$: delayed drop-off time for package $i$ after inserting package $u$ </li>
    <li> $b_i$: original drop-off time for package $i$ before inserting package $u$ </li>
    <li> $t_{start}$: travel time from start depot to package $u$ drop-off site</li>
    <li> $t_{unload}$: unloading time required by package $u$</li>
    <li> $t_{end}$: travel time from package $u$ drop-off site to end depot</li>
    <li> $t_{centroid}$ travel time from package $u$ drop-off site to route $r$'s centroid</li>
    <li> $p$: profit of package $u$</li>
    <li> $q$: volume of package $u$</li>
  </ul>
</details>

$c_1$ can be interpreted as the time cost required to add the new package to the route.

$c_2$ can be interpreted as the time cost saved by not having to deliver the new package separately in a new vehicle.

Of all the unrouted packages, the package with the largest $c_2$ should be prioritized, and that package should be inserted at the place that minimizes its $c_1$.

The formula for $c_2$ includes profit/volume ratio $p/q$ because the algorithm should prioritize packages with a higher profit ratio.

The use of a centroid-based penalty $t_{centroid}$ is inspired by Golden[[3]](#references). The centroids for each route are updated after every insertion.

The centroid of a route is calculated by averaging the coordinates of the route's depots and drop-off sites, weighted by their profit/volume ratio. Depots are assumed to have the average profit and volume of all packages in the dataset.

### Local Search

Local search iteratively improves the routes by making small intra/inter-route modifications.

Inspired by Aras[[4]](#references), Mcnabb[[5]](#references), and Vansteenwegen[[6]](#references), the local search operators used in this algorithm are *Replacement, Deletion, 1-0 Move, Chain Swap, 1-1 Move, Insertion,* and *Or-Opt*. Detailed descriptions of each operator can be found in <A href="#appendix-a-local-search-operators">Appendix A</A>.

The local search continues until it fails to make any improvement for *max_patience* number of loops around all the operators, where *max_patience* is a hyper-parameter.

Multi-threading is applied to a few of the local search operators to speed up the search while maintaining deterministic results. See <A href="#appendix-b-multi-threading-with-deterministic-results">Appendix B</A> for details.

## Results

Below are the results of the algorithm when tested on the dataset from 2022 CJ Logistics Future Technology Challenge. The dataset contains 2200 packages and 37 vehicles with a total capacity of 1805.

<div class="side-by-side" >
  <div style="width: 50%" class="responsive-width">
    <b> Result after insertion heuristic (<span data-figure-ref="before_local_search"></span>):</b>
    <ul>
      <li> Total number of delivered payloads: 1748 </li>
      <li> Total time cost: 136h 45m 05s </li>
      <li> Total distance cost: 3132.8km </li>
      <li> Algorithm runtime: 11.0s </li>
    </ul>
  </div>
  <div style="width: 50%" class="responsive-width">
    <b> Result after local search (<span data-figure-ref="after_local_search"></span>):</b>
    <ul>
      <li> Total number of delivered payloads: 1805 </li>
      <li> Total time cost: 133h 03m 05s </li>
      <li> Total distance cost: 2921.3km </li>
      <li> Algorithm runtime: 2m 00s </li>
    </ul>
  </div>
</div>

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width" id="before_local_search">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/centroid_before_local_search.png"
      alt="Routes produced by insertion heuristic.">
    <figcaption>Routes after insertion heuristic is complete.</figcaption>
  </figure>
  <figure style="width: 50%" class="responsive-width" id="after_local_search">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/centroid_after_local_search.png"
      alt="Routes produced by local search.">
    <figcaption>Routes after local search is complete.</figcaption>
  </figure>
</div>

## Limitations and Future Work

- Given $n$ packages, the algorithm has a time complexity of $O(n^3)$, making it inefficient for large datasets. For scalability, large datasets should be split into smaller, mutually exclusive subsets of packages and vehicles, and the algorithm should be applied separately to each subset. Future work should explore how such splitting can be done based on geographical regions and random sampling.
- Due to the competition's time limit (1.5 months), the algorithm has only been tested on small variations of the competition dataset. Future work should evaluate the algorithm on a broader set of randomized datasets alongside alternative designs to better justify its design choices.

## References {#references}

[1] SAVELSBERGH, M. W. P. A parallel insertion heuristic for vehicle routing with side constraints. *Statistica Neerlandica*, 1990, 44.3: 139-148.

[2] SOLOMON, Marius M. Algorithms for the vehicle routing and scheduling problems with time window constraints. *Operations research*, 1987, 35.2: 254-265.

[3] GOLDEN, Bruce L.; LEVY, Larry; VOHRA, Rakesh. The orienteering problem. *Naval Research Logistics (NRL)*, 1987, 34.3: 307-318.

[4] ARAS, Necati; AKSEN, Deniz; TEKIN, Mehmet Tuğrul. Selective multi-depot vehicle routing problem with pricing. *Transportation Research Part C: Emerging Technologies*, 2011, 19.5: 866-884.

[5] MCNABB, Marcus E., et al. Testing local search move operators on the vehicle routing problem with split deliveries and time windows. *Computers & Operations Research*, 2015, 56: 93-109.

[6] VANSTEENWEGEN, Pieter, et al. A guided local search metaheuristic for the team orienteering problem. *European journal of operational research*, 2009, 196.1: 118-127.

## Appendix A: Local Search Operators

The local search loop consists of *Replacement, Deletion, 1-0 Move, Chain Swap, 1-1 Move, Insertion,* and *Or-Opt*. With the exception of the deletion operator, all operators are only performed if they improve the profit and/or efficiency of the routes.

**Replacement**: Replace a routed package with an unrouted package.

**Deletion**: From each route, remove at most three packages with the lowest profit/volume ratio, largest distance cost, and longest wait time. By removing the worst packages, this operator creates leeway for packages to move around between routes.

**1-0 Move**: Remove a package from one route and insert it into another route.

<div class="side-by-side">
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/1_0_Move_before.jpg"
      alt="Before performing 1-0 Move.">
    <figcaption>Before performing 1-0 Move.</figcaption>
  </figure>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/1_0_Move_after.jpg"
      alt="After performing 1-0 Move.">
    <figcaption>After performing 1-0 Move.</figcaption>
  </figure>
</div>

**Chain Swap**: Given that routes $ r_x $ and $ r_y $ share the same end depot, swap all packages in $ r_x $ starting from its $i^{th}$ package with all packages of $ r_y $ starting from its $j^{th}$ package.

<div class="side-by-side">
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/Chain_Swap_before.jpg"
      alt="Before performing Chain Swap.">
    <figcaption>Before performing Chain Swap.</figcaption>
  </figure>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/Chain_Swap_after.jpg"
      alt="After performing Chain Swap.">
    <figcaption>After performing Chain Swap.</figcaption>
  </figure>
</div>

**1-1 Move**: Exchange a package from one route with a package from another route.

<div class="side-by-side">
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/1_1_Move_before.jpg"
      alt="Before performing 1-1 Move.">
    <figcaption>Before performing 1-1 Move.</figcaption>
  </figure>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/1_1_Move_after.jpg"
      alt="After performing 1-1 Move.">
    <figcaption>After performing 1-1 Move.</figcaption>
  </figure>
</div>

**Insertion**: Insert unrouted packages into routes by using the insertion heuristic. Ignore centroid-based penalty.

**Or-Opt**: swap $i^{th}$ to $(i+n)^{th}$ packages of route $ r_x $ with $j^{th}$ to $(j+m)^{th}$ packages of route $ r_y $.

<div class="side-by-side">
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/Or_Opt_before.jpg"
      alt="Before performing Or-Opt.">
    <figcaption>Before performing Or-Opt.</figcaption>
  </figure>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/Or_Opt_after.jpg"
      alt="After performing Or-Opt.">
    <figcaption>After performing Or-Opt.</figcaption>
  </figure>
</div>

## Appendix B: Multi-threading with Deterministic Results

Inter-route local search operators (e.g. 1-0 Move, 1-1 Move, and Or-Opt) move and swap packages between two routes. These operators all have a double-nested loop structure shown below.

```
for (int i = 0; i < routes.size(); ++i) {
	for (int j = i+1; j < routes.size(); ++j) {
		performLocalSearchOperator(routes[i], routes[j]);
	}
}
```

Given 6 routes $r_0$ through $r_5$, the above double-nested loop can be illustrated as <span data-figure-ref="single_thread"></span>. The number in each cell indicates the order in which a single-thread single-thread program would execute the local search operators on these routes.

<div class="side-by-side">
  <div style="width: 65%" class="responsive-width">
    <p>
      From <span data-figure-ref="single_thread"></span>, it can be seen that <b>each cell is dependent on the cell to its left</b>. For example, the result of cell 2 (performing the local search between $r_0$ and $r_3$) depends on the result of cell 1 (which performs local search between $r_0$ and $r_2$). This is because both cells share the same $r_0$. Likewise, we can see that <b>each cell is dependent on the cell above</b>. For example, the result of cell 10 (performing the local search between $r_2$ and $r_4$) depends on the result of cell 7 (which performs local search between $r_1$ and $r_4$), because both cells share the same $r_4$.
    </p>
  </div>
  <figure style="width: 35%" class="responsive-width" id="single_thread">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/single_thread.png"
      alt="Single-thread order of execution.">
    <figcaption>Single-thread order of execution.</figcaption>
  </figure>
</div>

<div class="side-by-side">
  <div style="width: 65%" class="responsive-width">
    <p>
      To get deterministic results, the multi-thread execution must maintain the same dependencies as the single-thread execution. <span data-figure-ref="multi_thread"></span> illustrates how multiple threads can split the workload while maintaining such dependencies. The number in each cell represents <b>(thread id)-(execution order within the thread)</b> and the threads that are executed simultaneously are given the same color. Each thread begins when the thread to its left and above has both completed executing.
    </p>
    <p>
      Hence, similar to an assembly pipeline, when thread 0 finishes executing, threads 1 and 2 begin simultaneously. When threads 1 and 2 both finish executing, then threads 3 and 4 begin simultaneously. Given a larger number of routes, this pattern of execution can be applied using a larger number of threads to improve efficiency.
    </p>
  </div>
  <figure style="width: 35%" class="responsive-width" id="multi_thread">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/vrp-algo/multi_thread.png"
      alt="Multi-thread order of execution.">
    <figcaption>Multi-thread order of execution.</figcaption>
  </figure>
</div>


<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/en/projects/">← Back to 'Projects'</a>
