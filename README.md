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


----------------------------------------------------------------------------------------------------------------------------

Implementaciones de instrucciones:

# Caso 1

## Descripción
Testeo del conjunto de instrucciones tipo I y tipo L que operan con constantes inmediatas
sin tocar memoria: `ADDI`, `LUI`, `ORI`, `ANDI` y `XORI` (con variantes `.H`). El objetivo es
verificar la extensión de signo en `ADDI`, la construcción de constantes de 32 bits combinando
`LUI`+`ORI`, y el comportamiento de `ANDI`/`ANDI.H`, dado que el manual RTM32 advierte
explícitamente (nota al pie 2, sección 1.2) que existe un bug conocido en esta última.

## Instrucciones
- ADDI
- LUI
- ORI / ORI.H
- ANDI / ANDI.H
- XORI

## Precondiciones
- `reset` del sistema (PC queda en `0xF0000000`, todos los registros en 0).
- No fue necesario precargar ningún registro con datos ya que todas las instrucciones
  usan `$0` (siempre 0) como registro fuente en la primera operación de la cadena.
- Se codificaron las instrucciones a mano en hexadecimal siguiendo los formatos I y L descriptos en la sección 1.2 del
  manual, y se inyectaron directamente en memoria con `set [dirección] palabra`.

## Code
Reposicionamiento de PC a zona escribible:
```
set pc 0x00000000
```
Inyección de instrucciones en memoria:
```
set [0x00000000] 0x0804000A   ; ADDI $2, $0, 10
set [0x00000004] 0x0807FFFB   ; ADDI $3, $0, -5
set [0x00000008] 0x38081234   ; LUI  $4, 0x1234
set [0x0000000C] 0x29085678   ; ORI  $4, $4, 0x5678
set [0x00000010] 0x210A00FF   ; ANDI $5, $4, 0x00FF
set [0x00000014] 0x310CFFFF   ; XORI $6, $4, 0xFFFF
set [0x00000018] 0x210FFFFF   ; ANDI.H $7, $4, 0xFFFF
```
Ejecución:
```
step 7
r
```

## Postcondiciones
Se inspeccionó el banco de registros con `r` luego de ejecutar las 7 instrucciones y se
comparó contra el valor esperado calculado manualmente:

| Registro | Valor esperado | Motivo |
|---|---|---|
| R2 | 0x0000000A | 0 + 10 |
| R3 | 0xFFFFFFFB | 0 + (-5), verifica extensión de signo de `ADDI` |
| R4 | 0x12345678 | `LUI` carga 0x1234 en MSB, `ORI` completa LSB con 0x5678 |
| R5 | 0x00000078 | 0x12345678 & 0x000000FF (verifica `ANDI` con h=0) |
| R6 | 0x1234A987 | 0x12345678 ^ 0x0000FFFF (verifica `XORI` con h=0) |
| R7 | 0x12340000 | 0x12345678 & 0xFFFF0000 (verifica `ANDI.H` con h=1) |
| PC | 0x0000001C | 0x00000000 + 4*7 |

### Resultado real (output de `r` tras `step 7`)
```
R[ 2]: 0x0000000A
R[ 3]: 0xFFFFFFFB
R[ 4]: 0x12345678
R[ 5]: 0x00000078
R[ 6]: 0x1234A987
R[ 7]: 0x12340000
PC   : 0x0000001C
```
Todos los valores coinciden exactamente con lo esperado.

## Conclusiones
**Anduvo.** Las 7 instrucciones (`ADDI` x2, `LUI`, `ORI`, `ANDI`, `XORI`, `ANDI.H`)
produjeron el resultado esperado según la especificación del manual:
- `ADDI` extiende signo correctamente (R3 = 0xFFFFFFFB con imm = -5).
- `LUI` + `ORI` permiten construir una constante arbitraria de 32 bits (R4 = 0x12345678).
- `ANDI` (h=0) y `ANDI.H` (h=1) enmascararon correctamente la mitad baja y alta
  respectivamente (R5 y R7), **sin reproducir el bug** que el manual advierte que existe
  en esta instrucción (nota al pie 2, sección 1.2).
- `XORI` invirtió correctamente los bits indicados por la máscara (R6).

**Nota sobre el bug de ANDI:** dado que con máscaras simples (`0x00FF`, `0xFFFF`) el
resultado fue correcto, se sospecha que el bug podría estar en algún caso más específico
no cubierto por este test, por ejemplo:
- Un `ims` con el bit más significativo en 1 (posible confusión con extensión de signo en
  vez de extensión con ceros, ya que la operación especifica `ZE`/`ZC`, no `SE`).
