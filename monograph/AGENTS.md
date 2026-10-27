# AGENTS.md

## Project objective

This repository contains an MBA monograph project based on a historical
Internet traffic time series from a PTT/IX.

The primary objective is to develop and evaluate time-series forecasting
models for Internet traffic, with particular attention to traffic peaks and
their relevance to network planning.

Exploratory analyses should support this objective by helping characterize:

- long-term traffic evolution;
- peak behavior;
- relevant temporal patterns and periodicities;
- appropriate temporal resolution;
- useful forecasting features;
- appropriate training windows;
- forecasting performance, including performance during high-demand periods.

Anomaly investigation is complementary and optional. It is not the primary
objective of the project.

---

## Project context and sources

Do not reconstruct the entire project context from the repository before
answering a request.

For decisions involving temporal splitting, leakage prevention,
training/validation/test periods, anomaly treatment, and forecasting
evaluation, consult:

- `DECISOES_RECORTE_TEMPORAL.md`

For analysis-specific methodology:

- inspect only the notebook relevant to the current request;
- use the objective and methodological notes documented inside that notebook;
- do not inspect other notebooks unless they are required to answer the
  question.

Do not treat old planning documents or generic reference material as sources
of truth for the current state of the project.

If a methodological suggestion conflicts with a documented project decision,
point out the conflict instead of silently changing the approach.

---

## Temporal methodology

Do not invent or redefine the temporal split.

The current temporal strategy is documented in
`DECISOES_RECORTE_TEMPORAL.md`.

Important principles:

- preserve chronological order;
- never randomly shuffle time-series observations for model evaluation;
- avoid temporal leakage;
- use training data for data-dependent discovery;
- use validation data for model and methodological choices;
- do not use the final test period to select features, frequencies,
  preprocessing rules, anomaly treatments, models, or hyperparameters;
- keep the final test evaluation separate from model development.

If exact periods or alternative training windows are needed for the task,
consult `DECISOES_RECORTE_TEMPORAL.md` rather than relying on memory.

---

## Methodological suggestions

Methodological recommendations must be tied to the research objective.

Before suggesting a new analysis, consider:

1. What research question would it answer?
2. Does it contribute to forecasting traffic or understanding peak demand?
3. Is that question already answered by an existing analysis?
4. Will the result affect a later modeling or evaluation decision?

Do not recommend analyses merely because they are standard time-series
techniques.

Prefer the simplest method that adequately answers the research question.

Distinguish clearly between:

- exploratory evidence;
- descriptive findings;
- hypotheses;
- methodological decisions;
- results validated out of sample.

Do not infer causality from temporal coincidence.

Possible explanations such as changes in user behavior, streaming adoption,
COVID-19, infrastructure expansion, or other external events should be
presented as hypotheses unless supported by evidence.

---

## Traffic metrics and interpretation

Keep conceptually different quantities separate.

In particular, distinguish between:

- overall traffic level;
- typical traffic behavior;
- daily or intraday peaks;
- extreme observations;
- periodicity and seasonality;
- forecasting targets and forecasting features.

Use absolute traffic values such as Gbps when the research question concerns
traffic magnitude, demand, peaks, or capacity-planning relevance.

Normalized metrics may be used when comparing temporal shapes across periods
with very different traffic scales.

When extreme observations strongly affect summaries, robust statistics such
as the median may be appropriate.

Do not silently change an established normalization or aggregation rule.
Explain why a different statistic is needed before proposing the change.

A high traffic value or peak is not automatically an anomaly or measurement
error.

Do not recommend removing observations solely because they are extreme.

---

## Frequency analysis

Frequency-domain analysis is exploratory and should support forecasting
decisions rather than become an independent objective.

When working on frequency analysis:

- distinguish discovery from out-of-sample verification;
- identify candidate periodicities using training data;
- do not select forecasting features using the final test period;
- treat spectral persistence as exploratory evidence;
- do not assume that a spectral peak improves forecasting performance;
- validate the predictive usefulness of frequency-derived features
  out of sample.

The detailed methodology for a frequency-analysis notebook should be obtained
from that notebook itself rather than duplicated here.

---

## Forecasting

