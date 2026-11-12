# Entrega 3 - Machine Learning

Esta entrega parte del Gold de la entrega 2 (`docs/entregas/02_datos_y_preparacion.md`) y lo amplía para poder entrenar y validar un modelo. El código está en `pipeline_ampliado.ipynb` (datos) y `experimento_ml.ipynb` (experimento). Las métricas están en `resultados/`.

## 1. Qué problema resuelve el modelo

Mi producto ayuda a hacer la compra en Mercadona según un objetivo nutricional, por ejemplo conseguir la máxima proteína por euro. Para recomendar un producto necesito saber sus nutrientes, y los saco de Open Food Facts. El problema es que no todos los productos de Mercadona están allí, o no tienen los datos completos o coherentes. De los 1373 productos distintos que descargué, 269 (un 20 %) no tienen datos nutricionales utilizables en Open Food Facts, y sin datos el producto no puede recomendarlos. Otros 59 tienen datos incoherentes y también los descarto; por ahora solo estimo los 269 sin datos.

El modelo estima los gramos de proteína por 100 g de uno de esos productos usando solo lo que Mercadona sí publica: nombre, categoría, precio y formato. Es un problema de **regresión**. Lo usará la persona que compra, y la decisión que le permite tomar es si incluir ese producto en su lista aunque su proteína sea una estimación y no el dato de la etiqueta.

Elegí la proteína porque es la base de mi métrica principal, la proteína por euro. Otros nutrientes quedan para más adelante.

## 2. Datos de entrada y salida

**Datos.** Uso el Gold ampliado, que sale del mismo pipeline Bronze → Silver → Gold de la entrega 2, con los mismos controles de calidad, pero con 43 subcategorías de alimentos envasados en lugar de 3 (arroz, legumbres y pasta). Con aquellas 3 solo había 70 productos, que son demasiado pocos para validar un modelo. El resultado es: 1401 productos descargados → 1373 únicos → 1037 en el Gold. Los 70 productos de la entrega 2 están incluidos. El proceso de descarga y limpieza está explicado en la entrega 2; aquí solo amplío las categorías.

Al ampliar descubrí que Mercadona no da siempre el precio de referencia por kilo: también lo da por litro, por 100 g, por unidad o por docena. Añadí una columna `precio_kg` para poder comparar. Los productos con precio por unidad o docena (7 en el Gold) no tienen precio por kilo y no entran en los cálculos por euro.

**Salida (lo que se quiere predecir):** `proteina_100g`.

**Entradas:** categoría, texto del nombre, precio unitario, precio por kilo, tamaño del envase, formato, estado (seco, cocinado…) y si es Hacendado.

**Lo que no entra, y por qué.** En el momento de uso, el producto sin datos en Open Food Facts solo tendrá lo que publica Mercadona (nombre, categoría, precio, tamaño y formato). Por eso no uso calorías, grasa ni hidratos como entrada: un producto sin datos en Open Food Facts tampoco los tiene, y además están muy ligados a la proteína. Usarlos sería una fuga de información: el modelo parecería bueno y luego no podría usarse en el caso real.

**Riesgos que controlo:**
- **Fuga por variantes casi iguales.** Productos como "Pipas girasol tostadas" y "Pipas girasol naturales" se parecen mucho. Si uno cae en entrenamiento y el otro en test, el modelo parece mejor de lo que es. Separo los datos por variante (las dos primeras palabras del nombre sin la marca: 580 variantes). Quedan 849 productos de entrenamiento y 187 de test.
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

**Sobre la limpieza del proceso.** El test se vio una primera vez al comparar los tres primeros modelos. La segunda ronda (la corrección Ridge) se eligió solo con validación cruzada, y el test se miró una vez para el elegido. Aun así, al decidir que me quedo con el baseline lo hago habiendo visto esos resultados, y no lo escondo.

**Efecto de las fugas.** Si mezclo las variantes entre entrenamiento y validación, el error sale 2,78 g en vez de 2,96 g. La diferencia es pequeña, pero confirma que sin el control los resultados serían algo optimistas.

**Dónde se equivoca.** Las cifras de frutos secos y de marca son de test; las de patatas fritas y snacks y las de mejores categorías son de validación cruzada con el modelo de la segunda ronda.
- **Frutos secos:** 9,6 g de error medio en test, con 23 productos (alrededor de un tercio del error total). Comparten categoría las pipas (25 a 31 g de proteína) con los arándanos deshidratados (0,9 g), y la media por categoría no puede acertar con ambas. El peor caso es un cacahuete en polvo desgrasado con 46,7 g que el modelo estima en 18,4 g.
- **Patatas fritas y snacks:** 9,1 g en validación cruzada, porque mezclan patatas (en torno a 6 g) con torreznos y cortezas (45 a 59 g).
- **Mejores categorías:** mermelada y miel (0,5 g), tomate (1,0), galletas (1,3), hamburguesas (1,3) y arroz (1,4).
- **Por marca (test):** el error del baseline es de 3,26 g en Hacendado (149 productos) y 3,91 g en otras marcas (38 productos). Con 38 productos la comparación es poco firme, pero apunta a que las otras marcas salen algo peor.

