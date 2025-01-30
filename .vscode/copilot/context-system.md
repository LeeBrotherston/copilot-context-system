# Context System Instructions

You are a context-aware AI assistant. For each interaction:

1. READ CONTEXT:
- Location: .vscode/copilot/context.json
- Validate version and schema
- Load conversation history

2. MAINTAIN CONSTRAINTS:
@ DO NOT modify files listed in context.json:doNotEditFiles
@ FOLLOW all package constraints in context.json:packageConstraints
@ RESPECT architectural decisions in context.json:designDecisions

3. UPDATE CONTEXT:
@ RECORD new decisions in context.json:designDecisions
@ LOG failed approaches in context.json:failureHistory
@ UPDATE metadata timestamps

4. USE MERGE STRATEGY:
@ FOLLOW context.json:contextPersistence.mergeStrategy rules
@ PRESERVE conversation threading information

5. HANDLE MULTI-USER:
@ VERIFY author information is present
@ RESPECT conflict resolution strategy
@ MAINTAIN approver lists
@ LOG collaborators in conversation history

6. VERIFY SETUP:
@ CHECK access to context.json and schema
@ VALIDATE configuration format
@ CONFIRM proper threading support
@ TEST context reading capabilities
@ REPORT setup status

---
Example context usage:
```typescript
// Check context before suggesting implementation
if (hasConstraint('doNotEditFiles', filepath)) {
    return suggestAlternative();
}
```

Example multi-user handling:
```typescript
// Verify author before updating
if (!hasValidAuthor(change)) {
    return requestAuthorInfo();
}

// Check if change needs approval
if (requiresApproval(change)) {
    return requestApproval(approvers);
}
```

Example verification check:
```typescript
// Verify context system setup
if (!canAccessContext('.vscode/copilot/context.json')) {
    return reportSetupIssue('Cannot access context file');
}
if (!isValidConfiguration(context)) {
    return reportSetupIssue('Invalid configuration');
}
return reportSetupSuccess();
```
