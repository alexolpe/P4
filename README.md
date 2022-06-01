PAV - P4: reconocimiento y verificación del locutor
===================================================

![image](https://user-images.githubusercontent.com/91085077/170586686-f053842a-981e-4345-8d7d-14aff02ee59c.png)

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.
  La misión del script wav2lp.sh consiste en realizar una parametrización de una señal de voz usando coeficientes de predicción lineal. El comando sox sirve para generar una nueva señal con los parámetros que se mencionan a continuación. La variable $X2X consiste en un comando llamado spkt x2x que permite parametrizar la señal i devuelve los coeficientes de predicción lineal. Dentro de ese comando se debe especificar el $FRAME que indica como partir la señal original para poder parametrizarla por trozos y la $WINDOW que indica dentro de cada frame que ventana utilizaremos para hacer la predicción. Finalmente, se encuentra el parámetro $LPC que indica basicamente que tipo de predicción lineal se va a realizar.
 

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  Para obtener un fichero de formato fmatrix a la salida se debe calcular el número de columnas que deberá tener la matriz y el número de filas. Como se puede observar el número de columnas se calcula según el parámetro lpc_order que indica cuantos coeficientes de predicción lineal se tienen y se suma uno porque el primer parámetro que entrega esta variable no lo debemos de tener en cuenta ya que es la ganancia de la prediccón. Las columnas en cambio se calculan en función de los frames que tiene en una señal de voz. 
  
  
  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    Conviene usar este formato porque entrega los datos de forma muy ordenada cosa que simplifica mucho la comparación con otras matrices. El formato de entrada es una señal unidimensional (un vector) con las muestras de la señal muestreada de áudio. El formato de salida es una matriz con las columnas que representan los coeficientes de predicción lineal de cada trama que son las filas. 

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
  + ¿Cuál de ellas le parece que contiene más información?

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |      |      |      |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
