# Annotations

## Ingress annotations

The following are the Ingress annotations supported by Citrix:

|**Annotations**|**Possible value**|**Description**|**Default**|
|---------------|------------------|---------------|-----------|
|ingress.citrix.com/frontend-ip| IP address | Use this annotation to customize the virtual IP address (VIP). This IP address is configured in Citrix ADC as VIP. The annotation is mandatory if you are using Citrix ADC VPX or MPX. </br>**Note:** Do not use the annotation if you want to use the Citrix ADC IP address as VIP. | Citrix ADC IP address is used as VIP. |
|Ingress.citrix.com/frontend-ipset-name|Name of the IPSET |Use this annotation to specify the IPSET name for the frontend configuration. The IPSET is bound to the CS virtual server. Use this annotation with the `ingress.citrix.com/frontend-ip`. </br>**Note:** The IPSET name that you specify in the annotation should already be configured in Citrix ADC.| NA|
|ingress.citrix.com/secure-port|Port number |Use this annotation to configure the port for HTTPS traffic. This port is configured in Citrix ADC as a port value for the corresponding Content Switching (CS) virtual server.| `443`|
|ingress.citrix.com/insecure-port| Port number | Use this annotation to configure the port for HTTP, TCP, or UDP traffic. This port is configured in Citrix ADC as a port value for the corresponding CS virtual server.| `80` |
|ingress.citrix.com/insecure-termination| `allow`, `redirect`, or `disallow` |Use `allow` to allow HTTP traffic, Use `redirect` to redirect the HTTP request to HTTPS, or Use `disallow` if you want to drop the HTTP traffic.</br></br> For example: `ingress.citrix.com/insecure-termination: "redirect"`| `disallow` |
|ingress.citrix.com/secure-backend|In JSON form, list of services for secure-backend |Use `True`, if you want to establish secure HTTPS between Citrix ADC and the application, Use `False`, if you want to establish insecure HTTP connection Citrix ADC to the application.</br> </br>For example: `ingress.citrix.com/secure-backend: {"app1":"True", "app2":"False", "app3":"True"}`| `False`|
|kubernetes.io/ingress.class|ingress class name| It is a way to associate a particular ingress resource with an ingress controller.</br> </br>For example: `kubernetes.io/ingress.class:"Citrix"` | Configures all ingresses |
| ingress.citrix.com/secure-service-type | `ssl` or `ssl_tcp` | The annotation allows L4 load balancing with SSL over TCP as protocol. Use `ssl_tcp`, if you want to use SSL over TCP. | `ssl` |
|ingress.citrix.com/insecure-service-type| `http`, `tcp`, `udp`, `sip_udp`, or `any` | The annotation allows L4 load balancing with tcp/udp/sip_udp any as protocol. Use `tcp`, if you want TCP as the protocol. Use `udp`, if you want UDP as the protocol.| `http` |
|ingress.citrix.com/path-match-method | `prefix` or `exact` | Use this annotation for ingress path matching. </br>-  Use `prefix` for Citrix ingress controller to consider any path string as a prefix expression.</br> - Use `exact` for the Citrix ingress controller to consider the path as an exact match.</br></br> For example, the `ingress.citrix.com/path-match-method: "prefix"` annotation defines the Citrix ingress controller to consider any path string as a prefix expression. | `prefix` |
| ingress.citrix.com/deployment | `dsr` | Use this annotation to create Direct Server Return (DSR) configuration on Citrix ADC. For example, the `ingress.citrix.com/deployment: "dsr"` annotation creates DSR configuration on the Citrix ADC. |

## Smart annotations for Ingress

Smart annotation is an option provided by the Citrix ingress controller to efficiently enable Citrix ADC features using the Citrix ADC entity name. The Citrix ingress controller converts the Ingress in Kubernetes to a set of Citrix ADC objects. You can efficiently control these objects using smart annotations.

