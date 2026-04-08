
# O-JSON-RPC (Om! JSON RPC)

Om! JSON RPC; is a [JSON](https://en.wikipedia.org/wiki/JSON) based [wire protocol](https://en.wikipedia.org/wiki/Wire_protocol) for [remote procedure calls](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC). It borrows concepts and ideas from [JSON-RPC](https://en.wikipedia.org/wiki/JSON-RPC) to define an opinionated, modern and [API](https://en.wikipedia.org/wiki/API) friendly standard.


# Main features

- API versioning support
- Multi procedure execution in a single request
- Support for sequential or parallel procedure execution
- Procedure execution timeout
- Resource subscriptions
- Authentication definition
- Pagination definition
- Automatic resource response deduplication

# v1 Specification

## Conventions

## Definitions

API: 

Procedure: 

Resource:

Subscription:

Input:

Output:

Authentication:



## Request definition

***protocol*** (required)

Defines the protocol version to be used when processing the request. \
So far there is only one supported version: `v1`

***api*** (required)

Defines the API to be used to process the request. \
*api* is an arbitrary value defined by the developer.

Below are some inspirational examples for API versioning values:
- `"api": "v1"`,
- `"api": "v2"`,
- `"api": "1.0.0"`,
- `"api": "version1"`

It can also be used to split an API by different domains.
- `"api": "application"`
- `"api": "management"`

Ultimately both versioning and domain names can be combined.
- `"api": "v1/application"`
- `"api": "1.0.0/management"`

***procedures*** (optional)

A collection of procedures to be executed.

*procedures[x].id* (required)

Unique identification value assigned to the procedure execution. \
The same procedure can be used multiple times in one single request. *id* allows to identify a single procedure execution in the response object.

*procedures[x].name* (required)

Name of the procedure to execute.

*procedures[x].input* (optional)

Input resource to pass to the procedure when executing. \
Although it is an optional property, procedures requiring an input parameter would return an {ERROR} error when the input is missing.

*procedures[x].pagination* (optional)

Set of properties to indicate to the procedure the result should be paginated.

*procedures[x].pagination.offset* (required)

*procedures[x].pagination.limit* (required)



***subscriptions*** (optional)

***options*** (optional)

## Response definition


## Errors definition

## Authentication options

TODO: review this part again
https://www.youtube.com/watch?v=iX8g4LqF8p8

### Basic <- not supported
Scheme: basic
Token: string
Token format: plain or base64


### Digest <- md5 not supported, only more secure hash formats are
Scheme: digest
Token: string
Token format: md5 | sha128


### API Key
Scheme: api_key
Token: string
Token format: sha128

### Session
Scheme: session
Token: string
Token format: 

### Bearer
Scheme: bearer
Token: string
Token format: jwt

### Access/Refresh token
Scheme: access_token | refresh_token
Token: string
Token type: access | refresh

### Identity Provider 
Scheme: identity_provider
Provider: string
Token: string
Token type: id | access | refresh



-----


## Let's start with a simple example:

Request:
```json
{
  "version": "1.0",
  "api": "v1",
  "procedures": [
    {
      "id": "1",
      "name": "ping"
    },
    {
      "id": "2",
      "name": "hello",
      "input": {
        "name": "World"
      }
    },
    {
      "id": "3",
      "name": "hello",
      "input": {
        "name": "JSON"
      }
    }
  ]
}
```


## procedures


# We have seen how to define a  request, let's see now what the server could respond back

```json
{
  "version": "v1",
  "api": "v1",
  "procedures": {
    "1": {
      "refs": [
        "Pong/0"
      ]
    },
    "2": {
      "refs": [
        "Hello/World"
      ]
    },
    "3": {
      "refs": [
        "Hello/JSON"
      ]
    }
  },
  "resources": {
    "Pong/0": "pong",
    "Hello/World": "Hello World!",
    "Hello/JSON": "Hello JSON!"
  }
}
```

Similar to the request, we get back `version` and `api`. This is the server hinting the client which protocol version processed the request and for which API. This information could be useful for clients to decide how to render the information when supporting multiple API versions.

`procedures` holds the result for the procedure execution. Notice that what we get back is the list of ids we defined in the request and for each of those ids a collection of resource references. Because a request could have the same procedure executing more than once and that procedure could yield the same resource for each execution the server deduplicates the resources.

We call a `resource` to the output of a procedure execution. 

The `resources` property holds a collection of all resources for the request.


# Pagination

# Execution strategy and other settings

# Authentication

# Debugging

# More examples




> When no additional settings is provided in the request the server executes the procedures in the order they are defined in the request.


-----

```json
{
  "jrpc": "v1",
  "api": "v1",
  "request": {
    "id": "request_id",
    "execution": {
      "strategy": "parallel",
      "timeout": 100,
      "procedure_timeout": 100
    },
    "authentication": {
      "scheme": "bearer",
      "token": "<redacted>",
      "token_format": "JWT"
    },
    "return": {
      "request_id": true,
      "total_execution_time": true,
      "procedure_execution_time": true,
      "execution_strategy": true,
      "execution_order": true
    }
  },
  "procedures": [
    {
      "id": "ping",
      "name": "ping"
    },
    {
      "id": "sayHi",
      "name": "sayHi",
      "input": {
        "person": "World"
      }
    },
    {
      "id": "transformText",
      "name": "transformText",
      "input": {
        "text": "text to transform"
      }
    },
    {
      "id": "getUsersList",
      "name": "getUsersList",
      "input": {
        "offset": 0,
        "limit": 2
      }
    }
  ],
  "subscriptions": [
    {
      "resource": "TransformedText"
    }
  ]
}
```


```json
{
  "version": "v1",
  "api": "v1",
  "request": {
    "id": "request_id",
    "execution": {
      "ping": {
        "order": 1,
        "time": 1
      },
      "sayHi": {
        "order": 2,
        "time": 5
      },
      "transformText": {
        "order": 3,
        "time": 5
      },
      "getUsersList": {
        "order": 4,
        "time": 1000
      }
    }
  },
  "procedures": {
    "ping": {
      "resources": [
        "Pong/0"
      ]
    },
    "sayHi": {
      "resources": [
        "Hello/0"
      ]
    },
    "transformText": {
      "resources": [
        "TransformedText/0"
      ]
    },
    "usersList": {
      "resources": [ 
        "User/1",
        "User/2"
      ],
      "offset": 0,
      "limit": 2,
      "total": 10
    }
  },
  "resources": {
    "Pong/0": "pong",
    "Hello/0": "Hello World!",
    "TransformedText/0": {
      "original": "text to transform",
      "transformed": "Text To Transform"
    },
    "User/1": {},
    "User/2": {}
  }
}
```



The next property we are going to talk about is `authentication`. Authentication allows the server to check the client has the right credentials before allowing access to execute any operation. The #authentication# section has more details regarding which options are supported by the protocol. 

In this request we want to authenticate the client using a JWT bearer token; as you can see; the `scheme` defines what type of authentication we want to execute; `token` is 
the token value and `token_format` in which format the token is.


`settings` is an optional property in the request that allows you to configure different aspects of the request 


- `operations` - list of operations to execute



Response:

```json
{
  "version": "v1",
  "api": "v1",
  "details": {
    "request_id": "request_id",
    "operations": {
      "ping": {
        "order": 1,
        "time_taken": 1
      },
      "sayHi": {
        "order": 2,
        "time_taken": 5
      },
      "transformText": {
        "order": 3,
        "time_taken": 5
      },
      "getUsersList": {
        "order": 4,
        "time_taken": 1000
      }
    }
  },
  "procedures": {
    "ping": {
      "resources": [
        "Pong/0"
      ]
    },
    "sayHi": {
      "resources": [
        "Hello/0"
      ]
    },
    "transformText": {
      "resources": [
        "TransformedText/0"
      ]
    },
    "usersList": {
      "resources": ["..."],
      "offset": 100,
      "limit": 100,
      "total": 10000
    }
  },
  "resources": {
    "Pong/0": "pong",
    "Hello/0": "Hello World!",
    "TransformedText/0": {
      "original": "text to transform",
      "transformed": "Text To Transform"
    },
    "User/1": {}
  }
}
```





## Operations


Assuming we have a `User` resource that looks like: 

```json
{
  "id": "123",
  "first_name": "John",
  "last_name": "Doe",
  "is_deleted": false,
  "is_active": "true"
}
```

let's see some operations example

```json
{
  "id": "create-user",
  "create": "User",
  "properties": {
    "id": "123",
    "first_name": "John",
    "last_name": "Doe"
  },
  "return": "*"
}
```


## Settings

The `settings` section allows you to provide a set of options to send out the request.
From instructing the server on how to execute the list of operations to securely authenticate the request.

Note: all main properties are optional

```json
{
  "settings": {
    "execution_strategy": "sequential|parallel",
    "return_execution_time": true,
    "authentication": {}
  }
}
```

- `execution_strategy` - use `sequential` to execute one operation after the other in the order provided; use `parallel` to execute all operations in parallel. The server always returns all operations results at once (excluding subscriptions).
- `return_execution_time` - when true, this asks the server to return how long the operation took to execute in milliseconds
- `authentication` - provides options to authenticate the request 

### Authentication
v1 support a few authentication options. 
Keep in mind that regardless of which authentication scheme you choose to use, relying on a secured connection improves
the overall security of the request.

#### authentication.schema = 'basic'
Is the simplest authentication scheme

```json
{
  "authentication": {
    "scheme": "basic",
    "credential": "user@email.com",
    "secret": "mypassword",
    "secret_format": "base64"
  }
}
```

- `scheme` - defines which schema to use for the authentication step, in this case `basic`
- `credential` - defines a publicly known information, mostly the user email address or their username
- `secret` - secret information that should be either encrypted or obfuscated, most of the time the user's password as the user enters it
- `secret_format` - the format in which the `secret` is transmitted 

#### authentication.schema = 'bearer'
A more advanced and secure authentication scheme

```json
{
  "authentication": {
    "scheme": "bearer",
    "token": "some token information",
    "token_format": "JWT"
  }
}
```

- `scheme` - defines which schema to use for the authentication step, in this case `bearer`
- `token` - the token that was previously assigned by the server 
- `token_format` - the format in which the `token` is transmitted



# Response section



# Implementations

## [o-json-rpc-ts-server](https://github.com/odiel/o-json-rpc-ts-server)

Is an open source implementation of [O-JSON-RPC](https://github.com/odiel/o-json-rpc) written in [Typescript](https://en.wikipedia.org/wiki/TypeScript) using [Deno runtime](https://en.wikipedia.org/wiki/Deno_(software)) and operating over [HTTP](https://en.wikipedia.org/wiki/HTTP) and [Websocket](https://en.wikipedia.org/wiki/WebSocket) protocols.