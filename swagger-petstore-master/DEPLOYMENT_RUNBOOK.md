# DEPLOYMENT_RUNBOOK.md

## 1) Dependencies & Connectivity
- **Required external services**: None evidenced. The app uses in-memory lists for pets/orders/users. Evidence: `swagger-petstore-master/src/main/java/io/swagger/petstore/data/*.java`
- **Optional integrations**:
  - Bugsnag error reporting via `BUGSNAG_API_KEY`. Evidence: `swagger-petstore-master/src/main/java/io/swagger/petstore/notification/BugSnagNotifier.java`
  - OpenTelemetry export via `Dockerfile-telemetry` (OTLP to Bugsnag). Evidence: `swagger-petstore-master/Dockerfile-telemetry`
- **Connectivity checks**:
  - Service availability: `GET /api/v3/openapi.json` (documented in README). Evidence: `swagger-petstore-master/README.md`

## 2) Build & Package Commands
- **Maven build**: `mvn package` (produces WAR). Evidence: `swagger-petstore-master/pom.xml`
- **Run with Jetty (dev)**: `mvn package jetty:run`. Evidence: `swagger-petstore-master/README.md`
- **Docker image build**: `docker build -t swaggerapi/petstore3:unstable .` (example). Evidence: `swagger-petstore-master/README.md`

## 3) Configuration & Secrets
**Config sources**:
- `inflector.yaml` for controller/model package mapping and `rootPath`. Evidence: `swagger-petstore-master/inflector.yaml`
- Environment variables:
  - `notifierClass` (optional). Evidence: `swagger-petstore-master/src/main/java/io/swagger/petstore/controller/PetController.java`
  - `BUGSNAG_API_KEY` (optional). Evidence: `swagger-petstore-master/src/main/java/io/swagger/petstore/notification/BugSnagNotifier.java`
  - `SWAGGER_OAUTH_HOST` (optional). Evidence: `swagger-petstore-master/src/main/java/io/swagger/petstore/utils/HandleAuthUrlProcessor.java`
- System property `swaggerUrl` passed in Docker CMD. Evidence: `swagger-petstore-master/Dockerfile`

## 4) Container Run Locally (Minimal)
- **Docker**:
  - Build: `docker build -t swaggerapi/petstore3:unstable .`
  - Run: `docker run --name swaggerapi-petstore3 -d -p 8080:8080 swaggerapi/petstore3:unstable`
  Evidence: `swagger-petstore-master/README.md`

## 5) Kubernetes Deployment Steps
### If manifests/Helm exist
- Unknown (Not Found in Repo). Searched: repo root for `helm/`, `kustomize/`, `*.yaml`.

### Blueprint (placeholders)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: swagger-petstore
spec:
  replicas: 1
  selector:
    matchLabels:
      app: swagger-petstore
  template:
    metadata:
      labels:
        app: swagger-petstore
    spec:
      containers:
        - name: swagger-petstore
          image: <REPLACE_WITH_BUILT_IMAGE>
          ports:
            - containerPort: 8080
          env:
            - name: BUGSNAG_API_KEY
              value: <OPTIONAL>
            - name: SWAGGER_OAUTH_HOST
              value: <OPTIONAL>
            - name: notifierClass
              value: <OPTIONAL>
          volumeMounts:
            - name: request-logs
              mountPath: /var/log
      volumes:
        - name: request-logs
          emptyDir: {}
