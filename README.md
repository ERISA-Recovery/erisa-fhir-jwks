<!-- README template 2.0 -->

# ERISA FHIR JWKS

Public JSON Web Key Sets (JWKS) that Epic on FHIR uses to verify our backend app's sign-in to each customer's Epic environment (SMART Backend Services: OAuth 2.0 with signed JWT client assertions). The repository is public on purpose: Epic downloads each file from its raw GitHub URL without signing in, and the files hold public keys only, never private keys. If the repository is made private, a file is renamed or moved, or the branch changes, that raw URL returns 404 and backend sign-in to that customer's Epic environment fails.

| Field | Value |
|---|---|
| **Status** | Active |
| **Stack** | JSON Web Key Sets (RFC 7517) · GitHub raw file hosting |
| **Runs** | On request: Epic downloads the file from its raw URL |
| **Handles PHI** | No |
| **Last reviewed** | 2026-11-04 |

**Contents:** [Architecture](#architecture) · [Getting started](#getting-started) · [Deployment](#deployment) · [Links](#links)

## Architecture

Each file is the JWK Set for one Epic customer and environment: `nonprod/<customer>.json` or `prod/<customer>.json`. Today only `nonprod/nw.json` exists, with one RS384 signing key; there are no `prod/` files yet. Our backend app signs a JWT client assertion with the matching private key, which is never stored here; Epic downloads the file from `https://raw.githubusercontent.com/ERISA-Recovery/erisa-fhir-jwks/main/<path>`, finds the key by its `kid` and checks the signature before it issues an access token.

### What it uses

**Resource group:** Not applicable: no Azure resources · **Infrastructure as code:** Not applicable: static files only

| Resource or system | Type | Purpose |
|---|---|---|
| GitHub raw file hosting (raw.githubusercontent.com) | Static file hosting | Serves each JWK Set at a public URL, no sign-in needed |
| Epic on FHIR app registration (fhir.epic.com) | Vendor app registration | Holds each file's raw URL as the app's JWK Set URL (JKU) for that customer and environment |

### What relies on it

| System or team | What they rely on |
|---|---|
| Epic on FHIR (fhir.epic.com app registration) | Downloads each file's raw URL as the JWK Set URL to verify our client assertions |
| Each customer's Epic team (today: `nonprod/nw.json`) | Configured the same raw URL on their side, by hand (as of the May 2026 Epic version) |

## Getting started

### Prerequisites

- Git and write access to this repository
- The new key's public part as a JWK (`kty`, `kid`, `use`, `alg`, `n`, `e`), never the private key

### Run locally

Not applicable: static JSON files, nothing to run and no settings.

To add or change a JWK Set, edit or add `nonprod/<customer>.json` or `prod/<customer>.json` on a branch and open a pull request into `main`:

```bash
git switch -c add-<customer>-key
# Edit the file: {"keys": [ ...public keys only... ]}
python -m json.tool prod/<customer>.json    # checks the JSON is valid
```

Each key may hold only public members (`kty`, `kid`, `use`, `alg`, `n`, `e`), never `d`, `p`, `q`, `dp`, `dq` or `qi`. Never rename or move an existing file (see the runbook below).

### Check that it works

```bash
curl -s https://raw.githubusercontent.com/ERISA-Recovery/erisa-fhir-jwks/main/nonprod/<customer>.json
```

It returns the JSON with a `keys` array. A 404 means Epic can't download it either.

### Run the tests

Not applicable: no tests in this repository.

## Deployment

**Pipeline:** Not applicable: no pipeline in this repository. Merging a pull request into `main` publishes the change at the raw URL straight away (GitHub may serve the previous version for up to 5 minutes). There is no `dev` branch.

**Release notes:** None: no release log yet.

**Rollback:** Revert the commit on `main`. The previous key set is served again at the same URL straight away (up to 5 minutes of GitHub caching).

## Links

- **Logs:** Not applicable: GitHub serves the files and keeps no download logs
- **Alerts:** None
- **Runbooks:** Key rotation: add the new key to the file next to the old one, wait for Epic to pick it up, then remove the old key. Don't rename, move or make private any file, the branch or the repository without first updating the registration on fhir.epic.com and telling the customer's Epic team.
- **More documentation:** [Epic backend OAuth 2.0 guide](https://fhir.epic.com/Documentation?docId=oauth2&section=BackendOAuth2Guide) · [SMART Backend Services specification](https://hl7.org/fhir/smart-app-launch/backend-services.html)
