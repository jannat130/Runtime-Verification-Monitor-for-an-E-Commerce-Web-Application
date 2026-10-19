# RV Monitor — Runtime Verification Monitor & Dashboard

A standalone Python application that watches the e-commerce app's activity log in real time, checks it against a set of business rules, and displays any violations on a live dashboard.

This component is independent of the ecommerce application — it only reads a log file the app writes to. It does not need to run for the storefront itself to work, and the storefront does not need to know this monitor exists.

## What's in this folder

| File | Purpose |
|---|---|
| `config.py` | The one file you edit — points the monitor at your log file |
| `tailer.py` | Watches the log file and yields new events as they're written |
| `rules.py` | The three business rules and the engine that checks events against them |
| `monitor.py` | Main script — run this to start watching |
| `dashboard.py` | Streamlit dashboard — run this to view results live in a browser |

## Prerequisites

- **Python 3.12 or newer** — download from [python.org/downloads](https://www.python.org/downloads). During install, check **"Add python.exe to PATH."**
- The ecommerce application already set up and running (see its own README), with `storage/logs/events.log` being written to.

## Setup (one-time)

### 1. Place this folder somewhere simple

Anywhere on your computer is fine, e.g. `D:\rv_monitor`.

> **Avoid special characters** (`&`, `%`, `$`, etc.) anywhere in the folder path. An `&` in a path is known to silently break the virtual environment's activation script on Windows — this cost real setup time during development. Stick to letters, numbers, underscores, and hyphens.

### 2. Create an isolated Python environment

This keeps the monitor's packages separate from anything else installed on your machine and avoids permission errors.

Open Command Prompt, navigate to this folder (using `/d` if it's on a different drive than `C:`):

```
cd /d D:\rv_monitor
py -3.12 -m venv venv
venv\Scripts\activate
```

You'll know activation worked if `(venv)` appears at the very start of your prompt. Confirm the right Python is active:

```
python --version
```

This should print `Python 3.12.x`. If it prints something older (e.g. `3.7.x`), something else on your PATH is being picked up instead — delete the `venv` folder and recreate it before continuing.

### 3. Install dependencies

```
pip install streamlit pandas
```

### 4. Point the monitor at your log file

Open `config.py` and change this line to match your actual ecommerce folder name:

```python
EVENTS_LOG_PATH = r"C:\xampp\htdocs\ecommerce\storage\logs\events.log"
```

## Running it

You need **two terminal windows** open at the same time, both with the virtual environment activated.

**Terminal 1 — the monitor:**
```
cd /d D:\rv_monitor
venv\Scripts\activate
python monitor.py
```
Leave this running. It prints a line every time it sees a new event from the website, and writes any rule violations to `violations.jsonl`.

**Terminal 2 — the dashboard:**
```
cd /d D:\rv_monitor
venv\Scripts\activate
streamlit run dashboard.py
```
This opens `http://localhost:8501` in your browser automatically, showing live events and violations, refreshing every few seconds. If asked for an email on first run, just press Enter to skip it.

## Every time after the first setup

You don't need to recreate the `venv` or reinstall packages again — just activate and run:

```
cd /d D:\rv_monitor
venv\Scripts\activate
python monitor.py
```
(and the same for `dashboard.py` in a second terminal)

## The Three Rules Monitored

| Rule | Watches | Flags a violation when |
|---|---|---|
| Duplicate Payment | `payment_recorded` events | More than one payment is recorded for the same order |
| Out-of-Stock Order | `stock_checked` events | Requested quantity exceeds available stock |
| Duplicate Discount | `discount_applied` events | The same discount code is applied more than once to one order |

## Testing / Demo Flags

ecommerce's `.env` file has two flags that control whether each defect is actually preventable or deliberately left open, so you can demonstrate detection on demand:

```
SIMULATE_STOCK_CHECK=true                 # true = order blocked if it exceeds stock
SIMULATE_DOUBLE_PAYMENT_GUARD=true        # true = second payment for one order is rejected
```

Set either to `false` (then run `php artisan config:clear` in the ecommerce folder) to let that specific defect through and confirm the monitor catches it. Set back to `true` to confirm the guard correctly blocks it.

There's no flag for the discount rule — no server-side guard against reapplying a discount exists in the current checkout flow, so it can be demonstrated directly without toggling anything.

## Troubleshooting

**"Access is denied" or strange package errors during `pip install`**
pip is probably installing into the wrong Python. With the venv activated, run `where python` — the first result should point inside `rv_monitor\venv`. If it doesn't, delete the `venv` folder and recreate it (Setup, Step 2).

**"The system cannot find the path specified" when activating**
Either you're not actually in the `rv_monitor` folder (run `cd` alone to check), or you're on a different drive and used plain `cd` instead of `cd /d` to switch to it.

**`TypeError: '>' not supported between instances of 'str' and 'int'`**
This means an old copy of `rules.py` is in use — Laravel sometimes sends quantities as strings. Make sure you're using the current `rules.py`, which converts values to numbers before comparing them.

**Dashboard shows no events even after placing an order**
- Double-check the path in `config.py` matches your actual ecommerce folder name.
- Confirm `storage/logs/events.log` exists and is growing after an order (open it in Notepad and check).
- Make sure `monitor.py` is still running in Terminal 1 — if it crashed, check the error and restart it.

**Changed a `.env` flag but behaviour didn't change**
Run `php artisan config:clear` in the ecommerce project folder after every `.env` edit — Laravel caches configuration and won't pick up changes otherwise.
