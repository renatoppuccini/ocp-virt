# ocp-virt
Repo that has steps to create bookinfo application with containers and VM and use ServiceMEsh to see traffic between different versions of the application.

Install Operators



Create Mesh namepsace
```
oc new-project istio-system
```
Create Mesh instance
```
oc new-project bookinfo

oc label namespace bookinfo istio-injection=enabled
```
Create ServiceMemberRoll with bookinfo namespace:
```
apiVersion: maistra.io/v1
kind: ServiceMeshMemberRoll
metadata:
  name: default
  namespace: istio-system
spec:
  members:
  - bookinfo
```
Deploy App
```
oc -n bookinfo apply -f \
  https://raw.githubusercontent.com/rh-mobb/rosa-workshop-content/main/rosa-content/assets/scripts/bookinfo.yaml
```
Deploy Ingress Gateway
```
oc -n bookinfo apply -f \
  https://raw.githubusercontent.com/rh-mobb/rosa-workshop-content/main/rosa-content/assets/scripts/bookinfo-gateway.yaml
```
Deploy Destination Rules
```
oc -n bookinfo apply -f \
  https://raw.githubusercontent.com/rh-mobb/rosa-workshop-content/main/rosa-content/assets/scripts/destination-rule-all.yaml
```

Get Gateway address
```
echo "http://$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')/productpage"
```
create virtual service
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 80
    - destination:
        host: reviews
        subset: v2
      weight: 20
```
```
echo "http://$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')/productpage"
```
Generate traffic
```
while true; do curl -sSL "http://$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')/productpage" | head -n 5; sleep 1; done
```

e.g.
http://istio-ingressgateway-istio-system.apps.cluster-zxhg5.dynamic.redhatworkshops.io/productpage

while true; do curl -sSL "http://istio-ingressgateway-istio-system.apps.cluster-zxhg5.dynamic.redhatworkshops.io/productpage" | head -n 5; sleep 1; done
