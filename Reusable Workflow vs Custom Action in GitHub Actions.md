# Reusable Workflow vs Custom Action in GitHub Actions

Both **Reusable Workflows** and **Custom Actions** allow for **code reuse**, but they serve different purposes in GitHub Actions.

---

## 📌 What is a Reusable Workflow?
A **Reusable Workflow** is a full **workflow file** that other workflows can call. It allows **teams** to standardize **CI/CD processes** across multiple repositories and reduce duplication.

### ✅ Best Used For:
- Standardizing **deployment pipelines**.
- Running **tests, linting, or security scans** across multiple repositories.
- Implementing **common DevOps tasks** like environment setup.

📍 **Stored in:** `.github/workflows/`

---

## 📌 What is a Custom Action?
A **Custom Action** is a small, reusable component that performs **a specific task**. It is used inside a workflow **as a step** and can be written in **JavaScript, Python, Bash, or Docker**.

### ✅ Best Used For:
- **Encapsulating** a **small task** (e.g., setting up a tool, authentication, or formatting code).
- **Reusing logic** in multiple workflows.
- Running **scripts** in a clean and maintainable way.

📍 **Stored in:** `.github/actions/`

---

## 🔹 Key Differences

| Feature             | Reusable Workflow | Custom Action |
|---------------------|------------------|--------------|
| **Scope**          | Full workflow (jobs & steps) | Single step within a workflow |
| **Best for**       | CI/CD pipelines | Reusable small tasks |
| **Inputs**         | ✅ Yes (`inputs:`) | ✅ Yes (`inputs:`) |
| **Secrets**        | ✅ Yes (`secrets:`) | ✅ Yes (`secrets:` but inside `env:` only) |
| **Outputs**        | ✅ Yes (`outputs:`) | ✅ Yes (`outputs:`) |
| **Location**       | `.github/workflows/` | `.github/actions/` |

---

## 🔹 1. Reusable Workflow (With Inputs & Secrets)
A **reusable workflow** is a **full workflow** that can be shared across multiple repositories.

### 📌 Example: A Reusable Deployment Workflow
📍 **Stored in**: `.github/workflows/deploy.yml`
```yaml
name: Deploy Workflow
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      app_name:
        required: true
        type: string
    secrets:
      DEPLOY_KEY:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Print Inputs
        run: |
          echo "Deploying app: ${{ inputs.app_name }} to environment: ${{ inputs.environment }}"

      - name: Authenticate with Secret
        run: |
          echo "Using deploy key: ${{ secrets.DEPLOY_KEY }}" # Do not print secrets in production!
```

### Calling the Reusable Workflow
📍 **Stored in**: `.github/workflows/main.yml`
```yaml
name: Main Workflow
on: push

jobs:
  call-deploy:
    uses: my-org/my-repo/.github/workflows/deploy.yml@main
    with:
      environment: production
      app_name: my-app
    secrets:
      DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

---

## 🔹 2. Custom Action (With Inputs & Secrets)
A **Custom Action** is a reusable unit of work **inside a workflow**, responsible for a **specific task**.

### 📌 Example: Setup Node.js Action
📍 **Stored in**: `.github/actions/setup-node/action.yml`
```yaml
name: Setup Node.js
description: Setup Node.js with a specific version
inputs:
  node-version:
    required: true
    description: "Node.js version"
runs:
  using: "composite"
  steps:
    - name: Install Node.js
      run: |
        echo "Setting up Node.js version ${{ inputs.node-version }}"
      shell: bash

    - name: Authenticate with Secret
      run: |
        echo "Using authentication token (hidden)"
      shell: bash
      env:
        NODE_AUTH_TOKEN: ${{ env.NODE_AUTH_TOKEN }}  # ✅ Use secret from env
```

### Using This Action in a Workflow
📍 **Stored in**: `.github/workflows/main.yml`
```yaml
name: CI Workflow
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Use Custom Setup Action
        uses: ./.github/actions/setup-node
        with:
          node-version: "16"
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_AUTH_TOKEN }}  # ✅ Secure and correct
```

---

## 🔹 Summary
| Feature             | Reusable Workflow | Custom Action |
|---------------------|------------------|--------------|
| **Scope**          | Full workflow | Single step |
| **Best for**       | CI/CD pipelines | Reusable small tasks |
| **Inputs**         | ✅ Yes | ✅ Yes |
| **Secrets**        | ✅ Yes | ✅ Yes (via `env:`) |
| **Outputs**        | ✅ Yes | ✅ Yes |
| **Location**       | `.github/workflows/` | `.github/actions/` |

---
