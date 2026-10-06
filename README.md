# ECS Deploy Pipeline

Zero-downtime ECS deployment via GitHub Actions.

Build → Push to ECR → Rolling ECS update →
Health check → Auto rollback on failure.

Production-tested pattern — runs on every push
to main for a fintech SaaS platform in Oman
handling 30,000+ transactions per hour during
peak festival events.

---

## How It Works

```
Developer pushes to main branch
         │
         ▼
GitHub Actions triggered
         │
         ▼
Build Docker image
         │
         ▼
Tag with git SHA → Push to ECR
         │
         ▼
Update ECS task definition
         │
         ▼
Rolling deployment (50% min healthy)
         │
         ▼
ECS health checks pass?
    │              │
   YES             NO
    │              │
    ▼              ▼
Deploy complete  Circuit breaker
                 triggers rollback
```

---

## Why This Pattern

Before this pipeline, deployments were done
manually — SSH into the server, pull the image,
restart the service. That process was error-prone
during high-traffic periods and had no automatic
rollback if something went wrong.

This pipeline replaced it with:
- Consistent builds (same environment every time)
- Immutable images tagged by git SHA
- Rolling deployment (no downtime window)
- Automatic rollback if health checks fail
- Full audit trail in GitHub Actions logs

---

## Files

```
ecs-deploy-pipeline/
├── .github/
│   └── workflows/
│       ├── deploy.yml          # Main deploy pipeline
│       ├── rollback.yml        # Manual emergency rollback
│       └── staging-deploy.yml  # Staging environment
├── scripts/
│   ├── health-check.sh         # Post-deploy verification
│   └── pre-deploy-check.sh     # Pre-flight checks
├── docs/
│   └── runbook.md              # Ops runbook
└── README.md
```

---

## Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | IAM user with ECS + ECR permissions |
| `AWS_SECRET_ACCESS_KEY` | IAM secret key |
| `AWS_REGION` | AWS region (e.g. me-south-1) |
| `ECR_REGISTRY` | ECR registry URI |
| `ECS_CLUSTER` | ECS cluster name |

> Never commit credentials.
> All sensitive values go in GitHub Secrets.

---

## Deployment Behavior

| Scenario | Behavior |
|----------|----------|
| Push to main | Auto deploy to production |
| Push to staging | Auto deploy to staging |
| Health check fails | Circuit breaker triggers rollback |
| Manual rollback needed | Run rollback.yml workflow |
| Deployment stuck | ECS waits 10 min then fails |

---

## Author

**Ahmad Khorsandi Pour**  
Cloud Infrastructure Engineer  
📍 Muscat, Oman  
🔗 [linkedin.com/in/khorsandipour](https://linkedin.com/in/khorsandipour)
