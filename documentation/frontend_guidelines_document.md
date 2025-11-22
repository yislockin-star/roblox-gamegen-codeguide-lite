# Frontend Guideline Document

This document outlines the frontend architecture, design principles, and technologies used in the **AI-Powered Roblox Game & Asset Generator**. It’s written in everyday language to make the setup clear and easy to follow.

## 1. Frontend Architecture

**Framework and Libraries**
- **Next.js 15 (App Router)**: Our base framework for server-rendered and client-rendered pages. It handles routing, data fetching, and serverless functions.
- **TypeScript**: Adds type safety to JavaScript, helping catch errors early.
- **Tailwind CSS v4**: A utility-first CSS framework that speeds up styling with pre-defined classes.
- **shadcn/ui**: A collection of accessible, customizable React components built on Tailwind.
- **Vercel AI SDK**: Streams AI-generated Lua code from large language models to the frontend.
- **Clerk**: Manages user authentication and sessions.
- **Supabase**: Provides a PostgreSQL database for storing user projects, generated assets, and credit balances.
- **next-themes**: Enables light/dark theme toggling.

**How the Architecture Scales**
- **Modular structure** (`src/app`, `src/components`, `src/lib`): Keeps code organized and easy to grow.
- **Serverless API routes**: Let us add new endpoints (for payments, analytics, etc.) without managing servers.
- **Supabase with Row Level Security**: Scales securely as more users join.
- **Vercel deployment**: Auto-scales on traffic spikes.

## 2. Design Principles

1. **Usability**: Clean, intuitive interfaces that let developers focus on writing prompts and viewing code.
2. **Accessibility**: All UI components from shadcn/ui follow accessibility best practices (keyboard navigation, ARIA attributes).
3. **Responsiveness**: Layouts adapt to different screen sizes—from desktops to tablets.
4. **Consistency**: Uniform styling and component behavior help users predict actions.

**Application in the UI**
- Clear input fields and buttons for prompt entry.
- Real-time streaming display of generated code, with a fixed “Copy” button.
- Light/dark toggle switch in the header for comfortable viewing.

## 3. Styling and Theming

**Styling Approach**
- **Utility-First CSS (Tailwind v4)**: We use Tailwind classes directly in JSX for rapid styling.
- **Component Recipes (shadcn/ui)**: Compose Tailwind classes into higher-level React components for reuse.

**Theming**
- **next-themes** handles light/dark mode.
- Themes are defined in `tailwind.config.js` under `theme.extend.colors` and loaded at runtime.

**Visual Style**
- Modern, flat design with subtle shadows for depth (no heavy gradients).
- Focus on legibility and minimalism to highlight code and inputs.

**Color Palette**
- Primary: Indigo 600 (#4F46E5)
- Secondary: Indigo 500 (#6366F1)
- Success: Emerald 500 (#10B981)
- Warning: Amber 500 (#F59E0B)
- Danger: Red 500 (#EF4444)
- Background Light: Gray 100 (#F3F4F6)
- Background Dark: Gray 900 (#111827)
- Text Light: Gray 800 (#1F2937)
- Text Dark: Gray 100 (#F3F4F6)

**Font**
- **Inter**: A modern, highly legible sans-serif font for code and UI text.

## 4. Component Structure

**Organization**
- `src/app/`: Contains route layouts and pages.
  - `layout.tsx`: Wraps all pages with theme and auth providers.
  - `page.tsx`: Renders the main prompt interface.
  - `api/chat/route.ts`: Serverless function for AI code generation.
- `src/components/`: Houses reusable UI pieces.
  - `ui/`: shadcn/ui button, input, textarea, code block components.
  - `ChatInterface.tsx`: Manages the prompt form and streaming output.
- `src/lib/`: Utility functions and service clients.
  - `supabase.ts`: Initializes Supabase clients.
  - `user.ts`: Fetches Clerk user data.

**Benefits of Component-Based Architecture**
- **Reusability**: Build once, use everywhere (buttons, inputs, code blocks).
- **Maintainability**: Isolate changes to a single component.
- **Testability**: Write tests against small, focused components.

## 5. State Management

**Approach**
- **Local state**: React’s `useState` for form inputs and UI toggles.
- **useChat hook (Vercel AI SDK)**: Manages streaming AI responses and holds conversation state.
- **Clerk Context**: Provides global auth state (user session) via `ClerkProvider`.
- **Supabase client**: Handles server API calls; no global store needed since data is fetched on demand.

**How State Flows**
1. User enters a prompt in `ChatInterface`.
2. `useChat` sends it to `/api/chat` and updates streaming messages.
3. On completion, a database write is triggered server-side; frontend shows a success status.

## 6. Routing and Navigation

**Routing**
- Built-in Next.js App Router (`src/app/`).
- File-based routing: each folder under `src/app` becomes a route.
- `middleware.ts` uses Clerk’s auth middleware to protect routes.

**Navigation Structure**
- **/ (Dashboard)**: Main prompt interface.
- **/projects**: (Future) Gallery of saved assets.
- **/profile**: (Future) User settings and credit balance.

Users navigate via header links or buttons; protected routes redirect to the sign-in page if unauthenticated.

## 7. Performance Optimization

1. **Code Splitting**: Next.js automatically splits code per page; AI streaming logic loads only when needed.
2. **Lazy Loading**: Components like code blocks lazy-load syntax highlighting libraries.
3. **Asset Optimization**: Static assets (icons, images) are compressed and served from Vercel’s CDN.
4. **Streaming Responses**: Shows partial AI output instantly, improving perceived speed.
5. **Production Builds**: Tailwind purges unused CSS in production.
6. **Edge Caching**: API responses can be cached when appropriate.

## 8. Testing and Quality Assurance

**Unit Tests**
- **Jest** + **React Testing Library**: Test component rendering, user interactions, and utility functions (e.g., `supabase.ts`).

**Integration Tests**
- Combine UI and API routes using Jest to verify end-to-end logic in `/api/chat`.

**End-to-End Tests**
- **Playwright**: Simulate a user logging in, entering a prompt, watching streaming code, and saving an asset.

**Linting & Formatting**
- **ESLint** with TypeScript rules enforces code consistency and catches errors.
- **Prettier** formats code automatically on save.

**Continuous Integration**
- GitHub Actions run linting, tests, and build checks on every pull request.

## 9. Conclusion and Overall Frontend Summary

This frontend setup brings together modern tools and best practices to deliver a fast, secure, and scalable AI-powered game generator:
- **Next.js 15** and **TypeScript** for robust page rendering and type safety.
- **Tailwind CSS** and **shadcn/ui** for a consistent, accessible, and responsive design.
- **Clerk** and **Supabase** for secure user management and data storage.
- **Vercel AI SDK** for real-time AI streaming.
- **next-themes** for seamless theming.

All parts work together to let developers log in, type a natural language prompt, and instantly see and save generated Lua code for Roblox. The clear folder structure, component-based approach, and rigorous testing pipeline ensure that new features can be added quickly and safely. This foundation supports future expansion—like project galleries, payment integration, and advanced prompt engineering—making it the ideal starting point for a polished, production-ready platform.