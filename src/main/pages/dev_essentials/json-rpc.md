---
title: "Communication: JSON-RPC"
keywords: framework, plugin, extension, json rpc, communication, events, subscribe
tags: [extensions, dev-docs]
sidebar: user_sidebar
permalink: json-rpc.html
folder: dev_essentials
---

{% include links.html %}

## Overview

In CHE we use [JSON-RPC](http://www.jsonrpc.org/specification) for asynchronous communication between clients and servers. JSON-RPC is asymmetric by nature so in order to be able to  information independently we use duplex approach where both sides (clients and servers) can act both as a JSON-RPC server and as JSON-RPC client. To make it clear, clients (e.g. IDE) can send JSON-RPC requests (acting as JSON-RPC client) and receive requests (acting as JSON-RPC server) and servers (e.g. workspace master or workspace agent) can do the same.


## Send a JSON-RPC request

To send request you can use [`org.eclipse.che.api.core.jsonrpc.commons.RequestTransmitter`](https://github.com/eclipse/che/blob/master/core/che-core-api-core/src/main/java/org/eclipse/che/api/core/jsonrpc/commons/RequestTransmitter.java), the most basic use case (actual for both client and server sides) looks quite simple:

```java
transmitter
         .newRequest()
         .endpointId(endpointId)
         .methodName(method)
         .paramsAsDto(dto)
         .sendAndReceiveResultAsDto(dtoClass)
         .onSuccess((result) -> {})
         .onFailure((jsonRpcError) -> {});
```


**endpointId** - in common sense can be an arbitrary **`String`** identifier that allows to distinguish endpoints. Please note that for server sides these endpoints are statically defined as constants:
`"ws-agent"` for workspace agent and `"ws-master"` for workspace master for example, while for clients it is a dynamic identifier because clients are dynamic by their nature.

**method** - method name corresponds to it's definition in JSON-RPC protocol specification. In Che we use the following naming convention: method name in most cases consists of two parts separated be slash - scope and action. For example `project/create` where `project` is the scope and `create` is the action.

**dto** - value of parameters passed (in our example it is a [DTO](https://en.wikipedia.org/wiki/Data_transfer_object)), please note that though JSON-RPC protocol specification requires parameters either as json objects or arrays, you can use single primitives here as well and they will be automatically converted to a singleton array. 

**dtoClass** - class of a result that will be contained within the response to our request

**result** - result of our request (in our example it is of class **dtoClass**)

**jsonRpcError** - error object that contains information about the error if any happened, its structure corresponds to error defined in JSON-RPC protocol specification

Source code example of using of the transmitter can be found here: https://github.com/eclipse/che/blob/master/wsagent/che-core-api-debug/src/main/java/org/eclipse/che/api/debugger/server/DebuggerJsonRpcMessenger.java#L76

## Receive a JSON-RPC request

To receive a request you can use [`org.eclipse.che.api.core.jsonrpc.commons.RequestHandlerConfigurator`](https://github.com/eclipse/che/blob/master/core/che-core-api-core/src/main/java/org/eclipse/che/api/core/jsonrpc/commons/RequestHandlerConfigurator.java), the use case (actual for both sides):

```java
configurator
         .newConfiguration()
         .methodName(method)
         .paramsAsDto(paramsDtoClass)
         .resultAsDto(resultDtoClass)
         .withFunction((endpointId, paramsDto) -> resultDto);
````

**method** - the name of JSON-RPC method that is to be handled by this configuration

**paramsDtoClass** - class of parameters that are passed in a JSON-RPC request (in our example it is a DTO)

**resultDtoClass** - class of result that is passed in a JSON-RPC response (in our example it is DTO)

**endpointId** - the identifier of an endpoint that sends the request that is handled by this configuration

**paramsDto** - the value of parameters passed

**resultDto** - the value of result passed

Source code example of using of the transmitter can be found here: https://github.com/eclipse/che/blob/master/wsagent/che-core-api-debug/src/main/java/org/eclipse/che/api/debugger/server/DebuggerJsonRpcMessenger.java#L117

## Client endpoint identification 
In order to receive a cross sessional identifier clients (e.g. IDE) can use a [`org.eclipse.che.api.core.websocket.impl.WebsocketIdService`](https://github.com/eclipse/che/blob/master/core/che-core-api-core/src/main/java/org/eclipse/che/api/core/websocket/impl/WebsocketIdService.java), otherwise client identifier will not be persisted between websocket sessions. The most common use case (client specific):

```java
requestTransmitter
        .newRequest()
        .endpointId("ws-master")
        .methodName("websocketIdService/getId")
        .noParams()
        .sendAndReceiveResultAsString()
        .onSuccess(appContext::setApplicationWebsocketId);
```
		
Source code example of using websocket identification service can be found here:
https://github.com/eclipse/che/blob/master/ide/che-core-ide-app/src/main/java/org/eclipse/che/ide/jsonrpc/WsMasterJsonRpcInitializer.java#L134



