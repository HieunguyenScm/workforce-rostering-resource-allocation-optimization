# Day–Shift–Zone-Staff Rostering

This project implements a MILP model for assigning staff to operational zones over a monthly planning horizon in a day–shift setting. The approach integrates Excel-based data structures with a Gurobi optimization model and a rule-based model in Python environment to generate feasible and balanced staff–zone assignments subject to demand satisfaction and rotation rules for future periods.

## 1. Problem description

We consider a finite set of zones $Z$, shifts $S$, workdays $T$, and staff $N$.

In the current implementation:

- Zones: $Z = \{\text{Cargo}, \text{Pax}, \text{Vehicles}, \text{Train}\}$.
- Shifts: $S = \{\text{M}, \text{A}, \text{N}\}$.
- Workdays: $T = \{\text{Day 1}, \text{Day 2}, \dots\}$.
- Staff: $N = \{\text{Staff 1}, \text{Staff 2}, \dots\}$.

The input data include:

- A staff schedule that specifies, for each staff $i \in N$, day $t \in T$, and shift $s \in S$, whether staff $i$ is scheduled to work shift $s$ on day $t$ (otherwise they are off, denoted by $O$).
- A demand table $d_{zst}$ indicating the required number of staff in zone $z \in Z$ during shift $s \in S$ on day $t \in T$.
- A rotation rule for next-month assignments, represented by a zone successor mapping $\sigma : Z \rightarrow Z$, currently instantiated as $\text{Cargo} \rightarrow \text{Pax} \rightarrow \text{Vehicles} \rightarrow \text{Train} \rightarrow \text{Cargo}$.

The project addresses two questions: 

(1) Staff-to-zone assignment (optimization-based): assign each staff member $i \in N$ to exactly one zone $z \in Z$ for the entire planning horizon such that the zone–shift–day demand $d_{zst}$ is satisfied, given staff availability by shift and day, and the number of staff assigned to each zone is approximately balanced; 

(2) Staff-to-zone assignment for next month (rule-based): given each staff member’s previous-month zone, determine next-month zones via the rotation rule $\sigma$, applying a simple fallback in case of missing historical data.

## 2. Data and implementation structure

The model reads a workbook `input.xlsx` containing three main sheets. Sheet `Description` includes a table `"Demand Table"` with rows indexed by $(z,s)$ and columns indexed by workdays $t$, providing demand values $d_{zst}$. Sheet `Q1 Answer` contains a table `"Schedule Table"` with staff names in column B and, for each day $t$ (columns $C, D, \dots$), entries in $\{M, A, N, O\}$ describing whether staff $i$ is working a given shift or is off. Sheet `Q2 Answer` contains a `"Schedule Table"` of the same structure to be filled with zone-annotated shifts and a `"Previous Month Schedule Table"` where rows correspond to staff and cells contain either `O` or entries of the form `"Shift Zone"`, from which the previous-month zone is extracted.

From these sheets, the following parameters are constructed. The availability-to-be-assigned indicator is defined as $a_{ist} = 1 \text{ if staff } i \text{ works shift } s \text{ on day } t,\ 0 \text{ otherwise.}$ The demand parameter is defined on a non-negative integer domain as $d_{zst} \in \mathbb{Z}_{\ge 0} \text{ for all } z \in Z, s \in S, t \in T.$ The previous-month zone assignment is $p_i \in Z \cup \{\text{None}\} \text{ for all } i \in N$, where $\text{None}$ denotes missing historical data.

## 3. Mathematical formulation (Q1)

The MILP model defines the following decision variables. The staff–zone assignment variable is $x_{iz} = 1 \text{ if staff } i \text{ is assigned to zone } z \text{ for the entire horizon, } 0 \text{ otherwise, for all } i \in N, z \in Z.$ The zone load bounds are $L^{\max} \in \mathbb{Z}$ and $L^{\min} \in \mathbb{Z}$, representing respectively the maximum and minimum number of staff assigned to any zone.

The objective is to balance staff loading across zones by minimizing the difference between the maximum and minimum zone loads, that is $\min L^{\max} - L^{\min}$. This promotes an equitable distribution of staff among zones, subject to feasibility of the demand and assignment constraints.

The model includes four groups of constraints. 

(1) Unique zone assignment per staff: each staff member must be assigned to exactly one zone over the entire planning horizon, i.e. $\sum_{z \in Z} x_{iz} = 1 \text{ for all } i \in N.$ 

(2) Demand satisfaction: for each zone, shift, and day, the number of staff assigned to the zone and available to work the shift must cover the demand, i.e. $\sum_{i \in N} a_{ist} x_{iz} \ge d_{zst} \text{ for all } z \in Z, s \in S, t \in T.$ 

(3) Definition of zone loads and bounds: the total number of staff assigned to each zone must lie between $L^{\min}$ and $L^{\max}$, i.e. $\sum_{i \in N} x_{iz} \le L^{\max} \text{ for all } z \in Z$ and $\sum_{i \in N} x_{iz} \ge L^{\min} \text{ for all } z \in Z.$ 

(4) Integrality conditions require $x_{iz} \in \{0,1\} \text{ for all } i \in N, z \in Z$ and $L^{\max}, L^{\min} \in \mathbb{Z}$.

## 4. Rule-based assignment for next month (Q2)

For the next-month assignment, the project uses a rule-based mapping instead of solving an optimization model. For each staff $i$, the previous-month zone $p_i$ is extracted from the `"Previous Month Schedule Table"` by scanning the row until the first non-off entry `"Shift Zone"` is found. The rotation function $\sigma : Z \rightarrow Z$ is then applied to obtain the next-month zone, that is $z_i^{\text{next}} = \sigma(p_i)$ if $p_i \in Z$ and $z_i^{\text{next}} = \text{Cargo}$ otherwise (fallback in case of missing or invalid historical data). For each day $t$ and shift entry in the `"Schedule Table"` of `Q2 Answer`, the shift code $s \in S$ is replaced by the composite label `"$s z_i^{\text{next}}"$, while off days `O` are kept unchanged, thereby generating a rotation-based zone plan consistent with the current month’s structure.

## 5. Software and solution workflow

The implementation is written in Python and relies on `openpyxl` for reading and writing the Excel workbook (`input.xlsx` → `output.xlsx`) and `gurobipy` for constructing and solving the MILP model. The execution workflow is: load `input.xlsx`; parse demand, schedule tables, and previous-month assignments; build and solve the MILP model for Question 1 to obtain staff–zone assignments; embed these assignments as `"Shift Zone"` entries into the `Q1 Answer` sheet; apply the rotation rule to construct Question 2 assignments and write them to the `Q2 Answer` sheet; and finally save the updated workbook as `output.xlsx`.
