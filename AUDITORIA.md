 # Bitácora de auditoría: AstroBitácora

**Estudiante:** [STANLEY JOSUE PINEDA ARGUETA]
**URL publicada:** https://astro-performance-lab-cyan.vercel.app
**Fecha:** 30/09/2026

---

## 1. Medición inicial

**Fecha y hora de la prueba:** 30/09/2026, 18:59
**Herramienta y modo:** Lighthouse en Chrome DevTools, modo Móvil, solo categoría Rendimiento

| Indicador | Resultado inicial | Observación |
|---|---|---|
| Desempeño | 67 | Zona naranja (50 a 89) |
| LCP | 3,7 s | Por encima del objetivo de 2,5 s. El elemento LCP es la imagen `hero__image`, servida desde Cloudinary |
| CLS | 0 | Ya era perfecto antes de optimizar |
| INP | No disponible | Lighthouse no lo mide; solo aparece con datos reales de campo |
| Bytes transferidos / peso total | ~31,9 MB (31.866 kB, 9 solicitudes) | Medido en la pestaña Network, sin limitaciones y con caché inhabilitada |

Otras métricas de la misma corrida: TBT 1060 ms (rojo), FCP 1,0 s, Speed Index 2,0 s.

Desglose del LCP (corrida de las 21:00): TTFB 80 ms, retraso de carga del recurso 90 ms, duración de carga del recurso 500 ms, retraso de renderizado del elemento 490 ms.


### Tres hallazgos principales

**1. Hallazgo:** El peso total de la página es desproporcionado: unos 31,9 MB para una sola pantalla de contenido.
**Evidencia:** Pestaña Network, barra inferior: "31.866 kB transferidos", 9 solicitudes.
**Recurso o archivo relacionado:** La imagen hero y las cinco imágenes de la galería, todas en formato JPEG.

**2. Hallazgo:** El LCP de 3,7 s depende de una imagen hero pesada y de un renderizado retrasado.
**Evidencia:** Desglose del LCP con duración de carga de 500 ms y retraso de renderizado de 490 ms. El elemento LCP es `<img class="hero__image">`.
**Recurso o archivo relacionado:** `index.html` (etiqueta del hero, sin `width` ni `height`) y la imagen `hero-cosmos` en JPEG.

**3. Hallazgo:** El script principal se carga de forma bloqueante y el TBT es de 1060 ms.
**Evidencia:** `<script src="script.js">` está en el `<head>` sin `defer`, y Lighthouse marca "Solicitudes que bloquean el renderizado". El contenido de `script.js` es ligero (solo manejadores de eventos), por lo que el TBT alto se relaciona más probablemente con la decodificación de imágenes pesadas que con el propio script.
**Recurso o archivo relacionado:** `script.js` y su etiqueta en `index.html`.

---

## 2. Hipótesis antes de modificar

**Preguntas de inspección**

- **¿Qué archivos pesan más?** Las imágenes JPEG: la imagen hero y las cinco de la galería concentran casi todo el peso total (~31,9 MB). `styles.css` y `script.js` pesan muy poco en comparación.
- **¿Todas las imágenes necesitan descargarse inmediatamente?** No. Solo el hero se ve al abrir la página. Las cinco imágenes de la galería (Andrómeda, Orión, Horizonte lunar, Mundo 51-b y La noche más antigua) están fuera del primer viewport y no tienen `loading="lazy"`.
- **¿Hay imágenes sin dimensiones explícitas?** Sí. Ninguna `<img>` de `index.html` tiene `width` ni `height`. Aun así, el CLS medido es 0 porque el CSS ya reserva espacio.
- **¿El recurso visual principal usa un formato y tamaño razonables?** No. El hero es un JPEG pesado y es el elemento LCP; su duración de carga fue de 500 ms.
- **¿Cómo se está cargando el JavaScript?** Con `<script src="script.js">` en el `<head>`, sin `defer` ni `async`, por lo que bloquea el parseo del HTML. El script solo actualiza el año del footer, abre y cierra el menú y cuenta clics en un botón.
- **¿Hay recursos que podrían entregarse en un formato más eficiente?** Sí. Todas las imágenes están en JPEG y pueden pasar a WebP.

**Hipótesis**

1. Si convierto la imagen hero a WebP y la redimensiono al tamaño con que realmente se muestra, espero mejorar el LCP (duración de carga de 500 ms) y reducir los bytes transferidos, porque el hero es el elemento LCP y un JPEG sobredimensionado tarda más en descargarse.
2. Si agrego `defer` al `<script>` del `<head>`, espero una mejora pequeña en el retraso de renderizado del LCP (490 ms), porque el navegador deja de detener el parseo del HTML, aunque el script sea ligero. Si además optimizo las imágenes, espero que el TBT baje bastante más, porque habrá menos datos que decodificar en el hilo principal.
3. Si agrego `loading="lazy"`, `width` y `height` a las cinco imágenes de la galería, espero reducir los bytes iniciales y evitar riesgo de CLS, porque el navegador no descargará lo que está fuera del primer viewport y reservará el espacio antes de que lleguen las imágenes.
4. Si agrego `width` y `height`, espero que el CLS cambie poco, porque ya era 0 antes de optimizar.

