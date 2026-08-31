---
layout: default
title: Actividad 4 — Interacción Frontal
nav_order: 5
permalink: /actividad4/
---

# Actividad 4 — Interacción Robot-Humano Frontal
{: .no_toc }

Aproximación deliberada usando el espacio de Kendon como referencia
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Planteamiento

El robot busca interactuar deliberadamente con el humano, aproximándose siempre **de
frente** y usando de referencia el **espacio de Kendon** (no las fronteras de Hall — el
enunciado es explícito en este punto). El humano deja de deambular en cuanto el robot logra
ingresar correctamente al espacio pertinente.

## Estrategia

1. El robot persigue un punto objetivo: la **punta del eje mayor** del espacio de Kendon del
   humano (es decir, un punto justo delante de él, a la distancia `kendon_frente`).
2. Como el humano sigue moviéndose y girando mientras deambula, ese objetivo se recalcula en
   cada paso — el robot persigue un "blanco móvil".
3. La interacción se considera establecida cuando el robot está **realmente dentro** de la
   elipse del espacio de Kendon — una prueba geométrica exacta, no una aproximación por
   distancia + ángulo por separado.

{% highlight matlab %}
% Objetivo: la PUNTA del espacio de Kendon (su eje mayor, al frente del humano)
objetivo = pos_humano + cfg.kendon_frente*[cos(heading_humano), sin(heading_humano)];
vec_ir = objetivo - pos_robot;
dist_obj = norm(vec_ir);
if dist_obj > 1e-3
    heading_robot = atan2(vec_ir(2), vec_ir(1));
    paso_mov = min(cfg.v_robot*cfg.dt, dist_obj);
    pos_robot = pos_robot + paso_mov*[cos(heading_robot), sin(heading_robot)];
end

% La interaccion se establece cuando el robot ESTA REALMENTE
% dentro del espacio de Kendon (prueba geometrica con la elipse
% orientada, no solo distancia+angulo aproximados).
if ~interaccion && dentro_de_kendon(pos_robot, pos_humano, heading_humano, cfg.kendon_frente, cfg.kendon_lado)
    interaccion = true;
end
{% endhighlight %}

## La prueba geométrica: ¿el robot está dentro de la elipse?

Se rota el punto al marco de referencia del humano (donde su "frente" es el eje X) y se
evalúa la ecuación de la elipse:

{% highlight matlab %}
function dentro = dentro_de_kendon(punto, centro, heading, a, b)
    % Prueba geometrica: verdadero/falso si "punto" cae dentro de la
    % elipse orientada del espacio de Kendon de la persona en "centro".
    d = punto - centro;
    xr =  d(1)*cos(-heading) - d(2)*sin(-heading);
    yr =  d(1)*sin(-heading) + d(2)*cos(-heading);
    dentro = (xr/a)^2 + (yr/b)^2 <= 1;
end
{% endhighlight %}

Una vez que `interaccion = true`, el humano deja de moverse (deja de llamarse a
`mover_aleatorio`) y la simulación mantiene la escena final unos segundos antes de terminar.

## Resultado de ejecución

{% highlight text %}
===== ACTIVIDAD 4: Interaccion robot-humano frontal =====
  ... paso 40 | dist. al humano: 1.82 m
Actividad 4: interaccion frontal establecida en el paso 49.
Video: videos/actividad4_interaccion_frontal.mp4
{% endhighlight %}

La interacción frontal se estableció en el paso 49 (~10 segundos simulados): el robot logró
ubicarse correctamente dentro del espacio de Kendon del humano, de frente, y el humano se
detuvo. El video muestra la flecha que indica hacia dónde mira el humano en todo momento,
para verificar visualmente que la aproximación fue efectivamente frontal y no lateral.

---

[&larr; Actividad 3](../actividad3/) &nbsp;&nbsp; [Actividad 5 — Navegación Multiagente &rarr;](../actividad5/){: .btn .btn-outline }
