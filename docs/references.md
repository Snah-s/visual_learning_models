# Metodología de Investigación: Visualización del Espacio Latente en ANNs

Esta metodología describe el proceso sistemático para extraer, reducir y proyectar en un plano bidimensional (2D) las activaciones de las capas ocultas y los pesos de una red neuronal, con el fin de evaluar y mejorar su aprendizaje a través de las épocas y capas.

```
[Flujo Metodológico General]
Datasets (MNIST/SVHN/CIFAR-10) ──> Entrenamiento de ANNs ──> Extracción de Activaciones ──> Reducción t-SNE ──> Scatterplot Interactivos (Mantra Visual)

```

---

## Fase 1: Selección y Preparación de Datos (Datasets)

El protocolo experimental requiere evaluar la generalización del modelo en conjuntos de datos de clasificación de imágenes con diferentes niveles de complejidad estructural:

1. 
**MNIST:** 50K imágenes de entrenamiento, 10K de validación y 10K de prueba ($28\times28$ píxeles, escala de grises de dígitos manuscritos). Es el dataset de control por su alta separabilidad lineal inicial.


2. 
**SVHN (Street View House Numbers):** 63.2K imágenes de entrenamiento, 10K de validación y 26K de prueba ($32\times32$ píxeles a color, dígitos de números de casas). Presenta desafíos de iluminación y contraste de fondo.


3. 
**CIFAR-10:** 30K imágenes de entrenamiento, 10K de validación y 10K de prueba ($32\times32$ píxeles a color de fotografías en 10 categorías de objetos).



---

## Fase 2: Arquitectura y Entrenamiento de Modelos (ANNs)

Se deben implementar y entrenar dos tipos de arquitecturas realistas para tareas de clasificación de imágenes:

### 1. Perceptrón Multicapa (MLP)

* 
**Capa de Entrada:** 3072 neuronas (784 para MNIST).


* 
**Capas Ocultas:** 4 capas ocultas de tipo lineal rectificada (ReLU), con 1000 neuronas cada una.


* 
**Regularización (Dropout):** Aplicado progresivamente desde la primera capa oculta, escalando de 0.2 a 0.5 con incrementos de 0.1 por capa.


* 
**Capa de Salida:** Capa Softmax de 10 neuronas.



### 2. Red Neuronal Convolucional (CNN)

* 
**Bloque Convolucional 1:** Entrada de $32\times32\times3$ -> 2 capas convolucionales consecutivas con 32 filtros de $3\times3$ -> Max-pooling de $2\times2$ con Dropout de 0.25.


* 
**Bloque Convolucional 2:** 2 capas convolucionales consecutivas con 64 filtros de $3\times3$ -> Max-pooling de $2\times2$ con Dropout de 0.25.


* 
**Bloque Completamente Conectado (Dense):** Una capa densa de 4096 neuronas (Dropout 0.5) -> Una capa densa de 512 neuronas. Todas las capas usan activación ReLU.


* 
**Capa de Salida:** Capa Softmax de 10 neuronas.



### 3. Protocolo de Entrenamiento

* 
**Algoritmo:** Descenso de Gradiente Estocástico (SGD) basado en *momentum*.


* 
**Configuración de hiperparámetros:** * **MLP:** Tamaño de lote (*batch size*) de 16, tasa de aprendizaje (*learning rate*) de 0.01, coeficiente de momentum de 0.9 y decaimiento (*learning decay*) de $10^{-9}$.


* 
**CNN:** Tamaño de lote de 32, tasa de aprendizaje de 0.01, coeficiente de momentum de 0.9 y decaimiento de $10^{-6}$.




* 
**Estrategia:** Inicializar los pesos mediante una distribución uniforme basada en las dimensiones de las capas adyacentes (Inicialización de Xavier/Glorot). Seleccionar hiperparámetros mediante validación cruzada y entrenar el modelo final integrando los datos de entrenamiento y validación.



---

## Fase 3: Extracción de Activaciones y Espacio Latente

Para evitar la saturación visual (*clutter*) en las proyecciones finales, no se deben usar todos los datos de prueba disponibles:

1. 
**Muestreo:** Extraer de forma aleatoria un subconjunto estrictamente fijo de **2000 observaciones** del conjunto de prueba para cada dataset.


2. 
**Puntos de Control (Épocas):** Guardar las activaciones en tres etapas críticas del ciclo de vida de la red: *Antes de entrenar* (Época 0), *Durante el entrenamiento* (intermedio) y *Al finalizar el entrenamiento*.


