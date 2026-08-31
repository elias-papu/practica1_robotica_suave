---
layout: default
title: Código Fuente
nav_order: 8
permalink: /codigo-fuente/
---

# Código Fuente
{: .no_toc }

Script MATLAB completo, documentado y estructurado
{: .fs-5 .fw-300 }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Descarga

El archivo completo está disponible en este repositorio:
[`practica1_completa.m`](../practica1_completa.m){: .btn .btn-primary }

Requiere **Fuzzy Logic Toolbox** de MATLAB. Al ejecutarlo corre las 5 actividades en
secuencia y genera un video `.mp4` por actividad en una carpeta `videos/`.

## Estructura del archivo

El script está organizado en bloques claramente delimitados:

1. Parámetros globales (`cfg`) — todas las constantes de las 5 actividades en un solo lugar.
2. Una función por actividad (`actividad1_...` a `actividad5_...`).
3. Funciones de dinámica de grupo compartidas por las Actividades 2 y 5.
4. Funciones auxiliares compartidas (controlador difuso, movimiento, dibujo, geometría de
   Kendon, grabación de video).

## Listado completo

{% highlight matlab %}
%% =======================================================================
%  PRACTICA DE LABORATORIO: PROXEMICA Y NAVEGACION ROBOTICA
%  TODAS LAS ACTIVIDADES (1 a 5) EN UN SOLO ARCHIVO
% -------------------------------------------------------------------
%  Universidad Iberoamericana | Ingenieria en Inteligencia Artificial
%  Robotica - Dr. Alexandro Lopez Gonzalez
% -------------------------------------------------------------------
%  QUE HACE ESTE ARCHIVO
%   Actividad 1: un humano camina al azar; se dibujan POR SEPARADO su
%                espacio de Kendon (elipse orientada, mas grande al
%                frente) y sus fronteras de Hall (circulos simetricos).
%   Actividad 2: 4 humanos caminan al azar; cuando se acercan forman
%                F-formations (O/P/R-space) que pueden crecer hasta 4
%                miembros, y se disuelven pasado un tiempo.
%   Actividad 3: robot navega esquina a esquina evitando al humano con
%                LOGICA DIFUSA + CAMPO POTENCIAL.
%   Actividad 4: el robot se aproxima usando el ESPACIO DE KENDON (la
%                elipse orientada) como referencia real, no un circulo.
%   Actividad 5: como la 3, pero con 4 humanos + F-formations dinamicas
%                (hasta 4 miembros); el robot suma la repulsion de
%                TODOS los obstaculos activos a la vez.
%
%  Ademas, cada actividad GUARDA UN VIDEO (.mp4) en la carpeta ./videos
%  -- ese es un entregable explicito de la practica.
%
%  IMPORTANTE - SOBRE "APRENDER":
%  La logica difusa NO APRENDE. Sus reglas son fijas por diseno. Eso es
%  correcto y esperado; lo que aprende (RL) se quito por incompatibilidad
%  de toolbox entre versiones de MATLAB, y no es necesario: la practica
%  permite explicitamente logica difusa o campos potenciales.
%
%  REQUIERE: Fuzzy Logic Toolbox
% =======================================================================

clear; clc; close all;

if ~exist('videos', 'dir'), mkdir('videos'); end

fprintf(['NOTA: Las Actividades 3 y 5 usan LOGICA DIFUSA + CAMPO POTENCIAL.\n' ...
         'Sus reglas son FIJAS: no aprenden ni mejoran de una corrida a otra.\n\n']);

%% -----------------------------------------------------------------
%  0. QUE ACTIVIDADES QUIERES CORRER ESTA VEZ
% --------------------------------------------------------------------
correr = [1 2 3 4 5];

%% -----------------------------------------------------------------
%  1. PARAMETROS GLOBALES
% --------------------------------------------------------------------
cfg.L               = 10;
cfg.r_intima        = 0.45;
cfg.r_personal      = 1.20;
cfg.r_social        = 3.60;

cfg.kendon_frente   = 1.50;   % semi-eje mayor del espacio de Kendon (frente de la persona, m)
cfg.kendon_lado     = 0.60;   % semi-eje menor (lados/atras, m) -- asimetrico, a diferencia de Hall

