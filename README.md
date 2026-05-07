# 🖥️ Proxmox Inventory — n8n Workflow

Automated Proxmox infrastructure inventory using n8n.  
Collects LXC containers and QEMU VMs metadata, generates Markdown reports, updates BookStack, and sends error alerts via ntfy.

---

## ✨ Features

- **LXC inventory** — collects VMID, hostname, status, CPU, RAM, network interfaces and IPs
- **QEMU inventory** — collects running VMs with IP discovery via QEMU Guest Agent
- **Multi-node** — supports up to 3 Proxmox nodes (easily extended)
- **Markdown generation** — clean tables grouped by node, sorted by status then hostname
- **BookStack update** — pushes inventory to dedicated BookStack pages via API
- **Error notifications** — ntfy alert on workflow failure (optional)
- **Scheduled** — runs daily at 08:00 (configurable)

---

## 🏗️ Architecture

```
Schedule Trigger
    │
    ├─► [LXC Branch]
    │       Get LXC List (Node 1/2/3)
    │       Map LXC Node 1/2/3
    │       Split LXC Items
    │       Merge LXC Nodes
    │           ├─► Combine LXC Config
    │           └─► Get LXC Config (per-container detail)
    │       Normalize LXC Data
    │       Generate LXC Markdown
    │       Update BookStack LXC Page
    │
    └─► [QEMU Branch]
            Get QEMU List (Node 1/2)
            Filter Running QEMU VMs
            Normalize QEMU Fields
            Merge QEMU Nodes
                ├─► Get QEMU Agent Config
                └─► Get QEMU Network Interfaces
            Merge QEMU Config And Network
            Merge QEMU Data
            Extract QEMU Network Info
            Deduplicate QEMU VMs
            Generate QEMU Markdown
            Update BookStack QEMU Page

Error Trigger ──► Send ntfy Error Notification
```

---

## 📦 Repository Structure

```
n8n-Proxmox-Inventory/
├── README.md
├── LICENSE
├── .gitignore
├── workflow/
│   ├── proxmox-inventory-full.json     ← LXC + QEMU + BookStack + ntfy
│   ├── proxmox-lxc-inventory.json      ← LXC only
│   └── proxmox-qemu-inventory.json     ← QEMU only
├── docs/
│   ├── installation.md
│   ├── proxmox-api-token.md
│   ├── bookstack-setup.md
│   ├── troubleshooting.md
│   └── assets/
├── examples/
│   └── sample-output.md
└── screenshots/
```

---

## 🚀 Quick Start

1. **Import** `workflow/proxmox-inventory-full.json` into your n8n instance
2. **Configure credentials** — see [Installation Guide](docs/installation.md)
3. **Replace all placeholders** — listed below
4. **Test manually** before enabling the schedule

---

## 🔧 Placeholders to Replace

Every placeholder must be replaced before the workflow runs.

| Placeholder | Description | Example |
|:---|:---|:---|
| `PROXMOX_HOST_1` | Proxmox node 1 hostname or IP | `pve1.example.com` |
| `PROXMOX_HOST_2` | Proxmox node 2 hostname or IP | `pve2.example.com` |
| `PROXMOX_HOST_3` | Proxmox node 3 hostname or IP | `pve3.example.com` |
| `PROXMOX_NODE_1` | Proxmox node 1 name (in API) | `pve1` |
| `PROXMOX_NODE_2` | Proxmox node 2 name (in API) | `pve2` |
| `PROXMOX_NODE_3` | Proxmox node 3 name (in API) | `pve3` |
| `YOUR_PROXMOX_CRED_ID` | n8n credential ID for Proxmox | Created in n8n |
| `YOUR_BOOKSTACK_DOMAIN` | BookStack base URL | `https://wiki.example.com` |
| `YOUR_LXC_PAGE_ID` | BookStack page ID for LXC report | `42` |
| `YOUR_QEMU_PAGE_ID` | BookStack page ID for QEMU report | `43` |
| `YOUR_BOOKSTACK_CRED_ID` | n8n credential ID for BookStack | Created in n8n |
| `YOUR_NTFY_DOMAIN` | ntfy server URL | `https://ntfy.example.com` |
| `YOUR_NTFY_TOPIC` | ntfy topic for notifications | `infra-alerts` |
| `YOUR_N8N_INSTANCE_ID` | n8n instance ID (auto-assigned) | Assigned on import |

---

## ✅ Prerequisites

| Requirement | Notes |
|:---|:---|
| n8n ≥ 1.30 | Self-hosted or cloud |
| Proxmox VE ≥ 7.x | API access required |
| Proxmox API token | Read-only, see [docs](docs/proxmox-api-token.md) |
| QEMU Guest Agent | Required for QEMU IP discovery |
| BookStack (optional) | For wiki page updates |
| ntfy (optional) | For error notifications |

---

## 🔐 Security

- **Never** commit real credentials, IPs, or tokens to this repository
- Use a **dedicated read-only** Proxmox API token (no write permissions needed)
- Use a **dedicated** BookStack API token with page-update permissions only
- The workflow accepts self-signed Proxmox certificates (`allowUnauthorizedCerts: true`) — consider using valid certs in production
- ntfy topics should be treated as secrets; use authentication if your ntfy instance is public

---

## ⚠️ Current Limitations

- QEMU IP discovery requires **QEMU Guest Agent** to be installed and running inside each VM
- The workflow queries the Proxmox API endpoint for each container individually — on large inventories (100+ LXC), execution time increases linearly
- BookStack pages are overwritten on each run — do not store manual notes on these pages
- No historical data retention — each run overwrites the previous inventory

---

## 🗺️ Roadmap

- [ ] Support more than 3 Proxmox nodes dynamically
- [ ] Add disk usage metrics to LXC inventory
- [ ] Send ntfy notification on inventory completion (not just errors)
- [ ] Export inventory to a CSV or JSON file
- [ ] Add Prometheus metrics output support

---

## 📚 Documentation

- [Installation Guide](docs/installation.md)
- [Proxmox API Token Setup](docs/proxmox-api-token.md)
- [BookStack Setup](docs/bookstack-setup.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Sample Output](examples/sample-output.md)

---

## 📄 License

MIT — see [LICENSE](LICENSE)
