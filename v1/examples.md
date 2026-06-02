
# List of request and responses examples to better illustrate the protocol behavior. 

## Assumptions

- `ping` procedure always returns `pong!`.
- `sayHello` procedure expects a string as input and returns a string where `{replace}` in `Hello {replace}!` is replaced by the input.
- `echo` procedure returns the string input value in the result after 2 seconds.
- `getUserAccount` procedure returns `email` and `signedInDate` for an authenticated user.
- `getVendorInfo` procedure produces an error
- the lowest amount of time a procedure takes to run is 10 milliseconds

## Successful responses

### Simplest server ping
Request:
```json
{
  "protocol": "v1",
  "api": "v1"
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1"
}
```

### Executing a `ping` procedure
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "ping1",
      "name": "ping"
    },
    {
      "id": "ping2",
      "name": "ping"
    }
  ]
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "ping1": {
      "result": "pong!"
    },
    "ping2": {
      "result": "pong!"
    }
  }
}
```

### Executing a procedure that expects an input value
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "sayHello",
      "name": "sayHello",
      "input": "World"
    }
  ]
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "sayHello": {
      "result": "Hello World!"
    }
  }
}
```

### Executing multiple procedures in a sequential order
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "sayHello",
      "name": "sayHello",
      "input": "World"
    },
    {
      "id": "ping",
      "name": "ping"
    },
    {
      "id": "echo",
      "name": "echo",
      "input": "Is anybody there?"
    }
  ],
  "options": {
    "execution": {
      "strategy": "sequential"
    }
  }
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "sayHello": {
      "result": "Hello World!"
    },
    "ping": {
      "result": "pong!"
    },
    "echo": {
      "result": "Is anybody there?"
    }
  }
}
```

### Executing procedures in parallel and prompting the server to return processing details
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "echo",
      "name": "echo",
      "input": "Is anybody there?"
    },
    {
      "id": "ping",
      "name": "ping"
    },
    {
      "id": "sayHello",
      "name": "sayHello",
      "input": "World"
    }
  ],
  "options": {
    "request_id": "request_1",
    "execution": {
      "strategy": "parallel",
      "procedure_timeout": 5
    },
    "return": [
      "request_id",
      "request_execution_time",
      "procedures_execution_details"
    ]
  } 
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "ping": {
      "result": "pong!"
    },
    "sayHello": {
      "result": "Hello World!"
    },
    "echo": {
      "result": "Is anybody there?"
    }
  },
  "details": {
    "request_id": "request_1",
    "execution_time": 2005,
    "procedures_execution": {
      "ping": {
        "id": "ping",
        "procedure": "ping",
        "order": 1,
        "timed_out": false,
        "execution_time": 10
      },
      "sayHello": {
        "id": "sayHello",
        "procedure": "sayHello",
        "order": 2,
        "timed_out": false,
        "execution_time": 10
      },
      "echo": {
        "id": "echo",
        "procedure": "echo",
        "order": 3,
        "timed_out": false,
        "execution_time": 2005
      }
    }
  }
}
```

### Authenticating a request
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "getUserAccount",
      "name": "getUserAccount"
    }
  ],
  "options": {
    "authentication": {
      "scheme": "access_token",
      "token": "jwt_token_value",
      "token_type": "jwt"
    }
  } 
}
```
Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "getUserAccount": {
      "email": "user@email.com",
      "signedInDate": "3000-01-01T10:10:10Z"
    }
  }
}
```

## Failed requests

### Incompatible request input
Request:
```json
{
  "version": "v1",
  "functions": [
    {
      "id": "getUserAccount",
      "name": "getUserAccount"
    }
  ] 
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "error": {
    "code": "SERVER:INCOMPATIBLE_REQUEST_CONTENT",
    "message": "Request content is incompatible with the protocol schema."
  }
}
```

### An unhandled error occurred during the request processing
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "getVendorInfo",
      "name": "getVendorInfo"
    }
  ] 
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "error": {
    "code": "SERVER:UNHANDLED_ERROR",
    "message": "Unhandled error."
  }
}
```

### An localized error occurs while executing a procedure
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "getVendorInfo",
      "name": "getVendorInfo"
    }
  ] 
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "getVendorInfo": {
      "error": {
        "code": "SERVER:UNHANDLED_ERROR",
        "message": "Unhandled error."
      }
    }
  }
}
```

### Calling a non existing procedure
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "nonExistingProcedure",
      "name": "nonExistingProcedure"
    }
  ] 
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "nonExistingProcedure": {
      "error": {
        "code": "PROCEDURE:NOT_FOUND",
        "message": "Procedure not found."
      }
    }
  }
}
```

### Incompatible procedure input
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "sayHello",
      "name": "sayHello",
      "input": { "message":  "World" }
    }
  ] 
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "sayHello": {
      "error": {
        "code": "PROCEDURE:INCOMPATIBLE_INPUT",
        "message": "Incompatible input content."
      }
    }
  }
}
```

### Procedure timed out
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "echo",
      "name": "echo",
      "input": "Is anybody there?"
    }
  ],
  "options": {
    "execution": {
      "procedure_timeout": 1
    }
  }
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "echo": {
      "error": {
        "code": "PROCEDURE:TIMEOUT",
        "message": "Procedure timed out."
      }
    }
  }  
}
```

### Partial request failure where 1 out of 3 procedures failed
Request:

```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "ping",
      "name": "ping"
    },
    {
      "id": "sayHello",
      "name": "sayHello",
      "input": { "message": "World" }
    },
    {
      "id": "echo",
      "name": "echo",
      "input": "Is anybody there?"
    }
  ]
}
```

Response:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "ping": {
      "result": "pong!"
    },
    "sayHello": {
      "error": {
        "code": "PROCEDURE:INCOMPATIBLE_INPUT",
        "message": "Incompatible input content."
      }
    },
    "echo": {
      "result": "Is anybody there?"
    }
  }
}
```

### Procedure execution not authorized
Request:
```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": [
    {
      "id": "getUserAccount",
      "name": "getUserAccount"
    }
  ],
  "options": {
    "authentication": {
      "scheme": "access_token",
      "token": "back_jwt_token_value",
      "token_type": "jwt"
    }
  } 
}
```

Response:

```json
{
  "protocol": "v1",
  "api": "v1",
  "procedures": {
    "getUserAccount": {
      "error": {
        "code": "SERVER:NOT_AUTHENTICATED",
        "message": "Not authenticated."
      }
    }
  }
}
```
