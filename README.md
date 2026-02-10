# solve-captchas-on-openclaw
Zero-code guide to load the CapSolver browser extension into OpenClaw’s Chromium session so your agent can complete authorized web flows without getting stuck on CAPTCHA prompts.


# Solve CAPTCHAs in OpenClaw (CapSolver Extension)



Enable automatic CAPTCHA handling in **OpenClaw** by loading the **CapSolver browser extension** into the agent’s Chromium browser profile.

- ✅ No custom integration code
- ✅ The extension solves challenges in the background
- ✅ Your prompts stay natural (you usually just add a short wait before submitting)


---

## What this repo is

This repo documents a simple setup:

1. Install the **CapSolver browser extension**
2. Load it inside OpenClaw’s browser configuration
3. Give the agent enough time (via a short `wait`) before submitting forms

OpenClaw runs a dedicated Chromium profile for the agent. Once the extension is loaded there, it can detect CAPTCHA widgets and handle them before form submission.

---

## Common use cases

This setup is useful when your OpenClaw agent needs to complete **authorized** web workflows such as:

- **Internal tools & admin panels** (approved automation for QA / ops)
- **Form submissions** (lead forms, support forms, onboarding forms) on sites you manage
- **Account/login flows** for your own products (testing + monitoring)
- **Public data workflows** where you have permission (navigation + extraction without manual stops)
- **Accessibility assistance** (reducing CAPTCHA friction for permitted tasks)
- **E2E testing** where CAPTCHA is present in staging environments

---

## Prerequisites

- OpenClaw installed and gateway running
- A CapSolver account + API key
- A Chromium-based browser that can load unpacked extensions (see note below)

### Important: Use Chromium / Chrome for Testing (not branded Chrome)

Some branded Chrome builds may ignore `--load-extension` in automated sessions.
If your extension doesn’t load, switch to:
- **Chrome for Testing**
- **Chromium**
- **Playwright’s bundled Chromium**

---

## Installation (Ubuntu / Linux)

### 1) Install CapSolver Extension (required)



https://github.com/capsolver/capsolver-browser-extension/releases/

Download the **Chrome** zip from the release assets and extract it to:

```bash
mkdir -p ~/.openclaw/capsolver-extension
unzip CapSolver.Browser.Extension-chrome-v*.zip -d ~/.openclaw/capsolver-extension
````

Verify:

```bash
ls ~/.openclaw/capsolver-extension/manifest.json
```

---

### 2) Set your CapSolver API key

Edit:

```text
~/.openclaw/capsolver-extension/assets/config.js
```

Set:

```js
export const defaultConfig = {
  apiKey: "CAP-XXXXXXXXXXXXXXXXXXXXXXXX",
  useCapsolver: true
};
```

---

### 3) Configure OpenClaw to load the extension

Edit your OpenClaw config:

```text
~/.openclaw/openclaw.json
```

Example:

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/path/to/chromium-or-chrome-for-testing",
    "extensions": ["~/.openclaw/capsolver-extension"],
    "noSandbox": true,
    "defaultProfile": "openclaw"
  }
}
```

Notes:

* `executablePath` must point to your Chromium / Chrome for Testing binary.
* `noSandbox: true` is commonly required on servers, Docker, CI.

---

### 4) Restart OpenClaw

```bash
pm2 restart opencrawl --update-env
# or
openclaw gateway restart
```

---

## Verify it’s loaded

### Check logs

```bash
pm2 logs opencrawl --lines 50 --nostream
```

Look for something like:

* `Loading 1 extension(s)`
* a log line showing Chromium spawned with extension args

### Optional: CDP check

```bash
curl -s http://127.0.0.1:8091/json/list
```

You should see a target with `chrome-extension://...` (often a `service_worker`).

---

## How to use (prompting pattern)

**Don’t talk about CAPTCHAs.** Just add a short wait before submitting.

Example:

```text
Go to https://example.com,
fill the form with the required info,
wait 45 seconds,
then click Submit and tell me what happens.
```

### Recommended waits

| Challenge type       | Recommended wait |
| -------------------- | ---------------- |
| reCAPTCHA v2         | 30–60s           |
| reCAPTCHA v3         | 20–30s           |
| Cloudflare Turnstile | 20–30s           |

When unsure: use **60 seconds**.

---

## Troubleshooting

### Extension not loading

* Try **Chromium / Chrome for Testing / Playwright Chromium**
* Confirm the extension directory contains `manifest.json`
* Confirm OpenClaw is actually using the `executablePath` you set

### CAPTCHA still blocks submission

* Increase wait time (try 60s)
* Verify API key is correct
* Check your CapSolver account balance / limits
* Confirm the extension is visible via logs / CDP

### Crash after switching browser binaries

Your profile may be incompatible across versions:

```bash
rm -rf ~/.openclaw/browser/openclaw/user-data
```

Restart the gateway.

---

## Headless servers

Chrome extensions typically need a display context.

```bash
sudo apt-get install -y xvfb
Xvfb :99 -screen 0 1280x720x24 &
export DISPLAY=:99
```

Also consider setting:

```json
"noSandbox": true
```

---

## Disclaimer

This project is an **unofficial** configuration guide and is not affiliated with OpenClaw or CapSolver.
Use only in compliant, authorized workflows.

```

If you want, tell me the final repo name you chose (e.g., `openclaw-capsolver`, `openclaw-captcha-solver`, etc.) and I’ll tweak the title + wording so it matches the branding perfectly.
::contentReference[oaicite:0]{index=0}
```
