# Success Metrics & KPIs

**Document**: #11 - Success Metrics & KPIs  
**Version**: 1.0.0  
**Last Updated**: December 2024

---

## Overview

This document defines measurable success metrics and Key Performance Indicators (KPIs) for TaskFlow Pro. These metrics will help track product performance, user engagement, business growth, and overall product-market fit.

---

## Executive Summary

### North Star Metric

**Weekly Active Teams (WAT)**

Our primary metric measuring teams that actively use TaskFlow Pro at least once per week. This metric reflects both user acquisition and retention while indicating real value delivery.

**Target**: 10,000 Weekly Active Teams by end of Year 1

---

## Metric Categories

1. **User Acquisition & Growth**
2. **User Engagement & Retention**
3. **Feature Adoption**
4. **Product Performance**
5. **Business Metrics**
6. **Customer Satisfaction**

---

## 1. User Acquisition & Growth Metrics

### 1.1 User Signups

**Definition**: Total new user registrations per period

**Measurement**:
```sql
SELECT COUNT(*) as signups
FROM users
WHERE created_at BETWEEN :start_date AND :end_date
```

**Targets**:
- Month 1-3: 500 signups/month
- Month 4-6: 2,000 signups/month
- Month 7-12: 5,000 signups/month

**Success Criteria**:
- ✅ Achieve 10,000 total users in first 6 months
- ✅ Month-over-month growth > 20%
- ✅ Signup conversion rate > 15% from landing page

---

### 1.2 Activation Rate

**Definition**: Percentage of users who complete key onboarding actions

**Key Actions**:
1. Email verification
2. Create first project
3. Create first task
4. Invite team member

**Measurement**:
```javascript
activationRate = (usersCompletedOnboarding / totalSignups) * 100
```

**Targets**:
- Week 1-4: 30% activation
- Month 2-3: 50% activation
- Month 4+: 65% activation

**Tracking**:
```sql
WITH onboarding_steps AS (
  SELECT 
    user_id,
    MAX(CASE WHEN event = 'email_verified' THEN 1 ELSE 0 END) as verified,
    MAX(CASE WHEN event = 'project_created' THEN 1 ELSE 0 END) as project,
    MAX(CASE WHEN event = 'task_created' THEN 1 ELSE 0 END) as task,
    MAX(CASE WHEN event = 'member_invited' THEN 1 ELSE 0 END) as invited
  FROM user_events
  GROUP BY user_id
)
SELECT 
  AVG(verified + project + task + invited) / 4 * 100 as activation_rate
FROM onboarding_steps
```

---

### 1.3 Viral Coefficient (K-factor)

**Definition**: Average number of new users each existing user brings

**Formula**:
```
K-factor = (Invitations Sent / Active Users) × (Signups from Invitations / Total Invitations)
```

**Target**: K > 1.2 (viral growth)

**Tracking**:
- Track invitation source in signup flow
- Measure conversion rate of invitations
- Identify viral loop bottlenecks

---

### 1.4 Traffic Sources

**Breakdown**:
- Organic Search: 30%
- Direct Traffic: 20%
- Social Media: 15%
- Paid Advertising: 20%
- Referrals: 10%
- Other: 5%

**Tools**: Google Analytics 4, UTM tracking

---

## 2. User Engagement & Retention Metrics

### 2.1 Daily Active Users (DAU)

**Definition**: Unique users who perform meaningful action per day

**Meaningful Actions**:
- Create/edit task
- Comment on task
- Update project
- View dashboard

**Target Growth**:
- Month 1: 200 DAU
- Month 3: 1,000 DAU
- Month 6: 3,000 DAU
- Month 12: 10,000 DAU

---

### 2.2 Weekly Active Users (WAU)

**Definition**: Unique users active at least once per week

**Target**: 15,000 WAU by Month 12

**DAU/WAU Ratio**: Target 0.4-0.5 (healthy engagement)

---

### 2.3 Monthly Active Users (MAU)

**Definition**: Unique users active at least once per month

**Target**: 30,000 MAU by Month 12

**Engagement Ratio**:
```
DAU/MAU Ratio = 0.25-0.35 (good)
WAU/MAU Ratio = 0.5-0.6 (good)
```

---

### 2.4 Retention Cohorts

**Day 1 Retention**: 60% (users return next day)  
**Day 7 Retention**: 40% (users return after 1 week)  
**Day 30 Retention**: 25% (users return after 1 month)  
**Day 90 Retention**: 20% (users still active after 3 months)

