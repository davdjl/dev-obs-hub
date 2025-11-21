# Instalación de Agente Java
La instalación del agente java puede ser realizada utilizando una bandera al momento de su ejecución

## Instalación en una aplicación
Para la instalación del agente de Java descargaremos el [JAR](https://mvnrepository.com/artifact/co.elastic.apm/elastic-apm-agent) correspondiente del repositorio de Maven y usaremos las siguentes flags en el comando de ejecución de la aplicación
```sh
java -javaagent:/path/to/elastic-apm-agent-<version>.jar -Delastic.apm.service_name=my-cool-service -Delastic.apm.server_url=http://127.0.0.1:8200 -jar my-application.jar
```

El comando esta compuesto por los siguientes parametros de APM:
* Javaagent: ruta del JAR descargado.
* -Delastic.apm.service_name: nombre del servicio que vamos a monitorear, este nombre será con el que identificaremos el servicio en kibana.
* -Delastic.apm.server_url: URL del servidor de APM

Estos valores son una configuración básica de un agente, pero es posible que se requieran otras configuraciones y llevar un control de cada una de ellas, por ello es posible realizar estas configuraciones a nivel de un archivo de propiedades, como propiedades del java system o como variables de ambiente y para cada una de ellas tendremos una notación diferente.

Por ejemplo, si quisieramos modificar el nivel de logging que la aplicación recolecta, podemos configurarlo de la siguiente manera

1. Para el archivo de propiedades, se deberá crear un archivo con el nombre *elasticapm.properties* en el mismo folder donde se encuentra el JAR del agente y su notación estará compuesta de la siguiente manera
```sh
log_level=CRITICAL
```
2. Para definir como una propiedad de sistema Java usaremos, esta misma estructura la podemos utilizar en el comando de ejecución de la aplicación, pero deberá iniciar con -D
```sh
elastic.apm.log_level=CRITICAL
```
3. Para definir como una variable de ambiente usaremos
```sh
ELASTIC_APM_LOG_LEVEL=CRITICAL
```

**Es importante mencionar que si tenemos más de una configuración aplicada, la precedencia se realiza como lo enlistamos, primero el archivo de propiedades y al final las variables de ambiente**

Todas las configuraciones disponibles se pueden revisar en la [documentación de elasticsearch](https://www.elastic.co/docs/reference/apm/agents/java/configuration)
