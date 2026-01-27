# Family Media Plan Builder Tool - Product Requirements Document

## Product Overview
The AI Family Media Plan Builder creates personalized, research-based family media plans with professionally designed PDFs that families can print, sign, and post at home. The tool generates comprehensive guidelines, screen-free zones/times, age-based limits, and parental control recommendations using AI.

## Current State

### Features Implemented

#### 1. Multi-Step Wizard Interface
- **Location**: `/tools/media-plan-builder` route
- **Steps**:
  1. **Family Information**: Collect family details
  2. **Preferences**: Screen-free zones, times, and concerns
  3. **Generated Plan**: AI-created comprehensive media plan

#### 2. Family Information Collection
- **Fields**:
  - Family name (e.g., "The Smith Family")
  - Family members:
    - Name
    - Age
    - Role (Parent, Child, Teen)
- **Features**:
  - Add multiple family members dynamically
  - Remove members
  - Validation (all fields required)
  - Role selection dropdown

#### 3. Preference Selection
- **Screen-Free Zones** (Multi-select):
  - Bedrooms (Sleep quality zones)
  - Dining Table (Family meal times)
  - Car (Travel together)
  - Bathroom (Personal hygiene)
- **Screen-Free Times** (Multi-select):
  - During Meals (Family bonding)
  - 1 Hour Before Bed (Better sleep)
  - During Homework (Focus time)
  - Family Time (Quality together)
- **Parental Concerns** (Optional multi-select):
  - Too much screen time
  - Affecting sleep
  - Behavior issues
  - Academic performance
  - Social skills
  - Inappropriate content

#### 4. AI Plan Generation
- **Powered By**: OpenAI GPT (via Edge Function)
- **Generated Components**:
  - **Family Summary**: Overview with member details
  - **Screen-Free Zones**: Selected areas with explanations
  - **Screen-Free Times**: Selected time periods with rationales
  - **Age-Based Limits**: Customized screen time limits per age group
  - **Content Guidelines**: Age-appropriate content rules
  - **Consequences**: Fair, consistent enforcement strategies
  - **Parental Controls**: Tool and app recommendations

#### 5. Professional PDF Generation
- **Design Features**:
  - Full-color gradient header
  - Family branding (name prominently displayed)
  - Icon-based sections
  - Professional typography
  - Print-optimized layout
  - Signature lines for commitment
  - Footer with date and branding
- **Sections**:
  - Family members with roles and ages
  - Screen-free zones grid
  - Screen-free times grid
  - Age-based time limits table
  - Content guidelines checklist
  - Consequences explanation
  - Recommended parental controls list
  - Parent signature and date lines

#### 6. Saved Plans (Authenticated Users)
- **Storage**: `family_media_plans` table
- **Features**:
  - Save plan to account
  - View plan history
  - Load previous plans
  - Update existing plans
  - Delete old plans
  - Cross-device access

#### 7. Plan Management
- **History Sidebar**:
  - Lists all saved plans
  - Shows family name and member count
  - Creation date display
  - Quick view and delete actions
- **Actions**:
  - View saved plan
  - Print/download PDF
  - Update plan
  - Create new plan

## Database Schema

### Family Media Plans Table
```sql
CREATE TABLE family_media_plans (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  family_name text NOT NULL,
  members jsonb NOT NULL,
  screen_free_zones text[] NOT NULL,
  screen_free_times text[] NOT NULL,
  concerns text[] DEFAULT '{}'::text[],
  plan_result jsonb NOT NULL,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE INDEX idx_media_plans_user ON family_media_plans(user_id);
CREATE INDEX idx_media_plans_created ON family_media_plans(created_at DESC);
```

### Plan Result Structure
```typescript
interface MediaPlan {
  familyName: string;
  members: FamilyMember[];
  screenFreeZones: string[];
  screenFreeTimes: string[];
  ageBasedLimits: Record<string, string>;
  contentGuidelines: string[];
  consequences: string;
  parentalControls: string[];
}
```

## Edge Function: analyze-screen-usage

