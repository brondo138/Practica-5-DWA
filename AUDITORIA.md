
**Estudiante:** Alex Francisco Lovos Argueta U20241471 
**Repositorio:** https://github.com/brondo138/Practica-5-DWA.git
**URL pública:** https://practica-5-dwa.netlify.app/  
**Fecha y hora de la auditoría:** 1 de octubre de 2026, 1:49 p. m.
**Herramienta:** Lighthouse 13.4.1, desde las herramientas de desarrollador de Google Chrome en sistema MacOS.  
**Modo utilizado:** Desktop, con limitación simulada de red y CPU.
# Fase 1. Publicación y diagnóstico inicial de AstroBitácora
## 1. Medición inicial

Se realizó una auditoría inicial del sitio publicado para identificar oportunidades de optimización y establecer una referencia para comparar los resultados después de modificar el proyecto.

|Indicador|Resultado inicial|Observación|
|---|---|---|
|Desempeño|90/100|Puntaje inicial de la categoría Performance.|
|Largest Contentful Paint (LCP)|3.7 s|El elemento LCP identificado fue la imagen principal de la sección de bienvenida.|
|Cumulative Layout Shift (CLS)|0.001|Se registraron movimientos mínimos del contenido durante la prueba.|
|Interaction to Next Paint (INP)|No disponible|La auditoría no presentó esta métrica.|
|Bytes transferidos|31,908,210 bytes, aproximadamente 31.91 MB|La mayor parte corresponde a imágenes de la galería.|

## 2. Tres oportunidades o diagnósticos principales

### 1. Mejorar la entrega de imágenes

**Hallazgo:** Lighthouse detectó imágenes de la galería con dimensiones y peso superiores a lo necesario. Recomendó utilizar formatos modernos, mejorar la compresión y entregar imágenes adaptadas al tamaño de visualización.

**Evidencia:** El diagnóstico “Improve image delivery” estimó un ahorro potencial de **30,445 KiB**. Las cinco imágenes señaladas tienen dimensiones de **3200 × 2000 píxeles**; el informe propone una referencia de **738 × 461 píxeles** para su visualización en esta prueba.

**Recursos relacionados:**
- `andromeda_yw826t.jpg`
    
- `exoplanet_njwv79.jpg`
    
- `lunar-horizon_cu8glp.jpg`
    
- `orion_kfemi7.jpg`
    
- `deep-field_grr0me.jpg`
    

Estas imágenes se alojan en Cloudinary y cada una transfiere aproximadamente entre 6.22 y 6.35 MB.

**Importancia:** Es la principal oportunidad para reducir los datos descargados. El ahorro indicado es una estimación de Lighthouse y deberá comprobarse después de optimizar.

### 2. Mejorar la prioridad de descarga de la imagen principal

**Hallazgo:** En “LCP request discovery”, Lighthouse indicó que la imagen principal debería incluir `fetchpriority="high"`.

**Evidencia:** La comprobación de prioridad no se cumplió. El informe confirmó que la imagen sí se encuentra en el HTML inicial y no utiliza carga diferida. El LCP inicial fue de **3.7 segundos**.

**Recurso relacionado:** `hero-cosmos_11zon_uqgf6b.jpg`, correspondiente al elemento `img.hero__image`.

**Importancia:** Esta imagen es el elemento LCP. Dar prioridad a su descarga es una oportunidad que se evaluará para mejorar el tiempo en que aparece el contenido principal. La auditoría no cuantifica una mejora garantizada.

### 3. Revisar los recursos que bloquean el renderizado

**Hallazgo:** Lighthouse identificó archivos CSS y JavaScript que bloquean el renderizado inicial mediante el diagnóstico “Render-blocking requests”.

**Evidencia:** El informe señaló los siguientes recursos:

|Archivo|Tamaño transferido|
|---|---|
|`script.js`|877 bytes|
|`styles.css`|1,999 bytes|

**Recursos relacionados:** `script.js` y `styles.css` del sitio publicado.

**Importancia:** Aunque su peso es pequeño, su forma de carga puede retrasar la presentación inicial. Se revisará especialmente si el JavaScript puede cargarse con `defer` sin afectar el funcionamiento. El reporte no estima un ahorro temporal para este cambio.

# Fase 2. Análisis del proyecto y formulación de hipótesis

## 1. ¿Qué archivos pesan más?

Según la auditoría inicial, los recursos con mayor tamaño transferido son las cinco imágenes JPEG de la galería, alojadas en Cloudinary:

|Archivo|Tamaño transferido aproximado|
|---|---|
|`andromeda_yw826t.jpg`|6.35 MB|
|`exoplanet_njwv79.jpg`|6.31 MB|
|`lunar-horizon_cu8glp.jpg`|6.30 MB|
|`orion_kfemi7.jpg`|6.30 MB|
|`deep-field_grr0me.jpg`|6.22 MB|

Estas imágenes representan aproximadamente el **98.7 % de los bytes transferidos** durante la auditoría. Por ello, su optimización tendrá prioridad frente a cambios menores, como la minificación de CSS y JavaScript.

