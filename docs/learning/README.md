# Medusa Learning Path - START HERE

**Welcome to Medusa!** This is your comprehensive guide to understanding and contributing to the Medusa e-commerce platform.

**Documentation Date:** November 18, 2025
**Medusa Version:** 2.11.3
**Tech Stack Research:** [View Current Tech Stack →](./TECH_STACK_RESEARCH.md)

---

## 🎯 Who Is This For?

You are a motivated mid-level developer who:
- ✅ Is proficient in JavaScript/TypeScript and React
- ✅ May be new to some technologies used in this project
- ✅ Wants to understand the complete system architecture
- ✅ Aims to contribute meaningfully within your first 2-3 weeks
- ✅ Learns best through connections to familiar concepts

---

## 🚀 Quick Start Paths

### Path 1: "Get It Running" (Day 1)
**Goal:** Run the project locally and see it work

1. [GETTING_STARTED.md](./GETTING_STARTED.md) - Setup & first run
2. Try the admin dashboard at `http://localhost:9000/app`
3. Explore the API at `http://localhost:9000/admin`

### Path 2: "Understand the Architecture" (Week 1)
**Goal:** Grasp how Medusa works at a high level

1. [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - System design
2. [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) - Monorepo layout
3. [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) - How data moves through the system

### Path 3: "Master the Stack" (Week 2)
**Goal:** Deep dive into the technologies

1. [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) - All technologies explained
2. [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - React admin dashboard
3. [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - API server & framework
4. [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - MikroORM & PostgreSQL

### Path 4: "Start Contributing" (Week 3)
**Goal:** Make your first meaningful contribution

1. [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Code standards
2. [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Common tasks
3. [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md) - Daily workflow
4. [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md) - Where to start

---

## 📚 Complete Documentation Index

### Foundation (Start Here)

#### 🔰 [GETTING_STARTED.md](./GETTING_STARTED.md)
**When to read:** First thing, before anything else
**What you'll learn:** How to install, configure, and run Medusa locally
**Time:** 30-60 minutes
**Prerequisites:** Node.js 20+, PostgreSQL, basic terminal skills

#### 🏗️ [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
**When to read:** After getting it running
**What you'll learn:** Modular architecture, commerce modules, framework design
**Time:** 45 minutes
**Prerequisites:** GETTING_STARTED.md

#### 📁 [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)
**When to read:** When you need to find specific code
**What you'll learn:** Monorepo organization, where everything lives
**Time:** 30 minutes
**Prerequisites:** Basic understanding of monorepos

#### 💾 [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md)
**When to read:** When diving into specific technologies
**What you'll learn:** Deep dive into each technology, why it's used, how to use it
**Time:** 2-3 hours (reference document)
**Prerequisites:** ARCHITECTURE_OVERVIEW.md

#### 🔄 [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)
**When to read:** When understanding request/response cycles
**What you'll learn:** API requests, module interactions, database queries
**Time:** 45 minutes
**Prerequisites:** ARCHITECTURE_OVERVIEW.md

---

### Deep Dives

#### ⚛️ [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)
**When to read:** When working on the admin dashboard
**What you'll learn:** React patterns, routing, state management, UI components
**Time:** 1 hour
**Technologies:** React 18, Vite, TanStack Query, React Hook Form, TailwindCSS

#### 🔧 [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)
**When to read:** When working on APIs or modules
**What you'll learn:** Express setup, dependency injection, module loading, workflows
**Time:** 1.5 hours
**Technologies:** Express, Awilix, Workflows SDK, Zod

#### 🗄️ [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)
**When to read:** When working with data models or migrations
**What you'll learn:** MikroORM entities, repositories, migrations, relationships
**Time:** 1 hour
**Technologies:** MikroORM, PostgreSQL

#### 🔗 [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)
**When to read:** When understanding how modules work together
**What you'll learn:** Module relationships, link modules, event bus, workflows
**Time:** 45 minutes
**Prerequisites:** ARCHITECTURE_OVERVIEW.md

---

### Practical Guides

#### 🎨 [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)
**When to read:** Before writing code
**What you'll learn:** Code patterns, naming conventions, best practices
**Time:** 1 hour
**Keep as reference:** Yes

#### 📖 [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)
**When to read:** When you need to accomplish a specific task
**What you'll learn:** Step-by-step recipes for common tasks
**Time:** Reference as needed
**Topics:** Creating modules, adding API routes, writing migrations, etc.

#### 🗺️ [CODE_TOURS.md](./CODE_TOURS.md)
**When to read:** When you want to follow real code flows
**What you'll learn:** Guided walkthroughs of key features
**Time:** 30 minutes per tour
**Tours:** Product creation, order placement, payment processing, etc.

#### 🔨 [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
**When to read:** When you start daily development
**What you'll learn:** Git workflow, building, testing, debugging
**Time:** 30 minutes
**Keep as reference:** Yes

---

### Quality & Reference

#### 🧪 [TESTING_GUIDE.md](./TESTING_GUIDE.md)
**When to read:** When writing tests
**What you'll learn:** Testing strategy, unit tests, integration tests
**Time:** 1 hour
**Technologies:** Jest, Vitest, Supertest

#### 🐛 [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)
**When to read:** When things break (or before they do)
**What you'll learn:** Debugging techniques, common issues, logging
**Time:** 45 minutes
**Keep as reference:** Yes

#### 🔒 [SECURITY_GUIDE.md](./SECURITY_GUIDE.md)
**When to read:** When handling authentication, authorization, or sensitive data
**What you'll learn:** Security best practices, auth patterns, data protection
**Time:** 45 minutes
**Important:** Read before working on auth/payment features

#### 📡 [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
**When to read:** When working with or creating APIs
**What you'll learn:** API structure, endpoints, authentication, conventions
**Time:** 1 hour
**Keep as reference:** Yes

#### 🗂️ [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)
**When to read:** When you need to understand the data model
**What you'll learn:** All entities, relationships, key fields
**Time:** Reference as needed
**Keep as reference:** Yes

---

### Learning & Contributing

#### ✏️ [EXERCISES.md](./EXERCISES.md)
**When to read:** When you want hands-on practice
**What you'll learn:** Build real features step-by-step
**Time:** 30 minutes - 2 hours per exercise
**Prerequisites:** GETTING_STARTED.md, ARCHITECTURE_OVERVIEW.md

#### 🎯 [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)
**When to read:** When ready to contribute
**What you'll learn:** Good first issues, contribution process, where to start
**Time:** 30 minutes
**Prerequisites:** DEVELOPMENT_WORKFLOW.md

#### ❓ [FAQ.md](./FAQ.md)
**When to read:** When you have questions
**What you'll learn:** Answers to common questions
**Time:** Browse as needed
**Keep as reference:** Yes

---

## 🔍 Finding What You Need

### By Role

**Frontend Developer:**
1. GETTING_STARTED.md
2. FRONTEND_ARCHITECTURE.md
3. PROJECT_STRUCTURE.md (admin packages)
4. PATTERNS_AND_CONVENTIONS.md
5. DEVELOPMENT_WORKFLOW.md

**Backend Developer:**
1. GETTING_STARTED.md
2. BACKEND_ARCHITECTURE.md
3. DATABASE_ARCHITECTURE.md
4. INTEGRATION_GUIDE.md
5. API_DOCUMENTATION.md

**Full-Stack Developer:**
- Follow Path 2 → Path 3 → Path 4 above

**DevOps/Infrastructure:**
1. GETTING_STARTED.md
2. ARCHITECTURE_OVERVIEW.md
3. PROJECT_STRUCTURE.md
4. DEVELOPMENT_WORKFLOW.md

### By Task

**"I need to add a new API endpoint"**
→ BACKEND_ARCHITECTURE.md + HOW_TO_GUIDE.md

**"I need to create a new module"**
→ BACKEND_ARCHITECTURE.md + HOW_TO_GUIDE.md + CODE_TOURS.md

**"I need to modify the admin dashboard"**
→ FRONTEND_ARCHITECTURE.md + PROJECT_STRUCTURE.md

**"I need to add a database field"**
→ DATABASE_ARCHITECTURE.md + HOW_TO_GUIDE.md (migrations section)

**"I need to understand how X works"**
→ CODE_TOURS.md + grep for the feature in the codebase

**"I'm getting an error"**
→ DEBUGGING_GUIDE.md + FAQ.md

**"I want to contribute"**
→ FIRST_CONTRIBUTIONS.md + DEVELOPMENT_WORKFLOW.md

---

## 🧠 Learning Strategies

### Visual Learners
Start with:
- ARCHITECTURE_OVERVIEW.md (has diagrams)
- DATA_FLOW_GUIDE.md (sequence diagrams)
- CODE_TOURS.md (visual code walkthroughs)

### Hands-On Learners
Start with:
- GETTING_STARTED.md (get it running)
- EXERCISES.md (build things)
- HOW_TO_GUIDE.md (follow recipes)

### Conceptual Learners
Start with:
- ARCHITECTURE_OVERVIEW.md (big picture)
- TECH_STACK_GUIDE.md (why each technology)
- PATTERNS_AND_CONVENTIONS.md (design principles)

---

## ⏱️ Suggested Timelines

### Week 1: Foundation
- **Day 1:** GETTING_STARTED.md + explore the running app
- **Day 2:** ARCHITECTURE_OVERVIEW.md + PROJECT_STRUCTURE.md
- **Day 3:** TECH_STACK_GUIDE.md (skim) + DATA_FLOW_GUIDE.md
- **Day 4:** FRONTEND_ARCHITECTURE.md or BACKEND_ARCHITECTURE.md (based on role)
- **Day 5:** DATABASE_ARCHITECTURE.md + INTEGRATION_GUIDE.md

### Week 2: Deep Dive
- **Day 1:** PATTERNS_AND_CONVENTIONS.md
- **Day 2:** CODE_TOURS.md (all tours)
- **Day 3:** HOW_TO_GUIDE.md + try building something small
- **Day 4:** TESTING_GUIDE.md + write tests for your code
- **Day 5:** DEBUGGING_GUIDE.md + SECURITY_GUIDE.md

### Week 3: Contribute
- **Day 1:** DEVELOPMENT_WORKFLOW.md
- **Day 2:** FIRST_CONTRIBUTIONS.md + pick an issue
- **Day 3-5:** Work on your first contribution

---

## 💡 Pro Tips

1. **Keep TECH_STACK_RESEARCH.md open** - It tells you which patterns are current vs outdated
2. **Use CODE_TOURS.md** - Following real code is the fastest way to learn
3. **Don't skip GETTING_STARTED.md** - Even if you're experienced, Medusa has unique setup requirements
4. **Skim first, deep-dive later** - Read TECH_STACK_GUIDE.md quickly, then return to specific sections as needed
5. **Code alongside reading** - Open the actual files mentioned in the docs
6. **Join the community** - [Discord](https://discord.gg/medusajs) has 14,000+ members who can help

---

## 🔗 External Resources

### Official Medusa Documentation
- **Main Docs:** https://docs.medusajs.com/
- **API Reference:** https://docs.medusajs.com/api
- **User Guide:** https://docs.medusajs.com/user-guide
- **Commerce Modules:** https://docs.medusajs.com/resources/commerce-modules

### Community
- **Discord:** https://discord.gg/medusajs
- **GitHub Discussions:** https://github.com/medusajs/medusa/discussions
- **GitHub Issues:** https://github.com/medusajs/medusa/issues
- **Blog:** https://medusajs.com/blog/

### Technology Documentation
See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) for links to current documentation for all technologies used.

---

## 📈 Currency & Version Notes

**Documentation Currency:** ✅ Current as of November 18, 2025

**Technology Status:**
- ✅ Most technologies are current or one minor version behind
- ⚠️ Some technologies are 1-2 major versions behind (see TECH_STACK_RESEARCH.md)
- All patterns in this codebase are production-ready and valid

**Version Markers in Documentation:**
- ✅ **CURRENT (Nov 2025)** - Pattern is up-to-date
- ⚠️ **OUTDATED PATTERN** - Works but newer approaches exist
- 🚨 **DEPRECATED** - Should not be used in new code
- 🆕 **NEW IN 2025** - Recently introduced feature
- 🔍 **NEEDS VERIFICATION** - Requires deeper investigation
- ❓ **ASSUMPTION** - Based on inference, not confirmed

---

## 🤝 Contributing to This Documentation

Found an error? Have a suggestion? Want to add content?

1. **Small fixes:** Edit the file and submit a PR
2. **New sections:** Open an issue to discuss first
3. **Questions:** Check FAQ.md, then ask in Discord

**Documentation principles:**
- Link to actual code examples
- Use analogies to React/JS/TS concepts
- Mark uncertainties clearly
- Keep it practical and actionable

---

## 📞 Need Help?

1. **Check [FAQ.md](./FAQ.md)** - Common questions answered
2. **Search the docs** - Use Ctrl+F or grep
3. **Ask in Discord** - https://discord.gg/medusajs
4. **GitHub Discussions** - https://github.com/medusajs/medusa/discussions
5. **Open an issue** - https://github.com/medusajs/medusa/issues

---

**Ready to start?** → [Begin with GETTING_STARTED.md](./GETTING_STARTED.md)

**Want to understand the big picture first?** → [Start with ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)

**Prefer hands-on learning?** → [Jump to EXERCISES.md](./EXERCISES.md) (after GETTING_STARTED.md)

---

**Last Updated:** November 18, 2025
**Next Review:** March 2026 (or when Medusa 3.0 is released)
