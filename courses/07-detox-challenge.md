# Digital Detox Challenge Tool - Product Requirements Document

## Product Overview
The Digital Detox Challenge is a goal-tracking system that helps users commit to specific digital wellness goals over configurable time periods (7, 14, 21, or 30 days). The tool features AI-powered daily coaching, progress visualization, streak tracking, and journal-style check-ins to support behavior change.

## Current State

### Features Implemented

#### 1. Challenge Setup Wizard
- **Location**: `/tools/detox-challenge` route
- **Setup Steps**:
  1. **Goal Selection**: Choose preset goal or create custom
  2. **Duration Selection**: Pick challenge length
  3. **Preview**: Review and confirm settings
  4. **Active Challenge**: Daily tracking interface

#### 2. Preset Goals
- **Social Media Reduction**: Limit to 30 min/day
- **Screen-Free Bedtime**: No screens 1 hour before bed
- **Mindful Usage**: Only intentional use, no mindless scrolling
- **Work-Life Balance**: No work emails after 6 PM
- **Tech-Free Meals**: No devices during meals
- **Custom Goal**: User-defined objective

#### 3. Duration Options
- **7-Day Sprint**: Quick start to build momentum
- **2-Week Challenge**: Solid habit foundation
- **21-Day Habit Builder**: Science-backed habit formation
- **30-Day Transformation**: Complete lifestyle change

#### 4. Daily Tracking System
- **Calendar Grid**: Visual display of all challenge days
- **Day States**:
  - Future days (disabled, grayed out)
  - Current day (highlighted border)
  - Past days (available to mark)
  - Completed days (green checkmark)
  - Missed days (red indicator)
- **Interaction**: Click to toggle completion

#### 5. Progress Dashboard
- **Metrics Displayed**:
  - Current day / Total days
  - Overall progress percentage
  - Current streak (consecutive completed days)
- **Visual Elements**:
  - Progress bar with color gradient
  - Large metric cards
  - Icon-based indicators

#### 6. AI Daily Coaching
- **Powered By**: OpenAI GPT (via Edge Function)
- **Coaching Components**:
  - **Encouragement**: Motivational message
  - **Today's Focus**: Daily priority
  - **Strategy**: Specific tactic for the day
  - **Reflection Question**: Thought-provoking prompt
- **Refresh Button**: Generate new coaching anytime
- **Context-Aware**: Considers progress, recent notes, and current day

#### 7. Daily Check-In System
- **Mood Selection**:
  - Great 😊 (green)
  - Good 🙂 (blue)
  - Okay 😐 (yellow)
  - Struggling 😔 (red)
- **Quick Note**: Freeform text field for thoughts
- **Optional**: Can skip check-in
- **Saved**: Persists with challenge data

#### 8. Saved Challenges (Authenticated Users)
- **Storage**: `detox_challenges` table
- **Features**:
  - Multiple active challenges (one at a time)
  - Completed challenge history
  - Load previous challenges
  - Delete old challenges
  - Cross-device sync
- **Anonymous Users**:
  - Single challenge in localStorage
  - Lost if browser data cleared
  - No cross-device sync
  - Upgrade prompt

#### 9. Challenge Completion
- **Completion Criteria**: All days marked complete
- **Celebration Screen**:
  - Achievement badge
  - Completion statistics
  - Streak count
  - Start new challenge CTA
  - Print certificate option
- **Post-Completion**:
  - Challenge marked as complete in database
  - No longer appears as active
  - Moved to history

## Database Schema

### Detox Challenges Table
```sql
CREATE TABLE detox_challenges (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  goal text NOT NULL,
  duration_days integer NOT NULL,
  start_date date NOT NULL,
  completed_days date[] DEFAULT '{}'::date[],
  daily_notes jsonb DEFAULT '{}'::jsonb,
  is_active boolean DEFAULT true,
  completed_at timestamptz,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE INDEX idx_detox_user ON detox_challenges(user_id);
CREATE INDEX idx_detox_active ON detox_challenges(is_active);
CREATE INDEX idx_detox_created ON detox_challenges(created_at DESC);
```

### Daily Notes Structure
```typescript
interface DailyNote {
  [date: string]: {
    feeling: 'great' | 'good' | 'okay' | 'struggling';
    note: string;
  };
}
```

### AI Coaching Structure
```typescript
interface AICoaching {
  encouragement: string;
  todaysFocus: string;
  strategy: string;
  reflection: string;
}
```

## Edge Function: analyze-screen-usage

### Request Type
`analysisType: 'detox_coaching'`

