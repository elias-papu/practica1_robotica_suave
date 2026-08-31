# Proxémica y Navegación Robótica
 
Práctica de Laboratorio — Robótica
Universidad Iberoamericana, Ciudad de México — Departamento de Estudios en Ingeniería para la Innovación (DEII)
 
**Estudiante:** Elías Santiago Jiménez Hernández
**Carrera:** Ingeniería en Inteligencia Artificial — 9no semestre
**Profesor:** Dr. Alexandro López González
 
---
 
## Descripción
 
Simulaciones y algoritmos de navegación robótica en MATLAB que aplican conceptos de
proxémica humana: espacios de Kendon, fronteras de Hall y F-formations. El proyecto cubre
las 5 actividades de la práctica:
 
1. **Espacio Individual** — un humano camina al azar; se dibujan por separado su espacio de
   Kendon (elipse orientada) y sus fronteras de Hall (círculos simétricos).
2. **Dinámica de Grupos y F-formations** — 4 humanos forman y disuelven F-formations
   (O-space, P-space, R-space) dinámicamente, hasta 4 miembros por grupo.
3. **Navegación con Evasión Proxémica** — un robot navega esquina a esquina evitando a un
   humano, usando lógica difusa combinada con un campo potencial.
4. **Interacción Robot-Humano Frontal** — el robot se aproxima de frente al humano usando el
   espacio de Kendon como referencia geométrica.
5. **Navegación Compleja Multiagente** — la misma navegación evasiva, ahora entre 4 humanos
   que además pueden formar F-formations.
Documentación completa, con explicación de cada actividad, fragmentos de código y resultados
de ejecución: **[sitio publicado en GitHub Pages](#publicar-el-sitio)**.
 
## Estructura del repositorio
 
```
├── practica1_completa.m       Script MATLAB con las 5 actividades
├── videos/                    Videos generados por el script (un .mp4 por actividad)
├── index.md                   Portada del sitio de documentación
├── actividad1.md … actividad5.md
├── comparativa.md             Actividad 3 vs. Actividad 5
├── codigo-fuente.md           Listado completo del código
├── _config.yml                Configuración de Jekyll / Just the Docs
├── Gemfile
├── _includes/                 head y footer personalizados
├── assets/css/custom.css      Tema visual IBERO
└── .github/workflows/deploy.yml   Despliegue automático a GitHub Pages
```
 
## Requisitos
 
- MATLAB con **Fuzzy Logic Toolbox**
- Ruby + Jekyll (solo si se quiere correr el sitio localmente; no es necesario para MATLAB)
## Cómo correr la simulación
 
1. Abre `practica1_completa.m` en MATLAB.
2. Ajusta la línea `correr = [1 2 3 4 5];` si quieres correr solo algunas actividades.
3. Ejecuta el script. Se generan las animaciones en pantalla y, automáticamente, un video
   `.mp4` por actividad dentro de la carpeta `videos/`.
## Publicar el sitio
 
1. Sube todo el contenido de este repositorio a GitHub (repo público).
2. En **Settings → Pages**, en "Source" selecciona **GitHub Actions**.
3. Ve a la pestaña **Actions** y espera a que termine el workflow *Deploy Jekyll to GitHub
   Pages* (1–2 minutos).
4. El sitio queda disponible en `https://<tu-usuario>.github.io/<nombre-del-repo>/`.
> Si el nombre del repositorio no es `practica-proxemica-robotica`, actualiza el valor de
> `baseurl` en `_config.yml` para que coincida exactamente con el nombre del repo.
 
## Notas de diseño
 
- **No se usa Reinforcement Learning.** El controlador es de lógica difusa (reglas fijas, no
  aprende), combinado con un campo potencial. Se optó por este enfoque porque el enunciado lo
  permite explícitamente y el Reinforcement Learning Toolbox presentó incompatibilidades de
  sintaxis entre versiones de MATLAB durante el desarrollo.
- Las violaciones de zona de peligro en la Actividad 5 (multiagente) no son un error: los
  humanos se mueven de forma aleatoria e independiente del robot, así que algunos encuentros
  cercanos son estadísticamente inevitables. Ver el análisis completo en `comparativa.md`.
 
