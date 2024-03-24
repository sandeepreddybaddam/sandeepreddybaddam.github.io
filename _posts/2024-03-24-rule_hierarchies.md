---
layout: page
classes: wide
title: Ping-Pong Playing Learner
---
[Not complete yet]

## How?

Discrete-time dynamics:
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/83ee6a17-48f6-42ea-a919-68f043b13c8f)

x- ego state
u - control inputs
t - discrete-time

S - space of trajectories
T - time horizon
x_ne - non-ego states
x_map - world map (lane lines, stop signs, etc)
w = (x_ne, x_map) - world scene

In practice, the planner will receive a predicted world scene which is generated from maps and by the prediction modules. (TODO IMAGE) Assume that this scene is provided to the planner.

### Rules and rule robustness
1. Rule - a boolean function that maps ego state and world scene to True or False.
$\phi$: S x W -> {True, False}

(TODO IMAGE) phi = 1 if rule satisfies, 0 otherwise

2. Robustness - a metric that provides the degree of satisfaction of the rule
Robustness of a rule phi_i, ρˆi: S × W → R
Positive scalar if the rule is satisfied and negative otherwise.
More positive values indicate greater satisfaction of the rule while more negative values indicate greater violation.


|Rules| STL formulae using STLCG, which comes equipped with back propagatable robustness metrics|
Expressing rules as STL formulae allows us to easily encode complex spatiotemporal specifications, such as maintaining a speed limit of 40mph from t+a to t+b.

Rule hierarchy - sequence of rules in precedence
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/b3c1d8cf-e59a-4087-9c11-4fdede04a435)

1: Highest Priority
N: Lowest Priority







