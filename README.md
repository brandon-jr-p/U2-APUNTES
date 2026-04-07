# **Unidad 2: Graficación 2D**

## 2.1. Transformación Bidimensional
Las transformaciones son operaciones que permiten alterar la posición, el tamaño o la orientación de un objeto en un plano bidimensional. Matemáticamente, un punto _P(x, y)_ se convierte en un nuevo punto _P'(x', y')_.

<img width="304" height="234" alt="image" src="https://github.com/user-attachments/assets/b39c70a3-daf7-4713-8eb9-db1bfb35dfcb" />

## 2.1.1. Traslación
Consiste en desplazar un objeto a una nueva posición. Se logra sumando distancias de traslación ($t_x, t_y$) a las coordenadas originales:
- $x' = x + t_x$
- $y' = y + t_y$

<img width="332" height="234" alt="image" src="https://github.com/user-attachments/assets/e1e55777-8ee4-4bd8-b59e-0caf1065de3a" />

## 2.1.2. Escalamiento
Altera el tamaño de un objeto. Se multiplica cada coordenada por un factor de escala ($s_x, s_y$):
- $x' = x \cdot s_x$
- $y' = y \cdot s_y$
- Nota: Si $s_x = s_y$, el escalamiento es uniforme; de lo contrario, se distorsiona la proporción.

<img width="244" height="170" alt="image" src="https://github.com/user-attachments/assets/59602389-d1db-4cdd-8515-c42a1197d487" />

## 2.1.3. Rotación
Gira un objeto alrededor de un punto (usualmente el origen) con un ángulo $\theta$. Las fórmulas trigonométricas son:
- $x' = x \cos(\theta) - y \sin(\theta)$
- $y' = x \sin(\theta) + y \cos(\theta)$

<img width="293" height="234" alt="image" src="https://github.com/user-attachments/assets/3f516492-f336-4e79-861e-e5fbdcef8e67" />

## 2.1.4. Sesgado (Shearing)
Es una transformación que deforma la forma de un objeto como si se deslizara en capas.
- Sesgado en X: $x' = x + sh_x \cdot y$
- Sesgado en Y: $y' = y + sh_y \cdot x$

<img width="282" height="275" alt="image" src="https://github.com/user-attachments/assets/90a40c56-9fbf-47d9-a8b0-93329e2f0302" />

## 2.2. Representación matricial de las transformaciones
Para realizar secuencias de transformaciones de forma eficiente, se utilizan coordenadas homogéneas. Esto permite representar todas las operaciones (incluyendo la traslación) como multiplicaciones de matrices de $3 \times 3$.
La forma general es: 

<img width="171" height="97" alt="image" src="https://github.com/user-attachments/assets/96226d4c-2c48-4a60-bfe4-362b832df4f1" />

- Matriz de Traslación:
  
<img width="126" height="96" alt="image" src="https://github.com/user-attachments/assets/d11ccc00-95e0-4bf9-96e8-d2e7406e6ed8" />

- Matriz de Escalamiento:

<img width="130" height="97" alt="image" src="https://github.com/user-attachments/assets/95e762b3-aa6e-4c42-801f-3177cf44fa9d" />

- Matriz de Rotación:

<img width="203" height="92" alt="image" src="https://github.com/user-attachments/assets/8786c338-608d-4827-a092-a4d73a1a739e" />

Ejercicio de Control (Teclas de Dirección):
En un entorno de programación, puedes capturar el evento de teclado. Si se presiona Flecha Derecha, incrementas $t_x$ en la matriz de traslación del objeto; si es Flecha Arriba, decrementas $t_y$ (dependiendo del origen del eje Y). Esto demuestra la aplicación en tiempo real de la traslación.

https://github.com/user-attachments/assets/f1041acc-52c1-4ab8-bbb8-411b5f28f767

Codigo: 

