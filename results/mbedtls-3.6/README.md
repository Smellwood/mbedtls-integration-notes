# integration/mbedtls-3.6 - tested tree and results (Elwood)

## The tested tree

- Published branch `integration/mbedtls-3.6` on https://github.com/doctor-elwood-phd/mbedtls, head `1a98fa0a43b213a587ce35aaaf8c26f5d8853956` - **predates two demotions and does not build.**
- The tree I actually tested: local commit `efdb9e2a81fc587a28b62b5b1c279c8beec28294` = published head + 4 commits (demotion of #10184, demotion of #11053, plus a net-zero revert/revert-of-revert pair around #10139).
- A repush is blocked on a credential-isolation requirement on my side (no new token may be minted into an environment that has executed merged-PR binaries). Until that is resolved, `tested-tree.patch` in this directory is the reproducible route: one combined diff from the published head to the tested tree.

## Reproduce

    git clone https://github.com/doctor-elwood-phd/mbedtls.git
    cd mbedtls
    git checkout 1a98fa0a43b213a587ce35aaaf8c26f5d8853956
    git apply tested-tree.patch     # tree now identical in content to tested head efdb9e2a81
    scripts/config.py full          # build-time config change, deliberately uncommitted
    cmake -B build && cmake --build build -j
    cd build && ctest               # serial; -j adds PSA flakes that pass standalone

Toolchain: gcc 11.4, cmake 3.22.1, Ubuntu 22.04.

## Demotions (vs published head)

- **Mbed TLS #10184** "Move NIST_KW to PSA API": clean git merge, but semantically conflicts with #10139's constified `mbedtls_cipher_supported` - KW entries remain in the supported list after removal from `mbedtls_cipher_definitions` (excess array initializer). The kind of conflict git cannot see.
- **Mbed TLS #11053** (OSS-Fuzz fuzz_pk_ops, 3.6 variant): its new fuzz target calls the development-branch `mbedtls_rsa_set_padding` signature (md_info pointer); 3.6 still has the md_type enum signature. Backport written against the wrong API surface.
- The #10139 revert pair nets to zero; #10139 stays merged.

## Results (tested tree, full config)

- Build: green.
- Test suites: 139/141 pass.
- Real failures:
  - **Mbed TLS #10432**: adds challenge-password CSR test data the 3.6 test parser rejects (FATAL PARSE ERROR).
  - **Mbed TLS #10169** "adjust_legacy_from_psa: Add XTS algorithm support": breaks the AES 512-bit XTS storage-format save case (1460/1461 cases pass in that suite).

## Scope

55 open PRs targeting mbedtls-3.6 (53 ready, 2 drafts): 45 textually merged, 10 conflicts, 2 post-merge demotions, 43 effective. Serial ctest only.
