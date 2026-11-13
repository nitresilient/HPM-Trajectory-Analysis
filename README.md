# Hand Palm Motion (HPM) gestures: do raw coordinates or simple invariant features survive a reference-frame change?

A small exploratory analysis in Python of a public hand-motion dataset. This is a gesture-recognition dataset, not a motor-learning dataset, so nothing here says anything about learning.

**Author:** Yashi | **Notebook:** `HPM_trajectory_analysis.ipynb`

## Data

- HPM dataset: Hand Palm Motion gestures, Zenodo record 15020058 (DOI: 10.5281/zenodo.15020058), licence CC BY 4.0. [Check the record page and add the full author list.]
- 7 gestures x 5 recorded trials, augmented to 420 trials across 12 "contexts": `Original_1`, `Original_2`, `Slower`, `Faster`, `First_Half`, `Combination`, `displaced_body_1-3` (body frame translated and rotated) and `displaced_world_1-3` (world frame translated and rotated).
- Each trial: time, x, y, z position and a quaternion orientation.
- The data is not included in this repo. Download it from Zenodo.

## Question

When the reference frame changes, do raw coordinates still identify the gesture, and do a few simple frame-independent features do better?

## Methods

- **Raw baseline:** each trajectory time-normalised and resampled to 50 points for all 7 channels.
- **Simple features:** path length, total rotation (sum of angles between successive orientations), straightness (net displacement / path length), net rotation. Duration and speed were left out because they change in the Faster and Slower contexts.
- **Classifier:** standardised logistic regression.
-**Trial-number-based split:** Train on trials 1–3 of one original context and test on trials 4–5 across every context (14 test samples per context). This assumes matching trial numbers identify corresponding source recordings. Correspondence was verified for Original_2 and displaced_world_1, but not for all other contexts.
- **Check of trial correspondence:** for all 35 `Original_2` / `displaced_world_1` pairs, a best-fit rotation and translation (Kabsch alignment) reduced the RMS distance between corresponding points from about 1.36 to 0.0014, and the estimated rotation matches the documented transformation (1 m shift and a 180 degree rotation about Z). So trial numbers correspond for this pair; for the other contexts the correspondence is assumed, not verified.

## Results (accuracy on held-out recordings; chance = 0.14)

| Context | Raw (train Orig_1) | Features (train Orig_1) | Raw (train Orig_2) | Features (train Orig_2) |
|---|---|---|---|---|
| Combination | 0.14 | 0.00 | 0.14 | 0.00 |
| Faster | 1.00 | 0.36 | 1.00 | 0.50 |
| First_Half | 0.57 | 0.00 | 0.57 | 0.00 |
| Original_1 | 1.00 | 0.57 | 1.00 | 0.43 |
| Original_2 | 1.00 | 0.29 | 1.00 | 0.64 |
| Slower | 1.00 | 0.14 | 1.00 | 0.21 |
| displaced_body_1 | 0.14 | 0.21 | 0.14 | 0.50 |
| displaced_body_2 | 0.71 | 0.29 | 0.71 | 0.57 |
| displaced_body_3 | 0.57 | 0.21 | 0.57 | 0.50 |
| displaced_world_1 | 0.14 | 0.29 | 0.14 | 0.64 |
| displaced_world_2 | 0.14 | 0.21 | 0.14 | 0.57 |
| displaced_world_3 | 0.14 | 0.21 | 0.14 | 0.64 |
| **Mean over contexts** | **0.55** | **0.23** | **0.55** | **0.43** |

## What I found

- Raw coordinates are at chance in all three world-frame contexts and in `displaced_body_1`, but near perfect when the frame is unchanged (Original, Faster, Slower). Time-normalised resampling removes speed effects.
- When trained on `Original_2`, the simple features reached 0.57-0.64 in the world-frame contexts and 0.50-0.57 in the body-frame contexts, well above raw coordinates there.
- The same features were worse than raw coordinates when the frame was unchanged, and failed on `First_Half` and `Combination`, where trajectories are truncated.
- The mean over contexts hides this pattern: raw looks better overall (0.55 vs 0.43), but only because most contexts keep the frame.
- Features trained on `Original_1` did much worse (mean 0.23). I do not know why. `Original_2` shows larger path-length and rotation sums than `Original_1`, which might reflect added perturbation, but I have not verified this.

## Limitations

- Only 14 test samples per context, so one sample changes accuracy by 0.07. Differences such as 0.57 vs 0.64 are not meaningful.
- The three world-frame contexts come from the same 14 test recordings, so they are not independent replications.
- One train/test split, 21 training samples, no tuning, no statistical test.
- My features are simple sums that depend on sampling and noise (for example, path length was larger in `Slower`). They are not the frame-invariant measures a careful method would use.

## Possible next steps

- Smooth and resample before computing features.
- Use cross-validation over recordings.
- Try more principled frame-invariant descriptors for rigid-body motion.

## Reproduce

1. Download and unzip the dataset from Zenodo.
2. Set `ROOT` in the first code cell of the notebook to the folder containing the 7 gesture folders.
3. Run all cells (Python 3; numpy, pandas, matplotlib, scikit-learn).
