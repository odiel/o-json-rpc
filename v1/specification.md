# v1 specification

## Definitions

### API 
An arbitrary value to define a version, namespace or a combination of both.

### Procedure
Reusable piece of code logic to be executed when called by its assigned name.

### Resource
Composed by a name and a JSON schema; it defines a typed dataset that is used as input arguments for executing procedures as well as the result of a procedure output.
Resource names also have a use as reference for client applications that are interested in getting notified every time a resource changes.    

### Subscription
A mechanism that allows clients to connect to the server and receive notifications when changes for a Resource are broadcast.

### Authentication
Authentication is a required step where the client is required to provide a set of credentials in order to gain access to content with limited access.

## Request object properties - [schema definition](./request.json)

This section defines the JSON request object and all its properties.

### protocol (required; string)

Defines the O-JSON-RPC protocol version to be used when processing the request. \
`v1` is the only supported version so far.

### api (required; string)

API to be used when processing the request. Is an arbitrary value defined by the developer to group a set of procedures.

Versioning examples:
- `"api": "v1"`,
- `"api": "v2"`,
- `"api": "1.0.0"`,
- `"api": "version1"`

It can also be used to split an API by different namespaces.
- `"api": "application"`
- `"api": "management"`

Examples combining both versioning and namespace.
- `"api": "v1/application"`
- `"api": "1.0.0/management"`
- `"api": "application/v1"`
- `"api": "management/1.0.0"`

### procedures (optional; strict object)

Collection of procedure definitions to be executed for the request.

`procedures[].id` (required; string)

Unique identifier assigned to the execution of the procedure for that request. \
Because the same procedure name can be used multiple times in one single request; *id* allows to identify a particular procedure execution.

`procedures[].name` (required; string)

Name of the procedure to execute.

`procedures[].input` (optional; JSON value)

Input resource to pass to the procedure for execution. \
Although it is listed as optional, procedures defining an input resource must enforce validating its content before execution.
Failing to validate the input resource must result in a PROCEDURE:INCOMPATIBLE_INPUT error.

### subscriptions (optional; collection of strict object)

This section in defines a list of objects to subscribe to specific resources to get notifications when a change happens.
Note: the client should establish a direct channel with the backend to get notified.

`subscriptions.resource_name` (required, name of the resource to subscribe)

### options (optional; strict object)

Allows the client to configure aspects of the request as well as hinting needed response details.

`options.id` (optional; string)

Arbitrary id value to assign to the request. \
The server implementation must either assign a random id value or use the one provided by the client to uniquely identify a request.

`options.execution` (optional; strict object)

Section to configure the request execution.

`options.execution.strategy` (optional; one of the following below)

- `sequential`: executes one procedure at the time; the response is provided once the last procedure execution is done; this must be the default execution strategy if none is provided.
- `parallel`: executes the procedures in parallel, the response is provided once all procedures execution finish.

`options.execution.procedure_timeout` (optional; number)

Number of milliseconds a procedure should run for before timing it out. 

`options.return` (optional; collection of fixed string values)

Instruct the server to provide additional details in the response object.
When this option is provided at least one value must be set.
One or any combination of values is acceptable.

- `request_id` - tells the server to return the request id.
- `request_execution_time` - tells the server to return the total execution time of the request; value must be in milliseconds. 
- `procedures_execution_details` - tells the server to return details for each procedure execution; check the `Response object properties` section below for more information.

`options.authentication` (optional; strict object); the `Authentication options` section below details this option.

## Authentication options

O-JSON-RPC does not specify any implementation details regarding the handling of the authentication mechanism;
this option is only a space in the request object for the client to present the credentials to the server application for a request.

`options.authentication.scheme` (required, string)

Scheme to use in the authentication.

`options.authentication.token` (required, string)

Value of the credential token for the authentication.

`options.authentication.token_type` (required, string)

Type or format of the token.

`options.authentication.provider` (optional, string)

Applicable only when `scheme` is `identity_provider`. Name of the provider for the identity.

### Supported mechanisms

