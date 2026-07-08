# WhatsApp Linked

![Frappe](https://img.shields.io/badge/Frappe-≥15-blue)
![ERPNext](https://img.shields.io/badge/ERPNext-≥15-red)
![License](https://img.shields.io/badge/License-MIT-green)
![Node](https://img.shields.io/badge/Node-≥18-brightgreen)


![App Icon](images/icon.png)

Send invoices, quotations, and documents via WhatsApp using a linked device. No third-party API required — just scan a QR code from your phone.

Powered by [Baileys](https://github.com/WhiskeySockets/Baileys) (WhatsApp Web API).

---

## Features

| Feature | Description |
|---|---|
| **Send from any document** | WhatsApp Notification channel integrates with standard ERPNext Notifications — works with any doctype, any trigger |
| **Auto-send on submit** | Automatically message customers when a document is submitted; deduplication prevents double-sending |
| **Recurring Campaigns** | Schedule daily/weekly/monthly bulk messaging with Jinja templates, filters, and optional PDF/file attachments |
| **QR device linking** | Connect your WhatsApp by scanning a QR code — no API keys, no third-party service |
| **Session persistence** | Stays connected across restarts; auth saved in site directory (survives Frappe Cloud redeploys) |
| **Retry on failure** | Failed messages queue and auto-retry every 5 minutes |
| **Delivery tracking** | Status pipeline: Queued → Sent → Delivered → Read |
| **PDF generation** | Attach any configured print format as PDF to your message |
| **File attachments** | Send images (JPG/PNG/GIF) alongside your message text |
| **Browser Tunnel mode** | For shared hosting where outbound WebSocket is restricted; routes WhatsApp through your browser tab |
| **Mobile number resolution** | Auto-detects phone from document field, linked Customer, or linked Contact |
| **Notification channel** | Use "WhatsApp" in the standard ERPNext Notification doctype for custom event-driven messaging |

---

## Requirements

| Dependency | Version |
|---|---|
| Frappe | ≥ 15 |
| ERPNext | ≥ 15 |
| Node.js | ≥ 18 |
| wkhtmltopdf | For PDF generation |

---

## Quick Start

1. Install the app on your site
2. Go to **WhatsApp Settings**
3. Choose connection mode:
   - **Direct (Server)** — Use if your server has a unique, reachable IP
   - **Browser Tunnel** — Use on Frappe Cloud or shared hosting (keep the tab open)
4. Click **Connect Device**
5. Scan the QR code with WhatsApp (Menu → Linked Devices → Link a Device)
6. Status changes to **Connected** — you're ready to send

---

## How to Send Messages

### Via Notifications (any doctype)

1. Create a standard ERPNext **Notification**
2. Set **Channel** → **WhatsApp**
3. Configure **Document Type**, **Trigger**, and **Condition**
4. Write your message template using `{{ doc.fieldname }}` syntax
5. Enable **Send PDF** to attach a print format
6. Save — messages send automatically when conditions trigger

### Via Recurring Campaigns (bulk)

1. Go to **WhatsApp Recurring Campaign** → **New**
2. Select **Document Type** (e.g. Sales Invoice) and optional **Filters**
3. Choose **Interval** (Daily / Weekly / Every 15 Days / Monthly)
4. Write your Jinja message template
5. Enable **Send PDF** or upload a **File Attachment** if needed
6. Set **Status** → **Active**
7. Click **Run Now** to test immediately, or wait for the scheduled run

### Via Message Log (retry)

- Go to **WhatsApp Message Log** to monitor all messages
- Status indicators: Queued (orange), Sent (green), Delivered (blue), Read (purple), Failed (red)
- Click **Retry All Failed** to re-send failed messages in bulk
- Individual failed messages show a **Retry Sending** button

---

## Bench Commands

| Command | Description |
|---|---|
| `bench whatsapp-link start-baileys` | Start the WhatsApp Baileys service |
| `bench whatsapp-link stop-baileys` | Stop the service |
| `bench whatsapp-link restart-baileys` | Restart the service |
| `bench whatsapp-link status` | Check if the service is running |
| `bench whatsapp-link install-npm` | Install/update Node.js dependencies |

---

## Connection Modes

### Direct (Server)

WhatsApp Web connects directly from your server. Works when:
- Your server has a unique, publicly reachable IP
- Outbound WebSocket connections to WhatsApp are not blocked

### Browser Tunnel

WhatsApp Web connects through your browser. Required for:
- Frappe Cloud and most shared hosting
- Servers behind NAT or without a dedicated IP
- Environments where outbound WebSocket is restricted

When active, an orange banner shows: **"Browser Tunnel Active — Keep this tab open or pin it."**

---

## Configuration

**WhatsApp Settings** single doctype:

| Field | Default | Description |
|---|---|---|
| Baileys API URL | `http://localhost:3001` | Address of the Node.js service (same machine, override for custom setups) |
| API Key | Auto-generated | Authenticates webhook callbacks from Baileys → Frappe |
| Connection Mode | Direct (Server) | Direct or Browser Tunnel |

The Node.js service runs on `localhost:3001` — no firewall or external access needed since Frappe and Baileys are on the same machine.

---

## Frappe Cloud Deployment

The app is fully compatible with Frappe Cloud:

- **`.env` auto-generated** on each deploy via `after_migrate` — correct webhook URL, auth directory, and site paths
- **Auth persistence** — WhatsApp credentials stored in `sites/site-name/whatsapp_auth_info/` (survives redeploys; no re-scanning QR)
- **Service auto-start** — Baileys service restarts automatically during migration
- **Browser Tunnel** required — Frappe Cloud servers cannot make outbound WebSocket connections to WhatsApp directly

---

## How It Works

```
┌──────────────┐     HTTP API      ┌──────────────┐     WebSocket     ┌─────────┐
│   Frappe     │ ◄──────────────►  │  Baileys     │ ◄──────────────► │ WhatsApp│
│  (Python)    │   localhost:3001  │  Node.js     │                  │  Web    │
│              │                   │  Service     │                  │         │
└──────────────┘                   └──────────────┘                  └─────────┘
        │                                │
        │ Webhook                        │ Auth
        ▼                                ▼
┌──────────────┐                   ┌──────────────┐
│  WhatsApp    │                   │  Auth Info   │
│  Message Log │                   │  (site dir)  │
└──────────────┘                   └──────────────┘
```

1. **Frappe → Baileys**: Sends messages and commands via REST API on `localhost:3001`
2. **Baileys → WhatsApp**: Handles the WhatsApp Web protocol via the Baileys library
3. **Baileys → Frappe**: Sends webhooks for QR codes, connection status, and delivery updates
4. **Auth persistence**: Credentials stored in the site directory survive restarts and redeploys

---

## Message Lifecycle

```
User Action        Log Created       Baileys Sends     WhatsApp Delivers    User Reads
    │                   │                  │                   │                │
    ▼                   ▼                  ▼                   ▼                ▼
  Trigger ──► Queued ──► Sent ──► server_ack ──► delivered ──► read
                │
                ▼
              Failed ──► Retry (auto, every 5 min)
```

---

## License

MIT
