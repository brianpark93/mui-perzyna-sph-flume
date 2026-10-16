# Rate-dependent mu(I)-Perzyna SPH granular flume: five models, seven slopes

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
number: it rises from 29.5 deg at a 30 deg chute to 45.1 deg at 60 deg. That
trend is usually read as a missing rate dependence, so the obvious fix is a
rate-dependent constitutive model.

**The obvious fix mostly does not work**, and that is the result this page
exists to let anyone check. Of the three rate-dependent formulations tried,
two score *below* the rate-independent baseline they were built to beat.

| Model | 30 | 35 | 40 | 45 | 50 | 55 | 60 | Mean | Worst |
|---|---|---|---|---|---|---|---|---|---|
| Mohr-Coulomb, one fixed phi | 60.3 | 85.3 | 91.3 | 90.8 | 87.0 | 79.7 | 78.9 | 81.88 | 60.3 |
| mu(I) + Cross model | 65.2 | 54.2 | 66.8 | 86.5 | 87.0 | 92.5 | 89.0 | 77.32 | 54.2 |
| mu(I) + Perzyna, N = 1 | 73.5 | 58.0 | 67.1 | 90.3 | 90.1 | 89.4 | 82.2 | 78.66 | 58.0 |
| **mu(I) + Perzyna, N = 3.71** | 84.3 | 81.6 | 86.1 | 83.9 | 88.1 | 87.7 | 87.6 | **85.63** | **81.6** |
| Mohr-Coulomb, phi per angle | 82.1 | 86.5 | 92.4 | 91.7 | 89.3 | 90.5 | 92.0 | 89.20 | 82.1 |

Every model in the upper block uses **one parameter set for all seven
slopes**. The last row fits a separate friction angle at each slope &mdash;
seven free parameters against the power-law model's five &mdash; and is a
ceiling, not a rival.

Read the **Worst** column rather than the Mean. The power-law model wins
outright at exactly one slope of seven (30 deg, where its 84.3 beats even
the per-angle reference) and is beaten at the other six. Being best
somewhere is not the claim; being adequate everywhere on one parameter set
is.

## The score

0 to 100, where 100 is an exact reproduction of the measured profile. It is
the equally weighted mean of five measures of the same final deposit, each
mapped by `S = 100 (1 - e / delta)` clipped to [0, 100]:

* **RMSE** of the deposit surface height over the measured x-range
* **position** and **height** of the deposit's highest point
* **runout**, the furthest x at which the deposit is still 5 mm deep
* **overlap**, the intersection of simulated and measured deposit areas
  divided by their union, entering as 100x that ratio

Tolerances are `delta_z = 0.15 m` for the two height measures and
`delta_x = 1.4 x_max` for the two horizontal ones. A score near 60 is not
"slightly worse" than one near 85: at 30 deg the fixed-phi baseline's 60.3
is a deposit in the wrong place entirely, with the material still on the
chute.

## What the page shows

Particles coloured by speed, one panel per selected model, playing in step.
States are 0.02 s apart, so 1x is half real speed, and the dashed red line
is the measured deposit &mdash; scrub to the end to see what each run was
aiming at.

This is where the models stop looking alike. Both Mohr-Coulomb cases shear
the release into a thin fast sheet and leave the body of the mass behind
(32 and 50 mm thick at 0.9 s, measured normal to the chute), while all
three rate-dependent models stay 69-106 mm thick and move as a body. Among
those three, Cross and N = 1 hold everything above the toe and then
avalanche &mdash; shedding 69-80% of the release in under a second &mdash;
and only N = 3.71 meters the delivery out, which is what the shallow slopes
needed.

Panels stack as rows where the deck is wide (30-40 deg) and run as columns
where it is tall (45-60 deg), so the canvas is used either way. **Overlaid**
puts every selected model in one panel instead.

## Files

| File | Contents |
|---|---|
| `index.html` | the explorer; no dependencies, no build step |
| `index.json` | models, slopes, scores, offsets, measured profiles, deck geometry, colour scales |
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

**The animation and the scores do not always come from the same run.** The
dense re-runs are not bit-identical to the published runs the table reports:
N = 1 re-solves through the current integrator, which reduces to that model
at N = 1 but is not the build its search used (it averages 77.66 against the
published 78.66, with larger per-angle scatter), and the two Mohr-Coulomb
cases are fresh solves rather than the phi-scan entries. The final frame of
an animation can therefore sit a little off the deposit its score was
computed on. The table keeps the published values, which are what those
stages actually achieved.

**The colour scale is set per slope.** Speed rises with the slope &mdash;
the 99th percentile runs 1.8 m/s at 30 deg to 4.3 m/s at 60 &mdash; so one
scale for all seven would saturate every steep panel. Models at the same
slope always share it. Compare colours across models freely; read the number
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

Experiment, SPH discretisation, deposit measures and the composite score are
inherited unchanged from the companion Mohr-Coulomb study, so the numbers
here are directly comparable with it:
<https://github.com/brianpark93/mohr-coulomb-sph-flume-grid>

The Cross-model runs are a **reconstruction**. That stage's executable was
not archived and its regularisation had been replaced in the material
subroutine; it was restored from the surviving formulation and rebuilt, and
reproduces the published per-angle scores to 0.04 (mean 77.32 against
77.31). Everything else is the original solver output.
