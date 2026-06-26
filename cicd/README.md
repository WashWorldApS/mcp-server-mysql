# mcp-server-mysql Cloud Build

Pipeline wzorowany na `cam/cicd/cloudbuild.yaml`:

1. Testy (lint, build, unit + integration) z lokalnym MySQL
2. Build obrazu Docker
3. Push do Artifact Registry z tagiem = **short SHA commita** (`$SHORT_SHA`)

Deploy odbywa się przez ArgoCD — po udanym buildzie zaktualizuj tag w `argo-config`:

```yaml
# workloads/apps/mcp-server-mysql/staging/kustomization.yaml
images:
  - name: mcp-server-mysql
    newName: europe-west3-docker.pkg.dev/production-283206/ar-github-corax/mcp-server-mysql
    newTag: <SHORT_SHA>   # np. a1b2c3d z logów Cloud Build
```

## Obraz

```
europe-west3-docker.pkg.dev/production-283206/ar-github-corax/mcp-server-mysql:<SHORT_SHA>
```

`SHORT_SHA` to 7-znakowy skrót commita z GitHuba (substytucja Cloud Build).

## Utworzenie triggera (jednorazowo)

```bash
gcloud builds triggers create github \
  --name=mcp-server-mysql-staging \
  --region=europe-west3 \
  --project=production-283206 \
  --repo-name=mcp-server-mysql \
  --repo-owner=WashWorldApS \
  --branch-pattern="^main$" \
  --build-config=cicd/cloudbuild.yaml
```

Wymagania:
- Połączenie GitHub `WashWorldApS/mcp-server-mysql` z Cloud Build
- Repozytorium AR `ar-github-corax` w `europe-west3`
- Uprawnienia SA Cloud Build: push do Artifact Registry