cfg.dt              = 0.2;
cfg.v_humano        = 0.3;
cfg.v_robot         = 0.6;
cfg.dist_llegada    = 0.30;

cfg.rO              = 0.40;
cfg.rP              = 0.90;
cfg.rR              = 1.60;

cfg.umbral_formacion   = 1.30;
cfg.duracion_formacion = 40;
cfg.max_miembros_formacion = 4;

cfg.semilla = 42;

fis = crear_fis_normalizado();

%% -----------------------------------------------------------------
%  2. EJECUTAR LAS ACTIVIDADES SELECCIONADAS
% --------------------------------------------------------------------
if ismember(1, correr)
    fprintf('\n===== ACTIVIDAD 1: Espacio individual (Kendon / Hall) =====\n');
    actividad1_espacio_individual(cfg);
end

if ismember(2, correr)
    fprintf('\n===== ACTIVIDAD 2: Dinamica de grupos y F-formations =====\n');
    actividad2_grupo_formaciones(cfg);
end

resultado3 = [];
if ismember(3, correr)
    fprintf('\n===== ACTIVIDAD 3: Navegacion con evasion (agente unico) =====\n');
    resultado3 = actividad3_navegacion_evasion(cfg, fis);
end

if ismember(4, correr)
    fprintf('\n===== ACTIVIDAD 4: Interaccion robot-humano frontal =====\n');
    actividad4_interaccion_frontal(cfg);
end

resultado5 = [];
if ismember(5, correr)
    fprintf('\n===== ACTIVIDAD 5: Navegacion compleja multiagente =====\n');
    resultado5 = actividad5_navegacion_multiagente(cfg, fis);
end

if ~isempty(resultado3) && ~isempty(resultado5)
    graficar_comparacion(resultado3, resultado5);
end

fprintf('\nListo. Revisa las figuras y la carpeta ./videos (un .mp4 por actividad corrida).\n');

%% =======================================================================
%  FUNCIONES DE CADA ACTIVIDAD
% =======================================================================

function actividad1_espacio_individual(cfg)
    rng(cfg.semilla);
    pos = [cfg.L/2, cfg.L/2];
    heading = rand*2*pi;
    trayectoria = pos;
    pasos_demo = 150;

    fig = figure('Name', 'Actividad 1 - Espacio Individual (Kendon / Hall)', 'Color', 'w');
    v = abrir_video('videos/actividad1_kendon_hall.mp4');

    for k = 1:pasos_demo
        [pos, heading] = mover_aleatorio(pos, heading, cfg.v_humano, cfg.dt, cfg.L, 0.3);
        trayectoria(end+1, :) = pos; %#ok<AGROW>

        clf; hold on; axis equal; grid on;
        xlim([0 cfg.L]); ylim([0 cfg.L]);
        title(sprintf('Actividad 1 | Paso %d/%d | Rojo=fronteras de Hall (simetricas) | Azul=espacio de Kendon (orientado)', k, pasos_demo));
        xlabel('X (m)'); ylabel('Y (m)');

        dibujarZonasHall(pos, cfg.r_intima, cfg.r_personal, cfg.r_social);
        dibujarEspacioKendon(pos, heading, cfg.kendon_frente, cfg.kendon_lado);
        plot(trayectoria(:,1), trayectoria(:,2), 'k:', 'LineWidth', 1);
        plot(pos(1), pos(2), 'ko', 'MarkerFaceColor', [0.2 0.2 0.2], 'MarkerSize', 10);
        drawnow;
        escribir_frame(v);

        if mod(k, 50) == 0, fprintf('  ... paso %d/%d\n', k, pasos_demo); end
    end
    cerrar_video(v);
    fprintf('Actividad 1 completada (%d pasos). Video: videos/actividad1_kendon_hall.mp4\n', pasos_demo);
end

