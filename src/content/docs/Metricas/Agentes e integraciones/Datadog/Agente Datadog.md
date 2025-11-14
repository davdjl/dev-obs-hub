---
title: "Agentes en elasticsearch"
description: "Describe las caracteristicas y principales configuraciones de un agente de elasticsearch"
customSlug: "agentes-en-elasticsearch"
folder: "Observabilidad"
subfolder: "Metricas Datadog"
---
# Consideraciones de Datadog agent
Para la instalación del agente de Datadog podemos seguir el paso a paso de la [pagina oficial](https://docs.datadoghq.com/es/agent/?tab=Linux), pero en esta sección hablaremos sobre los puntos a revisar durante la instalación o las consideraciones que hay que tener en cuenta.
## Instalación por medio de Fleet
Esta es la instalación recomendada por Datadog en servicios OnPremise y facilita la administración de todos los agentes desde la plataforma de datadog.
Para poder realizar esta instalación debemos tener en cuenta:

 - Versión del agente de Datadog 7.66 o mayor.
 - El usuario que administre estos agentes deberá contar con los siguientes roles
 - org_management: este rol permite realizar ciertas configuraciones en la organización de Datadog.
 - api_keys_write: Permite crear y renombrar api keys de la organización.
 - El agente debe ser instalado de forma manual en el equipo mediante un script, posterior a esto, es posible actualizar desde el portal de Datadog.
 - Datadog tiene un apartado particular llamado DogStatsD, este apartado permite crear un script para la creación y envio de metricas custom, esto se aplica principalmente con metricas que no estan definidas en el agente, si es habilitada esta función tomar en cuenta que Datadog realiza un cobro diferente a las metricas estándar.
# Instalación Kubernetes
Para la instalación en kubernetes debemos tomar en cuenta que la instalación lleva un proceso diferente a fleet y se recomiendan otros aspectos como:
 - Instalación de un Datadog Cluster Agent el cual permite recopilar datos de monitorización del clúster, este agente solo administra y gestiona los agentes de datadog encargados de recolectar las metricas.
 - La instalación del agente que recaba las metricas crea un pod del agente dentro de cada nodo de kubernetes.
 - La recolección de metricas se realiza por default al instalar el agente y no puede se deshabilitado.
 - Datadog realiza un cobro por cada nodo de kubernetes que se monitorea, tomar en cuenta que en un servicio de autoscaling puede haber un incremento de costos por el numero de nodos que se generen. 
