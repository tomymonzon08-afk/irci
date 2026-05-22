# instruccion_beq

BRANCH IF EQUAL

La instrucción beq es una instruccion de salto condicional.
Este toma dos registros y evalua si son iguales. En caso de ser iguales, se realiza un salto indicado por el inmediato. En caso contrario, continua con la siguiente instrucción.

Su sintaxis es:

beq $rs, $rt, inmediato

Se usa para implementar estructuras de control como if, while y for en lenguajes de alto nivel. Por ejemplo, un if (a == b) en C se traduce naturalmente a un beq.

CIRCUITO: 

El circuito tiene tres funciones principales: comparar registros, calcular el target address, y decidir si saltar. 

Tomamos un input de 32 bits para representar la instrucción. Luego dividimos con el splitter los bits del opcode, rt, rs y el offset.

Luego utiilizamos dos RAM's para que actuen como el Register File y lean las entradas rs y rt. 

Luego concetamos las salidas de los registros a la ALU configurada para restar los dos valores. 

Usamos un sub circuito "Zero" que devuelve 1 si el valor que salio de la ALU es distinto de cero o 0 si el valor que salio de la ALU es igual a cero. Si el resultado es igual a cero, significa que los valores en los registros son iguales, indicando que se debe realizar el salto. 

Agregamos un AND conectando el output de ese subcircuito Zero y una constante que representa la señal branch (Esta señal le indica al hardware que debe actualizar el Program Counter (PC) basándose en el resultado de la ALU para cambiar el flujo del programa), como la instrucción es beq, siempre esta en 1.

Faltaría realizar en el circuito el calculo del siguiente salto, del cual busqué y no encontré mucho. Y lo que encontré tampoco se da a entender bien. Profe si me haces las correciones y me podes explicar como funciona eso me vendría bien. Gracias de antemano.
