# Micronets Infrastructure Components

This repository is for managing tools, data, and other cross-component Micronets elements. 

## 1. API Keys

### 1.1 Generating the shared root certificate used for micronet API component communication:

This will produce the root certificate and key for validating/generating
leaf certificates used by certain micronets API:

```
bin/gen-root-cert --cert-basename lib/micronets-api-root \
    --subject-org-name "Micronets API Root Cert" \
    --expiration-in-days 3650
```

This root cert is be the basis for trust for all the clients
communicating via select Micronets APIs. APIs will only 
trust peers that present a cert signed by this root cert. 
The API server must have client cert verification enabled/required 
and `lib/micronets-api-root.cert.pem` must be added to the API 
endpoint's list of trusted CAs (and should really be the only
CA enabled).

The `micronets-ws-root.key.pem` file generated by this script should
only be retained for the purposes of generating new leaf certs for the
websocket peers. It should not be deployed with any software 
components.

### 1.1 Generating API client certificates:

```
bin/gen-leaf-cert --cert-basename lib/micronets-api-client \
    --subject-org-name "Micronets API Client Cert" \
    --expiration-in-days 3650 \
    --ca-certfile lib/micronets-api-root.cert.pem \
    --ca-keyfile lib/micronets-api-root.key.pem

cat lib/micronets-api-client.cert.pem lib/micronets-api-client.key.pem > lib/micronets-api-client.pkeycert.pem
```

The `lib/micronets-api-client.pkeycert.pem` file must be deployed with any
micronet software component that needs to access a cert-controlleed micronets API.

## 2. The websocket proxy

The websocket proxy allows 2 websocket clients to connect to each other using
outbound connections to a common URI.

### 2.1 Setting up proxy authorization using certificates

#### 2.1.1 Generating the shared root certificate used for websocket communication:

This will produce the root certificate and key for validating/generating
leaf certificates used by peers of the websocket proxy:

```
bin/gen-root-cert --cert-basename lib/micronets-ws-root \
    --subject-org-name "Micronets Websocket Root Cert" \
    --expiration-in-days 3650
```

The shared root cert will be the basis for trust for all the entities
communicating via the websocket proxy. The websocket proxy will only 
trust peers that present a cert (and can accept a challenge from) the
proxy.

The `micronets-ws-root.key.pem` file generated by this script should
only be retained for the purposes of generating new leaf certs for the
websocket peers.

#### 2.1.2 To generate the cert to be used for the Websocket Proxy:

```
bin/gen-leaf-cert --cert-basename lib/micronets-ws-proxy \
    --subject-org-name "Micronets Websocket Proxy Cert" \
    --expiration-in-days 3650 \
    --ca-certfile lib/micronets-ws-root.cert.pem \
    --ca-keyfile lib/micronets-ws-root.key.pem

cat lib/micronets-ws-proxy.cert.pem lib/micronets-ws-proxy.key.pem > lib/micronets-ws-proxy.pkeycert.pem
```

The `lib/micronets-ws-proxy.pkeycert.pem` file must be deployed with the 
Micronets websocket proxy and well-protected. The `lib/micronets-ws-root.cert.pem`
must be added to the Proxy's list of trusted CAs. (and should really be the only
CA enabled for the proxy)

#### 2.1.3 Generating the cert to be used for the Micronets Manager:

```
bin/gen-leaf-cert --cert-basename lib/micronets-manager \
    --subject-org-name "Micronets Manager Websocket Client Cert" \
    --expiration-in-days 3650 \
    --ca-certfile lib/micronets-ws-root.cert.pem \
    --ca-keyfile lib/micronets-ws-root.key.pem

cat lib/micronets-manager.cert.pem lib/micronets-manager.key.pem > lib/micronets-manager.pkeycert.pem
```

The `lib/micronets-manager.pkeycert.pem` file must be deployed with the 
Micronets Manager to connect to the websocket proxy and `lib/micronets-ws-root.cert.pem` 
must be added to the Micronet's Manager CA list.

#### 2.1.4 To generate the cert to be used for the Micronets Gateway Service:

