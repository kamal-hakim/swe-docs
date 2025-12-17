# Appendices

**Document**: #15 - Appendices  
**Version**: 1.0.0  
**Last Updated**: December 2024

---

## Overview

This document contains supporting materials, references, definitions, and additional resources for the TaskFlow Pro Product Requirements Document.

---

## Table of Contents

1. [Glossary](#glossary)
2. [Acronyms & Abbreviations](#acronyms--abbreviations)
3. [Technical References](#technical-references)
4. [Design Resources](#design-resources)
5. [API Endpoint Summary](#api-endpoint-summary)
6. [Database Schema Summary](#database-schema-summary)
7. [User Interview Summaries](#user-interview-summaries)
8. [Competitive Analysis Details](#competitive-analysis-details)
9. [Legal & Compliance](#legal--compliance)
10. [Version History](#version-history)
11. [Contributors](#contributors)

---

## 1. Glossary

### General Terms

**Acceptance Criteria**  
Specific conditions that must be met for a user story to be considered complete. Written in Given-When-Then format.

**Activation**  
The process of a new user completing key onboarding steps and deriving initial value from the product.

**Agile**  
An iterative software development methodology emphasizing flexibility, collaboration, and rapid delivery.

**API (Application Programming Interface)**  
A set of protocols and tools for building software applications, defining how components should interact.

**Architecture Blueprint**  
A comprehensive guide defining technical patterns, standards, and best practices for building applications.

**Backlog**  
A prioritized list of features, enhancements, and bug fixes waiting to be implemented.

**Burndown Chart**  
A graphical representation of work remaining versus time in a sprint or project.

**Churn Rate**  
The percentage of customers who stop using the product within a given time period.

**CRUD**  
Create, Read, Update, Delete - the four basic operations for persistent storage.

**Dark Mode**  
An alternative color scheme using dark backgrounds and light text, reducing eye strain in low-light environments.

**Definition of Done (DoD)**  
A checklist of criteria that must be met for work to be considered complete.

**Epic**  
A large body of work that can be broken down into smaller user stories.

**Feature Flag**  
A technique to enable/disable features without deploying code, useful for gradual rollouts and A/B testing.

**Kanban**  
A visual workflow management method using cards and columns to represent work items and their status.

**KPI (Key Performance Indicator)**  
A measurable value demonstrating how effectively a company achieves key business objectives.

**MVP (Minimum Viable Product)**  
The version of a product with just enough features to satisfy early customers and provide feedback.

**North Star Metric**  
The single metric that best captures the core value delivered to customers.

**OKR (Objectives and Key Results)**  
A goal-setting framework used to define and track objectives and their outcomes.

**Product-Market Fit (PMF)**  
The degree to which a product satisfies strong market demand.

**Progressive Disclosure**  
A UX pattern that sequences information and actions to reduce complexity and cognitive load.

**Real-time Collaboration**  
The ability for multiple users to work on the same content simultaneously with instant updates.

**Retrospective**  
A meeting held at the end of a sprint to reflect on what went well and what can be improved.

**SaaS (Software as a Service)**  
A software distribution model where applications are hosted by a vendor and made available over the internet.

**Scrum**  
An agile framework for managing complex projects, emphasizing iterative progress through sprints.

**Sprint**  
A time-boxed iteration (typically 2 weeks) in which specific work must be completed.

**SSR (Server-Side Rendering)**  
Rendering web pages on the server rather than in the browser, improving initial load time and SEO.

**Stakeholder**  
Any person or group with an interest in the project's outcome.

**Story Points**  
A unit of measurement for expressing the overall effort required to implement a user story.

**Technical Debt**  
The implied cost of additional rework caused by choosing an easy solution now instead of a better approach.

**User Persona**  
A semi-fictional representation of an ideal customer based on research and data.

**Velocity**  
The amount of work a team completes during a sprint, measured in story points.

**Vertical Slice**  
A cross-section of functionality that touches all layers of the application (UI, logic, data).

**Wireframe**  
A visual guide representing the skeletal framework of a webpage or application.

---

### TaskFlow Pro Specific Terms

**Board View**  
A kanban-style visualization of tasks organized by status columns.

**Project Template**  
A pre-configured project structure with tasks, statuses, and settings for common use cases.

**Subtask**  
A smaller task that is part of a larger parent task.

**Task Card**  
The visual representation of a task on the kanban board.

**Task Dependency**  
A relationship where one task must be completed before another can begin.

**Team Workspace**  
A shared environment where team members collaborate on projects.

**Time Entry**  
A record of time spent working on a task.

**Workspace Admin**  
A user with elevated permissions to manage workspace settings and users.

---

## 2. Acronyms & Abbreviations

### Technical

| Acronym | Full Form | Description |
|---------|-----------|-------------|
| **ADR** | Architecture Decision Record | Documentation of important architectural decisions |
| **API** | Application Programming Interface | Interface for software interaction |
| **ARR** | Annual Recurring Revenue | Yearly subscription revenue |
| **AWS** | Amazon Web Services | Cloud computing platform |
| **CDN** | Content Delivery Network | Distributed network for serving static assets |
| **CI/CD** | Continuous Integration/Continuous Deployment | Automated build and deployment pipeline |
| **CORS** | Cross-Origin Resource Sharing | Security mechanism for cross-domain requests |
| **CSAT** | Customer Satisfaction Score | Metric measuring customer satisfaction |
| **CSRF** | Cross-Site Request Forgery | Security vulnerability |
| **CSP** | Content Security Policy | Security standard to prevent XSS attacks |
| **CSS** | Cascading Style Sheets | Stylesheet language |
| **DAU** | Daily Active Users | Users active in a day |
| **DNS** | Domain Name System | Internet naming system |
| **DPA** | Data Processing Agreement | GDPR compliance contract |
| **E2E** | End-to-End | Testing from user perspective |
| **GDPR** | General Data Protection Regulation | EU privacy law |
| **GraphQL** | Graph Query Language | API query language |
| **HTML** | HyperText Markup Language | Web page structure |
| **HTTP** | HyperText Transfer Protocol | Web communication protocol |
| **HTTPS** | HTTP Secure | Encrypted HTTP |
| **IDE** | Integrated Development Environment | Software development tool |
| **JWT** | JSON Web Token | Authentication token format |
| **KPI** | Key Performance Indicator | Performance metric |
| **LCP** | Largest Contentful Paint | Core Web Vital metric |
| **LTV** | Lifetime Value | Revenue from customer over lifetime |
| **MAU** | Monthly Active Users | Users active in a month |
| **MFA** | Multi-Factor Authentication | Enhanced security authentication |
| **MRR** | Monthly Recurring Revenue | Monthly subscription revenue |
| **MVP** | Minimum Viable Product | Initial product version |
| **NPS** | Net Promoter Score | Customer loyalty metric |
| **OAuth** | Open Authorization | Authentication standard |
| **OKR** | Objectives and Key Results | Goal-setting framework |
| **ORM** | Object-Relational Mapping | Database abstraction |
| **P2P** | Peer-to-Peer | Direct communication |
| **PMF** | Product-Market Fit | Product-demand alignment |
| **PR** | Pull Request | Code review request |
| **PRD** | Product Requirements Document | Product specification |
| **PWA** | Progressive Web App | Web app with native features |
| **QA** | Quality Assurance | Testing process |
| **RBAC** | Role-Based Access Control | Permission system |
| **REST** | Representational State Transfer | API architecture |
| **RTO** | Recovery Time Objective | Acceptable downtime |
| **RPO** | Recovery Point Objective | Acceptable data loss |
| **SaaS** | Software as a Service | Cloud software model |
| **SAML** | Security Assertion Markup Language | SSO standard |
| **SDK** | Software Development Kit | Development toolkit |
| **SEO** | Search Engine Optimization | Discoverability improvement |
| **SLA** | Service Level Agreement | Service commitment |
| **SPA** | Single Page Application | Web app architecture |
| **SQL** | Structured Query Language | Database language |
| **SSO** | Single Sign-On | Unified authentication |
| **SSR** | Server-Side Rendering | Server rendering |
| **TDD** | Test-Driven Development | Testing methodology |
| **TLS** | Transport Layer Security | Encryption protocol |
| **TTFB** | Time To First Byte | Performance metric |
| **UI** | User Interface | Visual interface |
| **UX** | User Experience | User interaction design |
| **WAU** | Weekly Active Users | Users active in a week |
| **WCAG** | Web Content Accessibility Guidelines | Accessibility standards |
| **WebSocket** | Web Socket Protocol | Real-time communication |
| **XSS** | Cross-Site Scripting | Security vulnerability |

---

## 3. Technical References

### Framework Documentation

**Nuxt 4**
- Official Docs: https://nuxt.com/docs
- GitHub: https://github.com/nuxt/nuxt
- Release Notes: https://nuxt.com/releases

**Vuetify 3**
- Official Docs: https://vuetifyjs.com/
- Component API: https://vuetifyjs.com/en/components/
- Material Design 3: https://m3.material.io/

**Vue 3**
- Official Docs: https://vuejs.org/
- Composition API: https://vuejs.org/guide/extras/composition-api-faq.html
- TypeScript: https://vuejs.org/guide/typescript/overview.html

### State Management

**Pinia**
- Docs: https://pinia.vuejs.org/
- Getting Started: https://pinia.vuejs.org/getting-started.html
- Best Practices: https://pinia.vuejs.org/cookbook/

### Testing

**Vitest**
- Docs: https://vitest.dev/
- API Reference: https://vitest.dev/api/

**Playwright**
- Docs: https://playwright.dev/
- Best Practices: https://playwright.dev/docs/best-practices

**@nuxt/test-utils**
- Docs: https://nuxt.com/docs/getting-started/testing

### Validation

**VeeValidate**
- Docs: https://vee-validate.logaretm.com/v4/
- Yup Integration: https://vee-validate.logaretm.com/v4/guide/validation#validating-with-yup

**Zod**
- Docs: https://zod.dev/
- GitHub: https://github.com/colinhacks/zod

### Backend & API

**Prisma ORM**
- Docs: https://www.prisma.io/docs
- Schema Reference: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference

**tRPC**
- Docs: https://trpc.io/docs
- Nuxt Integration: https://trpc-nuxt.vercel.app/

### Internationalization

**Nuxt I18n**
- Docs: https://i18n.nuxtjs.org/
- Best Practices: https://i18n.nuxtjs.org/guide/

---

## 4. Design Resources

### Material Design 3

**Guidelines**
- Material 3 Design: https://m3.material.io/
- Color System: https://m3.material.io/styles/color/overview
- Typography: https://m3.material.io/styles/typography/overview
- Motion: https://m3.material.io/styles/motion/overview

**Tools**
- Material Theme Builder: https://m3.material.io/theme-builder
- Icon Library: https://fonts.google.com/icons
- Color Palette Generator: https://material-foundation.github.io/material-theme-builder/

### Design Inspiration

**Project Management Tools**
- Linear: https://linear.app/
- Asana: https://asana.com/
- Monday: https://monday.com/
- ClickUp: https://clickup.com/
- Notion: https://notion.so/

### Accessibility

**WCAG 2.1 Guidelines**
- Overview: https://www.w3.org/WAI/WCAG21/quickref/
- Level AA Requirements: https://www.w3.org/WAI/WCAG21/quickref/?currentsidebar=%23col_overview&levels=aaa

**Testing Tools**
- WAVE: https://wave.webaim.org/
- Axe DevTools: https://www.deque.com/axe/devtools/
- Lighthouse: https://developers.google.com/web/tools/lighthouse

---

## 5. API Endpoint Summary

### Authentication

```
POST   /api/v1/auth/register          - User registration
POST   /api/v1/auth/login             - User login
POST   /api/v1/auth/logout            - User logout
POST   /api/v1/auth/refresh           - Refresh access token
POST   /api/v1/auth/verify-email      - Verify email address
POST   /api/v1/auth/forgot-password   - Request password reset
POST   /api/v1/auth/reset-password    - Reset password
GET    /api/v1/auth/oauth/google      - Google OAuth login
GET    /api/v1/auth/oauth/github      - GitHub OAuth login
```

### Users

```
GET    /api/v1/users/me               - Get current user
PATCH  /api/v1/users/me               - Update current user
DELETE /api/v1/users/me               - Delete account
GET    /api/v1/users/:id              - Get user by ID
GET    /api/v1/users                  - List users (admin)
```

### Projects

```
GET    /api/v1/projects               - List projects
POST   /api/v1/projects               - Create project
GET    /api/v1/projects/:id           - Get project
PATCH  /api/v1/projects/:id           - Update project
DELETE /api/v1/projects/:id           - Delete project
POST   /api/v1/projects/:id/archive   - Archive project
GET    /api/v1/projects/:id/members   - List project members
POST   /api/v1/projects/:id/members   - Add project member
DELETE /api/v1/projects/:id/members/:userId - Remove member
```

### Tasks

```
GET    /api/v1/tasks                  - List tasks (with filters)
POST   /api/v1/tasks                  - Create task
GET    /api/v1/tasks/:id              - Get task
PATCH  /api/v1/tasks/:id              - Update task
DELETE /api/v1/tasks/:id              - Delete task
POST   /api/v1/tasks/:id/comments     - Add comment
GET    /api/v1/tasks/:id/comments     - List comments
POST   /api/v1/tasks/:id/attachments  - Upload attachment
GET    /api/v1/tasks/:id/attachments  - List attachments
POST   /api/v1/tasks/:id/subtasks     - Create subtask
GET    /api/v1/tasks/:id/time-entries - List time entries
POST   /api/v1/tasks/:id/time-entries - Add time entry
```

### Teams

```
GET    /api/v1/teams                  - List teams
POST   /api/v1/teams                  - Create team
GET    /api/v1/teams/:id              - Get team
PATCH  /api/v1/teams/:id              - Update team
DELETE /api/v1/teams/:id              - Delete team
GET    /api/v1/teams/:id/members      - List team members
POST   /api/v1/teams/:id/members      - Add team member
```

### Notifications

```
GET    /api/v1/notifications          - List notifications
PATCH  /api/v1/notifications/:id/read - Mark as read
POST   /api/v1/notifications/read-all - Mark all as read
DELETE /api/v1/notifications/:id      - Delete notification
```

See [07-api-specifications.md](./07-api-specifications.md) for complete API documentation.

---

## 6. Database Schema Summary

### Core Tables

```sql
-- Users
users (id, email, password_hash, name, avatar_url, created_at, updated_at)

-- Projects
projects (id, name, description, owner_id, status, created_at, updated_at)

-- Tasks
tasks (id, title, description, project_id, status, priority, assignee_id, 
       due_date, created_by, created_at, updated_at)

-- Comments
comments (id, task_id, user_id, content, created_at, updated_at)

-- Teams
teams (id, name, description, workspace_id, created_at, updated_at)

-- Project Members
project_members (project_id, user_id, role, joined_at)

-- Team Members
team_members (team_id, user_id, role, joined_at)

-- Attachments
attachments (id, task_id, filename, file_url, file_size, uploaded_by, 
             uploaded_at)

-- Time Entries
time_entries (id, task_id, user_id, duration, description, date)

-- Notifications
notifications (id, user_id, type, title, content, read, created_at)
```

See [08-data-models-schema.md](./08-data-models-schema.md) for complete schema with relationships.

---

## 7. User Interview Summaries

### Research Methodology

**Participants**: 25 project managers and team leads  
**Interview Duration**: 30-45 minutes  
**Format**: Semi-structured interviews  
**Date**: November 2024

### Key Findings

#### Pain Points with Current Tools

1. **Complexity** (18/25 participants)
   - "Too many features we never use"
   - "Takes weeks to onboard new team members"
   - "Configuration nightmare"

2. **Performance** (15/25 participants)
   - "Slow to load with large projects"
   - "Laggy when multiple people editing"
   - "Mobile app crashes frequently"

3. **Pricing** (14/25 participants)
   - "Too expensive for small teams"
   - "Forced to pay for features we don't need"
   - "Hidden costs add up quickly"

4. **Collaboration** (12/25 participants)
   - "Comments get buried and lost"
   - "No good way to @mention people"
   - "Hard to see who's working on what"

#### Desired Features

1. **Better UX** (23/25)
   - "Just want something that's easy to use"
   - "Clean, modern interface"
   - "Keyboard shortcuts for power users"

2. **Real-time Updates** (19/25)
   - "See changes immediately"
   - "Know when someone is working on same task"
   - "Live notifications"

3. **Flexible Workflows** (17/25)
   - "Customize statuses for our process"
   - "Different views: kanban, list, calendar"
   - "Automation for repetitive tasks"

4. **Good Mobile Experience** (16/25)
   - "Need to check tasks on phone"
   - "Update status while on the go"
   - "Quick task creation"

### Representative Quotes

> "We use Asana but it's way too much for our 5-person team. We just need projects, tasks, and comments."  
> — Sarah, Startup Founder

> "Linear is beautiful but it's designed for engineering teams. We need something for all departments."  
> — Mike, Operations Manager

> "Monday.com has everything but finding anything is impossible. Too cluttered."  
> — Jessica, Marketing Director

> "I just want something that works and doesn't cost $25/user/month."  
> — David, Small Business Owner

---

## 8. Competitive Analysis Details

### Feature Comparison Matrix

| Feature | TaskFlow Pro | Asana | Monday | Linear | ClickUp |
|---------|-------------|-------|--------|--------|---------|
| **Pricing** | $12/user | $11-24/user | $8-16/user | $8/user | $5-12/user |
| **Kanban Board** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Time Tracking** | ✅ | ❌ (paid add-on) | ✅ | ✅ | ✅ |
| **Dark Mode** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Real-time Collab** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Mobile App** | Phase 2 | ✅ | ✅ | ✅ | ✅ |
| **API Access** | ✅ | ✅ (higher tiers) | ✅ (higher tiers) | ✅ | ✅ |
| **Custom Fields** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Automation** | Phase 3 | ✅ | ✅ | ✅ | ✅ |
| **Reports** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **SSO/SAML** | Phase 4 | ✅ (Enterprise) | ✅ (Enterprise) | ✅ (Enterprise) | ✅ (Enterprise) |

### Differentiation Strategy

**TaskFlow Pro Advantages**:
1. ✅ **Modern Tech Stack** - Nuxt 4 + Vuetify 3 (fastest, most modern)
2. ✅ **Developer-Friendly** - GraphQL API, comprehensive docs, webhooks
3. ✅ **Best UX** - Material Design 3, consistent, intuitive
4. ✅ **Transparent Pricing** - Simple, predictable pricing
5. ✅ **Open Ecosystem** - Easy integrations, extensibility

See [15-competitive-analysis.md](./15-competitive-analysis.md) for full analysis.

---

## 9. Legal & Compliance

### Privacy Policy

**Last Updated**: December 2024

**Key Points**:
- Data collection transparency
- User rights (GDPR/CCPA)
- Cookie usage
- Third-party services
- Data retention
- Security measures

**Full Policy**: https://taskflowpro.com/privacy

### Terms of Service

**Last Updated**: December 2024

**Key Points**:
- Account registration requirements
- Acceptable use policy
- Intellectual property rights
- Payment terms
- Cancellation policy
- Limitation of liability
- Dispute resolution

**Full Terms**: https://taskflowpro.com/terms

### GDPR Compliance

**Data Controller**: TaskFlow Pro, Inc.

**Data Processing**:
- ✅ Lawful basis: Consent + Contract
- ✅ Data minimization
- ✅ Purpose limitation
- ✅ Storage limitation
- ✅ Integrity and confidentiality

**User Rights**:
- ✅ Right to access (data export)
- ✅ Right to rectification (edit profile)
- ✅ Right to erasure (delete account)
- ✅ Right to data portability (JSON/CSV export)
- ✅ Right to object (opt-out)

**DPO Contact**: privacy@taskflowpro.com

### Security Certifications (Roadmap)

- [ ] SOC 2 Type II (Year 1)
- [ ] ISO 27001 (Year 2)
- [ ] GDPR Certification (Year 1)
- [ ] HIPAA Compliance (if needed)

---

## 10. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1.0 | 2024-11-15 | Product Team | Initial draft |
| 0.2.0 | 2024-11-22 | Product Team | Added technical architecture |
| 0.3.0 | 2024-11-29 | Product Team | User personas and journeys |
| 0.4.0 | 2024-12-05 | Product Team | Feature specifications complete |
| 0.5.0 | 2024-12-08 | Product Team | API and data models |
| 0.6.0 | 2024-12-10 | Product Team | Wireframes and UI specs |
| 0.7.0 | 2024-12-12 | Product Team | User stories and acceptance criteria |
| 0.8.0 | 2024-12-14 | Product Team | Integration requirements |
| 0.9.0 | 2024-12-15 | Product Team | Risk mitigation and metrics |
| **1.0.0** | **2024-12-16** | **Product Team** | **Final PRD approved** |

---

## 11. Contributors

### Product Team

**Product Owner**: TBD  
**Product Manager**: TBD  
**Business Analyst**: TBD

### Engineering Team

**Tech Lead**: TBD  
**Frontend Lead**: TBD  
**Backend Lead**: TBD  
**DevOps Lead**: TBD  
**QA Lead**: TBD

### Design Team

**UX Lead**: TBD  
**UI Designer**: TBD  
**UX Researcher**: TBD

### Stakeholders

**Executive Sponsor**: TBD  
**Engineering Manager**: TBD  
**Marketing Lead**: TBD

---

## 12. Additional Resources

### Code Repositories

- **Frontend**: https://github.com/taskflowpro/taskflow-web (TBD)
- **Backend**: https://github.com/taskflowpro/taskflow-api (TBD)
- **Mobile**: https://github.com/taskflowpro/taskflow-mobile (Phase 2)
- **Docs**: https://github.com/taskflowpro/docs (TBD)

### Project Management

- **Jira Board**: TBD
- **Confluence**: TBD
- **Slack Channel**: #taskflow-pro
- **Email List**: taskflow-team@company.com

### Design Assets

- **Figma**: TBD
- **Brand Guidelines**: TBD
- **Icon Library**: Material Icons
- **Image Assets**: TBD

---

## 13. Templates & Boilerplates

### User Story Template

```markdown
### US-[MODULE][NUMBER]: [Title]

**As a** [user type]
**I want to** [action]
**So that** [benefit]

**Acceptance Criteria:**

```gherkin
Feature: [Feature Name]

Scenario: [Scenario Name]
  Given [context]
  When [action]
  Then [outcome]
```

**Implementation Notes:**
- [Note 1]
- [Note 2]

**Priority**: P[0-3]
**Story Points**: [1,2,3,5,8,13]
```

### Bug Report Template

```markdown
**Summary**: Brief description of the bug

**Steps to Reproduce**:
1. Step 1
2. Step 2
3. Step 3

**Expected Behavior**: What should happen

**Actual Behavior**: What actually happens

**Environment**:
- Browser: Chrome 120
- OS: macOS 14
- Screen Size: 1920x1080

**Screenshots**: [Attach if applicable]

**Severity**: Critical | High | Medium | Low

**Priority**: P0 | P1 | P2 | P3
```

### Feature Request Template

```markdown
**Feature Title**: Name of the feature

**Problem Statement**: What problem does this solve?

**Proposed Solution**: How should it work?

**User Stories**:
- As a [user], I want [action] so that [benefit]

**Success Criteria**: How do we know it's successful?

**Alternatives Considered**: Other options explored

**Priority**: P[0-3]
```

---

## 14. Measurement & Analytics

### Analytics Events Schema

```typescript
// User Events
interface UserSignedUpEvent {
  event: 'user_signed_up'
  properties: {
    method: 'email' | 'google' | 'github'
    referral_source?: string
    utm_campaign?: string
  }
}

// Project Events
interface ProjectCreatedEvent {
  event: 'project_created'
  properties: {
    project_id: string
    template?: string
    team_size: number
    has_description: boolean
  }
}

// Task Events
interface TaskCreatedEvent {
  event: 'task_created'
  properties: {
    task_id: string
    project_id: string
    source: 'kanban' | 'list' | 'quick_add'
    has_assignee: boolean
    has_due_date: boolean
    priority: 'low' | 'medium' | 'high' | 'urgent'
  }
}
```

### Cohort Analysis Query

```sql
-- Weekly cohort retention
WITH cohorts AS (
  SELECT
    user_id,
    DATE_TRUNC('week', created_at) AS cohort_week
  FROM users
),
activity AS (
  SELECT
    user_id,
    DATE_TRUNC('week', activity_date) AS activity_week
  FROM user_activities
)
SELECT
  c.cohort_week,
  COUNT(DISTINCT c.user_id) AS cohort_size,
  a.activity_week,
  COUNT(DISTINCT a.user_id) AS active_users,
  ROUND(COUNT(DISTINCT a.user_id)::numeric / COUNT(DISTINCT c.user_id) * 100, 2) AS retention_rate
FROM cohorts c
LEFT JOIN activity a ON c.user_id = a.user_id
GROUP BY c.cohort_week, a.activity_week
ORDER BY c.cohort_week, a.activity_week
```

---

## 15. Support & Documentation

### Help Center Structure

```
Help Center
├── Getting Started
│   ├── What is TaskFlow Pro?
│   ├── Creating your first project
│   ├── Inviting team members
│   └── Quick start guide
│
├── Projects
│   ├── Creating projects
│   ├── Project templates
│   ├── Project settings
│   └── Archiving projects
│
├── Tasks
│   ├── Creating tasks
│   ├── Kanban board
│   ├── Task details
│   ├── Comments and mentions
│   └── Time tracking
│
├── Collaboration
│   ├── Team management
│   ├── Notifications
│   ├── File sharing
│   └── Real-time updates
│
├── Account & Settings
│   ├── Profile settings
│   ├── Notification preferences
│   ├── Billing & subscription
│   └── Security settings
│  
└── Troubleshooting
    ├── Common issues
    ├── Browser compatibility
    ├── Performance tips
    └── Contact support
```

### Support Channels

- **Email**: support@taskflowpro.com
- **Live Chat**: Monday-Friday, 9am-5pm EST
- **Status Page**: status.taskflowpro.com
- **Community Forum**: community.taskflowpro.com
- **Twitter**: @TaskFlowPro

---

## 16. Localization

### Supported Languages (Phase 1)

1. **English (en-US)** - Primary
2. **Spanish (es-ES)**
3. **French (fr-FR)**
4. **German (de-DE)**

### Planned Languages (Phase 2)

5. Japanese (ja-JP)
6. Portuguese (pt-BR)
7. Chinese Simplified (zh-CN)
8. Korean (ko-KR)

### Translation Process

1. Extract strings using i18n extraction tool
2. Send to translation service (Lokalise, Crowdin)
3. Review translations with native speakers
4. Integrate into app
5. QA testing in each language

---

## 17. Accessibility Checklist

- [ ] **Keyboard Navigation**: All features accessible via keyboard
- [ ] **Screen Reader**: Tested with VoiceOver and NVDA
- [ ] **Color Contrast**: WCAG AA (4.5:1 for text)
- [ ] **Focus Indicators**: Visible focus states
- [ ] **Alt Text**: All images have descriptive alt text
- [ ] **Form Labels**: All inputs have associated labels
- [ ] **ARIA Labels**: Used where semantic HTML insufficient
- [ ] **Heading Hierarchy**: Proper H1-H6 structure
- [ ] **Skip Links**: Skip to main content
- [ ] **Error Messages**: Clear, accessible error messages
- [ ] **Responsive Text**: Scales up to 200%
- [ ] **Motion**: Respects prefers-reduced-motion
- [ ] **Video Captions**: All videos have captions
- [ ] **Language**: Proper lang attribute

---

## 18. Performance Budgets

### Page Load Budgets

| Metric | Budget | Warning | Critical |
|--------|--------|---------|----------|
| Total Page Size | < 500 KB | 500-750 KB | > 750 KB |
| JavaScript | < 200 KB | 200-300 KB | > 300 KB |
| CSS | < 50 KB | 50-75 KB | > 75 KB |
| Images | < 200 KB | 200-300 KB | > 300 KB |
| Fonts | < 50 KB | 50-75 KB | > 75 KB |
| LCP | < 2.0s | 2.0-2.5s | > 2.5s |
| FID | < 50ms | 50-100ms | > 100ms |
| CLS | < 0.05 | 0.05-0.1 | > 0.1 |

---

## 19. Browser Support Matrix

| Browser | Version | Support Level |
|---------|---------|---------------|
| Chrome | 100+ | Full Support |
| Firefox | 100+ | Full Support |
| Safari | 15+ | Full Support |
| Edge | 100+ | Full Support |
| Safari iOS | 15+ | Full Support |
| Chrome Android | 100+ | Full Support |
| Samsung Internet | 16+ | Best Effort |
| Opera | 85+ | Best Effort |
| IE 11 | - | Not Supported |

---

## 20. Related Documents

### Product Requirements
- [01-executive-summary.md](./01-executive-summary.md)
- [02-product-vision-goals.md](./02-product-vision-goals.md)
- [03-user-personas-journeys.md](./03-user-personas-journeys.md)
- [04-feature-specifications.md](./04-feature-specifications.md)

### Design & UX
- [05-user-stories-acceptance.md](./05-user-stories-acceptance.md)
- [06-wireframes-ux-design.md](./06-wireframes-ux-design.md)
- [13-wireframes-ui-specifications.md](./13-wireframes-ui-specifications.md)

### Technical
- [07-api-specifications.md](./07-api-specifications.md)
- [08-data-models-schema.md](./08-data-models-schema.md)
- [09-integration-requirements.md](./09-integration-requirements.md)
- [10-technical-architecture.md](./10-technical-architecture.md)
- [14-integration-requirements.md](./14-integration-requirements.md)

### Planning & Strategy
- [11-success-metrics-kpis.md](./11-success-metrics-kpis.md)
- [11-user-stories-acceptance-criteria.md](./11-user-stories-acceptance-criteria.md)
- [12-roadmap-phasing.md](./12-roadmap-phasing.md)
- [14-risk-mitigation.md](./14-risk-mitigation.md)
- [15-competitive-analysis.md](./15-competitive-analysis.md)

---

## Document End

**TaskFlow Pro - Product Requirements Document**  
**Version 1.0.0** | December 2024

For questions or updates, contact the Product Team.

---

**[Return to README](./README.md)**
