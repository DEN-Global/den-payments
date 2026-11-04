# DEN Explicit sandbox payment state machine

Private service package, version 0.1.0, JSON contract version `1.0.0`. Plain ESM with no third-party runtime dependencies. Backend-compatible exports from `src/index.js`: `createSandboxPayment, applySandboxEvent, orderReceipts`.

Run `npm test` and `npm run check` on Node 24. CI runs both. Tests cover successful state transitions and authorization/idempotency failure paths; they do not establish production readiness.

## Integration boundary

Server constructs payment with stored order/payer/amount. Provider adapter must authenticate events before invoking transitions; never accept body status as verified live payment. Persist event ID and new version atomically. Product payment is digital at ordering; delivery is cash/digital at arrival, with separate cash confirmation.

State transitions return new state; callers must store results transactionally with compare-and-swap/unique constraints. In-memory arrays are command fixtures, not production storage. The parent backend vendors a pinned copy plus checksum manifest for private-repository builds; this package is the authoritative source. Client-provided actors, statuses, receipt flags and balances have no authority.

## Current readiness

Sandbox-only. No processor credentials, live charge, live webhook, cash-code verifier, actual refund issuance, wallet posting or provider onboarding. Refund transition models a sandbox event only; policy remains unapproved.

No external resources are provisioned, no credentials are present, and no live money is moved. See `contract.json` for the version envelope; domain payload validation is exported by the functions above.

## Commission calculations and Pricing Lab

`@den/payments/commissions` exports progressiveCommission, monthlyCommission, selectSchedule, proposeCollection, warehouseHolidayEligible and pricingLab. All amounts use integer minor units; progressive marginal tier charges round once, half up. Monthly periods use explicit UTC month boundaries. Selectors require exactly one approved country/currency/plan/category schedule effective at period start. Example test rates are synthetic test inputs, never adopted production rates.

Automatic collection is only a proposal after seven full notice days, explicit admin enablement, a current matching customer/currency mandate and explicit absence of dispute/hold. No charge is created. The warehouse holiday lasts 30 elapsed days from verified business first approval, with no reset for facilities/staff. Pricing Lab outputs always carry approved:false and simulation:true. Proration, invoice/mandate persistence, regional business timezone rules, tax, real collection and refunds remain pending approval/integration.
