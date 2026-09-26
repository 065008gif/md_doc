# Frontend Build Brief — Tata Steel Financial Analysis RAG

**Paste or upload this entire document as the FIRST message to your AI assistant (ChatGPT, Claude, Gemini, or any other).** It contains everything the assistant needs: the project context, the exact API it must build against, the tech stack, the GitHub workflow, the deployment files, and the working rules. No other explanation should be needed.

---

## 0. Instructions to the AI assistant reading this

You are helping a student who has **no prior experience** with web development, Git, GitHub, Node.js, or the terminal. She is building the **frontend** of a two-person college project. Her teammate (Akshit) is building the **backend** separately, on a different laptop and network. They share one GitHub repository. You must guide her from an empty laptop to a finished, pushed, working frontend.

### 0.1 Mandatory working rules (follow these for the entire project, including your first reply)

1. **One instruction per message.** Give one terminal command, or one small action ("open VS Code", "click the green Code button"). Never put several commands in one block unless they are trivially sequential with no decision in between.
2. **Always name the exact application or window.** For example: "open **PowerShell** from the Start menu", "this goes in the **VS Code terminal**", "do this in **Chrome** on github.com". Never assume she knows which window you mean.
3. **State what success looks like before she runs anything.** For example: "you should see `v22.x.x`", "a browser tab should open showing ...". She should be able to tell immediately if something went wrong.
4. **Wait for her to paste the real output before giving the next step.** If she pastes an error, your next reply must diagnose **that specific error** from its actual text. Never give a generic troubleshooting list, and never say "try reinstalling".
5. **Isolate the cause before proposing a fix.** Ask for or run the smallest diagnostic (a version check, a status command, a log) first.
6. **Explain each new tool once, briefly, the first time it appears** (one or two plain sentences).
7. **Ask at most one question per message**, and only when truly necessary. If a reasonable assumption works, state it and continue.
8. **Your very first reply must be a single first step:** find out her operating system (Windows or macOS) and check whether Git and Node.js are already installed. Do not start with a plan, an overview, or a list of questions.
9. **When you write code, give complete files**, not fragments ("replace lines 10–14"). Tell her the exact file path to create or overwrite, relative to the repository root (for example `frontend/src/pages/AskPage.jsx`).
10. **Never invent backend behaviour.** The backend's behaviour is defined only by the API contract in Section 5. If something is unclear, build it against the contract and the mock data, and write down the question for Akshit.
11. **The visual design belongs to her.** She is strong at design: layout, positioning, colour, typography and visual storytelling. This brief fixes **what** the app must do (functionality, data, API contract, safety rules). It deliberately does **not** fix how it looks. Before building each page's look, ask what she has in mind. When she's unsure, offer 2–3 genuinely different directions to choose from (for example "editorial report", "dark analyst terminal", "clean fintech"), then implement **her** choice faithfully, including details she specifies (exact colours, fonts, spacing, placement, motion). Never override her visual decisions with your own taste, and never simplify her design to make coding easier. If a design idea would break a functional rule in this brief (e.g. hiding the verification badge), say so briefly and suggest a way to keep both.
12. **Anticipate beginner pitfalls** instead of letting her hit them. Examples: on Windows, PowerShell may block `npm` scripts ("running scripts is disabled on this system"); if so, tell her to use **Command Prompt**, or run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` in PowerShell. Remind her to stop a running dev server with **Ctrl + C** before running other commands in the same terminal. Tell her which folder she should be in before each command (the prompt path shows it).

---

## 1. Project context (what this system is)

### 1.1 What the whole system does

This is a **financial analysis assistant** built on the official annual reports of **Tata Steel Limited** (an Indian listed steel company) for four fiscal years: **FY2022-23, FY2023-24, FY2024-25 and FY2025-26**. The comparative columns in those reports also give figures for **FY2021-22**.

A user types a question such as *"What was the current ratio from FY2022-23 to FY2025-26 and why did it change?"*. The backend then:

1. uses an AI model (via Ollama Cloud) that decides what data it needs,
2. retrieves **exact, verified figures** from the balance sheet, profit and loss statement and cash flow statement, and searches the notes, auditor's report and management commentary,
3. does all arithmetic in code (never in the AI's head),
4. writes an answer in Markdown, with tables and a **citation for every figure** (report year, statement, page number such as `F143`),
5. runs an automatic **verification check**: every number in the answer must match a number from the source data. The result is returned as, for example, "58/58 figures grounded".

The project is graded on **trustworthiness**: a wrong number stated confidently is the worst possible failure. That leads directly to the most important frontend rule:

> **FRONTEND RULE #1: Never change, recompute, round or reformat financial numbers inside answer text.** Render the Markdown exactly as the backend sends it. In dashboard charts and cards, display the numbers the API gives you, formatted only for readability (Indian digit grouping, fixed decimals). Never calculate financial values (ratios, growth rates, totals) in the frontend.

### 1.2 Who does what

| Person | Responsibility |
|---|---|
| **Akshit** | Backend: PDF parsing, data extraction, search index, AI agent, evaluation, and the **FastAPI** web API described in Section 5. Also the final deployment (Docker Compose on his machine, published through a Cloudflare Tunnel). |
| **Teammate (you)** | Frontend: the website users see. It is built with React and talks to the backend only through the API in Section 5. |

### 1.3 How you work independently

- The backend **will not be running on your laptop**. It needs large data files, a search database and a private API key that only Akshit has.
- So you build against a **mock mode**: a set of fake API responses inside the frontend that follow the contract in Section 5 exactly, field for field. Section 6 specifies it.
- At the end, Akshit pulls your code, switches mock mode off, points the frontend at his real backend, and connects the two.
- **If your frontend works perfectly with the mock data and follows the contract exactly, integration will work on the first try.** That is the goal.

---

## 2. What to build (functional requirements)

A single-page web application with **four pages**. Sections 2.1–2.5 define **what** each page must contain and do. **How it looks is her creative decision** (Section 2.0).

### 2.0 Her creative space vs fixed requirements

**Entirely her choice:**

- Overall visual identity and mood: colour palette, typography (Google Fonts or self-hosted fonts are fine), iconography, illustration, imagery, light and/or dark mode
- Layout and positioning: where navigation lives (top bar, sidebar, floating), where history, sources and steps sit on the Ask page, the dashboard grid, card styles, chart colours and styles
- Motion and micro-interactions: transitions, hover effects, loading animations, and how the "analysing" state feels
- Landing experience: a hero section, a welcome message, how example questions are presented (chips, cards, a carousel...)
- Extra visual touches: a logo or wordmark, a favicon, the page title, empty-state illustrations
- Additional **presentation-only** features she thinks of, e.g. download answer as a `.md` file, print view, keyboard shortcuts, theme toggle

**Fixed (not negotiable):**

- The functional behaviour listed in 2.1–2.5 (every piece of information must be present and working, wherever she places it)
- The API contract (Section 5) and all field names
- **Frontend Rule #1:** financial numbers are displayed, never calculated or altered
- Clear, honest status signals: the verification badge (all three states), the "MOCK DATA" badge, error messages, and the "Backend unreachable" indicator must be **visible and understandable**. She chooses their style, but may not hide or downplay them.
- Readability and accessibility basics: sufficient text contrast, readable table text, keyboard-usable controls, and working on laptop and phone screens

### 2.1 Global layout

- **Title and navigation:** the title "Tata Steel Financial Analysis" and the subtitle "RAG assistant over Annual Reports FY2022-23 – FY2025-26" (she may restyle the wording's presentation, but keep the meaning). Navigation to **Ask**, **Dashboard**, **Evaluation**, **About & Sources**. Placement and style are her choice.
- **Backend status indicator** in the header. On app load, call `GET /api/health`: show "Connected" when `status == "ok"` and "Backend unreachable" if the request fails, clearly distinguishable (e.g. a green or red dot; style is hers). In mock mode, also show a visible "MOCK DATA" badge, so nobody ever mistakes mock numbers for real ones.
- **Footer disclaimer:** "Figures are extracted from Tata Steel's published annual reports. This tool is for educational analysis and is not investment advice."
- **Responsive:** it must be usable on a laptop and on a phone-sized screen. Everything else about the look is hers (Section 2.0). The audience is a professor and finance-minded viewers, so the design should make the data feel trustworthy; how she achieves that is up to her.
- Use **react-router-dom** for routing: `/` (Ask), `/dashboard`, `/evaluation`, `/about`.

### 2.2 Page 1: Ask (the main page, route `/`)

This is the most important page.

**Input area**
- A multi-line text box ("Ask a question about Tata Steel's financials…"), 3 to 2000 characters, with a **Ask** button. Pressing Enter submits; Shift+Enter makes a new line.
- **Example question chips** under the box. Clicking one fills the box and submits it. Use exactly these:
  - "What was Tata Steel's consolidated revenue from operations in FY2025-26?"
  - "Compute Tata Steel's consolidated current ratio for each year from FY2022-23 to FY2025-26 and explain the trend."
  - "What was Tata Steel's consolidated debt-to-equity ratio each year from FY2022-23 to FY2025-26?"
  - "What key audit matters did the auditor raise on the FY2025-26 consolidated financial statements?"
  - "Give an overall financial health verdict for Tata Steel across FY2022-23 to FY2025-26."
  - "What will Tata Steel's revenue be in FY2027-28?" (this demonstrates the system correctly refusing)

**While waiting**
- Answers take **between 5 seconds and 4 minutes** (complex analyses run many steps). This is normal and must be communicated clearly.
- Show a loading panel with a spinner, a **live elapsed-time counter** ("Analysing… 0:37"), and the text "Complex multi-year analyses can take 1–3 minutes. Every figure is being retrieved and verified."
- Show a **Cancel** button that aborts the request (use `AbortController`).
- Client timeout: **300 seconds**. After that, show "The analysis took too long. Please try a narrower question."
- Disable the Ask button while a request is running.

**Answer display** (from the `POST /api/ask` response, Section 5.2)
- Render `answer_markdown` with **react-markdown + remark-gfm**, so tables, bold and lists render. **Do not** enable raw HTML rendering (no `rehype-raw`), for security.
- Markdown tables are central (most answers contain them). They must be **clearly readable** (visible column separation, a distinct header row, numbers easy to scan; she designs the exact look), and must **scroll horizontally on small screens** (wrap tables in a container with `overflow-x: auto`).
- **Verification badge**, prominently near the answer, based on `verification.status`. The wording below is required; the visual style is hers, but the three states must look clearly different (conventionally green / amber / grey):
  - `"verified"`: "✓ Verified: {figures_grounded}/{figures_total} figures traced to source"
  - `"partially_verified"`: "⚠ Partially verified: {figures_grounded}/{figures_total}". Below it, list `unverified_figures` and `warnings`.
  - `"not_applicable"`: "No figures to verify" (for example, a refusal)
- **Sources panel:** list each item in `citations` as "{report} Annual Report · {section} · page {page_label}", as a link to `pdf_url` that opens in a new tab. `pdf_url` already includes `#page=N`, so the browser's PDF viewer jumps to the right page.
- **"How this answer was produced"**: a collapsible section listing `steps` (tool name plus description), then `model` and `elapsed_seconds`.
- A **Copy answer** button (copies `answer_markdown` to the clipboard).

