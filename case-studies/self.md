# Self Financial

### Rebuilding a consumer-fintech app across web and mobile

**Software Engineer · October 2019–November 2022**

**Scope:** React Native rewrite, shared UI components, WebView onboarding, financial account flows, and native integrations

Self Financial helps customers build credit through a Credit Builder Account and secured credit card. I helped replace its separate iOS and Android applications with a redesigned React Native app, while also shipping features in the existing web application.

I was one of two engineers doing most of the full-time implementation on the mobile team. Our engineering manager and tech lead also contributed code and provided substantial review. My work included shared UI primitives, onboarding integration, autopay, Plaid account linking, and native modules for date selection, deep links, push routing, and app-state masking.

## A new interface around established product behavior

The rewrite began around the time I joined. Our tech lead evaluated cross-platform options and selected React Native and the supporting tooling. I initially worked on the web application while product decisions and designs from an outside design firm were taking shape, then moved primarily to mobile. We shipped the replacement app around late summer 2020.

The existing web application already contained much of the product behavior we needed. It was also a large codebase with many contributors, accumulated technical debt, and predominantly class-based React components. Rebuilding the mobile app gave us an opportunity to carry that behavior into smaller function components using hooks and context. We used Apollo for server-data caching and kept additional global application state limited.

I contributed extensively to a UI primitive component library used by both web and mobile. That work translated the design firm's visual system into reusable building blocks for the product screens. Alongside the new layouts, the app incorporated animations, gestures, and responsive interactions that made it feel substantially more polished than its predecessor.

The work combined reuse at two levels: established financial-product behavior from the web application and shared interface components for the redesigned experience.

## Keeping onboarding on the web

Onboarding needed a different release cadence from the mobile app. The business continually tested and adjusted its acquisition funnel, and those experiments could not depend on an app-store review for every change. We embedded the web onboarding flow in a WebView so it could continue to evolve independently.

Login and registration lived in the app. If the customer's account data showed incomplete onboarding, the app opened the web flow. That customer was already authenticated, but the WebView had its own isolated browser session. Making the transition feel continuous required work on both sides of that boundary.

### Coordinating session handoff and page startup

One failure occurred when the web application's authentication check ran before the app had supplied the session to the WebView. The web flow interpreted the missing session as a logged-out customer and redirected them to web login.

I worked on coordinating session delivery with the web application's startup behavior so it could distinguish a pending handoff from a genuinely missing session. The important constraint was ordering: opening the page and supplying authentication could not be treated as unrelated operations. Resolving it required changes in both the app integration and the web flow.

### Handling completion and external redirects

The app also needed to recognize when onboarding had finished and return the customer to the appropriate mobile experience. We worked with messages from the embedded page and WebView navigation events to communicate that transition. These were integration points shared with a web application that other engineers continued to change.

Plaid added another navigation boundary: its embedded flow could initiate an OAuth redirect inside the WebView. I worked on intercepting the relevant navigation requests, extracting the redirect parameters, and passing them into the integration so the flow could continue.

Getting this working required iteration across iOS, Android, and different platform versions. After we worked through the session and navigation issues, onboarding became a dependable part of the app experience. Keeping the funnel on the web preserved the business's ability to experiment quickly, while the mobile integration handled the transitions around it.

## Financial flows and native modules

I implemented autopay management for both credit-building products and Plaid account linking, along with account, profile, settings, and payment-related interfaces. I also worked regularly in the web application, often implementing corresponding features as I worked on mobile.

Some of my app work crossed into native code:

| Integration | My contribution |
| --- | --- |
| Date selection | Authored the native date picker used in autopay. |
| Deep links and push routing | Authored native integrations that routed external entry points into the app. |
| App-state masking | Authored native behavior to obscure sensitive account information when the app entered the background or appeared in the app switcher. |

I also made small contributions to the existing biometric authentication library.

Our team maintained the GraphQL server supporting the clients. I added or updated endpoints when features needed them. My contributions to the underlying Python services were minor; most of my implementation work was in the web and mobile applications.

## Shipping with a growing engineering team

I participated in the department's rotating release-manager role for biweekly platform releases, coordinating staging builds, peer QA, fixes, and deployments across the web application, GraphQL server, and changed backend services.

Mobile releases followed a separate cadence because of store submission and review. We used TestFlight and manual regression checks, bringing in other engineers for broader validation of larger changes. This made release preparation and testing part of everyday implementation work.

During my tenure, Self reported an Apple App Store average of **4.9 stars across more than 150,000 reviews** in September 2021. That is context for the product our team supported, rather than a measure of any individual's contribution. [Self's September 2021 announcement](https://www.globenewswire.com/news-release/2021/09/16/2298490/0/en/self-financial-raises-50m-in-series-e-funding-led-by-altos-ventures.html)

## Stack

React Native · React · JavaScript / TypeScript · Apollo · GraphQL · Objective-C · Swift · Java

## About this case study

This account describes my work from 2019–2022, reconstructed from memory without access to Self's application source. Implementation details are kept at the level I can confidently describe. The product has continued to evolve since then.

---

[Back to profile](../README.md)
