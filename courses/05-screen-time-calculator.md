# Screen Time Calculator Tool - Product Requirements Document

## Product Overview
The AI-Powered Screen Time Analyzer is an interactive tool that helps users understand their digital habits through AI-generated insights. Users input their age, daily screen time, and activity breakdown to receive personalized analysis, concerns, and actionable recommendations powered by AI.

## Current State

### Features Implemented

#### 1. Multi-Step Form Interface
- **Location**: `/tools/screen-time-calculator` route
- **Steps**:
  1. **Input Step**: Collect basic information
  2. **Breakdown Step**: Detailed activity distribution
  3. **Analysis Step**: AI-generated results

#### 2. Basic Information Collection
- **Fields**:
  - Name (optional, for saved analyses)
  - Age (years)
  - Average daily screen time (hours)
- **Features**:
  - Numeric input validation
  - Age range: 0-120 years
  - Screen time range: 0-24 hours
  - Real-time weekly/yearly calculations
  - Visual feedback on time spent

#### 3. Activity Breakdown
- **Categories**:
  - Social Media
  - Streaming/Videos
  - Work/School
  - Gaming
  - Reading/Learning
  - Messaging
- **Interface**:
  - Slider controls (0 to total hours)
  - Numeric input fields
  - Category icons for visual identification
  - Real-time progress bars per category
  - Running total calculation
  - Over-allocation warning

#### 4. AI Analysis Generation
- **Powered By**: OpenAI GPT (via Edge Function)
- **Analysis Components**:
  - **Assessment**: Overall screen time evaluation
  - **Key Concerns**: 3 main issues identified
  - **Recommendations**: Prioritized action items (high/medium/low)
  - **30-Day Impact Prediction**: Expected outcomes if recommendations followed

#### 5. Results Display
- **Visual Design**:
  - Color-coded priority badges
  - Expandable recommendation cards
  - Gradient impact section
  - Print-friendly layout
- **Actions**:
  - Save analysis (authenticated users)
  - Print results
  - Start new analysis
  - Update existing analysis

#### 6. Saved Analyses (Authenticated Users)
- **Storage**: `screen_time_analyses` table in Supabase
- **Features**:
  - History sidebar showing all saved analyses
  - Load previous analysis
  - Update existing analysis
  - Delete saved analysis
  - Cross-device sync
  - Unlimited storage per user

#### 7. Analysis History
- **Display**:
  - Name/title of analysis
  - Age and daily hours
  - Creation date
  - Quick access buttons
- **Management**:
  - View button to load analysis
  - Delete button to remove
  - Chronological ordering (newest first)

## Database Schema

### Screen Time Analyses Table
```sql
CREATE TABLE screen_time_analyses (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  name text,
  age integer NOT NULL,
  daily_hours numeric(4,2) NOT NULL,
  main_activities jsonb NOT NULL,
  usage_times jsonb DEFAULT '[]'::jsonb,
  concerns jsonb DEFAULT '[]'::jsonb,
  analysis_result jsonb NOT NULL,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE INDEX idx_screen_time_user ON screen_time_analyses(user_id);
CREATE INDEX idx_screen_time_created ON screen_time_analyses(created_at DESC);
```

### Analysis Result Structure
```typescript
interface AIAnalysis {
  assessment: string;
  concerns: string[];
  recommendations: Array<{
    title: string;
    description: string;
    priority: 'high' | 'medium' | 'low';
  }>;
  predictedImpact: string;
}
```

## Edge Function: analyze-screen-usage

### Location
`supabase/functions/analyze-screen-usage/index.ts`

### Functionality
- Receives screen time data
- Constructs analysis prompt for AI
- Calls OpenAI API
- Structures response into defined format
- Returns comprehensive analysis

### AI Prompt Structure
```
Analyze this screen time data:
- Age: [age] years
- Daily usage: [hours] hours
- Activity breakdown: [breakdown]

Provide:
1. Assessment: Overall evaluation (2-3 sentences)
2. Concerns: 3 key issues with this usage pattern
3. Recommendations: 5 prioritized actions (with priority level)
4. Predicted Impact: Expected outcomes in 30 days if recommendations followed

Focus on:
- Age-appropriate concerns
- Practical, actionable advice
- Evidence-based recommendations
- Realistic expectations
- Supportive, non-judgmental tone
```

### Response Processing
- Parses AI response into structured format
- Validates all required fields present
- Handles malformed responses gracefully
- Returns JSON with typed structure

## Security Implementation

### Row Level Security
```sql
CREATE POLICY "Users can view own analyses"
  ON screen_time_analyses FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own analyses"
  ON screen_time_analyses FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own analyses"
  ON screen_time_analyses FOR UPDATE
  TO authenticated
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own analyses"
  ON screen_time_analyses FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);
```

### Data Privacy
- All analyses private to creating user
- No public sharing of analyses
- No analytics on user data
- User can delete all data on request

## User Experience Flows

### First-Time User (Unauthenticated)
1. Navigates to `/tools/screen-time-calculator`
2. Sees account creation prompt (non-blocking)
3. Enters age and daily screen time
4. Clicks "Continue to Breakdown"
5. Distributes hours across categories
6. Clicks "Get AI-Powered Analysis"
7. Loading state while AI generates
8. Views comprehensive analysis
9. Donation banner appears
10. Can print results or start new analysis
11. **Cannot save** analysis (prompt to create account)

