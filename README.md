# 🔎 PAN Lookup API

A lightweight **Flask-based PAN lookup API** that forwards PAN lookup requests to the configured TurtleMint Loans service and returns the upstream JSON response.

The project also contains a background Selenium task intended to refresh an upstream Bearer token automatically.

> ⚠️ **Important:** This project interacts with a third-party service and handles highly sensitive financial identity information. Use it only with proper authorization, consent, and in compliance with applicable laws, regulations, and the third party's terms of service.

---

## ✨ Features

- REST API built with **Flask**
- PAN lookup through a dedicated `/lookup_pan` endpoint
- Automatic PAN normalization to uppercase and whitespace trimming
- JSON responses with upstream HTTP status codes preserved
- Selenium/Chrome automation for attempting to refresh an upstream Bearer token
- Background token-refresh loop configured for a **15-minute interval**
- Vercel configuration included through `vercel.json`
- Dev Container configuration included for GitHub Codespaces / VS Code

---

## 🧰 Tech Stack

- **Python 3.11+**
- **Flask** — HTTP API
- **Requests** — upstream HTTP requests
- **Selenium** — browser automation
- **webdriver-manager** — ChromeDriver management
- **Vercel Python runtime** — deployment configuration

---

## 📁 Project Structure

```text
Pan-main/
├── .devcontainer/
│   └── devcontainer.json
├── requirements.txt
├── t2.py
└── vercel.json
```

### Main files

| File | Purpose |
|---|---|
| `t2.py` | Flask application, token-refresh logic, and PAN lookup endpoint |
| `requirements.txt` | Python dependencies |
| `vercel.json` | Vercel Python build and routing configuration |
| `.devcontainer/devcontainer.json` | Development container / Codespaces configuration |

---

## 🚀 Local Setup

