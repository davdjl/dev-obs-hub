---
title: "Agentes en elasticsearch"
description: "Describe las caracteristicas y principales configuraciones de un agente de elasticsearch"
customSlug: "agentes-en-elasticsearch"
folder: "Observabilidad"
subfolder: "Pilares de la observabilidad"
---
# Agentes en Elasticsearch
Existen dos tipos de agentes que pueden ser instalados y que permiten la recolección de metricas:
* Metricbeat: Es un agente dedicado a la recolección de metricas, permite habilitar módulos que definen el tipo de metricas a recolectar, siendo posible monitorear los recursos consumidos en bases de datos, datos del equipo o de sistema, servicios cloud, [etc](https://www.elastic.co/docs/reference/beats/metricbeat/metricbeat-modules)
* Agent beat: Es un agente que puede recolectar tanto metricas como logs, además de que es posible hasta cierto punto controlar ciertos aspectos del equipo donde se instala, tiene la ventaja de que puede ser administrado por la plataforma de kibana.
