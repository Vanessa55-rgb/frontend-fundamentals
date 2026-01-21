# Renderizado Web: CSR, SSR, SSG e ISR

## ¿Qué es el Renderizado?

El renderizado es el proceso de convertir código (HTML, CSS, JS) en una página web visual que el usuario puede ver e interactuar.

---

## 1. CSR - Client-Side Rendering

### Definición

El HTML se genera completamente en el **navegador del cliente** usando JavaScript.

### Flujo CSR

```
1. Cliente solicita página
2. Servidor envía HTML mínimo + JS bundle
3. Cliente descarga JS
4. JS se ejecuta y genera HTML
5. Cliente ve la página
```

### Ejemplo Completo

```html
<!-- index.html (mínimo) -->
<!DOCTYPE html>
<html>
<head>
  <title>Mi App</title>
</head>
<body>
  <div id="root"></div>
  <script src="/bundle.js"></script>
</body>
</html>
```

```jsx
// App.jsx (React CSR)
import { useState, useEffect } from 'react';

function App() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Fetch en el CLIENTE
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>Cargando...</div>;
  
  return (
    <div>
      <h1>Usuarios</h1>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}

export default App;
```

```javascript
// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

// Renderiza en el cliente
ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

### Visualización del Proceso

```
Servidor:
  ├─ Envía HTML vacío: <div id="root"></div>
  └─ Envía bundle.js (React + App)

Cliente:
  ├─ Recibe HTML vacío (pantalla blanca)
  ├─ Descarga bundle.js (200KB-2MB)
  ├─ Ejecuta JavaScript
  ├─ React renderiza componentes
  ├─ Fetch datos de API
  └─ Muestra contenido final
  
Total: 2-5 segundos hasta ver contenido
```

### Ventajas CSR

- ✅ **Interactividad completa**: Transiciones suaves, SPA
- ✅ **Menor carga del servidor**: Solo sirve archivos estáticos
- ✅ **Experiencia app-like**: Navegación sin recargas
- ✅ **Fácil de escalar**: Usa CDN para assets

### Desventajas CSR

- ❌ **SEO limitado**: Crawlers ven HTML vacío
- ❌ **Slow First Paint**: Pantalla blanca inicial
- ❌ **Depende de JavaScript**: Si falla JS, la app no funciona
- ❌ **Bundle grande**: Mucho código para descargar

### Cuándo usar CSR

```
✅ Aplicaciones internas (dashboards, admin panels)
✅ Apps que requieren autenticación
✅ SPAs con mucha interactividad
✅ Cuando SEO no es crítico
```

---

## 2. SSR - Server-Side Rendering

### Definición

El HTML se genera en el **servidor** en cada petición y se envía completamente renderizado al cliente.

### Flujo SSR

```
1. Cliente solicita página
2. Servidor ejecuta código (React/Vue)
3. Servidor genera HTML completo
4. Servidor envía HTML + JS
5. Cliente ve HTML inmediatamente
6. JS "hidrata" para interactividad
```

### Ejemplo Completo

```jsx
// pages/users.jsx (Next.js SSR)
export async function getServerSideProps(context) {
  // Se ejecuta en el SERVIDOR en cada request
  const res = await fetch('https://api.example.com/users');
  const users = await res.json();
  
  return {
    props: { users }
  };
}

export default function UsersPage({ users }) {
  // El componente recibe datos ya listos
  return (
    <div>
      <h1>Usuarios</h1>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

```javascript
// Express.js SSR manual
import React from 'react';
import { renderToString } from 'react-dom/server';
import App from './App';

app.get('/users', async (req, res) => {
  // Fetch datos en servidor
  const users = await db.users.findAll();
  
  // Renderizar React a HTML string
  const html = renderToString(<App users={users} />);
  
  res.send(`
    <!DOCTYPE html>
    <html>
      <head><title>Users</title></head>
      <body>
        <div id="root">${html}</div>
        <script>window.__DATA__ = ${JSON.stringify(users)}</script>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});
```

### Visualización del Proceso

```
Servidor:
  ├─ Recibe request /users
  ├─ Consulta base de datos
  ├─ Ejecuta React components
  ├─ Genera HTML completo
  └─ Envía HTML + JS al cliente

Cliente:
  ├─ Recibe HTML completo (ve contenido inmediatamente)
  ├─ Descarga bundle.js
  └─ React "hidrata" el HTML (agrega event listeners)
  
