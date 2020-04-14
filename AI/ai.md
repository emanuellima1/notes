# Notes on Artificial Intelligence

## Table of Contents

- [Notes on Artificial Intelligence](#notes-on-artificial-intelligence)
  - [Table of Contents](#table-of-contents)
  - [Intelligent Agents](#intelligent-agents)
  - [References](#references)

## Intelligent Agents

An agent is an object that perceives its environment through sensors and acts upon it through actuators. Example, a software agent that receives a file (sensor), processes it and writes a modified file to disk (actuator). A **percept** is the set of inputs an agent received at a given instant. A **percept sequence** is the set of all the inputs an agent has ever received. The choice of action of an agent at a certain point in time is dependant on the entire percept sequence observed until the point. If we have an **agent function** that maps any percept sequence to an action, we have almost completely described the agent. The actual implementation of the agent function is the **agent program** that runs inside the system.  
A rational agent is an agent that makes the right decision (being decision a choice of action on a set of possible actions). The right decision is the decision that produces the most desirable set of environmental states, measured by a certain **performance metric**. Remember that an action may change the environment, and that successive changes produce successive environmental states. This metric has to be chosen by the system designer. In other words, a rational agent is an agent that selects an action that it expects will maximize its performance metric given the evidence provided by the percept sequence and any built-in knowledge the agent has.  
It is important that the agent has absorbed as much information as possible from the environment in order to have the most informative percept sequence possible. Only then, it will be able to maximize correctly the performance metric. Apart from absorbing information, the agent must be able to learn from what it perceives from the environment.  
A rational agent should be autonomous, i. e. it should learn what it can to compensate for partial or incorrect prior knowledge. Therefore, agents should have some initial knowledge and the capacity to learn from future percepts. 
The performance metric, environment, actuators and sensors combined make up the so called **task environment**. There are some classifications that can be made about it:

- If the agent's sensors give it access to the complete state of the environment at each point in time, the task environment is **fully observable**. If the agent has no sensors, the task environment is **unobservable**. If it has partial access to the state of the environment, the task environment is **partially observable**.
- Depending on the number of agents present in an environment, the task environment can be **single agent** or **multiagent**. In a multiagent setting, if the agents compete, we have a **competitive multiagent** environment (an environment where an agent maximizing its metric means minimizing the metric of another agent). In the other hand, when agents cooperate, we have a **cooperative multiagent** environment.
- 

## References

- Stuart Russell, Peter Norvig. Artificial Intelligence, A Modern Approach. 3rd Edition.