**Cohort Analysis**:
```sql
SELECT 
  DATE_TRUNC('month', u.created_at) as cohort_month,
  COUNT(DISTINCT u.id) as cohort_size,
  COUNT(DISTINCT CASE WHEN ae.activity_date <= u.created_at + INTERVAL '1 day' THEN u.id END) as day_1,
  COUNT(DISTINCT CASE WHEN ae.activity_date <= u.created_at + INTERVAL '7 days' THEN u.id END) as day_7,
  COUNT(DISTINCT CASE WHEN ae.activity_date <= u.created_at + INTERVAL '30 days' THEN u.id END) as day_30
FROM users u
LEFT JOIN activity_events ae ON u.id = ae.user_id
GROUP BY cohort_month
```

**Visualization**: Cohort retention heatmap

---

### 2.5 Session Metrics

**Average Session Duration**: Target 12-15 minutes  
**Average Sessions per User per Week**: Target 3-5  
**Bounce Rate**: Target < 30%

**Deep Engagement Sessions**: Sessions > 10 minutes with 5+ actions

---

### 2.6 Feature Stickiness

**Definition**: How often users return to use specific features

**Formula**:
```
Feature Stickiness = DAU using feature / MAU using feature
```

**Target Features**:
- Kanban Board: 0.5 stickiness
- Task Comments: 0.4 stickiness
- Project Dashboard: 0.45 stickiness
- Time Tracking: 0.35 stickiness

---

## 3. Feature Adoption Metrics

### 3.1 Feature Discovery Rate

**Definition**: % of users who use a feature within 30 days

**Key Features**:

| Feature | Target Adoption | Current (Baseline) |
|---------|----------------|-------------------|
| Kanban Board | 85% | TBD |
| Task Comments | 70% | TBD |
| File Attachments | 60% | TBD |
| Time Tracking | 45% | TBD |
| Reports & Analytics | 40% | TBD |
| Integrations | 35% | TBD |
| Dark Mode | 50% | TBD |

**Measurement**:
```javascript
const featureAdoption = (usersWhoUsedFeature, totalUsers) => {
  return (usersWhoUsedFeature / totalUsers) * 100
}
```

---

### 3.2 Average Tasks per User

**Targets**:
- Week 1: 5 tasks/user
- Month 1: 15 tasks/user
- Month 3: 40 tasks/user
- Month 6: 80 tasks/user

**Segmentation**:
- Power Users (>100 tasks): Top 10%
- Active Users (20-100 tasks): 40%
- Casual Users (5-20 tasks): 35%
- Inactive Users (<5 tasks): 15%

---

### 3.3 Average Projects per Team

**Targets**:
- Month 1: 2 projects/team
- Month 3: 4 projects/team
- Month 6: 7 projects/team

**Distribution**:
- 1 project: 30% of teams
- 2-5 projects: 45% of teams
- 6-10 projects: 15% of teams
- 10+ projects: 10% of teams

---

### 3.4 Collaboration Metrics

**Comments per Task**: Target 3-5 comments/task  
**@Mentions per Week**: Target 10-15 mentions/active user  
**File Attachments**: Target 2 attachments/project  
**Team Size**: Average 5-8 members/team

**Real-time Collaboration**:
- Concurrent users editing: Track simultaneous edits
- Response time to comments: < 4 hours average

---

## 4. Product Performance Metrics

### 4.1 Page Load Performance

**Core Web Vitals**:

| Metric | Target | Max Acceptable |
|--------|--------|----------------|
| **LCP** (Largest Contentful Paint) | < 2.0s | 2.5s |
| **FID** (First Input Delay) | < 50ms | 100ms |
| **CLS** (Cumulative Layout Shift) | < 0.05 | 0.1 |
| **TTFB** (Time to First Byte) | < 500ms | 800ms |

**Tools**: Lighthouse, WebPageTest, Real User Monitoring

---

### 4.2 Application Performance

**API Response Times**:
- P50: < 200ms
- P95: < 500ms
- P99: < 1000ms

**Database Query Performance**:
- P50: < 50ms
- P95: < 200ms
- P99: < 500ms

**WebSocket Latency**: < 100ms for real-time updates

---

### 4.3 Error Rates

**Target Error Rates**:
- JavaScript Errors: < 0.1% of page views
- API Error Rate: < 0.5% of requests
- Failed Transactions: < 0.1%

**Monitoring**: Sentry error tracking

**Alert Thresholds**:
- Error rate > 1%: Warning
- Error rate > 5%: Critical
- API downtime > 1 minute: Critical

---

### 4.4 Uptime & Availability

**Target SLA**: 99.9% uptime (43 minutes downtime/month)

**Measurement**:
- Uptime monitoring (Pingdom, UptimeRobot)
- Status page updates
- Incident response time < 15 minutes
- Mean Time to Recovery (MTTR) < 1 hour

