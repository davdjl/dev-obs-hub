# Instalación Elasticsearch en Kubernetees.
Para la instalación de elasticsearch en kubernetes estaremos utilizando un operador de elastic que nos ayuda a administrar multiples servicios del stack como:
* La creación de secrets
* La creación de un balanceador
* La creación de certificados
* La configuración de comunicación entre nodos de elasticsearch y el stack.

De esta manera solo nos ocuparemos de la configuración custom en cada nodo de elastic y en la preparación de cada yaml de despliegue.

## Instalación paso a paso del operador
Para la instalación del operador seguiremos los siguientes pasos:
1. Descargamos el agente en kubernetes con el comando: 
```sh 
kubectl create -f https://download.elastic.co/downloads/eck/3.0.0/crds.yaml
```
**Validamos que sea la última versión y se cumplan los prerrequisitos y consideraciones en [Instalación operador](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/install-using-yaml-manifest-quickstart)**

2. Veremos una salida en consola como la siguiente: ![operador_consola](/public/images/OperadorEKSDescargaElastic.jpg)
3. Ejecutamos el siguiente comando para crear el operador:
```sh
kubectl apply -f https://download.elastic.co/downloads/eck/3.0.0/operator.yaml
```
4. Veremos una salida en consola como la siguiente: ![operador_consola_instalado](/public/images/OperadorEKSInstaladoElastic.png)
5. Con el comando anterior se comando creará el namespace elastic-system y el pod del operador, lo podemos consultar con el comando:
```sh
kubectl get pods -n elastic-system 
```
6. Cuando veamos el status en Running tendremos finalizado la instalación del operador.

## Instalación de Elasticsearch
Hasta la creación de este documento se tiene la versión 9.0.0 de elasticsearch como última versión.
Para la instalación estaremos siguiendo la buena recomendación de elasticsearch sobre manejar un cluster con al menos 3 nodos maestros y diferentes roles para multiples nodos.

Durante la creación del cluster tendremos en cuenta los siguientes datos de apoyo:

![Roles de nodo](/public/images/RecursosNodosElastic.jpg)
![Memoria de nods](/public/images/RecursosHot-Warm-ColdElastic.jpg)

El 50% de la memoria debe ser utilizado para elasticsearch y no puede ser mayor a 30GB de RAM el resto del espacio se usa para cache y otros procesos de elastic

### Instalación paso a paso de elasticsearch
Dentro de la [documentación de elasticsearch](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-deployment-quickstart) encontraremos el siguiente comando para desplegar un nodo en elasticsaerch, este comando lo iremos modificando, dependiendo de nuestras necesidades, también es de utilidad mencionar que este comando lo podemos ir guardando en un archivo .yaml

1. Comando de un despliegue básico de elastic:
```sh
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 9.0.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
EOF
```
*Se recomienda colocar el contenido anterior en un archivo yaml (**elastic_cluster.yaml** para esta guía) para mantener un registro de lo aplicado en el cluster de kubernetes, el contenido debe abarcar desde ``apiVersion``  a ``false``*

2. Agregamos 5 nodos más a nuestro cluter para mantener una arquitectura de 3 nodos hot y 2 warm (esto es lo más básico en un ambiente productivo para poder aplicar un ciclo de vida de los datos) con las siguientes configuraciones:
   * 3 nodos hot con los roles de:
     * Master
     * Ingest
     * Transform
     * Data_hot
     * Data_content
   * 2 nodos warm con los roles de:
     * Ingest
     * Transform
     * Data_warm
