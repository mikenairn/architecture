# RFC DNSPolicy v1 changes and improvements

- Feature Name: (fill me in with a unique ident, `dns_policy_api_changes`)
- Start Date: (fill me in with today's date, 2024-09-11)
- RFC PR: [Kuadrant/architecture#0000](https://github.com/Kuadrant/architecture/pull/0000)
- Issue tracking: [https://github.com/Kuadrant/architecture/issues/104](https://github.com/Kuadrant/architecture/issues/104)

# Summary
[summary]: #summary

DNSPolicy is soon to go GA. There are several improvement we have identified over time that will enhance the GA of this API

- Remove the need for labels on Gateways for GEO and custom weighting values

- Reduce the verbosity of GEO and Weighting definitions

- Remove the need for a strategy to be chosen as part of the policy definition.


# Motivation
[motivation]: #motivation

We want to simplify and improve the DNSPolicy API and remove some of the legacy structures that have hung on since its original inception, as this involves some breaking changes we want these before we create a v1 API.

**Weighting and GEO attributes:**

The loadbalancing options we provide were first designed as part of an API that was intended to work with OCM (open cluster management). This provided multiple views of gateways across multiple clusters. So in order to understand the GEO context or individual weighing needed for a given cluster, we needed that context applying separately from the DNSPolicy spec that for legacy reasons targeted a "template" Gateway in the hub cluster.

Now DNSPolicy is created on the same cluster as the actual Gateway and we do not use OCM or hub clusters, the need to label individual objects and Gateways with specific annotations and labels is now redundant and makes for a more complex and awkward API interaction.

**routingStrategy:**

We have also identified that the routingStrategy option in the DNSPolicy spec is redundant. When added we expected there to be more than two strategies. This has not emerged and so it is another awkward piece of the API that is not needed.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

You will no longer need to apply labels to Gateways in order to specify the GEO or Weighting for that Gateway. The policy targets a given Gateway and you will now just specify those values in the policy spec directly.

You will no longer need to specify what routingStrategy you want to use. Instead you will either specify a loadbalancing section (meaning it is a loadbalanced strategy) or you will leave it empty (meaning it has no loadbalancing).

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

Below is an example of what is currently needed to setup GEO and Custom Weighting with the existing API

```
apiVersion: kuadrant.io/v1alpha1
kind: DNSPolicy
metadata:
  name: prod-web
  namespace: ingress-gateway
spec:
  targetRef:
    name: prod-web
    group: gateway.networking.k8s.io
    kind: Gateway
  routingStrategy: loabalanced  
  loadBalancing:
    weighted:
      defaultWeight: 120
      custom: # <--- New Custom Weights being added
        - weight: 255
          selector:
            matchLabels:
              kuadrant.io/lb-attribute-custom-weight: AWS # slects gateways to apply it to (when there can only be one)
    geo: 
      defaultGeo: US #catch all geo              


```
So here to apply a custom weighting, you have to specify the weighting under the custom section and then apply the `kuadrant.io/lb-attribute-custom-weight: AWS` label to the gateway that is already being targeted by the policy. 


To change the GEO for the targeted cluster, you need to apply a different label to the gateway: `kuadrant.io/lb-attribute-geo-code: EU` for example. 

On top of this you also have to specify that it is a load balanced DNSPolicy even though you have specified a load balancing section. 

This is an awkward and disconnected API that evolved from the legacy requirements called out above.


Instead the new API to achieve the same goal will be:

```
apiVersion: kuadrant.io/v1alpha1
kind: DNSPolicy
metadata:
  name: prod-web
  namespace: ingress-gateway
spec:
  targetRef:
    name: prod-web
    group: gateway.networking.k8s.io
    kind: Gateway
  loadBalancing:
    weight: 100 #weight for listeners targeted 
    geo: US # geo for listeners targeted 
    defaultGEO: true # should this be consisdered the default GEO for the listener hosts
  providerRefs:
    - name: aws-credential-secret    
```

So no longer do you need to specify whether the policy is a load balanced one or not via the redundant `routingStrategy` field


Now you simplify specify the weight you want to use for the listeners in the gateway, the geo you want to use and whether it should be used as a default GEO or not (this is used by some cloud providers as a catch-all option if a user from a none specified GEO does a DNS lookup.). Each of the fields under "loadbalancing" will now be required.

From an implementation perspective, all changes will happen in the Kuadarant Operator, where it will no longer look for the attribute labels on the gateways but instead will simply use the spec of the DNSPolicy. The resulting DNSRecord will not change in structure. 

To setup a simple DNS structure (single A or CNAME record), the API would now look like:

```
apiVersion: kuadrant.io/v1alpha1
kind: DNSPolicy
metadata:
  name: prod-web
  namespace: ingress-gateway
spec:
  targetRef:
    name: prod-web
    group: gateway.networking.k8s.io
    kind: Gateway
  providerRefs:
    - name: aws-credential-secret    
```

Again no need for the redundant strategy field. 

# Drawbacks
[drawbacks]: #drawbacks

Introduces a breaking change to the API.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

These are breaking changes and we are about to move to v1. Changes like these should land pre v1. These changes provide a much simpler and better user experience.

# Prior art
[prior-art]: #prior-art

NA

# Unresolved questions
[unresolved-questions]: #unresolved-questions

NA

# Future possibilities
[future-possibilities]: #future-possibilities

NA