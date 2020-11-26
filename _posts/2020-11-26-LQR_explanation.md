---
title     : "Linear Quadratic Regulator (LQR) 설명 및 구현"
categories: 
  - optimal control
toc       : true
toc_sticky: false
layout    : single
classes   : wide
---

## Background

### Optimization Problem

- LQR은 최적화 문제의 대표적인 사례이다. 최적화 문제는 아래와 같이 수식화할 수 있다.

- $$\underset{x}{minimize}f(x) \\
s.t. \quad g_i(x)\leq 0, \forall i, \\
h_j(x) = 0, \forall j.$$

- 위에서 $f(x)$는 objective function이라 부른다.

### Optimality Condition

- KKT conditions은 Nonconvex problem의 경우, optimality를 위한 필요조건이고, convex problem의 경우, optimality를 위한 필요충분조건이다.

- LQR의 경우, 모든 변수에 대해 opjective가 quadratic 하므로 convex 하다. 따라서 KKT condtions을 통해 구한 control이 optimal함을 만족한다.

- 아래 챕터에서 KKT condtions을 통해 control 계산을 유도할 것이다.

- Lagrangian function : $L(x, \mu, \lambda) := f(x) + \mu^Tg(x) + \lambda^Th(x)$

- KKT condition :

  1. $$\frac{\partial L(x, \mu, \lambda)}{\partial x} = 0$$

  2. $$g_i(x)\leq 0, \forall i,\;\; h_j(x) = 0, \forall j$$

  3. $$\mu_i \geq 0, \forall i$$

  4. $$\mu_i g_i(x) = 0, \forall i$$

## LQR for Discrete Time & Finite Horizon

### Problem Setting

- $x_{\mathrm{des}} = 0, u_{\mathrm{des}} = 0$ 인 경우, 아래와 같이 문제를 정의할 수 있다.

- $$\underset{x_{0,1,...,N}, u_{0,1,...,N-1}}{minimize} \frac{1}{2}x_{N}^{T}Q_{f}x_{N} + \sum_{i=0}^{N-1}\left[\frac{1}{2}x_{i}^{T}Qx_{i} + \frac{1}{2}u_{i}^{T}Ru_{i} \right] \\
s.t. \quad x_{i+1} = Ax_i + Bu_i, \; i = 0,1,...,N-1, \\
x_0 =x_{\mathrm{init}}.$$

- 만일 $x_{\mathrm{des}} \neq 0$ 또는 $u_{\mathrm{des}} \neq 0$ 인 경우라 하더라도, $x = \hat{x} - \hat{x}\_{\mathrm{des}}$, $u = \hat{u} - \hat{u}\_{\mathrm{des}}$ ($\hat{x}, \hat{u}$ 이 실제 state 및 input) 으로 치환하면 위의 식을 통해 control을 구할 수 있다.

### Lagrangian Function 및 Hamiltonian Matrix

- Lagrangian function : $L(x, u, \lambda) := \frac{1}{2}x_{N}^{T}Q_{f}x_{N} + \sum_{i=0}^{N-1}\left[\frac{1}{2}x_{i}^{T}Qx_{i} + \frac{1}{2}u_{i}^{T}Ru_{i} + \lambda_i \cdot (Ax_i + Bu_i - x_{i+1}) \right]$
- Hamiltonian matrix : $H(x_i, u_i, \lambda_i) := \frac{1}{2}x_{i}^{T}Qx_{i} + \frac{1}{2}u_{i}^{T}Ru_{i} + \lambda_i \cdot (Ax_i + Bu_i)$

여기서 구한 Lagrangian 을 KKT conditions에 적용하여 control을 구할 수 있다. 먼저 $x$와 $u$에 대한 1차 미분이 0이 나와야 한다.

$$test5$$

$$
x = \hat{x} - \hat{x}_{\mathrm{des}}, u = \hat{u} - \hat{u}_{\mathrm{des}}
$$