- Una combinación particular del bit `h`.
Se recomienda un Caso adicional con `ANDI $x, $y, 0xFFFF` (h=0) sobre un registro con el
bit 15 en 1, para descartar que el bug sea una extensión de signo indebida.

------------------------------------------------------------------------------------------------------------------------------

# Caso 2

## Descripción
Testeo de las instrucciones tipo R aritmético-lógicas registro-registro: `ADD`, `SUB`,
`AND`, `OR`, `XOR`, `NOR`, `SLT` y `SLTU`. El objetivo principal es verificar las
operaciones básicas y, en particular, confirmar que `SLT` (signed) y `SLTU` (unsigned)
interpretan el mismo patrón de bits de forma distinta según haya o no signo, usando un
valor cuyo bit más significativo está en 1 (`0xFFFFFFFF`, que es -1 en complemento a dos
o 4294967295 sin signo).

## Instrucciones
- ADD, SUB
- AND, OR, XOR, NOR
- SLT, SLTU

## Precondiciones
- Se continuó directamente desde el estado final del Caso 1 (`PC = 0x0000001C`), sin
  volver a hacer `reset`, para reutilizar el mismo bloque de RAM escribible (`0x00000000`
  en adelante) sin pisar las instrucciones ya inyectadas.
- Se precargaron 4 registros fuente con `set r<n> <valor>` (sintaxis confirmada
  previamente):
  - `set r10 0x0000000F` (15)
  - `set r11 0x00000005` (5)
  - `set r12 0xFFFFFFFF` (-1 con signo / 4294967295 sin signo)
  - `set r13 0x00000001` (1, sin uso final en este caso, reservado)
- Se codificaron a mano las 8 instrucciones en formato R (opcode `00000`, diferenciadas
  por el campo `func`) y se inyectaron en memoria a partir de `0x0000001C`.

## Code
```
set r10 0x0000000F
set r11 0x00000005
set r12 0xFFFFFFFF
set r13 0x00000001

set [0x0000001C] 0x0296E01C   ; ADD  $14, $10, $11
set [0x00000020] 0x0296F01D   ; SUB  $15, $10, $11
set [0x00000024] 0x02970008   ; AND  $16, $10, $11
set [0x00000028] 0x02971009   ; OR   $17, $10, $11
set [0x0000002C] 0x0297200A   ; XOR  $18, $10, $11
set [0x00000030] 0x0297300B   ; NOR  $19, $10, $11
set [0x00000034] 0x0315400C   ; SLT  $20, $12, $10
set [0x00000038] 0x0315500D   ; SLTU $21, $12, $10

step 8
r
```

## Postcondiciones
Se inspeccionó el banco de registros con `r` luego de ejecutar las 8 instrucciones.

| Registro | Esperado | Real obtenido | Motivo |
|---|---|---|---|
| R14 | 0x00000014 | 0x00000014 | ADD: 15 + 5 = 20 |
| R15 | 0x0000000A | 0x0000000A | SUB: 15 - 5 = 10 |
| R16 | 0x00000005 | 0x00000005 | AND: 0xF & 0x5 |
| R17 | 0x0000000F | 0x0000000F | OR: 0xF \| 0x5 |
| R18 | 0x0000000A | 0x0000000A | XOR: 0xF ^ 0x5 |
| R19 | 0xFFFFFFF0 | 0xFFFFFFF0 | NOR: ~(0xF \| 0x5) |
| R20 | 0x00000001 | 0x00000001 | SLT: -1 < 15 (con signo) → true |
| R21 | 0x00000000 | 0x00000000 | SLTU: 0xFFFFFFFF no es < 15 (sin signo) → false |
| PC  | 0x0000003C | 0x0000003C | 0x1C + 4*8 |

## Conclusiones
**Anduvo.** Las 8 instrucciones dieron exactamente el resultado esperado. En particular
se confirma que `SLT` y `SLTU` leen el mismo patrón de bits en `$12` (`0xFFFFFFFF`) con
semántica distinta: con signo lo trata como -1 (menor que 15 → R20=1) y sin signo lo trata
como el valor máximo de 32 bits sin signo (no menor que 15 → R21=0). No se detectaron
anomalías en este grupo de instrucciones.

