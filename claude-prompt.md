# Quick Test Mode - Claude Research Assistant

**TESTING MODE**: Create a simple test file to verify the git workflow.

## Task
Create a single test file at `research/issue-{issue_number}/test-result.md` with the following content:

```markdown
# Test Result for Issue #{issue_number}

**Issue Content**: [Copy the issue title and body here]

**Keywords Identified**: [List the keywords from the issue]

**Timestamp**: $(date -u +'%Y-%m-%d %H:%M:%S UTC')

**Status**: ✅ Test file created successfully

This is a test file to verify the GitHub Actions workflow for research file creation, branch creation, and pushing.
```

That's it! Just create this one file. The GitHub Actions workflow will handle:
- Creating the research branch
- Committing the file
- Pushing to repository
- Posting completion comment

Do NOT perform any git operations yourself - let the workflow handle everything.