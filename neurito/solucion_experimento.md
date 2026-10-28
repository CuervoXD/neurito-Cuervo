# Solución: Experimentos con Neurito

Los resultados salen de ejecutar el código de Experimentos.txt sin la parte de pygame, con la misma semilla (42), 3000 ejemplos de entrenamiento y 1000 de prueba.

Resultado base (red 3-6-3, 1200 épocas, tasa de aprendizaje 0.1): la pérdida baja de 1.074 a 0.122 y la precisión de prueba llega a 99.4 %.

---

## 1. Experimentos

### 1.1 Dos necesidades fijas y una variable: ¿cuándo cambia la acción?

Variando la energía, con hambre = 0.2 y aburrimiento = 0.2:

| Energía | Cansancio (1 − energía) | COMER | DORMIR | JUGAR | Acción |
|---|---|---|---|---|---|
| 0.0 | 1.0 | 0 % | 100 % | 0 % | DORMIR |
| 0.4 | 0.6 | 0.1 % | 99.9 % | 0 % | DORMIR |
| 0.6 | 0.4 | 2.3 % | 95.7 % | 2.0 % | DORMIR |
| 0.7 | 0.3 | 12.1 % | 77.0 % | 10.9 % | DORMIR |
| 0.8 | 0.2 | 34.7 % | 33.6 % | 31.7 % | empate |
| 0.9 | 0.1 | 48.2 % | 7.1 % | 44.7 % | COMER |
| 1.0 | 0.0 | 50.9 % | 1.2 % | 47.9 % | COMER |

La acción pasa de DORMIR a COMER cuando el cansancio deja de ser la necesidad mayor. Aquí ocurre con energía cercana a 0.8, donde cansancio, hambre y aburrimiento valen 0.2. Esto coincide con la política original.

Variando el hambre (energía = 0.9, aburrimiento = 0.2): con hambre 0.1 gana JUGAR (76.6 %) y con 0.3 gana COMER (86.4 %). El cambio ocurre cuando el hambre supera al aburrimiento, cerca de 0.2.

Variando el aburrimiento (hambre = 0.2, energía = 0.9): con 0.1 gana COMER (77.9 %) y con 0.3 gana JUGAR (84.9 %). El cambio ocurre cuando el aburrimiento supera al hambre, cerca de 0.2.

Conclusión: la acción cambia justo donde una necesidad pasa a ser mayor que las otras dos. La transición no es un salto brusco, sino una curva suave en las probabilidades.

### 1.2 Prioridades casi iguales: ¿cómo cambia Softmax?

| Hambre | Energía | Aburrimiento | COMER | DORMIR | JUGAR | Acción |
|---|---|---|---|---|---|---|
| 0.5 | 0.5 | 0.50 | 35.2 % | 33.4 % | 31.3 % | COMER |
| 0.5 | 0.5 | 0.52 | 30.7 % | 29.0 % | 40.3 % | JUGAR |
| 0.6 | 0.4 | 0.60 | 35.4 % | 33.4 % | 31.2 % | COMER |
| 0.6 | 0.4 | 0.61 | 33.2 % | 31.3 % | 35.6 % | JUGAR |
| 0.7 | 0.3 | 0.69 | 37.6 % | 35.3 % | 27.1 % | COMER |

Cuando las tres prioridades son casi iguales, las probabilidades quedan cerca de 33 % cada una, es decir, la red tiene incertidumbre. Un cambio de 0.01 o 0.02 en una entrada puede cambiar la acción elegida. Cuando una prioridad domina claramente, Softmax asigna casi 100 % a esa acción y casi 0 % a las otras.

En el caso (0.5, 0.5, 0.5) hay un empate real en la política. La red elige COMER por pequeñas diferencias numéricas de sus pesos, no por una razón de fondo.

### 1.3 Capa oculta de 6 a 3, 10 o 20 neuronas

Con la semilla 42:

| Neuronas ocultas | Pérdida final | Precisión de prueba |
|---|---|---|
| 3 | 0.1618 | 96.3 % |
| 6 | 0.1225 | 99.4 % |
| 10 | 0.1262 | 99.3 % |
| 20 | 0.1220 | 98.8 % |

Como un solo resultado depende de la inicialización aleatoria, repetí el experimento con 5 semillas y promedié:

| Neuronas ocultas | Pérdida media | Precisión media |
|---|---|---|
| 3 | 0.277 | 88.9 % |
| 6 | 0.153 | 97.8 % |
| 10 | 0.122 | 99.0 % |
| 20 | 0.121 | 99.2 % |

Interpretación:

- Con 3 neuronas la red tiene poca capacidad. La política necesita varias fronteras de decisión (hambre vs. cansancio, hambre vs. aburrimiento, cansancio vs. aburrimiento) y a veces no alcanzan, sobre todo si alguna neurona ReLU queda inactiva. Es el caso con peor pérdida y más variabilidad.
- Con 6 neuronas ya funciona bien.
- Con 10 o 20 mejora un poco y es más estable, pero la ganancia es mínima para un problema tan simple y el costo computacional aumenta. En la semilla 42, 20 neuronas queda por debajo de 6 en precisión, lo cual es ruido de inicialización y no una tendencia.

### 1.4 Épocas y tasa de aprendizaje

Épocas (tasa 0.1):

| Épocas | Pérdida | Precisión |
|---|---|---|
| 50 | 0.939 | 60.6 % |
| 200 | 0.498 | 91.8 % |
| 600 | 0.194 | 98.6 % |
| 1200 | 0.123 | 99.4 % |
| 3000 | 0.071 | 99.5 % |

El entrenamiento mejora rápido al inicio y luego se estanca. De 1200 a 3000 épocas la pérdida sigue bajando, pero la precisión casi no cambia (rendimientos decrecientes).

Tasa de aprendizaje (1200 épocas):

| Tasa | Pérdida final | Precisión | Comportamiento |
|---|---|---|---|
| 0.001 | 1.033 | 41.8 % | Muy lenta, casi no aprende |
| 0.01 | 0.738 | 82.9 % | Lenta, le faltan épocas |
| 0.1 | 0.123 | 99.4 % | Adecuada |
| 0.5 | 0.048 | 99.6 % | Muy buena, converge más rápido |
| 1.0 | 0.045 | 98.7 % | Al límite, la pérdida sube a 1.09 al inicio |
| 3.0 | 0.729 | 58.9 % | Inestable, oscila y no converge |
| 10.0 | 2.877 | 33.8 % | Diverge |

Conclusión: una tasa muy pequeña se estanca porque aprende demasiado lento, una tasa adecuada converge y una tasa muy grande hace que los pasos del gradiente salten sobre el mínimo, volviendo inestable el entrenamiento.

### 1.5 ¿Qué datos y etiquetas requeriría una cuarta acción?

Ejemplo: agregar la acción BAÑARSE.

1. Una nueva entrada que la justifique, por ejemplo higiene. Sin ella la red no tiene información para decidir cuándo bañarse. La matriz de entrada pasa de 3 a 4 columnas: `X = np.random.rand(3000, 4)`.
2. Una regla de etiquetado para los ejemplos sintéticos (o datos reales), por ejemplo:
   `targets = np.argmax(np.column_stack((X[:,0], 1 - X[:,1], X[:,2], 1 - X[:,3])), axis=1)`
   donde `1 - X[:,3]` representa la suciedad.
3. Etiquetas con 4 clases (0, 1, 2, 3), con suficientes ejemplos de cada una, balanceados e incluyendo casos cercanos a las fronteras.
4. Cambios en la arquitectura: `W1` de forma (4, h), `W2` de (h, 4) y `b2` de (1, 4). La salida Softmax pasa a 4 neuronas.
5. Cambios en la interfaz: nueva tupla `actions`, textos en el diccionario `TEXT`, barra de la nueva necesidad, teclas para modificarla y nuevas imágenes.
6. Reentrenar y evaluar la precisión por clase.

---

## 2. Explica tus resultados

