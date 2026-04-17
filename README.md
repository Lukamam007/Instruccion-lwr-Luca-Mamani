# Instruccion-lwr-Luca-Mamani

# El objetivo de este módulo es hacer funcionar la instrucción `LWR`. Básicamente, lo que hace esta instrucción es tomar el valor actual del Program Counter (PC), sumarle un salto o desplazamiento (*offset*), ir a buscar el dato que está guardado en esa dirección de memoria, y guardarlo en un registro específico (`rd`). 

La lógica matemática que sigue es: `rd = Mem[PC + offset]`

---

## ⚙️ Desglose de la implementación en CircuitVerse

Para armar el circuito, dividí el problema en las siguientes etapas clave:

### 1. Desarmar la instrucción (Decodificación)
Agarré el bus principal de la instrucción (16 bits) y lo pasé por un **Splitter** para separar la señal en tres grupos, respetando el formato de nuestro ISA:
- **Bits 0 al 7:** `Offset` (el salto).
- **Bits 8 al 11:** `rd` (el registro destino).
- **Bits 12 al 15:** `Codop` (el código de operación).

*Nota: Esto es clave porque cada pedacito de la instrucción tiene que hacer un viaje distinto dentro del procesador.*

### 2. Agrandar el offset (Extensión de signo)
El *offset* extraído tiene 8 bits, pero todo el resto del procesador (PC, sumador, memoria) trabaja con 16 bits. 
Armé un **extensor de signo** que agarra el último bit del offset (el bit 7, que indica si es positivo o negativo) y lo "fotocopia" para rellenar los 8 espacios que faltan. Usé extensión de signo y no relleno de ceros porque en la memoria podemos necesitar saltar hacia adelante o hacia atrás (complemento a 2).

### 3. Calcular a dónde ir (Unidad Aritmética)
Puse un **Sumador Completo de 16 bits**. En una entrada conecté el valor del `PC`, y en la otra el `offset` ya extendido a 16 bits. 
- **Dato clave:** A la patita del carry de entrada ($C_{in}$) le clavé un `0` lógico fijo para asegurarme de hacer una suma limpia y no arrastrar basura matemática de otras operaciones. 
- De la salida de este sumador obtenemos la dirección exacta de la memoria.

### 4. Leer la memoria sin romper nada (RAM)
Mandé el cable con el resultado de la suma derecho al pin de direcciones (`Address`) de la memoria RAM. 
- **Seguridad:** Le clavé un `0` lógico al pin de escritura (`Write Enable`). Como el `LWR` es una instrucción de lectura (*Load*), solo queremos mirar qué hay adentro. Si se colaba un `1`, corríamos el riesgo de sobreescribir y arruinar los datos.

### 5. Rescatar el dato y probarlo
Como la memoria estaba protegida en modo lectura, hizo su trabajo y escupió el dato por el pin de salida (`Data Out`). 
En el diseño final, este cable viaja hacia el Banco de Registros para guardarse. Para probar este módulo por separado, lo conecté a un **Display Hexadecimal de 16 bits**, lo que permitió jugar cambiando las entradas a mano y ver en tiempo real cómo el circuito iba a buscar correctamente los datos a la RAM.

---

## 🛠️ ¿Cómo probar este circuito?

1. Abrí el proyecto en CircuitVerse.
2. Hacé doble clic en el componente **RAM** y cargá algunos datos de prueba (ej: `1111 2222 3333` en las primeras posiciones).
3. Modificá el input del **PC** (ej: `0`).
4. Modificá el input de la **Instrucción** asegurándote de que el *offset* apunte a una de las celdas con datos.
5. Verificá que el Output final muestre el dato exacto que guardaste en esa posición de la memoria.
