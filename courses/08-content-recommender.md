# Age-Appropriate Content Recommender Tool - Product Requirements Document

## Product Overview
The Age-Appropriate Content Recommender is an AI-powered tool that generates personalized, safe, and educational digital content recommendations for children based on their age, interests, and parental concerns. The system recommends apps, games, TV shows, books, and websites with detailed safety information and parental guidance.

## Current State

### Features Implemented

#### 1. Multi-Step Recommendation Form
- **Location**: `/tools/content-recommender` route
- **Steps**:
  1. **Basic Info**: Child's name and age
  2. **Interests**: Select child's interests
  3. **Content Types**: Choose what to recommend
  4. **Concerns**: Specify parental concerns (optional)

#### 2. Child Information Collection
- **Fields**:
  - Child's name (text input)
  - Child's age (0-18 years)
- **Validation**:
  - Name required
  - Age must be 0-18
  - Clear error states

#### 3. Interest Selection (Multi-Select)
- **Categories**:
  - Science & Nature 🔬
  - Arts & Crafts 🎨
  - Music 🎵
  - Sports & Fitness ⚽
  - Reading & Stories 📚
  - Math & Logic 🔢
  - Coding & Technology 💻
  - History & Culture 🏛️
  - Animals & Pets 🐾
  - Cooking & Food 👨‍🍳
- **Features**:
  - Icon-based selection
  - Multiple selections allowed
  - Visual feedback when selected
  - Minimum 1 interest required

#### 4. Content Type Selection (Multi-Select)
- **Types**:
  - Mobile Apps 📱
  - Video Games 🎮
  - TV Shows & Movies 📺
  - Books & eBooks 📚
  - Educational Websites 🌐
- **Default**: All types selected
- **Features**:
  - Icon-based buttons
  - Toggle on/off
  - At least 1 required

#### 5. Parental Concerns (Optional Multi-Select)
- **Concerns**:
  - Violence or aggressive content
  - In-app purchases & ads
  - Privacy & data collection
  - Social interaction with strangers
  - Addictive game mechanics
  - Inappropriate language or themes
- **Features**:
  - None required (can skip)
  - Multiple selections allowed
  - Influences AI recommendations
  - Filters out concerning content

#### 6. AI-Generated Recommendations
- **Powered By**: OpenAI GPT (via Edge Function)
- **Per Content Item**:
  - **Name**: Title of content
  - **Age Rating**: e.g., "Ages 6-8", "E for Everyone"
  - **Description**: What it is and what it does
  - **Educational Value**: Learning benefits
  - **Parental Guidance**: What parents should know
  - **Pros**: Why it's recommended (3-5 points)
  - **Time Limit**: Suggested daily usage
  - **Link**: Direct URL to content (when available)
- **Additional Sections**:
  - **General Guidelines**: Overall content rules
  - **Parental Tips**: Supervision and engagement advice

#### 7. Recommendation Display
- **Layout**:
  - Grouped by content type
  - Grid view (2 columns on desktop)
  - Card-based design
  - Color-coded information boxes
- **Features**:
  - Expandable cards
  - "Visit Website" buttons with external links
  - Icon indicators for safety
  - Print-friendly layout

#### 8. Saved Recommendations (Authenticated Users)
- **Storage**: `content_recommendations` table
- **Features**:
  - Save recommendations to account
  - View recommendation history
  - Load previous recommendations
  - Delete old recommendations
  - One recommendation saved per search
- **Anonymous Users**:
  - Results displayed but not saved
  - Upgrade prompt to save
  - Results lost on page refresh

#### 9. Recommendation History
- **Display**:
  - Child name and age
  - Creation date
  - Number of content types
  - Quick access buttons
- **Management**:
  - View saved recommendation
  - Delete recommendation
  - Create new search

## Database Schema

### Content Recommendations Table
```sql
CREATE TABLE content_recommendations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  child_name text NOT NULL,
  child_age integer NOT NULL CHECK (child_age >= 0 AND child_age <= 18),
  interests text[] NOT NULL,
  content_types text[] NOT NULL,
  concerns text[] DEFAULT '{}'::text[],
  recommendations jsonb NOT NULL,
  created_at timestamptz DEFAULT now()
);

CREATE INDEX idx_content_recs_user ON content_recommendations(user_id);
CREATE INDEX idx_content_recs_age ON content_recommendations(child_age);
CREATE INDEX idx_content_recs_created ON content_recommendations(created_at DESC);
```

