---
layout: default
title: Actividad 2 — F-formations
nav_order: 3
permalink: /actividad2/
---

# Actividad 2 — Dinámica de Grupos y F-formations
{: .no_toc }

4 humanos que caminan al azar y forman F-formations dinámicamente
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Planteamiento

4 humanos caminan al azar dentro del espacio cerrado. Cuando dos (o más) se acercan más de
un umbral de distancia, se organizan en una **F-formation**, con sus tres espacios
característicos:

- **O-space** — el núcleo vacío, el foco de la interacción (nadie se para ahí).
- **P-space** — el anillo donde se ubican los miembros del grupo, mirando hacia el centro.
- **R-space** — el límite exterior del grupo, que terceros deberían respetar.

Pasado un tiempo fijo, la formación se disuelve y cada miembro recupera su caminata
aleatoria y su espacio de Kendon/Hall individual.

## Diseño: formaciones que pueden crecer hasta 4 miembros

La implementación no se limita a parejas: una formación activa puede **absorber** a un
tercer o cuarto humano libre que se acerque, hasta un máximo configurable. Cada formación se
guarda como un elemento de un arreglo de structs (`activo`, `centro`, `miembros`,
`tiempo_restante`), lo que permite un número variable de miembros por formación:

{% highlight matlab %}
function [pos, heading, estado, grupo_id, formaciones] = actualizar_dinamica_grupo(pos, heading, estado, grupo_id, formaciones, cfg)
    n = size(pos, 1);

    % 1. mover a los que estan libres
    for i = 1:n
        if strcmp(estado{i}, 'libre')
            [pos(i,:), heading(i)] = mover_aleatorio(pos(i,:), heading(i), cfg.v_humano, cfg.dt, cfg.L, 0.3);
        end
    end

    % 2. absorber humanos libres cercanos a formaciones activas con espacio (hasta max_miembros_formacion)
    for f = 1:numel(formaciones)
        if ~formaciones(f).activo, continue; end
        if numel(formaciones(f).miembros) >= cfg.max_miembros_formacion, continue; end
        for i = 1:n
            if ~strcmp(estado{i}, 'libre'), continue; end
            d = norm(pos(i,:) - formaciones(f).centro);
            if d < cfg.umbral_formacion
                formaciones(f).miembros(end+1) = i;
                estado{i} = 'en_formacion';
                grupo_id(i) = f;
                pos = reposicionar_formacion(pos, formaciones(f).centro, formaciones(f).miembros, cfg.rP);
                if numel(formaciones(f).miembros) >= cfg.max_miembros_formacion, break; end
            end
        end
    end

    % 3. detectar NUEVAS formaciones (parejas) entre los libres restantes
    for i = 1:n
        if ~strcmp(estado{i}, 'libre'), continue; end
        for j = i+1:n
            if ~strcmp(estado{j}, 'libre'), continue; end
            d = norm(pos(i,:) - pos(j,:));
            if d < cfg.umbral_formacion
                centro = (pos(i,:) + pos(j,:)) / 2;
                nueva.activo = true;
                nueva.centro = centro;
                nueva.miembros = [i, j];
                nueva.tiempo_restante = cfg.duracion_formacion;
                if isempty(formaciones)
                    formaciones = nueva;
                else
                    formaciones(end+1) = nueva;
                end
                idx = numel(formaciones);
                estado{i} = 'en_formacion'; estado{j} = 'en_formacion';
                grupo_id(i) = idx; grupo_id(j) = idx;
                pos = reposicionar_formacion(pos, centro, [i, j], cfg.rP);
                break;
            end
        end
    end

    % 4. avanzar el reloj de las formaciones activas y disolver las vencidas
    for f = 1:numel(formaciones)
        if ~formaciones(f).activo, continue; end
        formaciones(f).tiempo_restante = formaciones(f).tiempo_restante - 1;
        if formaciones(f).tiempo_restante <= 0
            for m = formaciones(f).miembros
                estado{m} = 'libre';
                grupo_id(m) = 0;
                heading(m) = rand*2*pi;
            end
            formaciones(f).activo = false;
        end
    end
end
{% endhighlight %}

Los miembros de una formación se reparten de forma pareja sobre el anillo P-space, mirando
hacia el centro (el O-space):

{% highlight matlab %}
function pos = reposicionar_formacion(pos, centro, miembros, rP)
    k = numel(miembros);
    for idx = 1:k
        ang = 2*pi*(idx-1)/k;
        pos(miembros(idx), :) = centro + rP*[cos(ang), sin(ang)];
    end
end
{% endhighlight %}

## Dibujo de la F-formation (O/P/R-space)

{% highlight matlab %}
function dibujarZonasFormacion(centro, rO, rP, rR)
    th = linspace(0, 2*pi, 60);
    fill(centro(1)+rR*cos(th), centro(2)+rR*sin(th), [0.80 0.85 1.00], 'EdgeColor', 'none', 'FaceAlpha', 0.5);
    fill(centro(1)+rP*cos(th), centro(2)+rP*sin(th), [0.55 0.65 0.95], 'EdgeColor', 'none', 'FaceAlpha', 0.5);
    plot(centro(1)+rO*cos(th), centro(2)+rO*sin(th), 'b--', 'LineWidth', 1);
end
{% endhighlight %}

## Parámetros usados

| Parámetro | Valor | Significado |
|---|---|---|
| `rO` | 0.40 m | Radio O-space (núcleo vacío) |
| `rP` | 0.90 m | Radio P-space (donde se paran los miembros) |
| `rR` | 1.60 m | Radio R-space (límite exterior del grupo) |
| `umbral_formacion` | 1.30 m | Distancia a la que se dispara/absorbe una formación |
| `duracion_formacion` | 40 pasos (~8 s) | Cuánto dura activa una formación |
| `max_miembros_formacion` | 4 | Tamaño máximo de un grupo |

## Resultado de ejecución

{% highlight text %}
===== ACTIVIDAD 2: Dinamica de grupos y F-formations =====
  ... paso 50/250 | formaciones formadas hasta ahora: 1
  ... paso 100/250 | formaciones formadas hasta ahora: 1
  ... paso 150/250 | formaciones formadas hasta ahora: 2
  ... paso 200/250 | formaciones formadas hasta ahora: 3
  ... paso 250/250 | formaciones formadas hasta ahora: 4
Actividad 2 completada (250 pasos, 4 F-formations en total). Video: videos/actividad2_f_formations.mp4
{% endhighlight %}

En 250 pasos de simulación se formaron y disolvieron 4 F-formations distintas entre los 4
humanos, con crecimiento dinámico de grupo cuando corresponde.

---

[&larr; Actividad 1](../actividad1/) &nbsp;&nbsp; [Actividad 3 — Navegación con Evasión &rarr;](../actividad3/){: .btn .btn-outline }
