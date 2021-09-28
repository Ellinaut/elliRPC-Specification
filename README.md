Specification: elliRPC
======================
<small>provided by [Ellinaut](https://github.com/Ellinaut) </small>

---

## Introduction

*elliRPC* is a specification for how a client should execute remote procedure calls (RPCs) over HTTP, and how a server
should respond the procedure execution results to the client.

## Conventions

The keywords “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and
“OPTIONAL” in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### Client Responsibilities

A client in the sense of this specification can be any application that could execute remote procedures.

Clients MUST send all requests containing a request body with a correct `Content-Type` header.

Clients MUST ignore properties which they don't know.

### Server Responsibilities

A server in the sense of this specification can be any application that executes procedures.

Servers MUST respond to all requests with a correct `Content-Type` header.

Servers MUST respond with HTTP status `400 Bad Request` if a given property has an invalid value.

Servers MUST ignore properties which they don't know.

### Naming Conventions

All names MUST be interpreted case-sensitive. Schema names SHOULD be written in UpperCamelCase, all other names (e.g.
property, package or procedure names) SHOULD be written in lowerCamelCase.

Names MUST begin with a letter and MUST NOT contain any special characters.

All names within this specification and names from official extensions of this specification starts with the
prefix `elli`, so this prefix SHOULD NOT be used by other names to avoid problems.

## Packages and Procedures

A `procedure` is an executable. A client can execute a `procedure` via *elliRPC* and gets a response containing the
execution result for this procedure from the server. The `procedure` MAY create, update or delete data on the server but
can also be used to only fetch data from the server.

A `package` is like a context within the application. *elliRPC* provides procedures grouped by `packages`, which make it
possible to use the same procedure name in multiple contexts. Every `procedure` MUST be assigned to exactly
one `package`.

```
{
    "name": {packageName},
    "description": {packageDescription}|null,
    "deprecation": {
        "endOfLife": {endOfLifeDateTime}|null,
        "replacingPackage": {packageName}|null
    },
    "errorResponse": {
        "context": {schemaContext}|null,
        "schema": {schemaName},
        "wrappedBy": {
            "context": {schemaContext}|null,
            "schema": {schemaName}
        }
    },
    "procedures": [
        {
            "name": {procedureName},
            "description": {procedureDescription}|null,
            "deprecation": {
                "endOfLife": {endOfLifeDateTime}|null,
                "replacingPackage": {packageName}|null,
                "replacingProcedure": {procedureName}|null
            },
            "methods": {listOfAllowedHttpMethods},
            "request": {
                "data": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName},
                    "wrappedBy": {
                        "context": {schemaContext}|null,
                        "schema": {schemaName}
                    },
                },
                "paginatedBy": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName}
                },
                "sortedBy": {
                    {option}: {description}
                }
            },
            "response": {
                "context": {schemaContext}|null,
                "schema": {schemaName},
                "wrappedBy": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName}
                }
            }
        }
    ]
}
```

### PackageDefinition

| Property      | Type                            | Description                                                                                    |
| ------------- | ------------------------------- | ---------------------------------------------------------------------------------------------- |
| name          | string                          | The name of the package.                                                                       |
| description   | string or null                  | The human readable description of the package for documentation purposes.                      |
| deprecation   | PackageDeprecation or null      | Null, if package isn't deprecated, otherwise an object with information about the deprecation. |
| errorResponse | ProcedureDataDefinition or null | The reference to the schema and possibly a wrapper used for the responses in case of an error. |
| procedures    | ProcedureDefinition[]           | The procedures provided by this package.                                                       |

### PackageDeprecation

TODO

### ProcedureDefinition

| Property     | Type                            | Description                                                                                      |
| ------------ | ------------------------------- | ------------------------------------------------------------------------------------------------ |
| name         | string                          | The name of the procedure.                                                                       |
| description  | string or null                  | The description of the procedure for documentation purposes.                                     |
| deprecation  | ProcedureDeprecation or null    | Null, if procedure isn't deprecated, otherwise an object with information about the deprecation. |
| methods      | string[]                        | A list of possible http methods to call the procedure. Methods MUST be written UPPERCASE.        |
| request      | ProcedureRequestDefinition      | The definition of the procedure request.                                                         |
| response     | ProcedureDataDefinition or null | The reference to the schema and possibly a wrapper used for the procedure response.              |

### ProcedureDeprecation

TODO

### ProcedureRequestDefinition

| Property    | Type                              | Description                                                                                                   |
| ----------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| data        | ProcedureDataDefinition or null   | The reference to the schema and possibly a wrapper used to transfer data within the request to the procedure. |
| paginatedBy | SchemaReferenceDefinition or null | The reference to the schema used for pagination.                                                              |
| sortedBy    | string[]                          | A list of sort options where the key is the option and the value a description for documentation purposes.    |

<strong>Note:</strong> One of the default pagination schemas (`elliContextBasedPagination`
or `elliOffsetBasedPagination`) MAY be used for pagination. For context based pagination the
schema `elliContextBasedPagination` SHOULD be used, for offset based pagination the schema `elliOffsetBasedPagination`
SHOULD be used.

### ProcedureDataDefinition

| Property  | Type                              | Description                                                                          |
| --------- | --------------------------------- | ------------------------------------------------------------------------------------ |
| context   | string or null                    | The context MUST be `null` for `elliRPC` or an IRI for an external `JSON LD` schema. |
| schema    | string                            | The case sensitive name of (internal or external) schema, which should be extended.  |
| wrappedBy | SchemaReferenceDefinition or null | The schema reference to the schema, which is used as wrapper.                        |

## Schemas

A schema is the definition for how a data structure for a specific usage MUST look like. The schema MAY be used by
clients to automatically build requests or to handle and validate responses.

Properties not defined within a schema MUST be ignored by servers and clients.

Missing properties from the schema MUST be treated as null values.

A schema MUST define a schema name, which is unique within the application domain. For this purpose, the name of the
schema MAY be prefixed with the package name if the schema is to be used only in this package. The schema name MUST
consider the naming conventions of this specification. A schema MAY define a description, used for documentation
purposes. A schema SHOULD define one or more properties, which together define the structure of the schema.

A schema MAY be extended by other schemas. If a schema can only be used when extended, it MUST be marked as abstract.

A schema MAY extend another schema. It MAY redefine its properties by adding a property definition with the same
property name.

A property MUST define a property name, which is unique within the schema. The property name MUST consider the naming
conventions of this specification. A property MAY define a description, used for documentation purposes. A property MUST
define its type. By default, a property is not nullable. A property MAY define options, like nullable, in the type
definition. These options define how to interpret, validate and use the specific property value in the json structure
and data model.

```
{
    "name": {schemaName},
    "abstract": true|false,
    "extends": {
        "context": {schemaContext},
        "schema": {schemaName}
    },
    "description": {schemaDescription},
    "properties": [
        {
            "name": {propertyName},
            "description": {propertyDescription},
            "type": {
                "context": {propertyContext},
                "type": {propertyType},
                "options": {propertyOptionList}
            }
        }
    ]
}
```

### Schema Definition

| Property    | Type                              | Description                                                     |
| ----------- | --------------------------------- | --------------------------------------------------------------- |
| name        | string                            | The name of the schema, which is used as type.                  |
| abstract    | boolean                           | Indicates that this schema can only be used with other schemas. |
| extends     | SchemaReferenceDefinition or null | The schema, which is extended with this schema.                 |
| description | string or null                    | The description of the schema for documentation purposes.       |
| properties  | PropertyDefinition[]              | A list of properties for the schema.                            |

### SchemaReferenceDefinition

| Property    | Type           | Description                                                                          |
| ----------- | -------------- | ------------------------------------------------------------------------------------ |
| context     | string or null | The context MUST be `null` for `elliRPC` or an IRI for an external `JSON LD` schema. |
| schema      | string         | The case sensitive name of (internal or external) schema, which should be extended.  |

### PropertyDefinition

| Property    | Type                   | Description                                                   |
| ----------- | ---------------------- | ------------------------------------------------------------- |
| name        | string                 | The property name used as key in request or response body.    |
| description | string or null         | The property description for documentation.                   |
| type        | PropertyTypeDefinition | The property type.                                            |

### PropertyTypeDefinition

| Property    | Type                      | Description                                                                                      |
| ----------- | ------------------------- | ------------------------------------------------------------------------------------------------ |
| context     | string or null            | The context MUST be `null` for `elliRPC` or an IRI for an external `JSON LD` schema.             |
| type        | string                    | The case sensitive name of the build-in type or the used (internal or external) schema.          |
| options     | string[]                  | A list of property options.                                                                      |

### Default Property Types

These are the build-in property types of `elliRPC`. Additional types MAY be added by extensions of this specification.

| Type     | Description                                                                                                           |
| -------- | --------------------------------------------------------------------------------------------------------------------- |
| id       | Indicates that the property is used as id of type integer.                                                            |
| idString | Indicates that the property is used as id of type string.                                                             |
| uuid     | Indicates that the property is used as id and contains an uuid as defined in RFC 4122.                                |
| string   | -                                                                                                                     |
| integer  | -                                                                                                                     |
| decimal  | -                                                                                                                     |
| boolean  | -                                                                                                                     |
| email    | A string formatted as valid email address as defined by RFC 5322 and RFC 6530.                                        |
| date     | A date string formatted as ISO 8601 date (YYYY-MM-DD).                                                                |
| time     | A time string formatted as ISO 8601 time with timezone (hh:mm:ss±hh:mm).                                              |
| datetime | A datetime string formatted as ISO 8601 combined date and time with timezone (YYYY-MM-DD\Thh:mm:ss±hh:mm).            |
| duration | A duration string formatted as ISO 8601 duration (P<date>T<time>).                                                    |
| geoJson  | A json object formatted and to be interpreted as (GeoJson)[https://geojson.org/], defined in RFC 7946.                |
| wrapper  | A placeholder used in wrapper schemas where the wrapped content later should go. The schema it self MUST be abstract. |
| object   | A simple json object without defined schema. Whenever possible, a concrete schema SHOULD be defined instead.          |

<strong>Note:</strong> Each schema SHOULD only have one id property (Property of type `id`, `idString` or `uuid`). If
there are multiple id properties defined, the application MUST handle this in the defined order like composite keys in
SQL.

<strong>Note:</strong> A schema which defines a property as `wrapper` MUST be abstract. A schema can only contain
exactly one `wrapper` property.

### Property Options

Property options are used to describe a property technically.

Options are defined as a list so that multiple options can be used at the same time.

Options MUST be applied in the order they are defined.

Pre-defined *elliRPC* options MUST begin with an @ sign in the name and MAY have an impact on the JSON structure.

Options SHOULD be used by clients and servers to validate data structures before processing.

An application MAY offer more custom options. Custom option names MUST NOT begin with an @ sign and MUST NOT have an
impact on the JSON structure.

A client MUST ignore options it doesn't know but MUST support at least the official *elliRPC* options.

Official available *elliRPC* options are:

| Option             | Description                                                                                                    |
| ------------------ | -------------------------------------------------------------------------------------------------------------- |
| @nullable          | The property value is allowed to be `null`.                                                                    |
| @notEmpty          | The property value is not allowed to be empty. Empty values are lists without values, `null` or empty strings. |
| @positive          | The property value MUST be a positive number.                                                                  |
| @negative          | The property value MUST be a negative number.                                                                  |
| @map               | The property does contain a key-value-map (json object), where the key is a unique name for a value.           |
| @list              | The property does contain a ordered list (json array) of values.                                               |
| @set               | The property does contain a unordered list (json array) of values.                                             |
| @language          | The property does contain a key-value map, where the key MUST be a language designator (ISO 639-1).            |
| @extendedLanguage  | The property does contain a key-value map, where the key MUST be a language designator (ISO 639-2/T).          |
| @localized         | The property does contain a key-value map, where the key MUST be a region designator (ISO 3166-1).             |
| @scripted          | The property does contain a key-value map, where the key MUST be a script designator (ISO 15924).              |

#### Option Nesting

If several options are combined with one another, the order of the options is decisive. Every option in a chain MUST be
applied to the result which was generated by the previous option.

<strong>Here is an example:</strong>

Definition:

```json
{
  "name": "OptionsExample",
  "abstract": false,
  "extends": null,
  "description": "An example definition to demonstrate multiple chained options.",
  "properties": [
    {
      "name": "nullable",
      "description": "This property can be a string or null",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@nullable"
        ]
      }
    },
    {
      "name": "nullableList",
      "description": "This property can be a list of strings or null.",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@nullable",
          "@list"
        ]
      }
    },
    {
      "name": "nullableListValues",
      "description": "This property must be a list, which can contain string or null values.",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@list",
          "@nullable"
        ]
      }
    },
    {
      "name": "languageString",
      "description": "This property contains the same value in more than one language.",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@language"
        ]
      }
    },
    {
      "name": "listLanguage",
      "description": "This property contains a list of string values. Each value is translated into different languages.",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@list",
          "@language"
        ]
      }
    },
    {
      "name": "languageList",
      "description": "This property contains a set of languages. Each language contains a list of strings in the specific language.",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@language",
          "@list"
        ]
      }
    }
  ]
}
```

JSON object created by this schema:

```json
{
  "nullable": null,
  "nullableList": null,
  "nullableListValues": [
    "Example",
    null
  ],
  "languageString": {
    "de": "Beispiel",
    "en": "Example"
  },
  "listLanguage": [
    {
      "de": "Beispiel eins",
      "en": "Example one"
    },
    {
      "de": "Beispiel zwei",
      "en": "Example two"
    }
  ],
  "languageList": {
    "de": [
      "Beispiel eins",
      "Beispiel zwei"
    ],
    "en": [
      "Example one",
      "Example two"
    ]
  }
}
```

### Default Schemas

#### elliContextBasedPagination

This schema SHOULD be used for context based pagination.

With context based pagination, a procedure will be called every time with the same context to retrieve always the next
page of results with the next call with the same context. This could be useful for pagination with nosql databases which
sometimes use database pointers instead of offsets and limits.

```json
{
  "name": "elliContextBasedPagination",
  "abstract": false,
  "extends": null,
  "description": "This schema SHOULD be used for context based pagination.",
  "properties": [
    {
      "name": "context",
      "description": "The pagination context should be a uuid, but could be any string value.",
      "type": {
        "context": null,
        "type": "string",
        "options": []
      }
    }
  ]
}
```

A pagination url parameter with this schema looks like: `?pagination[context]=5980b081-d06a-4fa6-8f57-2fd4eb574e1f`.

#### elliOffsetBasedPagination

This schema SHOULD be used for offset based pagination.

```json
{
  "name": "elliOffsetBasedPagination",
  "abstract": false,
  "extends": null,
  "description": "This schema SHOULD be used for offset based pagination.",
  "properties": [
    {
      "name": "offset",
      "description": "The offset for pagination.",
      "type": {
        "context": null,
        "type": "integer",
        "options": []
      }
    },
    {
      "name": "limit",
      "description": "The limit (max results per page) for pagination.",
      "type": {
        "context": null,
        "type": "integer",
        "options": []
      }
    }
  ]
}
```

A pagination url parameter with this schema looks like: `?pagination[offset]=0&pagination[limit]=10`.

#### elliError

This schema MAY be used for errors occurred while executing a procedure.

```json
{
  "name": "elliError",
  "abstract": false,
  "extends": null,
  "description": "This schema MAY be used for errors occurred while executing procedures.",
  "properties": [
    {
      "name": "message",
      "description": "The human readable error message (could be displayed to end users).",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@language"
        ]
      }
    },
    {
      "name": "code",
      "description": "The (internal) error code for developers.",
      "type": {
        "context": null,
        "type": "integer",
        "options": []
      }
    }
  ]
}
```

#### elliCollection

This schema MAY be used as wrapper for multiple objects.

```json
{
  "name": "elliCollection",
  "abstract": true,
  "extends": null,
  "description": "This schema MAY be used as wrapper for multiple objects. ",
  "properties": [
    {
      "name": "entries",
      "description": "This property contains all entries of the collection.",
      "type": {
        "context": null,
        "type": "wrapper",
        "options": []
      }
    }
  ]
}
```

#### elliOffsetPaginatedCollection

This schema MAY be used as wrapper for multiple objects, which are paginated by offset based pagination.

```json
{
  "name": "elliOffsetPaginatedCollection",
  "abstract": true,
  "extends": {
    "context": null,
    "schema": "elliCollection"
  },
  "description": "This schema MAY be used as wrapper for multiple objects, which are paginated by offset based pagination.",
  "properties": [
    {
      "name": "numberOfEntries",
      "description": "The number of entries in the whole collection, needed to calculate offset based pagination.",
      "type": {
        "context": null,
        "type": "integer",
        "options": [
          "@positive"
        ]
      }
    }
  ]
}
```

#### elliContextPaginatedCollection

This schema MAY be used as wrapper for multiple objects, which are paginated by context based pagination.

```json
{
  "name": "elliOffsetPaginatedCollection",
  "abstract": true,
  "extends": {
    "context": null,
    "schema": "elliCollection"
  },
  "description": "This schema MAY be used as wrapper for multiple objects, which are paginated by context based pagination.",
  "properties": [
    {
      "name": "context",
      "description": "The pagination context or null if no more entries can be fetched.",
      "type": {
        "context": null,
        "type": "string",
        "options": [
          "@nullable"
        ]
      }
    }
  ]
}
```

## Versioning

Versioning of packages, procedures or schemas is not provided.

Optional properties MAY be added or removed in requests and responses at any time and MUST be ignored (treated
as `null`) by servers and clients if they are missing or unknown.

Additional required properties in responses MAY also be added at any time, since they only have to be supported by the
providing server. Unknown properties MUST be ignored by clients.

Major changes to defined structures are not allowed. Instead, an existing structure MUST be replaced with a new one when
major changes are made. Versioning MAY be achieved via different names for old and new structures.

If packages or procedures are deprecated, they MUST be marked as deprecated and MAY provide an end of life date and the
replacing package or procedure.

## Endpoint: Definition

The definition endpoint MUST respond with the full api definition.

This endpoint SHOULD be used as leading documentation and MAY be used for automatically creation of api clients or
request/response validation.

The url path for this endpoint MUST be `/elliRPC`. The endpoint MUST be accessible only via http method `GET`.

### Request

`GET /elliRPC`

### Response

```json
{
  "application": "elliRPC Example Application",
  "description": "The elliRPC Example Application is used to demonstrate the elliRPC usage.",
  "extensions": [],
  "packages": [],
  "schemas": []
}
```

| Property    | Type                | Description                                                                              |
| ----------- | ------------------- | ---------------------------------------------------------------------------------------- |
| application | string              | The name of the application.                                                             |
| description | string or null      | The description of the whole application.                                                |
| extensions  | string[]            | A list of used elliRPC extensions. Values MUST be URIs to their official specifications. |
| packages    | PackageDefinition[] | A list of all available packages within this application.                                |
| schemas     | SchemaDefinition[]  | A list of all schemas defined within this application.                                   |

## Endpoint: Execute Procedure

The `execute procedure` endpoint SHOULD be used to execute a single procedure.

The url path for this endpoint MUST be `/elliRPC/call/{packageName}/{procedureName}`.

This endpoint MUST only be accessible via the http methods which are defined in the procedure definition.

### Request

If the requested http method is `GET` or `DELETE`, request data MUST be sent in the url query. If the requested http
method is `POST`, `PATCH` or `PUT`, request data MUST be sent as json object in the request body.

If a content type header is requested and the requested content type header does not match `application/json`, the
server MUST respond with http status `415 Unsupported Media Type`.

If the request data does not match the defined schema or if package or procedure does not exist, the server MUST respond
with http status `400 Bad Request`.

Request data MUST be structured like defined in the procedure definition. The data MUST only consist of the defined
properties. Missing properties MUST be treated as `null`, additional properties MUST be ignored.

The request MAY contain additional headers for example for caching or authorization, which are not parts of this
specification. Unknown headers MUST be ignored by servers.

Pagination and sorting parameters MUST always be sent in the url query. The parameters are named `pagination` and
`sort`. The `pagination` parameter contains one or more key-value-pairs, which MUST match the defined pagination schema
in the procedure definition. The `sort` parameter contains a single value, which MUST be one of the keys defined in the
procedure request.

`{METHOD} /elliRPC/call/{packageName}/{procedureName}`

example with (offset based) pagination:
`GET /elliRPC/call/app/test?pagination[offset]=0&pagination[limit]=10`

example with sorting:
`GET /elliRPC/call/app/test?sort=titleAsc`

example with request data within url:
`GET /elliRPC/call/app/test?data[test]=test`

example with request data as json:

```
POST /elliRPC/call/app/test
Content-Type: application/json

{
    "test": "test"
}
```

### Response

The `execute procedure` endpoint MUST respond as defined in the procedure definition. The response MUST be formatted
as `json` and the content type header `application/json` MUST be used.

The endpoint MUST respond with a valid http status code, depending on success or error of the procedure execution. All
official http status codes are allowed.

The response body MUST be a valid json object as defined in the procedure definition. It MUST be empty, if the procedure
definition does define a response as `null`.

The response MUST only contain the properties defined by response schema. The client MUST ignore any additional
property. The client MAY validate all properties and MUST throw an error, if any property is missing or does not match
the defined property type.

The response MAY contain additional headers for example for caching, which is not parts of this specification. Unknown
headers MUST be ignored by clients.

## Endpoint: Bulk

The `bulk` endpoint SHOULD be used if several procedures are to be executed directly one after the other, but without
dependencies among each other. Using the bulk request can improve performance by reducing unnecessary http requests.

The url path for this endpoint MUST be `/elliRPC/bulk`.

### Request

The `bulk` endpoint MUST be called with http method `POST`. The request body MUST be formatted as JSON, where all
procedure execution requests are listed under the key `procedures`.

Since there must be no dependencies between the procedures in this request, the order of execution on the server is not
relevant and may differ from the order of the list. Likewise, the server could execute several of the procedures at the
same time.

If the requested content type header does not match `application/json`, the server MUST respond with http
status `415 Unsupported Media Type`.

Each procedure execution request MUST contain the keys `package`, `procedure`, `pagination`, `sorting` and `data`.
`pagination`, `sorting` and `data` MUST be `null`, if they are defined as `null` in the procedure request definition.
However, if defined `pagination` MAY be `null` if not used or MUST be a json object matching the defined pagination
schema. `Sorting` MAY be `null` if not used or MUST be a valid sorting option (string). `data` MUST be a valid json
object matching the defined request schema.

```
POST /elliRPC/bulk
Content-Type: application/json

{
    "procedures": [
        {
            "package": {packageName},
            "procedure": {procedureName},
            "pagination": {pagination},
            "sorting": {sorting},
            "data": {procedureData}
        }
    ]
}
```

### Response

The response MUST NOT be empty and MUST contain valid `JSON`. It MUST contain a procedure result for each requested
procedure execution.

Procedure results MUST appear in the same order as requested and are summarized under key `procedures` in the response
object. Each result consists of the requested `package`, the requested `procedure`, the `successful` execution
indicator, the procedure `data` and `meta`. `data` MUST be a json object formatted according to the procedure response
definition. `data` MAY be `null`, if the procedure response is defined as nullable. If `successful` is `false`, `data`
MUST contain the occurred error, formatted as defined by the global error object definition. `data` MAY be `null`, if no
error object is defined. `meta` MUST be a set of key-value-pairs MAY be containing additional information which would
otherwise sent via http headers.

The server SHOULD always respond with http status `200 OK`. If the client receive any other http status from the server,
it MUST assume, that all procedure executions failed in case of a server problem.

```
Content-Type: application/json

{
    "procedures": [
        {
            "package": {packageName},
            "procedure": {procedureName},
            "successful": {true|false},
            "meta": {metaData},
            "data": {procedureResponse|errorResponse}
        }
    ]
}
```

## Endpoint: Transaction

The `transaction` endpoint MUST be used if several procedures have to be executed contiguously and build on each other
or depend on each other. If an error occurs during the execution of a procedure, all previous procedures MUST be undone
and the entire request MUST fail.

The url path for this endpoint MUST be `/elliRPC/transaction`.

### Request

The `transaction` endpoint MUST be called with http method `POST`. The request body MUST be formatted as JSON, where all
procedure execution requests are listed under the key `procedures`.

Since there is a dependencies between the procedures in this request, the order of execution on the server is relevant
and MUST NOT differ from the order of the list. The server MUST execute a maximum of one procedure at a time and MUST
execute the procedures all in sequence. If an execution fails, the server MUST undo the previous executions and MUST NOT
execute any further procedure. However, the successful procedures are marked as successful in the response, so that it
is clear at which point the error occurred.

If the requested content type header does not match `application/json`, the server MUST respond with http
status `415 Unsupported Media Type`.

Each procedure execution request MUST contain the keys `package`, `procedure`, `pagination`, `sorting` and `data`.
`pagination`, `sorting` and `data` MUST be `null`, if they are defined as `null` in the procedure request definition.
However, if defined `pagination` MAY be `null` if not used or MUST be a json object matching the defined pagination
schema. `Sorting` MAY be `null` if not used or MUST be a valid sorting option (string). `data` MUST be a valid json
object matching the defined request schema.

```
POST /elliRPC/bulk
Content-Type: application/json

{
    "procedures": [
        {
            "package": {packageName},
            "procedure": {procedureName},
            "pagination": {pagination},
            "sorting": {sorting},
            "data": {procedureData}
        }
    ]
}
```

### Response

The response MUST NOT be empty and MUST contain valid `JSON`. It MUST contain a procedure result for each executed
procedure and MUST NOT contain results for non executed procedures.

Procedure results MUST appear in the same order as requested and are summarized under key `procedures` in the response
object. Each result consists of the requested `package`, the requested `procedure`, the `successful` execution
indicator, the procedure `data` and `meta`. `data` MUST be a json object formatted according to the procedure response
definition. `data` MAY be `null`, if the procedure response is defined as nullable. If `successful` is `false`, `data`
MUST contain the occurred error, formatted as defined by the global error object definition. `data` MAY be `null`, if no
error object is defined. `meta` MUST be a set of key-value-pairs MAY be containing additional information which would
otherwise sent via http headers.

The server MUST respond with http status `200 OK` if all procedures have been executed successfully. In case of an error
the server MUST respond with a suitable http status for the occurred error. If a client receives a http status other
than `200 OK`, it MUST assume that all procedures have failed or are not executed. It SHOULD parse the response to find
the failed procedure (`"successful": false`) and the concrete error if provided.

```
Content-Type: application/json

{
    "procedures": [
        {
            "package": {packageName},
            "procedure": {procedureName},
            "successful": {true|false},
            "meta": {metaData},
            "data": {procedureResponse|errorResponse}
        }
    ]
}
```

## Endpoint: Get File

The `get file` endpoint is used to retrieve a file from the server.

The `get file` endpoint MUST be called via http method `GET`.

The url path for this endpoint MUST be `/elliRPC/files/{fileName}`.

The `fileName` MUST contain a file extension separated by a point (`.`) at the end. For example: `test.jpg`.

The `fileName` MAY contain directories, which allows duplicated file names in different directories. Directories and at
least the file name MUST be separated by slashes (`/`). For example: `/elliRPC/files/testDirectory/test.jpg`.

The request MAY contain additional headers for example for caching or authorization, which are not parts of this
specification. Unknown headers MUST be ignored by servers.

This endpoint MAY respond with the http header `Content-Disposition` to give a filename which MAY be different from the
url filename.

The response MUST contain a valid `Content-Type` header.

The response MAY contain additional http headers for example for caching, which is not part of this specification.
Unknown headers MUST be ignored by clients.

The response body MUST contain only the file content.

If a file could not be found on the server, the server MUST respond with http status `404 Not Found`.

## Endpoint: Upload File

The `upload file` endpoint is used to store a file on the server.

The `upload file` endpoint MUST be called via http method `POST` or `PUT`. If called via `POST` a file will be created
on the server if it does not already exist, otherwise an error will be happened. If called via `PUT` a file will be
created on the server if it does not already exist, otherwise the existing file will be replaced with the new one.

The url path for this endpoint MUST be `/elliRPC/files/{fileName}`.

The `fileName` MUST contain a file extension separated by a point (`.`) at the end. For example: `test.jpg`.

The `fileName` MAY contain directories, which allows duplicated file names in different directories. Directories and at
least the file name MUST be separated by slashes (`/`). For example: `/elliRPC/files/testDirectory/test.jpg`.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

The request SHOULD contain a valid `Content-Type` header.

The request body MUST only contain the file content.

The server MUST respond with http status `201 Created` if the file upload was successfully. Each other http status MUST
be assumed as upload failed.

This server MAY respond with the http header `Content-Disposition` to give a new location where the file was created.
This MAY differ from the upload destination.

The response MAY contain additional http headers. Unknown headers MUST be ignored by clients.

## Endpoint: Delete File

The `delete file` endpoint is used to delete a file from the server.

The `delete file` endpoint MUST be called via http method `DELETE`.

The url path for this endpoint MUST be `/elliRPC/files/{fileName}`.

The `fileName` MUST contain a file extension separated by a point (`.`) at the end. For example: `test.jpg`.

The `fileName` MAY contain directories, which allows duplicated file names in different directories. Directories and at
least the file name MUST be separated by slashes (`/`). For example: `/elliRPC/files/testDirectory/test.jpg`.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

The server MUST respond with http status `204 No Content` in case of success. Otherwise, it MUST respond with a suitable
http error.

---
<small>Ellinaut is powered by [NXI GmbH & Co. KG](https://nxiglobal.com)
and [BVH Bootsvermietung Hamburg GmbH](https://www.bootszentrum-hamburg.de).</small>
