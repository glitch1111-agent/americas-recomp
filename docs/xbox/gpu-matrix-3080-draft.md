# TOOL-7 GPU matrix draft entry: RTX 3080, CachyOS Linux NVIDIA (review only, posted upstream only after approval)

Same review flow as the 760M entry: fork bookkeeping branch only. Tested on the fork owner's machine,
no game files, shader data, keys or SDK files involved.

## Proposed #47 comment

```
GPU:       NVIDIA GeForce RTX 3080 (Ampere, GA102), compute subgroup 32 only
Driver:    NVIDIA 620.70.06 (open kernel module), device Vulkan 1.4.357, loader 1.4.363
OS:        CachyOS, kernel 7.3.4-1-cachyos
Commit:    99cea85c13
Build:     cmake -S tools/xbox -B build-xbox -DCMAKE_BUILD_TYPE=Release; XBOX_RDNA2=recomp; device selected via VK_DRIVER_FILES
Passed:    quad_test (max err 5.96e-08, NVIDIA few-ulp class vs exact 0 on AMD), permlane_test, quadmask_test, mad_legacy_clamp_test, exec_smoke, exec_tests, exec_bvh_tests, image_tests, desc_chain_tests, pixel_smoke, interpolation_test (max err 1.2e-07), per_vertex_test, texture_test --bilinear (max err 5.2e-04), gs_test (point/line/triangle + adjacency inputs); 80/80 checks, wave64 programs ran as dual 32-wide halves (device subgroup 32)
Failed:    none
Not run:   bvh_test, ffx_test, dispatch_test: Windows only. No game files used or needed on our side.
```

## Per-suite results at 99cea85c13 (log-derived)

| Suite | PASS checks | FAIL | Last PASS line |
|---|---|---|---|
| `desc_chain_tests.py` | 11 | 0 | PASS test_irreducible_loop_non_dominating_srt_descriptor |
| `exec_bvh_tests.py` | 18 | 0 | PASS test_multilevel_rehydrated_bvh64_traversal_lowguest |
| `exec_smoke.py` | 1 | 0 | PASS: lanes 0..31 wrote 17..48; all 1024 output bytes match |
| `exec_tests.py` | 21 | 0 | PASS test_load_format_out_of_range_missing_components |
| `gs_test.py` | 1 | 0 | PASS: geometry shader filled 64 pixels and clipped a half-screen quad with point, line, triangle and adjacency |
| `image_tests.py` | 18 | 0 | PASS test_load_bc3_unorm |
| `interpolation_test.py` | 1 | 0 | PASS: 192 blended, 832 clear; max error 1.21e-07 |
| `mad_legacy_clamp_test.py` | 1 | 0 | PASS: v_mad_legacy_f32 clamp matched 32 lanes at 1.0, 0.0 and 0.25 |
| `per_vertex_test.py` | 1 | 0 | PASS: P0, P10 and P20 of two attributes matched at all 1024 pixels in six targets |
| `permlane_test.py` | 1 | 0 | PASS: v_permlane16/x16_b32 selected 32 lanes with op_sel[3] ignored |
| `pixel_smoke.py` | 1 | 0 | PASS: all 91 pixels (1456 RGBA32F bytes) matched |
| `quad_test.py` | 3 | 0 | PASS s_wqm (two quads): 1024 channels at 1.0, rest clear |
| `quadmask_test.py` | 1 | 0 | PASS: s_quadmask_b32 and s_quadmask_b64 matched 32 lanes |
| `texture_test.py` | 1 | 0 | PASS: 192 textured, 832 clear, 12 texels |

Totals: 80 PASS, 0 FAIL across 14 suites.

## Notes
- Same commit as the 760M entry (99cea85c13), so the two rows cross-reference cleanly.
- First Linux NVIDIA entry in the matrix with the pixel suites executed (the 2060 entry skipped pixel tests for a missing tool). CachyOS ships glslang 1.4.363 and spirv-tools 1.4.363.
- NVIDIA ulp detail: quad swizzle matched with max error 5.96e-08 on NVIDIA (exact 0 on RADV), inside the test threshold; same few-ulp behavior class noted for the compute reciprocal in #47.
- Device subgroup 32: exec ran wave64 programs as two 32-wide halves per the harness design.
