# Entrega 1: Estrategia Heurística

## Estrategia heurística empleada

La lógica de control se basa en un **autómata de estados** que reacciona únicamente a las lecturas de los sensores de proximidad, sin planificación global. Los pilares de la heurística son:

1. **Lectura de sensores**

   - El robot dispone de 8 sensores (`SL1–SL4` a la izquierda y `SR1–SR4` a la derecha).
   - Cada sensor devuelve una distancia (0–20 cm) o –1 si no detecta obstáculos.

2. **Avance libre (FREE_FORWARD)**

   - Si **todos** los sensores están en –1, el robot avanza a velocidad constante (vl=vr=0.8).
   - Objetivo: cubrir terreno abierto y detectar muros desde lejos.

3. **Detección frontal y giro de 90° (ROTATE_OBSTACLE_90_POSITIVE)**

   - Si los sensores frontales (`SL1` y `SR1`) detectan un obstáculo cercano (< WALL_DETECTION_DISTANCE), el robot se detiene y gira 90° a la derecha.
   - Se calcula un `target_angle` alineado al múltiplo más cercano de π/2, y se gira hasta alcanzarlo (±ANGLE_EPSILON).

4. **Seguimiento de pared (WALL_FOLLOWING)**

   - Tras el giro, si el sensor derecho medio (`SR3`) detecta una pared, entramos en wall‑following.
   - Se compara la lectura con la distancia objetivo (`WALL_FOLLOW_DISTANCE ± WALL_THRESHOLD`) y se aplican **correcciones** ajustando ligeramente las velocidades de cada rueda (por ejemplo, aumentar vr o vl) para mantenernos a la distancia deseada.

5. **Detección de “final de pasillo” y giro inverso (ROTATE_OBSTACLE_90_NEGATIVE)**

   - Cuando los sensores derechos (`SR1–SR3`) dejan de detectar pared, se acumula un contador (`off_wall_offset`).
   - Al superar un umbral (`STEPS_FILTER`), se asume que acabó el pasillo y se gira 90° a la izquierda para buscar nuevas paredes.

6. **Estado “after_turn”**

   - Tras cada giro mayor, se activa un breve modo “after_turn” en el que el robot avanza lento (vl=vr≈0.6) antes de retomar el seguimiento normal, evitando giros consecutivos prematuros.

7. **U‑turn en esquinas cerradas**
   - Si se detecta que no hay paredes a ningún lado (`SR1–SR3` y sus lecturas previas son –1), se valida un U‑turn y se aplica un giro de 180° dividido en dos de 90°.

## Resultados y conclusiones

- **Desempeño global:**  
  En los tres escenarios el controlador logró mapear la mayor parte de las paredes y obstáculos, alcanzando scores finales cercanos al 100 % en escenarios sencillos (escenario 1 y 2) y alrededor de 100 % en el laberinto complejo (escenario 3).
- **Fortalezas:**
  - Navegación estable en espacios abiertos y pasillos.
  - Detección fiable de obstáculos frontales y laterales.
- **Debilidades detectadas:**
  - Tendencia a atascarse en esquinas interiores sin suficiente giro (“corner turns”).
  - Oscilaciones frecuentes al seguir muros largos, debido a correcciones abruptas.

## Respuestas a las preguntas finales

### 1. Aprendizajes aplicados de teleoperación (semanas 2 y 3)

Al diseñar este controlador, se pudo reutilizar conceptos de la teleoperación:

- **Ciclo sensar–decidir–actuar:** Igual que en el control remoto, el robot lee sensores, calcula una acción y ejecuta, de forma iterativa.
- **Filtrado de señales:** Se empleó el suavizado de lecturas de sensores para evitar ruidos, lo que redujo giros erráticos.
- **Gestión de estados:** La experiencia definiendo modos de operación (p.ej. “manual” vs. “autónomo”) permitió estructurar mejor `state` y `sub_state`.

### 2. Otra métrica de eficiencia y su impacto

Podríamos haber usado **“tiempo para completar mapeo”** o **“distancia recorrida por unidad de área detectada”**.

- **Tiempo de mapeo:** Incentivaría al controlador a optimizar rutas y minimizar vueltas innecesarias.
- **Distancia/área:** Penalizaría largos recorridos en espacios ya mapeados, promoviendo exploración más compacta.  
  En ambos casos, el algoritmo requeriría un componente de planificación de trayectoria para priorizar zonas no exploradas.

### 3. Si el objetivo fuera llegar a un punto específico en el laberinto

a. **Mapeo incremental**

- Mientras exploramos, vamos marcando en la grilla (`Map_c.map`) qué celdas están libres u ocupadas.
- Esto nos da un modelo cada vez más completo del laberinto sin interrumpir el movimiento.

b. **Conversión a celdas discretas**

- Traducimos la posición real \((x,y)\) del robot y la del objetivo a índices de celda \((i,j)\).
- Trabajar en celdas simplifica el cálculo de rutas y evita manejar coordenadas continuas.

c. **Planificación periódica con A\***

- Cada cierto número de pasos (por ejemplo, 20) ejecutamos A\* sobre el grafo de celdas libres para obtener la ruta más corta hasta la meta.
- Esto nos asegura que el robot tenga un plan global y evite vueltas inútiles.

d. **Seguimiento reactivo del camino**

- Tomamos el siguiente nodo de la ruta y calculamos el ángulo hacia él:
  ```python
  ang_goal = atan2(Δy, Δx)
  ```
- Giramos y avanzamos hacia ese ángulo, manteniendo la detección de muros como capa de seguridad para corregir pequeñas desviaciones.

e. **Alternancia de modos**

- **Exploración pura:** si aún no hay ruta completa (pasillos no mapeados), el robot solo usa la heurística de evitar y seguir paredes.
- **Navegación dirigida:** cuando A\* encuentra un camino completo, el robot lo sigue hasta la meta.

f. **Criterio de llegada**

- Definimos un umbral (por ejemplo, 1 celda). Cuando el robot entra en la celda objetivo o queda dentro de ese radio, detenemos el movimiento y confirmamos que llegó.
- Así evitamos movimientos innecesarios después de alcanzar el punto.

Esto nos ofrece una forma de incluir planificación global al enfoque puramente reactivo (evitar choques).
