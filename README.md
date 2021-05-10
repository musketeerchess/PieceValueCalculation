# Piece Value Calculation
A method to calculate piece values of Chess, Musketeer Chess and other piece types.

## Introduction

There are some methods of calculating/estimating piece values, one is thru actual experience from playing games, machine learning by optimizing piece values from results of the games or from evaluation of millions of positions and others.

In this article we will be describing a simple approach using mobility as the main criteria to calculate its value relative to other pieces. It consists of calculating the distance mobility from the central square, the number of directions and the penalty for color-bound piece type.

## Mobility

Piece mobility is an important factor in determining the value of a piece. It affects how the game would continue, for example a piece with lesser mobility is limited by the positions it is able to create. The advantage of a piece having high mobility is that it has potential to discover many positions which increases its chances to have favourable positions.

#### Distance mobility
Consider figure 1, the queen is a very mobile piece in a game. It has 8 different directions making it very difficult to defend when it is attacking. It also has 3 counts of mobility that can reach a distance of 4 squares from its location at square E4. The 3 squares are A4, A8 and E8 or those marked with 4 on the board. This makes it a dangerous piece even at long distance. 

Another strong property of this piece is that it has 8 counts of squares that it controls at a distance of 1 from E4 square, these are E5, F5, F4, F3, E3, D3, D4 and D5 or all those squares that are marked with 1, which would mean that opponent’s pieces cannot easily attack this piece at close range.

#### Direction mobility
Apart from the distance mobility another criteria that would contribute to the calculation of a piece value is the number of directions. In Figure 1, queen has 8 directions. The higher the direction the more dangerous the piece is.

![](https://i.imgur.com/VbQoRUW.png)

Figure 1: Queen at E4 square shows its distance and direction mobility.

## Calculation

Piece value calculation will be based on the mobility described in mobility section and some other criteria. For every piece we store its mobility by a number of different ways a piece can move to at different distances. We also store the number of directions. Piece that can only move on one color (color-bound) like bishop will be given a penalty.

### Criteria
1. Mobility at specific distances
2. Number of directions
3. Color bound penalties

### Factors or Weights
Factors are used to scale the criteria accross different piece types. These factors are generated from thousands of game simulations of Musketeer Chess games. The criteria will be multiplied by these factors to get its value contribution. See Table 1 and 2 for mobility factors.

#### General formula
`final_value = distance_mobility + direction_mobility – color_bound_penalty`

### Example calculation
The first step is to place the piece in E4 square. Then calculate the distance mobility from critera 1, direction mobility from criteria 2 and color-bound penalty from criteria 3. These three values are added to get the final estimate of the piece value.

#### A. Queen
Location square: E4
Variant: chess  

#### 1. Mobility at specific distances  
```
Md1 = 8
Md2 = 8
Md3 = 8
Md4 = 3
```
Md1 refers to number of moves the queen can move from square E4 at distance 1. See figure 1 with squares that are marked by `1`, there are 8 of them. Md2, Md3 and Md4 are typical to Md1 but at different distances from square E4.

#### Formula 1
`value = criteria x factor`

##### Table 1: Distance Mobility Factors
Md1f | Md2f | Md3f | Md4f
---  | ---  | ---  | ---
 50  | 30   |  24  | 12

We will use formula 1 to get its mobility values

```
Md1v = Md1 x Md1f = 8 x 50 = 400
Md2v = Md2 x Md2f = 8 x 30 = 240
Md3v = Md3 x Md3f = 8 x 24 = 192
Md4v = Md4 x Md4f = 3 x 12 = 36
```

Md1v, Md2v and others are based on centipawn value, or 1 pawn = 100.

`distance_mobility = Md1v + Md2v + Md3v + Md4v = 400 + 240 + 192 + 36 = 868`

#### 2. Direction mobility
The more directions a piece has the more it becomes valuable as it can move at different directions, which is difficult to trap/capture.

##### Table 2: Direction Factor
direction factor |
--- |
8   |

We will use formula 1.
```
direction_mobility = num_direction x factor
num_direction = 8
direction_factor = 8
direction_mobility = 8 x 8 = 64
```

#### 3. Color bound penalty
Penalty is zero because a queen can move to a different color from its reference square at E4. However if a piece to be evaluated is color-bound like the bishop piece type then we will use `Formula 2`.

`color_bound_penalty = 0`

#### Formula 2
`color_bound_penalty = (distance_mobility + direction_mobility) / 3`

#### Final queen value
Queen final value will be the sum of the distance mobility plus direction mobility less color-bound penalty.

```
final_queen_value = distance_mobility + direction_mobility – color_bound_penalty
final_queen_value = 868 + 64 – 0 = 932
```

