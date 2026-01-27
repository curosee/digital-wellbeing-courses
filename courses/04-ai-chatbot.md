# AI Chatbot - Product Requirements Document

## Product Overview
The CuroSee AI Chatbot is a conversational assistant powered by OpenAI's GPT model, providing personalized guidance on digital wellness, screen time management, and parenting strategies. The chatbot implements a tiered usage system based on user authentication and includes sophisticated usage tracking.

## Current State

### Features Implemented

#### 1. Chatbot Interface
- **Location**: Global component, `src/components/ChatBot.tsx`
- **Trigger**: Floating button in bottom-right corner of all pages
- **Design**:
  - Fixed position overlay
  - Slide-in animation
  - Modern card-based UI
  - Gradient header with bot avatar
  - Message thread area
  - Input field with send button
  - Usage meter display
  - Close button

#### 2. Conversation System
- **Message Types**:
  - User messages (right-aligned, blue background)
  - Assistant messages (left-aligned, gray background)
  - System messages (centered, informational)
- **Features**:
  - Real-time message exchange
  - Auto-scroll to latest message
  - Message history within session
  - Loading indicators during AI response
  - Error handling with user feedback
  - Enter key to send
  - Multi-line support with Shift+Enter

#### 3. Usage Tracking System
- **Tiers**:
  - **Anonymous**: 3 messages per session
  - **Free (Authenticated)**: 100 messages per month
  - **Premium (Donated €15+)**: 200 messages per month

#### 4. Usage Meter
- **Display**:
  - Messages used / Total limit
  - Visual progress bar (color-coded)
  - Tier indication badge
  - Percentage calculation
- **Color Coding**:
  - Green: 0-69% used
  - Yellow: 70-89% used
  - Red: 90-100% used

#### 5. Limit Warning System
- **Behavior**:
  - Warning at 3 messages remaining
  - Hard block when limit reached
  - Upgrade prompts with CTA buttons
  - Contextual messaging per tier
- **Anonymous User Prompt**:
  - "Create an account to get 100 messages per month"
  - Link to registration page
  - Preserves context after auth
- **Free User Prompt**:
  - "Donate €15 to unlock 200 messages per month"
  - Link to donation page
  - Premium upgrade incentive

#### 6. Session Management
- **Anonymous Users**:
  - Session ID generated client-side
  - Stored in localStorage
  - Persists across page reloads
  - Tied to browser/device
  - Format: `anon_<timestamp>_<random>`
- **Authenticated Users**:
  - Usage tied to user ID
  - Synced across all devices
  - Monthly reset on same day as registration
  - Persistent history (future)

## Database Schema

### Chatbot Usage Tracking
```sql
CREATE TABLE chatbot_usage (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  session_id text,
  message_count integer DEFAULT 0,
  month_year text NOT NULL,
  tier text NOT NULL CHECK (tier IN ('anonymous', 'free', 'premium')),
  last_message_at timestamptz DEFAULT now(),
  created_at timestamptz DEFAULT now(),

  UNIQUE(user_id, month_year),
  UNIQUE(session_id, month_year),
  CHECK ((user_id IS NOT NULL AND session_id IS NULL) OR
         (user_id IS NULL AND session_id IS NOT NULL))
);

CREATE INDEX idx_chatbot_usage_user ON chatbot_usage(user_id, month_year);
CREATE INDEX idx_chatbot_usage_session ON chatbot_usage(session_id, month_year);
```

### Chat History (Future)
```sql
CREATE TABLE chat_messages (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  session_id text,
  role text NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
  content text NOT NULL,
  language text NOT NULL,
  created_at timestamptz DEFAULT now()
);
```

## Edge Function: chatbot

### Location
`supabase/functions/chatbot/index.ts`

### Functionality
- Receives user messages
- Checks usage limits
- Generates AI responses via OpenAI API
- Increments usage counter
- Returns formatted response

