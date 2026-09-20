# EvoPIE experiment workflow notes

These notes preserve the reusable knowledge from historical `experiment.sh` and
`slurm/` scripts. Those scripts were removed because they encoded one-off local
paths, cluster paths, and old job arrays rather than maintained workflows.

## Purpose

The experiment commands exercised the DECA space generator and quiz model
simulation commands from the Flask CLI. They were used to compare randomized,
sampling, and PPHC quiz-selection strategies over generated DECA spaces.

Common output folders were:

- `deca-spaces/` for generated DECA space JSON files.
- `algo/` for serialized algorithm state.
- `results/` for CSV experiment outputs.
- `figures/` for post-processed plots.

These folders are generated artifacts and are ignored by Git.

## Local experiment shape

A typical local run started from a clean database, initialized a course,
created synthetic quiz questions, created students, generated DECA spaces,
and then ran one or more quiz-selection algorithms.

```bash
flask DB-reboot
flask course init
flask quiz init -nq 4 -nd 25
flask student init -ns 100

flask deca init-many -ns 100 -nq 4 -nd 25 \
  -an 10 \
  -as 5 \
  --spanned-strategy NONE \
  --num-spanned 0 \
  --num-spaces 10 \
  --best-students-percent 0.05 \
  --noninfo 0.2 \
  --timeout 1000 \
  --random-seed 29 \
  --output-folder deca-spaces
```

Important space-generation parameters:

- `-ns`: number of synthetic students.
- `-nq`: number of quiz questions.
- `-nd`: number of distractors.
- `-an`: number of axes.
- `-as`: axis size.
- `--spanned-strategy`: strategy for spanned points.
- `--num-spanned`: number of spanned points.
- `--best-students-percent`: share of best-performing students.
- `--noninfo`: non-informativeness rate.
- `--random-seed`: deterministic experiment seed.

Historical runs compared spanned strategies such as `NONE`, `BEST`, `CENTER`,
`2AXES_BEST`, and `RANDOM`.

## Running quiz-model experiments

`flask quiz deca-experiments` accepts a folder of DECA spaces, an output folder
for algorithm state, a results folder, a random seed, a run count, and one or
more algorithm definitions.

```bash
flask quiz deca-experiments \
  --deca-spaces deca-spaces \
  --algo-folder algo \
  --results-folder results \
  --random-seed 23 \
  --num-runs 10 \
  --algo "$RAND_ALGO" \
  --algo "$SAMPLING_ALGO"
```

Where the algorithm values are JSON strings such as:

```text
RAND_ALGO={ "id": "rand-3", "algo": "evopie.rand_quiz_model.RandomQuizModel" }
SAMPLING_ALGO={ "id": "nond-2", "strategy": "non_domination" }
```

Historical algorithm examples included:

- `evopie.rand_quiz_model.RandomQuizModel` with `n` set to 3 or 5.
- `evopie.sampling_quiz_model.SamplingQuizModel` with `strategy` set to
  `non_domination`.
- Sampling variants with `group_size`, `knowledge_annealing`, `alpha`, `beta`,
  `gamma`, `delta`, `reduced_facts`, or `sample_best_one`.
- `evopie.pphc_quiz_model.PphcQuizModel` with `pop_size`, `pareto_n`,
  `child_n`, and `gene_size`.

Single-space runs used `flask quiz deca-experiment` with `--deca-input` instead
of `--deca-spaces`.

## Post-processing metrics

Historical post-processing focused on CSV outputs in `results/`. Common metrics
were:

- `dim_coverage`
- `arr`
- `population_redundancy`
- `deca_redundancy`
- `num_spanned`
- `deca_duplication`
- `population_duplication`
- `noninfo`

Example:

```bash
flask quiz post-process \
  --result-folder results \
  --figure-folder figures \
  --file-name-pattern '.*_20-\d+.csv' \
  -p dim_coverage \
  -p arr \
  -p population_redundancy \
  -p population_duplication \
  -p noninfo
```

`flask quiz plot-metric-vs-num-of-dims` was used to create figures across
folders of experiment results.

## Retired VS Code launch configs

The removed `.vscode/launch.json` file was a personal command scratchpad for
running Flask CLI commands from VS Code. It was not a maintained project API.
The reusable commands it captured belonged to these families:

- local Flask startup with `FLASK_APP=app.py` and `FLASK_DEBUG=1`;
- `flask DB-init`, `flask quiz init`, and `flask student init`;
- `flask quiz run`, `flask quiz result`, and `flask quiz export`;
- `flask deca init`, `flask deca init-many`, and `flask deca result`;
- `flask quiz deca-experiment` and `flask quiz deca-experiments`;
- DECA result plotting, ranks, distributions, and t-test helpers.

The file also contained hardcoded generated paths such as `data/rq*`, `algo/`,
`deca-spaces/`, `results/`, and `figures/`. Treat those as examples of
historical local experiment layouts, not as required repository directories.

## SLURM adaptation

The removed `slurm/` scripts ran the same CLI workflow as array jobs on an HPC
cluster. The reusable pattern was:

```bash
#!/bin/bash
#SBATCH --time=72:00:00
#SBATCH --mem=8G
#SBATCH --array=0-15

cd ~/evopie
DB="$WORK/evopie/data-$SLURM_ARRAY_TASK_ID/db.sqlite"
export EVOPIE_DATABASE_URI="sqlite:///$DB"
export PYTHONPATH=$(pipenv run which python)

pipenv run flask DB-reboot
pipenv run flask course init
pipenv run flask quiz init -nq 4 -nd 25
pipenv run flask student init -ns 100
pipenv run flask quiz deca-experiments \
  --deca-spaces "$WORK/evopie/spaces" \
  --algo-folder "$WORK/evopie/data-$SLURM_ARRAY_TASK_ID/algo" \
  --results-folder "$WORK/evopie/data-$SLURM_ARRAY_TASK_ID/results"
```

Each array task used a separate SQLite database and separate output folders.
The deleted scripts hardcoded historical `$WORK` paths and branch names, so new
cluster runs should create fresh job scripts from the pattern above.
