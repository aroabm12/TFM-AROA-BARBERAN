# Entrega 2 - Datos y preparación

Mi proyecto es una herramienta que ayuda a hacer la compra en Mercadona según un objetivo nutricional (por ejemplo, una dieta alta en proteína con un presupuesto ajustado). Para eso necesito una tabla con productos reales de Mercadona, con su precio y sus valores nutricionales. En esta entrega explico qué datos necesito, de dónde los saco, qué problemas he encontrado y cómo los preparo. Todo el proceso se puede reproducir ejecutando el notebook `pipeline_bronze_silver_gold.ipynb`.

**Una corrección sobre la Entrega 1.** Allí escribí que Mercadona publica la información nutricional completa de sus productos. Al mirar los datos reales vi que no es así en un formato que se pueda leer automáticamente: la ficha de cada producto solo trae alérgenos e ingredientes, no las calorías ni los macronutrientes, y en la web de la tienda la tabla nutricional aparece como una foto de la etiqueta, no como datos. Por eso el proyecto necesita una segunda fuente, que explico más abajo.

**Alcance de esta versión.** Empiezo con tres categorías: arroces, legumbres, y pasta y fideos (88 productos en total). Es una decisión de alcance: prefiero tener un flujo pequeño funcionando de principio a fin que un catálogo grande a medias. Además, son productos envasados, con código de barras y etiqueta nutricional, que es lo que me permite cruzar las dos fuentes. La ampliación a otras categorías está planteada como siguiente fase. Todas las cifras de este documento corresponden a la descarga del 28 de octubre de 2026.

---

## 1. Qué datos necesita el producto

**Qué es una observación.** Una observación es un producto concreto de Mercadona (por ejemplo, "Arroz redondo Hacendado"), identificado por su código de producto en Mercadona y por su código de barras (EAN). Cada fila de la tabla final es un producto.

**Datos imprescindibles.** Sin ellos el producto no funciona:
- Identificadores: código de producto y EAN.
- Nombre y categoría.
- Precio de referencia en euros por kilo, para poder comparar productos entre sí.
- Calorías, proteína, grasa e hidratos de carbono por cada 100 g, que es lo que necesito para calcular si una compra se ajusta a un objetivo.
- Estado del producto (seco, cocinado, fresco o instantáneo), porque no se pueden comparar bien un arroz seco y un arroz ya cocinado.

**Datos deseables.** Mejoran el resultado, pero se puede trabajar sin ellos:
- Fibra, azúcares y sal por 100 g.
- Tamaño del envase y formato (paquete, pack...).
- Alérgenos e ingredientes, que Mercadona sí publica.

**Granularidad.** Una fila por producto, con los valores nutricionales por 100 g y el precio por kg, tal como estaban en la fecha de descarga (guardada en `data/sample/fecha_descarga.txt`). No trabajo con series temporales de precios en esta versión.

---

## 2. Fuentes reales y viabilidad

Uso dos fuentes y las cruzo por el código de barras (EAN), nunca por el nombre del producto, porque los nombres cambian de una web a otra.

### Mercadona (catálogo, precio, tamaño y EAN)
- **Acceso:** no hay una API pública y documentada. Utilizo la misma API interna que usa su tienda online (`tienda.mercadona.es/api/...`), con el almacén de Valencia (`wh=vlc1`). El catálogo y los precios pueden depender del almacén.
- **Cobertura:** descargué las tres categorías completas: 88 productos. Los 88 tienen EAN y precio de referencia en euros por kg.
- **Actualización y estabilidad:** el catálogo cambia con el tiempo, y lo he comprobado dos veces. En una primera descarga la categoría de arroces tenía 15 productos y en una segunda tenía 14, porque uno desapareció. Y el 11 de noviembre, al volver a pedir las listas de categoría, encontré 4 productos nuevos y 2 que ya no estaban (arroces pasó de 14 a 15 y pasta de 56 a 57). Para que los resultados se puedan reproducir, el notebook reutiliza lo que ya está descargado, incluida la lista de cada categoría, así que todas las cifras de este documento son de la descarga del 28 de octubre. Además, al no ser una API oficial, Mercadona puede cambiarla o bloquearla sin avisar. Por eso guardo la fecha real de descarga y conservo los datos en bruto.
- **Alternativa si falla:** trabajar con el último catálogo descargado, que el notebook conserva en bruto en local, y buscar esos productos directamente en Open Food Facts, que tiene fichas de marcas como Hacendado.