### OpenAI Integration
- **Model**: GPT-4 or GPT-3.5-turbo (configurable)
- **System Prompt**:
  ```
  You are a digital wellness expert specializing in screen time management
  and the Indistractable methodology by Nir Eyal. Provide empathetic,
  practical, and evidence-based advice to parents and adults struggling
  with excessive screen time. Be supportive, non-judgmental, and actionable.
  Keep responses concise (under 150 words) unless the user asks for details.
  ```
- **Parameters**:
  - Temperature: 0.7 (balanced creativity)
  - Max tokens: 500
  - Top P: 1
  - Frequency penalty: 0.3
  - Presence penalty: 0.3

### Language Support
- Accepts user's current language setting
- AI responds in same language as user interface
- System prompt translated per language
- Maintains context across languages

### Request Format
```typescript
{
  message: string,
  language: string,
  sessionId?: string,
  userId?: string
}
```

### Response Format
```typescript
{
  success: boolean,
  response: string,
  usageInfo: {
    messageCount: number,
    limit: number,
    tier: string,
    remaining: number
  },
  error?: string
}
```

## Security Implementation

### Rate Limiting
- **Implementation**: Database-enforced limits
- **Anonymous**: Max 3 messages per browser session
- **Free**: Max 100 messages per calendar month
- **Premium**: Max 200 messages per calendar month
- **Cooldown**: None currently (future: 1 second between messages)

### Data Privacy
- Anonymous sessions tied to client-side ID only
- No PII collected for anonymous users
- Conversation history not stored (currently)
- OpenAI data retention: 30 days
- No conversation data shared with third parties

### Input Validation
- Message length limit: 500 characters
- XSS protection on rendered messages
- SQL injection prevention via parameterized queries
- Profanity filter (future)
- Content moderation (future)

## User Experience Flows

### Anonymous User Flow
1. User clicks chatbot button
2. Chatbot opens with welcome message
3. Session ID generated and stored
4. Usage meter shows "Guest - 0/3 messages"
5. User sends first message
6. AI responds
7. Counter updates to 1/3
8. After 3 messages:
   - Input disabled
   - Warning message displayed
   - "Create Account" CTA shown
9. User can create account or close chat

### Authenticated User Flow (First Time)
1. User logs in
2. Clicks chatbot button
3. Chatbot opens with personalized greeting
4. Usage meter shows "Free Tier - 0/100 messages"
5. User asks questions throughout month
6. Counter increments with each message
7. Warning at 97/100 messages
8. At 100/100:
   - Hard limit reached
   - Donation prompt displayed
   - Link to support page
9. Monthly reset on next billing cycle

### Premium User Flow
1. User donates €15+
2. Account upgraded to premium tier
3. Usage limit increases to 200/month
4. Badge changes to "Premium"
5. Enhanced features unlocked (future):
   - Conversation history
   - Priority response time
   - Advanced AI model
   - Export transcripts

## Conversation Context

### System Behavior
- **Persona**: Warm, empathetic expert coach
- **Tone**: Supportive, non-judgmental, encouraging
- **Response Style**:
  - Concise by default (2-3 paragraphs)
  - Actionable advice prioritized
  - Evidence-based recommendations
  - Real-world examples provided
  - Follow-up questions suggested

### Expertise Areas
- Screen time management strategies
- Indistractable methodology
- Parental controls and tools
- Digital wellness frameworks
- Behavior change psychology
- Family media plans
- Habit formation
- Trigger management
- Technology boundaries
- Mindful device usage

### Out of Scope
- Medical advice
- Mental health diagnosis
- Legal advice
- Product endorsements
- Political opinions
- Personal data collection

## Integration Points

### With Authentication System
- Usage tier determined by auth status
- User ID used for tracking
- Premium status checked via database
- Monthly reset tied to user creation date

### With Donation System
- Donation of €15+ upgrades to premium
- Manual tier upgrade in database
- Donation confirmation triggers tier change
- Donation amount tracked for analytics

### With Dashboard
- Chatbot usage statistics (future)
- Conversation history access (future)
- Monthly usage charts (future)

## Known Limitations

