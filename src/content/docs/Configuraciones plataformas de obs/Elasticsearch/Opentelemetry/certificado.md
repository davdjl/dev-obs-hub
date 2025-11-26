![api.o](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/api.o.png)

# Control de Cambio

Proyecto:

- Instalación operador de OpenTelemetry

Elaborado por:

### Api.o CONSULTING

FEBRERO 09 2025

## SECCIÓN I: HISTORIAL DE CAMBIOS AL DOCUMENTO

<table>
<tr>
<td>Fecha</td>
<td>Versión</td>
<td>Descripción</td>
<td>Autor</td>
</tr>
<tr>
<td>15/09/2025</td>
<td>0.1</td>
<td>Creación del documento</td>
<td>Fernando Rupitt</td>
</tr>
</table>

## SECCIÓN II: INFORMACIÓN DEL DOCUMENTO Y ACEPTACIÓN

<table>
<tr>
<td>Título</td>
<td>Control de Cambios</td>
</tr>
<tr>
<td>Versión</td>
<td>0.1</td>
</tr>
<tr>
<td>Propósito del Documento</td>
<td>Documentar el procedimiento de instalación del operador de OpenTelemetry.</td>
</tr>
</table>

La información contenida en este documento, es Propiedad del Proyecto por lo que no deberá ser divulgada, duplicada o dada a conocer, parcial o totalmente, fuera de alcance del Proyecto sin una autorización por escrito.

>SECCIÓN III: CONTENIDO
1. [SECCIÓN I: HISTORIAL DE CAMBIOS AL DOCUMENTO](#SECCIÓN-I:-HISTORIAL-DE-CAMBIOS-AL-DOCUMENTO)
2. [SECCIÓN II: INFORMACIÓN DEL DOCUMENTO Y ACEPTACIÓN](#SECCIÓN-II:-INFORMACIÓN-DEL-DOCUMENTO-Y-ACEPTACIÓN)
3. [SECCIÓN III: CONTENIDO](#SECCIÓN-III:-CONTENIDO)
4. [SECCIÓN IV: INSTALACIÓN OPERADOR DE OPENTELEMETRY](#SECCIÓN-IV:-OPERADOR-DE-OPENTELEMETRY)

## SECCIÓN IV: INSTALACIÓN OPERADOR DE OPENTELEMETRY

Procedimiento para la instalación del operador de Opentelemetry, este es requerido para poder realizar de manera automática la instrumentación de librerías en aplicaciones Java con OpenTelemetry (“zero instrumentation”).


**<u> PARA REALIZAR LA INSTALACIÓN ES REQUERIDO SALIDA A INTERNET Y CONTAR CON EL COMANDO HELM </u>**

1. Instalación de cert-manager
a. Ejecutamos el siguiente comando:
> kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.2/cert-manager.yaml

b. Validamos que se tenga el namespace de cert-manager con el comando:
>kubectl get ns

c. Validamos que estén ok los 3 pods en el namespace
![pods](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/pods.png)

2. Validamos si hay una instalación existente del operador:

>Kubectl get pods -n opentelemetry-operator-system

b. **<u> Solo si existe y cuenta con un pod aplicamos el siguiente comando: </u>**
> kubectl delete -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml --force

3. Ejecutamos el siguiente comando y validamos que se cree el namespace **opentelemetry-operator-system** junto con un pod
> kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml

![namespace_OTEL](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/namespace_OTEL.png)

4. Para validar la instalación del operador ejecutamos el comando:
>kubectl get crd opentelemetrycollectors.opentelemetry.io

Se obtendrá una salida como la siguiente:

![salida_consola](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/salida_consola.png)

**<u> En caso de no obtener esta salida, repetir la instalación del operador (punto 2 y 3) </u>**

**<u>Si ya se cuenta con la instrumentación de los microservicios validar los siguientes puntos </u>**

- Revisar que se tenga instrumentado el namespace:
> kubectl get instrumentation -n namespace

![instrumentación](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/instrumentación.png)

**<u> En caso de no tener ninguna instrumentación revisar el documento “CC_13022025_Instrumentación_Java_OpenTelemetry” punto 6 </u>** 

- Revisar que el microservicio tenga la etiqueta:
> instrumentation.opentelemetry.io/inject-java: "true"

- Al reiniciar el microservicio revisar que en los logs se vea la siguiente linea al iniciar el servicio.

![javaagent](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/javaagent.png)
