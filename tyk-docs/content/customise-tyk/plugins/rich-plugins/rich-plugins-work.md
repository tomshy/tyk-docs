---
date: 2017-03-24T13:04:21Z
title: How do rich plugins work ?
menu:
  main:
    parent: "Rich Plugins"
weight: 2
---

### ID Extractor & Auth Plugins

The ID Extractor is a caching mechanism that's used in combination with Tyk Plugins. It can be used specifically with plugins that implement custom authentication mechanisms. The ID Extractor works for all rich plugins: gRPC-based plugins, Python and Lua.

See [ID Extractor](https://tyk.io/docs/customise-tyk/plugins/rich-plugins/id-extractor/) for more details.

### Interoperability

This feature implements an in-process message passing mechanism, based on [Protocol Buffers][1], any supported languages should provide a function to receive, unmarshal and process this kind of messages.

The main interoperability task is achieved by using [cgo][2] as a bridge between a supported language -like Python- and the Go codebase.

Your C bridge function must accept and return a `CoProcessMessage` data structure like the one described in [`api.h`][3], where `p_data` is a pointer to the serialized data and `length` indicates the length of it.

```{.copyWrapper}
struct CoProcessMessage {
  void* p_data;
  int length;
};
```

The unpacked data will hold the actual `CoProcessObject` data structure.
`HookType` - the hook type (see below) 
`Request`  - the HTTP request
`Session`  - the [Tyk session object](https://tyk.io/docs/tyk-rest-api/token-session-object-details/).
`Metadata`  - the meta data from the session data above (key/value string map).
`Spec`     - the API specification data. Currently organization ID, API ID and config_data.

```{.copyWrapper}
type CoProcessObject struct {
  HookType string
  Request  CoProcessMiniRequestObject
  Session  SessionState
  Metadata map[string]string
  Spec     map[string]string
}
```

### Coprocess Dispatcher

`Coprocess.Dispatcher` describes a very simple interface for implementing the dispatcher logic, the required methods are: `Dispatch`, `DispatchEvent` and `Reload`.

`Dispatch` accepts a pointer to a `struct CoProcessObject` (as described above) and must return an object of the same type. This method will be called for every configured hook, on every request. Traditionally this method will perform a single function call on the target language side (like `Python_DispatchHook` in `coprocess_python`), and the corresponding logic will be handled from there (mostly because different languages have different ways of loading, referencing or calling middlewares).

`DispatchEvent` provides a way of dispatching Tyk events to a target language. This method doesn't return any variables but does receive a JSON-encoded object containing the event data. For extensibility purposes, this method doesn't use Protocol Buffers, the input is a `[]byte`, the target language will take this (as a `char`) and perform the JSON decoding operation.

`Reload` is called when triggering a hot reload, this method could be useful for reloading scripts or modules in the target language.

### Coprocess Dispatcher - Hooks

This component is in charge of dispatching your HTTP requests to the custom middlewares, in the right order. The dispatcher follows the standard middleware chain logic and provides a simple mechanism for "hooking" your custom middleware behavior, the supported hooks are:

*   **Pre**: gets executed before the request is sent to your upstream target and before any authentication information is extracted from the header or parameter list of the request. When enabled, this applies to both keyless and protected APIs.
*   **PostKeyAuth**: gets executed right after the authentication process.
*   **CustomAuthCheck**: gets executed as a custom authentication middleware, instead of the standard ones provided by Tyk. Use this to provide your own authentication mechanism.
*   **Post**: gets executed after the authentication, validation, throttling, and quota-limiting middleware has been executed, just before the request is proxied upstream. Use this to post-process a request before sending it to your upstream API. This is only called when using protected APIs. If you want to call a hook after the authentication but before the validation, throttling and other middleware, see **PostKeyAuth**.


### Coprocess Gateway API

[`coprocess_api.go`][4] provides a bridge between the gateway API and C, any function that needs to be exported should have the `export` keyword:

```{.copyWrapper}
//export TykTriggerEvent
func TykTriggerEvent( CEventName *C.char, CPayload *C.char ) {
  eventName := C.GoString(CEventName)
  payload := C.GoString(CPayload)

  FireSystemEvent(tykcommon.TykEvent(eventName), EventMetaDefault{
    Message: payload,
  })
}
```

You should also expect a header file declaration of this function in [`api.h`][3], like this:

```{.copyWrapper}
#ifndef TYK_COPROCESS_API
#define TYK_COPROCESS_API
extern void TykTriggerEvent(char* event_name, char* payload);
#endif
```

The language binding will include this header file (or declare the function inline) and perform the necessary steps to call it with the appropriate arguments (like an FFI mechanism could do). As a reference, this is how this could be achieved if you're building a [Cython][5] module:

```{.copyWrapper}
cdef extern:
  void TykTriggerEvent(char* event_name, char* payload);

def call():
  event_name = 'my event'.encode('utf-8')
  payload = 'my payload'.encode('utf-8')
  TykTriggerEvent( event_name, payload )
```

### Basic usage

The intended way of using a Coprocess middleware is to specify it as part of an API Definition:

```{.json}
"custom_middleware": {
  "pre": [
    {
      "name": "MyPreMiddleware",
      "require_session": false
    },
    {
      "name": "AnotherPreMiddleware",
      "require_session": false
    }
  ],
  "post": [
    {
      "name": "MyPostMiddleware",
      "require_session": false
    }
  ],
  "post_key_auth": [
    {
      "name": "MyPostKeyAuthMiddleware",
      "require_session": true
    }
  ],
  "auth_check": {
    "name": "MyAuthCheck"
  },
  "driver": "python"
}
```

> **Note**: All hook types support chaining except the custom auth check (`auth_check`).

### ReturnOverides
From version 1.3.6, you can now  override response code, headers and body using ReturnOverrides.

From version 1.3.8, you can now add custom error messages

#### Python Example

```{.copyWrapper}
from tyk.decorators import *

@Hook
def MyCustomMiddleware(request, session, spec):
    print("my_middleware: MyCustomMiddleware")
    request.object.return_overrides.headers['content-type'] = 'application/json'
    request.object.return_overrides.response_code = 200
    request.object.return_overrides.response_error = "{\"key\": \"value\"}\n"
    return request, session
```


 [1]: https://developers.google.com/protocol-buffers/
 [2]: https://golang.org/cmd/cgo/
 [3]: https://github.com/TykTechnologies/tyk/blob/master/coprocess/api.h
 [4]: https://github.com/TykTechnologies/tyk/blob/master/coprocess.go
 [5]: http://cython.org/
