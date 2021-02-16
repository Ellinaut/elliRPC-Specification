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

All names MUST be interpreted case-sensitive. Schema names SHOULD be written as UpperCamelCase, all other names SHOULD
be written as lowerCamelCase.

Package, procedure, schema and property names SHOULD NOT begin with an underscore, as this is reserved for internal
names within this specification.

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

A schema is the definition for how an api structure for a specific usage MUST look like. The schema MAY be used by
clients to build requests or handle and validate responses.

Properties not defined within a schema MUST be ignored by servers and clients.

Missing properties from the schema MUST be treated as null. If the schema defines the missing property as not nullable,
an error MUST be thrown.

A schema MUST define a schema name, which is unique within the application domain. The schema name MUST consider the
naming conventions of this specification. A schema MAY defines a description, used for documentation purposes. A schema
SHOULD define one or more properties, which together define the structure of the schema.

A schema MAY be extended by other schemas. If a schema can only be used when extended, it could be marked as abstract.

A schema MAY extends another schema. It can redefine its properties by adding a property definition with the same
property name.

A property MUST define a property name, which is unique within the schema. The property name MUST consider the naming
conventions of this specification. A property MAY defines a description, used for documentation purposes. A property
MUST define its type. By default, a property is not nullable. A property MAY defines options, like nullable, in the type
definition. These options define how to interpret and use a type in the json structure and data model.

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
| description | string                            | The description of the schema for documentation purposes.       |
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
| object   | A simple json object without defined schema.                                                                          |

<strong>Note:</strong> Each schema SHOULD only has one id property (Property of type `id`, `idString` or `uuid`). If
there are multiple id properties defined, the application MUST handle this like composite keys in SQL.

<strong>Note:</strong> A schema which defines a property as wrapper MUST be abstract. Each schema can contain only
one `wrapper` property.

### Property Options

Property options are used to describe a property technically. Options are defined as a list so that multiple options can
be used at the same time. Options MUST be applied in the order they are defined. Pre-defined *elliRPC* options MUST
begin with an @ sign in the name and MAY have an impact on the JSON structure. Options SHOULD be used by clients and
servers to validate data structures before processing.

An application MAY offers more custom options. Custom option names MUST NOT begin with an @ sign. Custom options MUST
NOT have an impact on the JSON structure.

A client MUST ignore options it doesn't know.

Available *elliRPC* options are:

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

## Endpoint: Packages

The packages' endpoint MUST respond with all available packages and their procedures.

This endpoint SHOULD be used as leading documentation for all available *elliRPC* procedures.

This endpoint MAY be used for automatically creation of api clients.

The url path for this endpoint MUST be `/elliRPC/_packages.{contentType}`.

Possible values for `{contentType}` MUST be `html` and `json`. Implementations of this specification MAY offer more
content types for this endpoint.

This endpoint MUST be accessible only via http method `GET`.

### Request (JSON)

`GET /elliRPC/_packages.json`

### Response (JSON)

```
{
    "packages": {
        {packageName}: {
            {procedureName}: {
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
        }
    }
}
```

@todo description for the package schema

@todo: Definition for schemas: "_ContextBasedPagination" and "_OffsetBasedPagination"

## Endpoint: Status

### Request

`GET /elliRPC/_status.json`

### Response

```
{
    "packages": {
        {packageName}: {
            {procedureName}: {
                "status": {statusName},
                "description": {description}|null
            }
        }
    }
}
```

@todo description for the status schema

## Endpoint: Schema

### Request

`GET /elliRPC/_schema/{schemaName}.json`

### Response

<strong>@see `Schema`</strong>

## Endpoint: Execute Procedure

@todo description

Content-Type-Header (Request)

- application/json
- application/x-www-form-urlencoded
- multipart/form-data

### Request

`{METHOD} /elliRPC/{packageName}/{procedureName}.{contentType}`

### Response

@todo describe

## Endpoint: Execute Procedures

### Request

```
POST /elliRPC/_bulk
Content-Type: application/json

{
    "procedures": {
        {procedureId}: {
            "package": {packageName},
            "procedure": {procedureName},
            "request": {procedureRequest}
        }
    }
}
```

### Response

```
Content-Type: application/json

{
    "procedures": {
        {procedureId}: {
            "package": {packageName},
            "procedure": {procedureName},
            "status": {statusName},
            "response": {procedureResponse}
        }
    }
}
```

## Endpoint: Fetch File

GET /elliRPC/_file/{name}.{extension}

## Endpoint: Upload File

@todo 

PUT /elliRPC/_file/{name}.{extension}

Content-Type-Header (Request)

- (application|image|text|video)/type (File Upload)


---
<small>Ellinaut is powered by [NXI GmbH & Co. KG](https://nxiglobal.com)
and [BVH Bootsvermietung Hamburg GmbH](https://www.bootszentrum-hamburg.de).</small>
