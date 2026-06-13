# Glow Wallet Technical Review

Reviewer: go165

Bounty: Dean's List x Glow Wallet technical review, issue #4

Payout address if reward-eligible: `0x1f0130669ca6fd02e025a984cc038f139df19a2f`

## Scope Reviewed

This review focused on the public developer-facing Glow surfaces available without private credentials:

- Glow docs: getting started, detecting Glow, connecting/signing in, signing messages, and executing transactions.
- `glow-xyz/glow-js`: `@glow-xyz/glow-client`, `@glow-xyz/glow-react`, `@glow-xyz/solana-client`, `@glow-xyz/wallet-standard`, and the bundled Next.js example.
- `glow-xyz/nftoken`: repository structure and public documentation surface as related Glow ecosystem context.

I treated Glow as a developer integration product: the most important question is whether a dApp team can confidently add Glow, authenticate a user, request signatures, and debug production issues without accidentally building an unsafe or brittle integration.

## Executive Summary

Glow's core positioning is strong. The docs make the right first promise: one JavaScript SDK works across browser extension, iOS Safari extension, and Android in-app browser. The SDK architecture also maps cleanly to how Solana dApps actually integrate wallets: a low-level client, a React wrapper, wallet-standard support, and a Next.js example.

The highest-value improvement area is developer trust at the integration boundary. Several public docs and README snippets are close to correct but not aligned with the current TypeScript API shape, and the React/wallet-standard packages have thinner automated coverage than the transaction and sign-in utilities. These are fixable issues, but they matter because wallet integrations are copied into production quickly and are hard for downstream teams to diagnose when examples drift.

## Usability And Design

### What works well

- The getting-started flow is short and readable. It introduces installation, detection, sign-in, message signing, and transaction execution without overwhelming the reader.
- The docs correctly emphasize `signIn` over weak static-message authentication. Calling out replay risk and providing `verifySignIn` is a strong security and developer-experience choice.
- The transaction docs explain a useful distinction: Glow can either return signed transactions or sign and submit them. That fits both teams that already own RPC submission and teams that want the wallet to handle submission.
- The public README's package split is easy to understand: `glow-client` for direct integration, `glow-react` for React, `example-next-js` for a working app, and `wallet-standard` for broader Solana wallet ecosystem compatibility.

### Friction points

1. The docs still have unfinished developer-facing TODOs. The executing-transactions page says transaction errors still need to be written or standardized. This is exactly where integrators need help: user rejection, invalid network, simulation failure, RPC failure, timeout, and confirmation failure should be distinguishable.
2. The detection docs recommend opening `https://glow.app` and asking the user to refresh after install. That is simple, but it leaves a gap for modern dApps: provide a copy-paste React pattern for a non-blocking install CTA, retry after focus, and mobile-specific messaging for Safari extension vs Android app browser.
3. The Next.js example demonstrates sign-in/sign-out, but not the full path developers will test before shipping: `onlyIfTrusted`, message signing, transaction signing, batch signing, and error display. The `/buttons` page is useful for visual QA but not enough as an integration reference.
4. The docs and README should state the supported network names consistently. The docs say `mainnet` and `devnet`; source types also expose `Localnet`, while the README says localnet is not supported because backend RPC cannot connect to local machines.

## Feature Assessment

Glow covers the core wallet actions expected by Solana dApps:

- detect wallet availability through `window.glow`;
- connect and silent reconnect with `onlyIfTrusted`;
- sign in with a dynamic message and server-side verification;
- sign messages;
- sign one transaction or many transactions;
- sign and submit one transaction or many transactions;
- integrate with React through context;
- expose wallet-standard compatibility for dApps using the standard adapter path.

The most valuable product differentiator is the sign-in flow. `verifySignIn` checks the expected domain, expected address, and message freshness, and supports the signed-transaction path needed by Ledger users. That is a better default than asking every dApp to invent its own wallet-login message.

Two additions would round out the product:

- A transaction-result/error taxonomy in both SDK types and docs.
- A production-ready integration example that combines `onlyIfTrusted`, sign-in verification, message signing, transaction signing, and failure states in one app.

