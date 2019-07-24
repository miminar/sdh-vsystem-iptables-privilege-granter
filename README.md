# Insecure registry setter for SAP Data Hub Pipeline Modeler

On Red Hat Enterprise Linux CoreOS, container needs to be run as *privileged*
in order to manage iptables on the host system. SAP Data Hub containers named
`vsystem-iptables` deployed as part of every `vsystem-app` deployment attempt
to modify iptables rules without having the necessary permissions. This
template fixes the permissions on-the-fly as the deployments are created.

The template spawns a pod that observes the particular namespace where
SAP Data Hub runs and marks each `"vsystem-iptables"` container in all
`"vsystem-app"` deployments as *privileged*.

The template shall be instantiated before the SAP Data Hub's installation.

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

The `ivsystem-iptables-privilege-granter-template.yaml` template shall be deployed
before SAP Data Hub's installation either in the same namespace/project
or in a different one.

### Deploying in the Data Hub project

If running the observer in the same namespace/project as Data Hub, instantiate the
template as is in the desired namespace:

    oc project $SDH_NAMESPACE
    oc process -f https://raw.githubusercontent.com/miminar/sdh-vsystem-iptables-privilege-granter/master/vsystem-iptables-privilege-granter-template.yaml \
       | oc create -f -

### Deploying in a different project

If running in a different/new namespace/project, instantiate the
template with parameters `SDH_NAMESPACE` and `NAMESPACE`, e.g.:

    SDH_NAMESPACE=sdh26
    NAMESPACE=sapdatahub-admin
    oc new-project $NAMESPACE
    oc process -f https://raw.githubusercontent.com/miminar/sdh-vsystem-iptables-privilege-granter/master/vsystem-iptables-privilege-granter-template.yaml \ \
        SDH_NAMESPACE=$SDH_NAMESPACE NAMESPACE=$NAMESPACE | oc create -f -

