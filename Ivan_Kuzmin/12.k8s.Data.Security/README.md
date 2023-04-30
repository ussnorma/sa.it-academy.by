# Homework for 11.K8s.Basic objects


## pods screenshot
![pods](pods.png)

## pod1 screenshot
![pod1](pod1.png)

## pod2 screenshot
![pod2](pod2.png)

## pod3 screenshot
![pod3](pod3.png)

## pod ssh screenshot
![podssh](podssh.png)

## sealed secret screenshot
![sealed](sealed.png)

## sealedd secret screenshot
![sealedd](sealedd.png)

## YAML file: 
```bash
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: index
data:
  index.html: |
    <!DOCTYPE html><html><body>
    <h3>Hello</h3>
    <h1>HOSTNAME</h1>
    </body></html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-webserver
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: "33%"
      maxUnavailable: "0%"
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 50m
            memory: 50Mi
          limits:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
          - mountPath: /usr/share/nginx/html/
            name: index1
          - name: secret
            mountPath: /root/.ssh/
      initContainers:
      - name: test
        image: nginx:latest
        command: ["bash", "-c", 'cd /tmp/; sed -e "s/HOSTNAME/$HOSTNAME/" /tmp/index.html > /usr/share/nginx/html/index.html']

        volumeMounts:
        - mountPath: /tmp/index.html
          name: test-index
          subPath: index.html
          readOnly: false
        - mountPath: /usr/share/nginx/html
          name: index1
        - name: ssh
          mountPath: /root/.ssh/
      volumes:
      - name: test-index
        configMap:
          name: index
      - name: index1
        emptyDir: {}
      - name: ssh
        emptyDir: {}
      - name: secret
        secret:
          secretName: secret-from-manifest
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    run: nginx-service
spec:
        #type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-sa
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/server-alias: "app.k8s-10.sa"
spec:
  rules:
    - host: app.k8s-10.sa
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-web-service
                port:
                  number: 80

```

## Unsealed file:
```bash
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-from-manifest
type: Opaque
data:
  id_rsa: !@#$%^&*()_+
  id_rsa.pub: !@#$%^&*()_+
```

## Sealed file:
```bash
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: secret-from-manifest
  namespace: default
spec:
  encryptedData:
    id_rsa: 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
    id_rsa.pub: AgBNKGXoCSCdubIi343Tuzzhd9bseaTKSTUPyzVC1oJD/ysQrDu1nFd5ROBKOVKKbyW4s/CSOJQ4rTi88P0Cg2OO8AHQxY+jmkkYSeJRbdo7C9naSfg4HptYoaVcZhkXfdn06yCuDQO5M4GFgF+l8Sh+j1rTqdiq1DKGxUQxdgzPwhkZRRq1Gd5gGu9GqSft0fxFvzNCaiOLJ//hKN5MPftvkLkXR5vbXoyBOdQyoqTGXFNV7DJ4Yc7fKK/jR0hAacR+t7c/O1Xl1xO5jSlt/uaNnkiMtIKAs/o8Th72j0t5ozY6l8eRkw5ad8n1g0XeR4YdCphYkmy077P/u6I9knoK1Ohh3Q1nYEO+xph15XBDCg3fg+J1RGuJuQohCpw0qXneNNGxdbCFOMAurjHjc8AzBSPvA/1qtckmMkk8f7HCWIRA27TQpL4WuBMQEHIdxdiMw0q7JrdRfSJPobcsq4D+THLMUBNts5ea92If8zklgsIpI/5TzXynboDrHIuKGtKVWZJnEQh3c9KQ0dvTJzJg0A78545GuD5ODK1iKPL+nNTz0fNkvT4yokcygPi+7LR7Kf0UlG7l2Qw59ZLRec80EaZ48E4GjBf7B5SGh5W6JYi0uKwbYe3uAxEIDn8PHwkkxpbJDH/QbmvGOeMHFLiy/LA/5gING6tZ33c9T91+F/H8KX1wBwl1XsSgCxZepA0a1uGQYSltGz74aKf2PamPEFzx7l0WDvbp2LtAqY1pGOFMzaxkoDY/jw6gIYTgEeJBCFzl4sM19KSw63Ehgfp6A7pIi98aFqLfxxX9lcKsAuX97xR+SLyUNaIiLte/D+9/hbHOdmcjUsrWA7HxgpvLN7UJJnY0g30IBtdtfmFxvaureUPSlxjAaoklb9L/t+jm/n1HFxmNGd4A7MeL3rDmEmzx/2AAppHdKD4h9MlXG47ISoRiaKESOXwyY5yb5dYRiVBQZaz+Izc/FL263vrsLOh9VcBY39f7rGoPaN5CltgyuqdhD7RaL0FqnrLM77kRBXoQJgVzwWM0ZAdblP+Vr0F3iMaAK0bKaOQazXS7QwEqdxqE0LHB8/EoQUZBX8Z/W7i8J3fLDmcEMPETi/YQ4fYpxODMzy/d/IJT1sb+1/X8BUr8L0GNNAxEdErcedSUHevetp7Rvh16E/5nWHC6lMk+PJQpVbrN9pdqcjB3OyjpAgqniwIboRsDiAlUq5k8ZEw0upramyV7G1mTwevTWQg0qt+NS5D5Tv8vhmjR+Ub69WAF33Kj+br1QryGm3GzXITsJXKYfFX/CEh9YK8vdtwJZkLYNjH/6Gj2hxxVdGUlpJyFjGk2Cy3ee8HI2WkmjK3eXdLHQyZjoSoQQr8axzmOu0F/movI3AxC1Gu8KiEAFEaE3eivE1ZDZAYKKXO78l4tVNByC06QrEj75NkbwADcXWk3yVhnNfQ11Tqhl5RMSwA=
  template:
    metadata:
      creationTimestamp: null
      name: secret-from-manifest
      namespace: default
    type: Opaque
```