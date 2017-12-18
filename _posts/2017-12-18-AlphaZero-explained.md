---
layout: post
title: AlphaZero Explained (for chess players)
---

Last week was pretty exciting for chess. 
Google DeepMind published a [paper](https://arxiv.org/abs/1712.01815) detailing how they created a chess engine, AlphaZero, that was able to crush the worlds top computer program, Stockfish, beginning only with knowledge of the rules of the game. 
Unfortunately, to many interested chess players, the details behind the algorithms - the methods AlphaZero was employing to make their moves - remains a complete mystery. 
I’m hoping this post will give an overview of how AlphaZero was coming up its moves. 

**So how is AlphaZero coming up with its moves?**

Imagine we had a way to estimate the probabilities for "an expert” to make each available move in a chess position, without searching at depth. 
That is, just by looking at the position, could there be some intrinsic patterns, that in itself might suggest the moves that should be played? 
For instance, if one player has a passed pawn, we might expect that the move of advancing this pawn might be given a high probability, given the general rule that “passed pawns must be pushed".
Of course, these “instincts” into a position need to be investigated, for any agent that plays chess - and this is what is known as going into depth into a position.

So if we had some way to get a reasonable characterization of potential optimal moves in the position without going into depth into the position, we might be able to use this “intuition” to guide our search. 
So for now, let’s suppose that we have a way to do this - that given any position, we can reasonably quantify the probabilities of an expert making each possible move. 
To get a more accurate assessment of the position — to validate our intuition — we need figure out the evaluations of the subsequent positions that might arise.
A reasonable way to search through the possible positions might be to focus our search on the moves with the highest probability of occurring. 
We could continue the game, evaluating each subsequent position with our “intuition function” and continue choosing the move with the highest possible probability of occurring, until we reach the end of the game.
In subsequent iterations of the search we’d need to incorporate the knowledge of whether the position(s) we searched resulted in a positive result, as well as explore new positions (as our intuition function just needs to be good - not perfect). 
Both of these tradeoffs are incorporated into the Monte Carlo Search Tree, used in the AlphaZero. 
As long as we have a reasonable "intuition function”, a reasonable way of characterizing the available moves in a position, we can search through the resulting positions efficiently. 
The search itself basically refines our intuition, it performs a sort of averaging of the resulting positions. We can think of the search as a function that takes in as inputs the initial move probabilities, and spits a more accurate, set of final move probabilities.

We relied heavily of having what I called an “intuition function”, a way of evaluating a given position, without going into a position’s depth. 
AlphaZero achieves this through what is called a neural network, which sounds complicated, but is basically a collection of tunable weights that can be tuned to achieve a particular function.

**What is a neural network?**

A neural network is a nonlinear function that simply put, transforms inputs to outputs. In our case, we need to somehow transform the input - that is a given position - into a set of move probabilities (and also an expected value output). A neural network sounds mysterious, but is actually made up of very understandable components. It consists of layers, with each layer performing a different operation, as well as connections between the layers. These connections are often called “weights”, and correspond to how important the previous layers are. The exact architecture of the network, for instance the number of layers and also their type, is a bit of a mystery, even for researchers who study neural networks extensively. It still is an empirical science, and certain architectures are found to work better for specific tasks.

It turns out that given sufficiently large training data, we can usually tune these weights in such a way that they can produce the desired outputs. In our case, our output consists of the initial move probabilities, outputted by our “intuition function”, or neural network, as well as a term that corresponds to the expected value of the game, given the current position. After a game of self play, we have the final result of the game, and can therefore minimize the difference between the actual and expected outcomes. If only, we had a more accurate assessment of the true move probability distributions… The thing is — we actually do have more accurate move probability assessments -- these were the move probabilities after our search! And not only that, we have this, for each move. So, we can have a cost function that includes both a term that minimizes the difference between actual and expected outcome, as well as a term that minimizes the difference between initial and final move probabilities. 

Still, to train neural networks, we need lots of data. In the case of this paper, they create their own data, by pitting the current version of neural network parameters against previous versions. By playing millions of training games, and by tuning the weights to match more accurate move probabilities, the neural network is effectively trained to look for important patterns, characteristics, or features of a position. Then, at the end, just by looking at a position, it is able to have a reasonable characterization of the most likely moves in the position.

**Concluding thoughts**

Some of the games that AlphaZero played were truly special - its ability to weight piece activity, and play long-term positional material sacrificing was quite remarkable. It was the first time, for many chess players, that we were blown away by computer chess. There’s a couple questions I have, after reading this paper. I’d be really curious to know how similar, after training, the initial move probabilities are with the final move probabilities. Moreover, based on the potential similarity, or lack thereof, can we perhaps characterize certain positions, in the sense that less concrete positions, those of a more “positional nature” might have initial move probabilities more similar to final move probabilities, whereas tactical positions perhaps consist of bigger differences between the initial and final move probabilities? Also, is there any intrinsic reason that the neural network needs to be refined by the MCTS, or is there perhaps some way that the network itself could learn to search more deeply through the position (without us humans telling us it should look deeper)?

