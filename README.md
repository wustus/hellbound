# hellbound

An amalgamation of a variety of tools to check, verify and lint Helm Charts.

## Dependencies

This tool is only supposed to merge some useful projects into one incestuous script. The useful parts stem from the following tools (in calling-order):
- [helm](https://github.com/helm/helm)
- [yq](https://github.com/mikefarah/yq)
- [kube-linter](https://github.com/stackrox/kube-linter)
- [kubeconform](https://github.com/yannh/kubeconform)
- [python](https://www.python.org)
  * [pyyaml](https://github.com/yaml/pyyaml) 

## Etymology

For calling-convenience the following decisions have been made.
`helmbouncer` -> `helmbounce` -> `hellbound`

## Usage

`hellbound` leans on the well-known flags provided by `kubectl` and `helm`. We mostly need to pass files (`-f | --file`) which are to be used for templating.
```bash
hellbound -h

An amalgamation of a variety of tools to check, verify and lint Helm Charts.

Usage: hellbound <path> -f <values file>
  <path> - the path to the Helm Chart (default: .)

  -f|--file - the Helm value file to pass (default: values.yaml)
  -c|--config - path to config file.
  -k|--ignore-missing - ignore missing dependencies (except for helm)
  -v|--verbose - print errors (default: false)
  -h|--help - show this help.
```

Test a helm Chart:
```bash
hellbound -f values.yaml --config .hellbound.yaml -v
hellbound:
  [x] - Helm Template
  [x] - Kube Linter
  [ ] - Kube Conform
cilium.io/v2/CiliumClusterwideNetworkPolicy - host-fw-policy:
  Message: could not find schema for CiliumClusterwideNetworkPolicy

cilium.io/v2alpha1/CiliumL2AnnouncementPolicy - cilium-l2-announcement-policy:
  Message: could not find schema for CiliumL2AnnouncementPolicy

cilium.io/v2/CiliumLoadBalancerIPPool - cilium-load-balancer-ip-pool:
  Message: could not find schema for CiliumLoadBalancerIPPool

monitoring.coreos.com/v1/ServiceMonitor - hubble:
  Message: could not find schema for ServiceMonitor
```

### Dependencies

`hellbound` is nothing more than a series of tools in series used to test Helm Charts. Thus it works best if all tools are installed. I've made a few small efforts to make this script run without all the tools but due to self-set time constraints, I've neglected this a bit in the end.
The script does not start without [helm](https://github.com/helm/helm) being installed. [yq](https://github.com/mikefarah/yq) is used for config- and result-parsing and may be optional.
[kube-linter](https://github.com/stackrox/kube-linter) and [kubeconform](https://github.com/yannh/kubeconform) are both optional and are skipped if missing. 
`kubeconform` is able to validate CRD schemas. This feature relies upon `yq`, [python](https://www.python.org) and [pyyaml](https://github.com/yaml/pyyaml) being installed. The CRD sources are then defined in the `.hellbound.yaml` config file.

## Configuration

`hellbound` will run without configuration but can be extended using a `<configfile>.yaml`, or canonically called `.hellbound.yaml`. 
This file is passed to `kube-linter` directly and thus inherits the [kube-linter configuration language](https://docs.kubelinter.io/#/configuring-kubelinter). 
For now, this configuration is only extended by the `crdSchemas` array-key. These are Links to CRDs which, given all dependencies are installed, will be downloaded, written to the `schemas/` directory and used for validation with `kubeconform`. The `schemas/.sources` file keeps track of already installed sources.

> [!CAUTION]
> The `schemas/` directory will be created wherever the `hellbound` script is placed. The script to download and transform the CRDs is expected under `scripts/openapi2jsonschema.py`. It is recommended to leave script in its git Repository.

The `.hellbound.yaml` file may for example look like this:
```yaml
---
checks:
  exclude:
    - unset-cpu-requirements
    - unset-memory-requirements

crdSchemas:
- https://raw.githubusercontent.com/cert-manager/cert-manager/refs/heads/master/deploy/crds/acme.cert-manager.io_challenges.yaml
- https://raw.githubusercontent.com/cert-manager/cert-manager/refs/heads/master/deploy/crds/acme.cert-manager.io_orders.yaml
# ...
```

`kubeconform` is already called with a variety of CRD sources though so in many cases none have to be added.
