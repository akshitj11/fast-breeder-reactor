#  Nuclear Fuel Breeding Ratio Calculator
### Simulating U-238 → Pu-239 conversion in India's PFBR using Python

---

---

## What is a Fast Breeder Reactor?

A **Fast Breeder Reactor (FBR)** uses *fast* (unmoderated) neutrons to convert fertile **U-238** into fissile **Pu-239** — creating more fuel than it consumes.

| Feature | Conventional PHWR | Fast Breeder (PFBR) |
|---|---|---|
| Neutron speed | Thermal (~0.025 eV) | Fast (~1 MeV) |
| Coolant | Heavy water (D₂O) | Liquid sodium |
| Fuel | Natural uranium (U-235) | MOX (Pu + U) |
| Breeds fuel? | No | **Yes** |
| Breeding Ratio | < 1 | **≥ 1.05** |

India's three-stage nuclear programme:

```
Stage I  ──►  Stage II  ──►  Stage III
  PHWR          FBR            Thorium
  (U-235)    (Pu-239)         (U-233)
   NOW        ← PFBR →       FUTURE
```

---

## The Transmutation Chain

When U-238 absorbs a fast neutron, it does **not** fission — instead it undergoes a two-step beta decay to become Pu-239:

<img width="1900" height="557" alt="chain" src="https://github.com/user-attachments/assets/f38258cf-050f-4bc2-835c-83dc2cec3bf8" />


```
U-238  +  n  →  U-239  →  Np-239  →  Pu-239
              (β⁻, 23.5 min)  (β⁻, 2.36 days)
```

Both intermediate isotopes are **short-lived** — within a few days of the neutron capture, fissile Pu-239 is produced and available as new fuel.

---

## The Math — Bateman Equations

The time evolution of each isotope's population is governed by a system of **coupled ordinary differential equations** (ODEs) called the **Bateman equations**:

### System of ODEs

$$\frac{dN_{U238}}{dt} = -\sigma_c \cdot \phi \cdot N_{U238}$$

$$\frac{dN_{U239}}{dt} = +\sigma_c \cdot \phi \cdot N_{U238} - \lambda_{U239} \cdot N_{U239}$$

$$\frac{dN_{Np239}}{dt} = +\lambda_{U239} \cdot N_{U239} - \lambda_{Np239} \cdot N_{Np239}$$

$$\frac{dN_{Pu239}}{dt} = +\lambda_{Np239} \cdot N_{Np239} - \sigma_f \cdot \phi \cdot N_{Pu239}$$

### Variable definitions

| Symbol | Meaning | Units |
|---|---|---|
| $N_i$ | Atom density of isotope $i$ | atoms / cm³ |
| $\sigma_c$ | Neutron capture cross-section of U-238 | cm² (barns) |
| $\sigma_f$ | Fission cross-section of Pu-239 | cm² (barns) |
| $\phi$ | Neutron flux | n · cm⁻² · s⁻¹ |
| $\lambda_i$ | Radioactive decay constant of isotope $i$ | s⁻¹ |

### Decay constant formula

$$\lambda = \frac{\ln 2}{t_{1/2}}$$

Where $t_{1/2}$ is the half-life in seconds.

### Reaction rate

The **rate** at which U-238 captures a neutron per unit volume:

$$R_c = \sigma_c \cdot \phi \cdot N_{U238} \quad \text{[reactions · cm}^{-3} \text{ · s}^{-1}\text{]}$$

---

## Key Parameters — PFBR Values

```python
# Physical constants
barn      = 1e-24          # cm² — unit of nuclear cross-section
sigma_c   = 0.30 * barn    # U-238 capture cross-section (fast spectrum)
sigma_f   = 1.80 * barn    # Pu-239 fission cross-section (fast spectrum)
phi       = 3e15            # Neutron flux: 3 × 10¹⁵ n/cm²/s

# Decay constants
lam_U239  = ln(2) / (23.5 * 60)       # U-239  t½ = 23.5 minutes
lam_Np239 = ln(2) / (2.36 * 86400)    # Np-239 t½ = 2.36 days
```

