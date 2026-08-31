---
layout: default
title: Actividad 1 — Espacio Individual
nav_order: 2
permalink: /actividad1/
---

# Actividad 1 — Espacio Individual
{: .no_toc }

Espacio de Kendon y fronteras de Hall alrededor de un humano que camina al azar
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Planteamiento

Un espacio de simulación continuo y cerrado (10×10 m) contiene un humano representado como
un punto que se mueve describiendo trayectorias aleatorias. Alrededor de él se dibujan **dos
conceptos proxémicos distintos**, que a propósito NO se modelan igual:

- **Fronteras de Hall** — tres círculos concéntricos y **simétricos** (zona íntima, personal
  y social), independientes de hacia dónde mire la persona.
- **Espacio de Kendon** — una **elipse orientada** según la dirección en la que camina el
  humano: más grande al frente (donde está su atención/foco de interacción) y más pequeña a
  los lados y atrás. A diferencia de Hall, es asimétrico.

Modelar ambos por separado (y no colapsarlos en un solo círculo, como haría una
simplificación excesiva) es importante porque la Actividad 4 usa específicamente el espacio
de Kendon —no las fronteras de Hall— como referencia para la aproximación frontal del robot.

## Caminata aleatoria (compartida por todas las actividades)

La función que mueve a cualquier humano en la simulación, con rebote suave en los bordes del
espacio cerrado:

{% highlight matlab %}
function [pos, heading] = mover_aleatorio(pos, heading, v, dt, L, sigma)
    heading = heading + sigma*randn;
    candidato = pos + v*dt*[cos(heading), sin(heading)];
    if candidato(1) < 0 || candidato(1) > L
        heading = pi - heading;
    end
    if candidato(2) < 0 || candidato(2) > L
        heading = -heading;
    end
    pos = min(max(pos + v*dt*[cos(heading), sin(heading)], [0 0]), [L L]);
end
{% endhighlight %}

## Dibujo de las fronteras de Hall (círculos simétricos)

{% highlight matlab %}
function dibujarZonasHall(centro, r1, r2, r3)
    th = linspace(0, 2*pi, 60);
    fill(centro(1)+r3*cos(th), centro(2)+r3*sin(th), [1 0.9 0.9], 'EdgeColor', 'none', 'FaceAlpha', 0.5);
    fill(centro(1)+r2*cos(th), centro(2)+r2*sin(th), [1 0.7 0.7], 'EdgeColor', 'none', 'FaceAlpha', 0.6);
    fill(centro(1)+r1*cos(th), centro(2)+r1*sin(th), [1 0.4 0.4], 'EdgeColor', 'none', 'FaceAlpha', 0.7);
end
{% endhighlight %}

## Dibujo del espacio de Kendon (elipse orientada)

{% highlight matlab %}
function dibujarEspacioKendon(centro, heading, a, b)
    % Elipse orientada segun hacia donde mira la persona: mas grande al
    % frente (semi-eje a), mas chica a los lados/atras (semi-eje b).
    % A diferencia de Hall (circulos simetricos), el espacio de Kendon
    % es asimetrico.
    th = linspace(0, 2*pi, 60);
    ex = a*cos(th);
    ey = b*sin(th);
    xr = centro(1) + ex*cos(heading) - ey*sin(heading);
    yr = centro(2) + ex*sin(heading) + ey*cos(heading);
    plot(xr, yr, 'b--', 'LineWidth', 1.5);
end
{% endhighlight %}

## Parámetros usados

| Parámetro | Valor | Significado |
|---|---|---|
| `r_intima` | 0.45 m | Radio zona íntima de Hall |
| `r_personal` | 1.20 m | Radio zona personal de Hall |
| `r_social` | 3.60 m | Radio zona social de Hall |
| `kendon_frente` | 1.50 m | Semi-eje mayor del espacio de Kendon (frente) |
| `kendon_lado` | 0.60 m | Semi-eje menor del espacio de Kendon (lados/atrás) |

## Resultado de ejecución

{% highlight text %}
===== ACTIVIDAD 1: Espacio individual (Kendon / Hall) =====
  ... paso 50/150
  ... paso 100/150
  ... paso 150/150
Actividad 1 completada (150 pasos). Video: videos/actividad1_kendon_hall.mp4
{% endhighlight %}

150 pasos de simulación, sin errores. El video generado muestra al humano caminando dentro
del recinto con ambas zonas siguiéndolo en cada instante — los círculos de Hall centrados en
él, y la elipse de Kendon rotando para siempre apuntar hacia donde camina.

> **Nota:** el "espacio de Kendon" individual es una simplificación pedagógica común en
> robótica social (asociada a menudo con modelos de espacio personal asimétrico orientado
> por la dirección de la mirada/marcha). El concepto formal de Kendon en la literatura de
> proxémica (F-formations) se aplica propiamente a grupos — ver la
> [Actividad 2](../actividad2/).

---

[&larr; Inicio](../) &nbsp;&nbsp; [Actividad 2 — F-formations &rarr;](../actividad2/){: .btn .btn-outline }
