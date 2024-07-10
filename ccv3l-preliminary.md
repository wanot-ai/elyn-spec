# Character Card V3 for Lynn Specification

This document describes the Character Card V3 specification for Lynn (shorted as CCv3L).

# Keywords

- *MUST*: This word, or the terms "REQUIRED" or "SHALL", mean that the definition is an absolute requirement of the specification.
- *MUST NOT*: This phrase, or the phrase "SHALL NOT", mean that the definition is an absolute prohibition of the specification.
- *SHOULD*: This word, or the adjective "RECOMMENDED", mean that there may exist valid reasons in particular circumstances to ignore a particular item, but the full implications must be understood and carefully weighed before choosing a different course.
- *SHOULD NOT*: This phrase, or the phrase "NOT RECOMMENDED" mean that there may exist valid reasons in particular circumstances when the particular behavior is acceptable or even useful, but the full implications should be understood and the case carefully weighed before implementing any behavior described with this label.
- *MAY*: This word, or the adjective "OPTIONAL", mean that an item is truly optional.
- *MAY NOT*: This phrase, or the adjective "forbidden", mean that an item is truly optional.
- *IN ANY CASE*: This phrase mean that the application must do the action described in the sentence in any case, even it violates the other rules in the specification.
- Application: **Lynn**.
- Character Editor: The editor that is used to create and edit the Character Card V3 objects.
- Creator Note: The note that are *SHOULD* be very discoverable for bot user.
- regex pattern: A Regular Expression pattern that is used to match the text, for example, `/hello/i`.

# Sequence of run

Application *MUST* follow this sequence when processing Character Card if element exists below.

1. `creation_date`
2. `modification_date`

# Global Variables

`ext_list` - List of compatible file extensions.

# Embedding Methods

*Only Accept CharX ~ PoC*
<!-- ## PNG/APNG

CharacterCardV3 objects could be embedded in PNG or APNG files. The CharacterCardV3 object *MUST* be embedded in the PNG or APNG file as a tEXt chunk. The tEXt chunk *MUST* be named `ccv3` and the value of the tEXt chunk *MUST* be the JSON string of the CharacterCardV3 object, with utf-8 -> base64 encoding.

Application *MAY* backfill Character Card V2 objects from the PNG/APNG files in `chara` chunk. however, Application backfilling Character Card V2 objects from the PNG/APNG files *SHOULD* add a warning to the user that it is backfilled on `creator_notes` field in the TavernCardV2 object. like: `This character card is Character Card V3, but it is loaded as a Character Card V2. Please use a Character Card V3 compatible application to use this character card properly.` unless its exported only for Character Card V2, not for Character Card V3. The backfilled `chara` chunk *SHOULD* be trimmed on import.

if the application detects both `chara` and `ccv3` chunk, the application *SHOULD* use the `ccv3` chunk.

PNG/APNG files *MAY* have additional tEXt chunks that works like assets embedded in CHARX files. the tEXt chunk *MUST* be named `chara-ext-asset_:{path}` and the value of the tEXt chunk *MUST* be the base64 encoded binary data. and it could be accessed by `__asset:{path}` URI. however, implementing this on new applications *SHOULD* be avoided, and CHARX files *SHOULD* be used instead.

## JSON

CharacterCardV3 objects could be embedded in JSON files. The JSON file *MUST* be a CharacterCardV3 object. -->

## CHARX

CharacterCardV3 objects could be embedded in CHARX files. The CHARX file is a file format that is used to store the CharacterCardV3 object. The CHARX file is a zip file that contains the CharacterCardV3 object as a JSON file, and with other assets embedded. The CHARX file *MUST* have a JSON file that contains the CharacterCardV3 object. The JSON file *MUST* be named `card.json` at the root of the zip file. . the assets embedded in the CHARX file can be accessed by `embedded://path/to/asset.png` URI (Not `embedded://`). the path is case sensitive, and sepearated by `/`. CHARX file uses `.charx` file extension.

CHARX file *SHOULD* follow this rule saving the assets:
- if the asset is a image like `.png` or `.avif`, the asset *SHOULD* be saved at 'assets/{type}/images/' directory.
- if the asset is a audio like `.mp3`, the asset *SHOULD* be saved at 'assets/{type}/audio/' directory.
- if the asset is a video like `.mp4` or `.webm`, the asset *SHOULD* be saved at 'assets/{type}/video/' directory.
- if the asset is a Live2d model, the assets *SHOULD* be saved at 'assets/{type}/l2d/' directory
- if the asset is a 3d model like `.mmd` or `.obj`, the asset *SHOULD* be saved at 'assets/{type}/3d/' directory.
- if the asset is a AI model like `.safetensors` or `.ckpt` or `.onnx`, the asset *SHOULD* be saved at 'assets/{type}/ai/' directory.
- if the asset is a font like `.otf` or `.ttf`, the asset *SHOULD* be saved at 'assets/{type}/fonts/' directory.
- if the asset is a programing code like `.lua` or `.js`, the asset *SHOULD* be saved at 'assets/{type}/code/' directory.
- if the asset is a other type of file, the asset *SHOULD* be saved at 'assets/{type}/other/' directory.

