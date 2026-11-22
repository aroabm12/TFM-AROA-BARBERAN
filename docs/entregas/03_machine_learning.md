# Entrega 3 - Machine Learning

Esta entrega parte del Gold de la entrega 2 (`docs/entregas/02_datos_y_preparacion.md`) y lo amplía para poder entrenar y validar un modelo. El código está en `pipeline_ampliado.ipynb` (datos) y `experimento_ml.ipynb` (experimento). Las métricas están en `resultados/`.

## 1. Qué problema resuelve el modelo

Mi producto ayuda a hacer la compra en Mercadona según un objetivo nutricional, por ejemplo conseguir la máxima proteína por euro. Para recomendar un producto necesito saber sus nutrientes, y los saco de Open Food Facts. El problema es que no todos los productos de Mercadona están allí, o no tienen los datos completos o coherentes. De los 1373 productos distintos que descargué, 269 (un 20 %) no tienen datos nutricionales utilizables en Open Food Facts, y sin datos el producto no puede recomendarlos. Otros 59 tienen datos incoherentes y también los descarto; por ahora solo estimo los 269 sin datos.

El modelo estima los gramos de proteína por 100 g de uno de esos productos usando solo lo que Mercadona sí publica: nombre, categoría, precio y formato. Es un problema de **regresión**. Lo usará la persona que compra, y la decisión que le permite tomar es si incluir ese producto en su lista aunque su proteína sea una estimación y no el dato de la etiqueta.

Elegí la proteína porque es la base de mi métrica principal, la proteína por euro. Otros nutrientes quedan para más adelante.

## 2. Datos de entrada y salida

**Datos.** Uso el Gold ampliado, que sale del mismo pipeline Bronze → Silver → Gold de la entrega 2, con los mismos controles de calidad, pero con 43 subcategorías de alimentos envasados en lugar de 3 (arroz, legumbres y pasta). Con aquellas 3 solo había 70 productos, que son demasiado pocos para validar un modelo. El resultado es: 1401 productos descargados → 1373 únicos → 1037 en el Gold. Los 70 productos de la entrega 2 están incluidos. El proceso de descarga y limpieza está explicado en la entrega 2; aquí solo amplío las categorías. Igual que allí, cada archivo descargado anota su fecha en `data/raw/registro_descargas.csv` en el momento de la descarga, sin depender de la fecha de modificación del archivo.

Al ampliar descubrí que Mercadona no da siempre el precio de referencia por kilo: también lo da por litro, por 100 g, por unidad o por docena. Añadí una columna `precio_kg` para poder comparar. Los 7 productos del Gold con precio por unidad o docena no tienen precio por kilo y no entran en los cálculos por euro. Los 39 productos con precio por litro tampoco entran en esos cálculos, porque no tengo una conversión justificada de litros a kilos. Su precio sí lo uso como variable del modelo, tratando 1 litro como 1 kilo, que es una aproximación.

**Salida (lo que se quiere predecir):** `proteina_100g`.

**Entradas:** categoría, texto del nombre, precio unitario, precio por kilo, tamaño del envase, formato, estado (seco, cocinado…) y si es Hacendado. Los huecos de `precio_kg` se rellenan con la mediana dentro del pipeline, calculada solo con los datos de entrenamiento de cada fold.

**Lo que no entra, y por qué.** En el momento de uso, el producto sin datos en Open Food Facts solo tendrá lo que publica Mercadona (nombre, categoría, precio, tamaño y formato). Por eso no uso calorías, grasa ni hidratos como entrada: un producto sin datos en Open Food Facts tampoco los tiene, y además están muy ligados a la proteína. Usarlos sería una fuga de información: el modelo parecería bueno y luego no podría usarse en el caso real.