**History**
- Keep a list of this session's questions and answers (newest first); placement and style are her choice. Clicking an entry shows that answer again, without calling the API.
- Save history in `localStorage` (key `finrag_history_v1`, maximum 30 entries), with a "Clear history" button.

**Errors**
- Show errors in a clear red box with the message from the API (see Section 5.8 for how to read it), plus a **Retry** button.

### 2.3 Page 2: Dashboard (route `/dashboard`)

A visual overview. **All numbers come from the API**, never computed in the frontend.

- A **scope toggle**: "Consolidated" (default) or "Standalone". Changing it reloads the data.
- **KPI cards (latest year, FY2025-26)** from `GET /api/financials`: Revenue from operations, Profit for the year, Total assets, Total equity, Total borrowings (show as two cards, non-current borrowings and current borrowings; do not add them yourself). Each card shows the value in ₹ crore with Indian grouping, and the year-on-year change from `yoy_changes` (change and %). Favourable and unfavourable changes must be visually distinguishable (conventionally green and red; the exact colours and treatment are her choice). **Note:** for borrowings, an increase is unfavourable and a decrease favourable.
- **Charts** (use **recharts**), over FY2021-22 to FY2025-26. The data each chart shows is required; the chart types, colours and arrangement below are suggestions she may change:
  1. A bar chart of revenue from operations and profit for the year
  2. A line chart of total assets and total equity
  3. A line chart of the key ratios from `GET /api/metrics`: current ratio, debt-to-equity, interest coverage
  4. A bar chart of the percentage metrics: net profit margin, EBITDA margin, ROE
