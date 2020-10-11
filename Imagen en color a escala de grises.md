# Imagen en color a escala de grises

### Ejemplo para convertir una imagen en color a escala de grises. Se vera su forma original, paralelizada, asi como utilizando malloc. 
En este ejemplo, podemos ver un ejemplo del uso de openMP y la paralelización. Primero, se verá el código para convertir una imagen que esta a color, a una imagen en escala de grises. 
Después, se verá cómo se paraleliza.
Ademas, se vera la forma de utilizar la función malloc para realizar el mismo trabajo. 
Al finalizar, simplemente se verá el código paralelizado con malloc, pues la técnica de paralelización será la misma. 

En el código inicial, primero creamos los punteros a los archivos necesarios, los iniciamos, asi como creamos las variables que necesitaremos. 

```c
    FILE *image, *outputImage, *lecturas;
    image = fopen("sample.bmp","rb");          //Imagen original a transformar
    outputImage = fopen("img2_dd.bmp","wb");    //Imagen transformada

    unsigned char r, g, b;               //Pixel
    unsigned char pixel;
```

Posteriormente, copiamos la cabecera del archivo original, al archivo copia que estamos creando. 
Sin esta cabecera, simplemente el archivo no funcionara. 

```c
    for(int i=0; i<54; i++) fputc(fgetc(image), outputImage);   //Copia cabecera a nueva imagen
```

Posteriormente, podemos simplemente realizar un ciclo while, en el cual obtenemos los valores relacionados a rojo, verde y azul, les aplicamos una pequeña formula, y los escribimos en el archivo nuevo. 

```c
    while(!feof(image)){                                        //Grises
       b = fgetc(image);
       g = fgetc(image);
       r = fgetc(image);

       pixel = 0.21*r+0.72*g+0.07*b;
       fputc(pixel, outputImage);
       fputc(pixel, outputImage);
       fputc(pixel, outputImage);
    }
```

Posteriormente, simplemente cerramos los archivos. 

```c
    fclose(image);
    fclose(outputImage);
    return 0;
```

Aquí se puede ver el archivo completo. 

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    FILE *image, *outputImage, *lecturas;
    image = fopen("sample.bmp","rb");          //Imagen original a transformar
    outputImage = fopen("img2_dd.bmp","wb");    //Imagen transformada

    unsigned char r, g, b;               //Pixel
    unsigned char pixel;
    for(int i=0; i<54; i++) fputc(fgetc(image), outputImage);   //Copia cabecera a nueva imagen
    while(!feof(image)){                                        //Grises
       b = fgetc(image);
       g = fgetc(image);
       r = fgetc(image);

       pixel = 0.21*r+0.72*g+0.07*b;
       fputc(pixel, outputImage);
       fputc(pixel, outputImage);
       fputc(pixel, outputImage);
    }

    fclose(image);
    fclose(outputImage);
    return 0;
}
```
Ahora, para paralelizarlo, tenemos que recordar que los ciclos que se pueden paralelizar correctamente y sin problemas, son los ciclos for. 
Entonces, necesitaremos nuevas variables para lograr esto. Además, para poder ver la diferencia de tiempo de ejecución entre el original, y el paralelizado, tenemos que añadir otras variables. 
```c
#define NUM_THREADS 110

