# Hackathon Protein Fitness Prediction

This project predicts protein fitness scores for single-point mutants using ESM2 embeddings and a 3-round active-learning loop.

The main workflow is:

1. Build ESM2 embeddings for the wild-type sequence and mutant sequences.
2. Train predictive models for mutant fitness.
3. Select 100 mutants per query round using different active-learning strategies.
4. Add the newly measured mutants back into the training set and repeat.

In total, the project performs **three query rounds**, with **100 mutants selected in each round**.

## Project files

- `hackthon_final.ipynb` - final modeling notebook with combined feature engineering and prediction export.
- `train.csv` - labeled training mutants with `DMS_score`.
- `test.csv` - unlabeled test mutants.
- `sequence.fasta` - wild-type protein sequence.
- `query.txt` - output file containing the selected mutants, one per line.
- `query1_result`, `query2_result`, `query3_result` - round-by-round query result files used to update training data.

## Model idea

The active-learning workflow uses:

- **ESM2 embeddings** for each sequence
- **Bootstrap ensemble of Ridge regressors**
- **Predictive uncertainty** measured by the standard deviation across ensemble outputs

This strategy is used for exploration-focused querying. Other rounds can use different strategies, but each round still returns 100 selected mutants.

## Feature engineering

In the final notebook, the ESM2 representation is combined with **six engineered features** to improve prediction quality:

1. **Evolutionary score** / masked-marginal log-likelihood ratio
2. **BLOSUM62 substitution score**
3. **Molecular weight delta** between mutant and wild type
4. **Hydrophobicity delta** between mutant and wild type
5. **Normalized mutation position** in the protein sequence
6. **Wild-type amino-acid index** at the mutation site

These features are concatenated with the ESM2 embedding before the final regression model is trained.

## Requirements

Typical Python packages used in this project:

- `numpy`
- `pandas`
- `torch`
- `fair-esm`
- `scikit-learn`

If you are using the notebook or script in a GPU environment, make sure PyTorch is installed with CUDA support.

## How to run the active-learning script

From the project directory:

```bash
python active_learning_query_uncertainty.py \
	--train_csv train.csv \
	--test_csv test.csv \
	--sequence_fasta sequence.fasta \
	--top_k 100
```

### Optional speed-up

To reuse saved embeddings on reruns:

```bash
python active_learning_query_uncertainty.py --use_cached_embeddings
```

## Script outputs

The script writes:

- `uncertainty_ranked_predictions.csv` - all test mutants ranked by uncertainty
- `query_candidates_top100.csv` - top 100 uncertain mutants with mean prediction and uncertainty
- `query.txt` - selected query mutants for submission or wet-lab follow-up
- `train_embeddings.npy` / `test_embeddings.npy` - cached embeddings if enabled

## Notebook workflow

If you prefer notebooks, the active-learning flow is also available in `with_active_learning.ipynb`.

The notebook includes:

- ESM2 embedding generation
- Feature engineering with six additional mutation descriptors
- Ridge vs MLP model comparison
- Final prediction export
- Query selection and query distribution analysis

## Active-learning rounds

Each query round follows the same general update loop:

1. Predict on the unlabeled candidate mutants.
2. Rank candidates using the round-specific acquisition strategy.
3. Select the top 100 mutants for that round.
4. Save the round output and append the new labeled data back into `df_train`.

The three rounds in this project are:

- **Round 1**: uncertainty-based sampling for exploration.
- **Round 2**: a different acquisition strategy for the next batch of 100 mutants.
- **Round 3**: a final acquisition strategy for the last batch of 100 mutants.

## Notes

- The repository contains multiple iterations of the workflow.
- The uncertainty-based query strategy is intended for the first active-learning round.
- If you add new query results to `df_train`, rerun the embedding and query-selection steps.
- Replace the round-specific strategy description above if you want the README to document the exact method used in Round 2 and Round 3.
