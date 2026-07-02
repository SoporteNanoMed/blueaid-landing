# BlueAid Landing · v6

Landing page en español — Sistema Operativo del Riesgo Humano (SORH)

## Estructura del proyecto

```
blueaid-deploy/                        ← raíz del repositorio git
├── .github/
│   └── workflows/
│       └── azure-static-web-apps.yml  ← CI/CD automático
├── .gitignore
└── blueaid-deploy/                    ← contenido del sitio (app_location)
    ├── index.html                     ← landing completa (autocontenida, ~60KB)
    ├── staticwebapp.config.json       ← config Azure SWA (headers, rutas, caché)
    ├── robots.txt
    ├── sitemap.xml
    └── vercel.json                    ← config Vercel (alternativa)
```

## Deploy en Azure Static Web Apps

### 1. Crear el recurso en Azure

```bash
# Con Azure CLI
az staticwebapp create \
  --name blueaid \
  --resource-group rg-blueaid \
  --location eastus2 \
  --sku Free
```

O desde el portal: [portal.azure.com](https://portal.azure.com) → Create resource → Static Web App

### 2. Obtener el token de deploy

```bash
az staticwebapp secrets list \
  --name blueaid \
  --resource-group rg-blueaid \
  --query "properties.apiKey" -o tsv
```

### 3. Configurar el secret en GitHub

En tu repositorio GitHub → Settings → Secrets and variables → Actions → New repository secret:

- **Name:** `AZURE_STATIC_WEB_APPS_API_TOKEN`
- **Value:** token del paso anterior

### 4. Push a main → deploy automático

```bash
git add .
git commit -m "Initial deploy"
git push origin main
```

El workflow en `.github/workflows/azure-static-web-apps.yml` se dispara automáticamente y despliega en ~1 minuto.

### Staging por Pull Request

Cada PR genera automáticamente un entorno de staging con URL propia. Al cerrar el PR, el entorno se destruye.

---

## Conectar dominio blueaid.ai

En Azure Portal → tu Static Web App → Custom domains → Add:

1. Agregar `blueaid.ai` → Azure te da un registro TXT o CNAME
2. Pegarlo en tu proveedor DNS (GoDaddy: dcc.godaddy.com → blueaid.ai → DNS)
3. Esperar 5-15 min → Azure valida y activa HTTPS automáticamente

---

## Desarrollo local

```bash
# Opción 1: abrir directo
open blueaid-deploy/index.html

# Opción 2: servidor local con Azure SWA CLI
npm install -g @azure/static-web-apps-cli
swa start blueaid-deploy
```

Cero dependencias de runtime — solo Google Fonts vía CDN (Inter + JetBrains Mono).

---

## Stack

- HTML5 semántico + CSS3 con custom properties
- SVG inline (íconos, mapas, gauges)
- Zero JavaScript en runtime — todas las animaciones son CSS
- Responsive: desktop / tablet / mobile

## Seguridad (headers configurados)

| Header | Valor |
|--------|-------|
| HSTS | max-age=63072000; includeSubDomains; preload |
| X-Frame-Options | SAMEORIGIN |
| X-Content-Type-Options | nosniff |
| Content-Security-Policy | style/font desde Google, sin scripts externos |
| Referrer-Policy | strict-origin-when-cross-origin |
| Permissions-Policy | camera, mic, geolocation desactivados |

---

© 2026 BlueAid SpA · Santiago, Chile · rafael@blueaid.ai
