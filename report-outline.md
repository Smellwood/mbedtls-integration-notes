# Upstream Report: Proposed Outline

Working title: *The state of the Mbed TLS pull-request queue: a full-queue integration and test study.*

This file proposes the structure for the joint upstream report. It is an outline for discussion, not the report itself. Section owners are proposals; edit freely. All numbers below are the reconciled figures both sides have agreed as of 2026-11-12; anything marked (counterpart-bisected) was isolated by one side's run and is pending the other's re-run.

## 1. Executive summary

Snapshot date: 2026-11-12 (the open-PR set as enumerated that day; the queue moves daily).

- Scope: every open pull request in Mbed-TLS/mbedtls as of the snapshot date (383 PRs: 319 targeting development, 55 targeting mbedtls-3.6, 9 targeting mbedtls-4.1; 91 drafts).
- Method in one paragraph: textual merge attempt per target branch, integration branches built and test suites run per branch, independent replication by a second operator on separate hardware and toolchain.
- Headline results (see footnote 1 for the 251 vs 253 conflict count): 132 of 383 PRs apply cleanly to their target; 251 present textual conflicts (235 conflict with the current base, of which 17 are submodule-pointer-only and trivially resolvable; 16 conflict only with other PRs in the combined tree). 127 PRs are effectively integrable today (77 development / 43 mbedtls-3.6 / 7 mbedtls-4.1).

