Specification: elliRPC
======================
<small>provided by [Ellinaut](https://github.com/Ellinaut) </small>

---

## Introduction

*elliRPC* is a specification for how a client should execute remote procedure calls (RPCs) over HTTP, and how a server
should respond the procedure results to the client.

## Conventions

The keywords “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and
“OPTIONAL” in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### Client Responsibilities

A client in the sense of this specification can be any browser, app or application that could execute remote procedures.

Clients MUST send all requests with a request body with a correct `Content-Type` header. Possible content types differ
between endpoints and are documented for each endpoint.

Clients MUST ignore properties which they don't know.

### Server Responsibilities

A server in the sense of this specification can be any application that executes procedures.

Servers MUST respond to all requests with a correct `Content-Type` header.

Servers MUST respond with HTTP status `400 Bad Request` if a given property has an invalid value. If the property is of
type `id` or `uuid` the server MUST respond with `404 Not Found` to an invalid id.

Servers MUST ignore properties which they don't know.

### Naming Conventions

All names MUST be interpreted case-sensitive. All names SHOULD be written as lowerCamelCase.

Package, procedure, schema and property names should not begin with an underscore, as this is reserved for internal
names in this specification.

Official extensions for *elliRPC* (provided by Ellinaut or in direct consultation) MAY use the underscore followed by
*elli* as naming prefix for their names: `_elli*`.

## Packages and Procedures

A `procedure` is an executable. A client can execute a `procedure` via *elliRPC* and gets a response containing the
execution result for this procedure from the server.

A `package` is a context within the application. *elliRPC* provides procedures grouped by `packages`, which make it
possible to use the same procedure name in multiple contexts.

Every `procedure` MUST be assigned to a `package`. If your application does not use multiple packages, `_app` MAY be
used as default package or for all procedures which shouldn't have a package within your application.

## Schemas

```
{
    "name": "{schemaName}",
    "properties": [
        {
            "name": {propertyName},
            "description": {propertyDescription},
            "type": {
                "context": {propertyContext},
                "type": {propertyType}
            },
            "nullable": true|false,
            "multiple": true|false
        }
    ]
}
```

### Schema Definition

@todo extend

| Property   | Type                 | Description                                                     |
| ---------- | -------------------- | --------------------------------------------------------------- |
| name       | string               | The name of the schema, which is used as type.                  |
| abstract   | boolean              | Indicates that this schema can only be used with other schemas. |
| properties | PropertyDefinition[] | A list of properties for the schema.                            |

### Property Definition

| Property    | Type                   | Description                                                   |
| ----------- | ---------------------- | ------------------------------------------------------------- |
| name        | string                 | The property name used as key in request or response body.    |
| description | string or null         | The property description for documentation.                   |
| type        | PropertyTypeDefinition | The property type.                                            |
| nullable    | boolean                | Indicates if this property can be null or not.                |
| multiple    | boolean                | Indicates if this property contains a list or a single value. |

### Property Type Definition

| Property    | Type           | Description                                                                             |
| ----------- | -------------- | --------------------------------------------------------------------------------------- |
| context     | string or null | The context MUST be `null` for `elliRPC` or an IRI for an external `JSON LD` schema.    |
| type        | string         | The case sensitive name of the build-in type or the used (internal or external) schema. |

### Default Property Types

| Type      | Description                                                                                                |
| --------- | ---------------------------------------------------------------------------------------------------------- |
| id        | -                                                                                                          |
| uuid      | -                                                                                                          |
| string    | -                                                                                                          |
| integer   | -                                                                                                          |
| decimal   | -                                                                                                          |
| boolean   | -                                                                                                          |
| date      | A date string formatted as ISO 8601 date (YYYY-MM-DD).                                                     |
| time      | A time string formatted as ISO 8601 time with timezone (hh:mm:ss±hh:mm).                                   |
| datetime  | A datetime string formatted as ISO 8601 combined date and time with timezone (YYYY-MM-DD\Thh:mm:ss±hh:mm). |
| duration  | A duration string formatted as ISO 8601 duration (P<date>T<time>).                                         |
| wrapper   | A placeholder used in wrapper schemas. The schema it self MUST be abstract.                                |

## Endpoint: Packages

The packages' endpoint MUST respond with all available packages and their procedures.

This endpoint SHOULD be used as leading documentation for all available elliRPC procedures.

This endpoint MAY be used for automatically creation of api clients.

The url path for this endpoint MUST be `/elliRPC/_packages.{contentType}`.

Possible values for `{contentType}` MUST be `html`, `json` and `xml`. Implementations of this specification MAY offer
more content types for this endpoint.

This endpoint MUST be accessible only via http method `GET`.

### Request (JSON)

`GET /elliRPC/_packages.json`

### Response (JSON)

@todo Pagination @todo Sorting @todo Wrapper

```
{
    "packages": {
        "{packageName}": {
            "{procedureName}": {
                "methods": [ "GET", "POST" ],
                "request": {
                    "context": null,
                    "schema": "{schemaName}"
                },
                "response": {
                    "context": null,
                    "schema": "{schemaName}"
                }
            }
        }
    }
}
```

## Endpoint: Status

### Request

`GET /elliRPC/_status.json`

### Response

```
{
    "packages": {
        "{packageName}": {
            "{procedureName}": {
                "status": "{statusName}",
                "description": "{description}"
            }
        }
    }
}
```

## Endpoint: Schema

### Request

`GET /elliRPC/_schema/{schemaName}.json`

### Response

```
{
    "name": "{schemaName}",
    "extends": { // | null
        "context": null|{context},
        "type": "{schemaName}"
    },
    "properties": [
        {
            "name": "{propertyName}",
            "description": "{propertyDescription}",
            "type": {
                "context": null|{context},
                "type": "{propertyType}"
            },
            "nullable": true|false,
            "multiple": true|false
        }
    ]
}
```

## Endpoint: Execute Procedure

GET|POST /elliRPC/{packageName}/{procedureName}.{contentType}

### Request

```
POST /elliRPC/_bulk
Content-Type: application/json

{
    "procedures": {
        "{procedureId}": {
            "package": "{packageName}",
            "procedure": "{procedureName}",
            "request": {}
        }
    }
}
```

### Response

```
Content-Type: application/json

{
    "procedures": {
        "{procedureId}": {
            "package": "{packageName}",
            "procedure": "{procedureName}",
            "status": "{statusName}",
            "response": {}
        }
    }
}
```

## Endpoint: Fetch File

GET /elliRPC/_file/{name}.{extension}

## Endpoint: Upload File

PUT /elliRPC/_file/{name}.{extension}

Content-Type-Header (Request)

- application/json
- application/xml
- application/x-www-form-urlencoded
- multipart/form-data
- (application|image|text|video)/type (File Upload)


---
<small>Ellinaut is powered by [NXI GmbH & Co. KG](https://nxiglobal.com)
and [BVH Bootsvermietung Hamburg GmbH](https://www.bootszentrum-hamburg.de).</small>
