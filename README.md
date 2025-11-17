# Falcon-Die-Casting-Optimization-Model
Linear programming optimization model for Falcon Die Casting, built using Python, PuLP, and CBC to generate weekly production schedules under machine-hour, capacity, and demand constraints. Final project for IE 4516 (Operations Research) at Northeastern University.

# ------------------------------------------------------------
OR Final Project — Falcon Die Casting LP Model
IE 4516 • Northeastern University
# ------------------------------------------------------------

This repository contains Python implementations of the Falcon Die Casting (FDC) production scheduling problem using linear programming (LP) and solved with PuLP + CBC or HiGHS solvers.

The goal of the project is to determine how FDC should allocate production of 5 parts across 5 machines (over 1 or 12 weeks) while minimizing overtime and satisfying weekly customer demand.

All models include decision variables, constraints, and objective functions defined according to the course requirements.

------------------------------------------------------------
Model Purpose
------------------------------------------------------------

The LP model determines:

How many units of each part to produce

Which machine produces which part

How many hours each machine runs

How much setup time is required

How many overtime hours are needed to meet demand

The model ensures all constraints (machine capacity, setups, yields, and weekly demand) are satisfied.

------------------------------------------------------------
Key Model Components
------------------------------------------------------------
Sets
Parts:    P1, P2, P3, P4, P5
Machines: M1, M2, M3, M4, M5
Weeks:    1 ... 12   (depending on file)

Decision Variables
x[(p, m)]        # Units of part p produced on machine m (Week 1 model)
x[(p, m, w)]     # Units of part p produced on machine m in week w (multi-week)

y[(p, m)]        # 1 if machine m is set up to produce part p (Week 1)
y[(p, m, w)]     # Setup indicator for multi-week models

ot[m]            # Overtime hours used by machine m (Week 1)
ot[(m, w)]       # Overtime hours used by machine m in week w (multi-week)


Meaning:

x = how much we produce

y = whether a machine is set up for a part

ot = overtime hours needed

Parameters
production_rate[(p, m)]   # units/hour for valid (part, machine) pairs
yield_rate[p]             # percent of units that are good
setup_time[(p, m)]        # hours needed to set up machine m for part p
demand[(p, w)]            # required good units for part p in week w

REGULAR_HOURS = 120       # regular capacity per machine per week
MAX_OVERTIME  = 48        # max overtime allowed per machine per week

------------------------------------------------------------
Objective Function
------------------------------------------------------------
Minimize Total Overtime

Used in WeekOneOnly.py and SolveWeekByWeek.py:

prob += lpSum(ot[m] for m in machines)


In multi-week version:

prob += lpSum(ot[(m, w)] for m in machines for w in weeks)


This finds the production schedule that requires the least total overtime while satisfying all constraints.

------------------------------------------------------------
Constraints
------------------------------------------------------------
1. Demand Satisfaction

Good units produced must meet weekly demand.

x[(p,m)] * yield_rate[p] ≥ demand[p]        # Week 1
x[(p,m,w)] * yield_rate[p] ≥ demand[(p,w)]   # Multi-week

2. Machine Time Capacity

Production time + setup time ≤ regular hours + overtime.

sum( x[(p,m)] * machine_time[(p,m)] ) +
sum( y[(p,m)] * setup_time[(p,m)] ) ≤ 120 + ot[m]

3. Overtime Limits
0 ≤ ot[m] ≤ 48

4. Setup Linking

A machine cannot produce a part unless it is set up for it.

x[(p,m)] ≤ BIG_M * y[(p,m)]

------------------------------------------------------------
Repository Structure
------------------------------------------------------------
ORFinalProject
 ┣ WeekOneOnly.py         # Week 1 model, minimizes overtime
 ┣ SolveWeekByWeek.py     # Solves Weeks 1–12 individually
 ┣ SolveForAllWeeks.py    # Attempts to solve all 12 weeks in one model
 ┣ README.md              # Documentation (this file)
 ┗ data                   # Demand tables or input data files

------------------------------------------------------------
Solver Choice (CBC vs HiGHS)
------------------------------------------------------------

CBC is not natively available on Windows ARM devices.
HiGHS is a fully supported alternative that produces equivalent results.

CBC Example
from pulp import PULP_CBC_CMD
solver = PULP_CBC_CMD(msg=True)
problem.solve(solver)

HiGHS Example
from pulp import HiGHS
solver = HiGHS(msg=True)
problem.solve(solver)

------------------------------------------------------------
Summary
------------------------------------------------------------

This project builds full LP-based production schedules for Falcon Die Casting.
Each file demonstrates a slightly different modeling approach:

WeekOneOnly.py → Optimize Week 1

SolveWeekByWeek.py → Optimize each week independently

SolveForAllWeeks.py → Optimize all twelve weeks together

All models ensure:

demand is met,

machine limits are respected,

setups are handled correctly,

and overtime is minimized.
