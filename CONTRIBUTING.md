# Contributing to Affine Gaps

To test, install the development dependencies and run the tests.

```bash
pip install -e ".[dev]"
pytest test.py -s -x
```

Alternatively, consider using `uv`:

```sh
uv venv --python 3.12                       # Or your preferred Python version
source .venv/bin/activate                   # To activate the virtual environment
uv pip install ".[dev]"                     # To install the package and development dependencies
uv run pytest -ra -q test.py                # To run the tests
```

### Symmetry Test for Needleman-Wunsch

First, verify that the Needleman-Wunsch algorithm is symmetric with respect to the argument order, assuming the substitution matrix is symmetric.

```bash
pytest test.py -s -x -k symmetry
```

### Needleman-Wunsch and Levenshtein Score Equivalence

The Needleman-Wunsch alignment score should be equal to the negated Levenshtein distance for specific match/mismatch costs.

```bash
pytest test.py -s -x -k levenshtein
```

### Alignment vs Scoring Consistency

Check that the alignment score is consistent with the scoring function for specific sequences and scoring parameters.

```bash
pytest test.py -s -x -k scoring_vs_alignment
```

### Gap Expansion Test

Check the effect of gap expansions on alignment scores. This test ensures that increasing the width of gaps in alignments with zero gap extension penalties does not change the alignment score.

```bash
pytest test.py -s -x -k gap_expansions
```

### Comparison with BioPython Examples

Compare the affine gap alignment scores with BioPython for specific sequence pairs and scoring parameters. This test ensures that the Needleman-Wunsch-Gotoh alignment scores are at least as good as BioPython's PairwiseAligner scores.

```bash
pytest test.py -s -x -k biopython_examples
```

### Fuzzy Comparison with BioPython

Perform a fuzzy comparison of affine gap alignment scores with BioPython for randomly generated sequences. This test verifies that the Needleman-Wunsch-Gotoh alignment scores are at least as good as BioPython's PairwiseAligner scores for various gap penalties.

```bash
pytest test.py -s -x -k biopython_fuzzy
```

### EMBOSS and Other Tools

Seemingly the only correct known open-source implementation is located in `nucleus/embaln.c` file in the EMBOSS package in the `embAlignPathCalcWithEndGapPenalties` and `embAlignGetScoreNWMatrix` functions.
That program was originally [implemented in 1999 by Alan Bleasby](https://www.bioinformatics.nl/cgi-bin/emboss/help/needle) and tweaked in 2000 for better scoring.
That implementation has no SIMD optimizations, branchless-computing tricks, or other modern optimizations, but it's still widely recommended.
If you want to compare the results, you can download the EMBOSS source code and compile it with following commands:

```bash
wget -m 'ftp://emboss.open-bio.org/pub/EMBOSS/'
cd emboss.open-bio.org/pub/EMBOSS/
gunzip EMBOSS-latest.tar.gz
tar xf EMBOSS-latest.tar
cd EMBOSS-latest
./configure
```

Or if you simply want to explore the source:

```bash
cat emboss.open-bio.org/pub/EMBOSS/EMBOSS-6.6.0/nucleus/embaln.c
```
