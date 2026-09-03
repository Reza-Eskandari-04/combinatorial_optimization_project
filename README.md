# Solving College Admissions at Scale: An Integer Programming Approach

Implementing mathematical optimization models to fairly assign students to colleges when traditional matching algorithms fail in real-world complex scenarios, such as tied scores and overlapping quotas.

**Full Implementation** of the paper *"College admissions with ties and common quotas: Integer programming approach"* by Kolos Csaba Ágoston, Péter Biró, Endre Kováts, Zsuzsanna Jankó.

**Coursework for**: Combinatorial Optimization and Network Analysis (Dr. Farnaz Hooshmand Khaligh)

## Problem Description

College admissions systems must match students to universities fairly while respecting preferences and capacity limits. The classic Gale-Shapley algorithm (1962) finds stable matchings efficiently—but it breaks down in real systems like Hungary's, which present two major mathematical challenges:

1. **Tied Scores (Ties)**: When multiple students have identical scores for the last available seat, algorithms face a deadlock. 
   - Should all tied students be rejected to strictly respect capacities (Hungarian policy)?
   - Should all of them be admitted, violating the capacity (Chilean policy)?
   - Or should a lottery decide (Irish policy)?

2. **Shared Capacity Constraints (Common Quotas)**: Multiple universities may compete for shared, overlapping resources (e.g., a faculty-wide quota or government-funded nationwide slots). 
   - This networked capacity structure makes finding a stable matching **NP-hard**.
   - It cannot be solved by greedy deferred-acceptance algorithms and requires a global optimization perspective.

**The Challenge**: Design efficient Integer Programming (IP) formulations that can handle these constraints simultaneously, guarantee market stability (Envy-Freeness and Non-Wastefulness), and enable quantitative policy comparison at scale.

## Implementations

### Dataset Generation

To rigorously test the models, we generated synthetic datasets representing students, colleges, overlapping subject groups (common quotas), and applications. The script `scripts/generate_datasets.py` (utilizing `src/generator.py`) generates instances at three scales:

- **Small**: 10 applicants, 5 colleges
- **Medium**: 50 applicants, 20 colleges  
- **Large**: 1,000 applicants, 20 colleges

Two types of datasets were designed to trigger different algorithmic behaviors:
1. **Strict Dataset**: Artificially avoids score collisions (strict rankings) to test baseline stability.
2. **Ties Dataset**: Scores are rounded to specific intervals to simulate real-world score collisions. Furthermore, it incorporates an 80% binding rule for common quotas to ensure shared capacity constraints are actively triggered.

Datasets are stored as `.json` files and parsed via `src/data_loader.py`. 
Models are compared based on:
- Solution quality (Objective value / Student assignments)
- Computational time (Solver efficiency)
- Strictness of capacity bounds vs. stability guarantees

### Integer Programming Formulations

Implemented comprehensive IP and MIP formulations exploring different logical structures (Continuous vs. Binary Cutoff variables) and policies:

- **Classic Models**: SO-BB, SO-NW-CUT, MIN-CUT, MSMR-CUT, MSMR-EF (Direct Pairwise)
- **Binary Cutoff Innovations**: SO-NW-BIN-CUT, MIN-BIN-CUT, MSMR-BIN-CUT
- **Models with Ties**: SO-H-NW-CUT / SO-H-NW-BIN-CUT (Hungarian Policy), SO-C-NW-CUT / SO-C-NW-BIN-CUT (Chilean Policy)
- **Advanced Network Models (Ties + Common Quotas)**: COM-Hungarian and COM-Chilean 

These formulations are implemented as modular `pyomo` models. A base class handles fundamental sets and variables, while each specific formulation extends it by adding tailored objective functions, auxiliary margin variables ($d_{ij}, e_{ij}^p$), and binary logic. The full implementations are available in the `src/models/` directory.

Each formulation is solved using CPLEX (with fallback to high-performance open-source solvers like HiGHS/GLPK).



## Results

The computational experiments yield results that are highly consistent with the findings of the original paper. Although the synthetic datasets used for this project were scaled down due to hardware and computational limits (meaning exact proportions differ from national-scale data), the relative performance and logical outcomes strongly align with theoretical expectations.

The most important insights regarding policy trade-offs in the presence of ties:

- **H-stability (Hungarian Policy)**: Strictly respects capacity constraints by conservatively rejecting all tied marginal students. It guarantees strong stability but results in unused seats and fewer students matched.
- **L-stability (Chilean Policy)**: Maximizes student admissions by allowing capacity violations for the last tied group. It ensures fairness but breaks strict capacity bounds.
- **Lottery (Irish Policy)**: Breaks ties randomly, acting as a middle ground that perfectly fills capacities without violations, though the randomness introduces variance in fairness.

**Computational Insights**: 
Solution times scale reasonably well for small and medium instances. Even large instances (e.g., 1,000 applicants) remain solvable within practical timeframes using modern IP solvers. Notably, **Binary Cutoff models**—despite generating significantly more variables and constraints—solve much faster than continuous models by providing tighter bounds and eliminating Big-M coefficients.

Another fundamental result confirms a core property of large-scale matching markets: optimizing for *Student-Optimal* versus *Student-Pessimal* stable matchings yields nearly identical assignments, showing that the core stable solution is robust regardless of the objective function's direction.

## Key Learnings

Through this project, I developed a deep understanding of:

1. **Combinatorial Optimization Theory**
   - Mechanics of two-sided matching algorithms and stability concepts.
   - Recognizing NP-hard structures in real-world resource allocation problems.
   - Managing trade-offs between theoretical solution quality and computational tractability.

2. **Integer Programming Modeling**
   - Translating complex, real-world logical conditions into algebraic mathematical formulations.
   - Comparing continuous vs. binary formulations for the exact same problem to evaluate solver efficiency.
   - Utilizing Pyomo and CPLEX/HiGHS in a structured, multi-model architecture.

3. **Software Engineering for Operations Research**
   - Designing modular OR pipelines for extensibility, moving beyond monolithic Jupyter notebooks.
   - Implementing reproducible synthetic data generation using strict seed controls and Python scripts.
