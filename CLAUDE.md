# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a Kubernetes training exercise repo (task 6 of a "DevOps ToDo List" course series: DaemonSets and Jobs). The `src/` directory holds a Django ToDo list app that is used only as the workload being deployed — the actual assignment is the Kubernetes manifests under `.infrastructure/`. Do not treat this as an app-development task unless the user explicitly asks for Django changes.

## Commands

Run from `src/`:

```
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

Run tests (Django test runner, no pytest config present):
```
python manage.py test                # all apps
python manage.py test lists          # single app
python manage.py test api.tests.SomeTestCase   # single test case
```

Docker build (multi-stage, runs migrations at build time, serves on 0.0.0.0:8080):
```
docker build -t todoapp -f src/Dockerfile src
```

## Kubernetes manifests (`.infrastructure/`)

Apply order matters because CronJob/DaemonSet curl the app's ClusterIP service by DNS name:

1. `namespace.yml` — creates the `mateapp` namespace (used by daemonset/cronjob).
2. `deployment.yml` + `clusterIP.yml` — the todoapp Deployment and its ClusterIP Service, in the `todoapp` namespace.
3. `daemonset.yml` — `busyboxplus:curl` DaemonSet in `mateapp`, loops curling `todoapp-service.todoapp.svc.cluster.local` every 5s.
4. `cronjob.yml` — `busyboxplus:curl` CronJob in `mateapp`, runs every 4 minutes hitting `todoapp-service.todoapp.svc.cluster.local/api/health`, `successfulJobsHistoryLimit: 10`, `failedJobsHistoryLimit: 5`, `concurrencyPolicy: Allow`.

Other manifests (`hpa.yml`, `nodeport.yml`, `todoapp-pod.yml`, `busybox.yml`) are alternate/supplementary resources from earlier tasks in the series, all scoped to the `todoapp` namespace.

**Cross-namespace split is intentional**: the app itself lives in the `todoapp` namespace while the curl-based checkers (DaemonSet/CronJob) live in `mateapp`, per the task's requirements — hence the fully-qualified `*.todoapp.svc.cluster.local` DNS names in `daemonset.yml`/`cronjob.yml` rather than short service names.

`INSTRUCTIONS.md` at the repo root documents the deploy and validation steps for `daemonset.yml`/`cronjob.yml` and should be kept in sync whenever those manifests change (this is a stated requirement of the task, not optional documentation).

## App architecture (`src/`)

Standard Django project with three apps sharing one project (`todolist`):
- `lists` — the server-rendered ToDo UI (templates, views, forms).
- `accounts` — login/registration.
- `api` — DRF `ViewSet`-based REST API (`UserViewSet`, `TodoListViewSet`, `TodoViewSet`) plus two plain function views, `health` and `ready`, mounted at `/api/health` and `/api/ready`. These are what the DaemonSet/CronJob/Deployment probes hit — do not rename/remove them without updating every manifest above.

URL layout: `/` → lists app, `/auth/` → accounts, `/api/` → DRF router + health/ready.
