# instruccion_beq

BRANCH IF EQUAL

La instrucción beq es una instruccion de salto condicional.
Este toma dos registros y evalua si son iguales. En caso de ser iguales, se realiza un salto indicado por el inmediato. En caso contrario, continua con la siguiente instrucción.

Su sintaxis es:

beq $rs, $rt, inmediato

Se usa para implementar estructuras de control como if, while y for en lenguajes de alto nivel. Por ejemplo, un if (a == b) en C se traduce naturalmente a un beq.