### Open Food Facts (valores nutricionales por 100 g)
- **Acceso:** API pública (`world.openfoodfacts.org/api/v2/product/<ean>`), que se consulta por código de barras. Pide identificarse con un `User-Agent` y limita las peticiones.
- **Cobertura:** de los 88 productos de Mercadona, 77 aparecen en Open Food Facts (87,5 %). Por categoría: arroces 92,9 %, pasta 87,5 % y legumbres 83,3 %. De esos 77, 72 tienen completos los cuatro macros principales (calorías, proteína, grasa e hidratos); los otros 5 aparecen, pero con los valores vacíos.
- **Un patrón que he observado:** los 11 productos que no aparecen son todos de la marca Hacendado (la marca blanca de Mercadona), que es el 80 % de lo que descargué. En Hacendado se encuentran 59 de 70 (84,3 %); en el resto de marcas, 18 de 18. No sé la causa, así que lo dejo como una observación.
- **Actualización y estabilidad:** es una base de datos que rellena cualquier persona, así que se actualiza de forma continua, pero la calidad de cada ficha varía. Durante la descarga me bloqueó temporalmente por exceso de peticiones (error 429), así que programé esperas y reintentos.
- **Alternativa si falla o falta un producto:** leer la etiqueta del producto (foto y lectura automática del texto). Para fruta y verdura, que no tienen código de barras, habría que buscar otra fuente distinta, que todavía no he evaluado.

---

## 3. Uso, privacidad y datos personales, licencias

**Datos personales.** Ninguna de las dos fuentes contiene datos personales: son datos de productos (nombres, precios, valores nutricionales). En esta fase no recojo información de ninguna persona. Más adelante, si el producto guardara el objetivo nutricional de un usuario, sí serían datos personales y habría que tratarlos como tales, hoy no forman parte del proyecto.

**Mercadona.** Sus términos y condiciones dicen que el contenido de la web está protegido por propiedad intelectual e industrial, que la información solo puede usarse con fines personales, que el uso comercial está prohibido sin autorización previa y que no se puede copiar ni reproducir sin su consentimiento por escrito. No mencionan expresamente los robots ni la extracción automatizada de datos, y la API que utilizo no es oficial. **No tengo autorización de Mercadona**, y que mi uso sea académico y la muestra pequeña no sustituye esa autorización. Por eso he reducido lo que publico al mínimo: 3 productos en bruto y 10 filas de la tabla final, que es lo mínimo que me permite enseñar el paso de las fichas originales a las métricas del Gold. El resto de los datos se regenera ejecutando el notebook. Si Mercadona me lo pidiera, retiraría esa muestra del repositorio, y para cualquier uso fuera de este trabajo, o para publicar más datos, le pediría autorización por escrito.

