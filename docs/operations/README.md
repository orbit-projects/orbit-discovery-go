# Orbit Discovery Go Contract Library: operations and security

This guide organizes runtime behavior documented by the package. It does not certify production readiness. Verify provider/client versions, permissions, transport security, limits, and failure behavior in the target environment before release.

## Configuration surface

Environment names found in the package README:

The package README does not name `ORBIT_*` variables. Use its typed constructors and application configuration, and confirm exact runtime inputs in the implementation before deployment.

Use the package README's constructor and deployment examples. Store credentials in a secret manager and avoid logging credentials, raw provider errors, request data, or opaque cursors.

## Lifecycle, failure behavior, and limits

`orbit-discovery-go` contains the generated Go bindings and bounded client/server helpers for the
provider-neutral `orbit.discovery.v1.ServiceDiscovery` contract. It helps a Go implementation
conform to the same capability API used by Python `orbit-discovery`; it is not itself a concrete
service-discovery provider. For example, `orbit-kubernetes-go` uses this contract library and
implements Kubernetes EndpointSlice discovery in Go. Concrete providers own their vendor SDKs,
credentials, configuration, transport, lifecycle, and provider failures.

The versioned Protobuf schema is mirrored from `orbit-discovery/proto/orbit_discovery/v1`. The
Python capability package owns the contract definition; keep this copy and generated bindings in
sync when that contract changes. Generated types, client, server interface, and registration helpers
are public under `github.com/orbit-projects/orbit-discovery-go/v1`.

```go
import (
	"context"

	"github.com/orbit-projects/orbit-discovery-go/discovery"
	"google.golang.org/grpc"
)

type catalogResolver struct{}

func (catalogResolver) Resolve(_ context.Context, service string) ([]discovery.Instance, error) {
	if service != "catalog" {
		return nil, nil
	}
	return []discovery.Instance{{
		Service: "catalog", InstanceID: "catalog-1", Host: "127.0.0.1",
		Port: 8081, Scheme: "http", Weight: 1,
	}}, nil
}

func register(server *grpc.Server) error {
	return discovery.Register(server, catalogResolver{})
}
```

`discovery.Register` validates requests and provider results against the language-neutral bounded
contract, returns sanitized gRPC errors, and honors context cancellation. `discovery.NewClient`
offers the matching Go client API. The package limits each result to 4,096 endpoints and each wire
message to 1 MiB, and bounds endpoint identifiers, host syntax, ports, weights, and metadata. The
SDK validates endpoint syntax; callers still need egress policy to prevent SSRF when connecting to
resolved hosts.

## Production validation

Validate startup/shutdown cleanup, timeout and cancellation behavior, concurrency and payload bounds where applicable, secret rotation and least-privilege access, data durability, backup/restore, and failover against the selected provider. Do not infer distributed or durable guarantees from an in-process API or fake-client tests.
