---
title: "Recolección de trazas mediante agentes"
description: "Se describen los agentes que permiten la recolección de trazas y las consideraciones que se deben tener en cuenta para una correcta implementación"
customSlug: "recoleccion-de-trazas-mediante-agentes"
folder: "Observabilidad"
subfolder: "Pilares de la observabilidad"
---
# Agentes de APM
Los agentes de APM son librerías que son instaladas en las aplicaciones para realizar la recolección de metricas en el performance de la aplicación, estas librerías en algunos lenguajes de programación son instaladas a nivel del codigo de la aplicación. Otros lenguajes de programación permite la instalación de las librerías agregando solo los parámetros necesarios al momento de ejecutar la aplicación.
## Consideraciones
* En general las librerías de APM buscan ser lo más ligeras posibles de tal manera que no impacte en el consumo de recursos de la aplicación, pero debemos tener en cuenta que sí tiene un consumo de recursos.
* Las mayoría de las librerías tienen una configuración para el envio de la data de manera asíncrona. En los casos de que la librería este configurada de manera síncrona, hay que contemplar el tamaño de las queue para el almacenamiento de los mensajes sin entregar y el numero de reintentos
## Correlación de eventos
Para poder llegar a una grafica que integre el flujo completo de una solicitud es importante que la traza y spans estén correlacionados por un identificador único llamado traceID (dependiendo de la plataforma de observabilidad puede cambiar la estructura del nombre) este identificador además de otros elementos es propagado en los headers a cada uno de los elementos del sistema, para comprender como es creado este identificador y los demás elementos podemos revisar la pagina [W3C TraceContext](https://www.w3.org/TR/trace-context/).
## Sampling de datos
Una buena practica en trazas es aplicar una función llamada Sampling, el cual consiste en no recolectar el 100% de los eventos generados en la plataforma, ya que el tamaño de almacenamiento en un ambiente productivo puede llegar a varios Terabytes de datos. Dependiendo el sampling que se aplique es posible recolectar solo los eventos de errores.
