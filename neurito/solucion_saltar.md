1. Juego sin entrenar

Simulación automática durante 10.000 frames:
Colisiones	Obstáculos esquivados	Saltos
66	0	0

La red inicial no supera el umbral de salto (0.5), por lo que el personaje nunca salta.

2. Entrenamiento base: ocultas=12, tasa 0.8

Resultados cada 100 épocas, con una simulación de 3.000 frames:
Épocas	Pérdida	Precisión prueba	Colisiones	Esquivados
100	0.2601	91.2%	1	22
200	0.2306	91.9%	0	24
300	0.2066	92.3%	0	24
400	0.1753	92.6%	0	24
500	0.1368	94.1%	0	24
600	0.1056	95.7%	0	24
700	0.0858	96.5%	0	24
800	0.0730	97.0%	0	24
900	0.0641	97.7%	0	24
1000	0.0574	97.8%	0	24
1100	0.0523	97.9%	0	24
1200	0.0481	98.1%	0	24

El comportamiento mejora claramente a partir de 200 épocas. La pérdida disminuye y la red aprende a saltar en la ventana adecuada.

3. Neuronas ocultas
Neuronas	Pérdida final	Precisión prueba	Comportamiento
3	0.2042	93.3%	Aprende, pero con menor precisión
12	0.0481	98.1%	Mejor equilibrio
24	0.0431	97.9%	Pérdida menor, pero generalización ligeramente peor

Con 3 neuronas la capacidad es limitada. Con 24 se ajusta mejor al entrenamiento, pero no mejora la precisión de prueba: aparece una pequeña señal de sobreajuste o variación por inicialización.

4. Tasa de aprendizaje
Tasa	Pérdida final	Precisión prueba	Colisiones / esquivados
0.01	0.421 aprox.	86.7%	62 / 0
0.8	0.0481	98.1%	0 / 81
1.2	0.036 aprox.	98.5%	0 / 81

La tasa 0.01 converge demasiado lentamente y todavía no aprende una política útil después de 1200 épocas. 0.8 converge de forma estable. 1.2 también funciona en este caso, aunque presenta más variación durante las primeras épocas.

Conclusión: una precisión alta sobre datos sintéticos no garantiza por sí sola evitar colisiones, porque el juego depende además del tiempo exacto del salto, la velocidad, la física y el umbral de decisión.