3. Aplicaremos estas caracteristicas al yaml y agregaremos configuraciones de versión de elastic, almacenamientos y limites
```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  #Agregamos un valor identificable a este servicio para futuras referencias en kibana, beats, apm, etc.
  name: elasticsearch
  #Agregamos un namespace para tener ubicado estos servicios
  namespace: elastic
spec:
  #Definimos la versión de elastic
  version: 9.0.0
  nodeSets:
  #Especificamos un nombre a esta configuración de nodos
  - name: hot
    #Numero de nodos a crear con esta configuración
    count: 3
    config:
      #Roles aplicados al nodo
      node.roles: ["master", "ingest","transform","data_hot", "data_content"]
    podTemplate:
      spec:
        #Para ambientes productivos, elasticsearch recomienda esta configuración de initContainers que tiene relación con los privilegios en el sistema operativo en la que se despliega elasticsearch
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
            runAsUser: 0
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
        containers:
        - name: elasticsearch
          resources:
            #Aparta los siguientes recursos en el cluster de kubernetes para cada nodo que se crea
            requests:
              memory: 8Gi
              cpu: 0.5
            #Asegura que elastic no tenga un consumo mayor a estos limites
            limits:
              memory: 16Gi
              cpu: 2
    volumeClaimTemplates:
    - metadata:
        #Nombre del volumen **Este nombre no debe cambiar**
        name: elasticsearch-data
      spec:
        #Tipo de acceso al volumen
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            #Tamaño del volumen de almacenamiento para cada nodo
            storage: 100Gi
        #Tipo de almacenamiento
        storageClassName: standard
  - name: warm
    count: 1
    config:
      node.roles: ["ingest","transform","data_warm"]
    podTemplate:
      spec:
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
            runAsUser: 0
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
        containers:
        - name: elasticsearch
          resources:
            requests:
              memory: 4Gi
              cpu: 0.5
            limits:
              memory: 8Gi
              cpu: 1
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 500Gi
        storageClassName: standard
```
*Segun las necesidades del cluster los valores de cada recurso pueden cambiar, incluso entre ambientes dev, qa y prod. Tomar en cuenta las imagenes presentadas al inicio de **Instalación de Elasticsearch***

4. Desplegamos el servicio de elastic
```sh
kubectl apply -f elastic_cluster.yaml
```
5. Al tener todos los elementos en running, podemos consultar la salud del cluster con el comando
```sh
kubectl get elastic -n elastic
```
6. Para ingresar a elasticsearch se pedirá el usuario "elastic" y la contraseña la podemos obtener de los secrets
```sh
kubectl get secrets -n elastic
```
7. Para obtener la contraseña ejecutamos el comando
```sh
kubectl get secret elasticsearch-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' -n elastic
```
8. Para acceder al servicio de elastic consultamos los servicios con el comando
```sh
kubectl get services -n elastic
``` 
9. Utilizamos el que tiene terminación -es-elastic-user
10. Para acceder utilizamos el servicio con terminación *-es-http*, para temas de pruebas podemos usar un port-forward para poder acceder desde nuestro local.
```sh
kubectl port-forward service/elasticsearch-es-http 9200 -n elastic
```
11.  Utilizamos curl para comprobar el acceso
```sh
curl https://localhost:9200 -u elastic -p --insecure
```
*Utilizamos --insecure para no agregar los certificados y al dar enter nos pedira la contraseña*

## Configuración de nodos
Ya sea para agregar, eliminar o modificar nodos, podemos colocarlo directamente en el yaml que utilizamos y volver aplicar
```sh
kubectl apply -f elastic_cluster.yaml
```
No requiere que se detenga ningun servicio, pero debemos tomar en cuenta que durante la actualización se puede presentar un error sobre el attach de los volumenes, esto pasa si el nodo actualizado debe crear un nuevo pod, los volumenes solo pueden estar ligado a un solo pod y el error se da cuando el anterior pod aun no es eliminado completamente y el nuevo pod con el nodo actualizado ya esta creado, este error generalmente se soluciona en automatico cuando el anterior pod es eliminado completamente del sistema y el volumen deja de estar ligado al pod y permite que el nuevo pod se pueda ligar.
## Eliminar elasticsearch
Para eliminar los elementos de elasticsearch podemos usar el comando
```sh
kubectl delete -f elastic_cluster.yaml
```
Para una mejor limpieza podemos eliminar el namespace completamente o en su defecto eliminar manualmente elementos que puedan persistir como los PV.
