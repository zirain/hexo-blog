---
title: 通过Telemetry API自定义Istio访问日志
date: 2022-07-27 22:48:14
tags:
    - istio
    - telemetry
    - accesslog
---

Istio 为网格内的所有服务通信生成详细的遥测数据，主要包含访问日志、监控指标和调用链。在 v1.11 版本之前我们可以通过`MeshConfig`中 `AccessLogFile`、`AccessLogFormat`以及`EnableEnvoyAccessLogService`等对访问日志进行自定义。引入 Telemetry API，提供标准化的 API 来配置访问日志和监控指标。

本文将介绍如何通过Telemetry API自定义Istio访问日志， 验证环境如下：
- istio 1.14.1
- kind 1.23.4

## 开始之前

- 下载istioctl：

```bash
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.14.1 TARGET_ARCH=x86_64 sh -
```

- 在集群中安装 istio:

```bash
istioctl install -y
```

- 开启 istio 自动注入：

```bash
kubectl label namespace default istio-injection=enabled --overwrite
```

- 安装 `sleep` 和 `httpbin`：

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.14/samples/sleep/sleep.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.14/samples/httpbin/httpbin.yaml
```

- 检查 istio 安装状态：

```bash
kubectl get po -nistio-system
```

```console
NAME                                    READY   STATUS    RESTARTS   AGE
istio-ingressgateway-57598d7c8b-8hg7m   1/1     Running   0          84s
istiod-859b487f84-9gq5x                 1/1     Running   0          88s
```

- 检查应用注入状态：

```bash
kubectl get po
```

```console
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-85d76b4bb6-wk4qw   2/2     Running   0          2m16s
sleep-698cfc4445-bkrrl     2/2     Running   0          2m18s
```

- 验证日志处于关闭状态：

```bash
kubectl exec -it `kubectl get po | grep sleep | awk '{print $1}'` -- curl httpbin:8000/get

kubectl logs `kubectl get po | grep httpbin | awk '{print $1}'` -cistio-proxy
```


## 配置访问日志

通过 Telemetry 配置网格级别的访问日志：

```bash
cat <<EOF | kubectl apply -n istio-system -f -
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-default
spec:
  accessLogging:
    - providers:
      - name: envoy
EOF
```

验证访问日志：

```bash
kubectl exec -it `kubectl get po | grep sleep | awk '{print $1}'` -- curl httpbin:8000/get

kubectl logs `kubectl get po | grep httpbin | awk '{print $1}'` -cistio-proxy
kubectl logs `kubectl get po | grep sleep | awk '{print $1}'` -cistio-proxy
```

```console
# httpbin 访问日志
[2022-07-27T15:39:08.021Z] "GET /get HTTP/1.1" 200 - via_upstream - "-" 0 605 1 1 "-" "curl/7.84.0-DEV" "2f4e1d78-33d4-484e-874c-712cb69c9c50" "httpbin:8000" "10.244.0.27:80" inbound|80|| 127.0.0.6:57695 10.244.0.27:80 10.244.0.26:39476 outbound_.8000_._.httpbin.default.svc.cluster.local default

# sleep 访问日志
[2022-07-27T15:39:08.018Z] "GET /get HTTP/1.1" 200 - via_upstream - "-" 0 605 5 4 "-" "curl/7.84.0-DEV" "2f4e1d78-33d4-484e-874c-712cb69c9c50" "httpbin:8000" "10.244.0.27:80" outbound|8000||httpbin.default.svc.cluster.local 10.244.0.26:39476 10.96.184.88:8000 10.244.0.26:41784 - default
```

## 禁用负载访问日志

通过如下的配置禁用 `httpbin` 访问日志：

```bash
cat <<EOF | kubectl apply -n default -f -
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: httpbin-als
spec:
  selector:
    matchLabels:
      app: httpbin
  accessLogging:
    - providers:
      - name: envoy
      disabled: true
EOF
```

验证访问日志:

```bash
kubectl exec -it `kubectl get po | grep sleep | awk '{print $1}'` -- curl httpbin:8000/get

kubectl logs `kubectl get po | grep httpbin | awk '{print $1}'` -cistio-proxy
kubectl logs `kubectl get po | grep sleep | awk '{print $1}'` -cistio-proxy
```

`sleep` 访问日志正常， `httpbin`访问日志被关闭：

```console
# httpbin 访问日志
[2022-07-27T15:39:08.021Z] "GET /get HTTP/1.1" 200 - via_upstream - "-" 0 605 1 1 "-" "curl/7.84.0-DEV" "2f4e1d78-33d4-484e-874c-712cb69c9c50" "httpbin:8000" "10.244.0.27:80" inbound|80|| 127.0.0.6:57695 10.244.0.27:80 10.244.0.26:39476 outbound_.8000_._.httpbin.default.svc.cluster.local default

