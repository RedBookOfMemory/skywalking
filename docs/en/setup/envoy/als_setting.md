# Observe service mesh through ALS
Envoy [ALS(access log service)](https://www.envoyproxy.io/docs/envoy/latest/api-v2/service/accesslog/v2/als.proto) provides
fully logs about RPC routed, including HTTP and TCP.

If solution initialized and first implemented by [Sheng Wu](https://github.com/wu-sheng), [Hongtao Gao](https://github.com/hanahmily), [Lizan Zhou](https://github.com/lizan), 
and [Dhi Aurrahman](https://github.com/dio) at 17 May. 2019, 
and presented on [KubeCon China 2019](https://kccncosschn19eng.sched.com/event/NroB/observability-in-service-mesh-powered-by-envoy-and-apache-skywalking-sheng-wu-lizan-zhou-tetrate).
Here is the recorded [Video](https://www.youtube.com/watch?v=tERm39ju9ew).

SkyWalking is the first open source project introducing this ALS based solution to the world. This provides a new way with very low payload to service mesh, but the same observability.

You need three steps to open ALS.
1. Open envoyAccessLogService in istio by [enabling **envoyAccessLogService** in ProxyConfig](https://istio.io/docs/reference/config/istio.mesh.v1alpha1/#ProxyConfig).

    Upper istio 1.6.0, if istio installed by demo profile, you can open ALS ues command:
    ```
    istioctl manifest apply --set profile=demo --set meshConfig.defaultConfig.envoyAccessLogService.address=skywalking-oap.skywalking.svc:11800 --set meshConfig.enableEnvoyAccessLogService=true
    ```
    Note:Skywalking oap service is at skywalking namespace, and the port of gRPC service is 11800
    
2. (Default is ACTIVATED) Activate SkyWalking [envoy receiver](../backend/backend-receivers.md). 
3. Active ALS k8s-mesh analysis, set system env variable `SW_ENVOY_METRIC_ALS_HTTP_ANALYSIS`=`k8s-mesh`
```yaml
envoy-metric:
  selector: ${SW_ENVOY_METRIC:default}
  default:
    acceptMetricsService: ${SW_ENVOY_METRIC_SERVICE:true}
    alsHTTPAnalysis: ${SW_ENVOY_METRIC_ALS_HTTP_ANALYSIS:""} # Setting the system env variable would override this. 
```
Note multiple value，please use `,` symbol split

Here's an example to deploy SkyWalking by Helm chart.

```
istioctl install --set profile=demo --set meshConfig.defaultConfig.envoyAccessLogService.address=skywalking-oap.istio-system:11800 --set meshConfig.enableEnvoyAccessLogService=true

git checkout https://github.com/apache/skywalking-kubernetes.git
cd skywalking-kubernetes/chart

helm repo add elastic https://helm.elastic.co

helm dep up skywalking

helm install 8.1.0 skywalking -n istio-system --set oap.env.SW_ENVOY_METRIC_ALS_HTTP_ANALYSIS=k8s-mesh --set fullnameOverride=skywalking --set oap.envoy.als.enabled=true
```

Notice, only use this when envoy under Istio controlled, also in k8s env. The OAP requires the read right to k8s API server for all pods IPs.

You can use `kubectl logs ${You-OAP-Pod} | grep "K8sALSServiceMeshHTTPAnalysis"` to ensure OAP ALS k8s-mesh analysis has been active.
