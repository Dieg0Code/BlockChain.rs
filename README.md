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