# sleep 访问日志
[2022-07-27T15:39:08.018Z] "GET /get HTTP/1.1" 200 - via_upstream - "-" 0 605 5 4 "-" "curl/7.84.0-DEV" "2f4e1d78-33d4-484e-874c-712cb69c9c50" "httpbin:8000" "10.244.0.27:80" outbound|8000||httpbin.default.svc.cluster.local 10.244.0.26:39476 10.96.184.88:8000 10.244.0.26:41784 - default
[2022-07-27T15:44:24.370Z] "GET /get HTTP/1.1" 200 - via_upstream - "-" 0 605 11 11 "-" "curl/7.84.0-DEV" "17c9cf5f-28e9-418a-ac87-41a255c1bd36" "httpbin:8000" "10.244.0.27:80" outbound|8000||httpbin.default.svc.cluster.local 10.244.0.26:55224 10.96.184.88:8000 10.244.0.26:57532 - default
```

## 筛选访问日志

通过如下的配置 `httpbin` 负载只记录状态码为 `500` 的访问日志：

```bash
cat <<EOF | kubectl apply -n default -f -
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: httpbin-als
spec:
  selector:
    matchLabels:
      app: httpbin
  accessLogging:
    - providers:
      - name: envoy
      filter:
        expression: response.code == 500
EOF
```

执行以下命令验证:

```bash
kubectl exec -it `kubectl get po | grep sleep | awk '{print $1}'` -- curl httpbin:8000/get
kubectl exec -it `kubectl get po | grep sleep | awk '{print $1}'` -- curl httpbin:8000/status/500
```

查看 `httpbin` 访问日志：

```bash
kubectl logs `kubectl get po | grep httpbin | awk '{print $1}'` -cistio-proxy
```

`httpbin`只记录响应码为500的请求：
```console
[2022-07-27T15:55:32.694Z] "GET /status/500 HTTP/1.1" 500 - via_upstream - "-" 0 0 1 0 "-" "curl/7.84.0-DEV" "4c6207ba-a9c6-4544-9282-8382c2f8e5a2" "httpbin:8000" "10.244.0.27:80" inbound|80|| 127.0.0.6:44495 10.244.0.27:80 10.244.0.26:59444 outbound_.8000_._.httpbin.default.svc.cluster.local default
```

## 配置输出 OpenTelemtry 格式日志

接下来我们通过 Telemetry 将访问日志以 [OpenTelemetry](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/access_loggers/open_telemetry/v3/logs_service.proto) 格式输出至 [OTel-collector](https://github.com/open-telemetry/opentelemetry-collector) 中。

- 安装 `OTel-collector` 负载，配置从 `4317` 端口接受日志，并输出至标准输入输出：

```console
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.14/samples/open-telemetry/otel.yaml
```

- 升级 istio 配置，增加 `ExtensionProvider`:

```console
istioctl install --set profile=demo -y
```

- 配置 `httpbin` 日志输出至 `OTel-collector`:

```bash
cat <<EOF | kubectl apply -n default -f -
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: httpbin-als
spec:
  selector:
    matchLabels:
      app: httpbin
  accessLogging:
    - providers:
      - name: otel
EOF
```

- 验证配置结果：

```bash
kubectl exec -it `kubectl get po | grep sleep | awk '{print $1}'` -- curl httpbin:8000/get

kubectl logs -l app=otel-collector -nistio-system --tail=-1
```

可看到如下输出：

```console
2022-07-27T16:15:54.163Z        DEBUG   loggingexporter/logging_exporter.go:81  ResourceLog #0
Resource labels:
     -> log_name: STRING(otel_envoy_accesslog)
     -> zone_name: STRING()
     -> cluster_name: STRING(httpbin.default)
     -> node_name: STRING(sidecar~10.244.0.27~httpbin-85d76b4bb6-wk4qw.default~default.svc.cluster.local)
InstrumentationLibraryLogs #0
InstrumentationLibrary
LogRecord #0
Timestamp: 2022-07-27 16:15:53.89784 +0000 UTC
Severity:
ShortName:
Body: [2022-07-27T16:15:53.897Z] "GET /get HTTP/1.1" 200 - via_upstream - "-" 0 605 1 1 "-" "curl/7.84.0-DEV" "2db7d544-4a80-91ae-b612-362047bbe11b" "httpbin:8000" "10.244.0.27:80" inbound|80|| 127.0.0.6:41313 10.244.0.27:80 10.244.0.26:36580 outbound_.8000_._.httpbin.default.svc.cluster.local default

Trace ID:
Span ID:
Flags: 0
```

## 参考资料

- [Telemetry API](https://istio.io/latest/docs/tasks/observability/telemetry/)
- [Telemetry AccessLogging](https://istio.io/latest/docs/reference/config/telemetry/#AccessLogging)