# Spool-Concurrente-De-Impresion-En-Pantalla

Su funcionamiento general consiste en controlar el acceso de procesos clientes al
recurso compartido, en este caso la pantalla, garantizando la utilización en exclusión mutua de
dicho recurso. Un sistema spool para impresión imprime un documento sólo cuando éste se ha
generado por completo. De esta forma, se consigue que un proceso no pueda apropiarse
indefinidamente, o durante mucho tiempo si el proceso es lento, del recurso compartido. Como
mecanismo de comunicación/sincronización entre procesos se van a utilizar cauces con nombre
(archivos FIFO) y en algún caso señales. En la siguiente figura se muestra el esquema general a
seguir para la implementación:

![Screenshot from 2023-04-12 17-30-12](https://user-images.githubusercontent.com/87938446/231507072-44fc8a29-246f-4afd-b005-8064a7cea149.png)
![Screenshot from 2023-04-12 17-32-06](https://user-images.githubusercontent.com/87938446/231507622-8969feee-5c24-4571-8449-c7eaa3c55beb.png)

Siguiendo los mensajes numerados obtendremos las interacciones entre procesos para poder llevar a cabo la impresión de un archivo, según se explica a continuación:
1. Un cliente solicita la impresión de un archivo enviando un mensaje (cuyo contenido no
tiene importancia) al servidor a través de un FIFO cuyo nombre es conocido.
Guía Práctica de Sistemas Operativos-0
2. El servidor lee esta petición delegando la recepción e impresión del documento en un
proceso (proxy) que crea específicamente para atender a dicho cliente. Una vez
servida dicha petición, el proxy terminará.
3. El servidor responde al cliente a través de otro FIFO de nombre conocido, informando
de la identidad (PID) del proxy que va a atender su petición. Este dato es la base para
poder comunicar al cliente con el proxy, ya que éste creará un nuevo archivo FIFO
especifico para esta comunicación, cuyo nombre puede ser el propio PID del proxy. El
proxy se encargará de eliminar dicho FIFO cuando ya no sea necesario (justo antes de
terminar su ejecución).
4. El cliente lee esta información que le envía el servidor, de manera que así sabrá donde
enviar los datos a imprimir.
5. Probablemente el cliente necesitará enviar varios mensajes como éste, tantos como
sean necesarios para transmitir toda la información a imprimir. El final de la
transmisión de la información lo indicará con un fin de archivo.
6. El proxy obtendrá la información a imprimir llevando a cabo probablemente varias
lecturas como ésta del FIFO.
7. Por cada lectura anterior, tendrá lugar una escritura de dicha información en un
archivo temporal, creado específicamente por el proxy para almacenar completamente
el documento a imprimir.
8. Una vez recogido todo el documento, volverá a leerlo del archivo temporal justo
después de comprobar que puede disponer de la pantalla para iniciar la impresión en
exclusión mutua.
9. Cada lectura de datos realizada en el paso anterior implicará su escritura en pantalla.

El formato de ejecución de este
programa es:
$> clientes <nombre_fifos_conocidos> <número_clientes>
El argumento <nombre_fifos_conocidos> es un único nombre, de forma que los clientes
suponen que el nombre del FIFO conocido de entrada al servidor es dicho nombre concatenado
con el carácter “e”. En el caso del FIFO de salida, se concatena dicho nombre con el carácter
“s”.
