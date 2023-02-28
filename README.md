# Security validation of kubeapi

kubernetes REST API is mostly based on gorilla mux (AFAIK) so it should implement standard security practices for REST API by default.
But let's check if it is true in all cases.


# Setup

After login to the kube API.

```bash
$ export API_PORT=8387
$ export NS=curl-debug
$ oc new-project ${NS}
$ oc proxy --port=${API_PORT} &
Starting to serve on 127.0.0.1:8387
```


Then check API is accessible:

```bash
$ curl http://localhost:8387/api/
$ curl  -i http://localhost:8387/api/
HTTP/1.1 200 OK
Cache-Control: no-cache, private
Content-Length: 185
Content-Type: application/json
Date: Wed, 24 Feb 2021 09:28:35 GMT
X-Kubernetes-Pf-Flowschema-Uid: b913929c-49cf-4233-8b79-23e00648cc2f
X-Kubernetes-Pf-Prioritylevel-Uid: 65145b44-d874-41ce-a614-ca5c2df0f0b5

{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "192.168.70.11:6443"
    }
  ]
}
```

# Validation tests

## 


## Accept header and content type in response

* Send secure response content types: You should not simply copy the "Accept" header into the content type header of the response.
* Send secure response content types: It is common for REST services to allow multiple response types (for example, "application / xml" or "application / json", and the client specifies the preferred order of response types by the heading "Accept" in the request.

Send json request but accept yaml:

```bash
$ curl --request POST \
  --url http://localhost:${API_PORT}/api/v1/namespaces/${NS}/pods \
  --header 'accept: application/yaml' \
  --header 'content-type: application/json' \
  --data '{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "api-validation-pod"
  },
  "spec": {
    "containers": [
      {
        "name": "api-validation",
        "image": "quay.io/xymox/vue-todos-ubi8:main",
        "command": [
          "/bin/sh",
          "-c",
          "sleep infinity"
        ]
      }
    ]
  }
}'


## Non whitelisted requests

* Non-whitelisted requests must be rejected with HTTP response code 405 Method not allowed

```bash
$ curl --request PUT \
  --url http://localhost:${API_PORT}/api/v1/namespaces/${NS}/pods \
  --header 'accept: application/yaml' \
  --header 'content-type: application/json' \
  --data '{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "api-validation-pod"
  },
  "spec": {
    "containers": [
      {
        "name": "api-validation",
        "image": "quay.io/xymox/vue-todos-ubi8:main",
        "command": [
          "/bin/sh",
          "-c",
          "sleep infinity"
        ]
      }
    ]
  }
}'

apiVersion: v1
code: 405
details: {}
kind: Status
message: the server does not allow this method on the requested resource
metadata: {}
reason: MethodNotAllowed
status: Failure
```

## Content length forgering


```bash
$ curl --request PUT \
  --url http://localhost:${API_PORT}/api/v1/namespaces/${NS}/pods \
  --header 'accept: application/yaml' \
  --header 'content-type: application/json' \
  --header 'content-length: 300' \
  --data '{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "api-validation-pod"
  },
  "spec": {
    "containers": [
      {
        "name": "api-validation",
        "image": "quay.io/xymox/vue-todos-ubi8:main",
        "command": [
          "/bin/sh",
          "-c",
          "sleep infinity"
        ]
      }
    ]
  }
}'
apiVersion: v1
code: 400
kind: Status
message: |-
  the object provided is unrecognized (must be of type Pod): couldn't get version/kind; json parse error: unexpected end of JSON input ({
    "kind": "Pod",
    "apiVersi ...)