function actividad2_grupo_formaciones(cfg)
    rng(cfg.semilla + 1);
    n = 4;
    pos = cfg.L*0.6*rand(n,2) + cfg.L*0.2;
    heading = rand(n,1)*2*pi;
    estado = repmat({'libre'}, n, 1);
    grupo_id = zeros(n,1);
    formaciones = struct('activo', {}, 'centro', {}, 'miembros', {}, 'tiempo_restante', {});

    trayectorias = cell(n,1);
    for i = 1:n, trayectorias{i} = pos(i,:); end

    pasos_demo = 250;
    colores = lines(n);

    fig = figure('Name', 'Actividad 2 - Dinamica de Grupos y F-formations', 'Color', 'w');
    v = abrir_video('videos/actividad2_f_formations.mp4');

    for k = 1:pasos_demo
        [pos, heading, estado, grupo_id, formaciones] = actualizar_dinamica_grupo(pos, heading, estado, grupo_id, formaciones, cfg);

        for i = 1:n, trayectorias{i}(end+1,:) = pos(i,:); end %#ok<AGROW>

        clf; hold on; axis equal; grid on;
        xlim([0 cfg.L]); ylim([0 cfg.L]);
        n_activas = sum([formaciones.activo]) * ~isempty(formaciones);
        if isempty(formaciones), n_activas = 0; end
        title(sprintf('Actividad 2 | Paso %d/%d | F-formations activas: %d | Total formadas: %d', ...
            k, pasos_demo, n_activas, numel(formaciones)));
        xlabel('X (m)'); ylabel('Y (m)');

        for f = 1:numel(formaciones)
            if formaciones(f).activo
                dibujarZonasFormacion(formaciones(f).centro, cfg.rO, cfg.rP, cfg.rR);
            end
        end
        for i = 1:n
            if strcmp(estado{i}, 'libre')
                dibujarZonasHall(pos(i,:), cfg.r_intima, cfg.r_personal, cfg.r_social);
            end
        end
        for i = 1:n
            plot(trayectorias{i}(:,1), trayectorias{i}(:,2), ':', 'Color', colores(i,:), 'LineWidth', 0.8);
            plot(pos(i,1), pos(i,2), 'o', 'MarkerFaceColor', colores(i,:), 'MarkerEdgeColor', 'k', 'MarkerSize', 10);
        end
        drawnow;
        escribir_frame(v);

        if mod(k, 50) == 0, fprintf('  ... paso %d/%d | formaciones formadas hasta ahora: %d\n', k, pasos_demo, numel(formaciones)); end
    end
    cerrar_video(v);
    fprintf('Actividad 2 completada (%d pasos, %d F-formations en total). Video: videos/actividad2_f_formations.mp4\n', pasos_demo, numel(formaciones));
end

