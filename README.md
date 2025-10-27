# DeployingAPI

A small Flask web API (single-file entrypoint `app.py`) that demonstrates user registration, login, and a small products resource backed by SQLite. This README explains how the API works, how to run it locally, and how to test the endpoints using Postman (GET and POST flows for register/login/product add).

## Quick overview

- Language: Python (Flask)
- Storage: SQLite (file `products.db` created next to `app.py`)
- Security: Passwords are hashed with SHA-256 (demo only — see security notes). The `POST /products` endpoint requires a static API token passed as `Authorization: Bearer <API_TOKEN>`.

Files of interest

- `app.py` — Flask application with routes: `/`, `/init`, `/products` (GET, POST), `/register`, `/login`.
- `requirements.txt` — Python dependencies to install.
- `products.db` — created automatically (or by calling `/init`).

## How the API works (endpoints)

All examples assume the app runs at http://localhost:5000 unless you configured a different host/port.

- GET `/` — health endpoint
	- Response: {"msg": "FLASK WEB SERVER is running!(made by Devansh Gupta)"}

- GET `/init` — initialize database
	- Creates `products` and `users` tables if they don't exist.
	- Response: {"msg":"Database Init Complete"}

- POST `/register` — register a new user
	- Request JSON: {"username": "alice", "password": "secret"}
	- On success: 201 Created, {"message":"User registered successfully"}
	- If a username already exists: 409 Conflict, {"error":"Username already exists"}
	- Passwords are stored as SHA-256 hashes.

- POST `/login` — login
	- Request JSON: {"username": "alice", "password": "secret"}
	- On success: 200 OK, {"message": "Welcome alice!"}
	- On failure: 401 Unauthorized, {"error":"Invalid credentials"}

- GET `/products` — list products
	- Response: JSON array of products: [{"id":1,"name":"Product A","price":9.99}, ...]

- POST `/products` — add a product (protected)
	- Requires header: `Authorization: Bearer <API_TOKEN>`
	- Request JSON: {"name": "Widget", "price": 12.5}
	- On success: 201 Created, {"msg":"Product added","product": {"id":2, "name":"Widget","price":12.5}}

Notes about auth/token

- The app reads `API_TOKEN` from the environment: `os.getenv("API_TOKEN")`. Set this environment variable before running the app. The app checks the `Authorization` request header for the exact value `Bearer <API_TOKEN>` when calling the protected `/products` POST endpoint.
- `dotenv` is imported in `app.py` but not loaded — you can either set environment variables in your shell or add `load_dotenv()` to `app.py` if you prefer a `.env` file.

## Setup and run locally (Windows / PowerShell)

1) Create and activate a virtual environment:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

2) Install dependencies:

```powershell
pip install -r requirements.txt
```

3) Set the API token environment variable (Pick a secret token for local testing):

```powershell
#$env:API_TOKEN sets it for the current PowerShell session
$env:API_TOKEN = "my_local_token_here"
```

4) Run the app:

```powershell
python app.py
```

When you start the app with `python app.py` it will call `init_db()` automatically on startup (the file's `__main__` calls it). You can also call `GET /init` manually to ensure tables exist.

The app listens on port 5000 by default.

## Postman: step-by-step testing (register, login, add product, list products)

Create a new Postman environment or use the default. Example variables you can add to the environment:

- base_url = http://localhost:5000
- api_token = my_local_token_here

### 1) Register a user (POST /register)

- Method: POST
- URL: {{base_url}}/register
- Body -> raw -> JSON:

```json
{
	"username": "alice",
	"password": "secret123"
}
```

Expected:
- Status: 201 Created
- Body: {"message":"User registered successfully"}

Postman Tests (tab) to assert success:

```javascript
pm.test("Status is 201", function () {
		pm.response.to.have.status(201);
});

pm.test("Has success message", function () {
		var json = pm.response.json();
		pm.expect(json.message).to.eql('User registered successfully');
});
```

If the username already exists you'll get status 409 and an error JSON.

### 2) Login (POST /login)

- Method: POST
- URL: {{base_url}}/login
- Body -> raw -> JSON:

```json
{
	"username": "alice",
	"password": "secret123"
}
```

Expected:
- Status: 200 OK
- Body: {"message":"Welcome alice!"}

Postman Tests example:

```javascript
pm.test("Login successful", function () {
		pm.response.to.have.status(200);
		var json = pm.response.json();
		pm.expect(json.message).to.include('Welcome');
});
```

### 3) Add a product (POST /products) — protected by API token

- Method: POST
- URL: {{base_url}}/products
- Authorization: (in Postman you can use the Authorization tab) -> Type: Bearer Token -> Token: {{api_token}}
- OR set a header: Key: `Authorization` Value: `Bearer {{api_token}}`
- Body -> raw -> JSON:

```json
{
	"name": "Blue Widget",
	"price": 19.99
}
```

Expected:
- Status: 201 Created
- Body sample: {"msg":"Product added","product":{"id":3,"name":"Blue Widget","price":19.99}}




If the Authorization header is missing or the token doesn't match, you'll get 401 Unauthorized: {"error":"Unauthorized"}.

### 4) List products (GET /products)

- Method: GET
- URL: {{base_url}}/products
- No auth required for GET

Expected: 200 OK and JSON array of products.


## Security notes & recommendations

- Password hashing: The demo stores passwords using SHA-256. This is not recommended for production. Use a password hashing library like `bcrypt`/`passlib` with salts and appropriate work factor.
- Token: The API uses a static token from the environment to protect product creation. For production, implement a proper auth flow (JWT, OAuth, session tokens) and secure storage.
- Consider enabling `load_dotenv()` in `app.py` if you want `.env` file loading. Example:

```python
from dotenv import load_dotenv
load_dotenv()
```

## Troubleshooting

- If you get SQLite errors, delete (or move) `products.db` and restart the app to recreate tables.
- Confirm the `API_TOKEN` environment variable is set in the same shell where you start the app.

## Summary

This repository provides a small Flask API showing registration/login and a protected product-creation endpoint. Use the instructions above to run locally and test with Postman. For production use, replace SHA-256 password hashing with a proper password hashing library and replace the static token with a secure auth system.



