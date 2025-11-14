---
title: "Recolección de metricas mediante agentes"
description: "Se describen los agentes que permiten la recolección de metricas y las consideraciones que se deben tener en cuenta para una correcta implementación"
customSlug: "recoleccion-de-metricas-mediante-agentes"
folder: "Observabilidad"
subfolder: "Metricas"
---

# Agentes
Para realizar la recolección de metricas existen multiples metodos de obtención que podemos estar utilizando, el más común es utilizar agentes desarrollados por las propias tecnologías de observabilidad.
## Consideraciones
Los agentes son desarrollados para ser lo menor intrusivo y más ligeros posible, pero como toda aplicación tiene un consumo de recursos, además de esto, debemos considerar que:
* Los agentes comúnmente obtienen las metricas del equipo por default, pero existen otros monitoreos que podemos habilitar en el mismo agente como metricas de bases de datos, servicios de encolamiento o servicios cloud. Para ver la capacidad de cada uno revisar la sección de cada tecnología.
* Los agentes son utilizados comúnmente para el monitoreo de servicios OnPremise ya que requieren ser instalados en el mismo equipo que genera las metricas.
* Para ciertos recurosos el agente también tiene la capacidad de moniorear otros equipos, esto funciona para casos donde por temas de seguridad no es posible instalar el agente en el mismo equipo a monitorear.
* No utilizar tecnologías de diferentes plataformas para la recolección y almacenamiento de los datos, por ejemplo: utilizar el agente de metricbeat de elasticsearch para la recolección de datos, teniendo un almacenamiento en Datadog.
  - **El agente de Opentelemetry es el único que mantiene un estándar para el nombrado de los campos y que puede ser utilizado en la mayoria de plataformas de observabilidad.**
* En la mayoría de los casos estos agentes no almacenan información, solo recolectan y envían la data, pero debemos tener en consideración revisar si los agentes tienen habilitados una queue de eventos en caso de que el destino de la información no pueda recibir la data.

# Integraciones
Las integraciones son funciones propias de la plataforma de observabilidad, estas nos permiten (en algunos casos) realizar la obtención de metricas sin la necesidad de instalar un agente, esto facilita la configuración y recolección de metricas desde la propia plataforma. 

# Tiempo de ejecución
Estos agentes generalmente estan configurados para obtener metricas del equipo cada 10 segundos.