function metrica = actividad3_navegacion_evasion(cfg, fis)
    rng(cfg.semilla + 2);
    pos_robot = [0.3, 0.3];
    meta = [cfg.L - 0.3, cfg.L - 0.3];
    pos_humano = [cfg.L/2, cfg.L/2];
    heading_humano = rand*2*pi;

    trayectoria_robot  = pos_robot;
    trayectoria_humano = pos_humano;
    historial_dist = [];
    historial_meta = [];
    violaciones = 0;
    llego_meta = false;
    max_pasos = 600;

    fig = figure('Name', 'Actividad 3 - Navegacion con Evasion Proxemica', 'Color', 'w');
    v = abrir_video('videos/actividad3_evasion_agente_unico.mp4');

    for paso = 1:max_pasos
        [pos_humano, heading_humano] = mover_aleatorio(pos_humano, heading_humano, cfg.v_humano, cfg.dt, cfg.L, 0.35);

        amenazas = [pos_humano, cfg.r_personal];
        [pos_robot, info] = paso_robot_evasivo(pos_robot, meta, amenazas, fis, cfg.v_robot, cfg.dt, cfg.L);

        trayectoria_robot(end+1,:)  = pos_robot;  %#ok<AGROW>
        trayectoria_humano(end+1,:) = pos_humano; %#ok<AGROW>
        historial_dist(end+1) = info.dist_amenaza; %#ok<AGROW>
        dist_meta_actual = norm(meta - pos_robot);
        historial_meta(end+1) = dist_meta_actual;  %#ok<AGROW>
        if info.margen < 0, violaciones = violaciones + 1; end

        if mod(paso, 2) == 0 || paso == 1
            clf; hold on; axis equal; grid on;
            xlim([0 cfg.L]); ylim([0 cfg.L]);
            title(sprintf('Actividad 3 | Paso %d | Dist. humano: %.2f m | Violaciones: %d', paso, info.dist_amenaza, violaciones));
            xlabel('X (m)'); ylabel('Y (m)');
            dibujarZonasHall(pos_humano, cfg.r_intima, cfg.r_personal, cfg.r_social);
            plot(trayectoria_robot(:,1), trayectoria_robot(:,2), 'b-', 'LineWidth', 1.5);
            plot(trayectoria_humano(:,1), trayectoria_humano(:,2), 'r:', 'LineWidth', 1);
            plot(pos_robot(1), pos_robot(2), 'bs', 'MarkerFaceColor', 'b', 'MarkerSize', 10);
            plot(pos_humano(1), pos_humano(2), 'ro', 'MarkerFaceColor', 'r', 'MarkerSize', 10);
            plot(meta(1), meta(2), 'g^', 'MarkerFaceColor', 'g', 'MarkerSize', 12);
            drawnow;
            escribir_frame(v);
        end
        if mod(paso, 40) == 0
            fprintf('  ... paso %d | dist. humano: %.2f m | violaciones hasta ahora: %d\n', paso, info.dist_amenaza, violaciones);
        end

        if dist_meta_actual < cfg.dist_llegada
            llego_meta = true;
            fprintf('Actividad 3: el robot llego a la meta en el paso %d (violaciones: %d).\n', paso, violaciones);
            break;
        end
    end
    cerrar_video(v);

    figure('Name', 'Actividad 3 - Analisis', 'Color', 'w');
    subplot(2,1,1);
    plot(historial_dist, 'r', 'LineWidth', 1.3); hold on;
    yline(cfg.r_intima, '--', 'Zona intima', 'Color', [0.6 0 0]);
    yline(cfg.r_personal, '--', 'Zona personal', 'Color', [0.8 0.3 0]);
    xlabel('Paso'); ylabel('Distancia al humano (m)');
    title('Actividad 3 - Distancia robot-humano'); grid on;
    subplot(2,1,2);
    plot(historial_meta, 'b', 'LineWidth', 1.3);
    xlabel('Paso'); ylabel('Distancia a la meta (m)');
    title('Actividad 3 - Progreso hacia la meta'); grid on;

    metrica = struct('nombre', 'Actividad 3 (agente unico)', 'pasos', paso, ...
        'violaciones', violaciones, 'dist_min', min(historial_dist), 'llego_meta', llego_meta);
    fprintf('Video: videos/actividad3_evasion_agente_unico.mp4\n');
end

function actividad4_interaccion_frontal(cfg)
    rng(cfg.semilla + 3);
    pos_robot = [0.3, 0.3];
    pos_humano = [cfg.L/2, cfg.L/2];
    heading_humano = rand*2*pi;
    interaccion = false;
    contador_post_interaccion = 0;
    pasos_max = 500;

    trayectoria_robot = pos_robot;

    fig = figure('Name', 'Actividad 4 - Interaccion Robot-Humano Frontal', 'Color', 'w');
    v = abrir_video('videos/actividad4_interaccion_frontal.mp4');

    for paso = 1:pasos_max
        if ~interaccion
            [pos_humano, heading_humano] = mover_aleatorio(pos_humano, heading_humano, cfg.v_humano, cfg.dt, cfg.L, 0.3);
        end

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
            fprintf('Actividad 4: interaccion frontal establecida en el paso %d.\n', paso);
        end

        trayectoria_robot(end+1,:) = pos_robot; %#ok<AGROW>

        if mod(paso, 2) == 0 || paso == 1
            clf; hold on; axis equal; grid on;
            xlim([0 cfg.L]); ylim([0 cfg.L]);
            if interaccion, estado_txt = 'INTERACCION ESTABLECIDA (robot dentro del espacio de Kendon)';
            else,           estado_txt = 'Buscando aproximacion frontal...'; end
            title(sprintf('Actividad 4 | Paso %d | %s', paso, estado_txt));
            xlabel('X (m)'); ylabel('Y (m)');

            dibujarZonasHall(pos_humano, cfg.r_intima, cfg.r_personal, cfg.r_social);
            dibujarEspacioKendon(pos_humano, heading_humano, cfg.kendon_frente, cfg.kendon_lado);
            quiver(pos_humano(1), pos_humano(2), cos(heading_humano), sin(heading_humano), 0.6, ...
                'k', 'LineWidth', 1.5, 'MaxHeadSize', 2);
            plot(trayectoria_robot(:,1), trayectoria_robot(:,2), 'b-', 'LineWidth', 1.5);
            plot(pos_robot(1), pos_robot(2), 'bs', 'MarkerFaceColor', 'b', 'MarkerSize', 10);
            plot(pos_humano(1), pos_humano(2), 'ro', 'MarkerFaceColor', 'r', 'MarkerSize', 10);
            drawnow;
            escribir_frame(v);
        end
        if ~interaccion && mod(paso, 40) == 0
            fprintf('  ... paso %d | dist. al humano: %.2f m\n', paso, norm(pos_robot-pos_humano));
        end

        if interaccion
            contador_post_interaccion = contador_post_interaccion + 1;
            if contador_post_interaccion > 30, break; end
        end
    end
    cerrar_video(v);
    if ~interaccion
        fprintf('Actividad 4: no se logro establecer la interaccion en %d pasos.\n', pasos_max);
    end
    fprintf('Video: videos/actividad4_interaccion_frontal.mp4\n');
