---
title: "Agentes en elasticsearch"
description: "Describe las caracteristicas y principales configuraciones de un agente de elasticsearch"
customSlug: "agentes-en-elasticsearch"
folder: "Observabilidad"
subfolder: "Pilares de la observabilidad"
---
# Consideraciones de Metricbeat
Para la instalación del agente de Metricbeat podemos seguir el paso a paso de la [pagina oficial](https://www.elastic.co/docs/reference/beats/metricbeat/metricbeat-installation-configuration), pero en esta sección hablaremos sobre los puntos a revisar durante la instalación o las consideraciones que hay que tener en cuenta.
## Instalación por APT o YUM (recomendado)
Esta es la instalación más recomendada en servicios OnPremise ya que ofrece los siguientes beneficios:
* Durante la instalación se crea el usuario de metricbeat, lo cual evita que su ejecución se realice con un usuario root.
* La instalación se divide en diferentes carpetas que permiten organizar logs, configuraciones y ejecutables.
* Los upgrades del agente se realizan de manera más simple, ya que solo necesitamos ejecutar el sistema de paqueterías, al ser aplicada la actualización **los archivos de configuración no son reemplazados** evitando tener que configurar nuevamente como pasa con otros metodos de instalación.
* El metodo de ejecución del agente se aplica a nivel del systemctl, lo que permite reiniciar en automatico el servicio en caso de un reinicio del equipo
## Archivo Zip:
Esta instalación es viable para equipos donde una salida a internet no sea posible.
* Todo el contenido se encuentra en una sola ruta, por lo que es rocomendable realizar una segmentación de las carpetas como la que se realiza con APT o YUM
 * Realizar un upgrade requiere descargar la nueva versión y ejecutar el servicio con un parámetro que contenga la ruta de la carpeta de configuraciones anteriores o en su defecto copiar la configuración anterior a la nueva carpeta.
 * Para ejecutar el servicio como un systemctl requiere crear un script personalizado o ejecutar mediante un nohup, pero esto tiene como consecuencia levantar el servicio cada que se sufra un reinició del equipo.
## Kubernetes:
La instalación del agente puede ser aplicada directamente sin la instalación del operador de elasticsearch, esto es viable si solo se instalará este agente y no el resto del stack de elastic.

* Si contamos con todo el stack de elastic en el cluster se recomienda que la instalación del agente se realice por medio del operador de ECK, esto facilita la configuración de recursos como certificados, servicios, secrets y temas de comunicación.
* La instalación del agente de metricbeat se aplica a nivel de un Demonset con lo cual se creará un pod de metricbeat en cada nodo de kubernetes que se tenga.
