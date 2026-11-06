# Entrega 1 - Propuesta inicial de ideas de producto

## Idea 1 - Detección de patrones de abandono en el seguimiento nutricional

### Problema que resuelve

Muchas personas empiezan a registrar su alimentación en una app con buena intención, pero con el tiempo dejan de hacerlo de forma constante o empiezan a registrar de forma sesgada (saltarse los fines de semana, minimizar cantidades, dejar huecos). Un nutricionista que sigue a varios pacientes a distancia no tiene forma sencilla de detectar este patrón a tiempo: solo puede revisar el registro manualmente, comida a comida. Cuando el problema se detecta, normalmente ya se ha perdido bastante adherencia al plan nutricional establecido.

**Solución que propongo:** un panel que analiza el histórico de registro de cada cliente y calcula un "score de riesgo de abandono" a partir de señales objetivas, como la frecuencia de registro (días seguidos sin registrar), la tendencia (si cada vez registra menos o más tarde), cambios de patrón (por ejemplo, dejar de registrar los fines de semana) o variabilidad sospechosa (cantidades redondas o genéricas, señal de que ya no se pesa la comida). Con esas señales, el sistema generaría alertas del tipo "paciente X lleva 4 fines de semana seguidos sin registrar → riesgo alto de abandono", para que el nutricionista pueda intervenir antes de perder al paciente y que este haya dejado de lado su objetivo, en lugar de detectarlo semanas después revisando manualmente. El resultado sería un dashboard con la lista de pacientes ordenados por riesgo, un histórico visual de constancia por cliente, y una explicación del motivo de cada nivel de riesgo, no solo un número.

**Datos que necesitaría:** por cada usuario, un histórico de registros de comida con fecha y hora exacta de cuándo se registró (no solo qué comió), a lo largo de varias semanas o meses para poder ver tendencia, e idealmente el detalle de cada entrada (alimento, cantidad, si fue manual o escaneado) para poder detectar registros sospechosos. La limitación que veo en esta idea es que los datos propios de los que dispongo son de mi misma, mientras que el caso de uso completo requeriría datos de varias personas distintas durante un periodo largo, algo a lo que no tengo acceso. Tendría que adaptar el enfoque a una versión con mis propios datos en distintos periodos.

- **¿Qué problema real estoy intentando resolver?** Que el abandono o la desviación en el registro nutricional pasa desapercibido hasta que ya es tarde para intervenir.
- **¿Quién tiene ese problema?** Nutricionistas que gestionan varios pacientes a distancia, y los propios usuarios que pierden constancia sin ser conscientes del patrón.
- **¿Qué decisión o proceso podría mejorar?** La decisión de cuándo y con quién debe intervenir el profesional, y la autoconciencia del usuario sobre su propia constancia.
- **¿Por qué una solución basada en datos podría aportar valor?** Porque el patrón de abandono no es evidente revisando registros a mano, pero sí es detectable analizando series temporales: frecuencia de registro, huecos, cambios de comportamiento a lo largo del tiempo.
- **¿Es razonable pensar que existen datos para trabajar esta idea?** Parcialmente. Dispongo de exportaciones propias de una app de nutrición con histórico de registros por fecha, lo que me permitiría prototipar la lógica del análisis, pero el caso de uso completo (varios clientes de un coach) requeriría datos de varias personas distintas, a los que no tengo acceso directo por ahora. Es la idea con mayor incertidumbre respecto a los datos de las tres que propongo.

### Motivación para realizar el proyecto

La constancia es uno de los mayores retos en el seguimiento nutricional real, no solo un problema teórico, y me parece un caso de uso con sentido de negocio. Me permitiría aplicar contenidos del máster como series temporales y detección de anomalías, y parto de una fuente de datos cercana desde la que empezar.

### A quién impacta

Nutricionistas que gestionan pacientes a distancia y necesitan priorizar su tiempo de seguimiento; y usuarios de apps de nutrición que quieren tomar conciencia de su propia constancia antes de abandonar por completo.

### Por qué tiene valor

- Detección temprana de riesgo de abandono.
- Ahorro de tiempo al profesional, que ya no necesita revisar el registro manualmente.
- Mejor toma de decisiones sobre cuándo y cómo intervenir con cada cliente.

