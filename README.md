# TFM-AROA-BARBERAN

# TFM: Lista de la compra ajustada a un objetivo nutricional (Mercadona)

Herramienta pensada para ayudar a hacer la compra en Mercadona según un objetivo nutricional (por ejemplo, una dieta alta en proteína). Este repositorio contiene, por ahora, la preparación de datos: desde la descarga hasta una tabla final lista para usar.

## Qué hay en el repositorio

- `pipeline_bronze_silver_gold.ipynb`: el proceso completo, con explicaciones, de los datos en bruto a la tabla final.
- `entregas/`: documentos de cada entrega (`01_ideas_producto.md`, `02_datos_y_preparacion.md`).
- `data/sample/`: una muestra pequeña de los datos (3 productos en bruto, 10 filas de la tabla final y la fecha de descarga).
- `requirements.txt`: librerías necesarias.

## Cómo ejecutarlo

1. Instala las librerías: `pip install -r requirements.txt`
2. Abre `pipeline_bronze_silver_gold.ipynb` con Jupyter o VS Code.
3. Cambia el texto del `User-Agent` de la celda 2 por tu nombre o email (Open Food Facts lo pide).
4. Ejecuta todas las celdas de arriba abajo. Tarda unos minutos, porque hace pausas entre peticiones.

El notebook crea las carpetas `data/raw`, `data/silver` y `data/gold`. Esas carpetas no se suben al repositorio (ver licencias), así que se generan al ejecutarlo. El catálogo de Mercadona cambia con el tiempo, por lo que el número de productos puede variar un poco.

## Fuentes de datos y licencias

**Mercadona.** Catálogo, precios, tamaños y códigos de barras, obtenidos de la API interna de su tienda online (https://tienda.mercadona.es). No es una API oficial. Sus términos limitan el uso de la información a fines personales y no permiten copiarla ni reproducirla sin consentimiento, por lo que uso los datos solo con fines académicos y publico únicamente una muestra mínima.

**Open Food Facts.** Los datos nutricionales contienen información de Open Food Facts (https://world.openfoodfacts.org), disponible bajo la licencia Open Database License (ODbL, https://opendatacommons.org/licenses/odbl/1-0/). Se cruzan con Mercadona por código de barras.

**Datos personales.** Ninguna de las dos fuentes contiene datos personales.

Más detalle en `entregas/02_datos_y_preparacion.md`.
