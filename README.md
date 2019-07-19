# Apache NiFi for Openshift

This repository contains all necessary resources to deploy NiFi flows in Openshift Container Platform. Concretely:

- A modified Dockerfile based on [the one provided in the official Apache NiFi Dockerhub image](https://hub.docker.com/r/apache/nifi/Dockerfile). Along the Dockerfile, a set of modified startup scripts and configuration files are included as well to allow configuration via environment variables.

- An **Openshift template** to simplify the deployment of NiFi flows in Openshift Container Platform.

- An example Dockerfile to demonstrate how to deploy flows using the modified image as base image.

The resources included in this repository **should be considered as a PoC and won't be maintained for future Apache NiFi versions**, but can be used as a starting point to understand how to deploy NiFi in OCP.

> **Note**:This is by no means an official image to deploy Apache NiFi in OCP, and it's not supported by Red Hat in any way.

## Modified base image

The Dockerfile included in the base directory contains all necessary modifications to support arbitrary user IDs on execution as specified in [the image creation guidelines for Openshift Container Platform.](https://docs.openshift.com/container-platform/3.11/creating_images/guidelines.html#openshift-specific-guidelines).

As stated before, the startup scripts and the bootstrap.conf configuration file have been modified to allow further configuration via environment variables. The following input variables have been added, all of them related to JVM configuration:

- **NIFI_JAVA_XMS**: Startup heap to be requested by the JVM.

- **NIFI_JAVA_XMX**: Maximum heap to be requested by the JVM.

- **NIFI_TIMEZONE**: Timezone to be used by the JVM. Must be expressed including the -Duser.timezone parameter as seen by default in the included Openshift Template.

This base image can be built in OCP by using a BuildConfig with binary source and Docker strategy such as the following:

```
apiVersion: v1
kind: BuildConfig
metadata:
  labels:
    app: nifi-ocp
  name: nifi-ocp
spec:
  nodeSelector:
  output:
    to:
      kind: ImageStreamTag
      name: "nifi-ocp:latest"
  runPolicy: Serial
  source:
    binary: {}
    type: Binary
  strategy:
    dockerStrategy: {}
```

Once the BuildConfig has been created, the base image can then be build for example with the following command, issued at the root path of this repository:

```
oc start-build nifi-ocp --from-file=./base
```

> **Note**: All modifications performed to the Dockerfile are focused on making the minimum amount of changes to the official image to make it work in OCP, and shouldn't be considered the optimal approach for application/middleware onboarding on OCP. For security purposes, **it is strongly recommended to use the supported RHEL based images as base image for anything deployed in OCP**.

## Openshift Template

An Openshift Template can be found as well in the repository. This template creates all necessary objects to build, deploy and run NiFi flows in OCP. This approach considers the flow as an artifact, and the NiFi image as a runtime image.

In order to use this template, the base image should be available in a ImageStream reachable by the created BuildConfig.

## Example Dockerfile

This Dockerfile can be used to build flow images using the template included in this repository. Once the template has been created and processed in the target namespace/project, flow images can be built using the following command, where the flow directory also contains the flow file to be deployed:

```
oc start-build myflow --from-file=./flow
```
