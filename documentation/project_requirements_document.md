# Project Requirements Document (PRD)

## 1. Project Overview

This project is an AI-powered web platform that helps Roblox developers generate game mechanics, scripts, and assets using natural language prompts. Built on a modern Next.js 15 starter kit, the system lets a developer describe what they want—for example, “create a spinning gold coin that gives 10 points on touch”—and instantly returns valid Lua code and asset configurations ready to paste into Roblox Studio.

It’s being built to accelerate Roblox game development, lower the barrier to entry for beginners, and streamline asset creation for seasoned scripters. Success criteria include a smooth, real-time prompt interface, secure per-user project storage, correct and secure Lua code generation, and an intuitive developer setup flow.

---

## 2. In-Scope vs. Out-of-Scope

### In-Scope
- **User Authentication & Accounts:** Secure signup, login, session management via Clerk.
- **Prompt Interface:** UI for typing prompts, viewing streaming AI responses, and copying Lua code.
- **AI Generation Engine:** Server-side endpoint (`/api/chat`) that wraps prompts in a system message, streams LLM responses via Vercel AI SDK, and returns valid Lua scripts and asset metadata.
- **Persistent Storage:** Supabase (PostgreSQL) schema including tables for `projects`, `generated_assets`, and `user_credits` with Row-Level Security policies.
- **Setup Dashboard:** Guide for configuring Clerk keys, Supabase connection, and AI model API credentials.
- **Basic Validation:** Server-side lint or syntax check of generated Lua before saving.
- **Theming & UI:** Light/dark mode toggle with `next-themes`, styled via Tailwind CSS and shadcn/ui components.
- **Code Quality Tooling:** ESLint and Prettier preconfigured.

### Out-of-Scope (Phase 1)
- **Payment Integration:** Stripe or other billing gateways.
- **Image-Based Asset Generation:** Generating textures or 3D models.
- **Mobile App / Native Clients:** Only web browser supported.
- **Rich Role-Based Permissions:** Beyond simple user isolation.
- **Multilingual Prompts:** Only English supported initially.

---

## 3. User Flow

A new developer visits the site, clicks “Sign up,” and is guided through Clerk’s authentication flow (email or social login). Once authenticated, they land on the main dashboard: a left sidebar with links to “Generate Asset,” “My Projects,” and “Settings,” and a central panel where the prompt interface appears. The Settings page lets them paste API keys for the AI model provider and configure Supabase credentials.

On “Generate Asset,” the user types a natural-language prompt into a text area (e.g., “make a part that kills a player on touch”). When they submit, the UI calls the `/api/chat` endpoint. They see the Lua code stream live in a code block with syntax highlighting and a “Copy” button. When the stream completes, the snippet is automatically saved to their Supabase account under a new project entry. The user can then navigate to “My Projects” to view, rename, or delete saved assets.

---

## 4. Core Features

- **Authentication & Authorization:** Clerk for signup/login; middleware secures all protected routes.
- **Prompt Interface:** A text input form, submit button, streaming code display, copy-to-clipboard.
- **AI Generation Engine:** `/api/chat/route.ts` that prepends system prompts, interfaces with the Vercel AI SDK, streams responses.
- **Database Schema & RLS:** Tables for `projects`, `generated_assets`, `user_credits`; RLS ensures users only access their own data.
- **Setup Dashboard:** UI to input API keys for Clerk, Supabase, AI provider; test connection buttons.
- **Theming:** Toggle between light and dark modes using `next-themes`.
- **Code Validation:** Basic Lua syntax check or lint before saving.
- **Streaming UX:** Real-time streaming of AI output for instant feedback.

---

## 5. Tech Stack & Tools

- **Frontend:** Next.js 15 (App Router), TypeScript, Tailwind CSS v4, shadcn/ui components.
- **Backend & API Routes:** Next.js Server components and API Route in `/api/chat`.
- **Authentication:** Clerk for user sessions and middleware.
- **Database:** Supabase (PostgreSQL) with row-level security.
- **AI Integration:** Vercel AI SDK to connect to LLMs (e.g., OpenAI GPT-4).
- **Theming:** `next-themes` for light/dark mode.
- **Linting & Formatting:** ESLint + Prettier.
- **Hosting:** Vercel (Serverless Functions).

---

## 6. Non-Functional Requirements

- **Performance:** Prompt submission to first streaming token under 500 ms; average code streaming chunk every 100 ms.
- **Scalability:** Support concurrent generations; stateless server functions.
- **Security:** All API keys kept server-side; RLS policies enforced; HTTPS everywhere.
- **Usability:** 95% Lighthouse accessibility score; responsive design.
- **Reliability:** 99.9% uptime; graceful error handling on AI or database failures.

---

## 7. Constraints & Assumptions

- **LLM Availability:** Requires continuous access to a streaming-capable model (e.g., OpenAI GPT-4 with streaming API).
- **Hosting Environment:** Deployed to Vercel; uses Vercel AI SDK.
- **Database Limits:** Supabase row limits and bandwidth within free tier; RLS must be configured manually.
- **User Load:** Initial phases assume dozens of active developers, not tens of thousands.
- **Supabase Credentials:** Users will input their own Supabase connection string in Settings.

---

## 8. Known Issues & Potential Pitfalls

- **Hallucinations & Invalid Code:** LLM might produce syntactically incorrect or unsafe Lua. Mitigation: implement server-side lint, incorporate strict system prompts, fallback error messages.
- **API Rate Limits:** Exceeding AI provider quotas. Mitigation: throttle requests, show usage meter in UI, implement retry/backoff.
- **RLS Misconfiguration:** Incorrect policies could leak data. Mitigation: write automated tests for RLS rules, review migration files carefully.
- **Latency Spikes:** Slow AI responses degrade experience. Mitigation: show loading skeletons, inform users of wait times, allow cancellation.
- **Authentication Edge Cases:** Clerks’ session expiration. Mitigation: global error handler to catch auth failures and redirect to login.

---

**This document covers all requirements, flows, and constraints for the first version of the AI-Powered Roblox Game & Asset Generator.** It will serve as the definitive guide for subsequent technical designs, front-end guidelines, and back-end architecture.