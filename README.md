# Hoyo 19 — Club de Café
## Guía de instalación completa

---

## ESTRUCTURA DEL REPO

```
hoyo19-cafe/
├── apps-script/
│   └── Code.gs          ← Backend completo (API REST)
├── app-socio/
│   └── index.html       ← App del socio (alta + ver cafés)
├── recepcion/
│   └── index.html       ← Panel de recepción (ya construido)
├── admin/
│   └── index.html       ← Panel admin (activar socios)
└── README.md
```

---

## PASO 1 — GOOGLE SHEETS

1. Ir a [sheets.google.com](https://sheets.google.com)
2. Crear nueva planilla → nombrarla **"Hoyo19 - Club de Café"**
3. Copiar el **ID** de la URL (lo que está entre `/d/` y `/edit`)
   - Ejemplo: `https://docs.google.com/spreadsheets/d/`**`1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms`**`/edit`
4. Guardarlo — lo necesitás en el Paso 2

---

## PASO 2 — APPS SCRIPT

1. Ir a [script.google.com](https://script.google.com) → **Nuevo proyecto**
2. Borrar el código vacío y pegar TODO el contenido de `apps-script/Code.gs`
3. En la línea 7, reemplazar `PEGAR_AQUI_EL_ID_DE_TU_GOOGLE_SHEET` con el ID del Paso 1
4. En la línea 8, reemplazar `549351XXXXXXX` con el número de WhatsApp del admin (formato: `549351234567`, sin `+` ni espacios)
5. Guardar el proyecto (Ctrl+S) → ponerle nombre: **"Hoyo19 API"**

### Configurar clave admin
1. En Apps Script → **Configuración del proyecto** (engranaje) → **Propiedades del script**
2. Agregar propiedad: clave = `ADMIN_KEY`, valor = una clave secreta (ej: `hoyo19-2026-admin`)
3. Guardar

### Crear las hojas automáticamente
1. En Apps Script, seleccionar la función `setupSheets` en el selector
2. Clic en ▶ **Ejecutar**
3. Aceptar los permisos que pide Google
4. Verificar en el Google Sheet que se crearon las hojas: **Socios**, **Consumos**, **Log_Activaciones**

### Publicar como API web
1. En Apps Script → **Implementar** → **Nueva implementación**
2. Tipo: **Aplicación web**
3. Ejecutar como: **Yo (tu cuenta de Google)**
4. Quién tiene acceso: **Cualquier persona**
5. Clic en **Implementar**
6. Copiar la **URL de la implementación** — se ve así:
   `https://script.google.com/macros/s/AKfycbx.../exec`
7. Guardar esa URL — la necesitás en el Paso 3

### Configurar trigger diario
1. En Apps Script → ⏰ **Activadores** (ícono del reloj)
2. Agregar activador:
   - Función: `verificarVencimientos`
   - Tipo de evento: **Basado en el tiempo**
   - Tipo de tiempo: **Temporizador día**
   - Hora: **De 2 a.m. a 3 a.m.**
3. Guardar

---

## PASO 3 — FRONTEND

En **cada archivo HTML** (`app-socio/index.html`, `recepcion/index.html`, `admin/index.html`):

Buscar esta línea:
```javascript
const API_URL = 'https://script.google.com/macros/s/TU_DEPLOYMENT_ID/exec';
```

Reemplazar `TU_DEPLOYMENT_ID` con el ID de la URL del Paso 2.

Para el admin, también reemplazar:
```javascript
const ADMIN_KEY = 'TU_CLAVE_ADMIN';
```
Con la clave que configuraste en las propiedades del script.

---

## PASO 4 — GITHUB

```bash
# En tu máquina
git init
git add .
git commit -m "feat: Hoyo 19 Club de Café - MVP inicial"
git remote add origin https://github.com/TU_USUARIO/hoyo19-cafe.git
git push -u origin main
```

---

## PASO 5 — NETLIFY

1. Ir a [netlify.com](https://netlify.com) → **Add new site** → **Import from Git**
2. Conectar con GitHub → seleccionar el repo `hoyo19-cafe`
3. Configuración del build:
   - **Build command**: (dejar vacío — es HTML estático)
   - **Publish directory**: `.` (punto — raíz del repo)
4. Clic en **Deploy site**
5. Netlify te da una URL tipo `hoyo19-cafe.netlify.app`

### URLs finales
- **App del socio**: `https://hoyo19-cafe.netlify.app/app-socio/`
- **Panel recepción**: `https://hoyo19-cafe.netlify.app/recepcion/`
- **Panel admin**: `https://hoyo19-cafe.netlify.app/admin/`

---

## FLUJO COMPLETO

```
Socio completa el form (app-socio)
    ↓
Apps Script crea fila PENDIENTE en Google Sheets
    ↓
Socio abre WhatsApp con mensaje pre-armado → manda comprobante al admin
    ↓
Admin ve el mensaje → entra al panel admin → busca el DNI → toca "Activar"
    ↓
Apps Script pone ACTIVO + asigna 20 cafés + fecha_vence = hoy + 30 días
    ↓
Socio entra a "Mis cafés" → ve sus 20 tacitas disponibles
    ↓
Va al café → el mozo consulta en el panel de recepción por DNI
    ↓
Recepción muestra nombre + cafés → mozo toca ☕ → se resta 1
    ↓
A los 30 días: trigger automático → si no renovó → INACTIVO, cafés = 0
```

---

## COSTO TOTAL

| Servicio | Costo |
|---|---|
| Google Sheets + Apps Script | Gratis |
| GitHub | Gratis |
| Netlify | Gratis (hasta 100GB/mes) |
| WhatsApp (link directo) | Gratis |
| **Total MVP** | **$0** |

---

## SOPORTE

Ante cualquier error de CORS en Apps Script, agregar esta función al final de `Code.gs`:

```javascript
function doOptions(e) {
  return ContentService.createTextOutput('')
    .setMimeType(ContentService.MimeType.JSON);
}
```

Y en cada `resp()`, agregar headers CORS si el navegador los pide.
