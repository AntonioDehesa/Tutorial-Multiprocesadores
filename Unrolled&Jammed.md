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

Primero, creamos las matrices y las variables que se necesitaran 
```c
  float matA[SIZE][SIZE];
  float matB[SIZE][SIZE];
  float c[SIZE][SIZE];
  double t, t1, t2;
  int j;
  int i;
  FILE *fptr;
  fptr=fopen("Multiplicacion.txt","w");
```
Posteriormente, y utilizando la técnica de Jamming, podemos iniciar las matrices. 
```c
  for (int i = 0; i < SIZE; i++)
  {
    for (int j = 0; j < SIZE; j++)
    {
      matA[i][j] = j;
      matB[i][j] = j;
    }
  }
```

Una vez que tenemos las matrices iniciadas, podemos aplicar la multiplicación de matrices, la cual se realiza de la siguiente manera: 
```c
  for ( i = 0; i < SIZE; i++)
  {
    for ( j = 0; j < SIZE; j++)
    {
      c[i][j] = 0;
      for (int k = 0; k < SIZE; k+=2)
      {
        c[i][j] = c[i][j] + matA[i][k]*matB[k][j];
        c[i][j] = c[i][j] + matA[i][k+1]*matB[k+1][j];
      }
    }
  }
```

Y, al final, podemos simplemente escribirlo en un archivo de texto para poder verlo de mejor manera a comparación de en la línea de comandos.
```c
  for (int i = 0; i < SIZE; i++) 
  {
    fprintf(fptr, "| ");
    for (int j = 0; j < SIZE; j+=2)
    {
      fprintf(fptr, "|  %f  |", c[i][j]);
      fprintf(fptr, "|  %f  |", c[i][j+1]);
    }
    fprintf(fptr, " |\n");
  }
``` 


Pero, ahora que hemos visto el código base, tenemos que paralelizarlo y ejecutarlo en múltiples threads. 

Para esto, debemos definir el numero de threads con el que nuestro código se ejecutara. Para esto, primero definimos el numero de threads, y luego, con la instrucción omp_set_num_threads, podemos declarar el numero de threads. 
```c
#include <stdio.h>
#include <omp.h>
#define SIZE 10000
#define NUM_THREADS 100
int main(int argc, char const *argv[])
{
  omp_set_num_threads(NUM_THREADS);
  float matA[SIZE][SIZE];
  float matB[SIZE][SIZE];
  float c[SIZE][SIZE];
  double t, t1, t2;
  int j;
  int i;
  FILE *fptr;
  fptr=fopen("Paralelizado.txt","w");
```
Ahora, para nuestro primer ciclo, podemos paralelizarlo fácilmente. Simplemente, con el pragma “omp parallel”, podemos declarar que la siguiente sección, la que se encuentre entre llaves, será paralela. 
Ahora, con el pragma “omp for collapse(2) ordered”, podemos declarar el comportamiento del ciclo for anidado. 
Collapse define que hay un ciclo for anidado en otro, y que están anidados de forma perfecta. Esto es, no hay instrucciones en medio de los ciclos. Las únicas instrucciones son las que se encuentran dentro del ciclo for interno. 
Ordered declara que se mantendrá el orden original del ciclo. Esto puede ayudar a que los valores, aunque sean los correctos, estén en lugares equivocados. 
Por último, con el pragma “omp ordered”, podemos declarar las instrucciones que tienen que estar ordenadas. 
```c
  #pragma omp parallel
  {
    #pragma omp for collapse(2) ordered
    for (int i = 0; i < SIZE; i++)
    {
      for (int j = 0; j < SIZE; j++)
        {
          #pragma omp ordered
          {
            matA[i][j] = j;
            matB[i][j] = j;
          }
        }
    }
  }
```

Pasando al siguiente ciclo, donde esta vez son dos ciclos for anidados. Podemos realizar exactamente lo mismo que en el anterior, con la diferencia de que el ciclo for más interno no está anidado de forma perfecta, por lo que no podemos paralelizarlo de la misma forma. Asi que el último ciclo simplemente estará dentro de las instrucciones ordenadas. 
```c
  #pragma omp parallel
  {
    #pragma omp for collapse(2) ordered
    for(i = 0; i < SIZE; i++)
    {
      for(j = 0; j < SIZE; j++)
      {
        #pragma omp ordered
        {
          c[i][j] = 0;
          for (int k = 0; k < SIZE; k+=2)
          {
            c[i][j] = c[i][j] + matA[i][k]*matB[k][j];
            c[i][j] = c[i][j] + matA[i][k+1]*matB[k+1][j];
          }
        }       
      }
    }
  }
``` 

Ahora, con el ultimo ciclo anidado, como es un solo ciclo for anidado, pero no de forma perfecta, simplemente paralelizamos al primero. 
```c
  #pragma omp parallel
  {
    #pragma omp for ordered
    for (int i = 0; i < SIZE; i++) 
    {
      #pragma omp ordered
      {
        fprintf(fptr, "| ");
        for (int j = 0; j < SIZE; j+=2)
        {
          fprintf(fptr, "|  %f  |", c[i][j]);
          fprintf(fptr, "|  %f  |", c[i][j+1]);
        }
        fprintf(fptr, " |\n");
      }
    }
  }
```

Con estos cambios, el programa estará paralelizado, los resultados deberían ser los mismos, pero los tiempos de ejecución deberían ser menores. 
Programa completo: 
```c
#include <stdio.h>
#include <omp.h>
#define SIZE 10000
#define NUM_THREADS 100
int main(int argc, char const *argv[])
{
  omp_set_num_threads(NUM_THREADS);
  float matA[SIZE][SIZE];
  float matB[SIZE][SIZE];
  float c[SIZE][SIZE];
  double t, t1, t2;
  int j;
  int i;
  FILE *fptr;
  fptr=fopen("Paralelizado.txt","w");
  #pragma omp parallel
  {
    #pragma omp for collapse(2) ordered
    for (int i = 0; i < SIZE; i++)
    {
      for (int j = 0; j < SIZE; j++)
        {
          #pragma omp ordered
          {
            matA[i][j] = j;
            matB[i][j] = j;
          }
        }
    }
  }
  
  t1 = omp_get_wtime();
  #pragma omp parallel
  {
    #pragma omp for collapse(2) ordered
    for(i = 0; i < SIZE; i++)
    {
      for(j = 0; j < SIZE; j++)
      {
        #pragma omp ordered
        {
          c[i][j] = 0;
          for (int k = 0; k < SIZE; k+=2)
          {
            c[i][j] = c[i][j] + matA[i][k]*matB[k][j];
            c[i][j] = c[i][j] + matA[i][k+1]*matB[k+1][j];
          }
        }       
      }
    }
  }
  t2 = omp_get_wtime();
  t = t2-t1;
  printf("\nt = %f \n", t );
  #pragma omp parallel
  {
    #pragma omp for ordered
    for (int i = 0; i < SIZE; i++) 
    {
      #pragma omp ordered
      {
        fprintf(fptr, "| ");
        for (int j = 0; j < SIZE; j+=2)
        {
          fprintf(fptr, "|  %f  |", c[i][j]);
          fprintf(fptr, "|  %f  |", c[i][j+1]);
        }
        fprintf(fptr, " |\n");
      }
    }
  }
  fclose(fptr);
}

```
