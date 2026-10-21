# TASK: Analyze Codebase and Generate SKILL.md

You are a Senior Software Architect, Codebase Analyst, and AI Agent Skill Engineer.

Your task is to deeply analyze the existing codebase and generate comprehensive, accurate, and reusable `SKILL.md` documentation optimized for OpenCode AI agents.

## Phase 1 — Codebase Discovery

Explore the entire project systematically.

1. Identify the programming languages, frameworks, libraries, and dependencies.
2. Analyze the folder structure and project organization.
3. Identify application entry points.
4. Discover configuration files and environment variable usage.
5. Identify architectural patterns and design principles.
6. Analyze database schemas, migrations, models, and relationships.
7. Identify API routes, controllers, services, and middleware.
8. Discover reusable utilities, helpers, and shared components.
9. Analyze authentication, authorization, validation, and error handling.
10. Identify testing frameworks, build tools, and deployment configurations.

Do not assume anything without verifying it against the actual code.

## Phase 2 — Architecture Understanding

Understand how the application works internally.

Analyze:

- Application initialization and lifecycle
- Data flow between modules
- Dependencies between components
- Business logic and domain boundaries
- API request and response lifecycle
- Database access patterns
- State management
- Error handling and logging
- Security conventions
- Background jobs and integrations

Trace representative execution paths from entry point to final output.

Identify the existing coding conventions and patterns that future AI agents must follow.

## Phase 3 — Extract Development Knowledge

Extract actionable knowledge from the codebase.

Document:

- How to add new features
- How to modify existing features
- How to create new modules
- How to add API endpoints
- How to interact with the database
- How to implement validation
- How to handle errors
- How to write tests
- How to run builds and development commands
- How to avoid breaking existing functionality

Include real file paths and code examples when useful.

Never invent file paths, APIs, commands, or conventions.

## Phase 4 — Generate OpenCode Skills

Create a directory:

`.opencode/skills/`

Generate a main skill:

`.opencode/skills/codebase-guide/SKILL.md`

Use this structure:

---
name: codebase-guide
description: Guide for understanding, navigating, maintaining, debugging, and extending this project's codebase using its actual architecture and coding conventions.
---

# Codebase Guide

## Project Overview
## Technology Stack
## Project Structure
## Architecture
## Important Modules
## Application Data Flow
## Coding Conventions
## Database Patterns
## API Patterns
## Authentication and Security
## Error Handling
## Testing Strategy
## Development Workflow
## Feature Implementation Guide
## Debugging Guide
## Common Pitfalls
## Important Files Reference
## AI Agent Instructions

Each section must contain verified, project-specific knowledge.

Do not fill sections with generic software engineering advice.

## Phase 5 — Specialized Skills

If the project is sufficiently complex, create additional focused skills.

Examples:

- `.opencode/skills/database-guide/SKILL.md`
- `.opencode/skills/api-development/SKILL.md`
- `.opencode/skills/frontend-development/SKILL.md`
- `.opencode/skills/testing-guide/SKILL.md`
- `.opencode/skills/architecture-guide/SKILL.md`

Only create skills relevant to the actual project.

Avoid duplication between skills.

Each skill must have a clear purpose and valid YAML frontmatter containing `name` and `description`.

## Phase 6 — Optimize for AI Agents

Every skill must be:

- Concise but technically complete
- Actionable rather than theoretical
- Based on actual repository evidence
- Easy for OpenCode agents to discover
- Focused on repeatable development tasks
- Structured with clear Markdown headings
- Equipped with real examples and references

Prefer progressive disclosure.

Keep the main skill focused and move extensive technical details into specialized skills or supporting reference files.

## Phase 7 — Validation

After generating the skills:

1. Verify every referenced file path exists.
2. Verify documented commands against project configuration.
3. Check YAML frontmatter.
4. Ensure skill directory names match their skill names.
5. Remove duplicate or speculative information.
6. Confirm all instructions match actual code patterns.
7. Verify that no secrets or sensitive environment values are exposed.
8. Ensure existing project files remain unchanged unless explicitly required.

## Critical Rules

- Read and understand the code before writing documentation.
- Do not modify application source code.
- Do not refactor existing implementations.
- Do not install dependencies.
- Do not execute destructive commands.
- Do not expose credentials or secrets.
- Do not hallucinate architecture or functionality.
- Clearly mark anything that cannot be verified.
- Prefer existing project conventions over generic best practices.
- Do not create unnecessary skills.
- Use the repository as the source of truth.

## Expected Output

Create actual `SKILL.md` files in the repository.

Finally, provide a report containing:

1. Codebase architecture summary
2. Technologies and frameworks discovered
3. Skills created and their locations
4. Key development patterns identified
5. Important architectural risks or limitations
6. Validation results
7. Recommendations for future AI agent development

Begin by exploring the repository structure, then analyze the codebase in depth before generating any skill files.