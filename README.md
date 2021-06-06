<div align="center">
<h1>A simple way to estimate the Piece Values in a Game</h1>
    
<b>Ferdinand Mosca, Zied Haddad (1)</b>  
((1) Musketeer Creations)  

2021-06-06</p>

<h2>Abstract</h2>

We describe a simple method to estimate the relative value of "atypical" pieces, used in the game of Musketeer Chess <sup>[[2]](#2)</sup> variant. Musketeer Chess, is a Chess Variant whom main characteristics are to add two pieces to the classic Chess game. The play is still on an 8x8 Chequered Board. The additional pieces are introduced during the game by a drop mechanism. 
This work is about the following 10 pieces: <b>Archbishop, Cannon, Chancellor, Dragon, Elephant, Fortress, Hawk, Leopard, Spider and Unicorn</b>. Our method evaluates the relative value of the Classic Chess pieces fairly precisely, which is a good indication of reliability. 

This method is usable for many other "unusual" pieces, provided that they move in a classic way (they capture in the squares they can go to). This method was not tested in pieces with "special" moves, like Xiangqi Cannon with captures when hopping over an obstacle [3], Witch that alters iteraction with surrounding pieces[4] etc.
</div>

## 1. &nbsp;Introduction

One of the first lessons when learning Chess is the piece relative value. The widely known system is the 1-3-3-5-9 system, with a Pawn worth 1 point, Knight and Bishop worth 3 points etc. It's not the most reliable one, but it's simplicity make it easy to remember.
Other systems where either based on World Champions own mastery of the game and their own experience (Euwe [5], Fischer [6], Kasparov [7]), Extensive Database analysis of Chess Masters games (Kaufman [8]), automated engine vs engine matches with search for optimized values using various methods (SPSA [9,10], Nevergrad [11] etc.), Monte-Carlo Tree Search and machine learning [12].
Our main goal was to provide a simple approach using piece mobility as the main criteria to calculate its relative value. The uniqueness of our methodology is that the mobility is calculated from central squares (we don't average mobility of the piece from all the squares): We sum weighted values for Distance Mobility, Number of Directions and Penalty for color-bound piece types.
This method is inspired from previous works. The first one, was published by Taylor as early as in the 19th century [13] and more recently Ralph Betza, one of the fathers of modern chess variants [14]. 
Our aim was to provide a starting point, to make introduction to Chess Variants and use of "Atypical" pieces easier, providing a relative value for these pieces, facilitating decision making and evaluation of forces on the board easier. 


## 2. &nbsp;Mobility

Piece mobility is an important factor in determining the value of a piece [13-15]. The higher mobility, the bigger is for the piece to create threats and have favorable positions.

### 2.1. &nbsp;Distance mobility
Consider figure 1, the queen is a very mobile piece in a game. It has 8 different directions making it very difficult to defend when it is attacking. It also has 3 counts of mobility that can reach a distance of 4 squares from its location at E4 central square (squares A4, A8 and E8): squares marked with 4 on the board. This makes the Queen, the strongest and most dangerous piece, with long range attacking capabilities. 

Another strong property of this piece is that it has 8 counts of squares that it controls at a distance of 1 from E4 square: the E5, F5, F4, F3, E3, D3, D4 and D5 squares. These squares are marked with 1, which would mean that opponent’s pieces cannot easily attack the Queen at close range.

### 2.2. &nbsp;Direction mobility
Apart from the distance mobility another criteria that would contribute to the calculation of a piece value is the number of directions. In Figure 1, queen can move in 8 different directions. The higher the number of different directions, the more dangerous the piece is (forking possibilities etc.)

### 2.3. &nbsp;Color-bound penalty
This is the penalty given to pieces that cannot move to other color on the board. A known example is the Bishop, stuck to one colored squares (Bishop beginning on Dark squares, cannot move to light squares and vice versa).

![queen](https://i.imgur.com/VbQoRUW.png)  
Figure 1. &nbsp;The distance and direction mobilities of the queen from E4 square.

## 3. &nbsp;Calculation

Piece value calculation is based on various mobility parameters, as described in the mobility section. 
For every piece we calculate the distance, where a piece can move to from the E4 square. We also calculate the number of different directions, the piece can go. We also take into account if a piece is color-bound, meaning that a piece is stuck to one color. The example of a color-bounded piece in Chess is Bishop. A colour-ounded piece will have a proportional penalty.

### 3.1. &nbsp;Criteria

#### 3.1.1. &nbsp;Distance mobility

On an 8x8 board there is at most a maximum distance of 4 that a piece can move to from E4 square for chess and musketeer chess variant piece types.

```
md1v = mobility distance 1 value
md2v
md3v
md4v
```

#### 3.1.2. &nbsp;Direction mobility

#### 3.1.3. &nbsp;Color-bound mobility penalty

### 3.2. &nbsp;Factors or Weights

Factors are used to scale the criteria across different distances. These factors are generated from thousands of game simulations of Musketeer Chess games. The criteria will be multiplied by these factors to get its value contribution. See [Table 1](#table-1-distance-mobility-factors) and [2](#table-2-direction-factor) for mobility factors.

### 3.3. &nbsp;Formula 1
See section 4 for an example calculations.

```
distance_mobility = Sum (mdnv)
where:
    mdnv = mdn x mdnf
    n = the distance from E4 square that a piece can move
    mdn = number of times that a piece can move to distance n from E4 square
    mdnf = the factor to scale the mdn, see table 1
    mdnv = the value from multiplying mdn and mdnf
```

### 3.4. &nbsp;Formula 2

```
direction_mobility = num_direction x direction_factor
where:
    num_direction = number of direction that a piece can move from E4 square
    direction_factor = the factor to scale num_direction, see table 2    
```

### 3.5. &nbsp;Formula 3

```
color_bound_penalty = (distance_mobility + direction_mobility) / cbf
where:
    cbf = color-bound factor = 3
    distance_mobility = a value from formula 1
    direction_mobility = a value from formula 2
```

### 3.6. &nbsp;General formula

`final_value = distance_mobility + direction_mobility – color_bound_penalty`

### 3.7. &nbsp;Algorithm 1

Run engine vs engine matches to find the best factors.

```python
def find_best_factors():
    current_best_factor = get_init_best()

    for i in range(10000):
        dm1f, dm2f, dm3f, dm4f, dirf, cbf = suggest_mobility_factors()

        # leopard/hawk, cannon/dragon ... 45 combinations
        sum_result = 0
        for pc_combo in all_45_musketeer_piece_combo:    
            test_result = run_match(dm1f, dm2f, dm3f, dm4f, dirf, cbf,
                                    current_best_factor, pc_combo, games=200)
            sum_result += test_result
            
        # Get the average result for all pc combo.
        ave_result = sum_result/45
        if ave_result > 0.5:
            update_current_best_factor()

    return get_current_best_factor()
```
The `get_current_best_factor()` would return the best dm1f, dm2f, dm3f, dm4f, dirf and cbf.

## 4. &nbsp;Example piece value calculations

The first step is to put the piece in E4 square. Then calculate the distance mobility, direction mobility and color-bound penalty. These three values are added to get the final estimate of the piece value.

### 4.1. &nbsp;Queen

Location square: E4  
Variant: Chess/Musketeer Chess

#### 4.1.1. &nbsp;Distance mobility

```
md1 = 8
md2 = 8
md3 = 8
md4 = 3
```

md1 refers to number of moves the queen can move from square E4 at distance 1. See figure 1 with squares that are marked by `1`, there are 8 of them. md2, md3 and md4 are typical to md1 but at different distances from square E4.

##### Table 1. &nbsp;Distance Mobility Factors

| md1f | md2f | md3f | md4f |
|:----:|:----:|:----:|:----:|
|  50  |  30  |  24  |  12  |

We will use formula 1 to get its mobility values

```
md1v = md1 x md1f = 8 x 50 = 400
md2v = md2 x md2f = 8 x 30 = 240
md3v = md3 x md3f = 8 x 24 = 192
md4v = md4 x md4f = 3 x 12 = 36
```

md1v, md2v and others are based on centipawn value, or 1 pawn = 100.

`distance_mobility = md1v + md2v + md3v + md4v = 400 + 240 + 192 + 36 = 868`

#### 4.1.2. &nbsp;Direction mobility

The more directions a piece has the more it becomes valuable as it can move at different directions, which is difficult to trap/capture.

##### Table 2. &nbsp;Direction Factor

| direction factor |
|:----------------:|
|       8          |

We will use formula 2.

```
direction_mobility = num_direction x factor
num_direction = 8
direction_factor = 8
direction_mobility = 8 x 8 = 64
```

#### 4.1.3. &nbsp;Color bound penalty

Penalty is zero because a queen can move to a different color from its reference square at E4. However if a piece to be evaluated is color-bound like the bishop piece type then we will use formula 3.

`color_bound_penalty = 0`

#### 4.1.4. &nbsp;Final queen value

```
final_value = distance_mobility + direction_mobility – color_bound_penalty
final_value = 868 + 64 – 0 = 932
```

### 4.2. &nbsp;Knight

Location square: E4  
Variant: Chess/Musketeer Chess

#### 4.2.1. &nbsp;Distance Mobility

```
md1 = 0
md2 = 8
md3 = 0
md4 = 0
```

We will use formula 1. See table 1 for mobility factors.

```
value = criteria x factor

md1v = 0 x 50 = 0
md2v = 8 x 30 = 240
md3v = 0 x 24 = 0
md4v = 0 x 12 = 0
distance_mobility = 0 + 240 + 0 + 0 = 240
```

#### 4.2.2. &nbsp;Direction mobility

We will use formula 2.

```
direction_mobility = num_direction x factor
num_direction = 8
direction_factor = 8
direction_mobility = 8 x 8 = 64
```

#### 4.2.3. &nbsp;Color-bound penalty

We will use formula 3.

`color_bound_penalty = 0`

#### 4.2.4. &nbsp;Final Knight value

`final_value = 240 + 64 – 0 = 304`

### 4.3. &nbsp;Bishop

Location square: E4  
Variant: Chess/Musketeer Chess

#### 4.3.1. &nbsp;Distance Mobility

```
md1 = 4
md2 = 4
md3 = 4
md4 = 1
```

We will use formula 1. See table 1 for mobility factors.

```
value = criteria x factor

md1v = 4 x 50 = 200
md2v = 4 x 30 = 120
md3v = 4 x 24 = 96
md4v = 1 x 12 = 12
distance_mobility = 200 + 120 + 96 + 12 = 428
```

#### 4.3.2. &nbsp;Direction mobility

We will use formula 2.
```
direction_mobility = num_direction x factor
num_direction = 4
direction_factor = 8
direction_mobility = 4 x 8 = 32
```

#### 4.3.3. &nbsp;Color-bound penalty

We will use formula 3.

`color_bound_penalty = (428 + 32) / 3 = 153`

#### 4.3.4. &nbsp;Final Bishop value

`final_value = 428 + 32 – 153 = 307`

#### Table 3. &nbsp;FIDE chess piece types except Pawn with values in centipawn

| type    | md1v    | md2v  | md3v  | md4v  | dirv  | penalty  | value |
|:-------:|:-------:|:-----:|:-----:|:-----:|:-----:|:--------:|:-----:|
| Knight  | 0	    | 240   | 0	    | 0 	| 64  	| 0        | 304   |
| Bishop  | 200	    | 120   | 96    | 12    | 32	| 153	   | 307   |
| Rook    | 200	    | 120	| 96    | 24    | 32	| 0	       | 472   |
| Queen	  | 400	    | 240	| 192   | 36    | 64	| 0	       | 932   |

### 4.4. &nbsp;Leopard

Location square: E4  
Variant: Musketeer Chess

Leopard is one of the pieces in musketeer chess variant. It can move like a knight. It can also move like a bishop but is limited to a maximum distance of 2 squares in any direction from its origin. Visit <sup>[[1]](#1)</sup> for the rest of the musketeer chess piece movements. </br>

![leopard](https://i.imgur.com/OhBykPN.png)  
Figure 2. &nbsp;The leopard in E4 square can move 4 times at distance 1 and 12 times at distance 2 and it has 12 directions.

#### 4.4.1. &nbsp;Distance Mobility

```
md1 = 4
md2 = 12
md3 = 0
md4 = 0
```

We will use formula 1. See table 1 for mobility factors.


```
value = criteria x factor

md1v = 4 x 50 = 200
md2v = 12 x 30 = 360
md3v = 0 x 24 = 0
md4v = 0 x 12 = 0
distance_mobility = 200 + 360 + 0 + 0 = 560
```

#### 4.4.2. &nbsp;Direction mobility

We will use formula 2.

```
direction_mobility = num_direction x factor
num_direction = 12
direction_factor = 8
direction_mobility = 12 x 8 = 96
```

#### 4.4.3. &nbsp;Color-bound penalty
We will use formula 3.

`color_bound_penalty = 0`

#### 4.4.4. &nbsp;Final Leopard value

`final_value = 560 + 96 – 0 = 656`

#### Table 4. &nbsp;The ten additional piece types for Musketeer chess variant with values in centipawn

|type	     | md1v	| md2v | md3v	| md4v	| mirv	| penalty | value |
|:----------:|:----:|:----:|:------:|:-----:|:-----:|:-------:|:-----:|
|Leopard     | 200  | 360  | 0	    | 0	    | 96    | 0	      | 656   |
|Cannon	     | 400  | 240  | 0	    | 0	    | 96    | 0	      | 736   |
|Unicorn     | 0	| 240  | 192	| 0	    | 128 	| 0	      | 560   |
|Dragon	     | 400	| 480  | 192	| 36	| 128   | 0	      | 1236  |
|Chancellor  | 200	| 360  | 96	    | 24	| 96	| 0	      | 776   |
|Archbishop  | 200	| 360  | 96	    | 12	| 96	| 0	      | 764   |
|Elephant	 | 400	| 240  | 0	    | 0	    | 64	| 0	      | 704   |
|Hawk	     | 0	| 240  | 192	| 0	    | 64	| 0	      | 496   |
|Fortress	 | 200	| 360  | 96	    | 0	    | 96	| 0	      | 752   |
|Spider	     | 200	| 480  | 0	    | 0	    | 128	| 0	      | 808   |

## 5. &nbsp;Acknowledgement

Zied Haddad the inventor of the Musketeer Chess variant.

## 6. &nbsp;References

<a id="1">[1]</a> Musketeer Chess Game Rules. URL http://musketeerchess.net/site/game-rules/.  
<a id="2">[2]</a> Musketeer Chess Rules and Performance Test. URL https://github.com/fsmosca/musketeer-chess.
<a id="3">[3]</a> Xiangqi cannon. URL https://en.wikipedia.org/wiki/Xiangqi#Cannon. 
<a id="4">[4]</a> The Witch Explained. URL https://www.chess.com/clubs/forum/view/witch-explained.
<a id="5">[5]</a> Euwe, Max; Kramer, Hans (1994) [1944], The Middlegame, vol. 1, Hays, ISBN 978-1-880673-95-9.
<a id="6">[6]</a> Fischer, Bobby; Mosenfelder, Donn; Margulies, Stuart (1972), Bobby Fischer Teaches Chess, Bantam Books, ISBN 0-553-26315-3.
<a id="7">[7]</a> Kasparov, Gary (1986), Kasparov Teaches Chess, Batsford, ISBN 0-7134-55268.
<a id="8">[8]</a> Kaufman, Larry (March 1999), "The Evaluation of Material Imbalances", Chess Life, 1999(3): 6-10. 
<a id="9">[9]</a> James C. Spall. Implementation of the Simultaneous Perturbation Algorithm for Stochastic Optimization. IEEE Transactions on Aerospace and electronic systems. 1998, 34(3): 817-23.
<a id="10">[10]</a> SPSA tuner, to optimize engine evaluation parameters. URL https://github.com/ianfab/spsa.
<a id="11">[11]</a> Nevergrad: A gradient-free optimization platform. URL https://github.com/facebookresearch/nevergrad.
<a id="12">[12]</a> C.M. Miller, "Evolutionary Artificial Neural Network Weight tuning to Optimize decision makig for an Abstract Game.". M Sc diss. Air Force Inst. of Technology, Ohio, USA 2010.
<a id="13">[13]</a> H.M. Taylor M.A. (1876) XXVII. On the relative values of the pieces in chess , Philosophical Magazine Series 5, 1:3, 221-229, DOI: 10.1080/14786447608639029
<a id="14">[14]</a> Ralph Betza (1996). About the Values of Chess Pieces. https://www.chessvariants.com/d.betza/pieceval/.
<a id="15">[15]</a> Soltis, Andy (2004), Rethinking the Chess Pieces, Batsford, ISBN 0-7134-8904-9.
