# User Dashboard - Product Requirements Document

## Product Overview
The User Dashboard serves as the central hub for authenticated users, providing an at-a-glance view of their learning progress, saved content, achievements, and quick access to platform features. The dashboard aggregates data from courses, tools, and engagement to present a personalized experience.

## Current State

### Features Implemented

#### 1. Dashboard Overview
- **Location**: `/dashboard` route (requires authentication)
- **Components**:
  - Welcome header with user's name
  - Statistics cards grid
  - Quick access sections
  - Contextual donation banner

#### 2. Statistics Cards
- **Courses In Progress**:
  - Count of enrolled but incomplete courses
  - Icon: BookOpen
  - Color: Primary
  - Links to: `/training`
- **Courses Completed**:
  - Count of fully completed courses
  - Icon: Award
  - Color: Green
  - Links to: `/training`
- **Bookmarks**:
  - Count of saved articles
  - Icon: Bookmark
  - Color: Blue
  - Links to: `/articles`
- **Achievements**:
  - Count of earned achievements
  - Icon: MessageCircle (placeholder)
  - Color: Purple
  - Links to: `#` (not implemented)

#### 3. Courses Section
- **In Progress Courses**:
  - Shows count if any in progress
  - "No courses in progress" state
  - "Continue your learning journey" messaging
  - "Browse Courses" CTA button
- **Features**:
  - Quick navigation to training platform
  - Encouragement to start or continue learning
  - Card-based layout

#### 4. Bookmarks Section
- **Saved Articles**:
  - Shows bookmark count
  - "No bookmarked articles yet" state
  - "You have X saved articles" messaging
  - "Browse Articles" CTA button
- **Features**:
  - Quick access to article library
  - Visual feedback on collection size

#### 5. Conditional Donation Banner
- **Trigger**: Appears when user has completed at least 1 course
- **Context**: "achievement"
- **Placement**: Bottom of dashboard
- **Purpose**: Encourage support after value received
- **Design**: Non-intrusive, contextual

#### 6. User Greeting
- **Personalization**:
  - "Welcome, [Full Name]" if name provided
  - "Welcome, [Email]" if no name
  - Friendly, encouraging tone
  - Subtitle: "My Progress"

## Database Integration

### User Progress Query
```sql
SELECT
  COUNT(DISTINCT CASE WHEN completed = false THEN course_id END) as in_progress,
  COUNT(DISTINCT CASE WHEN completed = true THEN course_id END) as completed,
  COUNT(DISTINCT bookmark_id) as bookmarks,
  COUNT(DISTINCT achievement_id) as achievements
FROM user_stats_view
WHERE user_id = auth.uid();
```

### Data Sources
- `course_enrollments`: Enrolled courses
- `user_progress`: Lesson completion status
- `chat_bookmarks`: Saved articles (future)
- `user_achievements`: Earned achievements (future)

## API Implementation

### Stats Endpoint
- **Method**: `api.user.stats()`
- **Response**:
```typescript
{
  coursesInProgress: number;
  coursesCompleted: number;
  bookmarks: number;
  achievements: number;
}
```
- **Calculation**:
  - In Progress: Enrolled courses with < 100% lesson completion
  - Completed: Enrolled courses with 100% lesson completion
  - Bookmarks: Count of saved articles
  - Achievements: Count of unlocked achievements

## User Experience Flow

### First-Time User (New Account)
1. User creates account
2. Automatically redirected to `/dashboard`
3. Sees all stats at 0
4. Empty state messages displayed
5. CTAs encourage first actions:
   - "Browse Courses" to start learning
   - "Browse Articles" to discover content
6. No donation banner (no completion yet)

### Active Learner
1. User logs in
2. Dashboard shows current progress:
   - 2 courses in progress
   - 1 course completed
   - 5 bookmarked articles
   - 3 achievements
3. Can quickly navigate to continue learning
4. Sees donation banner after first completion
5. Visual progress motivates continued engagement

### Returning User
1. User logs in after break
2. Dashboard reminds of progress:
   - Courses waiting to be continued
   - Bookmarked content to review
   - Achievements to pursue
3. Quick access buttons reduce friction
4. Progress preserved motivates return

## Design System

### Card Layout
- **Grid**: 4 columns on desktop, 2 on tablet, 1 on mobile
- **Styling**:
  - White background
  - Subtle border (secondary-200)
  - Rounded corners (xl)
  - Hover effect (shadow lift)
- **Content**:
  - Icon with colored background circle
  - Large metric number
  - Descriptive label
  - Click target: entire card

### Color Coding
- **Primary** (courses in progress): #667eea
- **Green** (completed): #22c55e
- **Blue** (bookmarks): #3b82f6
- **Purple** (achievements): #a855f7

### Typography
- **Welcome Header**: 4xl, bold
- **Subtitle**: Regular, secondary-600
- **Metric Numbers**: 3xl, bold
- **Labels**: Regular, secondary-600
- **Section Titles**: 2xl, semibold

## Loading States