---

### 4.5 Mobile Performance

**Mobile Page Load**: < 3s on 4G  
**Mobile Error Rate**: < 0.2%  
**Mobile Bounce Rate**: < 40%

**Device Breakdown**:
- Desktop: 60%
- Mobile: 30%
- Tablet: 10%

---

## 5. Business Metrics

### 5.1 Revenue Metrics

**Monthly Recurring Revenue (MRR)**

**Targets**:
- Month 3: $5,000 MRR
- Month 6: $25,000 MRR
- Month 12: $100,000 MRR

**Formula**:
```
MRR = Sum of monthly subscription revenue
```

---

### 5.2 Average Revenue Per User (ARPU)

**Target ARPU**: $12-15/user/month

**Calculation**:
```
ARPU = Total MRR / Total Active Paying Users
```

**Segmentation**:
- Free Plan: $0
- Pro Plan: $12/user/month
- Enterprise: $20-50/user/month (custom)

---

### 5.3 Customer Acquisition Cost (CAC)

**Target CAC**: < $50 per customer

**Formula**:
```
CAC = (Sales + Marketing Costs) / New Customers Acquired
```

**Channels**:
- Paid Ads: $80 CAC
- Content Marketing: $30 CAC
- Referrals: $15 CAC
- Organic Search: $20 CAC

---

### 5.4 Lifetime Value (LTV)

**Target LTV**: $500+

**Formula**:
```
LTV = ARPU × Average Customer Lifespan (months)
```

**Goal**: LTV:CAC ratio > 3:1

**Calculation Example**:
```
ARPU = $15/month
Average lifespan = 36 months
LTV = $15 × 36 = $540
CAC = $50
LTV:CAC = 10.8:1 ✅
```

---

### 5.5 Churn Rate

**Target Monthly Churn**: < 5%

**Formula**:
```
Monthly Churn Rate = (Customers Lost in Month / Customers at Start of Month) × 100
```

**Cohort Churn Analysis**:
- Month 1-3: 15% (acceptable for new users)
- Month 4-6: 8%
- Month 7-12: 5%
- Month 13+: < 3% (loyal customers)

**Revenue Churn**: Track separately from user churn

---

### 5.6 Conversion Rates

**Free to Paid Conversion**: Target 15%

**Trial to Paid Conversion**: Target 25% (if trial offered)

**Funnel**:
```
Landing Page → Signup: 15% conversion
Signup → Activation: 65% conversion
Activation → Paid: 15% conversion
Overall: 1.5% landing page to paid
```

---

### 5.7 Net Revenue Retention (NRR)

**Target NRR**: > 100% (growth from existing customers)

**Formula**:
```
NRR = (Starting MRR + Expansion - Churn) / Starting MRR × 100
```

**Expansion Revenue**:
- Upsells (Free → Pro)
- Add-on features
- Additional users

---

## 6. Customer Satisfaction Metrics

### 6.1 Net Promoter Score (NPS)

**Target NPS**: > 50

**Measurement**:
- Survey: "How likely are you to recommend TaskFlow Pro?" (0-10)
- Promoters (9-10): 60%
- Passives (7-8): 30%
- Detractors (0-6): 10%

**Calculation**:
```
NPS = % Promoters - % Detractors = 60% - 10% = 50
```

**Survey Frequency**: Quarterly in-app survey

---

### 6.2 Customer Satisfaction Score (CSAT)

**Target CSAT**: > 4.5/5

**Survey**: "How satisfied are you with TaskFlow Pro?"
- 5 stars: Very Satisfied (70%)
- 4 stars: Satisfied (20%)
- 3 stars: Neutral (7%)
- 2 stars: Unsatisfied (2%)
- 1 star: Very Unsatisfied (1%)

**Trigger Points**:
- After onboarding completion
- After support interaction
- Monthly active users

---

### 6.3 Feature Satisfaction

**Per-Feature Ratings**:

| Feature | Target Rating | Importance |
|---------|--------------|------------|
| Kanban Board | 4.7/5 | High |
| Task Management | 4.6/5 | High |
| Collaboration | 4.4/5 | High |
| Reports | 4.2/5 | Medium |
| Integrations | 4.3/5 | Medium |
| Mobile App | 4.0/5 | Low |

---

### 6.4 Support Metrics

**First Response Time**: < 2 hours  
**Resolution Time**: < 24 hours  
**Support Ticket Volume**: < 5% of MAU  
**Self-Service Resolution**: Target 60%

**CSAT for Support**: > 90%

---

### 6.5 Product-Market Fit Score

**Survey Question**: "How would you feel if you could no longer use TaskFlow Pro?"

