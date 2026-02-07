---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
---

# Brainstorming Ideas Into Designs

## Overview

Turn ideas into designs and specs efficiently. Understand the project context, clarify intent with batched questions, then present the design in sections for validation.

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits)
- Use AskUserQuestion to clarify intent — batch up to 4 related questions per call
- Prefer multiple-choice options (2-4 per question) over open-ended when possible
- Focus on: purpose, constraints, success criteria
- One round of questions is usually enough. Follow up only if answers reveal ambiguity

**Exploring approaches:**
- Propose 2-3 approaches with trade-offs using AskUserQuestion
- Lead with your recommended option and explain why
- Let the user pick (or write in "Other")

**Presenting the design:**
- Present the design in sections of 200-300 words
- Ask after each section whether it looks right so far
- Cover: architecture, components, data flow, error handling, testing

## After the Design

**Documentation:**
- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Implementation (if continuing):**
- Use AskUserQuestion: "Ready to set up for implementation?" with options for yes/not yet/need changes
- Use superpowers:using-git-worktrees to create isolated workspace
- Use superpowers:writing-plans to create detailed implementation plan

## Key Principles

- **Batch questions** - Use AskUserQuestion to ask up to 4 questions at once, not one-at-a-time interviews
- **Multiple choice preferred** - Easier to answer than open-ended
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Propose 2-3 approaches before settling
- **Incremental validation** - Present design in sections, validate each
