---
layout: default
title: Actividad 5 — Navegación Multiagente
nav_order: 6
permalink: /actividad5/
---

# Actividad 5 — Navegación Compleja entre Múltiples Agentes
{: .no_toc }

El robot navega entre 4 humanos que además pueden formar F-formations
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Planteamiento

Empleando el espacio de simulación de la Actividad 2 (4 humanos + F-formations dinámicas), el
robot móvil realiza la misma tarea de navegación esquina a esquina de la Actividad 3, pero
ahora sorteando dinámicamente al grupo de humanos y respetando los espacios sociópetos
(O-space, P-space, R-space) cuando los individuos se organizan en F-formations.

## Cómo se generaliza el controlador de la Actividad 3

La clave de diseño es que el controlador difuso de la Actividad 3 recibe una **distancia
normalizada** (distancia real ÷ radio de peligro del obstáculo), no una distancia absoluta.
Eso permite construir, en cada paso, una lista de "amenazas" heterogénea — humanos sueltos
(radio de peligro = `r_personal`) y F-formations activas completas (radio de peligro =
`rR`, el R-space) — y evaluar todas con el **mismo** controlador:

{% highlight matlab %}
% ---- construir la lista de obstaculos para el robot ----
amenazas = [];
for f = 1:numel(formaciones)
    if formaciones(f).activo
        amenazas(end+1, :) = [formaciones(f).centro, cfg.rR];
    end
end
for i = 1:n
    if strcmp(estado{i}, 'libre')
        amenazas(end+1, :) = [pos(i,:), cfg.r_personal];
    end
end

[pos_robot, info] = paso_robot_evasivo(pos_robot, meta, amenazas, fis, cfg.v_robot, cfg.dt, cfg.L);
{% endhighlight %}

## Combinando la evasión de varios obstáculos a la vez

En vez de reaccionar solo a la amenaza más cercana (lo que dejaba al robot ciego ante otras
amenazas simultáneas), cada obstáculo activo "vota" cuánto desviarse del rumbo directo a la
meta, y los votos se combinan en un **promedio angular ponderado** por la urgencia de cada
uno. Cuando solo hay un obstáculo, la fórmula se reduce exactamente a la de la Actividad 3:

{% highlight matlab %}
function [pos_robot_nuevo, info] = paso_robot_evasivo(pos_robot, meta, amenazas, fis, v_robot, dt, L)
    vec_meta = meta - pos_robot;
    heading_deseado = atan2(vec_meta(2), vec_meta(1));

    suma_ponderada = 0;
    suma_pesos = 0;
    mejor_margen = inf;
    dist_amenaza = inf;
    radio_amenaza = 0;

    for a = 1:size(amenazas, 1)
        vec_a = amenazas(a, 1:2) - pos_robot;
        d = max(norm(vec_a), 1e-6);
        radio = amenazas(a, 3);

        ang_hacia = atan2(vec_a(2), vec_a(1));
        ang_rel = envolver_angulo(ang_hacia - heading_deseado);
        angulo_abs_deg = abs(rad2deg(ang_rel));

        desviacion_mag = evaluar_evasion(fis, d, radio, angulo_abs_deg);  % 0 a 70 grados
        signo = -sign(ang_rel);
        if signo == 0, signo = 1; end

        % El peso de cada obstaculo en el promedio es su propia
        % magnitud de desviacion: el mas urgente domina el resultado.
        peso = desviacion_mag;
        suma_ponderada = suma_ponderada + peso * (signo * desviacion_mag);
        suma_pesos = suma_pesos + peso;

        margen = d - radio;
        if margen < mejor_margen
            mejor_margen = margen; dist_amenaza = d; radio_amenaza = radio;
        end
    end

    if suma_pesos > 1e-6
        offset_deg = suma_ponderada / suma_pesos;
    else
        offset_deg = 0;
    end

    heading_robot = heading_deseado + deg2rad(offset_deg);
    pos_robot_nuevo = pos_robot + v_robot*dt*[cos(heading_robot), sin(heading_robot)];
    pos_robot_nuevo = min(max(pos_robot_nuevo, [0 0]), [L L]);

    info.dist_amenaza = dist_amenaza; info.radio_amenaza = radio_amenaza; info.margen = mejor_margen;
end
{% endhighlight %}

> **Nota de diseño:** se descartó deliberadamente una versión anterior basada en suma de
> vectores cartesianos de repulsión. Esa versión podía hacer que la atracción hacia la meta y
> la repulsión de un obstáculo quedaran exactamente opuestas, atrapando al robot oscilando
> contra el obstáculo (el clásico problema de "mínimo local" de los campos potenciales
> puros). El promedio angular ponderado no tiene ese modo de falla.

## Resultado de ejecución

{% highlight text %}
===== ACTIVIDAD 5: Navegacion compleja multiagente =====
  ... paso 40 | amenaza mas cercana: 2.48 m | violaciones hasta ahora: 0
  ... paso 80 | amenaza mas cercana: 2.10 m | violaciones hasta ahora: 18
Actividad 5: el robot llego a la meta en el paso 110 (violaciones: 18).
Video: videos/actividad5_navegacion_multiagente.mp4
{% endhighlight %}

18 violaciones de zona de peligro en 110 pasos (~16%), distancia mínima alcanzada: 0.70 m. El
robot llegó a la meta en el mismo número de pasos que en la Actividad 3 (110), pero con un
entorno mucho más exigente.

## Por qué no es razonable esperar 0 violaciones aquí

Los 4 humanos caminan **de forma aleatoria e independiente del robot** — la evasión es
unidireccional (solo el robot evade a los humanos, no al revés). En un espacio de 10×10 m con
4 personas moviéndose libremente, es estadísticamente inevitable que ocasionalmente alguien
se cruce en el camino del robot antes de que le dé tiempo a reaccionar, sobre todo cuando dos
amenazas aparecen casi simultáneamente desde direcciones distintas. Forzar 0 violaciones
absolutas requeriría cambiar el escenario (evasión bidireccional, espacio más grande,
humanos más lentos) — ver la comparación cuantitativa completa en la página de
[Comparativa y Resultados](../comparativa/).

---

[&larr; Actividad 4](../actividad4/) &nbsp;&nbsp; [Comparativa y Resultados &rarr;](../comparativa/){: .btn .btn-outline }
