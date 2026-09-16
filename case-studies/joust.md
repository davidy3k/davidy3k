# Joust

### Low-stakes prediction markets for communities that already trust each other

**Co-founder & Full-stack Product Engineer · 2025–present**

**Scope:** Full-stack application architecture, wallet and contract integration, transaction recovery, settlement workflows, and product delivery

**Team:** Three founders; I own the web application and established its engineering patterns. One engineering cofounder owns the smart contracts and contributes across the app; the other leads product, design, and partnerships.

Joust lets people create prediction pools, stake tokens on outcomes, and choose someone they trust to settle the result. The intended setting is a friend group, group chat, niche Discord, or streamer community where an arbiter already has relationships and a reputation to protect.

We launched on Abstract in late October 2025 for friends and family. That release let us exercise the core creation, entry, arbitration, settlement, and refund flows together. The public app is now in maintenance while I develop its next iteration; the Abstract contract remains deployed.

<p align="center">
  <img src="../assets/joust/joust-create-mobile.jpeg" width="360" alt="Joust mobile creation flow with arbiter, outcomes, expiration, and collateral controls" />
</p>

*The mobile creation flow exposes pool terms before opening the wallet. The screen uses testnet data, not activity from the 2025 launch.*

## Make trust a product decision

The premise is simple: join a market when you trust its arbiter. A community moderator or friend can judge an outcome that would be difficult to resolve through a general-purpose data feed. That makes small, community-specific questions possible, while making the choice of arbiter central to participation.

Pools support binary questions or multiple outcomes. Participants contribute tokens to a shared pool, and the contract calculates payouts when the arbiter selects the result. My responsibility was the application around that contract: creation and entry flows, wallet interactions, lifecycle state, settlement records, and background jobs.

An invited arbiter must accept before a pool opens. The invitation presents the terms and obligations, including the deadline, collateral token, minimum entry, and arbiter fee. If the invitation is never accepted, the pool remains pending until declined or expired. If an unsettled pool passes its expiry by a day, a participant can trigger a refund for everyone rather than wait indefinitely for its arbiter.

Honor voting adds a record of participants' experience. After settlement or refund, participants can cast one vote on the pool's arbiter; self-voting is excluded, and a downvote requires a comment. A background job calculates a weighted score using stake value, account age, and voter diversity, with diminishing weight for repeated positive votes and stronger penalties for negative feedback. This supplements the community's existing trust rather than establishing that an unfamiliar arbiter is trustworthy.

## Recover the gap between wallet and application

A successful wallet submission and a saved application record are separate events. If the browser closes between them, the user's money may have moved while the interface still shows unfinished work.

After the initial launch, I built a shared transaction-intent flow for creating pools, entering, accepting arbitration, closing, settling, and refunding. Before opening the wallet, the server records the requested action and the browser saves a local reference. Once the wallet returns a transaction hash, the browser saves that hash locally before attaching it to the server record.

```mermaid
flowchart LR
    A[Record intended action] --> B[Open wallet]
    B --> C[Save returned hash locally]
    C --> D[Attach hash to server record]
    D --> E[Retrieve and check receipt]
    E --> F[Write application records]
    F --> G[Run follow-up jobs and refresh UI]
```

On a later visit, the app can reattach a locally saved hash or resume tracking an action already submitted to the server. The server retrieves the receipt and checks its sender, target, contract-emitted event, and action-specific values against the stored intent. For an entry, that includes the pool, participant, option, and amount. Event records are unique by chain, transaction hash, and log index, and the entry handler writes the event, entry, and fund movement in a database transaction.

Centralizing this flow gives the interfaces shared progress states, attachment retries, error reporting, and cache refresh behavior. Settlement and refund jobs can continue after the user leaves the page.

Recovery has a defined boundary: it needs a hash saved locally or attached to the server. Discovering submissions lost before either happens, and activity initiated outside the app, needs the independent blockchain observer I still plan to add. Receipt confirmation also does not yet include application rollback for chain reorganizations.

## Turn settlement into an understandable result

The contract handles settlement and payouts. I built the application workflow that turns that result into participant histories, fee records, and notifications.

An Inngest workflow reads the contract's settlement summary in a retryable step and passes it to downstream jobs. Those jobs update entry outcomes, record fees, and build participant summaries with stakes, payouts, and profit. Reusing the contract result keeps the application from maintaining a separate payout calculation for each screen. Refunds have a corresponding workflow and explicit fund-movement records.

The same separation carries into the interface: Next.js server rendering populates the initial screens, while TanStack Query supports filtering, pagination, and refreshes after transactions. A user can be a participant in one pool and an arbiter in another, with both views drawing from shared query functions and keys.

## Building the application end to end

Joust gives me ownership across product interactions, persistence, wallet integration, asynchronous processing, and production delivery, alongside a cofounder responsible for the contract layer. Current work is focused on strengthening transaction observation and recovery, simplifying the product flows, and preparing the application for another community launch.

The playful castle setting is part of the product identity. A layered PixiJS background adds atmosphere behind the React interfaces, with a reduced-motion path.

![Animated Joust castle atmosphere with a star field and fireflies](../assets/joust/joust-atmosphere-1.gif)

*The layered PixiJS scene animates independently behind the React application.*

## Stack

TypeScript · React · Next.js · TanStack Query · Prisma · PostgreSQL · Inngest · wagmi · viem · Abstract Global Wallet · PixiJS · Sentry

---

[Back to profile](../README.md)