### Initial Load
1. Dashboard route accessed
2. Loading spinner displayed
3. API call fetches stats
4. Data populates cards
5. Fade-in animation (optional)

### Error Handling
- Network error: "Unable to load dashboard"
- Retry button provided
- Fallback to last known state (if cached)
- Clear error messaging

## Security Implementation

### Route Protection
```typescript
// Dashboard requires authentication
if (!user) {
  return (
    <div className="text-center">
      <p>Please log in to view your dashboard</p>
    </div>
  );
}
```

### Data Privacy
- User can only view own statistics
- No public dashboards
- RLS enforced on all queries
- No sharing or export (yet)

## Integration Points

### With Training Platform
- Course statistics pulled from enrollments
- Progress calculated from lesson completion
- Quick navigation to continue learning
- Completion triggers donation banner

### With Articles System
- Bookmark count displayed (future)
- Quick access to saved articles
- Reading progress tracking (future)

### With Tools
- Tool usage statistics (future)
- Saved analyses count (future)
- Quick access to history (future)

### With Achievements System (Future)
- Achievement count displayed
- Recent achievements highlighted
- Progress toward next achievement
- Celebration animations

## Known Limitations

### Current Constraints
1. **Static Stats**: Only 4 metrics shown
2. **No Trends**: No historical data visualization
3. **No Bookmarks**: Bookmark feature not implemented
4. **No Achievements**: Achievement system not built
5. **No Customization**: Cannot rearrange or hide cards
6. **No Notifications**: No updates or alerts
7. **No Calendar**: No upcoming events
8. **No Activity Feed**: No recent actions log
9. **No Goals**: Cannot set or track goals
10. **No Quick Actions**: No shortcuts to common tasks

### Missing Features
- Recent activity timeline
- Progress charts and graphs
- Streak tracking
- Daily check-in system
- Recommendation widget
- Quick course access
- Tool history
- Community feed
- Personalized tips

## Future Enhancements

### Priority 1 (Data Richness)
- Add charts for progress trends
- Show recent activity feed
- Display current streaks
- Add goal tracking
- Show next recommended action
- Weekly summary widget

### Priority 2 (Personalization)
- Customizable dashboard layout
- Widget selection
- Color theme options
- Data export functionality
- Calendar integration
- Reminders and notifications

### Priority 3 (Engagement)
- Gamification elements
- Achievement showcase
- Leaderboards (optional, private)
- Daily challenges
- Habit tracking
- Milestone celebrations

### Priority 4 (Advanced)
- Social features (friends, sharing)
- Family dashboard view
- Multi-user account support
- Advanced analytics
- AI-powered insights
- Integration with external tools

## Success Metrics

### Usage Metrics
- Dashboard views per day
- Average time on dashboard
- Click-through rate on CTAs
- Return rate to dashboard
- Mobile vs desktop usage

### Engagement Metrics
- Courses started from dashboard
- Articles accessed from dashboard
- Tool usage from dashboard
- Action completion rate
- Weekly active users

### Retention Metrics
- 7-day return rate
- 30-day return rate
- Dashboard utility score
- User satisfaction
- Feature adoption rate

## Technical Implementation

### State Management
- React useState for stats
- useEffect for data fetching
- Error boundaries for failures
- Loading states for async operations

### Data Fetching
```typescript
const fetchStats = async () => {
  try {
    const result = await api.user.stats();
    if (result.data) {
      setStats(result.data);
    }
  } catch (error) {
    console.error('Error fetching stats:', error);
  }
};
```

### Responsive Design
- Mobile-first approach
- Grid adapts to screen size
- Touch-friendly targets
- Readable text at all sizes
- Optimized images and icons

## Performance Considerations

### Optimization
- Lazy load non-critical components
- Cache stats data for 5 minutes
- Minimize API calls
- Efficient database queries
- Indexed columns for fast lookups

### Loading Strategy
- Skeleton screens during load
- Progressive enhancement
- Optimistic UI updates
- Background refresh

## Accessibility

### Current Implementation
- Semantic HTML structure
- ARIA labels on interactive elements
- Keyboard navigation support
- Screen reader compatible
- Sufficient color contrast
- Focus indicators

### Future Improvements
- High contrast mode
- Customizable text size
- Screen reader announcements for stats
- Keyboard shortcuts
- Voice commands (future)

## Internationalization

### Current
- All text uses translation keys
- Supports 5 languages (EN, FR, ES, AR, FA)
- RTL layout for Arabic and Farsi
- Number formatting by locale
- Date formatting by locale

### Future
- More languages
- Regional content variations
- Currency localization
- Time zone handling
- Cultural adaptations

## Monitoring & Analytics

### Tracking
- Page views
- Time on page
- CTA clicks
- Navigation patterns
- Error rates
- API latency

### Insights
- Most used features
- Abandoned sections
- User journey mapping
- Conversion funnels
- Drop-off points

## Compliance

### Data Privacy
- GDPR compliant
- CCPA compliant
- User data minimization
- Transparent data usage
- Export and deletion on request

### Security
- Authentication required
- Session validation
- CSRF protection
- XSS prevention
- Input sanitization
