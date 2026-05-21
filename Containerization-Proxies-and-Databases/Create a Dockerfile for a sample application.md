# **Create a Dockerfile for a sample application**

### **Overview**

In this lab, you will learn how to create a Dockerfile that packages a simple web application into a container image and run it locally.

### **Prerequisites**

- Docker Desktop or Docker Engine installed

- Basic knowledge of Linux commands

- Sample web app (a simple Node.js or Python Flask app)

---

### **Steps**

**Step 1 — Create a new folder and navigate into it**

```bash
mkdir docker-lab && cd docker-lab
```

**Step 2 — Create a simple Python Flask app**

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return 'Hello from Docker!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Step 3 — Create a Dockerfile**

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY . .

RUN pip install flask

EXPOSE 5000

CMD ["python", "app.py"]
```

**Step 4 — Build the Docker image**

```bash
docker build -t flask-app .
```

**Step 5 — Run the container**

```bash
docker run -d -p 5000:5000 flask-app
```

---

### **Tips**

- Use lightweight base images like python:3.9-slim

- Always define a working directory using WORKDIR

- Use .dockerignore to exclude unnecessary files from your image

---

### **Expected outcome**

You should have a running Docker container serving a Flask app accessible at http://localhost:5000
