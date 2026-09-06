# Conectar gaelvaldivieso.com (guía completa, paso a paso)

El sitio ya está desplegado y público en Vercel:
**https://portfolio-gaels-projects-beb77207.vercel.app**

Todo el código ya apunta a `https://gaelvaldivieso.com` (canonical, Open Graph, etc.).
No hay que tocar ni un archivo más — solo comprar el dominio y crear 2 registros DNS.

Tiempo total: ~10 minutos de trabajo + espera de SSL (minutos).

---

## Paso 1 — Comprar el dominio en Cloudflare

1. **dash.cloudflare.com** → misma cuenta que usás para el Email Routing de Torre de Control.
2. Menú lateral: **Domain Registration → Register Domains**.
   - Primera compra: te pide agregar un método de pago.
3. Buscá **`gaelvaldivieso.com`**. Precio esperado: **~$10.44/año** (precio a costo, sin markup).
4. Al carrito → datos de contacto → pagar. El **WHOIS privacy es gratis y automático**.
5. Cloudflare crea la **zona DNS** del dominio automáticamente. No tocás nameservers.

> **Si tu cuenta no muestra "Register Domains":** comprá en **Porkbun.com** (~$11, mismo
> precio, sin cuenta previa). Después, en Porkbun → el dominio → **Authoritative Nameservers**,
> podés (a) dejar los de Porkbun y crear ahí los registros del Paso 3, o (b) cambiar a los
> nameservers que te dé Cloudflare al añadir el dominio como sitio en Cloudflare y crear los
> registros ahí. Los valores de los registros son los mismos.

---

## Paso 2 — Añadir el dominio en Vercel

1. **vercel.com** → proyecto **`portfolio`** → **Settings** → **Domains**.
2. Escribí `gaelvaldivieso.com` → **Add**.
3. Cuando pregunte por la variante `www`, elegí **"Add gaelvaldivieso.com and redirect
   www.gaelvaldivieso.com to it"** (o añadí las dos y marcá `gaelvaldivieso.com` como **Primary**).
4. Vercel ahora muestra los registros DNS que necesita. **Usá los valores que muestre
   la pantalla** (los IPs de Vercel cambian con el tiempo). Van a ser algo así:

   | Para | Tipo | Valor (confirmá en la pantalla de Vercel) |
   |---|---|---|
   | `gaelvaldivieso.com` (apex) | **A** | `216.198.79.1`  *(a veces `76.76.21.21`)* |
   | `www.gaelvaldivieso.com` | **CNAME** | `cname.vercel-dns.com` |

   Dejá esta pestaña abierta para el Paso 3.

---

## Paso 3 — Crear los 2 registros DNS en Cloudflare

Cloudflare dashboard → dominio `gaelvaldivieso.com` → **DNS** → **Records**.

**Primero:** si Cloudflare creó registros de ejemplo al comprar el dominio (un `A` hacia
`192.0.2.x`, un `CNAME` de `www`, un `AAAA`, etc.), **borralos o editalos** — no deben quedar.

Luego **Add record** dos veces:

### Registro 1 — apex
| Campo | Valor |
|---|---|
| Type | `A` |
| Name | `@`  *(Cloudflare lo muestra como `gaelvaldivieso.com`)* |
| IPv4 address | el IP que mostró Vercel (ej. `216.198.79.1`) |
| Proxy status | **DNS only** — click en la nube para que quede **GRIS**, no naranja |
| TTL | Auto |

### Registro 2 — www
| Campo | Valor |
|---|---|
| Type | `CNAME` |
| Name | `www` |
| Target | `cname.vercel-dns.com` |
| Proxy status | **DNS only** — nube **GRIS** |
| TTL | Auto |

> ⚠️ **La nube tiene que estar GRIS (DNS only), no naranja (Proxied).**
> Con la nube naranja, Vercel no puede emitir el certificado SSL y puede haber bucles de
> redirección. Más adelante, si querés el CDN/caché de Cloudflare, se puede activar el proxy
> — pero para arrancar: **gris**.

---

## Paso 4 — Esperar SSL (automático)

- Volvé a Vercel → Settings → Domains. Los dominios pasan de **"Invalid Configuration"**
  (rojo) a un **check verde** cuando el DNS resuelve.
- Vercel emite el certificado **Let's Encrypt solo, sin configurar nada**.
- **Tiempos:** con Cloudflare el DNS propaga en segundos–minutos. El SSL suele estar listo
  en **1–5 min**, a veces hasta ~30 min. El peor caso teórico de DNS (48 h) prácticamente
  no pasa con Cloudflare.

---

## Paso 5 — Verificar (comandos para correr en la terminal)

```bash
dig gaelvaldivieso.com +short
# esperado: el IP de Vercel (ej. 216.198.79.1)

dig www.gaelvaldivieso.com +short
# esperado: cname.vercel-dns.com. y luego un IP

curl -sI https://gaelvaldivieso.com | grep -iE '^HTTP|^server'
# esperado: HTTP/2 200   +   server: Vercel

curl -sI https://www.gaelvaldivieso.com | grep -iE '^HTTP|^location'
# esperado: HTTP/2 308   +   location: https://gaelvaldivieso.com/

curl -sI https://gaelvaldivieso.com/og.png | grep -iE '^HTTP|content-type'
# esperado: HTTP/2 200   +   content-type: image/png

echo | openssl s_client -connect gaelvaldivieso.com:443 -servername gaelvaldivieso.com 2>/dev/null \
  | openssl x509 -noout -issuer -dates
# esperado: issuer= ...Let's Encrypt... + fechas vigentes
```

Si los `curl` a `https://` dan error de certificado durante los primeros minutos, es normal
— esperá y reintentá; Vercel todavía está emitiendo el cert.

---

## Paso 6 — Últimas cosas

- **Probar la tarjeta social:** pegá `https://gaelvaldivieso.com` en
  [opengraph.dev](https://www.opengraph.dev/) o en el
  [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/). Debería salir la
  tarjeta naranja `og.png`. (LinkedIn cachea fuerte: si ya compartiste el link antes de que
  el dominio estuviera vivo, el Post Inspector fuerza el refresco.)
- **Actualizar el link viejo:** donde hayas pegado el link del artifact de Claude
  (`claude.ai/code/artifact/...`) — LinkedIn "Contact info" / "Featured", CV, firma de correo,
  bio de X/GitHub — cambialo por **https://gaelvaldivieso.com**.
- **El artifact de Claude sigue existiendo** pero ya **no es la fuente de verdad**. De acá en
  adelante los cambios van en este repo (editás `index.html` → `git push` → Vercel redespliega
  solo en ~30 s).
