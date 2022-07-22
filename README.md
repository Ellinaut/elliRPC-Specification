Specification: elliRPC
======================

<small>provided by [Ellinaut](https://github.com/Ellinaut)</small>

---

## Table of Contents

1. [Introduction](#introduction)
2. [Conventions and Terms](#conventions-and-terms)
    1. [Naming Conventions](#naming-conventions)
    2. [Client Responsibilities](#client-responsibilities)
    3. [Server Responsibilities](#server-responsibilities)
    4. [Packages, Procedures and Schemas](#packages-procedures-and-schemas)
3. [Definition Structures](#definition-structures)
    1. [PackageDefinition](#packagedefinition)
    2. [ProcedureDefinition](#proceduredefinition)
    3. [TransportDefinition](#transportdefinition)
    4. [DataDefinition](#datadefinition)
    5. [SchemaReferenceDefinition](#schemareferencedefinition)
    6. [SchemaDefinition](#schemadefinition)
    7. [PropertyDefinition](#propertydefinition)
    8. [PropertyTypeDefinition](#propertytypedefinition)
        1. [Build-In Property Types](#build-in-property-types)
        2. [Property Options](#property-options)
            1. [Option Chaining](#option-chaining)
4. [Error Handling](#error-handling)
5. [Versioning](#versioning)
6. [Endpoints](#endpoints)
    1. [Endpoint: Get Documentation](#endpoint-get-documentation)
    2. [Endpoint: Get Package Definition](#endpoint-get-package-definition)
    3. [Endpoint: Execute Procedure](#endpoint-execute-procedure)
    4. [Endpoint: Execute Bulk](#endpoint-execute-bulk)
    5. [Endpoint: Execute Transaction](#endpoint-execute-transaction)
    6. [Endpoint: Get File](#endpoint-get-file)
    7. [Endpoint: Upload File](#endpoint-upload-file)
    8. [Endpoint: Delete File](#endpoint-delete-file)
7. [Extending this specification](#extending-this-specification)

## Introduction

*elliRPC* is a specification for how a client should execute remote procedure calls (RPCs) over HTTP, and how a server
should respond the procedure execution results to the client.

## Conventions and Terms

The keywords “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and
“OPTIONAL” in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### Naming Conventions

All names MUST be interpreted case-sensitive. Schema names SHOULD be written in UpperCamelCase, all other names (e.g.
property, package or procedure names) SHOULD be written in lowerCamelCase.

Names MUST begin with a letter and MUST NOT contain any special characters.

All names within this specification and names from official extensions of this specification starts with the
prefix `elli`, so this prefix SHOULD NOT be used by other names to avoid problems.

### Client Responsibilities

A client in the sense of this specification can be any application that could execute remote procedures.

Clients MUST send all requests containing a request body with a correct `Content-Type` header.

Clients MUST ignore properties which they don't know.

### Server Responsibilities

A server in the sense of this specification can be any application that executes procedures.

Servers MUST respond to all requests with a correct `Content-Type` header.

Servers MUST respond with HTTP status `400 Bad Request` if a given property has an invalid value.

Servers MUST ignore properties which they don't know.

### Packages, Procedures and Schemas

A `package` is like a context within the application. *elliRPC* provides procedures and schemas grouped by `packages`,
which make it possible to use the same procedure or schema name in multiple contexts. Every `procedure` or `schema` MUST
be assigned to exactly one `package`.

A `procedure` is an executable. A client can execute a `procedure` via *elliRPC* and gets a response containing the
execution result for this procedure from the server. The `procedure` MAY create, update or delete data on the server but
can also be used to only fetch data from the server.

A `schema` is the definition for how a data structure MUST look like. The `schema` MAY be used by clients to
automatically build requests or to handle and validate responses.

Properties not defined within a `schema` MUST be ignored by servers and clients.

Missing properties from the `schema` MUST be treated as null values.

A `schema` MUST define a schema name, which is unique within its package. The schema name MUST consider the naming
conventions of this specification. A `schema`  MAY define a description, used for documentation purposes. A schema
SHOULD define one or more properties, which together define the structure of the `schema`.

A `schema` MAY be extended by other schemas. If a schema can only be used when extended, it MUST be marked as abstract.

A `schema` MAY extend another schema. It MAY redefine its properties by adding a property definition with the same
property name.

A `property` MUST define a property name, which is unique within its schema. The property name MUST consider the naming
conventions of this specification. A property MAY define a description, used for documentation purposes. A property MUST
define its type. By default, a property is not nullable. A property MAY define options, like `@nullable`, in the type
definition. These options define how to interpret, validate and use the specific property value in the json structure
and data model.

## Definition Structures

This chapter describes all available definition structures used for the technical API definition of your API defined
according to this standard.

The definition structures are meant to be used for documentation purposes, for automatically client creation and for
request/response validations.

Definition structures are used in the documentation- and package-endpoints where they are present as JSON objects.

The following descriptions for available definition structures are structured as tables which contains the
columns `Property`, `Type` and `Description`. The column `Property` contains the technical name of a property, which
will be the same in a JSON object and which MUST be unique within a single definition. The column `Type` contains the
technical property type. If the type is written in lowerCamelCase, it presents a scalar type. If the type is written in
UpperCamelCase, it presents another definition structure, which will be lead to nested structure. If the type is
prefixed with `?`, the value COULD be `null`. If it is suffixed with `[]`, the value MUST be a list with one or more
entries. The column `Description` contains the human-readable description of the property for developers. The
description MAY contain more detailed specifications for a property.

### PackageDefinition

The `PackageDefinition` is used to describe a package within an application.

| Property    | Type                                          | Description                                                                                            |
|-------------|-----------------------------------------------|--------------------------------------------------------------------------------------------------------|
| name        | string                                        | The name of the package. This name MUST be unique within an application context.                       |
| description | ?string                                       | The human readable description of the package for documentation purposes, COULD be parsed as markdown. |
| procedures  | [ProcedureDefinition](#proceduredefinition)[] | The list of procedures provided by this package.                                                       |
| schemas     | [SchemaDefinition](#schemadefinition)[]       | The list of schemas provided by this package.                                                          |
| errors      | [ErrorDefinition](#errordefinition)[]         | The list of possible errors within this package.                                                       |

A full `PackageDefinition` as JSON looks like:

```
{
    "name": {packageName},
    "description": {packageDescription}|null,
    "fallbackLanguage": {packageFallbackLanguage}|null,
    "procedures": [
        {
            "name": {procedureName},
            "description": {procedureDescription}|null,
            "request": {
                "data": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName},
                    "wrappedBy": {
                        "context": {schemaContext}|null,
                        "schema": {schemaName}
                    },
                    "nullable": true|false,
                },
                "meta": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName}
                }
            },
            "response": {
                "data": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName},
                    "wrappedBy": {
                        "context": {schemaContext}|null,
                        "schema": {schemaName}
                    },
                    "nullable": true|false,
                },
                "meta": {
                    "context": {schemaContext}|null,
                    "schema": {schemaName}
                }
            },
            "errors": {errorCodeList}
        }
    ],
    "schemas": [
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
    ],
    "errors": [
      {
         "code": {errorCode},
         "description": {errorDescription},
         "context": {
             "context": {schemaContext},
             "schema": {schemaName}
         }
      }
    ]
}
```

### ProcedureDefinition

The `ProcedureDefinition` is used to describe a single procedure within a package.

| Property     | Type                                            | Description                                                                                                                                                                                                                                          |
|--------------|-------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name         | string                                          | The name of the procedure. This name MUST be unique within a package context.                                                                                                                                                                        |
| description  | string or null                                  | The description of the procedure for documentation purposes, COULD be parsed as markdown.                                                                                                                                                            |
| request      | [TransportDefinition](#transportdefinition)     | The definition of the procedure request.                                                                                                                                                                                                             |
| response     | [TransportDefinition](#transportdefinition)     | The definition of the procedure response.                                                                                                                                                                                                            |
| errors       | string[]                                        | The list of possible error codes for this procedure. Each code SHOULD be defined as possible error in the package.                                                                                                                                   |
| allowedUsage | string or null <br/>  STANDALONE or TRANSACTION | If value is "STANDALONE" this procedure can not be used in transactions, if value is "TRANSACTION" this procedure can only be used in transactions, if value is `null` the procedure can be used within a transactions and outside of a transaction. |

### TransportDefinition

The `TransportDefinition` is used to describe how data MUST be transferred between client and server in the request or
response for a procedure call.

| Property | Type                                                     | Description                                                                          |
|----------|----------------------------------------------------------|--------------------------------------------------------------------------------------|
| data     | ?[DataDefinition](#datadefinition)                       | The structure how data MUST be sent. MUST be`null` if no data can be sent.           |
| meta     | ?[SchemaReferenceDefinition](#schemareferencedefinition) | The structure how meta data MUST be sent. MUST be`null` if no meta data can be sent. |

**Note:** `data` MUST contain all required data which are essential for a procedure call or which have an impact to the
procedure logic. `meta` MUST contain only data, which COULD help to process a procedure or indicates client/server
behaviours but is not relevant to the procedure logic itself. `mata` is always optional, so if its `null`, it MUST not
have an impact to the procedure logic. `meta` COULD include sorting, pagination or caching parameters. `meta` or
partials of it MAY be ignored by clients or servers while `data` MUST be observed.

### DataDefinition

The `DataDefinition` is used to describe how the `data` property MUST be structured in requests or responses.

| Property  | Type                                                     | Description                                                                     |
|-----------|----------------------------------------------------------|---------------------------------------------------------------------------------|
| context   | ?string                                                  | The context if the schema isn't defined in the same package.                    |
| schema    | string                                                   | The case sensitive name of (internal or external) schema, which should be used. |
| wrappedBy | ?[SchemaReferenceDefinition](#schemareferencedefinition) | The schema reference to the schema, which is used as wrapper.                   |
| nullable  | boolean                                                  | Indicates if data MUST contain an object or also COULD be null.                 |

**Note:** The context MUST be `null` for a schema in the same package. It MUST be `elliRPC` for a pre-defined schema
from this specification. It MUST be the package name if the schema is defined in another package of the same
application. It MUST be a URL to the concrete package endpoint if the schema is defined in another package in another
application. It MUST be an IRI for an external `JSON LD` schema if an external schema (e.g. from schema.org) should be
used.

### SchemaReferenceDefinition

The `SchemaReferenceDefinition` is used to reference a schema which should be used to structure data.

| Property | Type    | Description                                                                         |
| -------- | ------- | ----------------------------------------------------------------------------------- |
| context  | ?string | The context if the schema isn't defined in the same package.                        |
| schema   | string  | The case sensitive name of (internal or external) schema, which should be extended. |

**Note:** The context MUST be `null` for a schema in the same package. It MUST be `elliRPC` for a pre-defined schema
from this specification. It MUST be the package name if the schema is defined in another package of the same
application. It MUST be a URL to the concrete package endpoint if the schema is defined in another package in another
application. It MUST be an IRI for an external `JSON LD` schema if an external schema (e.g. from schema.org) should be
used.

### SchemaDefinition

The `SchemaDefinition` is used to describe how data MUST be structured into one or more properties.

| Property    | Type                                                     | Description                                                                            |
| ----------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| name        | string                                                   | The name of the schema, which MUST be unique within a package.                         |
| abstract    | boolean                                                  | Indicates that this schema can only be used with other schemas.                        |
| extends     | ?[SchemaReferenceDefinition](#schemareferencedefinition) | The schema, which is extended with this schema.                                        |
| description | ?string                                                  | The description of the schema for documentation purposes, COULD be parsed as markdown. |
| properties  | [PropertyDefinition](#propertydefinition)[]              | A list of properties for the schema.                                                   |

### PropertyDefinition

The `PropertyDefinition` is used to describe how a single property MUST be used within a schema.

| Property    | Type                                              | Description                                                                                      |
| ----------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| name        | string                                            | The property name used as key in request or response body, which MUST be unique within a schema. |
| description | ?string                                           | The property description for documentation, COULD be parsed as markdown.                         |
| type        | [PropertyTypeDefinition](#propertytypedefinition) | The property type definition.                                                                    |

### PropertyTypeDefinition

The `PropertyTypeDefinition` is used to describe how a property look like and how it MUST be handled.

| Property | Type     | Description                                                                              |
| -------- | -------- | ---------------------------------------------------------------------------------------- |
| context  | ?string  | The context of the schema which is used as type or`null` for build-in types.             |
| type     | string   | The case sensitive name of the build-in type, or the used (internal or external) schema. |
| options  | string[] | A list of property options.                                                              |

**Note:** The context MUST be `null` if the type is a schema in the same package or a build-in type. It MUST
be `elliRPC` if the type is a pre-defined schema from this specification. It MUST be the package name if the type is a
schema defined in another package of the same application. It MUST be a URL to the concrete package endpoint if the type
is a schema defined in another package in another application. It MUST be an IRI for an external `JSON LD` schema if the
type is an external schema (e.g. from schema.org).

#### Build-In Property Types

These are the build-in property types of *elliRPC*. Additional types MAY be added by extensions of this specification.

| Type          | Description                                                                                                           |
|---------------|-----------------------------------------------------------------------------------------------------------------------|
| uuid          | A string formatted as valid uuid defined by RFC 4122.                                                                 |
| string        | -                                                                                                                     |
| integer       | -                                                                                                                     |
| decimal       | -                                                                                                                     |
| boolean       | -                                                                                                                     |
| email         | A string formatted as valid email address as defined by RFC 5322 and RFC 6530.                                        |
| uri           | A string formatted as valid uri defined by RFC 3986.                                                                  |
| dataUrl       | A string formatted as valid data url defined by RFC 2397.                                                             |
| fileReference | A string formatted as valid uri to a file ressource.                                                                  |
| htmlContent   | A string which contains html content.                                                                                 |
| xmlContent    | A string which contains xml content.                                                                                  |
| mdContent     | A string which contains markdown content.                                                                             |
| svgContent    | A string which contains svg content.                                                                                  |
| binaryContent | A string which contains binary content encoded as base64.                                                             |
| date          | A date string formatted as ISO 8601 date (YYYY-MM-DD).                                                                |
| time          | A time string formatted as ISO 8601 time with timezone (hh:mm:ss±hh:mm).                                              |
| datetime      | A datetime string formatted as ISO 8601 combined date and time with timezone (YYYY-MM-DD\Thh:mm:ss±hh:mm).            |
| duration      | A duration string formatted as ISO 8601 duration (P<date>T<time>).                                                    |
| geoJson       | A json object formatted and to be interpreted as [GeoJson](https://geojson.org/), defined in RFC 7946.                |
| wrapper       | A placeholder used in wrapper schemas where the wrapped content later should go. The schema it self MUST be abstract. |
| object        | A simple json object without defined schema. Whenever possible, a concrete schema SHOULD be defined instead.          |

**Note:** A schema which defines a property as `wrapper` MUST be abstract. A schema MUST only contain exactly
one `wrapper` property. If the schema is used at "wrappedBy", the wrapper property contains an object matching the given
schema, which is wrapped by the schema which defines the wrapper property. For example: The wrapper schema could be a
reusable list which will be used with different values depending on the procedure where is will be used.

#### Property Options

Property options are used to describe a property technically.

Options are defined as a list so that multiple options can be used at the same time.

Options MUST be applied in the order they are defined.

Pre-defined *elliRPC* options MUST begin with an @ sign in the name and MAY have an impact on the JSON structure.

Option MAY specify more details about itself. If an option specifies more details, details are provided in round
brackets behind the option: `@option(details)`.

Options SHOULD be used by clients and servers to validate data structures before processing.

An application MAY offer more custom options. Custom option names MUST NOT begin with an @ sign and MUST NOT have an
impact on the JSON structure.

A client MUST ignore options it doesn't know but MUST support at least the official *elliRPC* options.

Official available *elliRPC* options are:

| Option            | Description                                                                                                                                                                                  |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| @id               | The property value is used as identifier for the object which contains the property.                                                                                                         |
| @nullable         | The property value is allowed to be`null`. By default, without this option values are not allowed to be `null`.                                                                              |
| @notEmpty         | The property value is not allowed to be empty. Empty values are lists without values and empty strings. `null` as value will be forbidden by default, if option `@nullable` is not set.      |
| @positive         | The property value MUST be a positive number.                                                                                                                                                |
| @negative         | The property value MUST be a negative number.                                                                                                                                                |
| @map              | The property MUST contain a key-value-map (object in json), where the key is a unique name for a value.                                                                                      |
| @list             | The property MUST contain a ordered list (array in json) of values.                                                                                                                          |
| @set              | The property MUST contain a unordered list (array in json) of values.                                                                                                                        |
| @language         | The property MUST contain a key-value map, where the key MUST be a language designator (ISO 639-1). Possible languages MAY be defined as comma seperated list: `@language(de,en,fr)`         |
| @extendedLanguage | The property MUST contain a key-value map, where the key MUST be a language designator (ISO 639-2/T). Possible languages MAY be defined as comma seperated list: `@language(de-DE,en-GB)`    |
| @localized        | The property MUST contain a key-value map, where the key MUST be a region designator (ISO 3166-1). Possible regions MAY be defined as comma seperated list: `@language(DE,GB)`               |
| @scripted         | The property MUST contain a key-value map, where the key MUST be a script designator (ISO 15924). Possible script designators MAY be defined as comma seperated list: `@language(Latn,Latf)` |
| @enum             | The property value MUST be one of the defined values. Possible values are defined as comma seperated list: `@enum(VALUE_1,VALUE_2)`                                                          |
| @min              | The property value MUST be greater than or equal to the defined numeric value. Value is defined as follows: `@min(1)` or `@min(1.2)`                                                         |
| @max              | The property value MUST be lower than or equal to the defined numeric value. Value is defined as follows: `@max(1)` or `@max(1.2)`                                                           |
| @minLength        | The property value MUST be a string with string length greater than or equal to the defined value. Value is defined as follows: `@minLength(2)`                                              |
| @maxLength        | The property value MUST be a string with string length lower than or equal to the defined numeric value. Value is defined as follows: `@maxLength(100)`                                      |
| @regex            | The property value MUST match the given regular expression. Expression is defined as follows: `@regex(/^[A-Za-z]+$/)`                                                                        |

##### Option Chaining

If several options are combined with one another, the order of the options is decisive. Every option in the chain MUST
be applied to the result which was generated by the previous option.

**Example 1:** The options `@list` and `@language` are applied to a property of type string. This leads to the fact that
first by `@list` a list is expected instead of a single value. Each element in this list will contain a key-value object
with language as key and translation as string value due to `@language`. If the options had been specified the other way
around, i.e. `@language` and then `@list`, the value of the property would be a Key-Value object with language as key
and a list with several translated strings as value.

**Example 2:** The options `@language` and `@nullable` are applied to a property of type string. This leads to the fact
that first by `@language` a key-value object with language as key and translation as string value is expected instead of
a single value. Each language could contain `null` as value due to `@nullable`. If the options had been specified the
other way around, i.e. `@nullable` and then `@language`, the value of the property would be a Key-Value object with
language as key and a list with several translated strings as value or `null` but defined language strings can not
be `null`.

### ErrorDefinition

The `ErrorDefinition` is used to describe a possible error and how it COULD be customized.

| Property    | Type                                                     | Description                                                                       |
|-------------|----------------------------------------------------------|-----------------------------------------------------------------------------------|
| code        | string                                                   | The unique error code within this package.                                        |
| description | ?string                                                  | The human readable description how and when this error could occur.               |
| context     | ?[SchemaReferenceDefinition](#schemareferencedefinition) | The schema reference to the schema, which is used for the optional error context. |

## Error Handling

The application MAY abort the procedure call immediately if an error occurs but COULD also continue execution to respond
with a list of errors, which otherwise would occur after fixing the first occurred error.

See endpoint definition to see how errors are responded.

Error objects MUST be structured like this:

| Property | Type                | Description                                                                                                                                                                            |
|----------|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message  | string{`@language`} | The message contains an human-readable description which MAY is displayed to end-users. This property MUST contain an object with language designator as key and translation as value. |
| code     | ?string             | The error code is an optional string value which contains an application specific error code for each kind of error.                                                                   |
| source   | ?string             | A JSON Pointer[RFC6901](https://datatracker.ietf.org/doc/html/rfc6901) to the associated property in the request document which lead to the occurred error.                            |
| context  | ?object             | An optional object with contextual properties for this error. The schema SHOULD be previously be defined by an error definition.                                                       |

Here is how this would look like in JSON:

```json
{
  "message": {
    "en": "This is an error example."
  },
  "code": "not_found",
  "source": "/data/test",
  "context": null
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

Real api versioning MAY be achieved by providing multiple versions of the same api under different url paths, where the
url path SHOULD contain the version number as a prefix before the actual *elliRPC* path or by replacing packages and
naming the packages with versions.

## Endpoints

Several endpoints are defined by this specification.

The url path for each endpoint MAY be prefixed with one or more additional parts to use this specification beside
others. If a prefix is used, the same prefix MUST be used for all endpoints.

The following endpoints are defined by this specification, other endpoints MAY be defined by extensions of this
specification.

| Endpoint                                                   | HTTP method | URL path                   | Description                                                                               |
| ---------------------------------------------------------- | ----------- | -------------------------- | ----------------------------------------------------------------------------------------- |
| [Get Documentation](#endpoint-get-documentation)           | GET         | /definitions               | This endpoint provides the full api definition.                                           |
| [Get Package Definition](#endpoint-get-package-definition) | GET         | /definitions/{packageName} | This endpoint provides the definition for a single package.                               |
| [Execute Procedure](#endpoint-execute-procedure)           | POST        | /procedures/execute        | This endpoint is used to execute a single procedure.                                      |
| [Execute Bulk](#endpoint-execute-bulk)                     | POST        | /procedures/bulk           | This endpoint is used to execute multiple procedures independently of each other.         |
| [Execute Transaction](#endpoint-execute-transaction)       | POST        | /procedures/transaction    | This endpoint is used to execute multiple procedures dependently in a single transaction. |
| [Get File](#endpoint-get-file)                             | GET         | /files/{fileName}          | This endpoint is used to retrieve a file from the server.                                 |
| [Upload File](#endpoint-upload-file)                       | POST or PUT | /files/{fileName}          | This endpoint is used to upload a file to the server.                                     |
| [Delete File](#endpoint-delete-file)                       | DELETE      | /files/{fileName}          | This endpoint is used to delete a file from the server.                                   |

### Endpoint: Get Documentation

This endpoint MUST respond with the full api definition and SHOULD be used as the leading documentation. It MAY be used
for automatically creation of api clients or the request/response validation.

The url path for this endpoint (after a possible prefix) MUST be `/definitions`. The endpoint MUST be accessible only
via HTTP method `GET`.

#### Request

`GET /definitions`

#### Response

```json
{
  "application": "elliRPC Example Application",
  "description": "The elliRPC Example Application is used to demonstrate the elliRPC usage.",
  "extensions": [],
  "packages": []
}
```

| Property    | Type                                      | Description                                                                                    |
| ----------- | ----------------------------------------- | ---------------------------------------------------------------------------------------------- |
| application | string                                    | The name of the application.                                                                   |
| description | string or null                            | The description of the whole application. It COULD be parsed as markdown.                      |
| extensions  | string[]                                  | A list of used specification extensions. Values MUST be URIs to their official specifications. |
| packages    | [PackageDefinition](#packagedefinition)[] | A list of all available packages within this application.                                      |

### Endpoint: Get Package Definition

This endpoint MUST respond with the definition for a single package. The URI for this endpoint MAY be used as the
context in schema references.

The url path for this endpoint (after a possible prefix) MUST be `/definitions/{packageName}`. The endpoint MUST be
accessible only via HTTP method `GET`.

#### Request

```
GET /definitions/examplePackage
Content-Type: application/json
```

#### Response

The endpoint MUST respond with a valid HTTP status `200 OK` if requested package is available. If a content type header
is requested and the requested content type header does not match `application/json`, the server MUST respond with HTTP
status `415 Unsupported Media Type` and a single error object in the response body. If the requested package does not
exist, the server MUST respond with HTTP status `404 Not Found` and a single error object in the response body.

The package response MUST be sent as json object of type [`PackageDefinition`](#packagedefinition) in the response body.

*Example response body:*

```json
{
  "name": "examplePackage",
  "description": "This is an example package for documentation purposes.",
  "procedures": [
    {
      "name": "exampleProcedure",
      "description": "This is an example procedure for documentation purposes.",
      "request": {
        "data": null,
        "meta": null
      },
      "response": {
        "data": {
          "context": null,
          "schema": "exampleSchema",
          "wrappedBy": null
        },
        "meta": null
      },
      "errors": [],
      "allowedUsage": null
    }
  ],
  "schemas": [
    {
      "name": "exampleSchema",
      "abstract": false,
      "extends": null,
      "description": "This is an example schema for documentation purposes.",
      "properties": [
        {
          "name": "test",
          "description": "Test property contains a simple string which is not nullable.",
          "type": {
            "context": null,
            "type": "string",
            "options": []
          }
        }
      ]
    }
  ]
}
```

### Endpoint: Execute Procedure

This endpoint SHOULD be used to execute a single procedure.

The url path for this endpoint (after a possible prefix) MUST be `/procedures/execute`. This endpoint MUST only be
accessible via HTTP method `POST`.

This endpoint can only execute procedures which are defined as `null` or `STANDALONE` via property "allowedUsage".

#### Request

The request body MUST contain a valid json object with properties `package`, `procedure`, `data` and `meta`.

`package` and `procedure` MUST contain valid string values to identify the procedure which should be executed.

The request data MUST be structured like defined in the procedure definition and MUST be given under key `data`. The
data MUST only consist of the defined properties. Missing properties MUST be treated as `null`, additional properties
MUST be ignored. Request data MUST be `null` if defined as `null` in the procedure definition.

Request metadata MUST be structured like defined in the procedure definition and MUST be given under key `meta`. The
metadata MUST only consist of the defined properties. Missing properties MUST be treated as `null`, additional
properties MUST be ignored. Metadata MUST be `null` if defined as `null` in the procedure definition.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

*Example request:*

```
POST /procedures/execute
Content-Type: application/json
```

with body

```
{
    "package": "examplePackage",
    "procedure": "exampleProcedure",
    "data": null,
    "meta": null
}
```

#### Response

This endpoint MUST respond with a valid json document and the content type header `application/json` MUST be used.

The endpoint MUST respond with a valid HTTP status `200 OK` if called with a valid data structure. If the request data
does not match the defined schema, does not contain valid json or if package or procedure does not exist, the server
MUST respond with HTTP status `400 Bad Request` and a single error object in the response body. If a content type header
is requested and the requested content type header does not match `application/json`, the server MUST respond with HTTP
status `415 Unsupported Media Type` and a single error object in the response body.

The response body MUST be a valid json object structured like:

| Property | Type                       | Description                                                                                           |
| -------- | -------------------------- | ----------------------------------------------------------------------------------------------------- |
| success  | boolean                    | Indicator if procedure execution was successful or not.                                               |
| data     | ?object                    | The procedure result data for the client, structured as defined by the procedure response definition. |
| meta     | ?object                    | Additional meta data for the client, structured as defined by the procedure response definition.      |
| errors   | [Error](#error-handling)[] | A list of error objects.                                                                              |

The response data MUST only contain the properties defined by response schema. The client MUST ignore any additional
property. The client MAY validate all properties and MUST throw an error, if any property is missing or does not match
the defined property type.

The response MAY contain additional headers for example for caching, which is not parts of this specification. Unknown
headers MUST be ignored by clients.

**Example Response with data and meta and without errors:**

```json
{
  "success": true,
  "data": {
    "test": "Test value"
  },
  "meta": null,
  "errors": []
}
```

### Endpoint: Execute Bulk

This endpoint SHOULD be used if several procedures are to be executed directly one after the other, but without
dependencies among each other. Using the bulk request can improve performance by reducing unnecessary HTTP requests.

The url path for this endpoint (after a possible prefix) MUST be `/procedures/bulk`. This endpoint MUST only be
accessible via HTTP method `POST`.

This endpoint can only execute procedures which are defined as `null` or `STANDALONE` via property "allowedUsage".

#### Request

The request body MUST be formatted as JSON, where all procedure calls are listed under the key `procedures`.

Since there must be no dependencies between the procedures in this request, the order of execution on the server is not
relevant and may differ from the order of the list. Likewise, the server could execute several of the procedures at the
same time.

Each procedure call MUST contain the properties `package`, `procedure`, `data` and `meta`.

`package` and `procedure` MUST contain valid string values to identify the procedure which should be executed.

Data for the procedure execution MUST be structured like defined in the procedure definition and MUST be given under
key `data` for each procedure call. The data MUST only consist of the defined properties. Missing properties MUST be
treated as `null`, additional properties MUST be ignored. `data` MUST be `null` if defined as `null` in the procedure
definition.

Metadata for the procedure execution MUST be structured like defined in the procedure definition and MUST be given under
key `meta` for each procedure call. The metadata MUST only consist of the defined properties. Missing properties MUST be
treated as `null`, additional properties MUST be ignored. `meta` MUST be `null` if defined as `null` in the procedure
definition.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

*Example request:*

```
POST /procedures/bulk
Content-Type: application/json
```

with body

```json
{
  "procedures": [
    {
      "package": "examplePackage",
      "procedure": "exampleProcedure",
      "data": null,
      "meta": null
    }
  ]
}
```

#### Response

The response MUST NOT be empty and MUST contain a valid json object. It MUST contain a procedure result for each
requested procedure call.

The endpoint MUST respond with a valid HTTP status `200 OK` if called with a valid data structure. Application errors
MUST NOT change the HTTP status but MUST be reported via success indicator in the response object. If the request data
does not match the defined schema, does not contain valid json or if package or procedure does not exist, the server
MUST respond with HTTP status `400 Bad Request` and a single error object in the response body. If a content type header
is requested and the requested content type header does not match `application/json`, the server MUST respond with HTTP
status `415 Unsupported Media Type` and a single error object in the response body.

Procedure results MUST appear in the same order as requested and are summarized under key `procedures` in the response
object. Each result consists of the requested `package`, the requested `procedure`, the `success` execution indicator,
the procedure result `data` and `meta`. `data` MUST be a json object formatted according to the procedure response
definition. `data` MUST be `null` if defined as `null` by the procedure definition. If `success` is `false` the
procedure call has failed. `meta` MUST be a json object formatted according to the procedure response definition. `meta`
MUST be `null` if defined as `null` by the procedure definition. If any errors (critical or not) occurred on procedure
call, they SHOULD be reported and MUST be given as list of error objects under key `errors` for each procedure result.

*Example response body:*

```json
{
  "procedures": [
    {
      "package": "examplePackage",
      "procedure": "exampleProcedure",
      "success": true,
      "data": {
        "test": "Test value"
      },
      "meta": null,
      "errors": []
    }
  ]
}
```

### Endpoint: Execute Transaction

This endpoint MUST be used if several procedures have to be executed contiguously and build on each other or depend on
each other. If an error occurs during the execution of a procedure, all previous procedures MUST be undone and the
entire request MUST fail.

The url path for this endpoint (after a possible prefix) MUST be `/procedures/transaction`. This endpoint MUST only be
accessible via HTTP method `POST`.

This endpoint can only execute procedures which are defined as `null` or `TRANSACTION` via property "allowedUsage".

#### Request

The request body MUST be formatted as a valid json object, where all procedure calls are listed under the
key `procedures`.

Since there are dependencies between the procedures in this request, the order of execution on the server is relevant
and MUST NOT differ from the order of the requested list. The server MUST execute a maximum of one procedure at a time
and MUST execute the procedures all in sequence. If an execution fails, the server MUST undo the previous executions and
MUST NOT execute any further procedure. However, the successful procedures are marked as successful in the response, so
that it is clear at which point the error occurred.

Each procedure call MUST contain the properties `package`, `procedure`, `data` and `meta`.

`package` and `procedure` MUST contain valid string values to identify the procedure which should be executed.

Data for the procedure execution MUST be structured like defined in the procedure definition and MUST be given under
key `data` for each procedure call. The data MUST only consist of the defined properties. Missing properties MUST be
treated as `null`, additional properties MUST be ignored. `data` MUST be `null` if defined as `null` in the procedure
definition.

Metadata for the procedure execution MUST be structured like defined in the procedure definition and MUST be given under
key `meta` for each procedure call. The metadata MUST only consist of the defined properties. Missing properties MUST be
treated as `null`, additional properties MUST be ignored. `meta` MUST be `null` if defined as `null` in the procedure
definition.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

*Example request:*

```
POST /procedures/transaction
Content-Type: application/json
```

with body

```json
{
  "procedures": [
    {
      "package": "examplePackage",
      "procedure": "exampleProcedure",
      "data": null,
      "meta": null
    },
    {
      "package": "examplePackage",
      "procedure": "exampleProcedure",
      "data": null,
      "meta": null
    }
  ]
}
```

#### Response

The response MUST NOT be empty and MUST contain valid `JSON`. It MUST contain a procedure result for each called
procedure under key `procedures` and MUST NOT contain results for non executed procedures.

The endpoint MUST respond with a valid HTTP status `200 OK` if all procedures have been executed successfully. In case
of an error the server MUST respond with a suitable HTTP status for the occurred error. If a client receives a HTTP
status other than `200 OK`, it MUST assume that all procedures have failed or are not executed.It SHOULD parse the
response to find the failed procedure (`"success": false`) and the concrete error if provided. If the request data does
not match the defined schema, does not contain valid json or if package or procedure does not exist, the server MUST
respond with HTTP status `400 Bad Request` and a single error object in the response body. If a content type header is
requested and the requested content type header does not match `application/json`, the server MUST respond with HTTP
status `415 Unsupported Media Type` and a single error object in the response body.

Procedure results MUST appear in the same order as requested and are summarized under key `procedures` in the response
object. Each result consists of the requested `package`, the requested `procedure`, the `success` execution indicator,
the procedure result `data` and `meta`. `data` MUST be a json object formatted according to the procedure response
definition. `data` MUST be `null` if defined as `null` by the procedure definition. If `success` is `false` the
procedure call has failed. `meta` MUST be a json object formatted according to the procedure response definition. `meta`
MUST be `null` if defined as `null` by the procedure definition. If any errors (critical or not) occurred on procedure
call, they SHOULD be reported and MUST be given as list of error objects under key `errors` for each procedure result.

*Example response body:*

```json
{
  "procedures": [
    {
      "package": "examplePackage",
      "procedure": "exampleProcedure",
      "success": true,
      "data": {
        "test": "Test value"
      },
      "meta": null,
      "errors": []
    },
    {
      "package": "examplePackage",
      "procedure": "exampleProcedure",
      "success": true,
      "data": {
        "test": "Test value"
      },
      "meta": null,
      "errors": []
    }
  ]
}
```

### Endpoint: Get File

This endpoint is used to retrieve a file from the server.

This endpoint MUST be called via HTTP method `GET`.

The url path for this endpoint MUST be `/elliRPC/files/{fileName}`.

The `fileName` MUST contain a file extension separated by a point (`.`) at the end. For example: `test.jpg`.

The `fileName` MAY contain directories, which allows duplicated file names in different directories. Directories and at
least the file name MUST be separated by slashes (`/`). For example: `/elliRPC/files/testDirectory/test.jpg`.

The request MAY contain additional headers for example for caching or authorization, which are not parts of this
specification. Unknown headers MUST be ignored by servers.

This endpoint MAY respond with the HTTP header `Content-Disposition` to give a filename which MAY be different from the
url filename.

The response MUST contain a valid `Content-Type` header.

The response MAY contain additional HTTP headers for example for caching, which is not part of this specification.
Unknown headers MUST be ignored by clients.

The response body MUST contain only the raw file content.

If a file could not be found on the server, the server MUST respond with HTTP status `404 Not Found`.

### Endpoint: Upload File

This endpoint is used to store a file on the server.

This endpoint MUST be called via HTTP method `POST` or `PUT`. If called via `POST` a file will be created on the server
if it does not already exist, otherwise an error will be happened. If called via `PUT` a file will be created on the
server if it does not already exist, otherwise the existing file will be replaced with the new one.

The url path for this endpoint MUST be `/elliRPC/files/{fileName}`.

The `fileName` MUST contain a file extension separated by a point (`.`) at the end. For example: `test.jpg`.

The `fileName` MAY contain directories, which allows duplicated file names in different directories. Directories and at
least the file name MUST be separated by slashes (`/`). For example: `/elliRPC/files/testDirectory/test.jpg`.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

The request SHOULD contain a valid `Content-Type` header.

The request body MUST only contain the raw file content.

The server MUST respond with HTTP status `201 Created` if the file upload was successfully. Each other HTTP status MUST
be assumed as upload failed.

This server MAY respond with the HTTP header `Location` to give a new location where the file was created.
This MAY differ from the upload destination.

The response MAY contain additional HTTP headers. Unknown headers MUST be ignored by clients.

### Endpoint: Delete File

This endpoint is used to delete a file from the server.

This endpoint MUST be called via HTTP method `DELETE`.

The url path for this endpoint MUST be `/elliRPC/files/{fileName}`.

The `fileName` MUST contain a file extension separated by a point (`.`) at the end. For example: `test.jpg`.

The `fileName` MAY contain directories, which allows duplicated file names in different directories. Directories and at
least the file name MUST be separated by slashes (`/`). For example: `/elliRPC/files/testDirectory/test.jpg`.

The request MAY contain additional headers for example for authorization, which is not part of this specification.
Unknown headers MUST be ignored by servers.

The server MUST respond with HTTP status `204 No Content` in case of success. Otherwise, it MUST respond with a suitable
HTTP error.

## Extending this specification

TODO

---

<small>Ellinaut is powered by [NXI GmbH & Co. KG](https://nxiglobal.com)
and [BVH Bootsvermietung Hamburg GmbH](https://www.bootszentrum-hamburg.de).</small>
