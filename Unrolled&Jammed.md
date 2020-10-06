# Unrolled & Jammed con OpenMP

## Unrolled y Jamming son técnicas de optimización de ciclos.
* Unrolling: Es una técnica que permite reducir de forma efectiva la ejecución de un ciclo, al incrementar la variable de control y modificar la prueba de continuación y salto a la línea de ejecución. 
La forma en la que funciona es aumentando las instrucciones ejecutadas por cada iteración del ciclo. Por ejemplo, en lugar de realizar una sola instrucción, por ejemplo, en un arreglo, podríamos realizar una instrucción con el elemento al que indica la variable de control, y al elemento que indica la variable de control mas uno. Al realizar esto, por cada iteración podremos evaluar dos elementos del arreglo, cortando la cantidad de iteraciones a la mitad. 
Ejemplo: 


```c
#include <stdio.h>
#include <omp.h>
```

Una desventaja de este método es que el tiempo de ejecución de cada iteración aumenta proporcionalmente a la cantidad de instrucciones que se le añadan. Por lo tanto, puede no ser lo ideal si necesitamos que los efectos se muestren de forma continua. Ademas, el tamaño del programa incrementa cuanto más código contenga, lo cual puede no ser lo ideal para aplicaciones embebidas. 

Como resultado, podemos ver los resultados:
* [Sin paralelización]()
* [Con paralelización]()