**Riesgos que controlo:**
- **Fuga por variantes casi iguales.** Productos como "Pipas girasol tostadas" y "Pipas girasol naturales" se parecen mucho. Si uno cae en entrenamiento y el otro en test, el modelo parece mejor de lo que es. Separo los datos por variante (las dos primeras palabras del nombre sin la marca: 580 variantes). Quedan 849 productos de entrenamiento y 187 de test.
- **Fuga al rellenar huecos (corregida).** En una primera versión calculaba la mediana de `precio_kg` con todo el Gold antes de separar entrenamiento y test, de modo que el entrenamiento recibía información del test. Ahora el relleno está dentro del pipeline y se ajusta solo con el entrenamiento de cada fold. Los resultados no cambiaron a dos decimales.
- **Elegir el modelo mirando el test.** El test se reserva, y los modelos se comparan con validación cruzada solo sobre el entrenamiento.
- **Sesgo de selección.** Entre los productos sin datos, el 55,8 % son Hacendado; entre el resto, el 80,6 %. El modelo aprende sobre todo con Hacendado y luego se aplicaría a un grupo con más marcas. Por eso muestro el error por marca.
- **Calidad del objetivo.** La proteína sale de Open Food Facts, que tiene errores. Detecté uno: "Queso curado mezcla con trufa Hacendado", con 0 g de proteína y 900 kcal, que el control de calorías no pilla. Lo excluyo del entrenamiento y lo dejo escrito. Los controles de rango detectaron 9 productos con valores imposibles (por ejemplo, 5232 kcal por 100 g); 8 de ellos habrían entrado al Gold y quedan fuera.

## 3. Baseline y selección de modelos

**Baseline:** la proteína media de la categoría del producto, calculada con los datos de entrenamiento. Lo elegí porque la categoría ya dice mucho (un yogur no tiene la proteína de un jamón) y porque es lo que haría una persona sin ML.

**Alternativas de ML:**
- **Ridge** (regresión lineal regularizada) sobre categoría, texto del nombre, precio y formato. Es sencilla y aguanta bien muchas columnas de texto con pocos datos.
- **Random Forest.** Es un modelo no lineal, para ver si hay interacciones que Ridge no capta.
- **Baseline + corrección Ridge.** La primera ronda mostró que Ridge no aprovechaba bien la categoría, así que probé que el modelo aprendiera solo la desviación respecto a la media de su categoría. Probé 5 valores del parámetro de regularización para Ridge y para esta variante.

No busqué algoritmos más complejos: con 1036 productos no tendrían datos suficientes.

## 4. Validación y evaluación

**Estrategia.** Separación 80/20 agrupada por variante. Las comparaciones entre modelos se hacen con validación cruzada agrupada de 5 pliegues solo sobre el entrenamiento. El test se usa para el resultado final.

**Métricas.** El **MAE** es el error medio en gramos de proteína por 100 g, y es el más fácil de explicar. También muestro RMSE (penaliza los errores grandes), R² y el porcentaje de productos con error menor de 2 g.

| Modelo | MAE validación cruzada | MAE test | RMSE test | R² test | % error < 2 g |
|---|---|---|---|---|---|
| Baseline (media por categoría) | 3,20 | 3,39 | 5,14 | 0,60 | 48,1 |
| Ridge | 3,24 | 3,41 | 4,91 | 0,64 | 42,3 |
| Random Forest | 3,64 | 3,67 | 5,76 | 0,50 | 48,1 |
| Baseline + corrección Ridge (α=3) | 2,96 | 3,36 | 4,96 | 0,63 | 42,8 |

**Lo que dicen los resultados, sin adornos.** La mejor variante baja el error en validación cruzada de 3,20 a 2,96 g, pero en test pasa de 3,39 a 3,36 g. La mejora media es de 0,03 g con un intervalo del 95 % de −0,18 a +0,24 g, que incluye el cero. El modelo se equivoca menos que el baseline solo en el 47,1 % de los productos. **Con estos datos, el ML no se distingue de la media por categoría.**

Ridge y Random Forest, probados solos, no mejoran el MAE del baseline. Ridge y la corrección sí mejoran algo el RMSE (4,91 y 4,96 frente a 5,14) y el R² (0,64 y 0,63 frente a 0,60), lo que sugiere que corrigen algún error grande. Pero aciertan por debajo de 2 g en menos productos (42 % frente a 48 %) y no lo he contrastado con bootstrap, así que no lo uso como argumento. Además, elegí la mejor de 10 variantes por validación cruzada, por lo que su 2,96 g es algo optimista.