### Recommendations Structure
```typescript
interface ContentRecommendation {
  apps?: ContentItem[];
  games?: ContentItem[];
  shows?: ContentItem[];
  books?: ContentItem[];
  websites?: ContentItem[];
  guidelines: string[];
  parentalTips: string[];
}

interface ContentItem {
  name: string;
  ageRating: string;
  description: string;
  pros: string[];
  educationalValue: string;
  parentalGuidance: string;
  timeLimit?: string;
  link?: string;
}
```

## Edge Function: content-recommendations

### Location
`supabase/functions/content-recommendations/index.ts`

### Functionality
- Receives child profile and preferences
- Constructs detailed AI prompt
- Calls OpenAI API
- Parses and structures response
- Returns categorized recommendations

### AI Prompt Structure
```
Generate age-appropriate content recommendations for:
- Child: [name], Age [age]
- Interests: [list]
- Content types requested: [types]
- Parental concerns: [concerns]

For each selected content type, recommend 3-5 items:
1. Name and age rating
2. Description (2-3 sentences)
3. Educational value
4. Parental guidance notes
5. 3-5 reasons why it's recommended
6. Suggested time limit
7. Official website URL (if known)

Also provide:
- 5-7 general content guidelines
- 5-7 parental engagement tips

Ensure:
- Age-appropriate content only
- Educational value emphasized
- Safety concerns addressed
- Practical, actionable advice
- Specific, named recommendations (not generic)
```

### Response Processing
- Validates all required fields
- Structures data by content type
- Adds safety indicators
- Formats for UI display
- Error handling for incomplete responses

## Security Implementation

### Row Level Security
```sql
CREATE POLICY "Users can view own recommendations"
  ON content_recommendations FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own recommendations"
  ON content_recommendations FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own recommendations"
  ON content_recommendations FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);
```

### Data Privacy
- All recommendations private to user
- Child names not shared publicly
- No tracking of specific content accessed
- User can delete all data

### External Links
- All links marked with external link icon
- `target="_blank"` for safety
- `rel="noopener noreferrer"` for security
- No affiliate links (neutral recommendations)

## User Experience Flows

### Parent Search Flow (Authenticated)
1. Parent navigates to content recommender
2. Enters child's name and age
3. Clicks "Continue" to interests
4. Selects 2-3 interests
5. Proceeds to content types
6. Keeps all types selected (default)
7. Proceeds to concerns
8. Selects "In-app purchases" and "Privacy"
9. Clicks "Get Recommendations"
10. Loading state (15-20 seconds)
11. Views categorized recommendations
12. Expands app recommendations
13. Reads safety information
14. Clicks "Visit Website" for one
15. Returns to recommendations
16. Recommendation automatically saved

### Anonymous Parent Flow
1. Visits tool page
2. Sees account creation prompt
3. Ignores and continues
4. Completes form
5. Receives recommendations
6. Sees "Save Your Recommendations" banner
7. Can view but not save
8. If refreshes, recommendations lost
9. Prompted to create account

### Returning User Flow
1. Opens recommender tool
2. Clicks "History" button
3. Views previous recommendations
4. Selects recommendation for older child
5. Reviews previous results
6. Clicks "New Search" for younger child
7. Completes new search
8. Both recommendations saved

## Content Curation Strategy

### Age Appropriateness
- **0-2 years**: No screen time recommended (AAP guidelines)
- **3-5 years**: Simple, educational, parent-supervised
- **6-8 years**: Structured learning, creative exploration
- **9-12 years**: Independence with guardrails, skill-building
- **13-15 years**: Productive tools, social with safety
- **16-18 years**: College prep, career skills, responsibility

### Educational Value Framework
- **Cognitive Development**: Problem-solving, critical thinking
- **Creativity**: Art, music, storytelling
- **Social Skills**: Cooperation, communication, empathy
- **Physical Activity**: Movement, sports, health
- **Academic Skills**: Math, reading, science, languages
- **Life Skills**: Cooking, money, time management

### Safety Criteria
- **Age Rating**: ESRB, PEGI, Common Sense Media
- **Privacy**: COPPA compliant, no data harvesting
- **Ads**: Minimal or no advertising
- **In-App Purchases**: Clearly disclosed, parent-gated
- **Social Features**: Moderated or disabled
- **Content Moderation**: Active, responsive

## Content Sources

### Apps
- Apple App Store
- Google Play Store
- Amazon Appstore
- Common Sense Media reviews
- Educational app directories

