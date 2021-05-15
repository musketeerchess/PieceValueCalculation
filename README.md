# Piece Value Calculation

## Abstract
A method to estimate the piece values [[2]](#2) of the Musketeer Chess variant piece types such as `dragon`, `queen`, `archbishop`, `cannon` and others.

## 1. &nbsp;Introduction

There are some methods of calculating/estimating piece values, one is thru actual experience from playing games, machine learning by optimizing piece values from results of the games or from evaluation of millions of positions and others.

In this article we will be describing a simple approach using mobility as the main criteria to calculate its value relative to other pieces. It consists of calculating the distance mobility from the central square, the number of directions and the penalty for color-bound piece type along with some factors or weights applied to those criteria.

## 2. &nbsp;Mobility

Piece mobility is an important factor in determining the value of a piece. It affects how the game would continue, for example a piece with lesser mobility is limited by the positions it is able to create. The advantage of a piece having high mobility is that it has potential to discover many positions which increases its chances to have favourable positions.

### 2.1 &nbsp;Distance mobility
Consider figure 1, the queen is a very mobile piece in a game. It has 8 different directions making it very difficult to defend when it is attacking. It also has 3 counts of mobility that can reach a distance of 4 squares from its location at square E4. The 3 squares are A4, A8 and E8 or those marked with 4 on the board. This makes it a dangerous piece even at long distance. 

Another strong property of this piece is that it has 8 counts of squares that it controls at a distance of 1 from E4 square, these are E5, F5, F4, F3, E3, D3, D4 and D5 or all those squares that are marked with 1, which would mean that opponent’s pieces cannot easily attack this piece at close range.

### 2.2 &nbsp;Direction mobility
Apart from the distance mobility another criteria that would contribute to the calculation of a piece value is the number of directions. In Figure 1, queen has 8 directions. The higher the direction the more dangerous the piece is.

### 2.3 &nbsp;Color-bound penalty
This is the penalty given to pieces that cannot move to other color on the board. An example of this is the bishop piece type in Chess.

## 3. &nbsp;Calculation

Piece value calculation will be based on the mobility described in mobility section and some other criteria. For every piece we store its mobility by a number of different ways a piece can move to at different distances. We also store the number of directions. Piece that can only move on one color (color-bound) like bishop will be given a penalty.

### 3.1 &nbsp;Criteria

#### 3.1.1 &nbsp;Distance mobility

On an 8x8 board there are at most a maximum distance of 4 that a piece can move to from E4 square for chess and musketeer chess variant piece types.

```
md1v = mobility distance 1 value
md2v
md3v
md4v
```

#### 3.1.2 &nbsp;Direction mobility

#### 3.1.3 &nbsp;Color-bound mobility penalty

### 3.2 &nbsp;Factors or Weights

Factors are used to scale the criteria across different distances. These factors are generated from thousands of game simulations of Musketeer Chess games. The criteria will be multiplied by these factors to get its value contribution. See Table 1 and 2 for mobility factors.

### 3.3 &nbsp;Formula 1

```
distance_mobility = Sum (mdnv)
where:
    mdnv = mdn x mdnf
    n = the distance from E4 square that a piece can move
    mdn = number of times that a piece can move to distance n from E4 square
    mdnf = the factor to scale the mdn, see table 1
    mdnv = the value from multiplying mdn and mdnf
```

### 3.4 &nbsp;Formula 2

```
direction_mobility = num_direction x direction_factor
where:
    num_direction = number of direction that a piece can move from E4 square
    direction_factor = the factor to scale num_direction, see table 2    
```

### 3.5 &nbsp;Formula 3

```
color_bound_penalty = (distance_mobility + direction_mobility) / cbf
where:
    cbf = color-bound factor = 3
    distance_mobility = a value from formula 1
    direction_mobility = a value from formula 2
```

### 3.6 &nbsp;General formula

`final_value = distance_mobility + direction_mobility – color_bound_penalty`

### 3.7 &nbsp;Algorithm 1

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
            # This new best factors will be tested against the next suggested factors.
            update_current_best_factor()

    return get_current_best_factor()
```
The `get_current_best_factor()` would return the best dm1f, dm2f, dm3f, dm4f, dirf and cbf.

## 4. &nbsp;Example piece value calculations

The first step is to put the piece in E4 square. Then calculate the distance mobility, direction mobility and color-bound penalty. These three values are added to get the final estimate of the piece value.

### 4.1 &nbsp;Queen

Location square: E4  
Variant: Chess/Musketeer Chess

![queen](https://i.imgur.com/VbQoRUW.png)  
Figure 1 &nbsp;The distance and direction mobilities of the queen from E4 square.

#### 4.1.1 &nbsp;Distance mobility

We will use formula 1.  

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

#### 4.1.2 &nbsp;Direction mobility

The more directions a piece has the more it becomes valuable as it can move at different directions, which is difficult to trap/capture.

##### Table 2. &nbsp;Direction Factor

| direction factor |
:-----------------:|
|       8          |

We will use formula 2.

```
direction_mobility = num_direction x factor
num_direction = 8
direction_factor = 8
direction_mobility = 8 x 8 = 64
```

#### 4.1.3 &nbsp;Color bound penalty

Penalty is zero because a queen can move to a different color from its reference square at E4. However if a piece to be evaluated is color-bound like the bishop piece type then we will use formula 3.

`color_bound_penalty = 0`

#### 4.1.4 &nbsp;Final queen value

```
final_value = distance_mobility + direction_mobility – color_bound_penalty
final_value = 868 + 64 – 0 = 932
```

### 4.2 &nbsp;Knight

Location square: E4  
Variant: Chess/Musketeer Chess

#### 4.2.1 &nbsp;Distance Mobility

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

#### 4.2.2 &nbsp;Direction mobility

We will use formula 2.

```
direction_mobility = num_direction x factor
num_direction = 8
direction_factor = 8
direction_mobility = 8 x 8 = 64
```

#### 4.2.3 &nbsp;Color-bound penalty

We will use formula 3.

`color_bound_penalty = 0`

#### 4.2.4 &nbsp;Final Knight value

`final_value = 240 + 64 – 0 = 304`

### 4.3 &nbsp;Bishop

Location square: E4  
Variant: Chess/Musketeer Chess

#### 4.3.1 &nbsp;Distance Mobility

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

#### 4.3.2 &nbsp;Direction mobility

We will use formula 2.
```
direction_mobility = num_direction x factor
num_direction = 4
direction_factor = 8
direction_mobility = 4 x 8 = 32
```

#### 4.3.3 &nbsp;Color-bound penalty

We will use formula 3.

`color_bound_penalty = (428 + 32) / 3 = 153`

#### 4.3.4 &nbsp;Final Bishop value

`final_value = 428 + 32 – 153 = 307`

#### Table 3 &nbsp;FIDE chess piece types with values in centipawn

type	  | md1v	| md2v	| md3v	| md4v	| dirv	| penalty  | value |
:--------:|:-------:|:-----:|:-----:|:-----:|:-----:|:--------:|:-----:|
Knight	  | 0	    | 240   | 0	    | 0 	| 64  	| 0        | 304   |
Bishop	  | 200	    | 120   | 96    | 12    | 32	| 153	   | 307   |
Rook	  | 200	    | 120	| 96    | 24    | 32	| 0	       | 472   |
Queen	  | 400	    | 240	| 192   | 36    | 64	| 0	       | 932   |

### 4.4 &nbsp;Leopard

Location square: E4  
Variant: Musketeer Chess

Leopard is a one of the pieces in musketeer chess variant. It can move like a knight. It can also move like a bishop but is limited to a maximum distance of 2 squares in any direction from its origin. Visit [[1]](#1) for the rest of the musketeer chess piece movements. </br>

![leopard](https://i.imgur.com/OhBykPN.png)  
Figure 2 &nbsp;The leopard in E4 square can move 4 times at distance 1 and 12 times at distance 2 and it has 12 directions.

#### 4.4.1 &nbsp;Distance Mobility

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

#### 4.4.2 &nbsp;Direction mobility

We will use formula 2.

```
direction_mobility = num_direction x factor
num_direction = 12
direction_factor = 8
direction_mobility = 12 x 8 = 96
```

#### 4.4.3 &nbsp;Color-bound penalty
We will use formula 3.

`color_bound_penalty = 0`

#### 4.4.4 &nbsp;Final Leopard value

`final_value = 560 + 96 – 0 = 656`

#### Table 4. &nbsp;The ten additional piece types for Musketeer chess variant with values in centipawn

type	     | md1v	| md2v | md3v	| md4v	| mirv	| penalty | value |
:-----------:|:----:|:----:|:------:|:-----:|:-----:|:-------:|:-----:|
Leopard	     | 200  | 360  | 0	    | 0	    | 96    | 0	      | 656   |
Cannon	     | 400  | 240  | 0	    | 0	    | 96    | 0	      | 736   |
Unicorn	     | 0	| 240  | 192	| 0	    | 128 	| 0	      | 560   |
Dragon	     | 400	| 480  | 192	| 36	| 128   | 0	      | 1236  |
Chancellor   | 200	| 360  | 96	    | 24	| 96	| 0	      | 776   |
Archbishop   | 200	| 360  | 96	    | 12	| 96	| 0	      | 764   |
Elephant	 | 400	| 240  | 0	    | 0	    | 64	| 0	      | 704   |
Hawk	     | 0	| 240  | 192	| 0	    | 64	| 0	      | 496   |
Fortress	 | 200	| 360  | 96	    | 0	    | 96	| 0	      | 752   |
Spider	     | 200	| 480  | 0	    | 0	    | 128	| 0	      | 808   |

## 5. &nbsp;Acknowledgement

Zied Haddad the inventor of Musketeer Chess variant.

## 6. &nbsp;References

<a id="1">[1]</a> Musketeer Chess Game Rules. URL http://musketeerchess.net/site/game-rules/.  
<a id="2">[2]</a> Musketeer Chess Rules and Performance Test. URL https://github.com/fsmosca/musketeer-chess.



