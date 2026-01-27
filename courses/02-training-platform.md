# Training Platform - Product Requirements Document

## Product Overview
The CuroSee Training Platform provides structured courses on digital wellness, screen time management, and the Indistractable methodology. The system supports multi-language content, progress tracking, and a combination of free preview lessons with authenticated access to full courses.

## Current State

### Features Implemented

#### 1. Course Catalog
- **Location**: `/training` route, `src/pages/Courses.tsx`
- **Features**:
  - Grid layout of all published courses
  - Course difficulty filtering (All, Beginner, Intermediate, Advanced)
  - Localized course titles and descriptions
  - Course metadata display:
    - Lesson count
    - Total duration in minutes
    - Difficulty level badge
    - Thumbnail image
  - Direct link to course detail pages
  - Responsive design (1-3 columns based on viewport)

#### 2. Course Detail Page
- **Location**: `/training/:id` route, `src/pages/CourseDetail.tsx`
- **Layout**: Two-column design
  - Main content area (2/3 width): Lesson viewer
  - Sidebar (1/3 width): Course content navigation
- **Features**:
  - Course metadata header
  - Lesson list with completion status
  - Video placeholder integration
  - HTML content rendering
  - Progress tracking
  - Lesson completion marking
  - Enrollment system

#### 3. Lesson System
- **Content Structure**:
  - Title (5 languages: EN, FR, ES, AR, FA)
  - Rich text content (HTML formatted, 5 languages)
  - Video URL (placeholder, not implemented)
  - Duration in minutes
  - Sequential ordering
  - Preview flag for free lessons

#### 4. Course Enrollment
- **Functionality**:
  - Free lesson preview (Lesson 1) without authentication
  - Registration prompt after free lesson
  - Automatic enrollment for authenticated users
  - Enrollment tracking in `course_enrollments` table
  - Enrollment gates access to full course content
  - One-click enrollment for signed-in users

#### 5. Progress Tracking
- **Database**: `user_progress` table
- **Tracks**:
  - Course ID
  - Lesson ID
  - User ID
  - Completion status (boolean)
  - Completion timestamp
  - Last accessed date
- **Features**:
  - Per-lesson completion marking
  - Visual checkmarks on completed lessons
  - Dashboard statistics integration
  - Persistent across sessions
  - Synced across devices for authenticated users

#### 6. Multi-Language Support
- **Supported Languages**: EN, FR, ES, AR, FA
- **Implementation**:
  - Separate columns for each language in database
  - Runtime language selection
  - Fallback to English if translation missing
  - RTL layout for Arabic and Farsi
  - Language-specific content rendering
  - User preference persistence

## Database Schema

### Courses Table
```sql
CREATE TABLE courses (
  id uuid PRIMARY KEY,
  title_en text NOT NULL,
  title_fr text NOT NULL,
  title_es text NOT NULL,
  title_ar text NOT NULL,
  title_fa text NOT NULL,
  description_en text NOT NULL,
  description_fr text NOT NULL,
  description_es text NOT NULL,
  description_ar text NOT NULL,
  description_fa text NOT NULL,
  thumbnail_url text,
  difficulty text CHECK (difficulty IN ('beginner', 'intermediate', 'advanced')),
  duration_minutes integer DEFAULT 0,
  lessons_count integer DEFAULT 0,
  published boolean DEFAULT false,
  created_at timestamptz DEFAULT now()
);
```

### Lessons Table
```sql
CREATE TABLE lessons (
  id uuid PRIMARY KEY,
  course_id uuid REFERENCES courses(id) ON DELETE CASCADE,
  title_en text NOT NULL,
  title_fr text NOT NULL,
  title_es text NOT NULL,
  title_ar text NOT NULL,
  title_fa text NOT NULL,
  content_en text NOT NULL,
  content_fr text NOT NULL,
  content_es text NOT NULL,
  content_ar text NOT NULL,
  content_fa text NOT NULL,
  video_url text,
  "order" integer NOT NULL,
  duration_minutes integer DEFAULT 0,
  is_preview boolean DEFAULT false,
  created_at timestamptz DEFAULT now()
);
```

