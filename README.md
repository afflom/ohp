# OCI Host Provisioning

OHP is in the planning/design phase of development. This repo is intended to contain the specification of OHP and a client that implements a reference design of OHP. Any feedback on the below content and/or OHP concepts would be very appreciated. 

## Overview
OCI host provisioning (OHP) is a host provisioning framework based on OCI registry as storage ([ORAS](https://oras.land)) technology. OHP bundles and configures artifacts used for network booting into an OCI artifact that can be pushed into an OCI registry. Several host provisioning workflows can then be used to provision hosts from the OCI artifact such as; A network boot client chainloading the network boot artifacts from the OCI registry as HTTP requests (without registry client) or a virtual machine (VM) manager (such as kubevirt) consuming the entire OCI artifact and applying it to a VM config.  


## Host Chains
A configuration called a hostchain-config and an OCI registry tag are used to link the network booting resources together to create a provisioning chain. OCI manifests are used as the OHP host chain definitions. 

When a host chain is rendered from a hostchain-config and an OCI registry tag, OHP performs a series of substitutions while constructing a [deep graph of OCI artifacts](https://oras.land/cli/6_reference_types/#:~:text=example%0A%20%20%20%20%E2%94%94%E2%94%80%E2%94%80%20sha256%3A1b6308bc4a2dd8933e9f66ff5bbc47e685516e5378208b46c58dc...-,creating%20deep%20graphs%20of%20artifacts,-The%20ORAS%20Artifacts). Artifacts are hashed in reverse order of their chainloading dependencies. The hash of each artifact is combined with the OCI registry tag to create the artifact URI. Each rendered artifact URI is substituted into the configuration of the preceding boot resource until all substitutions have been performed and all artifacts have been hashed. When the first boot resource in the provisioning chain is hashed, the OCI manifest is finalized and the URI of the first boot resource in the provisioning chain is output for user reference. That URI can then be referenced via DHCP options or the URI with in-line basic auth can also be directly injected into a UEFI host via Redfish. There are many ways to configure pre-boot environments to reference a OHP host chain or the entire OCI artifact. 

A host chain might contain the following resources:

1. preboot env
2. os image(s)
3. ignition/cloud-init
4. additional artifacts (configs/arbitrary files)

## hostchain-config

A hostchain-config is an array of boot resource objects that contain key/value pairs that correspond to variable substitutions within the referenced boot resources. OHP provides an extensible framework for adding resource types that OHP uses to process the hostchain-config

[Here is an example of a basic hostchain-config for Fedora CoreOS](./example-hostchain-config.json) that implements [this example](https://docs.fedoraproject.org/en-US/fedora-coreos/live-booting-ipxe/#_setting_up_the_boot_script) from the Fedora CoreOS docs. 

OHP will perform the following actions when processing the above hostchain-config:

1. Generate a custom preboot config
2. Generate a custom  preboot image with the custom config
3. Hash each boot resource
4. Render an OCI manifest for the hostchain
5. Output relevant information to configure DHCP network booting options



[Here is an example manifest that would be generated from ./example-hostchain-config.json referenced above.](./example-manifest.json)

## Distribution
Once the OHP hostchain has been rendered in OCI format, it can be pushed into an OCI registry. The 


## Multi-arch
Multi-arch boot media is supported by utilizing oci manifest lists. 

## Custom Resource Types

Support for custom resource types can be added to OHP by adding the logic to parse a hostchain-config to a program and adding it to the execution environment's $PATH with the hostconfig prefix. For example, to add a custom pxelinux resource type, author a script or program to process a hostchain-config resource object and add that program or script to the execution environment's $PATH as `hostchain-pxelinux`. When OHP encounters a `pxelinux` resource type within the hostchain config, it will call `hostchain-pxelinux` from the $PATH.


