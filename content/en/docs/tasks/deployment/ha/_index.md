---
title: "Deploying IAM in HA"
weight: 120
description: >
  Instructions on how to deploy the IAM service in High Availability on k8s
---

Starting from version 1.8.0, the IAM service can be deployed in High Availability mode.  
It relies on [redis] as external component used for storing the
log-in sessions and the default in-memory IAM cache, which has to be deployed as well.

Here is an example on how to deploy three IAM replicas on a [kubernetes] cluster.

### IAM Deployment in HA

An example of three replicas of the IAM service deployment on k8s is the following:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iam-login-service
spec:
  selector:
    matchLabels:
      app: iam
      tier: login-service
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: iam
        tier: login-service
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions: 
                - key: tier
                  operator: In
                  values:
                    - login-service
              topologyKey: kubernetes.io/hostname
      volumes:
      - name: json-keystore-secret-vol
        secret:
          secretName: json-keystore-secret
      containers:
        - name: iam-login-service
          image: indigoiam/iam-login-service:v1.8.0
          imagePullPolicy: Always
          ports:
          - containerPort: 8080
            name: iam
          - containerPort: 1044
            name: iam-debug
          volumeMounts:
            - name: json-keystore-secret-vol
              mountPath: /secrets/json-keystore
              readOnly: true
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 90
            timeoutSeconds: 10
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 1
              memory: 2Gi
          env:
            - name: IAM_BASE_URL
              value: https://iam.example
            - name: IAM_ISSUER
              value: https://iam.example/
          envFrom:
            - configMapRef:
                name: login-service-env
            - secretRef:
                name: db-secret-env
```

Here a [configMap][config-map] for the `login-service-env` file is used.  
You can set there the [Configuration properties](../../../reference/configuration/#redis-configuration)
used to enable Redis. The minimal ones required, for a password less Redis service are

```bash
IAM_SPRING_REDIS_HOST=redis.example
IAM_SPRING_SESSION_STORE_TYPE=redis
IAM_CACHE_REDIS_ENABLED=true
```

### Redis service Deployment

A basic example of the Redis deployment on k8s (without replicas)
based on the `redis` [docker image](https://hub.docker.com/_/redis) is the following:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6
        resources:
          requests:
            memory: "512Mi"
            cpu: "2"
          limits:
            memory: "512Mi"
            cpu: "2"
        ports:
        - containerPort: 6379
```

[redis]: https://redis.io/
[kubernetes]: https://kubernetes.io/
[config-map]: https://kubernetes.io/docs/concepts/configuration/configmap/