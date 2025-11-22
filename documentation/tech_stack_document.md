# Tech Stack Document

Below is a clear, non-technical overview of the main technologies and services used to build the AI-powered Roblox Game & Asset Generator. Each section explains what we chose and why, so anyone can understand how the pieces fit together.

## 1. Frontend Technologies

We built the user interface with modern web tools that make it fast, responsive, and easy to style:

- **Next.js 15 (App Router)**
  • Provides a powerful React framework for building pages and API routes in one place.  
  • Enables server-side rendering and client-side navigation for snappy performance.
- **TypeScript**  
  • Adds clear types (labels) to our JavaScript code so you catch mistakes early.  
  • Makes the code easier to maintain as we grow the project.
- **Tailwind CSS v4 + PostCSS**  
  • A utility-first styling system – write small classes for colors, layouts, and spacing.  
  • PostCSS processes Tailwind and adds browser compatibility.
- **shadcn/ui**  
  • A collection of accessible, customizable components (buttons, inputs, code blocks).  
  • Speeds up building a polished prompt interface and code display.
- **next-themes**  
  • Simple light/dark mode toggling for a comfortable user experience.

Together, these tools let us create a clean, branded dashboard where developers type prompts and see the generated Lua code in real time.

## 2. Backend Technologies

On the server side, we manage user data, handle AI requests, and store generated assets:

- **Next.js API Routes**  
  • Our `/api/chat/route.ts` endpoint talks to the AI model and streams back Roblox-compatible Lua code.  
  • Keeps API keys hidden and lets us enforce business rules (like credit checks) before generation.
- **Clerk**  
  • Manages user sign-up, login, and session security.  
  • Comes with ready-made UI components and middleware to protect dashboard routes.
- **Supabase (PostgreSQL)**  
  • A hosted database for storing user profiles, generated scripts, and project configurations.  
  • Row-Level Security (RLS) ensures each developer only sees their own data.
- **Vercel AI SDK**  
  • Streams requests and responses to and from large language models (like OpenAI or Anthropic).  
  • Powers real-time code generation so developers watch their Lua script being written.

These backend pieces work together to receive a user prompt, ask the AI for code, save the result, and send it back to the user

## 3. Infrastructure and Deployment

To keep development smooth and the service reliable:

- **Vercel**  
  • Hosting platform optimized for Next.js.  
  • Automatic optimizations, edge caching, and global CDN for fast load times.
- **GitHub & Version Control**  
  • All code is tracked in GitHub, making collaboration and rollbacks simple.
- **CI/CD Pipeline (recommended)**  
  • Use GitHub Actions to run tests and deploy to Vercel on every push to main.  
  • Ensures every change is checked and automatically live.
- **Environment Variables**  
  • Stored securely in Vercel (or a `.env` file locally) for API keys and database URLs.  
  • Keeps sensitive credentials out of the public codebase.

These choices give us easy deployments, instant updates, and a stable production environment.

## 4. Third-Party Integrations

We rely on a few specialized services to add key features without reinventing the wheel:

- **Clerk** for user authentication and session handling.
- **Supabase** for a managed PostgreSQL database and simple migration scripts.
- **Vercel AI SDK** (and underlying LLM providers such as OpenAI or Anthropic) for streaming code generation.
- **Optional Payment Provider (e.g., Stripe)**  
  • Can be integrated later to handle credit purchases or subscription billing.

Each integration is chosen to reduce setup time, improve security, and offer a professional user experience.

## 5. Security and Performance Considerations

Keeping your code and data safe while delivering a smooth experience:

- **Authentication & Middleware**  
  • Clerks middleware wraps protected routes so only logged-in developers can access the generator.
- **Row-Level Security (RLS)**  
  • Supabase policies ensure users only read/write their own assets.
- **Server-Side API Calls**  
  • Hides AI model and database credentials from client code.  
  • Allows credit checks or prompt validations before asking the AI for code.
- **Streaming & Incremental Rendering**  
  • The Vercel AI SDK streams responses so the UI updates line by line, reducing perceived wait time.
- **Linting & Formatting**  
  • ESLint and Prettier keep the code style consistent and help catch errors early.

These measures protect user data, safeguard API keys, and ensure the app feels fast.

## 6. Conclusion and Overall Tech Stack Summary

Our stack is centered on a modern Next.js 15 app, with TypeScript for reliability and Tailwind + shadcn/ui for a sleek interface. Clerk and Supabase handle secure users and data storage, while the Vercel AI SDK connects to powerful language models for real-time code generation. Hosting on Vercel plus GitHub-driven CI/CD gives us quick, dependable deployments.

This combination of technologies aligns perfectly with our goal: enabling Roblox developers to describe game ideas naturally and get working Lua scripts and assets almost instantly. The result is a scalable, maintainable, and user-friendly platform that stands out in the AI-powered game development space.