Total: 0.5-1 segundo hasta ver contenido
```

### Ventajas SSR

- ✅ **SEO excelente**: HTML completo para crawlers
- ✅ **Fast First Paint**: Usuario ve contenido inmediato
- ✅ **Funciona sin JS**: Contenido visible aunque JS falle
- ✅ **Mejor para móviles**: Menos procesamiento en el cliente

### Desventajas SSR

- ❌ **Carga del servidor**: Renderiza en cada request
- ❌ **TTFB lento**: Espera procesamiento del servidor
- ❌ **Costoso**: Requiere servidores potentes
- ❌ **Difícil de escalar**: No puede usar solo CDN

### Cuándo usar SSR

```
✅ E-commerce (productos, precios dinámicos)
✅ Noticias y blogs con comentarios
✅ Redes sociales (feeds personalizados)
✅ Dashboards con datos en tiempo real
✅ Cuando SEO es crítico
```

---

## 3. SSG - Static Site Generation

### Definición

El HTML se genera en **tiempo de build** (una sola vez) y se sirve como archivos estáticos.

### Flujo SSG

```
1. Build time: Generar HTML de todas las páginas
2. Cliente solicita página
3. Servidor (o CDN) envía HTML pre-generado
4. Cliente ve HTML inmediatamente
5. JS hidrata para interactividad
```

### Ejemplo Completo

```jsx
// pages/blog/[slug].jsx (Next.js SSG)

// 1. Genera lista de páginas en build time
export async function getStaticPaths() {
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());
  
  const paths = posts.map(post => ({
    params: { slug: post.slug }
  }));
  
  return {
    paths,
    fallback: false // o 'blocking' o true
  };
}

// 2. Genera contenido de cada página en build time
export async function getStaticProps({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`)
    .then(r => r.json());
  
  return {
    props: { post }
  };
}

// 3. Componente se renderiza a HTML estático
export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
      <p>Autor: {post.author}</p>
    </article>
  );
}
```

### Build Process

```bash
# npm run build genera:

.next/
├─ blog/
│  ├─ post-1.html    # HTML estático pre-generado
│  ├─ post-2.html
│  └─ post-3.html
└─ index.html

# Cada archivo ya contiene el HTML completo
```

### Visualización del Proceso

```
Build Time (una vez):
  ├─ Fetch todos los posts
  ├─ Para cada post:
  │  ├─ Ejecutar getStaticProps
  │  ├─ Renderizar componente a HTML
  │  └─ Guardar como archivo .html
  └─ Deploy a CDN

Runtime (cada request):
  ├─ Cliente solicita /blog/post-1
  ├─ CDN sirve post-1.html (instantáneo)
  └─ Cliente ve contenido inmediatamente
  
Total: <100ms hasta ver contenido
```

### Ventajas SSG

- ✅ **Velocidad máxima**: Archivos estáticos desde CDN
- ✅ **SEO perfecto**: HTML completo pre-generado
- ✅ **Económico**: No requiere servidor dinámico
- ✅ **Escalable**: CDN maneja millones de requests
- ✅ **Seguro**: No hay servidor para hackear

### Desventajas SSG

- ❌ **No dinámico**: Mismo contenido para todos
- ❌ **Build lento**: Regenerar miles de páginas
- ❌ **Contenido desactualizado**: Requiere rebuild para actualizar
- ❌ **No personalizable**: No puede mostrar datos del usuario

### Cuándo usar SSG

```
✅ Blogs y sitios de contenido
✅ Documentación
✅ Landing pages de marketing
✅ Portfolios
✅ E-commerce (páginas de producto estables)
✅ Cualquier contenido que cambia poco
```

---

## 4. ISR - Incremental Static Regeneration

### Definición

Combina SSG con regeneración automática en background. Las páginas se regeneran periódicamente sin rebuild completo.

### Flujo ISR

```
1. Primera request → Sirve HTML estático
2. Después de X segundos → Regenera en background
3. Siguientes requests → Reciben versión actualizada
```

### Ejemplo Completo

```jsx
// pages/products/[id].jsx (Next.js ISR)

export async function getStaticPaths() {
  // Solo generar las 100 páginas más populares
  const topProducts = await fetch('https://api.example.com/top-products')
    .then(r => r.json());
  
  const paths = topProducts.map(product => ({
    params: { id: product.id.toString() }
  }));
  
  return {
    paths,
    fallback: 'blocking' // Genera otras páginas on-demand
  };
}

export async function getStaticProps({ params }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`)
    .then(r => r.json());
  
  return {
    props: { product },
    revalidate: 60 // Regenerar cada 60 segundos
  };
}

export default function ProductPage({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>Precio: ${product.price}</p>
      <p>Stock: {product.stock}</p>
    </div>
  );
}
```

### Visualización del Proceso

```
Build time:
  └─ Genera HTML de top 100 productos

Request 1 (t=0s):
  └─ Usuario A solicita producto #1
  └─ Sirve HTML estático (rápido)

Background (t=60s):
  └─ ISR regenera producto #1 automáticamente
  └─ Actualiza HTML con datos frescos

Request 2 (t=65s):
  └─ Usuario B solicita producto #1
  └─ Sirve HTML actualizado (con nuevos datos)

Request para producto no generado:
  └─ Usuario solicita producto #5000
  └─ Genera HTML on-demand (primera vez: lento)
  └─ Cachea para siguientes requests (rápido)
```

### Configuración de Revalidación

```jsx
// Revalidar cada 10 segundos
return {
  props: { data },
  revalidate: 10
};