---

## 3. Cambios aplicados

| Cambio | Motivo | Archivo(s) | Resultado esperado |
|---|---|---|---|
| Agregar `defer` al script principal | El script bloquea el parseo del HTML en el `<head>` | `index.html` | Mejora pequeña en el retraso de renderizado; el script sigue funcionando porque usa `DOMContentLoaded` |
| Convertir el hero a WebP, redimensionarlo y agregar `width`, `height` y `fetchpriority="high"` (sin `lazy`) | El hero es el elemento LCP y es un JPEG pesado | `index.html`, `assets/img/` | Menor duración de carga del LCP y gran reducción de bytes |
| Convertir la galería a WebP y agregar `width`, `height`, `loading="lazy"` y `decoding="async"` | Cinco imágenes pesadas que no se ven al cargar | `index.html`, `assets/img/` | Menos bytes iniciales, menor TBT y espacio reservado para cada imagen |
| Minificar CSS y JS (descartado) | `script.js` pesa menos de 1 KB, el ahorro sería insignificante | No aplica | Impacto despreciable |

---

## 4. Medición final


| Indicador | Antes | Después | Diferencia |
|---|---|---|---|
| Desempeño | 67 | [completar] | [completar] |
| LCP | 3,7 s | [completar] | [completar] |
| CLS | 0 | [completar] | [completar] |
| INP | No disponible | [completar] | [completar] |
| Bytes transferidos / peso total | ~31,9 MB | [completar] | [completar] |

Otras métricas de apoyo: TBT 1060 ms antes, [completar] después.

---

## 5. Conclusión breve

**Cambio con mayor impacto:** [completar tras la medición final. Se espera que sea la optimización de imágenes]

**Cambio con menor impacto:** [completar tras la medición final. Se espera `defer` y `width`/`height`, porque el script es ligero y el CLS ya era 0]

---

## Preguntas finales

**1. ¿Qué significa LCP y cuál fue el elemento LCP de tu página antes y después de optimizar?**
LCP (Largest Contentful Paint) mide cuánto tarda en pintarse el elemento más grande visible en el primer viewport. Antes de optimizar, el elemento LCP fue la imagen `hero__image` (imagen del campo estelar con nebulosa, servida desde Cloudinary en JPEG). Después de optimizar: [completar].

**2. ¿Cómo puede una imagen sin dimensiones explícitas contribuir a CLS?**
Sin `width` y `height`, el navegador no sabe cuánto espacio reservar hasta que la imagen se descarga. Cuando llega, empuja el contenido que está debajo y se produce un cambio de diseño. En este proyecto el CLS ya era 0 porque el CSS reserva espacio, pero declarar las dimensiones sigue siendo una buena práctica.

**3. ¿Por qué `loading="lazy"` es útil en una galería, pero puede ser una mala idea para una imagen principal visible al cargar?**
En una galería, las imágenes están fuera del primer viewport, así que diferirlas ahorra bytes iniciales. Una imagen principal visible al cargar es normalmente el LCP; con `lazy` el navegador retrasa su descarga hasta calcular el layout, lo que empeora el LCP. Por eso el hero lleva `fetchpriority="high"` y no `lazy`.

**4. Explica la diferencia entre `defer` y `async`. ¿Cuál elegiste para este proyecto y por qué?**
Ambos descargan el script sin bloquear el parseo del HTML. `async` lo ejecuta apenas termina de descargarse, sin garantizar el orden y posiblemente antes de que el DOM esté listo. `defer` espera a que el HTML termine de parsearse y ejecuta los scripts en orden, justo antes de `DOMContentLoaded`. Elegí `defer` porque `script.js` registra su lógica dentro de `DOMContentLoaded`: con `async` podría ejecutarse después de ese evento y el menú, el contador y el año nunca funcionarían.

**5. ¿Qué formato de imagen elegiste y qué comparación de peso obtuviste?**
Elegí WebP por su soporte amplio en navegadores y su menor peso a calidad visual similar. Comparación de peso: [completar con el peso del hero y de la galería antes y después, y el total de la página].

**6. Si el puntaje de desempeño sube, pero el LCP sigue por encima del objetivo, ¿consideras terminada la optimización? Justifica.**
No. El puntaje es una combinación ponderada de varias métricas y puede subir por otras (por ejemplo, el TBT) mientras el LCP sigue lento. El LCP es una de las métricas centrales que percibe el usuario; si sigue sobre 2,5 s, hay que investigar su desglose (TTFB, retraso de carga, duración de carga y retraso de renderizado) y seguir optimizando la fase que más pesa.

