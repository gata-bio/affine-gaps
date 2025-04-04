![Affine Gaps Thumbnail](https://github.com/ashvardanian/ashvardanian/blob/master/repositories/affine-gaps.jpg?raw=true)

__Affine Gaps__ is a __less-wrong__ single-file Numba-accelerated Python implementation of Osamu Gotoh affine gap penalty extensions 1982 [paper](https://doc.aporc.org/attach/Course001Papers/gotoh1982.pdf) for the Needleman-Wunsch and Smith-Waterman algorithms often used for global and local sequence alignment in Bioinformatics.
Thanks to the Numba JIT compiler, it's also competitive in terms of performance.
But if you want to go even faster and need more hardware-accelerated string operations, check out [StringZilla](https://github.com/ashvardanian/stringzilla) 🦖

## Less Wrong

As reported in the "Are all global alignment algorithms and implementations correct?" [paper](https://www.biorxiv.org/content/10.1101/031500v1.full.pdf) by Tomas Flouri, Kassian Kobert, Torbjørn Rognes, and Alexandros Stamatakis:

> In 1982 Gotoh presented an improved algorithm with lower time complexity. 
> Gotoh’s algorithm is frequently cited...
> While implementing the algorithm, we discovered two mathematical mistakes in Gotoh’s paper that induce sub-optimal sequence alignments.
> First, there are minor indexing mistakes in the dynamic programming algorithm which become apparent immediately when implementing the procedure.
> Hence, we report on these for the sake of completeness.
> Second, there is a more profound problem with the dynamic programming matrix initialization.
> This initialization issue can easily be missed and find its way into actual implementations.
> This error is also present in standard text books.
> Namely, the widely used books by Gusfield and Waterman.
> To obtain an initial estimate of the extent to which this error has been propagated, we scrutinized freely available undergraduate lecture slides.
> We found that 8 out of 31 lecture slides contained the mistake, while 16 out of 31 simply omit parts of the initialization, thus giving an incomplete description of the algorithm.
> Finally, by inspecting ten source codes and running respective tests, we found that five implementations were incorrect.

During my exploration of exiting implementations, I've noticed several bugs:

- several libraries initialize the header row/columns of penalty matrices with ±∞, causing overflows on the first iteration.
- initialize matrices to zeros, ignoring the first gap opening cost.
- combining opening and expansion costs where only the opening cost should be applied.
- even the most correct `needle` from EMBOSS uses `float` representation, which would obviously be numerically unstable on very long sequences.

## Installation

Even without installing Python, you can just use `uv` to get the latest version of the library:

```bash
$ uv tool install git+https://github.com/ashvardanian/affine-gaps.git
$ affine-gaps --help
```

Alternatively, you can install it using `pip`:

```bash
$ pip install git+https://github.com/ashvardanian/affine-gaps.git
$ affine-gaps --help
```

## Using the Library

To obtain the alignment of two sequences, use the `needleman_wunsch_gotoh_alignment` function.

```python
from affine_gaps import needleman_wunsch_gotoh_alignment

insulin = "GIVEQCCTSICSLYQLENYCN"
glucagon = "HSQGTFTSDYSKYLDSRAEQDFV"
aligned_insulin, aligned_glucagon, aligned_score = needleman_wunsch_gotoh_alignment(insulin, glucagon)

print("Alignment 1:", aligned_insulin)  # GI-V---EQCC-TSICSLY---QL-ENYCN-
print("Alignment 2:", aligned_glucagon) # --D-FVHSQGTFTSDYSKYLDSRAEQDF--V
print("Score:", aligned_score)          # 41
```

If you only need the alignment score, you can use the `needleman_wunsch_gotoh_score` function, which uses less memory and works faster.

```python
from affine_gaps import needleman_wunsch_gotoh_score

score = needleman_wunsch_gotoh_score(insulin, glucagon)

print("Score:", score)
```

By default, a BLOSUM62 substitution matrix is used.
You can specify a different substitution matrix by passing it as an argument.

```python
from numpy import np

alphabet = "ARNDCQEGHILKMFPSTWYVBZX"
substitutions = np.zeros((len(alphabet), len(alphabet)), dtype=np.int8)
substitutions.fill(-1)
np.fill_diagonal(substitutions, 1)

aligned_insulin, aligned_glucagon, aligned_score = needleman_wunsch_gotoh_alignment(
    insulin, glucagon,
    substitution_alphabet=alphabet,
    substitution_matrix=substitutions,
    gap_opening=-2,
    gap_extension=-1,
)
```

That is similar to the following usage example of BioPython:

```python
from Bio import Align
from Bio.Align import substitution_matrices

aligner = Align.PairwiseAligner(mode="global")
aligner.substitution_matrix = substitution_matrices.load("BLOSUM62")
aligner.open_gap_score = open_gap_score
aligner.extend_gap_score = extend_gap_score
```

## Using the Command Line Interface

To compute the optimal global alignment of insulin and glucagon sequences with (5x-scaled) BLOSUM62 substitution matrix through CLI:

```bash
$ affine-gaps GIVEQCCTSICSLYQLENYCN HSQGTFTSDYSKYLDSRAEQDFV
>
> Sequence 1: GIVEQCCTSICSLYQLENYCN
> Sequence 2: HSQGTFTSDYSKYLDSRAEQDFV
>
> Alignment 1: GIVEQCCTSICSLY---QL-ENYCN-
> Alignment 2: GTF----TSDYSKYLDSRAEQDF--V
> Score:       22
```

To compute the local alignment of insulin and glucagon sequences through CLI:

```bash

$ affine-gaps GIVEQCCTSICSLYQLENYCN HSQGTFTSDYSKYLDSRAEQDFV --local
> 
> Sequence 1: GIVEQCCTSICSLYQLENYCN
> Sequence 2: HSQGTFTSDYSKYLDSRAEQDFV
> 
> Alignment 1: TSICSLYQLEN
> Alignment 2: TSDYSKY-LDS
> Score:       80
```