end

function metrica = actividad5_navegacion_multiagente(cfg, fis)
    rng(cfg.semilla + 4);
    n = 4;
    pos = cfg.L*0.5*rand(n,2) + cfg.L*0.25;
    heading = rand(n,1)*2*pi;
    estado = repmat({'libre'}, n, 1);
    grupo_id = zeros(n,1);
    formaciones = struct('activo', {}, 'centro', {}, 'miembros', {}, 'tiempo_restante', {});

    pos_robot = [0.3, 0.3];
    meta = [cfg.L - 0.3, cfg.L - 0.3];
    trayectoria_robot = pos_robot;
    historial_dist = [];
    violaciones = 0;
    llego_meta = false;
    pasos_max = 700;
    colores = lines(n);

    fig = figure('Name', 'Actividad 5 - Navegacion Compleja Multiagente', 'Color', 'w');
    v = abrir_video('videos/actividad5_navegacion_multiagente.mp4');

    for paso = 1:pasos_max
        [pos, heading, estado, grupo_id, formaciones] = actualizar_dinamica_grupo(pos, heading, estado, grupo_id, formaciones, cfg);

        % ---- construir la lista de obstaculos para el robot ----
        amenazas = [];
        for f = 1:numel(formaciones)
            if formaciones(f).activo
                amenazas(end+1, :) = [formaciones(f).centro, cfg.rR]; %#ok<AGROW>
            end
        end
        for i = 1:n
            if strcmp(estado{i}, 'libre')
                amenazas(end+1, :) = [pos(i,:), cfg.r_personal]; %#ok<AGROW>
            end
        end

        [pos_robot, info] = paso_robot_evasivo(pos_robot, meta, amenazas, fis, cfg.v_robot, cfg.dt, cfg.L);
        trayectoria_robot(end+1, :) = pos_robot; %#ok<AGROW>
        historial_dist(end+1) = info.dist_amenaza; %#ok<AGROW>
        if info.margen < 0, violaciones = violaciones + 1; end

        if mod(paso, 2) == 0 || paso == 1
            clf; hold on; axis equal; grid on;
            xlim([0 cfg.L]); ylim([0 cfg.L]);
            title(sprintf('Actividad 5 | Paso %d | Amenaza mas cercana: %.2f m | Violaciones: %d', paso, info.dist_amenaza, violaciones));
            xlabel('X (m)'); ylabel('Y (m)');

            for f = 1:numel(formaciones)
                if formaciones(f).activo
                    dibujarZonasFormacion(formaciones(f).centro, cfg.rO, cfg.rP, cfg.rR);
                end
            end
            for i = 1:n
                if strcmp(estado{i}, 'libre')
                    dibujarZonasHall(pos(i,:), cfg.r_intima, cfg.r_personal, cfg.r_social);
                end
            end
            for i = 1:n
                plot(pos(i,1), pos(i,2), 'o', 'MarkerFaceColor', colores(i,:), 'MarkerEdgeColor', 'k', 'MarkerSize', 9);
            end
            plot(trayectoria_robot(:,1), trayectoria_robot(:,2), 'b-', 'LineWidth', 1.5);
            plot(pos_robot(1), pos_robot(2), 'bs', 'MarkerFaceColor', 'b', 'MarkerSize', 10);
            plot(meta(1), meta(2), 'g^', 'MarkerFaceColor', 'g', 'MarkerSize', 12);
            drawnow;
            escribir_frame(v);
        end
        if mod(paso, 40) == 0
            fprintf('  ... paso %d | amenaza mas cercana: %.2f m | violaciones hasta ahora: %d\n', paso, info.dist_amenaza, violaciones);
        end

        if norm(meta - pos_robot) < cfg.dist_llegada
            llego_meta = true;
            fprintf('Actividad 5: el robot llego a la meta en el paso %d (violaciones: %d).\n', paso, violaciones);
            break;
        end
    end
    cerrar_video(v);

    figure('Name', 'Actividad 5 - Analisis', 'Color', 'w');
    plot(historial_dist, 'm', 'LineWidth', 1.3); hold on;
    yline(cfg.r_personal, '--', 'Radio peligro individual', 'Color', [0.8 0.3 0]);
    yline(cfg.rR, '--', 'Radio peligro F-formation', 'Color', [0.2 0.2 0.8]);
    xlabel('Paso'); ylabel('Distancia a la amenaza mas cercana (m)');
    title('Actividad 5 - Distancia a la amenaza mas cercana'); grid on;

    metrica = struct('nombre', 'Actividad 5 (multiagente)', 'pasos', paso, ...
        'violaciones', violaciones, 'dist_min', min(historial_dist), 'llego_meta', llego_meta);
    fprintf('Video: videos/actividad5_navegacion_multiagente.mp4\n');