### Games
- Steam (age-filtered)
- Nintendo eShop
- PlayStation Store
- Xbox Store
- Educational game platforms

### Shows/Movies
- Netflix Kids
- Disney+
- PBS Kids
- YouTube Kids (curated)
- Educational streaming services

### Books
- Age-appropriate series
- Award winners (Newbery, Caldecott, etc.)
- Educational publishers
- Popular children's authors
- Digital libraries

### Websites
- Khan Academy Kids
- National Geographic Kids
- NASA Kids
- PBS LearningMedia
- Code.org
- Duolingo
- Educational YouTube channels

## Technical Implementation

### Form Flow
- Multi-step wizard with progress indicator
- Step validation before proceeding
- Back button to revise
- State persistence during session

### AI Integration
- Async API call to Edge Function
- 30-second timeout
- Loading state with progress message
- Error handling with retry
- Structured response parsing

### External Link Handling
```typescript
<a
  href={item.link}
  target="_blank"
  rel="noopener noreferrer"
  className="external-link"
>
  Visit Website
  <ExternalLink className="w-4 h-4" />
</a>
```

## Known Limitations

### Current Constraints
1. **Static Recommendations**: No real-time content database
2. **Link Accuracy**: URLs may become outdated
3. **No Ratings**: Cannot rate recommendations
4. **No Follow-Up**: Cannot mark as tried
5. **Single Child**: One search per session
6. **No Comparison**: Cannot compare recommendations
7. **No Filtering**: Cannot re-filter results
8. **Anonymous Can't Save**: Must create account
9. **No Updates**: Old recommendations not updated
10. **No API Integration**: Not connected to app stores

### Technical Limitations
- AI may hallucinate content names
- Links may break over time
- No real-time pricing information
- No availability checking
- No platform compatibility checking
- Limited to AI knowledge cutoff date

## Future Enhancements

### Priority 1 (Data Quality)
- Integrate with app store APIs
- Real-time content validation
- Pricing information
- Availability checking
- User ratings and reviews
- Automated link validation

### Priority 2 (User Features)
- Multiple children profiles
- "Tried it" tracking
- Favorite recommendations
- Recommendation sharing
- Download as PDF
- Email recommendations

### Priority 3 (Intelligence)
- Content database with structured data
- Personalized based on past searches
- Learning from user feedback
- Community recommendations
- Expert-curated lists
- Seasonal/trending content

### Priority 4 (Advanced)
- Browser extension for safe browsing
- Parental control app integration
- Content filtering service
- Subscription tracking
- Age-up notifications (content grows with child)
- Family dashboard

## Success Metrics

### Usage Metrics
- Recommendations generated per day
- Most searched age ranges
- Popular interest combinations
- Content type preferences
- Save rate (authenticated users)

### Engagement Metrics
- Average interests selected
- Concern selection patterns
- External link clicks
- Recommendation views
- Return search rate

### Quality Metrics
- Parent satisfaction (future: ratings)
- Link validity rate
- Content accuracy
- AI response quality
- Recommendation relevance

## Integration Points

### With Authentication
- Saving requires login
- History per user account
- Family accounts (future)

### With Dashboard (Future)
- Recent recommendations widget
- Quick search shortcut
- Children's profiles

### With Other Tools
- Link from Media Plan Builder
- Cross-reference with Screen Time Calculator

## Cost Analysis

### AI API Costs
- Average cost per recommendation: $0.15
- Estimated monthly usage: 300 searches
- Monthly cost: $45
- Offset by user value and donations

### Optimization
- Cache common age/interest combinations
- Use smaller model for simple requests
- Batch processing
- Rate limiting

## Monitoring

### Health Checks
- AI generation success rate
- Response time tracking
- Link validation monitoring
- Error rate tracking
- User satisfaction tracking

### Maintenance
- Quarterly prompt optimization
- Content source updates
- Link validation sweeps
- Age guideline reviews
- Safety criteria updates

## Compliance & Safety

### Child Safety
- COPPA compliant recommendations
- Age-gated content only
- Privacy-respecting services
- No direct child data collection
- Parental control emphasis

### Content Standards
- Common Sense Media alignment
- AAP screen time guidelines
- Educational research-backed
- Diversity and inclusion
- Accessibility considerations

## Accessibility

### Current
- Keyboard navigation
- Screen reader labels
- High contrast
- Clear focus indicators
- Simple language

### Future
- Text-to-speech for recommendations
- Adjustable text size
- Printable large-print version
- Multi-language support
- Simplified visual mode
