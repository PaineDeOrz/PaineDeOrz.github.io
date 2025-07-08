---
title: 'Human-Aligned Chess'
date: 2025-05-15
permalink: /posts/2025/05/Human-Aligned-Chess/
tags:
- draft
---

Introduction
===========

- Current status of the game (popularity, trends, problems and hopes)
- Motivation of chess as AI/ML application
- Motivation for human-alignment in the world of chess computers

History
==========

- Early hardware-based issues & developments, brute force computing
- Modern algorithmic approaches
- Stockfish (heuristic-based) vs AlphaZero/Leela (NN-based)
- (?)Detailed comparison in the workings of engines (with study)
- Examples of limitations and powers of modern-day engines, again motivation to human-alignment

MAIA
==========

- Data, move prediction task
- Implementation of the NN
- Motivation of no MCTS
- Results (Turing test?)
- Blunder prediction and possible uses

ALLIE
==========

- Data again (now with metadata)
- Deep-dive in decoder-only transformer model
- Design of LLM for chess & why natural language - weights work
- Motivation for MCTS
- Results (both offline evaluation & human qualitative feedback)
- Personal opinion (?)

Looking at the future
==========

- Issues in present and upcoming models, tough problems regarding data
- What computers can do, should become able to do, and what they can't be used for
- How research in chess can translate into other applications


Introduction
==========

