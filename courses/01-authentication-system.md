# Authentication System - Product Requirements Document

## Product Overview
The CuroSee platform implements a secure email/password authentication system using Supabase Auth, enabling users to create accounts, sign in, and manage their profiles. The system is designed to protect user data while providing a seamless onboarding experience.

## Current State

### Features Implemented

#### 1. User Registration
- **Location**: `/register` route, `src/pages/Register.tsx`
- **Functionality**:
  - Email and password-based registration
  - Full name collection during signup
  - Real-time form validation
  - Password strength requirements
  - Automatic profile creation via database trigger
  - Immediate sign-in after successful registration
  - Redirect to dashboard post-registration

#### 2. User Login
- **Location**: `/login` route, `src/pages/Login.tsx`
- **Functionality**:
  - Email and password authentication
  - Session management via Supabase
  - Remember me functionality through browser storage
  - Error handling for invalid credentials
  - Redirect to original destination after login
  - "Forgot Password" link (UI only, functionality not implemented)

#### 3. User Profile Management
- **Database Schema**: `profiles` table
- **Fields**:
  - `id` (UUID, references auth.users)
  - `email` (text, required)
  - `full_name` (text, optional)
  - `avatar_url` (text, optional)
  - `preferred_language` (text, default 'en')
  - `created_at` (timestamptz)
  - `updated_at` (timestamptz)

#### 4. Authentication Context
- **Location**: `src/contexts/AuthContext.tsx`
- **Provides**:
  - Global authentication state
  - `user` object with user data
  - `loading` state for async operations
  - `signIn()` method
  - `signUp()` method
  - `signOut()` method
  - `updateProfile()` method
  - Session persistence via localStorage
  - Automatic session validation on app load

#### 5. Protected Routes
- **Implementation**: Dashboard and course enrollment require authentication
- **Behavior**:
  - Anonymous users can browse content
  - Authentication modal triggers for protected actions
  - Redirect to original destination after sign-in
  - Preserved state across authentication flow

#### 6. Auth Modal Component
- **Location**: `src/components/AuthModal.tsx`
- **Usage**: Triggered when unauthenticated users attempt protected actions
- **Features**:
  - Context-aware messaging
  - Quick sign-in or registration options
  - Redirect preservation
  - Dismissible overlay

## Security Implementation

### Row Level Security (RLS)
- **Profiles Table**:
  - Users can read their own profile only
  - Users can update their own profile only
  - No public access to user data
  - Enforced at database level

### Password Security
- Handled by Supabase Auth
- Minimum password requirements enforced
- Passwords never stored in plain text
- Secure password reset flow (backend ready)

### Session Management
- JWT-based authentication tokens
- Tokens stored in localStorage
- Automatic token refresh
- Session expiration handling
- Secure token transmission

## Database Triggers

### Automatic Profile Creation
```sql
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.profiles (id, email, full_name)
  VALUES (
    new.id,
    new.email,
    new.raw_user_meta_data->>'full_name'
  );
  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```
- Triggers on new user creation in `auth.users`
- Automatically creates corresponding profile
- Preserves full_name from registration metadata
- Ensures data consistency

## User Experience Flow

### Registration Flow
1. User visits `/register`
2. Enters email, password, and full name
3. Frontend validates input
4. Request sent to Supabase Auth
5. User created in `auth.users`
6. Database trigger creates profile in `profiles`
7. User automatically signed in
8. Redirected to dashboard

### Login Flow
1. User visits `/login`
2. Enters email and password
3. Frontend validates input
4. Credentials verified by Supabase Auth
5. Session token generated
6. Token stored in localStorage
7. User context updated
8. Redirected to original destination or dashboard

### Session Persistence Flow
1. User visits site
2. AuthContext checks localStorage for token
3. Token validated with Supabase
4. If valid, user data loaded
5. Authentication state restored
6. User remains logged in across page refreshes

## Integration Points

### With Course System
- Course enrollment requires authentication
- Progress tracking tied to user ID
- Bookmarks associated with user account
- Achievement tracking per user

### With Tools
- Saved analyses require user account
- History tracking across devices
- Data persistence in user-specific tables
- Cross-device synchronization

### With Chatbot
- Usage limits based on authentication status:
  - Anonymous: 3 messages
  - Authenticated (free): 100 messages/month
  - Premium (donated): 200 messages/month

## Technical Implementation

### Authentication API
- **Library**: `@supabase/supabase-js`
- **Methods Used**:
  - `supabase.auth.signUp()`
  - `supabase.auth.signInWithPassword()`
  - `supabase.auth.signOut()`
  - `supabase.auth.getSession()`
  - `supabase.auth.onAuthStateChange()`

### State Management
- React Context API for global auth state
- Local component state for forms
- localStorage for session persistence
- No external state management libraries

### Error Handling
- Graceful error messages for:
  - Invalid credentials
  - Network errors
  - Duplicate email registration
  - Session expiration
  - Server errors

## Localization Support
- Interface available in 5 languages: EN, FR, ES, AR, FA
- User's preferred language stored in profile
- Automatic language detection on first visit
- Persistent language preference per user
- RTL support for Arabic and Farsi

## Known Limitations

### Not Yet Implemented
1. **Password Reset**: UI exists but functionality not connected
2. **Email Verification**: Registration doesn't require email confirmation
3. **Social Login**: No OAuth providers (Google, Facebook, etc.)
4. **Two-Factor Authentication**: Not available
5. **Profile Avatar Upload**: Field exists but no upload functionality
6. **Account Deletion**: Users cannot delete their accounts
7. **Session Timeout Notifications**: Silent expiration
8. **Password Change**: No in-app password update feature

### Technical Debt
- Auth tokens stored in localStorage (vulnerable to XSS)
- No refresh token rotation
- Limited password complexity requirements
- No rate limiting on login attempts
- Session validation on every page load

## Future Enhancements

### Priority 1 (Security)
- Implement password reset functionality
- Add email verification requirement
- Move tokens to httpOnly cookies
- Implement rate limiting
- Add CAPTCHA for registration

### Priority 2 (User Experience)
- Add social login options (Google, Apple)
- Implement two-factor authentication
- Add profile avatar upload
- Create account management page
- Add password change functionality

### Priority 3 (Features)
- Account deletion workflow
- Session management (view active sessions)
- Login history tracking
- Security notifications
- Account recovery options

## Success Metrics

### Current Metrics (Trackable)
- User registration count
- Login success rate
- Session duration
- User retention rate
- Authentication errors

### User Experience Metrics
- Time to complete registration
- Login abandonment rate
- Authentication error frequency
- Session persistence effectiveness
- Cross-device usage patterns

## Compliance Considerations

### Data Privacy
- GDPR compliance ready (user data exportable)
- Password security meets modern standards
- User data isolated via RLS
- No third-party data sharing
- Transparent data usage

### Accessibility
- Keyboard navigation supported
- Screen reader compatible
- Error messages clearly announced
- Form validation accessible
- High contrast mode compatible
