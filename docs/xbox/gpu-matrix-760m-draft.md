# TOOL-7 GPU matrix entry: Radeon 760M (review branch, final record)

STATUS: POSTED on upstream #47 as https://github.com/realdody/americas-recomp/issues/47#issuecomment-7079471635
at 2026-11-22 12:57 UTC under glitch1111-agent. Posted body verified identical to the draft below
by read-back at post time (80/80 checks at upstream tip 99cea85c13; one check more than the earlier
3dab00a088 rerun because upstream merged PR #141 in between. Comment reformatted
into a fenced code block at 13:15 UTC, matching the template style used by other
matrix entries; content verified identical to the draft below). This branch is bookkeeping only.

# TOOL-7 GPU matrix draft entry: Radeon 760M (review only, not for an upstream PR)

Fork review artifact only (branch `tool7-760m-results`). The upstream #47 comment text below
is proposed and waits for maintainer-side approval by the fork owner. Run on the fork owner's
own machine; no game files, shader data, keys or SDK files involved (CONTRIBUTING.md rules respected).

## Proposed #47 comment

```
GPU:       AMD Radeon 760M (RDNA3 iGPU, Phoenix/gfx1103, 8 CUs, LPDDR5)
Driver:    RADV Mesa 26.0.8, Vulkan 1.4 API
OS:        Ubuntu 26.04 LTS (server), kernel 7.0.0-38
Commit:    99cea85c13
Build:     cmake -S tools/xbox -B build-xbox -DCMAKE_BUILD_TYPE=Release; XBOX_RDNA2=recomp; device selected via VK_DRIVER_FILES
Passed:    quad_test, permlane_test, quadmask_test, mad_legacy_clamp_test, exec_smoke, exec_tests, exec_bvh_tests, image_tests, desc_chain_tests, pixel_smoke, interpolation_test (max err 4.9e-07), per_vertex_test, texture_test --bilinear (max err 5.2e-04), gs_test (point/line/triangle + adjacency inputs); 80/80 checks, all at the default host subgroup
Failed:    none
Not run:   bvh_test, ffx_test, dispatch_test: Windows only. No game files used or needed on our side.
```

## Per-suite results at 99cea85c13 (log-derived)

| Suite | PASS checks | FAIL | Last PASS line |
|---|---|---|---|
| `desc_chain_tests.py` | 10 | 0 | PASS test_irreducible_loop_non_dominating_srt_descriptor |
| `exec_bvh_tests.py` | 18 | 0 | PASS test_multilevel_rehydrated_bvh64_traversal_lowguest |
| `exec_smoke.py` | 1 | 0 | PASS: lanes 0..31 wrote 17..48; all 1024 output bytes match |
| `exec_tests.py` | 21 | 0 | PASS test_load_format_out_of_range_missing_components |
| `gs_test.py` | 1 | 0 | PASS: geometry shader filled 64 pixels and clipped a half-screen quad with point, line, triangle and |
| `image_tests.py` | 18 | 0 | PASS test_load_bc3_unorm |
| `interpolation_test.py` | 1 | 0 | PASS: 192 blended, 832 clear; max error 4.92e-07 |
| `mad_legacy_clamp_test.py` | 1 | 0 | PASS: v_mad_legacy_f32 clamp matched 32 lanes at 1.0, 0.0 and 0.25 |
| `per_vertex_test.py` | 1 | 0 | PASS: P0, P10 and P20 of two attributes matched at all 1024 pixels in six targets |
| `permlane_test.py` | 1 | 0 | PASS: v_permlane16/x16_b32 selected 32 lanes with op_sel[3] ignored |
| `pixel_smoke.py` | 1 | 0 | PASS: all 91 pixels (1456 RGBA32F bytes) matched |
| `quad_test.py` | 3 | 0 | PASS s_wqm (two quads): 1024 channels at 1.0, rest clear |
| `quadmask_test.py` | 1 | 0 | PASS: s_quadmask_b32 and s_quadmask_b64 matched 32 lanes |
| `texture_test.py` | 1 | 0 | PASS: 192 textured, 832 clear, 12 texels |

Totals: 79 PASS, 0 FAIL across 14 suites.

## Reproduction

```bash
cd ~/src/americas-recomp
git fetch upstream xbox-series --depth 1 && git checkout upstream/xbox-series
git submodule update --init --depth 1
cmake -S tools/xbox -B build-xbox -DCMAKE_BUILD_TYPE=Release && cmake --build build-xbox -j12
ln -sf "$PWD/build-xbox/exec" "$PWD/build-xbox/exec.exe"
export XBOX_RDNA2=recomp VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json
python3 tools/xbox/quad_test.py --exec build-xbox/exec        # (and each suite listed above)
```

Environment: AMD Radeon 760M Graphics (RADV PHOENIX) enumerated at Vulkan 1.4.335 API,
RADV Mesa 26.0.8, glslang 16.2.0, SPIRV-Tools v2026.1, Ubuntu 26.04 LTS server, kernel 7.0.0-38.
Device pinned via `VK_DRIVER_FILES`; results obtained at default subgroup sizes (RADV runs wave64 natively).

Notes:
- `gs_test` exercised the new point/line/triangle + adjacency geometry inputs merged upstream on 2026-11-22 (#103).
- quad_test passes at the default subgroup with the #37 fix at HEAD, consistent with the RDNA3 discrete entries.
- `bvh_test`, `ffx_test`, `dispatch_test` are Windows-only; those package-DLL suites also need game files, which are never committed or shared.
