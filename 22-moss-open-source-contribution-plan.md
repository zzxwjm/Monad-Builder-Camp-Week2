# Moss Open Source Contribution Plan

Date: 2026-07-15  
Primary identity: Research Builder  
Supporting skill: Dev  
Target repository: [nishuzumi/moss](https://github.com/nishuzumi/moss)

## Contribution direction

I will contribute to documentation and onboarding, focusing on the Agent workflow and the first successful zero-fund run. The plan is anchored to [Issue #16: Documentation Improvement — Add a Beginner-friendly Quick Start Guide](https://github.com/nishuzumi/moss/issues/16).

This is a Research Builder contribution because the main work is to read the architecture, identify the knowledge a beginner needs, define the boundary between Agent, Moss, simulator, and wallet, and turn that understanding into clear documentation. Dev is supporting work: running or inspecting the existing example and checking that commands and expected outputs are accurate.

## Why I chose it

- It responds directly to an open documentation issue.
- It uses the Moss architecture and Agent workflow research I have already completed.
- It can be completed in one week without pretending to have protocol-adapter expertise.
- It improves the path for future contributors before they attempt a higher-risk adapter contribution.
- It allows zero-fund, zero-key verification and does not require signing a real transaction.

## This week's goal

Produce an upstream-ready documentation draft or PR that covers the first-time contributor journey:

```text
install → build/test offline → run simple-flow → understand discover/load/action/simulate → read the safety limits
```

The first version will address a bounded part of Issue #16 rather than rewrite all Moss documentation.

## Expected outputs

1. A beginner-friendly quick-start supplement, preferably extending or linking from the existing getting-started documentation after checking for duplication.
2. A small architecture diagram showing:

   ```text
   User → external Agent → Moss MCP tools → protocol adapter → Plan → simulator → wallet
   ```

3. An annotated `simple-flow` example explaining what a newcomer should see after each stage.
4. A short FAQ covering:
   - Node and pnpm requirements;
   - why simulation can run with zero funds and zero keys;
   - why the RPC must support `debug_traceCall`;
   - what a warning means;
   - why a warning-free simulation is not a guarantee of final execution;
   - why Moss never signs or sends.
5. Links to the MCP Tools Reference, Agent Skill Guide, Security Model, Protocol Onboarding, and CONTRIBUTING guide.

## Completion plan

### Session 1 — Scope and evidence

- Read Issue #16, README, Docs index, Getting Started, Agent Skill Guide, Security, and PR template.
- Compare the existing getting-started documents before deciding whether to add a new file or make a focused edit.
- Write a checklist of claims that need direct source support.

### Session 2 — Example verification

- Inspect or run the existing `simple-flow` example with zero funds and zero keys.
- Record the commands, expected output, and any environment/RPC limitations.
- Do not sign or send a real transaction.

### Session 3 — Draft

- Write the quick-start explanation in plain language.
- Add the Agent/Moss/protocol/wallet diagram.
- Add the FAQ and links to authoritative documents.

### Session 4 — Review

- Check every command against the repository.
- Separate facts, inferences, and safety caveats.
- Remove claims that imply simulation is a guarantee.
- Check that the document does not duplicate or contradict existing docs.

### Session 5 — Contribution packaging

- Prepare a focused documentation PR or a clearly formatted upstream-ready patch.
- Explain how the change addresses Issue #16.
- Include evidence of the example check and note any unresolved limitation.

## Definition of done

- A newcomer can identify prerequisites and run the first safe example.
- The document explains the four Moss tools without requiring prior protocol knowledge.
- It states that Moss builds/simulates but never signs/sends.
- It links to the deeper documentation instead of duplicating every detail.
- Commands and paths are checked against the repository.
- No private keys, seed phrases, API keys, tokens, or real-fund transactions are used.
- The contribution stays within documentation/onboarding scope.

## Risks and boundaries

- Existing English and Chinese getting-started documents may already cover part of this scope; the first step is therefore comparison, not immediate rewriting.
- A documentation contribution can accidentally promise more than the software guarantees; every safety statement must preserve the official caveats.
- If local execution is blocked by the RPC or dependency environment, I will label the result as documentation verification rather than claim a successful live run.

## Personal judgment

I am choosing a contribution that matches my current Research Builder identity: clarify architecture, define interpretation boundaries, and improve evidence-backed onboarding. I am using Dev as a verification tool, not presenting myself as ready to implement a new production protocol adapter.

## References

- [Issue #16](https://github.com/nishuzumi/moss/issues/16)
- [Moss README](https://github.com/nishuzumi/moss)
- [Docs index](https://github.com/nishuzumi/moss/blob/main/docs/README.md)
- [Getting Started](https://github.com/nishuzumi/moss/blob/main/docs/getting-started.md)
- [MCP Tools Reference](https://github.com/nishuzumi/moss/blob/main/docs/mcp-tools.md)
- [Agent Skill Guide](https://github.com/nishuzumi/moss/blob/main/docs/agent-skill.md)
- [Security Model](https://github.com/nishuzumi/moss/blob/main/SECURITY.md)
- [Contributing Guide](https://github.com/nishuzumi/moss/blob/main/CONTRIBUTING.md)