### AI Prompt Structure
```
Provide daily coaching for a digital detox challenge:
- Current day: [day] of [total]
- Goal: [user's goal]
- Completed days: [count]
- Recent mood: [last 3 check-ins]

Generate:
1. Encouragement: Motivational message (2 sentences)
2. Today's Focus: Main priority for today (1 sentence)
3. Strategy: Specific tactic to use today (2 sentences)
4. Reflection: Thought-provoking question

Tone:
- Supportive and empathetic
- Specific to their goal
- Realistic and actionable
- Progress-aware
- Non-judgmental
```

## Security Implementation

### Row Level Security
```sql
CREATE POLICY "Users can view own challenges"
  ON detox_challenges FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own challenges"
  ON detox_challenges FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own challenges"
  ON detox_challenges FOR UPDATE
  TO authenticated
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own challenges"
  ON detox_challenges FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);
```

### Anonymous User Storage
- localStorage key: `detoxChallenge`
- JSON serialized challenge object
- Cleared on browser cache clear
- No server backup
- Account creation prompted

## User Experience Flows

### First-Time Setup (Authenticated)
1. User navigates to detox challenge tool
2. Views goal selection screen
3. Selects preset or creates custom goal
4. Reviews goal tips and strategies
5. Clicks "Continue to Duration"
6. Selects 7, 14, 21, or 30 days
7. Views preview with goal and duration
8. Clicks "Start My Challenge"
9. Challenge created in database
10. Active tracking interface loads
11. Day 1 highlighted
12. AI coaching generated automatically

### Daily Engagement Flow
1. User opens challenge page
2. Views current progress metrics
3. Sees today highlighted on calendar
4. Reads AI coaching (if generated)
5. Completes daily goal
6. Clicks to mark day complete
7. Green checkmark appears
8. Streak counter updates
9. Optional: Fills out check-in
10. Selects mood emoji
11. Writes quick note
12. Clicks "Save Check-In"

### Completion Flow
1. User marks final day complete
2. Completion banner appears
3. Celebration screen displayed
4. Statistics shown:
   - Total days completed
   - Final streak
   - Completion date
5. Options presented:
   - Start new challenge
   - Print certificate
   - View achievement
6. Challenge marked inactive
7. Moved to history

### Challenge Recovery Flow
1. User misses a day
2. Day remains unmarked (red indicator)
3. Streak resets to 0
4. AI coaching adapts with encouragement
5. User can still complete challenge
6. Missing days don't block future days
7. Completion requires 100% regardless

## Challenge Design

### Preset Goal Design
Each preset includes:
- **Icon**: Visual identifier
- **Title**: Clear goal name
- **Description**: Specific commitment
- **Tips**: 3 strategy suggestions
- **Why it matters**: Benefit explanation

### Duration Rationale
- **7 Days**: Quick win, builds confidence
- **14 Days**: Habit starts to form
- **21 Days**: Classic habit formation period
- **30 Days**: Full lifestyle integration

### Streak Psychology
- **Motivation**: Don't break the chain
- **Visual**: Green checkmarks create satisfaction
- **Recovery**: Can rebuild streak after miss
- **Celebration**: Milestones at 7, 14, 21, 30 days

## AI Coaching Strategy

### Coaching Principles
1. **Personalized**: References user's specific goal
2. **Progress-Aware**: Adjusts based on how far along
3. **Mood-Responsive**: Considers recent check-ins
4. **Actionable**: Provides specific tactics
5. **Encouraging**: Always supportive, never critical
6. **Science-Based**: Grounded in behavior change research

### Coaching Frequency
- **Day 1**: Automatic generation
- **Daily**: User can request refresh
- **Completion**: Special congratulatory message
- **Struggle Days**: Extra encouragement if mood low

### Example Coaching
```
Encouragement:
"You're on Day 3 of your social media reduction challenge—already building momentum! Remember, you're not just cutting back; you're reclaiming time for what truly matters."

Today's Focus:
"Notice what triggers your urge to scroll and try replacing it with a 5-minute walk or stretch."

Strategy:
"Before opening social media, ask yourself: 'What am I actually looking for right now?' This simple pause can interrupt the autopilot habit and help you choose consciously."

Reflection:
"What would you do with an extra 30 minutes today if social media wasn't an option?"
```

## Technical Implementation

### State Management
- React useState for challenge state
- Date-fns for date calculations
- localStorage for anonymous users
- Database for authenticated users
- Optimistic UI updates