- **Missing values** (`null` or absent years) must appear as gaps, never as zero. Tooltips should show "n/a".
- **Restated values:** if a value has `"restated": true`, show a small `*` next to it, with the tooltip "Restated in a later annual report".
- Under each chart, show a small source line, e.g. "Source: consolidated statements, Tata Steel Annual Reports".

### 2.4 Page 3: Evaluation (route `/evaluation`)

This shows the professor how well the system was measured. It uses `GET /api/evaluation`.

- If `available` is `false`, show "No evaluation run available yet."
- Otherwise show:
  - The **run date** (`run_at`)
  - **Summary tiles**, one per key in `summary`:
    - Filtered retrieval: Hit Rate@4 and MRR (shown as decimals)
    - Numeric questions fully correct
    - Expected numbers found
    - Source pages retrieved
    - Expected pages cited
    - Abstention correct
    - Answers fully grounded
    - Average seconds per answer
    - Average tool calls
  - A **table of per-question results** (`records`), with columns ID, Category, Question, Numeric, Citations, Abstention (✓/✗), Grounded, Seconds. Show `null` fields as "–".
- Add a short explanation text:

  > "The evaluation separates four tiers so errors can be traced to their source: (1) filtered retrieval: does search find the right page when given the right filters; (2) numeric accuracy: does the final answer contain the correct figures, verified against the PDF; (3) citations: were the correct source pages retrieved and cited; (4) abstention: does the system refuse questions outside the reports (forecasts, other companies, years not covered) while answering valid ones."

### 2.5 Page 4: About & Sources (route `/about`)

- **Sources:** a table built from `GET /api/reports`, with fiscal year, title, page count, and an "Open PDF" link to `pdf_url` that opens in a new tab.
- **"How it works"** section, with this text (you may format it nicely, but keep the meaning):

  > "The four official annual reports are parsed page by page. The primary financial statements (balance sheet, profit and loss, cash flow) are extracted with a coordinate-based parser and checked automatically: total assets must equal total equity plus liabilities, and every section's items must add up to its reported total. Notes to accounts are extracted with Docling's table-structure model, and narrative sections (MD&A, risk management, strategy) are split into searchable passages. Search combines meaning-based (dense) and keyword (BM25) retrieval with filters for year, statement and scope. An AI agent (gpt-oss:120b via Ollama Cloud) decides which figures and passages it needs, computes every ratio in code, and cites every figure. Before an answer is returned, every number in it is checked against the retrieved source data, and restated or reclassified figures are flagged."

- **Limitations** section:

  > "Answers are limited to the four annual reports. The system does not use market prices, forecasts or other companies' data, and it will say so rather than guess. Automatic verification confirms that each number exists in the source data; interpretation should still be read critically."

- **Team:** "Backend: Akshit Kansal · Frontend: [her name]". Ask her what name to use.

---

## 3. Technology stack (fixed, do not substitute)

| Purpose | Choice |
|---|---|
| Runtime | **Node.js LTS** (v22 or v24). Check with `node -v`. |
| Build tool | **Vite**, React template, **JavaScript** (not TypeScript) |
| UI library | **React** (the version the Vite template installs) |
| Routing | `react-router-dom` |
| Markdown | `react-markdown` + `remark-gfm` |
| Charts | `recharts` |
| Styling | **Her choice.** The default is plain CSS with CSS variables (simplest setup). CSS Modules or **Tailwind CSS** are also fine if she prefers; the assistant must then follow Tailwind's *current* official Vite setup guide exactly. **Avoid component libraries that impose their own look** (Bootstrap, MUI, Ant Design, Chakra); they fight a custom design. |
| Visual extras (optional, her choice) | Icons: `lucide-react` or similar. Fonts: Google Fonts or `@fontsource/*`. Animation: CSS transitions or `framer-motion`. Any other small presentation library is fine if it doesn't replace the fixed stack above. |
| HTTP | The browser's built-in `fetch` (no axios) |
| Number formatting | `Intl.NumberFormat('en-IN', { minimumFractionDigits: 2, maximumFractionDigits: 2 })`, which gives Indian grouping like `2,32,139.94` |
| Production server | **nginx** in Docker (files are given in Section 9) |

Install the extra libraries with one command, from inside `frontend/`:

```
npm install react-router-dom react-markdown remark-gfm recharts
```

---

