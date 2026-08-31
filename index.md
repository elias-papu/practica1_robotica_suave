---
layout: default
title: Inicio
nav_order: 1
description: "Práctica de Laboratorio: Proxémica y Navegación Robótica"
permalink: /
---

# Proxémica y Navegación Robótica
{: .no_toc }

Práctica de Laboratorio — Robótica
{: .fs-5 .fw-300 }

---

## Datos generales

| | |
|---|---|
| **Estudiante** | Elías Santiago Jiménez Hernández |
| **Carrera** | Ingeniería en Inteligencia Artificial |
| **Semestre** | 9no semestre |
| **Materia** | Robótica |
| **Profesor** | Dr. Alexandro López González |
| **Institución** | Universidad Iberoamericana, Ciudad de México |
| **Departamento** | Estudios en Ingeniería para la Innovación (DEII) |

## Objetivo de la práctica

Comprender y aplicar los conceptos fundamentales de la proxémica humana —espacios de
Kendon, fronteras de Hall y F-formations— mediante la programación de simulaciones y
algoritmos de navegación robótica en MATLAB.

## Qué contiene este sitio

Las 5 actividades solicitadas, cada una con su explicación, su fragmento de código relevante
y sus resultados reales de ejecución:

- [Actividad 1 — Espacio Individual](actividad1/) — espacio de Kendon (elipse orientada) vs.
  fronteras de Hall (círculos simétricos) alrededor de un humano que camina al azar.
- [Actividad 2 — F-formations](actividad2/) — dinámica de grupo: 4 humanos que forman y
  disuelven F-formations (O-space, P-space, R-space) dinámicamente.
- [Actividad 3 — Navegación con Evasión](actividad3/) — un robot navega esquina a esquina
  evitando a un humano, usando lógica difusa combinada con un campo potencial.
- [Actividad 4 — Interacción Frontal](actividad4/) — el robot se aproxima deliberadamente
  de frente al humano, usando el espacio de Kendon como referencia geométrica real.
- [Actividad 5 — Navegación Multiagente](actividad5/) — la misma navegación evasiva, ahora
  entre 4 humanos que además pueden formar F-formations dinámicamente.
- [Comparativa y Resultados](comparativa/) — Actividad 3 vs. Actividad 5: cómo afecta la
  complejidad del entorno al desempeño del robot.
- [Código Fuente](codigo-fuente/) — el script MATLAB completo, documentado.

## Decisiones de diseño relevantes

> **Sobre "aprender":** este trabajo usa lógica difusa, no aprendizaje reforzado. Las reglas
> del controlador son fijas por diseño — no mejoran de una corrida a otra, y eso es
> exactamente lo que distingue a este método de un agente de RL. La lógica difusa está
> permitida explícitamente en el enunciado de la práctica.

> **Sobre las violaciones de espacio en la Actividad 5:** los humanos caminan de forma
> aleatoria e independiente del robot (solo el robot los evade a ellos, no al revés). En un
> espacio de 10×10 m con 4 personas moviéndose libremente, algunas violaciones ocasionales
> son estadísticamente inevitables — no un error del controlador. Ver la sección de
> [Comparativa](comparativa/) para el análisis completo.

---

Todo el código fuente y los videos generados están disponibles para descarga en este sitio.