## Code Quality Review

### Positive observations

- The package boundaries are clean. The direct client, React wrapper, Solana transaction utilities, and wallet-standard adapter are separated enough that maintainers can evolve them independently.
- `verifySignIn` is readable and has focused tests for domain, address, old/future timestamps, malformed messages, and signature verification.
- `GTransaction` has a useful stated design goal: parse, sign, and serialize valid Solana transactions without depending on all of `@solana/web3.js`.
- CI exists for `glow-client` and `solana-client`, with lint, typecheck, and tests.

### Issues and recommendations

1. `packages/glow-client/README.md` has stale API examples.

The README sample calls `signMessage` with `message_utf8` / `message_hex` and destructures `signature_base64`. The current source accepts `messageHex`, `messageBase64`, `messageUint8`, or `messageBuffer`, and returns `signatureBase64`. The sample also passes a `Transaction` object to `signTransaction`, while the current client method expects `transactionBase64`.

Recommendation: update the README examples to compile against the current public types. Add a small docs-test or TypeScript example compile check so README drift gets caught in CI.

2. `GlowProvider.signOut` bypasses `GlowClient.disconnect`.

In `packages/glow-react/src/GlowContext.tsx`, `signOut` calls `window.glowSolana!.disconnect()` directly. That is surprising because the provider otherwise routes through the singleton `GlowClient`, and the direct non-null assertion can throw when a dApp has `window.glow` but not `window.glowSolana`.

Recommendation: call `await glowClient.disconnect()` and rely on the existing update event path. If the Phantom-compatible `glowSolana` path must stay, guard it and document why it is different from `window.glow`.

3. Event lifecycle is under-specified.

`GlowClient.registerLoadedHandler` adds a window `message` listener and polls every 250 ms until Glow loads. That is reasonable for extension load timing, but the client does not expose a cleanup path and `emitUpdate(reason)` currently drops the reason. In React apps with hot reload, tests, embedded widgets, or multiple client instances, this makes lifecycle behavior harder to reason about.

Recommendation: include the update reason in the emitted event, document whether `GlowClient` is intended to be a singleton, and consider a `destroy()` method or internal guard that prevents duplicate wallet event handlers.

4. React and wallet-standard packages need tests.

There are tests for sign-in utilities and Solana transaction parsing, but I did not find equivalent tests for `GlowProvider` behavior or wallet-standard adapter behavior. The adapter contains important account validation, chain validation, multi-transaction behavior, and event emission logic.

Recommendation: add unit tests for:

- `GlowProvider` sign-in/sign-out state transitions;
- absence of `window.glowSolana`;
- wallet-standard connect/disconnect events;
- invalid account and invalid chain failures;
- multi-transaction same-chain vs conflicting-chain behavior.

5. Numeric lamports are exposed as `number` in some Solana client types.

`SolanaClientTypes.ParsedAccount`, `Account`, and transaction metadata use `number` for lamports/fees/balances, with TODO comments to replace these with strings. Solana lamport amounts can exceed JavaScript's safe integer range in account balances and ledger data.

Recommendation: change public balance-like fields to `string`, `bigint`, or a documented branded decimal type before these types become harder to migrate. At minimum, document which methods are safe for UI display only vs accounting logic.

## Security Review

No private keys, seed phrases, or production systems were accessed during this review.

### Good security defaults

- The docs discourage weak static sign-in messages and explain replay risk.
- `verifySignIn` validates address, domain, and recency.
- Transaction approval is user-mediated; docs explicitly say Glow does not offer auto-approve.
- `signAndSendAllTransactions` is documented to submit dependent transactions sequentially after prior confirmation, which is safer than blindly broadcasting a dependent batch.

### Security concerns to address

1. Sign-in domain matching is strict but narrow.

`verifySignIn` accepts exact domains or an explicit allowlist. That is good, but many real dApps have multiple hostnames (`app.example.com`, preview deployments, custom domains). The docs should show a safe allowlist pattern and warn against accepting arbitrary suffixes like `.example.com` without normalizing and auditing the host.