## 4. Repository, accounts and Git workflow

### 4.1 One-time setup (her laptop)

1. **GitHub account:** she needs her own account at github.com. She sends her **GitHub username** to Akshit.
2. **Collaborator invite:** Akshit adds her to the private repository `065008gif/financial-rag` (Settings → Collaborators → Add people). She **accepts the invitation** from the email or from github.com/notifications. **She cannot clone the repo until she accepts.**
3. **Install** (the assistant guides her one step at a time, adapted to her OS):
   - **Git**
   - **Node.js LTS** (this includes npm)
   - **VS Code**
   - **GitHub CLI (`gh`)**, used to log in to GitHub from the terminal through the browser, so she never handles tokens
   - *(Optional)* **Docker Desktop**, only to test the production container at the end. If her laptop can't run Docker, skip it; Akshit will test the container.
4. **Log in to GitHub from the terminal:** `gh auth login` → choose GitHub.com → HTTPS → "Login with a web browser". Then `gh auth setup-git`, so Git uses that login.
5. **Set her Git identity:** `git config --global user.name "Her Name"` and `git config --global user.email "the email on her GitHub account"`. This is her own laptop, so `--global` is fine.

> Why not a personal access token? Fine-grained tokens can't access a repository owned by another user, and classic tokens are easy to leak. The `gh` browser login avoids both problems.

### 4.2 Get the code and create her branch

```
git clone https://github.com/065008gif/financial-rag.git
cd financial-rag
git checkout -b frontend
```

- The repository already contains Akshit's backend code (`src/`, `scripts/`, `eval/`, `.gitignore`, etc.). **She must not modify, move or delete anything outside the `frontend/` folder.** She can ignore all Python files.
- Data files (PDFs, database) are **not** in the repository, and she does not need them.

### 4.3 Create the app inside the repo

From the repository root (`financial-rag/`):

```
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm install react-router-dom react-markdown remark-gfm recharts
```

The result must be exactly `financial-rag/frontend/` containing `package.json`, `vite.config.js`, `index.html`, `src/` and so on. **Not** nested deeper (`financial-rag/frontend/frontend/` is wrong).

Notes:

- `create-vite` may ask extra interactive questions depending on its version (for example about experimental bundlers, or "Install with npm and start now?"). Choose the standard, non-experimental options, and answer **No** to "install and start now", because we run `npm install` ourselves in the next step.
- `npm install` creates **`package-lock.json`**. It **must be committed**, because the production `Dockerfile` uses `npm ci`, which requires it.

### 4.4 Daily workflow

- Work only on the **`frontend`** branch. Check with `git branch` (the current branch has a `*`).
- **Commit small and often**, with clear messages, e.g. `git add frontend` then `git commit -m "Ask page: loading state and cancel"`.
- **Push:** the first time `git push -u origin frontend`, after that just `git push`.
- **Get Akshit's latest changes** occasionally (they only touch other folders, so there should be no conflicts): `git fetch origin` then `git merge origin/main`.
- **Never** use `git push --force`, never commit to `main`, and never commit `node_modules/`, `dist/` or any `.env*` file except `.env.example`.
- **Contract updates:** if Akshit ever needs to change the API, he will commit the updated contract as `docs/API_CONTRACT.md` on `main` and tell her. After each `git merge origin/main`, check whether that file exists or changed (`git log -1 -- docs/API_CONTRACT.md`). If it has, the file takes priority over Section 5 of this brief.
- **When finished:** on github.com, open a **Pull Request** from `frontend` into `main`, titled "Frontend", with 3–5 screenshots of the pages. **Do not merge it**; Akshit merges after integration testing.

### 4.5 `.gitignore` for the frontend

Vite creates `frontend/.gitignore`. Make sure it contains at least:

```
node_modules
dist
.env
.env.*
!.env.example
```

---

## 5. API CONTRACT (build exactly against this)

- **Base path:** every endpoint starts with `/api`. The frontend reads the base from `import.meta.env.VITE_API_BASE_URL`, defaulting to `"/api"`.
- **Format:** all requests and responses are JSON (`Content-Type: application/json`).
- **Units:** all money is in **INR crore**. Fiscal years are strings like `"FY2025-26"`, always in this set: `"FY2021-22"`, `"FY2022-23"`, `"FY2023-24"`, `"FY2024-25"`, `"FY2025-26"`.
- **Scope** is `"consolidated"` (the whole group, the default) or `"standalone"` (the parent company only).
- Any field documented as a number may be `null` when unavailable. The UI must handle `null` everywhere.

### 5.1 `GET /api/health`

**200 response:**

```json
{
  "status": "ok",
  "llm_model": "gpt-oss:120b",
  "index_points": 4393,
  "reports": ["FY2022-23", "FY2023-24", "FY2024-25", "FY2025-26"],
  "version": "1.0.0"
}
```

### 5.2 `POST /api/ask`

**Request body:**

```json
{ "question": "Compute Tata Steel's consolidated current ratio for each year from FY2022-23 to FY2025-26." }
```

`question`: string, required, 3–2000 characters.

**200 response:**

```json
{
  "id": "5f0c2e8a-1d2b-4c1e-9a55-2f1f7e3d9b10",
  "question": "Compute Tata Steel's consolidated current ratio for each year from FY2022-23 to FY2025-26.",
  "answer_markdown": "**Tata Steel – Current Ratio (Consolidated)**\n\n| Fiscal year | Total current assets (₹ cr) | Total current liabilities (₹ cr) | Current ratio |\n|---|---|---|---|\n| FY2022-23 | 86,606.14 | 97,295.13 | 0.89 |\n...",
  "verification": {
    "status": "verified",
    "figures_grounded": 58,
    "figures_total": 58,
    "unverified_figures": [],
    "warnings": []
  },
  "citations": [
    {
      "report": "FY2025-26",
      "section": "consolidated balance sheet",
      "page_label": "F143",
      "pdf_page": 421,
      "pdf_url": "https://www.tatasteel.com/media/25904/tatasteel-iar-2025-26.pdf#page=421"
    }
  ],
  "steps": [
    { "tool": "get_financials", "description": "Fetched total_current_assets, total_current_liabilities (consolidated)" },
    { "tool": "compute_metric", "description": "Computed total_current_assets / total_current_liabilities" }
  ],
  "model": "gpt-oss:120b",
  "elapsed_seconds": 31.4
}
```

