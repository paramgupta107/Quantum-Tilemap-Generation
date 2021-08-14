# Quantum Tilemap Generation

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

<img src="https://i.upmath.me/svg/M_%7Br%2Cc%2Ct%7D%5E%7Bi%7D%20%5Cin%20%5C%7B0%2C1%5C%7D" alt="M_{r,c,t}^{i} \in \{0,1\}" />

Where,

the value is 1 if tile **t** is present at row **r** and column **c** in sample map **i**.

<img src="https://i.upmath.me/svg/AllowedTop%20%3D%20%5C%7B(t_%7B1%7D%2Ct_%7B2%7D)%20%7C%20M_%7Br%2Cc%2Ct_%7B1%7D%7D%5E%7Bi%7D%3D1%2C%20M_%7Br-1%2Cc%2Ct_%7B2%7D%7D%5E%7Bi%7D%3D0%20%5Cnewline%20%5Cforall%200%20%5Cleq%20i%20%5Cleq%20%20numMaps%2C%0A1%20%5Cleq%20r%20%5Cleq%20%20numRows%2C%0A0%20%5Cleq%20c%20%5Cleq%20%20numColumns%2C%0A0%20%5Cleq%20t%20%5Cleq%20%20numTiles%5C%7D" alt="AllowedTop = \{(t_{1},t_{2}) | M_{r,c,t_{1}}^{i}=1, M_{r-1,c,t_{2}}^{i}=0 \newline \forall 0 \leq i \leq  numMaps,
1 \leq r \leq  numRows,
0 \leq c \leq  numColumns,
0 \leq t \leq  numTiles\}" />

**AllowedTop** is the set of all pairs of tiles where 2nd tile is allowed to be on top of the first tile

<img src="https://i.upmath.me/svg/AllowedLeft%20%3D%20%5C%7B(t_%7B1%7D%2Ct_%7B2%7D)%20%7C%20M_%7Br%2Cc%2Ct_%7B1%7D%7D%5E%7Bi%7D%3D1%2C%20M_%7Br%2Cc-1%2Ct_%7B2%7D%7D%5E%7Bi%7D%3D0%5Cnewline%20%5Cforall%200%20%5Cleq%20i%20%5Cleq%20%20numMaps%2C%0A0%20%5Cleq%20r%20%5Cleq%20%20numRows%2C%0A1%20%5Cleq%20c%20%5Cleq%20%20numColumns%2C%0A0%20%5Cleq%20t%20%5Cleq%20%20numTiles%5C%7D" alt="AllowedLeft = \{(t_{1},t_{2}) | M_{r,c,t_{1}}^{i}=1, M_{r,c-1,t_{2}}^{i}=0\newline \forall 0 \leq i \leq  numMaps,
0 \leq r \leq  numRows,
1 \leq c \leq  numColumns,
0 \leq t \leq  numTiles\}" />

**AllowedLeft** is the set of all pairs of tiles where 2nd tile is allowed to be on the left of the first tile

<img src="https://i.upmath.me/svg/AllCombinations%20%3D%20%5C%7B(t_%7B1%7D%2Ct_%7B2%7D)%7C0%5Cleq%20t_%7B1%7D%5Cleq%20numTiles%2C0%5Cleq%20t_%7B2%7D%5Cleq%20numTiles%5C%7D" alt="AllCombinations = \{(t_{1},t_{2})|0\leq t_{1}\leq numTiles,0\leq t_{2}\leq numTiles\}" />

**AllCombinations** is the set of all possible pairs of tiles.

<img src="https://i.upmath.me/svg/NotAllowedTop%20%3D%20AllCombinations-AllowedTop" alt="NotAllowedTop = AllCombinations-AllowedTop" />

**NotAllowedTop** is the set of all pairs of tiles where 2nd tile is not allowed to be on top of the first tile.

<img src="https://i.upmath.me/svg/NotAllowedLeft%20%3D%20AllCombinations-AllowedLeft" alt="NotAllowedLeft = AllCombinations-AllowedLeft" />

**NotAllowedLeft** is the set of all pairs of tiles where 2nd tile is not allowed to be on the left of the first tile.


The Objective Function can then be defined as:-

<img src="https://i.upmath.me/svg/E(M)%20%3D%20%5Csum_%7Bc%3D0%7D%5E%7BnumColumns-1%7D%5Csum_%7Br%3D1%7D%5E%7BnumRows-1%7D%5Csum%20%5C%7BM_%7Br%2Cc%2Ct_%7B1%7D%7D%20%5Ccdot%20M_%7Br-1%2Cc%2Ct_%7B2%7D%7D%7C%5Cforall%20(t_%7B1%7D%2Ct_%7B2%7D)%20%5Cin%20NotAllowedTop%5C%7D%0A%5Cnewline%0A%2B%0A%5Csum_%7Bc%3D1%7D%5E%7BnumColumns-1%7D%5Csum_%7Br%3D0%7D%5E%7BnumRows-1%7D%5Csum%20%5C%7BM_%7Br%2Cc%2Ct_%7B1%7D%7D%20%5Ccdot%20M_%7Br%2Cc-1%2Ct_%7B2%7D%7D%7C%5Cforall%20(t_%7B1%7D%2Ct_%7B2%7D)%20%5Cin%20NotAllowedLeft%5C%7D%0A%5Cnewline%0A%2B%0A%5Cgamma%20%20%5Cbigg(%5Csum_%7Bc%3D0%7D%5E%7BnumColumns-1%7D%5Csum_%7Br%3D0%7D%5E%7BnumRows-1%7D(1-%5Csum_%7Bt%3D0%7D%5E%7BnumTiles-1%7D%20M_%7Br%2Cc%2Ct%7D)%5E%7B2%7D%5Cbigg)" alt="E(M) = \sum_{c=0}^{numColumns-1}\sum_{r=1}^{numRows-1}\sum \{M_{r,c,t_{1}} \cdot M_{r-1,c,t_{2}}|\forall (t_{1},t_{2}) \in NotAllowedTop\}
\newline
+
\sum_{c=1}^{numColumns-1}\sum_{r=0}^{numRows-1}\sum \{M_{r,c,t_{1}} \cdot M_{r,c-1,t_{2}}|\forall (t_{1},t_{2}) \in NotAllowedLeft\}
\newline
+
\gamma  \bigg(\sum_{c=0}^{numColumns-1}\sum_{r=0}^{numRows-1}(1-\sum_{t=0}^{numTiles-1} M_{r,c,t})^{2}\bigg)" />

