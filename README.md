# Despliegue de un Clúster Multi-Nodo Apache Hadoop con Docker, Auditoría Física en HDFS y Simulación de Tolerancia a Fallos
## 1. Descripción del proyecto
Desplegar y auditar un Clúster Distribuido Apache Hadoop sobre Contenedores Docker: Almacenamiento Resiliente en HDFS y Cómputo MapReduce sobre YARN.

El proyecto se desarrolla como parte del laboratorio del módulo 5075.

## 2. Arquitectura del proyecto
| Elemento | Configuración |
|---|---|
| Plataforma | Apache Hadoop 3.3.6 |
| Imagen Docker | `apache/hadoop:3` |
| Orquestación | Docker Compose |
| Red Docker | `hadoop-net` |
| Directorio Hadoop | `/opt/hadoop` |
| NameNode | `namenode` |
| DataNodes | `datanode1`, `datanode2`, `datanode3` |
| ResourceManager | `resourcemanager` |
| NodeManager | `nodemanager` |
| Factor de replicación HDFS | 2 |
| Tamaño de bloque configurado | 64 MiB |
| Memoria NodeManager | 2048 MB |
| CPU NodeManager | 2 vcores |

### Componentes desplegados

| Contenedor | Servicio | Función |
|---|---|---|
| `namenode` | HDFS NameNode | Gestiona los metadatos y el espacio de nombres de HDFS. |
| `datanode1` | HDFS DataNode | Almacena bloques de datos. |
| `datanode2` | HDFS DataNode | Almacena bloques de datos. |
| `datanode3` | HDFS DataNode | Almacena bloques de datos. |
| `resourcemanager` | YARN ResourceManager | Gestiona los recursos y las aplicaciones. |
| `nodemanager` | YARN NodeManager | Ejecuta y supervisa los contenedores de procesamiento. |

Los tres DataNodes forman la infraestructura de almacenamiento HDFS. La ejecución de tareas YARN se realiza mediante un único NodeManager.


### Interfaces y puertos
| Servicio | Dirección |
|---|---|
| HDFS NameNode Web UI | http://localhost:9870 |
| YARN ResourceManager Web UI | http://localhost:8088 |


## 3. Estructura del repositorio
```text
IABD-5075-LabHadoop-Alejandro-Colchero/
├── .gitignore
├── README.md
├── docker/
│   ├── config
│   └── docker-compose.yml
├── evidencias/
│   ├── 01_namenode_ui_9870.png
│   ├── 02_hdfs_fsck_bloques.png
│   ├── 03_simulacion_fallo_datanode2.png
│   └── 04_yarn_wordcount_8088.png
└── informe/
    └── Colchero_Alejandro_InformeLabHadoop.pdf
```

## 4. Instrucciones de arranque del clúster

### Paso 1. Acceder al directorio del proyecto

Desde la terminal de nuestro equipo nos vamos a situar en la raiz del repositorio:

```bash
cd IABD-5075-LabHadoop-Alejandro-Colchero
```

### Paso 2. Acceder al repositorio de Docker
```bash
cd Docker
```

### Paso 3. Iniciar los seis servicios
Vamos a ejecutar Docker Compose para levantar los servicios en segundo plano:

```bash
docker compose up -c
```

### Paso 4. Comprobamos el estado de los contenedores
Antes de hacer nada, vamos a comprobar que los contenedores estan activos, lo sabremos porque nos lo marcaran con (Up):

```bash
docker compose ps
```

Los servicios que veremos desplegados son:

- namenode

- datanode1

- datanode2

- datanode3

- resourcemanager

- nodemanager

### Paso 5. Accedemos al contenedor NameNode
Abrimos una sesión dentro del contenedor maestro:

```bash
docker exec -it namenode bash
```

### Paso 6. Comprobamos el estado de HDFS
Desde la terminal del contenedor __namenode__ ejecutamos un informe:

```bash
hdfs dfsadmin -report
```

Con este comando podremos verificar el registro de los tres __DataNodes__ y podemos consultar informacion sobre la capacidad y el estado del almacenamiento distribuido.

### Paso 7. Comprobar la interfaz web de HDFS
Ahora, vamos a abrir nuetro navegador de preferencia y escribimos el siguiente enlace:
https://localhost:9870