```
bin/gen-leaf-cert --cert-basename lib/micronets-gateway-service \
    --subject-org-name "Micronets Gateway Service Websocket Client Cert" \
    --expiration-in-days 3650 \
    --ca-certfile lib/micronets-ws-root.cert.pem \
    --ca-keyfile lib/micronets-ws-root.key.pem

cat lib/micronets-dhcp-manager.cert.pem lib/micronets-dhcp-manager.key.pem > lib/micronets-dhcp-manager.pkeycert.pem
```

The `lib/micronets-manager.pkeycert.pem` file must be deployed with the 
Micronets Manager to connect to the websocket proxy and `lib/micronets-ws-root.cert.pem` 
must be added to the Micronet's Manager CA list.

#### 2.1.5 To generate the cert to be used by the test client:

```
bin/gen-leaf-cert --cert-basename lib/micronets-ws-test-client \
    --subject-org-name "Micronets Websocket Test Client Cert" \
    --expiration-in-days 3650 \
    --ca-certfile lib/micronets-ws-root.cert.pem \
    --ca-keyfile lib/micronets-ws-root.key.pem

cat lib/micronets-ws-test-client.cert.pem lib/micronets-ws-test-client.key.pem > lib/micronets-ws-test-client.pkeycert.pem
```

### 2.2 Starting the websocket proxy server

#### 2.2.1 Setting the proxy parameters

For now, the websocket proxy's parameters are stored in the bin/websocket-test-client.py source.

```
proxy_bind_address = "localhost"
proxy_port = 5050
proxy_service_prefix = "/micronets/v1/ws-proxy/"
proxy_cert_path = bin_path.parent.joinpath ('lib/micronets-ws-proxy.pkeycert.pem')
root_cert_path = bin_path.parent.joinpath ('lib/micronets-ws-root.cert.pem')
```

#### 2.2.2 Setting up the websocket proxy environment

The virtualenv with the library dependancies is setup by performing the following steps:

From the python-infrastructure directory:
```
virtualenv --clear -p $(which python3.6) $PWD/virtualenv
source virtualenv/bin/activate
pip install -r requirements.txt 
```

And the systemctl service is installed with the following steps:
```
sudo systemctl enable $PWD/micronets-ws-proxy.service
sudo systemctl daemon-reload
```

#### 2.2.3 Starting the websocket proxy server

To start the server on the linode ws proxy server:

```
sudo systemctl start micronets-ws-proxy.service
```

To restart the server on the linode ws proxy server:

```
sudo systemctl restart micronets-ws-proxy.service
```

#### 2.2.4 Checking the log/status of the proxy server

To monitor the log:

```
sudo tail -f /var/log/syslog
```

#### 2.2.5

To stop the proxy server:

```
sudo systemctl stop micronets-ws-proxy.service
````

### 2.3 Connecting an instance of the micronets gateway service to the websocket proxy

The steps below are for testing using the mock DHCP adapter (a Micronet’s DHCP server that doesn’t actually write DHCP lease reservations or restart a real DHCP server). Setting the micronets DHCP server to create IP reservations for the dnsmasq DHCP server simply requires changing the DnsMasqTestingConfig instead of the MockTestingConfig.

#### 2.3.1 Installing the micronets dhcp server

```
mkdir -p ~/projects/micronets
cd ~/projects/micronets
git clone git@github.com:cablelabs/micronets-gw.git
cd micronets-gw
mkvirtualenv -r micronets-gw-service/requirements.txt -a $PWD/micronets-gw-service -p $(which python3) micronets-gw-service
workon -c micronets-gw-service
```

#### 2.3.2 Configuring the service to connect to the websocket proxy

```
workon micronets-gw-service
nano -w config.py
```

#### 2.3.3 Configuring the BaseConfig to connect to a gateway-specific URI on the websocket proxy and use a TLS cert that will allow it to connect

```
class BaseConfig:
    WEBSOCKET_SERVER_PATH = '/micronets/v1/ws-proxy/micronets-dhcp-0001'
    WEBSOCKET_TLS_CERTKEY_FILE = pathlib.Path (__file__).parent.joinpath ('lib/micronets-dhcp-manager.pkeycert.pem')
    WEBSOCKET_TLS_CA_CERT_FILE = pathlib.Path (__file__).parent.joinpath ('lib/micronets-ws-root.cert.pem')
