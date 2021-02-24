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