```
Evidence: `swagger-petstore-master/Dockerfile` (port/log path), `swagger-petstore-master/src/main/java/io/swagger/petstore/notification/BugSnagNotifier.java`, `.../utils/HandleAuthUrlProcessor.java`, `.../controller/PetController.java`

## 6) Post‑deploy Validation
- **Smoke test**: `GET /api/v3/openapi.json` (documented as readiness check). Evidence: `swagger-petstore-master/README.md`
- **Basic API checks**:
  - `GET /api/pet/findByStatus?status=available` (OpenAPI). Evidence: `swagger-petstore-master/src/main/resources/openapi.yaml`
  - `GET /api/store/inventory` (OpenAPI). Evidence: `swagger-petstore-master/src/main/resources/openapi.yaml`

## 7) Troubleshooting Playbook (Top 10)
| Symptom | Likely Cause | Checks | Fix | Evidence |
|---|---|---|---|---|
| 400 “No petId provided” | Missing `petId` path param | Inspect request path | Provide `/pet/{petId}` | `swagger-petstore-master/src/main/java/io/swagger/petstore/controller/PetController.java` |
| 404 “Pet not found” | In-memory record missing | Query existing IDs | Add pet before get | `.../PetController.java`, `.../data/PetData.java` |
| 400 “No Order provided” | Missing order body | Validate request body | Send Order JSON | `.../controller/OrderController.java` |
| 404 “Order not found” | Order ID not in list | Check order IDs | Place order first | `.../OrderController.java`, `.../data/OrderData.java` |
| 404 “User not found” | User not in list | Verify username | Create user first | `.../UserController.java`, `.../data/UserData.java` |
| No data persists after restart | In-memory storage | Restart loses lists | Accept demo-only or add DB | `.../data/*.java` |
| `/api` paths 404 | Wrong context mapping | Ensure `/api/*` ingress | Route to `/api/*` | `swagger-petstore-master/src/main/webapp/WEB-INF/web.xml`, `swagger-petstore-master/inflector.yaml` |
| Missing Bugsnag events | `BUGSNAG_API_KEY` not set | Check env var | Set `BUGSNAG_API_KEY` | `.../notification/BugSnagNotifier.java` |
| OpenAPI auth URL incorrect | `SWAGGER_OAUTH_HOST` not set | Check env var | Set `SWAGGER_OAUTH_HOST` | `.../utils/HandleAuthUrlProcessor.java` |
| Telemetry not exported | Using non‑telemetry image | Verify image | Use `Dockerfile-telemetry` | `swagger-petstore-master/Dockerfile-telemetry` |

## 8) Rollback Strategy
- If using Kubernetes Deployment, roll back to previous ReplicaSet: `kubectl rollout undo deployment/swagger-petstore`. **Generic guidance** (no manifests in repo). Evidence: no k8s assets found.

## 9) Operational Checklists
### Go-live Checklist
- [ ] Image built from `Dockerfile` and pushed to registry. Evidence: `swagger-petstore-master/Dockerfile`
- [ ] Ingress routes `/` (UI) and `/api/*` (API) correctly. Evidence: `swagger-petstore-master/src/main/webapp/WEB-INF/web.xml`, `swagger-petstore-master/src/main/webapp/index.html`
- [ ] Optional env vars set (`BUGSNAG_API_KEY`, `SWAGGER_OAUTH_HOST`, `notifierClass`). Evidence: `.../notification/BugSnagNotifier.java`, `.../utils/HandleAuthUrlProcessor.java`, `.../controller/PetController.java`
- [ ] Smoke test passes (`/api/v3/openapi.json`). Evidence: `swagger-petstore-master/README.md`

### Day-2 Checklist
- [ ] Validate request log file path `/var/log/yyyy_mm_dd-requests.log` exists and is writable. Evidence: `swagger-petstore-master/Dockerfile`
- [ ] Confirm data reset behavior on restart is acceptable. Evidence: `.../data/*.java`
- [ ] Review OpenAPI security schemes vs. actual enforcement. Evidence: `openapi.yaml`, `controller/*.java`

## 10) Monitoring Suggestions (Evidence‑backed)
- **HTTP request rate / latency** (Jetty) — implied by HTTP service nature. Evidence: `swagger-petstore-master/src/main/webapp/WEB-INF/web.xml`
- **Error notifications** via Bugsnag if enabled. Evidence: `swagger-petstore-master/src/main/java/io/swagger/petstore/notification/BugSnagNotifier.java`
- **OTel traces** if using telemetry image. Evidence: `swagger-petstore-master/Dockerfile-telemetry`