#### Session
```
scheme: session
token_type: plain-text | base64
````

#### API Key
```
scheme: api_key
token_type: plain-text | base64
````

#### Token
```
scheme: access_token | refresh_token
token_type: jwt
````

#### Identity Provider
```
scheme: identity_provider
token_type: id | access | refresh
provider: string
````

## Response object properties - [schema definition](./response.json)

This section defines the JSON response object and all its properties.

### protocol (required)

When the request is properly handled the protocol version must match the same version from the request. \
When the server is unable to identify the protocol in the request it must default to the lowest known protocol version, that is `v1`.

### api (required)

API value, must match the same value from the request when the API is registered in the server; otherwise `unknown`.

### error (optional; strict object)

Section produced as a result of a failure during the processing of the request.

- `error.code` (required; string) error code, see the list below
- `error.message` (optional; string) additional error information

### procedures (optional; strict object)

Map of results for the procedures execution.
Indexed by the procedure id given in the request object.

There are 2 possible outcomes of a procedure execution `error` or `result`, only one or the other must appear.

`procedures[].error` (required; strict object) required when there is any type of failure during the procedure execution.

- `procedures[].error.code` (required; string) error code, see the list below.
- `procedures[].error.message` (optional; string) additional error information.

`procedures[].result` (required; any JSON value) when the procedure execution is successful the output must be stored in the `result` property.

### details (optional; strict object)

This section must appear when the request is sent with any value in `options.return`.

`details.request_id` (required when `options.return['request_id']`) id of the request. \
`details.execution_time` (required when `options.return['request_execution_time']`) total number of milliseconds the request took to process. \
`details.procedures_execution` (required when `options.return['procedures_execution_details']` map of the detailed run for each procedure.
- `details.procedures_execution[].id` (required; string) id assigned to the procedure execution in the request.
- `details.procedures_execution[].procedure` (required, string) name of the procedure executed.
- `details.procedures_execution[].order` (required, number) order number in which the procedure was executed in the server.
- `details.procedures_execution[].timed_out` (required, boolean) true when the procedure timed out.
- `details.procedures_execution[].execution_time` (required, number) number of milliseconds it took to run the procedure; value must be set to the timeout configuration when exceeding it.

## Errors definition

There are 2 main categories of errors:

- `SERVER` for errors specific to the server execution and general request processing. Server errors are must be returned in the main response object.
- `PROCEDURE` for errors related to each procedure execution. Procedure errors are scoped to a particular procedure execution result.

For errors where the request content can not be parsed the server must default to the lowest protocol version, that is `v1`.

`SERVER:UPGRADE_REQUEST_NOT_SUPPORTED`

Must be returned when upgrading the client request is not supported by the server implementation.

`SERVER:REQUEST_CONTENT_TOO_BIG`

Must be returned when the request content exceed the allowed content size by the server configuration.

`SERVER:REQUEST_METHOD_NOT_SUPPORTED`

Must be returned when the request method is not supported by the server implementation.

`SERVER:INCOMPATIBLE_REQUEST_CONTENT`

Must be returned when the request object does not match the [schema](./request.json) definition.

`SERVER:INCOMPATIBLE_RESPONSE_CONTENT`

Must be returned when the response object does not match the [schema](./response.json) definition.

`SERVER:NOT_AUTHENTICATED`

Must be returned when the client has not previously authenticated.

`SERVER:NOT_AUTHORIZED`

Must be returned when the client is not authorized to execute the request.

`SERVER:UNHANDLED_ERROR`

Must be returned when the server catches an unhandled error.


`PROCEDURE:INCOMPATIBLE_INPUT`

Must be returned when the input for the procedure is either missing or does not match its resource schema.

`PROCEDURE:INCOMPATIBLE_OUTPUT`

Must be returned when the output of the procedure is either missing or does not match its resource schema.

`PROCEDURE:NOT_FOUND`

Must be returned when the procedure in the request can not be found.

`PROCEDURE:TIMEOUT`

Must be returned when the procedure execution exceeds its configured time.

`PROCEDURE:NOT_AUTHORIZED`

Must be returned when the client is not authorized to execute the procedure.