Entramos seguidamente en la pestaña __DataNodes__ y comprobamos que aparecen los tres nodos en estado __In Service__

### Paso 8. Acceder a la interfaz web de YARN
Abrimos otra pestaña en nuestro navegador y escribimos:
https://localhost:8088

Esta interfaz nos va a permitir consultar el estado del __ResourceManager__ y supervisar las aplicaciones ejecutads sobre YARN

## 5. Autoría
- Autor: Alejandro Colchero.

- Laboratorio: Despliegue de un Clúster Multi-Nodo Apache Hadoop con Docker, Auditoría Física en HDFS y Simulación de Tolerancia a Fallos.

- Módulo: 5075.

- Repositorio: IABD-5075-LabHadoop-Alejandro-Colchero.

- Tecnologías: Apache Hadoop 3.3.6, HDFS, YARN, MapReduce, Docker y Docker Compose.

## 6. Auditoría física del almacenamiento HDFS
En esta fase vamos a trabajar con el sistema de archivos distribuido HDFS. El objetivo es introducir un archivo de gran tamaño en el clúster y comprobar cómo se almacena y replica entre los DataNodes.

### Paso 1. Crear el directorio de auditoría e HDFS
```bash
hdfs dfs -mkdir -p /datos/auditoria
```

Este comando nos crea la ruta __/datos/auditoria__ en el sistema de archivos distribuido. La opcion __-p__ nos permite crear los directorios necesarios si todavía no existen

### Paso 2. Generar el archivo de telemetría
En esta práctica se utiliza un archivo de telemetría de 150 MiB, generado en el directorio temporal del contenedor:
```bash
/tmp/telemetria_bruta.dat
```
El archivo se utiliza para comprobar cómo HDFS distribuye y replica los bloques de un fichero de gran tamaño.

### Paso 3. Introducimos el archivo de prueba en HDFS
Copiamos el archivo de telemetría desde el sistema local del contenedor al directorio de auditoría de HDFS:
```bash
hdfs dfs -put /tmp/telemetria_bruta.dat /datos/auditoria/
``` 

El archivo queda almacenado en la siguiente ruta:
```bash
/datos/auditoria/telemetria_bruta.dat
``` 

### Paso 4. Comprobar que el archivo se ha almacenado correctamente
Vamos a consultar el contenido del directorio:
```bash
hdfs dfs -ls /datos/auditoria
``` 

También podemos consultar el espacio ocupado por el archivo y sus réplicas:
```bash
hdfs dfs -du -h /datos/auditoria
``` 

Con estos comandos comprobamos que el archivo está presente en HDFS y podemos consultar su tamaño y el espacio total ocupado teniendo en cuenta la replicación.

### Paso 5. Auditar los bloques del archivo
Utilizamos la herramienta fsck de HDFS para comprobar el estado del archivo y obtener información sobre sus bloques:
```bash
hdfs fsck /datos/auditoria/telemetria_bruta.dat -files -blocks -locations
``` 

Este comando nos permite consultar:
- El estado de integridad del archivo.
- El número de bloques en los que se ha dividido.
- La información de replicación.
- Las ubicaciones de los bloques en los DataNodes.
La finalidad de esta comprobación es verificar que HDFS mantiene los datos distribuidos y replicados.

### Paso 6. Verificar la integridad del archivo
Comprobamos el número de bytes que se pueden leer desde HDFS:
```bash
hdfs dfs -cat /datos/auditoria/telemetria_bruta.dat | wc -c
``` 

El resultado obtenido para el archivo utilizado en esta práctica es: __157286400__

Este tamaño equivale a 150 MiB.

## 7. Simulación de tolerancia a fallos en HDFS
En esta parte vamos a simular la caída de uno de los DataNodes para observar cómo responde el clúster ante la pérdida temporal de un nodo de almacenamiento.

### Paso 1. Detener el DataNode 2
Desde la terminal de nuestro equipo, detenemos el contenedor correspondiente:
```bash
docker stop datanode2
```

Con esta operación simulamos la pérdida de uno de los nodos que almacenan bloques de HDFS.

### Paso 2. Comprobar el estado del clúster
Volvemos a consultar el informe de HDFS desde el contenedor namenode:
```bash
hdfs dfsadmin -report
```