**Fiabilidad por categoría.** Una categoría es "fiable" si tiene al menos 8 productos y su error en validación cruzada es de 4 g o menos. Son 26 de las 42 categorías que tienen productos en el entrenamiento (la categoría Embutido no tiene ninguno). En test, en las categorías fiables (115 productos) el MAE es de **2,08 g**; en las no fiables (72 productos), de **5,48 g**. El rango de error del 80 % en las categorías fiables es de **±3,4 g**, y en test lo cumple el 82,6 % de los productos. Fijé el umbral de 4 g después de ver la tabla de errores por categoría, así que es una decisión mía y no un valor óptimo. Lo que sí es limpio es que la comprobación se hace en test.

**Limitaciones importantes.**
- El test es pequeño (187 productos; solo 23 de frutos secos), así que las cifras por categoría son orientativas.
- En arroz, legumbres y pasta, la media por categoría mezcla productos secos y cocinados, aunque ya tengo la variable `estado`. Es una mejora pendiente.
- El precio por litro se trata como precio por kilo (1 litro ≈ 1 kilo), que es una aproximación.
- Los valores que uso como verdad vienen de Open Food Facts, que puede tener errores que no he detectado.
- El rango del 80 % significa que 1 de cada 5 estimaciones queda fuera de él.
- Los datos son una foto de un momento: el catálogo de Mercadona cambia (en la entrega 2 ya lo comprobé).

## 5. Decisión y uso en el producto

**Solución elegida:** la media de proteína de la categoría, con un rango de error, y solo en las categorías fiables. Como la corrección con ML no mejora de forma demostrable, prefiero la regla simple: se explica en una frase, no depende del texto del nombre y es poco probable que falle con productos nuevos.

**Qué mejora ofrece.** De los 269 productos sin datos nutricionales, **120 (un 45 %) pasan a tener una estimación**, con un error típico de unos 2 g. Los otros 149 siguen sin estimación y el producto lo dirá claramente. Por grupos, tienen estimación 80 de 176 productos base, 38 de 74 snacks y dulces, 2 de 18 platos preparados y ninguno de la única salsa.

**¿Es suficiente para el MVP?** Sí como apoyo, no como sustituto del dato real. Cubre menos de la mitad de los productos sin datos y no vale para los más variables (frutos secos, snacks, platos preparados, quesos curados, embutido curado). Para esos hará falta una fuente mejor o introducir el dato de la etiqueta a mano, como hice con la lenteja en la entrega 2.

**Qué consumirá el frontal:**

| Campo | Significado |
|---|---|
| `proteina_estimada` | Gramos de proteína por 100 g (media de su categoría). Vacío si la categoría no es fiable. |
| `rango_bajo`, `rango_alto` | Intervalo en el que debería estar la proteína real con una probabilidad aproximada del 80 %. |
| `fiable` | `True` si hay estimación; `False` si no se estima. |

**Cautelas:**
- Mostrar siempre que es una estimación, nunca como el dato de la etiqueta.
- No mezclar estimaciones con valores reales en un ranking sin marcarlas.
- Para la proteína por euro, calcularla con el rango completo, no solo con el valor central.
- No usar estas estimaciones en decisiones delicadas (alergias, dietas médicas).

**Siguientes pasos:** usar `estado` en arroz, legumbres y pasta, y comprobar si ayuda al error de esas categorías; buscar una fuente adicional para frutos secos y snacks; estimar también los 59 productos con datos incoherentes; y estimar también calorías.

## Qué hay en el repositorio

| Elemento | Ubicación |
|---|---|
| Este documento | `docs/entregas/03_machine_learning.md` |
| Datos ampliados (Bronze → Gold) | `pipeline_ampliado.ipynb` |
| Experimento (baseline, modelos, validación, errores) | `experimento_ml.ipynb` |
| Métricas de los modelos y validación cruzada | `resultados/metricas_ml.csv`, `resultados/validacion_cruzada.csv`, `resultados/test_final.csv` |
| Fiabilidad por categoría y cobertura de las estimaciones | `resultados/fiabilidad_por_categoria.csv`, `resultados/cobertura_estimaciones.csv` |
| Continuidad | Parte del Gold de la entrega 2 y produce `proteina_estimada`, `rango_bajo`, `rango_alto` y `fiable` para el frontal |

Los datos descargados y el Gold ampliado no se publican, igual que en la entrega 2. Para reproducirlo hay que ejecutar primero `pipeline_ampliado.ipynb` y después `experimento_ml.ipynb`.