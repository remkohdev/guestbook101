```
# Source: web-terminal/templates/web-terminal-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: terminal-ingress
  annotations:
    ingress.bluemix.net/rewrite-path: "serviceName=terminal-service-1 rewrite=/;serviceName=terminal-service-2 rewrite=/;serviceName=terminal-service-3 rewrite=/;"
    ingress.bluemix.net/proxy-connect-timeout: "serviceName=terminal-service-1 timeout=75s;serviceName=terminal-service-2 timeout=75s;serviceName=terminal-service-3 timeout=75s;"
    ingress.bluemix.net/proxy-read-timeout: "serviceName=terminal-service-1 timeout=3600s;serviceName=terminal-service-2 timeout=3600s;serviceName=terminal-service-3 timeout=3600s;"
    kubernetes.io/ingress.class: nginx
spec:
  tls:
  - hosts:
    - testing-cluster.us-east.containers.appdomain.cloud
    secretName: testing-cluster
  rules:
  - host: testing-cluster.us-east.containers.appdomain.cloud
    http:
      paths:
      - path: "/term1"
        backend:
          serviceName: "terminal-service-1"
          servicePort: 80
      - path: "/term2"
        backend:
          serviceName: "terminal-service-2"
          servicePort: 80
      - path: "/term3"
        backend:
          serviceName: "terminal-service-3"
          servicePort: 80
```