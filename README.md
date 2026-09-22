# GitOps_Kubernetes

Dépôt GitOps synchronisé par ArgoCD (installé et géré par
[`terraform/app`](https://github.com/LoicPierret/Projet-DevOps) — dépôt
infra), avec le pattern *app of apps*.

## Structure

```
bootstrap/          Application racine + une Application par composant
  platform-app.yaml   -> apps/platform
  ic-webapp-app.yaml  -> apps/ic-webapp
  odoo-app.yaml        -> apps/odoo
  pgadmin-app.yaml      -> apps/pgadmin

apps/
  platform/    StorageClass EBS + Ingress partagé (un seul ALB pour les 3 hôtes)
  ic-webapp/   Frontend applicatif
  odoo/        ERP Odoo
  pgadmin/     Interface d'administration PostgreSQL
```

`terraform/app` crée une unique `Application` racine pointant vers
`bootstrap/`. ArgoCD découvre alors les quatre `Application` de ce dossier
et synchronise chacune vers son propre chemin `apps/<nom>`.

Le namespace `ic-webapp` est créé par Terraform (`kubernetes_namespace_v1`),
pas par ce dépôt : les manifests le référencent (`namespace: ic-webapp`)
sans le déclarer.

## Secrets

**Aucun secret n'est commité dans ce dépôt public.** Les applications `odoo`
et `pgadmin` référencent des Secrets Kubernetes qui doivent exister dans le
cluster avant leur premier démarrage :

- voir [`apps/odoo/README.md`](apps/odoo/README.md)
- voir [`apps/pgadmin/README.md`](apps/pgadmin/README.md)

Une phase ultérieure (External Secrets Operator + AWS Secrets Manager)
automatisera cette étape sans changer les manifests des applications.

## Accès à ArgoCD

```bash
aws eks update-kubeconfig --name main-cluster --region us-east-1
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Puis ouvrir `https://localhost:8080` (utilisateur `admin`, mot de passe
initial : `kubectl -n argocd get secret argocd-initial-admin-secret -o
jsonpath='{.data.password}' | base64 -d`).

## Limites connues

- Les images `ic-webapp:latest` et `odoo` (sans tag) ne sont pas figées :
  une future étape CI écrira un tag immuable (SHA du commit) dans ce dépôt.
- L'Ingress partagé (`apps/platform`) référence des Services possédés par
  d'autres `Application` : couplage volontaire pour ne garder qu'un seul ALB.
