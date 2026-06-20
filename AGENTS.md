# Repository Guidelines

## Project Structure & Module Organization

This repository is a Markdown study acervo for backend software engineering topics, focused on .NET/C#. Content is organized by numbered topic areas, such as `01-csharp-dotnet/`, `03-ef-dapper-postgresql/`, and `06-docker-k8s-cicd-azure/`. Each area contains an `_index.md` with the suggested reading order and one Markdown file per topic in kebab-case, for example `async-await.md` or `n-plus-one-problem.md`.

Root-level files provide project context: `README.md` explains the current structure and conventions, `CLAUDE.md` gives agent-specific workflow notes, and `study-archive-prompt.md` contains source prompting material. Do not store job descriptions in the repository; use them only to identify topics.

## Build, Test, and Development Commands

There is no application build pipeline or package manager configuration in this repository. Most work is documentation editing and review.

- `rg "term"`: search the acervo quickly.
- `git diff --check`: detect trailing whitespace and malformed whitespace before committing.
- `git status --short`: review changed files.
- `git log --oneline -5`: inspect recent commit style.

## Coding Style & Naming Conventions

Write in Brazilian Portuguese, keeping established technical terms in English when they are standard, such as `deadlock`, `change tracking`, or `deferred execution`. Use concise Markdown headings, short paragraphs, tables where they improve scanning, and fenced code blocks for examples.

Topic files should use kebab-case names and include YAML frontmatter with fields such as `id`, `area`, `difficulty`, `prerequisites`, `related`, `tags`, `sources`, `status`, and `last_updated`. Code examples should be idiomatic for .NET 8 and C# 12, clean, and compilable when practical.

## Testing Guidelines

There is no automated test suite. Treat review as documentation QA: verify links, frontmatter consistency, source quality, and technical accuracy. When adding C# examples, mentally validate syntax and prefer small snippets that demonstrate one idea clearly.

## Commit & Pull Request Guidelines

Use Conventional Commits in English, matching the existing history, for example `docs: add review notes for async-await`. Keep commits focused on one topic or structural change.

Pull requests should summarize the changed areas, list newly added or updated topic files, mention source or accuracy considerations, and link any relevant issue. Include screenshots only if a rendered Markdown view or diagram changed materially.