> **Why sodium coolant?**  
> Water slows neutrons to thermal energies, which *destroys* the fast spectrum needed for breeding.  
> Liquid sodium keeps neutrons fast, and has excellent heat transfer at high temperatures (~550°C).

---

## Breeding Ratio (BR)

The **Breeding Ratio** is the fundamental metric of an FBR:

$$\text{BR} = \frac{\text{Rate of Pu-239 produced}}{\text{Rate of Pu-239 consumed}}$$

$$\text{BR} = \frac{\lambda_{Np239} \cdot N_{Np239}}{\sigma_f \cdot \phi \cdot N_{Pu239}}$$

| BR value | Meaning |
|---|---|
| BR < 1 | Converter — consumes more fuel than it makes |
| BR = 1 | Self-sustaining — break-even point |
| **BR > 1** | **True breeder — creates surplus fissile material** |
| BR = 1.15 | PFBR design target |

### Doubling Time

The **doubling time** $T_d$ is how long it takes for the Pu-239 inventory to double.  
It is inversely proportional to **(BR − 1)**:

$$T_d \approx \frac{t_{\text{irradiation}}}{\text{BR} - 1}$$

A shorter doubling time means India can build **more FBRs faster** — using bred plutonium from existing reactors as fuel for new ones.

---

## Simulation Results

### Isotope Populations & Breeding Ratio over 1 Year

Solved using `scipy.integrate.solve_ivp` with the **Radau** stiff ODE solver:

<img width="2020" height="738" alt="bateman" src="https://github.com/user-attachments/assets/51b52e7a-2d6c-4336-a807-affa0deb2592" />


**What to observe:**
- U-238 depletes slowly (long half-life, low capture rate)
- U-239 and Np-239 reach **quasi-steady state** quickly (short-lived)
- Pu-239 builds up continuously as long as U-238 is present
- BR settles to a **steady value** once the chain reaches equilibrium

### Flux Sensitivity

How the Breeding Ratio changes with neutron flux at 180 days:

<img width="1261" height="696" alt="flux_sensitivity" src="https://github.com/user-attachments/assets/426ce59b-7b01-4d5e-8345-6caa2eeb463c" />


**Key insight:** BR peaks in the intermediate flux range. At very high flux, Pu-239 is fissioned faster than it accumulates, and BR drops. The **green shaded zone** is where true breeding occurs (BR > 1).

---

## Doubling Time

How long until the Pu-239 stock doubles at PFBR operating conditions (φ = 3×10¹⁵):

<img width="1261" height="699" alt="doubling_time" src="https://github.com/user-attachments/assets/a1010b3d-05b3-43e2-bcf8-5e70a870365d" />


This is strategically important — each doubling provides enough plutonium to **fuel a new reactor**, enabling India to expand its nuclear fleet using domestically bred fuel rather than imported uranium.

---

## Installation & Usage

```bash
# Clone the repo
git clone https://github.com/your-username/fbr-breeding-ratio
cd fbr-breeding-ratio

# Install dependencies
pip install numpy scipy matplotlib

# Run the simulator
python fbr_simulator.py
```



---

## Project Structure

```
fbr-breeding-ratio/
├── README.md
├── fbr_simulator.py        ← core ODE solver + BR calculator
├── generate_diagrams.py    ← reproduces all plots in /images
├── notebooks/
│   └── exploration.ipynb   ← interactive Jupyter walkthrough
└── images/
    ├── chain.png           ← transmutation chain diagram
    ├── bateman.png         ← isotope populations + BR over 1 year
    ├── flux_sensitivity.png← BR vs neutron flux sweep
    └── doubling_time.png   ← Pu-239 doubling time plot
```

---

## References

| Source | Link |
|---|---|
| IGCAR — FBR Technology Development in India | [ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0149197017300604) |
| India's fast reactor programme — review | [ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0149197020300251) |
| Indian fast reactor: current status | [Springer Sādhanā](https://link.springer.com/article/10.1007/s12046-013-0167-8) |
| PFBR — Wikipedia | [Wikipedia](https://en.wikipedia.org/wiki/Prototype_Fast_Breeder_Reactor) |
| World Nuclear News — PFBR criticality | [WNN](https://www.world-nuclear-news.org/articles/first-criticality-for-indian-fast-breeder-reactor) |

---
