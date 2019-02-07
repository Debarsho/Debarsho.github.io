---
layout: post
title: The Monty Hall problem - an explanation that actually makes sense
published: true
---
My explanation for a classic problem which, in my opinion, makes more sense than most explanations found online.
<!--break-->

## Introduction

From [Wikipedia](https://en.wikipedia.org/wiki/Monty_Hall_problem#Simple_solutions), the problem statement is as follows:
>Suppose you're on a game show, and you're given the choice of three doors: Behind one door is a car; behind the others, goats. You pick a door, say No. 1, and the host, who knows what's behind the doors, opens another door, say No. 3, which has a goat. He then says to you, "Do you want to pick door No. 2?" Is it to your advantage to switch your choice?

Most people’s initial answer would be that it doesn’t make a difference whether one switches or not. Why would anyone’s chances of winning increase if each door initially had an equal chance of being the winning choice? Surprisingly, the answer is that it is advantageous to switch your choice.

## My Explanation
There are only two possible strategies: either you change doors or you don’t. Consider persons *A* and *B*, who play the game independently. *A* never switches doors after the host opens one revealing a goat, but *B* always switches. For example if a player chooses door 1 and the host reveals that a goat was in door 3, *A* would stick to door 1 while *B* would change their choice to door 2.

Let’s examine how *A*’s game would proceed. Whatever door they initially pick, there is a ⅓ chance they chose a car and a ⅔ chance they chose a goat. They never change their pick no matter what, so it is safe to assume they will win the game with a probability of ⅓.

Coming to *B*’s game, they always switch doors no matter what. Similar to *A*, their initial pick has a ⅓ chance of being a car and a ⅔ chance of being a goat. But, a key observation to make is that if a player initially chose a car, and then switches, they will always get a goat. Similarly, if a player initially chooses a goat and then switches, they will always get the car (remember, the player can only switch after one door is revealed to contain a goat).

Visualize the game as such: you choose door 1 and the host reveals that door 3 has a goat in it. If door 1 has a car in it, the door 2 has to have a goat. Similarly if door 1 has a goat in it, door 2 has to have a car. **So if you switch, you are guaranteed that your prize (car/goat) changes from goat to car or car to goat.** Because there is a ⅔ chance your initial pick had a goat, and you are guaranteed that your prize changes if you change your choice of door, it is advantageous to always change your choice of door.

## TL;DR
Once the observation is made that a change of prize (car/goat) is guaranteed if one’s strategy is to always switch doors, it can be concluded that switching is advantageous because if we switch, we are more likely to pick a goat then switch to car, rather than pick a car and then switch to goat.