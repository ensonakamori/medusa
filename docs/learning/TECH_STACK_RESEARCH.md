# Technology Stack Research (November 2025)

**Research conducted:** November 18, 2025
**Project:** Medusa (E-commerce Platform)
**Knowledge cutoff context:** This research bridges the gap between January 2025 (AI knowledge cutoff) and November 2025 (current date)

---

## Table of Contents

1. [Core Runtime & Language](#core-runtime--language)
2. [Frontend Technologies](#frontend-technologies)
3. [Backend Technologies](#backend-technologies)
4. [Database & ORM](#database--orm)
5. [Build & Development Tools](#build--development-tools)
6. [Testing Frameworks](#testing-frameworks)
7. [UI & Component Libraries](#ui--component-libraries)
8. [State & Form Management](#state--form-management)
9. [Monorepo Management](#monorepo-management)
10. [Other Key Dependencies](#other-key-dependencies)
11. [Summary of Currency Status](#summary-of-currency-status)

---

## Core Runtime & Language

### TypeScript - v5.6.2

**Current Status (Nov 2025):**
- Latest stable: v5.9 (released August 2025)
- Project uses: v5.6.2
- Status: ⚠️ **Slightly outdated** (2 minor versions behind)

**Important Updates Since Jan 2025:**
- TypeScript 5.7 released in November 2024
- TypeScript 5.8 reached GA on February 28, 2025 with improved JavaScript ecosystem interoperability
- TypeScript 5.9 released August 1, 2025 (current stable)
- TypeScript 6.0-dev is in development, designed as a transition point before TypeScript 7.0
- TypeScript 7.0 is in development (Go port of the compiler announced March 2025)

**What This Means for Learning:**
- The patterns in this project use TypeScript 5.6, which is very current
- All features and patterns are valid and modern
- No breaking changes between 5.6 and 5.9
- Safe to learn from this codebase's TypeScript patterns

**Official Resources:**
- Current Docs: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-9.html
- TypeScript 5.8 announcement: https://devblogs.microsoft.com/typescript/announcing-typescript-5-8/
- Migration guide: https://www.typescriptlang.org/docs/handbook/release-notes/overview.html

---

### Node.js - v20+

**Current Status (Nov 2025):**
- Latest LTS: Node.js 22.21.1 "Jod" (Maintenance LTS as of October 2025)
- Node.js 24.x "Krypton" is now Active LTS
- Project requires: v20+
- Status: ✅ **Supported** (Node.js 20 is still supported until April 2026)

**Important Updates Since Jan 2025:**
- Node.js 22.x moved to Maintenance LTS on October 21, 2025
- Node.js 24.x is now the Active LTS version
- Node.js 22.21.0+ bundles OpenSSL 3.5.2
- Built-in proxy support added in http/https.request and Agent
- Node.js 18 reached EOL at the end of April 2025

**What This Means for Learning:**
- This project's Node.js 20+ requirement is current and well-supported
- The codebase will work perfectly on Node.js 20, 22, or 24
- All modern Node.js features are available
- Patterns shown are current best practices

**Official Resources:**
- Node.js releases: https://nodejs.org/en/about/previous-releases
- Node.js 22 LTS announcement: https://nodesource.com/blog/Node.js-v22-Long-Term-Support-LTS
- Release schedule: https://github.com/nodejs/Release

---

## Frontend Technologies

### React - v18.3.1

**Current Status (Nov 2025):**
- Latest stable: v19.2.0 (released October 1, 2025)
- Project uses: v18.3.1
- Status: ⚠️ **One major version behind**

**Important Updates Since Jan 2025:**
- React 19.0.0 released December 2024
- React 19.1.0 released March 2025
- React 19.1.1 released July 28, 2025
- React 19.2.0 released October 1, 2025 (current stable)

**React 19 Major Features:**
- ✅ React Server Components (now stable)
- ✅ Actions API for form handling
- ✅ React Compiler for automatic optimization
- 🆕 `<Activity>` API for show/hide UI with state preservation
- 🆕 `useEffectEvent` hook for extracting non-reactive logic
- 🆕 Enhanced concurrent rendering capabilities
- 🆕 New Chrome DevTools performance profiling integration

**What This Means for Learning:**
- React 18 patterns in this project are **still valid and widely used**
- The project uses current React 18 best practices
- When upgrading to React 19, be aware of Server Components and Actions API
- The core concepts (hooks, components, state) remain the same

**Migration Considerations:**
- React 19 is backward compatible for most use cases
- Server Components and Actions are opt-in features
- Upgrading from 18 to 19 is generally smooth

**Official Resources:**
- React 19 announcement: https://react.dev/blog/2024/12/05/react-19
- React 19.2 release: https://react.dev/blog/2025/10/01/react-19-2
- Migration guide: https://react.dev/blog/2024/12/05/react-19#upgrading

---

### Vite - v5.4.21

**Current Status (Nov 2025):**
- Latest stable: v7.2 (with v6.4 receiving security patches)
- Project uses: v5.4.21
- Status: 🚨 **Two major versions behind**

**Important Updates Since Jan 2025:**
- Vite 6.0 released November 26, 2024
- Vite 7.0 released (date not specified in search, but current as of Nov 2025)
- Vite downloads grew from 7.5M to 17M per week

**Vite 6 Changes:**
- Dropped Node.js 21 support, added 22+ support
- Introduced Environment API for better dev/prod parity
- Significant performance improvements

**Vite 7 Changes:**
- Requires Node.js 20.19+, 22.12+ (dropped Node.js 18 after EOL)
- Changed default browser target from 'modules' to 'baseline-widely-available'
- Currently at v7.2 with active development

**What This Means for Learning:**
- Vite 5 patterns are still valid
- The project should consider upgrading to Vite 6 or 7
- Core build concepts remain the same
- Vite 7 has better performance and modern defaults

**Official Resources:**
- Vite 6 announcement: https://vite.dev/blog/announcing-vite6
- Vite 7 announcement: https://vite.dev/blog/announcing-vite7
- Migration from v5: https://v6.vite.dev/guide/migration
- Migration from v6: https://vite.dev/guide/migration

---

## Backend Technologies

### Express.js - v4.21.0

**Current Status (Nov 2025):**
- Latest stable: v5.0.0 (released October 15, 2024, now default on npm)
- Project uses: v4.21.0
- Status: ⚠️ **One major version behind** (but v4 is actively maintained)

**Important Updates Since Jan 2025:**
- Express 5.0.0 released October 2024 as the new default
- Express 4.21.0 (current in project) is a recent release with security fixes
- Express 4.21.2 includes security fix for CVE-2024-45590
- Express 6 is planned for future with performance focus

**Express 5.0 Major Changes:**
- ✅ **Promise Support**: Middleware can return rejected promises (caught automatically as errors)
- ✅ **Security**: Updated to path-to-regexp@8.x (ReDoS mitigation)
- ⚠️ **Breaking**: Dropped support for Node.js < v18
- 🆕 Body parser improvements (urlencoded depth customization)
- ⚠️ **Deprecated**: `res.location("back")` and `res.redirect("back")` magic strings

**What This Means for Learning:**
- Express 4.21 is still excellent for learning and production
- The patterns in this project are current
- Express 5 is mainly about modernization and security
- When migrating to Express 5, focus on promise-based error handling

**Official Resources:**
- Express 5.0 release notes: https://expressjs.com/en/changelog/
- What's new in Express 5: https://www.trevorlasn.com/blog/whats-new-in-express-5
- Migration guide: https://expressjs.com/en/guide/migrating-5.html

---

## Database & ORM

### MikroORM - v6.4.16

**Current Status (Nov 2025):**
- Latest stable: v6.6 (released approximately November 11, 2025)
- Project uses: v6.4.16
- Status: ⚠️ **Two minor versions behind**

**Important Updates Since Jan 2025:**
- MikroORM 6.5 released with filter improvements
- MikroORM 6.6 released (latest) with enhanced filter configurability on relations
- Improved entity generator features
- Better handling of nullable relations with filters

**MikroORM 6.6 Features:**
- Enhanced filter configurability on relations
- Strict filter option for nullable relations (useful for tenant filters)
- Improved entity generator
- Better nullable value handling in populated relations

**What This Means for Learning:**
- MikroORM 6.4 is very current and patterns are valid
- The upgrade to 6.6 is non-breaking for most use cases
- Focus on learning the fundamentals (entities, repositories, migrations)
- Tenant filtering patterns are better in 6.6 if needed

**Official Resources:**
- MikroORM 6.6 announcement: https://mikro-orm.io/blog/mikro-orm-6-6-released
- MikroORM 6.5 announcement: https://mikro-orm.io/blog/mikro-orm-6-5-released
- Documentation: https://mikro-orm.io/docs/
- Migration guides: https://mikro-orm.io/docs/upgrading

---

### PostgreSQL (pg driver) - v8.16.3

**Current Status (Nov 2025):**
- Project uses: v8.16.3
- Status: ✅ **Current**

**What This Means for Learning:**
- PostgreSQL patterns in this project are current
- The pg driver is the standard Node.js PostgreSQL client
- All modern PostgreSQL features are supported

**Official Resources:**
- pg documentation: https://node-postgres.com/
- PostgreSQL docs: https://www.postgresql.org/docs/

---

### Redis (ioredis) - v5.4.1

**Current Status (Nov 2025):**
- Project uses: v5.4.1
- Status: ✅ **Current**

**What This Means for Learning:**
- ioredis is the recommended Redis client for Node.js
- Patterns in this project are current
- Used for caching, sessions, and event bus in Medusa

**Official Resources:**
- ioredis documentation: https://github.com/redis/ioredis
- Redis docs: https://redis.io/docs/

---

## Build & Development Tools

### Turborepo (Turbo) - v1.6.3

**Current Status (Nov 2025):**
- Latest stable: v2.6.1 (released November 11, 2025)
- Project uses: v1.6.3
- Status: 🚨 **One major version behind**

**Important Updates Since Jan 2025:**
- Turborepo 2.0 released July 2024 (major milestone)
- Turborepo 2.6.0 released October 31, 2025
- Turborepo 2.6.1 released November 11, 2025 (current)

**Turborepo 2.x Features:**
- 🆕 **New Terminal UI**: Enhanced developer experience
- 🆕 **Watch Mode**: Automatic rebuilds on file changes
- 🆕 **Bun v1 lockfile support** (added in 2.6)
- 🆕 **Microfrontends proxy**: Native support for microfrontends
- 🆕 **Improved task search**: Press `/` to filter tasks
- ✅ **Performance**: Builds measured in microseconds for incremental changes
- ✅ **Rust-powered**: Completed migration from Go to Rust

**What This Means for Learning:**
- Turborepo 1.6 patterns still work but are outdated
- **Recommendation**: The project should upgrade to Turbo 2.x
- The core concepts (task caching, pipeline) remain similar
- Turborepo 2.x has significantly better DX and performance

**Official Resources:**
- Turborepo 2.0 announcement: https://turborepo.com/blog/turbo-2-0
- Turborepo 2.6 release: https://turborepo.com/blog/turbo-2-6
- Documentation: https://turbo.build/repo/docs
- Migration guide: https://turbo.build/repo/docs/upgrading

---

### tsup - v8.4.0

**Current Status (Nov 2025):**
- Project uses: v8.4.0
- Status: ✅ **Current**

**What This Means for Learning:**
- tsup is a modern TypeScript bundler built on esbuild
- The project uses current version and patterns
- Excellent for building TypeScript libraries

**Official Resources:**
- tsup documentation: https://tsup.egoist.dev/

---

### esbuild - v0.25.0

**Current Status (Nov 2025):**
- Project uses: v0.25.0
- Status: ✅ **Current**

**What This Means for Learning:**
- esbuild is an extremely fast JavaScript bundler
- Used by tsup and other build tools in this project
- Current version, no concerns

**Official Resources:**
- esbuild documentation: https://esbuild.github.io/

---

## Testing Frameworks

### Vitest - v3.0.5

**Current Status (Nov 2025):**
- Latest stable: v4.0.8 (released October 22, 2025)
- Project uses: v3.0.5
- Status: ⚠️ **One major version behind**

**Important Updates Since Jan 2025:**
- Vitest 3.0 released January 17, 2025
- Vitest 4.0 released October 22, 2025 (current stable)

**Vitest 4.0 Features:**
- ✅ **Browser Mode**: Removed experimental tag (now stable)
- 🆕 **Visual Regression Testing**: Added support in Browser Mode
- ✅ **Vite 7.0 Support**: Available from Vitest 3.2+
- ✅ **Performance improvements**

**What This Means for Learning:**
- Vitest 3 patterns are current and valid
- The project should consider upgrading to Vitest 4
- Core testing concepts remain the same
- Vitest 4 adds visual testing capabilities

**Official Resources:**
- Vitest 4.0 announcement: https://vitest.dev/blog/vitest-4
- Vitest 3.0 announcement: https://vitest.dev/blog/vitest-3
- Documentation: https://vitest.dev/guide/
- Migration guide: https://vitest.dev/guide/migration.html

---

### Jest - v29.7.0

**Current Status (Nov 2025):**
- Project uses: v29.7.0
- Status: ✅ **Current**

**What This Means for Learning:**
- Jest 29 is the current stable version
- Widely used for testing (alongside Vitest in this monorepo)
- All patterns are current

**Official Resources:**
- Jest documentation: https://jestjs.io/docs/getting-started

---

## UI & Component Libraries

### TailwindCSS - v3.4.3

**Current Status (Nov 2025):**
- Latest stable: v4.1.17 (released approximately November 7, 2025)
- Stable v4.0 released: January 22, 2025
- Project uses: v3.4.3
- Status: 🚨 **One major version behind**

**Important Updates Since Jan 2025:**
- TailwindCSS v4.0 stable released January 22, 2025
- TailwindCSS v4.1.17 is current (November 2025)
- Complete rewrite with major performance and DX improvements

**TailwindCSS v4.0 Major Changes:**
- ✅ **5x faster full builds**, 100x+ faster incremental builds (microseconds!)
- ✅ **Modern CSS**: Built on cascade layers, @property, color-mix()
- ✅ **Simplified setup**: Single line in CSS, zero config
- ✅ **Automatic content detection**: No manual content paths
- 🆕 **Built-in container queries**: No plugin needed
- ⚠️ **Browser requirements**: Safari 16.4+, Chrome 111+, Firefox 128+
- 🆕 **New configuration**: Uses CSS-based config instead of JavaScript

**What This Means for Learning:**
- TailwindCSS v3 patterns are **still widely used** in production
- The core utility-first approach is the same
- **Major paradigm shift**: v4 uses CSS config instead of JS
- Migration requires configuration rewrite
- Consider this a good time to learn modern v4 patterns

**Official Resources:**
- TailwindCSS v4.0 announcement: https://tailwindcss.com/blog/tailwindcss-v4
- Documentation: https://tailwindcss.com/docs/
- Upgrade guide: https://tailwindcss.com/docs/upgrade-guide
- Browser support: https://tailwindcss.com/docs/browser-support

---

### Storybook - v8.3.5

**Current Status (Nov 2025):**
- Latest in Storybook 8: v8.6.14 (released approximately 6 months ago)
- Latest overall: v10.0.7 (released 5 days ago)
- Project uses: v8.3.5
- Status: ⚠️ **Several minor versions behind in v8 line** / 🚨 **Two major versions behind overall**

**Important Updates Since Jan 2025:**
- Storybook 8.4 released October 2024
- Storybook 8.5 released January 2025
- Storybook 8.6.14 is the latest in the 8.x line
- Storybook 10.0 is now available (skipped v9)

**What This Means for Learning:**
- Storybook 8.3 patterns are valid
- The project could upgrade to 8.6 for bug fixes
- Storybook 10 represents a major leap forward
- Core concepts (stories, addons, decorators) remain consistent

**Official Resources:**
- Storybook 8.5 release: https://storybook.js.org/releases/8.5
- Storybook 10 release: https://storybook.js.org/releases/10.0
- Documentation: https://storybook.js.org/docs
- Migration guides: https://storybook.js.org/docs/releases/upgrading

---

## State & Form Management

### TanStack Query (React Query) - v5.64.2

**Current Status (Nov 2025):**
- Latest stable: v5.90.10 (released approximately November 16, 2025)
- Project uses: v5.64.2
- Status: ⚠️ **Several patch versions behind** (but v5.x is current major version)

**Important Updates Since Jan 2025:**
- Frequent patch releases throughout 2025
- v5.90.10 is the latest as of November 18, 2025
- Suspense for data fetching is stable in v5

**What This Means for Learning:**
- TanStack Query v5 is the current recommended version
- The patterns in this project are modern and current
- Upgrading to 5.90 is non-breaking (patch releases)
- Excellent library for server state management

**Official Resources:**
- TanStack Query docs: https://tanstack.com/query/latest
- Migration to v5: https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5
- GitHub: https://github.com/TanStack/query

---

### React Hook Form - v7.49.1

**Current Status (Nov 2025):**
- Latest stable: v7.66.1 (released November 17, 2025)
- Project uses: v7.49.1
- Status: ⚠️ **Several minor versions behind**

**Important Updates Since Jan 2025:**
- Frequent updates throughout 2025
- v7.66.1 is very recent (published 20 hours ago as of Nov 18)
- Still the most popular form library despite React 19's new form features

**What This Means for Learning:**
- React Hook Form v7 is current and widely adopted
- The patterns in this project are valid
- Simple upgrade to v7.66.1 (non-breaking)
- Excellent for form handling in React

**Note:** React 19 introduced built-in form actions, but React Hook Form still offers advantages for complex forms.

**Official Resources:**
- React Hook Form docs: https://react-hook-form.com/
- GitHub: https://github.com/react-hook-form/react-hook-form

---

### Zod - v3.25.76

**Current Status (Nov 2025):**
- Latest stable: v4.1.12 (released approximately October 2025)
- Zod 4.0.0 officially released: July 2025
- Project uses: v3.25.76
- Status: 🚨 **One major version behind**

**Important Updates Since Jan 2025:**
- Zod 4.0.0 released in July 2025
- Zod 4.1.12 is current as of November 2025
- Major performance and size improvements

**Zod 4.0 Major Improvements:**
- ✅ **57% smaller bundle size**
- ✅ **20× reduction in compiler instantiations** (faster type-checking, smoother IDE)
- 🆕 **Built-in JSON Schema conversion**
- ✅ **Better TypeScript performance**
- ⚠️ **Breaking changes** require migration

**What This Means for Learning:**
- Zod v3 patterns are still valid and widely used
- **Consider upgrading to Zod 4** for performance benefits
- Core API is similar but check migration guide
- Zod 4 is significantly better for large schemas

**Official Resources:**
- Zod 4 announcement: https://zod.dev/v4/versioning
- Documentation: https://zod.dev/
- Migration guide: Check GitHub releases
- What's new: https://peerlist.io/blog/engineering/zod-4-is-here-everything-you-need-to-know

---

## Monorepo Management

### Yarn - v3.2.1

**Current Status (Nov 2025):**
- Project uses: Yarn v3.2.1 (Berry)
- Status: ✅ **Current** (Yarn 3.x is modern)

**What This Means for Learning:**
- Yarn 3 (Berry) is the modern Yarn with Plug'n'Play
- Different from Yarn 1.x (Classic)
- Works well with monorepos
- Current and recommended

**Official Resources:**
- Yarn documentation: https://yarnpkg.com/

---

## Other Key Dependencies

### Awilix - v8.0.1

**Current Status (Nov 2025):**
- Latest stable: v12.0.5 (released approximately March 2025)
- Project uses: v8.0.1
- Status: ⚠️ **Four major versions behind**

**Important Updates:**
- Awilix 10+ introduced "strict mode" for better correctness checks
- Current version is v12.0.5
- 544 projects use Awilix in npm

**What This Means for Learning:**
- Awilix v8 dependency injection patterns are still valid
- **Recommendation**: Upgrade to v12 for latest features
- Core DI concepts remain the same
- Strict mode in v10+ helps catch bugs early

**Official Resources:**
- Awilix GitHub: https://github.com/jeffijoe/awilix
- Documentation: https://github.com/jeffijoe/awilix#readme

---

## Summary of Currency Status

### ✅ Current and Up-to-Date
- **TypeScript 5.6.2** - Very current (2 minor versions behind latest 5.9)
- **Node.js 20+** - Fully supported, modern
- **MikroORM 6.4.16** - Current (2 minor behind 6.6)
- **PostgreSQL (pg) 8.16.3** - Current
- **Redis (ioredis) 5.4.1** - Current
- **tsup 8.4.0** - Current
- **esbuild 0.25.0** - Current
- **Jest 29.7.0** - Current
- **Yarn 3.2.1** - Current

### ⚠️ Slightly Outdated (Minor Updates Available)
- **React 18.3.1** → React 19.2.0 (one major version, but 18 still widely used)
- **Express 4.21.0** → Express 5.0.0 (v4 still maintained, v5 is modernization)
- **TanStack Query 5.64.2** → 5.90.10 (same major version, patches available)
- **React Hook Form 7.49.1** → 7.66.1 (minor versions behind)
- **Vitest 3.0.5** → 4.0.8 (one major version behind)
- **Storybook 8.3.5** → 8.6.14 or 10.0.7 (minor updates in v8, or 2 major versions behind)

### 🚨 Significantly Behind (Upgrade Recommended)
- **Vite 5.4.21** → 7.2 (two major versions behind)
- **TailwindCSS 3.4.3** → 4.1.17 (one major version, paradigm shift)
- **Turborepo 1.6.3** → 2.6.1 (one major version, significant improvements)
- **Zod 3.25.76** → 4.1.12 (one major version, major performance improvements)
- **Awilix 8.0.1** → 12.0.5 (four major versions behind)

---

## Key Takeaways for New Developers

### What Patterns Are Safe to Learn From
1. **TypeScript patterns**: ✅ Completely current
2. **React patterns**: ✅ React 18 hooks and patterns are still standard
3. **MikroORM usage**: ✅ Current patterns, entities, repositories
4. **Express middleware & routing**: ✅ Valid patterns (Express 4/5 similar)
5. **Dependency Injection (Awilix)**: ✅ Core concepts valid
6. **Testing patterns**: ✅ Both Jest and Vitest patterns are current

### What to Research for Modern Alternatives
1. **Vite configuration**: Look into Vite 6/7 improvements
2. **TailwindCSS**: Consider learning v4 CSS-based config
3. **Turbo configuration**: Check Turbo 2.x features (watch mode, UI)
4. **React 19**: Learn Server Components and Actions for future projects
5. **Zod 4**: Better performance for schema validation

### Patterns That Are Legacy
- None of the patterns in this project are "legacy" or deprecated
- All technologies are actively maintained
- Some are a few versions behind but still production-ready
- The architectural patterns are modern and current

---

## Upgrade Priority Recommendations

### High Priority (Security & Performance)
1. **Vite** 5 → 7 (major performance gains, security updates)
2. **Turbo** 1 → 2 (better DX, performance)
3. **Zod** 3 → 4 (57% smaller bundle, 20x faster type-checking)

### Medium Priority (Features & Improvements)
1. **Vitest** 3 → 4 (stable browser mode, visual regression testing)
2. **React** 18 → 19 (Server Components, Actions - when ready)
3. **TailwindCSS** 3 → 4 (if team is ready for config migration)
4. **Awilix** 8 → 12 (strict mode, bug detection)

### Low Priority (Minor Updates)
1. **TypeScript** 5.6 → 5.9 (minor improvements)
2. **TanStack Query** 5.64 → 5.90 (patch releases)
3. **React Hook Form** 7.49 → 7.66 (minor updates)
4. **Express** 4 → 5 (when team is ready, mainly modernization)
5. **MikroORM** 6.4 → 6.6 (filter improvements)
6. **Storybook** 8.3 → 8.6 or 10.0 (when major upgrade is planned)

---

**Last Updated:** November 18, 2025
**Next Review Recommended:** March 2026 (or when major versions of key dependencies are released)
