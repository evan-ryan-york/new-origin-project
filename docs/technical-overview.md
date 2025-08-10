Technical Overview - The New Origin Project
1. Project Summary
The New Origin Project is a comprehensive educational platform designed for AI-enhanced middle school learning. Built as a modern monorepo, it combines a unified Next.js frontend with a suite of backend microservices, all orchestrated to deliver personalized learning experiences, project-based education, and competency tracking.

2. Guiding Architectural Principles
This platform is built with a few core principles in mind, designed to balance development speed with long-term scalability and maintainability.

Pragmatism over Purity: We favor practical solutions that solve the immediate problem cleanly over architecturally "pure" patterns that add unnecessary complexity for our current scale.

Logical Separation First: We begin with a single database instance but enforce strict logical boundaries using schemas. This gives us the discipline of microservices without the initial operational overhead.

Event-Driven Communication: Services are decoupled and communicate asynchronously via events. This creates a resilient and scalable system where services can evolve independently.

Centralized Identity, Distributed Logic: User identity and authentication are managed by one central service, while all other business logic is distributed across domain-specific services.

3. Architecture Overview
Monorepo Structure
This project uses a monorepo architecture managed by Turborepo and pnpm workspaces, enabling:

Unified Development Experience: Single repository for all services and applications

Code Sharing: Shared types, utilities, and components across services

Coordinated Builds: Parallel and cached builds with dependency awareness

Independent Deployment: Each service can be deployed independently

/new-origin-project
├── apps/
│   └── web/                # Next.js frontend application (acting as API Gateway)
├── packages/
│   ├── services/           # Backend microservices
│   │   └── identity-service/
│   ├── shared-types/       # Shared TypeScript definitions
│   └── ui-components/      # Reusable React components
├── docs/                   # Technical documentation
├── turbo.json              # Turborepo build pipeline
└── pnpm-workspace.yaml     # Workspace configuration

4. Core Technology Stack
Frontend Technologies
Next.js 15 (App Router): React-based full-stack framework with server-side rendering

TypeScript: Type-safe JavaScript for enhanced developer experience

Tailwind CSS v4: Utility-first CSS framework for rapid UI development

Shadcn/UI: High-quality, accessible React component library

TanStack Query: Data fetching and server state management

Zustand: Lightweight global client state management

Backend & Infrastructure
Supabase: Backend-as-a-Service providing:

PostgreSQL database with logical schema separation

Authentication and user management

Real-time subscriptions

File storage

Edge functions for serverless logic

Drizzle ORM: Type-safe SQL query builder for database interactions

Vercel: Deployment platform for the Next.js application

Development Tools
pnpm: Fast, disk-efficient package manager with workspace support

Turborepo: Build system for monorepos with caching and parallelization

ESLint: Code linting and style enforcement

5. Service Architecture
Service Boundaries
The platform is decomposed into the following microservices, grouped by their primary function.

Core Platform Services
Identity & Access Service: Manages user profiles, authentication, authorization, and roles.

Schedule & Orchestration Service: Generates and manages personalized daily schedules for students.

Learning Experience Services
Reflection Engine: Powers AI-driven morning/evening check-ins and goal setting.

Humanities Platform: Manages synchronous seminars and discussions.

AI Tutor Service: Delivers personalized, adaptive learning paths.

Project Hub: Manages the entire Project-Based Learning (PBL) lifecycle.

Workshop Platform: Manages the library of asynchronous skill workshops.

Advisory System: Facilitates 1:1 advisor meetings and parent communication.

Assessment & Progress Services
Competency Tracker: Aggregates evidence from all services to determine student mastery.

Portfolio Service: Manages the curated collection of student work and artifacts.

Support & Infrastructure Services
Notification Service: Handles all system notifications (in-app, email, etc.).

Analytics Service: Collects and analyzes learning and engagement data.

Content Management Service (CMS): A central repository for all curriculum content and rubrics.

AI Orchestration Service: Manages all interactions with external LLMs, prompts, and cost.

Community Service: Manages in-person meetups and events.

Communication Patterns
API Gateway: The Next.js web application serves as the API Gateway, providing a single entry point for all client requests and orchestrating calls to the backend services.

Asynchronous Event-Driven Communication: Services are decoupled and communicate via an event bus. For example, when a project is completed, the Project Hub emits a project.completed event. The Competency Tracker and Notification Service can then independently react to this event. This is initially implemented using Supabase database webhooks or triggers.

Type Contracts: A shared @new-origin/shared-types package ensures that all services and the frontend communicate using consistent, type-safe data structures.

6. Data Architecture
Database Design
Single Database, Multiple Schemas: We use one PostgreSQL instance on Supabase, with logical data isolation enforced by using a separate schema for each service (e.g., identity, schedule, projects).

Row Level Security (RLS): Data access is protected at the database level using Supabase's RLS policies, ensuring users can only access data they are permitted to see.

Type Safety Flow
Database Schema → Supabase introspection defines the source of truth.

Generated Types → The Supabase CLI generates TypeScript types from the schema.

Shared Types Package → These generated types are placed in @new-origin/shared-types for distribution.

Service & Frontend Consumption → All packages import types from the shared package, ensuring end-to-end type safety from the database to the UI.

7. Development Workflows
Local Development
# Install all dependencies across the monorepo
pnpm install

# Start all development servers simultaneously
pnpm dev

# Run dev server for a specific app or package
pnpm dev --filter web
pnpm dev --filter @new-origin/identity-service

Build Pipeline (Turborepo)
The turbo.json file orchestrates the build process, leveraging caching to only rebuild packages that have changed.

// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}

8. Security & Authentication
Authentication Flow: Handled by Supabase Auth, using JWTs for session management. The Next.js middleware protects routes, ensuring only authenticated users can access them.

Security Practices: All secrets are managed via environment variables. SQL injection is prevented by using the Drizzle ORM, and user input is validated using libraries like Zod.

9. Deployment Architecture
Staging: Each pull request automatically generates a preview deployment on Vercel, connected to a dedicated Supabase preview environment.

Production: Merges to the main branch are automatically deployed to the production environment on Vercel and Supabase.

10. Future Considerations
Service Splitting: As the platform scales, services currently living in the monorepo can be extracted into their own independent deployments and databases.

Caching Layer: A dedicated caching layer (e.g., Redis) can be introduced for high-traffic queries and session management.

Observability: A comprehensive logging and monitoring solution will be implemented to track application performance and errors in production.