**Open Food Facts.** Su base de datos está bajo la licencia Open Database License (ODbL). Los datos nutricionales de este trabajo contienen información de Open Food Facts (https://world.openfoodfacts.org), disponible bajo esa licencia (https://opendatacommons.org/licenses/odbl/1-0/). La licencia tiene dos condiciones: citar la fuente y compartir en abierto, bajo la misma licencia, las bases de datos derivadas que se publiquen. Si se combinan con otra base de datos, solo se pueden combinar con fuentes que permitan esa redistribución, y los términos de Mercadona no la permiten. Por eso no publico la tabla completa, que mezcla las dos fuentes. La muestra de 10 filas también las mezcla: la mantengo porque es lo mínimo para entender el proceso, y la retiraría si alguien me indicara que no puedo publicarla. Además, la API exige identificarse con un User-Agent propio y limita las peticiones, y mi código lo cumple.


---

## 4. Bronze → Silver → Gold

| Capa | Qué contiene | Dónde está |
|---|---|---|
| **Bronze** | Los datos tal cual los devuelven las dos webs: una ficha de categoría y una ficha de producto de Mercadona, y una ficha de Open Food Facts por cada código de barras. No se modifica nada. | `data/raw/` (se genera al ejecutar el notebook; no se sube completa, y en el repositorio hay una muestra en `data/sample/raw/`) |
| **Silver** | Los datos limpios y unidos, con marcas de calidad. 88 productos. | `data/silver/productos_silver.csv` (se genera al ejecutar el notebook; no se sube) |
| **Gold** | La tabla final: una fila por producto utilizable. 70 productos y 20 columnas. | `data/gold/productos_gold.csv` (se genera al ejecutar el notebook; no se sube, y hay una muestra de 10 filas en `data/sample/gold_muestra.csv`) |

### De Bronze a Silver
1. Quito los productos repetidos (uno puede aparecer en dos categorías).
2. Extraigo de las fichas de Open Food Facts calorías, proteína, grasa, hidratos, fibra, azúcares y sal por 100 g.
3. Añado el precio de referencia, el tamaño y el formato desde la ficha de Mercadona. Convierto los precios, que llegan como texto, a números, y compruebo que todos están en euros por kg.
4. Marco los productos que tienen los cuatro macros completos y los que tienen un código de barras con 13 dígitos (hay uno de 12).
5. **Compruebo la coherencia de las calorías.** Como Open Food Facts lo rellena cualquier persona, no doy los datos por buenos sin más. Calculo las calorías a partir de los nutrientes (4 por gramo de proteína, 4 por gramo de hidratos, 9 por gramo de grasa y 2 por gramo de fibra) y las comparo con las que declara la ficha. Con una regla simple del 10 % salían muchas falsas alarmas, porque muchas fichas no traen fibra y la fibra solo puede sumar calorías. Por eso distingo dos casos: si la fibra está informada (incluido el valor 0), la uso en el cálculo y la diferencia debe quedar dentro del 10 %; si falta, hasta un 10 % por encima de lo calculado lo doy por coherente, y entre el 10 % y el 25 % por encima lo marco como no concluyente. Clasifico cada producto en cuatro grupos:

| Estado de calidad | Productos | Significado |
|---|---|---|
| coherente | 64 | El cálculo y las calorías declaradas coinciden |
| no_concluyente | 6 | Sin dato de fibra y calorías más altas de lo calculado: puede ser fibra no informada |
| incoherente | 2 | La diferencia no se explica con la fibra |
| sin_datos | 16 | Faltan los valores (11 no están en Open Food Facts y 5 están con datos vacíos) |

Comprobé a mano las etiquetas de los dos productos incoherentes (arroz cocido redondo Sabroz y quinoa cocida Sabroz): en los dos casos el dato de Open Food Facts estaba mal.

6. **Correcciones manuales.** Al separar la fibra cero de la fibra ausente apareció otro caso. Hay dos productos que se llaman "Lenteja cocida Hacendado", en tarros distintos y con códigos de barras distintos, y en los dos Open Food Facts tenía la fibra como 0. Miré sus etiquetas en la web de Mercadona y en ninguna aparece una línea de fibra: era un dato que faltaba, escrito como 0. Lo dejo vacío, y con la fibra ausente los dos quedan como no concluyentes, que es lo que son: no sé cuánta fibra tienen, y podría explicar la diferencia de calorías. No toco los datos originales: las dos correcciones están en `data/manual/correcciones_manuales.csv`, con el motivo, la fuente y la fecha, y la columna `correccion_manual` las marca en Silver.
7. **Controles de rangos y precios.** Que las calorías cuadren no demuestra que el resto de los valores estén bien. Compruebo que las calorías estén entre 0 y 900 por 100 g, que ningún nutriente sea negativo ni supere 100 g, que proteína, grasa, hidratos y fibra no sumen más de 100 g, que los azúcares no superen a los hidratos y que el precio y el precio de referencia sean mayores que cero. Los 88 productos pasan los controles. Los que no pasaran quedarían marcados y fuera del Gold.
8. **Fecha real de descarga.** Guardo la fecha de cada archivo descargado (su fecha de modificación en `data/raw`). El notebook no reescribe lo que ya existe, ni siquiera la lista de cada categoría, así que la fecha no cambia al reutilizar la caché. Las 88 fichas de Mercadona y las de Open Food Facts son del 28 de octubre de 2026.
9. **Clasifico el estado del producto** (seco, cocinado, fresco o instantáneo) a partir del nombre, y compruebo que las calorías por 100 g de cada grupo son coherentes con su estado: los cocinados quedan por debajo de 165, los frescos entre 214 y 306 y los secos entre 292 y 394 (frescos y secos se solapan un poco).

### De Silver a Gold
- Entran los productos coherentes y los no concluyentes (70) que además superan los controles de rangos y precios (los 88 los superan). Los incoherentes y los que no tienen datos (18) se quedan en Silver, para saber qué se descartó y por qué. La columna `estado_calidad` se conserva en el Gold para que quien lo use pueda ser más estricto.
- Renombro las columnas con nombres claros y añado tres columnas calculadas: gramos de proteína por euro, calorías por euro y porcentaje de las calorías que viene de la proteína.
- **Estado del alimento.** Para comparar precio y nutrición, ambos tienen que referirse al mismo estado del alimento. En seco, cocinado y fresco, el precio por kg es el del producto tal como se vende y los rangos de calorías de cada grupo son coherentes con eso, aunque no es una comprobación completa. En los 4 productos instantáneos no he podido confirmar si la información nutricional es del producto seco o del plato preparado: tienen entre 93 y 204 kcal por 100 g, muy pocas para unos fideos secos. Por eso los dejo en la tabla pero con las columnas de proteína por euro y calorías por euro vacías, hasta resolverlo.
- El reparto por estado en el Gold es: 48 secos, 9 cocinados, 9 frescos y 4 instantáneos.

**Qué significa el Gold y para qué se usa.** Cada fila es un producto de Mercadona con datos utilizables (70 en esta versión). Las columnas principales son:
- Identificación: `product_id`, `ean`, `nombre`, `categoria_id`, `formato` y `estado` (seco, cocinado, fresco o instantáneo).
- Precio: `precio_unidad`, `precio_referencia` (en euros por kg) y `tamano_envase`.
- Nutrición por 100 g: `kcal_100g`, `proteina_100g`, `grasa_100g`, `hidratos_100g`, `fibra_100g`, `azucares_100g` y `sal_100g`.
- Calculadas: `proteina_g_por_euro`, `kcal_por_euro` y `pct_kcal_proteina`. En los productos instantáneos las dos primeras van vacías.
- Calidad: `estado_calidad`.

La siguiente fase usará esta tabla para recomendar qué productos incluir en la compra según el objetivo nutricional del usuario, por ejemplo, llegar a una cantidad de proteína diaria dentro de un presupuesto.

---

## 5. Riesgos y limitaciones

- **Dato que me falta:** Mercadona no publica macronutrientes en un formato legible, y 11 de los 88 productos (todos de Hacendado) no están en Open Food Facts, así que no entran en el Gold.
- **Sesgo y límites:** solo cubro tres categorías de productos envasados, y los datos de Open Food Facts son colaborativos: mis controles solo detectan algunos errores (un 0 puede ser un dato que falta), y en los 4 productos instantáneos no sé si la nutrición es del producto seco o del preparado, por eso no entran en los cálculos por euro.
- **Fuente no oficial:** la API de Mercadona puede cambiar (el 11 de noviembre ya había 4 productos nuevos y 2 menos) y no tengo su autorización, por lo que publico solo una muestra mínima.
- **Si una fuente principal fallara:** sin Mercadona, trabajaría con el último catálogo descargado y buscaría los productos en Open Food Facts; sin Open Food Facts, leería las etiquetas de los productos.