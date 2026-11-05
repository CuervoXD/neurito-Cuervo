# Solución de los experimentos de Neurito

## Método

Se usaron los tres mapas incluidos, la semilla `42`, una división de 80 % para entrenamiento y 20 % para prueba, y los 716 casos de prueba generados por `generar_datos`. La arquitectura base es `165 -> 96 -> 4`, con ReLU, Softmax, mini-lotes y tasa de aprendizaje `0.45`.

En navegación, cada caso tuvo como límite `min(65, 2 * distancia_optima + 8)`, igual que en `test_laberinto.py`. "Éxito" significa llegar a la meta. Los movimientos promedio cuentan solo los pasos válidos de los casos que sí llegaron.

## 1. Neurito sin entrenar

Los pesos se inicializan aleatoriamente. Por eso las cuatro salidas Softmax no representan todavía las etiquetas producidas por BFS: la red puede elegir una dirección incorrecta, chocar con una pared o repetir posiciones.

Con la semilla usada:

| Estado | Exactitud de prueba | Red pura | Asistido | Choques en red pura |
|---|---:|---:|---:|---:|
| 0 épocas | 27.65 % | 22/716 | 412/716 | 589 |

La asistencia mejora el comportamiento incluso sin aprendizaje porque elimina movimientos contra paredes y penaliza las casillas ya visitadas. No sustituye el aprendizaje: al principio todavía solo logra 412 éxitos de 716.

## 2. Efecto del número de épocas

| Épocas | Pérdida | Exactitud prueba | Red pura | Asistido |
|---:|---:|---:|---:|---:|
| 100 | 0.135209 | 84.92 % | 367/716, 15 choques | 701/716, 0 choques |
| 200 | 0.037642 | 90.08 % | 582/716, 5 choques | 708/716, 0 choques |
| 400 | 0.010665 | 91.48 % | 592/716, 3 choques | 713/716, 0 choques |

La pérdida baja y la exactitud sube con el entrenamiento. La mejora de la red pura se nota mucho entre 100 y 200 épocas, pero después se ralentiza. La navegación asistida mantiene una ventaja mayor porque aplica restricciones que la red no aprendió explícitamente.

## 3. Red pura frente a navegación asistida

La **red neuronal** hace el cálculo aprendido:

1. Recibe el identificador del mapa, la posición actual y la meta codificados como vectores one-hot.
2. Produce cuatro probabilidades mediante Softmax.
3. En red pura se elige `argmax`, es decir, la acción con mayor probabilidad.

Las **reglas de asistencia** están en `elegir_accion` y no son parte de los pesos:

- descartan acciones que llevan a una pared;
- restan `0.35` por cada visita previa a una casilla;
- eligen la mejor puntuación restante.

A 400 épocas, la red pura consiguió 592/716 casos y tuvo 3 choques. La asistencia consiguió 713/716, sin choques. La precisión de clasificación no equivale al porcentaje de laberintos completados: una predicción incorrecta puede corregirse en el siguiente paso, y una secuencia de errores puede producir un ciclo.

## 4. Número de neuronas ocultas

Todas las variantes se entrenaron durante 400 épocas con tasa `0.45`.

| Neuronas ocultas | Pérdida | Exactitud prueba | Red pura | Asistido |
|---:|---:|---:|---:|---:|
| 16 | 0.034239 | 87.15 % | 532/716, 23 choques | 709/716 |
| 32 | 0.016075 | 89.25 % | 561/716, 12 choques | 704/716 |
| 96 | 0.010665 | 91.48 % | 592/716, 3 choques | 713/716 |
| 128 | 0.010282 | 90.78 % | 611/716, 10 choques | 707/716 |

Con 16 neuronas hay menos capacidad y la red aprende peor. 32 mejora, pero sigue por debajo de 96. Aumentar a 128 no garantiza una mejora: con esta semilla y este número de épocas obtuvo una pérdida ligeramente menor, pero una exactitud de prueba menor que 96. La capacidad debe evaluarse junto con la generalización, no solo con la pérdida de entrenamiento.

## 5. Tasa de aprendizaje

| Tasa | Pérdida | Exactitud prueba | Red pura | Asistido | Observación |
|---:|---:|---:|---:|---:|---|
| 0.10 | 0.149048 | 87.15 % | 431/716 | 708/716 | Converge más despacio |
| 0.45 | 0.010665 | 91.48 % | 592/716 | 713/716 | Buen equilibrio en esta prueba |
| 1.00 | 0.002819 | 93.30 % | 612/716 | 711/716 | Converge aquí, pero requiere vigilar estabilidad |

La tasa `0.45` sí converge. En esta ejecución `1.0` terminó con mejores métricas, aunque una tasa alta puede producir oscilaciones o inestabilidad con otros datos, inicializaciones o arquitecturas. No conviene decidir solo por una ejecución.

## 6. Cambio de mapa

La red base entrenada con 400 épocas se evaluó por separado en cada mapa.

| Mapa | Casos | Red pura | Asistido |
|---:|---:|---:|---:|
| 1 | 210 | 180/210 | 210/210 |
| 2 | 251 | 197/251 | 248/251 |
| 3 | 255 | 215/255 | 255/255 |

Los mapas sí están incluidos en el universo de entrenamiento mediante su identificador, pero los pares concretos de posición y meta de prueba no aparecen en el entrenamiento. Por tanto, esto mide generalización a nuevos retos dentro de los mismos tres mapas, no generalización a un mapa completamente desconocido. Para probar mapas nuevos habría que incluir una representación de sus paredes y separar mapas de entrenamiento y evaluación.

## 7. DFS, BFS y A*

Se hizo una comparación adicional generando etiquetas con cada planificador y entrenando la misma red durante 400 épocas. BFS es el experto que usa el proyecto original. DFS y A* se implementaron solo en el experimento para comparar las etiquetas; el código base sigue usando BFS.

| Planificador de etiquetas | Pérdida | Exactitud prueba | Red pura | Asistido |
|---|---:|---:|---:|---:|
| BFS | 0.010665 | 91.48 % | 592/716 | 713/716 |
| DFS | 0.009999 | 94.83 % | 82/716 | 256/716 |
| A* | 0.010920 | 90.92 % | 613/716 | 713/716 |

La precisión de DFS resulta engañosa en este caso: DFS enseña rutas válidas, pero no necesariamente cortas. Sus trayectorias son mucho más largas y menos adecuadas para la regla de penalizar visitas, por lo que el éxito de navegación cae drásticamente. BFS produce rutas cortas y consistentes; por eso es el mejor profesor para este experimento. A* también encuentra rutas óptimas en estos mapas y queda cerca de BFS, con una pequeña diferencia debida al desempate y a las etiquetas/split utilizados.

## Conclusión

Neurito aprende a imitar al experto BFS, no a descubrir por sí mismo la solución mediante refuerzo. El entrenamiento reduce la pérdida y mejora la clasificación, pero las reglas de navegación son decisivas para evitar paredes y ciclos. La configuración de 96 neuronas, 400 épocas y tasa `0.45` ofrece un resultado sólido y estable en el proyecto original: 91.48 % de exactitud, 592/716 éxitos en red pura y 713/716 con asistencia.

