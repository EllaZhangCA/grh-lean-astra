# GRH in Lean 4, Astra edition

We used **GPT-6.2 Astra** to shorten parts of the compact Lean proof in [lsernau/grh-lean at `72fead1`](https://github.com/lsernau/grh-lean/tree/72fead102f663d5c7a407964a13dc0cf9d977207/grh-compact). The underlying argument is due to Levent Alpöge with Claude. The upstream team at Anthropic worked with Claude to produce the compact Lean edition.

The modified [compact edition](grh-compact/) proves the five statements in [Main.lean](grh-compact/src/Main.lean), including Mathlib's Riemann hypothesis and the critical-line result for Dirichlet L-functions. The [readable edition](grh/) and its guide are also available.

## Changes to the compact proof

* The proof of the main inequality (⋆) bounds the contribution of the missing rows directly. If the row space has dimension `d` and the family contains `n` rows, the loss is `(d - n) * (-h)`, where `h` bounds the log squared covolume per row. Adding this estimate to the index bound and the lower bound for integral forms cancels the common height term. [Main inequality](grh-compact/src/New/Main.lean)
* The Euler product estimate and the bound for the odd part of a Tate–Shafarevich group both count prime ideals lying over a rational prime. Each now uses Mathlib's `Ideal.card_primesOverFinset_le_finrank` in place of a separate derivation from ramification indices and inertia degrees. [Euler product](grh-compact/src/Lib/ClassField/HeckeL/EulerProduct.lean) · [Odd Sha bound](grh-compact/src/Lib/Cyclotomic/OddSha/OddShaBound.lean)
* The polynomial partial-derivative commutation lemma now applies `MvPolynomial.pderiv_comm` directly. This removes the coefficient calculation from the local proof. [Partial derivatives](grh-compact/src/Lib/General/LinearAlgebra/PartialDerivComm.lean)

## Size

Four proof bodies changed across the same 794 Lean files under `grh-compact/src`.

| Measure | Upstream compact | Astra edition | Reduction |
| --- | ---: | ---: | ---: |
| Nonblank lines of Lean code, excluding comments | 76,149 | 76,138 | 11 |
| All source lines, including comments and blank lines | 112,827 | 112,820 | 7 |
| Source bytes in Git, including modification notices | 7,420,187 | 7,419,439 | 748 |

Run this command from the repository root to count source bytes at `HEAD`.

```sh
git ls-tree -rl HEAD -- grh-compact/src | awk '$5 ~ /\.lean$/ {bytes += $4} END {print bytes}'
```

Replace `HEAD` with `72fead1` for the upstream count.

## Reproduce the Lean check

The [toolchain](grh-compact/lean-toolchain) pins Lean 4.35.0. The [manifest](grh-compact/lake-manifest.json) pins Mathlib to `56a005855ea17e0b0958f73ec801ae140b0a1679`.

```sh
cd grh-compact
lake exe cache get
lake build
lake env lean scripts/PrintAxioms.lean
```

The full compact project built on x86-64 Linux. Each of the five declarations in [the axiom check](grh-compact/scripts/PrintAxioms.lean) reported exactly `propext`, `Classical.choice`, and `Quot.sound`. The theorem statements and dependency pins match the upstream baseline. The pinned Mathlib marks the prime-counting lemma used in two modules as deprecated, so those references emit warnings.

We also replayed the four edited modules and the public theorem module with Lean's kernel. The following command repeats that check after the build.

```sh
lake env leanchecker -v Lib.General.LinearAlgebra.PartialDerivComm \
  Lib.ClassField.HeckeL.EulerProduct Lib.Cyclotomic.OddSha.OddShaBound New.Main Main
```

## License and attribution

Distributed under [Apache License 2.0](LICENSE). The compact edition includes the upstream [NOTICE](grh-compact/NOTICE) and [Mathlib attributions](grh-compact/ATTRIBUTION.md). This README and the marked proof files describe changes to the upstream work.
