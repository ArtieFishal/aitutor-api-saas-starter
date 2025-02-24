# AI Tutor API Subscription Starter

Use workflows and chatbots created with [AI Tutor API](https://aitutor-api.vercel.app/).  
This template is based on [Vercel SAAS Starter](https://github.com/nextjs/saas-starter).

## Overview

The **AI Tutor API Subscription Starter** is your complete starting point to build SaaS applications that integrate AI workflows and chatbots powered by AI Tutor API. This template leverages the powerful foundation of Vercel SAAS Starter and extends it with Stripe-based subscription management, PostgreSQL (using Drizzle ORM), and per‑month message tracking with limits based on subscription tiers.

## Tech Stack

- **Framework:** Next.js (experimental App Router)
- **Database:** PostgreSQL, powered by [Drizzle ORM](https://orm.drizzle.team/)
- **Payments:** Stripe (integrated with Stripe CLI for local testing)
- **AI Service:** [AI Tutor API](https://aitutor-api.vercel.app/)
- **Authentication:** JWT-based authentication stored in cookies
- **UI Components:** shadcn/ui and custom components
- **Deployment:** Vercel

## Getting Started

### Installation

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/your-username/ai-tutor-api-saas-starter.git
   cd ai-tutor-api-saas-starter
   ```

2. **Install Dependencies:**

   ```bash
   pnpm install
   ```

3. **Environment Setup:**

   - Copy the example environment file and rename it to `.env`:

     ```bash
     cp .env.example .env
     ```

   - Update the following keys in your `.env` file:
     - `POSTGRES_URL` – Your database URL.
     - `STRIPE_SECRET_KEY` – Your Stripe secret key.
     - `STRIPE_WEBHOOK_SECRET` – Your Stripe webhook secret.
     - `BASE_URL` – Your application’s base URL (e.g., `http://localhost:3000`).
     - `AUTH_SECRET` – A secret key for JWT signing (e.g. generated with `openssl rand -base64 32`).
     - `AITUTOR_API_KEY` – Your API key for AI Tutor API.
     - `WORKFLOW_ID` – The workflow ID used by the AI Tutor API.
     - `CHATBOT_ID` (optional) – Your chatbot ID.
     - `NEXT_PUBLIC_AITUTOR_TOKEN` – The public token for AI Tutor interactions.

4. **Run the Development Server:**

   ```bash
   pnpm dev
   ```

   The app will then be accessible at [http://localhost:3000](http://localhost:3000).

## Configuration & Setup

### Stripe Setup

1. **Install Stripe CLI:**  
   Follow the [Stripe CLI installation guide](https://stripe.com/docs/stripe-cli) if not already installed.

2. **Authenticate Stripe CLI:**  
   Run `stripe login` and follow the instructions.

3. **Set Up Products and Prices:**  
   In your Stripe Dashboard, create your products and corresponding prices for your subscription tiers. Then edit the file `lib/tiers.ts` with the proper details for each tier.

### Tiers & Message Limits

The available tiers are defined in **lib/tiers.ts**:

- **Free:**
  - Price: `null` (no payment required)
  - Description: For individuals who need to track their work.
  - Message Limit: **5 messages per month**
  - `productId`: *empty* (indicates the free plan)
  
- **Starter:**
  - Price: `$10/month`
  - Description: For small teams with basic collaboration needs.
  - Message Limit: **100 messages per month**
  - `productId`: *Set to your Starter tier Stripe product ID*

- **Pro:**
  - Price: `$30/month`
  - Description: For large teams needing advanced features.
  - Message Limit: **Unlimited** (represented as `-1`)
  - `productId`: *Set to your Pro tier Stripe product ID*
  
**How Message Limits are Enforced:**

- When a workflow runs, the API route (e.g., `/api/run/route.ts`) checks the current team's message count:
  - If the team has an active Stripe subscription (i.e. its `stripeSubscriptionId` is set) and its `stripeProductId` matches one of the tiers' `productId`, then the message limit for that tier is enforced.
  - Otherwise, the team is considered to be on the free plan with a limit of 5 messages per month.
- Each time a workflow is triggered, one message credit is consumed and the team's `currentMessages` count is incremented.

## Workflow and Chatbot Integration

- **Workflow Page:**  
  When a workflow is executed, the backend checks the message limit. If the limit is reached (e.g. for free users, 5 messages per month), the workflow is blocked and an error is returned.
  
- **Chatbot Page:**  
  Build and test chatbots created with AI Tutor API as part of your SaaS application.

## Sidebar Subscription Status Display

The sidebar features a dedicated component that displays:
- The **Subscription Tier Badge:**  
  Displays the full tier name when expanded or just the first letter when collapsed. If no active subscription exists, "Free" is shown.
  
- The **Messages Left Badge:**  
  Shows “Messages: X left” (or just the number when collapsed). If the tier provides unlimited messages, it displays “Unlimited” or an infinity symbol (∞). Additionally, the badge is styled green when remaining messages are available and red when the limit is reached.

This display automatically updates (using a polling mechanism) to reflect usage without requiring a page refresh.

## Database Setup & Seeding

Ensure the database schema includes:
- The teams table with `messageLimit` and `currentMessages` columns.
- Correct associations for users, team members, activity logs, invitations, and messages as defined in **lib/db/schema.ts**.
  
Run the seed script using:
  
```bash
pnpm db:seed
```

This creates a default user and team, initializing the free plan appropriately.

## Running in Production

Before deployment:
- Update your environment variables accordingly.
- Test your Stripe integration (webhooks, portal sessions, etc.).
- Verify that your database and subscription tiers are properly configured.

## License

This project is licensed under the MIT License.
