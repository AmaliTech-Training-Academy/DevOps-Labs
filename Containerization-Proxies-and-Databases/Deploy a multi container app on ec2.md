# Project: Deploy a multi-container app on EC2 (Flask/Node.js + MySQL)

**Objective:** Use Docker Compose to deploy a two-tier app (Flask/Node.js + MySQL) on EC2.

---

## Instructions

- Provision EC2 (Amazon Linux 2) and install Docker and Docker Compose
- Create the following structure:
  - `docker-compose.yml`
  - `web/app.py`
  - `web/requirements.txt`
  - `db/init.sql`
  - or similar for a Node.js application
- Configure Compose with: services `web` and `db`, environment variables for DB, and ports mapping (or similar for a Node.js application)
- Verify with `curl http://localhost:5000` then cleanup with `docker compose down --volumes`
- Use AWS Free Tier or sandbox

---

## Deliverables

- Compose file
- App source
- DB script
- Logs and screenshots

---

## Submission requirements

| Item | Format | Contents |
|---|---|---|
| GitHub repo | `docker-compose.yml`, `app.py`/`app.js`, `requirements.txt`, `init.sql`, evidence of run (screenshots and documentation) | All project files |
| Tools evidence | Screenshot or version output | Docker v25+, Docker Compose v2+ |
| Run evidence | Screenshots | Accessible on port 5000 and cleaned up after run |
