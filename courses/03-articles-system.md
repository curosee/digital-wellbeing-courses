# Articles System - Product Requirements Document

## Product Overview
The CuroSee Articles System provides blog-style content sourced from an external WordPress site, cached locally for performance, and displayed with search and filtering capabilities. The system supports multi-language content and automatic daily synchronization.

## Current State

### Features Implemented

#### 1. Article Listing Page
- **Location**: `/articles` route, `src/pages/Articles.tsx`
- **Features**:
  - Grid display (1-3 columns responsive)
  - Article cards with:
    - Featured image
    - Title
    - Excerpt (preview text)
    - Publication date
    - Author name
    - Categories and tags
  - Search functionality (real-time)
  - Category filtering dropdown
  - Pagination controls
  - Language-specific content filtering
  - Loading states
  - Error handling with retry

#### 2. Article Detail Page
- **Location**: `/articles/:slug` route, `src/pages/ArticleDetail.tsx`
- **Features**:
  - Full article content rendering
  - Featured image display
  - Author information
  - Publication date
  - Category badges
  - Tag display
  - Related articles (future)
  - Social sharing buttons (future)
  - Back navigation
  - Print-friendly layout

#### 3. WordPress Integration
- **Source**: External WordPress REST API
- **Synchronization**: Automated daily sync via Edge Function
- **Content Cached**:
  - Article title, slug, excerpt, content
  - Featured images
  - Author names
  - Categories and tags
  - Publication dates
  - Language metadata

#### 4. Content Caching System
- **Database**: `articles_cache` table
- **Purpose**:
  - Improve performance (no external API calls on each page load)
  - Reduce WordPress server load
  - Enable offline-like browsing
  - Faster search and filtering
  - Consistent availability
- **Update Frequency**: Once per day (configurable)

#### 5. Search Functionality
- **Implementation**: Client-side search in cached data
- **Search Fields**:
  - Article title
  - Article excerpt
  - Article content
  - Author name
  - Tags (future)
- **Features**:
  - Real-time search as user types
  - Case-insensitive matching
  - Debounced for performance
  - Clear search button

#### 6. Category Filtering
- **Dynamic Categories**: Extracted from cached articles
- **Features**:
  - Dropdown selector
  - "All Categories" option
  - Filter combination with search
  - Category counts (future)
  - Alphabetically sorted

#### 7. Pagination
- **Implementation**: Backend pagination via Supabase
- **Configuration**:
  - 12 articles per page
  - Previous/Next buttons
  - Page number display (current / total)
  - Preserved filters across pages
  - URL parameter support (future)

## Database Schema

### Articles Cache Table
```sql
CREATE TABLE articles_cache (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  wp_id integer UNIQUE NOT NULL,
  slug text UNIQUE NOT NULL,
  language text NOT NULL,
  title text NOT NULL,
  excerpt text,
  content text NOT NULL,
  featured_image text,
  author_name text,
  categories jsonb DEFAULT '[]'::jsonb,
  tags jsonb DEFAULT '[]'::jsonb,
  published_at timestamptz NOT NULL,
  cached_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),

  CHECK (language IN ('en', 'fr', 'es', 'ar', 'fa'))
);

CREATE INDEX idx_articles_language ON articles_cache(language);
CREATE INDEX idx_articles_published ON articles_cache(published_at DESC);
CREATE INDEX idx_articles_slug ON articles_cache(slug);
```

### Sync Metadata Table
```sql
CREATE TABLE article_sync_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  sync_started_at timestamptz NOT NULL,
  sync_completed_at timestamptz,
  articles_synced integer DEFAULT 0,
  articles_added integer DEFAULT 0,
  articles_updated integer DEFAULT 0,
  articles_deleted integer DEFAULT 0,
  success boolean DEFAULT true,
  error_message text,
  language text NOT NULL
);
```

## WordPress Integration

### WordPress API Endpoints Used
- `GET /wp-json/wp/v2/posts`
  - Fetches published articles
  - Parameters: per_page, page, _embed
  - Returns: Post data with embedded media and author