### Calendar Logic
```typescript
const calculateCurrentDay = () => {
  const start = new Date(challenge.startDate);
  const today = new Date();
  const daysPassed = differenceInDays(today, start) + 1;
  return Math.min(daysPassed, challenge.duration);
};
```

### Streak Calculation
```typescript
const calculateStreak = () => {
  let count = 0;
  for (let i = duration - 1; i >= 0; i--) {
    const dayDate = addDays(startDate, i);
    if (completedDays.includes(format(dayDate, 'yyyy-MM-dd'))) {
      count++;
    } else {
      break;
    }
  }
  return count;
};
```

## Integration Points

### With Authentication
- Account required for persistence
- Prominent upgrade prompts for anonymous
- Cross-device sync for authenticated
- Challenge history per user

### With Dashboard (Future)
- Active challenge widget
- Quick check-in from dashboard
- Progress at a glance
- Milestone notifications

### With Donation System
- Donation banner after first completion
- Premium features for donors (future)
- Support acknowledgment

## Known Limitations

### Current Constraints
1. **One Active Challenge**: Cannot run multiple simultaneously
2. **Manual Tracking**: No automatic verification
3. **No Reminders**: No push notifications for check-ins
4. **No Social Features**: Cannot share or compete
5. **No Custom Durations**: Limited to 4 preset options
6. **No Pause Function**: Cannot pause and resume
7. **Simple Calendar**: Cannot batch-mark days
8. **No Milestones**: No celebrations at 7, 14, 21 days
9. **Anonymous Limitations**: No persistence without account
10. **No Certificate Export**: Print only, no PDF download

### Technical Limitations
- localStorage can be cleared
- No offline mode for AI coaching
- AI coaching takes 10-15 seconds
- No push notifications
- No calendar integrations
- Limited data visualization

## Future Enhancements

### Priority 1 (Engagement)
- Daily reminder notifications
- Milestone celebrations (7, 14, 21 days)
- Printable/shareable certificate
- Progress charts and graphs
- Habit stacking suggestions

### Priority 2 (Social)
- Share progress on social media
- Challenge friends
- Community leaderboards
- Accountability partners
- Group challenges

### Priority 3 (Intelligence)
- Automatic verification (device usage tracking)
- Smart reminders based on user patterns
- Predictive analytics for success
- Personalized goal recommendations
- Integration with Apple Screen Time / Digital Wellbeing

### Priority 4 (Advanced)
- Multiple simultaneous challenges
- Custom challenge durations
- Challenge templates library
- Coaching call scheduling
- Integration with wearables
- Behavioral insights dashboard

## Success Metrics

### Challenge Metrics
- Challenges started per day
- Completion rate
- Average duration chosen
- Most popular goals
- Active challenges at any time

### Engagement Metrics
- Daily check-in rate
- AI coaching request frequency
- Streak lengths (average)
- Return challenge rate
- Multi-challenge users

### Behavior Change Metrics
- Before/after surveys (future)
- Reported habit change
- User testimonials
- Long-term tracking (3+ months)
- Relapse and recovery patterns

## Cost Analysis

### AI API Costs
- Average cost per coaching: $0.03
- Estimated daily coaching requests: 500
- Monthly cost: $450
- Offset by premium users
- Optimization: Cache common coaching

## Monitoring

### Health Checks
- AI coaching success rate
- Database save reliability
- localStorage persistence
- Streak calculation accuracy
- Date handling edge cases

### Maintenance
- Weekly user feedback review
- Monthly coaching prompt optimization
- Quarterly goal library expansion
- Behavior change research updates

## Behavioral Science Foundation

### Habit Formation Principles
1. **Clear Trigger**: Daily calendar check
2. **Simple Behavior**: Binary complete/incomplete
3. **Immediate Reward**: Checkmark satisfaction
4. **Social Proof**: Community (future)
5. **Identity Shift**: "I am someone who..."

### Behavior Change Techniques
- **Goal Setting**: Specific, measurable commitments
- **Self-Monitoring**: Daily tracking
- **Feedback**: Progress metrics
- **Social Support**: AI coaching as proxy
- **Relapse Prevention**: Restart capability

### Psychology of Streaks
- **Loss Aversion**: Don't want to break streak
- **Visual Progress**: Satisfying to see chain
- **Momentum**: Easier to continue than start
- **Forgiveness**: Can recover from misses

## Accessibility

### Current
- Keyboard navigation for calendar
- Screen reader labels for all controls
- High contrast day indicators
- Clear visual states
- Focus management

### Future
- Voice input for check-ins
- Audio coaching option
- Haptic feedback for completion
- Large print mode
- Simplified visual design
- Color-blind friendly indicators
