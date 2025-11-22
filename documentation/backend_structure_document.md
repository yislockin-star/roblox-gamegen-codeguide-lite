# Backend Structure Document

This document outlines the backend setup for the AI-powered Roblox Game & Asset Generator. It is written in everyday language so anyone can understand how the system works, why certain choices were made, and how all the pieces fit together.

## Backend Architecture

### Overall Design
- We use **Next.js 15** with its App Router to host both our frontend and serverless backend in one codebase.  
- Our code is written in **TypeScript** for type safety and clarity.  
- We follow a **serverless pattern**: each API route is a standalone function that runs on demand. This makes scaling automatic—when traffic goes up, the platform spins up more instances for you.
- **Design Patterns**:  
  • Middleware for route protection (Clerk)  
  • Separation of concerns (UI in `src/app`, business logic in `src/lib`, database migrations in `supabase/migrations`)  
  • Streaming responses from the AI model directly to the client for a smooth user experience

### Scalability, Maintainability, Performance
- **Scalability**: Serverless functions on Vercel auto-scale. Our PostgreSQL database lives on Supabase, which handles database scaling.  
- **Maintainability**: Clear folder structure (`src/app`, `src/components`, `src/lib`, `supabase`). ESLint and Prettier keep code style consistent.  
- **Performance**: Real-time streaming of AI responses, edge caching for static assets, Vercel’s global network for low latency.

## Database Management

### Database Technologies
- **Type**: Relational (SQL)  
- **System**: **Supabase** (managed PostgreSQL)  

### Data Structure and Practices
- We store:  
  • Developer profiles (linked via Clerk user IDs)  
  • Projects and asset metadata  
  • Generated Lua scripts and their prompts  
  • Usage credits for each developer  
- **Row Level Security (RLS)** ensures that each developer only sees their own data.  
- We use **Supabase migrations** (in the `supabase/migrations` folder) to version-control schema changes.  
- Data is accessed via our server-side Supabase client in `src/lib/supabase.ts`, so API keys never hit the browser.

## Database Schema

### Human-Readable Overview

1. **users** (managed by Clerk, not directly in our schema)  
2. **projects**  
   - `id`: unique project identifier  
   - `user_id`: reference to the developer who owns it  
   - `name`: friendly project name  
   - `created_at`, `updated_at` timestamps  
3. **generated_assets**  
   - `id`: unique asset record  
   - `project_id`: which project it belongs to  
   - `asset_type`: e.g., "Script", "Model"  
   - `prompt_text`: the developer’s input prompt  
   - `generated_lua_code`: the AI output  
   - `created_at` timestamp  
4. **user_credits**  
   - `user_id`: reference to developer  
   - `credits_remaining`: integer balance  
   - `updated_at` timestamp

### SQL Definition (PostgreSQL)
```sql
CREATE TABLE projects (
  id             uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id        uuid NOT NULL,
  name           text NOT NULL,
  created_at     timestamp with time zone DEFAULT now(),
  updated_at     timestamp with time zone DEFAULT now()
);

CREATE TABLE generated_assets (
  id                uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id        uuid REFERENCES projects(id) ON DELETE CASCADE,
  asset_type        text NOT NULL,
  prompt_text       text NOT NULL,
  generated_lua_code text NOT NULL,
  created_at        timestamp with time zone DEFAULT now()
);

CREATE TABLE user_credits (
  user_id         uuid PRIMARY KEY,
  credits_remaining integer DEFAULT 0,
  updated_at      timestamp with time zone DEFAULT now()
);

-- RLS Policies to ensure users only access their own rows
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
CREATE POLICY user_can_manage_projects ON projects
  FOR ALL USING (user_id = auth.jwt() ->> 'sub');
ALTER TABLE generated_assets ENABLE ROW LEVEL SECURITY;
CREATE POLICY user_can_manage_assets ON generated_assets
  FOR ALL USING (project_id IN (SELECT id FROM projects WHERE user_id = auth.jwt() ->> 'sub'));
``` 

## API Design and Endpoints

We use **RESTful** API routes under `src/app/api/`. All endpoints require a valid session from Clerk.

