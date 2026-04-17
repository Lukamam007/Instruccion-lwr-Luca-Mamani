# Instruccion-lwr-Luca-Mamani

Este módulo simula en CircuitVerse el datapath para la instrucción Load Word Relative (LWR), cuya función es rescatar un dato de la memoria sumándole un desplazamiento al Program Counter. La lógica matemática que sigue es: $rd = Mem[PC + offset]$.

Para implementarlo, la instrucción completa de 16 bits ingresa primero a un **Splitter** que la divide respetando nuestro ISA: 8 bits para el *offset*, 4 para el registro destino (*rd*) y 4 para el código de operación (*Codop*). Como el resto de la arquitectura opera en 16 bits, el offset de 8 bits debe pasar por un **extensor de signo**. Este mecanismo copia el bit más significativo para rellenar los espacios faltantes, permitiéndonos calcular saltos de memoria tanto hacia adelante como hacia atrás sin generar errores de tamaño (BitWidth Error).

Una vez adaptados los tamaños, el offset extendido y el valor actual del PC ingresan a un **Sumador Completo de 16 bits**. Para garantizar un cálculo limpio y no arrastrar basura de otras operaciones, el acarreo de entrada ($C_{in}$) del sumador se fijó en `0`. La salida de esta suma nos da la dirección absoluta buscada, por lo que se conecta directamente al pin `Address` de la **memoria RAM**.

Un detalle crítico en esta etapa es la protección de la memoria: el pin de habilitación de escritura (`Write Enable`) de la RAM se clavó en un `0` lógico. Como `LWR` es exclusivamente una instrucción de lectura, esto evita que el procesador sobrescriba o corrompa datos accidentalmente. 

Finalmente, la RAM expone el valor solicitado a través de su pin `Data Out`. Si bien en el procesador terminado este bus viaja hacia el Banco de Registros (usando los 4 bits del *rd* para elegir el destino), en este módulo de prueba se conectó a un **Display Hexadecimal de 16 bits**. Esto permite aislar el circuito, alterar los inputs manualmente y verificar en tiempo real que la extracción de datos funciona a la perfección.
