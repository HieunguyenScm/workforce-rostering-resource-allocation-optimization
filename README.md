# Workforce Rostering & Resource Allocation Optimization

## Project Overview

This is an instructor-guided operations analytics project focused on workforce rostering, shift planning, zone coverage, and resource allocation optimization.

The project implements an Excel-integrated Mixed-Integer Linear Programming (MILP) model in Python using Jupyter Notebook and Gurobi solver. The model generates feasible monthly staff-zone rosters while satisfying staffing demand, balancing workload across zones, and enforcing predefined cyclic rotation rules.

This project demonstrates how optimization models can support operational planning and decision-making in environments where workforce allocation, resource constraints, and service coverage are critical.

---

## Business Context

In operations and supply chain environments, workforce planning plays an important role in ensuring that the right number of staff are assigned to the right zones and shifts at the right time.

Manual rostering can be time-consuming and prone to imbalance, especially when planners need to consider:

- Staffing demand by zone and shift
- Staff availability
- Workload balance
- Zone rotation rules
- Multi-day planning periods
- Operational coverage requirements

This project addresses these challenges by using an optimization-based decision-support model to automate and improve staff assignment decisions.

---

## Problem Description

The model considers a finite set of:

- Zones: Cargo, Pax, Vehicles, Train
- Shifts: Morning, Afternoon, Night
- Workdays: Day 1, Day 2, ...
- Staff members: Staff 1, Staff 2, ...

The objective is to assign staff to zones and shifts across a monthly planning horizon while satisfying operational requirements and maintaining balanced staff-zone rotation.

---

## Scope of Work

### 1. Workforce Rostering Optimization

- Formulated a staff assignment problem as a Mixed-Integer Linear Programming model.
- Assigned staff to day-shift-zone combinations based on operational coverage requirements.
- Ensured that each zone and shift receives the required staffing level.

### 2. Resource Allocation & Workload Balancing

- Applied workload balancing logic to avoid unfair concentration of assignments.
- Supported fair staff allocation across operational zones.
- Reduced manual planning effort by generating structured roster outputs.

### 3. Rule-Based Rotation Planning

- Incorporated cyclic rotation rules across zones and planning periods.
- Ensured staff-zone assignments follow predefined rotation logic.
- Improved planning consistency for future roster cycles.

### 4. Excel-Integrated Planning Workflow

- Used Excel files as input and output layers for planning data.
- Connected Excel-based data structures with Python optimization logic.
- Generated roster outputs that can be reviewed and used by planners.

### 5. Operations Analytics & Decision Support

- Translated operational planning requirements into mathematical constraints.
- Evaluated roster feasibility against staffing demand, shift coverage, and rotation rules.
- Built a structured decision-support workflow for resource planning.

---

## Tools & Technologies

- Microsoft Excel
- Python
- Jupyter Notebook
- Gurobi Optimizer
- Mixed-Integer Linear Programming
- Operations Research
- Resource Allocation
- Workforce Planning

---

## Key Files

| File | Description |
|---|---|
| `day_shift_zone_staff_rostering.ipynb` | Main Jupyter Notebook containing the optimization model and logic |
| `input.xlsx` | Excel input file containing planning requirements and roster data |
| `output.xlsx` | Excel output file containing generated staff-zone-shift roster |
| `README.md` | Project documentation and business explanation |

---

## Key Learning Outcomes

Through this project, I strengthened my ability to:

- Translate operational planning problems into structured optimization models.
- Understand how workforce allocation decisions affect service coverage and workload balance.
- Apply MILP modeling logic to solve real-world resource planning problems.
- Integrate Excel-based planning files with Python-based analytics workflows.
- Document business context, model assumptions, input/output structure, and decision-support logic.
- Connect operations research concepts with supply chain and workforce planning applications.

---

## Relevance to Supply Chain & Operations

This project is relevant to supply chain and operations roles because it demonstrates practical capabilities in:

- Operational planning
- Resource allocation
- Capacity and workforce planning
- Optimization-based decision support
- Excel-integrated analytics
- Process automation
- Structured problem solving

Although the project focuses on workforce rostering, the same planning logic can be extended to supply chain use cases such as warehouse labor planning, delivery shift planning, production workforce allocation, and operational capacity balancing.

---

## Source & Attribution

This repository is forked from the instructor’s original public GitHub project:

**Original repository:** `thanhtranviet248/day-shift-zone-staff-rostering`

The original project demonstrates an Excel-integrated MILP optimization model for assigning staff to operational zones and shifts under staffing demand, workload balance, and cyclic rotation constraints.

This fork is used as a portfolio evidence project. My contribution focuses on understanding the model logic, documenting the business context, explaining the input/output structure, and connecting the project to supply chain operations planning and resource allocation.

---

## Disclaimer

This project is used for learning and portfolio demonstration purposes. It is based on an instructor-guided public repository and does not contain confidential company data.