!!! Info "Important"
    To use smart annotations, you must have good understanding of Citrix ADC features and their respective entity names. For more information on Citrix ADC features and entity names, see [Citrix ADC Documentation](https://docs.citrix.com/en-us/citrix-adc/12-1.html).

Smart annotation takes JSON format as input. The key and value that you pass in the JSON format must match the Citrix ADC NITRO format. For more information on the Citrix ADC NITRO API, see [Citrix ADC 12.1 REST APIs - NITRO Documentation](https://developer-docs.citrix.com/projects/netscaler-nitro-api/en/latest/).

For example, if you want to enable the `SRCIPDESTIPHASH` based lb method, you must use the corresponding NITRO key and value format `lbmethod`, `SRCIPDESTIPHASH` respectively.

The following table details the smart annotations provided by the Citrix ingress controller:

| Citrix ADC Entity Name | Smart Annotation | Example |
| ----------------------- | ---------------- | ------- |
| lbvserver | ingress.citrix.com/lbvserver | `ingress.citrix.com/lbvserver: '{"citrix-svc":{"lbmethod":"SRCIPDESTIPHASH"}}'` |
| servicegroup | ingress.citrix.com/servicegroup | `ingress.citrix.com/servicegroup: '{"appname":{"cip": "Enabled","cipHeader":"X-Forwarded-For"}}'` |
| monitor | ingress.citrix.com/monitor | `ingress.citrix.com/monitor: '{"appname":{"type":"http"}}'` |
| csvserver| ingress.citrix.com/csvserver| `ingress.citrix.com/csvserver: '{"stateupdate": "ENABLED"}` |

### Sample ingress YAML with smart annotations

The following is a sample Ingress YAML. It includes smart annotations to enable Citrix ADC features using the entities such as, lbvserver, servicegroup, and monitor:

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: citrix
  annotations:
    ingress.citrix.com/insecure-port: '80'
    ingress.citrix.com/frontend-ip: '192.168.1.1'
    ingress.citrix.com/lbvserver: '{"citrix-svc":{"lbmethod":"LEASTCONNECTION", "persistenceType":"SOURCEIP"}}'
    ingress.citrix.com/servicegroup: '{"citrix-svc":{"usip":"yes"}}'
    ingress.citrix.com/monitor: '{"citrix-svc":{"type":"http"}}'
spec:
  rules:
  - host:  citrix.org
    http:
      paths:
      - path: /
        backend:
          serviceName: citrix-svc
          servicePort: 80
```

The sample Ingress YAML includes use cases related to the service, `citrix-svc`, and the following table explains the smart annotations used in the sample:

| Smart Annotation | Description |
| ---------------- | ----------- |
| `ingress.citrix.com/lbvserver: '{"citrix-svc":{"lbmethod":"LEASTCONNECTION", "persistenceType":"SOURCEIP"}}'` | Sets the load balancing method as [Least Connection](https://docs.citrix.com/en-us/citrix-adc/12-1/load-balancing/load-balancing-customizing-algorithms/leastconnection-method.html) and also configures [Source IP address persistence](https://docs.citrix.com/en-us/citrix-adc/12-1/load-balancing/load-balancing-persistence/source-ip-persistence.html). |
| `ingress.citrix.com/servicegroup: '{"citrix-svc":{"usip":"yes"}}'` | Enables [Use Source IP Mode (USIP)](https://docs.citrix.com/en-us/citrix-adc/12-1/networking/ip-addressing/enabling-use-source-ip-mode.html) on the Ingress Citrix ADC device. When you enable USIP on the Citrix ADC, it uses the client's IP address for communication with the back-end pods. |
| `ingress.citrix.com/monitor: '{"citrix-svc":{"type":"http"}}'` | Creates a [custom HTTP monitor](https://docs.citrix.com/en-us/citrix-adc/12-1/load-balancing/load-balancing-custom-monitors.html) for the servicegroup. |

**Note:** When multiple ingresses are sharing the same front-end IP address and port, you cannot have conflicting configurations provided through multiple ingress configurations.

By default, the content switching virtual server does not depend on the state of the target load balancing virtual servers bound to it. The annotation `ingress.citrix.com/csvserver: '{"stateupdate": "ENABLED"}` sets the content switching virtual server to consider its state based on the state of the load balancing virtual server bound to it via the content switching policies. 


## Smart annotations for routes

Similar to Ingress, you can also use smart annotations with OpenShift routes.
The Citrix ingress controller converts the routes in OpenShift to a set of Citrix ADC objects.

The following table details the smart annotations provided by the Citrix ingress controller:

| Citrix ADC entity name | Smart annotation | Example |
| ----------------------- | ---------------- | ------- |
| `lbvserver` | route.citrix.com/lbvserver | `route.citrix.com/lbvserver: '{"citrix-svc":{"lbmethod":"SRCIPDESTIPHASH"}}'` |
| `servicegroup` | route.citrix.com/servicegroup | `route.citrix.com/servicegroup: '{"appname":{"cip": "Enabled","cipHeader":"X-Forwarded-For"}}'` |
| `monitor` | route.citrix.com/monitor | `route.citrix.com/monitor: '{"appname":{"type":"http"}}'` |


### Sample route manifest with smart annotations

The following is a sample route YAML file.

```yml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: citrix
  annotations:
    route.citrix.com/lbvserver: '{"citrix-svc":{"lbmethod":"LEASTCONNECTION", "persistenceType":"SOURCEIP"}}'
    route.citrix.com/servicegroup: '{"citrix-svc":{"usip":"yes"}}'
    route.citrix.com/monitor: '{"citrix-svc":{"type":"http"}}'
spec:
  host:  citrix.org
  port:
    targetPort: 80
  to:
    kind: Service
    name: citrix-svc
    weight: 100
  wildcardPolicy: None
```

The sample route manifest includes use cases related to the service `citrix-svc` and the following table explains the smart annotations used in the sample Route:

| Smart annotation | Description |
| ---------------- | ----------- |
| `route.citrix.com/lbvserver: '{"citrix-svc":{"lbmethod":"LEASTCONNECTION", "persistenceType":"SOURCEIP"}}'` | Sets the load balancing method as [Least Connection](https://docs.citrix.com/en-us/citrix-adc/12-1/load-balancing/load-balancing-customizing-algorithms/leastconnection-method.html) and also configures [Source IP address persistence](https://docs.citrix.com/en-us/citrix-adc/12-1/load-balancing/load-balancing-persistence/source-ip-persistence.html). |
| `route.citrix.com/servicegroup: '{"citrix-svc":{"usip":"yes"}}'` | Enables [Use Source IP Mode (USIP)](https://docs.citrix.com/en-us/citrix-adc/12-1/networking/ip-addressing/enabling-use-source-ip-mode.html) on the Citrix ADC device. When you enable USIP on the Citrix ADC, it uses the IP address of the client for communication with the back-end pods. |
| `route.citrix.com/monitor: '{"citrix-svc":{"type":"http"}}'` | Creates a [custom HTTP monitor](https://docs.citrix.com/en-us/citrix-adc/12-1/load-balancing/load-balancing-custom-monitors.html) for the servicegroup. |


## Service annotations

The following are the service annotations supported by Citrix.

**Note:**
In service annotations, `index` is the ordered index of the ports in a service specification file. For example, if there are two ports in the service specification, then the index for the first port is zero and the second one is one.

|**Annotations**|**Description**|**Example**|
|---------------|---------------|-----------|
|`service.citrix.com/service-type-<index>`|Use this annotation to specify the service type for the Citrix ADC entities created. The acceptable values are `TCP`, `HTTP`, `SSL`,`UDP`,`ANY`, `SSL_TCP`, and `SIP_UDP`. | service.citrix.com/service-type-0: ‘SSL’|
|`service.citrix.com/lbmethod-<index>`| Use this annotation to specify the method for load balancing. The accepted values are: `ROUNDROBIN`, `LEASTCONNECTION`, and `LEASTRESPONSETIME`.| service.citrix.com/lbmethod-0: ‘LEASTCONNECTION’|
|`service.citrix.com/persistence-<index>`| Use this annotation to specify the persistence type. The accepted values are: `NONE`, `COOKIEINSERT`, `SOURCEIP`, `SRCIPDESTIP`, and `DESTIP`.| service.citrix.com/persistence-0: ‘SOURCEIP’|
|`service.citrix.com/ssl-certificate-data-<index>`| Use this annotation to specify the server certificate value in the PEM format.|  service.citrix.com/ssl-certificate-data-0: \| <`certificate`>|
| `service.citrix.com/ssl-key-data-<index>`| Use this annotation to specify the server key value in the PEM format.  | service.citrix.com/ssl-key-data-0: \| <`key`>|
| `service.citrix.com/ssl-ca-certificate-data-<index>` | Use this annotation to specify the server CA certificate value to verify the client certificate in PEM format.| service.citrix.com/ssl-ca-certificate-data-0: \| <`certificate`> |
|`service.citrix.com/ssl-backend-ca-certificate-data-<index>`| Use this annotation to specify the CA certificate value to verify the server certificate of the back-end in PEM format.| service.citrix.com/ssl-backend-ca-certificate-data-0: \| <`certificate`> |
| `service.citrix.com/ssl-termination-<index>` | Use this annotation to specify the SSL termination. The accepted values are `EDGE` and `REENCRYPT`.  | service.citrix.com/ssl-termination-0: 'EDGE' |
| `service.citrix.com/insecure-redirect` | Use this annotation to redirect insecure traffic to a secure port. You can either specify the secure port using {`secure-portname` : `port-number`} or {`secure-portnumber`- `secure-port-protocol` : `insecure-portnumber` } to redirect traffic from an insecure port.  | service.citrix.com/insecure-redirect: '{"port-443": 80 }'  <br> or <br> service.citrix.com/insecure-redirect: '{"443-tcp": 80 }' |
| `service.citrix.com/frontend-ip` | Use this annotation to pass the VIP for services of type `LoadBalancer`.|service.citrix.com/frontend-ip: '192.168.1.1' |
| `service.citrix.com/ipam-range` | Use this annotation to select a particular IP address range from a set of ranges specified to the Citrix IPAM controller. This annotation is used for services of type LoadBalancer.|service.citrix.com/ipam-range: 'Dev'|
| `service.citrix.com/secret` | Use this annotation to specify the name of the secret resource for the front-end server certificate. For more information and example, see [SSL certificate for services of type LoadBalancer](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/configure/service-type-lb-ssl-secret/).| service.citrix.com/secret: 'hotdrink-secret' |
| `service.citrix.com/ca-secret` |Use this annotation to provide a CA certificate for client certificate authentication. This certificate is bound to the front-end SSL virtual server in Citrix ADC. For more information and example, see [SSL certificate for services of type LoadBalancer](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/configure/service-type-lb-ssl-secret/).| service.citrix.com/ca-secret: 'hotdrink-ca-secret'|
| `service.citrix.com/backend-secret` | Use this annotation if the back-end communication between Citrix ADC and your workload is on an encrypted channel, and you need the client authentication in your workload. This certificate is sent to the server during the SSL handshake and it is bound to the back end SSL service group. For more information and example, see [SSL certificate for services of type LoadBalancer](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/configure/service-type-lb-ssl-secret/).| service.citrix.com/backend-secret: 'hotdrink-secret'|
| `service.citrix.com/backend-ca-secret` |Use this annotation to enable server authentication which authenticates the back-end server certificate. This configuration binds the CA certificate of the server to the SSL service on the Citrix ADC. For more information and example, see [SSL certificate for services of type LoadBalancer](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/configure/service-type-lb-ssl-secret/).|  service.citrix.com/backend-ca-secret: 'hotdrink-ca-secret'|
| `service.citrix.com/preconfigured-certkey` |Use this annotation to specify the name of the preconfigured certificate key in the Citrix ADC to be used as a front-end server certificate. |service.citrix.com/preconfigured-certkey: 'coffee-cert' |
| `service.citrix.com/preconfigured-ca-certkey`|Use this annotation to specify the name of the preconfigured certificate key in the Citrix ADC to be used as a CA certificate for client certificate authentication. This certificate is bound to the front-end SSL virtual server in Citrix ADC. | service.citrix.com/preconfigured-backend-certkey: 'coffee-cert'|
|`service.citrix.com/preconfigured-backend-certkey` |Use this annotation to specify the name of the preconfigured certificate key in the Citrix ADC to be bound to the back-end SSL service group. This certificate is sent to the server during the SSL handshake for server authentication. | service.citrix.com/preconfigured-ca-certkey: 'coffee-ca-cert'|
|`service.citrix.com/preconfigured-backend-ca-certkey` |Use this annotation to specify the name of the preconfigured CA certificate key in the Citrix ADC to bound to the back-end SSL service group for server authentication.|service.citrix.com/preconfigured-backend-ca-certkey: 'coffee-ca-cert' |

### Sample YAML with the service annotation to redirect insecure traffic

This example shows how to redirect traffic from clients making requests on an insecure port 80 to the secure port 443.

The following annotation is specified in the service YAML file to redirect traffic:

    service.citrix.com/insecure-redirect: '{"port-443": 80}'

Following is a sample service definition:

```yml

apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  annotations:
    service.citrix.com/service-type-0: SSL
    service.citrix.com/frontend-ip: '192.2.170.26'
    service.citrix.com/secret: '{"port-443": "web-ingress-secret"}'
    service.citrix.com/ssl-termination-0: 'EDGE'
    service.citrix.com/insecure-redirect: '{"port-443": 80}'
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 443
    targetPort: 80
    name: port-443

```

## Smart annotations for services

Smart annotations for services are used to configure the Citrix ADC with custom values for Citrix ADC configuration parameters. The annotations are used for services of type `LoadBalancer` and for the services in Citrix ADC CPX used for East-West traffic.

> **Note:** If you have configured a service with NodePort or ClusterIP for the North-South traffic, then the Citrix ADC is configured using the applicable ingress smart annotations rather than then service annotations.

Smart annotations for services take JSON format as input. The key and value that you pass in the JSON format must match the Citrix ADC NITRO format. For more information on the Citrix ADC NITRO API, see [Citrix ADC 12.1 REST APIs - NITRO Documentation](https://developer-docs.citrix.com/projects/netscaler-nitro-api/en/latest/).

The following is a sample smart annotation for services:

    service.citrix.com/lbvserver: '{"80-tcp":{"lbmethod":"SRCIPDESTIPHASH"}}'

This annotation sets the load balancing method as `SRCIPDESTIPHASH` in the load balancing virtual server for the `80-tcp` port of the given service.

The following table details the smart annotations for services:

| Citrix ADC Entity Name | Smart Annotation for Service | Example |
| ----------------------- | ---------------- | ------- |
| lbvserver | service.citrix.com/lbvserver | `service.citrix.com/lbvserver: '{"80-tcp":{"lbmethod":"SRCIPDESTIPHASH"}}'` |
| csvserver | service.citrix.com/csvserver | `service.citrix.com/csvserver: '{"l2conn":"on"}'` |
| servicegroup | service.citrix.com/servicegroup | `service.citrix.com/servicegroup: '{"80-tcp":{"usip":"yes"}}'` |
| monitor | service.citrix.com/monitor | `service.citrix.com/monitor: '{"80-tcp":{"type":"http"}}'` |
| analyticsprofile | service.citrix.com/analyticsprofile | `service.citrix.com/analyticsprofile: '{"80-tcp":{"webinsight": {"httpurl":"ENABLED", "httpuseragent":"ENABLED"}}}'` |

You can use the smart annotations for services as follows:

-  **By providing the `port-protocol` value in the annotation.** In the service definition, if you provide the `port-protocol` value in the annotation then the annotation is restricted to the particular port of that service.
-  **By not providing the `port-protocol` value in the annotation.** If you do not provide the `port-protocol` value in the annotation, then the annotation is applicable to all the ports used by the service.

### Sample ingress YAML with smart annotations for services

The following is a sample deployment and service definition for a basic apache web server based application. It includes smart annotations for services to enable Citrix ADC features using the entities such as, lbvserver, csvserver, servicegroup, monitor, and analyticsprofile:


```yml
# If using this on GKE, eusure sure you have cluster-admin role for your account
#The sample is a basic apache web server as application for illustration
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: apache
  labels:
      name: apache
spec:
  selector:
    matchLabels:
      app: apache
  replicas: 8
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache
        image: httpd:latest
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent

---
#Expose the apache web server as a service
apiVersion: v1
kind: Service
metadata:
  name: apache
  annotations:
    service.citrix.com/csvserver: '{"l2conn":"on"}'
    service.citrix.com/lbvserver: '{"80-tcp":{"lbmethod":"SRCIPDESTIPHASH"}}'
    service.citrix.com/servicegroup: '{"80-tcp":{"usip":"yes"}}'
    service.citrix.com/monitor: '{"80-tcp":{"type":"http"}}'
    service.citrix.com/frontend-ip: '10.217.212.16'
    service.citrix.com/analyticsprofile: '{"80-tcp":{"webinsight": {"httpurl":"ENABLED", "httpuseragent":"ENABLED"}}}'
    NETSCALER_VPORT: '80'
  labels:
    name: apache
spec:
  externalTrafficPolicy: Local
  type: LoadBalancer
  selector:
    name: apache
  ports:
  - name: http
    port: 80
    targetPort: http
  selector:
    app: apache
---
```

## Examples

### Sample Ingress YAML for SIP_UDP support in insecure service type annotation

The following is a sample Ingress YAML which includes the configuration for enabling SIP over UDP support using the `ingress.citrix.com/insecure-service-type` annotation. 

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: sip-ingress
  annotations:
    kubernetes.io/ingress.class: "cic-vpx"
    ingress.citrix.com/insecure-service-type: "sip_udp"
    ingress.citrix.com/frontend-ip: "1.1.1.1"
    ingress.citrix.com/insecure-port: "5060"
    ingress.citrix.com/lbvserver: '{"asterisk17":{"lbmethod":"CALLIDHASH","persistenceType":"CALLID"}}'
spec:
  backend:
    serviceName: asterisk17
    servicePort: 5060
```
