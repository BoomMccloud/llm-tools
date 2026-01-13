# LLM Tools

A collection of Claude Code configuration files including custom agents, commands, and skills.

## Structure

```
.claude/
├── agents/           # Custom agent definitions
│   ├── agent-ralph.md
│   ├── gemini.md
│   └── tdd-test-writer.md
├── commands/         # Custom slash commands
│   └── feature-pipeline.md
├── skills/           # Reusable skill definitions
│   ├── first-principle/
│   ├── frontend-design/
│   ├── guide/
│   ├── kiss/
│   ├── planning/
│   ├── review/
│   └── verify/
└── settings.json
```

## Usage

Copy the `.claude` directory to your project root to use these configurations with Claude Code.

```bash
cp -r .claude /path/to/your/project/
```

## Components

### Agents
- **agent-ralph** - Ralph Wiggum debugging technique
- **gemini** - Gemini API integration agent
- **tdd-test-writer** - Test-driven development test writer

### Commands
- **feature-pipeline** - Multi-stage feature development workflow

### Skills
- **first-principle** - Architectural design using first-principle thinking
- **frontend-design** - Production-grade frontend interface design
- **guide** - Step-by-step implementation guide generation
- **kiss** - Keep It Simple, Stupid principle evaluation
- **planning** - Structured documentation workflow
- **review** - Code quality and maintainability review
- **verify** - Specification verification against codebase
