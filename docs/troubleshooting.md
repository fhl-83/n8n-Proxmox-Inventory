# Troubleshooting

Common issues and solutions when running the Proxmox inventory workflow.

---

## No IP Detected for QEMU VMs

**Symptom:** QEMU VMs appear in the inventory but have no IP address listed (blank or `—`).

**Root cause:** IP discovery for QEMU VMs relies on the **QEMU Guest Agent** (QGA). If the agent is not installed, not enabled in Proxmox, or not running inside the VM, no IP is returned.

**Fix:**

1. **Install the agent inside the VM:**
   - Debian/Ubuntu: `apt install qemu-guest-agent`
   - RHEL/CentOS: `dnf install qemu-guest-agent`
   - Windows: install VirtIO drivers from the Proxmox ISO

2. **Enable the agent in Proxmox:**
   - Go to VM → Hardware → Add/verify "QEMU Guest Agent" is enabled
   - Or via CLI: `qm set VMID --agent enabled=1`

3. **Start and enable the agent service inside the VM:**
   ```bash
   systemctl enable --now qemu-guest-agent
   ```

4. Re-run the workflow and verify the IP appears.

---

## SSL Error: Self-Signed Certificate

**Symptom:** HTTP Request nodes targeting Proxmox return an SSL error.

**Root cause:** Proxmox uses a self-signed certificate by default.

**Fix (quick):** The workflow already has `allowUnauthorizedCerts: true` enabled on Proxmox nodes. If you still get SSL errors, verify this option is set in the node's **Options** tab.

**Fix (proper):** Replace the self-signed Proxmox certificate with a valid certificate (e.g., Let's Encrypt via ACME in Proxmox) and remove `allowUnauthorizedCerts`.

---

## Inventory Is Empty

**Symptom:** The workflow runs without error but the generated Markdown contains no rows.

**Possible causes:**

1. **Wrong node name** — the node name in the URL (`/nodes/PROXMOX_NODE_1/lxc`) must match the actual node name shown in the Proxmox web UI. Check **Datacenter** → **Nodes** for the exact name.

2. **No running containers/VMs** — the workflow only processes items returned by the API. If your node has no LXC containers or no running QEMU VMs, the output will be empty.

3. **Authentication error silently fails** — check the node output for HTTP status codes. A `200` with empty `data: []` means no containers; a `401`/`403` means authentication failed.

---

## BookStack Error 401 Unauthorized

**Symptom:** `Update BookStack LXC Page` or `Update BookStack QEMU Page` returns HTTP 401.

**Fix:**

1. Verify the `Authorization` header format: `Token TOKEN_ID:TOKEN_SECRET` (no extra spaces)
2. Verify the token has not expired (BookStack → Profile → API Tokens)
3. Verify the n8n credential is assigned to the node

---

## BookStack Error 403 Forbidden

**Symptom:** The request is authenticated but returns HTTP 403.

**Fix:**

1. Verify the BookStack user has **Edit** permission on the target page
2. Verify the page is not locked or protected
3. If using a non-admin account, ensure the user role includes content editing

---

## BookStack Error 404 Not Found

**Symptom:** The API returns HTTP 404 when updating a page.

**Fix:**

1. Verify the Page ID in the URL is correct — see [BookStack Setup](bookstack-setup.md) for how to find it
2. Verify the page still exists in BookStack (it may have been deleted)
3. Verify the BookStack base URL is correct and accessible from the n8n server

---

## ntfy Notification Not Received

**Symptom:** Workflow errors occur but no ntfy notification arrives.

**Fix:**

1. Verify the ntfy URL and topic are correct
2. Test the ntfy endpoint directly:
   ```bash
   curl -d "test" https://YOUR_NTFY_DOMAIN/YOUR_NTFY_TOPIC
   ```
3. If using a private ntfy instance with authentication, add the required `Authorization` header to the `Send ntfy Error Notification` node
4. Check that the `Error Trigger` node is connected to `Send ntfy Error Notification`

Note: ntfy is triggered only when the workflow encounters an error. Normal execution does not send a notification.

---

## Proxmox Permissions Error

**Symptom:** API returns `403 Permission check failed` or similar.

**Fix:**

1. Verify the API token has the `PVEAuditor` role assigned at path `/`
2. Verify **Privilege Separation** is enabled on the token (if enabled, the token needs its own permissions — the user's permissions are not inherited)
3. Verify the permission propagates to child paths (the **Propagate** checkbox must be checked)
4. Test with a curl command — see [Proxmox API Token Setup](proxmox-api-token.md#6-testing-the-token)

---

## Workflow Takes Very Long to Execute

**Symptom:** The workflow runs correctly but execution takes several minutes.

**Root cause:** The `Get LXC Config` node runs once per container to fetch individual container network details. On large inventories (50+ containers), this creates many sequential API calls.

**Workaround:** Reduce the number of containers processed by splitting the workflow into per-node sub-workflows, or accept the longer execution time (the workflow runs in the background and does not block anything).