Forecasting evaluation must be out of sample and respect temporal ordering.

When models are introduced:

- include simple seasonal or persistence baselines;
- compare more complex models against those baselines;
- evaluate models on the same temporal periods and forecast horizons;
- avoid using information unavailable at the forecast origin.

Model evaluation should not rely only on average error.

When appropriate, consider:

- MAE;
- RMSE;
- errors during high-demand periods;
- peak underestimation;
- performance around extreme but legitimate traffic events.

If probabilistic forecasts are used, also consider interval calibration or
coverage.

Do not select a model based on final-test performance.

---

## Anomalies and data quality

Anomaly detection is secondary to the forecasting objective.

Potential anomalies may be investigated when there is evidence of:

- measurement errors;
- collection failures;
- impossible values;
- unusual events that materially affect modeling.

Do not automatically classify historical peaks as errors.

If anomaly treatment is proposed:

- preserve the original observations;
- document the criterion;
- fit any data-dependent rule using training data only;
- evaluate whether the treatment improves validation performance;
- do not clean validation or test data merely to improve metrics.

If reliable criteria for distinguishing errors from real events are not
available, prefer retaining the observations and documenting the limitation.

---

## File modification policy

Do not modify repository files unless the user explicitly authorizes the
modification.

This includes:

- notebooks;
- Python scripts;
- Markdown files;
- CSV files;
- configuration files;
- directory structure;
- filenames.

Do not:

- create files;
- edit files;
- rename files;
- move files;
- delete files;
- apply patches;

unless explicitly requested.

When proposing a change:

1. briefly explain why the change is needed;
2. identify the file involved;
3. show the relevant code, text, or diff;
4. wait for authorization if the user has not already requested the edit.

Reading files does not require authorization.

If permission is ambiguous, assume that modification is NOT authorized.

---

## Git policy

Do not modify Git state unless explicitly requested.

Do not automatically:

- stage files;
- commit;
- push;
- merge;
- rebase;
- reset;
- checkout another branch;
- rewrite history.

Read-only Git commands may be used when necessary to answer the current
question.

Never run destructive Git commands without explicit authorization.

---

## Context and token efficiency

Minimize repository reads and context usage.

Start with:

1. the user's current request;
2. information already available in the conversation;
3. the file explicitly mentioned by the user, if one is relevant.

Read additional files only when necessary.

Do not:

- scan the repository proactively;
- read all notebooks to understand the project;
- inspect unrelated notebooks;
- inspect `legacy/` unless explicitly relevant;
- inspect generated HTML or large raw datasets unless necessary;
- repeatedly read files whose relevant content is already available.

If answering the request would require a broad repository review, ask before
performing it.

Prefer targeted searches or targeted file sections over reading entire files.

---

## Notebook workflow

Treat notebooks as independent analytical documents whenever possible.

A notebook should be reproducible from its declared inputs and should not
depend on hidden state created by another notebook.

When debugging a notebook error, first check:

- whether the required cell was executed;
- whether the variable is defined in that notebook;
- whether the notebook accidentally depends on state from another notebook.

Do not propose restructuring multiple notebooks to solve a local problem
unless there is a clear methodological or reproducibility reason.

---

## Coding style

Prefer the tools already used in the project.

Typical analytical tools include:

- pandas;
- NumPy;
- Matplotlib;
- standard Python scientific libraries already present in the environment.

Avoid introducing new dependencies unless they provide a clear benefit.

Prefer:

- small changes;
- readable code;
- explicit transformations;
- reproducible calculations.

Avoid unnecessary abstractions for one-off exploratory notebook analyses.

Preserve existing variable names, terminology, and notebook organization when
possible.

---

## Response style

Be concise and direct.

Do not provide long tutorials unless requested.

For debugging, prioritize:

1. the likely cause;
2. the smallest fix;
3. why the fix works.

For methodological questions, prioritize:

1. the research question;
2. what the proposed method adds;
3. its limitations;
4. how it contributes to the forecasting objective.

When code is requested, provide the code but do not modify repository files
unless modification was explicitly requested.

When advice is requested, provide advice only.

Do not turn a narrow question into a large refactoring or a new analysis
roadmap unless explicitly requested.