```

#### 2.3.4 Configuring the MockTestingConfig to connect to the websocket proxy running on the linode instance

```
class MockTestingConfig (BaseMockConfig):
    WEBSOCKET_SERVER_ADDRESS = "74.207.229.106"
    WEBSOCKET_SERVER_PORT = 5050
    USE_MOCK_DHCP_CONFIG = True
    DEBUG = True
```

#### 2.3.5 Starting the DHCP server with the mock config

```
python runner.py --config config.MockTestingConfig 
```

The Micronets gateway service will still listen on the configured LISTEN_HOSY/PORT as it normally does. But it also will connect to the designated micronet’s websocket proxy - so long as the proxy server is running and the client cert is accept (and the TLS connection/challenge is successful).

#### 2.3.6 Checking/following the micronets gateway service log

```
tail -f micronets-gw.log
```

A successful connection to the websocket proxy will look like this:

```
micronets-gw-service: INFO WSConnector: init_connect opening wss://74.207.229.106:5050/micronets/v1/ws-proxy/micronets-dhcp-0001...
websockets.protocol: DEBUG client - state = CONNECTING
websockets.protocol: DEBUG client - event = connection_made(<asyncio.sslproto._SSLProtocolTransport object at 0x1103cb710>)
websockets.protocol: DEBUG client - state = OPEN
micronets-gw-service: INFO WSConnector: init_connect opened wss://74.207.229.106:5050/micronets/v1/ws-proxy/micronets-dhcp-0001.
mmicronets-gw-service: INFO WSConnector Sending HELLO message...
```

### 2.4 Testing the micronets websocket proxy

#### 2.4.1 Downloading the micronets infrastructure source (containing the test client)

```
mkdir -p ~/projects/micronets
cd ~/projects/micronets
git clone git@github.com:cablelabs/micronets-infrastructure.git
cd micronets-infrastructure
mkvirtualenv -r requirements.txt -a $PWD -p $(which python3) micronets-websocket-proxy
```

#### 2.4.2 Connecting the websocket test client to the proxy (using the same URI as a connected micronets gateway)

The websocket test client takes all its parameters via arguments. Use "-h" to see the options. A typical test session would look like this:

```
workon micronets-websocket-proxy
bin/websocket-test-client.py --client-cert lib/micronets-manager.pkeycert.pem --ca-cert lib/micronets-ws-root.cert.pem  wss://74.207.229.106:5050/micronets/v1/ws-proxy/micronets-dhcp-0001
```

The test client should startup with log messages similar to:

```
Loading test client certificate from lib/micronets-manager.pkeycert.pem
Loading CA certificate from lib/micronets-ws-root.cert.pem
ws-test-client: Starting stdin reader...
ws-test-client: Opening websocket to wss://74.207.229.106:5050/micronets/v1/ws-proxy/micronets-dhcp-0001...
ws-test-client: Connected to wss://74.207.229.106:5050/micronets/v1/ws-proxy/micronets-dhcp-0001.
ws-test-client: Sending HELLO message...
ws-test-client: > sending hello message:  {"message": {"messageId": 0, "messageType": "CONN:HELLO", "requiresResponse": false, "peerClass": "micronets-ws-test-client", "peerId": "12345678"}}
ws-test-client: Waiting for HELLO message...
ws-test-client: process_hello_messages: Received message: {'message': {'messageId': 0, 'messageType': 'CONN:HELLO', 'peerClass': 'micronets-dhcp-service', 'peerId': '12345678', 'requiresResponse': False}}
ws-test-client: process_hello_messages: Received HELLO message
ws-test-client: HELLO handshake complete.
MyHTTPServerThread.__init__(): state: ready
ws-test-client: Starting event loop...
ws-test-client: receive: starting...
MyHTTPServerThread: Starting HTTP server on localhost port 5001...
```

#### 2.4.3 Using the websocket test client's built-in HTTP proxy to test the websocket

To use the websocket-test-client’s built-in server to test the proxying of an HTTP REST request and response to the Micronet’s DHCP server running on the gateway, requests can be sent to the port specified using the "--http-proxy-port" argument (or port "5001" if not specified).

e.g.

```
curl -H "Content-Type: application/json" http://localhost:5001/micronets/v1/dhcp/subnets
```

When sending a request via the built-in http test server, the websocket-test-client program will encapsulate the HTTP request sent by curl into a “REST:REQUEST” websocket message and send it to the proxy - which will send it to the gateway. After processing the request, the micronets-dhcp server will send a response, encapsulate it in a “REST:RESPONSE” websocket message, send it to the proxy - which will send it to the websocket-test-client program. The websocket-test-client program will then take the contents out of the websocket message and send the response to curl.

See below for details on the micronets websocket proxy message format.

## 3. Websocket message format

### 3.1 Base message definition

All messages exchanged via the websocket channel must have these fields:

```
{
   “message”: {
      “messageId”: <client-supplied session-unique string>,
      “messageType”: <string identifying the message type>,
      “requiresResponse”: <boolean>
      “inResponseTo”: <id string of the originating message> (optional)
   }
}
```

### 3.2 HELLO message definition

```
{
    "message": {
        "messageId": 0,
        "messageType": "CONN:HELLO",
        "peerClass": <string identifying the type of peer connecting to the websocket>,
        "peerId": <string uniquely identifying the peer in the peer class>,
        "requiresResponse": false
    }
}
```

Example:

```
{
    "message": {
        "messageId": 0,
        "messageType": "CONN:HELLO",
        "requiresResponse": false,
        "peerClass": "micronets-ws-test-client",
        "peerId": "12345678"
    }
}
```

### 3.3 REST Request definition

This defines a REST Request message:

```
“message”: {
   “messageType”: “REST:REQUEST”,
   “requiresResponse”: true,
   “method”: <HEAD|GET|POST|PUT|DELETE|…>,
   “path”: <URI path>,
   “queryStrings”: [{“name”: <name string>, “value”: <val string>}, …],
   “headers”: [{“name”: <name string>, “value”: <val string>}, …],
   “dataFormat”: <mime data format for the messageBody>
   “messageBody”: <either a string encoded according to the mime type, base64 string if dataFormat is “application/octet-stream”, or JSON object if dataFormat is “application/json”>
```

Note that Content-Length, Content-Type, and Content-Encoding should not be communicated via the "headers" element as they are conveyed via the dataFormat and messageBody elements. If the request is handled by a HTTP processing system, these header elements may need to be derived from dataFormat and messageBody.

Example GET request:

```
{
  "message": {
    "messageId": 3,
    "messageType": "REST:REQUEST",
    "requiresResponse": true,
    "method": "GET",
    "path": "/micronets/v1/dhcp/subnets",
    "headers": [
      {
        "name": "Host",
        "value": "localhost:5001"
      },
      {
        "name": "User-Agent",
        "value": "curl/7.54.0"
      },
      {
        "name": "Accept",
        "value": "*/*"
      }
    ]
  }
}
```

Example POST request:

```
{
  "message": {
    "messageId": 1,
    "messageType": "REST:REQUEST",
    "requiresResponse": true,
    "method": "POST",
    "path": "/micronets/v1/dhcp/subnets",
    "headers": [
      {
        "name": "Host",
        "value": "localhost:5001"
      },
      {
        "name": "User-Agent",
        "value": "curl/7.54.0"
      },
      {
        "name": "Accept",
        "value": "*/*"
      }
    ],
    "dataFormat": "application/json",
    "messageBody": {
      "subnetId": "mocksubnet007",
      "ipv4Network": {
        "network": "192.168.1.0",
        "mask": "255.255.255.0",
        "gateway": "192.168.1.1"
      },
      "nameservers": [
        "1.2.3.4",
        "1.2.3.5"
      ]
    }
  }
}
```

Example PUT request:

```
{
    "message": {
        "messageId": 3,
        "messageType": "REST:REQUEST",
        "requiresResponse": true,
        "method": "PUT",
        "path": "/micronets/v1/dhcp/subnets/mocksubnet007",
        "dataFormat": "application/json",
        "headers": [
           {"name": "Host", "value": "localhost:5001"},
           {"name": "User-Agent", "value": "curl/7.54.0"},
           {"name": "Accept", "value": "*/*"}
        ],
        "messageBody": {
            "ipv4Network": {
                "gateway": "192.168.1.3"
            }
        }
    }
}
```

### 3.4 REST Response definition

This defines a REST Response message:

```
{
    “message”: {
        “messageType”: “REST:RESPONSE”,
        "inResponseTo": <integer message ID of the REST:REQUEST that generated the response>
        “requiresResponse”: false,
        “statusCode”: <HTTP integer status code>,
        “reasonPhrase”: <HTTP reason phrase string>,
        “headers”: [{“name”: <name string>, “value”: <val string>}, ],
        “dataFormat”: <mime data format for the messageBody>,
        “messageBody”: <either a string encoded according to the dataFormat, base64 string if dataFormat is         “application/octet-stream”, or JSON object if dataFormat is “application/json”>
 “application/octet-stream”, or JSON object if dataFormat is “application/json”>
    }
}
```

Note that Content-Length, Content-Type, and Content-Encoding should not be communicated via the "headers" element as they are conveyed via the dataFormat and messageBody elements. If the request is handled by a HTTP processing system, these header elements may need to be derived from dataFormat and messageBody.

Example GET response:
```
{
    "message": { 
        "messageId": 2,
        "inResponseTo": 3,
        "messageType": "REST:RESPONSE",
        "reasonPhrase": null,
        "requiresResponse": false, 
        "statusCode": 200,
        "dataFormat": "application/json", 
        "messageBody": {
            "subnets": [
                {
                    "ipv4Network": {
                        "gateway": "192.168.30.2",
                        "mask": "255.255.255.0",
                        "network": "192.168.30.0"
                    }, 
                    "subnetId": "wireless-network-1"
                }, 
                {
                    "ipv4Network": {
                        "gateway": "192.168.40.1",
                        "mask": "255.255.255.0",
                        "network": "192.168.40.0"
                    }, 
                    "subnetId": "wired-network-3"
                },
                {
                    "ipv4Network": {
                        "gateway": "10.40.0.1",
                        "mask": "255.255.255.0",
                        "network": "10.40.0.0"
                    }, 
                    "nameservers": ["10.40.0.1"],
                    "subnetId": "testsubnet001"
                }
            ]
        }
    }
}
```

Example POST response:

```
{
    "message": {
        "messageId": 2,
        "inResponseTo": 1,
        "messageType": "REST:RESPONSE",
        "requiresResponse": false,
        "statusCode": 201
        "dataFormat": "application/json",
        "messageBody": {
            "subnet": {
                "subnetId": "mocksubnet007",
                "ipv4Network": {
                    "gateway": "192.168.1.1",
                    "mask": "255.255.255.0",
                    "network": "192.168.1.0"
                }, 
                "nameservers": ["1.2.3.4", "1.2.3.5"]
            }
        }
    }
}
```

Example PUT response:
```
 {
     "message": {
         "messageId": 2,
         "inResponseTo": 3,
         "messageType": "REST:RESPONSE",
         "requiresResponse": false,
         "statusCode": 200,
         "dataFormat": "application/json",
         "messageBody": {
             "subnet": {
                 "ipv4Network": {
                     "gateway": "192.168.1.3",
                     "mask": "255.255.255.0",
                     "network": "192.168.1.0"
                 },
                 "nameservers": ["1.2.3.4", "1.2.3.5"],
                 "subnetId": "mocksubnet007"
             }
         }
     }
 }
```

Example DELETE response:
```
{
    "message": {
        "messageId": 3,
        "inResponseTo": 5,
        "messageType": "REST:RESPONSE",
        "requiresResponse": false,
        "statusCode": 200
    }
}
```

### 3.5 Event Message definition

```
{
    “message”: {
        “messageType”: “EVENT:<client-supplied event name>”,
        “requiresResponse”: False,
        “dataFormat”: <mime data format for the messageBody>,
        “messageBody”: <either a string encoded according to the mime type, base64 string if dataFormat is “application/octet-stream”, or JSON object if dataFormat is “application/json”>
    }
}
```