### Data Mapping
```typescript
WordPress Post → articles_cache
{
  id → wp_id,
  slug → slug,
  title.rendered → title,
  excerpt.rendered → excerpt,
  content.rendered → content,
  _embedded['wp:featuredmedia'][0].source_url → featured_image,
  _embedded.author[0].name → author_name,
  _embedded['wp:term'][0] → categories,
  _embedded['wp:term'][1] → tags,
  date → published_at
}
```

### Synchronization Logic
- **Trigger**: Scheduled daily (via Supabase Cron)
- **Process**:
  1. Fetch all published posts from WordPress
  2. Compare with existing cached data
  3. Insert new articles
  4. Update modified articles (based on modified date)
  5. Mark orphaned articles as deleted (optional)
  6. Log sync results
- **Error Handling**:
  - Retry logic on WordPress API failures
  - Partial sync recovery
  - Notification to admin on repeated failures

## Edge Function: wordpress-sync

### Location
`supabase/functions/wordpress-sync/index.ts`

### Functionality
- Fetches articles from WordPress REST API
- Transforms data to match database schema
- Performs bulk upsert operations
- Language detection and filtering
- Error logging and reporting

### Invocation
- **Schedule**: Daily at 2:00 AM UTC (pg_cron)
- **Manual**: Admin dashboard trigger (future)
- **Webhook**: WordPress post publish hook (future)

### Performance
- Batch processing (50 posts at a time)
- Parallel API requests
- Efficient database operations
- Timeout handling (10 minute max)

## Security Implementation

### Row Level Security

#### Articles Cache
```sql
CREATE POLICY "Anyone can view published articles"
  ON articles_cache FOR SELECT
  TO anon, authenticated
  USING (published_at <= now());

CREATE POLICY "No direct user modifications"
  ON articles_cache FOR INSERT
  TO authenticated
  USING (false);
```
- Public read access (even unauthenticated)
- No user write access
- Only Edge Function can modify
- Service role authentication for sync function

#### Sync Logs
- Admin-only access
- Used for monitoring and debugging
- Not exposed in public API

## API Implementation

### Secure API Wrapper
- **Location**: `src/lib/secureApi.ts`
- **Method**: `secureApi.getArticles(options)`
- **Parameters**:
  - `page`: Page number (default: 1)
  - `per_page`: Articles per page (default: 12)
  - `language`: Filter by language
  - `search`: Search query string
  - `category`: Filter by category name

### Response Format
```typescript
{
  data: Article[],
  count: number,
  page: number,
  per_page: number,
  total_pages: number
}
```

## User Experience Flows

### Article Discovery Flow
1. User navigates to `/articles`
2. System loads 12 most recent articles in user's language
3. Articles displayed in grid with images and excerpts
4. User can:
   - Scroll through grid
   - Use search box to find specific content
   - Select category from dropdown
   - Navigate to next/previous page
5. Clicks on article card
6. Redirected to `/articles/:slug`

### Article Reading Flow
1. User lands on article detail page
2. Full content rendered with formatting
3. Featured image displayed at top
4. Metadata shown (author, date, categories)
5. Related content suggested (future)
6. User can:
   - Read full article
   - Navigate back to listing
   - Share on social media (future)
   - Bookmark article (future)
   - Print article

### Search Flow
1. User types in search box
2. System filters cached articles in real-time
3. Results update dynamically
4. Search applies across title, excerpt, and content
5. Pagination resets to page 1
6. Category filter remains active
7. "No results" message if no matches

## Content Management

### WordPress Side (External)
- Articles written and published in WordPress
- Categories and tags managed in WordPress
- Featured images uploaded to WordPress
- Multi-language content via WPML or Polylang plugin
- SEO metadata configured per article

### CuroSee Side (Automated)
- Daily sync pulls new/updated content
- No manual intervention required
- Automatic language detection
- Image URLs preserved (linked to WordPress)
- Content formatted and cleaned

## Technical Implementation

### Content Rendering
- HTML content from WordPress displayed as-is
- Uses `dangerouslySetInnerHTML` (trusted content)
- No sanitization applied (WordPress handles it)
- Responsive images via WordPress image sizes
- Embedded media support (YouTube, etc.)