int main()
{
    omp_set_num_threads(NUM_THREADS);
    
    FILE *image, *outputImage, *lecturas;
    image = fopen("sample.bmp","rb");          //Imagen original a transformar
    outputImage = fopen("img2_dd.bmp","wb");    //Imagen transformada

    unsigned char r, g, b;               //Pixel
    unsigned char pixel;
    long ancho, alto;
    unsigned char xx[54];
    double t, t1, t2;
```

Posteriormente a eso, dentro del ciclo for inicial, tenemos que obtener datos de la cabecera. Estos datos se utilizarán para saber el alto y ancho de la imagen, y asi poder realizar el ciclo posterior donde se escribirá en la nueva imagen. Esto se hace de la siguiente manera: 
```c
    #pragma omp parallel
    {
      #pragma omp for ordered
      for(int i=0; i<54; i+=2) 
      {
        #pragma omp ordered
        {
          xx[i] = fgetc(image);
          xx[i+1] = fgetc(image);
          fputc(xx[i], outputImage);   //Copia cabecera a nueva imagen
          fputc(xx[i+1], outputImage);   //Copia cabecera a nueva imagen
        }
      }
    }
```

Una vez obtenemos los datos de la cabecera, podemos obtener el ancho y el alto. Además, empezamos a medir el tiempo de ejecución, para poderlo comparar con el original. 
Posteriormente, pasamos el ciclo while a un ciclo for, el cual podemos paralelizar de forma correcta. 
```c
    ancho = (long)xx[20]*65536+(long)xx[19]*256+(long)xx[18];
    alto = (long)xx[24]*65536+(long)xx[23]*256+(long)xx[22];
    t1 = omp_get_wtime();

    #pragma omp parallel
    {
      #pragma omp for ordered
      for(int i = 54; i < ancho*alto*3; i++)
      {
        #pragma omp ordered
        {
          b = fgetc(image);
           g = fgetc(image);
           r = fgetc(image);

           pixel = 0.21*r+0.72*g+0.07*b;
           fputc(pixel, outputImage);
           fputc(pixel, outputImage);
           fputc(pixel, outputImage);
        }
      }
    }
```

Al final, simplemente imprimimos el tiempo que ha tardado, para poder compararlo con el original. 
```c
t2 = omp_get_wtime();
    t = t2-t1;
  printf("\nt = %f \n", t );
    fclose(image);
    fclose(outputImage);
    return 0;
```

Desafortunadamente, en este caso, creo que no ha valido la pena paralelizarlo, pues si lo comparamos con el original, es mucho más lento
* [Original](https://github.com/AntonioDehesa/Tutorial-Multiprocesadores/blob/main/Images/OriginalColorAGris.JPG)
* [Paralelizado](https://github.com/AntonioDehesa/Tutorial-Multiprocesadores/blob/main/Images/ParaleloColorAGris.JPG)
Ahora, hay que hacer lo mismo, pero utilizando malloc. 

Primero, declaramos las variables, archivos y apuntadores que utilizaremos. 
```c
    FILE *image, *outputImage, *lecturas;
    image = fopen("sample.bmp","rb");          //Imagen original a transformar
    outputImage = fopen("img2_dd.bmp","wb");    //Imagen transformada
    long ancho;
    long alto;
    unsigned char r, g, b;               //Pixel
    unsigned char* ptr;
    unsigned char pixel;
    unsigned char xx[54];
    int cuenta = 0;
```

Después, simplemente se realiza el ciclo for que hemos realizado antes, asi como obtenemos los valores de la cabecera para obtener el alto y ancho de la imagen. La única diferencia ahora es que al apuntador lo ampliamos con malloc, al tamaño alto*ancho*3. 
```c
    for(int i=0; i<54; i++)
    {
      xx[i] = fgetc(image);
      fputc(xx[i], outputImage);   //Copia cabecera a nueva imagen
    }
    ancho = (long)xx[20]*65536+(long)xx[19]*256+(long)xx[18];
    alto = (long)xx[24]*65536+(long)xx[23]*256+(long)xx[22];
```

Después, tenemos el mismo código que en archivos anteriores, pero con un apuntador en lugar de valores directos. 

```c
    ptr = (unsigned char*)malloc(alto*ancho*3* sizeof(unsigned char));

    while(!feof(image)){
      b = fgetc(image);
      g = fgetc(image);
      r = fgetc(image);


      pixel = 0.21*r+0.72*g+0.07*b;

      ptr[cuenta] = pixel; //b
      ptr[cuenta+1] = pixel; //g
      ptr[cuenta+2] = pixel; //r
      fputc(ptr[cuenta], outputImage);
      fputc(ptr[cuenta+1], outputImage);
      fputc(ptr[cuenta+2], outputImage);
      cuenta++;

    }
```

El código completo sería el siguiente: 
```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    FILE *image, *outputImage, *lecturas;
    image = fopen("sample.bmp","rb");          //Imagen original a transformar
    outputImage = fopen("img2_dd.bmp","wb");    //Imagen transformada
    long ancho;
    long alto;
    unsigned char r, g, b;               //Pixel
    unsigned char* ptr;
    unsigned char pixel;
    unsigned char xx[54];
    int cuenta = 0;
    for(int i=0; i<54; i++)
    {
      xx[i] = fgetc(image);
      fputc(xx[i], outputImage);   //Copia cabecera a nueva imagen
    }
    ancho = (long)xx[20]*65536+(long)xx[19]*256+(long)xx[18];
    alto = (long)xx[24]*65536+(long)xx[23]*256+(long)xx[22];

    ptr = (unsigned char*)malloc(alto*ancho*3* sizeof(unsigned char));

    while(!feof(image)){
      b = fgetc(image);
      g = fgetc(image);
      r = fgetc(image);


      pixel = 0.21*r+0.72*g+0.07*b;

      ptr[cuenta] = pixel; //b
      ptr[cuenta+1] = pixel; //g
      ptr[cuenta+2] = pixel; //r
      fputc(ptr[cuenta], outputImage);
      fputc(ptr[cuenta+1], outputImage);
      fputc(ptr[cuenta+2], outputImage);
      cuenta++;

    }
    free(ptr);
    fclose(image);
    fclose(outputImage);
    return 0;
}

