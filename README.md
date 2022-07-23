# Gatekeeper for Istio

> Never Trust, Always Verify.

Install Gatekeeper
==================
Please follow the [Gatekeeper Installation Guide](https://open-policy-agent.github.io/gatekeeper/website/docs/install) to install Gatekeeper (>= v3.9.0).

```
$ kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.9.0/deploy/gatekeeper.yaml
...
```

Install ContraintTemplates
==========================
[OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper) is based on [OPA Constraint Framework](https://github.com/open-policy-agent/frameworks/tree/master/constraint).

```
$ kubectl apply -k ./
...
```

Install Contraints
==================
```
$ kubectl apply -f explicit-protocol-selection/examples/require-explicit-protocol-selection/constraint.yaml 
istioexplicitprotocolselection.constraints.gatekeeper.sh/explicitprotocolselection created
```

Verify no contraint violations:
```
$ kubectl get constraints
NAME                                                                                 ENFORCEMENT-ACTION   TOTAL-VIOLATIONS
istioexplicitprotocolselection.constraints.gatekeeper.sh/explicitprotocolselection   deny                 0
```

Unit Test
=========
Install [Gator](https://open-policy-agent.github.io/gatekeeper/website/docs/gator) for contraints evaluation. This model also can be used in CICD pipeline (thinking about shift-left) as well.

```
$ gator verify explicit-protocol-selection/suite.yaml
ok      Users/I546303/Documents/Workspace/github/gatekeeper-istio/explicit-protocol-selection/suite.yaml        0.027s
PASS
```
System Test
===========
1. Test allowed Service appProtocol
```
$ kubectl apply -f explicit-protocol-selection/examples/require-explicit-protocol-selection/allow-app-protocol.yaml --dry-run=server
service/my-service created (server dry run)
```

2. Test disallowed Service port name
```
$ kubectl apply -f explicit-protocol-selection/examples/require-explicit-protocol-selection/disallow-port-name.yaml --dry-run=server 
Error from server (Forbidden): error when creating "explicit-protocol-selection/examples/require-explicit-protocol-selection/disallow-port-name.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [explicitprotocolselection] port: {"name": "http3", "port": 443, "protocol": "TCP", "targetPort": 443} name or appProtocol is invalid
```