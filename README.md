Una pipeline de datos es una construcción lógica que representa un proceso dividido en fases.
Este ejercicio consiste en montar una pipeline que nos permita calcular los KPIs de las peticiones que se realizan a una pagina web. Para ello utilizaremos la siguiente arquitectura:


![image](https://github.com/user-attachments/assets/3f37d298-f1b5-4a04-a463-8da365fc2ef9)

RESUMEN

El proceso empezará con la ingesta de un fichero de logs de Nginx. Esta herramienta nos permite entre otras cosas , monotorizar el tráfico HTTP de una página web y exportar los datos en ficheros log.
Un elemento de las listas que vamos a generar con esta herramienta tiene la siguiente forma:

163.116.184.104 - - [23/May/2023:14:21:08 +0000] "POST /data.php?fl=1&url=https://www.google.com/ HTTP/1.1" 200 9258 "https://www.google.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36" "83.45.66.225"

Como vemos obtenemos bastante información de cada petición, aunque para este ejercicio nos centraremos en la fecha y horas para calcular las KPIs y carga de peticiones que se realizan en nuestra página web.

Posteriormente comenzaremos a procesar datos del fichero de logs con fluentbit. Esta herramienta es una utilidad que nos permite leer, procesar y guardar datos de distintas fuentes como ficheros de logs, bases de datos, etc.
En este caso lo estamos usando para leer datos del fichero de logs nginx.log y enviarlo a una cola de kafka. 

Kafka nos permitirá administrar los flujos de datos de varias fuentes (aunque solo utilizemos uno en este ejemplo) y servirá como buffer para las peticiones que se vayan realizando en la página web.  Desde un script en python podremos consumir el flujo
de salida de datos que nos proporciona Kafka y calcular los KPIs para posteriormente guardarlos en una base de datos Postgres.

Finalmente, utilizaremos una aplicación llamada CloudBeaver para consultar los datos y así cerrar el ciclo: Origen de datos, procesado, almacenado y uso de datos.

PROCESO

1. Para montar la infractuctura necesaria utilizaremos docker. Definiremos y configuraremos las partes en el archivo docker-compose.yml. Por ultimo levantaremos los contenedores con: 'docker-compose up'.
2. Inicializaremos la base datos y crearemos una tabla para almacenar almacenar los KPIs con una estructura sencilla.
   
   ![Captura desde 2024-08-19 12-12-22](https://github.com/user-attachments/assets/cfeeb013-3cb0-4940-83e9-ab9017b2e963)

   La idea es tener en el campo 'kpi_key' el nombre del KPI, por ejemplo 'Peticiones por minuto'. El campo 'kpi_value' tendrá el valor del KPI, por ejemplo "el minuto 15, de la hora 10, del día 12 de Agosto de 2023" y finalmente el campo 'vcount' con el contador de peticiones, por ejemplo 24.

4. Instalaremos fluentbit si todavia no lo tuvieramos instalado: 'curl https://raw.githubusercontent.com/fluent/fluent-bit/master/install.sh | sh'
5. Primero hay que configurar el fichero 'parser.conf' que contiene la información para parsear el contenido de nuestro fichero de logs. 
6. Configuramos el 'fluent-bit.conf' que contiene la información del origen(INPUT) y destino(OUTPUT) de los datos(En este caso el origen es el fichero de logs y el destino es el kafka).
7. Finalmente, podemos ejecutar fluent-bit con el siguiente comando:
'/opt/fluent-bit/bin/fluent-bit -c ./fluent-bit.conf -R ./parser.conf'.
De esta manera el flujo de datos comienza y es consumido por Kafka.
8. Creamos el scrip en python para procesar los datos de una cola de Kafka y almacenar los datos en Postgres. Necesitaremos instalar previamente las dependencias en un entorno virtual de python (python === 2.0.2, psycopg2 === 2.9.6).
El script tendrá un consumidor que obtendra los datos de la cola de Kafka para posteriormente hacer insercciones en la tabla de nuestra base de datos que definimos con anterioridad.
9. Por último tras ejecutar nuestro script ya lo quedaría verificar que todo ha funcionando correctamente utilizando CloudBeaver (o directamente haciendo una petición SQL).

   ![Captura desde 2024-08-19 12-11-06](https://github.com/user-attachments/assets/43bef49f-7104-46ce-84f4-215f66b1c569)