In the last few years, chess has absolutely boomed. It feels like it's everywhere right now, whether in the train, in the parc, even in the lecture hall, there are people playing chess on their phones. The number of tournaments, movies, TV shows, learning material etc. has increased exponentially since the pandemic, even though this game has existed for centuries. There are over 10 million people logging in every day and playing an online game. This means an enormous amount of games, being played by all types of people (be they professionals, players who don't even know the rules, 5 year-olds, 95 year-olds and everything in between) are recorded and stored in databases, along with all sorts of metadata (players' ELO ratings, nationality, time format etc.). To put it in short: a simple, deterministic, fully observable game with an insane amount of well organized data. For any AI enthusiast, this is Heaven!

Of course, the fact that chess is a great environment to develop AI tools has been known ever since computers were created. Alan Turing himself created an algorithm even before computers were advanced enough to run it. The 70's and 80's saw the creation of computers built specifically for chess (also called engines), despite massive costs, culminating with IBM's DeepBLue, a multi-million dollars project that ultimately defeated world champion at the time (1997) and arguably greatest chess player of all time, Garry Kasparov. After this legendary moment, investing in chess engines increased exponentially, and today we have computers that can easily defeat the best players even without a whole lot of processing power.

However, the world of chess isn't without its problems. Cheating, in particular, is probably the greatest one. Another one is the very inefficient learning process of booting up an engine, seeing a ridiculous move being recommended, not even beginning to understand it and just giving up and moving on to the next game. Just imagine a personalized chess trainer that would be able to recommend moves based on one's skill level and style of play.

Both of these, as well as many other possibilities in the world of chess, start, unfortunately, with an ambiguous concept: "human", or "natural" moves. What makes a move "natural", so that our trainer could recommend it? What makes a move "unnatural", so that we can detect it as a form of cheating? These are the kind of questions we are going to take the first step into answering in this blogpost, by seeing how to create a "human-aligned engine" that can predict not the best move in a position, but the most "human" one.


Modern Chess Engines
===========

In order to understand how to create a "human-aligned engine", first we need to dive into the workings of a "normal" engine. Ever since the launch of Alpha-Zero, an engine developed by Google in 2017 with a revolutionary neural network-based approach, the trend has been the same: A really complex, feed-forward neural network with lots of layers, training on self play (playing millions or even billions of games against itself) to create an evaluation function, based on which an iterative deepening Monte-Carlo Tree Search (in short MCTS) is run. The more time and processing power available, the better the tree search could be, though limited by the neural network (which however complex cannot be perfect). This limitation of using machine learning algorithms (based on lots of data rather than ground truths) instead of more simple AI ones (such as basic alpha-beta pruning with lots of handcrafted heuristics, which Stockfish was), may seem unintuitive and is not even always better. A famous example for it is Lasker's puzzle, a study that puts white in a completely winning position, with forced checkmate in #(put how many moves) moves, which LeeLa, a continuation of the Alpha-Zero concept, doesn't find. The seemingly more basic Stockfish, however, does find it, albeit after a very long time in the world of computers. Today, though, even Stockfish has moved towards the neural network approach, which we are also going to use for our "human-aligned" engines, albeit for more obvious reasons.

Human-Aligned Engines - a First Attempt
==========

The first question one might have when talking about designing a human-level engine is as follows: If we already have Stockfish or Leela which are much better than human, why can't we simply cut a lot of their processing power, or limit the depth of their search tree, until they are just as weak as us? Well, here is where the concept of "human" moves comes into play. Humans, whether they are complete beginners, amateurs or advanced players, don't just play random bad moves. They play certain bad moves. There is always some thought process going into a move, sometimes erroneous for hidden reasons, such as missing an insane tactic 10 moves into a variation, sometimes for obvious reasons, such as blundering a queen in one move. Some moves are extremely intuitive, what we call "natural", even though an engine may deem it a blunder. 

In order to understand a little bit more clearly what an "unnatural" move is, let's take an example. In the following position, the best player in the world, Magnus Carlsen, has a rook and a knight for the opponent's queen, which is already disadvantageous. Stockfish thinks that the only way to draw the game (which is the best result Magnus can hope for here), is to sacrifice a knight, though not for immediate tactical purposes (even Stockfish doesn't find a forced sequence leading to the draw). Despite having more than 20 minutes to think, Magnus doesn't find the move, and the commentators don't even blame him, because that engine recommendation is simply "unnatural".

Just toning down existing engines doesn't emulate human play, it just leads to random bad moves, so we have to come up with smarter methods.

MAIA
==========
MAIA, the first "human-aligned" engine, was created with a very simple idea in mind. Taking exactly the model of LeeLa, but instead of training the neural network with self-played games (which are good to find the best move because they investigate many moves), train it with human games. We have already talked about there being billions of games nicely recorded in databases, so we have more than enough data to train the neural network. Of course, when we talk about human-level play, it is very important to distinguish between a 5 year old beginner playing a game on the park and Magnus Carlsen playing in a tournament, but we already have the tools to differentiate the two: ELO. This is a sistem that separates people based on their strength, and it does the job really well (it has been used for more then 50 years). So MAIA is not actually one engine, it is a family of nine different ones, each working exactly in the same way, but the neural network is trained on games between players of different rating ranges. 

The rather unintuitive change that MAIA brings, however, is a lack of the MCTS following the training of the neural network. This is a purely ad-hoc finding. As the following graph portrays, the results are simply a bit better when the neural network alone predicts the move. This is quite surprising. There are two main reasons for it: (1) MAIA only focuses on amateur-level games (under 2000) (2) the authors only used a fixed depth MCTS (which becomes relevant when we talk about ALLIE). 

Apart from this peculiarity, there is not much else to say about MAIA's implementation. Much more interesting, however, are the results. We can see an accuracy a little over 50%, which intuitively sounds quite poor, but let's analyze what this accuracy actually means. Normally in chess, when we talk about accuracy, we compare human play against the engine. Thus, 100% accuracy means playing all the best moves recommended by the computer, and any other moves at any point decrease that accuracy, some by a lot (blunders), other by a little (sight mistakes or just alternatives to the best moves). In our case here, it is a little bit differently. Because we don't actually measure how "human" each move is, we cannot objectively measure whether a predicted move is good for us or not. Instead, all we have is already played games, and we either predict the move played in the game used for testing, or we don't. This means that a rather low accuracy score is very much normal, because in a position there can be multiple different "human" moves, but only one is tested. As such, using only the accuracy metric is clearly not enough for evaluating our engine.

Instead, by far the best analysis of a human-aligned engine is, of course, a Turing test. After all, if humans playing against the engine cannot tell that their opponent is not human, our goal is accomplished. That is, after all, as human as it gets. For the Turing test we go to a different paper, and the results are really quite astounding. Humans were paired against other humans, MAIA, and a toned-down version of Stockfish, then asked to rate their confidence in whether their opponent was human or not. While Stockfish was clearly recognized, MAIA is hardly so. The difference between playing a human opponent and MAIA is very small, which shows that the accuracy metric should be taken with a grain of salt. Of course, the higher the better, that is why we'll be taking a look at an even better engine next, but these results are already very good.

ALLIE
==========