```
import bpy

class ModalMoveOperator(bpy.types.Operator):
    """Mover objeto con flechas: 4 direcciones"""
    bl_idname = "object.modal_move_operator"
    bl_label = "Tiburon Controller"

    def modal(self, context, event):
        # 1. Intentamos buscar por nombre, si no, usamos el objeto activo
        obj = bpy.data.objects.get("Tiburon") 
        if not obj:
            obj = context.active_object

        # Si aún así no hay nada seleccionado, cancelamos
        if not obj:
            self.report({'WARNING'}, "No hay objeto seleccionado ni se encontró 'Tiburon'")
            return {'FINISHED'}

        # 2. Lógica de movimiento
        if event.value == 'PRESS':
            # Movimiento Horizontal (Eje X)
            if event.type == 'LEFT_ARROW':
                obj.location.x -= 0.5
            elif event.type == 'RIGHT_ARROW':
                obj.location.x += 0.5
            
            # Movimiento Vertical (Eje Z para altura, o Y para profundidad)
            elif event.type == 'UP_ARROW':
                obj.location.z += 0.5  # Sube
            elif event.type == 'DOWN_ARROW':
                obj.location.z -= 0.5  # Baja

            # 3. Salida con ESC o ENTER
            elif event.type in {'ESC', 'RET'}:
                print("Control de Tiburon finalizado.")
                return {'FINISHED'}

        return {'RUNNING_MODAL'}

    def invoke(self, context, event):
        context.window_manager.modal_handler_add(self)
        print("--- CONTROL ACTIVO ---")
        print("Usa las flechas para mover a Tiburon. ESC para terminar.")
        return {'RUNNING_MODAL'}

# Registro del script
def register():
    bpy.utils.register_class(ModalMoveOperator)

if __name__ == "__main__":
    register()
    # Ejecuta el operador inmediatamente al dar a "Run Script"
    bpy.ops.object.modal_move_operator('INVOKE_DEFAULT')
```
## 2.3. Trazo de líneas curvas
A diferencia de las líneas rectas, las curvas requieren funciones polinómicas para definir trayectorias suaves.

<img width="354" height="234" alt="image" src="https://github.com/user-attachments/assets/3e3ae119-514e-4b82-9b7d-3487bbb0c7b1" />

## 2.3.1. Bézier
Se definen mediante puntos de control. La curva no pasa necesariamente por los puntos intermedios, sino que es "atraída" por ellos. Una curva de grado $n$ tiene $n+1$ puntos. La fórmula se basa en los polinomios de Bernstein.

<img width="338" height="234" alt="image" src="https://github.com/user-attachments/assets/3624daf0-bbe6-420f-8c7b-de5f7948327b" />

## 2.3.2. B-spline
Son una generalización de las curvas de Bézier. Ofrecen control local: mover un punto de control solo afecta a una sección de la curva, no a toda la trayectoria. Esto las hace ideales para el diseño industrial y modelado complejo.

<img width="418" height="175" alt="image" src="https://github.com/user-attachments/assets/3cce21e4-4e54-4409-ae23-04ccb9d64c75" />

Ejercicio Dibujo de Animación:
Puedes crear una animación donde un objeto siga la trayectoria de una curva de Bézier. Calculando el punto $P(t)$ de la curva para valores de $t$ entre $0$ y $1$ en cada frame.

https://github.com/user-attachments/assets/a11b828d-fc0e-43db-a417-4aaca1d7d754

## 2.4. Fractales
Los fractales son estructuras geométricas que se caracterizan por la auto-similitud (su estructura se repite a diferentes escalas) y su complejidad infinita.

- Fractales Geométricos: Como el Conjunto de Mandelbrot o el Copo de Nieve de Koch.
- Aplicación: Se usan en computación para generar paisajes naturales (montañas, nubes, árboles) de forma procedural.
  
<img width="291" height="234" alt="image" src="https://github.com/user-attachments/assets/1e9717c6-5ac5-409b-9373-f4b939f25f1d" />

## 2.5. Uso y creación de fuentes de texto
En graficación 2D, el texto puede tratarse de dos formas:

1. Mapas de bits (Bitmap): Cada carácter es una matriz de píxeles. Son rápidos pero pierden calidad al escalarse.
2. Fuentes de contorno (Vectores): Los caracteres se definen mediante curvas de Bézier y líneas (ej. TrueType, OpenType). Esto permite escalarlos infinitamente sin pérdida de nitidez.

<img width="366" height="191" alt="image" src="https://github.com/user-attachments/assets/8ace02d5-dd6b-4ec1-952c-2307960ce102" />

## Referencias Bibliográficas

- Hearn, D., & Baker, M. P. (2006). Gráficas por computadora con OpenGL (3ra ed.). Pearson Educación.
- Hughes, J. F., Van Dam, A., McGuire, M., Sklar, D. F., Foley, J. D., Feiner, S. K., & Akeley, K. (2013). Computer Graphics: Principles and Practice (3ra ed.). Addison-Wesley Professional.
- Shirley, P., & Marschner, S. (2009). Fundamentals of Computer Graphics (3ra ed.). A K Peters/CRC Press.
- Watt, A. (2000). 3D Computer Graphics (3ra ed.). Addison-Wesley.
  
