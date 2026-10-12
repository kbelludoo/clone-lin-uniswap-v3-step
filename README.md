# clone-lin-uniswap-v3-step

Experimental i64-scale LIN clone of [Uniswap/v3-core](https://github.com/Uniswap/v3-core) `v1.0.0` SwapMath **computeSwapStep**.

- **LIN clone:** [`src/lin_uniswap_v3_step.lin`](src/lin_uniswap_v3_step.lin)
- **C11 oracle:** [`test/oracles/uniswap_v3_step_c11.c`](test/oracles/uniswap_v3_step_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_UNISWAP_V3_STEP_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** Uniswap V3, **not** a Pool / SwapRouter / NFT position manager, and does **not** claim Solidity uint256 / mainnet / Q64.96 parity (`Q96=2^96` exceeds signed i64 and is rejected). The kernel is the **swap-step cap** (remaining-less-fee vs amount-to-target, exact-in vs exact-out sign, fee remainder vs `mulDivRoundingUp`). It is **not** getNextSqrtPriceFromInput ([clone-lin-uniswap-v3-next-sqrt](https://github.com/kbelludoo/clone-lin-uniswap-v3-next-sqrt)), **not** getNextSqrtPriceFromOutput ([clone-lin-uniswap-v3-output](https://github.com/kbelludoo/clone-lin-uniswap-v3-output)), **not** getAmount0Delta ([clone-lin-uniswap-v3-sqrt](https://github.com/kbelludoo/clone-lin-uniswap-v3-sqrt)), and **not** getSqrtRatioAtTick ([clone-lin-uniswap-v3-tick](https://github.com/kbelludoo/clone-lin-uniswap-v3-tick)).

Upstream was BUSL-1.1 with Change Date 1 April 2023; the Change License is GPL-2.0-or-later (this run is 2026-10-12). This LIN kernel is an original uint64-scale implementation of the published swap-step formulas at Q32=2^32, not a verbatim Solidity dump.

Pinned upstream: commit `e3589b192d0be27e100cd0daaf6c97204fdb1899` (tag `v1.0.0`), `contracts/libraries/SwapMath.sol` sha256 `d6cb9a153be4ea9fb2377ef88641ef7979b5cee6933162f1b732d0289e26e1b6` blob `ee176fbee4ae2fb2ba7da4ddd00391fe6b685f39`.

Canonical goldens (uint64-scale, 128-bit product, Q32=2^32, L=1000000, feePips=3000 unless noted):

- exact-in remaining=1e6 zeroForOne **does not** hit Q/2: next = **2150709713**
- skip-fee of the same remaining **hits** Q/2 = **2147483648** (named missing-fee bug)
- exact-in remaining=1003011 **hits** Q/2; fee = **3010** (`mulDivRoundingUp`) vs not-reached fee = **3000** (remainder)
- exact-out remaining=-5e5 **hits** Q/2; treating +5e5 as exact-in yields **2866177709**
- exact-out remaining=-499999 does not hit: next = **2147487942**
- oneForZero exact-in remaining=1e6 next = **8577049690** (not 2Q)
- naive i64 `(1e13*997000)/1e6` wraps to **-8476744073709**; 128-bit remaining_less_fee = **9970000000000**
- `1e18 * 1e18 / 1` fail-closes to **0**
