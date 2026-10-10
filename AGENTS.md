# AGENTS.md

## Project Overview

This is **yudao** (芋道), a Spring Boot 2.7.18 / JDK 8 admin framework with a Maven multi-module structure. The active modules (defined in the root `pom.xml`) are: `yudao-dependencies`, `yudao-framework`, `yudao-server`, `yudao-module-system`, `yudao-module-infra`. All other modules (mall, CRM, ERP, BPM, AI, etc.) are commented out.

## Key Facts

- **Backend**: Spring Boot 2.7.18, JDK 8, Maven. Runs on port **48080**.
- **Database**: MySQL 8, database name `ruoyi-vue-pro`. SQL init scripts in `sql/mysql/` (loaded by MySQL container's docker-entrypoint-initdb.d). The local profile config references `ruoyi-vue-pro-jdk8` but we override the datasource URL to use `ruoyi-vue-pro` (matching the SQL file).
- **Cache**: Redis 6 (no password).
- **Frontend**: `yudao-ui/yudao-ui-admin-vue3/` contains only 6 files (partial MES module views/api) — NOT a runnable frontend. The full admin frontend lives in a separate repo.
- **No Spring Boot DevTools**: The project does not include devtools, so there is no live-reload. After source changes, rebuild the server module and restart the container, then call `reload_preview`.
- **No external secrets required**: All AI/API keys have placeholder defaults (`sk-xxxx`). The AI module is not compiled in. MySQL and Redis use local credentials wired in compose.

## Sandbox Setup (docker-compose.base44.yml)

- **mysql**: MySQL 8 with `ruoyi-vue-pro` database, initialized from `sql/mysql/ruoyi-vue-pro.sql` and `sql/mysql/quartz.sql`.
- **redis**: Redis 6 Alpine.
- **server**: Built from `yudao-server/Dockerfile.dev` (Maven 3.9 + JDK 8). Source is bind-mounted at `/app`. On startup it rebuilds `yudao-server` with `mvn package -DskipTests -pl yudao-server -am` then runs the jar with datasource/Redis overrides pointing to the compose services.
- **proxy**: Nginx on port **3000** proxying to the backend on 48080, so the preview iframe can show the Swagger/Knife4j API UI.

## How to Verify

- After `docker compose -f docker-compose.base44.yml up -d --build`, wait for the server healthcheck to pass (start_period 420s — the Maven rebuild + Spring Boot startup takes several minutes).
- The Swagger UI is at `/swagger-ui`, Knife4j at `/doc.html`.
- Health endpoint: `/actuator/health`.
- The local profile disables captcha and enables mock security (`yudao.security.mock-enable: true`).

## Build Notes

- The Docker build runs `mvn install -DskipTests` to cache the `.m2` repository and compiled artifacts in the image. At runtime, the bind-mounted source triggers a rebuild of the server module (using cached dependencies).
- First startup takes 3–5 minutes (Maven rebuild + Spring Boot startup). Subsequent restarts are faster.
