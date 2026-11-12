# TFM: Lista de la compra ajustada a un objetivo nutricional (Mercadona)

Herramienta pensada para ayudar a hacer la compra en Mercadona según un objetivo nutricional (por ejemplo, una dieta alta en proteína). Este repositorio contiene, por ahora, la preparación de datos (entrega 2) y un primer experimento de Machine Learning para estimar la proteína de los productos que no tienen datos nutricionales (entrega 3).

## Qué hay en el repositorio

- `pipeline_bronze_silver_gold.ipynb`: el proceso completo de la entrega 2, con explicaciones, de los datos en bruto a la tabla final (3 categorías: arroz, legumbres y pasta).
- `pipeline_ampliado.ipynb`: el mismo proceso ejecutado sobre 43 subcategorías de alimentos envasados, para tener datos suficientes para entrenar y validar un modelo (entrega 3).
- `experimento_ml.ipynb`: el experimento de Machine Learning de la entrega 3 (baseline, modelos, validación, errores y salida para el producto).
- `docs/entregas/`: documentos de cada entrega (`01_ideas_producto.md`, `02_datos_y_preparacion.md`, `03_machine_learning.md`).
- `resultados/`: métricas de los modelos, validación cruzada y fiabilidad por categoría (solo cifras agregadas, sin datos de productos).
- `data/sample/`: una muestra pequeña de los datos de la entrega 2 (3 productos en bruto, 10 filas de la tabla final y la fecha real de descarga).
- `data/manual/correcciones_manuales.csv`: las correcciones manuales que hice tras comprobar etiquetas, con el motivo y la fuente de cada una.
- `requirements.txt`: librerías necesarias.

## Cómo ejecutarlo

1. Instala las librerías: `pip install -r requirements.txt`
2. Abre `pipeline_bronze_silver_gold.ipynb` (entrega 2) o `pipeline_ampliado.ipynb` (entrega 3) con Jupyter o VS Code.
3. Cambia el texto del `User-Agent` en la primera celda de código (la línea `headers = ...`) por tu nombre o email (Open Food Facts lo pide).
4. Ejecuta todas las celdas de arriba abajo. El de la entrega 2 tarda unos minutos y el ampliado bastante más (entre 30 y 60 minutos la primera vez), porque hacen pausas entre peticiones. Si se corta, al volver a ejecutarlo retoma lo que ya está descargado.
5. Para el experimento de la entrega 3, ejecuta después `experimento_ml.ipynb`. Necesita el archivo `data/gold/productos_gold_ampliado.csv` que genera el pipeline ampliado.

Los notebooks crean las carpetas `data/raw`, `data/silver` y `data/gold`. Esas carpetas no se suben al repositorio (ver licencias), así que se generan al ejecutarlos. El catálogo de Mercadona cambia con el tiempo, por lo que el número de productos puede variar un poco.

## Fuentes de datos y licencias

**Mercadona.** Catálogo, precios, tamaños y códigos de barras, obtenidos de la API interna de su tienda online (https://tienda.mercadona.es). No es una API oficial y no tengo autorización de Mercadona. Sus términos limitan el uso de la información a fines personales y no permiten copiarla ni reproducirla sin consentimiento por escrito. Uso los datos solo con fines académicos y publico únicamente una muestra mínima (incluidas las pocas filas de ejemplo que se ven en las salidas de los notebooks), que retiraría si me lo pidieran. Los datos descargados, el Gold ampliado y las estimaciones por producto tampoco se suben al repositorio.

**Open Food Facts.** Los datos nutricionales contienen información de Open Food Facts (https://world.openfoodfacts.org), disponible bajo la licencia Open Database License (ODbL, https://opendatacommons.org/licenses/odbl/1-0/). Se cruzan con Mercadona por código de barras.

**Datos personales.** Ninguna de las dos fuentes contiene datos personales.

Más detalle en `docs/entregas/02_datos_y_preparacion.md` y `docs/entregas/03_machine_learning.md`.