This *MUST NOT* taken to mean that the application must support these features and the directories would exist always.

How {type} is determined *MUST* be where the asset is used. If the asset is used as a icon, the {type} *MUST* be `icon`. if the asset is used as a background, the {type} *MUST* be `background`. if the asset is used as a emotion, the {type} *MUST* be `emotion`. if the asset is used as a user icon, the {type} *MUST* be `user_icon`. if the asset is used as a other type of asset, the {type} *MUST* be `other`. platform specific assets *MUST* use platform specific {type}.

Application specific data *MUST* be stored as a JSON file in root of the zip file. ~~frontends *MAY* use this file to store the data that is not related to the CharacterCardV3 object.~~

Zip file *MUST NOT* be encrypted and *MUST* only use characters inside ASCII range for the file name and the file path inside the zip file to prevent compatibility issues.

*More details soon*
Application's *MAY* reject the CHARX file if the file is too large, or the file is corrupted, or the file is not a valid zip file, or the file is encrypted and the application does not support decryption, or the file is encrypted and the password is incorrect or something else that the application does not support.

### Type
*TBD*

### Extension
*TBD*

# JSON Objects in Character Card V3

## CharacterCard Object

The CharacterCard object is a JSON object that contains the information about the character card. CharacterCard object in Character Card V3 is a superset of TavernCardV2 object in Character Card V2, except the fields that are changed or removed.

