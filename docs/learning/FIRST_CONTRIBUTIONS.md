# Making Your First Contribution to Medusa

**A friendly guide for first-time contributors**

**Last Updated:** November 19, 2025
**Version:** 2.11.3
**Time to First PR:** 1-3 hours

---

## Table of Contents

1. [Welcome!](#welcome)
2. [Understanding the Contribution Process](#understanding-the-contribution-process)
3. [Finding Good First Issues](#finding-good-first-issues)
4. [Types of Contributions](#types-of-contributions)
5. [Before You Code](#before-you-code)
6. [Development Checklist](#development-checklist)
7. [Pull Request Best Practices](#pull-request-best-practices)
8. [After Your PR](#after-your-pr)
9. [Recommended First Contributions](#recommended-first-contributions)
10. [Common Questions](#common-questions)

---

## Welcome!

Thank you for considering contributing to Medusa! We're excited to have you join our community.

**Why Contribute?**
- Learn from experienced developers
- Build your portfolio
- Give back to open source
- Improve a tool you use
- Join a welcoming community

**Don't be intimidated!** Everyone starts somewhere. The maintainers are friendly and will help guide you through the process.

---

## Understanding the Contribution Process

### The Git Workflow

Here's the typical flow for contributing to Medusa:

```
1. Fork the repository
   ↓
2. Clone your fork locally
   ↓
3. Create a new branch
   ↓
4. Make your changes
   ↓
5. Commit with a clear message
   ↓
6. Push to your fork
   ↓
7. Open a Pull Request (PR)
   ↓
8. Respond to feedback
   ↓
9. Celebrate! 🎉
```

### Visual Guide

```
medusajs/medusa (upstream)
    ↓ (fork)
your-username/medusa (origin)
    ↓ (clone)
Your local machine
    ↓ (work on branch)
Make changes → Commit → Push
    ↓
Pull Request → Review → Merge!
```

### Code Review Expectations

- **Response time:** Maintainers usually respond within 1-3 days
- **Review rounds:** Expect 1-3 rounds of feedback (this is normal!)
- **Feedback style:** Constructive and educational
- **Approval:** At least one maintainer approval needed

### CI/CD Checks

Your PR will automatically run:
- **Tests:** Unit and integration tests must pass
- **Linting:** Code style checks (ESLint, Prettier)
- **Type checking:** TypeScript validation
- **Build:** Ensure code compiles

Don't worry if checks fail - you can fix them!

---

## Finding Good First Issues

### GitHub Labels to Look For

1. **`good-first-issue`**
   - Explicitly marked for newcomers
   - Usually well-documented
   - Smaller in scope
   - [View all good first issues](https://github.com/medusajs/medusa/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)

2. **`help-wanted`**
   - Community contributions encouraged
   - May be slightly more complex
   - Maintainers available to help

3. **`documentation`**
   - Perfect for first-timers
   - Low risk of breaking things
   - High impact for users

4. **`bug`** + **`good-first-issue`**
   - Small, well-defined bug fixes
   - Clear expected behavior
   - Good learning opportunity

### Where to Look

**GitHub Issues:**
```
https://github.com/medusajs/medusa/issues
```
Filter by labels mentioned above.

**GitHub Discussions:**
```
https://github.com/medusajs/medusa/discussions
```
Browse for feature requests or questions that need documentation.

**Discord Community:**
```
https://discord.gg/medusajs
```
Check #contributions channel for ideas.

**Roadmap:**
```
https://github.com/medusajs/medusa/projects
```
See what's planned and offer to help.

### How to Choose

**Good first issue characteristics:**
- Clear description of the problem
- Suggested solution provided
- Limited scope (touches few files)
- Good test coverage exists
- Not tagged with "complex" or "advanced"

**Example good first issues:**
- "Fix typo in documentation"
- "Add code example to API docs"
- "Update outdated screenshot"
- "Add validation to existing endpoint"
- "Write test for uncovered function"

---

## Types of Contributions

### 1. Documentation (Easiest Start)

**Why start here:**
- Low risk of breaking things
- Immediate impact
- Learn the codebase
- Build confidence

**Examples:**
- Fix typos or grammar
- Improve explanations
- Add code examples
- Update outdated information
- Add missing API documentation

**Effort:** 10-30 minutes

### 2. Bug Fixes

**Why do this:**
- Clear problem definition
- Existing tests often catch bugs
- Valuable contribution
- Good learning experience

**Examples:**
- Fix validation errors
- Correct data type issues
- Resolve edge cases
- Fix incorrect behavior

**Effort:** 1-3 hours

### 3. Tests

**Why this matters:**
- Improve code quality
- Learn codebase deeply
- Safe changes (just tests)
- Always appreciated

**Examples:**
- Add missing unit tests
- Improve test coverage
- Add integration tests
- Fix flaky tests

**Effort:** 1-2 hours

### 4. Small Features

**Why contribute features:**
- Solve real problems
- Creative work
- Significant impact
- Portfolio piece

**Examples:**
- Add optional parameter to API
- Create utility function
- Add configuration option
- Extend existing functionality

**Effort:** 3-6 hours

### 5. Examples and Tutorials

**Why create these:**
- Help other developers
- Showcase Medusa capabilities
- Document patterns
- Build reputation

**Examples:**
- Integration guide
- Deployment tutorial
- Best practices guide
- Video walkthrough

**Effort:** 2-4 hours

---

## Before You Code

### 1. Read CONTRIBUTING.md

**Location:** `/home/user/medusa/CONTRIBUTING.md`

Key things to note:
- Code style guidelines
- Commit message format
- Testing requirements
- Development setup

**Spend 15 minutes** reading this document carefully.

### 2. Comment on the Issue to Claim It

**Before starting work, comment on the issue:**

```
Hi! I'd like to work on this issue. I'm thinking of approaching it by [brief description]. Does that sound good?
```

**Why this matters:**
- Avoids duplicate work
- Gets maintainer feedback early
- Shows you've thought about the solution
- Establishes communication

**Example:**
```
I noticed the documentation for the product endpoint is outdated. I'll update it with the new fields added in v2.11 and add a complete example. I should have a PR ready by the end of the week. Does this approach work?
```

### 3. Discuss Your Approach

**For anything beyond trivial fixes, discuss first:**
- How you plan to solve it
- What files you'll change
- Any concerns or questions
- Timeline estimate

**This prevents:**
- Wasted effort on wrong approach
- Major revisions later
- Misunderstanding requirements

### 4. Set Up Development Environment

**Follow the setup guide:**

See `/home/user/medusa/docs/learning/GETTING_STARTED.md`

**Verify setup:**
```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/medusa.git
cd medusa

# Add upstream remote
git remote add upstream https://github.com/medusajs/medusa.git

# Install dependencies
yarn install

# Build packages
yarn build

# Run tests to ensure everything works
yarn test

# Start development server
yarn dev
```

**If tests pass, you're ready to contribute!**

---

## Development Checklist

Follow this checklist for every contribution:

### 1. Create a Feature Branch

```bash
# Update your main branch
git checkout main
git pull upstream main

# Create a new branch with a descriptive name
git checkout -b fix/product-validation-error
# or
git checkout -b docs/update-api-examples
# or
git checkout -b feat/add-sku-prefix
```

**Branch naming conventions:**
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `feat/` - New features
- `test/` - Test additions
- `chore/` - Maintenance tasks

### 2. Make Your Changes

**Write clean code:**
- Follow existing code style
- Add comments for complex logic
- Keep changes focused and minimal
- Reuse existing utilities

**Example - Good:**
```typescript
// Good: Clear, focused change
export function validateSKU(sku: string): boolean {
  // SKU must be alphanumeric and 3-20 characters
  return /^[A-Z0-9]{3,20}$/i.test(sku)
}
```

**Example - Avoid:**
```typescript
// Avoid: Multiple unrelated changes
export function validateSKU(sku: string): boolean {
  // ... validation logic
  // Also reformatting entire file
  // Also fixing unrelated bugs
  // Also adding new features
}
```

### 3. Add Tests

**Required for:**
- Bug fixes (test that proves the bug is fixed)
- New features (test all new functionality)
- API changes (test new behavior)

**Example test:**
```typescript
describe("validateSKU", () => {
  it("should accept valid SKUs", () => {
    expect(validateSKU("ABC123")).toBe(true)
    expect(validateSKU("PROD001")).toBe(true)
  })

  it("should reject invalid SKUs", () => {
    expect(validateSKU("AB")).toBe(false) // too short
    expect(validateSKU("ABC-123")).toBe(false) // invalid character
    expect(validateSKU("")).toBe(false) // empty
  })
})
```

**Run tests:**
```bash
# Run all tests
yarn test

# Run specific test file
yarn test path/to/test.spec.ts

# Run in watch mode
yarn test --watch
```

### 4. Update Documentation

**Update if you:**
- Add new features
- Change API behavior
- Fix bugs that affect usage
- Add configuration options

**Files to check:**
- JSDoc comments in code
- `/home/user/medusa/docs/` files
- README.md
- Type definitions

**Example:**
```typescript
/**
 * Validates a product SKU format.
 *
 * @param sku - The SKU string to validate
 * @returns True if valid, false otherwise
 *
 * @example
 * ```typescript
 * validateSKU("PROD001") // true
 * validateSKU("AB") // false (too short)
 * ```
 */
export function validateSKU(sku: string): boolean {
  return /^[A-Z0-9]{3,20}$/i.test(sku)
}
```

### 5. Run Linting

**Check code style:**
```bash
# Run ESLint
yarn lint

# Auto-fix issues
yarn lint --fix

# Run Prettier
yarn format
```

**Common linting errors:**
- Unused imports
- Missing semicolons
- Inconsistent spacing
- Unused variables

**Fix before committing!**

### 6. Test Locally

**Beyond automated tests:**
- Run the application
- Test your changes manually
- Try edge cases
- Check UI changes in browser

**Example:**
```bash
# Start dev server
yarn dev

# In another terminal, test your API change
curl http://localhost:9000/admin/products \
  -H "Authorization: Bearer TOKEN"

# Check the admin UI
open http://localhost:9000/app
```

### 7. Commit with Clear Message

**Good commit messages:**
```bash
# Format: <type>: <description>

git commit -m "fix: correct product SKU validation regex"
git commit -m "docs: add example for product API endpoint"
git commit -m "feat: add SKU prefix configuration option"
git commit -m "test: add unit tests for order workflow"
```

**Types:**
- `fix:` - Bug fixes
- `feat:` - New features
- `docs:` - Documentation
- `test:` - Tests
- `chore:` - Maintenance
- `refactor:` - Code refactoring
- `style:` - Formatting

**Bad commit messages (avoid):**
```bash
git commit -m "fixed stuff"
git commit -m "WIP"
git commit -m "asdf"
git commit -m "I hope this works"
```

---

## Pull Request Best Practices

### 1. Title and Description

**Good PR title:**
```
fix: correct product SKU validation regex
```

**Good PR description:**
```markdown
## What

Fixes the product SKU validation to properly handle alphanumeric characters.

## Why

The current regex only accepts uppercase letters, causing valid lowercase SKUs to be rejected.

## How

Updated the regex to include the case-insensitive flag.

Fixes #1234
```

**Template:**
```markdown
## What
[Brief description of the change]

## Why
[Why this change is needed]

## How
[How you implemented it]

## Testing
[How to test the change]

Fixes #[issue-number]
```

### 2. Link to Issue

**Always reference the related issue:**
```markdown
Fixes #1234
Closes #1234
Resolves #1234
```

**This:**
- Links PR to issue automatically
- Closes issue when PR is merged
- Provides context for reviewers

### 3. Keep PRs Small and Focused

**Good PR:**
- Changes 1-5 files
- Addresses one issue
- Clear scope
- Easy to review

**Too large:**
- Changes 20+ files
- Multiple unrelated fixes
- Hard to review
- Higher chance of errors

**If your PR is getting large, consider splitting it:**
```
PR 1: Core functionality
PR 2: Additional features
PR 3: Documentation updates
```

### 4. Add Screenshots/Demos

**For UI changes, include:**
- Before/after screenshots
- GIFs showing interaction
- Video demos for complex flows

**Example:**
```markdown
## Screenshots

### Before
![before](url)

### After
![after](url)
```

**Tools:**
- Screenshots: OS built-in tools
- GIFs: [Kap](https://getkap.co/), [LICEcap](https://www.cockos.com/licecap/)
- Videos: [Loom](https://loom.com/)

### 5. Request Review

**After submitting:**
- Don't request review immediately
- Wait for CI checks to pass
- Fix any failing checks first
- Then request review from maintainers

**Be patient:**
- Maintainers are volunteers or have other work
- Response time: 1-3 days typically
- You can politely ping after 5-7 days

### 6. Respond to Feedback

**When you get review comments:**

**Good responses:**
```
Thanks for the feedback! I'll update the code to use the existing helper function.
```

```
Good catch! I've fixed the edge case and added a test for it.
```

```
I'm not sure I understand the suggestion. Could you provide an example?
```

**Avoid:**
```
No, my way is better.
```

```
This is how I always do it.
```

```
[No response for weeks]
```

**Remember:**
- Reviewers want to help
- Feedback makes code better
- Questions are welcome
- Be open to learning

### 7. Make Requested Changes

**Process:**
1. Read all comments carefully
2. Ask questions if unclear
3. Make the changes
4. Commit with clear message
5. Push to your branch
6. Mark comments as resolved
7. Re-request review

**Commits for changes:**
```bash
# Good: Clear what was addressed
git commit -m "refactor: use existing validateSKU helper"
git commit -m "test: add edge case for empty SKU"

# Avoid: Vague
git commit -m "updates"
git commit -m "fixed review comments"
```

---

## After Your PR

### What Happens Next

1. **Review process:**
   - Maintainers review your code
   - May request changes
   - May approve immediately

2. **Approval:**
   - Need at least 1 maintainer approval
   - All checks must pass
   - No merge conflicts

3. **Merge:**
   - Maintainer merges your PR
   - Code goes into main branch
   - Will be in next release

### Addressing Review Comments

**Stay positive and professional:**
- Thank reviewers for their time
- Be open to suggestions
- Ask questions when unclear
- Learn from feedback

**Example conversation:**
```
Reviewer: "Can we use the existing validateFormat() function instead?"

You: "Good idea! I didn't know that function existed. I've updated the code to use it. Could you point me to where these helper functions are documented so I can reference them in the future?"

Reviewer: "They're in utils/validation.ts. I'll add a comment to the docs about this!"
```

### Keeping PR Updated

**If main branch changes:**
```bash
# Update your main branch
git checkout main
git pull upstream main

# Update your feature branch
git checkout your-feature-branch
git rebase main

# Force push (rebase rewrites history)
git push --force-with-lease
```

**If you need to make changes:**
```bash
# Make your changes
# Commit them
git commit -m "fix: address review comments"

# Push to your branch
git push
```

The PR updates automatically when you push!

### Celebration Time!

**When your PR is merged:**
- Celebrate! You're an open source contributor!
- Share on social media
- Add to your portfolio
- Thank the reviewers
- Look for your next contribution

**You're now part of the Medusa community!**

---

## Recommended First Contributions

Start with one of these to build confidence:

### 1. Fix a Typo in Documentation (VERY EASY)

**Time:** 10 minutes
**Files:** 1
**Risk:** None

**Example:**
```markdown
File: /docs/learning/GETTING_STARTED.md

Change:
- "Medusa is a headles e-commerce platform"
+ "Medusa is a headless e-commerce platform"
```

**Why start here:**
- Understand PR process
- Zero risk
- Quick win
- Helpful contribution

### 2. Add Code Example to Docs (EASY)

**Time:** 20-30 minutes
**Files:** 1-2
**Risk:** Low

**Example:**
Add a complete example to API documentation showing how to create a product with variants.

**Why do this:**
- Learn API
- Help other developers
- Practice documentation writing
- Build confidence

### 3. Fix a Small Bug (MEDIUM)

**Time:** 1-2 hours
**Files:** 2-3
**Risk:** Low (with tests)

**Example:**
Fix validation error that rejects valid input.

**Why try this:**
- Real bug fixing experience
- Learn debugging
- Write tests
- Tangible impact

### 4. Write a Tutorial (MEDIUM)

**Time:** 2-3 hours
**Files:** 1-2
**Risk:** None

**Example:**
Write a guide for "How to add a custom field to products".

**Why create tutorials:**
- Deep learning
- Help community
- Demonstrate expertise
- Reusable content

### 5. Add Tests (MEDIUM)

**Time:** 1-2 hours
**Files:** 1-2
**Risk:** Low

**Example:**
Add unit tests for an untested utility function.

**Why write tests:**
- Improve code quality
- Learn testing patterns
- Safe contribution
- Always appreciated

---

## Common Questions

### Q: I'm not sure if my idea is good. Should I still suggest it?

**A:** Yes! Open a discussion or issue to get feedback. The worst that can happen is someone says "no thanks" - and they'll do it kindly.

### Q: How do I know if someone is already working on an issue?

**A:** Check:
1. Is there a linked PR?
2. Did someone comment saying they're working on it?
3. Is the issue assigned to someone?

If unclear, ask in a comment: "Is anyone working on this?"

### Q: My PR has been sitting for a week with no response. What should I do?

**A:** Politely ping:
```
Hi! Just checking if this is ready for review. Happy to make any changes needed. Thanks!
```

### Q: The CI checks are failing and I don't know why.

**A:** Don't panic!
1. Click on the failing check
2. Read the error message
3. Google the error if unclear
4. Ask for help in PR comments
5. Maintainers can help debug

### Q: Someone requested changes but I don't understand them.

**A:** Ask! For example:
```
Thanks for the review! Could you clarify what you mean by "use the existing pattern"? An example would be helpful.
```

### Q: Can I work on multiple issues at once?

**A:** Better to focus on one at a time:
- Faster to complete
- Easier to manage
- Better learning
- Less context switching

Once your first PR is merged, take on another!

### Q: I made a mistake in my PR. Can I fix it?

**A:** Absolutely! Just push new commits to your branch. The PR updates automatically.

### Q: Do I need to be an expert to contribute?

**A:** No! We welcome all skill levels. Start small and learn as you go.

### Q: What if my PR gets rejected?

**A:** It happens to everyone!
- Don't take it personally
- Learn from the feedback
- Try another issue
- Keep contributing

Rejection is learning!

---

## Additional Resources

### Medusa Resources

- **Documentation:** `/home/user/medusa/docs/learning/`
- **Contributing Guide:** `/home/user/medusa/CONTRIBUTING.md`
- **Code of Conduct:** `/home/user/medusa/CODE_OF_CONDUCT.md`
- **Discord:** https://discord.gg/medusajs
- **GitHub:** https://github.com/medusajs/medusa

### Git & GitHub Resources

- **GitHub Docs:** https://docs.github.com/
- **Git Basics:** https://git-scm.com/book/en/v2
- **How to Write a Git Commit Message:** https://chris.beams.io/posts/git-commit/
- **First Contributions:** https://github.com/firstcontributions/first-contributions

### General Open Source

- **How to Contribute to Open Source:** https://opensource.guide/how-to-contribute/
- **Open Source Friday:** https://opensourcefriday.com/

---

## Final Tips

**Do:**
- Start small
- Ask questions
- Be patient
- Learn from feedback
- Celebrate wins
- Help others once you learn

**Don't:**
- Rush your PR
- Skip tests
- Ignore feedback
- Take criticism personally
- Give up after rejection
- Be afraid to ask for help

---

## Your First PR Checklist

Before submitting your first PR, verify:

- [ ] I've read CONTRIBUTING.md
- [ ] I've claimed the issue in comments
- [ ] Tests pass locally
- [ ] Linting passes
- [ ] I've added/updated tests
- [ ] I've updated documentation
- [ ] Commit messages are clear
- [ ] PR description is complete
- [ ] I've linked to the issue
- [ ] I'm ready for feedback

---

**Welcome to the Medusa community!** We're excited to see your contributions. Don't hesitate to ask for help - we're all here to learn and grow together.

**Now go make your first contribution!**

**Questions?** Ask in:
- Discord #contributions channel
- GitHub Discussions
- PR comments

**Good luck, and happy contributing!**
