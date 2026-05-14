# 🤖 The Naminator

An AI-powered name combination generator built with Next.js, Auth.js, Prisma, and the Anthropic Claude API.

Enter two names, and Claude will generate creative combinations with "goodness" scores rating how appealing each combination sounds.

## Tech Stack

| Layer          | Technology                              |
| -------------- | --------------------------------------- |
| Framework      | Next.js 16 (App Router)                 |
| Language       | TypeScript                              |
| Authentication | Auth.js v5 (NextAuth) + Google OAuth    |
| Database       | PostgreSQL (Neon)                       |
| ORM            | Prisma 6                                |
| AI             | Anthropic Claude API (claude-sonnet-4-20250514) |
| Styling        | Tailwind CSS 4                          |

## Project Structure

```
naminator/
├── prisma/
│   └── schema.prisma          # Database schema (Auth.js + app models)
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── auth/[...nextauth]/route.ts   # Auth.js route handler
│   │   │   └── name-combinations/route.ts    # POST & GET endpoints
│   │   ├── login/page.tsx     # Login page with Google OAuth
│   │   ├── globals.css        # Tailwind imports
│   │   ├── layout.tsx         # Root layout
│   │   └── page.tsx           # Main page (server component)
│   ├── components/
│   │   ├── Navbar.tsx         # Navigation bar with auth
│   │   ├── NameForm.tsx       # Name input form (client component)
│   │   └── ResultCard.tsx     # Individual result card display
│   ├── lib/
│   │   ├── anthropic.ts       # Claude API client & name generation
│   │   └── prisma.ts          # Prisma client singleton
│   ├── types/
│   │   └── next-auth.d.ts     # Auth.js type extensions
│   ├── auth.ts                # Auth.js configuration
│   └── middleware.ts          # Route protection middleware
├── .env.example               # Environment variable template
├── next.config.ts
├── package.json
└── tsconfig.json
```

## Prerequisites

