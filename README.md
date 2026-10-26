# Rate-dependent Drucker-Prager-Perzyna SPH granular flume: five models, seven slopes

**Interactive explorer:
<https://brianpark93.github.io/mui-perzyna-sph-flume/>** &mdash; play the
granular flow itself, model against model, at any of the seven slopes.

Every panel is a solved LS-DYNA SPH run of the same dry sand flume. Nothing
on the page is interpolated, fitted or predicted.

The page deliberately does **one** thing. The deposits these models leave
are already figures in the paper, and a static comparison is better typeset
on a page than in a browser; what a figure cannot carry is the time in
between, which is the only reason this exists.

## The question this is published for

A Mohr-Coulomb friction angle calibrated against this flume is not one
number: it rises from 30.8 deg at a 30 deg chute to 43.8 deg at 60 deg. That
trend is usually read as a missing rate dependence, so the obvious fix is a
rate-dependent constitutive model.

**Rate dependence pays, but only a weak one**, and that is the result this
page exists to let anyone check. The three rate-dependent formulations do
not line up with how strongly each depends on the rate. The strongest of
them scores *below* the rate-independent baseline it was built to beat; the
weakest of the three wins.

| Model | 30 | 35 | 40 | 45 | 50 | 55 | 60 | Mean | Worst |
|---|---|---|---|---|---|---|---|---|---|
| Mohr-Coulomb, one fixed phi | 42.9 | 78.4 | 88.1 | 82.6 | 80.5 | 70.3 | 68.0 | 72.97 | 42.9 |
| Cross regularization | 60.7 | 54.6 | 69.0 | 76.6 | 74.7 | 78.1 | 84.3 | 71.14 | 54.6 |
| Perzyna overstress, N = 1 | 83.2 | 75.3 | 77.4 | 77.0 | 74.1 | 81.0 | 82.8 | 78.70 | 74.1 |
| **Perzyna overstress, N = 3.4** | 84.0 | 78.0 | 84.0 | 78.6 | 78.1 | 82.0 | 80.9 | **80.82** | **78.0** |
| Mohr-Coulomb, phi per angle | 82.8 | 82.3 | 88.5 | 83.3 | 81.2 | 85.1 | 84.1 | 83.91 | 81.2 |

All four models in the upper block carry a **constant** friction
coefficient. Setting phi_2 = phi_s makes the mu(I) closure identically
mu_s, and measured over every particle and every state the closure raises
the yield stress by 0.4% at the inertial numbers these flows reach, so
none of them uses it.

Every model in the upper block uses **one parameter set for all seven
slopes**. The last row fits a separate friction angle at each slope &mdash;
seven free parameters against the power-law model's five &mdash; and is a
ceiling, not a rival.

Read the **Worst** column rather than the Mean. The power-law model wins
outright at two slopes of seven (30 deg, where its 84.0 beats even the
per-angle reference, and 55 deg) and is beaten at the other five, by at
most 4.1 points. A fixed friction angle, beaten at 30 deg, is beaten there
by 41.1. Being best somewhere is not the claim; being adequate everywhere
on one parameter set is.

## The score

0 to 100, where 100 is an exact reproduction of the measured profile. It is
the equally weighted mean of **three** measures of the same final deposit:

* **RMSE** of the deposit surface height over the measured x-range, mapped
  by `S = 100 (1 - e / delta_z)` with `delta_z = 0.15 m`, clipped to [0, 100]
* **deposited area**, the same mapping applied to
  `|A_sim - A_exp| / A_exp`
* **overlap**, the intersection of simulated and measured deposit areas
  divided by their union, entering as 100x that ratio

The companion Mohr-Coulomb study scores the same experiment on a mean of
**five**, the other two being the position and the height of the deposit's
highest point, with the runout as a third single-point measure. Those three
are dropped here because they are where the noise is. Six solves differing
only in the overstress exponent at its fifth decimal place -- a relative
change of 3e-6 -- leave deposits that are barely distinguishable, yet their
five-measure scores at one slope have a standard deviation of 0.96 points
against 0.79 for these three. More to the point, scoring all five models
under each candidate, the smallest gap between two adjacent models is 1.83
points on the three measures against 0.85 on the five: six resolutions
rather than two. **Numbers here are therefore not comparable with the
companion repo's.**

A score near 43 is not "slightly worse" than one near 78: at 30 deg the
fixed-phi baseline's 42.9 is a deposit in the wrong place entirely, with the
material still on the chute.

## What the page shows

Particles colored by speed, one panel per selected model, playing in step.
States are 0.02 s apart, so 1x is half real speed, and the dashed red line
is the measured deposit &mdash; scrub to the end to see what each run was
aiming at.

This is where the models stop looking alike. At 30 deg, both Mohr-Coulomb
cases shear the release into a thin fast sheet and leave the body of the
mass behind &mdash; 50 and 47 mm deep at 0.9 s, measured normal to the chute
at the along-chute middle of the moving material &mdash; while all three
rate-dependent models move as a body 92-132 mm deep. Among those three, the
Cross model avalanches: it holds the whole release above the hinge at 1.5 s,
has shed all but 15% of it by 2.4 s, and comes to rest with 2% left on the
chute, carrying the mass to 0.88 m against a measured toe at 0.33 m. Both
overstress models meter the delivery out instead, still holding 80% at 2.4 s
and finishing with 55% on the chute.