end

%% =======================================================================
%  DINAMICA DE GRUPOS / F-FORMATIONS (compartida por Actividades 2 y 5)
% =======================================================================

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

function pos = reposicionar_formacion(pos, centro, miembros, rP)
    % Reparte a los miembros de una F-formation de forma pareja sobre
    % el anillo P-space, mirando hacia el centro (el O-space).
    k = numel(miembros);
    for idx = 1:k
        ang = 2*pi*(idx-1)/k;
        pos(miembros(idx), :) = centro + rP*[cos(ang), sin(ang)];
    end
end

%% =======================================================================
%  FUNCIONES AUXILIARES COMPARTIDAS
% =======================================================================

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
        1 1 3 1 1
        1 2 2 1 1
        1 3 1 1 1
        2 1 2 1 1
        2 2 1 1 1
        2 3 1 1 1
        3 1 1 1 1
        3 2 1 1 1
        3 3 1 1 1
    ];
    fis = addRule(fis, ruleList);
end

function desviacion_mag = evaluar_evasion(fis, dist_real, radio_peligro, angulo_abs_deg)
    dist_norm = dist_real / radio_peligro;
    desviacion_mag = evalfis(fis, [dist_norm, angulo_abs_deg]);
end

function [pos_robot_nuevo, info] = paso_robot_evasivo(pos_robot, meta, amenazas, fis, v_robot, dt, L)
    % Combina la evasion de VARIOS obstaculos mediante un PROMEDIO
    % ANGULAR PONDERADO: cada obstaculo "vota" cuanto desviarse del
    % rumbo directo a la meta (signo = hacia que lado, magnitud = que
    % tan fuerte, ambos decididos por el controlador difuso), y el
    % voto de cada obstaculo pesa segun su propia urgencia. Cuando solo
    % hay UN obstaculo, esta formula se reduce exactamente a "girar
    % signo*magnitud grados respecto al rumbo deseado" -- la misma
    % formula que ya se probo y da 0 violaciones en la Actividad 3.
    % A proposito NO se usa una suma de vectores cartesianos: esa
    % version anterior podia hacer que la atraccion a la meta y la
    % repulsion quedaran exactamente opuestas, atrapando al robot
    % oscilando contra el obstaculo (minimo local clasico de los
    % campos potenciales). El promedio angular no tiene ese problema.
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
            mejor_margen = margen;
            dist_amenaza = d;
            radio_amenaza = radio;
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

    info.dist_amenaza  = dist_amenaza;
    info.radio_amenaza = radio_amenaza;
    info.margen        = mejor_margen;
end

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

function angulo_envuelto = envolver_angulo(angulo)
    angulo_envuelto = mod(angulo + pi, 2*pi) - pi;
