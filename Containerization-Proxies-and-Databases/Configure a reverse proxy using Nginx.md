# **Configure a reverse proxy using Nginx**

### **Overview**

In this lab, you will learn to set up a reverse proxy using Nginx to route incoming traffic to a backend web application.

### **Prerequisites**

- Docker installed

- Completed previous Docker lab (with Flask app running)

---

### **Steps**

**Step 1 — Create a new folder and navigate into it**

```bash
mkdir nginx-proxy && cd nginx-proxy
```

**Step 2 — Create a file named nginx.conf**

```nginx
events {}

http {
  server {
    listen 80;

    location / {
      proxy_pass http://host.docker.internal:5000;
    }
  }
}
```

**Step 3 — Create a Dockerfile for Nginx**

```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
```

**Step 4 — Build and run the container**

```bash
docker build -t nginx-proxy .
docker run -d -p 8080:80 nginx-proxy
```

**Step 5 — Verify**

Visit http://localhost:8080 — it should display the response from your Flask app.

---

### **Tips**

- Ensure your backend app container is running before starting the proxy

- Use docker network to connect multiple containers

- Update the Nginx configuration to handle different routes as needed

---

### **Expected outcome**

You should have an Nginx reverse proxy container successfully routing traffic to your Flask app container.
