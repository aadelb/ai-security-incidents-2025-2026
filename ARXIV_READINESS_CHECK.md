# arXiv Readiness Check — Three Fixes (issue #77 / T070)

**Date:** 2026-10-21 · **Branch:** `feat/77-arxiv-three-fixes` (HEAD `21e8758`, exactly 1 commit ahead of `origin/main` `1d51529`, which is the merge-base) · **Checker:** ZCode agent
**Toolchain:** pdfTeX 3.141592653-2.6-1.40.24 (TeX Live 2022/Debian), BibTeX 0.99d, `IEEEtran.bst` v1.14. `biber` not installed — not needed, the paper uses `\bibliographystyle{IEEEtran}` + `\bibliography{bibliography}` (classic BibTeX).

## 1. The three fixes in ARXIV_FIXES_DETAILED.md vs `git diff main...HEAD -- paper.tex '*.bib'`

### CRITICAL #1 — Remove duplicate "Regulatory Interpretation": **DONE** (no hunk in the branch diff — already fixed on `main`)
- `git diff main...HEAD` contains **no** hunk for this: the duplicate was removed *before* the branch point, in `main` commits `ddf234b` / `e98f5dd`.
- Verified on both sides: `grep -n "Regulatory Interpretation"` returns exactly **one** occurrence, at `paper.tex:471` on HEAD and on `origin/main` — the kept paragraph matches the "after deletion" text in the fixes doc.
- Branch commit `21e8758`'s message mentions it, but its tree change for this item is a no-op relative to `main`.

### CRITICAL #2 — Add Data Availability subsection: **DONE** (hunk `@@ -516,9 +516,19 @@ paper.tex`, new text at `paper.tex:519–531`)
```diff
-\section*{DATA AVAILABILITY}
+\subsection{Data Availability}
 
-The complete dataset (AISecIncidents-2025-26, 55 incidents, 12 verified) is publicly available at Zenodo \url{https://doi.org/[ZENODO_DOI]} under CC-BY-4.0 license, ... (\path{DATASET_CARD.md}).
+All data artifacts from this research are publicly available without access restrictions:
+
+\begin{itemize}
+    \item \textbf{AISecIncidents-2025-26 Dataset:} 55 incidents with ... available at \url{https://huggingface.co/datasets/Ahmed-AdelB/ai-security-incidents-2025-2026}
+    \item \textbf{aisec-euaiact-v1.0 Benchmark:} 12 verified incidents with ground-truth multi-label EU AI Act Article classifications (Articles 9, 13, 15) ...
+    \item \textbf{Evaluation Scripts and Metrics:} Reproducible Python scripts for computing F1-micro, F1-macro, Exact Match, Hamming Loss, and Mean Jaccard Index ...
+\end{itemize}
+
+The dataset is released under Creative Commons Attribution 4.0 (CC-BY-4.0) ... no access restrictions or embargo periods. We welcome community contributions, ...
```
Content matches the fixes doc verbatim; placed directly before `\bibliographystyle{IEEEtran}` as specified. Side effect: removes the `[ZENODO_DOI]` placeholder that `main` had in its old `\section*{DATA AVAILABILITY}` (line 521 on main).

### CRITICAL #3 — Add CVSS citation: **DONE** (bib hunk `@@ -51,3 +51,10 @@`, entry at `bibliography.bib:55–60`; cites at `paper.tex:98` and `paper.tex:186`)
```diff
+@misc{cvss2024,
+  title={{Common Vulnerability Scoring System Version 3.1 Specification}},
+  author={{Forum of Incident Response and Security Teams (FIRST)}},
+  howpublished={\url{https://www.first.org/cvss/v3.1/specification-document}},
+  year={2019}
+}
```
```diff
-    \item ... CVSS-inspired criteria adapted for AI-specific risks~\cite{first2019cvss}.
+    \item ... CVSS-inspired criteria~\cite{cvss2024} adapted for AI-specific risks.        % paper.tex:98
-Severity assessment follows CVSS-inspired criteria adapted for AI-specific risks~\cite{first2019cvss}. ...
+Severity assessment follows CVSS-inspired criteria~\cite{cvss2024} adapted for AI-specific risks. ...   % paper.tex:186
```
- Citation placement matches the fixes doc exactly ("criteria~\cite{cvss2024} adapted") at both required locations.
- **Deviation from the doc:** `year={2019}` instead of the doc's `year={2024}` — intentional per the commit message ("CVSS v3.1 (FIRST, 2019)"); the FIRST CVSS v3.1 specification is indeed a 2019 document, so 2019 is factually correct.
- Note: `main` already cited a CVSS entry at those two lines via a different key (`first2019cvss`, `@techreport` at `bibliography.bib:1`); the branch swaps the key to `cvss2024`. `first2019cvss` remains in the `.bib` but is now uncited (harmless — BibTeX only emits cited entries).

