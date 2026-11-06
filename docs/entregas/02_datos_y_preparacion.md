# Entrega 2 - Datos y preparación

Mi proyecto es una herramienta que ayuda a hacer la compra en Mercadona según un objetivo nutricional (por ejemplo, una dieta alta en proteína con un presupuesto ajustado). Para eso necesito una tabla con productos reales de Mercadona, con su precio y sus valores nutricionales. En esta entrega explico qué datos necesito, de dónde los saco, qué problemas he encontrado y cómo los preparo. Todo el proceso se puede reproducir ejecutando el notebook `pipeline_bronze_silver_gold.ipynb`.

**Una corrección sobre la Entrega 1.** Allí escribí que Mercadona publica la información nutricional completa de sus productos. Al mirar los datos reales vi que no es así en un formato que se pueda leer automáticamente: la ficha de cada producto solo trae alérgenos e ingredientes, no las calorías ni los macronutrientes. Por eso el proyecto necesita una segunda fuente, que explico más abajo.

**Alcance de esta versión.** Empiezo con tres categorías: arroces, legumbres, y pasta y fideos (88 productos en total). Es una decisión de alcance: prefiero tener un flujo pequeño funcionando de principio a fin que un catálogo grande a medias. Además, son productos envasados, con código de barras y etiqueta nutricional, que es lo que me permite cruzar las dos fuentes. La ampliación a otras categorías está planteada como siguiente fase.

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
- **Actualización y estabilidad:** el catálogo cambia con el tiempo. Lo comprobé yo misma: en una primera descarga la categoría de arroces tenía 15 productos y en una segunda tenía 14, porque un producto desapareció. Además, al no ser una API oficial, Mercadona puede cambiarla o bloquearla sin avisar. Por eso guardo la fecha de descarga y conservo los datos en bruto.
- **Alternativa si falla:** repetir la descarga a mano con otra herramienta es poco práctico. La alternativa realista sería partir de una muestra guardada (la del repositorio) y construir el catálogo con una descarga puntual.

### Open Food Facts (valores nutricionales por 100 g)
- **Acceso:** API pública (`world.openfoodfacts.org/api/v2/product/<ean>`), que se consulta por código de barras. Pide identificarse con un `User-Agent` y limita las peticiones.
- **Cobertura:** de los 88 productos de Mercadona, 77 aparecen en Open Food Facts (87,5 %). Por categoría: arroces 92,9 %, pasta 87,5 % y legumbres 83,3 %. De esos 77, 72 tienen completos los cuatro macros principales (calorías, proteína, grasa e hidratos); los otros 5 aparecen, pero con los valores vacíos.
- **Un patrón que he observado:** los 11 productos que no aparecen son todos de la marca Hacendado (la marca blanca de Mercadona), que es el 80 % de lo que descargué. En Hacendado se encuentran 59 de 70 (84,3 %); en el resto de marcas, 18 de 18. No sé la causa, así que lo dejo como una observación.
- **Actualización y estabilidad:** es una base de datos que rellena cualquier persona, así que se actualiza de forma continua, pero la calidad de cada ficha varía. Durante la descarga me bloqueó temporalmente por exceso de peticiones (error 429), así que programé esperas y reintentos.
- **Alternativa si falla o falta un producto:** leer la etiqueta del producto (foto y lectura automática del texto). Para fruta y verdura, que no tienen código de barras, habría que buscar otra fuente distinta, que todavía no he evaluado.

---

## 3. Uso, privacidad y datos personales, licencias

**Datos personales.** Ninguna de las dos fuentes contiene datos personales: son datos de productos (nombres, precios, valores nutricionales). En esta fase no recojo información de ninguna persona. Más adelante, si el producto guardara el objetivo nutricional de un usuario, sí serían datos personales y habría que tratarlos como tales, hoy no forman parte del proyecto.

**Mercadona.** Sus términos y condiciones indican que el contenido de la web está protegido por propiedad intelectual e industrial, que la información solo puede usarse con fines personales, que el uso comercial está prohibido sin autorización previa y que no se puede copiar ni reproducir sin su consentimiento por escrito. No mencionan expresamente el uso de robots ni la extracción automatizada de datos, y la API que utilizo no es oficial. Por eso lo planteo como un uso académico y limitado: descargo solo tres categorías, con una pausa de un segundo entre peticiones, y no publico el catálogo. En el repositorio solo hay una muestra mínima (3 productos en bruto y 10 filas de la tabla final), y los datos completos se regeneran ejecutando el notebook. No se puede dar un uso comercial al producto sin pedir autorización a Mercadona.

