# WrapVision - AI Car Wrap Visualizer

## Overview

WrapVision is an AI-powered web application that allows users to visualize car wrap colors on their vehicles. Users can upload a photo of their car and preview different wrap colors from premium brands like Nice Wrap USA and Rvinyl. The application uses OpenAI's image generation capabilities to apply wrap colors to uploaded vehicle photos.

Key features:
- Upload and crop car photos
- Browse wrap colors by brand, style, and color family
- Custom color selection via color wheel
- AI-powered wrap visualization using OpenAI image editing
- User authentication via Google Sign-In
- Project saving for authenticated users

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture
- **Framework**: React 18 with TypeScript
- **Routing**: Wouter (lightweight React router)
- **State Management**: TanStack React Query for server state
- **UI Components**: shadcn/ui with Radix UI primitives
- **Styling**: Tailwind CSS with CSS variables for theming
- **Build Tool**: Vite with custom plugins for Replit integration

The frontend follows a page-based structure with reusable components. Pages include landing, gallery (main app), and custom color picker. The app defaults to dark mode.

### Backend Architecture
- **Framework**: Express.js with TypeScript
- **Database ORM**: Drizzle ORM with PostgreSQL
- **Authentication**: Google OAuth 2.0 with Passport.js (passport-google-oauth20)
- **Session Storage**: PostgreSQL-backed sessions via connect-pg-simple
- **API Design**: RESTful endpoints under `/api/` prefix

The server handles:
- User authentication and session management
- Wrap color catalog management
- AI image processing via OpenAI API
- User project CRUD operations

### Data Storage
- **Database**: PostgreSQL (provisioned via Replit)
- **Schema Management**: Drizzle Kit for migrations
- **Tables**:
  - `users` - User profiles from Google Sign-In
  - `sessions` - Session storage for authentication
  - `wrap_colors` - Catalog of available wrap colors
  - `wrap_projects` - User's saved visualization projects
  - `conversations`/`messages` - Chat history (Replit AI integration)

### Authentication
- Uses Google OAuth 2.0 via passport-google-oauth20
- Sessions stored in PostgreSQL for persistence
- Protected routes require authentication middleware
- User data synced from Google profile on login
- Auth files: `server/replit_integrations/auth/googleAuth.ts` (main), `replitAuth.ts` (legacy, unused)
- Login: `/api/login` -> Google OAuth -> `/api/auth/google/callback` -> `/gallery`
- Logout: `/api/logout` -> destroys session -> redirects to `/`

### AI Integration
- **Provider**: OpenAI via Replit AI Integrations
- **Image Processing**: GPT-image-1 model for wrap visualization
- **Environment Variables**: 
  - `AI_INTEGRATIONS_OPENAI_API_KEY`
  - `AI_INTEGRATIONS_OPENAI_BASE_URL`

The wrap application sends base64-encoded car images with color parameters to OpenAI's image editing API, which returns a visualization of the car with the selected wrap color applied.

**Note on AI Limitations**: The gpt-image-1 model used for image editing does not support masking/inpainting. Preservation of non-body elements (wheels, windows, lights) is achieved through detailed prompt engineering. Results may vary depending on image complexity.

## External Dependencies

### APIs and Services
- **OpenAI API** (via Replit AI Integrations): Image generation and editing for wrap visualization
- **Google OAuth 2.0**: User authentication via Google Sign-In
- **PostgreSQL**: Database hosted by Replit

### Key NPM Packages
- `drizzle-orm` / `drizzle-kit`: Database ORM and migrations
- `openai`: OpenAI API client
- `passport` / `passport-google-oauth20`: Authentication
- `express-session` / `connect-pg-simple`: Session management
- `@tanstack/react-query`: Server state management
- `react-image-crop`: Image cropping functionality
- `zod`: Schema validation

### Environment Variables Required
- `DATABASE_URL`: PostgreSQL connection string
- `SESSION_SECRET`: Secret for session encryption
- `GOOGLE_CLIENT_ID`: Google OAuth 2.0 client ID
- `GOOGLE_CLIENT_SECRET`: Google OAuth 2.0 client secret
- `AI_INTEGRATIONS_GEMINI_API_KEY`: Gemini API key (via Replit AI Integrations)
- `AI_INTEGRATIONS_GEMINI_BASE_URL`: Gemini API base URL

### Database Commands
- `npm run db:push`: Push schema changes to database
- Schema defined in `shared/schema.ts`

## Experiments Tried (Do Not Repeat)

### Canvas Post-Processing for Pearlescent (FAILED — reverted)
Attempted client-side HTML5 Canvas overlays to add pearlescent/iridescent shimmer after the AI returns the wrapped image. Two approaches were tried:
1. Hue-shifted gradient overlays computed from the base color — caused wrong colors (e.g. blue car got orange top because +195° hue rotation lands on yellow for blue hues)
2. Color-neutral rainbow shimmer at low opacity via screen blend mode — subtle but affected background too, not convincing enough

Both approaches were reverted. The AI-only approach (prompt engineering) remains in place. If revisiting, consider: server-side image compositing (sharp/canvas npm package with a proper car-body mask), or a completely different AI model that supports inpainting/masking.