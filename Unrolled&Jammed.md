# Unrolled & Jammed con OpenMP

## Unrolled y Jamming son técnicas de optimización de ciclos.
* Unrolling: Es una técnica que permite reducir de forma efectiva la ejecución de un ciclo, al incrementar la variable de control y modificar la prueba de continuación y salto a la línea de ejecución. 
La forma en la que funciona es aumentando las instrucciones ejecutadas por cada iteración del ciclo. Por ejemplo, en lugar de realizar una sola instrucción, por ejemplo, en un arreglo, podríamos realizar una instrucción con el elemento al que indica la variable de control, y al elemento que indica la variable de control mas uno. Al realizar esto, por cada iteración podremos evaluar dos elementos del arreglo, cortando la cantidad de iteraciones a la mitad. 
Ejemplo: 

Este es un ejemplo sencillo, en el cual simplemente imprimimos la iteración actual. 


```c
#include <stdio.h>

int main()
{
	for(int i = 0; i < 20; i++)
	{
		printf("Iteracion: %d\n", i);
	}
	return 0;
}
```
Posteriormente, tenemos el ejemplo con *Unrolled*, en el cual simplemente realizamos unas instrucciones adicionales dentro de cada iteración. 
Ejemplo: 
```c
#include <stdio.h>

int main()
{
	for(int i = 0; i < 20; i+=3)
	{
		printf("Iteracion: %d\n", i);
		printf("Iteracion: %d\n", i+1);
		printf("Iteracion: %d\n", i+2);
	}
	return 0;
}
```

Una desventaja de este método es que el tiempo de ejecución de cada iteración aumenta proporcionalmente a la cantidad de instrucciones que se le añadan. Por lo tanto, puede no ser lo ideal si necesitamos que los efectos se muestren de forma continua. Ademas, el tamaño del programa incrementa cuanto más código contenga, lo cual puede no ser lo ideal para aplicaciones embebidas. 

Ahora que ya sabemos cómo utilizar *Unrolled* y *Unrolled & Jammed*, podemos comenzar a ver ejemplos un poco más grandes.
El ejemplo que veremos será una multiplicación de matrices utilizando *Unrolled* y *Unrolled & Jammed*. 
