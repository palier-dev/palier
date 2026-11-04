# Infrastructure — local lab

Local POC platform: a Talos Kubernetes cluster (1 control plane, worker nodes for workloads), deployed with Argo CD, with off-site backups on the Hostinger VPS (NixOS, reached over WireGuard).

| Path | Content |
|---|---|
| `talos/` | Talos machine-config patches (apply with `talosctl`) |
| `security/` | Argo CD Applications for cluster security components (cert-manager) |
| `storage/` | Argo CD Applications for storage (OpenEBS) |
| `storage/postgres/` | Argo CD Applications for CloudNativePG, the Barman Cloud plugin and the Palier cluster |
| `storage/postgres/palier-postgres/` | CloudNativePG cluster, backup target, schedule and sealed S3 credentials (Kustomize) |

Decisions and rationale: see the decision log (Talos cluster storage, Postgres, backups).

## Data layer — install order

| Step | What | How |
|---|---|---|
| 1 | User volume on each worker | `talosctl patch machineconfig -n <worker-ip> --patch @talos/uservolume-openebs.yaml` |
| 2 | OpenEBS LocalPV Hostpath (default StorageClass) | `storage/openebs.yaml` |
| 3 | cert-manager (required by the Barman Cloud plugin) | `security/cert-manager.yaml` (sync wave -1); check with `cmctl check api` |
| 4 | CloudNativePG operator | `storage/postgres/cloudnative-pg.yaml` (sync wave 0) |
| 5 | Barman Cloud plugin | `storage/postgres/plugin-barman-cloud.yaml` (sync wave 1) |
| 6 | Garage bucket and key on the VPS | see "Backup target" below |
| 7 | Sealed S3 credentials | see "Secrets" below |
| 8 | Postgres cluster + backups | `storage/postgres/palier-postgres.yaml` (sync wave 2) |

Sync waves only order Applications managed by a parent (app-of-apps). Otherwise, sync them in this order by hand.

Pin every `targetRevision: PIN_ME` to an explicit chart version before syncing.

## Backup target (Garage on the VPS)

Garage runs natively on the VPS (NixOS module, see the `hostinger` repo, `hosts/hostinger-vps1/garage.nix`). Its S3 API listens on port 3900, reachable only through WireGuard (`wg0` is trusted; no public port is opened).

One-time setup on the VPS, after `nixos-rebuild switch`:

```bash
sudo garage status                                   # note the node ID
sudo garage layout assign -z vps1 -c 100G <node-id>
sudo garage layout apply --version 1
sudo garage bucket create palier-pg-backups
sudo garage key create cnpg-backup
sudo garage bucket allow --read --write palier-pg-backups --key cnpg-backup
sudo garage key info cnpg-backup --show-secret       # access key ID + secret
```

Barman Cloud 3.16+ no longer creates buckets automatically: the bucket must exist before the first backup.

Connectivity check from the cluster (expects an S3 XML error, which proves the endpoint is reachable):

```bash
kubectl run s3check --rm -it --restart=Never --image=curlimages/curl -- curl -s http://192.168.27.66:3900
```

## Secrets (sealed, safe to commit)

Plain Secrets never go to Git. Credentials are encrypted with Sealed Secrets (`kubeseal`): only the controller in the cluster can decrypt them. A SealedSecret is bound to its name and namespace, so seal it for `palier-data`.

```bash
kubectl create secret generic garage-backup-creds \
  --namespace palier-data \
  --from-literal=ACCESS_KEY_ID=<from garage key info> \
  --from-literal=ACCESS_SECRET_KEY=<from garage key info> \
  --from-literal=REGION=garage \
  --dry-run=client -o yaml \
| kubeseal --format yaml \
  > storage/postgres/palier-postgres/garage-backup-creds.sealed.yaml
```

Then uncomment `garage-backup-creds.sealed.yaml` in `storage/postgres/palier-postgres/kustomization.yaml`. The namespace is created by Argo CD (`CreateNamespace=true`).

Back up the Sealed Secrets controller key outside the cluster: without it, sealed files in Git can't be decrypted after a cluster rebuild.

## Checks after the Postgres sync

```bash
kubectl -n palier-data get cluster palier-pg                 # Cluster in healthy state, 2/2 ready
kubectl -n palier-data get pods -o wide                      # one instance per worker
kubectl -n palier-data get backups                           # first backup completed
kubectl -n palier-data exec -it palier-pg-1 -- psql -d palier -c '\dx'   # vector, unaccent
```

Then test a restore into a throwaway cluster: a backup is only proven by a restore.
