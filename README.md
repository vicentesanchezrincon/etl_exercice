Una pipeline de datos es una construcción lógica que representa un proceso dividido en fases.
Este ejercicio consiste en montar una pipeline que nos permita calcular los KPIs de las peticiones que se realizan a una pagina web. Para ello utilizaremos la siguiente arquitectura:


![image](https://github.com/user-attachments/assets/3f37d298-f1b5-4a04-a463-8da365fc2ef9)


El proceso empezará con la ingesta de un fichero de logs de Nginx. Esta herramienta nos permite entre otras cosas , monotorizar el tráfico HTTP de una página web y exportar los datos en ficheros log.
Un elemento de las listas que vamos a generar con esta herramienta tiene la siguiente forma:

163.116.184.104 - - [23/May/2023:14:21:08 +0000] "POST /data.php?fl=1&url=https://www.google.com/ HTTP/1.1" 200 9258 "https://www.google.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36" "83.45.66.225"

Como vemos obtenemos bastante información de cada petición, aunque para este ejercicio nos centraremos en la fecha y horas para calcular las KPIs y carga de peticiones que se realizan en nuestra página web.

Posteriormente comenzaremos a procesar datos del fichero de logs con fluentbit. Esta herramienta es una utilidad que nos permite leer, procesar y guardar datos de distintas fuentes como ficheros de logs, bases de datos, etc.
En este caso lo estamos usando para leer datos del fichero de logs nginx.log y enviarlo a una cola de kafka. 

Kafka nos permitirá administrar los flujos de datos de varias fuentes (aunque solo utilizemos uno en este ejemplo) y servirá como buffer para las peticiones que se vayan realizando en la página web.  Desde un script en python podremos consumir el flujo
de salida de datos que nos proporciona Kafka y calcular los KPIs para posteriormente guardarlos en una base de datos Postgres.

Finalmente, utilizaremos una aplicación llamada CloudBeaver para consultar los datos y así cerrar el ciclo: Origen de datos, procesado, almacenado y uso de datos.
