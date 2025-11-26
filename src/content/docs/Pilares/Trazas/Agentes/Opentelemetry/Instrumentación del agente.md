![api.o](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/api.o.png)

# Control de Cambio

Proyecto:

- Habilitar la instrumentación de aplicaciones Java con OpenTelemetry

Elaborado por:

### Api.o CONSULTING

FEBRERO 13 2025

## SECCIÓN I: HISTORIAL DE CAMBIOS AL DOCUMENTO

<table>
<tr>
<td>Fecha</td>
<td>Versión</td>
<td>Descripción</td>
<td>Autor</td>
</tr>
<tr>
<td>13/02/2025</td>
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
<td>Documentar el control de cambio para Habilitar la instrumentación de aplicaciones Java con OpenTelemetry.</td>
</tr>
</table>

La información contenida en este documento, es Propiedad del Proyecto por lo que no deberá ser divulgada, duplicada o dada a conocer, parcial o totalmente, fuera de alcance del Proyecto sin una autorización por escrito.

>SECCIÓN III: CONTENIDO
1. [SECCIÓN I: HISTORIAL DE CAMBIOS AL DOCUMENTO](#SECCIÓN-I:-HISTORIAL-DE-CAMBIOS-AL-DOCUMENTO)
2. [SECCIÓN II: INFORMACIÓN DEL DOCUMENTO Y ACEPTACIÓN](#SECCIÓN-II:-INFORMACIÓN-DEL-DOCUMENTO-Y-ACEPTACIÓN)
3. [SECCIÓN III: CONTENIDO](#SECCIÓN-III:-CONTENIDO)
4. [SECCIÓN IV: INSTALACIÓN](#SECCIÓN-IV:-INSTALACIÓN)

Procedimiento para habilitar la instrumentación de aplicaciones Java con OpenTelemetry Operator de manera automática (“zero instrumentation”).

1. Instalamos el operador:

>kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml

2. Creamos el Namespace para el colector:

>kubectl create ns otel

3. Creamos un nuevo archivo con el contenido del despliegue del collector:

>apiVersion: opentelemetry.io/v1beta1
>kind: OpenTelemetryCollector
>metadata:
>name: telemetry
>namespace: otel
>spec:
>image: otel/opentelemetry-collector-contrib:0.127.0
>replicas: 1
>config:
>receivers:
>otlp:
>protocols:
>grpc:
>endpoint: 0.0.0.0:4317
>http:
>endpoint: 0.0.0.0:4318
>processors:
>tail_sampling:
>decision_wait: 10s
>num_traces: 100
>policies: [
>{
>name: trace-status-policy,
>type: status_code,
>status_code: {status_codes: [ERROR]}
>},
>{
>name: probabilistic-policy,
>type: probabilistic,
>probabilistic: {sampling_percentage: 30}
>}
>]
>filter:
>error_mode: ignore
>traces:
>span:
>\- attributes["http.target"] == "/inbursa/home"
>attributes:
>actions:
>\- key: environment
>value: prod
>action: insert
>memory_limiter:
>check_interval: 1s
>limit_percentage: 75
>spike_limit_percentage: 15
>batch:
>send_batch_size: 10000
>timeout: 10s
>exporters:
>otlphttp:
>endpoint: http://apm-server-inbursa-apm-http.default:8200
>headers:
>Authorization: "Bearer pdm9ukI5145j9uG8W0Gr4S4N"
>tls:
>insecure: true
>service:
>pipelines:
>traces:
>receivers: [otlp]
>processors: [memory_limiter, batch, filter, tail_sampling, attributes]
>exporters: [otlphttp]

4. Aplicamos el deploy con:

> kubectl apply -f archive.yaml

5. Creamos un despliegue de nginex

6. Creamos la instrumentación para java, esta instrumentación se deberá aplicar por cada namespace donde se implementará el agente

>kubectl apply -f - <<EOF
>apiVersion: opentelemetry.io/v1alpha1
>kind: Instrumentation
>metadata:
>name: telemetry-instrumentation
>namespace: otel
>spec:
>exporter:
>endpoint: "http://telemetry-collector.otel:4317"
>propagators:
>\- tracecontext
>\- baggage
>\- b3
>sampler:
>type: parentbased_traceidratio
>argument: "1"
>EOF

7. Agregamos la anotación y variables en el deployment o deployments correspondientes.

>annotations:
>instrumentation.opentelemetry.io/inject-java: "true"
>env:
>\- name: OTEL_METRICS_EXPORTER
>value: "none"
>\- name: OTEL_LOGS_EXPORTER
>value: "none"
>\- name: OTEL_TRACES_EXPORTER
>value: otlp
>apiVersion: apps/v1
>kind: Deployment
>metadata:
>name: java-sample
>namespace: java-otel
>spec:
>replicas: 1
>selector:
>matchLabels:
>app: java-sample
>template:
>metadata:
>labels:
>app: java-sample
>annotations:
>instrumentation.opentelemetry.io/inject-java: "true"
>spec:
>containers:
>\- name: java-sample
>image: pinakispecial/spring-boot-rest
>env:
>\- name: OTEL_METRICS_EXPORTER
>value: "none"
>\- name: OTEL_LOGS_EXPORTER
>value: "none"
>\- name: OTEL_TRACES_EXPORTER
>value: otlp
>ports:
>\- containerPort: 8080

![consola_1](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/consola_1png)

> kubectl describe otelinst -n otel

![consola_2](https://raw.githubusercontent.com/davdjl/dev-obs-hub/main/public/images/consola_2.png)
