<!-- Markdown with HTML -->
<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://refref.ai/github-readme-header-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://refref.ai/github-readme-header-light.png">
  <img alt="RefRef" src="https://refref.ai/github-readme-header-light.png">
</picture>
</div>

<p align="center">
  <a href='http://makeapullrequest.com'>
    <img alt='PRs Welcome' src='https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=shields'/>
  </a>
  <a href="https://opensource.org/license/agpl-v3/">
    <img src="https://img.shields.io/github/license/refrefhq/refref?logo=opensourceinitiative&logoColor=white&label=License&color=8A2BE2" alt="license">
  </a>
  <br>
  <a href="https://refref.ai/community">
    <img src="https://img.shields.io/badge/discord-7289da.svg?style=flat-square&logo=discord" alt="discord" style="height: 20px;">
  </a>
</p>

<p align="center">
  <a href="https://refref.ai">Website</a> - <a href="https://refref.ai/docs">Docs</a> - <a href="https://refref.ai/community">Community</a> - <a href="https://github.com/refrefhq/refref/issues/new?assignees=&labels=bug&template=bug_report.md">Bug reports</a>
</p>

## Table of Contents

- [🔮 Overview](#-overview)
- [🚀 Getting Started](#-getting-started)
- [✨ Features](#-features)
- [🔰 Tech Stack](#-tech-stack)
- [🤗 Contributing](#-contributing)
- [🎗 License](#-license)

> [!CAUTION]
> RefRef is still in alpha, expect bugs and breaking changes.

## 🔮 Overview

Build powerful referral programs for your products with RefRef's open source referral management platform.

## 🚀 Getting Started

### Quick Start with Docker (Recommended)

Get RefRef running in under a minute:

```bash
# Clone the repository
git clone https://github.com/refrefhq/refref.git
cd refref

# Start everything with Docker Compose
docker-compose up
```

That's it! 🎉 The webapp portal will be available at http://localhost:3000.

Docker Compose automatically handles:

- PostgreSQL database setup
- Database migrations
- Initial data seeding
- Webapp portal configuration

To configure optional services (Google OAuth, email), pass environment variables:

```bash
GOOGLE_CLIENT_ID=xxx GOOGLE_CLIENT_SECRET=xxx RESEND_API_KEY=xxx docker-compose up
```

### Local Development Setup

If you prefer running RefRef locally without Docker:

#### Prerequisites

- Node.js 20+
- pnpm 10.23.0
- PostgreSQL database
- [portless](https://github.com/vercel-labs/portless) (`npm install -g portless`)
- *(Optional)* [Infisical CLI](https://infisical.com) (`winget install Infisical.Infisical` on Windows) to suppress `'infisical' is not recognized` warnings.

> You may see `'infisical' is not recognized`. This is safe to ignore. Scripts try Infisical first, then run without it. Alternatively, install it on Windows via `winget install Infisical.Infisical`.

#### Installation

```bash
# Install dependencies
pnpm install

# Build internal packages (required before any db command)
pnpm --filter "./packages/*" build

# Create the database (skip if it already exists)
createdb -U postgres refref

# Set up environment variables
cp apps/webapp/.env.example apps/webapp/.env

# Edit .env and add your database URL and auth secret
# Generate auth secret with: openssl rand -base64 32

# Export DATABASE_URL for database commands
# (must match DATABASE_URL in apps/webapp/.env)
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/refref"

# Push database schema
pnpm -F @refref/coredb db:push

# (Optional) Seed with template data
pnpm -F @refref/coredb db:seed

# Start portless proxy on port 1355 with plain HTTP (once, portless remembers it)
# (newer portless defaults to HTTPS on port 443, but .env expects http://*.localhost:1355)
portless proxy start -p 1355 --no-tls

# Start development server
pnpm dev
```

<details>
<summary>Windows (PowerShell) equivalents</summary>

```powershell
# Install dependencies
pnpm install

# Build internal packages (required before any db command)
pnpm --filter "./packages/*" build

# Create the database (skip if it already exists)
& "C:\Program Files\PostgreSQL\17\bin\createdb.exe" -U postgres refref

# Set up environment variables
copy apps\webapp\.env.example apps\webapp\.env

# Edit .env and add your database URL and auth secret
# Generate auth secret with: openssl rand -base64 32

# Export DATABASE_URL for database commands
# (must match DATABASE_URL in apps/webapp/.env)
$env:DATABASE_URL="postgresql://postgres:postgres@localhost:5432/refref"

# Push database schema
pnpm -F @refref/coredb db:push

# (Optional) Seed with template data
pnpm -F @refref/coredb db:seed

# Start portless proxy on port 1355 with plain HTTP (once, portless remembers it)
# (newer portless defaults to HTTPS on port 443, but .env expects http://*.localhost:1355)
portless proxy start -p 1355 --no-tls

# Start development server
pnpm dev
```
</details>

Each app gets a stable `.localhost` URL via [portless](https://github.com/vercel-labs/portless):

| App    | URL                                 |
| ------ | ----------------------------------- |
| Webapp | http://refref-webapp.localhost:1355 |
| WWW    | http://refref-www.localhost:1355    |
| API    | http://refref-api.localhost:1355    |
| Refer  | http://refref-refer.localhost:1355  |
| Acme   | http://refref-acme.localhost:1355   |

> Portless is a global CLI tool (`npm install -g portless`). The proxy auto-starts when you run `pnpm dev`. To bypass portless, set `PORTLESS=0 pnpm dev`.

### Environment Variables

#### Required

- `DATABASE_URL` - PostgreSQL connection string (e.g., `postgresql://user:password@localhost:5432/refref`)
- `BETTER_AUTH_SECRET` - Authentication secret key (generate with `openssl rand -base64 32`)

#### Optional

- `GOOGLE_CLIENT_ID` & `GOOGLE_CLIENT_SECRET` - For Google OAuth authentication
- `RESEND_API_KEY` - For sending emails via Resend
- `BETTER_AUTH_URL` - Authentication URL (defaults to http://refref-webapp.localhost:1355)

### Development Commands

```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Run linting
pnpm lint

# Format code
pnpm format

# Type checking
pnpm type:check

# Database commands
pnpm -F @refref/coredb db:push     # Push schema changes
pnpm -F @refref/coredb db:migrate  # Run migrations
pnpm -F @refref/coredb db:studio   # Open Drizzle Studio GUI
pnpm -F @refref/coredb db:seed     # Seed with templates
```

## ✨ Features

- **Referral Attribution**: JS snippet for tracking referrals, enabling accurate attribution of referrals to referrers

- **Customizable Rewards**: Flexible reward system for different referral programs

- **Referrer Portal**: UI components for referrers to refer and track rewards

- **Partner Portal**: Dedicated interface for affiliates

- **Personalized Pages**: Automatic personalization of referral landing pages

- **Nudges**: Automated reminders to boost referral engagement

- **Fraud Monitoring**: Detect and prevent fraudulent referral activity

- **Manual Reward Approval**: Review and approve rewards manually

- **Automatic Reward Approval**: Set rules for automatic reward validation

- **Manual Reward Dispersal**: Control when rewards are sent out

- **Automatic Reward Dispersal**: Schedule automated reward payments

- **Engagement Analytics**: Track referral program performance metrics

- **Testing Environment**: Sandbox for testing referral programs

## 🔰 Tech Stack

- 💻 [Typescript](https://www.typescriptlang.org/)
- 🚀 [React](https://react.dev/)
- ☘️ [Next.js](https://nextjs.org/)
- 🎨 [TailwindCSS](https://tailwindcss.com/)
- 🧑🏼‍🎨 [Shadcn](https://ui.shadcn.com/)
- 🔒 [Better-Auth](https://better-auth.com/)
- 🧘‍♂️ [Zod](https://zod.dev/)
- 🐞 [Vitest](https://vitest.dev/)
- 🗄️ [PostgreSQL](https://www.postgresql.org/)
- 📚 [Fumadocs](https://github.com/fuma-nama/fumadocs)
- 💽 [Drizzle](https://drizzle.dev/)
- 🌀 [Turborepo](https://turbo.build/)

## 🤗 Contributing

Contributions are welcome! Please read the [Contributing Guide][contributing] to get started.

- **💡 [Contributing Guide][contributing]**: Learn about our contribution process and coding standards.
- **🐛 [Report an Issue][issues]**: Found a bug? Let us know!
- **💬 [Start a Discussion][discussions]**: Have ideas or suggestions? We'd love to hear from you.

# 🎗 License

Released under [AGPLv3][license].

<!-- REFERENCE LINKS -->

[contributing]: https://github.com/refrefhq/refref/blob/main/CONTRIBUTING.md
[license]: https://github.com/refrefhq/refref/blob/main/LICENSE
[discussions]: https://discuss.refref.ai
[issues]: https://github.com/refrefhq/refref/issues
[pulls]: https://github.com/refrefhq/refref/pulls "submit a pull request"
