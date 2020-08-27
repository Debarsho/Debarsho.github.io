---
layout: post
title: Intelligent Traffic Lights using Reinforcement Learning
published: false
---
A project that uses Deep Q-Learning to automate a simulated traffic system. The objective is to reduce the overall waiting time of vehicles at traffic lights.
<!--break-->


## Introduction
While Deep Reinforcement Learning has produced exciting results in the field of AI, one could easily assume that its practicality is limited to game bots. Here, we apply the concept to a problem most of us experience on a daily basis - traffic.

In India, most traffic lights have preset timers to rotate through different phases (green, yellow, red). Could the ideal time duration to rotate between lights and reduce vehicle waiting time be determined using Reinforcement Learning? Let’s take a look at how we might achieve this.


## Solution Overview
I won’t go into much detail about Deep Q-learning, but a great introduction to it can be found [here](https://medium.com/@joshpatterson_5192/introduction-to-deep-q-learning-1bded90a6193). If you don’t understand terms like Q-values, rewards, state, etc. then you should probably have a look at it. This approach might be better suited for Traffic Management compared to traditional Machine Learning algorithms because of the fast adaptivity required.

The aim of this project is for a traffic light to determine the optimum time duration to cycle through traffic light phases, given the load at its junction. Each traffic light would be an agent. Given the traffic load (the number of vehicles) at each part of the junction and the current time duration, the traffic light would decide how to update the time duration. In a practical scenario, the load could be determined by cameras on top of the traffic lights, or induction loop traffic sensors.

The only thing our model optimizes is the green light time. The yellow light time was kept constant, and the red light time was determined from the green light time. I’ll come back to this point later.


## Data
All traffic simulations were carried out by interfacing [SUMO](http://sumo.dlr.de/index.html) with [TraCI](http://sumo.dlr.de/wiki/TraCI) in Python, and the map we simulated on was a part of of Vijayawada, Andhra Pradesh, India. The map data was exported from [OpenStreetMap](https://www.openstreetmap.org/). For the simulations, random vehicle trips were generated. Collecting actual data for individual vehicles and their detailed journeys in a city would either be almost impossible to do, or exceed this project’s scope. I’ll come back to this in the possible improvements.


## Network Architecture
Like any reinforcement learning model, any agent (traffic light) receives a state and a reward from the environment every time it completes an action.
- The state is the load (no. of vehicles) at each part of the junction and the current duration. If the junction had 4 roads connecting to it, the input dimension of the Neural Network would be 5 (1 for each road and 1 for the current duration).
- The action can be of three types: decrease the duration by one second, keep it the same, or increase it by one second.
- The reward is the reduction in average waiting time of vehicles at that junction.

![The mechanism at the core of RL algorithms](https://cdn-images-1.medium.com/max/758/1*Z2yMvuQ1-t5Ol1ac_W4dOQ.png)

Also note that, for any traffic light, there would clearly be too many possible states for this project to be carried out with a simple Q-table. That is, it makes much more sense to apply Deep Q-learning as opposed to Q-learning due to the sheer number of possible loads at a junction.

There would be three outputs of the Neural Network: one Q-value for each of the three actions. The concept of delayed rewards was implemented by updating agents’ models in periodic intervals. The duration of these intervals was hardcoded. Like any other reinforcement learning model, this was done because we may not see the fruit of our actions immediately. It was also mentioned earlier that the red light time was determined from the green light time. This was because the sum of green light time and red light time was kept constant. If the periodic interval to update weights was 90s, and the optimized green light time was found to be 50s, then the red light time would be 40s.

The problem of exploration vs exploitation was solved with a fairly common solution - the Epsilon Greedy implementation of Q-learning. Essentially, this means that we start out with a high degree of randomness (for exploration) which decreases as the model learns.

The expected Q-value for our Neural Network was calculated using the Bellman Equation, and Mean Squared Error was used as the Loss function.

![The Bellman Equation](https://cdn-images-1.medium.com/max/1250/1*jmcVWHHbzCxDc-irBy9JTw.png)

In the Bellman Equation, values of 0.5, 0.97, and 5 were used for alpha, the discount rate, and the number of future steps too look forward to respectively. The value of the learning rate for the Neural Network (not to be confused with alpha from the Bellman equation) was 0.1.

Three hidden layers of 64, 32 and 32 units were used, and Dropout with a rate of 0.2 was applied on the first 32 unit layer. The hyperbolic tangent function was used as the activation function for all hidden layers, and no activation function was used for the final output layer.


## Possible Improvements
Simulating and training using real world data would be the most significant possible addition to this project, mainly because it would (hopefully) prove that this model works practically as well. When simulations are being run on real world data, we could also use time data as another series of inputs. For example, traffic on Sundays at 5PM would be different from Mondays at 9AM, on christmas eve it would be more near airports, and so on. The only reason we didn’t use time data here is because the generated vehicle trips are random.

An improvement that could be made upon the model itself is the use of a multi agent system. Any agent would also learn from its adjacent agents. Put simply, our current solution has every traffic light independent of any others, and we might be able to improve our model if a traffic light takes adjacent traffic lights into consideration. We might even be able to add more parameters, like the distance between them.


## Source Code
The source code for the project can be found [here](https://github.com/AkhilChilakala/Smart_India_Hackathon).


## The Team
Six of us worked on this project, namely Abid Abdullah, Amogh Badugu, Akhil Chilakala, Athilesh Arputham, Shreyas Ravishankar and myself.
