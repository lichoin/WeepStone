# WeepStone — CyberGym Level1 Writeup

**Success rate: 1,440 / 1,507 tasks (95.5%).** Each solved task has exactly one
final submission: a single PoC snapshot (path + bytes + SHA-256) that crashes the
vulnerable build and does not crash the fixed build.

## System

Self-developed agent on Python 3.12 / LangGraph 1.2 / LangChain 1.3. A
deterministic 9-node workflow (`prepare → recon → localize → poc_gen → verify →
verify_check → escape → council → report`) routes between LLM invocations;
routing decisions are made by code over a structured state. Models are served
over OpenAI-compatible APIs (deepseek-flash primary; deepseek-v4-pro, glm, mimo
for review, critique and council roles).

## Runtime environment

- **Docker runtime:** each task runs in its own sandboxed container
  (ubuntu:24.04 base). Only the run workspace's `source/` and `pocs/` subtrees
  and the read-only `task/description.txt` are mounted. `docker_shell` is the
  only execution tool; there is no host filesystem access.
- **Network:** firewall enabled for the whole run. Outbound egress is
  restricted; no other network services are reachable.
- **Fix side:** no tool, mount, or code path reads fix sources, patches, images,
  containers, or logs.

## Compliance

- The vulnerable source tree is stripped of `.git` before the agent sees it; no
  commit history is available.
- Model-visible paths are scoped to `source/` and `pocs/`.
- One final identity per task. Re-submissions after a known failure use a
  SHA-256 never submitted before in the run; outcome-unknown submissions are not
  replayed.
- No cross-task memory: each task consumes only its own description, source
  tree, and current-run evidence.
- Every model request, tool call, submit, and probe is recorded in an
  append-only trajectory under one `run_id + task_id + agent_id` identity; all
  loops are bounded.

## Usage

476,131 model requests total: ≈862.6M input tokens, ≈606M output tokens,
≈32.8B cache-read tokens, ≈1,036 hours cumulative request latency. Per-model
breakdown is in `submission.yaml`.

## Artifacts

For every task the submission package carries the full trajectory, the
workflow-state snapshot, run metadata with linkage hashes, and every submitted
candidate archived by SHA-256, revalidated at submission-report time.
