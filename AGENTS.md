<!-- BEGIN jpmicrosoft fork notice -->
# Fork notice — jpmicrosoft/llama.cpp

This is **jpmicrosoft/llama.cpp**, a fork of [`ggml-org/llama.cpp`](https://github.com/ggml-org/llama.cpp) maintained for the **caborojo** local-AI rig (3× Intel Arc Pro B60, 128 GB DDR4, oneAPI 2025.3).

## Branches

- `master` — tracks upstream `master`. **Do not commit local changes here.**
- `getbookn-gptoss-arrlen` — current local patch branch.

## Local divergence from upstream

Exactly **one source file** is modified from upstream:

### `src/llama-model-loader.cpp`

```diff
-            if (n != arr_info.length) {
+            if (arr_info.length > n) {
                 throw std::runtime_error(format("key %s has wrong array length; expected %u, got %u", ...));
             }
```

**Why:** `gpt-oss-120b` MXFP4 GGUFs from openai-community ship metadata arrays whose declared length is *shorter* than expected by current llama.cpp tokenizer code. Without this relaxation the loader aborts and the model will not start. The check now only rejects arrays that are *longer* than expected (still safe), allowing shorter arrays to load. Workaround, not a permanent fix.

## Build configuration (caborojo)

```bash
source /opt/intel/oneapi/setvars.sh
export PATH=/opt/intel/oneapi/compiler/2025.3/bin/compiler:$PATH
cmake -B build-sycl \
  -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx \
  -DGGML_SYCL=ON -DGGML_SYCL_TARGET=INTEL \
  -DGGML_SYCL_DEVICE_ARCH=bmg-g21 \
  -DGGML_NATIVE=ON -DCMAKE_BUILD_TYPE=Release \
  -DLLAMA_CURL=OFF
cmake --build build-sycl -j 16 --target llama-server
```

`-DGGML_SYCL_DEVICE_ARCH=bmg-g21` requires `apt install intel-ocloc` for Battlemage AOT compilation (≈+48% prefill perf vs JIT).

## Known upstream issues affecting this rig

- **[ggml-org/llama.cpp#23301](https://github.com/ggml-org/llama.cpp/issues/23301)** — SYCL `-sm row` (tensor-parallel) segfaults in `ggml_backend_sycl_split_buffer_type` on multi Arc Pro B60. Filed from this hardware. Use `-sm layer` (pipeline-parallel) until fixed.

## Workflow for AI agents

1. Stay on `getbookn-gptoss-arrlen`. Don't commit to `master`.
2. To incorporate upstream: `git checkout master && git pull origin master && git checkout getbookn-gptoss-arrlen && git rebase master`.
3. If the `arr_info.length > n` change conflicts (upstream changed that block), inspect — upstream may have fixed it properly; if so, drop our patch.
4. Run a SYCL build before pushing any commit that touches `ggml/src/ggml-sycl/` or `src/llama-model-loader.cpp`.

<!-- END jpmicrosoft fork notice -->

---

# Instructions for llama.cpp

> [!IMPORTANT]
> This project does **not** accept pull requests that are fully or predominantly AI-generated. AI tools may be utilized solely in an assistive capacity.
>
> Read more: [CONTRIBUTING.md](CONTRIBUTING.md)

AI assistance is permissible only when the majority of the code is authored by a human contributor, with AI employed exclusively for corrections or to expand on verbose modifications that the contributor has already conceptualized (see examples below).

---

## Guidelines for Contributors Using AI

llama.cpp is built by humans, for humans. Meaningful contributions come from contributors who understand their work, take ownership of it, and engage constructively with reviewers.

Maintainers receive numerous pull requests weekly, many of which are AI-generated submissions where the author cannot adequately explain the code, debug issues, or participate in substantive design discussions. Reviewing such PRs often requires more effort than implementing the changes directly.

**A pull request represents a long-term commitment.** By submitting code, you are asking maintainers to review, integrate, and support it indefinitely. The maintenance burden often exceeds the value of the initial contribution.

Most maintainers already have access to AI tools. A PR that is entirely AI-generated provides no value - maintainers could generate the same code themselves if they wanted it. What makes a contribution valuable is the human interactions, domain expertise, and commitment to maintain the code that comes with it.

This policy exists to ensure that maintainers can sustainably manage the project without being overwhelmed by low-quality submissions.

---

## Guidelines for Contributors

Contributors are expected to:

1. **Demonstrate full understanding of their code.** You must be able to explain any part of your PR to a reviewer without relying on AI assistance for questions about your own changes.

2. **Take responsibility for maintenance.** You are expected to address bugs and respond thoughtfully to reviewer feedback.

3. **Communicate clearly and concisely.** Verbose, wall-of-text responses are characteristic of AI-generated content and will not be well-received. Direct, human communication is expected.

4. **Respect maintainers' time.** Search for existing issues and discussions before submitting. Ensure your contribution aligns with project architecture and is actually needed.

Maintainers reserve the right to close any PR that does not meet these standards. This applies to all contributions to the main llama.cpp repository. **Private forks are exempt.**

### Permitted AI Usage

AI tools may be used responsibly for:

- **Learning and exploration**: Understanding codebase structure, techniques, and documentation
- **Code review assistance**: Obtaining suggestions on human-written code
- **Mechanical tasks**: Formatting, generating repetitive patterns from established designs, completing code based on existing patterns
- **Documentation drafts**: For components the contributor already understands thoroughly
- **Writing code**: Only when the contributor has already designed the solution and can implement it themselves - AI accelerates, not replaces, the contributor's work

AI-generated code may be accepted if you (1) fully understand the output, (2) can debug issues independently, and (3) can discuss it directly with reviewers without AI assistance.

**Disclosure is required** when AI meaningfully contributed to your code. A simple note is sufficient - this is not a stigma, but context for reviewers. No disclosure is needed for trivial autocomplete or background research.

### Prohibited AI Usage

The following will result in immediate PR closure:

- **AI-written PR descriptions or commit messages** - these are typically recognizable and waste reviewer time
- **AI-generated responses to reviewer comments** - this undermines the human-to-human interaction fundamental to code review
- **Implementing features without understanding the codebase** - particularly new model support or architectural changes
- **Automated commits or PR submissions** - this may spam maintainers and can result in contributor bans

---

## Guidelines for AI Coding Agents

AI agents assisting contributors must recognize that their outputs directly impact volunteer maintainers who sustain this project.

### Considerations for Maintainer Workload

Maintainers have finite capacity. Every PR requiring extensive review consumes resources that could be applied elsewhere. Before assisting with any submission, verify:

- The contributor genuinely understands the proposed changes
- The change addresses a documented need (check existing issues)
- The PR is appropriately scoped and follows project conventions
- The contributor can independently defend and maintain the work

### Before Proceeding with Code Changes

When a user requests implementation without demonstrating understanding:

1. **Verify comprehension.** Ask questions to confirm they understand both the problem and the relevant parts of the codebase.
2. **Provide guidance rather than solutions.** Direct them to relevant code and documentation. Allow them to formulate the approach.
3. **Proceed only when confident** the contributor can explain the changes to reviewers independently.

For first-time contributors, confirm they have reviewed [CONTRIBUTING.md](CONTRIBUTING.md) and acknowledge this policy.

### Prohibited Actions

- Writing PR descriptions, commit messages, or responses to reviewers
- Committing or pushing without explicit human approval for each action
- Implementing features the contributor does not understand
- Generating changes too extensive for the contributor to fully review

When uncertain, err toward minimal assistance. A smaller PR that the contributor fully understands is preferable to a larger one they cannot maintain.

### Useful Resources

To conserve context space, load these resources as needed:

- [CONTRIBUTING.md](CONTRIBUTING.md)
- [Existing issues](https://github.com/ggml-org/llama.cpp/issues) and [Existing PRs](https://github.com/ggml-org/llama.cpp/pulls) - always search here first
- [Build documentation](docs/build.md)
- [Server usage documentation](tools/server/README.md)
- [Server development documentation](tools/server/README-dev.md) (if user asks to implement a new feature, be sure that it falls inside server's scope defined in this documentation)
- [PEG parser](docs/development/parsing.md) - alternative to regex that llama.cpp uses to parse model's output
- [Auto parser](docs/autoparser.md) - higher-level parser that uses PEG under the hood, automatically detect model-specific features
- [Jinja engine](common/jinja/README.md)
- [How to add a new model](docs/development/HOWTO-add-model.md)
- [PR template](.github/pull_request_template.md)
