# ROCm and llama.cpp Stack

## Purpose

This document records the validated container-first ROCm 7.2.4 and llama.cpp architecture used with the AMD Radeon AI PRO R9700 on `basement-node`.

It covers the ROCm development validation, HIP execution, PyTorch validation, pinned llama.cpp build, permanent service stack, model/cache placement, router-mode UI, security boundary, and observed model benchmarks.

This record is based on the project-history handoff. No new live-system inspection was performed for this documentation catch-up.

## Architecture

Status: **CONFIRMED**

The host deliberately remains free of ROCm development packages and AI-framework dependencies.

The architecture is:

```text
host amdgpu
→ /dev/kfd + /dev/dri
→ Docker
→ ROCm userspace
→ AI framework/application
→ R9700
```

The host owns the kernel driver and device interfaces. ROCm userspace, compiler tools, AI frameworks, and application dependencies remain inside containers.

This is not a host-installed ROCm deployment.

## GPU Device Access

Status: **CONFIRMED**

The containerized workloads use:

```text
/dev/kfd
/dev/dri
```

Required host groups:

```text
video
render
```

Observed Docker group additions on this host:

```text
44
992
```

The IDs map to `video` and `render` on this host. They are host-specific rather than portable defaults.

## ROCm Development Validation

Status: **CONFIRMED**

Official validation image:

```text
rocm/dev-ubuntu-24.04:7.2.4
```

Validated image digest:

```text
sha256:bdc8e61026cbb844ede93d44d2c50055f51ebb2041906b60182bf3bee3139054
```

Validated through `rocminfo`:

```text
GPU architecture: gfx1201
Compute units: 64
Visible GPU memory: approximately 31.9 GiB
```

Observed toolchain:

```text
HIP: 7.2.53211
Clang: 22
```

## HIP Compilation and Runtime

Status: **CONFIRMED**

The ROCm development container successfully provided `hipcc`.

Validation included:

- HIP source compilation
- A compiled vector-add test
- Runtime execution on the R9700
- Successful GPU-produced results

This verifies both compiler availability and real GPU runtime execution. It is stronger than device enumeration alone.

## PyTorch ROCm Validation

Status: **CONFIRMED**

Official image:

```text
rocm/pytorch:rocm7.2.4_ubuntu24.04_py3.12_pytorch_release_2.9.1
```

Observed framework state:

```text
PyTorch 2.9.1+ROCm 7.2.4
GPU availability: true
```

A real `4096x4096` tensor workload completed successfully.

Observed result:

```text
PYTORCH ROCM TEST: PASS
```

## Pinned llama.cpp Build

Status: **CONFIRMED**

Source tree:

```text
~/ai-tests/llama-rocm/llama.cpp
```

Pinned commit:

```text
60addddf3c567c43ec3caf70fc953fba3572d96f
```

Build identifier:

```text
b10498-60addddf3
```

CMake configuration included:

```text
-DGGML_HIP=ON
-DGPU_TARGETS=gfx1201
-DLLAMA_OPENSSL=ON
-DCMAKE_BUILD_TYPE=Release
```

ROCm libraries included:

```text
hipBLAS
hipBLASLt
rocBLAS
```

Pinning the source revision and recording the build identifier makes the validated application layer reproducible and easier to compare with future builds.

## Runtime GPU Validation

Status: **CONFIRMED**

llama.cpp discovered the GPU as:

```text
ROCm0: AMD Radeon Graphics
```

Observed runtime memory:

```text
Total VRAM: approximately 32624 MiB
Free VRAM: approximately 32088 MiB
```

Large GGUF workloads were successfully fully offloaded to the GPU.

The generic runtime device label does not replace the independent `gfx1201`, compute-unit, driver, and memory validation.

## Permanent Stack

Status: **CONFIRMED**

Stack location:

```text
/opt/stacks/llama-rocm
```

Pinned image:

```text
lich-llama-rocm:7.2.4-pinned
```

Persistent storage:

```text
Models: /mnt/media-secondary/ai/models
Cache:  /mnt/media-secondary/ai/cache
```

GPU interfaces:

```text
/dev/kfd
/dev/dri
```

Docker group additions:

```text
44
992
```

Host endpoint:

```text
127.0.0.1:8088
```

## Router Mode and Web UI

Status: **CONFIRMED**

`llama-server` runs against a models directory with:

```text
models-max=1
```

The native llama.cpp web UI provides current model selection.

Older `lich-model` CLI behavior predates router mode and is not the current model-switching authority.

Any work to align or replace that older CLI behavior is:

Status: **PLANNED / BACKLOG**

It must not be described as a currently working router-mode control path.

## Storage Policy

Status: **CONFIRMED**

Storage roles are intentionally separated:

- `/mnt/media-primary` is reserved for VR video content.
- `/mnt/media-secondary` is the default for AI models, caches, application data, documentation-related storage, downloads, and new persistent workloads.

The local-AI stack uses:

```text
/mnt/media-secondary/ai/models
/mnt/media-secondary/ai/cache
```

No inventory of private media or download contents is required for this architecture record.

## Benchmark Results

Status: **OBSERVED**

Observed llama.cpp benchmark results:

| Model | Prompt processing (`pp512`) | Token generation (`tg128`) |
|---|---:|---:|
| Gemma3 4B Q4_K_M | ~6750.50 t/s | ~115.96 t/s |
| Qwen3 14B Q4_K_M | ~2264.75 t/s | ~51.03 t/s |
| Qwen3-30B-A3B Q4_K_M | ~3328.57 t/s | ~92.55 t/s |

Based on the tested models and observed results, Qwen3-30B-A3B is the leading daily-driver candidate.

That is an operational preference from this test set, not a permanent or immutable model selection.

## Exposure and Security Boundary

Status: **CONFIRMED**

The llama.cpp endpoint is bound to:

```text
127.0.0.1:8088
```

It is localhost-only and is not publicly exposed.

This boundary is separate from the repository's documented Cloudflare-published services. No public AI endpoint, tunnel route, or direct inbound exposure is documented.

## Result

The R9700 successfully supports the container-first ROCm 7.2.4 architecture.

Validation covered `rocminfo`, `gfx1201`, 64 compute units, visible GPU memory, HIP compilation and runtime execution, a real PyTorch tensor workload, a pinned HIP-enabled llama.cpp build, full GPU offload of large GGUF workloads, and a permanent localhost-only router-mode deployment.

The host remains responsible for `amdgpu` and GPU device access while ROCm development tools, frameworks, and applications remain containerized.
