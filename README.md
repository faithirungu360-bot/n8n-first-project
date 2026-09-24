# Imani Hair Brand – AI Customer Support Workflow (n8n)

An n8n automation that takes customer orders/inquiries submitted through a **Tally form**, uses an **AI model (Qwen-Plus)** to draft a personalized support reply to the customer's question, and logs every submission into a **Google Sheet**.

---

## 🧩 What this workflow does

1. **Receives form submissions** — A Tally form posts customer data (name, email, phone, product, size, and a free-text inquiry) to an n8n webhook.
2. **Extracts and structures the data** — The raw payload is mapped into clean, named fields.
3. **Generates a support reply with AI** — The extracted data is passed to an AI model, which drafts a short, polite, professional reply to the customer's specific question only.
4. **Logs the submission** — The customer's details are appended as a new row in a Google Sheet for record-keeping.

---

## 🔁 Workflow diagram

```
Tally Form Submission
        │
        ▼
   [ Webhook ]                  Receives the POST request from Tally
        │
        ▼
 [ Edit Fields ]                Maps raw form data to named fields
        │
        ▼
[ Message a model ]             AI (Qwen-Plus) drafts a reply to the inquiry
        │
        ▼
[ Append row in sheet ]         Saves the submission to Google Sheets
```

---

## ⚙️ Nodes breakdown

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | **Webhook** | `n8n-nodes-base.webhook` | Listens for `POST` requests from the Tally form at path `/Imani_hair_brand`. |
| 2 | **Edit Fields** | `n8n-nodes-base.set` | Pulls specific values out of the Tally payload (`body.data.fields[...]`) and renames them into readable fields: `Full Name`, `Email`, `Phone Number`, `Which products would you like to order?`, `Select size`, `Other inquiries`. |
| 3 | **Message a model** | `@n8n/n8n-nodes-langchain.openAi` | Sends the customer's info and inquiry to the `qwen-plus` model with a system prompt instructing it to answer *only* the customer's question, stay factual, and sign off as "Imani Hair Brand Customer Support". |
| 4 | **Append row in sheet** | `n8n-nodes-base.googleSheets` | Appends a new row containing the customer's details to the connected Google Sheet. |

---

## 📋 Expected Tally form fields (in order)

The **Edit Fields** node reads specific indexes from the Tally payload, so the form fields must be created/ordered like this:

| Index | Field | Mapped to |
|-------|-------|-----------|
| `fields[0]` | Full Name | `Full Name` |
| `fields[1]` | Email | `Email` |
| `fields[2]` | Phone Number | `Phone Number` |
| `fields[5]` | Product selection | `Which products would you like to order?` |
| `fields[7]` | Size selection | `Select size` |
| `fields[8]` | Free-text inquiry | `Other inquiries` |

> ⚠️ **Important:** These are hard-coded index positions. If you add, remove, or reorder fields in your Tally form, you must update the corresponding index numbers in the **Edit Fields** node, or the workflow will pull the wrong data.

---

## 🔑 Credentials required

| Credential | Used by | Notes |
|------------|---------|-------|
| **OpenAI-compatible API key** | Message a model | Configured to call the `qwen-plus` model. You'll need an API key from your provider (e.g., Alibaba Cloud / DashScope for Qwen, or whichever OpenAI-compatible endpoint you're using). |
| **Google Sheets OAuth2** | Append row in sheet | Connect the Google account that owns the target spreadsheet. |

Set these up under **n8n → Credentials** before activating the workflow.

---

## 🚀 Setup instructions

1. **Import the workflow**
   - In n8n, go to **Workflows → Import from File** and select `Customer_support_workflow_.json`.

2. **Connect your credentials**
   - Open the **Message a model** node and select/create your OpenAI-compatible credential.
   - Open the **Append row in sheet** node and select/create your Google Sheets credential.

3. **Point it at your Google Sheet**
   - In **Append row in sheet**, choose the spreadsheet and sheet/tab you want submissions logged to. Make sure the column headers match: `Full Name`, `Email`, `Phone number`, `Which products would you like to order?`, `Select size`, `Other inquiries`.

4. **Connect it to Tally**
   - Activate the workflow in n8n and copy the **Webhook URL** from the Webhook node.
   - In your Tally form settings, add an **Integration → Webhook** and paste the n8n Webhook URL so submissions are sent to n8n.

5. **Match the field order**
   - Confirm your Tally form's field order matches the indexes listed above (or update the **Edit Fields** node to match your form).

6. **Test it**
   - Submit a test entry through your Tally form and confirm:
     - A row appears in your Google Sheet.
     - The AI-generated reply looks correct in the execution log.

---

## 📝 Notes & limitations

- **This workflow does not currently send an email or message to the customer.** The AI-generated reply is produced by the **Message a model** node but is only passed along to the Google Sheets node — it is not automatically emailed or sent back to the customer. If you want the customer to actually receive the reply, you'll need to add an email/messaging node (e.g., Gmail, SMTP, or similar) after **Message a model** and map its output into that node.
- The AI is instructed to answer **only** the customer's specific inquiry, avoid inventing product/pricing/shipping details, and avoid em dashes and asterisks in its reply — useful for keeping tone consistent and avoiding hallucinated promises.
- Since this is a beginner project, it's a good idea to test with a few sample submissions before going live, and to double-check the Tally field indexes any time the form is edited.

---

## 🛠️ Tech stack

- [n8n](https://n8n.io/) — workflow automation
- [Tally](https://tally.so/) — form builder
- Qwen-Plus (via OpenAI-compatible API) — AI reply generation
- Google Sheets — data logging
