PAV - P4: reconocimiento y verificación del locutor
===================================================

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.
  
  La misión del script wav2lp.sh consiste en realizar una parametrización de una señal de voz usando coeficientes de predicción lineal.
  
 •	Sox: se utiliza para convertir una señal de audio en formato WAVE a (-t) un señal sin cabeceras, codificado como (-e) signed de (-b) 16 bits. De entre todas las funciones que tiene destacan: mezclar múltiples ficheros de entrada, normalizar, definir tipo de archivo de salida, número de canales…
  
 •	$X2X: Programa de SPTK que nos permite convertir datos de una entrada estándard a otro tipo de datos (+sf, short format en nuestro caso), enviando el resultado a una salida estándar.
 
 •	$WINDOW: se utiliza para enventanar una secuencia de datos. La ventana escogida (-w) (Blackman, Hamming, Barlett,...) se multiplica por la secuencia de datos de entrada de una determinada longitud (-l) obteniendo una salida de una nueva longitud (-L). En este caso se usa la ventana de Blackman por defecto y una longitud de 240 muestras tanto para los datos de entrada como los de salida.
 
 •	$FRAME: permite convertir una secuencia de datos de entrada en un conjunto de frames. Estos pueden estar o no superpuestos, con un periodo (-p) y una longitud (-l). En este caso la longitud es de 240 muestras y el periodo es de 80 muestras.
 
 •	$LPC: Calcula los coeficientes LPC (predicción lineal) usando el método Levinson-Durbin. Se pueden fijar parámetros como la longitud del frame (-l) a 240 muestras y el orden del LPC (-m).


- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  
  Primero de todo se debe entrenar la señal de entrada, de esa manera se obtiene el fichero $base.lp que proporciona los coeficientes LPC.
  Para obtener un fichero de formato fmatrix a la salida se debe calcular el número de columnas que deberá tener la matriz y el número de filas. Como se puede observar el número de columnas se calcula según el parámetro lpc_order que indica cuantos coeficientes de predicción lineal se tienen y se suma uno porque el primer parámetro que entrega esta variable no se debe tener en cuenta ya que es la ganancia de la prediccón. Eso lo extraemos del fichero .lp convirtiendo el contenido a ASCII con X2X +fa y contando el número de líneas con el comando wc -l.
  
  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    
 Utilizando este formato se puede pasar de una señal de entrada que es un señal unidimensional (un vector) con las muestras de la señal de audio a una matriz en la que se tiene un fácil y rápido acceso a todos los datos almacenados. Además, tienen una correspondencia directa entre la posición en la matriz y el orden del coeficiente y número de trama, por lo que simplifica mucho su manipulación a la hora de trabajar. También ofrece información directa en la cabecera sobre el número de tramas y de coeficientes calculados

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:

    ```sh
    sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
       $LPC -l 240 -m $lpc_order | $LPCC -m $lpc_order -M $cepstrum_order > $base.lpcc
    ```


- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
    ```sh
    sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
       $MFCC -l 240 -m $mfcc_order -n $filter_bank_order -s $freq > $base.mfcc
    ```

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
  
  ![imagen](https://user-images.githubusercontent.com/91128741/171908817-2d96b849-4672-43e1-ad49-2603489abcec.png)
  
  ![imagen](https://user-images.githubusercontent.com/91128741/171911071-41cd1354-8eda-4e35-9335-0ba92745b60b.png)

  ![imagen](https://user-images.githubusercontent.com/91128741/171911349-48c31094-d375-467e-bead-1954ee1c836f.png)


  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    
    Primero para gestionar los datos del locuutor se ha creado una nueva carpeta llamada graph donde almazenaremos los ficheros .txt que interesan para poder graficar los coeficientes. Para empezar, se ha analizado los coeficientes lp, concretamente el segundo y el tercero. Para realizar la extracción de datos se procede a utlizar el siguiente comando en el shell que nos permite extraer de la matriz los coeficientes que nos iteresan. 
    ```sh
     fmatrix_show work/lp/BLOCK00/SES005/*.lp | egrep '^\[' | cut -f3,4 > graph/lp.txt
    ```
 Este comando lo repetimos para los dos otros métodos de predicción lineal (lpcc y mfcc).
 
 ```sh
 fmatrix_show work/lpcc/BLOCK00/SES005/*.lpcc | egrep '^\[' | cut -f3,4 > graph/lpcc.txt
 ```
```sh
 fmatrix_show work/mfcc/BLOCK00/SES005/*.mfcc | egrep '^\[' | cut -f3,4 > graph/mfcc.txt
 ```
 En matlab para crear las graficas hemos realizado el siguiente comando:
 ```matlab
 A = importdata('mfcc.txt');
figure
plot(A(:,1),A(:,2),'.')
grid on
xlabel('a(2)')
ylabel('a(3)')
title('MFCC')
 
 ```
 + ¿Cuál de ellas le parece que contiene más información?


La gráfica que parece que tiene más información es la del MFCC seguido de la del LPCC porque los parámetros no están muy correlados. Esto nos aporta mayor entropia que a su vez significa mayor información. En cambio la gráfica LP vemos que todos los parámetros están muy correlados ya que se genera una recta estrecha (no hay demasiada dispersión), cosa que idica que no nos aporta demasiada información.


- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.
  ```sh
  pearson work/lp/BLOCK00/SES005/*.lp
  ```
  ```sh
  pearson work/lpcc/BLOCK00/SES005/*.lpcc
  ```
  ```sh
  pearson work/mfcc/BLOCK00/SES005/*.mfcc
  ```
  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |   -0.705651   |0.300302      |   0.231577   |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.

Sabemos que un valor alto de la rho[2,3] nos indica que los coeficientes están muy correlados y un valor bajo nos indica lo contrario. Si comparamos las gráficas vemos que estamos en lo cierto, el valor de rho más cercano a 0 es el del MFCC tal y como se ha comentado en las gráficas y esto es porqué los coeficientes son poco correlados. Seguidamente tenemos el método LPCC que también muestra poca correlación en sus coeficientes pero en mayor medida que los MFCC. Por útlimo, tenemos los coeficientes extraídos con LP que tienen una correlación muy elevada. 
  En conclusión, los resultados de las tablas concuerdan con los de las gráficas.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

LPCC

De 8 a 12 coeficientes de predicción (P) y (3/2)P coeficientes cepstrales (Q).

MFCC

Se utilizan entre 14 y 18 coeficientes para reconocimiento del hablante.
    Se suele utilizar un banco de 24 a 40 filtros paso-banda en la escala Mel, aunque también obtenemos buenos resultados con 20 filtros.

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.





- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
  
![imagen](https://user-images.githubusercontent.com/91128741/172026208-effc06f1-6850-412a-a93e-339d0061cb36.png)


  
  
  
  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
  
 ![imagen](https://user-images.githubusercontent.com/91128741/172026774-a3c986f3-a608-44fe-8eb4-0b7471844837.png)
 
  La primera fila de imagenes corresponden al modelo gausiano del locutor 145 comparada con las poblaciones 145 y 189. 
  La segunda fila de imagenes corresponden al modelo gausiano del locutor 190 comparada con las poblaciones 145 y 189.
  
  Se puede observar que el modelado GMM consigue ajustarse muy bien a las características de su locutor (en este caso, los dos primeros coeficientes MFCC), así cuando se compara el modelo de un locutor con la población de otro, se aprecia la diferencia entre ambos. Las zonas de población más densa (curva 50%), se muestra de manera muy visual la gran diferencia entre un locutor y otro, por lo que el modelado GMM es una herramienta muy útil para el reconocimiento y verificación del locutor.



### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
  
  
  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | tasa error |  8.15%  |0.51%    |   0.89%   |

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
    |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | Missed|  78/250=0.312 |  12/250=0.0480  |     |
  | FalsaAlarm|  22/1000=0.022 |  4/1000=0.0040  |   0.89%   |
  | CostDetection|  51 |  8.4  |   0.89%   |
  | Threshold|  0.392219510702497 |  0.298366013850301  |   0.89%   |

Para obtener los mejores resultados se ha decidido crear dos scripts que permitieran automatizar la búsquesda de los parámetros que nos dieran mejores resultados. Estos scripts son el optim_mfcc.sh y optim_train.sh. Con el primero hemos buscado que valores de las opciones del mfcc daban mejores resultados. Con el segundo, se pretende buscar que valor de las opciones que ofrece gmm_train.cpp daban mejores resultados. En un primer momento se pretendia utilizar las opciones usadas en clase y posteriormente añadir otras que tambien ofrece gmm_train.cpp. Cabe mencionar que debido al elevado coste computacional de ejecutar los ficheros gmm_train.cpp, gmm_classify.cpp y gmm_verify.cpp para cada una de las señales de la base de datos, no nos ha dado tiempo a realizar todas las pruebas que nos hubiera gustado. Estas hubieran permitido obtener unas tasas de error y unos CostDetection mejores de los que disponemos.

### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
