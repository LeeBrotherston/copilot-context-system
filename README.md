# Copilot Context System
[![Context Validation](https://github.com/LeeBrotherston/copilot-context-system/actions/workflows/context-validation.yml/badge.svg)](https://github.com/LeeBrotherston/copilot-context-system/actions/workflows/context-validation.yml)
[![Context Maintenance](https://github.com/LeeBrotherston/copilot-context-system/actions/workflows/context-maintenance.yml/badge.svg)](https://github.com/LeeBrotherston/copilot-context-system/actions/workflows/context-maintenance.yml)

A system for maintaining persistent context across conversations with GitHub Copilot.

## Overview

This system helps Copilot maintain context across conversations by:
- Tracking design decisions and constraints
- Recording failed approaches
- Maintaining conversation history
- Supporting repository-specific contexts

This context is intended to help copilot avoid things like breaking previously discussed, but since forgotten, constraints and patterns, avoid reintroducing previously failed code, etc.

This has been tested via use with the `Claude 3.5 Sonnet` model, and seems fine with `GPT 4o` from a cursory check, but it does not contain anything specific to this or any other model and so should work fairly universally.


## Setup

### VSCode Configuration (manual)

If you are using VSCode with the CoPilot plugin, you should be able to simply copy the `.vscode` directory (and it's contents) to the root of you repo. The included `.vscode/settings.json` configures Copilot to use the context system. The settings enable:

- Global instruction files
- Per-file thread management
- Smart context merging
- Context validation

You may also want to copy the workflows into your own workflows directory as they perform validation checks and context history cleanup activities, if you are committing your context to the repo.

Note: The `templates/context.json` file is a clean template with no existing context.  You should update the repository information and timestamps when you first copy it.


### VSCode Configuration (less manual)

You can use this repository as a template for your new project.

Note: The `templates/context.json` file is a clean template with no existing context.  You should update the repository information and timestamps when you first copy it.


### (optional) Add `.vscode/copilot/context.json` to your `.gitignore`

This system is designed so that it can be commited into a repository that is used by multiple people so that context can be shared, and particular items in that context attributed to specific individuals. However that means that this context is available to anyone with access to the repository and so for publicly available or sensitive repositories you may want to add that file to your `.gitignore`

`.gitignore` only stops new files from being tracked, and so you may also need to:

```
git rm --cached .vscode/copilot/context.json
```


### (optional) Configure Git for context file handling:
```bash
# Configure merge driver
git config merge.copilot-context.name "Copilot Context Merge Driver"
git config merge.copilot-context.driver "npx copilot-merge %O %A %B"

# Verify .gitattributes is present
# .vscode/copilot/**/*.json merge=copilot-context
```


## Usage

### VSCode

The `.vscode/settings.json` file should have been picked up by VSCode and you can simply use CoPilot as you did before, you can test that everything is working with the following prompt:

```
Please perform a context system verification:

1. Read and parse .vscode/copilot/context.json
2. Confirm schema version and validation settings
3. Verify your access to:
   - Context file location
   - Merge strategy
   - Threading configuration
4. List any existing:
   - Constraints
   - Design decisions
   - Conversation history

Please respond with:
- Setup status (success/failure)
- Any access or parsing errors
- List of missing or incorrect settings
- Confirmation that you can read and update context
```

### Manual Usage

If you are using CoPilot without VSCode then you can manually prompt like so:

#### Start New Conversations
```
Please read the context-instructions.md and conversation-context.json before proceeding.
<your question here>
```

#### Continue Existing Threads
```
Please read the context from .vscode/copilot/<thread-name>.json and continue our discussion about <topic>.
```

#### Record Important Decisions
   - Let Copilot update the context file when making significant decisions
   - Review context updates in Copilot's responses
   - Commit context changes with related code changes



# Random info most people won't care about

## Directory Structure
```
<repository-root>/
└── .vscode/
    └── /
        ├── contexts/            # Thread-specific contexts
        │   ├── feature-a/
        │   │   └── context.json
        │   └── feature-b/
        │       └── context.json
        ├── context.json        # Global context
        ├── schema.json         # JSON Schema
        └── context-system.md   # Copilot instructions
```
## Context File Structure

### Key Sections

- `metadata`: Version and repository information
- `constraints`: Technical limitations and requirements
- `designDecisions`: Architecture and implementation choices
- `failureHistory`: Record of approaches that didn't work
- `contextPersistence`: Storage and threading information

### Threading Support

For complex projects, create topic-specific context files:
```
.vscode/copilot/contexts/
  ├── auth-feature.json
  ├── database-schema.json
  └── api-design.json
```

## Example Context File

Here's a minimal example of a context file:

```json
{
  "metadata": {
    "version": "1.0.0",
    "schemaUrl": "../schema.json",
    "compatibility": {
      "minVersion": "1.0.0",
      "maxVersion": "2.0.0"
    },
    "repositoryInfo": {
      "url": "https://github.com/user/repo",
      "branch": "main"
    },
    "conversationHistory": {
      "currentId": "init",
      "thread": "main",
      "author": "user",
      "collaborators": []
    },
    "conversationId": "initial",
    "created": "2024-01-20T10:00:00Z",
    "lastUpdated": "2024-01-20T10:00:00Z"
  },
  "validation": {
    "lastValidated": "2024-01-20T10:00:00Z",
    "schemaVersion": "1.0.0",
    "validationTool": "ajv-cli"
  },
  "constraints": {
    "doNotEditFiles": [
      {
        "path": "src/generated/*",
        "reason": "Auto-generated files",
        "severity": "strict",
        "addedAt": "2024-01-20T10:00:00Z",
        "author": "system"
      }
    ]
  },
  "designDecisions": {
    "architecture": [
      {
        "decision": "Use TypeScript for all new code",
        "rationale": "Type safety and better IDE support",
        "alternatives": ["JavaScript", "Flow"],
        "madeAt": "2024-01-20T10:00:00Z",
        "author": "tech-lead"
      }
    ]
  },
  "contextPersistence": {
    "storageLocation": ".vscode/copilot/contexts",
    "previousContextFiles": [],
    "mergeStrategy": "smart",
    "threadNaming": "feature/name",
    "conflictResolution": "manual-merge"
  }
}
```

## Multi-User Workflow

1. **Author Attribution:**
   - All design decisions require an author
   - Conversations track participants
   - Changes to constraints need approval

2. **Conflict Resolution:**
   Configure in `contextPersistence.conflictResolution`:
   - `latest-wins`: Last commit takes precedence
   - `manual-merge`: Require manual resolution
   - `require-review`: Changes need approval

### Git Configuration

1. **Setup Merge Driver:**
```bash
git config merge.copilot-context.name "Copilot Context Merge Driver"
git config merge.copilot-context.driver "npx copilot-merge %O %A %B"
```

2. **Configure .gitattributes:**
```
.vscode/copilot/**/*.json merge=copilot-context
```

### Conflict Resolution Examples

1. **Latest Wins:**
```json
// Automatically take the most recent change based on timestamp
{
  "contextPersistence": {
    "conflictResolution": "latest-wins"
  }
}
```

2. **Manual Merge:**
```json
// Requires manual resolution with clear markers
{
  "designDecisions": {
    "architecture": [
      <<<<<<< HEAD
      {"decision": "Use MongoDB", "author": "alice"}
      =======
      {"decision": "Use PostgreSQL", "author": "bob"}
      >>>>>>> feature/database
    ]
  }
}
```

3. **Require Review:**
```json
// Changes must be approved before merge
{
  "contextPersistence": {
    "conflictResolution": "require-review",
    "reviewers": ["tech-leads", "architects"]
  }
}
```

### Author Validation

1. **Username Format:**
   - Alphanumeric characters and hyphens only
   - Must match git config user.name or specified team members
   - Examples: `john-doe`, `tech-lead-1`

2. **Required Fields:**
   - All decisions require author
   - All constraints require author
   - All changes to existing content require approver

## Maintenance
```

1. **Regular Cleanup:**
   - Archive completed conversation threads
   - Remove obsolete constraints
   - Update design decisions as architecture evolves

2. **Context File Management:**
   When your context file grows large:
   - Archive old conversations to dated files (e.g., `contexts/archived/2024-01.json`)
   - Keep only recent and relevant threads in main context
   - Maintain important design decisions and constraints
   - Use `previousContextFiles` array to reference archived context

3. **Version Control:**
   - Commit context changes with related code changes
   - Use meaningful commit messages to describe context updates

## Schema Validation

JSON Schema is provided for context file validation:
```bash
npx ajv-cli validate -s schema.json -d conversation-context.json
```
