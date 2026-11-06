# Class 5 Code Guide

This guide explains the numerical work in [`Class 5 Code.ipynb`](Class%205%20Code.ipynb). The notebook models a gliding aircraft, compares time-stepping methods, estimates numerical error, and computes flight range up to ground contact.

## Model and state

The state is stored in the order `[v, theta, x, y]`: speed, flight-path angle, horizontal position, and altitude. The notebook sets gravity, trim speed, lift and drag coefficients, and an initial launch state. The governing differential equations are supplied by `rhs_full_phugoid` in `phugoid.py`; that module also supplies the Euler update and a discrete L1 difference helper.

The notebook downloads `phugoid.py` from the Numerical MOOC repository before importing it. This setup cell therefore needs network access the first time it runs, unless the module is already available locally.

## Time integration and method comparison

`rk2_step` implements the explicit midpoint, second-order Runge–Kutta method. It first estimates the state halfway through the step using the slope at the start, then advances a full step using the slope at that midpoint. Euler's method, imported from `phugoid.py`, uses only the starting slope.

The notebook advances both methods over the same two-second interval and stores every state. It plots the paths and the horizontal-position difference. It then repeats RK2 with several step sizes and compares each horizontal-position history with the finest-step history using a discrete L1 difference. A second refinement calculation compares coarse, medium, and fine grids; the ratio of successive differences is used to estimate the observed convergence order. For a method behaving in its expected asymptotic regime, halving the step should reduce the error at a predictable rate.

## Touchdown detection and validation

`locate_touchdown` linearly interpolates between the last above-ground state and the first state at or below ground. This estimates touchdown time and position within the time step rather than treating the below-ground endpoint as the event.

The first range routine is a simple draft that stops at touchdown or at a time limit. The later `state_is_valid` helper rejects states with non-finite values or non-positive speed. `integrate_until_touchdown` adds input checks, validates every computed state, records accepted times and states, and returns a status (`touchdown`, `time_limit`, or `invalid_state`) along with any touchdown data and range. Its checks demonstrate how to distinguish a completed flight from a failed or unfinished integration.

The steady-flight case provides a validation example with an analytic touchdown time and range. Euler and RK2 are run at several time steps and checked against those known values. The baseline launch is then integrated with both methods, their paths are plotted, and computational work is estimated by counting right-hand-side evaluations (one per Euler step and two per RK2 step).

## Reference range and launch search

The reference-range section repeatedly halves the RK2 step and tracks changes in computed range. It accepts a reference only after consecutive refinements change the result by less than a tolerance tied to the requested range accuracy. The resulting `range_reference` is used to assess later candidate calculations, while `dt_search` records the selected search step.

The launch-search section evaluates a grid of 17 initial speeds and 25 launch angles, for 425 total combinations. Each run retains its status and number of steps; only successful touchdowns have a range and can enter the list of five largest ranges. Follow-up checks rerun those leading launches at a finer step and sample nearby angles and speeds to examine time-step sensitivity and local variation.

Finally, the notebook defines a candidate launch and computes its range at progressively finer steps to establish `range_reference_candidate`. The last comparison section tests Euler and RK2 against that reference, halving the step up to twelve times per method. It reports each run's range, absolute difference, step count, right-hand-side work, and status. Failed or unfinished runs do not count as qualifying evidence.

## Reading results

- A `touchdown` status means the ground crossing was detected and interpolated; other statuses do not provide a completed range.
- Range differences and refinement ratios are numerical diagnostics. Their meaning depends on the chosen time-step range and whether the solution is in the expected convergence regime.
- The launch grid is a finite sampling of speed and angle. Its largest sampled range and nearby checks describe that search, not a proof of a global optimum.
- The comparison cells report measurements; interpret them alongside their statuses, tolerances, and computational work.