### Location
Same Edge Function used by Screen Time Calculator
`supabase/functions/analyze-screen-usage/index.ts`

### Request Type
`analysisType: 'media_plan'`

### AI Prompt Structure
```
Create a comprehensive family media plan for:
- Family: [family name]
- Members: [names, ages, roles]
- Screen-free zones: [selected zones]
- Screen-free times: [selected times]
- Parental concerns: [selected concerns]

Generate:
1. Age-Based Screen Time Limits: Specific limits for each age group
2. Content Guidelines: 5-7 age-appropriate rules
3. Consequences: Fair, consistent enforcement strategy
4. Parental Controls: 5-7 specific tool/app recommendations

Guidelines:
- Be specific and actionable
- Age-appropriate for all family members
- Evidence-based recommendations
- Practical and realistic
- Supportive, not punitive tone
```

## Security Implementation

### Row Level Security
```sql
CREATE POLICY "Users can view own plans"
  ON family_media_plans FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own plans"
  ON family_media_plans FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own plans"
  ON family_media_plans FOR UPDATE
  TO authenticated
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own plans"
  ON family_media_plans FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);
```

## PDF Generation

### Technology
- Client-side HTML generation
- `window.open()` with print styles
- Print dialog triggered automatically
- No server-side rendering

### PDF Structure
```html
<!DOCTYPE html>
<html>
<head>
  <title>Family Media Plan - [Family Name]</title>
  <style>
    /* Professional print styles */
    - Gradient headers
    - Icon integration
    - Grid layouts
    - Color-coded sections
    - Signature lines
    - Page break controls
  </style>
</head>
<body>
  <div class="container">
    <div class="header">...</div>
    <div class="content">
      <section>Family Members</section>
      <section>Screen-Free Zones</section>
      <section>Screen-Free Times</section>
      <section>Age-Based Limits</section>
      <section>Content Guidelines</section>
      <section>Consequences</section>
      <section>Parental Controls</section>
      <div class="signature-line">...</div>
    </div>
    <div class="footer">...</div>
  </div>
</body>
</html>
```

### Print Optimization
- A4/Letter paper size
- Proper margins for printing
- No fixed backgrounds in print
- Page break controls
- High contrast for readability
- Professional fonts

## User Experience Flows

### First-Time User (Unauthenticated)
1. Navigates to `/tools/media-plan-builder`
2. Sees account prompt (informational)
3. Enters family name
4. Adds family members (minimum 1)
5. Clicks "Continue to Preferences"
6. Selects screen-free zones
7. Selects screen-free times
8. Optionally selects concerns
9. Clicks "Generate AI-Powered Plan"
10. Loading state (10-15 seconds)
11. Views generated plan preview
12. Clicks "Print Family Plan"
13. Print dialog opens with formatted PDF
14. **Cannot save** (prompt to create account)

### Authenticated User
1. Opens tool page
2. Sees "My Plans" button with count
3. Can view plan history
4. Enters family information
5. Completes preference selection
6. Generates plan with AI
7. Reviews plan
8. Clicks "Save Plan"
9. Plan stored in database
10. Clicks "Print Family Plan"
11. Can access later from history

### Family Using Plan
1. Prints PDF
2. Reviews together as family
3. Parents and children sign
4. Posts in visible location (fridge, wall)
5. Refers to plan when conflicts arise
6. Revisits and updates quarterly

## Content Strategy

### Age-Based Screen Time Limits
- **Under 2 years**: No screen time (AAP guidelines)
- **2-5 years**: 1 hour/day, high-quality content
- **6-12 years**: 1-2 hours/day, balanced with activities
- **13-17 years**: 2-3 hours/day recreational, negotiated limits
- **Adults**: Self-determined, work vs leisure balance

### Content Guidelines Examples
- "All content must be age-rated appropriate"
- "No social media for children under 13"
- "Parent approval required for new apps/games"
- "No devices in bedrooms overnight"
- "Educational content prioritized"
- "Co-viewing encouraged for younger children"
- "Regular content audits by parents"