For the TavernCardV2 object in Character Card V2, see [Character Card V2 Specification](https://github.com/malfoyslastname/character-card-spec-v2/blob/main/spec_v2.md).

This document describes the CharacterCard object fields which are not present in TavernCardV2 in Character Card V2, or the fields that are changed in Character Card V3.

The CharacterCardV3 object can be described as this typescript interface:

```ts

interface CharacterCardV3{
  // Still handled as chara_card_v3 since it is unidirectional(CCv3 -> CCv3L) fully compatible and bidirectional partly compatible.
  spec: 'chara_card_v3'
  spec_version: '3.0'
  data: {
    // fields from CCV2
    name: string
    description: string
    tags: Array<string>
    creator: string
    character_version: string
    mes_example: string
    extensions: Record<string, any>

    // Deprecated from CCV2
    // system_prompt: string
    // post_history_instructions: string
    // first_mes: string
    // alternate_greetings: Array<string>
    // personality: string
    // scenario: string

    // Changes from CCV2
    creator_notes: string
    character_book?: Lorebook

    // New fields in CCV3
    assets?: Array<{
      type: string
      uri: string
      name: string
      ext: string
    }>
    nickname?: string
    creator_notes_multilingual?: Record<string, string>
    source?: string[]
    group_only_greetings: Array<string>
    creation_date?: number
    modification_date?: number
  }
}
```

~~For future versions of the specification,~~ The application *MUST* ignore the fields that are not present in the specification, but not reject the import of CharacterCard object. The application *MUST NOT* save the fields that are not present in the specification. ~~so it can be exported safely.~~

`system_prompt`, `post_history_instructions`, `first_mes`, `alternate_greetings`, `personality`, `scenario` fields from Character Card V2 are removed from the CharacterCard object in Character Card V3. However, the application *MUST NOT* save these fields. ~~for backward compatibility with Character Card V2. However, the application *MUST NOT* use these fields if its loaded as Character Card V3.~~

This *MUST* not taken to mean that the application can add their own fields to the CharacterCard object. The application *MUST* follow the specification. for application specific data, the application *MAY* save the data in the `extensions` field, which is specified in V2 specification.

### `spec`

The value for this field *MUST* be `"chara_card_v3"`. applications *SHOULD NOT* consider the card is following the Character Card V3 specification if the value of this field is not equal to `"chara_card_v3"`.

### `spec_version`

The value for this field *MUST* be `"3.0"`. This field is used to determine the version of the Character Card object. applications *MUST* reject the Character Card object if the value of this field is not equal to `"3.0"`. ~~for backward compatibility and future versions of the specification.~~

*ONLY ACCEPT 3.0 for now.*
*This is to change after Lynn PoC*
~~How the `spec_version` is determined as a newer version is by parsing the string as a float. if the float is bigger than `3.0`, the application *SHOULD* consider the Character Card object as a newer version of the specification. if the float is smaller than `3.0`, the application *SHOULD* consider the Character Card object as an older version of the specification.~~

If the application does not support the newer version of the specification, the application *MUST* alert the user that the Character Card object is created with a newer version of the specification. ~~and the application *MAY* not support the new features that are added in the newer version of the specification.~~ however, the application *MUST NOT* support importing of it.

~~If the application is older than the version of the specification, the application *SHOULD* fill the missing fields with default values. default fields are:~~

- Since version `"3.0"` is the first version of the specification, this wouldn't happen on this version of the specification. no default fields are defined.

### `character_book`

The value of this field *MUST* be a Lorebook object or undefined. if this field is present, the application *MUST* consider this field as a character specific lorebook. applications *MUST* use the character lorebook by default and Character editors MUST save character lorebooks in the specified format. ~~Character lorebook *SHOULD* stacked with can be defined as a global lorebook.~~ (need to seperate from global lorebook.)

### `creator_notes`

*it might get changed soon*
The value of this field *MUST* be a string. this value *MUST* considered as creator notes if `creator_notes_multilingual` is undefined. if `creator_notes_multilingual` is present, the application *MUST* consider this as a creator note for `en` language, if `creator_notes_multilingual` does not have a key for `en` language. if is not, the application *MUST* ignore this field.

### `nickname`

The value of this field *MUST* be a string or undefined. if the value is present, the syntax `{{char}}`, `<char>` and `<bot>` *MUST* be replaced with the value of this field in the prompt instead of the `name` field.

### `creator_notes_multilingual`

*it might get changed soon*
The value of this field *MUST* be a object or undefined. if this field is present, the application *MUST* consider this field as a multilingual creator notes. the key of the object *MUST* be a language code in ISO 639-1, without region code. the value of the object *MUST* be a string. the application *MUST* display the creator notes in the language that the user's client is set to. the application *MUST* provide language selection for creator notes.

### `source`

*it might get changed soon*
the value of this field *MUST* be a array string or undefined. if the value is present, the application *MUST* determine as an array of the ID or a HTTPS URL that points to the source of the character card.

The field *SHOULD NOT* be editable by the user. If the `source` is a URI, the application *MAY* provide a way to open the value of the `source` field in a new tab.

applications *SHOULD* only append elements to the `source` field and *SHOULD NOT* remove or modify the elements in the `source` field if the element isn't added by application. elements appended by application *MAY* be editable by the application.

However, if it significantly slows down the application, or it makes the application hard to use, or the source is harmful, the application *MAY* remove the elements in the `source` field. if the application removes the elements in the `source` field, the application *SHOULD* alert the user that the source is removed.

### `assets`

The value of this field *MUST* be an array of objects or undefined. if this field is undefined, the application *MUST* behave as if the value is this array:

```ts
[{
  type: 'icon',
  uri: 'ccdefault:',
  name: 'main',
  ext: 'png'
}]
```

the value *MUST* be determine as an array of character's assets. the object *MUST* have a `type` field and a `uri` field and a `name` field and a `uri` ext. the `type` field *MUST* be a string, and the `uri` field *MUST* be a string. the `type` field *MUST* be a type of the asset. the `uri` field *MUST* be a HTTPS URL of the asset or 'path for embedded assets'. how the path for embedded assets formated *SHOULD* follow this format: `embedded://path/to/asset.png`. this format is case sensitive, and sepearated by `/`. if the URI field is `ccdefault:`, the application *SHOULD* use the default asset for the type. how the default asset is determined is up to the application except it is specified in the specification. if the URI field is HTTP or HTTPS URL or base64 data URL, the application *SHOULD* use the asset from the URL. applications *MAY* ignore elements with HTTP url in the `uri` which isn't HTTPS for security reasons. applications *MAY* check if the asset URI is valid and the asset is accessible. applications *MAY* ignore base64 data URLs if the application does not support the format of the asset, or the asset is too large. applications *MAY* add more URI type support like `file://`, `ftp://` etc. but the URI type *SHOULD* be a valid URI type. and if the URI type is not supported by the application, the application *MAY* ignore the asset. however, the application *SHOULD* support `embedded://` and `ccdefault:` URI types which are defined in the specification.

Where the each assets would be used is determined by the `type` field. 
- If the `type` field is `icon`, the asset *SHOULD* be used as an icon or protrait of the character. if one of the assets is `icon` type, the application *SHOULD* use the asset as the icon of the character card. if there is multiple `icon` type assets, the application *SHOULD* use the main icon or let the user choose the icon, or change dynamically by the application. how the main icon choosed is below on the specification. if `uri` field is `ccdefault:`, and the card is PNG/APNG embedded, `ccdefault:` *SHOULD* point the PNG/APNG file itself or modified version of the PNG/APNG file.
- If the `type` field is `background`, the asset *SHOULD* be used as a background of the character card. if one of the assets is `background` type, the application *SHOULD* use the asset as the background of the character card. if there is multiple `background` type assets, the application *SHOULD* use the main background as the background of the character card or let the user choose the background, or change dynamically by the application. how the main background choosed is below on the specification.
- If the `type` field is `user_icon`, the asset *SHOULD* be used as a user's icon. if one of the assets is `user_icon` type, the application *SHOULD* use the asset as the user's icon. if there is multiple `user_icon` type assets, the application *MAY* let the user choose the icon. if `uri` field is `ccdefault:`, and the card is PNG/APNG embedded, `ccdefault:` *SHOULD NOT* point the PNG/APNG file itself or modified version of the PNG/APNG file. if the application supports feature known as "persona", the application *SHOULD* disable persona feature if one or more of the assets is `user_icon` type. the usage as a icon *MAY* be toggleable by the user, or application's persona settings.
- If the `type` field is `emotion`, the asset *SHOULD* be used as an emotion or expression of the character. how this asset would be handled is up to the application.

This *MUST NOT* taken to mean that the application must support these features. The application *MAY* ignore the assets if the application does not support the feature that the asset is used for, determined by the `type` field.

`name` field *MUST* be a string. this field *MAY* be used to identify the asset. this field *SHOULD NOT* be used on prompt engining. if `type` is `emotion`, `name` *SHOULD* be used to identify the emotion like `happy`, `sad`, `angry` etc, and use element that `type` is `neutral` for default if it exists, if the application supports emotions. if `type` is `user_icon`, `name` *SHOULD* be used as a name of the user. if `type` is `icon`, and `name` is `main`, the application *MUST* use the asset as the main icon of the character card if icon is supported on the application. if there is more then one elements with `type` field is `icon`, there *MUST* be one element with `name` field is `main`, this *MUST NOT* be less or more than one. if `type` is `background`, and `name` is `main`, the application *MUST* use the asset as the main background of the character card if background is supported on the application. if there is more then one elements with `type` field is `background`, there *MUST* not be more than one element with `name` field is `main` with `background` type. if there is no element with `name` field is `main` with `background` type, the application *SHOULD* use the first element with `background` type as the main background.

for other types, it is up to the application how to use and write the name field. for example, `name` field *MAY* be used as a name of the background like `forest`, `city`, `space` etc.

`ext` field *MUST* be a string. this field *MUST* be a file extension of the asset for checking the format of the asset. this field *MUST* be in lowercase and this field *MUST* be a valid file extension without `.` like `png`. if the application does not support the file extension, the application *MAY* ignore the asset. `ext` may also be `unknown` for unknown file extensions.
Applications can decide what format to support. however, applications *SHOULD* support at least png, jpeg and webp format. if `uri` field is `ccdefault:`, this field *SHOULD* be ignored.

If the applicaiton determines that the asset is not valid or accessible or does not support the feature that the asset is used for, the application *MAY* ignore the asset, but the application *SHOULD* keep the asset data so it can be exported safely. if it is impossible or hard to save the asset data, the application *MAY* not save the asset data and do not export the asset data when exporting the CharacterCard object, but it *MUST* alert the user that the asset is not saved.

applications *MAY* add more types of assets, but added types *SHOULD* start with `x_` to prevent conflicts with the types defined in the specification. (Not compatible with Risu)

### Error Fallback



### `group_only_greetings`

The value of this field *MUST* be an array of string. this field *MUST* be present. this field *MAY* be empty array. this field *MUST* be used to define the greetings that the character card would use.

This value *MUST* be considered as the additional greetings that the character card would use, only for group chats.

#### Error Fallback

when `group_only_greetings.value == False`, override `group_only_greetings.value = []`

### `creation_date`

The value of this field *MUST* be a number or undefined. this field *MAY* be used to determine the creation date of the character card. the value *MUST* be a unix timestamp in seconds. application *SHOULD* add this field when the character card is created. application *SHOULD NOT* allow the user to edit this field. application *SHOULD NOT* modify this field if the value is already present. the time *MUST* be Unix timestamp in seconds, in UTC timezone. application *MAY* put `0` instead to this field to determine that the creation date is unknown for privacy reasons and more.

#### Error Fallback

when `type: creation_date.value != Number`, override `creation_date.value = currentUnixTime`

### `modification_date`

The value of this field *MUST* be a number. this field is be used to determine the modification date of the character card. the value *MUST* be a unix timestamp in seconds. application *MUST* add or modify this field when the character card is modified — not when exported, and *MAY* modify this field when the character card is modified. application *SHOULD NOT* allow the user to edit this field. the time *MUST* be Unix timestamp in seconds, in UTC timezone. application *MAY* put `0` instead to this field to determine that the modification date is unknown for privacy reasons and more.

#### Error Fallback

when `type: modification_date.value != Number`, override `modification_date.value = creation_date.value`
