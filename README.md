# Contribution [#]: [Issue Title]

**Contribution Number:** [1 / 2 / 3]  
**Student:** [Rafeek Hanna]  
**Issue:** [[GitHub issue link](https://github.com/wso2/product-is/issues/23288)]  
**Status:** [Phase I] [Complete]

---

## Why I Chose This Issue

[The "Back to sign in" link on the WSO2 Identity Server self-registration page fails with a 400 error because the `claims` JSON parameter in the callback URL is not URL-encoded. I chose this issue because it has clear reproduction steps, a bounded scope, and involves frontend URL handling which matches my web development background. The issue is labeled for new contributors and has active maintainer engagement.]

---

## Understanding the Issue

### Problem Description

[When a user clicks "Register" on an organization's My Account page and then clicks "Sign in" to go back, the server returns 400 Bad Request because the `claims` parameter contains raw JSON characters (`{`, `}`, `"`, `:`) that are not percent-encoded.]

### Expected Behavior

[The "Back to sign in" link should redirect to the login page successfully with the claims parameter properly encoded.]

### Current Behavior

[The link fails with HTTP 400 due to malformed query parameters.]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
