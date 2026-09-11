# Broyden's PoC

C# implementation of Broyden's method for finding the fixed point of a function `f(x) = x`, extracted from the Integrated Geometallurgical Simulator (IGS).

## Context

The system being solved was a simulation of an industrial process plant: processing units (selective splitters) whose performance depended on the flows around the circuit and the flows that were the result of the splits at each unit. The plants had layers of recycle, so flows depended on flows as well.

Naive iteration didn't converge — it bounced between solutions. Later we found out the model math in fact had two solutions under some conditions, which is a neat analogy to the "poorly phrased prompt" problem: two valid readings, and the solver oscillates between them instead of committing. Broyden's method fixes it by reshaping the local landscape so one basin wins.

## How it works

Solves `g(x) = f(x) - x = 0` using a quasi-Newton iteration with an inverse Jacobian update (Broyden's "good" formula). Each step:

1. Evaluate `g(x)` at the current point
2. Compute the step `Δx = J⁻¹ · g(x)` using the maintained inverse Jacobian
3. Update the inverse Jacobian via the rank-1 Broyden formula
4. Apply bounds checking and backtracking to keep the iteration stable

## Key features

- **Inverse Jacobian maintenance** — avoids explicit matrix inversion; updated in-place each iteration
- **Backtracking** — kicks in after 50 iterations on difficult systems; scales the step by 0.9 until the residual decreases
- **Bounds enforcement** — steps that push variables out of bounds are halved until feasible
- **NaN guard** — clears the Jacobian if an update produces NaNs rather than throwing
- **Pre-allocated vectors** — all working vectors are allocated once in the constructor; no per-iteration heap traffic

## Files

- `BroydensMethod` — the `Broyden` class (C#, no extension; original IGS source layout)
