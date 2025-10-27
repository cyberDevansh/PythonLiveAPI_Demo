# DeployingAPI

A small Python web API served from `app.py`. This repository contains the application entrypoint, a `Procfile` for production, and a requirements file (`requiredments.txt`).

Short, focused contents:

- `app.py` — the application entrypoint
- `Procfile` — process declaration for production (e.g. `web: gunicorn app:app`)
- `requiredments.txt` — Python dependencies (note: filename contains a typo; keep as-is here)

Requirements
- Python 3.8+ (or your project's supported version)
- pip

Run locally (development)

1. Create and activate a virtual environment:

```powershell
python -m venv venv
```
```
.\venv\Scripts\Activate.ps1
```

2. Install dependencies:
```
pip install -r requiredments.txt
```
3. Start the app for development (if `app.py` runs a dev server):
```
python app.py
```
<!-- 
Deploying to Render (short)

1. Push this repository to GitHub (or connect your Git provider).
2. On Render (render.com) create a new Web Service and connect the repo.
3. Set the Build Command to:

	pip install -r requiredments.txt

4. Set the Start Command to either:

	gunicorn app:app

	or let Render use the `Procfile`.

5. Ensure your app binds to the port provided in the environment (Render sets the $PORT env var).

That's it — this app can be deployed to Render. Deployed via Render. -->