### Current Constraints
1. **No Conversation History**: Messages not persisted
2. **No Context Across Sessions**: Each session starts fresh
3. **Simple Usage Tracking**: Month-based only, no daily limits
4. **No Export**: Cannot save or export conversations
5. **No Rating System**: Cannot rate AI responses
6. **Anonymous Restrictions**: Limited to 3 messages total per session
7. **No Multi-Turn Context**: AI doesn't remember earlier in conversation
8. **Manual Tier Upgrade**: Premium requires manual database update
9. **No Payment Integration**: Donation link external, no auto-upgrade
10. **Single Language Per Session**: Cannot switch mid-conversation

### Technical Limitations
- OpenAI API dependency (single point of failure)
- No fallback AI model
- No offline mode
- No caching of common questions
- No smart suggestions/prompts
- Limited error recovery
- No retry logic for failed requests
- localStorage dependency for anonymous users

### UX Limitations
- No typing indicators from AI
- No message editing
- No message deletion
- No conversation search
- No bookmarking of responses
- No sharing of conversations
- Mobile keyboard issues (potential)

## Future Enhancements

### Priority 1 (Core Experience)
- Persist conversation history for authenticated users
- Implement conversation context (remember earlier messages)
- Add typing indicators
- Create smart prompt suggestions
- Add conversation rating system
- Export conversations as PDF/text

### Priority 2 (Premium Features)
- Automated tier upgrades via Stripe webhook
- Advanced AI model for premium users
- Priority response queue
- Longer message limits
- Conversation search
- Conversation sharing

### Priority 3 (Intelligence)
- Suggested follow-up questions
- Resource recommendations based on conversation
- Integration with course content
- Personalized action plans
- Progress tracking over time
- AI-generated summaries of conversations

### Priority 4 (Advanced)
- Voice input/output
- Multi-language conversation switching
- Video response generation (AI avatar)
- Group chat for families
- Scheduled check-ins
- Integration with calendar for reminders

## Success Metrics

### Usage Metrics
- Messages per user (average)
- Active chatbot sessions per day
- Conversation length (messages per session)
- Return user rate
- Premium conversion from chatbot usage

### Quality Metrics
- Response time (latency)
- Error rate
- User satisfaction (ratings, future)
- Upgrade rate from free to premium
- Anonymous to authenticated conversion

### Business Metrics
- Cost per message (OpenAI API costs)
- Revenue per premium user
- Monthly recurring revenue from chatbot
- ROI on chatbot feature
- Support ticket reduction

## Cost Analysis

### OpenAI API Costs
- **GPT-3.5-turbo**: ~$0.002 per message
- **GPT-4**: ~$0.03 per message
- **Estimated Monthly Costs** (1000 users, avg 20 messages):
  - GPT-3.5: $40/month
  - GPT-4: $600/month
- **Revenue Offset**: Premium tier at €15 = ~$16.50
  - Break-even: ~2 premium users per month

### Optimization Strategies
- Use GPT-3.5 for free tier
- Reserve GPT-4 for premium tier
- Cache common questions/responses
- Implement smart prompts to reduce messages
- Optimize token usage in prompts

## Monitoring & Maintenance

### Health Checks
- OpenAI API uptime monitoring
- Response time tracking
- Error rate monitoring
- Usage quota alerts
- Cost anomaly detection

### Maintenance Tasks
- Weekly usage pattern review
- Monthly cost analysis
- Quarterly prompt optimization
- Tier limit adjustment as needed
- AI model updates/testing

## Compliance & Ethics

### AI Ethics Guidelines
- Transparent about AI limitations
- Clear disclosure that it's an AI
- No medical/legal advice
- Bias monitoring and mitigation
- User privacy protection
- Content moderation

### Data Handling
- GDPR compliant (EU users)
- CCPA compliant (California users)
- Data minimization principle
- User data deletion on request
- Transparent data usage policy
- No selling of user data

## Accessibility

### Current Implementation
- Keyboard navigation supported
- Screen reader compatible
- High contrast text
- Sufficient color contrast
- Focus indicators
- Alt text for icons

### Future Improvements
- Voice input/output
- Adjustable text size in chat
- Customizable color themes
- Simplified language mode
- Message read-aloud option
- Keyboard shortcuts
