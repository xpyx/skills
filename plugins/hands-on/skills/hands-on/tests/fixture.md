# Repo fixture (for scenario runs)

You are an AI coding assistant working in this repo. Treat it as your working
context. You cannot browse it beyond what is described below — work from this
description, and write any code inline in your reply rather than editing repo
files.

## Repo: `orders-service`

Express + TypeScript, vitest for tests, pnpm.

```
src/
  routes/orders.ts      POST /orders, GET /orders/:id — existing, working
  pricing.ts            EMPTY FILE — new work goes here
  types.ts              Order, Tier, Money
  db/index.ts           knex instance
test/
  orders.test.ts        existing route tests, 12 passing
```

`src/types.ts` (existing):

```ts
export type Money = { cents: number; currency: 'EUR' }

export type Tier = {
  id: string
  minSubtotalCents: number
  percentOff: number
  expiresAt: string | null   // ISO date, null = never
}

export type Order = {
  id: string
  lines: { sku: string; qty: number; unitPrice: Money }[]
  subtotal: Money
  customerId: string
}
```

## Product context

Discount tiers are configured per merchant. A customer's order gets at most
one tier applied — the best one they qualify for. Expired tiers must be
ignored. Discounts must never push an order below zero.
