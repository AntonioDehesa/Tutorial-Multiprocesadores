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
