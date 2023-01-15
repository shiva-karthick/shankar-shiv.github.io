---
layout: post
title:  "Explaining Logistic Regression"
date:   2023-01-15 13:00:00 +0800
categories: machine_learning
mathjax: true
---

- Different Loss Functions
- Cost Function
- checking the model accuracy
- explaining 2 different solutions using least squares regression and gradient descent
- Using a small example, explain all the mathematics crystal clear




Regularization

$w^Tw=w_0^2+w_1^2+w_2^2+...+w_d^2$ - also known as L2 regularization

It encourages $w_0,...,w_d$ to be small chapter 7 slide 28

Cost Function
$argmin\ (Pw - y)^T\ (Pw - y)\ +\lambda w^Tw$