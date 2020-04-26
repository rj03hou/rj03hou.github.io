---
layout: single
title: 简述k8s service的可访问性
description: 简述k8s service的可访问性，包括集群外和集群内通过service访问内部资源，集群内通过service访问外部资源。
headline: 简述k8s service的可访问性，包括集群外和集群内通过service访问内部资源，集群内通过service访问外部资源。
categories: k8s
headline: 
tags: [k8s]
comments: true
published: true
---



根据内部/外部，分为内部访问内部，内部访问外部，外部访问内部三种。



# 集群外和集群内通过service访问内部资源

service type类型来区分了集群外和集群内如何访问；我们目前使用的版本通过iptables来实现的service访问的路由。

- `ClusterIP`：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 `ServiceType`。对应node的iptables规则抽象描述如果访问的目标是clusterip:port，则改写目标为endpointlist（直接到具体的node的具体端口上）。
- `NodePort`：通过每个 Node 上的 IP 和静态端口（`NodePort`）暴露服务。`NodePort` 服务会路由到 `ClusterIP` 服务，这个 `ClusterIP` 服务会自动创建。对应node的iptables规则抽象描述就是访问的目标是本机ip:nodeport，则改写目标为endpointlist（直接到具体的node的具体端口上）。当然type是nodeport也会有有一个clusterip，也会支持clusterip模式。
- `LoadBalancer`：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 `NodePort` 服务和 `ClusterIP` 服务。比如aws的lb路由到nodeport服务上。
- `ExternalName`：通过返回 `CNAME` 和它的值，可以将服务映射到 `externalName` 字段的内容（例如， `foo.bar.example.com`）。 没有任何类型代理被创建。

# 集群内通过service访问外部资源

## 一 仅DNS映射

比如说在default namespace下面创建testdb服务指向aws rds

```bash
kubectl create service -n default externalname testdb --external-name test.ap-northeast-1.rds.amazonaws.com
```

## 二 包括端口映射

### 1. 集群内访问外部资源并且进行端口映射

下面的例子背景就是我们需要将自己建的k8s迁移到eks，迁移的过程中新的集群还需要依赖于之前集群的服务；通过下面的方式进行；其中ip是原来集群的中的一个node节点。

在default namespace创建一个服务svc1，在集群内访问svc1.default:7064映射到192.168.1.2:31496。

多个端口的时候需要行业下面的例子一样指定port的name，否则会报下面的错误：

Error from server (Invalid): error when creating "test.yaml": Service "svc1" is invalid: [spec.ports[0].name: Required value, spec.ports[1].name: Required value]
Error from server (Invalid): error when creating "test.yaml": Endpoints "svc1" is invalid: [subsets[0].ports[0].name: Required value, subsets[0].ports[1].name: Required value]

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc1
  namespace: default
spec:
  ports:
  - name: port7064
    protocol: TCP
    port: 7064
    targetPort: 31496
  - name: port7164
    protocol: TCP
    port: 7164
    targetPort: 31337
---
apiVersion: v1
kind: Endpoints
metadata:
  name: svc1
  namespace: default
subsets:
  - addresses:
      - ip: 192.168.1.2
    ports:
    - name: port7064
      port: 31496
    - name: port7164
      port: 31337
```

### 2. 集群外部的机器使用node-exporter&prometheus进行监控

比如下面的场景我们有个rocketmq集群，部署在集群外部，现在需要使用node-exporter&prometheus进行监控

```
kind: Service
apiVersion: v1
metadata:
  name: rocketmq
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/port: "9100"
    prometheus.io/path: "/metrics"
  labels:
    app: rocketmq
    type: ec2
spec:
  ports:
  - protocol: TCP
    port: 9100
    targetPort: 9100
---
kind: Endpoints
apiVersion: v1
metadata:
  name: rocketmq
  annotations:
    prometheus.io/port: "9100"
    prometheus.io/scrape: 'true'
    prometheus.io/path: "/metrics"
  labels:
    app: rocketmq
    type: ec2
subsets:
  - addresses:
      - ip: 192.168.1.2
      - ip: 192.168.1.3
      - ip: 192.168.1.4
    ports:
      - port: 9100
```

