# Guías Diagnósticas — Firebase + Netlify (sin login)

## Archivos

```
guias-diagnosticas/
├── index.html                 ← App (editar con tu config de Firebase)
├── netlify.toml               ← Config de Netlify
└── netlify/functions/
    └── claude.js              ← Proxy seguro a Anthropic
```

---

## PASO 1 — Firebase: crear proyecto y Firestore

1. Ve a https://console.firebase.google.com → **Crear proyecto**
2. Menú izquierdo: **Build → Firestore Database → Crear base de datos**
   - Modo: **Producción**
   - Región: `us-central` (o la más cercana)

### Reglas de Firestore

En Firestore → pestaña **Reglas**, pega esto y publica:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /historial/{docId} {
      allow read, write: if true;
    }
  }
}
```

> Esto es abierto para pruebas. Cuando agregues autenticación después,
> cambiaremos las reglas para que cada usuario solo acceda a sus datos.

### Índice compuesto (necesario para el historial)

Firestore → **Índices → Crear índice**:
- Colección: `historial`
- Campo 1: `deviceId` (Ascending)
- Campo 2: `savedAt` (Descending)
- Clic en **Crear** (tarda ~2 min)

---

## PASO 2 — Obtener la config de Firebase

Firebase Console → ⚙️ → **Configuración del proyecto** → sección **Tus apps** → ícono **</>**

Registra una app web y copia el objeto `firebaseConfig`.

Abre `index.html` y reemplaza el bloque marcado con 🔥:

```js
const firebaseConfig = {
  apiKey:            "AIzaSy...",
  authDomain:        "tu-proyecto.firebaseapp.com",
  projectId:         "tu-proyecto",
  storageBucket:     "tu-proyecto.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123:web:abc"
};
```

---

## PASO 3 — Subir a Netlify

1. https://app.netlify.com → **Add new site → Deploy manually**
2. Arrastra la carpeta `guias-diagnosticas`
3. **Site configuration → Environment variables → Add**:
   - Key: `ANTHROPIC_API_KEY`
   - Value: tu llave `sk-ant-...`
4. **Deploys → Trigger deploy**

---

## Cómo funciona el historial sin login

Cada navegador genera un ID único anónimo que se guarda en `localStorage`.
El historial en Firestore se vincula a ese ID, así persiste aunque el
estudiante cierre la pestaña o recargue la página.

Cuando agregues autenticación más adelante, migraremos el historial
al `uid` real del usuario.

---

## Costos (todo gratis para pruebas)

| Servicio | Límite gratuito |
|---|---|
| Netlify hosting | Ilimitado |
| Netlify Functions | 125,000 req/mes |
| Firestore | 1 GB, 50,000 lecturas/día |
| Anthropic API | ~$0.003 por consulta |
