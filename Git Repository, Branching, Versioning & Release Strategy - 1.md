# Git Repository, Branching, Versioning & Release Strategy

## 1. Git Architecture

A production project should have a clear relationship between:

```text
Repository
   │
   ├── Branches
   │     ├── main
   │     ├── develop
   │     ├── feature/*
   │     ├── release/*
   │     └── hotfix/*
   │
   ├── Tags
   │     ├── v1.0.0
   │     ├── v1.1.0
   │     └── v1.1.1
   │
   ├── Releases
   │     ├── Release 1.0
   │     ├── Release 1.1
   │     └── Release 2.0
   │
   └── CI/CD
         ├── Test
         ├── Build
         ├── Staging
         └── Production
```

The important point is:

```text
Branch ≠ Version ≠ Release
```

They have different purposes.

---

# 2. Branch vs Version vs Release

## Branch

A branch is a **development line**.

Example:

```text
feature/payment
```

It answers:

> Where am I developing this change?

---

## Version

A version identifies a specific state of the software.

Example:

```text
v1.4.2
```

It answers:

> Exactly which version of the software is this?

---

## Tag

A Git tag permanently points to a specific commit.

```text
v1.4.2
   ↓
commit abc123
```

It answers:

> Which exact commit represents version 1.4.2?

---

## Release

A release is the **deliverable package/version provided to users or production**.

Example:

```text
Release v1.4.2

Features:
- Payment retry
- Invoice export

Bug Fixes:
- Checkout issue
- Email issue
```

---

# 3. Recommended Branch Hierarchy

For a medium-sized production application:

```text
main
 │
 ├── release/*
 │
 ├── hotfix/*
 │
 └── develop
       │
       ├── feature/*
       ├── bugfix/*
       ├── refactor/*
       └── chore/*
```

Flow:

```text
feature
   ↓
develop
   ↓
release
   ↓
main
   ↓
production
```

Hotfix:

```text
main
 ↓
hotfix
 ↓
main
 ↓
production

and

hotfix
 ↓
develop
```

---

# 4. Main Branch

`main` represents production-ready code.

```text
main
 │
 ├── v1.0.0
 ├── v1.1.0
 ├── v1.2.0
 └── v1.2.1
```

Rules:

```text
❌ No direct development
❌ No direct random commits
❌ No unreviewed code
```

Recommended:

```text
Pull Request
    ↓
Code Review
    ↓
CI
    ↓
Merge
```

---

# 5. Develop Branch

`develop` contains the latest integrated development work.

```text
develop
   │
   ├── feature/login
   ├── feature/payment
   ├── feature/order
   └── bugfix/email
```

After features are tested:

```text
develop
    ↓
release/1.5.0
```

---

# 6. Feature Branch

Every feature gets its own branch.

Example:

```text
feature/payment-stripe
```

Naming:

```text
feature/<feature-name>
```

Examples:

```text
feature/user-registration
feature/social-login
feature/stripe-payment
feature/order-management
feature/notification-system
feature/ai-chat
```

Flow:

```text
develop
   │
   └── feature/stripe-payment
             │
             ├── commit
             ├── commit
             └── commit
                    │
                    ▼
                  PR
                    │
                    ▼
                 develop
```

---

# 7. Bugfix Branch

For normal bugs:

```text
bugfix/payment-validation
```

Flow:

```text
develop
   ↓
bugfix/payment-validation
   ↓
PR
   ↓
develop
```

Examples:

```text
bugfix/login-error
bugfix/invoice-total
bugfix/email-template
```

---

# 8. Hotfix Branch

A hotfix is for a critical production problem.

Example:

```text
Production
    │
    └── v1.5.2
            │
            ▼
      Critical Bug
            │
            ▼
    hotfix/payment-error
```

Flow:

```text
main
  ↓
hotfix/payment-error
  ↓
Fix
  ↓
Test
  ↓
main
  ↓
v1.5.3
  ↓
Production
```

Also merge the fix back into:

```text
develop
```

so the problem does not return in the next release.

---

# 9. Release Branch

