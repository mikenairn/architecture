# Kuadrant Proposal - Modular Installation

Kuadrant is developing a set of loosely coupled functionalities built directly on top of Kubernetes.
Kuadrant aims to allow customers to just install, use and understand those functionalities they need.
Currently, the installation tool of kuadrant, the [kuadrantctl CLI](https://github.com/Kuadrant/kuadrantctl),
installs all or nothing. For instance, a customer not wanting to rate limit the API,
should not have to install the rate limit component ([Limitador](https://github.com/Kuadrant/limitador)).

## High Level Goals

* Install only required components. Operate only required components.
* Expose only the kuadrant API that has available implementation by the with the deployed components

## Installation profiles

The kuadrant installation mechanism should offer modular installation to enable pieces of kuadrant.
Modular installation options should be feature oriented rather than deployment component oriented.
Then, it is up to the installation tool to decide what components need to be deployed and how to
configure it.

Kuadrant features:

* Multiple APIs bundled in a product
* Service Discovery
* TLS
* Authentication (mTLS, API key, OIDC)
* Rate Limiting (pre-auth, post-auth)

Each feature, or part of it, is eligible to be included or excluded when installing kuadrant.

Some profiles can be defined to group set of commonly required features. Naming the profiles
allows the customer to easily express wanted installation configuration. Furthermore, profiles
not only can be used to group a set of features, profiles can be used to define deployment options.

| Name | Description |
| ---  | --- |
| **Minimal** | Minimal installation required to run an API without any protection, analytics or API management |
| **Authenticated** | MTLS, API key and OIDC authentication features |
| **RateLimit** | Basic rate limit (only pre-auth rate limit) features |
| **Full** | Full featured kuadrant installation |
| **Development** | Full featured kuadrant for development purposes. Development deployment mode: debug log level, no cache, tracing |

A kuadrant operator, together with a design of a kuadrant CRD is desired.
Not only for kuadrant installation, but also for lifecycle management.
Additionally, the [kuadrantctl](https://github.com/Kuadrant/kuadrantctl) CLI tool can also
be used to install kuadrant in a profile basis. The CLI tool may also accept, optionally, the kuadrant
custom resource as input.

## Available API

The API exposed to the customer is defined in terms of k8s Custom Resource Definitions.
Kuadrant provides multiple CRDs to configure kuadrant API management.
Often, a kuadrant feature's fully definition spans multiple CRDs, so the available API cannot be
accurately defined with a set of CRD. In other words, there is not such a mapping function that maps
each feature with a CRD.

The [kuadrant-controller](https://github.com/Kuadrant/kuadrant-controller) will be aware of the
installation profile via env vars (or command line params). With that information,
the kuadrant controller will reject any configuration object (all the object at once as partial rejection is not supported)
when that configuration requires something (could be a component or deployment option) from kuadrant
that is not enabled as part of the installation process.

The customer will be only able to configure available features, nothing more.
