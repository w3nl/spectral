# OpenAPI Rules

Spectral has a built-in "oas" ruleset, with OAS being shorthand for the [OpenAPI Specification](https://openapis.org/specification).

In your ruleset file you can add `extends: "spectral:oas"` and you'll get all of the following rules applied, depending on the appropriate OpenAPI version used (detected through [formats](../getting-started/3-rulesets.md#formats)).

## OpenAPI v2 & v3

These rules apply to both OpenAPI v2.0, v3.0, and most likely v3.1, although there are some differences.

### contact-properties

The [info-contact](#info-contact) rule will ask you to put in a contact object, and this rule will make sure it's full of the most useful properties: `name`, `url`, and `email`.

Putting in the name of the developer/team/department/company responsible for the API, along with the support email and help-desk/GitHub Issues/whatever URL means people know where to go for help. This can mean more money in the bank, instead of developers just wandering off or complaining online.

**Recommended:** No

**Good Example**

```yaml
openapi: "3.0.2"
info:
  title: Awesome API
  description: A very well-defined API
  version: "1.0"
  contact:
    name: A-Team
    email: a-team@goarmy.com
    url: goarmy.com/apis/support
```

### duplicated-entry-in-enum

Each value of an `enum` must be different from one another.

**Recommended:** Yes

**Good Example**

```yaml
TheGoodModel:
  type: object
  properties:
    number_of_connectors:
      type: integer
      description: The number of extension points.
      enum:
        - 1
        - 2
        - 4
        - 8
```

**Bad Example**

```yaml
TheBadModel:
  type: object
  properties:
    number_of_connectors:
      type: integer
      description: The number of extension points.
      enum:
        - 1
        - 2
        - 3
        - 2
```

### info-contact

Info object should contain `contact` object.

Hopefully, your API description document is so good that nobody ever needs to contact you with questions, but that is rarely the case. The contact object has a few different options for contact details.

**Recommended:** Yes

**Good Example**

```yaml
openapi: "3.0.2"
info:
  title: Awesome API
  version: "1.0"
  contact:
    name: A-Team
    email: a-team@goarmy.com
```

### info-description

OpenAPI object info `description` must be present and non-empty string.

Examples can contain Markdown so you can really go to town with them, implementing getting started information like where to find authentication keys, and how to use them.

**Recommended:** Yes

**Good Example**

```yaml
openapi: 3.0.0
info:
  version: "1.0.0"
  title: Descriptive API
  description: >+
    Some description about the general point of this API, and why it exists when another similar but different API also exists.## AuthenticationThis API uses OAuth2 and tokens can be requested from [Dev Portal: Tokens](https://example.org/developers/tokens).
```

### info-license

The `info` object should have a `license` key.

It can be hard to pick a license, so if you don't have a lawyer around you can use [TLDRLegal](https://tldrlegal.com/) and [Choose a License](https://choosealicense.com/) to help give you an idea.

How useful this is in court is not entirely known, but having a license is better than not having a license.

**Recommended:** No

**Good Example**

```yaml
openapi: "3.0.2"
info:
  license:
    name: MIT
```

### license-url

Mentioning a license is only useful if people know what the license means, so add a link to the full text for those who need it.

**Recommended:** No

**Good Example**

```yaml
openapi: "3.0.2"
info:
  license:
    name: MIT
    url: https://www.tldrlegal.com/l/mit
```

### no-\$ref-siblings

Before OpenAPI v3.1, keywords next to `$ref` were ignored by most tooling, but not all. This leads to inconsistent experiences depending on what combinations of tools are used. As of v3.1 $ref siblings are allowed, so this rule will not be applied.

**Recommended:** Yes

**Bad Example**

```yaml
TheBadModel:
  $ref: "#/components/TheBadModelProperties"
  # This property should be ignored
  example: May or may not show up
```

### no-eval-in-markdown

This rule protects against an edge case, for anyone bringing in description documents from third parties and using the parsed content rendered in HTML/JS. If one of those third parties does something shady like injecting `eval()` JavaScript statements, it could lead to an XSS attack.

**Recommended:** Yes

**Bad Example**

```yaml
openapi: "3.0.2"
info:
  title: 'some title with eval(',
```

### no-script-tags-in-markdown

This rule protects against a potential hack, for anyone bringing in description documents from third parties and then generating HTML documentation. If one of those third parties does something shady like injecting `<script>` tags, they could easily execute arbitrary code on your domain, which if it's the same as your main application could be all sorts of terrible.

**Recommended:** Yes

**Bad Example**

```yaml
openapi: "3.0.2"
info:
  title: 'some title with <script>alert("You are Hacked");</script>',
```

### openapi-tags

OpenAPI object should have non-empty `tags` array.

Why? Well, you _can_ reference tags arbitrarily in operations, and definition is optional...

```yaml
/invoices/{id}/items:
  get:
    tags:
      - Invoice Items
```

Defining tags allows you to add more information like a `description`. For more information see [tag-description](#tag-description).

### openapi-tags-alphabetical

OpenAPI object should have alphabetical `tags`. This will be sorted by the `name` property.

**Recommended:** No

**Bad Example**

```yaml
tags:
  - name: "Badger"
  - name: "Aardvark"
```

**Good Example**

```yaml
tags:
  - name: "Aardvark"
  - name: "Badger"
```

### openapi-tags-uniqueness

OpenAPI object must not have duplicated tag names (identifiers).

**Recommended:** Yes

**Bad Example**

```yaml
tags:
  - name: "Badger"
  - name: "Badger"
```

**Good Example**

```yaml
tags:
  - name: "Aardvark"
  - name: "Badger"
```

### operation-description

**Recommended:** Yes

### operation-operationId

This operation ID is essentially a reference for the operation, which can be used to visually suggest a connection to other operations. This is like some theoretical static HATEOAS-style referencing, but it's also used for the URL in some documentation systems.

Make the value `lower-hyphen-case`, and try and think of a name for the action which does not relate to the HTTP message. Base it off the actual action being performed. `create-polygon`? `search-by-polygon`? `filter-companies`?

**Recommended:** Yes

### operation-operationId-unique

Every operation must have a unique `operationId`.

Why? A lot of documentation systems use this as an identifier, some SDK generators convert them to a method name, among other things.

**Recommended:** Yes

**Bad Example**

```yaml
paths:
  /pet:
    patch:
      operationId: "update-pet"
      responses:
        200:
          description: ok
    put:
      operationId: "update-pet"
      responses:
        200:
          description: ok
```

**Good Example**

```yaml
paths:
  /pet:
    patch:
      operationId: "update-pet"
      responses:
        200:
          description: ok
    put:
      operationId: "replace-pet"
      responses:
        200:
          description: ok
```

### operation-operationId-valid-in-url

Seeing as operationId is often used for unique URLs in documentation systems, it's a good idea to avoid non-URL-safe characters.

**Recommended:** Yes

**Bad Example**

```yaml
paths:
  /pets:
    get:
      operationId: get cats
```

### operation-parameters

Operation parameters are unique and non-repeating.

1. Operations must have unique `name` + `in` parameters.
2. Operation cannot have both `in: body` and `in: formData` parameters. (OpenAPI v2.0)
3. Operation must have only one `in: body` parameter. (OpenAPI v2.0)

**Recommended:** Yes

### operation-singular-tag

Use just one tag for an operation, which is helpful for some documentation systems which use tags to avoid duplicate content.

**Recommended:** No

### operation-success-response

Operation must have at least one `2xx` or `3xx` response. Any API operation (endpoint) can fail, but presumably, it is also meant to do something constructive at some point. If you forget to write out a success case for this API, then this rule will let you know.

**Recommended:** Yes

**Bad Example**

```yaml
paths:
  /path:
    get:
      responses:
        418:
          description: teapot
```

### operation-tags

Operation should have non-empty `tags` array.

**Recommended:** Yes

### operation-tag-defined

Operation tags should be defined in global tags.

**Recommended:** Yes

### path-declarations-must-exist

Path parameter declarations cannot be empty, ex.`/given/{}` is invalid.

**Recommended:** Yes

### path-keys-no-trailing-slash

Keep trailing slashes off of paths, as it can cause some confusion. Some web tooling (like mock servers, real servers, code generators, application frameworks, etc.) will treat `example.com/foo` and `example.com/foo/` as the same thing, but other tooling will not. Avoid any confusion by just documenting them without the slash, and maybe some tooling will let people shove a / on there when they're using it, or maybe not, but at least the docs are suggesting how it should be done properly.

**Recommended:** Yes

### path-not-include-query

Don't put query string items in the path, they belong in parameters with `in: query`.

**Recommended:** Yes

### path-params

Path parameters are correct and valid.

1. For every parameter referenced in the path string (i.e: `/users/{userId}`), the parameter must be defined in either
   `path.parameters`, or `operation.parameters` objects (non-standard HTTP operations will be silently ignored.)

2. every `path.parameters` and `operation.parameters` parameter must be used in the path string.

**Recommended:** Yes

### tag-description

Tags alone are not very descriptive. Give folks a bit more information to work with.

```yaml
tags:
  - name: "Aardvark"
    description: Funny-nosed pig-head raccoon.
  - name: "Badger"
    description: Angry short-legged omnivores.
```

If your tags are business objects then you can use the term to explain them a bit. An 'Account' could be a user account, company information, bank account, potential sales lead, or anything. What is clear to the folks writing the document is probably not as clear to others.

```yaml
tags:
  - name: Invoice Items
    description: |+
      Giant long explanation about what this business concept is, because other people _might_ not have a clue!
```

**Recommended:** No

### typed-enum

Enum values should respect the `type` specifier.

**Recommended:** Yes

**Good Example**

```yaml
TheGoodModel:
  type: object
  properties:
    number_of_connectors:
      type: integer
      description: The number of extension points.
      enum:
        - 1
        - 2
        - 4
        - 8
```

**Bad Example**

```yaml
TheBadModel:
  type: object
  properties:
    number_of_connectors:
      type: integer
      description: The number of extension points.
      enum:
        - 1
        - 2
        - "a string!"
        - 8
```

## OpenAPI v2.0-only

These rules will only apply to OpenAPI v2.0 documents.

### oas2-anyOf

OpenAPI v3 keyword `anyOf` detected in OpenAPI v2 document.

**Recommended:** Yes

### oas2-api-host

OpenAPI `host` must be present and non-empty string.

**Recommended:** Yes

### oas2-api-schemes

OpenAPI host `schemes` must be present and non-empty array.

**Recommended:** Yes

### oas2-discriminator

The discriminator property MUST be defined at this schema and it MUST be in the required property list.

**Recommended:** Yes

### oas2-host-not-example

Server URL should not point to example.com.

**Recommended:** No

### oas2-host-trailing-slash

Server URL should not have a trailing slash.

**Recommended:** Yes

### oas2-oneOf

OpenAPI v3 keyword `oneOf` detected in OpenAPI v2 document.

**Recommended:** Yes

### oas2-operation-formData-consume-check

Operations with an `in: formData` parameter must include `application/x-www-form-urlencoded` or `multipart/form-data` in their `consumes` property.

**Recommended:** Yes

### oas2-operation-security-defined

Operation `security` values must match a scheme defined in the `securityDefinitions` object.
Ignores empty `security` values for cases where authentication is explicitly not required or optional.

**Recommended:** Yes

### oas2-parameter-description

Parameter objects should have a `description`.

**Recommended:** No

### oas2-schema

Validate structure of OpenAPI v2 specification.

**Recommended:** Yes

### oas2-unused-definition

Potential unused reusable `definition` entry has been detected.

<!-- theme: warning -->

> #### Warning
>
> This rule may identify false positives when linting a specification
> that acts as a library (a container storing reusable objects, leveraged by other
> specifications that reference those objects).

**Recommended:** Yes

### oas2-valid-media-example

Examples must be valid against their defined schema. Common reasons you may see errors are:

- The value used for property examples is not the same type indicated in the schema (`string` vs. `integer`, for example).
- Examples contain properties not included in the schema.

**Recommended:** Yes

For example, if you have a Pet object with an `id` property as type `integer`, and `name` and `petType` properties as type `string`, the `examples` properties type should match the schema:

```yaml
schemas:
  Pet:
    title: Pet
    type: object
    properties:
      id:
        type: integer
      name:
        type: string
      petType:
        type: string
    required:
      - id
      - name
      - petType
```

**Good Example**

```yaml
paths:
  '/pet/{petId}':
    get:
      ...
      responses:
        '200':
          description: Pet Found
          schema:
            $ref: '#/definitions/Pet'
          examples:
            Get Pet Bubbles:
              id: 123
              name: 'Bubbles'
              petType: 'dog'
```

**Bad Example**

This would throw an error since `petType` is an `integer`, not a `string`.

```yaml
paths:
  '/pet/{petId}':
    get:
      ...
      responses:
        '200':
          description: Pet Found
          schema:
            $ref: '#/definitions/Pet'
          examples:
            Get Pet Bubbles:
              id: 123
              name: 'Bubbles'
              petType: 123
```

## OpenAPI v3-only

These rules will only be applied to OpenAPI v3.0 documents.

### oas3-api-servers

OpenAPI `servers` must be present and non-empty array.

**Recommended:** Yes

Share links to any servers that people might care about. If this is going to be given to internal people then usually that is localhost (so they know the right port number), staging, and production.

```yaml
servers:
  - url: https://example.com/api
    description: Production server
  - url: https://staging.example.com/api
    description: Staging server
  - url: http://localhost:3001
    description: Development server
```

If this is going out to the world, maybe have production and a general sandbox people can play with.

### oas3-examples-value-or-externalValue

Examples for `requestBody` or response examples can have an `externalValue` or a `value`, but they cannot have both.

**Recommended:** Yes

**Bad Example**

```yaml
paths:
  /pet:
    put:
      operationId: "replace-pet"
      requestBody:
        content:
          "application/json":
            examples:
              foo:
                summary: A foo example
                value: { "foo": "bar" }
                externalValue: "http://example.org/foo.json"
                # marp! no, can only have one or the other
```

### oas3-operation-security-defined

Operation `security` values must match a scheme defined in the `components.securitySchemes` object.

**Recommended:** Yes

### oas3-parameter-description

Parameter objects should have a `description`.

**Recommended:** No

### oas3-schema

Validate structure of OpenAPI v3 specification.

**Recommended:** Yes

### oas3-server-not-example.com

Server URL should not point to example.com.

**Recommended:** No

**Bad Example**

```yaml
servers:
  - url: https://example.com/api
    description: Production server
  - url: https://staging.example.com/api
    description: Staging server
  - url: http://localhost:3001
    description: Development server
```

We have example.com for documentation purposes here, but you should put in actual domains.

### oas3-server-trailing-slash

Server URL should not have a trailing slash.

Some tooling forgets to strip trailing slashes off when it's joining the `servers.url` with `paths`, and you can get awkward URLs like `https://example.com/api//pets`. Best to just strip them off yourself.

**Recommended:** Yes

**Good Example**

```yaml
servers:
  - url: https://example.com
  - url: https://example.com/api
```

**Bad Example**

```yaml
servers:
  - url: https://example.com/
  - url: https://example.com/api/
```

### oas3-unused-component

Potential unused reusable `components` entry has been detected.

<!-- theme: warning -->

> #### Warning
>
> This rule may identify false positives when linting a specification
> that acts as a library (a container storing reusable objects, leveraged by other
> specifications that reference those objects).

**Recommended:** Yes

### oas3-valid-media-example

Examples must be valid against their defined schema. This rule is applied to Media Type objects.

**Recommended:** Yes

For example, if you have a Pet object with an `id` property as type `integer`, and `name` and `petType` properties as type `string`, the examples properties type should match the schema:

```yaml
schemas:
  Pet:
    title: Pet
    type: object
    properties:
      id:
        type: integer
      name:
        type: string
      petType:
        type: string
    required:
      - id
      - name
      - petType
```

**Good Example**

```yaml
paths:
  '/pet/{petId}':
    get:
      ...
      responses:
        '200':
          description: Pet Found
          schema:
            $ref: '#/definitions/Pet'
          examples:
            Get Pet Bubbles:
              id: 123
              name: 'Bubbles'
              petType: 'dog'
```

**Bad Example**

This would throw an error since `petType` is an `integer`, not a `string`.

```yaml
paths:
  '/pet/{petId}':
    get:
      ...
      responses:
        '200':
          description: Pet Found
          schema:
            $ref: '#/definitions/Pet'
          examples:
            Get Pet Bubbles:
              id: 123
              name: 'Bubbles'
              petType: 123
```

### oas3-valid-schema-example

Examples must be valid against their defined schema. This rule is applied to Schema objects.

**Recommended:** Yes

**Good Example**

```yaml
schemas:
  Pet:
    title: Pet
    type: object
    properties:
      id:
        type: integer
        example: 123
      name:
        type: string
        example: Bubbles
      petType:
        type: string
        example: dog
    required:
      - id
      - name
      - petType
```

**Bad Example**

This would throw an error since the example value for `petType` is an `integer`, not a `string`.

```yaml
schemas:
  Pet:
    title: Pet
    type: object
    properties:
      id:
        type: integer
        example: 123
      name:
        type: string
        example: Bubbles
      petType:
        type: string
        example: 123
    required:
      - name
      - petType
```

### oas3-server-variables

This rule ensures that server variables defined in OpenAPI Specification 3 (OAS3) and 3.1 are valid, not unused, and result in a valid URL. Properly defining and using server variables is crucial for the accurate representation of API endpoints and preventing potential misconfigurations or security issues.

**Recommended**: Yes

**Bad Examples**

1. **Missing definition for a URL variable**:

```yaml
servers:
  - url: "https://api.{region}.example.com/v1"
    variables:
      version:
        default: "v1"
```

In this example, the variable **`{region}`** in the URL is not defined within the **`variables`** object.

2. **Unused URL variable:**

```yaml
servers:
  - url: "https://api.example.com/v1"
    variables:
      region:
        default: "us-west"
```

Here, the variable **`region`** is defined but not used in the server URL.

3. **Invalid default value for an allowed value variable**:

```yaml
servers:
  - url: "https://api.{region}.example.com/v1"
    variables:
      region:
        default: "us-south"
        enum:
          - "us-west"
          - "us-east"
```

The default value 'us-south' isn't one of the allowed values in the **`enum`**.

4. **Invalid resultant URL**:

```yaml
servers:
  - url: "https://api.example.com:{port}/v1"
    variables:
      port:
        default: "8o80"
```

Substituting the default value of **`{port}`** results in an invalid URL.

**Good Example**

```yaml
servers:
  - url: "https://api.{region}.example.com/{version}"
    variables:
      region:
        default: "us-west"
        enum:
          - "us-west"
          - "us-east"
      version:
        default: "v1"
```

In this example, both **`{region}`** and **`{version}`** variables are properly defined and used in the server URL. Also, the default value for **`region`** is within the allowed values.
