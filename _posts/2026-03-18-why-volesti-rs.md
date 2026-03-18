---
layout: post
title: "Why I am building volesti-rs"
date: 2026-03-18
---

It started with a slow algorithm.

I was implementing a portfolio sampling routine in Python.
The math was clean. The code was readable. And it was painfully,
unusably slow. The kind of slow where you run it, go make tea, come back,
and it is still running.

The obvious fix was to drop into a lower-level language. C++ was the natural
choice as this  is what every quantitative library is built on. But the more I
read C++ code, the more I noticed something that bothered me: memory
management errors are not edge cases in C++.
They are a routine part of working with it. Buffer overflows,
dangling pointers, undefined behaviour these are not just academic concerns. In a trading system, they are the
difference between a correct execution and a silent, catastrophic failure.

That is when I found Rust.

---

## The key insight

Rust makes a promise that C++ cannot: memory safety without a garbage
collector. No null pointer dereferences. No use-after-free. No data races.
The compiler catches these at compile time, before the code ever runs.

For a finance algorithm running in production where a single memory error
can corrupt a risk calculation silently — this is not a nice-to-have. It is
the right foundation.

I also noticed that serious engineering organisations had reached the same
conclusion. Jane Street uses Rust in parts of their infrastructure.
Keyrock, a leading market maker, builds in Rust. A Microsoft engineer I met
on LinkedIn told me he spends his days porting legacy C++ codebases to Rust —
not because Rust is trendy, but because the safety guarantees reduce the cost
of maintaining production systems over time. St. Jude Children's Research
Hospital is building their core cancer research platform in Rust, targeting
HPC integration. The world is not moving toward Rust for aesthetic reasons.
It is moving because the tradeoffs make engineering sense.

---

## Finding the gap

After contributing to the rust-analyzer project — which taught me how large,
idiomatic Rust codebases are structured — I started looking for geometric
sampling libraries on crates.io.

Zero results.

There is nalgebra for linear algebra. There is ndarray for numerical arrays.
But there is nothing for the kind of high-dimensional geometric sampling that
researchers in computational geometry, portfolio theory, and Bayesian
statistics actually need.

Then I found GeomScale.

---

## Why GeomScale

GeomScale maintains volesti — a C++ library that implements state-of-the-art
geometric random walks for sampling from convex polytopes. The mathematics
behind it is genuinely beautiful: the idea that a portfolio with fixed
volatility corresponds to points on an ellipsoid intersecting a simplex, and
that you can sample this geometric object with a Markov chain, is the kind of
thing that makes abstract algebra feel alive.

The volesti codebase is well-established, well-tested, and used in published
research. But it requires a C++ toolchain to build, Rcpp to access from R,
and Cython to access from Python. For a Python data scientist who wants to run
portfolio simulations, the barrier is significant.

I contacted Apostolos Chalkis — the primary author of volesti and the AISTATS
2023 paper on anomaly detection in stock markets — directly on LinkedIn. I
described the idea: port the three core MCMC walks to Rust, add a Python
interface via PyO3, and extend his crisis detection methodology to a
reproducible Rust implementation. He was immediately interested.

---

## Why PyO3, not Cython

The natural question is: why not just wrap the existing C++ with Cython, the
way dingo already does?

The answer is that Cython wraps C++. If the underlying C++ has memory safety
issues, the Python layer inherits them. PyO3 wraps Rust. The memory safety
guarantee propagates up through the binding layer. A Python data scientist
calling `volesti_rs.sample_portfolios()` gets Rust's safety guarantees without
writing a line of Rust.

This is a meaningful difference, not a technical curiosity.

---

## What I have built so far

The proof-of-concept is running. Ball Walk is implemented and benchmarked —
**15 microseconds per sample at 50 dimensions** in release profile. The
portfolio sampling API and copula estimation are ported from
`include/volume/copulas.h` in the volesti source. Eighteen tests pass,
including a PSRF convergence test (Gelman-Rubin R-hat = 1.03) that directly
mirrors volesti's own C++ test suite.

The next steps are Hit-and-Run, Billiard Walk, the PyO3 bindings, and the
crisis detection demo on real ETF data — extending Apostolos's AISTATS 2023
results to a Rust implementation for the first time.

The project continues regardless of GSoC selection. The crate will be
published to crates.io. The gap is real, the foundation is built, and the
work is worth doing.

---

**Repo:** [github.com/akashchakrabortymsc-cmd/Volesti_Rust](https://github.com/akashchakrabortymsc-cmd/Volesti_Rust)

**GSoC 2026 organization:** [GeomScale](https://github.com/GeomScale)