### Course Enrollments Table
```sql
CREATE TABLE course_enrollments (
  id uuid PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  course_id uuid REFERENCES courses(id) ON DELETE CASCADE,
  enrolled_at timestamptz DEFAULT now(),
  last_accessed_at timestamptz,
  UNIQUE(user_id, course_id)
);
```

### User Progress Table
```sql
CREATE TABLE user_progress (
  id uuid PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  course_id uuid REFERENCES courses(id) ON DELETE CASCADE,
  lesson_id uuid REFERENCES lessons(id) ON DELETE CASCADE,
  completed boolean DEFAULT false,
  completed_at timestamptz,
  last_accessed_at timestamptz DEFAULT now(),
  UNIQUE(user_id, lesson_id)
);
```

## Security Implementation

### Row Level Security Policies

#### Courses
- **Public Read**: Anyone can view published courses (unauthenticated users)
- **No Write Access**: No direct user modifications
- **Admin Only**: Content management through admin interface (not in this codebase)

#### Lessons
- **Authenticated Read**: Only authenticated users can view lessons
- **Preview Exception**: First lesson can be viewed by anyone if marked as preview
- **Course Constraint**: Can only view lessons of published courses
- **No Write Access**: Lesson content managed by admins only

#### Course Enrollments
- **User Can View**: Users can see their own enrollments
- **User Can Enroll**: Users can create enrollments for themselves
- **No Modification**: Cannot change existing enrollments
- **Ownership Check**: All operations verify user_id matches auth.uid()

#### User Progress
- **User Can View**: Users see only their own progress
- **User Can Update**: Users can mark lessons complete
- **Automatic Upsert**: Progress records created/updated on completion
- **No Deletion**: Progress records cannot be deleted

## User Experience Flows

### Discovery Flow (Unauthenticated User)
1. User lands on home page
2. Clicks "Explore Free Courses" or navigates to `/training`
3. Browses course catalog
4. Filters by difficulty level
5. Clicks on course card
6. Views course overview
7. Clicks "Preview First Lesson Free"
8. Views Lesson 1 content
9. Sees registration prompt to continue
10. Can register or leave

### Enrollment Flow (Authenticated User)
1. Logged-in user browses courses
2. Clicks on course of interest
3. Views course detail page
4. Clicks "Start Course"
5. Automatically enrolled
6. First lesson loads
7. Can navigate through all lessons
8. Marks lessons as complete
9. Progress tracked automatically

### Learning Flow
1. User selects lesson from sidebar
2. Lesson content loads in main area
3. Video section displayed (placeholder)
4. Rich text content rendered below
5. User reads/watches content
6. Clicks "Mark as Complete" at bottom
7. Checkmark appears next to lesson in sidebar
8. Dashboard statistics update
9. Next lesson becomes active

### Progress Tracking Flow
1. User completes lessons in any order
2. Each completion triggers database update
3. Progress synced across all user devices
4. Dashboard shows aggregate statistics:
   - Courses in progress
   - Courses completed
   - Total lessons completed
   - Recent activity
5. Course detail page shows per-lesson status

## Content Management

### Course Creation (Admin Side - Not in UI)
- Courses created directly in database
- All 5 language translations required
- Difficulty level assignment
- Thumbnail URL specification
- Publication toggle
- Metadata calculation (lesson count, duration)

### Lesson Creation (Admin Side - Not in UI)
- Lessons linked to parent course
- Sequential ordering required
- Content in all 5 languages
- HTML formatting supported
- Video URL optional
- Duration estimation
- Preview flag for free lessons

## Technical Implementation

### API Layer
- **Location**: `src/lib/api.ts`
- **Course Methods**:
  - `api.courses.list()` - Fetch all published courses
  - `api.courses.get(id)` - Fetch single course with lessons
  - `api.courses.enroll(courseId)` - Enroll user in course
  - `api.courses.progress(courseId, lessonId)` - Update lesson progress

### Data Fetching
- Supabase client direct queries
- Real-time progress updates
- Optimistic UI updates
- Error boundary implementation
- Loading states for all async operations

### Content Rendering
- `dangerouslySetInnerHTML` for lesson content (HTML)
- Sanitization not implemented (trusted content only)
- Responsive images in content
- Support for embedded media
- Code syntax highlighting (if in content)

