# Secure Notes API

A compact FastAPI application for exploring secure backend architecture, authentication, authorization, encryption at rest, and explicit trust boundaries.

The goal of this project is not to present a production-ready notes service. It is to make the security decisions in a small backend system visible, understandable, and easy to inspect.

## What it demonstrates

- User registration and authentication
- Short-lived JWT-based access control
- Password hashing with bcrypt
- Application-level encryption of note content using Fernet
- SQLAlchemy-backed persistence
- Environment-based secret configuration
- Separation of authentication, encryption, database, and routing responsibilities
- Automated tests for core authentication and note workflows

## Security model

A request passes through several distinct trust transitions rather than being treated as implicitly safe after login:

1. A user authenticates with a username and password.
2. The server verifies the password hash and issues a signed, expiring JWT.
3. Protected routes validate the token before authorizing access.
4. Note ownership is enforced at the application layer.
5. Note content is encrypted before database persistence.
6. Ciphertext is decrypted only when an authorized user retrieves the note.

This separation is intentional. Authentication establishes identity; it does not automatically make every action or data access trustworthy.

## Password security

Passwords are never stored in plaintext. They are hashed with bcrypt through Passlib before persistence, and login attempts are verified against the stored hash.

For a production service, password policy, rate limiting, credential monitoring, stronger session controls, and centralized secret management would be additional requirements.

## Encryption at rest

Note content is encrypted using Fernet symmetric encryption before it is written to the database. The encryption key is supplied through environment configuration rather than embedded in source code.

A production deployment should replace local environment-managed keys with a dedicated secrets or key-management service such as AWS KMS, Azure Key Vault, HashiCorp Vault, or an HSM-backed solution.

## Architecture

```text
app/
  config.py        Application configuration
  db.py            Database initialization
  models.py        Database models
  schemas.py       Request/response schemas
  security.py      Authentication and encryption logic
  routes_auth.py   Authentication endpoints
  routes_notes.py  Note-management endpoints
  main.py          FastAPI application entry point
```

SQLite keeps local setup lightweight, while the architecture can be migrated to PostgreSQL or another production-grade relational database.

## Run locally

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e ".[dev]"

cp .env.example .env
uvicorn app.main:app --reload
```

## Run with Docker

```bash
cp .env.example .env
docker compose up --build
```

## Production gaps

This repository is deliberately a learning and portfolio project. A production service would require additional controls including centralized key management, rate limiting, audit logging, migration management, monitoring, alerting, infrastructure hardening, dependency management, and stronger operational controls.

## Portfolio context

Secure Notes API is a smaller demonstration of principles that I apply more extensively in [BattleReef Marine Controller](https://github.com/DCMedic/BattleReef-Marine-Controller): explicit identity, bounded authorization, separation of trust domains, auditable actions, and systems designed to fail safely rather than silently.

See also [SOC Log Triage](https://github.com/DCMedic/soc-log-triage), [Security Program Starter](https://github.com/DCMedic/security-program-starter), and my [portfolio](https://github.com/DCMedic/about-dcmedic).
