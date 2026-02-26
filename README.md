# proyectoU1
---

# Importación de módulos

```python
import bpy
import math
```

* `bpy`  Es la librería de Python para controlar **Blender**.
* `math`  Se usa para cálculos matemáticos (senos, cosenos, π, radianes, etc.), necesarios para curvas y rotaciones.

---

# Función para crear materiales

```python
def crear_material(nombre, color_rgb):
```

### ¿Qué hace?

Crea un material personalizado con un color específico.

### Paso a paso:

```python
mat = bpy.data.materials.new(name=nombre)
```

Crea un nuevo material.

```python
mat.use_nodes = True
bsdf = mat.node_tree.nodes["Principled BSDF"]
```

Activa los nodos y obtiene el shader principal.

```python
bsdf.inputs['Base Color'].default_value = (*color_rgb, 1.0)
```

Asigna el color base (RGB + alfa).

```python
bsdf.inputs['Roughness'].default_value = 0.7
```

Hace el material más mate (menos brillante).

### Parte interesante 

```python
if color_rgb[0] > 0.5 or color_rgb[2] > 0.5:
```

Si el color tiene mucho rojo o azul:

```python
bsdf.inputs['Emission Color'].default_value = (*color_rgb, 1.0)
bsdf.inputs['Emission Strength'].default_value = 0.5
```

Hace que el material emita luz (efecto neón suave).

---

# Función principal: `generar_escenario()`

Esta es la función que construye todo el entorno 3D.

---

## 3.1 Limpiar la escena

```python
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
```

Elimina todo lo que haya en la escena antes de empezar.

---

## 3.2 Crear materiales

```python
mat_pared_a = crear_material("ParedOscura", (0.05, 0.05, 0.05))
mat_pared_b = crear_material("ParedDetalle", (0.0, 0.4, 1.0))
```

* `ParedOscura`  Gris casi negro.
* `ParedDetalle`  Azul eléctrico con emisión.

---

# Parámetros del escenario

```python
largo_pasillo = 10
ancho_pasillo = 4
radio_curva = 12
```

* Largo del pasillo recto.
* Ancho entre paredes.
* Radio de la curva.

---

# Tramo Recto del Pasillo

```python
for i in range(largo_pasillo):
```

Se generan cubos alineados en el eje Y.

### Pared izquierda

```python
bpy.ops.mesh.primitive_cube_add(location=(-ancho_pasillo, i * 2, 1))
```

Coloca cubos cada 2 unidades.

Alterna materiales:

```python
if i % 2 == 0:
```

Si es par → oscuro
Si es impar azul y más alto (`scale.z = 1.5`)

### Pared derecha

Siempre usa material oscuro.

---

# Tramo Curvo

Aquí se forma una curva de 90° usando trigonometría.

```python
angulo = math.pi - (j * (math.pi / 2) / largo_pasillo)
```

Divide la curva en segmentos pequeños.

Se calculan posiciones con:

```python
x = cx + radio * cos(angulo)
y = cy + radio * sin(angulo)
```

Se crean dos paredes:

* Izquierda (radio mayor)
* Derecha (radio menor)

También se rotan los cubos para que sigan la curva.

---

# Suelo

```python
bpy.ops.mesh.primitive_plane_add(...)
```

Crea un plano grande debajo de todo.

Se escala para cubrir todo el escenario.

---

# Iluminación

```python
bpy.ops.object.light_add(type='SUN')
```

Se añade una luz tipo sol.

```python
sun.data.energy = 2
```

Controla la intensidad.

---

# Cámara

```python
bpy.ops.object.camera_add()
```

Se crea la cámara y se define como activa.

---

# Crear el camino de la cámara (Cam_Path)

Se crea una curva 3D:

```python
curve_data = bpy.data.curves.new(...)
```

Se le agregan puntos:

```python
puntos_camino = [(0, -6, 1.5), (0, cy, 1.5)]
```

Luego se generan puntos para la parte curva usando trigonometría.

Esta curva será la guía que seguirá la cámara.

---

# Restricciones (Constraints)

## Empty como objetivo

```python
bpy.ops.object.empty_add(type='PLAIN_AXES')
```

Es un objeto invisible que servirá como punto hacia donde mira la cámara.

---

## Follow Path

Hace que:

* La cámara siga la curva.
* El empty también siga la curva.

---

## Track To

```python
track_to = camera.constraints.new(type='TRACK_TO')
```

Hace que la cámara siempre mire al empty.

Esto crea el efecto de movimiento cinematográfico.

---

# Animación

```python
bpy.context.scene.frame_start = 1
bpy.context.scene.frame_end = 200
```

La animación dura 200 frames.

### Inicio:

```python
follow_path.offset_factor = 0.0
```

La cámara comienza al inicio del camino.

### Final:

```python
follow_path.offset_factor = 0.95
```

La cámara avanza casi hasta el final.

Se insertan keyframes para animar el movimiento.

---

# ¿Qué hace todo el programa en conjunto?

1. Limpia la escena.
2. Genera un pasillo recto.
3. Añade una curva.
4. Crea materiales alternados (estilo futurista).
5. Coloca luz y suelo.
6. Crea una ruta.
7. Hace que la cámara recorra el pasillo animadamente.

---

# Resultado final

Una animación donde la cámara:

* Avanza por un pasillo recto.
* Gira suavemente en curva.
* Con paredes alternando colores.
* Con efecto de iluminación tipo neón.