### Localization Logic
```typescript
const getLocalizedText = (obj: any, field: string) => {
  const key = `${field}_${i18n.language}`;
  return obj[key] || obj[`${field}_en`] || '';
};
```
- Runtime language detection
- Automatic fallback to English
- Consistent across all components

## Integration Points

### With Authentication System
- Login required for full course access
- Preview lesson available without login
- Registration prompt after preview
- Progress tied to user account
- Enrollment requires user ID

### With Dashboard
- Course statistics displayed
- In-progress courses listed
- Completed courses tracked
- Recent activity feed
- Quick links to continue learning

### With Donation System
- Donation banner on course completion
- Encourages support after value received
- Non-intrusive placement
- Context-aware messaging

## Content Strategy

### Current Course Topics
- Digital wellness fundamentals
- Indistractable methodology
- Screen time management for parents
- Mindful technology use
- Building healthy digital habits
- Content recommendations for families

### Content Structure
- Modular lesson design
- Progressive complexity
- Practical exercises
- Real-world examples
- Actionable takeaways
- Community best practices

### Difficulty Levels
- **Beginner**: Introduction to concepts, basic strategies
- **Intermediate**: Deep dives, implementation tactics
- **Advanced**: Complex scenarios, behavior change psychology

## Known Limitations

### Not Yet Implemented
1. **Video Integration**: Video URLs exist but no player implemented
2. **Downloadable Resources**: No PDF or worksheet downloads
3. **Quizzes/Assessments**: No knowledge verification
4. **Course Certificates**: No completion certificates
5. **Course Ratings**: No user feedback system
6. **Discussion Forums**: No community interaction
7. **Assignments**: No practical homework
8. **Course Recommendations**: No personalized suggestions
9. **Learning Paths**: No guided curriculum
10. **Instructor Profiles**: No course creator information

### Content Limitations
- Limited course catalog (courses must be manually created)
- No course preview trailers
- No sample content before enrollment
- No course syllabus/outline view
- No estimated completion time per lesson
- No content search within courses

### Technical Limitations
- No offline access
- No mobile app
- No content versioning
- No A/B testing framework
- No analytics dashboard for instructors
- No automated progress emails

## Future Enhancements

### Priority 1 (Core Learning)
- Implement video player integration
- Add downloadable course materials
- Create quizzes and knowledge checks
- Issue course completion certificates
- Build course recommendation engine

### Priority 2 (Engagement)
- Add discussion forums per course
- Implement course rating system
- Create assignments and projects
- Add instructor Q&A feature
- Build learning paths and curricula

### Priority 3 (Content & Discovery)
- Course search functionality
- Content filtering by topic
- Bookmark favorite courses
- Course sharing features
- Content tagging system
- Preview videos for courses

### Priority 4 (Advanced Features)
- Live webinar integration
- Course co-creation tools
- Peer review system
- Gamification elements
- Learning analytics dashboard
- Mobile application

## Success Metrics

### User Engagement
- Course enrollment rate
- Lesson completion rate
- Average time per lesson
- Course completion rate
- Return user percentage
- Session duration

### Content Metrics
- Most popular courses
- Most completed courses
- Average course rating (future)
- Completion bottlenecks
- Drop-off points
- Content effectiveness

### Business Metrics
- Free to paid conversion (future)
- User retention by course
- Course completion correlation with donation
- Multi-course enrollment rate
- Language distribution of users

## Accessibility Considerations

### Current Implementation
- Keyboard navigation for lesson selection
- Screen reader compatible lesson list
- Alt text for course thumbnails
- High contrast text
- Readable font sizes
- Focus indicators

### Future Improvements
- Video captions and transcripts
- Audio descriptions for visual content
- Adjustable text size
- Color blind friendly design
- Keyboard shortcuts for navigation
- Screen reader optimized content structure

## Performance Considerations

### Optimization Strategies
- Lazy loading of lesson content
- Efficient database queries with indexes
- Caching of course listings
- Optimized image loading
- Minimal re-renders on progress updates
- Connection pooling for database

### Current Performance
- Fast initial page load
- Smooth lesson transitions
- Responsive progress updates
- Efficient language switching
- Minimal API calls
