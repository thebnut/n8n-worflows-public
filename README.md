# n8n-worflows-public
Shared N8N workflows

## Creating your own N8N instance on Render
# Deploying a Forked n8n Instance to Render

Below is a complete, production‑ready playbook for **forking the n8n GitHub repository and running your own instance on Render using Infrastructure‑as‑Code (`render.yaml`)**.

---

## 1. Prerequisites

| What                       | Why                                                                  |
| -------------------------- | -------------------------------------------------------------------- |
| **GitHub account**         | You need it to fork n8n and to let Render pull your code.            |
| **Render account** (free)  | Render will build and host the web service and optional Postgres DB. |
| **Docker image knowledge** | Optional — n8n runs directly from the official Docker image.         |

---

## 2. Fork the n8n Repository

1. Open the official repo [https://github.com/n8n-io/n8n](https://github.com/n8n-io/n8n) and click **Fork → Create fork**.
2. Clone it locally (optional) so you can add config files:

```bash
git clone git@github.com:<your-org>/n8n.git
cd n8n
```

---

## 3. Add a `render.yaml` Blueprint

Create a `render.yaml` file in the repo root with the following content:

```yaml
services:
  - type: web
    name: n8n-service
    runtime: image
    plan: free
    image:
      url: docker.io/n8nio/n8n:latest
    envVars:
      - key: DB_TYPE
        value: postgresdb
      - key: DB_POSTGRESDB_HOST
        fromDatabase:
          name: n8n-db
          property: host
      - key: DB_POSTGRESDB_PORT
        fromDatabase:
          name: n8n-db
          property: port
      - key: DB_POSTGRESDB_DATABASE
        fromDatabase:
          name: n8n-db
          property: database
      - key: DB_POSTGRESDB_USER
        fromDatabase:
          name: n8n-db
          property: user
      - key: DB_POSTGRESDB_PASSWORD
        fromDatabase:
          name: n8n-db
          property: password
      - key: N8N_HOST
        value: $(RENDER_EXTERNAL_HOSTNAME)
      - key: N8N_PORT
        value: "5678"
      - key: N8N_BASIC_AUTH_ACTIVE
        value: "true"
      - key: N8N_BASIC_AUTH_USER
        value: ""
      - key: N8N_BASIC_AUTH_PASSWORD
        value: ""
      - key: WEBHOOK_URL
        value: https://$(RENDER_EXTERNAL_HOSTNAME)/
databases:
  - name: n8n-db
    databaseName: n8n
    plan: free
```

---

## 4. Commit & Push

```bash
git add render.yaml
git commit -m "Add Render blueprint"
git push origin main
```

---

## 5. Connect GitHub to Render & Deploy

1. In the **Render Dashboard**, click **New → Blueprint**.
2. Connect GitHub if prompted.
3. Select your forked repo and branch.
4. Click **Deploy Blueprint**.

---

## 6. Add Sensitive Environment Variables

In Render's dashboard, go to **n8n-service → Environment** and add the following:

| Key                       | Value                  |
| ------------------------- | ---------------------- |
| `N8N_BASIC_AUTH_USER`     | `admin`                |
| `N8N_BASIC_AUTH_PASSWORD` | `your-strong-password` |

---

## 7. Verify & Log In

Visit the onrender.com URL, enter your credentials, and you’ll see the n8n canvas. Workflows are stored in Postgres.

---

## 8. Enable Auto Deploy

Auto-deploy is enabled by default. Pushes to your repo will trigger automatic redeploys.

---

## 9. Optional Enhancements

| Feature                | How                                                   |
| ---------------------- | ----------------------------------------------------- |
| **Pin image version**  | Replace `latest` with a tag like `1.34.0`             |
| **Custom domain**      | Add under **Settings → Custom Domains**               |
| **Use SQLite + disk**  | Use a paid Render plan and attach a persistent disk   |
| **CI previews**        | Enable Render **Service Previews**                    |
| **Extra/custom nodes** | Place in `/home/node/.n8n/custom` in your forked repo |

---

## 10. Troubleshooting Tips

| Symptom                         | Fix                                                                 |
| ------------------------------- | ------------------------------------------------------------------- |
| Invalid DB credentials          | Ensure the Postgres database is fully provisioned before first boot |
| Workflow data lost after deploy | Use Postgres or attach persistent disk for SQLite setup             |
| Push doesn’t redeploy           | Verify auto-deploy is enabled or use manual deployment              |

---

You're now running your own forked and Git-tracked instance of n8n on Render. 🎉