### Language Handling
- Auto-detection from WordPress language metadata
- Filtering by user's current language (i18n)
- Fallback to English if translation not available
- Language switcher affects articles shown

### Performance Optimizations
- All articles cached in Supabase
- No external API calls on page load
- Efficient database queries with indexes
- Lazy loading of images
- Pagination reduces initial load
- Search operates on cached data (fast)

## Integration Points

### With WordPress CMS
- Content originates from WordPress
- Daily synchronization keeps content fresh
- WordPress authors become article authors
- WordPress categories map to CuroSee categories
- WordPress permalinks become CuroSee slugs

### With Authentication (Future)
- Bookmarking requires login
- Commenting requires login
- Article recommendations based on user profile

### With Dashboard (Future)
- Recently read articles
- Bookmarked articles list
- Reading progress tracking

## Known Limitations

### Current Constraints
1. **No Real-Time Sync**: 24-hour delay between WordPress publish and CuroSee display
2. **No Comments**: No commenting system on articles
3. **No Bookmarks**: Cannot save articles for later
4. **No Related Articles**: No recommendations based on categories/tags
5. **No Social Sharing**: Share buttons not functional
6. **No Analytics**: No tracking of article views or engagement
7. **No Content Editing**: Cannot edit cached content without re-sync
8. **No Draft Preview**: Cannot preview unpublished WordPress drafts
9. **No Multi-Author Attribution**: Only first author shown for co-authored posts
10. **No Full-Text Search**: Search is basic substring matching

### Technical Limitations
- Images hosted on WordPress (external dependency)
- No CDN for article images
- No progressive image loading
- Limited to WordPress REST API capabilities
- No content versioning
- No rollback mechanism if bad sync

### WordPress Dependencies
- Requires WordPress site to be online
- WordPress API must be accessible
- Depends on WordPress REST API stability
- Relies on WordPress for content security

## Future Enhancements

### Priority 1 (Engagement)
- Implement article bookmarking
- Add commenting system
- Create related articles algorithm
- Add social sharing buttons
- Track article views and engagement

### Priority 2 (Content)
- Full-text search with Algolia or Postgres FTS
- Author profile pages
- Content recommendations based on reading history
- Newsletter subscription per category
- RSS feed generation

### Priority 3 (Performance)
- Migrate images to CuroSee CDN
- Implement progressive image loading
- Add infinite scroll option
- Create article table of contents
- Reading time estimation

### Priority 4 (Advanced)
- Real-time sync via WordPress webhook
- Multi-source content (other CMSs)
- Content versioning and history
- A/B testing for article headlines
- Advanced analytics dashboard

## Success Metrics

### Content Metrics
- Total articles published
- Articles per language
- Average article length
- Publishing frequency
- Category distribution

### User Engagement
- Article views per day
- Average time on article
- Bounce rate from articles
- Search usage rate
- Category filter usage
- Pagination depth

### Technical Metrics
- Sync success rate
- Sync duration
- Cache hit rate
- Search query latency
- Page load time
- Error rate

## Monitoring & Maintenance

### Health Checks
- Daily sync completion verification
- WordPress API availability monitoring
- Database cache freshness
- Error log review
- Broken image link detection

### Maintenance Tasks
- Weekly sync log review
- Monthly cache cleanup (old articles)
- Quarterly content audit
- Image CDN migration planning
- Performance optimization reviews

## SEO Considerations

### Current Implementation
- Article slugs preserved from WordPress
- Meta titles and descriptions (future)
- Canonical URLs pointing to CuroSee
- Structured data markup (future)
- Sitemap generation (future)

### Future Improvements
- Open Graph tags for social media
- Twitter Card metadata
- JSON-LD structured data
- XML sitemap for articles
- Robots.txt configuration
- Pagination meta tags (rel=prev/next)

## Accessibility

### Current Implementation
- Semantic HTML for articles
- Alt text preserved from WordPress
- Heading hierarchy maintained
- Keyboard navigation for article list
- Focus indicators on interactive elements

### Future Improvements
- Skip links for long articles
- Print stylesheet optimization
- High contrast mode
- Text resize compatibility
- Screen reader testing
- WCAG 2.1 AA compliance audit