---

## Idea 2 - Generador de lista de la compra ajustada a un objetivo nutricional

### Problema que resuelve

La mayoría de la gente no sabe realmente qué está comiendo: si un producto es más o menos sano, si tiene mucha o poca grasa, o si su cesta de la compra en conjunto se ajusta a un objetivo nutricional concreto. Esto es especialmente relevante para quien sigue una dieta estricta -por ejemplo, alta en proteína o con déficit calórico, porque hacer la compra "a ojo" no garantiza que lo que se compre realmente cumpla ese objetivo, y calcularlo a mano producto a producto es tedioso y poco práctico en el día a día. El resultado es que la compra se hace por costumbre o intuición, no en base a lo que realmente se necesita. Para mí no es un ejercicio técnico aislado: afecta directamente a si una persona logra o no seguir su objetivo nutricional de forma sostenible.

**Solución que propongo:** una app/web en la que el usuario indica su objetivo nutricional (por ejemplo, dieta alta en proteína, déficit calórico, o simplemente comer de forma más sana) y, opcionalmente, el periodo para el que quiere hacer la compra (una semana, dos semanas...). Con esos datos, el sistema calcula sus necesidades diarias aproximadas (kcal y macronutrientes: proteína, grasa, carbohidrato) y a partir de ahí permite ir construyendo la compra añadiendo productos reales del catálogo de Mercadona, mostrando en todo momento cómo de bien se ajusta lo añadido al objetivo marcado (por ejemplo, avisando si falta proteína o si ya se ha superado el límite de grasa previsto).

- **¿Qué problema real estoy intentando resolver?** Que las personas no saben si lo que compran y comen realmente se ajusta a su objetivo nutricional (dieta estricta en proteína, déficit calórico, comer más sano), ni cómo traducir ese objetivo en una compra concreta.
- **¿Quién tiene ese problema?** Personas que siguen un objetivo nutricional concreto y compran habitualmente en Mercadona: deportistas, personas en seguimiento nutricional, o cualquiera que quiera comer de forma más controlada sin complicarse con cálculos manuales.
- **¿Qué decisión o proceso podría mejorar?** La decisión de qué productos incluir en la compra semanal o quincenal para que se ajuste al objetivo nutricional marcado.
- **¿Por qué una solución basada en datos podría aportar valor?** Porque convierte un cálculo manual y propenso a error (necesidades nutricionales según el objetivo + composición nutricional de cada producto) en una recomendación automática y visual, mostrando en tiempo real cómo de bien se ajusta la compra que se está montando.
- **¿Es razonable pensar que existen datos para trabajar esta idea?** Sí. Mercadona publica la información nutricional completa de sus productos en su propia web, lo que me da una fuente de datos acotada, estable y accesible, sin depender de un dataset ya empaquetado de terceros.

### Motivación para realizar el proyecto

Acotar el proyecto a un único supermercado y a un único perfil de usuario lo hace viable de verdad: el dataset es controlado, no depende de fuentes externas impredecibles, y la lógica de cálculo es clara y manejable como MVP. Convierte un problema muy habitual y poco resuelto hoy en día -saber si se está comiendo según el objetivo marcado y traducirlo en una compra real- en un caso de uso concreto de recomendación y personalización con datos accesibles, permitiéndome aplicar contenidos de recomendación y optimización propios del máster sobre un problema cotidiano y con sentido. Además, a nivel personal me gustaría tener esta herramienta para mí misma: me resultaría útil en el día a día para organizar mi propia compra según mis objetivos nutricionales, así que es un proyecto que usaría de verdad, no solo un ejercicio académico.

### A quién impacta

Personas que siguen un objetivo nutricional concreto y compran habitualmente en Mercadona, sin tiempo ni conocimientos para calcular manualmente si su compra se ajusta a ese objetivo.

### Por qué tiene valor

- Traduce un objetivo nutricional (dieta estricta en proteína, déficit calórico...) en decisiones de compra concretas.
- Da visibilidad sobre lo que realmente se está comiendo, algo que hoy la mayoría de la gente desconoce.
- Ahorra tiempo y esfuerzo de planificación de la compra, evitando el cálculo manual producto a producto.

