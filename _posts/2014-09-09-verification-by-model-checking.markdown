---
layout: post
title: "verification by model checking"
date: 2014-09-09 15:28:31 -0500
comments: true
categories: formal verification
---


Formal verification techniques can be thought of comprising of three parts:

1. a **framework for modelling systems**, typically a description language.
2. a **specific language** for describing the properties of the world.
3. a **verification method** to establish whether the description of the system satisfies the specification.

Approaches to method verification:

1. Proof Based: Here the system description is a set of formulas *Γ* and specific language is another formula φ. The verification method consists of trying to find a proof that *Γ* |− φ.
2. Model Based: the system is represented by model M, specification is again by a formula φ and the verification method consists of computing whether a model M satisfies φ.

  