**Open Food Facts.** Su base de datos está bajo la licencia Open Database License (ODbL). Los datos nutricionales de este trabajo contienen información de Open Food Facts (https://world.openfoodfacts.org), disponible bajo esa licencia (https://opendatacommons.org/licenses/odbl/1-0/). La licencia tiene dos condiciones: citar la fuente y compartir en abierto, bajo la misma licencia, las bases de datos derivadas que se publiquen. Si se combinan con otra base de datos, solo se pueden combinar con fuentes que permitan esa redistribución, y los términos de Mercadona no la permiten. Por eso no publico la tabla completa, que mezcla las dos fuentes, y dejo en el repositorio solo una muestra pequeña. Además, la API exige identificarse con un User-Agent propio y limita las peticiones, y mi código lo cumple.

---

## 4. Bronze → Silver → Gold

| Capa | Qué contiene | Dónde está |
|---|---|---|
| **Bronze** | Los datos tal cual los devuelven las dos webs: una ficha de categoría y una ficha de producto de Mercadona, y una ficha de Open Food Facts por cada código de barras. No se modifica nada. | `data/raw/` (no se sube completa; en el repositorio hay una muestra en `data/sample/`) |
| **Silver** | Los datos limpios y unidos, con marcas de calidad. 88 productos. | `data/silver/productos_silver.csv` |
| **Gold** | La tabla final: una fila por producto utilizable. 70 productos y 20 columnas. | `data/gold/productos_gold.csv` |

### De Bronze a Silver
1. Quito los productos repetidos (uno puede aparecer en dos categorías).
2. Extraigo de las fichas de Open Food Facts calorías, proteína, grasa, hidratos, fibra, azúcares y sal por 100 g.
3. Añado el precio de referencia, el tamaño y el formato desde la ficha de Mercadona.
4. Marco los productos que tienen los cuatro macros completos y los que tienen un código de barras con 13 dígitos (hay uno de 12).
5. **Compruebo la coherencia de las calorías.** Como Open Food Facts lo rellena cualquier persona, no doy los datos por buenos sin más. Calculo las calorías a partir de los nutrientes (4 por gramo de proteína, 4 por gramo de hidratos, 9 por gramo de grasa y 2 por gramo de fibra) y las comparo con las que declara la ficha. Con una regla simple del 10 % salían muchas falsas alarmas, porque muchas fichas no traen fibra y la fibra solo puede sumar calorías. Por eso clasifico cada producto en cuatro grupos:

| Estado de calidad | Productos | Significado |
|---|---|---|
| coherente | 64 | El cálculo y las calorías declaradas coinciden |
| no_concluyente | 6 | Sin dato de fibra y calorías más altas de lo calculado: puede ser fibra no informada |
| incoherente | 2 | La diferencia no se explica con la fibra |
| sin_datos | 16 | Faltan los valores (11 no están en Open Food Facts y 5 están con datos vacíos) |

Comprobé a mano las etiquetas de los dos productos incoherentes (arroz cocido redondo Sabroz y quinoa cocida Sabroz): en los dos casos el dato de Open Food Facts estaba mal.

6. **Clasifico el estado del producto** (seco, cocinado, fresco o instantáneo) a partir del nombre, y compruebo que las calorías por 100 g de cada grupo quedan en rangos separados.

### De Silver a Gold
- Entran los productos coherentes y los no concluyentes (70). Los incoherentes y los que no tienen datos (18) se quedan en Silver, para saber qué se descartó y por qué. La columna `estado_calidad` se conserva en el Gold para que quien lo use pueda ser más estricto.
- Renombro las columnas con nombres claros y añado tres columnas calculadas: gramos de proteína por euro, calorías por euro y porcentaje de las calorías que viene de la proteína.
- El reparto por estado en el Gold es: 48 secos, 9 cocinados, 9 frescos y 4 instantáneos.

---

## 5. Riesgos y limitaciones

- **Calidad de los datos nutricionales:** Open Food Facts lo rellena cualquier persona y puede tener errores. Mi comprobación solo detecta los que afectan a las calorías, así que no puedo asegurar que los 64 productos "coherentes" estén todos bien. Además, el estado del producto (seco, cocinado...) se deduce del nombre y puede fallar con productos nuevos; los 4 productos instantáneos tienen calorías muy bajas para ser fideos secos y no he podido confirmar por qué.
- **Cobertura incompleta:** 11 de los 88 productos (el 12 %) no están en Open Food Facts, todos de Hacendado, la marca más habitual, y quedan fuera de la tabla final.
- **Muestra limitada:** los resultados son de tres categorías de productos envasados y no se pueden generalizar a todo el supermercado. La fruta, la verdura y la carne a granel quedan para una segunda fase.
- **Fuente no oficial y cambiante:** Mercadona puede cambiar o cerrar la API, y su catálogo cambia con el tiempo, así que una nueva ejecución puede dar un número de productos distinto.
- **Licencias:** los términos de Mercadona restringen el uso de su información a fines personales, y Open Food Facts exige compartir en abierto los datos derivados. Por eso solo publico una muestra pequeña, y un uso comercial del producto exigiría la autorización de Mercadona.