### Consequences Framework
- First offense: Reminder and discussion
- Second offense: 30-minute reduction next day
- Third offense: Device-free day
- Fourth offense: Week-long break
- Focus on natural consequences
- Emphasize learning, not punishment
- Consistent across all family members

### Parental Control Tools
- **iOS**: Screen Time settings
- **Android**: Digital Wellbeing & Family Link
- **Router-Level**: Circle, Firewalla
- **Apps**: Qustodio, Net Nanny, Bark
- **Browser Extensions**: BlockSite, StayFocusd
- **Game Consoles**: Native parental controls
- **Smart TVs**: Viewing time limits

## Technical Implementation

### Form State Management
- React useState for multi-step form
- Dynamic array for family members
- Multi-select checkboxes for zones/times
- Validation on each step
- Progress indicator

### AI Integration
- Async call to Edge Function
- Loading state during generation
- Error handling with retry
- 30-second timeout
- Structured response parsing

### PDF Generation
- HTML template with inline CSS
- Dynamic data injection
- Print-specific media queries
- Window.print() for native print dialog
- No external PDF library needed

## Known Limitations

### Current Constraints
1. **No Editable Fields**: Cannot customize AI-generated text
2. **Single PDF Style**: No theme/color customization
3. **No Digital Signatures**: Signature lines are manual
4. **No Reminders**: No check-in prompts for plan review
5. **No Progress Tracking**: Cannot track adherence to plan
6. **No Plan Comparison**: Cannot see changes between versions
7. **No Sharing**: Cannot share with co-parents digitally
8. **Single Language**: Plan generated in interface language only
9. **No Partial Saves**: Must complete all steps to save
10. **Print Only**: No downloadable PDF file

### Technical Limitations
- Print dialog browser-dependent
- No server-side PDF generation
- No email delivery of plan
- Limited print customization
- No plan templates
- AI takes 10-15 seconds

## Future Enhancements

### Priority 1 (Usability)
- Editable text fields in generated plan
- Multiple PDF themes/styles
- Email delivery of PDF
- Share plan with co-parents
- Downloadable PDF file (not just print)

### Priority 2 (Tracking)
- Adherence tracking system
- Weekly check-in reminders
- Progress reports
- Violation logging
- Success celebration

### Priority 3 (Customization)
- Plan templates by age
- Expert-reviewed plan library
- Community-shared plans
- Custom consequences builder
- Tool-specific guides

### Priority 4 (Integration)
- Calendar integration for screen-free times
- Smart home integration (device lockdown)
- Parental control app partnerships
- Digital signature capture
- Family agreement contract generator

## Success Metrics

### Usage Metrics
- Plans created per day
- Completion rate (start to PDF)
- Save rate (authenticated users)
- Return usage for updates
- Print action rate

### Engagement Metrics
- Average family members per plan
- Common zone/time selections
- Concern patterns
- Plan updates frequency
- Historical plan views

### Quality Metrics
- Plan usefulness (future: ratings)
- PDF download/print rate
- Revision frequency
- User satisfaction
- Recommendation likelihood

## Integration Points

### With Authentication
- Plan saving requires login
- History across devices
- Family account (future)

### With Donation System
- Donation banner after generation
- Context: "tool_used"
- Value proposition clear

### With Other Tools
- Link to Screen Time Calculator
- Link to Content Recommender
- Cross-reference tool usage

## Cost Analysis

### AI API Costs
- Average cost per plan: $0.08
- Estimated monthly usage: 200 plans
- Monthly cost: $16
- Offset by donations
- Break-even: 1 premium user per month

## Monitoring

### Health Checks
- AI generation success rate
- PDF rendering reliability
- Print dialog triggering
- Save success rate
- Error tracking

### Maintenance
- Monthly prompt optimization
- Quarterly content update
- Parental control tool updates
- Age guideline reviews

## Accessibility

### Current
- Keyboard navigation
- Screen reader compatible
- High contrast
- Clear labels
- Focus indicators

### Future
- Accessible PDF
- Audio version of plan
- Large print option
- Simplified language mode
- Multiple format exports
