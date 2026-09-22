# Secret `pgadmin-secret` (temporaire, jusqu'à la phase 3)

Ce dépôt est public : aucun secret n'y est commité, y compris temporairement.
Le Deployment référence un Secret `pgadmin-secret` (clé
`PGADMIN_DEFAULT_PASSWORD`) qui doit exister dans le namespace `ic-webapp`
avant que le pod ne démarre correctement.

En attendant la phase 3 (External Secrets Operator + AWS Secrets Manager),
créez-le manuellement :

```bash
kubectl create secret generic pgadmin-secret \
  --namespace ic-webapp \
  --from-literal=PGADMIN_DEFAULT_PASSWORD='<mot-de-passe>'
```

La phase 3 remplacera cette étape manuelle par un `ExternalSecret` qui crée
ce même Secret (même nom, même clé) automatiquement : aucun autre fichier de
cette application n'aura à changer.
