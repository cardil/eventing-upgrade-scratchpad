# Knative Eventing - upgrade testing scratchpad

Scratchpad for eventing upgrade tests

## Install

### OCP

1. Run OCP 4.2 and have oc logged in as admin
1. Run `oc apply -f install/eventing-openshift.yaml` 
1. Wait until `oc get clusterserviceversion -n openshift-operators` report succeeded for all operators
1. Run `oc apply -f install/eventing-cr.yaml`
1. Wait until `oc get knativeserving/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'` and `oc get knativeeventing/knative-eventing -n knative-eventing --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'` reports as Ready
1. Run `oc apply -f install/event-example.yaml`
1. Wait until `oc get broker default -n event-example` reports Ready

### Kubernetes

1. Run Kubernetes cluster 1.15+ and login as admin
1. Run `kubectl apply -f https://raw.githubusercontent.com/cardil/eventing/feature/upgrade-tests/third_party/istio-1.4.3/istio-crds.yaml`, 
   and wait until jobs in `istio-system` are completed
1. Run `kubectl apply -f https://raw.githubusercontent.com/cardil/eventing/feature/upgrade-tests/third_party/istio-1.4.3/istio-minimal.yaml`
   and wait until all pods in `istio-system` are running
1. Run `kubectl apply -f https://github.com/knative/serving/releases/download/v0.11.0/serving.yaml`
1. Run `kubectl apply -f https://github.com/knative/eventing/releases/download/v0.11.0/release.yaml`
1. Wait until all pods in `knative-serving` and `knative-eventing` are running

## Tests

1. Run `for y in tests/config.yaml tests/trigger.yaml tests/receiver.yaml tests/forwarder.yaml; do kubectl apply -f $y; done`
1. Wait until `kubectl get deployment $(kubectl get ksvc -n event-example -o jsonpath='{.items[0].status.latestCreatedRevisionName}')-deployment -n event-example` report that Knative Service scaled to zero `READY: 0/0`
1. Run `kubectl apply -f tests/sender.yaml` and wait until forwarder pods scale up `READY: 14/14`
1. Run `kubectl delete -f tests/sender.yaml`. Receiver pod should receive finished event and report state. You should observe missing
   events. Sometimes this pass, in that case delete receiver pod (deployment will recreate it) and reperform from step 2 of reproduction.
