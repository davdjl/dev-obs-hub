---
title: "Recolección de metricas mediante agentes"
description: "Se describen los agentes que permiten la recolección de metricas y las consideraciones que se deben tener en cuenta para una correcta implementación"
customSlug: "recoleccion-de-metricas-mediante-agentes"
folder: "Observabilidad"
subfolder: "Pilares de la observabilidad"
---

# Agentes
Para realizar la recolección de metricas existen multiples metodos de obtención que podemos estar utilizando, el más común es utilizar agentes desarrollados por las propias tecnologías de observabilidad.
## Consideraciones
Los agentes son desarrollados para ser lo menor intrusivo y más ligeros posible, pero como toda aplicación tiene un consumo de recursos.
* Los agentes son utilizados comúnmente para el monitoreo de servicios OnPremise ya que requieren ser instalados en el mismo equipo que genera las metricas.
* No utilizar agentes de diferentes plataformas para la recolección y almacenamiento de los datos, por ejemplo: utilizar el agente de metricbeat de elasticsearch para la recolección de datos, teniendo un almacenamiento de la información en Datadog.
* El único agente que mantiene un estándar para el nombrado de los campos y que puede ser utilizado en cualquier plataforma de observabilidad es el de Opentelemetry.
* En la mayoría de los casos estos agentes no almacenan información, solo recolectan y envían la data.
## Tiempo de ejecución
Estos agentes generalmente estan configurados para obtener metricas del equipo cada 10 segundos 
