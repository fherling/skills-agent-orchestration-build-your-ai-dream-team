# Agent team

This is the custom agent team I use to build Mona's Project Pulse dashboard. I orchestrate the work using GitHub Copilot CLI running in a Codespace, delegating tasks to the specialist agents defined below.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Breaks down complex requests into phases, delegates work to the Planner, Coder, and Designer, assigns each specialist an explicit file scope, and verifies the integrated result before reporting back.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository and documentation, identifies risks and edge cases, and produces an ordered implementation plan with file assignments and parallel/sequential guidance. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code-oriented tasks within the file scope assigned by the Orchestrator, including Project Pulse app logic and supporting configuration such as `.vscode/launch.json`.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Handles UI/UX, accessibility, information architecture, and visual design for Project Pulse, ensuring a polished dashboard look with deterministic CSS hooks like `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`