What the 30 deg panels do **not** separate is N = 1 from N = 3.4: they
release within 0.08 s of each other, leave the same fraction on the chute
and end 6 mm apart. Letting the stress exceed yield at all is what rescues
the shallow slope; the exponent earns its margin at 40-50 deg, and it earns
it in the deposit's height rather than in where the material is put.

Panels stack as rows where the deck is wide (30-40 deg) and run as columns
where it is tall (45-60 deg), so the canvas is used either way. **Overlaid**
puts every selected model in one panel instead.

## Files

| File | Contents |
|---|---|
| `index.html` | the explorer; no dependencies, no build step |
| `index.json` | models, slopes, scores, offsets, measured profiles, deck geometry, color scales |
| `deposits.bin` | the 35 final deposits, 89 KB |
| `flow_<model>.bin` | one model's dense time series, fetched only when that model is asked for |

### `deposits.bin`

Little-endian `int16` pairs &mdash; x then z, in millimetres &mdash; with the
35 cells laid end to end. `index.json` carries `o` (offset, in particles) and
`n` (count) per cell:

```js
const P = new Int16Array(await (await fetch("deposits.bin")).arrayBuffer());
const c = d.deposits["perzynaN"]["30"];
for (let i = 0; i < c.n; i++) {
  const o = (c.o + i) * 2;
  const x = P[o] / 1000, z = P[o + 1] / 1000;   // metres
}
```

These are the particles `keep_largest_cluster` retains, which is the set
every reported score is computed on. The page itself uses this file only to
fix the drawing frame and to label each model with its score; it is here
because it is the data behind the table above, and it is 89 KB.

### `flow_<model>.bin`

One file per model, so the page downloads only what is being compared.
Positions first &mdash; `int16` millimetre pairs, every state of every slope
laid end to end &mdash; then the speed of each of those particles as `int16`
millimetres per second. The particle count is **fixed across time** within a
slope, so a frame is a stride rather than a per-frame offset:

```js
const m = d.flow["perzynaN"]["30"];            // {o, nt, np, t0, dt, so}
const pos = (m.o + frame * m.np) * 2;          // x,z pairs
const spd =  m.so * 2 + m.o + frame * m.np;    // one value per particle
```

`o` and `so` count particle slots, not `int16` elements. Millimetres in
`int16` loses nothing: the particle spacing in the model is 10 mm.

## Two things to know before reading too much into it

**Flow coverage is 23 of the 35 cells.** The three rate-dependent models
were re-solved at 0.02 s output for all seven slopes; the two Mohr-Coulomb
cases only at 30 deg, which is the angle the paper's velocity-field figure
uses. The page greys out a model that has no dense run at the selected
slope rather than failing.

**The animation and the score come from the same model, and all but
agree.** The dense re-runs differ from the calibration runs only in how
often the state is written, and the Cross branch is solved by a restored
build rather than the one its search used. Scored, all three reproduce
their own cell to within 0.05 points at every slope &mdash; means of 71.14,
78.70 and 80.82 against 71.14, 78.69 and 80.80. The two Mohr-Coulomb cases
are fresh solves at the same friction angle as the scan entry nearest the
reported value, so their final frame can sit a fraction off the deposit the
table's number was computed on.

**The color scale is set per slope.** Speed rises with the slope &mdash;
the 99th percentile runs 1.75 m/s at 30 deg to 4.5 m/s at 60 &mdash; so one
scale for all seven would saturate every steep panel. Models at the same
slope always share it. Compare colors across models freely; read the number
on the bar before comparing them across slopes.

## Sharing a particular comparison

The page keeps its state in the query string, so a view can be sent to
someone:

```
?angle=60&models=cross,perzyna1,perzynaN&t=40
?angle=30&models=mc_fixed,perzyna1,perzynaN&t=75&layout=overlay
```

`t` is the frame index, not seconds (states are 0.02 s apart).

## Provenance

Experiment, SPH discretization and deposit measures are inherited unchanged
from the companion Mohr-Coulomb study. The figure of merit is not: it is the
three of that study's five measures that do not depend on single points of
the surface, so the numbers here are **not** directly comparable with it:
<https://github.com/brianpark93/mohr-coulomb-sph-flume-grid>

Both Mohr-Coulomb references are the maximum of a one-degree local average
of the phi-only scan rather than its largest single sample. Taking the
largest of 214 noisy samples estimates the best achievable score plus the
largest upward excursion of the noise; the correction is worth 1.14 points
on the per-angle ceiling and 0.39 on the fixed-phi baseline.

The Cross-model runs are a **reconstruction**. That stage's executable was
not archived and its regularization had been replaced in the material
subroutine; it was restored from the surviving formulation and rebuilt, and
reproduces the search's per-angle scores to 0.05 (mean 71.14 against
71.14). Everything else is the original solver output.
