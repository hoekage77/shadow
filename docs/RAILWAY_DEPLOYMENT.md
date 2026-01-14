# Deploying Suna Backend to Railway

> **Note on Railway Plans:** As of late 2023, Railway no longer offers a permanent free tier for hosting web services (compute). The "Trial" plan is limited and often restricted to databases only after the initial credits expire. To host the Suna backend, you will likely need a **Hobby** plan ($5/mo) or use an alternative like **Render** or **Fly.io** for the compute portion.

This guide walks you through deploying the Suna backend to Railway using the provided `railway.json`.

## Prerequisites

1.  A Railway account.
2.  Railway CLI installed (optional but recommended).

## Deployment Steps

### 1. Create a New Project on Railway

*   Go to [Railway](https://railway.app/) and create a new project.
*   Select "Deploy from GitHub repo" and choose your repository.

### 2. Configure the Backend Service

Railway should automatically detect the `railway.json` in the root directory. If you need to manually configure the service:

*   **Builder:** Dockerfile
*   **Dockerfile Path:** `backend/Dockerfile`
*   **Source Directory (Context):** `backend`

### 3. Add a Redis Service

The Suna backend requires Redis for caching and background tasks.

*   In your Railway project, click **+ New** -> **Database** -> **Redis**.
*   Railway will create a new service called `Redis` (or similar).

### 4. Link Redis Variables to Backend

To link the Redis database to your backend service so they communicate:

1.  Open your **Backend Service** settings.
2.  Go to the **Variables** tab.
3.  Add the following variables using Railway's reference syntax. This ensures that if the Redis password or host changes, the backend updates automatically.

| Variable | Railway Reference Value |
| :--- | :--- |
| `REDIS_HOST` | `${{Redis.REDIS_HOST}}` |
| `REDIS_PORT` | `${{Redis.REDIS_PORT}}` |
| `REDIS_PASSWORD` | `${{Redis.REDIS_PASSWORD}}` |
| `REDIS_SSL` | `true` |

*Note: If you named your Redis service something other than "Redis", replace `Redis` in the references above (e.g., `${{my-custom-redis.REDIS_HOST}}`).*

### 5. Set Remaining Environment Variables

Add these additional environment variables to your backend service:

| Variable | Recommended Value / Source |
| :--- | :--- |
| `ENV_MODE` | `production` |
| `NEXT_PUBLIC_URL` | `https://your-backend-url.up.railway.app` |
| `TEMPORAL_CLOUD_ADDRESS` | From your Temporal Cloud dashboard |
| `TEMPORAL_NAMESPACE` | From your Temporal Cloud dashboard |
| `TEMPORAL_API_KEY` | From your Temporal Cloud dashboard |
| `SUPABASE_URL` | From your Supabase project |
| `SUPABASE_ANON_KEY` | From your Supabase project |
| `KORTIX_ADMIN_API_KEY` | Your secret admin key |

### 6. (Optional) Run the Temporal Worker

If you need to run the Temporal worker as a separate service on Railway:

1.  Create another service in the same project pointing to the same repo.
2.  Override the **Start Command** to: `uv run python -m core.temporal.worker`
3.  Ensure it has the same environment variables.

## Health Check

The `railway.json` is configured to use `/v1/health` for health checks. Ensure your backend is configured to respond to this path.