**Próximos pasos posibles:** una vez validado el MVP para una única persona, el proyecto podría evolucionar para dar soporte a una unidad familiar completa, permitiendo indicar el número de miembros, si hay niños, y si cada uno sigue un objetivo distinto dentro de la misma compra.

---

## Idea 3 - Correlación entre alimentación y rendimiento deportivo

### Problema que resuelve

No es evidente, sin analizarlo, qué combinación de alimentación (qué se come, cuándo, cuánta proteína o carbohidrato) se asocia a mejores o peores entrenamientos: más peso movido, más series completadas, mejor progresión. Por eso, las decisiones sobre qué comer antes o después de entrenar se suelen tomar por intuición y no por evidencia propia. Para mí no es un ejercicio técnico aislado, cruzar ambas fuentes de datos personales me permitiría descubrir patrones que ninguna app genérica de nutrición o entrenamiento ofrece por separado.

- **¿Qué problema real estoy intentando resolver?** Que no es evidente qué patrones de alimentación se asocian a mejor o peor rendimiento en el entrenamiento, por lo que las decisiones al respecto se toman por intuición.
- **¿Quién tiene ese problema?** En el MVP, yo misma como caso de uso demostrable; de forma más amplia, cualquier persona que entrene con regularidad y quiera decisiones basadas en evidencia propia en vez de intuición.
- **¿Qué decisión o proceso podría mejorar?** La planificación de comidas antes y después de entrenar, ajustándola a lo que realmente ha funcionado mejor según mi propio historial.
- **¿Por qué una solución basada en datos podría aportar valor?** Porque genera conocimiento personalizado que ninguna app genérica ofrece: cruza dos fuentes de datos reales (nutrición y entrenamiento) para encontrar correlaciones específicas de la persona, no promedios genéricos de la población.
- **¿Es razonable pensar que existen datos para trabajar esta idea?** Sí. La app Hevy me permite exportar mi histórico de entrenamientos (ejercicios, series, pesos, fechas), aunque al ser datos de una sola persona el análisis sería más exploratorio que estadísticamente robusto.

### Motivación para realizar el proyecto

Combina dos fuentes de datos propias y reales (mi registro nutricional y mi registro de entrenamientos en Hevy), lo que me da acceso directo a datos verificables sin depender de terceros. Es un proyecto acotado desde mi propio caso de uso personal, y me permitiría aplicar análisis de correlación y modelos predictivos sobre datos reales, un enfoque muy alineado con los contenidos del máster.

### A quién impacta

En el MVP, a mí misma como caso demostrable; la solución sería generalizable a cualquier persona que entrene con regularidad y quiera tomar decisiones nutricionales basadas en datos en vez de intuición.

### Por qué tiene valor

- Genera conocimiento personalizado que ninguna app genérica proporciona.
- Mejora la planificación de comidas pre y post entreno.
- Sienta una base metodológica ampliable a otras personas más adelante, aunque en esta primera fase el análisis sería principalmente exploratorio dado el tamaño de muestra limitado a una persona.

---

## Ideas más prometedoras para seguir trabajando

De las tres, creo que la **Idea 2** es la que tiene mayor viabilidad inmediata y también la de mayor potencial: el catálogo nutricional de Mercadona es una fuente de datos pública, estable y acotada, y el problema que resuelve (saber si estoy comiendo según mi objetivo y traducirlo en una compra real) es concreto, fácil de defender, y además es algo que a mí misma me gustaría usar en mi día a día. La **Idea 3** es la segunda opción que más me interesa por el valor analítico que aporta cruzar dos fuentes de datos personales reales, aunque su alcance está limitado por tratarse de datos de una sola persona. La **Idea 1** es la que tiene más potencial de impacto a nivel profesional (para nutricionistas), pero también la que presenta mayor incertidumbre respecto a los datos, al depender de acceder a registros de varios usuarios distintos en lugar de una única fuente propia; por eso es la que necesitaría más trabajo de validación antes de confirmarse como idea definitiva.com
