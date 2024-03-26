---
layout: page
classes: wide
title: Rule Hierarchies
---
[Not complete yet]

**Paper summary:** [Receding Horizon Planning with Rule Hierarchies for Autonomous Vehicles - 13 Dec 2023](https://arxiv.org/pdf/2212.03323.pdf)

## Why?
Rule hierarchies can enhance **interpretability**, but introduce combinatorial complexity to planning; on the other hand, differentiable reward structures can be leveraged by modern gradient-based optimization tools, but are less interpretable and unintuitive to tune.

### 1. Rule hierarchies
| Pros | Cons |
|---|---|
| 1. They provide a systematic approach to plan motions that prioritize more important rules (e.g., safety) over the less important ones (e.g., comfort) if all rules cannot be simultaneously satisfied <br> 2. Greater transparency in planner design <br> 3. More amenable to introspection | 1. Rule hierarchies introduce significant combinatorial complexity to planning <br> 2. Sometimes challenging to implement |

### 2. Differential reward structures
| Pros | Cons |
|---|---|
| 1. Less computation time from the reward optimization scheme <br> 2. Relatively easier to implement | 1. Not intuitive to tune and difficult to interpret the planner's performance |

In the worst case, type 1 takes up to $2\^N$ time complexity for _N_ rules (?). This paper presents an approach to equivalently express rule hierarchies as differentiable reward structures amenable to modern gradient-based optimizers, thereby, achieving the best of both worlds. 
* Good interpretation from ranking scheme: **Expressive power of hierarchies**
* Less computation time from reward optimization scheme **Computational advantages of gradient-based optimization through modern optimization tools**
Hierarchy-preserving reward function that allows us to plan without checking all combinations of rule satisfaction.

## How?

### A. Problem formulation
* **Dynamics:** <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/fcd81337-75e1-4946-89dd-095a2de6a6e8)<br>
`x`- ego state <br>
`u` - control inputs <br>
`t` - discrete-time <br>

* **Non-ego agents and map** <br>
`S` - space of trajectories <br>
`T` - time horizon <br>
$x\_{ne}$ - non-ego states <br>
$x\_{map}$ - world map (lane lines, stop signs, etc) <br>
`w` = ($x\_{ne}$, $x\_{map}$) - world scene

In practice, the planner receives a predicted world scene that is generated from maps and prediction modules. (TODO IMAGE) <br>
Assume that this scene is provided to the planner.

* **Rules and rule robustness**
    1. **Rule** ($\phi$) - a boolean function that maps ego state and world scene to True or False
        - $\phi$: `S` x `W` → {`True`, `False`}
        - $\phi$ = `1` if rule satisfies, `0` otherwise
    2. **Robustness** ($\rho$) - a metric that provides the degree of satisfaction of the rule
        - Robustness of a rule ($\phi_i$), $\rho_i$: `S` × `W` → R
        - Positive scalar if the rule is satisfied and negative otherwise.
        - More positive values indicate greater satisfaction of the rule while more negative values indicate greater violation.
  ![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/6ab1e5ac-cfa9-4536-ba46-7bb2dcb0bff1)
[Source](https://youtu.be/hrjt6abUPDA)

    * Rules → expressed as Signal Temporal Logic (STL) formulae using STLCG (STL Computational Graphs), which comes equipped with back propagatable robustness metrics
        - Expressing rules as STL formulae allow us to easily encode complex spatiotemporal specifications, such as maintaining a speed limit of `40mph` from `t+a` to `t+b`
    3. **Rule hierarchy** ($\varphi$) - sequence of rules in precedence <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/b3c1d8cf-e59a-4087-9c11-4fdede04a435)
        - `1`: Highest priority
        - `N`: Lowest priority <br>
        - For a trajectory, the robustness ($\rho$) of a rule hierarchy ($\varphi$) is an `N`-dimensional vector-valued function, where `N` is the number of rules. <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/497e56cd-a89e-4f84-86b3-49dc89fac4eb) <br>
    4. **Rank of a trajectory**(r) <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/b068d389-f084-453a-9e34-3179c528f7a7)<br>
