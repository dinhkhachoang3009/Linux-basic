# Week 12 Network Worksheet

| Machine | Public IP | Private IP (VCN) | Node role |
|---|---|---|---|
| VM2 | 129.221.148.199 | 10.0.0.220 | Node 1 — LB & Gateway |
| VM3 | 168.138.168.191 | 10.0.0.48 | Node 2 — Worker 1 |
| VM1 | 129.221.159.58 | 10.0.0.97 | Node 3 — Worker 2 + NFS |
| GCP VM | 34.41.104.198 | (other cloud) | attacker / 2nd client |
| My Ubuntu machine | (dynamic — discovered at each playbook run) | — | Control Node + client |

VCN subnet (Security List source for intra-cluster rules): 10.0.0.0/24

## SSH access keys (2026-11-09 rebuild)

| Host | Key |
|---|---|
| vm1 | ~/Documents/Key/Oracle/VM1/ssh-key-2026-11-09.key |
| vm2 | ~/Documents/Key/Oracle/VM2/ssh-key-2026-11-09.key |
| vm3 | ~/Documents/Key/Oracle/VM3/ssh-key-2026-11-09.key |
| gcp | ~/.ssh/id_ed25519_gcp |
