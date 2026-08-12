# ERISA FHIR JWKS

Public JWK Sets used for JWT-based backend OAuth 2.0 authentication 
to Epic on FHIR customer environments (SMART Backend Services).

Only public key material is stored here — never private keys.

## Structure

- `nonprod/<customer>.json` — non-production JWK Set for a given Epic customer
- `prod/<customer>.json` — production JWK Set for a given Epic customer

## Usage

Each file's raw URL is registered with Epic as the app's JWK Set URL 
(JKU) for that customer/environment. Do not rename or move files 
without updating the corresponding registration on fhir.epic.com 
and re-notifying the customer's Epic team (they manually configure 
the local JKU on their end as of the May 2026 Epic version).

## Key rotation

See Epic's JWK Set URL documentation for the recommended rotation 
process (add new key alongside old, wait for propagation, then 
remove old key).
