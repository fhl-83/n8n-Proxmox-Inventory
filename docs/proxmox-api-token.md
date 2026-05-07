# Proxmox API Token Setup

This workflow uses the Proxmox VE REST API with token-based authentication. This guide explains how to create a dedicated read-only API token.

---

## 1. Create a Dedicated User (recommended)

Using a dedicated user limits blast radius if the token is ever compromised.

In Proxmox web UI:

1. Go to **Datacenter** → **Permissions** → **Users**
2. Click **Add**
3. Fill in:
   - **User ID**: `inventory@pve`
   - **Realm**: `pve` (Proxmox VE authentication server)
   - **Password**: (set a strong password, even if you'll use token auth)
4. Click **Add**

---

## 2. Create the API Token

1. Go to **Datacenter** → **Permissions** → **API Tokens**
2. Click **Add**
3. Fill in:
   - **User**: `inventory@pve`
   - **Token ID**: `n8n-inventory`
   - **Privilege Separation**: ✅ enabled (token has its own permission set)
4. Click **Add**
5. **Copy the token secret immediately** — it is shown only once

The resulting token string will look like:
```
inventory@pve!n8n-inventory=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

## 3. Set Minimum Required Permissions

The inventory workflow only needs **read** access. Assign the following permissions to the token:

1. Go to **Datacenter** → **Permissions** → **Add** → **API Token Permission**
2. Set:
   - **Path**: `/`  (root — applies to all nodes)
   - **API Token**: `inventory@pve!n8n-inventory`
   - **Role**: `PVEAuditor` (built-in read-only role)
   - **Propagate**: ✅

The `PVEAuditor` role includes:
- `VM.Audit` — list and inspect VMs and containers
- `Sys.Audit` — inspect node system information
- `Datastore.Audit` — inspect storage

No write permissions are granted.

---

## 4. Authorization Header Format

In n8n, configure the credential as an **HTTP Header Auth**:

- **Header Name**: `Authorization`
- **Header Value**:
```
PVEAPIToken=inventory@pve!n8n-inventory=YOUR_TOKEN_SECRET
```

Replace `YOUR_TOKEN_SECRET` with the actual secret copied in step 2.

---

## 5. Security Recommendations

- Use `Privilege Separation: enabled` on the token — this prevents token inheritance of the user's full permissions
- Do **not** reuse the same token for different automations
- Store the token secret in n8n credentials only — never in environment variables or config files committed to Git
- Rotate the token periodically (Proxmox → Permissions → API Tokens → Regenerate)
- Consider restricting the permission path to specific nodes (`/nodes/pve1`) instead of `/` if your cluster has sensitive nodes

---

## 6. Testing the Token

You can verify the token works with a quick curl test:

```bash
curl -k -H "Authorization: PVEAPIToken=inventory@pve!n8n-inventory=YOUR_SECRET" \
  https://YOUR_PROXMOX_HOST:8006/api2/json/nodes
```

Expected response: a JSON object with a `data` array listing your nodes.

If you receive a `401 Unauthorized` or `403 Permission check failed`, see [Troubleshooting](troubleshooting.md).
