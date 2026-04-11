# Day–Shift–Zone-Staff Rostering

This project implements a MILP model for assigning staff to operational zones over a monthly planning horizon in a day–shift setting. The approach integrates Excel-based data structures with a Gurobi optimization model and a rule-based model in Python environment to generate feasible and balanced staff–zone assignments subject to demand satisfaction and rotation rules for future periods.

## 1. Problem description

We consider a finite set of zones $Z$, shifts $S$, workdays $T$, and staff $N$.

In the current implementation:

- Zones: $Z = \{\text{Cargo}, \text{Pax}, \text{Vehicles}, \text{Train}\}$.
- Shifts: $S = \{\text{M}, \text{A}, \text{N}\}$.
- Workdays: $T = \{\text{Day 1}, \text{Day 2}, \dots\}$, parsed from the input workbook.
- Staff: $N = \{\text{Staff 1}, \text{Staff 2}, \dots\}$, parsed from the schedule table.

The input data include:

- A staff schedule that specifies, for each staff $i \in N$, day $t \in T$, and shift $s \in S$, whether staff $i$ is scheduled to work shift $s$ on day $t$ (otherwise they are off, denoted by $O$).
- A demand table $d_{zst}$ indicating the required number of staff in zone $z \in Z$ during shift $s \in S$ on day $t \in T$.
- A rotation rule for next-month assignments, represented by a zone successor mapping
  $$
  \sigma : Z \rightarrow Z,
  $$
  currently instantiated as
  $\text{Cargo} \rightarrow \text{Pax} \rightarrow \text{Vehicles} \rightarrow \text{Train} \rightarrow \text{Cargo}$.

The project addresses two questions:

1. **Q1 – Staff-to-zone assignment (optimization-based):**  
   Assign each staff member $i \in N$ to exactly one zone $z \in Z$ for the entire planning horizon such that:
   - The zone–shift–day demand $d_{zst}$ is satisfied, given staff availability by shift and day.
   - The number of staff assigned to each zone is approximately balanced.

2. **Q2 – Staff-to-zone assignment for next month (rule-based):**  
   Given each staff member’s previous-month zone, determine next-month zones via the rotation rule $\sigma$, applying a simple fallback in case of missing historical data.

## 2. Data and implementation structure

### 2.1 Excel input

The model reads a workbook `input.xlsx` containing:

- Sheet `Description`:
  - Table `"Demand Table"` with rows indexed by $(z,s)$ and columns indexed by workdays $t$, providing demand values $d_{zst}$.

- Sheet `Q1 Answer`:
  - Table `"Schedule Table"` with:
    - Staff names in column B.
    - For each day $t$ (columns $C, D, \dots$), entries in $\{M, A, N, O\}$ describing whether staff $i$ is working a given shift or is off.

- Sheet `Q2 Answer`:
  - Table `"Schedule Table"` of the same structure, used as the template to be filled with zone-annotated shifts.
  - Table `"Previous Month Schedule Table"` where rows correspond to staff and cells contain either `O` (off) or entries of the form `"Shift Zone"`. The first non-off entry per staff is used to recover the previous-month zone.

### 2.2 Derived parameters

From these sheets, the following parameters are constructed:

- Availability indicator:
  $$
  a_{ist} =
  \begin{cases}
  1 & \text{if staff } i \text{ works shift } s \text{ on day } t, \\
  0 & \text{otherwise.}
  \end{cases}
  $$

- Demand:
  $$
  d_{zst} \in \mathbb{Z}_{\ge 0}
  \quad \forall z \in Z, s \in S, t \in T.
  $$

- Previous-month zone assignment:
  $$
  p_i \in Z \cup \{\text{None}\}
  \quad \forall i \in N,
  $$
  extracted from the `"Previous Month Schedule Table"`.

## 3. Mathematical formulation (Q1)

### 3.1 Decision variables

The MIP model defines:

- Staff–zone assignment:
  $$
  x_{iz} =
  \begin{cases}
  1 & \text{if staff } i \text{ is assigned to zone } z \text{ for the entire horizon}, \\
  0 & \text{otherwise,}
  \end{cases}
  \quad \forall i \in N, z \in Z.
  $$

