
# O-JSON-RPC (Om JSON RPC)

Om JSON RPC; is a [JSON](https://en.wikipedia.org/wiki/JSON) based [wire protocol](https://en.wikipedia.org/wiki/Wire_protocol) for [remote procedure calls](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC). \
It borrows concepts and ideas from different technologies to define a slightly opinionated, modern and backend friendly [API](https://en.wikipedia.org/wiki/API) standard.

# Main features

- API versioning and/or namespacing
- Multi procedure execution in a single request
- Configurable procedure execution, sequentially or in parallel
- Procedure timeout configuration
- Resource change notifications through subscriptions
- Request authentication definition

## v1 protocol
- [Specification](./v1/specification.md)
- [Request and response examples](./v1/examples.md)
- [Q&A](./v1/qa.md)

## Known implementations

- [O-JSON-RPC-TS](https://github.com/odiel/o-json-rpc-ts) - Typescript implementation using Deno


