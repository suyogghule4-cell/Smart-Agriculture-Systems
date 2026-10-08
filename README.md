# Smart Agriculture System (Combined)

All 7 Smart Agriculture tools combined into **one** Flask web app: a single
login, one shared database, a hub page to reach every tool, a unified
dashboard, a single activity history across all tools, and one admin panel.

## Tools included
| Tool | URL | What it does |
|---|---|---|
| 🌾 Crop Recommendation | `/crop` | Best crop to plant from soil & climate readings |
| 🌿 Leaf Disease Detector | `/disease` | Upload a leaf photo → disease/health diagnosis |
| 💧 Smart Irrigation | `/irrigation` | Irrigation decision + liters to use |
| 🧪 Fertilizer Advisor | `/fertilizer` | Fertilizer recommendation from soil/crop/N-P-K |
| 📊 Crop Yield Predictor | `/yield` | Predicted yield (t/ha) & total production |
| 📈 Market Price Forecaster | `/price` | Multi-day price forecast + sell/hold advice |
| 📡 Farm Monitor | `/monitor` | Live multi-zone simulated IoT sensor dashboard |

## Setup
```bash
pip install -r requirements.txt
export ADMIN_PASSWORD='choose-a-strong-password'   # Windows: set ADMIN_PASSWORD=...
export SECRET_KEY='a-long-random-string'            # Windows: set SECRET_KEY=...
PORT=8080 python app.py
```
Visit **http://localhost:8080**. The database and an admin account (`admin`) are
created automatically on first run.

- **`ADMIN_PASSWORD`**: sets the admin password when the admin account is first
  created. If it is not set, a random password is generated and printed once in
  the console. Save it.
- **`SECRET_KEY`**: signs login sessions. Set it in production. If it is not set,
  a random key is used and everyone is logged out when the server restarts.

To reset the admin password on an existing install, delete `database/app.db`
and start again with `ADMIN_PASSWORD` set (this also deletes all user data).

Datasets and pre-trained models are already included, so it works
immediately. To regenerate them:
```bash
python generate_datasets.py   # rebuilds all 5 synthetic datasets
python train_models.py        # retrains all 5 models
```
(The disease detector needs no training — it's rule-based image analysis.
The farm monitor needs no dataset — it simulates sensors directly.)

## How it's organized
- **One login** (`utils/__init__.py`) shared by every tool.
- **One `activities` table** logs every prediction/recommendation from all
  6 predictive tools (crop, disease, irrigation, fertilizer, yield, price)
  with a `tool` column — this is what powers the unified Dashboard and
  History pages.
- **A separate `readings` table** logs the farm monitor's continuous sensor
  stream (a different pattern — many readings per minute rather than one
  action per click).
- **`/home`** is the hub page — a grid of cards linking to each tool.
- Every tool's own page (`crop_index.html`, `crop_result.html`, etc.) shares
  the same visual theme (`static/css/style.css`) as the rest of the suite.

## Notes
As with every project in this series, all training data is synthetically
generated since real datasets weren't available and this environment has no
internet access. Swap in real datasets (same column names — see the
individual project READMEs from earlier for the exact schemas) and re-run
`train_models.py` for production-grade accuracy.

## Python version note
`requirements.txt` intentionally has no pinned versions, so pip installs
whichever build already has a prebuilt wheel for your Python version. If
you're on a very new Python release and hit a "requires GCC" or metadata
error during `pip install`, use Python 3.11 or 3.12 instead:
```
py -3.12 -m venv venv
venv\Scripts\activate        # Windows
pip install -r requirements.txt
```

## Deploying online (permanent link)
The project includes `render.yaml`, so it can be deployed on Render:

1. Push this folder to a GitHub repository.
2. On https://render.com, choose **New → Blueprint** and select the repository.
3. When asked, enter `ADMIN_PASSWORD` (your admin password). `SECRET_KEY` is generated automatically.
4. Deploy. Render gives you a fixed `https://<name>.onrender.com` link. Open it on your phone and log in.

The database is stored on a persistent disk at `/var/data/app.db`, so accounts survive restarts. Persistent disks require a paid Render plan.

Other hosts (Railway, Google Cloud Run, a VPS) work with the `Procfile` start command:
`gunicorn -b 0.0.0.0:$PORT app:app`. Set `SECRET_KEY`, `ADMIN_PASSWORD`, and `DB_PATH` (a path on persistent storage), and set `OPEN_BROWSER=0`.
