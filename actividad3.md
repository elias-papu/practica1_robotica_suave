---
layout: default
title: Actividad 3 — Navegación con Evasión
nav_order: 4
permalink: /actividad3/
---

# Actividad 3 — Navegación con Evasión Proxémica
{: .no_toc }

Agente único: lógica difusa + campo potencial
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Planteamiento

Un robot móvil navega desde una esquina del espacio hasta la esquina diametralmente opuesta,
evitando en todo momento invadir las fronteras de Hall de un humano que camina al azar. El
enunciado permite explícitamente campos potenciales artificiales, máquinas de estados
finitos o lógica difusa — aquí se usa una **combinación de las dos primeras opciones**: un
controlador de lógica difusa decide la *magnitud* de la evasión, y esa magnitud se aplica
como una corrección angular sobre el rumbo directo a la meta (equivalente conceptualmente a
un campo potencial, pero evitando su problema clásico de mínimos locales — ver el diseño
completo en la [página de código fuente](../codigo-fuente/)).

## El controlador difuso

Sistema de inferencia difuso tipo Mamdani con **dos entradas normalizadas** y una salida:

- **`distancia_norm`** — la distancia real dividida entre el radio de peligro del obstáculo.
  Un valor de 1 significa "justo en el borde del área que hay que respetar". Esta
  normalización es la clave que permite reutilizar el mismo controlador tanto para un humano
  individual (Actividad 3) como para una F-formation completa (Actividad 5).
- **`angulo`** — el ángulo absoluto entre el rumbo deseado y la dirección hacia el obstáculo
  (0° = justo al frente, 180° = justo atrás).
- **`desviacion`** (salida) — cuántos grados debe desviarse el robot de su rumbo directo.

{% highlight matlab %}
function fis = crear_fis_normalizado()
    fis = mamfis('Name', 'EvasionProxemicaNormalizada');
    fis = addInput(fis, [0 15], 'Name', 'distancia_norm');
    fis = addMF(fis, 'distancia_norm', 'trapmf', [0 0 0.375 1], 'Name', 'CERCA');
    fis = addMF(fis, 'distancia_norm', 'trimf',  [0.375 1 3],   'Name', 'MEDIA');
    fis = addMF(fis, 'distancia_norm', 'trapmf', [1 3 150 150], 'Name', 'LEJOS');

    fis = addInput(fis, [0 180], 'Name', 'angulo');
    fis = addMF(fis, 'angulo', 'trapmf', [0 0 30 70],    'Name', 'FRENTE');
    fis = addMF(fis, 'angulo', 'trimf',  [40 90 140],    'Name', 'LADO');
    fis = addMF(fis, 'angulo', 'trapmf', [110 150 180 180], 'Name', 'ATRAS');

    fis = addOutput(fis, [0 70], 'Name', 'desviacion');
    fis = addMF(fis, 'desviacion', 'trapmf', [0 0 5 15],   'Name', 'NULA');
    fis = addMF(fis, 'desviacion', 'trimf',  [10 30 50],   'Name', 'MODERADA');
    fis = addMF(fis, 'desviacion', 'trapmf', [40 60 70 70],'Name', 'FUERTE');

    ruleList = [
        1 1 3 1 1   % CERCA  + FRENTE -> FUERTE
        1 2 2 1 1   % CERCA  + LADO   -> MODERADA
        1 3 1 1 1   % CERCA  + ATRAS  -> NULA
        2 1 2 1 1   % MEDIA  + FRENTE -> MODERADA
        2 2 1 1 1   % MEDIA  + LADO   -> NULA
        2 3 1 1 1   % MEDIA  + ATRAS  -> NULA
        3 1 1 1 1   % LEJOS  (cualquier angulo) -> NULA
        3 2 1 1 1
        3 3 1 1 1
    ];
    fis = addRule(fis, ruleList);
end
{% endhighlight %}

> **Detalle técnico importante:** el plateau de la función `LEJOS` se extiende mucho más
> allá del universo real de entrada (`[1 3 150 150]` sobre un universo `[0 15]`). Esto evita
> un bug encontrado durante el desarrollo: si el plateau termina justo en el borde del
> universo, los valores que llegan recortados ("clipped") a ese borde caen a membresía 0 y
> ninguna regla se dispara, produciendo una desviación por defecto arbitraria.

## Bucle principal de navegación

{% highlight matlab %}
for paso = 1:max_pasos
    [pos_humano, heading_humano] = mover_aleatorio(pos_humano, heading_humano, cfg.v_humano, cfg.dt, cfg.L, 0.35);

    amenazas = [pos_humano, cfg.r_personal];
    [pos_robot, info] = paso_robot_evasivo(pos_robot, meta, amenazas, fis, cfg.v_robot, cfg.dt, cfg.L);

    if info.margen < 0, violaciones = violaciones + 1; end

    if dist_meta_actual < cfg.dist_llegada
        llego_meta = true;
        break;
    end
end
{% endhighlight %}

## Resultado de ejecución

{% highlight text %}
===== ACTIVIDAD 3: Navegacion con evasion (agente unico) =====
  ... paso 40 | dist. humano: 3.21 m | violaciones hasta ahora: 0
  ... paso 80 | dist. humano: 5.54 m | violaciones hasta ahora: 0
Actividad 3: el robot llego a la meta en el paso 110 (violaciones: 0).
Video: videos/actividad3_evasion_agente_unico.mp4
{% endhighlight %}

**0 violaciones de la zona personal, distancia mínima alcanzada al humano: 3.10 m.** El robot
llegó a la meta en 110 pasos (22 segundos simulados) sin invadir en ningún momento las
fronteras de Hall del humano. Este resultado se repitió de forma consistente en múltiples
corridas con distintas semillas aleatorias.

La figura de análisis generada por el script muestra la distancia al humano y el avance hacia
la meta a lo largo del tiempo, con líneas de referencia en las fronteras de zona íntima y
zona personal.

---

[&larr; Actividad 2](../actividad2/) &nbsp;&nbsp; [Actividad 4 — Interacción Frontal &rarr;](../actividad4/){: .btn .btn-outline }
