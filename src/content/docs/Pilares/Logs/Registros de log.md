# Archivos de logs
Los eventos de logs son registros que se almacenan generalmente en un archivo de texto y que permiten guardar los resultados de una tarea realizada por una tecnología o sistema.
Existen dos tipos de logs:
* Logs estructurados: Son archivos donde cada evento de log esta almacenado de manera que una maquina puede interpretar y analizar la información (Json, XML, CSV, etc). **Este formato será nuestro objetivo en el procesamiento de logs**.
* Logs no estructurados: Cada registro de log esta almacenado solo por una cadena de texto, solo una persona puede interpretar el contenido sin la necesidad de realizar un procesamiento de la información.
  
Para el procesamiento de logs tendremos que definir los siguientes elementos:

 1. Para los archivos no estructurados lo primero es definir la estructura que tiene, por ejemplo 
``31.56.96.51 - - [22/Jan/2019:03:56:16 +0330] "GET /image/61474/productModel/200x200 HTTP/1.1" 200 5379 "https://www.zanbil.ir/m/filter/b113" "Mozilla/5.0 (Linux; Android 6.0; ALE-L21 Build/HuaweiALE-L21) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.158 Mobile Safari/537.36"``

2. Se define el patrón del log como
``$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"``

1. Para el analisis del mensaje realizaremos una tarea llamada **parsing** que nos entregará un log estructurado:
```json
{
  "remote_addr": "31.56.96.51",
  "remote_user": "-",
  "time_local": "22/Jan/2019:03:56:16 +0330",
  "request": "GET /image/61474/productModel/200x200 HTTP/1.1",
  "status ": 200,
  "body_bytes_sent": 5379,
  "http_referer": "https://www.zanbil.ir/m/filter/b113",
  "http_user_agent": "Mozilla/5.0 (Linux; Android 6.0; ALE-L21 Build/HuaweiALE-L21) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.158 Mobile Safari/537.36"
}
```

**El procesamiento de log es una tarea opcional pero recomendada para la mayoria de las tecnologías de observabilidad, ya que permite crear graficos que facilitan la interpretación de la información, ya sea por la volumetria de datos o la complegidad en la estructura del log, es viable evitar el parsing y solo mandar el texto en un unico campo llamado message.**

4. Utilizar un nombrado acorde al esquema utilizado por la tecnología de observabilidad que se utiliza, por ejemplo: 
Para el log anterior tendremos que renombrar los elementos dependiendo la tecnología
   * Original: status
   * Datadog: http.status_code
   * Elasticsearch: http.response.status_code

1. Agregar atributos que permitan la correlación de eventos, generalmente son la fecha (deben tener una unica zona horaria, se recomienda UTC), nombre del host, tecnología que genero el log, trace_id en caso de que se recolecten logs por otro medio que no sea el agente de APM.
2. **Remover y asegurarse de evitar datos sensibles** como tokens, contraseñas, número de tarjetas, etc. esto es una normativa de seguridad y lo puedes revisar en [Data to exclude](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html#data-to-exclude)
