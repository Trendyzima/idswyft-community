# Testagram self-owned identity verification

This deployment is the identity-verification engine operated by Testagram.
It does not call Didit, Idswyft Cloud, Persona, Onfido, or another hosted
verification provider.

## Runtime

The stack contains PostgreSQL, the ML verification engine, the API,
the verification portal, and optional Caddy TLS termination.

Application containers are built from this repository's source. Verification
documents stay in local Docker volumes.

## Start from source

1. Copy deploy/testagram/.env.self-owned.example to .env.
2. Generate real secrets locally and chmod .env to 600.
3. Point verify.testagram.site DNS at the server.
4. Start:

   docker compose -f docker-compose.yml -f docker-compose.build.yml -f docker-compose.testagram.yml --profile https up -d --build

5. Confirm:

   docker compose -f docker-compose.yml -f docker-compose.build.yml -f docker-compose.testagram.yml ps

Only Caddy should be public. PostgreSQL, API, and engine must not be
internet-facing.

## Testagram integration contract

Testagram should never receive document images or raw ID numbers. The
application integration should:

1. Create a verification session using the self-hosted API.
2. Redirect or embed the self-hosted verification UI.
3. Accept only signed verified, failed, or manual_review results.
4. Store the minimum result in Testagram: status, provider reference,
   country/type, last four digits where operationally required, and a keyed
   HMAC fingerprint for one-person-one-account enforcement.
5. Never copy source ID images into Supabase Storage.
6. Never expose the self-hosted API key to browser code.

## No hosted-provider dependency

The production overlay does not use Didit, Idswyft Cloud, hosted OCR/KYC APIs,
S3, or Watchtower. The verification data path is local to the server.

## Backup and recovery

Back up PostgreSQL and the verification upload volume together. Perform a
restore drill before accepting real identity documents. Treat .env as a
secret and store its backup in an encrypted server secret store, not Git.

## Production gates

Before production:
- TLS is active and HTTP redirects to HTTPS.
- API and engine are unreachable from the public internet.
- Upload limits and rate limiting are enabled.
- Session expiry and document retention are configured.
- Verification callbacks are authenticated and replay-resistant.
- Liveness and face-match decisions are auditable.
- Kenyan National ID is validated on real devices.
