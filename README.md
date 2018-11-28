# sd2018b-project
**Icesi University**  
**Course:** Distributed Systems   
**Professor:** Daniel Barragán C.  
**Subject:** Artifact building for Continous Delivery  
**Email:** daniel.barragan at correo.icesi.edu.co  
**Students:**  
Juan Camilo Swan - A00054620  
Carlos Andres Afanador - A00054652  
Tomas Julian Lemus - A00054616  
Luis Alejandro Trochez - A00054648  
**Git URL:** https://github.com/juanswan13/sd2018b-project/tree/jcswan/develop  

## Objetivos
* Identificar los componentes de un cluster de kubernetes
* Desplegar un cluster de kubernetes
* Desplegar aplicaciones en un cluster de kubernetes empleando sus propiedades
de volumenes persistentes, balanceo de carga y descubrimiento de servicio
* Diagnosticar y ejecutar de forma autónoma las acciones necesarias para lograr infraestructuras estables

## Descripción
En este proyecto se va a realizar el despliegue de un cluster de kubernetes en cuatro nodos independientes (un maestro y tres nodos de trabajo). Para ello se realiza la instalación y configuración de las dependencias necesarias en AWS, y la configuración de los demás servicios complementarios que permiten el funcionamiento del kluster de kubernetes.

Para la prueba del funcionamiento del cluster se va a realizar el despliegue de un pod ejecutando un servicio web. La especificación del deployment debe cumplir con los siguientes requerimientos:

* Cantidad de replicas: 3
* Asignar una etiqueta que identifique al pod (En este caso el servicio será un servidor nginx, por lo tanto la etiqueta será nginx)

La aplicación desplegada se expone por lo tanto puede ser accedida a través de un navegador o cliente de consola (curl, wget).

