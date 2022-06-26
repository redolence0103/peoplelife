## kafdrop-headless-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kafdrop
  namespace: istio-system
spec:
  type: ExternalName
  externalName: kafdrop.kafka.svc.cluster.local
```

## ingress.yaml in istio-system
```yaml
apiVersion: "extensions/v1beta1"
kind: "Ingress"
metadata: 
  name: "istio-ingress"
  namespace: "istio-system"
  annotations: 
    kubernetes.io/ingress.class: "nginx"
spec: 
  rules: 
    - host: "kafdrop.service.com"
      http: 
        paths: 
          - 
            path: /
            pathType: Prefix
            backend: 
              serviceName: kafdrop
              servicePort: 9000
```
