# Autonomous Drug Discovery Lab

**A virtual, closed-loop discovery workflow using machine learning and simulated experiments.**

This project demonstrates how an iterative discovery campaign can use experimental feedback to select promising candidates. A synthetic library of 1,000 hypothetical candidates is screened using three strategies: model-guided exploitation, uncertainty-aware upper confidence bound (UCB) selection, and random screening.

> **Scope:** This is a computational teaching and benchmarking project. Candidate descriptors, performance scores, and experiments are simulated; they do **not** represent measured lipid delivery, drug efficacy, or validated molecular structures.

## Project overview

The virtual lab follows a design–test–learn loop:

1. Generate a library of 1,000 hypothetical candidates, each described by four numeric features.
2. Simulate an initial experimental screen of 30 randomly selected candidates.
3. Train a random forest regression model on observed performance.
4. Select 20 new candidates per round using one of three strategies.
5. Simulate noisy experimental measurements, update the training data, and retrain the model.
6. Repeat for 10 rounds and compare discovery outcomes.

Each strategy has a total budget of **230 tested candidates** (30 initial + 10 × 20 additional). Candidate selection uses only descriptors and previously observed measurements, not the simulator's hidden true-performance values.

## Candidate descriptors and synthetic performance

| Feature | Simulated range |
| --- | --- |
| Lipophilicity | 0–10 |
| Molecular size | 300–1,000 |
| Charge | −2 to 2 |
| Flexibility | 0–1 |

The hidden performance function rewards candidates near chosen feature optima, with a multiplicative flexibility contribution. Simulated experimental measurements add Gaussian noise (standard deviation **0.03**) and are clipped to the interval [0, 1]. The hidden function is used to **evaluate** discovery success, not to train the model.

These variables are abstract descriptors; they should not be interpreted as experimentally characterized molecular properties.

## Selection strategies

- **AI (exploitation):** Select candidates with the highest predicted performance from a 200-tree random forest regressor.
- **UCB (exploration + exploitation):** Rank candidates by `predicted performance + β × uncertainty`, with **β = 1.0**. Uncertainty is approximated by the standard deviation of predictions across trees; it is **not calibrated predictive uncertainty**.
- **Random:** Select 20 untested candidates at random per round.

All strategies begin from the same 30-candidate initial screen within each simulation replicate.

## Results

The comparison was repeated over **20 simulations** with different initial screens and random seeds, using the **same underlying 1,000-candidate library and hidden performance function**.

### Best candidate found

| Strategy | Mean best true performance | SD | Mean library optimum recovered |
| --- | ---: | ---: | ---: |
| AI (exploitation) | 0.814 | 0.000 | 100.0% |
| UCB | 0.814 | 0.000 | 100.0% |
| Random | 0.695 | 0.082 | 85.5% |

### Efficiency: reaching at least 95% of the library optimum

| Strategy | Success rate | Mean completed-batch budget among successes | Median completed-batch budget among successes |
| --- | ---: | ---: | ---: |
| AI (exploitation) | 100% (20/20) | 71 | 70 |
| UCB | 100% (20/20) | 75 | 70 |
| Random | 10% (2/20) | 130 | 130 |

**Interpretation:** Both AI strategies consistently reached the target and outperformed random screening in this synthetic benchmark. UCB did **not** show an advantage over exploitation: both reached the same final optimum in all 20 runs, while exploitation used a slightly lower mean completed-batch budget. Random screening's mean/median budget is calculated only from its two successful runs and should not be interpreted as an unconditional average.

### Important methodological notes

- The synthetic performance landscape is smooth and relatively easy for a random forest to learn. It is not evidence of performance on real drug discovery data.
- All 20 replicates reuse the same candidate library and underlying performance function. Replicates vary initial selections and simulated experimental outcomes, **not** the biological landscape.
- The completed-batch budget counts all 20 candidates in a screening round, even when the target is found partway through that round.
- The current simulator draws experimental noise separately within each strategy's campaign. A future version can assign a shared, reproducible measurement to each candidate for stricter paired comparisons.
- The results are descriptive; no statistical significance claim is made.

## Repository structure

```text
autonomous-drug-discovery-lab/
├── notebooks/
│   └── 01_virtual_lab.ipynb
├── README.md
├── requirements.txt
└── .gitignore
```

## Getting started

Requires **Python 3.11** (or a compatible recent Python version).

### Windows PowerShell

```powershell
git clone https://github.com/naveenreddy88/autonomous-drug-discovery-lab.git
cd autonomous-drug-discovery-lab
py -3.11 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Open the folder in **VS Code**, open `notebooks/01_virtual_lab.ipynb`, select the `.venv` Python kernel, and run cells from top to bottom.

If you use another operating system, activate its virtual environment and install the same packages using `python -m pip install -r requirements.txt`.

## Tools

Python, NumPy, pandas, Matplotlib, scikit-learn, and Jupyter notebooks (via VS Code).

## Future work

- Benchmark on harder synthetic landscapes with multiple optima and descriptor interactions.
- Compare additional acquisition functions and tune the exploration parameter.
- Use paired, candidate-specific measurement noise and larger numbers of simulation replicates.
- Add experimental failures, costs, and realistic constraints.
- Evaluate on suitable real experimental datasets, with appropriate validation.

## Reproducibility

The notebook uses explicit random seeds for library generation, initial screens, and simulated campaigns. Reproducing the reported values requires running the notebook in order with the same simulation settings and compatible library versions. The notebook should be saved with all final analysis cells before publishing.
