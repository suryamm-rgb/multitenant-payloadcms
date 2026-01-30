# 🏢 Multi-Tenant Setup Documentation (Payload CMS + Next.js)

This document explains **how multi-tenancy is implemented** in this project and provides a **step-by-step guide** to understand and extend it.

---

## 📌 What is Multi-Tenancy?

Multi-tenancy allows **one application** to serve **multiple websites (tenants)** while keeping their content separated.

Each tenant has:
- Its **own domain**
- Its **own pages and posts**
- Its **own header and footer**
- Its **own branding (logo, etc.)**

All managed from a **single Payload CMS instance**.

---

## 🎯 Why We Use Multi-Tenancy

| Benefit | Description |
|--------|-------------|
| 🌍 Multiple websites | One app can power many domains |
| 💰 Lower cost | No need for separate servers/projects |
| 🧠 Centralized CMS | Manage all tenants from one admin panel |
| 🔒 Data isolation | Tenants only access their own content |
| ⚡ Faster updates | Features apply to all tenants instantly |

---

# 🛠 Step-by-Step Implementation

---

## ✅ Step 1: Create Tenants Collection

This collection stores each website (tenant).

```ts
// collections/Tenants.ts
export const Tenants: CollectionConfig = {
  slug: 'tenants',
  fields: [
    { name: 'name', type: 'text', required: true },
    { name: 'domain', type: 'text' }, // client domain (example.com)
    { name: 'slug', type: 'text', required: true }, // used for routing
    { name: 'logo', type: 'upload', relationTo: 'media' },
    { name: 'allowPublicRead', type: 'checkbox', defaultValue: false },
  ],
}
```

🔹 Each tenant represents **one website/domain**

---

## ✅ Step 2: Enable Multi-Tenant Plugin

This plugin automatically links content to tenants.

```ts
// plugins/index.ts
multiTenantPlugin<Config>({
  collections: {
    pages: {},
    posts: {},
    header: { isGlobal: true },
    footer: { isGlobal: true },
    media: {},
  },
  userHasAccessToAllTenants: (user) => isSuperAdmin(user),
})
```

### What this does:
✔ Adds a hidden `tenant` relationship field  
✔ Filters content per tenant  
✔ Makes Header & Footer tenant-specific  

---

## ✅ Step 3: Detect Tenant from Domain

When a request comes in, we identify the tenant using the domain.

```ts
// utilities/fetchTenantByDomain.ts
export async function fetchTenantByDomain(domain: string) {
  const domainClean = domain.split(':')[0]

  const { docs } = await payload.find({
    collection: 'tenants',
    where: { domain: { equals: domainClean } },
    limit: 1,
  })

  return docs[0] || null
}
```

Used in **Root Layout**:

```ts
const headersList = await headers()
const host = headersList.get('host') || ''
const tenant = await fetchTenantByDomain(host)
```

---

## ✅ Step 4: Rewrite Routes Based on Domain

Next.js rewrites URLs to include the tenant.

```ts
// next.config.js
async rewrites() {
  return [
    {
      source: '/:path*',
      destination: '/:tenant/:path*',
      has: [{ type: 'host', value: '(?<tenant>.*)' }],
    },
  ]
}
```

### Example

| Domain | URL | Internal Route |
|--------|-----|----------------|
| client1.com | /about | /client1.com/about |
| client2.com | /blog | /client2.com/blog |

---

## ✅ Step 5: Fetch Tenant-Specific Pages

Pages are filtered using the tenant’s domain.

```ts
where: {
  and: [
    { slug: { equals: slug } },
    { 'tenant.domain': { equals: tenantDomain } }
  ]
}
```

This ensures **only the correct tenant’s page loads**.

---

## ✅ Step 6: Tenant-Specific Header & Footer

Each tenant has its own header and footer.

```ts
const headerData = await getTenantCachedGlobal('header', 1, tenantId)()
const footerData = await getTenantCachedGlobal('footer', 1, tenantId)()
```

✔ Unique navigation per tenant  
✔ Unique branding  

---

## ✅ Step 7: Store Tenant in Cookie After Login

When a user logs into the admin panel, we store the tenant ID in a cookie.

```ts
export const setCookieBasedOnDomain = async ({ req, user }) => {
  const tenant = await req.payload.find({
    collection: 'tenants',
    where: { domain: { equals: req.headers.get('host')?.split(':')[0] } },
  })

  const tenantCookie = generateCookie({
    name: 'payload-tenant',
    value: String(tenant.docs[0].id),
  })
}
```

This ensures the admin panel shows the correct tenant’s content.

---

# 🔄 How the Full Flow Works

```text
User visits → client1.com/about
        ↓
Next.js reads domain from request
        ↓
fetchTenantByDomain("client1.com")
        ↓
Tenant found (Tenant ID = 3)
        ↓
All CMS queries filter by tenant = 3
        ↓
Correct pages, header, footer, posts are shown
```

---

# 🧪 How to Add a New Tenant

1. Go to **Payload Admin → Tenants**
2. Click **Create New Tenant**
3. Fill:
   - Name
   - Domain (example: newclient.com)
   - Slug
   - Logo (optional)
4. Save
5. Add pages/posts and assign them to this tenant

---

# 🚀 Result

| Tenant | Domain | Content | Header | Footer |
|-------|--------|---------|--------|--------|
| Tenant A | a.com | A pages only | A nav | A footer |
| Tenant B | b.com | B pages only | B nav | B footer |

All powered by **one codebase, one CMS, one deployment** 🎉

---

**End of Documentation**