Field rules:

- `verification.status` is one of `"verified"`, `"partially_verified"` or `"not_applicable"`.
- `unverified_figures`: array of numbers. `warnings`: array of strings.
- `citations` may be an empty array. `page_label` may be `null` for narrative pages; then display "PDF page {pdf_page}" instead.
- `steps` may be an empty array.
- **Response time is 5–240 seconds.** Use a 300-second client timeout.

### 5.3 `GET /api/financials?items=<comma-separated keys>&scope=<consolidated|standalone>`

Example: `/api/financials?items=revenue_from_operations,profit_for_the_year&scope=consolidated`

**200 response:**

```json
{
  "scope": "consolidated",
  "unit": "INR crore",
  "years": ["FY2021-22", "FY2022-23", "FY2023-24", "FY2024-25", "FY2025-26"],
  "items": {
    "profit_for_the_year": {
      "description": "Profit/(loss) for the year",
      "values": {
        "FY2025-26": { "value": 10885.82, "source": "FY2025-26 Annual Report, consolidated profit and loss, page F145", "restated": false }
      },
      "yoy_changes": [
        { "from": "FY2024-25", "to": "FY2025-26", "change": 7712.04, "change_pct": 242.99, "restated_basis": false }
      ]
    }
  },
  "unknown_items": []
}
```

- `values` contains only the years that exist. Missing years are simply absent.
- `change_pct` may be `null`.
- `restated_basis: true` means the change compares both years as presented in the later report, and a figure was restated or reclassified. Show a `*` with the tooltip "Restated/reclassified between reports".

**Valid item keys** (both scopes, except `non_controlling_interests`, which is consolidated only):
`total_assets`, `total_equity`, `total_non_current_assets`, `total_current_assets`, `total_non_current_liabilities`, `total_current_liabilities`, `property_plant_equipment`, `capital_work_in_progress`, `goodwill`, `non_current_investments`, `inventories`, `current_investments`, `trade_receivables`, `cash_and_cash_equivalents`, `other_bank_balances`, `equity_share_capital`, `other_equity`, `non_controlling_interests`, `non_current_borrowings`, `current_borrowings`, `non_current_lease_liabilities`, `current_lease_liabilities`, `trade_payables`, `revenue_from_operations`, `other_income`, `total_income`, `cost_of_materials_consumed`, `employee_benefits_expense`, `finance_costs`, `depreciation_amortisation`, `other_expenses`, `total_expenses`, `exceptional_items`, `profit_before_exceptional_and_tax`, `profit_before_tax`, `tax_expense`, `profit_for_the_year`, `net_cash_from_operating`, `net_cash_from_investing`, `net_cash_from_financing`, `purchase_of_capital_assets`.

### 5.4 `GET /api/items`

**200 response:**

```json
{
  "consolidated": [ { "key": "total_assets", "description": "Total assets" } ],
  "standalone":   [ { "key": "total_assets", "description": "Total assets" } ]
}
```

### 5.5 `GET /api/metrics?scope=<consolidated|standalone>`

**200 response:**

```json
{
  "scope": "consolidated",
  "years": ["FY2021-22", "FY2022-23", "FY2023-24", "FY2024-25", "FY2025-26"],
  "metrics": [
    {
      "key": "current_ratio",
      "name": "Current ratio",
      "formula": "total_current_assets / total_current_liabilities",
      "unit": "x",
      "decimals": 2,
      "higher_is_better": true,
      "values": { "FY2021-22": 1.0206, "FY2022-23": 0.8901, "FY2023-24": 0.7165, "FY2024-25": 0.7944, "FY2025-26": 0.7456 }
    }
  ]
}
```

- `unit` is `"x"` (a multiple, e.g. 0.75x) or `"%"` (e.g. 11.16%).
- Format each value with the metric's `decimals`.
- The metric keys will be: `current_ratio`, `debt_to_equity`, `interest_coverage`, `net_profit_margin`, `ebitda_margin`, `roe`.
- Any value can be `null`, e.g. ROE in FY2021-22, since there is no prior year to average.

### 5.6 `GET /api/evaluation`

**200 response when a run exists:**

```json
{
  "available": true,
  "run_at": "2026-09-26T09:20:00",
  "summary": {
    "tier1_filtered_retrieval": { "hit_rate@4": 1.0, "mrr": 1.0 },
    "numeric_questions_fully_correct": "2/2",
    "expected_numbers_found": "5/5",
    "source_pages_retrieved": "2/2 questions",
    "expected_pages_cited": "7/7",
    "abstention_correct": "3/3",
    "answers_fully_grounded": "3/3",
    "avg_seconds": 90,
    "avg_tool_calls": 5.0
  },
  "records": [
    { "id": "A1", "category": "lookup", "question": "What was Tata Steel's consolidated revenue from operations in FY2025-26?",
      "numeric": "1/1", "citation": "1/1", "mentions": null, "abstention_ok": true, "grounded": "1/1", "seconds": 8 }
  ]
}
```

When no run exists: `{ "available": false }`.

### 5.7 `GET /api/reports`

**200 response** (these are the real values; use them in the mock too):

