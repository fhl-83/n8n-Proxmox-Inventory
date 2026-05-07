# Installation Guide

This guide walks you through importing and configuring the Proxmox inventory workflow in n8n.

---

## 1. Import the Workflow

1. Open your n8n instance
2. Go to **Workflows** → **Import**
3. Select the appropriate JSON file:
   - `workflow/proxmox-inventory-full.json` — recommended (LXC + QEMU + BookStack + ntfy)
   - `workflow/proxmox-lxc-inventory.json` — LXC containers only
   - `workflow/proxmox-qemu-inventory.json` — QEMU virtual machines only
4. Click **Import**

The workflow will be imported in **inactive** state. Do not activate it until all steps below are completed.

---

## 2. Configure Proxmox Credentials

1. In n8n, go to **Credentials** → **New Credential**
2. Select type: **HTTP Header Auth**
3. Fill in:
   - **Name**: `Proxmox API Token`
   - **Name** (header): `Authorization`
   - **Value**: `PVEAPIToken=user@realm!tokenid=YOUR_SECRET`
4. Save the credential
5. Note the credential ID (visible in the URL or credential list)

See [Proxmox API Token Setup](proxmox-api-token.md) for how to create the token.

---

## 3. Configure BookStack Credentials (optional)

Skip this step if you are not using BookStack.

1. In n8n, go to **Credentials** → **New Credential**
2. Select type: **HTTP Header Auth**
3. Fill in:
   - **Name**: `BookStack API Token`
   - **Name** (header): `Authorization`
   - **Value**: `Token YOUR_TOKEN_ID:YOUR_TOKEN_SECRET`
4. Save the credential

See [BookStack Setup](bookstack-setup.md) for how to generate the API token.

---

## 4. Replace All Placeholders

Open the workflow in n8n and replace every placeholder:

### LXC HTTP Nodes (`Get LXC List (Node 1/2/3)`)

Change the URL from:
```
https://PROXMOX_HOST_1:8006/api2/json/nodes/PROXMOX_NODE_1/lxc
```
To your actual Proxmox host and node name:
```
https://pve1.example.com:8006/api2/json/nodes/pve1/lxc
```

Assign the **Proxmox API Token** credential to each node.

### QEMU HTTP Nodes (`Get QEMU List (Node 1/2)`)

Change the URL from:
```
https://PROXMOX_HOST_1:8006/api2/json/cluster/resources?type=vm
```
To your actual host.

### `Get LXC Config` Node

This node uses dynamic URLs built from item data — no URL change needed.  
Assign the **Proxmox API Token** credential.

### QEMU Agent Nodes

`Get QEMU Agent Config (Node 1)` and `Get QEMU Network Interfaces (Node 2)` also use dynamic URLs — no URL change needed.  
Assign the **Proxmox API Token** credential.

### BookStack Nodes

Change the URL in `Update BookStack LXC Page`:
```
https://YOUR_BOOKSTACK_DOMAIN/api/pages/YOUR_LXC_PAGE_ID
```

Change the URL in `Update BookStack QEMU Page`:
```
https://YOUR_BOOKSTACK_DOMAIN/api/pages/YOUR_QEMU_PAGE_ID
```

Assign the **BookStack API Token** credential to both nodes.

### ntfy Node (`Send ntfy Error Notification`)

Change the URL from:
```
https://YOUR_NTFY_DOMAIN/YOUR_NTFY_TOPIC
```
To your actual ntfy server and topic.

---

## 5. Map Nodes to Credentials

After creating credentials, make sure every HTTP node has the correct credential assigned:

| Node | Credential |
|:---|:---|
| Get LXC List (Node 1/2/3) | Proxmox API Token |
| Get LXC Config | Proxmox API Token |
| Get QEMU List (Node 1/2) | Proxmox API Token |
| Get QEMU Agent Config (Node 1) | Proxmox API Token |
| Get QEMU Network Interfaces (Node 2) | Proxmox API Token |
| Update BookStack LXC Page | BookStack API Token |
| Update BookStack QEMU Page | BookStack API Token |

---

## 6. Test Manually

Before activating the schedule:

1. Click **Execute Workflow** (play button)
2. Check each node output for errors
3. Verify the LXC and QEMU inventories are populated
4. Check that BookStack pages are updated (if configured)
5. Verify no errors in the execution log

Common issues are documented in [Troubleshooting](troubleshooting.md).

---

## 7. Activate the Schedule

Once the manual test succeeds:

1. Enable the **Schedule Trigger** node
2. Toggle the workflow to **Active**
3. The workflow will now run daily at 08:00

To change the schedule, edit the **Schedule Trigger** node parameters.