**Sobre la limpieza del proceso.** El test se vio una primera vez al comparar los tres primeros modelos. La segunda ronda (la corrección Ridge) se eligió solo con validación cruzada, y el test se miró una vez para el elegido. Aun así, al decidir que me quedo con el baseline lo hago habiendo visto esos resultados, y no lo escondo. Por eso añadí las dos comprobaciones que siguen, con datos que no había visto.

**Efecto de las fugas.** Si mezclo las variantes entre entrenamiento y validación, el error sale 2,78 g en vez de 2,96 g. La diferencia es pequeña, pero confirma que sin el control los resultados serían algo optimistas.

### Un test nuevo

El test de la primera ronda ya lo había visto, así que no sirve para confirmar la comparación con honestidad. Descargué 10 categorías que no estaban en el Gold (platos listos, pizzas congeladas, rebozados, empanados, arroz y pasta congelados, salazones y ahumados, bífidus, postres de soja, sopas y turrones). Quité los productos que ya tenía y los que tenían una variante presente en el entrenamiento, y quedaron **101 productos con proteína informada**. Entrené con todos mis datos anteriores y medí en estos, sin cambiar nada del modelo.

Como esas categorías no existen en el entrenamiento y la media por categoría necesita conocer la categoría, asigné cada una a una categoría parecida **antes de ver los resultados** (por ejemplo, pizzas congeladas a pizzas). Es una limitación: algunas se parecen poco, y el test mide también si mi asignación fue buena.

| Modelo | MAE test nuevo (g) | RMSE (g) | R² |
|---|---|---|---|
| Media por categoría | 4,26 | 6,00 | 0,01 |
| Ridge | 4,21 | 5,32 | 0,22 |
| Random Forest | 3,36 | 4,56 | 0,43 |

- **Ridge frente a la media por categoría** son equivalentes (diferencia de −0,05 g, intervalo del 95 % de −0,56 a +0,47). Se confirma lo que veía en el primer test.
- **Random Forest** mejora en 0,9 g (intervalo de −1,42 a −0,37), pero casi toda esa mejora viene de los rebozados congelados (24 de los 101 productos), donde mi asignación a «hamburguesas» es dudosa. Sin esa categoría, la diferencia baja a unos 0,2 g. En la comprobación con etiquetas reales (más abajo) Random Forest es el peor. Lo tomo como una pista a investigar y no cambio de modelo después de ver este test.

### Comprobación con etiquetas reales de productos sin datos

Todo lo anterior mide el error en productos que sí tienen ficha nutricional. Los productos donde se usaría la estimación son justo los que no la tienen, así que miré a mano la tabla nutricional de una muestra de ellos en la web de Mercadona, el 22 de noviembre de 2026. Hice dos muestras:
- Una primera de 26 productos elegidos al azar entre los 269 sin datos (1 sin etiqueta, 1 instantáneo que miro aparte porque no sé si su etiqueta es del producto seco o preparado, y 13 de ellos en categorías fiables).
- Una segunda de 27 productos elegidos al azar solo entre los que sí recibirían estimación (categorías fiables), para tener más.

Con los 40 productos de categorías fiables:

| | Test del experimento (187 productos) | Etiquetas reales (40 productos) |
|---|---|---|
| Error medio, media por categoría | 2,08 g | 2,41 g (intervalo 95 %: 1,67 a 3,25) |
| Error medio, Ridge | | 2,49 g |
| Error menor de 2 g | | 60 % |
| Dentro de ±3,4 g | 82,6 % | 80 % (32 de 40; intervalo 95 %: 65 % a 90 %) |

- **El rango del 80 % se mantiene** (32 de 40), aunque con una muestra pequeña el intervalo es amplio.
- **El error medio es algo mayor que en test**, pero compatible con él. Lo honesto es decir que el error típico en productos sin ficha es de **2 a 2,5 g**, no de 2 g.
- Ridge vuelve a quedar igual que la media por categoría, así que sigo con la regla sencilla.
- Los 8 productos con error mayor de 3,4 g son sobre todo embutidos y platos especiales: longaniza de pavo (12 g), mini biscotes de trigo (9,2 g; la etiqueta marca 1,1 g, comprobado), cocas de dacsa congeladas (8,0 g), paté de hígado (5,8 g) y torrezno (4,9 g).
- Esta comprobación usa la regla de fiabilidad por categoría. La regla por categoría y estado (más abajo) solo cambia arroz, legumbres y pasta, y entre los 40 productos solo hay un macarrón, que sigue siendo fiable.

