# HELM API
* Helm API provides a RESTful API service for Helm-Tiller

## Introduction
Helm API provides REST endpoints for accessing Helm charts, releases, and repos. 

## Chart Details
This chart deploys a single instance of a Helm API pod on the master node of a Kubernetes environment. 

## Prerequisites
* Kubernetes 1.11.0 or later, with beta APIs enabled
* Custom IBM Tiller version v2.12.3
* IBM core services including auth-idp service, MongoDB, and management-ingress
* Cluster Admin role for installation

### PodSecurityPolicy Requirements

This chart requires a PodSecurityPolicy to be bound to the target namespace prior to installation. Choose either a predefined [`ibm-anyuid-psp`](https://ibm.biz/cpkspec-psp) PodSecurityPolicy or have your cluster administrator create a custom PodSecurityPolicy for you:
* Custom PodSecurityPolicy definition:

```
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: icp-helm-repo-psp
spec:
  allowPrivilegeEscalation: true
  fsGroup:
    rule: RunAsAny
  requiredDropCapabilities:
  - MKNOD
  allowedCapabilities:
  - SETPCAP
  - AUDIT_WRITE
  - CHOWN
  - NET_RAW
  - DAC_OVERRIDE
  - FOWNER
  - FSETID
  - KILL
  - SETUID
  - SETGID
  - NET_BIND_SERVICE
  - SYS_CHROOT
  - SETFCAP
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - emptyDir
  - projected
  - secret
  - persistentVolumeClaim
  forbiddenSysctls:
  - '*'
```

### Red Hat OpenShift SecurityContextConstraints Requirements

This chart requires a SecurityContextConstraints to be bound to the target namespace prior to installation. To meet this requirement there may be cluster-scoped, as well as namespace-scoped, pre- and post-actions that need to occur.

The predefined SecurityContextConstraints [`ibm-anyuid-scc`](https://ibm.biz/cpkspec-scc) has been verified for this chart. If your target namespace is not bound to this SecurityContextConstraints resource you can bind it with the following command:

`oc adm policy add-scc-to-group ibm-anyuid-scc system:serviceaccounts:<namespace>` For example, for release into the `default` namespace:
``` 
oc adm policy add-scc-to-group ibm-anyuid-scc system:serviceaccounts:default
```

* Custom SecurityContextConstraints definition:

```
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: icp-helm-repo-scc
readOnlyRootFilesystem: false
allowedCapabilities: []
allowHostPorts: true
seLinuxContext:
  type: RunAsAny
supplementalGroups:
  type: MustRunAs
  ranges:
  - max: 65535
    min: 1
runAsUser:
  type: MustRunAsNonRoot
fsGroup:
  type: MustRunAs
  ranges:
  - max: 65535
    min: 1
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
```

## Resources Required
* At least 556Mb of available memory
* At least 350m of available CPU
* At least a few GB of free storage

## Installing the Chart
Only one instance of helm-api should be installed on a cluster
* Include at the basic things necessary to install the chart from the Helm CLI - the general happy path
* Include setup of other items required
* Security privileges required to deploy chart (role, PodSecurityPolicy, etc)
* Include verification of the chart 
* Ensure CLI only and avoid any product-specific language used
* If installing via the Helm CLI on ICP v3.1.1 or earlier, then the user needs read-only access to the kube-system namespace or needs to be a cluster-admin.

To install the chart with the release name `my-release`:

```bash
$ helm install <chartname>.tgz --namespace kube-system --name helm-api --tls
```

The command deploys Helm API on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.


> **Tip**: List all releases using `helm list --tls`

### Verifying the Chart
View the catalog page on the UI. If visible then helm-api is functioning properly.

### Uninstalling the Chart

To uninstall/delete the deployment:

```bash
$ helm delete helm-api --purge --tls
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the <CHARTNAME> chart and their default values.
Parameter                                        | Description                                               | Default
------------------------------------------------ | --------------------------------------------------------- | --------------------
`replicaCount`                                   | number of pod replications                                | 1                   
`arch`                                           | Node architecture to deploy to                            | amd64
`nodeSelector.master`                            | run on master node                                        | true                   
`tolerations.NoSched.effect`                     | kubernetes pod toleration effect for NoSchedule           | NoSchedule                   
`tolerations.NoSched.key`                        | kubernetes pod toleration key for NoSchedule              | dedicated                   
`tolerations.NoSched.operator`                   | kubernetes pod toleration operator for NoSchedule         | Exists                   
`tolerations.CriticalAddonsOnly.key`             | kubernetes pod toleration key for CriticalAddonsOnly      | CriticalAddonsOnly                   
`tolerations.CriticalAddonsOnly.operator`        | kubernetes pod toleration operator for CriticalAddonsOnly | Exists                    
`helmapi.meta.namespace`                         | namespace for this chart                                  | kube-system                   
`helmapi.image.name`                             | helmapi container name                                    | helmapi                   
`helmapi.image.repository`                       | helmapi image path                                        | ibmcom/icp-helm-api     
`helmapi.image.tag`                              | helmapi image tag                                         | latest                   
`helmapi.image.pullPolicy`                       | helmapi pullpolicy                                        | IfNotPresent                   
`helmapi.livenessProbe.httpGet.path`             | helmapi livenessProbe path                                | /healthcheck                   
`helmapi.livenessProbe.httpGet.scheme`           | helmapi livenessProbe scheme                              | HTTP                   
`helmapi.livenessProbe.httpGet.port`             | helmapi livenessProbe port                                | 3000                   
`helmapi.livenessProbe.initialDelaySeconds`      | helmapi livenessProbe initial delay seconds               | 30                   
`helmapi.livenessProbe.periodSeconds`            | helmapi livenessProbe period seconds                      | 30                   
`helmapi.livenessProbe.timeoutSeconds`           | helmapi livenessProbe timeout seconds                     | 5                   
`helmapi.livenessProbe.failureThreshold`         | helmapi livenessProbe fail ure threshold                  | 5                   
`helmapi.livenessProbe.successThreshold`         | helmapi livenessProbe success threshold                   | 1                   
`helmapi.readinessProbe.httpGet.path`            | helmapi readinessProbe path                               | /healthcheck                            
`helmapi.readinessProbe.httpGet.port`            | helmapi readinessProbe port                               | 3000                                    
`helmapi.readinessProbe.httpGet.scheme`          | helmapi readinessProbe scheme                             | HTTP                                    
`helmapi.readinessProbe.initialDelaySeconds`     | helmapi readinessProbe initial delay seconds              | 5                                       
`helmapi.readinessProbe.periodSeconds`           | helmapi readinessProbe period seconds                     | 30                       
`helmapi.readinessProbe.timeoutSeconds`          | helmapi readinessProbe timeout seconds                    | 15                                       
`helmapi.readinessProbe.failureThreshold`        | helmapi readinessProbe failure threshold                  | 5                                       
`helmapi.volumeMounts.mongoCaName`               | helmapi mongoDB CA mount name                             | mongodb-ca-cert                            
`helmapi.volumeMounts.mongoCaMountPath`          | helmapi mongoDB CA mount path                             | /certs/mongodb-ca            
`helmapi.volumeMounts.mongoClientCertMountPath`  | helmapi mongoDB client cert mount path                    | /certs/mongodb-client                   
`helmapi.volumeMounts.mongoClientCertName`       | helmapi mongoDB client cert mount name                    | mongodb-client-cert        
`helmapi.volumeMounts.helmApiCertsName`          | helmapi certs mount name                                  | helmapi-certs                       
`helmapi.volumeMounts.helmApiCertsMountPath`     | helmapi certs mount path                                  | /etc/certs                             
`helmapi.env.CLUSTER_CA_DOMAIN`                  | cluster domain name                                       | mycluster.icp                           
`helmapi.env.CLUSTER_PORT`                       | cluster port                                              | 8443                                    
`helmapi.env.DBHOST`                             | mongoDB host name                                         | mongodb                                 
`helmapi.env.DBPORT`                             | mongoDB port                                              | 27017                                   
`helmapi.env.HTTPS_PROXY`                        | helmapi HTTPS proxy server                                | ""                                      
`helmapi.env.HTTP_PROXY`                         | helmapi HTTP proxy server                                 | ""                                      
`helmapi.env.ISICP`                              | helmapi running on icp                                    | true                                    
`helmapi.env.MONGO_ISSSL`                        | helmapi mongoDB use ssl                                   | true                                    
`helmapi.env.MONGO_SSL_CA`                       | helmapi mongoDB ssl CA path                               | /certs/mongodb-ca/tls.crt               
`helmapi.env.MONGO_SSL_CERT`                     | helmapi mongoDB ssl cert path                             | /certs/mongodb-client/tls.crt           
`helmapi.env.MONGO_SSL_KEY`                      | helmapi mongoDB ssl key path                              | /certs/mongodb-client/tls.key           
`helmapi.env.NO_PROXY`                           | helmapi urls for no-proxy                                 | mycluster.icp,mongodb,platform-identity-provider,icp-management-ingress, iam-pap,localhost,127.0.0.1
`helmapi.env.PROXY_EXTERNAL_ADDR`                | helmapi proxy external address for launch links           | proxy_external_address      
`helmapi.env.SET_TLS`                            | helmapi set tls                                           | true                                    
`helmapi.env.WLP_REDIRECT_URL`                   | helmapi wlp redirect url                                  | http://localhost:3000/auth/liberty/callback
`helmapi.secretKeyRef.clientIdSecretKey`         | helmapi secret key for client id                          | WLP_CLIENT_ID                           
`helmapi.secretKeyRef.clientIdSecretName`        | helmapi secret name for client id                         | platform-oidc-credentials               
`helmapi.secretKeyRef.clientSecretSecretKey`     | helmapi secret key for client secret                      | WLP_CLIENT_SECRET                       
`helmapi.secretKeyRef.clientSecretSecretName`    | helmapi secret name for client secret                     | platform-oidc-credentials 
`helmapi.secretKeyRef.dbSecretName`              | helmapi secret name for mongoDB                           | icp-mongodb-admin                       
`helmapi.secretKeyRef.dbSecretPassKey`           | helmapi secret key for mongoDB password                   | password                                
`helmapi.secretKeyRef.dbSecretUserKey`           | helmapi secret key for mongoDB user                       | user                                    
`helmapi.service.name`                           | helmapi service name                                      | helm-api                                
`helmapi.service.port`                           | helmapi service port                                      | 3000                                    
`helmapi.service.portName`                       | helmapi service port name                                 | helmapiport                             
`helmapi.resources.limits.cpu`                   | helmapi cpu limits                                        | 300m                                    
`helmapi.resources.limits.memory`                | helmapi memory limits                                     | 400Mi                                   
`helmapi.resources.requests.cpu`                 | helmapi cpu requests                                      | 200m                                    
`helmapi.resources.requests.memory`              | helmapi memory requests                                   | 300Mi             
`rudder.image.name`                              | rudder container name                                     | rudder                                  
`rudder.image.pullPolicy`                        | rudder image pull policy                                  | IfNotPresent                            
`rudder.image.repository`                        | rudder image path                                         | ibmcom/icp-helm-rudder                  
`rudder.image.tag`                               | rudder image tag                                          | latest                            
`rudder.ports.containerPort`                     | rudder container port                                     | 5000                                    
`rudder.ports.containerPortName`                 | rudder container port name                                | p5000                                   
`rudder.ports.protocol`                          | rudder container port protocol                            | TCP   
`rudder.env.RUDDER_CLIENT_CERTS`                 | rudder client certs                                       | /etc/certs                              
`rudder.env.RUDDER_TILLER_ADDRESS`               | tiller address                                            | tiller-deploy.kube-system:44134         
`rudder.env.USE_TLS`                             | rudder use tls                                            | true      
`rudder.volumeMounts.rudderCertsMountPath`       | rudder certs mount path                                   | /etc/certs                              
`rudder.volumeMounts.rudderCertsName`            | rudder certs mount name                                   | rudder-certs     
`rudder.resources.limits.cpu`                    | rudder cpu limits                                         | 150m                                    
`rudder.resources.limits.memory`                 | rudder memory limits                                      | 128Mi                                   
`rudder.resources.requests.cpu`                  | rudder cpu requests                                       | 100m                                    
`rudder.resources.requests.memory`               | rudder memory requests                                    | 128Mi    
`volumes.helmApiCertsName`                       | helmapi certs mount name                                  | helmapi-certs                           
`volumes.helmApiCertsSecretName`                 | helmapi certs secret                                      | helmapi-secret                          
`volumes.mongoCaCertName`                        | mongoDB CA mount name                                     | mongodb-ca-cert                         
`volumes.mongoCaCertSecretName`                  | mongoDB CA mount secret                                   | cluster-ca-cert                         
`volumes.mongoClientCertName`                    | mongoDB client cert mount name                            | mongodb-client-cert                     
`volumes.mongoClientCertSecretName`              | mongoDB client cert mount secret                          | cluster-ca-cert                         
`volumes.rudderCertsName`                        | rudder certs mount name                                   | rudder-certs                            
`volumes.rudderCertsSecretName`                  | rudder certs mount secret                                 | rudder-secret        
`auditService.name`                              | audit service container name                              | icp-audit-service 
`auditService.image.pullPolicy`                  | audit service image pull policy                           | IfNotPresent                  
`auditService.image.repository`                  | audit service image path                                  | ibmcom/icp-audit-service      
`auditService.image.tag`                         | audit service image tag                                   | 3.2.1                   
`auditService.config.enabled`                    | audit service container enabled                           | true        


## Storage
Chart and repo information is stored on volume mounts within the pod but persisted with MongoDB in the case of a pod crash or restart.

## Limitations
Only one instance of Helm API can run at a single time, and should be restricted to running only on the master node.

## Documentation

Information about Helm API's endpoints can be found here:
https://www.ibm.com/support/knowledgecenter/SSBS6K_3.2.0/apis/helm_apis.html