## 2. ¿Todas las imágenes necesitan descargarse inmediatamente?

No. La imagen principal es visible al abrir la página y corresponde al elemento LCP, por lo que debe comenzar a descargarse lo antes posible.

Las imágenes de la galería están debajo de la primera pantalla en la prueba realizada. Son candidatas a utilizar `loading="lazy"` para aplazar su descarga hasta que se acerquen al área visible.

Los fragmentos de HTML incluidos en el informe muestran estas imágenes sin el atributo `loading="lazy"`.

## 3. ¿Hay imágenes sin dimensiones explícitas?

Sí. El diagnóstico “Image elements do not have explicit width and height” señaló la imagen principal y las cinco imágenes de la galería.

Se propone agregar atributos `width` y `height` con las proporciones correspondientes, conservando su adaptación mediante CSS.

El CLS inicial fue de **0.001**, por lo que los movimientos observados ya fueron mínimos. Este cambio busca reservar correctamente el espacio de las imágenes y mantener la estabilidad visual; no se espera necesariamente una reducción importante de CLS.

## 4. ¿El recurso visual principal utiliza un formato y tamaño razonables para la web?

La imagen principal, `hero-cosmos_11zon_uqgf6b.jpg`, utiliza JPEG y transfirió aproximadamente **382 KB**. Su peso es considerablemente menor al de las imágenes de la galería, pero, al ser el elemento LCP, merece una revisión específica.

Se comparará su versión actual con una alternativa en WebP o AVIF, verificando que conserve la calidad visual. También se revisarán sus dimensiones originales frente al tamaño de presentación, ya que los datos analizados no permiten concluir que esté sobredimensionada.

Lighthouse señaló que esta imagen no tiene `fetchpriority="high"`. Sí está disponible desde el HTML inicial y no utiliza carga diferida.

## 5. ¿Cómo se está cargando el JavaScript?

Lighthouse identificó `script.js` como un recurso que bloquea el renderizado inicial. Su tamaño transferido fue de **877 bytes**, por lo que el aspecto principal a revisar es su forma de carga.

Antes de modificarlo, se inspeccionará la etiqueta `<script>` y sus dependencias. Si es un script externo clásico que necesita el HTML y admite ejecución diferida, se evaluará agregar `defer`.

No se elegirá `async` sin revisar primero si el script depende del documento o del orden de ejecución de otros scripts.

## 6. ¿Hay recursos que podrían entregarse en un formato más eficiente?

Sí. Lighthouse identificó las cinco imágenes JPEG de la galería como candidatas a utilizar WebP o AVIF, o una compresión mayor.

Además, tienen dimensiones de **3200 × 2000 píxeles**, superiores a la referencia de **738 × 461 píxeles** indicada por el informe para esta prueba. Se evaluará reducir sus dimensiones y ofrecer tamaños adaptados a diferentes pantallas.

El ahorro potencial estimado para la entrega de imágenes fue de **30,445 KiB**. Este valor es una estimación, no un resultado ya conseguido.

## 7. Hipótesis antes de modificar el código

### Hipótesis 1: Optimización de las imágenes de la galería

Si reduzco las dimensiones de las imágenes de la galería y las convierto a WebP o AVIF con una calidad visual adecuada, espero disminuir considerablemente los bytes transferidos porque actualmente se descargan archivos JPEG de aproximadamente 6 MB cada uno.

### Hipótesis 2: Carga diferida de imágenes fuera de la primera pantalla

Si agrego `loading="lazy"` a las imágenes de la galería que aparecen debajo de la primera pantalla, espero reducir las descargas durante la carga inicial porque el navegador podrá aplazar imágenes que todavía no son necesarias.

### Hipótesis 3: Prioridad de la imagen principal

Si agrego `fetchpriority="high"` a la imagen principal y mantengo su carga sin `loading="lazy"`, espero favorecer su descarga temprana y posiblemente mejorar LCP porque es el elemento visual principal identificado por Lighthouse.



## 8. Prioridad propuesta

Se comenzará con las imágenes de la galería por su elevado peso. Después se evaluará la carga diferida y la prioridad de la imagen principal. Finalmente, se agregarán dimensiones explícitas y se revisará la carga del JavaScript.

Estas hipótesis son expectativas previas a la optimización. Su cumplimiento se determinará con las mediciones posteriores.

# Fase 3. Optimización por prioridad

Se actualizaron las referencias de las imágenes para utilizar archivos WebP de la carpeta local `assets`. También se configuró la carga diferida de la galería y se asignó prioridad alta a la imagen principal.

