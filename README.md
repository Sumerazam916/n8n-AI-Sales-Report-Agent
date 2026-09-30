# AI Sales Report Agent (n8n)

An n8n workflow where you ask questions about a sales spreadsheet in plain English and an AI agent answers in chat. If you ask, it also emails you the result.

Example: *"Find my top 5 customers by total sales and email me their details."*

<!-- Add screenshots to the screenshots/ folder and link them here, e.g.
![Workflow canvas](screenshots/workflow.png)
![Chat answer](screenshots/chat.png)
![Email received](screenshots/email.png)
-->

## How it works

```
Chat trigger  →  AI Agent  ←  Anthropic Claude (chat model)
                    ↑   ↑
                    |   └── Simple Memory (last 10 messages)
                    ├────── Tool: Google Sheets (read rows)
                    └────── Tool: Gmail (send email)
```

The chat model is swappable — this workflow works the same with Google Gemini, OpenAI, or any other n8n chat model node. Swap it by deleting the model node and connecting a different one to the AI Agent's "Chat Model" input; nothing else needs to change.

- The **AI Agent** decides when to look up data (Google Sheets tool) and when to send an email (Gmail tool).
- The Gmail tool uses n8n's `$fromAI()` so the agent writes the subject and body itself. The recipient address is fixed, so the AI cannot email anyone else.
- The system prompt tells the agent to answer in chat by default and only send email when explicitly asked.

## Files

| File | What it is |
|---|---|
| `workflow.json` | The n8n workflow (credentials and personal details removed) |
| `data/sales_data_sample.csv` | Sample dataset to test with |

## Demo Video

https://github.com/user-attachments/assets/e37f8c7b-c85f-4ffe-9985-b1c9d3669c43



## Setup

**1. Put the dataset in Google Sheets**
1. Go to [sheets.new](https://sheets.new).
2. File → Import → Upload → choose `data/sales_data_sample.csv`.
3. Choose "Create new spreadsheet" (or "Replace spreadsheet" in the blank one) and import.

**2. Import the workflow into n8n**
1. In n8n: Workflows → ⋯ menu → Import from file → choose `workflow.json`.

**3. Add your own credentials** (none are included in this repo)
- **Chat model**: the workflow ships wired to Anthropic's Claude. You'll need your own Anthropic API key, or swap the model node for a free alternative like **Google Gemini** (free API key from [Google AI Studio](https://aistudio.google.com)) — see the note above on swapping the chat model.
- **Google Sheets**: OAuth2 connection to your Google account.
- **Gmail**: OAuth2 connection to your Google account.

**4. Point the nodes at your data**
- Open the **chat model** node and confirm a valid model is selected from its dropdown — the model ID saved in this file may not exist on your account or may have been superseded, so pick whatever's current once your credential is connected.
- Open the **Google Sheets** node and select your spreadsheet and its first sheet from the dropdowns (the ID in the file is a placeholder).
- Open the **Gmail** node and replace `YOUR_EMAIL@example.com` with your address.

**5. Test**
Open the chat in n8n and try the prompts below.

## Test prompts and expected answers

The dataset has 42 order lines from 29 customers, so ask for *total* sales per customer when you want customer rankings.

| Prompt | Expected answer |
|---|---|
| "Which row has the highest sales? Who is the customer?" | Order 10150, sales 10,993.50, Dragon Souveniers, Ltd. (Classic Cars) |
| "Top 5 customers by total sales" | 1. Dragon Souveniers, Ltd. (21,987.00) · 2. Technics Stores Inc. (18,227.96) · 3. Australian Gift Network, Co (16,029.64) · 4. Baane Mini Imports (15,203.62) · 5. Euro Shopping Channel (15,032.16) |
| "Find the top 5 customers by total sales and email me their info" | Same top 5, delivered to the email address set in the Gmail node |

Language models can miscalculate, so compare the agent's numbers against the CSV, especially before trusting it on your own data.

## About the dataset

`data/sales_data_sample.csv` has 42 rows and 25 columns (order number, quantity, price, sales, order date, status, product line, customer, contact and address fields, deal size), covering 2003–2005. It is a small extract of the widely shared "Sample Sales Data" (Classic Models) dataset, which contains fictional companies.

- Source: *add the link to where you downloaded it, and check its license before redistributing.*
- One `PHONE` cell shows `#ERROR!`. This came from the spreadsheet export and is left unchanged from the original.

## Known limitations and ideas

- **Large sheets can be slow.** If the agent has to read and reason over every row, long runs can time out on n8n Cloud (I hit a gateway timeout during testing). A better design is to sort and limit the rows with n8n nodes first, and give the AI only the top results.
- The agent depends on the model correctly choosing tools, so results can vary between runs.
- Possible next steps: send the email as formatted HTML, add a scheduled daily report, and connect a SQL database instead of a sheet.

## What I learned building this

- An agent only uses tools the way they are described, so tool descriptions and the system prompt matter.
- Connecting a node to the agent as a **tool** (optional, chosen by the AI) is different from wiring it as the next step (always runs).
- `$fromAI()` lets the agent fill in fields like the email body at runtime.
- The chat model is just one input to the AI Agent node, so it's easy to swap between providers (I moved between Google Gemini and Anthropic Claude while testing) without touching the rest of the workflow.