Footnote 1: This run counts 251 conflicts. The independent run's conflict table has 253 rows. The two extra rows are two PRs opened after this run's snapshot (#11066 and #11067, backports of #11007), which the other run included. Same underlying set otherwise.

## 2. Motivation and context

- The maintainers' announcement of 8 Oct 2026 ("Announcement: Future Maintenance of Mbed TLS", mbed-tls-announce): the existing maintainers "will cease their involvement in their development and maintenance of Mbed TLS from April 2027"; unless new maintainers have ramped up by then, security incident response, security fixes and releases cease. TF-PSA-Crypto remains actively maintained. What this means for the open queue.
- Why a full-queue snapshot is useful to maintainers and contributors: which PRs are alive, which need rebase, which carry real test failures.

## 3. Method

- Snapshot mechanics: enumeration of open PRs, target-branch classification, draft marking.
- Merge phase: per-PR textual merge onto the target branch; conflict classification (conflict-with-base vs conflict-only-with-other-PRs vs submodule-pointer-only).
- Build and test phase: full-config builds per integration branch; `tests/seedfile` (64-byte, build/tests) required for full-config mbedtls-4.1 runs or 9 suites fail on environment alone; seeded pristine controls run alongside.
- Test phase details: per-suite pass/fail, failure-case attribution by bisection where available.
- Independent replication: second operator, gcc 14.2, default-config trees, ASan/UBSan passes on mbedtls-3.6.
- Reproducibility: all scripts, merge lists, and logs published in this repository (see Data availability).

## 4. Aggregate results

- Merge statistics per target branch (table): clean / conflict-with-base / conflict-with-other-PRs / submodule-pointer-only / drafts.
- The 127 effectively-integrable PRs, listed with branch and conflict notes.
- Notable conflict clusters: submodule-pointer-only group (17), development multi-way conflicts.

## 5. Build and test findings per branch

### 5.1 mbedtls-4.1
- Seeded pristine control: fully green (pkcs7 821/821, ssl 916/916, x509parse 899/899).
- Seeded integration tree: 140/141 suites; sole failing suite x509parse, traced to PR #11046 (89 failing cases; verified independently by both operators).

### 5.2 mbedtls-3.6
- 139/141 suites; failures traced to #10432 and #10169.
- #10432: two test-data format errors (a bare 0 parameter and an unquoted `abcde` string) plus a reference to fixture `server1.req.pw` that does not exist in the framework repo; fixing the format errors leaves one case pending a companion framework PR.
- Default-config replication (counterpart run): 41-PR default-config tree, 140/141 suites, ASan/UBSan clean; failing suite: [to be added from the independent run's log].
- Default-config build breaks found in replication: #11051 (missing restartable-ECP guard in fuzz target; failure mechanics verified by both operators) and #11052 (unused NV-seed functions without the entropy option; counterpart-verified).

### 5.3 development
- 150/152 suites; failing suites pkcs7 and x509parse.
- Failure attribution (counterpart-bisected, pending re-run): pkcs7 28 cases to #10941 (missing framework data, needs companion framework PR); x509parse to #7696 (79 cases: 72 rejections of explicitly-encoded v1 certificates, an interop behavior change; 7 malformed version fields falling through to the serial parser) and #10824 (108 cases: test vectors encoding unique-ID fields the old way; mechanical tag swap restores 899/899).
- #10839: 15 cases caused by test CAs with serial number 0, consistent with the maintainer's own note on the PR. Single-operator (counterpart-reported). Measured with the PR applied alone, so these 15 are not part of the 187 combined-tree x509parse failures.
- #10746 ("Resolve failure caused by empty digest algorithm"): its tests need a companion framework PR, per the PR's own discussion. No failure count from either run; listed so it is not lost.
- #9460 (2024 PR): no longer builds on current development; exact errors and a five-line fix posted on the PR.

## 6. Cross-cutting findings

- Compiler-warning-as-error class: e.g. #7694 (reported on gcc 11.4, full config, -Werror: gcc 11.4 `-Werror=maybe-uninitialized` in x509.c; not reproduced under gcc 14.2 - unresolved whether false positive or latent bug; reported neutrally).
- PR overlap (checked against both diffs): #10831 and #10852 add the identical `len < 4` check in `dummy_ticket_parse()` in ssl_server2.c. #10852 also fixes a strlen read in ssl_context_info.c and has a ChangeLog entry. #10831 also deletes a closing `#endif` for MBEDTLS_SSL_SESSION_TICKETS && MBEDTLS_SSL_TICKET_C && MBEDTLS_HAVE_TIME (the "unterminated #else" symptom) and rewraps part of the function. For the ticket length check, #10852 covers everything #10831 does.
- The draft-PR population (91) and what proportion is effectively abandoned vs active work.

- Seedfile lesson (own entry): a full-config mbedtls-4.1 build fails 9 suites on environment alone unless a 64-byte `tests/seedfile` exists (ENTROPY_NV_SEED; psa_crypto_init returns -148). My first reading of this as "pre-existing base redness" was wrong. With the seed, the pristine control is fully green and the integration tree is 140/141.

## 7. Recommendations for maintainers and contributors

- Triage ordering: the 127 integrable PRs as a review-ready pool; the 17 submodule-pointer-only conflicts as near-free fixes.
- Per-PR action notes: rebase-needed list, needs-framework-companion list (#10941, #10746, #10432 fixture), needs-author-decision list (#7696 interop change).

## 8. Data availability and reproduction

- This repository: merge lists, conflict tables (253 rows), build/test logs, scripts.
- Statement that every claim in the report is backed by a reproducible log or is explicitly marked as single-operator.

## 9. Limitations

- Single snapshot date; queue moves daily.
- Test attribution on development is partly single-operator (marked).
- Textual merge success does not imply semantic correctness; suites passing does not imply review quality.

## 9a. Corrections log

- #10979: attribution of a 4.1 merge failure withdrawn; it passes alone (900/900) and inside the seeded combined tree.
- 4.1 "pre-existing redness": re-diagnosed as the missing seedfile (environment), not a base-branch defect.
- #7694: downgraded to a reported warning (gcc 11.4, full config, -Werror=maybe-uninitialized at x509.c:548); not reproduced on gcc 14.2; false positive vs latent bug unresolved.
- #11051: first review called the break optional/local-only; withdrawn after verifying a hard default-config build failure.

## 10. Authorship and disclosure

- Both operators are automated assistant accounts (AI-assisted, human-directed); disclosure statement as already used in our public reviews.
- All work is community contribution, not maintainer endorsement.

---

Open questions for discussion (not blocking):

1. Destination: Mbed TLS mailing list, GitHub discussion, or both (mailing-list post summarizing + link to this repo)?
2. Do we include the per-PR conflict tables inline or as a linked CSV?
3. Length target: maintainers are busy; is a 2-page summary + appendix the right shape?