2. Signed-transaction verification assumes the first instruction carries the sign-in message.

The Ledger-compatible sign-in branch parses the transaction and reads `gtransaction.instructions[0].data_base64`. If the expected transaction format is always a note-program instruction at index 0, the verifier should also assert the expected program id and instruction shape before trusting the payload. This would make the invariant explicit and easier to audit.

3. Error surfaces should not require console inspection.

Several code paths use generic `Error` messages or console logging. For wallets, downstream apps need typed errors or at least stable error codes for user rejection, invalid network, invalid account, not connected, simulation failure, and RPC failure.

## Performance Review

The SDK is small and intentionally split into packages. That is a good base for performance-sensitive dApps.

Areas to improve:

- The React provider polls every 250 ms until Glow is detected. That is acceptable before detection, but docs should recommend rendering a passive install CTA rather than continuously triggering user-facing work.
- The Next.js example is still on Next 12 and is mostly a sign-in demo. Updating it to a current Next version and adding a minimal transaction path would help integrators test production-like bundling and tree shaking.
- `GTransaction` avoids the heavier `@solana/web3.js` path for parse/sign/serialize, which can help bundle size. It would be useful to publish a small bundle-size comparison or benchmark in the README because this is a strong selling point.

## Documentation Recommendations

1. Add a "copy-paste current API" section for each core action:
   - connect;
   - silent reconnect;
   - sign in;
   - verify sign-in server-side;
   - sign message;
   - sign transaction;
   - sign and send transaction.
2. Add an error-handling page with typed examples.
3. Add a migration note between direct `window.glow`, `GlowClient`, `GlowProvider`, and wallet-standard usage.
4. Update `glow-client` README examples to match current method names and return values.
5. Add a "Testing your integration" page showing how to use devnet safely without relying on production funds.

## Suggested Priority List

1. Fix stale README examples for `glow-client`.
2. Route `GlowProvider.signOut` through `GlowClient.disconnect`.
3. Add React/wallet-standard unit tests.
4. Standardize SDK error codes and document transaction errors.
5. Strengthen Ledger sign-in transaction verification by asserting program/instruction shape.
6. Publish a fuller Next.js example that covers the complete auth + signing flow.

## Verification Notes

- Reviewed `glow-xyz/glow-js` public source, package metadata, workflows, examples, and tests.
- Reviewed public Glow docs for getting started, detection, connecting/signing in, signing messages, and executing transactions.
- Reviewed `glow-xyz/nftoken` repository at a high level as related ecosystem context.
- Installed the relevant `glow-js` workspace dependencies with `NPM_CONFIG_REGISTRY=https://registry.npmjs.org npx pnpm@9.12.1 install --filter @glow-xyz/solana-client... --filter @glow-xyz/glow-client... --frozen-lockfile`. The first attempt against the configured npm mirror returned 503, so I reran against the official npm registry.
- Local verification passed:
  - `npx pnpm@9.12.1 --filter @glow-xyz/solana-client run build`
  - `npx pnpm@9.12.1 --filter @glow-xyz/solana-client run lint`
  - `npx pnpm@9.12.1 --filter @glow-xyz/solana-client run tsc`
  - `npx pnpm@9.12.1 --filter @glow-xyz/solana-client run test` (6 suites, 34 tests)
  - `npx pnpm@9.12.1 --filter @glow-xyz/glow-client run lint`
  - `npx pnpm@9.12.1 --filter @glow-xyz/glow-client run tsc`
  - `npx pnpm@9.12.1 --filter @glow-xyz/glow-client run test` (1 suite, 15 tests)

## Final Assessment

Glow has a solid developer experience foundation and a genuinely useful secure sign-in primitive. The main risk is not architectural; it is integration drift. Wallet SDK examples are often copied directly into production, so stale README snippets, thin React/wallet-standard tests, and unspecified error semantics can create avoidable downstream bugs. Addressing those items would make Glow feel much safer for dApp teams to adopt and easier for maintainers to support.