- **Node.js** 18+ (LTS recommended)
- A **Neon** account (free tier): [neon.tech](https://neon.tech)
- A **Google Cloud** project with OAuth credentials
- An **Anthropic** API key: [console.anthropic.com](https://console.anthropic.com)

## Setup Instructions

### 1. Clone and Install

```bash
git clone <your-repo-url>
cd naminator
npm install
```

### 2. Create Environment File

```bash
cp .env.example .env
```

Then edit `.env` and fill in all values (see below for each service).

### 3. Set Up Neon PostgreSQL

1. Go to [neon.tech](https://neon.tech) and sign up (free).
2. Create a new project.
3. Copy the connection string from the dashboard.
4. Paste it as `DATABASE_URL` in your `.env` file.

### 4. Set Up Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project (or select an existing one).
3. Go to **APIs & Services → Credentials**.
4. Click **Create Credentials → OAuth client ID**.
5. Choose **Web application**.
6. Add an **Authorized redirect URI**: `http://localhost:3000/api/auth/callback/google`
7. Copy the **Client ID** and **Client Secret** into your `.env`:
   - `AUTH_GOOGLE_ID` = Client ID
   - `AUTH_GOOGLE_SECRET` = Client Secret

### 5. Generate Auth Secret

```bash
openssl rand -base64 32
```

Paste the output as `AUTH_SECRET` in your `.env`.

### 6. Get Anthropic API Key

1. Go to [console.anthropic.com](https://console.anthropic.com).
2. Create an API key.
3. Paste it as `ANTHROPIC_API_KEY` in your `.env`.

### 7. Push Database Schema

```bash
npx prisma db push
```

This creates all the tables in your Neon database.

### 8. Generate Prisma Client

```bash
npx prisma generate
```

### 9. Run the Dev Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). You'll be redirected to the login page.

## API Endpoints

### `POST /api/name-combinations`

Generate new name combinations.

**Request:**
```json
{
  "name1": "John",
  "name2": "Jacob"
}
```

**Response:**
```json
{
  "id": 1,
  "name1": "John",
  "name2": "Jacob",
  "createdAt": "2025-02-16T12:00:00.000Z",
  "results": [
    { "id": 1, "name": "Jocob", "goodness": 4.2 },
    { "id": 2, "name": "Jacohn", "goodness": 3.8 },
    { "id": 3, "name": "JohnJacob", "goodness": 2.5 }
  ]
}
```

### `GET /api/name-combinations`

Retrieve all previously generated combinations for the authenticated user.

**Response:** An array of combination sets (same structure as POST response).

## Database Schema

```
NameCombinationSet          GeneratedName
┌──────────────────┐        ┌──────────────────┐
│ id (PK)          │───┐    │ id (PK)          │
│ name1            │   │    │ name             │
│ name2            │   └───>│ goodness         │
│ createdAt        │        │ setId (FK)       │
│ userId (FK)      │        └──────────────────┘
└──────────────────┘
```

## Helpful Commands

| Command              | Description                              |
| -------------------- | ---------------------------------------- |
| `npm run dev`        | Start development server                 |
| `npm run build`      | Build for production                     |
| `npm run db:push`    | Push schema to database (no migrations)  |
| `npm run db:migrate` | Create and apply a migration             |
| `npm run db:studio`  | Open Prisma Studio (visual DB browser)   |

## CI/CD and Environment Variables

### GitHub Actions CI Pipeline

This project uses GitHub Actions for continuous integration. The workflow (`.github/workflows/ci.yml`) runs on every push and pull request to `main`.

**What the CI Pipeline Does:**
1. **Installs dependencies** using `npm ci`
2. **Lints code** with ESLint
3. **Runs tests** with Vitest
4. **Builds the application** with Next.js
5. **Uploads build artifacts** for Node 20

**Matrix Strategy:** Tests run on both Node 20 and Node 22 to ensure compatibility.

**Concurrency:** Only the latest commit's workflow runs; outdated workflows are cancelled to save runner time.

### Setting Up GitHub Secrets

To enable the CI pipeline to access your database:

1. Go to your GitHub repository settings
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **New repository secret**
4. Add `DATABASE_URL` with your test database connection string

The secret is injected into the workflow environment and available to build and test jobs.

### Environment Variables: Three Approaches

GitHub Actions supports three ways to inject environment variables:

1. **Workflow level** — Available to all jobs
   ```yaml
   env:
     DATABASE_URL: ${{ secrets.DATABASE_URL }}
   ```

2. **Job level** — Available to all steps in that job (used in this project)
   ```yaml
   jobs:
     build:
       env:
         DATABASE_URL: ${{ secrets.DATABASE_URL }}
   ```

3. **Step level** — Available only to that step
   ```yaml
   steps:
     - name: Build
       env:
         DATABASE_URL: ${{ secrets.DATABASE_URL }}
       run: npm run build
   ```

This project uses **job level** for clarity and reusability.

### FAQ: .env.local vs .env vs CI

**Q: Why should .env.local never be committed?**

A: `.env.local` contains sensitive credentials (API keys, database passwords) specific to your local machine. Committing it exposes secrets to anyone with repo access and in git history forever. The `.gitignore` prevents accidental commits, but you should never remove this protection.

**Q: Why are GitHub Secrets safer than hardcoding values?**

A: GitHub Secrets are:
- Encrypted at rest and in transit
- Masked in logs (you'll see `***` instead of the actual value)
- Only accessible to authorized workflows
- Rotatable without code changes

Hardcoded values are visible to anyone reading the repository or logs, making them a critical security risk.

**Q: What happens if you hardcode the secret in your YAML?**

Example (DON'T DO THIS):
```yaml
env:
  DATABASE_URL: "postgresql://user:password@localhost:5432/devdb"
```

**Problems:**
- The credential is visible in every git commit and pull request
- CI logs may expose it if the build fails
- Anyone with repo access sees your production database URL
- Rotating the password requires committing code changes
- Audit trails become useless

Use `${{ secrets.DATABASE_URL }}` instead, which masks the value in logs.

**Q: What's the difference between .env and .env.local?**

- **`.env`** — Default environment file; safe to commit (shared team config)
- **`.env.local`** — Local overrides; git-ignored (machine-specific secrets)
- **`.env.example`** — Template for contributors (shows required variables without values)

In CI, environment variables come from GitHub Secrets, not `.env` files.

### Debugging Environment Variable Failures in CI

If your CI build fails with "DATABASE_URL is not defined":

1. **Verify the secret exists** in GitHub Settings → Secrets
2. **Check the secret is spelled correctly** (case-sensitive: `DATABASE_URL`, not `database_url`)
3. **Ensure the job has `env:` block** with `${{ secrets.DATABASE_URL }}`
4. **Look at the logs** — sensitive values are masked with `***`
5. **Don't hardcode values to debug** — redeploy the secret instead

## Troubleshooting

- **"NEXT_REDIRECT" error**: This is normal — Auth.js uses Next.js redirects internally. It's not a bug.
- **Google OAuth not working**: Make sure your redirect URI matches exactly: `http://localhost:3000/api/auth/callback/google`
- **Database connection error**: Verify your `DATABASE_URL` includes `?sslmode=require` for Neon.
- **Prisma client not found**: Run `npx prisma generate` after installing dependencies.