### Authenticated User
1. Navigates to tool page
2. Sees "My Analyses" button with count
3. Can view analysis history
4. Enters new analysis information
5. Names the analysis
6. Completes breakdown
7. Receives AI analysis
8. Clicks "Save Analysis"
9. Analysis stored in database
10. Can access later from history
11. Can update saved analysis

### Returning User
1. Opens tool page
2. Clicks "My Analyses" button
3. Views list of previous analyses
4. Clicks "View" on one
5. Analysis loads instantly (no new AI call)
6. Can update with new data
7. Saves overwrites previous version
8. Or creates new analysis

## Technical Implementation

### State Management
- **React useState** for form data
- **Multi-step flow** with conditional rendering
- **Optimistic UI** for save operations
- **Local state** for analysis in progress
- **Database state** for saved analyses

### Form Validation
- Age: Required, 0-120
- Screen time: Required, 0-24 hours
- Breakdown: Warning if exceeds total
- Name: Optional, freeform text

### AI Integration
- Async API call to Edge Function
- Loading state during generation
- Error handling with fallback
- Retry logic on failure
- Timeout after 30 seconds

### Data Persistence
- Unauthenticated: Session only (lost on refresh)
- Authenticated: Database storage
- Auto-save on update
- Optimistic updates for better UX

## Content Strategy

### Age-Appropriate Analysis
- **Children (0-12)**: Focus on development, sleep, physical activity
- **Teens (13-17)**: Balance, social relationships, academic performance
- **Adults (18-64)**: Work-life balance, productivity, relationships
- **Seniors (65+)**: Cognitive health, isolation, purposeful use

### Recommendation Categories
1. **Screen-Free Zones**: Bedrooms, dining table, etc.
2. **Screen-Free Times**: Before bed, during meals, etc.
3. **Alternative Activities**: Exercise, hobbies, social connections
4. **Technology Boundaries**: App limits, notifications, website blockers
5. **Behavior Change**: Habit stacking, trigger identification, mindfulness

### Priority Levels
- **High**: Immediate action needed, significant impact
- **Medium**: Important but not urgent, moderate impact
- **Low**: Nice to have, long-term benefits

## Known Limitations

### Current Constraints
1. **No Time-of-Day Analysis**: Doesn't track when usage occurs
2. **No Device Breakdown**: Doesn't differentiate phone/tablet/computer
3. **No Historical Trends**: Single snapshot, no progress over time
4. **No Goals**: Can't set targets or track toward them
5. **No Reminders**: No follow-up prompts to reassess
6. **No Comparative Data**: Doesn't show how user compares to peers
7. **Anonymous Users Can't Save**: Must create account for persistence
8. **Single Language**: AI analysis in interface language only
9. **No Export Options**: Only print, no CSV/PDF download
10. **No Sharing**: Cannot share analysis with family/therapist

### Technical Limitations
- AI generation takes 10-15 seconds
- No caching of similar analyses
- Each analysis costs API tokens
- No offline mode
- Limited to text-based results (no charts)

## Future Enhancements

### Priority 1 (Data Richness)
- Add time-of-day usage tracking
- Device type breakdown
- Contextual usage (work vs leisure)
- Weekend vs weekday patterns
- Historical trend tracking

### Priority 2 (Goal Setting)
- Set reduction targets
- Track progress over time
- Milestone celebrations
- Weekly check-ins
- Habit tracking integration

### Priority 3 (Visualization)
- Charts and graphs of breakdown
- Trend lines over time
- Comparison to recommended limits
- Heat map of usage times
- Visual progress indicators

### Priority 4 (Social & Sharing)
- Family accounts (multi-user)
- Share analysis with family members
- Anonymous community averages
- Export for healthcare providers
- Integration with Apple Screen Time / Digital Wellbeing

## Success Metrics

### Usage Metrics
- Tool completions per day
- Average time to completion
- Abandonment rate by step
- Return usage rate
- Save rate (authenticated users)

### Engagement Metrics
- Analyses per user (average)
- Update frequency
- Historical analysis views
- Print action usage
- Donation conversion from tool

### Quality Metrics
- AI generation success rate
- Analysis relevance (future: ratings)
- Recommendation actionability
- User satisfaction scores
- Perceived value

## Integration Points

### With Authentication
- Save/load requires login
- Usage history per user
- Cross-device sync
- Account upgrade prompts

### With Donation System
- Banner shown after analysis
- Context: "tool_used"
- Non-intrusive placement
- Value-add positioning

### With Dashboard (Future)
- Quick access to recent analyses
- Progress tracking widget
- Goal completion status
- Weekly summary emails

## Cost Analysis

### AI API Costs
- Average cost per analysis: $0.05
- Estimated monthly usage: 500 analyses
- Monthly cost: $25
- Offset by donations (€15 = ~$16.50 per premium user)
- Break-even: 2 premium users per month

### Optimization
- Cache common recommendations
- Reuse analysis for similar inputs
- Batch API calls
- Use smaller model for simple cases

## Monitoring

### Health Checks
- AI API success rate
- Average response time
- Error tracking
- Cost monitoring
- Usage patterns

### Maintenance
- Weekly cost review
- Monthly prompt optimization
- Quarterly accuracy audit
- User feedback incorporation

## Accessibility

### Current
- Keyboard navigation
- Screen reader compatible
- High contrast colors
- Clear labels
- Focus indicators

### Future
- Voice input for data entry
- Results read-aloud
- Simplified mode
- Adjustable text size
- Dyslexia-friendly fonts
