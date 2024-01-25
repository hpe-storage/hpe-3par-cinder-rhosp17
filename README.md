# HPE 3PAR, Primera and Alletra 9000 Deployment Guide for RHOSP17.1

## Overview

This page provides detailed steps on how to enable the containerization of HPE 3PAR, Primera and Alletra 9000 Cinder driver on top of the OSP Cinder images.
It also contains steps to deploy & configure HPE 3PAR family backends for RHOSP17.1.

## Prerequisites

* Red Hat OpenStack Platform 17.1 with RHEL 9.

* HPE 3PAR array 3.3.1 OR HPE Primera array 4.2 or higher.

## Steps

### 1.	Prepare the Environment Files for HPE 3PAR cinder-volume container and cinder backend

#### 1.1 Environment File for cinder-volume container

To use HPE 3PAR/Primera/Alletra 9000 hardware as a Block Storage back end, cinder-volume container having python-3parclient needs to be deployed.

Procedure

Create a new container images file for your overcloud:

```
openstack tripleo container image prepare default \
    --local-push-destination \
    --output-env-file containers-prepare-parameter-hpe.yaml
```

Edit the containers-prepare-parameter-hpe.yaml file.

Sample file is available in [templates](https://github.com/hpe-storage/hpe-3par-cinder-rhosp17/blob/master/templates) folder for reference.

When director deploys the overcloud, the overcloud uses the HPE container image instead of the standard container image.


#### 1.2 Environment File for cinder backend

The environment file is an OSP director environment file. The environment file contains the settings for each backend you want to define.

Create the environment file “cinder-hpe-primera-[iscsi|fc].yaml” under /home/stack/templates/ with below parameters and other backend details.

```
parameter_defaults:
  CinderEnableIscsiBackend: false
  ControllerExtraConfig:
```

Sample files for both iSCSI and FC backends are available in [templates](https://github.com/hpe-storage/hpe-3par-cinder-rhosp17/blob/master/templates) folder for reference.

#### Additional Help

For further details of HPE 3PAR cinder driver, kindly refer documentation [here](https://docs.openstack.org/cinder/wallaby/configuration/block-storage/drivers/hpe-3par-driver.html)


### 2.	Deploy the overcloud and configured backends

After creating ```cinder-hpe-primera-[iscsi|fc].yaml``` file with appropriate backends, deploy the backend configuration by running the openstack overcloud deploy command using the templates option.
Use the ```-e``` option to include the environment file ```cinder-hpe-primera-[iscsi|fc].yaml```.

The order of the environment files (.yaml) is important as the parameters and resources defined in subsequent environment files take precedence.

```
openstack overcloud deploy --templates /usr/share/openstack-tripleo-heat-templates \
    -e /home/stack/templates/node-info.yaml \
    -e /home/stack/containers-prepare-parameter-hpe.yaml \
    -e /home/stack/templates/cinder-hpe-primera-[iscsi|fc].yaml \
    --ntp-server <ntp_server_ip> \
    --debug
```

### 3.	Verify the configured changes

* Run "openstack volume service list" on the overcloud, and verify the 3PAR backend is up

* Verify a volume can be created

