# MintStars

### Rebuilding creator commerce around internal balances and reliable payment flows

**Lead Software Engineer · November 2023–February 2025**

**Scope:** Application architecture, financial systems, infrastructure, product delivery, and engineering mentorship

MintStars was a subscription and commerce platform for independent adult creators. I architected and led its Next.js rewrite, moving everyday purchases and tips from onchain transactions to an internal balance system, with crypto used at the funding and payout boundaries.

I built the replacement with a UI-focused mid-level engineer while we maintained the existing application. After early guidance from a part-time CTO, I became the company's technical owner: responsible for architecture, infrastructure, code review, and releases. As the team changed, I mentored mid-level and junior engineers and helped them take on more release and operational responsibility. I worked closely with our PM, founders, and customer-service lead.

![MintStars creator dashboard with sample earnings, weekly metrics, and top fans](../assets/mintstars/mintstars-dashboard.png)

*Recreated interface from my tenure, using synthetic names and figures. This is sample data, not a record of company performance.*

## Moving everyday commerce offchain

The previous Nest application put each purchase and tip on Polygon, then relied on blockchain listeners to update application state. Transactions could stall, leaving users waiting for ordinary product interactions to complete.

The founders and part-time CTO had outlined a different approach: keep everyday commerce in an internal ledger and use crypto for moving funds into and out of the platform. I turned that direction into a full-stack TypeScript application using Next.js, tRPC, Prisma, and PostgreSQL. Purchases could update balances and content ownership in the database without waiting for a blockchain confirmation.

This was an application rewrite with deliberate reuse. We retained the existing PostgreSQL database and adapted the Prisma schema around it, reducing the amount of data restructuring required at cutover. We also carried over UI components, repairing and refactoring them as we rebuilt publishing, subscriptions, purchases, messaging, and account flows.

## Keeping purchases, balances, and history together

I built a shared transaction model linking business events to their balance changes, fees, and content movements. A content purchase illustrates the boundary:

1. Debit the buyer's balance and check that sufficient funds remain.
2. Create the purchased content ownership record and credit the creator.
3. Create the payment event, balance-change records, platform-fee record, and content-movement record under one transaction identifier.

These writes run in a serializable database transaction: they commit together or roll back together. The purchase's insufficient-funds check can therefore reject the operation without leaving a partial debit or ownership change. Transaction-conflict handling retries the database operation.

The same transaction model connects deposits, subscriptions, tips, resales, royalties, withdrawals, and administrative adjustments to a common history. It is an application ledger, not a full double-entry general ledger: its purpose is to make a product event and its associated movements traceable.

![MintStars transaction history with sample deposits, purchases, tips, royalties, and withdrawals](../assets/mintstars/mintstars-transaction-history.png)

*Recreated interface with synthetic people, dates, titles, and amounts. Creators could filter their history and export it as CSV.*

## Connecting external payments to internal balances

Incoming provider payments had a different lifecycle from purchases using an existing balance. I integrated Coinflow and used authenticated webhooks to dispatch payment processing through Inngest background jobs.

For a deposit, the application first creates an expected deposit record. On a settled payment event, processing checks that record's status, provider identifier, wallet presence, and expected amount. It then credits the balance, completes the deposit, and creates the linked transaction and balance-change records in one database transaction. Event identifiers, status checks, and unique provider identifiers provide safeguards against repeated processing.

Withdrawals crossed the boundary in the other direction. Bank and debit-card payouts required provider onboarding and a configured payout destination. The application submitted the provider-supplied Polygon transaction, then an Inngest workflow waited for chain confirmation before completing the withdrawal and committing the balance debit and transaction history together. This was the application's funding-confirmation boundary; it did not mean the recipient's bank had already posted the payout.

Creators could also withdraw native USDC directly on Polygon, including where the fiat provider was unavailable. The UI explained the network and token requirements and used a press-and-hold confirmation. Pending-withdrawal checks, retryable confirmation jobs, failure alerts, and manual recovery tools supported the operational side of both payout paths.

## Improving discovery and publishing

I simplified the homepage's feed queries and added Redis caching for paginated results and counts, applying viewer-specific permissions when preparing the response. This made discovery noticeably faster, though I no longer have a comparable before-and-after measurement.

Publishing needed similar attention. I added Firebase image processing for web delivery and improved the upload experience with progress, cancellation, retry, and error handling. API Video uploads, processing webhooks, and the post-saving flow had to coordinate so creators could publish image and video content through one interface. Much of this work involved improving inherited components alongside the new backend.

## Migration, validation, and a delivery lesson

We launched the replacement around late February or early March 2024. Before cutover, we rehearsed the data migration against a production snapshot in staging, compared balances using scripts that exercised asset and balance movements, and had the team test uploads, purchases, tips, messaging, search, and filtering on a preview deployment.

The launch also depended on a new payment-provider integration. I worked directly with Coinflow on the capabilities we needed, but the complete funding and payout flows came together too close to launch. Infrastructure integration issues during the move from GCP hosting to Vercel then extended the overnight cutover to roughly six hours, against about 90 minutes planned.

We completed the migration, but I had underestimated the integration buffer. My main lesson was to bring complete provider flows and migration rehearsals forward in the schedule, with automated regression coverage for core payment flows. Our scripted checks and manual staging QA were useful; they needed more time and stronger automated coverage before that release.

## Business context

During my tenure, COO reporting showed roughly **12× growth in onboarded creators and 10× growth in monthly total sales volume**. These are approximate company-level figures from that period. Creator admission involved invitations or operations review; sales volume represents platform commerce, not MintStars' revenue. The figures provide context for the platform we supported rather than attributing that growth solely to engineering.

## Stack

TypeScript · React · Next.js · tRPC · Prisma · PostgreSQL · Redis · Inngest · Coinflow · Polygon/USDC · Firebase · API Video · Vercel · Sentry · Axiom

---

MintStars' source code and production data are private. Screenshots recreate the interface using sample data; no company source code is included here.

[Back to profile](../README.md)
