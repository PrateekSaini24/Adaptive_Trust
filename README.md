# AdaptiveTrust

A runnable reference implementation of an **adaptive, risk-based access control** system — an 8-factor risk engine, policy engine, identity graph, and a full SOC (Security Operations Center) analyst dashboard, all wired together behind a live, real-time UI.

It ships with real backends for every piece of infrastructure it uses (Redis, Neo4j, Kafka, PostgreSQL, OPA), but defaults to **in-memory fallbacks** so it runs with zero setup. Flip one environment variable per service to point it at the real thing.

## Features

- **8-factor risk engine** — token age/MFA, device fingerprinting, geo-velocity (haversine distance), session history, credential breach checks (HIBP), and more
- **Policy engine** — Open Policy Agent (Rego) or a pure-Python fallback
- **Identity graph** — Neo4j-backed (or in-memory), explainable risk decisions
- **Real-time SOC dashboard** — live feed via Server-Sent Events, geo activity map, risk score trends, side-by-side request comparison, CSV/PDF compliance export
- **Full SOC analyst platform** (`/soc`) — alert triage, incident management, threat intel, IP investigation, MITRE ATT&CK browser, endpoint/process/file-integrity monitoring, role-based access via signed JWTs
- **Resilience** — circuit breakers, timeouts/retries, chaos testing (including real Docker-container-kill scripts)
- **Observability** — Prometheus metrics + Grafana dashboards out of the box
- **Supply chain security** — SBOM generation, artifact signing
- **Async pipeline** — Kafka-backed (or in-process) audit, alert, and analytics consumers

## Quick start (zero setup)

```bash
git clone <this-repo-url>
cd adaptivetrust
pip install -r requirements.txt
python app.py
```

Open **http://localhost:5000/** for the dashboard.

## Running with real infrastructure

Everything defaults to an in-memory fallback. To switch any piece over to its real backend:

```bash
cp .env.example .env
# uncomment the real-backend client libraries in requirements.txt
pip install -r requirements.txt

docker compose up -d redis neo4j kafka zookeeper postgres opa prometheus grafana

export $(cat .env | xargs)
export USE_REAL_REDIS=true
export USE_REAL_NEO4J=true
export USE_REAL_KAFKA=true
export USE_REAL_POSTGRES=true
export USE_REAL_OPA=true

python app.py
```

Each `USE_REAL_*` flag is independent. Or run the one-command version:

```bash
chmod +x go_real.sh
./go_real.sh
```

### Kubernetes

```bash
minikube start
eval $(minikube docker-env)
docker build -t adaptivetrust:local .
kubectl apply -f k8s-service.yaml
kubectl apply -f k8s-deployment.yaml
```

## Running tests

```bash
pip install -r requirements.txt
pytest
```

For load testing:

```bash
pip install locust
python app.py            # terminal 1
locust -f locustfile.py  # terminal 2
```

## Project layout

| Path | Purpose |
|---|---|
| `app.py` | Main Flask app, `/access/decide` request flow |
| `risk_engine.py` | 8-factor risk scoring |
| `policy_engine.py`, `policy.rego` | Policy evaluation (OPA or Python) |
| `graph_engine.py` | Identity graph (Neo4j or in-memory) |
| `cache.py` | Redis or in-memory caching |
| `event_bus.py`, `consumers/` | Kafka (or in-process) async pipeline |
| `circuit_breaker.py`, `resilience.py` | Fault tolerance |
| `soc/` | Full SOC analyst dashboard backend |
| `dashboard/` | Frontend for the main AdaptiveTrust dashboard |
| `supply_chain/` | SBOM + artifact signing |
| `tests/` | Unit + integration tests |
| `k8s-*.yaml`, `docker-compose.yml`, `Dockerfile` | Deployment |
| `grafana/`, `prometheus.yml` | Monitoring |

## SOC dashboard login

Visit `/soc` and log in as one of:

- **viewer** — read-only access
- **analyst** — can triage/resolve alerts, manage incidents, add threat intel
- **admin** — analyst + audit log access + demo-generator control

Roles are enforced server-side via signed JWTs (`soc/auth.py`).

## Notes

- The SOC demo-event generator (`soc/demo_generator.py`) produces synthetic activity so the platform is usable with no real security infrastructure connected; every event it creates is tagged `detection_engine="demo-generator"`.
- MITRE ATT&CK data (`soc/mitre_data.py`) is real, from the public matrix.
- IP investigation uses a real lookup (ipapi.co).

## License

Add a license of your choice (e.g. MIT) before making this repository public.
