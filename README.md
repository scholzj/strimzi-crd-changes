# Strimzi CRD Changes

## Install cluster operator

* Create namespace/project `myproject` and set it as default.
* On OpenShift 
  * `oc new-project myproject`
  * `oc project myproject`
* On Kubernetes
  * `kubectl create ns myproject`
  * `kubectl config set-context --current --namespace=myproject`

## CRD Changes

* Install Strimzi 0.22.1
  * `kubectl create -f strimzi-0.22.1/`
* Deploy the Kafka cluster
  * `kubectl apply -f kafka.yaml`
* Check the CRDs installed with Strimzi 0.22 / AMQ Streams 1.7
  * Notice in the `.spec` section that they have now the `v1beta2` version. Version `v1beta1` or `v1alpha1` is marked as stored.
  * Notice that in the `.status` section they have only the `v1beta1` (or `v1alpha1`) as stored versions
  * You can use this command to check the versions from spec: `kubectl get crd kafkas.kafka.strimzi.io -o jsonpath="{.spec.versions[*].name}"`
* Try to edit the Kafka CR we have:
  * Do `kubectl edit kafka my-cluster`
    * Notice that `kubectl` uses `v1beta2` by default
    * Try to change something in the CR - for example the timeout of Kafka liveness probe - and save the changes
    * You should get an error because the resource is not valid `v1beta2` resource - it is using many deprecated fields
    * Change the `apiVersion` on the top of the file from `kafka.strimzi.io/v1beta2` to `kafka.strimzi.io/v1beta1` and save the changes again
    * This time the should pass
  * Try to also use `kubectl edit kafka.v1beta1.kafka.strimzi.io my-cluster` to use the `v1beta1` API version directly
  * Check the status section and notice the large amount of warnings about deprecated fields
* Have a look at the API Conversion tool located in [`api-conversion`](./api-conversion)
* Use the API Conversion tool to convert the [`kafka.yaml`](./kafka.yaml) file
  * Run `api-conversion/bin/api-conversion.sh convert-file -f kafka.yaml`
  * The converted resource will be printed to the standard output
  * Notice how it converted all the deprecated / removed fields
  * Notice the new Config Maps with the metrics configuration
* Use the API Conversion tool to convert the resource directly inside Kubernetes
  * Run `api-conversion/bin/api-conversion.sh convert-resources`
  * Check that no rolling update is happening (we converted the resource, but nothing really changed in the deployment itself)
  * Edit the resource with `kubectl edit kafka my-cluster` and check that:
    * The resource is now converted
    * The warnings about deprecations are gone
    * Since we do not use any old deprecated fields, we can now use the `v1beta2` API to edit it
  * Check the automatically created Config Maps with metrics configuration
* After you convert all resources, change the CRDs to use the `vbeta2` API as the stored version
  * Run `api-conversion/bin/api-conversion.sh crd-upgrade`
  * Check the `.status` section of the CRDs to see that only `v1beta2` is now used as stored version
  * You can use the command `kubectl get crd kafkas.kafka.strimzi.io -o jsonpath="{.status.storedVersions}"`
* You can now proceed with installing newer Strimzi version with `apiextensions/v1` CRDs
  * `kubectl replace -f strimzi-crd-v1/` (this right now uses the master from 15th April 🤞)
* Check the CRDs installed now
  * Notice in the `.spec` section that they have now only the `v1beta2` version. Version `v1beta1` or `v1alpha1` are gone.
  * Notice that in the `.status` section they have only the `v1beta2` as stored versions
  * You can use this command to check the versions from spec: `kubectl get crd kafkas.kafka.strimzi.io -o jsonpath="{.spec.versions[*].name}"`
