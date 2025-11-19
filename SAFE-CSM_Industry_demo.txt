import numpy as np
import matplotlib.pyplot as plt

# ============================================================
# SAFE-CSM (Industry Edition)
# External Observation Model for Contextual Stability
# ------------------------------------------------------------
# This script demonstrates the industrial SAFE-CSM version:
# - 10 independent evaluation factors (J1–J10)
# - Δφ computed as difference between J_affect and J_reflex
# - J_total as a composite stability score
# - A minimal reloop/balance mechanism for SAFE stabilization
#
# All values are FIXED or RANDOMIZED (no learning, no adaptation).
# ============================================================


# Number of steps for the demo
STEPS = 100

# Stability band thresholds
LOWER = -0.3
UPPER = 0.3

# Small fixed correction applied by the SAFE-CSM
EPS = 0.05


def generate_factors():
    """
    Generate 10 independent normalized evaluation factors.
    For industrial SAFE-CSM, these factors do NOT represent internals;
    they are simply observable abstract scores in [-1, 1].
    """
    return np.random.uniform(-1, 1, size=10)


def compute_delta_phi(J):
    """
    Δφ = difference between affective and reflexive components.
    Here:
      J_affect = J[2]
      J_reflex = J[5]
    """
    return J[2] - J[5]


def compute_J_total(J, delta_phi):
    """
    Aggregate stability score J_total.
    This is a simple observable combination:
    mean(J) weighted by (1 - |Δφ|).
    """
    return np.mean(J) * (1 - abs(delta_phi))


def csm_reloop(delta_phi):
    """
    SAFE-CSM correction:
    - If Δφ < LOWER → nudge upward by EPS
    - If Δφ > UPPER → nudge downward by EPS
    """
    if delta_phi < LOWER:
        delta_phi += EPS
    elif delta_phi > UPPER:
        delta_phi -= EPS
    return delta_phi


# =========================
# Simulation
# =========================

delta_phi_list = []
delta_phi_csm_list = []
jtotal_list = []

delta_phi = 0.0
delta_phi_csm = 0.0

for step in range(STEPS):

    # Generate 10 factors
    J = generate_factors()

    # Δφ (raw)
    delta_phi = compute_delta_phi(J)

    # J_total (for plotting only)
    jtotal_list.append(compute_J_total(J, delta_phi))

    # Apply SAFE-CSM correction
    delta_phi_csm = csm_reloop(delta_phi)

    # Store traces
    delta_phi_list.append(delta_phi)
    delta_phi_csm_list.append(delta_phi_csm)


# =========================
# Plotting
# =========================

plt.figure(figsize=(10, 5))
plt.plot(delta_phi_list, label="Δφ (no CSM)", linestyle="--")
plt.plot(delta_phi_csm_list, label="Δφ with SAFE-CSM", alpha=0.8)
plt.axhline(LOWER, color="gray", linestyle=":", label="SAFE-CSM band")
plt.axhline(UPPER, color="gray", linestyle=":")
plt.xlabel("step")
plt.ylabel("Δφ")
plt.title("SAFE-CSM (Industry): Stabilization of Δφ Drift")
plt.legend()
plt.tight_layout()
plt.show()