### 2.1 ¿En qué se diferencian pesos y sesgos?

Los pesos (`W1`, `W2`) multiplican cada entrada y definen cuánta importancia tiene esa entrada para cada neurona, y con qué signo. Determinan la inclinación de la frontera de decisión. Por ejemplo, un peso grande y positivo del hambre hacia COMER hace que el hambre favorezca esa acción.

Los sesgos (`b1`, `b2`) se suman al resultado sin depender de la entrada. Desplazan la frontera de decisión y permiten que una neurona tenga un umbral de activación propio.

En `salida = entrada @ W + b`, el peso cumple el papel de la pendiente y el sesgo el del desplazamiento, como `m` y `b` en `y = mx + b`. Sin sesgo, todas las fronteras tendrían que pasar por el origen.

### 2.2 ¿Por qué el cansancio es 1 − energía?

Porque son magnitudes opuestas: con energía alta hay poco cansancio y con energía baja hay mucho. Así, la necesidad de dormir crece cuando la energía baja, igual que el hambre y el aburrimiento crecen cuando suben sus valores. De esta forma las tres cantidades quedan en la misma escala de urgencia (más valor, más prioridad) y se pueden comparar con `argmax`. Si se usara la energía directamente, la política elegiría DORMIR cuando hay más energía, lo cual no tiene sentido.

### 2.3 ¿Qué hace ReLU y qué hace Softmax?

ReLU (`max(0, x)`) deja pasar los valores positivos y convierte los negativos en 0. Introduce no linealidad, lo que permite aprender fronteras de decisión más complejas que una recta. Sin ella, varias capas lineales equivaldrían a una sola. Se usa en la capa oculta, y su derivada (`z1 > 0` en el código) se usa en la retropropagación.

Softmax convierte las salidas de la última capa (logits) en probabilidades positivas que suman 1. La acción con el logit más alto recibe la mayor probabilidad. Se usa en la capa de salida junto con la pérdida de entropía cruzada. En el código se resta el máximo (`values - max`) por estabilidad numérica, sin cambiar el resultado.

### 2.4 ¿Por qué una precisión alta puede ser engañosa?

1. Clases desbalanceadas: si el 95 % de los ejemplos fueran de una sola clase, un modelo que siempre responde esa clase tendría 95 % de precisión sin haber aprendido nada. Aquí las clases están balanceadas (cerca de 33 % cada una), así que el azar daría alrededor de 33 % y el 99.4 % sí es significativo.
2. No muestra la confianza: en prioridades casi iguales la red acierta la acción, pero con probabilidades de alrededor de 35 %. La precisión no distingue acertar con 99 % de acertar con 35 %. La pérdida sí lo hace.
3. Los errores se concentran en las fronteras: el 0.6 % que falla corresponde a casos de empate o casi empate, que son los más ambiguos.
4. Solo mide el ajuste a la política sintética: la red aprende la regla `argmax` que definimos nosotros, no una verdad del mundo real. Además, el estado NEUTRAL que se ve en pantalla es una regla de visualización que la red nunca aprendió.
5. Los datos de prueba vienen de la misma distribución que los de entrenamiento. Si la distribución cambiara, el rendimiento podría caer.

Por eso conviene revisar también la pérdida, la precisión por clase (en la prueba: COMER 99.9 %, DORMIR 98.8 %, JUGAR 99.0 %), la matriz de confusión y las probabilidades de salida.

---

## 3. Conclusiones

- La red aprende la política de prioridades con cerca de 99 % de precisión usando solo 6 neuronas ocultas.
- Los puntos de cambio de acción coinciden con los cruces de prioridad (hambre = 1 − energía = aburrimiento).
- Más neuronas dan una mejora pequeña y estable a partir de 10. Con 3 la red se queda corta.
- La tasa de aprendizaje es el hiperparámetro más sensible: muy baja no aprende, muy alta diverge, y entre 0.1 y 0.5 funciona bien.
- Agregar una acción exige una nueva entrada, una política de etiquetado y una neurona de salida adicional.
