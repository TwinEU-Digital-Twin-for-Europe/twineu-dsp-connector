## Migration Guide: From MinIO Open Source to AIStor

This guide explains how to migrate from the previous MinIO Open Source deployment to the new **MinIO AIStor (Free Tier)** provided in this repository.

---

### 🔹 Prerequisites

Before starting, ensure you have:

- Docker and Docker Compose installed
- Access to your existing MinIO data (if applicable)
- A valid **AIStor Free Tier license file (`minio.license`)**

---

### 🔹 Step 1 — Obtain Your AIStor Free License

AIStor requires a **license file even for the Free Tier**.

1. Go to the official [MinIO website](https://www.min.io/pricing)
2. Request a **Free Tier license** with **Get Started** and register an account using the form
3. Download the file `minio.license`  
4. Place it in the following path:

```bash
docker/minio/minio.license
```

> ⚠️ Each partner must obtain and use their **own license file**

---

### 🔹 Step 2 — Use the Provided Docker Compose

The repository already includes the updated `docker-compose.yml` with the AIStor service configuration.

No manual changes are required. Ensure that:

- The `minio.license` file is correctly placed in `docker/minio/`
- The environment variables are defined in your `.env` file:

```env
MINIO_USER=...
MINIO_PASSWORD=...
```

---

### 🔹 Step 3 — Preserve Existing Data (Optional)

If you were using a previous MinIO deployment:

- Ensure your existing Docker volume (`minio_data`) is preserved  
- AIStor is **S3-compatible**, so existing data should remain accessible  

> ⚠️ Always back up your data before upgrading

---

### 🔹 Step 4 — Start the Deployment

Run:

```bash
docker compose up -d
```

Then check logs:

```bash
docker logs -f minio
```

---

### 🔹 Troubleshooting

- If the container does not start:
  - Check that `minio.license` exists and is correctly mounted
  - Verify file permissions
  - Inspect logs:

    ```bash
    docker logs minio
    ```

- If you see license-related errors:
  - Ensure the license file is valid and not corrupted
  - Verify the path `docker/minio/minio.license` is correct

---