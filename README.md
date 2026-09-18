# Direct Sequence Spread Spectrum: A Simulation Study

A Jupyter-based simulation of Direct Sequence Spread Spectrum (DSSS) communication,
built from baseband primitives in NumPy. The project demonstrates spreading and
despreading, quantifies processing gain, and evaluates receiver performance under
additive noise, narrowband jamming, and frequency-selective multipath channels.

> **Status:** Proposal / work in progress

---

## Motivation

A conventional BPSK link occupies a bandwidth proportional to its bit rate. DSSS
deliberately multiplies the data by a much faster pseudo-random chip sequence,
spreading the signal across a far wider band and lowering its power spectral
density — often below the noise floor.

The receiver multiplies by the same synchronized sequence. Because the chip
sequence is `±1`, the data collapses back to its original bandwidth, while any
signal *uncorrelated* with that sequence is spread out instead. This asymmetry is
the single mechanism behind three otherwise unrelated properties:

- **Anti-jamming** — a narrowband jammer is spread into a low-level wideband nuisance
- **Low probability of intercept** — the transmitted signal hides under the noise floor
- **Multiple access (CDMA)** — users separated by code rather than time or frequency

The aim of this project is not to restate these claims but to *measure* them, and
to show precisely where spread spectrum helps and where it does not.

---

## Objectives

1. Implement a complete DSSS transmit/receive chain in complex baseband.
2. Generate and characterise m-sequences and Gold codes, verifying their
   autocorrelation and cross-correlation properties.
3. Establish that DSSS provides **no** gain against additive white Gaussian noise,
   confirming simulated BER against the BPSK theoretical bound.
4. Quantify processing gain against tone, partial-band, and pulsed jammers.
5. Demonstrate multipath resilience, and implement a RAKE receiver that converts
   resolvable multipath from a penalty into a diversity gain.
6. (Extension) Evaluate multi-user CDMA capacity and the near-far problem.
7. (Extension) Implement code acquisition and measure sensitivity to timing error.

---

## Theoretical Background

For a bit rate `R_b` and chip rate `R_c`, the **spreading factor** is

```
N = R_c / R_b        (chips per bit)
```

giving a **processing gain**

```
G_p = 10 · log10(N)   dB
```

The transmitted signal for bit `b_k ∈ {−1, +1}` is

```
s[n] = b_k · c[n],    n = kN … (k+1)N − 1
```

and the despread decision statistic at the receiver is

```
z_k = Σ r[n] · c[n]
```

For an m-sequence of length `N`, the periodic autocorrelation is `N` at zero lag
and `−1` at all other lags. This near-ideal thumbtack shape is what allows the
correlator to reject delayed replicas and foreign codes alike.

---

## Planned Experiments

| # | Experiment | Key output | Expected finding |
|---|---|---|---|
| 1 | Spreading / despreading | Time waveforms, PSD before and after | Spectrum flattens and widens by factor `N` |
| 2 | Code properties | Autocorrelation and cross-correlation plots | Peak `N`, sidelobes `−1`; low Gold cross-correlation |
| 3 | AWGN performance | BER vs `Eb/N0` for several `N`, with theory overlay | Curves coincide — **no gain against white noise** |
| 4 | Tone jammer | BER vs jammer-to-signal ratio | Link survives JSR up to ≈ `G_p` dB |
| 5 | Partial-band and pulsed jammers | BER vs jammed fraction / duty cycle | Pulsed jamming defeats spreading alone; motivates coding |
| 6 | Multipath channel | BER vs delay spread, `N = 1` vs `N > 1` | Unspread link shows an irreducible error floor |
| 7 | RAKE receiver | BER vs number of combined fingers | Multipath becomes an energy gain via MRC |
| 8 | CDMA (extension) | BER vs number of active users | Graceful degradation; near-far problem without power control |
| 9 | Synchronisation (extension) | BER vs chip timing offset | Sharp degradation beyond ±½ chip |

---

## Methodology

- **Signal model.** Complex baseband, 4 samples per chip for waveform and spectral
  plots; 1 sample per chip for large Monte Carlo BER runs.
- **Codes.** m-sequences via LFSR with primitive feedback polynomials; Gold codes
  as the XOR of a preferred pair of m-sequences.
- **Channel.** AWGN, plus a sparse tapped-delay-line model with delays specified in
  chip periods and complex tap gains, e.g. `h = [1, 0, 0, 0.7e^{jθ}, 0, 0.4]`.
- **Jammers.** Additive, with power set relative to signal power by a specified JSR.
- **Statistics.** Fully vectorised Monte Carlo; on the order of 10⁶ bits per point so
  that a BER of 10⁻⁴ is resolved with adequate confidence.
- **Validation.** Every simulated curve is checked against a closed-form bound or a
  known limiting case wherever one exists.


**Dependencies:** `numpy`, `scipy`, `matplotlib`, `jupyterlab`

---

## Deliverables

- Seven reproducible notebooks, each self-contained with narrative explanation
- A reusable `dsss` package with unit-tested code generation and correlation routines
- A figure set covering PSD, correlation, BER-vs-`Eb/N0`, BER-vs-JSR, and RAKE gain
- A written report summarising results, including the negative result in Experiment 3

---

## References

1. R. L. Peterson, R. E. Ziemer, and D. E. Borth, *Introduction to Spread Spectrum
   Communications*, Prentice Hall.
2. J. G. Proakis and M. Salehi, *Digital Communications*, 5th ed., McGraw-Hill.
3. A. J. Viterbi, *CDMA: Principles of Spread Spectrum Communication*, Addison-Wesley.
4. S. W. Golomb, *Shift Register Sequences*, Aegean Park Press.

