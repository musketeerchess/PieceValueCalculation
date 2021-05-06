# Piece Value Calculation
A method to calculate piece values of Chess, Musketeer Chess and other piece types.

## Introduction

There are some methods of calculating/estimating piece values, one is thru actual experience from playing games, machine learning by optimizing piece values from results of the games or from evaluation of millions of positions and others.

In this article we will be describing a simple approach using mobility as the main criteria to calculate its value relative to other pieces. It consists of calculating the distance mobility from the central square, the number of directions and the penalty for color-bound piece type.

## Mobility

Piece mobility is an important factor in determining the value of a piece. It affects how the game would continue, for example a piece with lesser mobility is limited by the positions it is able to create. The advantage of a piece having high mobility is that it has potential to discover many positions which increases its chances to have favourable positions.

### Distance mobility
Consider figure 1, the queen is a very mobile piece in a game. It has 8 different directions making it very difficult to defend when it is attacking. It also has 3 counts of mobility that can reach a distance of 4 squares from its location at square E4. The 3 squares are A4, A8 and E8 or those marked with 4 on the board. This makes it a dangerous piece even at long distance. 

Another strong property of this piece is that it has 8 counts of squares that it controls at a distance of 1 from E4 square, these are E5, F5, F4, F3, E3, D3, D4 and D5 or all those squares that are marked with 1, which would mean that opponent’s pieces cannot easily attack this piece at close range.

### Direction mobility
Apart from the distance mobility another criteria that would contribute to the calculation of a piece value is the number of directions. In Figure 1, queen has 8 directions. The higher the direction the more dangerous the piece is.

![](https://i.imgur.com/VbQoRUW.png)

Figure 1: Queen at E4 square and its distance mobility counts