To represent the Objective Function as QUBO, it needs to be simplified.

We know that,

<img src="https://i.upmath.me/svg/(M_%7Br%2Cc%2Ct%7D)%5E%7B2%7D%20%3D%20M_%7Br%2Cc%2Ct%7D%0A%5Cbecause%20M_%7Br%2Cc%2Ct%7D%20%5Cin%20%5C%7B0%2C1%5C%7D%20%5Cnewline" alt="(M_{r,c,t})^{2} = M_{r,c,t}
\because M_{r,c,t} \in \{0,1\} \newline" />

So, the Objective Function can be simplified as:-

<img src="https://i.upmath.me/svg/E(M)%20%3D%20%5Csum_%7Bc%3D0%7D%5E%7BnumColumns-1%7D%5Csum_%7Br%3D1%7D%5E%7BnumRows-1%7D%5Csum%20%5C%7BM_%7Br%2Cc%2Ct_%7B1%7D%7D%20%5Ccdot%20M_%7Br-1%2Cc%2Ct_%7B2%7D%7D%7C%5Cforall%20(t_%7B1%7D%2Ct_%7B2%7D)%20%5Cin%20NotAllowedTop%5C%7D%0A%5Cnewline%0A%2B%0A%5Csum_%7Bc%3D1%7D%5E%7BnumColumns-1%7D%5Csum_%7Br%3D0%7D%5E%7BnumRows-1%7D%5Csum%20%5C%7BM_%7Br%2Cc%2Ct_%7B1%7D%7D%20%5Ccdot%20M_%7Br%2Cc-1%2Ct_%7B2%7D%7D%7C%5Cforall%20(t_%7B1%7D%2Ct_%7B2%7D)%20%5Cin%20NotAllowedLeft%5C%7D%0A%5Cnewline%0A%2B%0A%5Cgamma%20%5Cbigg(%5Csum_%7Bc%3D0%7D%5E%7BnumColumns-1%7D%5Csum_%7Br%3D0%7D%5E%7BnumRows-1%7D(1-%5Csum_%7Bt%3D0%7D%5E%7BnumTiles-1%7D%20M_%7Br%2Cc%2Ct%7D%20%2B%20%5Csum_%7Bt_%7B1%7D%3D0%7D%5E%7BnumTiles-1%7D%20%5Csum_%7Bt_%7B2%7D%3Et_%7B1%7D%7D%5E%7B%7D%202M_%7Br%2Cc%2Ct_%7B1%7D%7DM_%7Br-1%2Cc%2Ct_%7B2%7D%7D)%5Cbigg)" alt="E(M) = \sum_{c=0}^{numColumns-1}\sum_{r=1}^{numRows-1}\sum \{M_{r,c,t_{1}} \cdot M_{r-1,c,t_{2}}|\forall (t_{1},t_{2}) \in NotAllowedTop\}
\newline
+
\sum_{c=1}^{numColumns-1}\sum_{r=0}^{numRows-1}\sum \{M_{r,c,t_{1}} \cdot M_{r,c-1,t_{2}}|\forall (t_{1},t_{2}) \in NotAllowedLeft\}
\newline
+
\gamma \bigg(\sum_{c=0}^{numColumns-1}\sum_{r=0}^{numRows-1}(1-\sum_{t=0}^{numTiles-1} M_{r,c,t} + \sum_{t_{1}=0}^{numTiles-1} \sum_{t_{2}&gt;t_{1}}^{} 2M_{r,c,t_{1}}M_{r-1,c,t_{2}})\bigg)" />

## Results

### Example 1

In this example, there are 9 tiles and a set of 5 sample maps with 10 rows and 10 columns.

#### Tiles

<img src="images/allTiles.png"/>

#### Sample Maps

<img src="images/allMaps.png"/>

#### Generated Maps

<img src="images/output 1.png"/>
<img src="images/output 2.png"/>

### Example 2
In this example, there are 40 tiles and a set of 1000 sample maps with 10 rows and 10 columns.

#### Tiles

Following are some of the tiles from the set of 40 tiles.

<img src="images/allTilesb.png"/>

#### Sample Maps

Following are some of the sample maps from the set of 1000 sample maps.

<img src="images/allMapsb.png"/>

#### Generated Maps

<img src="images/outputb 1.png"/>
<img src="images/outputb 2.png"/>