A release branch freezes new features and prepares the application for production.

Example:

```text
release/1.5.0
```

Flow:

```text
develop
   ↓
release/1.5.0
   │
   ├── Testing
   ├── Bug fixes
   ├── Documentation
   ├── Version update
   └── Final QA
          │
          ▼
        main
```

During release preparation:

```text
❌ New features
❌ Large refactoring

Allowed:

✅ Bug fixes
✅ Documentation
✅ Configuration fixes
✅ Release preparation
```

---

# 10. Complete Git Flow

Example project:

```text
                    main
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      v1.0.0        v1.1.0        v1.2.0
        │             │             │
        │             │             │
      Production    Production    Production
                      ▲
                      │
                release/1.2.0
                      ▲
                      │
                   develop
                ┌─────┼─────┐
                │     │     │
             feature feature bugfix
             /login  /pay   /email
```

---

# 11. Simple Git Strategy

Not every project needs Git Flow.

For a small SaaS:

```text
main
 │
 ├── feature/*
 ├── bugfix/*
 └── hotfix/*
```

Flow:

```text
feature
   ↓
Pull Request
   ↓
CI
   ↓
main
   ↓
Tag
   ↓
Production
```

This is often easier to maintain.

---

# 12. Git Strategy Selection

```text
How large is the project?
        │
        ├── Small
        │     ↓
        │   main + feature/*
        │
        ├── Medium
        │     ↓
        │   main + develop + feature/*
        │
        ├── Large
        │     ↓
        │   main + develop
        │   + release/*
        │   + hotfix/*
        │
        └── Enterprise
              ↓
        Branch strategy based on
        multiple teams/environments
```

Do not create branches just because a branch model looks professional.

The goal is:

```text
Less complexity
+
Clear ownership
+
Safe releases
+
Easy rollback
```

---

# 13. Semantic Versioning

Use:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
2.4.7
```

Meaning:

```text
2 = MAJOR
4 = MINOR
7 = PATCH
```

---

# 14. MAJOR Version

Increase MAJOR when you introduce breaking changes.

```text
1.9.0
 ↓
2.0.0
```

Examples:

```text
API changed
Database contract changed
Old authentication removed
Old API endpoints removed
Major architecture change
```

Example:

```text
GET /api/v1/users
```

becomes:

```text
GET /api/v2/users
```

---

# 15. MINOR Version

Increase MINOR for new backward-compatible functionality.

```text
1.4.0
 ↓
1.5.0
```

Examples:

```text
New payment method
New reporting module
New notification feature
New dashboard
```

Existing functionality should continue working.

---

# 16. PATCH Version

Increase PATCH for bug fixes.

```text
1.5.0
 ↓
1.5.1
```

Examples:

```text
Login bug
Email bug
Calculation bug
Security patch
Small performance fix
```

---

# 17. Version Examples

```text
v1.0.0
Initial Production

v1.1.0
New Payment Module

v1.1.1
Payment Bug Fix

v1.2.0
New Reporting

v2.0.0
Breaking API Changes
```

---

# 18. Git Tags

Create tags for production releases.

```bash
git tag v1.0.0
git push origin v1.0.0
```

Annotated tag:

```bash
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```

View tags:

```bash
git tag
```

Checkout:

```bash
git checkout v1.2.0
```

---

# 19. Why Tags Are Important

Suppose production is currently:

```text
v1.5.2
```

You deploy:

```text
v1.6.0
```

and discover a serious issue.

You can identify the exact previous production state:

```text
v1.5.2
```

This makes rollback much easier.

---

# 20. Release Structure

Example:

```text
Release v1.5.0
│
├── Features
│   ├── Stripe Payment
│   └── Invoice Export
│
├── Improvements
│   └── Faster Dashboard
│
├── Bug Fixes
│   ├── Login issue
│   └── Email issue
│
├── Database
│   └── Added invoices table
│
└── Deployment
    ├── Migration
    ├── Cache clear
    └── Queue restart