|Cambio aplicado|Motivo|Archivo(s)|Resultado esperado|
|---|---|---|---|
|Sustitución de las imágenes JPEG externas por imágenes WebP locales.|La auditoría inicial identificó las imágenes de la galería como los recursos de mayor peso, con aproximadamente 6 MB por imagen.|`index.html` y las seis imágenes de `assets/`: `andromeda.webp`, `deep-field.webp`, `exoplanet.webp`, `hero-cosmos.webp`, `lunar-horizon.webp` y `orion.webp`.|Reducir el peso de las imágenes y los bytes transferidos, conservando una calidad visual adecuada. La reducción se comprobará comparando los archivos y la auditoría final.|
|Incorporación de `loading="lazy"` en las cinco imágenes de la galería.|Las imágenes están debajo de la primera pantalla y no necesitan descargarse todas inmediatamente.|`index.html`: imágenes de Andrómeda, Orión, Horizonte lunar, Exoplaneta y Campo profundo.|Aplazar la descarga de imágenes alejadas del área visible y reducir los datos transferidos durante la carga inicial.|
|Incorporación de `fetchpriority="high"` en la imagen principal, sin utilizar `loading="lazy"`.|Lighthouse identificó esta imagen como el elemento LCP y señaló que no tenía una indicación de prioridad alta.|`index.html`: elemento `img.hero__image`, que ahora utiliza `assets/hero-cosmos.webp`.|Favorecer la descarga temprana de la imagen principal y posiblemente mejorar el LCP inicial de 3.7 segundos.|
# Fase 4. Medición posterior y análisis de resultados

## 1. Condiciones de las auditorías

Se realizó una segunda auditoría del sitio después de incorporar imágenes WebP locales, carga diferida en la galería y prioridad alta en la imagen principal.

**Herramienta:** Lighthouse 13.4.1 en Chrome.  
**Segunda auditoría:** 1 de octubre de 2026, 3:12 p. m.
## 2. Comparación de resultados registrados

| Indicador           | Antes            | Después         | Diferencia observada            |
| ------------------- | ---------------- | --------------- | ------------------------------- |
| Desempeño           | 90/100           | 97/100          | +7 puntos                       |
| LCP                 | 3.7 s            | 1.3 s           | Aproximadamente 2.4 s menos     |
| CLS                 | 0.000896         | 0.000357        | Aproximadamente 0.000539 menos  |
| INP                 | No disponible    | No disponible   | No se puede comparar            |
| Bytes transferidos  | 31,908,210 bytes | 3,824,971 bytes | 28,083,239 bytes menos: 88.01 % |
| TBT, dato adicional | 0 ms             | 0 ms            | Sin cambio                      |

Los valores de CLS se muestran con mayor precisión y Lighthouse los presenta redondeados como **0.001** y **0**, respectivamente; el resultado posterior no es exactamente cero.

La reducción de bytes se calculó utilizando el diagnóstico `total-byte-weight` en ambos informes. Las dos auditorías registraron la descarga de las seis imágenes

## 3. Análisis de los cambios

**¿Qué métrica cambió más?**
La reducción más clara fue la de bytes transferidos: aproximadamente un **88.01 %**. Entre los tiempos registrados, LCP pasó de 3.7 a 1.3 segundos

**¿Qué optimización redujo más bytes?**
La sustitución de las imágenes JPEG de la galería por WebP. Las 6 imágenes pasaron de transferir **31,495,303 bytes** a **3,013,282 bytes**, una reducción aproximada del **90.43 %**.

**¿Qué cambio tuvo poco impacto?**

No es posible identificar con certeza el cambio de menor impacto porque se aplicaron varias modificaciones antes de medir.

El informe confirma que la imagen principal recibió prioridad alta, pero no permite separar su efecto del cambio de formato, alojamiento y configuración de la prueba. Asimismo, las seis imágenes se descargaron durante la auditoría posterior, por lo que no se puede atribuir la reducción total de bytes a `loading="lazy"`.

**¿Apareció algún problema nuevo?**

Se detectó un aumento en el peso de la imagen principal: pasó de **382,256 bytes** a **780,994 bytes**, aproximadamente un **104.31 % más**. La nueva versión WebP pesa más que el JPEG que estaba publicado inicialmente.

Además, permanecen oportunidades de mejora:

- `script.js` y `styles.css` continúan apareciendo como recursos que bloquean el renderizado.
    
- El diagnóstico de entrega de imágenes estima un ahorro adicional de **2,985 KiB**.
    

**¿Conservaría todos los cambios en producción? ¿Por qué?**

Conservaría las versiones WebP de la galería, después de comprobar su calidad visual, por la reducción demostrada de peso. También mantendría la carga diferida en imágenes fuera de la primera pantalla y la prioridad alta en la imagen principal.

Revisaría la versión actual de `hero-cosmos.webp`, ya que aumentó el peso transferido. Antes de conservarla definitivamente, compararía alternativas con menores dimensiones o mayor compresión.

## 4. Evaluación de las hipótesis

| Hipótesis                                                          | Evaluación                                                                                                                                                                                               |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Optimizar dimensiones y formato de las imágenes de la galería      | Implementada. WebP redujo considerablemente los bytes                                                                                                                                                    |
| Aplicar carga diferida a las imágenes fuera de la primera pantalla | Implementada. Su efecto sobre las descargas iniciales requiere una comprobación específica                                                                                                               |
| Prioridad de la imagen principal                                   | Cumplida en su implementación. Se agregó `fetchpriority="high"` y se mantuvo la imagen principal sin `loading="lazy"`. Lighthouse confirmó ambas condiciones; su efecto individual sobre LCP no se aisló |