3. 
**Puntos de Control (Estructura/Capas):** Extraer el vector de salida de cada una de las capas ocultas por separado (para las CNNs, extraer específicamente de las capas totalmente conectadas/densas).



---

## Fase 4: Reducción de Dimensionalidad (Proyección 2D)

Para transformar los vectores de alta dimensión (ej. las 1000 activaciones por neurona de una capa oculta) a coordenadas manejables de $x, y$:

1. 
**Algoritmo Base:** Utilizar una implementación optimizada (rápida/aproximada) de **t-SNE** (*t-distributed Stochastic Neighbor Embedding*) con sus parámetros por defecto.


2. 
**Justificación Teórica:** Se selecciona t-SNE por su robustez matemática para preservar vecindades locales de alta dimensión y revelar estructuras de agrupamiento (*clusters*) nítidas en comparación con técnicas lineales.


3. 
**Generación del Gráfico:** Renderizar las coordenadas calculadas en un Gráfico de Dispersión (*Scatterplot*).



---

## Fase 5: Codificación Visual, Control de Calidad e Interacción

El entorno visual final debe configurarse bajo reglas estrictas para facilitar el análisis predictivo y deductivo del usuario:

### 1. Canales Visuales y Glifos

* 
**Puntos de Clase:** Mapear cada una de las 10 clases reales (*true class labels*) a un color categórico diferenciable dentro de una paleta fija (los puntos representan las muestras de imágenes).


* 
**Manejo de Errores (Misclassifications):** Aquellas observaciones que la red clasificó de forma errónea no se dibujan como círculos; se deben reemplazar por **glifos triangulares**. Estos triángulos se colorean con base en la etiqueta de la clase *incorrecta* predicha por el modelo, permitiendo identificar rápidamente errores sistemáticos.



### 2. Métrica de Calidad de la Proyección

Para evaluar cuantitativamente si la proyección 2D conserva la fidelidad de la alta dimensión, se debe calcular el **Neighborhood Hit (NH)**:

* Para cada punto proyectado $a_p$, evaluar sus $k$-vecinos más cercanos (fijar $k=6$).


* Calcular el ratio de vecinos que comparten su misma clase real.


* El NH global de la visualización será el promedio aritmético del ratio de todos los puntos. Un aumento en el porcentaje de NH debe correlacionarse directamente con el aumento de la precisión (*Accuracy*) de la red.



---

## Fase 6: Análisis Analítico Visual (Tareas de Exploración)

El agente o evaluador debe utilizar la interfaz gráfica interactiva para responder a las dos preguntas fundamentales del proyecto de visualización:

### T1: Exploración Inter-Épocas (Evolución Temporal)

* 
**Acción:** Comparar el scatterplot de la Época 0 (Antes del entrenamiento) con la Época Final.


* **Insight Esperado:** Verificar la hipótesis de la formación de características avanzadas. Al inicio, la red aleatoria posee cierta separación geométrica nativa debido a la arquitectura física (NH aceptable) , pero tras el entrenamiento, los clusters de colores deben compactarse de forma drástica, aislando los elementos difíciles.



### T2: Exploración Inter-Capas (Evolución Estructural)

* 
**Acción:** Comparar las proyecciones de las activaciones de la Capa Oculta 1 contra la Capa Oculta 4 (Capa final).


* 
**Insight Esperado:** Constatar que las capas tempranas muestran una separación de clases difusa e incompleta , mientras que las capas tardías procesan abstracciones de alto nivel que resuelven los límites de decisión, incrementando notablemente el Neighborhood Hit visual.



### Análisis de Casos Anómalos (Outliers y Refinamiento)

* 
**Detección Visual:** Inspeccionar mediante técnicas de selección interactiva (*brushing*) los puntos aislados dentro de clusters ajenos.


* 
**Diagnóstico de Datos:** Por ejemplo, si en el dataset SVHN un color se divide sistemáticamente en dos sub-clusters independientes, inspeccionar las imágenes de origen para deducir variables físicas ocultas (ej. dígitos oscuros sobre fondos claros vs. dígitos claros sobre fondos oscuros).


* 
**Bucle de Mejora:** Utilizar los hallazgos visuales para diseñar un sub-paso de preprocesamiento matemático en los datos (ej. aplicar filtros de gradiente de bordes como Sobel) , reentrenar bajo el mismo protocolo y verificar visualmente la disolución de los sub-clusters anómalos y el incremento del rendimiento del modelo.
