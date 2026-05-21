# **Deploy a multi-container app on EC2 (Flask/Node.js + MySQL)**

**Objective:** Use Docker Compose to deploy a two-tier application (Flask/Node.js + MySQL) on EC2.

---

### **Instructions**

- Provision an EC2 instance (Amazon Linux 2) and install Docker and Docker Compose

- Create the project structure:

  - docker-compose.yml

  - web/app.py (or web/app.js for Node.js)

  - web/requirements.txt (or web/package.json for Node.js)

  - db/init.sql

- Configure Docker Compose with:

  - Services: web and db

  - Environment variables for database connection

  - Port mapping

- Verify the app is running with:

```bash
curl http://localhost:5000
```

- Clean up after verification:

```bash
docker compose down --volumes
```

- Use AWS Free Tier or sandbox environment

---

### **Deliverables**

- docker-compose.yml

- Application source code (app.py or app.js)

- Dependencies file (requirements.txt or package.json)

- Database initialization script (init.sql)

- Logs and screenshots as evidence of a successful run

---

### **Submission requirements**

| **Item** | **Format** | **Contents** |
| --- | --- | --- |
| GitHub repository | All project files and evidence | docker-compose.yml, app.py/app.js, requirements.txt, init.sql, screenshots and documentation |
| Tools evidence | Version output or screenshot | Docker v25+, Docker Compose v2+ |
| Run evidence | Screenshots | App accessible on port 5000 |
| Cleanup evidence | Screenshot | Successful docker compose down --volumes |

---

### **Required file structure**

```
project/
├── docker-compose.yml
├── web/
│   ├── app.py          # or app.js for Node.js
│   ├── requirements.txt  # or package.json for Node.js
│   └── Dockerfile
├── db/
│   └── init.sql
└── screenshots/
    ├── curl_output.png
    ├── compose_up.png
    └── compose_down.png
```

---

### **Tips**

- Ensure MySQL is fully initialized before the web service starts — use depends_on with a health check in your docker-compose.yml

- Use environment variables for all database credentials — never hardcode them in your application source

- Add a .dockerignore file to exclude unnecessary files from your image build

- Test your database connection from inside the web container before verifying via curl

- Tag your EC2 instance for easy identification and cleanup

- Always run docker compose logs to diagnose issues if the app is not responding
