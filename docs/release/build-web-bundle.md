# Build Web Bundle

This document explains how to build and package the ScipionWeb frontend.

---

# Step 1 — Install Dependencies

```bash
npm install
```

---

# Step 2 — Production Build

```bash
npm run build
```

This produces:

```
dist/
```

---

# Step 3 — Validate Dist

Ensure:

```
dist/index.html
dist/assets/
```

Open locally:

```bash
npx serve dist
```

---

# Step 4 — Create Bundle

Option A — Zip dist directly:

```bash
zip -r scipionweb-1.2.0-dist.zip dist/
```

Option B — Versioned folder:

```bash
mkdir scipionweb-1.2.0
cp -r dist scipionweb-1.2.0/
zip -r scipionweb-1.2.0-dist.zip scipionweb-1.2.0/
```

---

# Step 5 — Upload

Upload to:

```
https://scipion.cnb.csic.es/downloads/scipion/scipionWeb/
```

---

# Runtime Configuration

The API installer injects:

```
dist/config.js
```

Which defines:

```js
window.__SCIPION_WEB_CONFIG__ = {
  apiBaseUrl: "/api"
}
```

No rebuild required for different deployments.

---

# Advantages

✔ Same web build everywhere  
✔ Runtime-configurable API URL  
✔ CDN friendly  
✔ Cacheable  

---

# Validation Checklist

- Web loads without API
- API base URL injected correctly
- SPA routing works