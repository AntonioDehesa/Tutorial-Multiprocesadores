# Uso e instalación de OpenMP en Windows 10

### Originalmente, Windows 10 no cuenta con un compilador de C. Por lo que nuestro tutorial también incluirá como descargar e instalar un compilador para C en Windows 10, con el cual también podremos instalar OpenMP, y solucionar los problemas que pueden surgir.
A continuación, tenemos los pasos a seguir: 
1. Descargar y ejecutar el administrador de instalación de [MinGW](http://www.mingw.org/). 
2. Usando el instalador, instalar los paquetes necesarios. 
Aunque lo más sencillo es simplemente dar clic en la esquina superior izquierda en el botón *Installation*, nos instalaría paquetes que pueden no ser muy útiles. 
Para el propósito de este tutorial, los únicos paquetes que nos interesan son los relacionados a C, los paquetes que vienen por default, y, para solucionar problemas que pueden surgir de utilizar la librería OpenMP, los paquetes relacionados a mingw32-pthreads-w32.
3. Uso de OpenMP
Para poder utilizar OpenMP, simplemente tenemos que importar la librería como haríamos con cualquier otra librería en C
```c
#include <stdio.h>
#include <omp.h>
```
Posteriormente, para declarar una sección a paralelizar, se realiza con la siguiente instrucción: 
```c
#pragma omp parallel for
```
Esto hará que el siguiente for se realice paralelamente. 
Posterior a esta instrucción, podemos declarar el ciclo for como en cualquier otro código. 
```c
for(int i=0;i<10;i++)
  {
    printf("%i\n",i);
  }
```
Entonces, este es el resultado final de nuestro código. 
```c
#include <stdio.h>
#include <omp.h>
int main()
{
  #pragma omp parallel for
  for(int i=0;i<10;i++)
  {
    printf("%i\n",i);
  }
  return 0;
}#include <stdio.h>
#include <omp.h>
int main()
{
  #pragma omp parallel for
  for(int i=0;i<10;i++)
  {
    printf("%i\n",i);
  }
  return 0;
}
```
Como resultado, podemos ver los resultados:
* [Sin paralelización]()
* [Con paralelización]()
