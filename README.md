# Hadoop WordCount con Docker

Para poder desplegar un clúster de Hadoop, usando Docker y Docker Compose y asi ejecutar un ejemplo de WordCount con MapReduce, seguir los siguientes pasos.

## Requisitos Previos

- Tener **Docker** y **Docker Compose** instalados.
- Un directorio de trabajo local donde estén todos los archivos necesarios, incluyendo el archivo `docker-compose.yml` (se encuentra en este mismo repositorio de github) y un archivo.txt con contenido (para posteriormente hacer el WordCount).

## Paso a Paso para Ejecutar el Ejemplo

### 0. Preparacion

Primero situate en una direccion valida local como puede ser /user/home

```bash
cd
```
Comprueba que estas en el directorio esperado
```bash
pwd
```
Crea un directorio para guardar los archivos necesarios citados anteriormente en requisitos Previos
```bash
mkdir directorio
```
Úbicate dentro del directorio
```bash
cd directorio
```
Comprueba que estas en el directorio esperado
```bash
pwd
```
Crea los archivos necesarios con (VS) para copiar su contenido dentro:
```bash
code archivo.txt
code docker-compose.yml
```

"Preparativos terminados"

### 1. Crea el Clúster de Hadoop
Asegurate de estar en el directorio esperado (directorio en el que tienes tus archivos en este caso /home/user/directorio)
```bash
pwd
```
Ejecuta el comando `docker-compose up` para crear los servicios del clúster de Hadoop. Utiliza el flag `-d` para ejecutarlos en segundo plano:

```bash
docker-compose up -d
```

### 2. Verifica que los Servicios Están Corriendo

Asegúrate de que todos los servicios necesarios estén activos:

```bash
docker ps
```

Deberías ver los contenedores `namenode`, `datanode1`, `datanode2`, `datanode3`, `resourcemanager`, `nodemanager`, y `historyserver` en la lista (en caso de no verlos, repite los pasos y busca el error concreto).

### 3. Sube el Archivo de Entrada a HDFS

Primero, copia el archivo `archivo.txt` al contenedor **NameNode**:

```bash
docker cp archivo.txt namenode:/archivo.txt
```

Luego, crea los directorios necesarios dentro de HDFS (lugar donde subiras los archivos):

```bash
docker exec -it namenode hdfs dfs -mkdir -p /user/nombreUsuario
```

Por ultimo, sube el archivo a HDFS al directorio creado en el paso anterior:

```bash
docker exec -it namenode hdfs dfs -put /archivo.txt /user/nombreUsuario/
```

### 4. Ejecuta el Ejemplo de WordCount

Ahora ejecuta el ejemplo de **WordCount** usando MapReduce. Esto contará las palabras del archivo que subiste a HDFS.

```bash
docker exec -it resourcemanager yarn jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /user/nombreAlumno/archivo.txt /user/nombreAlumno/output
```

### 5. Verifica los Resultados

Para verificar los resultados del ejemplo de WordCount,  lista los archivos del directorio de salida en HDFS con el siguiente comando:

```bash
docker exec -it namenode hdfs dfs -ls /user/Alumno_AI/output
```

Una vez veas que los resultados estan bien, imprime el contenido del archivo resultante con el comando:

```bash
docker exec -it namenode hdfs dfs -cat /user/Alumno_AI/output/part-r-00000
```

## Resolución de Problemas

### Error de Conexión Entre NameNode y DataNodes

- Asegúrate de que todos los contenedores estén conectados a la red **`shared_network`**.
- Verifica los logs del NameNode y DataNodes:
  ```bash
  docker logs namenode
  docker logs datanode1
  docker logs datanode2
  docker logs datanode3
  ```

### FileAlreadyExistsException

Si obtienes un error indicando que el directorio de salida ya existe, debes de eliminar el directorio de salida antes de volver a ejecutar el trabajo:

```bash
docker exec -it namenode hdfs dfs -rm -r /user/nombreUsuario/output
```

## Limpieza
(La limpieza se puede hacer desde docker Desktop)

Para detener y eliminar todos los contenedores y redes:

```bash
docker-compose down
```

Si en su lugar deseas eliminar también los volúmenes:

```bash
docker-compose down -v
```

## Notas Finales

- Asegúrate de que los **DataNodes** estén registrados correctamente en el **NameNode** antes de intentar subir archivos a HDFS.
- Puedes acceder a la interfaz web del NameNode en `http://localhost:9870` para monitorear el estado del clúster desde el navegador.

# WordCountxDocker