// Revalidar cada hora
return {
  props: { data },
  revalidate: 3600
};

// Revalidar on-demand (API route)
// pages/api/revalidate.js
export default async function handler(req, res) {
  await res.revalidate('/products/1');
  return res.json({ revalidated: true });
}
```

### Ventajas ISR

- ✅ **Rápido como SSG**: Primera carga es estática
- ✅ **Siempre actualizado**: Regeneración automática
- ✅ **Escalable**: Usa CDN + regeneración inteligente
- ✅ **Sin rebuild**: Actualiza sin redesplegar
- ✅ **On-demand generation**: Genera páginas según necesidad

### Desventajas ISR

- ❌ **Complejidad**: Más difícil de configurar
- ❌ **Stale content**: Usuarios pueden ver versión vieja
- ❌ **Solo Next.js**: No disponible en todos los frameworks
- ❌ **Costos**: Requiere servidor para regeneración

### Cuándo usar ISR

```
✅ E-commerce grande (miles de productos)
✅ Noticias (contenido cambia cada hora)
✅ Blogs con muchos posts
✅ APIs externas con datos cambiantes
✅ Cuando necesitas SSG pero con datos frescos
```

---

## Comparación Completa

### Tabla Comparativa

| Característica | CSR | SSR | SSG | ISR |
|----------------|-----|-----|-----|-----|
| **Velocidad inicial** | ❌ Lento | ⚡ Rápido | ⚡⚡ Muy rápido | ⚡⚡ Muy rápido |
| **SEO** | ❌ Malo | ✅ Excelente | ✅ Excelente | ✅ Excelente |
| **Contenido dinámico** | ✅ Sí | ✅ Sí | ❌ No | ⚡ Parcial |
| **Costo servidor** | 💰 Bajo | 💰💰💰 Alto | 💰 Muy bajo | 💰💰 Medio |
| **Build time** | ⚡ Rápido | ⚡ Rápido | ❌ Lento | ⚡ Rápido |
| **Actualización** | ⚡ Instantánea | ⚡ Instantánea | ❌ Requiere rebuild | ⚡ Automática |
| **Personalización** | ✅ Completa | ✅ Completa | ❌ No | ❌ No |
| **Escalabilidad** | ✅ Excelente | ❌ Limitada | ✅ Excelente | ✅ Excelente |

### Diagramas de Timeline

```
CSR:
|--- HTML vacío ---|--- Download JS ---|--- Fetch datos ---|--- Render ---|
0ms              200ms               1000ms            1500ms         2000ms
                                                                     ⬆ FCP

SSR:
|--- Request ---|--- Server render ---|--- HTML listo ---|--- Hydrate ---|
0ms           200ms                 500ms              800ms           1000ms
                                      ⬆ FCP

SSG:
|--- HTML desde CDN ---|--- Hydrate ---|
0ms                  50ms            200ms
     ⬆ FCP

ISR:
|--- HTML desde CDN ---|--- Hydrate ---|--- (Background revalidate) ---|
0ms                  50ms            200ms
     ⬆ FCP
```

---

## Estrategias Híbridas

### Combinar Múltiples Técnicas

```jsx
// Next.js App Router - Mixing strategies

// app/page.jsx - SSG (home page)
export default async function Home() {
  const data = await fetch('https://api.example.com/featured', {
    next: { revalidate: 3600 } // ISR: cada hora
  });
  
  return <FeaturedSection data={data} />;
}

// app/dashboard/page.jsx - SSR (user specific)
export default async function Dashboard() {
  const user = await getUser(); // Cada request
  
  return <UserDashboard user={user} />;
}

// app/blog/[slug]/page.jsx - ISR
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());
  return posts.map(post => ({ slug: post.slug }));
}

export default async function BlogPost({ params }) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`, {
    next: { revalidate: 60 } // ISR: cada minuto
  }).then(r => r.json());
  
  return <Post data={post} />;
}
```

---

## Casos de Uso Reales

### E-commerce

```
├─ Home page: ISR (revalidate: 1h)
├─ Categorías: ISR (revalidate: 30min)
├─ Productos: ISR (revalidate: 5min)
├─ Carrito: CSR
├─ Checkout: SSR
└─ Perfil usuario: SSR
```

### Blog/Noticias

```
├─ Home: ISR (revalidate: 5min)
├─ Artículos: ISR (revalidate: 1h)
├─ Comentarios: CSR
└─ Búsqueda: CSR
```

### SaaS Dashboard

```
├─ Landing: SSG
├─ Pricing: SSG
├─ Docs: SSG
├─ Login: CSR
├─ Dashboard: CSR
└─ Admin panel: SSR
```

---

## Recursos

- [Next.js Rendering](https://nextjs.org/docs/basic-features/pages)
- [Web.dev - Rendering Patterns](https://web.dev/rendering-on-the-web/)
- [Patterns.dev](https://www.patterns.dev/)
- [React Server Components](https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023)