```

---

# 21. Repository Directory Structure

The Git repository should also have a clean application structure.

For Laravel:

```text
my-saas/
│
├── app/
│   ├── Console/
│   ├── Exceptions/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   └── Requests/
│   │
│   ├── Models/
│   ├── Services/
│   ├── Repositories/
│   ├── Actions/
│   ├── Jobs/
│   ├── Events/
│   ├── Listeners/
│   └── Notifications/
│
├── bootstrap/
│
├── config/
│
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
│
├── resources/
│   ├── views/
│   ├── css/
│   └── js/
│
├── routes/
│   ├── web.php
│   ├── api.php
│   └── console.php
│
├── storage/
│
├── tests/
│   ├── Unit/
│   └── Feature/
│
├── docker/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── staging.yml
│       └── production.yml
│
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── deployment/
│   └── database/
│
├── .env.example
├── .gitignore
├── composer.json
├── docker-compose.yml
├── Dockerfile
├── README.md
└── CHANGELOG.md
```

---

# 22. Important Repository Rule

Do NOT commit:

```text
.env
/vendor
/node_modules
/storage/logs
/storage/framework/cache
/private keys
production backups
password files
```

Use:

```text
.gitignore
```

Example:

```text
.env
/vendor/
/node_modules/
/storage/logs/*
/storage/framework/cache/*
/storage/framework/sessions/*
/storage/framework/views/*
```

---

# 23. Modular Application Structure

For a large Laravel application, you can organize business domains separately.

Example:

```text
app/
│
├── Modules/
│   │
│   ├── User/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Requests/
│   │   ├── Jobs/
│   │   └── Routes/
│   │
│   ├── Payment/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Jobs/
│   │   └── Routes/
│   │
│   ├── Order/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   └── Routes/
│   │
│   └── Notification/
│       ├── Services/
│       ├── Jobs/
│       └── Channels/
```

This is useful when the application becomes large.

---

# 24. Git + Modular Architecture

The relationship can be:

```text
Repository
│
├── Modules
│   ├── User
│   ├── Payment
│   ├── Order
│   └── Notification
│
├── Infrastructure
│
├── Tests
│
└── Deployment
```

Feature branches can then map to modules:

```text
feature/payment-stripe
feature/order-refund
feature/user-2fa
feature/notification-sms
```

---

# 25. Commit Strategy

Commits should describe one logical change.

Good:

```text
feat: add stripe payment integration
fix: resolve invoice total calculation
refactor: simplify order service
docs: update deployment guide
test: add payment feature tests
chore: update laravel dependencies
```

Bad:

```text
update
changes
final
final2
test
new changes
```

---

# 26. Conventional Commit Format

Recommended:

```text
type(scope): description
```

Examples:

```text
feat(payment): add stripe checkout

fix(order): resolve duplicate order creation

refactor(user): simplify authentication service

test(payment): add refund test cases

docs(api): update payment documentation

chore(deps): update laravel dependencies
```

Common types:

```text
feat
fix
refactor
test
docs
chore
perf
build
ci
```

---

# 27. Commit Hierarchy

Example:

```text
feature/payment
│
├── feat(payment): add payment model
├── feat(payment): add stripe service
├── test(payment): add checkout tests
├── feat(payment): add webhook handler
└── docs(payment): update API documentation
```

After PR review, commits can be squashed if your team prefers a clean history.

---

# 28. Pull Request Structure

A PR should contain:

```text
Title:
Add Stripe Payment Integration

Description:

Problem:
Customers cannot pay using Stripe.

Changes:
- Added Stripe service
- Added payment model
- Added webhook
- Added payment tests

Database:
- payments table

Testing:
- Unit tests
- Feature tests

Deployment:
- Run migration
- Configure Stripe keys

Risk:
Medium
```

---

# 29. Pull Request Flow

```text
Developer
   ↓
Create Branch
   ↓
Code
   ↓
Commit
   ↓
Push
   ↓
Pull Request
   ↓
CI
   ↓
Code Review
   ↓
Approval
   ↓
Merge
```

Recommended branch protection:

```text
main
 │
 ├── Require PR
 ├── Require CI
 ├── Require approval
 ├── Prevent force push
 └── Prevent direct push
```

---

# 30. Complete Feature Example

Suppose we need:

```text
Stripe Payment
```

Create:

```bash
git checkout develop

git pull origin develop

git checkout -b feature/stripe-payment
```

Develop:

```text
feature/stripe-payment
```

Commits:

```text
feat(payment): add payment model
feat(payment): add stripe service
feat(payment): add checkout API
test(payment): add checkout tests
```

Push:

```bash
git push origin feature/stripe-payment
```

Then:

```text
Pull Request
      ↓
develop
```

---

# 31. Release Example

Suppose the current production version is:

```text
v1.4.2
```

New features:

```text
Stripe
Reports
Notifications
```

Create:

```bash
git checkout develop
git checkout -b release/1.5.0
```

Test:

```text
release/1.5.0
       │
       ├── QA
       ├── Bug fixes
       ├── Security
       └── Performance
```

Then merge:

```text
release/1.5.0
       ↓
main
```

Tag:

```bash
git tag -a v1.5.0 -m "Release v1.5.0"
git push origin v1.5.0
```

Deploy:

```text
v1.5.0
   ↓
Production
```

Then merge release changes back:

```text
release/1.5.0
       ↓
develop
```

---

# 32. Hotfix Example

Production:

```text
v1.5.0
```

Critical payment bug discovered.

Create:

```bash
git checkout main

git checkout -b hotfix/payment-error
```

Fix:

```text
fix(payment): resolve failed transaction handling
```

Merge:

```text
hotfix/payment-error
        ↓
       main
```

Tag:

```bash
git tag -a v1.5.1 -m "Hotfix payment error"
git push origin v1.5.1
```

Deploy:

```text
v1.5.1
   ↓
Production
```

Then merge into:

```text
develop
```

---

# 33. Complete Version Timeline

Example:

```text
v1.0.0
 │
 ├── v1.0.1
 ├── v1.0.2
 │
 └── v1.1.0
       │
       ├── v1.1.1
       └── v1.2.0
             │
             ├── v1.2.1
             ├── v1.2.2
             │
             └── v2.0.0
```

Meaning:

```text
v1.0.0 → Initial release

v1.0.1 → Bug fix

v1.1.0 → New feature

v1.1.1 → Bug fix

v2.0.0 → Breaking change
```

---

# 34. Git Repository + CI/CD

Complete architecture:

```text
                    Git Repository
                          │
              ┌───────────┴───────────┐
              │                       │
           develop                   main
              │                       │
        feature branches          Production
              │                       │
              ▼                       ▼
             CI                    Release
              │                       │
              ▼                       ▼
           Testing                  Tag
              │                       │
              ▼                       ▼
          Staging                 v1.5.0
                                      │
                                      ▼
                                  Deployment
                                      │
                                      ▼
                                  Production
```

---

# 35. Recommended Repository Structure

For a professional SaaS:

```text
project/
│
├── app/
├── bootstrap/
├── config/
├── database/
├── resources/
├── routes/
├── tests/
│
├── docs/
│   ├── architecture/
│   ├── database/
│   ├── api/
│   ├── deployment/
│   └── security/
│
├── docker/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── staging.yml
│       └── production.yml
│
├── .env.example
├── .gitignore
├── composer.json
├── Dockerfile
├── docker-compose.yml
├── CHANGELOG.md
└── README.md
```

---

# 36. Documentation Hierarchy

Keep technical decisions inside the repository:

```text
docs/
│
├── architecture/
│   ├── system-design.md
│   ├── application-architecture.md
│   ├── database-architecture.md
│   └── integration-architecture.md
│
├── api/
│   ├── authentication.md
│   ├── users.md
│   └── payments.md
│
├── database/
│   ├── schema.md
│   └── migrations.md
│
├── deployment/
│   ├── server.md
│   ├── ci-cd.md
│   └── rollback.md
│
└── security/
    ├── authentication.md
    ├── permissions.md
    └── secrets.md
```

---

# 37. Monorepo vs Multiple Repositories

## Monorepo

Everything in one repository:

```text
company-platform/
│
├── backend/
├── frontend/
├── mobile/
├── admin/
└── infrastructure/
```

Useful when:

```text
Same team
Shared code
Same release cycle
Strong integration
```

---

## Multiple Repositories

Separate repositories:

```text
company-backend
company-frontend
company-mobile
company-admin
company-infrastructure
```

Useful when:

```text
Different teams
Different release cycles
Different deployment lifecycle
Independent ownership
```

---

# 38. Repository Decision Tree

```text
Is everything tightly connected?
        │
        ├── YES
        │    ↓
        │  Consider Monorepo
        │
        └── NO
             ↓
        Multiple Repositories
```

For a small team:

```text
Start simple.
```

Do not create 10 repositories for a small application.

---

# 39. Git Security

Protect:

```text
main
production
release
```

Use:

```text
Branch Protection
Pull Requests
Code Review
CI Checks
MFA
SSH Keys
Signed Commits (where appropriate)
Secret Scanning
Dependency Scanning
```

Never store:

```text
.env
Passwords
API Secrets
Private Keys
Database Dumps
Customer Data
Production Credentials
```

---

# 40. Git Backup Strategy

Git is not your complete backup strategy.

Keep:

```text
Git Repository
+
Database Backup
+
Object Storage Backup
+
Infrastructure Configuration
```

Example:

```text
GitHub
   │
   ├── Source Code
   ├── Tags
   └── Releases

Backup Storage
   │
   ├── Database
   ├── User Files
   └── Application Backups
```

---

# 41. Recommended Production Git Model

For a medium Laravel SaaS:

```text
                    main
                      │
             Production Code
                      │
              ┌───────┴───────┐
              │               │
          release/*        hotfix/*
              │               │
              ▼               ▼
           Staging         Emergency Fix
              ▲               │
              │               │
              └───────┬───────┘
                      │
                   develop
                      │
          ┌───────────┼───────────┐
          │           │           │
       feature      feature     bugfix
       /payment     /users      /email
```

---

# 42. Final Git Decision Framework

Before designing your Git strategy, ask:

```text
1. How many developers?

2. How many teams?

3. How many environments?

4. How often do we release?

5. Do we need staging?

6. Do we need zero downtime?

7. Do we support multiple application versions?

8. Do we have backward compatibility requirements?

9. Do we need hotfix releases?

10. Is the project monolithic or modular?
```

Then choose the simplest model that satisfies those requirements.

---

# 43. Recommended Default

For a typical Laravel SaaS with a small/medium team:

```text
Repository
│
├── main
│
├── develop
│
├── feature/*
├── bugfix/*
└── hotfix/*
```

Deployment:

```text
feature
   ↓
Pull Request
   ↓
CI
   ↓
develop
   ↓
Staging
   ↓
release/1.x.x
   ↓
main
   ↓
Git Tag
   ↓
Production
```

Versioning:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v1.0.0
v1.1.0
v1.1.1
v2.0.0
```

Documentation:

```text
README.md
docs/
CHANGELOG.md
```

Security:

```text
Branch Protection
+
PR Review
+
CI
+
Secrets Management
```

---

# 44. Golden Rule

The complete software lifecycle should look like:

```text
IDEA
  ↓
ISSUE / TASK
  ↓
FEATURE BRANCH
  ↓
CODE
  ↓
COMMIT
  ↓
PULL REQUEST
  ↓
CODE REVIEW
  ↓
CI
  ↓
DEVELOP
  ↓
STAGING
  ↓
QA
  ↓
RELEASE
  ↓
TAG
  ↓
PRODUCTION
  ↓
MONITORING
  ↓
ROLLBACK (if required)
```

And the architecture should remain:

```text
Simple
   ↓
Organized
   ↓
Automated
   ↓
Tested
   ↓
Versioned
   ↓
Releasable
   ↓
Recoverable
```

**The goal of Git is not to create many branches. The goal is to make code changes traceable, reviewable, versioned, releasable, and recoverable.**
