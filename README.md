# DataMind — protecting the build

## The honest position

**You cannot hide client-side code.** The browser must receive everything it
runs, so it is already on the reader's machine. Obfuscation makes the code
*tedious* to read, not secret — one-click deobfuscators exist, and your API
calls stay plainly visible in the network tab.

What actually protects DataMind is that the valuable logic lives on the
**server**: the multi-agent prompts, the ATS rubric and anonymisation, the LLM
chain, your API keys. Someone reading your HTML gets your *interface*, not your
product. The backend hardening below matters more than the obfuscation.

---

## 1. Building the obfuscated front end

```bash
cd build
npm install                 # first time only
node build.js               # balanced  -> ../index.html   (~388 KB)
node build.js --max         # maximum   -> ../index.html   (~681 KB, slower)
```

| Profile | Size | What it does |
|---|---|---|
| **balanced** (default) | 324 → 388 KB | Every name mangled, strings encoded. Defeats casual reading. |
| **--max** | 324 → 681 KB | Adds control-flow flattening and dead code. Much harder to follow — but **doubles the file** and runs slower. On mobile data that is a real cost. |

**Deploy the output (`index.html`). Keep `datamind_frontend.html` — that is your
source.** Never edit the obfuscated file.

### When something breaks

Deploy the **readable** source temporarily. With mangled names:
- console errors read `Uncaught TypeError: a.b is not a function` instead of naming the function
- you cannot search the page source to check whether a fix actually shipped —
  which is exactly how we have diagnosed several deploys

Debug on the readable file, then rebuild.

---

## 2. Backend hardening (this is the part that matters)

Two real exposures were closed in `main.py`:

### `/docs` was public
Anyone could open `your-backend/docs` and get an interactive map of **every
endpoint** — ATS, notebooks, workspaces, all of it. Far more revealing than the
front end. Now off unless you opt in:

```
ENABLE_API_DOCS = true      # only when you need it; remove afterwards
```

### CORS allowed every origin
`allow_origins=["*"]` let any website call your API using a visitor's browser.
Now locked to your domain. Add this Railway variable:

```
ALLOWED_ORIGINS = https://nanakwame7225.github.io
```

Comma-separate for more than one:

```
ALLOWED_ORIGINS = https://nanakwame7225.github.io,https://app.nkaysolutions.com
```

**If you skip this variable** it defaults to your GitHub Pages domain — so the
app keeps working, but a custom domain later will need adding here or the API
will refuse it.

---

## What is still worth doing

- **Rate limiting** — nothing stops someone hammering `/analyse` and burning your
  LLM quota. Worth adding if the app goes public.
- **The Atlas password** pasted in an early session should be rotated.
- Your API keys are already server-side and were never exposed. Good.
