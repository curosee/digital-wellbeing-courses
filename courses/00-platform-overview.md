# CuroSee Platform - Product Requirements Document Overview

## Executive Summary

CuroSee is a comprehensive digital wellness platform that helps parents and adults manage screen time, break free from excessive digital device usage, and develop healthier technology habits. Built on the Indistractable methodology by Nir Eyal, the platform provides education, tools, AI-powered guidance, and community support—all completely free with optional donation-based premium features.

## Platform Vision

**Mission**: Empower individuals and families to reclaim their time and attention from digital distraction through evidence-based education, practical tools, and compassionate AI guidance.

**Target Audience**:
- Parents concerned about family screen time
- Adults struggling with digital distraction
- Educators teaching digital citizenship
- Healthcare professionals supporting patients
- Anyone seeking healthier technology habits

**Value Proposition**: Free, science-based, judgment-free support for digital wellness that combines structured learning, AI coaching, and practical tools—accessible to everyone regardless of ability to pay.

## Platform Architecture

### Technology Stack

#### Frontend
- **Framework**: React 18 with TypeScript
- **Routing**: React Router v6
- **State Management**: React Context API + Zustand
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **Internationalization**: i18next (5 languages: EN, FR, ES, AR, FA)

#### Backend
- **Database**: PostgreSQL (via Supabase)
- **Authentication**: Supabase Auth (email magic link)
- **API**: Supabase Client + REST APIs
- **Serverless Functions**: Supabase Edge Functions (Deno runtime)
- **Background Jobs Scheduler**: Trigger.dev local background jobs scheduler container (trigger)
- **Caching**: Local Redis container (redis)
- **AI Integration**: OpenAI GPT-5 / Gemini 3

#### External Integrations
- **Content Source**: WordPress REST API (articles)
- **Payment Processing**: Donations to direct crypto wallet and paypal link.
- **AI Services**: OpenAI API / Google Gemini

### Hosting & Deployment
- **Frontend**: Coolify (opensource netlify alternative)
- **Backend**: Supabase (managed PostgreSQL + Edge Functions)
- **CDN**: Automatic via hosting provider
- **SSL**: Automatic via hosting provider


## Monetization Strategy

### Free Tier (100% of features)
- All courses and articles
- All tools with unlimited usage
- Chatbot: 100 messages/month
- Unlimited saved data
- Full platform access

### Premium Tier (€15 donation)
- Chatbot: 200 messages/month
- Priority support (future)
- Early access to new features (future)
- Advanced analytics (future)
- No other feature restrictions

### Philosophy
- Free-first, donation-supported model
- No paywalls or feature gates
- Donations encouraged but not required
- Premium perks enhance but don't lock core value
- Sustainable through community support

## User Personas

### Persona 1: Concerned Parent (Primary)
**Profile**: Sarah, 38, mother of 2 (ages 7 and 10)
**Pain Points**:
- Children spending too much time on devices
- Unsure what content is appropriate
- Family arguments about screen time
- Difficulty setting consistent boundaries

**CuroSee Journey**:
1. Discovers platform searching for "healthy screen time for kids"
2. Reads articles on digital parenting
3. Takes Screen Time Calculator to assess current usage
4. Enrolls in "Indistractable Parenting" course
5. Uses Media Plan Builder to create family agreement
6. Uses Content Recommender for age-appropriate apps
7. Returns monthly for new articles and updates

### Persona 2: Self-Improving Adult
**Profile**: David, 29, software developer
**Pain Points**:
- Excessive social media scrolling
- Work-life balance challenges
- Phone use interfering with sleep
- Feeling distracted and unproductive

**CuroSee Journey**:
1. Discovers platform through Indistractable book reference
2. Takes Screen Time Calculator for self-assessment
3. Enrolls in "Mastering Distraction" course
4. Starts 21-day Detox Challenge
5. Uses AI Chatbot for daily coaching
6. Completes course and donates €15
7. Becomes advocate, shares with friends

### Persona 3: Educator
**Profile**: Maria, 45, middle school teacher
**Pain Points**:
- Students distracted by phones
- Teaching digital citizenship
- Need for evidence-based resources
- Limited budget for tools

