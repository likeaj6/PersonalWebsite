---
layout: post
title:  "Training and Implementing AlphaZero to play Hex"
date:   2018-05-20 20:30:27 -0700
categories: projects
---
### Check out the code to the project

## [--> here <--](https://github.com/likeaj6/alphazero-hex)

## A brief summary of what AlphaZero is and how it works:

**AlphaGo** is the original AI that was trained to play Go by Google in 2015.

It absolutely obliterated all the top human Go Masters, marking a landmark event in game-playing AI following IBM Watson in Jeopardy (2011) and Deep Blue versus Kasparov in Chess (1997).
!['alpha go playing a top go master'][alphago]

**AlphaGo Zero** is a vastly improved version, later released in 2017, that blew this original AlphaGo out of the waters, surpassing it within 3 days.
Furthermore, while AlphaGo trained itself by playing against experts, AlphaGo Zero *only ever plays itself*.
!['alpha zero's progress'][alphazeroprogress]

**AlphaZero** is an general game-playing AI that is trained completely on games played against itself, with no other info or human knowledge besides the rules of the game. This means it learns the game *completely from scratch*. This method is called **self-play**, which is a variation of **reinforcement learning**.

---


## Breakdown of AlphaZero's **self-play reinforcement learning algorithm**:
AlphaZero uses two core methods to learn to play a game:



### MCTS (Monte Carlo Tree Search)
* An algorithm that plays games by simulating the outcomes of various moves and choosing the best move based on said outcomes.
* Additional variables of MCTS include whether you want to explore new moves or exploit good moves we've already explored.
!['mcts diagram'][mcts]



### Deep Neural Network
* !['neural network'][neuralnet]
* A neural network is a model that takes in an input, feeds it into, many, many *hidden* layers full of neurons that activate based on a learned function, and outputs a prediction. It then goes backward and adjusts the hidden layers of nodes to make the prediction closer to the actual output.
* How a neural network does this is beyond the scope of this post, but definitely check out links here and here about backpropagation and gradient descent because they are awesome.
* The AlphaZero deep neural network is used to predict the **policy** (or how to go about playing a game) and **value** from a board state.
* The neural network outputs the *probabilities of playing various moves* and *the probability of winning that board state*.

### Both are interdependent:
* MCTS uses the predictions from the Deep Neural Network as a guide for simulating the results and probabilities of a game move.
* These simulations are done to predict the final outcome of playing a move in the current board.
* After exploring a bunch of potential moves, MCTS picks a move probabilistically based on its value according to these outcomes and the number of times MCTS explores each move.
* The Deep Neural Network then uses the final game outcome and probabilities of the game played by MCTS to correct and update itself.
!['mcts and neural network in alphazero'][mctsnn]

### Self-play:
* In the original AlphaGo Zero, during every iteration of self-play, the current best player (MCTS w/ Deep Neural Network), plays a large number of games (25,000) against itself.
* The resulting game data from these 25,000 games are used to train the neural network
* For the purposes of this project, we are only playing 1000 games per iteration, due to time and processing power constraints.

### How the Neural Network in AlphaZero evaluates and improves:
* In the original AlphaGo Zero, after every checkpoint (1000 training steps), the current best Neural Network is evaluated against the latest one by playing several hundred games (400) against itself. If it wins by a decent margin (in this case 55%), the latest Neural Network becomes the new best Neural Network to be used in future self-play.
* In our project, we are only playing 50 evaluation games per iteration.


## Applying AlphaZero to Hex
Rules of Hex:

!['how hex works'][hex]

* The win condition of the game is when a player fully reaches one side of the board from the other. Player 1 goes left to right and player 2 goes top to bottom.

Input to Neural Network:
* 8x8x1 board state (1 if player 1's piece, -1 if player 2's piece, 0 if empty)
* Output: probabilities of each move and the probabilities of winning (value between 0 and 1) based on the board

### A splash of Game Theory:

#### Symmetry and simplicity of the game:
* In Hex, the board is symmetric - this means you can flip the board and pieces and result in the same game.
* Our game representation is a simple 8x8x1 matrix, instead of requiring additional dimensions to encode rules as in chess, go and other games. The original AlphaGoZero also needed the past several board states, due to a rule in Go that you can't play the same move twice. In Hex, there are no additional constraints of where you can play a move, it just needs to be in an empty cell.
* Additionally, playing a move has the same reward for both players, there are no penalties for going first like in Go.
* All these allow us to simply flip the board and swap all of one player's pieces for the other, and end up with the same game, but from the other player's perspective.
* This allows us to have our neural network only learn the game from one player's perspective, and whenever we make predictions for the other player, we can just transpose the board and invert the pieces.
* This means that we can generate more game data (from both players) from a single game, which is a plus as we don't have the processing power and time to only take a single data point from each game as the original AlphaGoZero did.

## Bootstrapping the neural network with Supervised Learning

* Naturally, playing thousands and thousands of self-play games requires an enormous amount of processing power and time, neither of which we had for this project.
* Furthermore, watching the initial games showed us that AlphaZero was taking a really long time to learn the game itself.

!['self play at the beginning'][slowselfplay]

* We decided to utilize pre-existing “expert moves” generated by a traditional Monte Carlo Tree Search agent with a obscene number of rollouts (number of games simulated per move), something like 2^16.
* Using these expert moves, we bootstrapped and trained the neural network so that it could start at a higher skill-level and begin producing better self-play game data for itself.

### Results?

* After bootstrapping and about 2 iterations of self-play, each iteration consisting of 1000 games, our AlphaZero Hex agent was beating a Monte Carlo Tree Search agent 7/10 games. Given more time and processing power (that I did not have), I think AlphaZero could probably easily destroy any human player.
* Here's some bits of it learning the game and playing itself

!['self play game clip 1'][selfplay1] | !['self play game clip 2'][selfplay2]


### In conclusion, AI is such an interesting topic and super cool to experiment with. The barrier to start playing around with it really isn't as high as you think! You can check out the code to the project

## [--> here <--](https://github.com/likeaj6/alphazero-hex)

### Thanks for reading!



[alphago]: https://media.shellypalmer.com/wp-content/images/2016/03/alphago.jpg "alpha go playing lee sedol"
[alphazeroprogress]: https://www.kdnuggets.com/images/alphago-zero-progress-585.jpg "alpha go zero's progress vs other alphago versions"
[mcts]: https://upload.wikimedia.org/wikipedia/commons/thumb/6/62/MCTS_%28English%29_-_Updated_2017-11-19.svg/808px-MCTS_%28English%29_-_Updated_2017-11-19.svg.png "monte carlo tree search"
[neuralnet]: https://cdn-images-1.medium.com/max/2000/1*1mpE6fsq5LNxH31xeTWi5w.jpeg "neural networks"
[mctsnn]:https://louishenrifranc.github.io/img/2017-10-23-alphago/thumbnail.PNG "how mcts & deep neural net interact in alphazero"
[hex]: https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Hex-board-11x11-%282%29.jpg/250px-Hex-board-11x11-%282%29.jpg "hex"
[slowselfplay]:https://lh3.googleusercontent.com/_slJRCAvLIxx8ri5G9546JlrjhNdqQ8J4hKZsgrPluprO9WtMP8bs77XbnwWDlYkXUsOap5KxmYtLP-rOZH25ndkua1E_N_cNb-1wtwVWiBDoFOjbczPVgNpozxL353NAiVLGIEffDgXID9B5lKLu-3dwl_2SaQf8RcELPpaB6UPrwVDp6-_XyZPzSNwMqx9fOR22RZzITf6VzlUU4v-TUe249yrdP8Nmru6ntQ2gS4c8dMUyWBSWxEQhAObBO65BHENpueUmTXNbmXrROluaBAgAAd8RtnIXP5UUEA9XrvkulDKn8aP0iJPni6ZftYeSi6W2DjII_m-T22UWhJ1GbjVuLbhZzuX0z2eR4MwTRwi0QC5uNcR_mjuB5Y8lf53zAr3Lm1cYMqbY2KxYnTuCzcFf9Dyjm8udfNHVdA0S9TDRoSM_yL92NB7KRUswzTdIkclX0AH83KM33WPEXUSDCpPFWxiigDkwqfnLSElng0gVkcSKBo5GPuVDaxsSuzZ8877jR1Zbc7RqucDNkOk0mA4zyGQRGnjgLEauQf_vhy9JRUvH-G7LPH-tLl52-jHsITjezwK6fAqokysNoGNl4vGozkv65ZEw9F6fMQ=w1000-h800-no "self play taking a while to learn game"
[selfplay1]:https://lh3.googleusercontent.com/ieoAVlFfiw98ABe1qsEV3ZiFQHqFa_Cel1TKOwM4wX6f83NQXFtKVRHsob0P6cP-nsJA9QmWur5KGgd0R0rWF4dN5PiGufkOBxoYG9ENoaFG6wgOF5AYUW88qXLUtviVm3qbWuBs8NWbJxyr46p3hWiYVWZXvXeTDvHqZ16r91Sf00pR6BySbWl_tuVbnAZz6nI8bIJ54FWXsT-1uJGPvSB641eM_5LveLIcQ9O7X7cWTsui5oAnkbQS36PnVjqUjbWi7yrQVTRO8CfqzaS-oDqtDw6qUJRdN2cpTECiQvXE8EKdIdPrhFPuRWjL-kPA8To7zJIbx7UJOMPzJLhk-ps576P1Ok_wJzQWNgLKv3ZsjBV-5LYSUh4gtCgSV9PZc57Ed3mQi8a7Z9ltlxcL1tAAU7rul42h5v2txj9Fgw2Ivs-zgzuJnjiNjIZURSbV9UGQpZI--fW0TfNOHyLSmdgK6YE7BZX1eOv_bN2rg4tKYnNtrP2nJGYB-JWWYj9TnSGG4UwKWUZwY0Q8ltUAlvMrNcRMDKK1QBDkoxB-UXLGYc3uEbnfkJWSNIltZ0MXNH-lLCO2A6iOr66T2HCOicKeHjBYk83_3_ENwzM=w500-h400-no "self play game 1"
[selfplay2]:https://lh3.googleusercontent.com/JEQDxlJBuHvrWgW7G3vwEii3XvvqQnoyzXNBPvet9spaM2D7ngkKyVTxLe088TKZaCcXURJYbRMXLdJybLNfb7MBNkfXiPxeBUkS8LdPQ3o7F5_cT9XPWTxUMhbDHszGZGT27USjA2H7TBY6bl5S-1TBccokCkFJjgwatNZHAbn4Q12K-nTo8WPBKRaRs1KHtGhs7CwERi2cqjMkRWHKxxuDOoPsCSxZ7JTkdjmxRnI0ZgVllTQCqU6NVWoe3HHZbOAboHFgl6x9PDok-eewixX5MTXskfxlAkJ4NLBN6n33Ghba31WMyAC_3LA5RprGg3G5Jf0bz80__Ng8_eMLHGzOIjqjwleI7F7YzvI0l9G6I7KBkjE0_0mxAXJIFClrBvmu8K35V5K2WZUidwSvGdBoTHPdm9pK3iCLQV-lQyhgzzSJLekOFAZ98DkS8kwUxSfvRq6Pw6po9sX79mXdaU6-Zbu8nTCnfjKiCLdrHNfVhcV2SbZgFYGW7AMQ5jSfzhAK0PhDv5ztCZcOIG18CM5GyGWD2vuhpYhLudSaK_ZxhWJEMkZOyzVzH4O1uJGKQq4goMDNOQTYbwRfBD1N-c1g5MmV0zUG4b9-Yrw=w500-h400-no "self play game 2"
