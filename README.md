# Proyecto: Robot Autónomo de Mapeo

## Descripción

Este proyecto implementa un controlador para un robot autónomo que debe mapear su entorno utilizando una estrategia heurística. El robot utiliza sensores de proximidad para detectar obstáculos y tomar decisiones sobre su movimiento, con el objetivo de crear un mapa preciso del ambiente mientras evita colisiones.

## Características

- Estrategia reactiva basada en sensores
- 8 sensores de proximidad distribuidos alrededor del robot 
- Mapeo mediante rejilla de ocupación
- Tres escenarios de prueba:
  1. Obstáculo central
  2. Habitación con entrada 
  3. Laberinto complejo

## Implementación

El controlador está implementado en la clase `Controller_c` con las siguientes funcionalidades clave:

- Estado del robot y máquina de estados
- Procesamiento de lecturas de sensores
- Algoritmos de navegación y evasión de obstáculos
- Control de velocidad de las ruedas
- Mapeo del entorno

## Métricas de Evaluación

El desempeño del robot se evalúa mediante:

- Porcentaje del ambiente mapeado correctamente
- Eficiencia en el movimiento del robot
- Capacidad de evitar colisiones
- Comportamiento consistente en diferentes escenarios

## Requisitos

- Python 3.9+
- NumPy
- Matplotlib

## Uso

```python
# Crear instancia del controlador
controller = Controller_c()

# Ejecutar simulación
my_robot = Robot_c()
vl, vr = controller.update(my_robot)  # Obtener velocidades de las ruedas
```

## Autores

* Víctor Pérez De Los Ríos
* Alejandro Aristizabal

## Licencia

Este proyecto es parte del curso de MAIA:Robótica y Aprendizaje; está basado en el simulador de Paul O'Dowd y Hemma Philamore.