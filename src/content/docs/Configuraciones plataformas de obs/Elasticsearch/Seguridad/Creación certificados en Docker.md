# Manual detallado: Configuración de clúster ELK con certificados en Docker

**Contenido**

[Infraestructura del clúster](#infraestructura-del-clúster)

1. [Creación del docker-compose para Elasticseach](#creación-del-docker-compose-para-elasticseach)

2. [Generar el CA (certificado de autoridad)](#generar-el-ca-certificado-de-autoridad)

3. [Configurar HTTPS](#configurar-https)

4. [Validación](#validación)

5. [Creación del docker-compose para Kibana](#creación-del-docker-compose-para-kibana)

6. [Certificados para Kibana](#certificados-para-kibana)

7. [Cifrado de tráfico entre el navegador y kiabana](#cifrado-de-tráfico-entre-el-navegador-y-kiabana)

8. [Usuario de Kibana](#usuario-de-kibana)

9. [Validación](#validación-1)

10. [Creación del docker-compose para APM](#creación-del-docker-compose-para-apm)

11. [Validación](#validación-2)

# Infraestructura del clúster:

- 3 nodos de Elasticsearch
- 1 nodo de Kibana
- 1 nodo de APM
- Versión más reciente: 9.0.2

# 1. Creación del docker-compose para Elasticseach

Para mantener una mejor automatización y tener reinicios específicos, se
optó por separar los docker-compose por componentes: uno para los tres
nodos de elastic, otro para Kibana y uno último para APM.

- No se utilizaron variables de entorno, por lo que todo se mantiene en el docker-compose.

- Se recomienda una carpeta exclusiva para el proyecto, dentro las carpetas extras necesarias para cada servicio:

![carpetas](/public/images/carpetas.png)

- Dentro de la carpeta de cada servicio, guardar el archivo docker-compose.yml correspondiente.

El ejemplo del archivo docker-compose para Elasticsearch está disponible
en:

(URL DE GITHUB)

Consideraciones

El mapeo de puertos por default para Elasticsearch es: 9200:9200 y para
Kibana: 5601:5601, sin embargo, para este proyecto se utilizó un mapeo
diferente para evitar conflictos con otros contenedores previamente
creados.

- Creación de contenedores basados en el archivo docker-compose.

![contenedores](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/contenedores.png)

# 2. Generar el CA (certificado de autoridad)

- un CA para el clúster.
![certificado CA](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/certificado_CA.png) 

- Generar el certificado p12 firmado por el CA.
![certificado firmado](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/certificado_firmado.png) 

- Copiar ambos certificados en la carpeta local creada en pasos anteriores
![certificados](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/certificados.png) 

# 3. Configurar HTTPS

- Generar los certificados necesarios para habilitar HTTPS en los nodos de Elasticsearch.

![HTTPS](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/HTTPS.png)

Nota: seguir todos los pasos descritos en la documentación:
<https://www.elastic.co/docs/deploy-manage/security/set-up-basic-security-plus-https>

Consideraciones:

En el paso (i) solicitarán las direcciones IP, sin embargo, al utilizar docker las IP´s cambian de manera dinámica, por lo que, para esta instrumentación se utilizaron IPs aleatorias, como: 255.255.255.255, 172.0.0.0, 255.0.0.0

- El resultado será una carpeta .zip llamada elasticsearch-http.zip y se recomienda alojarla en la ruta: /usr/share/elasticsearch/config/ tanto para elasticsearch como para Kibana.

- Una vez extraída la carpeta, deberá haber 2 carpetas una de elasticsearch y otra de Kibana, ambas se pueden copiar localmente.

![ELK y kibana](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/ELK_y_kibana.png)

- Dentro de la carpeta de ambas carpetas están los certificados correspondientes para cada uno.

![SSL](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/SSL.png)

- En el docker-compose de elasticsearch, se agregarán las configuraciones necesarias para extraer los certificados por cada nodo.

![Seguridad](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Seguridad.png)

# 4. Validación

- Desde la carpeta donde se encuentra el archivo docker-compose.yml ejecutar: docker compose

- Colocar la dirección <https://9210> para validar que elasticsearch levantó correctamente

![localhost](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/localhost.png) 

# 5. Creación del docker-compose para Kibana

El ejemplo del archivo docker-compose para Kibana está disponible en:

> (URL DE GITHUB)

- Creación de contenedores basados en el archivo docker-compose

# 6. Certificados para Kibana

- En el certificado http previamente creado, se encuentra una carpeta de Kibana con el certificado .pem

![Certificado pem](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Certificado_pem.png)

- Ese archivo se utiliza para configurar Kibana y que confíe en la CA de Elasticsearch para la capa HTTP.

- El archivo deberá configurarse como autoridad confiable con la variable descrita en el docker.compose de Kibana:

>ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=/usr/share/kibana/config/elasticsearch-ca.pem

- Montaje del archivo .pem al contenedor

También se está montando el archivo .pem desde el sistema host al
contenedor con este volumen:

>../certs/elasticsearch-ssl-http/kibana/elasticsearch-ca.pem:/usr/share/kibana/config/elasticsearch-ca.pem:ro

# 7. Cifrado de tráfico entre el navegador y kibana

- Generar una solicitud de firma de certificado (CSR), y una clave privada (.key) para Kibana, tal como lo describe la documentación: <https://www.elastic.co/docs/deploy-manage/security/set-up-basic-security-plus-https#encrypt-kibana-elasticsearch>

Comando: ./bin/elasticsearch-certutil csr -name kibana-server

![Kibana server](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Kibana_server.png)

- Descomprimir la carpeta zip para obtener el kibana-server.csr certificado de seguridad sin firmar y la kibana-server.key clave privada sin cifrar.

- Para esta práctica **Kibana está funcionando con HTTP en su interfaz web**, pero conectándose **vía HTTPS hacia Elasticsearch**.

- El comando: ELASTICSEARCH_SSL_VERIFICATIONMODE=certificate, en el docker-compose de Kibana solo valida el certificado del servidor de Elasticsearch, es decir, Kibana confía en los certificados de los nodos de Elasticsearch.

# 8. Usuario de Kibana

- **Restablecer la contraseña del usuario interno kibana**, que es el que Kibana usa para autenticarse en Elasticsearch.

- Este usuario se creó automáticamente cuando se activó la seguridad en Elasticsearch (X-Pack).

- Comando: bin/elasticsearch-reset-password -u kibana

Consideraciones

1.  El clúster debe estar encendido y funciona

2.  Se ejecuta desde el nodo maestro del clúster

![Usuario de kibana](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/usuario_de_kibana.png)

# 9. Validación

- Desde la carpeta donde se encuentra el archivo docker-compose.yml ejecutar: docker compose

- Colocar la dirección <http://5610> para validar que kibana levantó correctamente

![Kibana](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Kibana.png)

- Colocar usuario (elastic por default) y contraseña (definida previamente en el docker-compose de elasticsearch)

![Contraseña](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Contraseña.png)

# 10. Creación del docker-compose para APM

El ejemplo del archivo docker-compose y el archivo de configuración para APM está disponible en:

>(URL DE GITHUB)

Consideraciones

- Creación del contenedor basado en el archivo docker-compose y en el apm-server.yml.

![APM](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/APM.png)

- El archivo apm-server.yml define el comportamiento de APM, como los servicios a los que se conectará, por medio de qué puertos escuchará, qué seguridad aplicará, entre otras cosas.

![Configuración APM](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Configuración_APM.png)

- Es importante montar el certificado .pem (que se monto para el contenedor de Kibana) y el archivo de configuración apm-server.yml anteriormente creado.

# 11. Validación

- Desde la carpeta donde se encuentra el archivo docker-compose.yml y el apm-server.yml ejecutar: docker compose

- Colocar la dirección http://localhost:5610/app/apm para validar que kibana levantó correctamente

![Consola de Kibana](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Consola_de_Kibana.png)

- Se comprueba el envío y recibo de datos

![Aplicación](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/Aplicación.png)
