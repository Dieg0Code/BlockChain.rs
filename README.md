# Blockchain.rs

Blockchain implementation in Rust :crab:

## Block Hashing:

1. Concatena juntos todos los bytes que componen un bloque
2. Genera una huella digital única del bloque (el hash) 

## Nonce

Un hash es único, como una huella digital de cierta data. Por lo tanto, para que un bloque sea valido, debemos de alguna manera cambiar los bytes que enviamos a la función. Recuerda que tan solo un pequeño cambio en el input cambia el hash resultante drásticamente. Este efecto es llamado *avalanching*.

Por supuesto, no podemos cambiar la información almacenada en un bloque. Por lo tanto, introducimos un pieza de data adicional llamada **nonce**: un arbitrario(aunque no necesariamente aleatorio) valor añadido como campo a cada bloque, y hasheado junto con la data. Ya que ha sido declarado arbitrariamente, podemos cambiarlo como nos parezca conveniente.

Puedes pensar en esto como: generar el hash correcto para un bloque es como un puzzle, y el nonce es la llave para ese puzzle. The proceso de intentar encontrar la llave es llamado **mining**.

## Mining Strategy (Algorithm)

1. Generar un nuevo nonce
2. Hash bytes(esto es un paso computacionalmente pesado)
3. Check hash against difficulty
   a. Insifficient? Go back to step 1
   b. Sufficient? Continue to step 2
4. Añade el bloque a la cadena

## Review: Mining

Que un bloque haya sido minado significa que se ha puesto una cantidad de esfuerzo en descubrir un **nonce** la "llave" que "desbloquea" el "puzzle" basado en Hash.

Minar tiene la propiedad de que es un problema difícil de solucionar, mientras que la solución es fácil de verificar.

Tiene una dificultad la cual debe irse adaptando a la cantidad de esfuerzo que es puesto por los mineros de la red para mantener la media de tiempo que toma minar un bloque.

Bitcoin ajusta su dificultad cada 2016 bloques tal que los próximos 2016 bloques debieran de tardar dos semanas en ser minados.

## Block Verification

Dada la implementación que tenemos, podemos implementar unos rudimentarios test de verificación de bloques. Estos pasos deben ser ejecutados cuando sea que recibimos un nuevo bloque de un peer.

Cada "supuesto" bloque valido tiene un nonce añadido a el, por lo que asumimos que tomo una aproximada cierta cantidad de esfuerzo en ser generado. Esta "aproximada cierta cantidad de esfuerzo" es descrita por el valor de la dificultad.

## Transaction Verification Requirements

Tenemos que protegernos contra:

- Overspending (de donde salio el dinero adicional?)
- Double-spending (esta disponible el dinero?)
- Impersonation (Quien es dueño del dinero y quien lo esta enviando?)

## The Blockchain as a "Distributed Ledger"

¿Que significa ser dueño de una moneda?

| Block 123 | Block 124 | Block 125 | Block 126 |
| :-------: | :-------: | :-------: | :-------: |
| Jaime -> Andrew(15) | Francis -> Chris(34) | **alice -> Bob(12)**  | Chirs -> Zach(3)   |
| Chirs -> Alice(50)  | Michiko -> Bob(7)    | Zach -> Jaime(2)      | Zach -> Chris(2)   |
|                     | Terrance -> Bob(87)  | Chris -> Terrance(18) | Chris -> Zach(10)  |
|                     |                      |                       | Zach -> Sophia(18) |

Distribuida significa que todos los participantes de la red tienen una copia y la capa significa que hay un histórico de todas las transacciones que han ocurrido en la red.

En el bloque 125 Alice envió 12 monedas a Bob, ¿de donde saco Alice esas monedas?, si en el bloque anterior no hay ningún registro sobre Alice.
R: Las obtuvo cuando Chris le envió 50 monedas a Alice en el bloque 123

Se puede seguir ese hilo, preguntando ¿de donde saco las moneas x o y? para ver de donde salieron las monedas, en este caso las de Chris, de esta forma se puede trackear de donde vino cada una de las monedas que hay en la red.

## Structure of a Transaction

Cada transacción tiene dos componentes importantes **inputs** y **outputs**, pero también de cierta forma los Inputs son a la vez Outputs, ¿que quiere decir esto?. Asumimos que tenemos el histórico de las transacciones el cual podemos revisar, un input seria como si Alice dijera: "en el bloque 123, Chris me envió 50 monedas, quiero usar esas 50 monedas ahora y enviar 12 de estas a Bob".
Alice diciendo "en el bloque 123 recibí 50 monedas" seria el **input**, una referencia ya que esta apuntando a un transacción especifica.
Un **output** es Alice diciendo "ahora voy a tomar 12 de esas monedas y se las voy a enviar a Bob".

**De hecho para esta transacción hipotética habrían probablemente 2 outputs, tendríamos 1 solo input de 50 monedas, pero luego 2 outputs donde 12 monedas se enviaron a Bob y las restante 38 fueron enviadas de vuelta a Alice, porque tan pronto como usamos un output como un input se considera gastado, no se puede usar de nuevo, tendríamos que hacer un nuevo output para poder gastar luego lo que queda de esas monedas**.

Tenemos dos componentes en las transacciones, una lista de **inputs** las cuales son solo **outputs** antiguos y luego una lista de nuevos **outputs** los cuales podemos usar en transacciones posteriores.

### Regular Transactions

Por ahora solo es importante que sepamos que las transacciones contienen solo 2 piezas de información importante:

- Un conjunto de inputs (los cuales son outputs sin usar de transacciones previas)
- Un conjunto de outputs (nuevos outputs los cuales pueden ser usados en futuras transacciones)

Entonces podemos calcular que:

- El valor de la transacción es: ∑inputs
- El valor de la tarifa: ∑inputs - ∑outputs (cuando enviamos una transacción deberíamos incluir un poco mas para quien mine el bloque, para incentivar a que se incluya la transacción en el siguiente bloque, ya que se necesita de un cierto trabajo para que el minero mine un bloque)

### Coinbase Transaction

Es donde todo comienza

- no requiere inputs
- produce un solo output

## Transactions Verification Requirements

### Overspending

La suma de los valores de los inputs debe ser mayor o igual a la suma de los valores de los outputs generados

I can't input 5 coins and be able to ouput 7.

### Double-Spending

Hay que asegurarse de que cada uno de los outputs nunca sea usado como un input mas de una vez.

Esto se puede hacer manteniendo una pool de outputs no gastados y rechazar cualquier transacción que trate de gastar outputs que no existen en la pool.

### Impersonation

Esto puede ser solucionado añadiendo un firma criptográfica a los outputs para verificar que están siendo gastados por su dueño.

No podemos asumir que quien sea que nos envió la transacción en la red es también la persona que creo la transacción.