Este informe permite consultar los DataNodes registrados y su estado, así como la capacidad y el almacenamiento utilizado.

### Paso 3. Consultar la integridad del archivo
Ejecutamos de nuevo la auditoría del archivo:
```bash
hdfs fsck /datos/auditoria/telemetria_bruta.dat -files -blocks -locations
```

Esta comprobación permite observar el estado de los bloques y sus réplicas después de detener el DataNode.

### Paso 4. Volver a iniciar el DataNode 2
Una vez realizada la simulación, arrancamos de nuevo el contenedor:
```bash
docker start datanode2
```

Volvemos a consultar el estado del clúster:
```bash
hdfs dfsadmin -report
```

De esta manera podemos comprobar la reincorporación del DataNode al clúster.

## 8. Ejecución de MapReduce mediante YARN
En esta fase ejecutamos el ejemplo WordCount de Hadoop. El objetivo es procesar un archivo de texto almacenado en HDFS y contar las apariciones de cada palabra utilizando el modelo de programación MapReduce.

### Paso 1. Creamos el directorio de entrada en HDFS
Creamos el directorio donde almacenaremos el corpus de texto:
```bash
hdfs dfs -mkdir -p /datos/wordcount/input
```

### Paso 2. Creamos el corpus de texto
El archivo __corpus_bigdata.txt__ utilizado en la práctica contiene las siguientes líneas:
```bash
hadoop hdfs yarn mapreduce
big data arquitectura distribuida hadoop
tolerancia a fallos replicacion bloques hdfs
yarn gestiona contenedores mapreduce procesa datos
hadoop escalado horizontal commodity hardware
```

El archivo contiene cinco líneas y ocupa 210 bytes.

### Paso 3. Copiamos el corpus a HDFS
Una vez creado el archivo en el entorno de trabajo, lo copiamos al directorio de entrada de HDFS:
```bash
hdfs dfs -put corpus_bigdata.txt /datos/wordcount/input/
```

Comprobamos que se ha almacenado correctamente:
```bash
hdfs dfs -ls /datos/wordcount/input
```

También podemos consultar el espacio ocupado:
```bash
hdfs dfs -du -h /datos/wordcount/input
```

### Paso 4. Ejecutar WordCount mediante YARN
Ejecutamos el ejemplo WordCount incluido en los ejemplos de MapReduce de Hadoop:
```bash
hadoop jar 
$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount -Dyarn.app.mapreduce.am.resource.mb=1024 /datos/wordcount/input /datos/wordcount/output
```

El parámetro __-Dyarn.app.mapreduce.am.resource.mb=1024__ establece una solicitud de memoria de 1024 MB para el ApplicationMaster de MapReduce durante esta ejecución.

Este ajuste fue necesario porque los intentos anteriores superaban el límite de recursos asignables al ApplicationMaster en la cola de YARN.

Durante la ejecución, Hadoop realiza dos operaciones principales:
- Map: lee el texto de entrada y genera pares de palabras y valores.
- Reduce: agrupa las palabras y suma sus apariciones para obtener el resultado final.
La ejecución correcta finalizó con las fases Map y Reduce al 100 %.

### Paso 5. Consultar el estado de WordCount
Una vez finalizado el trabajo, consultamos el directorio de salida:
```bash
hdfs dfs -ls /datos/wordcount/output
```

Mostramos el contenido del resultado:
```bash
hdfs dfs -cat /datos/wordcount/output/part-r-00000
```

El archivo __part-r-00000__ contiene las palabras encontradas y el número de veces que aparece cada una.

### Paso 6. Consultar las aplicaciones de YARN
Para consultar las aplicaciones ejecutadas en YARN:
```bash
yarn application -list
```

También podemos consultar la información de los nodos de ejecución:
```bash
yarn node -list
```

La interfaz web del ResourceManager permite supervisar las aplicaciones y consultar el estado de los trabajos: http://localhost:8088

## 9. Cierre del entorno y final
Una vez finalizadas las pruebas, cerramos la sesión del contenedor NameNode:
```bash
exit
```

Luego, desde la terminal de nuestro equipo, detenemos los servicios y eliminamos los volúmenes del entorno:
```bash
docker compose down -v
```

Con esto daríamos por finalizado el laboratorio.