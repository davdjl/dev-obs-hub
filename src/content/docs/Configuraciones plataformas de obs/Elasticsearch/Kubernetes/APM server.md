# Instalación APM Server en kubernetes
APM Server es un servicio del Stack de elastic que nos permite recibir los eventos recolectados por los agentes de APM de elastic.

# Instalación paso a paso
Para la instalación utilizaremos el mismo operador de ECK que vimos en la instalación de elasticsearch. Crearemos el archivo apm_server.yaml con el siguiente contenido:

```sh
apiVersion: apm.k8s.elastic.co/v1
kind: ApmServer
metadata:
  name: apm-server
  namespace: elastic
spec:
  #Colocar la misma versión que elasticsearch
  version: 9.0.0
  count: 1
  #Colocar el nombre del servicio de elastic en el mismo cluster de kubernetes
  elasticsearchRef:
    name: elasticsearch
  #Colocar el nombre del servicio de kibana en el mismo cluster de kubernetes
  kibanaRef: 
    name: kibana
```

1. Creamos el servicio de APM con el comando 
```sh
kubectl apply -f apm_server.yaml
```
2. Podemos crear un port-forward para validar que el servicio este correcto y usamos el servicio con terminación *-apm-http*
```sh
kubectl port-forward service/apm-server-apm-http 8200 -n elastic
```
3. Obtenemos el token de seguridad del secret con terminación *-apm-token* con el comando:
```sh
kubectl get secret/apm-server-apm-token -n elastic -o go-template='{{index .data "secret-token" | base64decode}}'
```
4. Comprobamos el estado del servidor con el comando
```sh
curl https://localhost:8200 -H 'Authorization: Bearer $Token' --insecure
```
5. Debemos tener un mensaje como el siguiente
```json
{
    "build_date": "2025-03-25T15:32:57Z",
    "build_sha": "8a80b42fe664d49038342d721bb6bc4b25f52e47",
    "publish_ready": true,
    "version": "9.0.0"
}
```
## Eliminar APM server
Para eliminar el apm server usamos el comando
```sh
kubectl delete -f apm_server.yaml
```