**Target**:
- Very Disappointed: > 40% (indicates PMF)
- Somewhat Disappointed: 30-40%
- Not Disappointed: < 30%

**Milestone**: Achieve PMF score > 40% by Month 6

---

## 7. Analytics Implementation

### 7.1 Event Tracking

**Core Events**:

```javascript
// User Events
analytics.track('User Signed Up', {
  method: 'email' | 'google' | 'github',
  referral_source: string,
  onboarding_completed: boolean
})

analytics.track('User Activated', {
  days_to_activation: number,
  projects_created: number,
  tasks_created: number
})

// Project Events
analytics.track('Project Created', {
  template: string,
  team_size: number,
  has_description: boolean
})

// Task Events
analytics.track('Task Created', {
  source: 'kanban' | 'list' | 'quick_add',
  has_assignee: boolean,
  has_due_date: boolean,
  has_description: boolean,
  priority: string
})

analytics.track('Task Completed', {
  time_to_complete_hours: number,
  comments_count: number,
  attachments_count: number
})

// Collaboration Events
analytics.track('Comment Added', {
  task_id: string,
  has_mention: boolean,
  has_attachment: boolean
})

analytics.track('User Mentioned', {
  context: 'task' | 'comment' | 'project'
})

// Feature Usage
analytics.track('Feature Used', {
  feature_name: string,
  time_spent_seconds: number
})
```

---

### 7.2 Custom Dashboards

**Real-Time Dashboard**:
- Current active users
- New signups (last hour)
- Tasks created (last hour)
- Error rate

**Weekly Dashboard**:
- WAU trend
- Activation rate
- Retention cohorts
- Feature adoption

**Monthly Dashboard**:
- MRR growth
- Churn rate
- NPS score
- LTV:CAC ratio

**Tools**: Mixpanel, Amplitude, Custom Grafana dashboards

---

### 7.3 A/B Testing

**Test Framework**:
- Onboarding flow variations
- Pricing page layouts
- Feature discovery prompts
- Email campaigns

**Minimum Sample Size**: 1,000 users per variant

**Statistical Significance**: p < 0.05

**Test Duration**: Minimum 2 weeks

---

## 8. Success Milestones

### Month 3 Milestones
- ✅ 1,000 total users
- ✅ 200 WAT (Weekly Active Teams)
- ✅ 40% activation rate
- ✅ $5,000 MRR
- ✅ 99.5% uptime
- ✅ NPS > 40

### Month 6 Milestones
- ✅ 5,000 total users
- ✅ 1,000 WAT
- ✅ 55% activation rate
- ✅ $25,000 MRR
- ✅ 99.9% uptime
- ✅ NPS > 45
- ✅ Product-Market Fit achieved

### Month 12 Milestones
- ✅ 20,000 total users
- ✅ 5,000 WAT
- ✅ 65% activation rate
- ✅ $100,000 MRR
- ✅ 99.95% uptime
- ✅ NPS > 50
- ✅ < 5% monthly churn

---

## 9. Reporting Cadence

### Daily Reports (Automated)
- Active users (DAU)
- New signups
- Error rates
- Uptime status

### Weekly Reports
- WAU trend
- Activation funnel
- Top features used
- Support ticket volume

### Monthly Reports (Executive)
- MRR/ARR growth
- User acquisition & retention
- Feature adoption
- NPS & CSAT
- Product roadmap progress

### Quarterly Business Reviews
- OKR performance
- Market analysis
- Competitive positioning
- Strategic planning

---

## 10. Tools & Infrastructure

### Analytics Platform
- **Primary**: Mixpanel or Amplitude
- **Web Analytics**: Google Analytics 4
- **Session Replay**: Hotjar or FullStory
- **Error Tracking**: Sentry
- **Uptime Monitoring**: Pingdom

### Data Warehouse
- **Storage**: BigQuery or Snowflake
- **ETL**: Fivetran or Airbyte
- **BI Tool**: Metabase or Looker

### Custom Metrics API
```javascript
// GET /api/v1/metrics/summary
{
  "period": "week",
  "metrics": {
    "wau": 1250,
    "wau_growth": "+12%",
    "activation_rate": 58,
    "mrr": 15000,
    "nps": 48,
    "uptime": 99.95
  }
}
```

---

## Related Documents

- [Product Vision & Goals](./02-product-vision-goals.md)
- [Feature Specifications](./04-feature-specifications.md)
- [Roadmap & Phasing](./12-roadmap-phasing.md)
- [Technical Architecture](./10-technical-architecture.md)

---

**Next**: [Roadmap & Phasing](./12-roadmap-phasing.md) →
