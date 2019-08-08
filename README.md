# Deployment observer for SAP Data Hub

**NOTE**: obsoleted by [miminar/sdh-helpers](https://github.com/miminar/sdh-helpers)

On Red Hat Enterprise Linux CoreOS, container needs to be run as *privileged*
in order to manage iptables on the host system. SAP Data Hub containers named
`vsystem-iptables` deployed as part of every `vsystem-app` deployment attempt
to modify iptables rules without having the necessary permissions. This
template fixes the permissions on-the-fly as the deployments are created.

The template spawns a pod that observes the particular namespace where
SAP Data Hub runs and marks each `"vsystem-iptables"` container in all
`"vsystem-app"` deployments as *privileged*.

Additionally `vsystem-vrep` `statefulset` will be patched to mount `emptyDir`
volume at `/exports` directory in order to enable NFS exports in the container
running on top overlayfs which is the default filesystem in RHCOS.

The template must be instantiated before the SAP Data Hub's installation.
Both the target namespace and the namespace where the SAP Data Hub will be
instaled must exist before the instantiation.

Targeted at:

- OpenShift 4.1 or higher
- SAP Data Hub is 2.6 or higher

**NOTE**: you will likely see the very first deployment of license-management
during the SAP Data Hub's installation to fail with a message like the following:

```
2019-07-24T07:28:54+0000 [INFO] Solutions were successfully imported!
2019-07-24T07:28:54+0000 [INFO] Initializing system tenant...
2019-07-24T07:28:54+0000 [INFO] Initializing License Manager in system tenant...2019-07-24T07:29:35+0000 [ERROR] Couldn't start License Manager!
The response: Error: http status code 502 Bad Gateway (502)
2019-07-24T07:29:35+0000 [ERROR] Failed to initialize vSystem, will retry in 30 sec...
2019-07-24T07:30:05+0000 [INFO] Wait until vora cluster is ready...
```

The error is harmless and can be ignored. The next deployment will succeed.

## Usage

The `vsystem-observer.yaml` template shall be deployed
before SAP Data Hub's installation either in the same namespace/project
or in a different one.

Required arguments:

- `NAMESPACE` - the project name, where the template shall be instantiated

### Deploying in the Data Hub project

If running the observer in the same namespace/project as Data Hub, instantiate the
template as is in the desired namespace:

    oc project $SDH_NAMESPACE
    oc process -f https://raw.githubusercontent.com/miminar/sdh-vsystem-observer/master/vsystem-observer.yaml \
        NAMESPACE=$SDH_NAMESPACE | oc create -f -

### Deploying in a different project

If running in a different/new namespace/project, instantiate the
template with parameters `SDH_NAMESPACE` and `NAMESPACE`, e.g.:

    SDH_NAMESPACE=sdh26                 # where SDH will be installed
    NAMESPACE=sapdatahub-admin          # where the contents of this template will be instantiated
    oc new-project $SDH_NAMESPACE
    oc new-project $NAMESPACE
    oc process -f https://raw.githubusercontent.com/miminar/sdh-vsystem-observer/master/vsystem-observer.yaml \
        SDH_NAMESPACE=$SDH_NAMESPACE NAMESPACE=$NAMESPACE | oc create -f -

