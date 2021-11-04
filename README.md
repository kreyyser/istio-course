
# istio-course

## Setup k8s

1. Enable docker kubernetes single node cluster by clicking on the `Enable kubernetes` button under the `Kubernetes` section of the docker desktop application preferences and `Apply & Restart` after.
![Docker-settings](/hw1/docker-settings.png)

2. After k8s cluster is up - verify it by checking the k9s dashboard (don't forget to hit `0` to show the pods from all namespaces)
![k9s-k8s-setup](/hw1/k9s-k8s-setup.png)

3. Install k8s dashboard by running following command and verify following output
```shell script
> kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

4. Enable k8s proxy to get access to the Dashboard UI
```shell script
> kubectl proxy
Starting to serve on 127.0.0.1:8001
```

5. Access the dashboard page to verify it is up (it will prompt you to pass the authentication details - it is okay, we will pass this gate at the next steps)
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

6. Create dashboard service account to be able to access Dashboard UI.
```shell script
> kubectl apply -f hw1/dashboard-serviceaccount.yml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

7. Retrieve the service account secret token.
```shell script
> kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
<REDACTED_JWT_VALUE>
```

8. Pass the token to the Dashboard UI and login. Verify you see the similar picture
![k8s-dashboard](/hw1/k8s-dashboard.png)

## Install Istio

1. Follow the [`istioctl` tool installation instructions](https://istio.io/latest/docs/setup/getting-started/#download) to have it enabled in the PATH

2. Install istio demo profile
```shell script
> istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Egress gateways installed
✔ Ingress gateways installed
✔ Installation complete
Thank you for installing Istio 1.10.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/KjkrDnMPByq7akrYA
```

3. Label default namespace to enable envoy sidecars injection
```shell script
> kubectl label namespace default istio-injection=enabled
namespace/default labeled
```

4. Verify in k9s that all the istio related pods are up and running (you won't have the infrastructe pods available - we will install those later)
![istio-pods](/hw1/istio-pods.png)

## Install Kiali

1. Enable kiali
```shell script
> kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/addons/kiali.yaml
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created

> kubectl -n istio-system get svc kiali
NAME    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
kiali   ClusterIP   10.107.192.254   <none>        20001/TCP,9090/TCP   14s
```

2. Install addons to be able to consume metrics by applying addons provided on the 1st step of the istioctl installation.
```shell script
> kubectl apply -f istio-1.10.5/samples/addons
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/monitoringdashboards.monitoring.kiali.io created
serviceaccount/kiali configured
configmap/kiali configured
clusterrole.rbac.authorization.k8s.io/kiali-viewer configured
clusterrole.rbac.authorization.k8s.io/kiali configured
clusterrolebinding.rbac.authorization.k8s.io/kiali configured
role.rbac.authorization.k8s.io/kiali-controlplane configured
rolebinding.rbac.authorization.k8s.io/kiali-controlplane configured
service/kiali configured
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

2. Serve the kiali dashboard
```shell script
> istioctl dashboard kiali
http://localhost:20001/kiali
```

## Configure Jaeger

1. Install the Jaeger into the cluster
```shell script
> kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/addons/jaeger.yaml
deployment.apps/jaeger configured
service/tracing configured
service/zipkin unchanged
service/jaeger-collector configured
```

2. Serve the jaeger dashboard
```shell script
> istioctl dashboard jaeger
http://localhost:16686
```

## Install example app

1. Install demo app configuration and verify everything is up and running.
```shell script
> kubectl apply -f istio-1.10.5/samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created

> kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.98.118.36     <none>        9080/TCP   85s
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    55m
productpage   ClusterIP   10.106.228.242   <none>        9080/TCP   84s
ratings       ClusterIP   10.101.232.213   <none>        9080/TCP   85s
reviews       ClusterIP   10.98.221.134    <none>        9080/TCP   85s

> kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79f774bdb9-vsqtf       2/2     Running   0          2m42s
productpage-v1-6b746f74dc-pmq74   2/2     Running   0          2m42s
ratings-v1-b6994bb9-gdgtj         2/2     Running   0          2m42s
reviews-v1-545db77b95-hqljx       2/2     Running   0          2m42s
reviews-v2-7bf8c9648f-4f2r5       2/2     Running   0          2m42s
reviews-v3-84779c7bbc-sj6qh       2/2     Running   0          2m42s
```

2. Verify the app is working by calling it from within the cluster
```shell script
> kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

3. Deploy a gateway sample to expose the app outside of the cluster
```shell script
> kubectl apply -f istio-1.10.5/samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

4. Get the ingress-gateway external ip
```shell script
> kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.106.199.140   localhost     15021:31692/TCP,80:31111/TCP,443:31051/TCP,31400:31203/TCP,15443:31940/TCP   14m
```

5. Export env vars to be able to call the app outside of the cluster
```shell script
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

6. Verify app is available outside of the cluster
```shell script
> curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

## Verify Kiali and Jaeger

1. Generate random traffic to the app
```shell script
> watch -n 1 curl -o /dev/null -s -w %{http_code} $GATEWAY_URL/productpage
```

2. Verify traces are coming by accessing [Jaeger UI page](http://localhost:16686) and selecting any of apps

![jaeger-ui](/hw1/jaeger.png)

3. Verify kiali monitores the traffic by selecting the default namespace (where demo app is deployed) and any display values
![kiali-ui](/hw1/kiali.png)