### Dónde se equivoca

Las cifras de frutos secos y de marca son de test; las de patatas fritas y snacks y las de mejores categorías son de validación cruzada con el modelo de la segunda ronda.
- **Frutos secos:** 9,6 g de error medio en test, con 23 productos (alrededor de un tercio del error total). Comparten categoría las pipas (25 a 31 g de proteína) con los arándanos deshidratados (0,9 g), y la media por categoría no puede acertar con ambas. El peor caso es un cacahuete en polvo desgrasado con 46,7 g que el modelo estima en 18,4 g.
- **Patatas fritas y snacks:** 9,1 g en validación cruzada, porque mezclan patatas (en torno a 6 g) con torreznos y cortezas (45 a 59 g).
- **Mejores categorías:** mermelada y miel (0,5 g), tomate (1,0), galletas (1,3), hamburguesas (1,3) y arroz (1,4).
- **Por marca (test):** el error del baseline es de 3,26 g en Hacendado (149 productos) y 3,91 g en otras marcas (38 productos). Con 38 productos la comparación es poco firme, pero apunta a que las otras marcas salen algo peor.

### Fiabilidad por categoría y estado

La estimación es la media de proteína del grupo del producto: su categoría y, en arroz, legumbres y pasta, también su estado (seco, cocinado o fresco). Un grupo es "fiable" si tiene al menos 8 productos y su error en validación cruzada es de 4 g o menos. Fijé el umbral de 4 g después de ver la tabla de errores por categoría, así que es una decisión mía y no un valor óptimo.

| | Solo por categoría | Por categoría y estado |
|---|---|---|
| Grupos fiables | 26 de 42 | 26 de 46 |
| Rango de error del 80 % | ±3,4 g | −3,2 g a +3,5 g |
| Test, grupos fiables | 115 productos, MAE 2,08 g | 115 productos, MAE 2,05 g |
| Test, cobertura del rango | 82,6 % | 80,9 % |
| Test, grupos no fiables | 72 productos, MAE 5,48 g | |
| Productos sin datos con estimación | 120 de 269 | 115 de 269 |

La precisión no cambia, pero añadir el estado evita mezclar productos que no se pueden comparar: en legumbres, el error de las secas es de 8,8 g y el de las cocinadas de 1,6 g, algo que la media por categoría (5,5 g) escondía. Se pierden 5 estimaciones: el arroz seco solo tiene 7 productos en el entrenamiento (menos del mínimo de 8) y los 4 instantáneos no llegan al mínimo ni tienen una base nutricional clara.

### Limitaciones importantes

- El test es pequeño (187 productos; solo 23 de frutos secos), así que las cifras por categoría son orientativas.
- El test nuevo depende de cómo asigné las categorías nuevas a categorías parecidas, y la comprobación con etiquetas es de 40 productos (intervalo del 80 %: de 65 % a 90 %). Son comprobaciones de orden de magnitud, no cifras exactas.
- Al separar por estado, algunos grupos quedan con pocos productos (arroz seco: 7; legumbres cocinadas: 7) y se quedan sin estimación por no llegar al mínimo de 8.
- El precio por litro se trata como precio por kilo solo como variable del modelo; no uso esa conversión en ningún cálculo por euro.
- Los valores que uso como verdad vienen de Open Food Facts, que puede tener errores que no he detectado.
- El rango del 80 % significa que 1 de cada 5 estimaciones queda fuera de él.
- Los datos son una foto de un momento: el catálogo de Mercadona cambia (en la entrega 2 ya lo comprobé).

## 5. Decisión y uso en el producto

**Solución elegida:** la media de proteína de la categoría y el estado del producto, con un rango de error, y solo en los grupos fiables. Como la corrección con ML no mejora de forma demostrable (ni en test, ni en el test nuevo, ni con etiquetas reales), prefiero la regla simple: se explica en una frase, no depende del texto del nombre y es poco probable que falle con productos nuevos.