An illustration of trajectory ranks for three rules <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/fa8be243-b75b-4b0b-b448-2be27580810c)
        - **Objective**: Obtain control inputs that result in a trajectory with the highest achievable rank in accordance with a rule hierarchy: <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/abbc4636-a4b2-4b85-bb6e-f9c506014c11)
            - This is a single optimization that replaces checking $2\^N$ combinations of rule satisfaction.

---

### B. Rank-preserving reward function
To optimize the above equation, the paper presents a differentiable rank-preserving reward function.<br>
Monotonicity:<br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/66d6dedd-44f1-4e56-b4ee-b7f0d92ebae6) <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/384a9245-f074-4d64-906f-5401085283cb)<br>
(_You can find proof in the appendix section of the paper_)

- The intuition behind the above equation construction
  - Ensure that the reward contribution on satisfaction of rule _i_ should exceed the sum of the reward contributions by all rules with lower priority
  - This is achieved by multiplying the step function with a constant that grows exponentially with the priority of a rule
  - Average robustness to break the ties between the same-ranked trajectories

![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/e9a12fc2-92a1-42e9-8423-77c6cf230855) <br>
  - To understand the above equation more intuitively, look at the following plot.<br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/af93c36d-f7a7-479d-8cce-1f46f17e5611)

---

### C. Receding horizon planning with rule hierarchies
Now that, we have a method to cast rule hierarchy into a nonlinear differentiable reward function.

A two-stage algorithm to solve (3).

| | Stage 1: Planning with motion primitives | Stage 2: Continuous trajectory optimization|
|---|---|---|
| Objective | To generate a coarse initial trajectory for warm-starting the continuous optimizer in the second stage | To refine the trajectory obtained from the first stage by solving <br> ![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/9f29c613-c541-4dc2-beb2-394e7123d911) <br> * You could further maximize the reward by still complying with rule hierarchy Ex: Safety improvement using STL robustness |
| Process | * Choose a set `M` of open-loop controls that coarsely cover a variety of actions that the AV can take; e.g., `(accelerate, turn right)`, `(decelerate, keep straight)`, etc. <br> * Parallelized process <br> ![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/84678b3b-6f2a-4d92-899a-30fc9b040a51) <br> * Select the trajectory with the largest rule-hierarchy reward from sampled primitives | * Compute the gradients analytically using STLCG and refine the controls <br> ![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/f24356dd-b909-4747-a873-2e5048717b10) |

#### Pseudo code
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/ff8b5516-4705-46e4-b1ea-fbbc0caee692)

## Signal Temporal Logic
[Source](https://youtu.be/hrjt6abUPDA) - Autonomous Systems Laboratory<br>
It is a formal language that converts specifications made in natural language to mathematical representation, that can be used in synthesis. <br>
![image](https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/bce8b773-bcd2-441c-8a28-bd2ecee0d55d)

| Term | Explanation |
|---|---|
| Logic | Made up of logical operations. For example AND, OR, NOT, etc. |
| Temporal | These logical operations can be specified over time intervals |
| Signal | We are reasoning about these logic and temporal operations over continuous real-valued time series such as a trajectory of the robot as it moves in time and space |

<img width=400 align="center" src="https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/54ebc0ae-986d-4c35-aad2-e620751792d5">

### Boolean semantics
<img width=800 align="center" src="https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/bd93d138-724b-47da-a242-ba3fd3beb8e4">

### Robustness
A measure of how much a signal satisfies or violates an STL formula

### Quantitative semantics
<img width=800 align="center" src="https://github.com/sandeepreddybaddam/sandeepreddybaddam.github.io/assets/100727983/55de216f-5f49-4147-ac4d-8f42627b5bd9">

### STLCG
Signal Temporal Logic Computational Graphs - It is a toolbox to compute robustness and their gradients using STL formulae.
Repository [here](https://github.com/StanfordASL/stlcg).





























