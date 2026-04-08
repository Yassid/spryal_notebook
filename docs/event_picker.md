# `event_picker.ipynb`

Interactive viewer for Spyral output. Lets you click a track on the
kinematics, PID, or excitation-energy-vs-vertex-z plot and see the corresponding
event's pointcloud and cluster decomposition.

## Multi-run loading

Set `run_min` and `run_max` in the first code cell. Each run's
`Pointcloud/run_XXXX.h5`, `Cluster/run_XXXX.h5`, and
`Estimation/run_XXXX.parquet` are opened, the Estimation parquets are
concatenated with an added `run` column, and the h5 file groups are stored in
`cloud_groups[run]` / `cluster_groups[run]` so the picker can fetch the right
event from the right run on demand.

Event numbers restart at 1 in every file, so internally a track is identified
by the `(run, event)` pair, not by `event` alone.

## Excitation energy vs vertex Z

Each loaded run's `InterpSolver/run_XXXX_<ejectile>.parquet` is read; the
residual excitation energy is computed using the same kinematics formula as
`example_analysis.ipynb` and joined back onto the Estimation dataframe on
`(run, event, cluster_label)`. The physics configuration (nuclei, target
material, beam energy, IC/window materials) lives in the "Physics
configuration" cell and **must be kept in sync with `example_analysis.ipynb`**.

## Track tiers

Every track falls into exactly one of three tiers, all of which are pickable:

| Tier             | Meaning                                            | Style on C/D | Style on E    |
|------------------|----------------------------------------------------|--------------|---------------|
| `solver_failed`  | InterpSolver did not produce a row                 | red `x`      | not shown     |
| `cuts_failed`    | Solver row exists but fails the analysis cuts     | grey `.`     | grey `.`      |
| `cuts_passed`    | Solver row exists and passes the analysis cuts    | black `,`    | black `,`     |

The cuts are defined in the "Analysis cuts" code cell as a single
`analysis_cuts` dict — edit it and rerun that cell + the plotting cell to
change the criterion. Available keys:

- `vertex_z_min`, `vertex_z_max` — keep tracks with vertex Z in this range
- `polar_max_deg` — drop tracks with polar angle above this value (set to
  `None` to disable)
- `redchisq_max` — drop tracks with reduced chi-squared above this value
  (set to `None` to disable)

When a track is selected, the status text in the kinematics panel reports both
the solver status and whether the cuts were passed.