```json
[
  { "fiscal_year": "FY2022-23", "title": "116th Integrated Report & Annual Accounts 2022-23", "pages": 597,
    "pdf_url": "https://www.tatasteel.com/media/18370/tata-steel-ir-2022-23.pdf" },
  { "fiscal_year": "FY2023-24", "title": "117th Integrated Report & Annual Accounts 2023-24", "pages": 582,
    "pdf_url": "https://www.tatasteel.com/media/21244/integrated-report-and-annual-accounts-fy2023-24.pdf" },
  { "fiscal_year": "FY2024-25", "title": "118th Integrated Report & Annual Accounts 2024-25", "pages": 550,
    "pdf_url": "https://www.tatasteel.com/media/23973/fy25-integratedreport.pdf" },
  { "fiscal_year": "FY2025-26", "title": "119th Integrated Report & Annual Accounts 2025-26", "pages": 582,
    "pdf_url": "https://www.tatasteel.com/media/25904/tatasteel-iar-2025-26.pdf" }
]
```

### 5.8 Errors (all endpoints)

- **Validation error (HTTP 422, FastAPI's standard format):**
  `{ "detail": [ { "loc": ["body", "question"], "msg": "String should have at least 3 characters", "type": "string_too_short" } ] }`
- **Other errors (HTTP 400, 503 or 500):** `{ "error": "Human-readable message" }`. A 503 means the AI model is temporarily unavailable.
- **How the frontend reads the message:** use `data.error` if present. Otherwise, if `data.detail` is an array, join its `msg` fields with "; ". Otherwise use `data.detail` as a string. Otherwise use `"Request failed (HTTP {status})"`. A network failure (no response) shows "Cannot reach the backend. Is it running?"

---

## 6. Mock mode (how you develop without the backend)

### 6.1 Environment variables

| Variable | Meaning | Default |
|---|---|---|
| `VITE_USE_MOCK` | `"true"` makes every API function return mock data (no network) | `"false"` |
| `VITE_API_BASE_URL` | API base path | `"/api"` |
| `VITE_PROXY_TARGET` | Where the dev server forwards `/api` requests (used by Akshit at integration) | `"http://localhost:8000"` |

Files in `frontend/`:

- **`.env.example`** (committed), which documents the three variables with their defaults.
- **`.env.development.local`** (NOT committed), on her laptop only, containing `VITE_USE_MOCK=true`.

### 6.2 API client design

Create `frontend/src/api/client.js` exporting one object, `api`, with these functions: `health()`, `ask(question, signal)`, `financials(items, scope)`, `items()`, `metrics(scope)`, `evaluation()`, `reports()`. Every page uses **only** this object, and never calls `fetch` directly. Inside it:

- If `import.meta.env.VITE_USE_MOCK === "true"`, return data from `src/api/mockData.js` after an artificial delay: **4 seconds for `ask`** (so the loading UI can be tested) and 300 ms for everything else. Respect the `signal`, so Cancel works in mock mode too.
- Otherwise, call `fetch(BASE + path)` with a 300-second timeout (`AbortController` combined with the caller's signal). Parse JSON. On non-2xx responses, throw an `Error` with the message rule from Section 5.8.

**Mock `ask` behaviour** (so every UI state can be tested):

- A question containing **"JSW"**, **"FY2027"** or **"FY2009"**: return a refusal answer, with `verification.status = "not_applicable"`, `figures_total = 0`, and the answer text "I could not find this in the reports. They cover Tata Steel only, for FY2021-22 to FY2025-26, and contain no forecasts."
- A question containing **"error test"**: reject with `Error("The AI model is temporarily unavailable (mock 503)")`.
- A question containing **"partial"**: return the standard answer, but with `status = "partially_verified"`, `figures_grounded = 56`, `figures_total = 58`, `unverified_figures = [19.1, 7.5]`, and `warnings = ["Unsupported claims: example warning"]`.
- **Anything else:** return the standard current-ratio answer below.

### 6.3 Mock data (real, verified figures, so the mock looks realistic)

**Standard ask answer**: use this as `answer_markdown` (it's a shortened real answer):

```markdown
**Tata Steel – Current Ratio (Consolidated)**

| Fiscal year | Total current assets (₹ cr) | Total current liabilities (₹ cr) | Current ratio | Change vs prior year | % change |
|---|---|---|---|---|---|
| FY2022-23 | 86,606.14 | 97,295.13 | 0.8901 | n/a | n/a |
| FY2023-24 | 70,503.59 | 98,403.48 | 0.7165 | -0.1737 | -19.51% |
| FY2024-25 | 68,391.54 | 86,093.55 | 0.7944 | +0.0779 | +10.87% |
| FY2025-26 | 72,265.11 | 96,917.64 | 0.7456 | -0.0488 | -6.14% |

*Current ratio = Total current assets ÷ Total current liabilities (consolidated balance sheets, FY2023-24 AR p. F144–F145, FY2024-25 AR p. F150–F151, FY2025-26 AR p. F142–F143).*

**What drove the FY2025-26 decline:** current liabilities rose by ₹10,824.09 crore (+12.57%), led by trade payables to creditors other than micro and small enterprises (+₹5,148.34 crore, 47.6% of the increase) and provisions (+₹2,159.91 crore, 20.0%), while current assets rose by only ₹3,873.57 crore (+5.66%), mainly inventories (+₹2,659.28 crore).
```

Mock citations: three entries for `FY2025-26`, `consolidated balance sheet`, `F142` (pdf_page 420) and `F143` (pdf_page 421), plus `FY2024-25`, `F150`, pdf_page 404. Build each `pdf_url` from the report URLs in Section 5.7, plus `#page=N`. Mock steps: `get_financials`, `compute_metric`, `breakdown`. Model: `"gpt-oss:120b"`. `elapsed_seconds`: 31.4. Verification: verified, 58/58.

**Mock financials** (consolidated). These are real values. Absent years stay absent, which tests gap handling:

| Item | FY2021-22 | FY2022-23 | FY2023-24 | FY2024-25 | FY2025-26 |
|---|---|---|---|---|---|
| profit_for_the_year | 41,749.32 | 8,075.35 | -4,909.61 | 3,173.78 | 10,885.82 |
| total_assets | 2,85,445.60 | 2,88,021.74 | 2,73,423.50 | 2,79,394.80 | 3,01,254.11 |
| total_equity | 1,17,098.46 | 1,05,175.21 | 92,432.74 | 91,352.78 | 1,03,780.45 |
| non_current_borrowings | – | 51,446.33 | 51,576.73 | 68,551.81 | 63,753.67 |
| current_borrowings | – | 26,571.37 | 29,997.19 | 20,412.00 | 21,202.72 |
| revenue_from_operations | – | – | – | – | 2,32,139.94 |

Mock `yoy_changes` are simple differences between consecutive available years, for **mock display only**. In the real system they come from the API. For the mock, you may hardcode them.

For standalone scope in mock mode, it's fine to return the same structure with fewer items (e.g. only `revenue_from_operations` FY2025-26 = 1,39,720.22).

**Mock metrics** (consolidated, real values, `null` = not available in the mock):

| key | name | unit | decimals | higher_is_better | FY2021-22 | FY2022-23 | FY2023-24 | FY2024-25 | FY2025-26 |
|---|---|---|---|---|---|---|---|---|---|
| current_ratio | Current ratio | x | 2 | true | 1.0206 | 0.8901 | 0.7165 | 0.7944 | 0.7456 |
| debt_to_equity | Debt-to-equity | x | 2 | false | null | 0.7418 | 0.8825 | 0.9738 | 0.8186 |
| roe | Return on equity (avg equity) | % | 2 | true | null | 7.27 | -4.97 | 3.45 | 11.16 |
| net_profit_margin | Net profit margin | % | 2 | true | null | null | null | null | 4.69 |
| interest_coverage | Interest coverage | x | 2 | true | null | null | null | null | null |
| ebitda_margin | EBITDA margin | % | 2 | true | null | null | null | null | null |

**Mock evaluation:** use the JSON example in Section 5.6. The three records are real results: A1 (numeric "1/1", citation "1/1", grounded "1/1", 8 s), C1 (category "ratio", numeric "4/4", citation "6/6", grounded "315/315", 30 s), and H2 (category "abstain", question "What will Tata Steel's revenue be in FY2027-28?", numeric null, citation null, abstention_ok true, grounded "0/0", 233 s).

**Mock health:** the JSON in Section 5.1. **Mock reports:** the JSON in Section 5.7.

**Negative values** (e.g. FY2023-24 profit of -4,909.61) must always show a minus sign and be visually marked as negative on cards and in tooltips (colour or treatment is her choice). Charts must plot them below zero.

---

## 7. Folder structure (starting point)

The API layer (`src/api/client.js`, `src/api/mockData.js`) and the production files must keep these exact names and locations. Components and pages can be renamed, split or added to however her design needs.

```
financial-rag/
└── frontend/
    ├── .env.example
    ├── .gitignore
    ├── .dockerignore
    ├── Dockerfile
    ├── nginx.conf
    ├── README.md
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── main.jsx
        ├── App.jsx                  (router + layout)
        ├── styles.css
        ├── api/
        │   ├── client.js
        │   └── mockData.js
        ├── utils/
        │   └── format.js            (formatINR, formatMetric, formatPct; formatting only, no calculations)
        ├── components/
        │   ├── Header.jsx
        │   ├── Footer.jsx
        │   ├── StatusIndicator.jsx
        │   ├── MarkdownAnswer.jsx
        │   ├── VerificationBadge.jsx
        │   ├── Citations.jsx
        │   ├── StepsPanel.jsx
        │   ├── LoadingPanel.jsx
        │   ├── ErrorBox.jsx
        │   ├── HistoryList.jsx
        │   ├── KpiCard.jsx
        │   └── ScopeToggle.jsx
        └── pages/
            ├── AskPage.jsx
            ├── DashboardPage.jsx
            ├── EvaluationPage.jsx
            └── AboutPage.jsx
```

---

## 8. Development server configuration

`frontend/vite.config.js` must contain the `/api` proxy, so that at integration time Akshit only has to set one variable:

```js
import { defineConfig, loadEnv } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  return {
    plugins: [react()],
    server: {
      port: 5173,
      proxy: {
        '/api': {
          target: env.VITE_PROXY_TARGET || 'http://localhost:8000',
          changeOrigin: true,
          timeout: 300000,
          proxyTimeout: 300000,
        },
      },
    },
  }
})
```

Run it with `npm run dev` (from inside `frontend/`), then open **http://localhost:5173** in the browser.

---

## 9. Production files (for the final Docker deployment)

In production, Akshit runs everything with Docker Compose on his machine: a `backend` container on port 8000, and your `frontend` container, which serves the built site with nginx and forwards `/api` to the backend. Create these exactly.

**`frontend/Dockerfile`**

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV VITE_API_BASE_URL=/api
ENV VITE_USE_MOCK=false
RUN npm run build

FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

**`frontend/nginx.conf`**

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Docker's internal DNS; resolving at request time lets nginx start even when the backend is absent
    resolver 127.0.0.11 valid=30s ipv6=off;
    set $backend_upstream http://backend:8000;

    location /api/ {
        proxy_pass $backend_upstream;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;
        proxy_buffering off;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

The 300-second timeouts are essential: answers can take up to 4 minutes, and nginx's default of 60 seconds would cut them off.

**`frontend/.dockerignore`**

```
node_modules
dist
.env
.env.*
```

**Optional test, only if she has Docker Desktop:** from `frontend/`, run `docker build -t finrag-frontend .` then `docker run --rm -p 8080:80 finrag-frontend`, and open http://localhost:8080. The pages should load. API calls will fail because there's no backend, and the UI should show its "Backend unreachable" state gracefully. That's the expected result, not a bug.

---

## 10. Build order (milestones) — do them in this order and commit after each

1. **Setup:** tools installed, `gh` login working, repo cloned, `frontend` branch created, Vite app created in `frontend/`, `npm run dev` shows the Vite starter page. **Commit and push.**
2. **Design direction (her call):** the assistant asks about her vision (mood, palette, fonts, layout, references she likes), or offers 2–3 directions if she wants ideas. It then sets up the design foundation she chooses (colour and typography tokens, global styles). **Commit.**
3. **Skeleton:** routing, navigation, footer, four empty pages in her chosen design, `format.js`. **Commit.**
4. **API layer:** `client.js` + `mockData.js` + `.env.example` + `.env.development.local` (`VITE_USE_MOCK=true`), the health indicator, and the MOCK DATA badge. **Commit.**
5. **Ask page:** input, example chips, loading with timer and cancel, Markdown answer with styled tables, verification badge (all three states), citations with PDF links, steps panel, copy button, history with localStorage, error box with retry. Test all four mock behaviours (normal, refusal, error, partial). **Commit.**
6. **Dashboard:** scope toggle, KPI cards, the four charts, null gaps, negative values, restated markers. **Commit.**
7. **Evaluation page** and **About & Sources page.** **Commit.**
8. **Polish:** mobile layout, keyboard use (Enter to submit, focus states), no console errors, `npm run build` succeeds with no errors. **Commit.**
9. **Production files:** `Dockerfile`, `nginx.conf`, `.dockerignore`, `vite.config.js` proxy. **Commit.**
10. **`frontend/README.md`**, containing: prerequisites, `npm install`, `npm run dev`, how to toggle mock mode, how to point at a real backend (`VITE_USE_MOCK=false` and `VITE_PROXY_TARGET=http://localhost:8000`), `npm run build`, and the Docker commands. **Commit and push.**

---

## 11. Acceptance checklist (all must be true before handoff)

- [ ] `git branch` shows she is on `frontend`, and `git status` is clean after the final push.
- [ ] Everything she created is inside `frontend/`. Nothing outside it was changed (`git diff --stat origin/main` lists only `frontend/...` paths).
- [ ] `npm run build` completes with no errors.
- [ ] In mock mode, all four pages work, with no errors in the browser console (F12 → Console).
- [ ] Ask page: normal, refusal, partial-verification and error states all display correctly. Cancel works. The timer counts. History survives a page reload.
- [ ] Markdown tables render as real tables and scroll horizontally on a narrow window.
- [ ] Citation links open the correct PDF in a new tab.
- [ ] Dashboard: nulls appear as gaps (never zero), negatives are red and plotted below zero, and the scope toggle works.
- [ ] No financial number is calculated in the frontend (only formatting).
- [ ] All API calls go through `src/api/client.js` and use the relative base `/api` (no hard-coded `http://localhost:8000` anywhere in `src/`).
- [ ] `.env.development.local` and `node_modules` are **not** committed.
- [ ] `frontend/package-lock.json` **is** committed (the Dockerfile needs it).
- [ ] The design reflects her choices, and every fixed status signal (verification badge, MOCK DATA badge, errors, backend status) is visible.
- [ ] `frontend/README.md` explains how to run it.

---

## 12. Things she must NOT do

- Do not modify anything outside `frontend/`.
- Do not commit to `main` or force-push.
- Do not commit `node_modules`, `dist`, or any `.env*` file except `.env.example`.
- Do not switch to TypeScript or axios, and do not add component libraries that impose their own look (Bootstrap, MUI, Ant Design, Chakra). Styling tools and visual extras from Section 3 are fine.
- Do not compute or "correct" any financial number.
- Do not render raw HTML from answers.
- Do not change the API field names. If she believes the contract needs a change, she writes it down for Akshit instead.
- Do not put any API keys or passwords in the frontend. The frontend needs none.

---

## 13. Handoff message (send to Akshit when done)

When the checklist is complete, the assistant helps her send Akshit a message like this:

> "Frontend is done and pushed to branch **`frontend`** (latest commit: `<paste the output of git log -1 --oneline>`). To run it: `git fetch origin && git checkout frontend && cd frontend && npm install`, create `frontend/.env.development.local` with `VITE_USE_MOCK=false` and `VITE_PROXY_TARGET=http://localhost:8000`, start your backend on port 8000, then `npm run dev` and open http://localhost:5173. The production `Dockerfile` and `nginx.conf` are in `frontend/` (nginx proxies `/api/` to `http://backend:8000` with a 300 s timeout, so the Compose service must be named `backend`). Open questions for you: `<list any, or 'none'>`."

---

## 14. Glossary (for the assistant to explain when needed)

- **RAG (Retrieval-Augmented Generation):** an AI system that first retrieves relevant source material, then writes an answer based on it, instead of answering from memory.
- **Consolidated vs standalone:** consolidated is the whole Tata Steel group, including overseas subsidiaries in the Netherlands and UK. Standalone is only the Indian parent company.
- **₹ crore:** 1 crore = 10 million. Indian grouping writes 2,32,139.94 (not 232,139.94).
- **F143:** the printed page number in the financial statements section of the annual report. `pdf_page` is the page number in the PDF file.
- **Restated:** a later report changed a prior year's figure (e.g. after a merger or reclassification). The system flags this.
- **API:** the set of URLs the frontend calls to get data from the backend.
- **Mock:** fake but realistic API responses used while the real backend isn't available.
