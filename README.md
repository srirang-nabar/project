Dynamic Pricing Model for Indian Railways
========================================

This repository replicates the dynamic pricing strategy proposed in *Singh, Dhake, and Narayanaswami (2023)* for Indian Railways. The main deliverable is `singh_et_al_replication.ipynb`, which implements the formula

$$
\text{Dynamic Fare} = (S + w \times (1 - T)) \times M \times B
$$

and

$$
\text{Total Fare} = 0.8 \times B + \text{Dynamic Fare}
$$

where:

- $S$ is seat occupancy (0–1)
- $T$ is the time remaining percentage ($\min(days/120, 1)$)
- $w$ is the time weight (default 0.187)
- $M$ is the multiplier (0.5 base with optional +25% adjustments)
- $B$ is the base fare supplied by the user

Key features:

- `DynamicPricingModel` class with adjustable `weight`, `multiplier`, and booking window
- Adjustment flags for weekend, festival week, and overnight runs (each adds 25% to the multiplier)
- Batch pipeline (`calculate_dynamic_fare_pipeline`) that appends fare breakdowns to pandas DataFrames
- Built-in fairness checks and visualizations (dynamic surface, fare evolution, multiplier heatmap)

Project Structure
-----------------

- `singh_et_al_replication.ipynb`: Detailed walkthrough, examples, and plots showcasing the pricing surface, scenario outputs, adjustment impact, and fairness verification.
- `VandeBharat_Complete_Data.csv`: Data for 82 Vande Bharat trains with occupancy and fare data.
- `main.py`: Minimal starter (`Hello from project!`); use it as a template if you want to wrap the model into a CLI or script.
- `pyproject.toml` / `uv.lock`: Poetry-style metadata defining Python >= 3.13 and key dependencies (`jupyter`, `matplotlib`, `pandas`).
- `*.pdf`: Reference papers, including the computed dynamic pricing model and the broader revenue management literature.

Installation
------------

1. Create and activate an environment with Python 3.13+ (matching the project requirement):

```bash
python -m venv .venv
source .venv/bin/activate
```

2. Install the project in editable mode so you can experiment with the notebook/API:

```bash
pip install -e .
```

This installs `pandas`, `matplotlib`, and `jupyter`, which are already pinned in `pyproject.toml`.

3. Launch the notebook environment (VS Code, Jupyter Lab, or Jupyter Notebook) from the repo root:

```bash
jupyter lab
```

Usage
-----

### Notebook

- Open `singh_et_al_replication.ipynb` and run the cells in order. The notebook:
  - Shows default parameter values, fair pricing examples, and batch pipeline usage.
  - Generates `dynamic_fare_surface.png` and `fare_evolution.png` for insights into multiplier regimes.
  - Demonstrates fairness properties via `verify_fairness()` (higher occupancy + less time → higher fare).

- The notebook reuses the `DynamicPricingModel` defined near the top. Pay attention to the `set_adjustments` and `reset_adjustments` helpers before calculating fares.

### Python API

Import and use the core model in scripts or other notebooks:

```python
from singh_et_al_replication import DynamicPricingModel, calculate_dynamic_fare_pipeline

model = DynamicPricingModel(weight=0.187, multiplier=0.5)
result = model.calculate_fare(seat_occupancy=0.5, days_remaining=60, base_fare=1000)
print(result)
```

`calculate_fare` returns a dictionary containing:

- `base_fare`, `dynamic_fare`, `total_fare`
- `fare_multiplier` (total vs base)
- `time_remaining_pct`, `adjusted_multiplier`
- Original `seat_occupancy` and `days_remaining`

`calculate_dynamic_fare_pipeline` expects a DataFrame with columns `seat_occupancy`, `days_remaining`, and `base_fare` (names configurable via kwargs). The function returns the DataFrame augmented with `dynamic_fare`, `total_fare`, `fare_multiplier`, `time_remaining_pct`, and `adjusted_multiplier`.

Adjustment Factors & Bounds
---------------------------

- Each of weekend, festival, and overnight flags adds 25% to the multiplier (stacked multiplicatively).
- Total fare is clamped between 0.8× and 1.4× the base fare to match the paper's bounds.
- These behaviors are exercised in the notebook’s “Impact of Adjustment Factors” section and the batch pipeline examples.

Verification & Visualization
---------------------------

- Fairness checks ensure:
  1. Higher occupancy → higher fare
  2. Less time remaining → higher fare
  3. Combined occupancy/time effect also respects monotonicity
- Visualization cells:
  - 3D surface plot of total fare vs. occupancy × days remaining
  - Heatmap of fare multipliers with contour thresholds at 0.8, 1, 1.2, 1.4
  - Time-series of fare evolution for select occupancy levels
- Outputs are saved as:
  - `dynamic_fare_surface.png` - 3D pricing surface and fare multiplier heatmap
  - `fare_evolution.png` - fare evolution over time for different occupancy levels
  - `vandebharat_fare_distribution.png` - fare multiplier distributions for CC and EC classes
  - `vandebharat_revenue_gain.png` - top 10 trains by revenue gain for CC and EC

Vande Bharat Data Analysis
-------------------------

The notebook includes a complete analysis of the `VandeBharat_Complete_Data.csv` dataset (Sections 9-13):

- **Dataset**: 82 Vande Bharat trains with 20 days of occupancy data (March 31 - April 19)
- **Analysis covers both fare classes**:
  - **Chair Car (CC)**: Using CC Fare and average CC occupancy
  - **Executive Chair (EC)**: Using EC Fare and average EC occupancy
- **Model Parameters**: Weight (w)=0.187, Multiplier (M)=0.5, Max Booking Days=120, Fare Range=0.8× to 1.4×

### Key Results

| Class | Current Avg Fare | Dynamic Avg Fare | Avg Multiplier | Revenue Gain |
|-------|-----------------|-----------------|----------------|--------------|
| Chair Car | ₹1,105 | ₹1,332 | 1.205× | ₹34.5M (69.4%) |
| Executive Chair | ₹2,050 | ₹2,398 | 1.172× | ₹9.3M (81.9%) |

**Combined Impact**: 70.6% revenue increase (₹44M additional revenue per 20-day period)

The analysis demonstrates the revenue potential of implementing dynamic pricing on Vande Bharat trains, with EC class showing higher percentage gains due to typically lower base occupancy.

Contributors
------------

Harsha Boddhu
Spandana Konda 
Srikar Chetti
Gudaru Pragathi
Sanket Wathore
Srirang Nabar

References & Citation
---------------------

- Singh, N., Dhake, N., & Narayanaswami, K. (2023). *A dynamic pricing strategy model for Indian Railways*. Journal of Revenue and Pricing Management. [s41272-023-00450-w.pdf]
- Telematics paper on airline revenue management is provided (`1-s2.0-S0965856408000840-main.pdf`) for additional theoretical context.

Feel free to cite or adapt the notebook as long as you keep the original authorship of the model intact.
