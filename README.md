# TFM: Lista de la compra ajustada a un objetivo nutricional (Mercadona)

Herramienta pensada para ayudar a hacer la compra en Mercadona según un objetivo nutricional (por ejemplo, una dieta alta en proteína). Este repositorio contiene, por ahora, la preparación de datos (entrega 2), un primer experimento de Machine Learning para estimar la proteína de los productos que no tienen datos nutricionales (entrega 3) y el diseño del frontal con sus mockups (entrega 4).

## Qué hay en el repositorio

- `pipeline_bronze_silver_gold.ipynb`: el proceso completo de la entrega 2, con explicaciones, de los datos en bruto a la tabla final (3 categorías: arroz, legumbres y pasta).
- `pipeline_ampliado.ipynb`: el mismo proceso ejecutado sobre 43 subcategorías de alimentos envasados, para tener datos suficientes para entrenar y validar un modelo (entrega 3). Incluye el registro de la fecha real de cada descarga, el tratamiento de los productos en litros y los instantáneos (fuera del ranking por euro) y la preparación de un test nuevo con productos que el modelo no ha visto.
- `experimento_ml.ipynb`: el experimento de Machine Learning de la entrega 3: baseline, modelos, validación, rango de error, test nuevo, comprobación con etiquetas reales de productos sin datos y regla final por categoría y estado.
- `docs/entregas/`: documentos de cada entrega (`01_ideas_producto.md`, `02_datos_y_preparacion.md`, `03_machine_learning.md`, `04_diseno_frontal_ux.md`). Las imágenes de los mockups y el logo de la entrega 4 están en `docs/entregas/img/`.
- `resultados/`: métricas de los modelos, validación cruzada, test nuevo y fiabilidad por categoría y por categoría y estado (solo cifras agregadas, sin datos de productos).
- `data/sample/`: una muestra inventada (no son datos reales de Mercadona) para enseñar cómo se cruzan los precios y la nutrición por código de barras.
- `data/manual/correcciones_manuales.csv`: las correcciones manuales que hice tras comprobar etiquetas, con el motivo y la fuente de cada una.
- `data/manual/muestra_etiquetas.csv` y `data/manual/muestra_etiquetas_2.csv`: las dos muestras de productos sin datos nutricionales cuya etiqueta miré a mano (26 y 27 productos) para comprobar el error de las estimaciones. Solo guardan el identificador, el nombre, el grupo y el enlace del producto.
- `data/raw/registro_descargas.csv`: se crea al ejecutar el pipeline y anota la fecha real en que llegó cada archivo descargado. Como `data/raw` no se sube al repositorio, este archivo tampoco.
- `requirements.txt`: librerías necesarias.

## Cómo ejecutarlo

1. Instala las librerías: `pip install -r requirements.txt`
2. Abre `pipeline_bronze_silver_gold.ipynb` (entrega 2) o `pipeline_ampliado.ipynb` (entrega 3) con Jupyter o VS Code.
3. Cambia el texto del `User-Agent` en la primera celda de código (la línea `headers = ...`) por tu nombre o email (Open Food Facts lo pide).
4. Ejecuta todas las celdas de arriba abajo. El de la entrega 2 tarda unos minutos y el ampliado bastante más (entre 30 y 60 minutos la primera vez), porque hacen pausas entre peticiones. Si se corta, al volver a ejecutarlo retoma lo que ya está descargado.
5. Para el experimento de la entrega 3, ejecuta después `experimento_ml.ipynb`. Necesita `data/gold/productos_gold_ampliado.csv`, `data/gold/test_nuevo.csv` y `data/silver/productos_silver_ampliado.csv`, que genera el pipeline ampliado, así que hay que ejecutar primero ese notebook entero.

Los notebooks crean las carpetas `data/raw`, `data/silver` y `data/gold`. Esas carpetas no se suben al repositorio (ver licencias), así que se generan al ejecutarlos. El catálogo de Mercadona cambia con el tiempo, por lo que el número de productos puede variar un poco. Las etiquetas que miré a mano están escritas en `experimento_ml.ipynb` con la fecha en que las miré (22/11/2026), así que esa comprobación se puede repetir con otro catálogo, pero las muestras de productos pueden no coincidir.

## Fuentes de datos y licencias

**Mercadona.** Catálogo, precios, tamaños y códigos de barras, obtenidos de la API interna de su tienda online (https://tienda.mercadona.es). No es una API oficial y no tengo autorización de Mercadona. Sus términos limitan el uso de la información a fines personales y no permiten copiarla ni reproducirla sin consentimiento por escrito. Uso los datos solo con fines académicos. No subo los datos descargados, el Gold ampliado ni las estimaciones por producto, y la muestra de `data/sample/` es inventada. Sí hay unos pocos datos reales de Mercadona en el repositorio, y prefiero decirlo claramente: las filas que se ven en las salidas de los notebooks, las correcciones manuales (`data/manual/correcciones_manuales.csv`) y las dos muestras de etiquetas (`data/manual/muestra_etiquetas*.csv`, con nombre, grupo y enlace de 53 productos), además de la proteína de las etiquetas que miré a mano, que está escrita en `experimento_ml.ipynb`. Los retiraría si me lo pidieran.

**Open Food Facts.** Los datos nutricionales contienen información de Open Food Facts (https://world.openfoodfacts.org), disponible bajo la licencia Open Database License (ODbL, https://opendatacommons.org/licenses/odbl/1-0/). Se cruzan con Mercadona por código de barras.

**Datos personales.** Ninguna de las dos fuentes contiene datos personales.

Más detalle en `docs/entregas/02_datos_y_preparacion.md`, `docs/entregas/03_machine_learning.md` y `docs/entregas/04_diseno_frontal_ux.md`.