- **POST /api/chat**  
  • Purpose: Send a user prompt to the AI model and receive streaming Lua code in response.  
  • Behavior: Prepends a system prompt, calls the Vercel AI SDK, streams partial results back to the client, then saves the final code in `generated_assets`.

- **GET /api/projects**  
  • Returns a list of projects belonging to the authenticated developer.

- **POST /api/projects**  
  • Creates a new project for the user.

- **GET /api/projects/[id]/assets**  
  • Lists all generated assets for a given project, ensuring RLS checks the project’s owner.

- **POST /api/stripe/webhook** (optional for monetization)  
  • Handles incoming Stripe webhook events to top up or deduct credits in `user_credits`.

Each route sits behind Clerk’s `authMiddleware` to ensure only logged-in developers can call them.

## Hosting Solutions

- **Backend & Frontend**: Deployed on **Vercel** (serverless, globally distributed).  
- **Database**: Hosted by **Supabase**, a managed PostgreSQL service.  

Benefits of this pairing:  
- Reliability: Vercel guarantees at least 99.9% uptime for serverless functions. Supabase offers automated backups and failover.  
- Scalability: Vercel scales functions on demand; Supabase scales your database without manual sharding.  
- Cost-Effectiveness: Pay only for what you use. No need for always-on servers.

## Infrastructure Components

- **Load Balancer**: Built into Vercel’s edge network; directs traffic to the nearest serverless instance.  
- **Caching Mechanisms**:  
  • Vercel caches static assets (UI bundles, images) at the edge.  
  • We can optionally cache frequent API responses using Next.js built-in revalidation.  
- **Content Delivery Network (CDN)**: Vercel’s global CDN delivers the site and any static assets from edge locations around the world.  
- **Database Migrations**: Managed via SQL files in `supabase/migrations`. Keeps schema changes in source control.

All these pieces work together to ensure fast load times, smooth API responses, and a reliable experience for developers anywhere.

## Security Measures

- **Authentication & Authorization**:  
  • Clerk handles user signup, login, session management.  
  • API routes are protected with Clerk’s middleware.  
  • Row Level Security in PostgreSQL guarantees data isolation per user.  
- **Data Encryption**:  
  • All traffic is served over HTTPS/TLS.  
  • Secrets (API keys, database URLs) are stored in environment variables on Vercel—never in source code.  
- **Server-Side Logic**: AI calls and database writes happen on the backend, keeping API keys and business logic hidden.  
- **Input Validation & Sanitization**: We validate prompt text and perform basic syntax checks on generated Lua code before saving.

Together, these measures keep developer data safe and help us comply with data-protection best practices.

## Monitoring and Maintenance

- **Performance Monitoring**:  
  • Vercel’s built-in analytics for function execution times, bandwidth usage, and error rates.  
  • Supabase dashboard for database query performance and error logs.  
- **Error Tracking**:  
  • Capture runtime errors in serverless routes. You can integrate a tool like Sentry if desired.  
- **Backups & Migrations**:  
  • Supabase automatically backs up your database daily.  
  • Migrations in version control let you roll forward or backward on schema changes.  
- **CI/CD Pipeline**:  
  • Deploy previews on every Git push (via Vercel).  
  • Optional: GitHub Actions to run tests (Jest, Playwright) before merging.

Regular reviews of logs, performance dashboards, and security audits keep the backend healthy and responsive.

## Conclusion and Overall Backend Summary

Our backend uses a modern, serverless architecture to power an AI-driven Roblox generator. By combining Next.js on Vercel, Supabase’s managed PostgreSQL, and Clerk for authentication, we achieve:  
- **Scalability**: Automatic function scaling and a robust database.  
- **Maintainability**: Clear structure, code quality tools, and version-controlled migrations.  
- **Performance**: Streaming AI responses, global CDN, edge caching.  
- **Security**: End-to-end encryption, RLS, and secret management.

This setup aligns perfectly with the project goal of providing a secure, responsive, and developer-friendly platform for generating Roblox game mechanics and assets via natural language prompts.

By following this document, any new team member or stakeholder can understand and work with the backend infrastructure without needing a deep technical background. If you have questions or need to extend the system, the clear boundaries and patterns laid out here will guide you.