end

function dibujarZonasHall(centro, r1, r2, r3)
    th = linspace(0, 2*pi, 60);
    fill(centro(1)+r3*cos(th), centro(2)+r3*sin(th), [1 0.9 0.9], 'EdgeColor', 'none', 'FaceAlpha', 0.5);
    fill(centro(1)+r2*cos(th), centro(2)+r2*sin(th), [1 0.7 0.7], 'EdgeColor', 'none', 'FaceAlpha', 0.6);
    fill(centro(1)+r1*cos(th), centro(2)+r1*sin(th), [1 0.4 0.4], 'EdgeColor', 'none', 'FaceAlpha', 0.7);
end

function dibujarZonasFormacion(centro, rO, rP, rR)
    th = linspace(0, 2*pi, 60);
    fill(centro(1)+rR*cos(th), centro(2)+rR*sin(th), [0.80 0.85 1.00], 'EdgeColor', 'none', 'FaceAlpha', 0.5);
    fill(centro(1)+rP*cos(th), centro(2)+rP*sin(th), [0.55 0.65 0.95], 'EdgeColor', 'none', 'FaceAlpha', 0.5);
    plot(centro(1)+rO*cos(th), centro(2)+rO*sin(th), 'b--', 'LineWidth', 1);
end

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

function dentro = dentro_de_kendon(punto, centro, heading, a, b)
    % Prueba geometrica: verdadero/falso si "punto" cae dentro de la
    % elipse orientada del espacio de Kendon de la persona en "centro".
    d = punto - centro;
    xr =  d(1)*cos(-heading) - d(2)*sin(-heading);
    yr =  d(1)*sin(-heading) + d(2)*cos(-heading);
    dentro = (xr/a)^2 + (yr/b)^2 <= 1;
end

function graficar_comparacion(resultado3, resultado5)
    fprintf('\n===================== COMPARATIVA: Actividad 3 vs Actividad 5 =====================\n');
    fprintf('%-30s %-20s %-20s\n', 'Metrica', resultado3.nombre, resultado5.nombre);
    fprintf('%-30s %-20d %-20d\n', 'Pasos hasta la meta', resultado3.pasos, resultado5.pasos);
    fprintf('%-30s %-20d %-20d\n', 'Violaciones', resultado3.violaciones, resultado5.violaciones);
    fprintf('%-30s %-20.2f %-20.2f\n', 'Distancia minima (m)', resultado3.dist_min, resultado5.dist_min);
    fprintf('=====================================================================================\n');

    figure('Name', 'Comparativa Actividad 3 vs Actividad 5', 'Color', 'w');
    subplot(1,2,1);
    bar([resultado3.violaciones, resultado5.violaciones], 'FaceColor', [0.7 0.7 0.9]);
    set(gca, 'XTickLabel', {'Act. 3 (1 humano)', 'Act. 5 (4 humanos)'});
    ylabel('Violaciones de zona de peligro');
    title('Violaciones: agente unico vs. multiagente'); grid on;

    subplot(1,2,2);
    bar([resultado3.dist_min, resultado5.dist_min], 'FaceColor', [0.7 0.9 0.7]);
    set(gca, 'XTickLabel', {'Act. 3 (1 humano)', 'Act. 5 (4 humanos)'});
    ylabel('Distancia minima alcanzada (m)');
    title('Distancia minima: agente unico vs. multiagente'); grid on;
end

%% =======================================================================
%  GRABACION DE VIDEO (con respaldo si el perfil MPEG-4 no esta disponible)
% =======================================================================

function v = abrir_video(nombre_archivo)
    try
        v = VideoWriter(nombre_archivo, 'MPEG-4');
    catch
        [carpeta, nombre, ~] = fileparts(nombre_archivo);
        v = VideoWriter(fullfile(carpeta, [nombre '.avi']), 'Motion JPEG AVI');
    end
    v.FrameRate = 10;
    open(v);
end

function escribir_frame(v)
    frame = getframe(gcf);
    writeVideo(v, frame);
end

function cerrar_video(v)
    close(v);
end
{% endhighlight %}

---

[&larr; Comparativa y Resultados](../comparativa/) &nbsp;&nbsp; [Volver al inicio &rarr;](../){: .btn .btn-outline }