## 2. Other changes on the branch beyond the three fixes

Exactly one, bookkeeping only:
- `ARXIV_SUBMISSION_REVIEW.txt`: three READY-FOR-SUBMISSION checklist lines ticked `[ ]` → `[x]` (CRITICAL-1/2/3).

Full diffstat vs main: `bibliography.bib +7` · `paper.tex +18/−4` · `ARXIV_SUBMISSION_REVIEW.txt 3 lines` — nothing else; the diff is minimal.

## 3. Compilation — branch vs main (`git worktree add /tmp/main-wt main`)

Commands run verbatim in both working trees: `pdflatex -interaction=nonstopmode paper.tex` → `bibtex paper` → `pdflatex` ×2. All 8 runs exit code 0; BibTeX clean on both sides ("Done.", no warnings/errors).

| Metric (final pdflatex run) | branch `feat/77-arxiv-three-fixes` | main `1d51529` |
|---|---|---|
| `LaTeX Warning` lines | **0** | **0** |
| Undefined reference/citation warnings (verbatim) | **none** | **none** |
| Overfull boxes | 0 | 0 |
| Pages / bytes | 9 pages / 199,457 B | 9 pages / 201,854 B |
| `paper.pdf` produced | yes (`ls -la` → `-rw-r--r-- ... 199457 ... paper.pdf`) | yes (`-rw-r--r-- ... 201854 ... /tmp/main-wt/paper.pdf`) |

Last lines of the final pdflatex run (branch; the 13 preceding lines are the wrapped Type-1 font-embedding list, identical on both sides):
```
Output written on paper.pdf (9 pages, 199457 bytes).
Transcript written on paper.log.
```
Main's final line differs only in size: `Output written on paper.pdf (9 pages, 201854 bytes).` **No compile regression on the branch — warning counts are equal (0 vs 0).**

## 4. Data Availability dataset id vs DATASET_CARD.md

- Paper (`paper.tex:523`) names: `https://huggingface.co/datasets/Ahmed-AdelB/ai-security-incidents-2025-2026`.
- **`DATASET_CARD.md` contains no dataset id or HF URL at all** — only `pretty_name: AI System Security Incidents 2025-2026` and a citation block with `publisher = {HuggingFace}`. So there is nothing in the card that *conflicts*, but the literal check "names the id exactly as DATASET_CARD.md does" cannot be satisfied against the card itself.
- The paper's id matches the repo-canonical id everywhere it does appear: `push_to_huggingface.py:16`, `CITATION.cff:21`, `README.md:8/17/160`.
- **Recommendation:** add the canonical id/URL to `DATASET_CARD.md` before submission.

## 5. CVSS v3.1 bib key cited in paper.tex

Yes. Key `cvss2024` is cited via `\cite{cvss2024}` at `paper.tex:98` and `paper.tex:186`; entry at `bibliography.bib:55–60`; it resolves into the compiled `paper.bbl` (line 53: `{Forum of Incident Response and Security Teams (FIRST)}, ``{Common ...`), and the final run reports zero undefined-citation warnings.

## 6. Still open (outside the three fixes, unchanged from main)

- `[ZENODO_DOI]` placeholder at `paper.tex:104` (MODERATE-2 — Option A real DOI still pending).
- MODERATE-1 ("production AI agent" citation/footnote) and the optional MINOR fixes remain open per `ARXIV_SUBMISSION_REVIEW.txt`.

**Verdict:** all three critical fixes are in place; the paper compiles cleanly on the branch with zero warnings and no regression vs main. Ready for arXiv from a compile/citation standpoint once the Zenodo DOI placeholder is resolved.
