![api.o](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/api.o.png)

# Control de Cambio

Proyecto:

- Configuración Opentelemetry Collector

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
<td>Documentar el procedimiento de configuración del OpenTelemetry Collector.</td>
</tr>
</table>

La información contenida en este documento, es Propiedad del Proyecto por lo que no deberá ser divulgada, duplicada o dada a conocer, parcial o totalmente, fuera de alcance del Proyecto sin una autorización por escrito.

>SECCIÓN III: CONTENIDO
1. [SECCIÓN I: HISTORIAL DE CAMBIOS AL DOCUMENTO](#SECCIÓN-I:-HISTORIAL-DE-CAMBIOS-AL-DOCUMENTO)
2. [SECCIÓN II: INFORMACIÓN DEL DOCUMENTO Y ACEPTACIÓN](#SECCIÓN-II:-INFORMACIÓN-DEL-DOCUMENTO-Y-ACEPTACIÓN)
3. [SECCIÓN III: CONTENIDO](#SECCIÓN-III:-CONTENIDO)
4. [SECCIÓN IV: AGREGAR SAMPLEO DE DATOS EN OPENTELEMETRY COLLECTOR](#SECCIÓN-IV:-AGREGAR-SAMPLEO-DE-DATOS-EN-OPENTELEMETRY-COLLECTOR)

>SECCIÓN IV: Agregar Sampleo de datos en Opentelemetry Collector

Procedimiento para la configuración de Opentelemetry Collector, esta configuración registra el porcentaje de trazas que serán enviadas a la plataforma de elasticsearch.

1. Abrimos el archivo yaml de despliegue del Opentelemetry-Collector.
2. Editamos la sección de processors y colocamos el siguiente contenido:

>probabilistic_sampler:
>sampling_percentage: 15

3. Revisamos que el contenido quede como lo siguiente:

**<u> Si existe un apartado de nombre tail_sampling debemos de eliminarla o comentarla. </u>**

![tail_sampling](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/tail_sampling.png)

4. Al final del archivo veremos una sección de nombre **service > pipeline > traces > processors**, en esta sección agregamos el siguiente contenido dentro de los corchetes al inicio

**<u> Si existe un elemento de nombre tail_sampling, debemos eliminarla.  </u>**

![sampling](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/sampling.png)

5. Guardamos el archivo y reiniciamos el servicio.