### 1. Clone the repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd Pan-main
```

### 2. Create a virtual environment

**Windows:**

```bash
python -m venv .venv
.venv\Scripts\activate
```

**Linux / macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Start the API

```bash
python t2.py
```

The Flask server listens on:

```text
http://127.0.0.1:5000
```

---

## 🔌 API Reference

### Health / root behavior

The current application does not define a dedicated `/health` or `/` JSON endpoint. The primary application route is:

```text
GET /lookup_pan
```

### PAN Lookup

**Request:**

```http
GET /lookup_pan?pan=ABCDE1234F
```

Example with `curl`:

```bash
curl "http://127.0.0.1:5000/lookup_pan?pan=ABCDE1234F"
```

The application converts the supplied PAN to uppercase and removes leading/trailing whitespace before sending the request upstream.

### Missing PAN

Calling the endpoint without the `pan` query parameter returns HTTP `400` with a response similar to:

```json
{
  "status": "error",
  "message": "Usage: /lookup_pan?pan=ABCDE1234F"
}
```

### Upstream errors

If the upstream response cannot be parsed or the request fails, the API returns a JSON error response with HTTP `500`.

---

## 🔐 Authentication / Token Handling

The application currently contains an upstream Bearer token directly in `t2.py` and also attempts to refresh it through Selenium.

For production use, **do not commit API tokens, cookies, credentials, or other secrets to GitHub**.

A safer pattern is to use environment variables or a managed secret store, for example:

```bash
UPSTREAM_BEARER_TOKEN=your-token-here
```

Then load the secret in Python instead of hard-coding it in source code.

### Recommended security changes

- Rotate any token that has previously been committed to a public or shared repository.
- Move secrets to environment variables / Vercel Environment Variables.
- Add authentication and rate limiting to your own API.
- Validate the PAN format before sending requests upstream.
- Log only non-sensitive metadata; never log full PANs or access tokens.
- Do not expose upstream credentials to API clients.
- Use HTTPS in production.

---

## 🤖 Automatic Token Refresh

`t2.py` starts a daemon thread that repeatedly attempts to:

1. Launch Chrome in headless mode.
2. Open the configured TurtleMint personal-loan page.
3. Wait for the page to load.
4. Inspect Chrome performance logs.
5. Search the captured logs for a Bearer token.
6. Replace the in-memory token when one is found.
7. Repeat approximately every 15 minutes.

This mechanism depends on the third-party website's current browser behavior and authentication implementation. Changes to the website can break token detection without any changes to this repository.

---

## ☁️ Vercel Deployment

The repository includes a `vercel.json` configured to use Vercel's Python runtime for `t2.py`.

Typical deployment flow:

```bash
npm install -g vercel
vercel login
vercel
```

For production:

```bash
vercel --prod
```

### ⚠️ Vercel compatibility note

The current application uses **Selenium + ChromeDriver + a continuously running background thread**. These are a poor fit for Vercel's serverless execution model.

In particular, you should not assume that:

- a Chrome browser and ChromeDriver are available in the Vercel Python runtime;
- a background thread will stay alive between requests;
- a 15-minute infinite loop will persist reliably in a serverless function;
- the token will remain warm in process memory between invocations.

For a production deployment, separate the token-refresh worker from the API and use a platform designed for persistent/background workloads, or replace browser-based token extraction with an official authentication/API mechanism provided by the upstream service.

The Vercel configuration is therefore best treated as an **attempted serverless deployment configuration**, not a guarantee that the full Selenium/token-refresh workflow will work unchanged on Vercel.

---

## 🧪 Development Notes

The bundled `.devcontainer/devcontainer.json` currently contains a `postAttachCommand` that invokes Streamlit:

```text
streamlit run t2.py
```

However, `t2.py` is a **Flask application**, not a Streamlit application. For the current codebase, the development command should instead be equivalent to:

```bash
python t2.py
```

or:

```bash
flask --app t2 run --host 0.0.0.0 --port 5000
```

The forwarded port should also be `5000` rather than Streamlit's default `8501` if Flask is the intended application server.

---

## 🛡️ Privacy & Responsible Use

A PAN is highly sensitive financial identity information. Avoid using this API to perform unauthorized lookups, bulk profiling, identity discovery, or any activity that violates privacy, financial, or data-protection requirements.

Recommended practices:

- Obtain appropriate consent and authorization.
- Minimize collection and retention of PAN data.
- Encrypt data in transit and at rest where applicable.
- Do not store PANs in application logs.
- Restrict access to the API.
- Implement rate limiting and abuse detection.
- Follow applicable Indian data-protection and financial-sector requirements.
- Prefer official APIs and documented integrations whenever available.

---

## 🧩 Environment Variables

The current source code does not yet use environment variables for its upstream configuration. A production-ready implementation should introduce variables such as:

```env
UPSTREAM_BASE_URL=https://example.com/api/...
UPSTREAM_BEARER_TOKEN=replace-me
UPSTREAM_BROKER=turtlemint
UPSTREAM_PROVIDER=signzy
UPSTREAM_TENANT=turtlemint
```

Never commit a real value for `UPSTREAM_BEARER_TOKEN` to source control.

---

## 📦 Dependencies

From `requirements.txt`:

```text
Flask
requests
selenium
webdriver-manager
```

Install them with:

```bash
pip install -r requirements.txt
```

---

## 🐛 Error Handling

The API currently handles two primary error cases:

### Missing input

```text
HTTP 400
```

Returned when the `pan` query parameter is absent.

### Internal/request failure

```text
HTTP 500
```

Returned when the upstream request or JSON processing raises an exception.

The upstream response status code is returned when the request succeeds.

---

## 📈 Suggested Production Improvements

Before exposing this service publicly, consider adding:

- API-key or OAuth authentication
- PAN format validation
- Request rate limiting
- Structured logging with sensitive-data redaction
- Environment-based configuration
- Secret management
- Automated tests
- Health/readiness endpoints
- Request timeouts and retry policy
- Upstream circuit breaking
- Monitoring and alerting
- A persistent worker for browser automation
- An official upstream API/authentication flow instead of extracting browser tokens

---

## 📄 License

No license file is included in the current project.

Add a `LICENSE` file before distributing the repository publicly if you want to define permissions for reuse.

---

## 👤 Author

**Aritra Nath Hazra**

Python • Flask • Automation • AI & Full-Stack Development

---

## ⭐ Contributing

Contributions are welcome for improvements that make the project more reliable, secure, maintainable, and compliant with applicable requirements.

Before submitting changes, please ensure that:

```bash
pip install -r requirements.txt
python t2.py
```

runs successfully in your development environment and that no secrets or real PAN data are committed to the repository.