metadata: {}
reason: BadRequest
status: Failure
```

```bash
$ curl --request PUT \
  --url http://localhost:${API_PORT}/api/v1/namespaces/${NS}/pods \
  --header 'accept: application/yaml' \
  --header 'content-type: application/json' \
  --header 'content-length: 400' \
  --data '{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "api-validation-pod"
  },
  "spec": {
    "containers": [
      {
        "name": "api-validation",
        "image": "quay.io/xymox/vue-todos-ubi8:main",
        "command": [
          "/bin/sh",
          "-c",
          "sleep infinity"
        ]
      }
    ]
  }
}'
# hang up (timeout after a while).
```

# Data validation - Regular expression


```bash
$ curl --request POST \
  --url http://localhost:${API_PORT}/api/v1/namespaces/${NS}/pods \
  --header 'accept: application/yaml' \
  --header 'content-type: application/json' \
  --data '{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-"
  },
  "spec": {
    "containers": [
      {
        "name": "api-validation",
        "image": "quay.io/xymox/vue-todos-ubi8:main",
        "command": [
          "/bin/sh",
          "-c",
          "sleep infinity"
        ]
      }
    ]
  }
}'
apiVersion: v1
code: 422
details:
  causes:
  - field: metadata.name
    message: 'Invalid value: "api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-":
      must be no more than 253 characters'
    reason: FieldValueInvalid
  - field: metadata.name
    message: 'Invalid value: "api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-":
      a DNS-1123 subdomain must consist of lower case alphanumeric characters, ''-''
      or ''.'', and must start and end with an alphanumeric character (e.g. ''example.com'',
      regex used for validation is ''[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*'')'
    reason: FieldValueInvalid
  kind: Pod
  name: api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-
kind: Status
message: 'Pod "api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-"
  is invalid: [metadata.name: Invalid value: "api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-":
  must be no more than 253 characters, metadata.name: Invalid value: "api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-api-validation-pod-":
  a DNS-1123 subdomain must consist of lower case alphanumeric characters, ''-'' or
  ''.'', and must start and end with an alphanumeric character (e.g. ''example.com'',
  regex used for validation is ''[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*'')]'
metadata: {}
reason: Invalid
status: Failure
```


## Try to forge Content-type header

```bash
# Working request

$ curl -k --verbose -H 'Content-Type: application/json' --request GET http://127.0.0.1:8387/api/
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8387 (#0)
> GET /api/ HTTP/1.1
> Host: 127.0.0.1:8387
> User-Agent: curl/7.61.1
> Accept: */*
> Content-Type: application/json
>
< HTTP/1.1 200 OK
< Audit-Id: f5170c35-dbc1-4aa0-84dd-71f56374f532
< Cache-Control: no-cache, private
< Content-Length: 184
< Content-Type: application/json
< Date: Tue, 28 Feb 2023 09:32:44 GMT
< X-Kubernetes-Pf-Flowschema-Uid: 44c2e9c8-ffa5-4e7e-bd5e-bfdd68492a38
< X-Kubernetes-Pf-Prioritylevel-Uid: f25f62c0-a03c-4895-b06a-17889e49a681
<
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "172.16.1.214:6443"
    }
  ]
* Connection #0 to host 127.0.0.1 left intact


# Forged request

$ curl -k --verbose -H 'Content-Type: application/exe' --request GET http://127.0.0.1:8387/api/
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8387 (#0)
> GET /api/ HTTP/1.1
> Host: 127.0.0.1:8387
> User-Agent: curl/7.61.1
> Accept: */*
> Content-Type: application/exe
>
< HTTP/1.1 406 Not Acceptable
< Audit-Id: ba857783-ffd1-4196-8f85-d88962590894
< Cache-Control: no-cache, private
< Content-Length: 182
< Content-Type: application/json
< Date: Tue, 28 Feb 2023 09:33:07 GMT
< X-Kubernetes-Pf-Flowschema-Uid: 44c2e9c8-ffa5-4e7e-bd5e-bfdd68492a38
< X-Kubernetes-Pf-Prioritylevel-Uid: f25f62c0-a03c-4895-b06a-17889e49a681
<
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "406: Not Acceptable",
  "reason": "NotAcceptable",
  "details": {},
  "code": 406
* Connection #0 to host 127.0.0.1 left intact

```
