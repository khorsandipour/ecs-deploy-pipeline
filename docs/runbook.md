# Ops Runbook — ECS Deploy Pipeline

Quick reference for common scenarios.

---

## Normal Deploy

Push to main branch — pipeline runs automatically.
Monitor in GitHub Actions tab.

Expected time: 8–12 minutes end to end.

---

## Manual Deploy

Go to Actions → Deploy to ECS — Production →
Run workflow → Select main → Run.

Use when you need to redeploy without a code change.

---

## Emergency Rollback

Go to Actions → Emergency Rollback → Run workflow.

Select:
- Environment: production
- Service: all (or specific service)
- Reason: describe what happened

Expected time: 3–5 minutes.

---

## Pipeline Stuck

If deployment is stuck for more than 15 minutes:

```bash
# Check ECS service events
aws ecs describe-services \
  --cluster [CLUSTER-NAME] \
  --services laravel-backend \
  --query 'services[0].events[:5]'

# Check running tasks
aws ecs list-tasks \
  --cluster [CLUSTER-NAME] \
  --service-name laravel-backend

# Force new deployment if needed
aws ecs update-service \
  --cluster [CLUSTER-NAME] \
  --service laravel-backend \
  --force-new-deployment
```

---

## Health Check Fails After Deploy

1. Check ECS task logs in CloudWatch:
   `/ecs/[cluster-name]/laravel-backend`

2. Check ALB target group health:
   AWS Console → EC2 → Target Groups →
   Select group → Targets tab

3. If unhealthy — pipeline circuit breaker
   should have triggered auto rollback.
   Verify in Actions tab.

4. If auto rollback failed — run manual
   rollback workflow.

---

## Festival / High-Traffic Events

Before the event:
- Test rollback pipeline 48 hours before
- Verify Auto Scaling min/max counts
- Open CloudWatch dashboards
- Check RDS Read Replica lag

During the event:
- Monitor ALB RequestCount and TargetResponseTime
- Watch ECS CPU and memory per service
- Keep rollback.yml ready to run
- Direct line with backend team open

After the event:
- Check cost anomaly alerts
- Review WAF blocked requests
- Scale back down if needed
- Write incident summary

---

## Useful AWS CLI Commands

```bash
# List running tasks
aws ecs list-tasks \
  --cluster [CLUSTER] \
  --service-name laravel-backend

# Get service status
aws ecs describe-services \
  --cluster [CLUSTER] \
  --services laravel-backend nextjs-frontend \
  --query 'services[*].{Name:serviceName,Status:status,Running:runningCount,Desired:desiredCount}'

# Get last 5 service events
aws ecs describe-services \
  --cluster [CLUSTER] \
  --services laravel-backend \
  --query 'services[0].events[:5]'

# Check ALB target health
aws elbv2 describe-target-health \
  --target-group-arn [TARGET-GROUP-ARN]
```
