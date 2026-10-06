# Orbit Discovery Go Contract Library: architecture and boundaries

## Responsibility

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

## Declared dependencies

The following dependency declarations come from the checked-in manifests. Optional groups and development dependencies are called out separately.

### `go.mod`
- `golang.org/x/net v0.41.0`
- `google.golang.org/grpc v1.75.1`
- `google.golang.org/protobuf v1.36.6`
- `golang.org/x/sys v0.33.0 // indirect`
- `golang.org/x/text v0.26.0 // indirect`
- `google.golang.org/genproto/googleapis/rpc v0.0.0-20250707201910-8d1bb00bc6a7 // indirect`

Declared dependencies do not mean that optional providers or services are bundled with this package.

## Implementation layout

Representative implementation files in this checkout:

- `discovery/discovery.go`
- `v1/service_discovery.pb.go`
- `v1/service_discovery_grpc.pb.go`

## Public contract and scope

# Orbit Discovery Go Contract Library

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

## Development

Requires Go 1.24 or later and `protoc` for regeneration. Generated Go files are checked in, so
consumers do not need the compiler.

```sh
make tools
make generate
make test
make vet
make build
```

Tests use an in-memory gRPC transport. They do not verify a live registry, a Kubernetes cluster,
Python Core process hosting, or release provenance. This package is pre-alpha and is not a
production certification.

Licensed under Apache-2.0.

## Boundary rules

Keep provider SDKs, credentials, transports, and provider-specific error translation in provider adapters. Keep reusable capability contracts in the matching capability package and lifecycle orchestration in Core. Apply the relevant layer for this repository and preserve the dependency direction shown above.
