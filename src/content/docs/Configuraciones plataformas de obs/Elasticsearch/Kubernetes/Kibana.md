# Instalación kibana en kubernetes
Para esta instalación seguiremos usando el operador que se instalo al inicio del tema **Instalación de elasticsearch kubernetes**
## Instalación paso a paso
A diferencia de elasticsearch, kibana es un servicio más simple de instalar ya que del yml proporcionado en la documentación de elasticsearch, solo se agregan limites de los recursos consumidos.
1. configuración del archivo kibana.yaml
```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana
  #Se recomienda sea el mismo namespace de elasticsearch
  namespace: elastic
spec:
  #Debe ser la misma versión que elasticsearch
  version: 9.0.0
  count: 1
  #Este nombre debe ser el mismo que se coloco en el archivo yaml de elasticsearch (al nivel del namespace)
  elasticsearchRef:
    name: "elasticsearch"
  podTemplate:
    spec:
      containers:
      - name: kibana
        env:
          - name: NODE_OPTIONS
            value: "--max-old-space-size=2048"
        resources:
          requests:
            memory: 1Gi
            cpu: 0.5
          limits:
            memory: 2.5Gi
            cpu: 2
```
2. Desplegamos el servicio con el comando
```sh
kubectl apply -f kibana.yaml
```
3. Al igual que con elastic revisamos los servicios, seleccionamos el que tenga terminación *-kb-http*, creamos el port-forward
```sh
kubectl port-forward service/kibana-kb-http 5601 -n elastic
```
4. Accedemos desde un navegador a https://localhost:5601 y usamos el mismo usuario y contraseña de elastic.

## Eliminar kibana
Para eliminar kibana usamos el comando
```sh
kubectl delete -f kibana.yaml
```
