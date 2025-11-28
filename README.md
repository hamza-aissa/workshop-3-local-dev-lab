# Local DevOps Lab – Runbook

## 1. Start the stack

cd ~/projects/local-devops-lab
docker-compose up -d gitea woodpecker-server woodpecker-agent prometheus grafana

Check containers:

docker ps

## 2. Web UIs and ports

- Gitea: `http://localhost:3000`
- Woodpecker: `http://localhost:8000`
- Sample app: `http://localhost:8080`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3001`

## 3. Gitea setup

1. Open `http://localhost:3000`.
2. Finish install wizard.
3. Create an admin user (e.g. `admin`).
4. Create a repo (e.g. `local-devops-app`) and push your app + `.woodpecker.yml` + `app/docker-compose.yml`.

## 4. Gitea OAuth app for Woodpecker

In Gitea (logged in as admin):

1. Go to `Settings → Applications → OAuth2 Applications → New`.
2. Name: `woodpecker`.
3. Redirect URI: `http://localhost:8000/authorize`.
4. Save and copy:
   - Client ID
   - Client Secret
These are used in `docker-compose.yml` as:

    WOODPECKER_GITEA_CLIENT=<Client ID>

    WOODPECKER_GITEA_SECRET=<Client Secret>
Restart after changes:

docker-compose down -v
docker-compose up -d gitea woodpecker-server woodpecker-agent prometheus grafana

## 5. Woodpecker login & the gitea→localhost fix

1. Open `http://localhost:8000`.
2. Click “Login with Gitea”.
3. Browser will be redirected to a URL starting with:

   `http://gitea:3000/login/oauth/authorize?...`

4. In the browser address bar, **replace** `gitea` with `localhost`:

   `http://localhost:3000/login/oauth/authorize?...`

   (Leave the rest of the query string unchanged.)

5. Press Enter, log in to Gitea as `admin`, click “Authorize”.
6. You should land back on `http://localhost:8000` logged into Woodpecker.

## 6. CI/CD pipeline behavior

- On `git push` to the Gitea repo:
  - Woodpecker builds `localhost:5000/sample-service:${CI_COMMIT_SHA}`.
  - Pushes image to local registry `localhost:5000`.
  - Runs `docker compose -f app/docker-compose.yml up -d` to deploy.
- Check app:
curl http://localhost:8080

## 7. Prometheus & Grafana access

- Prometheus UI: `http://localhost:9090`
- You can run queries like `up` to see scraped targets.
- Grafana UI: `http://localhost:3001`
- First login: `admin` / `admin` (then set a new password).
- Add data source:
  - Type: Prometheus
  - URL: `http://prometheus:9090`
- Create a simple dashboard using the `up` metric or container metrics.

## 8. Stopping / restarting everything

- Stop and remove everything:

