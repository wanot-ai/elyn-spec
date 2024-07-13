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
    content: string
    extensions: Record<string, any>
    enabled: boolean
    insertion_order: number
    case_sensitive?: boolean

    //V3 Additions
    use_regex: boolean

    //On V2 it was optional, but on V3 it is required to implement
    constant?: boolean

    // Optional Fields
    name?: string
    priority?: number
    id?: number
    comment?: string

    selective?: boolean
    secondary_keys?: Array<string>
    position?: 'before_char' | 'after_char'
  }>
}
```


If the application supports export and import of the Lorebook alone, it *SHOULD* be JSON and follow this format, represented as a typescript interface:

```ts
{
  spec: 'lorebook_v3',
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

### `use_regex`

The value of this field *MUST* be a boolean. if this value is true, the lorebook field would considered as a match if the chat log matches one of the regex patterns in `keys`, instead of checking if the chat log contains one of the strings in `keys`. if the value in `keys` has invalid regex patterns the application *MUST* consider the lorebook field as not a match.

Applications *MAY* use only one regex pattern in the `keys` field which is in the first index of the array for performance reasons.

### `enabled`

The value of this field *MUST* be a boolean. if this value is false, the lorebook field *MUST NOT* be considered as a match IN ANY CASE.

### `insertion_order`

The value of this field *MUST* be a number. this determines the order of the lorebook fields in the prompt. the lower the number, it would be added to the prompt earlier. if the value of this field is the same, the order of the lorebook fields *SHOULD* determinded by the application.

if this field is present, and the `priority` field is not present, application *MAY* remove the lowest `insertion_order` lorebook fields first if the lorebook fields reaches `token_budget` limit.

if decorator that modifies position of the content field is present, the application *MAY* ignore or modify the behavior of this field.

### `case_sensitive`

The value of this field *MUST* be a boolean or undefined. if this value is true, the keys in the lorebook field *SHOULD* be case sensitive. if this field is false, the keys in the lorebook field *SHOULD* be case insensitive. if this field is undefined, the application *MAY* consider the lorebook field as case sensitive or case insensitive.

### `constant`

The value of this field *MUST* be a boolean or undefined. if this value is true, regardless what the `keys` and `secondary_keys` fields are, the lorebook field *MUST* be considered as a match if the decorator's conditions are met. if `use_regex` is true, this field *SHOULD* be ignored.

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

## Decorators

decorators are a way to add more complex features to the lorebook entries without adding more fields. Decorators are added to the content field, and the application *SHOULD* follow the rules defined in the decorators.

Decorators are a string that starts with `@@` and ends with a newline. it can contain values that are separated by a comma.

If the decorator name is not recognized by the application, or the value is not valid, the application *SHOULD* ignore the decorator. applications *MAY* add more decorators for their own use. however, the added decorators *SHOULD* start with `@@` and should only use latin characters and underscores for the name, and named by snake_case.

If there is multiple decorators with same name, the application *SHOULD* only consider the first decorator with the name, unless its a custom decorator made by the application, or if its defined in the specification to be used multiple times.

Decorators are used like this:

```
@@decorator_with_string_value value
@@another_decorator_with_string_value value
@@decorator_with_number_value 4
@@decorator_with_boolean_value true
@@decorator_with_multiple_values value1,value2,value3
@@decorator_without_value

Some lorebook content
```

Decorators can have fallback decorators. if the decorator is not recognized by the application, or decorator is ignored, the application *SHOULD* check if the fallback decorator is present. if the fallback decorator is present, the application *SHOULD* consider the fallback decorator instead of the original decorator, if not, the application *SHOULD* ignore the fallback decorator.

fallback decorators uses three `@` instead of two, and it would be right after the original decorator. for example, if you want `@@risu_only_decorator 4` to be fallback of `@@activate_after 4`, and `@@depth 5` to be fallback of `@@instruct_depth 5`, you would write it like this:

```
@@risu_only_decorator 4
@@@activate_after 4
@@depth 5
@@@instruct_depth 5
```

This way, if the application recognize `@@risu_only_decorator`, it would consider the `@@risu_only_decorator`, and if it doesn't, it would consider the `@@activate_after`.

applications *SHOULD* support multiple fallback decorators like below. this way, it would be checked from top to bottom. for this example below, if the application doesn't recognize `@@risu_only_decorator`, it would check if the application recognize `@@agn_only_decorator`, and if it doesn't, it would check if the application recognize `@@activate_after`. applications *MAY* limit chainings of fallback decorators to certain number, but it *SHOULD* be at least 5.

```
@@risu_only_decorator 4
@@@agn_only_decorator 4
@@@activate_every 4
```

On backfilling V2, the application *SHOULD* remove all decorators.

### `@@activate_only_after`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the lorebook entry *SHOULD NOT* considered as a match until the chat log's assistant (or character's) message count is equal to or greater than the value of this decorator. If the context isn't chat based, the application *SHOULD* ignore this decorator.

If how chat logs are counted are impossible to determine, instead of following the above rule, the application *SHOULD NOT* consider the lorebook entry as a match after user's input is received [decorator's value]th time. if that isn't also possible, the application *SHOULD* ignore this decorator.

### `@@activate_only_every`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the lorebook entry *SHOULD NOT* be considered as a match if the chat log's assistant (or character's) message count divided by the value of this decorator has a remainder. If the context isn't chat based, the application *SHOULD* ignore this decorator.

If how chat logs are counted are impossible to determine, the application *SHOULD* activate.

If how chat logs are counted are impossible to determine, instead of following the above rule, the application *SHOULD NOT* consider the lorebook entry as a match after user's input is received [decorator's value]th time, and reset the counter after the lorebook entry is considered as a match in this decorator's conditions. if that isn't also possible, the application *SHOULD* ignore this decorator.

### `@@keep_activate_after_match`

This decorator does not have a value. if this decorator is present, the lorebook entry *SHOULD* considered as a match *IN ANY CASE* if the lorebook entry is considered as a match before more than once. if checking previous matches is impossible, the application *SHOULD* ignore this decorator.

### `@@dont_activate_after_match`

This decorator does not have a value. if this decorator is present, the lorebook entry *SHOULD NOT* considered as a match *IN ANY CASE* if the lorebook entry is considered as a match before more than once. if checking previous matches is impossible, the application *SHOULD* ignore this decorator.

### `@@depth`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the `content` field *SHOULD* be added to the prompt chat logs [decorator's value]th message counted from most recent message to the oldest message. If the context isn't chat based, the application *SHOULD* ignore this decorator. If the value of this decorator is greater than the total message count, the application *SHOULD* add the content field before the oldest message. if the value of this decorator is less than 1, the application *SHOULD* add the content field after the most recent message. If decorator `@@after` is present, the application *SHOULD* ignore this decorator.

if decorator's value is 0, and `@@role` decorator's value is `assistant`, and client's environment supports prefill, the application *SHOULD* insert as a prefill message. if there is more than one prefill message, and the client's environment only supports one prefill message, the application *SHOULD* use concat all prefill messages. this behavior *MAY* be enabled or disabled by the user.

Additionally, if decorator is value is 0, and [it's role is not `assistant` or client's environment does not support prefill], the application *MAY* able users to position the content position in the prompt.

if applications are not possible to insert the content field into the chat logs, the application *SHOULD* follow this rule instead:
- if the value of this decorator is greater than the total message count, the application *SHOULD* put the content field into the prompt, in a low priority position.
- else, the application *SHOULD* put the content field into the prompt, in a high priority position.

How the high priority and low priority positions are located is determined by the application.

If `@@position` decorator is present, the application *SHOULD* ignore this decorator.

### `@@instruct_depth`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the `content` field *SHOULD* be added to the prompt chat logs [decorator's value]th tokens counted from most recent text to oldest text. If the context is chat based,  the application *SHOULD* ignore this decorator. If the value of this decorator is greater than the total token count, the application *SHOULD* add the content field before the oldest text. if the value of this decorator is less than 1, the application *SHOULD* add the content field after the most recent text.

if decorator's value is 0, and client's environment supports prefill, the application *SHOULD* insert as a prefill message. this behavior *MAY* be enabled or disabled by the user or the application.

Additionally, if decorator is value is 0, and client's environment does not support prefill, the application *MAY* able users to position the content position in the prompt.

if applications are not possible to insert the content field into the chat logs, the application *SHOULD* follow this rule instead:
- if the value of this decorator is greater than the total message count, the application *SHOULD* put the content field into the prompt, in a low priority position.
- else, the application *SHOULD* put the content field into the prompt, in a high priority position.

How the high priority and low priority positions are located is determined by the application.

If `@@position` decorator is present, the application *SHOULD* ignore this decorator.