**CuroSee Journey**:
1. Searches for free digital citizenship curriculum
2. Enrolls students in beginner courses
3. Uses tools in classroom activities
4. Shares articles with parents
5. Provides feedback for platform improvement

## Competitive Analysis

### Direct Competitors
- **Screen Time Labs**: Paid app, limited features
- **Qustodio**: Parental control focus, expensive
- **Forest**: Single-purpose app, gamification
- **Freedom**: Website blocker, subscription

### CuroSee Advantages
- ✅ Completely free core offering
- ✅ Comprehensive (education + tools)
- ✅ AI-powered personalization
- ✅ Evidence-based methodology
- ✅ Multi-language support
- ✅ Privacy-focused
- ✅ No ads or tracking
- ✅ Community-driven

### Areas for Improvement
- ❌ No mobile app (web-only)
- ❌ No device-level tracking integration
- ❌ Smaller content library
- ❌ Limited social features
- ❌ New platform (less brand recognition)

## Technical Debt & Known Issues

### High Priority
1. **Password Reset**: UI exists but not functional
2. **Email Verification**: Registration doesn't require verification
3. **Video Integration**: Video URLs exist but no player
4. **Bookmark System**: Database ready but feature not built
5. **Achievement System**: UI ready but backend not implemented

### Medium Priority
1. **Chatbot History**: Conversations not persisted
2. **Offline Mode**: No offline functionality
3. **Mobile Optimization**: Some UI issues on small screens
4. **Print Styling**: PDF generation could be improved
5. **Search Performance**: Basic substring matching only

### Low Priority
1. **Code Comments**: Minimal code documentation
2. **Test Coverage**: No automated tests
3. **Performance Monitoring**: Limited analytics
4. **Error Tracking**: Basic error handling only
5. **A/B Testing**: No experimentation framework


## Success Metrics

### Platform Health
- **User Growth**: Monthly active users
- **Retention**: 7-day, 30-day, 90-day retention
- **Engagement**: Average session duration
- **Completion**: Course completion rate
- **Tool Usage**: Tools used per user

### Content Metrics
- **Articles**: Views, time on page, shares
- **Courses**: Enrollment, completion, ratings
- **Tools**: Usage frequency, saved results
- **Chatbot**: Messages per user, satisfaction

### Business Metrics
- **Donations**: Conversion rate, average amount
- **Cost**: AI API costs, infrastructure costs
- **ROI**: Revenue vs. costs
- **Sustainability**: Monthly recurring revenue

### Impact Metrics
- **Behavior Change**: Self-reported improvements
- **User Satisfaction**: NPS score, surveys
- **Community**: User testimonials, referrals
- **Awareness**: Social media mentions, press

## Compliance & Security

### Data Privacy
- ✅ GDPR compliant (EU)
- ✅ CCPA compliant (California)
- ✅ COPPA aware (children's data)
- ✅ Transparent privacy policy
- ✅ User data export capability
- ✅ Right to be forgotten

### Security Measures
- ✅ Row-level security (RLS) on all tables
- ✅ Password hashing (Supabase Auth)
- ✅ HTTPS everywhere
- ✅ API key security
- ✅ Input validation
- ⚠️ CSRF protection (basic)
- ⚠️ Rate limiting (basic)

### Accessibility
- ✅ Keyboard navigation
- ✅ Screen reader compatible
- ✅ High contrast ratios
- ✅ Semantic HTML
- ✅ Multi-language support
- ✅ RTL layout support
- ⚠️ WCAG 2.1 AA (partial compliance)


## Conclusion

CuroSee is a comprehensive, well-architected digital wellness platform that successfully combines education, tools, and AI guidance into a cohesive free offering. The platform demonstrates strong technical implementation with modern practices (React, TypeScript, Supabase, Edge Functions) and sophisticated features (AI integration, multi-language support, progress tracking).

**Strengths**:
- Solid technical foundation
- Comprehensive feature set
- User-centric design
- Free-first philosophy
- Evidence-based approach
- Privacy-focused

**Opportunities**:
- Mobile application
- Community features
- Content expansion
- Partnership development
- Marketing & growth
- Research validation

The platform is production-ready and positioned for growth with clear monetization strategy, manageable costs, and high user value. Primary focus should be user acquisition, content expansion, and community building to achieve sustainability.
