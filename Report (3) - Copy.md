# Navmap Reconstruction
## *A solution for smuggler Spike's problem.*  
*Jaco schalij - s4356357*  
*Timo van Niedek - s4326164*  

### Basic Idea ###
To reconstruct the NavMap, we first take the shortest path matrix, and extract all possible wormholes. The amount of possible wormholes is *(N(N-1))/2.* Now we draw a graph with all the possible wormholes as vertices, and all the stars as nodes.  
Since we need a set of *N* vertices/wormholes, we need to get rid of a few redundant wormholes. A wormhole is redundant when there is a different path formed by two or more wormholes from *endpoint _ a* to *endpoint _ b* with the same *traveltime* as the original wormhole.  
To get rid of these redundant wormholes, we start by sorting the array of all possible wormholes from largest *traveltime* to smallest. Then we start with the wormhole with the largest *traveltime* (which is at the front of the array), and check if there is a different star *C* for which holds:    
  
*endpoint _ a --> C + C --> endpoint _ b = endpoint _ a --> endpoint _ b*
      
If so, the wormhole is redundant and can be removed from the array. Because we start with the wormhole with the largest traveltime, we only have to check if there is a way to make a path of two wormholes, not three or more. We repeat this for all wormholes, until the array size is equal to N. Now we have a result array of size *N*, with all the wormholes 
that are part of the NavMap.

### Code Explanation ###

The first step of our algorithm is to fill an `ArrayList` with all the possible wormholes. This is done by looping through half of the `travelTimes` matrix (because the wormholes form a undirected graph we only have to check one half of the matrix). This is accomplised in our `fillWormholes()` function.

    // For every distance in the travelTimes matrix, create a new wormhole.
	for (int i = 0; i < starCount; i++) {
		for (int j = i+1; j < starCount; j++){
			Wormhole wormhole = new Wormhole(i, j, travelTimes[i][j]);
			wormholes.add(wormhole);
		}
	}

The next step is to sort the `wormholes` from highest travel time to smallest. We made a custom Comparator `WormholeComparator` in order to be able to use the `Collections.sort` function. After the `wormholes` are sorted, we have to remove the redundant edges. This is accomplished in `removeDoubleEdges()`.

    Iterator<Wormhole> itr = wormholes.iterator();
	// For every wormhole...
	while (itr.hasNext()){
		// If there are N wormholes, stop removing them
		if (wormholes.size() == starCount) break;
		
		Wormhole wormhole = itr.next();
		// For every star..
		for (int i = 0; i < starCount; i++) {
			// .. that is not an endpoint of the wormhole ..
			if (i != wormhole.endpoint_a && i != wormhole.endpoint_b){
				// .. check if it forms a path from endpoint a to the star to endpoint b ..
				if (travelTimes[wormhole.endpoint_a][i] + 
					travelTimes[i][wormhole.endpoint_b] == wormhole.traveltime){
					// .. and if so, remove the wormhole.
					itr.remove();
					
					break;
				}
			}
		}
	}

We needed an `Iterator` to loop through the `wormholes`. For every wormhole, we check for every star that is not an endpoint of the wormhole if it satisfies:  
  
*endpoint _ a --> C + C --> endpoint _ b = endpoint _ a --> endpoint _ b* 
  
If this is the case, the wormhole can be removed because it is redundant. Because the `wormholes` are sorted from longest traveltime to smallest, the loop will converge to the set of smallest possible paths. The loop stops if there are no more redundant wormholes, or the amount of wormholes left is equal to the amout of stars. The resulting wormholes ArrayList is converted to a `NavMap` and returned as a solution. 
  
  






### Example ###

As example, we take the following shortest path table:  
  
| Stars | A | U | N | E | M |
|---    |---|---|---|---|---|
|**A**  | 0 | 3 | 5 | 5 | 11|
|**U**  | 3 | 0 | 2 | 2 | 8 |
|**N**  | 5 | 2 | 0 | 1 | 7 |
|**E**  | 5 | 2 | 1 | 0 | 6 |
|**M**  |11 | 8 | 7 | 6 | 0 |
  
This matrix corresponds to the following NavMap:  
![Example1](http://cl.ly/YPfv/Graph1.jpg)

First, we draw the graph with all *(N(N-1))/2* vertices:  
![Example2](http://cl.ly/YPib/Graph2.jpg)

The sorted array with all the possible wormholes looks like this:  
  
`{A-->M, U-->M, N-->M, E-->M, A-->N, A-->E, A-->U, U-->N, U-->E, E-->N}`  
  
We start to check the wormhole with the largest traveltime, which is *A-->M*:  
*A-->E + E-->M = 5+6 = 11 = A-->M*.  
The wormhole from A to M is redundant and can be removed.  
In the same manner we can remove wormholes *U-->M* and *N-->M*. Now we have a graph that looks like this:  
![Example3](http://cl.ly/YPu3/Graph3.jpg)  
Then we check wormhole *E-->M* which has the largest travel time of the remaining wormholes.
We can see that there is no point X for which holds:
*E-->X + X-->M = E-->M*.
This wormhole is not redundant and cannot be removed.  
In the same manner as the first three removed wormholes, we can remove both wormhole *A-->N* and wormhole *A-->E*.  
This leaves us with wormholes *E-->M, A-->U, U-->N, U-->E* and *E-->N*. This is a correct solution to the problem, with the number of wormholes equal to the number of stars.
  
  
  
### Complexity Analysis ###

We will analyse the complexity of the algorithm by analyzing each function individually.  
The time function of `fillWormholes()` is *T(N) = (N(N-1))/2* because it contains a nested for-loop which loops through half of the `travelTimes` matrix (not including the zeroes on the diagonal), which is size *N\*N*. Therefore the complexity class of `fillWormholes()` is *O(N^2)*.  
The complexity class of the line  `Collections.sort(wormholes, new WormholeComparator());`  is *O(N log(N))* because it uses a modifier version of the merge-sort algorithm.  
The time function of `removeDoubleEdges()` is *T(N) = N((N(N-1))/2)*, which is based on the number of stars times maximum number of edges. Therefore, the complexity of `removeDoubleEdges()` is in *O(N^3)*.  
Converting the resulting edges into a NavMap is in complexity class *O(N)*.  
Therefore, the function with the highest complexity is `removeDoubleEdges()`, which determines the complexity of the entire algorithm. The algorithm is in complexity class *O(N^3)*.