## Desarrollo
El presente proyecto de cluster de kubernetes se realizo en la nube utilizando los servicios de AWS. Para esto, inicialmente se configuraron los permisos de la cuenta para asegurar que el despliegue fuera posible. Luego se configuró un Rol que posteriormente se utiliza para brindarle ciertos permisos a la instancia EC2 que controla el kubectl. Una vez se tiene los roles listos se puede comenzar.
  
  
1. Para empezar lo primero que debe hacerse es seleccionar un nombre de dominio. (Para este proyecto se ha utilizado: dsproject.icesi.edu.co)
2. Una vez se ha establecido un nombre de dominio se debe crear una "Hosted zones" en el servicio de AWS llamado "Route 53"; los parametros para crear esta Hosted Zone son el nombre del dominio que se estableció en el punto anterior, el tipo de zona (en este caso al ser una zona interna se debe elegir 'private hosted zone') y por último la VPC en la que se crearán el cluster.
![][1]  
3. Ahora se crea en S3 un Bucket, este Bucket contiene todos los archivos de configuracion del cluster, por este motivo lo he llamado cluster.dsproject.icesi.edu.co
![][2]  
4. Crear una instancia EC2 Ubuntu server 18.04
5. Ingresar a dicha instancia mediante PuTTY
6. Crear certificados ssh en este servidor utilizando el comando: ```  $ ssh-keygen  ``` Presionar la tecla enter hasta que los certificados se hayan generado. Estos permitiran que lo haya conflictos más adelante con la manipulación del cluster.
![][3] 
7. Se debe ahora instalar la AWSCLI ``` $ pip install awscli --upgrade --user ``` (para poder instalarlo se debe primero tener instalado python2 y su respectivo PIP).
![][4] 
8. Se debe ahora instalar la kubectl que es la interfaz de comandos que permite ejecutar comandos contra clusters de kubernetes.  

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
``` 
![][5] 
9. Se debe ahora instalar kops que es actualmente el mejor complemento para manejar clusters de kubernetes en AWS.
```
$ wget https://github.com/kubernetes/kops/releases/download/1.8.1/kops-linux-amd64
$ chmod +x kops-linux-amd64
$ sudo mv kops-linux-amd64 /usr/local/bin/kops
```
![][6] 
10. Exponer la variable de ambiente KOPS_STATE_STORE, esta variable de ambiente es utilizada por kops para tener la referencia del bucket de S3 donde se encuentran los archivos de configuración del cluster (Inicialmente el Bucket no tiene ningun archivo, estos se crean de forma automatica).
```
$ export KOPS_STATE_STORE=s3://cluster.dsproject.icesi.edu.co
```
11. En este punto ya se puede crear el cluster de kubernetes. Para eso se utiliza el siguiente comando:
```
$ kops create cluster --cloud=aws --zones=us-east-1a --name=dscluster.dsproject.icesi.edu.co --dns-zone=dsproject.icesi.edu.co --dns private
```
donde los parametros son los siguientes: 
* --cloud: proveedor de nube (en este caso AWS)
* --zones: zonas en las que el cluster puede desplegar sus nodos (En este caso será en la zona us-east-1a, que correspondia a US East, N Virginia).
* --name: Nombre que se le pondra al cluster, la convención es un nombre seguido de la zona dns (en este caso dscluster.zona)
* --dns-zone: zona dns que se ha establecido (En este caso la zona establecida fue dsproject.icesi.edu.co)
* --dns: tipo del dns. (Como se mencionó en el paso 2, esta es una private hosted zone por lo que el tipo del dns es private).
![][7] 
Una vez el cluster ha sido creado, en el Bucket de S3 se generan de forma automatica todos los archivos de configuración que permiten que el cluster se despliegue y comporte de forma efectiva.

| Antes de crear el cluster  | Después de crear el cluster |
|------|------|
| ![][2] | ![][8] | 
12. En este punto el cluster solo se ha creado pero aun no se ha inicializado. Para inicializar el cluster se debe ejectura el siguiente comando:
```
$ kops update cluster nombre_del_cluster --yes
```
![][9]
Una vez que este comando es ejecutado, el cluster de kubernetes se despliega y las instancisa EC2 correspondientes a los nodos empiezan a ser levantadas de forma automatica. (El valor de los nodos de trabajo por defecto es 2, y un nodo maestros. Pero para cumplir con el objetivo de tres nodos de trabajo; se ha agregado uno). En la siguiente foto se puede evidenciar las instancias de los nodos en estado activo  
  
| Antes de iniciar el cluster  | Después de iniciar el cluster |
|------|------|
| ![][10] | ![][11] | 

13. En este punto ya el cluster esta listo y los nodos se estan todos ejecutando. No hay por el momento ningún servicio funcionando en este cluster. Los nodos se pueden ver con el comando:
```
$ kubectl get nodes
```
![][12]

14. Para ver los pods se utiliza el comando: ``` $ kubectl get pods  ``` sin embargo, en este punto este comando no retorna nada ya que no hay ningún servicio en ejecución.  
![][14]

## Demostración

Para demostrar el funcionamiento del proyecto se ha implementado sobre el cluster de kubernetes el servicio de nginx. 
Para ejecutar dicho servicio se ha utilizado el comando:
```
kubectl run sample-nginx --image=nginx --replicas=3 --port=80
```
Con esta linea de comando se ejecuta el servicio con nombre nginx, la imagen que utiliza para lanzar el servicio es la imagen nginx, se le indica al cluster que debe generar tres replicas y se realiza un mapeo al puerto 80 para poder exponer posteriormente el servicio al publico.
En este punto el servicio ya se encuentra en ejecución pero no se puede aun utilzar publicamente.

1. Para poder utilizar el servicio desde un cliente web se debe exponer. Esto se logra con el comando:
```
kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
```  
![][15]

2. El servicio ya se encuentra expuesto, mediante el comando ``` kubectl get services -o wide ``` Se obtiene la información de los servicios expuestos y se nos da una URL y una IP-pública por donde se puede acceder al servicio. en las imágenes a continuación se demuestra que el servicio funciona tanto en un curl hecho en la misma maquina EC2 donde esta el kubectl y funciona desde el navegador web de un computador portatil personal.
![][16]
![][17]

3. Por último, se debe demostrar que cuando un nodo muere el cluster redistribuye los pods a otros nodos saludables y mantiene las replicas que se le indicó desde un principio. Para esto se terminó forzosamente una EC2 perteneciente a uno de los nodos de trabajo y se verificó que los pods estuvieras activos. El resultado a la prueba anterior fue positivo y como evidencia se tienen las siguientes imágenes.
![][18]
![][19]

## References  
https://www.youtube.com/watch?v=IImQrJWbaDo  
https://jee-appy.blogspot.com/2017/10/setup-kubernetes-cluster-kops-aws.html  
https://kubernetes.io/docs/setup/custom-cloud/kops/
https://cloudacademy.com/blog/kubernetes-operations-with-kops/
https://stackoverflow.com/questions/50248179/how-to-add-an-node-to-my-kops-cluster-node-in-here-is-my-external-instance
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

[1]: images/Route53BeforeCluster.PNG
[2]: images/emptyS3BeforeCluster.PNG
[3]: images/1_CertificadosSSH-EC2_Instance.PNG
[4]: images/2_AWSCLI-Installed.PNG
[5]: images/3_kubectl-installed.PNG
[6]: images/4_kops-installed.PNG
[7]: images/6_ClusterCreationCommand.PNG
[8]: images/9_S3AfterClusterCreation.PNG
[9]: images/11_ClusterInit.PNG
[10]: images/10_EC2-InstancesBeforeClusterInit.PNG
[11]: images/13_EC2-InstacesAfterClusterInit.PNG
[12]: images/14_ClusterNodes.PNG
[13]: images/15_PodsBeforeAnyServiceRun.PNG
[14]: images/16_ServiceCreated_PodsRunning.PNG
[15]: images/17_ServiceExposed.PNG
[16]: images/18_CurltoService.PNG
[17]: images/19_AlsoWorksInWebBrowser.PNG
[18]: images/20_NodoTerminado.PNG
[19]: images/21_PodsFuncionandoNormal.PNG
