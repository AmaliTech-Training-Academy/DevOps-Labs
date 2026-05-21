## **Scripting and cloud fundamentals**

**Learning objective:** Construct well-formed configuration files using YAML and JSON syntax

**Assessment type:** Lab — Create a YAML config for an application

---

### **Lab exercise: Create a YAML configuration for an application**

#### **Overview**

In this lab, you will learn how to create and validate YAML configuration files for a sample web application. YAML is commonly used for defining infrastructure-as-code templates, CI/CD pipelines, and application configurations.

#### **Prerequisites**

- A text editor (VS Code or Sublime Text)

- Basic understanding of indentation and data structures

- Installed YAML/JSON validator (optional: https://yamlvalidator.com or https://jsonlint.com)

---

#### **Steps**

**Step 1 — Open your code editor and create a new file named app-config.yaml**

**Step 2 — Define a simple configuration for a web application:**

```yaml
application:
  name: my-web-app
  version: 1.0.0
  environment: production

server:
  host: 0.0.0.0
  port: 8080

database:
  engine: mysql
  host: db.example.com
  port: 3306
  username: admin
  password: mysecurepassword
```

**Step 3 — Save and validate your file using an online YAML validator**

**Step 4 — Convert your YAML file into JSON using a tool like yq or an online converter**

---

#### **Tips**

- Use consistent spacing (2 or 4 spaces per indentation level)

- Avoid using tabs in YAML files

- Always validate your configuration before deploying

---

#### **Expected outcome**

You should have a valid YAML configuration file that accurately represents the application's structure and parameters.
