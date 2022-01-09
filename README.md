# OCI Host Provisioning

OCI host provisioning (OHP) is a host provisioning framework based on OCI registry as storage (ORAS) technology. OHP bundles and configures artifacts used for network booting into an OCI artifact that can be pushed into an OCI registry. A network boot client then chainloads the network boot artifacts from the OCI registry as http requests (without registry client).  

A configuration called a hostchain-config and an OCI registry tag are used to link the network booting resources together to create a provisioning chain. OCI manifests are used as the OHP hostchain definitions. 

When a hostchain is rendered from a hostchain-config and an OCI registry tag, OHP performs a series of substitutions while constructing a deep graph of OCI artifacts. Artifacts are hashed in reverse order of their chainloading precedence. The hash of each artifact is combined with the OCI registry tag to create the artifact URL. Each rendered artifact URL is substituted into the configuration of the preceding boot resource until all substitutions have been performed and all artifacts have been hashed. When the first boot resource in the provisioning chain is hashed, the OCI manifest is finalized and the URL of the first boot resource in the provisioning chain is output to the terminal. That URL is then referenced via DHCP options 66 and 67. 

Multi-arch boot media is supported by utilizing oci manifest lists. 


A hostchain might contain the following resources:

1. preboot env
2. os image(s)
3. ignition/cloud-init
4. additional artifacts (configs/arbitrary files)

