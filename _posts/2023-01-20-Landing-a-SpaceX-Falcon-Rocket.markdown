---
layout: post
title:  "Landing a SpaceX Falcon Rocket"
date:   2023-01-14 13:00:00 +0800
categories: controls
mathjax: true
---

(WIP)

Contents
- Introduction to PID
- How to choose gains?
- Example
- Conclusion

My motivation for this blog post was from learning Signals and Systems during my 2nd year in National University of Singapore.

## Introduction to PID

The **Proportional** term is used to adjust the system's input based on the current error between the desired output and the actual output. This term is used to bring the system's output closer to the desired output.

$error(e)$ $=$ $x_{desired}$ - $x_{actual}$

$$Force(f) = K_p * e$$

Here $K_p$ is known as the proportional gain and e (error).

Our goal is to make the error as close to 0 to reach the desired set-point. The force which the thrusters applied is proportional to the measured error, however when the rocket is almost near to the destination (set-point), the thrusters apply lesser force. This naturally causes the thrusters to slow down. Thus, the proportional gain tells us how far away we are from where we want to get. This terms behaves like a spring ($F=-kx$ as per Hooke's law)

The gains are always positive because negative gains tend to be unstable. The direction of the error is what is controlling the direction of the force.

The **Derivative term**, $K_d$ describes how much the error is changing over time, $\frac{\partial e}{\partial t}$. The derivative terms acts like a critically damped system getting rid of the oscillations and **slowing down** to reac the desired set point. (Explain more about critically damped)

The **Integral term** is used to adjust the system's input based on the accumulated error over time. This term is used to eliminate any steady-state errors that may be present in the system. The integral error removes the stead state errors by adding up all of the errors form the past and multiplying it by another gain called $K_i$. Additionally, the integral terms keeps on adding the errors from the past where it will eventually gives of a extra small push of force to push us forward again. If air friction, exists that would be a perfect example to demonstrate the use of the Integral component. The rocket would have to push harder to accumuklate the error to land properly.

However, do take note that the Integral terms also makes the system accidentally unstable. It will overshoot if not properly managed.

Keep track of the of the added up error.

Thus, so far we have,

$$u = K_p e(t) + K_d \frac{\partial e}{\partial t} + K_i \int_{0}^t e(t) dt$$

## How to choose gains (PID Tuning Guide)?

Okay, now we have understood the 3 main components which make up a PID system. How do we go on about choosing the gains appropritely? 

1. Set all the gains to Zero
2. Increase the P gain until the response to a disturbance is steady oscillation
3. Increase the D gain until the oscillations goes away (i.e. it's critically damped)
4. Repeat steps 2 and 3 until increasing the D gain does not stop the oscillations.
5. Set P and D to the last stable values.
6. Increase the I gain until it brings you to the setpoint with the number of oscillations desired (normally zero but a quicker response can be achieved if you don't mind a couple oscillations of overshoot)

## Rocket Simulation

![screenshot](/assets/images/Landing-a-SpaceX-Falcon-rocket/screenshot.png)

## Conclusion


### Future work and Suggestions

Noise Filtering. Real world systems can accumulate noise from the environment and implementation of the system.

There also exists other types of controllers such as LQR (Linear Quadratic Regulator) regulator, using control theory to get the gains of the controller in optimal control. There exists a cost function that penalizes from the set point but also penalizes how much of "effort" to reach there. LQR supposedly gives the most optimal gains to minimize the cost.

Linear should be a linear system. For example, the pendulum is linear.  However, genetic algorithms does not provide the stability where a PID can guarantee.

MPC (Model Predictive Control) predicting into the future what the next state is, aka known as trasjectory optimization, rapidly changes its prediction.


## References:
- https://www.youtube.com/playlist?list=PLn8PRpmsu08pQBgjxYFXSsODEF3Jqmm-y
- https://ocw.mit.edu/courses/16-06-principles-of-automatic-control-fall-2012/pages/lecture-notes/
- 