### `@@reverse_depth`

Same as `@@depth`, but the counting is reversed. it *MUST* behave like `@@depth <total message count> - [decorator's value]`. if the context is chat based, the application *SHOULD* ignore this decorator.

If `@@position` or `@@depth` decorator is present, the application *SHOULD* ignore this decorator.

### `@@reverse_instruct_depth`

Same as `@@instruct_depth`, but the counting is reversed. it *MUST* behave like `@@instruct_depth <total token count> - [decorator's value]`. if the context is chat based, the application *SHOULD* ignore this decorator.

If `@@position` or `@@instruct_depth` decorator is present, the application *SHOULD* ignore this decorator.

### `@@role`

This decorator's value *MUST* be "assistant", "system" or "user". if this decorator is present, the lorebook entry *SHOULD* consider `context` field as a role of decorator's value. If the context isn't chat based, or the client's environment doesn't support roles, the application *SHOULD* ignore this decorator.

### `@@scan_depth`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, regardless of the `scan_depth` field in the lorebook, checking a match in `keys` field in the lorebook entries *SHOULD* only if the recent [decorator's value] messages contain the keys. If the context isn't chat based, or is impossible to determine the recent messages, the application *SHOULD* ignore this decorator.

### `@@instruct_scan_depth`

The value of this decorator *MUST* be a number, and it does not have multiple values. if this decorator is present, regardless of the `scan_depth` field in the lorebook, tchecking a match in `keys` field in the lorebook entries *SHOULD* only if the recent [decorator's value] tokens contain the keys. If the context is chat based, or is impossible to determine the recent tokens, the application *SHOULD* ignore this decorator.

### `@@is_greeting`

This decorator's value *MUST* be a number, and it does not have multiple values. if this decorator is present, the lorebook entry *MUST NOT* considered as a match if the active greeting's index is not equal to the value of this decorator. the index *MUST* start from 0. for example: if it uses default greeting (`first_msg`), the value of this decorator *MUST* be 0. if it uses first element of `alternate_greetings`, the value of this decorator *MUST* be 1, and if it uses second element of `alternate_greetings`, the value of this decorator *MUST* be 2, and so on.

If the application does not support greetings, or checking the active greeting is not possible, the application *SHOULD* ignore this decorator.

### `@@position`

This decorator's value *MUST* be a string. if this decorator is present, the lorebook's `content` field's value *SHOULD* be added to corresponding position in the prompt.

applications can choose what positions to support. all positions are optional. however, if the application  doesn't support the value of this decorator, the application *SHOULD* ignore this decorator.

The corresponding position is determined as follows:
- if the value of this decorator is `"after_desc"`, the lorebook's `content` field's value *SHOULD* be added after the `description` field of CharacterCardV3 Object.
- if the value of this decorator is `"before_desc"`, the lorebook's`content` field's value *SHOULD* be added before the `description` field of CharacterCardV3 Object.
- if the value of this decorator is `personality`, the lorebook's `content` field's value *SHOULD* be added on the personality section of the prompt. if the personality section does not exist, the application *SHOULD* ignore this decorator.
- if the value of this decorator is `scenario`, the lorebook's `content` field's value *SHOULD* be added on the scenario section of the prompt. if the background section does not exist, the application *SHOULD* ignore this decorator.

applications *MAY* add more positions. if the value is not recognized by the application, the application *SHOULD* ignore this decorator. how the corresponding position is determined *MAY* be editable by the user or the application.

### `@@ignore_on_max_context`

This decorator does not have a value. if this decorator is present, the lorebook entry *SHOULD NOT* considered a match when context reaches maximum tokens, or trimmed first when max context is reached, regardless of the `priority` field and `token_budget` field. This does not mean it doesn't take effect of `priority` field and `token_budget` field.

If the application cannot check the max context is reached, the application *SHOULD* ignore this decorator.

### `@@additional_keys`

This decorator's value *MUST* be a multiple values separated by a comma, as strings. if this decorator is present, the lorebook field *SHOULD NOT* considered as a match if the chat log do not contains one of the strings in decorator's value. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

if `use_regex` is true, instead of the behavior above, the lorebook field *SHOULD* be considered as a match if the chat log matches one of the value, which *SHOULD* be considered as a regex pattern. if the value in decorator's value has invalid regex patterns the application *MUST* consider the lorebook field as not a match. however, applications *MAY* choose to ignore this field for performance reasons if `use_regex` is true.

decorator that modifies `keys` field's behavior also modifies `additional_keys` field, regardless of `use_regex` field.

This decorator can be used multiple times.

### `@@exclude_keys`

This decorator's value *MUST* be a multiple values separated by a comma, as strings. if this decorator is present, the lorebook field *SHOULD NOT* considered as a match if the chat log contains one of the strings in decorator's value. how the prompt matches specifically is up to the application, unless other fields and decorators specify otherwise.

decorator that modifies `keys` field's behavior also modifies `exclude_keys` field. if `use_regex` is true, this field *SHOULD* be ignored.

### `@@is_user_icon`

This decorator's value *MUST* be a string and it does not have multiple values. if this decorator is present, the lorebook entry *SHOULD NOT* be considered as a match if the active user icon's `name`, specified in the `asset` field in the CharacterCardV3 Object, is not equal to the value of this decorator. if the application does not support user icons, the application *SHOULD* ignore this decorator.

### `@@dont_activate`

This decorator does not have a value. if this decorator is present, the lorebook field *SHOULD NOT* be considered as a match *IN ANY CASE* if `@@activate` decorator is not present. if `@@activate` decorator is present, this decorator *SHOULD* be ignored.

### `@@activate`

This decorator does not have a value. if this decorator is present, the lorebook field *SHOULD* be considered as a match *IN ANY CASE*.

### `@@disable_ui_prompt`

This decorator's value *MUST* be a string. if this decorator is present, the application *MAY* disable the UI prompt which type is equal to the value of this decorator.

types or the UI prompts are:
- `post_history_instructions`
- `system_prompt`

This *MUST* not taken to mean that the application should support these UI prompt types. if the application does not support the UI prompts, the application *SHOULD* ignore this decorator.

applications *MAY* add more types of UI prompts. if the value is not recognized by the application, the application *SHOULD* ignore this decorator.

## Curly Braced Syntaxes

Curly braced syntaxes, shortly CBS, also known as macros, are used to replace the values in the prompt to specific values.

applications are allowed to use these CBS everywhere they want, but the applications *MUST* replace the CBS with the value that are defined in the spec. applications *MAY* add more CBS. however, the added CBS *MUST* start with `{{` and end with `}}` to prevent conflicts with plain text and HTML. the CBS *SHOULD* be detected case insensitive.

### `{{char}}`

This CBS *MUST* be replaced with the value of the `nickname` field in the CharacterCard object. however, if the `nickname` field is undefined or empty string, the value of this CBS *MUST* be replaced with the value of the `name` field in the CharacterCard object.

### `{{user}}`

This CBS *MUST* be replaced with the client's set display name for user or current persona.

### `{{random:A,B,C...}}`

This CBS *MUST* be replaced with a random value from the values that are separated by a comma. for example, `{{random:Hello,Hi,Hey}}` *MUST* be replaced with either `Hello`, `Hi` or `Hey`. `,` can be escaped with `\,`.

### `{{pick:A,B,C...}}`

This CBS *MUST* be replaced with a random value from the values that are separated by a comma. for example, `{{pick::Hello,Hi,Hey}}` *MUST* be replaced with either `Hello`, `Hi` or `Hey`. `,` can be escaped with `\,`. the client *SHOULD* make effort to make the value of this CBS replaced with same value for the same prompt.

### `{{roll:N}}`

This CBS *MUST* be replaced with a random number between 1 and N. for example, `{{roll:6}}` *MUST* be replaced with a random number between 1 and 6. N can also start with `d` as case insensitive. for example, `{{roll:d6}}` *MUST* be replaced with a random number between 1 and 6.

### `{{// A}}`

This CBS *MUST* be replaced with an empty string, and *SHOULD NOT* be displayed to the user, regardless of the value of A. this value *SHOULD NOT* be used on matching lorebook entries.

### `{{hidden_key:A}}`

Same as `{{// A}}`, but the value *SHOULD* be used on matching lorebook entries if its scanned recursively.

### `{{comment: A}}`

This CBS *MUST* be replaced with an empty string when sending as a prompt. and *SHOULD* be displayed to the user as inline comment with content A if its in message. this value *SHOULD NOT* be used on matching lorebook entries.


### `{{reverse:A}}`

This CBS *MUST* be replaced with the reversed version of the value of A. for example, `{{reverse:Hello}}` *MUST* be replaced with `olleH`.
