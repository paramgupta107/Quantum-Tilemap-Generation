# Quantum Tilemap Genration

This Project generates tilemaps using quantum computing. It uses a set of sample tiles and tilemaps and generates new tilemaps that are similar. The generation algorithm is based on the Wavefunction Collapse Algorithm.

First, we represent our problem as a quadratic unconstrained binary optimization (QUBO) using a sample set of tilemaps. The QUBO is then solved on D-Wave's Quantum Annealer using Ocean SDK

# Objective Function
The objective function is a function of binary variables that represent qubits. We will represent our objective function as QUBO which would be used to generate new tilemaps.

## Objective Function Generation

Let **numMaps** be the number of sample maps.

Let **numTiles** be the number of tiles.

Let **numRows** be the number of rows in a tilemap.

Let **numColumns** be the number of columns in a tilemap.

Let,

$$M_{r,c,t}^{i} \in \{0,1\}$$

Where,

the value is 1 if tile **t** is present at row **r** and column **c** in sample map **i**.

$$AllowedTop = \{(t_{1},t_{2}) | M_{r,c,t_{1}}^{i}=1, M_{r-1,c,t_{2}}^{i}=0 \newline \forall 0 \leq i \leq  numMaps,
1 \leq r \leq  numRows,
0 \leq c \leq  numColumns,
0 \leq t \leq  numTiles\}$$

**AllowedTop** is the set of all pairs of tiles where 2nd tile is allowed to be on top of the first tile

$$AllowedLeft = \{(t_{1},t_{2}) | M_{r,c,t_{1}}^{i}=1, M_{r,c-1,t_{2}}^{i}=0\newline \forall 0 \leq i \leq  numMaps,
0 \leq r \leq  numRows,
1 \leq c \leq  numColumns,
0 \leq t \leq  numTiles\}$$

**AllowedLeft** is the set of all pairs of tiles where 2nd tile is allowed to be on the left of the first tile

$$AllCombinations = \{(t_{1},t_{2})|0\leq t_{1}\leq numTiles,0\leq t_{2}\leq numTiles\}$$

**AllCombinations** is the set of all possible pairs of tiles.

$$NotAllowedTop = AllCombinations-AllowedTop$$

**NotAllowedTop** is the set of all pairs of tiles where 2nd tile is not allowed to be on top of the first tile.

$$NotAllowedLeft = AllCombinations-AllowedLeft$$

**NotAllowedLeft** is the set of all pairs of tiles where 2nd tile is not allowed to be on the left of the first tile.

$$countPerTile = \left \lfloor \frac{numColumns*numRows}{numTiles}\right \rceil$$

**countPerTile** is the desired count of each tile in our generated tilemap.

The Objective Function can then be defined as:-

$$E(M) = \alpha \bigg(\sum_{c=0}^{numColumns-1}\sum_{r=1}^{numRows-1}\sum \{(M_{r,c,t_{1}}+M_{r-1,c,t_{2}}-1)^{2}|\forall (t_{1},t_{2}) \in NotAllowedTop\}\bigg)
\newline
+
\beta \bigg(\sum_{c=1}^{numColumns-1}\sum_{r=0}^{numRows-1}\sum \{(M_{r,c,t_{1}}+M_{r,c-1,t_{2}}-1)^{2}|\forall (t_{1},t_{2}) \in NotAllowedLeft\}\bigg)
\newline
+
\delta \bigg(\sum_{c=0}^{numColumns-1}\sum_{r=0}^{numRows-1}(1-\sum_{t=0}^{numTiles-1} M_{r,c,t})^{2}\bigg)
\newline
+
\epsilon \bigg(\sum_{t=0}^{numTiles-1} \big (countPerTile -\sum_{c=0}^{numColumns-1}\sum_{r=0}^{numRows-1} M_{r,c,t})^{2}\bigg)$$

To represent the Objective Function as QUBO, it needs to be simplified.

We know that,

$$(M_{r,c,t})^{2} = M_{r,c,t}
\because M_{r,c,t} \in \{0,1\} \newline$$

So, the Objective Function can be simplified as:-

$$E(M) = \alpha \bigg(\sum_{c=0}^{numColumns-1}\sum_{r=1}^{numRows-1}\sum \{(1+2M_{r,c,t_{1}}M_{r-1,c,t_{2}}-M_{r,c,t_{1}}-M_{r-1,c,t_{2}})|\forall (t_{1},t_{2}) \in NotAllowedTop\} \bigg)
\newline
+
\beta \bigg(\sum_{c=1}^{numColumns-1}\sum_{r=0}^{numRows-1}\sum \{(1+2M_{r,c,t_{1}}M_{r,c-1,t_{2}}-M_{r,c,t_{1}}-M_{r,c-1,t_{2}})|\forall (t_{1},t_{2}) \in NotAllowedLeft\}\bigg)
\newline
+
\gamma \bigg(\sum_{c=0}^{numColumns-1}\sum_{r=0}^{numRows-1}(1-\sum_{t=0}^{numTiles-1} M_{r,c,t} + \sum_{t_{1}=0}^{numTiles-1} \sum_{t_{2}>t_{1}}^{} 2M_{r,c,t_{1}}M_{r-1,c,t_{2}})\bigg)
\newline
+
\epsilon \bigg(\sum_{t=0}^{numTiles-1} \big (countPerTile^{2} + (-2a+1)countPerTile\sum_{c=0}^{numColumns-1}\sum_{r=0}^{numRows-1} M_{r,c,t}
\newline
+
\sum_{c_{1}=0}^{numColumns-1}\sum_{r_{1}=0}^{numRows-1} \sum_{c_{2}>c_{1}}^{} \sum_{r_{2}>r_{1}}^{} 2M_{r_{1},c_{1},t}M_{r_{2},c_{2},t}
)\bigg)$$

## Results

In this example, there are 9 tiles and a set of 5 sample maps with 10 rows and 10 columns.

#### Tiles

#### Sample Maps

#### Generated Maps
