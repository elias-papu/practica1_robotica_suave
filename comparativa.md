---
layout: default
title: Comparativa y Resultados
nav_order: 7
permalink: /comparativa/
---

# Comparativa y Resultados
{: .no_toc }

Actividad 3 (agente único) vs. Actividad 5 (multiagente)
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Por qué esta comparación

Las Actividades 3 y 5 usan **exactamente el mismo controlador** (misma lógica difusa, mismos
parámetros de zonas proxémicas). Lo único que cambia es la complejidad del entorno: un
humano vs. cuatro humanos con F-formations dinámicas. Eso hace que la comparación sea
limpia: cualquier diferencia en desempeño se debe a la complejidad del entorno, no a un
cambio de método.

## Resultados de la última corrida

| Métrica | Actividad 3 (1 humano) | Actividad 5 (4 humanos) |
|---|---|---|
| Pasos hasta la meta | 110 | 110 |
| Violaciones de zona de peligro | 0 | 18 |
| Distancia mínima alcanzada | 3.10 m | 0.70 m |

{% highlight text %}
===================== COMPARATIVA: Actividad 3 vs Actividad 5 =====================
Metrica                        Actividad 3 (agente unico) Actividad 5 (multiagente)
Pasos hasta la meta            110                  110
Violaciones                    0                    18
Distancia minima (m)           3.10                 0.70
=====================================================================================
{% endhighlight %}

## Interpretación

- **El tiempo hasta la meta no se degrada** significativamente (110 pasos en ambos casos):
  el controlador sigue encontrando una ruta razonablemente directa incluso con más
  obstáculos.
- **Las violaciones sí se disparan** (de 0 a 18): con cuatro agentes moviéndose de forma
  independiente y aleatoria, y una evasión que es unidireccional (solo el robot reacciona a
  los humanos, no al revés), algunos encuentros cercanos son estadísticamente inevitables.
- **La distancia mínima cae de 3.10 m a 0.70 m**: es la métrica que más claramente muestra el
  costo de la complejidad añadida — en el peor momento de la corrida, el robot estuvo mucho
  más cerca de un obstáculo de lo que llegó a estar nunca en la Actividad 3.

## Limitaciones honestas del diseño actual

- La evasión es **reactiva**, no predictiva: el robot no anticipa hacia dónde se moverá un
  humano, solo reacciona a su posición actual. Un controlador con predicción de trayectoria
  reduciría las violaciones, a costa de mayor complejidad.
- Los humanos **no evitan al robot**. Un modelo bidireccional (donde ambos se evaden
  mutuamente, como ocurre en espacios compartidos reales) probablemente bajaría las
  violaciones de forma sustancial.
- El controlador atiende a **todos** los obstáculos a la vez mediante un promedio ponderado
  por urgencia, lo cual es más robusto que atender solo al más cercano, pero sigue siendo una
  heurística reactiva, no una planificación de trayectoria global.

## Sobre el aprendizaje (o la ausencia de él)

Ninguna de las dos actividades usa aprendizaje reforzado. El controlador de lógica difusa
tiene reglas fijas, escritas de antemano, que no cambian entre corridas ni mejoran con la
experiencia. Esa es una diferencia fundamental frente a un agente de RL, que sí aprendería
(por prueba y error, con una función de recompensa) a reducir las violaciones con el tiempo.
Se optó por lógica difusa en este trabajo porque el enunciado la permite explícitamente y
porque el Reinforcement Learning Toolbox de MATLAB presentó incompatibilidades de sintaxis
entre versiones durante el desarrollo.

---

[&larr; Actividad 5](../actividad5/) &nbsp;&nbsp; [Código Fuente &rarr;](../codigo-fuente/){: .btn .btn-outline }
