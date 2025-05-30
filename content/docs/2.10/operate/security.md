+++
title = "Security"
description = "Guidance to configure security options"
weight = 100
+++

## Use your own TLS Certificates

KEDA uses self-signed certificates for different things. These certificates are generated and rotated by the operator. Certificates are stored in a Kubernetes secret (`kedaorg-certs`) that it's mounted to all KEDA components in the (default) path `/certs`. Generated files are named `tls.crt` and `tls.key` for TLS certificate and `ca.crt` and `ca.key` for CA certificate. KEDA also patches Kubernetes resources to include the `caBundle`, making Kubernetes to trust in the CA.

While this is a good starting point, some end-users may want to use their own certificates which are generated from their own CA in order to improve security. This can be done by disabling the certificate generation/rotation in the operator and updating default values in other components (if required). 

The KEDA operator is responsible for generating certificates for all the services, this behaviour can be disabled removing the console argument `--enable-cert-rotation=true` or setting it to `false`. Once this setting is disabled, user given certs can be placed in the secret `kedaorg-certs` which is automatically mounted in all the components or they can be patched to use other secret (this can be done through helm values too).

All components inspect the folder `/certs` for any certificates inside it. Argument `--cert-dir` can be used to specify another folder to be used as a source for certificates, this argument can be patched in the manifests or using Helm values. Because these certificates are also used for internal communication between KEDA components, the CA is also required to be registered as a trusted CA inside KEDA components.

## Register your own CA in KEDA Operator Trusted Store

There are use cases where we need to use self-signed CAs (cases like AWS where their CA isn't registered as trusted etc.). Some scalers allow skipping the cert validation by setting the `unsafeSsl` parameter, but this isn't ideal because it allows any certificate, which is not secure.

To overcome this problem, KEDA supports registering custom CAs to be used by SDKs where it is possible. To register custom CAs, you need to ensure that the certs are in `/custom/ca` folder and KEDA will try to register as trusted CAs all certificates inside this folder. This can be done with kustomize or helm (using `volumes.keda.extraVolumes` and `volumes.keda.extraVolumeMounts`).
