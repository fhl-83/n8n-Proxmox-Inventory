# BookStack Setup

This workflow can automatically update BookStack wiki pages with the generated inventory. This guide explains how to set up the API token and configure the workflow.

BookStack integration is **optional** — if you don't use BookStack, simply delete or disable the `Update BookStack LXC Page` and `Update BookStack QEMU Page` nodes.

---

## 1. Create a BookStack API Token

1. Log in to BookStack as an admin
2. Go to your **Profile** → **API Tokens** (top-right menu)
3. Click **Create Token**
4. Fill in:
   - **Name**: `n8n-inventory`
   - **Expiry Date**: set an expiry date or leave blank for no expiry
5. Click **Save**
6. **Copy both the Token ID and Token Secret** — the secret is shown only once

---

## 2. Configure the n8n Credential

In n8n, create a new **HTTP Header Auth** credential:

- **Name**: `BookStack API Token`
- **Header Name**: `Authorization`
- **Header Value**:
```
Token YOUR_TOKEN_ID:YOUR_TOKEN_SECRET
```

Replace `YOUR_TOKEN_ID` and `YOUR_TOKEN_SECRET` with the values from step 1.

---

## 3. Create Dedicated Inventory Pages

Create dedicated pages in BookStack for the inventory output. These pages will be **fully overwritten** on each workflow run.

> ⚠️ **Important**: Never use a page that contains manual documentation. The workflow replaces the entire page content.

**Recommended structure:**

```
📚 Infrastructure (book)
 └─ 🗂️ Inventory (chapter)
      ├─ 📄 LXC Containers      ← used by this workflow
      └─ 📄 QEMU Virtual Machines ← used by this workflow
```

---

## 4. Find the Page ID

The Page ID is required in the workflow URL.

**Method 1 — from the URL:**  
Navigate to the page. The URL looks like:
```
https://wiki.example.com/books/infrastructure/page/lxc-containers
```
Go to **Edit** on the page, then check the URL — it will show the numeric ID:
```
https://wiki.example.com/books/1/page/42/edit
```
In this example, the Page ID is `42`.

**Method 2 — from the BookStack API:**
```bash
curl -H "Authorization: Token YOUR_TOKEN_ID:YOUR_TOKEN_SECRET" \
  https://wiki.example.com/api/pages?filter[name]=LXC+Containers
```
The response includes the `id` field.

---

## 5. Configure the Workflow Nodes

In `Update BookStack LXC Page`, set the URL to:
```
https://YOUR_BOOKSTACK_DOMAIN/api/pages/YOUR_LXC_PAGE_ID
```

In `Update BookStack QEMU Page`, set the URL to:
```
https://YOUR_BOOKSTACK_DOMAIN/api/pages/YOUR_QEMU_PAGE_ID
```

Assign the `BookStack API Token` credential to both nodes.

---

## 6. API Method

The workflow uses `PUT /api/pages/{id}` with the following body:

```json
{
  "name": "LXC Containers",
  "editor": "markdown",
  "markdown": "... generated content ..."
}
```

The `editor: "markdown"` field is required to ensure BookStack renders the content as Markdown and not as HTML.

---

## 7. Required Permissions

The BookStack API token inherits the permissions of the user who created it. Ensure that user has:

- **View** permission on the target book/chapter
- **Edit** permission on the target pages

The simplest approach is to use an admin account's token, but a dedicated editor role is more secure.