```

Ahora, podemos paralelizarlo. Claro, el proceso es exactamente el mismo que en los ejemplos anteriores, entonces solo se mostrara el código completo.
```c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

#define NUM_THREADS 110

int main()
{
    omp_set_num_threads(NUM_THREADS);
    FILE *image, *outputImage, *lecturas;
    image = fopen("sample.bmp","rb");          //Imagen original a transformar
    outputImage = fopen("img2_dd.bmp","wb");    //Imagen transformada
    long ancho;
    long alto;
    unsigned char r, g, b;               //Pixel
    unsigned char* ptr;
    unsigned char pixel;
    unsigned char xx[54];
    int cuenta = 0;
    #pragma omp parallel
    {
      #pragma omp for ordered
      for(int i=0; i<54; i+=2)
      {
        #pragma omp ordered
        {
          xx[i] = fgetc(image);
          fputc(xx[i], outputImage);   //Copia cabecera a nueva imagen
          xx[i+1] = fgetc(image);
          fputc(xx[i+1], outputImage);   //Copia cabecera a nueva imagen
        }
        
      }
    }
    
    ancho = (long)xx[20]*65536+(long)xx[19]*256+(long)xx[18];
    alto = (long)xx[24]*65536+(long)xx[23]*256+(long)xx[22];

    ptr = (unsigned char*)malloc(alto*ancho*3* sizeof(unsigned char));

    #pragma omp parallel
    {
      #pragma omp for ordered
      for(int i = 54; i < ancho*alto*3; i++)
      {
        #pragma omp ordered
        {
          b = fgetc(image);
           g = fgetc(image);
           r = fgetc(image);

           pixel = 0.21*r+0.72*g+0.07*b;
           ptr[cuenta] = pixel; //b
            ptr[cuenta+1] = pixel; //g
            ptr[cuenta+2] = pixel; //r
            fputc(ptr[cuenta], outputImage);
            fputc(ptr[cuenta+1], outputImage);
            fputc(ptr[cuenta+2], outputImage);
            cuenta++;
        }
      }
    }
    free(ptr);
    fclose(image);
    fclose(outputImage);
    return 0;
}

```