- Zone load bounds:
  $$
  L^{\max} \in \mathbb{Z}, \quad L^{\min} \in \mathbb{Z},
  $$
  representing the maximum and minimum number of staff assigned to any zone.

### 3.2 Objective function

The objective is to balance staff loading across zones by minimizing the difference between the maximum and minimum zone loads:

$$
\min L^{\max} - L^{\min}.
$$

This promotes an equitable distribution of staff among zones, subject to feasibility.

### 3.3 Constraints

1. **Unique zone assignment per staff:**

   Each staff member must be assigned to exactly one zone over the entire planning horizon:

   $$
   \sum_{z \in Z} x_{iz} = 1
   \quad \forall i \in N.
   $$

2. **Demand satisfaction:**

   For each zone, shift, and day, the number of staff assigned to the zone and available to work the shift must cover the demand:

   $$
   \sum_{i \in N} a_{ist} \, x_{iz} \ge d_{zst}
   \quad \forall z \in Z, s \in S, t \in T.
   $$

   The left-hand side counts, for each $(z,s,t)$, how many staff are both assigned to zone $z$ and scheduled to work shift $s$ on day $t$.

3. **Definition of zone loads and bounds:**

   The total number of staff assigned to each zone must lie between $L^{\min}$ and $L^{\max}$:

   $$
   \sum_{i \in N} x_{iz} \le L^{\max}
   \quad \forall z \in Z,
   $$
   $$
   \sum_{i \in N} x_{iz} \ge L^{\min}
   \quad \forall z \in Z.
   $$

4. **Integrality:**

   $$
   x_{iz} \in \{0,1\}
   \quad \forall i \in N, z \in Z,
   $$
   $$
   L^{\max}, L^{\min} \in \mathbb{Z}.
   $$

The resulting model is a mixed-integer linear program solvable with standard MIP solvers such as Gurobi.

## 4. Rule-based assignment for next month (Q2)

For the next-month assignment, we do not solve an optimization model. Instead, we use a rule-based mapping based on previous-month zones:

1. Extract each staff’s previous-month zone $p_i$ from the `"Previous Month Schedule Table"` by scanning their row until the first non-off entry `"Shift Zone"` is found.
2. Apply the rotation function $\sigma : Z \rightarrow Z$ to obtain the next-month zone:
   $$
   z_i^{\text{next}} =
   \begin{cases}
   \sigma(p_i) & \text{if } p_i \in Z, \\
   \text{Cargo} & \text{otherwise (fallback).}
   \end{cases}
   $$
3. For each day $t$ and shift entry in the `"Schedule Table"` of `Q2 Answer`, replace shift codes $s \in S$ by the composite label `"s z_i^{\text{next}}"$, keeping off days (`O`) unchanged.

This logic produces a rotation-based zone plan consistent with the current month’s structure.

## 5. Software and solution workflow

The implementation is written in Python and relies on:

- `openpyxl` for reading and writing the Excel workbook (`input.xlsx` → `output.xlsx`).
- `gurobipy` for constructing and solving the MIP model.

The execution workflow is:

1. Load `input.xlsx`.
2. Parse demand, schedule tables, and previous-month assignments.
3. Build and solve the MIP model for Q1 to obtain staff–zone assignments.
4. Embed the assignments as `"Shift Zone"` entries into the `Q1 Answer` sheet.
5. Apply the rotation rule to construct Q2 assignments and write them to the `Q2 Answer` sheet.
6. Save the updated workbook as `output.xlsx`.

## 6. References

- Pinedo, M. (2016). *Scheduling: Theory, Algorithms, and Systems*. Springer.
- Ernst, A. T., Jiang, H., Krishnamoorthy, M., & Sier, D. (2004). Staff scheduling and rostering: A review of applications, methods and models. *European Journal of Operational Research*, 153(1), 3–27.
- Gurobi Optimization, LLC. (2025). *Gurobi Optimizer Reference Manual*. Available at: https://www.gurobi.com