**Qué mejora ofrece.** De los 269 productos sin datos nutricionales, **115 (un 43 %) pasan a tener una estimación**, con un error típico de **2 a 2,5 g**. Los otros 154 siguen sin estimación y el producto lo dirá claramente. Por grupos, tienen estimación 75 de 176 productos base, 38 de 74 snacks y dulces, 2 de 18 platos preparados y ninguno de la única salsa.

**¿Es suficiente para el MVP?** Sí como apoyo, no como sustituto del dato real. Cubre menos de la mitad de los productos sin datos y no vale para los más variables (frutos secos, snacks, platos preparados, quesos curados, embutido curado). Para esos hará falta una fuente mejor o introducir el dato de la etiqueta a mano, como hice con la lenteja en la entrega 2.

**Qué consumirá el frontal.** Una estimación por categoría y estado, solo si el grupo tiene al menos 8 ejemplos compatibles; si no, el campo queda vacío. El archivo es `data/gold/estimaciones_proteina_por_estado.csv` (no se publica).

| Campo | Significado |
|---|---|
| `proteina_estimada` | Gramos de proteína por 100 g (media de su categoría y estado). Vacío si el grupo no es fiable. |
| `rango_bajo`, `rango_alto` | Intervalo en el que debería estar la proteína real con una probabilidad aproximada del 80 %. |
| `fiable` | `True` si hay estimación; `False` si no se estima. |

**Cautelas:**
- Mostrar siempre que es una estimación, nunca como el dato de la etiqueta.
- No mezclar estimaciones con valores reales en un ranking sin marcarlas.
- Para la proteína por euro, calcularla con el rango completo, no solo con el valor central.
- Dejar fuera del ranking por euro los productos instantáneos, porque no sé si su información nutricional es del producto seco o preparado, y los productos con precio por litro, porque no tengo una conversión justificada de litros a kilos.
- No usar estas estimaciones en decisiones delicadas (alergias, dietas médicas).

**Siguientes pasos:** conseguir más productos de arroz y legumbres para que sus grupos lleguen al mínimo de 8 ejemplos; buscar una fuente adicional para frutos secos y snacks; estimar también los 59 productos con datos incoherentes; y estimar también calorías.

## Qué hay en el repositorio

| Elemento | Ubicación |
|---|---|
| Este documento | `docs/entregas/03_machine_learning.md` |
| Datos ampliados (Bronze → Gold) y preparación del test nuevo | `pipeline_ampliado.ipynb` |
| Experimento (baseline, modelos, validación, errores, test nuevo, etiquetas, estimación por estado) | `experimento_ml.ipynb` |
| Métricas de los modelos y validación cruzada | `resultados/metricas_ml.csv`, `resultados/validacion_cruzada.csv`, `resultados/test_final.csv`, `resultados/metricas_test_nuevo.csv` |
| Fiabilidad y cobertura de las estimaciones (solo categoría y por categoría y estado) | `resultados/fiabilidad_por_categoria.csv`, `resultados/cobertura_estimaciones.csv`, `resultados/fiabilidad_por_categoria_y_estado.csv`, `resultados/cobertura_estimaciones_por_estado.csv` |
| Muestras de etiquetas miradas a mano | `data/manual/muestra_etiquetas.csv`, `data/manual/muestra_etiquetas_2.csv` |
| Muestra inventada del cruce | `data/sample/cruce_ejemplo_inventado.csv` |
| Continuidad | Parte del Gold de la entrega 2 y produce `proteina_estimada`, `rango_bajo`, `rango_alto` y `fiable` para el frontal |

Los datos descargados, el registro de descargas y el Gold ampliado no se publican, igual que en la entrega 2. Las pocas cifras de etiquetas reales que se ven en el notebook (la proteína de 51 productos, que miré a mano) son lo mínimo necesario para poder comprobar la validación, y las retiraría si Mercadona me lo pidiera. Para reproducirlo hay que ejecutar primero `pipeline_ampliado.ipynb` y después `experimento_ml.ipynb`.