## Lorebook Fields

These fields are inside the Lorebook object.

Lorebook object is a JSON object that contains the information about the lorebook. Lorebook object is used to define the lorebook entries and the lorebook itself.

The Lorebook object can be described as this typescript interface:
```ts
type Lorebook = {
  name?: string
  description?: string
  scan_depth?: number
  token_budget?: number
  recursive_scanning?: boolean
  extensions: Record<string, any>
  entries: Array<{
    keys: Array<string>
    secondary_keys: Array<string>
    selective: boolean // option to enable secondary key
    content: string
    extensions: Record<string, any>
    enabled: boolean
    insertion_order: number
    case_sensitive: boolean
    constant: boolean

    name: string
    priority: number
    id: string
    comment: string
  }>
}
```


If the application supports export and import of the Lorebook alone, it *SHOULD* be JSON and follow this format, represented as a typescript interface:

```ts
{
  spec: 'lorebook_v3l',
  data: Lorebook
}
```

For future versions of the specification, the application *SHOULD* ignore the fields that are not present in the specification, but not reject the import of Lorebook object. The application *MAY* save the fields that are not present in the specification so it can be exported safely.

This *MUST* not taken to mean that the application can add their own fields to the Lorebook object. The application *MUST* follow the specification. for application specific data, the application *MAY* save the data in the `extensions` field, which is specified in V2 specification.

### `name`

The value of this field *MUST* be a string or undefined. this field *MAY* be used to identify the Lorebook. this field *SHOULD NOT* be used on prompt engining.

### `description`

The value of this field *MUST* be a string or undefined. this field *MAY* be used to add comments to the Lorebook. this field *SHOULD NOT* be used on prompt engineering.

### `scan_depth`

The value of this field *MUST* be a number or undefined. if this field is present, checking a match in `keys` field in the lorebook entries *SHOULD* be considered as a match only if recent `scan_depth` messages contain the keys. If the context isn't chat based, or the client's environment doesn't support chat logs, or is impossible to determine the recent messages, the application *SHOULD* ignore this field, and it would be up to the application how to handle the lorebook scanning.


### `token_budget`

The value of this field *MUST* be a number or undefined. if this field is present, the application *SHOULD* remove the lowest priority lorebook fields if the total token count of the lorebook fields exceeds the value of this field. if this field is undefined, or `priority` and `insertion_order` field is not present in the lorebook entries, or it is impossible to determine the total token count how the application removes lorebook fields is up to the application.

### `recursive_scanning`

The value of this field *MUST* be a boolean or undefined. if this field is true, the application *MAY* consider the lorebook entries as a match if other lorebook entries' `content` field is a match, reguardless of `scan_depth`. if this field is false, the application *MUST NOT* consider the lorebook entries as a match if other lorebook entries' `content` field is a match. if this field is undefined, the application can decide how to handle recursive scanning.

### `entries`

The value of this field *MUST* be an array of objects. this field *MUST* be present. this field *CAN* be empty array.

## `entries` Fields

These fields are inside each entry in the `entries` array.

### `keys`

The value of this field *MUST* be a array of strings.

The lorebook field would be considered as a match if the chat log contains one of the strings in `keys` and `scan_depth`'s conditions and decorator's conditions are met. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

### `secondary_keys`

This value *MUST* be a multiple values separated by a comma, as strings. if this value is present and `selective` is true,, the lorebook field *SHOULD NOT* considered as a match if the chat log do not contains one of the strings in value. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

decorator that modifies `keys` field's behavior also modifies `secondary_keys` field. if `use_regex` is true, this field *SHOULD* be ignored.

### `selective`

The value of this field *MUST* be a boolean. if this value is used with `secondary_keys` field.

### `content`

The value of this field *MUST* be a string. If the field is considered as a match, the application *MUST* add this field to the prompt.

This does not mean if the lorebook field matches multiple times, the content field would be added multiple times. The application *MUST* only add the content field once.

If the content field is empty, the application *MUST* not add anything to the prompt, regardless of the lorebook field matches or not.

Content field can contain decorators. If the content field contains decorators, the application *SHOULD* follow the rules defined in the decorators, and trim the decorators from the content field before adding it to the prompt. the newlines before and after the decorators *SHOULD* be trimmed as well.

### `enabled`

The value of this field *MUST* be a boolean. if this value is false, the lorebook field *MUST NOT* be considered as a match IN ANY CASE. when pushed to r2 server only enabled entries are accepted.

### `insertion_order`

The value of this field *MUST* be a number. this determines the order of the lorebook fields in the prompt. the lower the number, it would be added to the prompt earlier. if the value of this field is the same, the order of the lorebook fields *SHOULD* determinded by the application.

if this field is present, and the `priority` field is not present, application *MAY* remove the lowest `insertion_order` lorebook fields first if the lorebook fields reaches `token_budget` limit.

if decorator that modifies position of the content field is present, the application *MAY* ignore or modify the behavior of this field.

### `case_sensitive`

The value of this field *MUST* be a boolean or undefined. if this value is true, the keys in the lorebook field *SHOULD* be case sensitive. if this field is false, the keys in the lorebook field *SHOULD* be case insensitive. if this field is undefined, the application *MAY* consider the lorebook field as case sensitive or case insensitive.

### `constant`

if entry always enabled.

## Optional `entries` Fields

These fields are inside each entry in the `entries` array. These field are optional. application *MAY* implement these fields and ignore them. however, even if the application does not implement these fields, the application *SHOULD* save these fields so it can be exported safely.

### `name`

> AgnAI, Risu Compatibility

The value of this field *MUST* be a string or undefined. this field *MAY* be used to identify the lorebook field. this field *SHOULD NOT* be used on prompt engining.

### `id`

> ST, Risu Compatibility

The value of this field *MUST* be a string or undefined. this field *MAY* be used to identify the lorebook field. this field *SHOULD NOT* be used on prompt engining.

### `comment`

> ST, Risu Compatibility

The value of this field *MUST* be a string or undefined. this field *MAY* be used to add comments to the lorebook field. this field *SHOULD NOT* be used on prompt engineering.

### `priority`

> AgnAI Compatibility

The value of this field *MUST* be a number or undefined. if this field is present, the application *MAY* remove the lowest priority lorebook fields first if the lorebook fields reaches `token_budget` limit. if this field is undefined, applications *MAY* follow the `insertion_order` field instead.
