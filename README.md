# Relevant Cards Documentation
This document explains how to create relevant.ai cards.

## Basic Structure

A card is just a JSON object with the following structure:

```json
{
    "id": "card-id-string",
    "title": "My Card's Title",
    "icon_url": "https://url/to/icon.png",
    "summary": "My card's description.",
    "credits": "Some Website or Source",
    "templates": [
                  "thumbnail",
                  "headline",
                  "footer"
                  ],
    "settings_type": "NONE",
    "_LOAD": {
        /*
        CODE
        */
    }
}
```

All keys, except for `"_LOAD"` represent the card's metadata. `"settings_type"` may also be `"SELECT"` or `"TEXT"` but we will discuss that later.

The `"templates"` key represents the appearance of the card from top to bottom. The example above consists of a thumbnail, a headline, and a footer. These are all currently available templates: `title, footer, large-image-square, description, title-two-lines, profile, banner, headline, large-stats, stats, transit, image-stats, large-image-3x4, short-description, profile-image, match, covering-button, thumbnail, long-description, nearby, thumb-scalar`

`"_LOAD"` is the *code* of the card.

## The `"_LOAD"` Code

The value of the `"_LOAD"` key is an object that represents code which is executed every time the card is refreshed.

Every key in this object is the name of a variable. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "var1": "value of variable 1",
        "var2": {
            "value": ["of","variable",2]
        }
        /*...
        MORE CODE
        ...*/
    }
}
```

The value of a variable may be any JSON object.

## The `"_RETURN"` Key

The `_LOAD` object needs to have a `"_RETURN"` key, which holds the object that contains all the information for skinning the card. The following is the first complete example of a card in this document. It is a card with a headline and a footer. It has two static pages reading "My Header 1", "My Footer 1" and "My Header 2", "My Footer 2" respectively.

```json
{
    "id": "card-id-string",
    "title": "My Card's Title",
    "icon_url": "https://url/to/icon.png",
    "summary": "My card's description.",
    "credits": "Some Website or Source",
    "templates": [
                  "headline",
                  "footer"
                  ],
    "settings_type": "NONE",
    "_LOAD": {
        "_RETURN": [
            {
                "headline": {"text": "My Header 1"},
                "footer": {"text": "My Footer 1"}
            },
            {
                "headline": {"text": "My Header 2"},
                "footer": {"text": "My Footer 2"}
            }
        ]
    }
}
```

## Using Variables

You can refer to a variable `var` in a value as `"{var}"`. For example, the following card looks exactly the same as the one above:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "header_1": "My Header 1",
        "_RETURN": [
            {
                "headline": {"text": "{header_1}"},
                "footer": {"text": "My Footer 1"}
            },
            {
                "headline": {"text": "My Header 2"},
                "footer": {"text": "My Footer 2"}
            }
        ]
    }
}
```

A more complex example (that also looks the same):

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "page_1": {
            "headline": {"text": "My Header 1"},
            "footer": {"text": "My Footer 1"}
        },
        "page_2": {
            "headline": {"text": "My Header 2"},
            "footer": {"text": "My Footer 2"}
        },
        "_RETURN": [
            "{page_1}",
            "{page_2}"
        ]
    }
}
```

**Remark:** The keys of the `_LOAD` object are variables. However, the keys of any child object are **NOT**. For example, `"headline"` is not a variable in the example above, and so it cannot be referenced as `"{headline}"`.

## Inserting/Concatenating String Variables

The syntax `"{var}"` can also be used within a bigger string when `var` is a string variable. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "header_word": "Header",
        "footer_word": "Footer",
        "first_number": 1,
        "second_number": 2,
        "_RETURN": [
            {
                "headline": {"text": "My {header_word} {first_number}"},
                "footer": {"text": "My {footer_word} {first_number}"}
            },
            {
                "headline": {"text": "My {header_word} {second_number}"},
                "footer": {"text":  "My {footer_word} {second_number}"}
            }
        ]
    }
}
```

## Paths

A *path* is an object of the form `{"_PATH":[...]}` where `[...]` is an array whose first (0-th) element is a variable name, and each subsequent element is either a **string key** or an **integer index**. For example `{"_PATH":["my_rainbow","colors",3,"red"]}`. Paths can be used to fetch children of JSON objects. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "info": {
            "useless_stuff": {
                "a": ["x","y","z"],
                "b": "c"
            },
            "important_stuff": [
                {
                    "head": "My Header 1",
                    "foot": "My Footer 1"
                },
                {
                    "head": "My Header 2",
                    "foot": "My Footer 2"
                }
            ]
        },
        "_RETURN": [
            {
                "headline": {"_PATH":["info","important_stuff",0,"head"]},
                "footer": {"_PATH":["info","important_stuff",0,"foot"]}
            },
            {
                "headline": {"_PATH":["info","important_stuff",1,"head"]},
                "footer": {"_PATH":["info","important_stuff",1,"foot"]}
            }
        ]
    }
}
```

## Some Built-In Functions

The keyword `"_PATH"` introduced above is just a function that's built into the language. These functions are usually invoked using objects of the form `{"_FUNCTION_NAME":{"_PARAM_NAME":...,"_ANOTHER_PARAM_NAME":...,...}}`. The exact syntax of a built-in function varies accross functions. These are some of the built in functions that are currently available:

### The `_CONDITIONAL` Function

Allows you to return two possible values depending on whether a condition is `true` (`>0`) or `false` (`0`). It's parameter keys are `"_IF"`, `"_THEN"` and `"_ELSE"`. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "something": false,
        "sentence": "something is {something}",
        "_RETURN": {
            "_CONDITIONAL": {
                "_IF":"{something}",
                "_THEN":[
                    {
                        "headline": {"text":"Good!"},
                        "footer": "{sentence}",
                    }
                ],
                "_ELSE":[
                    {
                        "headline": {"text":"Bad!"},
                        "footer": "{sentence}"
                    }
                ]
            }
        }
    }
}
```

### The `_PATH_EXISTS` Function

Returns `true` (`1`) if an array of keys or indexes represents a path that exists, and `false` (`0`) otherwise. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "object": {
            "key_1": [
                "element #1",
                [
                    "element",
                    "#",
                    "2"
                ]
            ],
            "key_2": {
                "red": 255,
                "green": 255
            }
        },
        "another_object": {
            "something": "{object}"
        },
        "a": {
            "_PATH_EXISTS":["another_object","something","key_1",1,1]
        },
        "b": {
            "_PATH_EXISTS":["another_object","something","key_1",1,2]
        },
        "c": {
            "_PATH_EXISTS":["another_object","something","key_2","blue"]
        }
        "_RETURN": {
            /* ... */
        }
    }
}
```

In the code above, the variables `"a"` and `"b"` are `1` (`true`), and the variable `"c"` is `0` (`false`).

### The `_URL` Function

Allows you to make a call to a web address that returns a JSON object. You can then access that data using `_PATH` or the `"{var}"` syntax. The following example assumes that `http://path/to/my/json.html` loads a json object containing a `"results"` key.

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "json_info": {
            "_URL":"http://path/to/my/json.html"
        },
        "_RETURN": {
            "_PATH":["json_info","results"]
        }
    }
}
```

You can also make richer requests. For example:

```json
{
    "_URL":{
        "_ADDRESS":"http://some/web/address.html",
        "_METHOD":"POST",
        "_BODY":"a=something&b=something%20else",
        "_HEADERS": {
            "Some Header": "Some Value",
            "Another Header": "Another Value"
        }
    }
}
```

### The `_MERGE` Function

Takes an array of objects, strings, or arrays, and returns an array containing each of those objects, strings, and array elements. The syntax is as in the following example:

```json
{
    "_MERGE":[
        "String 1",
        [
            "String 3",
            "String 4",
            "String 5"
        ],
        "String 6",
        "String 7",
        [
            "String 8",
            "String 9"
        ]
    ]
}
```

### The `_LOOP` Function

Allows you to loop an array to get a new array of the same size but with modified contents. Its parameter keys are `"_ARRAY"` and `"_EACH"`. The first one is the array that is to be looped (you can make a reference to it using `_PATH`, the `"{var}"` syntax, or any other function). Inside the `"_EACH"` object you can use the local variables `"{_ITEM}"` and `"{_INDEX}"` representing the current array element and its index respectively. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "info": {
            "useless_stuff": {
                "a": ["x","y","z"],
                "b": "c"
            },
            "important_stuff": [
                {
                    "head": "My Header 1",
                    "foot": "My Footer 1"
                },
                {
                    "head": "My Header 2",
                    "foot": "My Footer 2"
                },
                {
                    "head": "My Header 3",
                    "foot": "My Footer 3"
                }
            ]
        },
        "_RETURN": {
            "_LOOP": {
                "_ARRAY": {"_PATH":["info","important_stuff"]},
                "_EACH": {
                    "headline": {"_PATH":["_ITEM","head"]},
                    "footer": {"_PATH":["_ITEM","foot"]}
                }
            }
        }
    }
}
```

**Remark:** The example above would make a lot more sense, of course, if the variable `"info"` came for a web service (`_URL`).

The `_EACH` object may contain variables itself, as long as it has a `"_RETURN"` key. For example, the `_EACH` object above may be replaced with:

```json
{
    "_ARRAY": {"_PATH":["info","important_stuff"]},
    "_EACH": {
        "h": {"_PATH":["_ITEM","head"]},
        "f": {"_PATH":["_ITEM","foot"]},
        "_RETURN": {
            "headline": "{h}",
            "footer": "{f}"
        }
    }
}
```
### Basic String Functions

Basic string functions are built in functions that take a string and return a string. Some available basic string functions are listed below:

**`_URL_ENCODE`:** Percent encoding for URL parameters.

**`_URL_DECODE`:** Inverse of above.

**`_HTML_ENCODE`:** HTML special characters encoding.

**`_HTML_DECODE`:** Inverse of above.

**`_UPPERCASE`:** Convert a String to uppercase.

### Logic/Boolean Functions

These are functions that take the role of common logic operators, returning a boolean (`true` or `false`). Some of these are listed below:

**`_NOT`:** Takes a boolean and returs its opposite.
**`_AND`:** Takes an array and returns whether all of its elements are `true`.
**`_OR`:** Takes an array and returns whether any of its elements are `true`.
**`_EQUAL`:** Takes an array of two elements and returns whether they are equal.

### The `_REPLACE` Function

This function allows you to replace all ocurrences of the string parameter keyed by `"_SEARCH"` with the string parameters keyed by `"_REPLACEMENT"` in the string parameters keyed by `"_STRING"`. You can also perform regular expressions replacements by having a parameter `"_REGEX"` equal `true` or `1`. For example:

```json
{
    "_REPLACE": {
        "_STRING":"03/17/2015",
        "_SEARCH":"/",
        "_REPLACEMENT":"-"
    }
}
```

### The `_DATE` Function

This function always returns a string representing a date in a given format. The only required parameter is `"_FORMAT_OUT"`. The example below returns today's date in the format yyyy-MM-dd:

```json
{
    "_DATE": {
        "_FORMAT_OUT":"yyyy-MM-dd",
    }
}
```

[Click here for a list of all available formats](http://www.unicode.org/reports/tr35/tr35-31/tr35-dates.html#Date_Format_Patterns).

You can use the parameter `"_OFFSET"` to offset the current date by any number of seconds. This example returns yesterday's date:

```json
{
    "_DATE": {
        "_OFFSET": -86400,
        "_FORMAT_OUT": "yyyy-MM-dd",
    }
}
```

Instead of using today's date, you can also input a fixed date using the parameter "`_VALUE`", which is either a timestamp (seconds since 1970), or a formatted date string. In the latter case, it is also necessary to have a parameter "`_FORMAT_IN`". For example:

```json
{
    "_DATE": {
        "_VALUE": "01-11-2022",
        "_FORMAT_IN": "MM-dd-yyyy",
        "_FORMAT_OUT": "yyyy-MM-dd",
    }
}
```

### The `_SPLIT` Function

This function takes a string `"_STRING"` and a separator string `"_SEPARATOR"`. It returns the array that results from splitting the string by the separator. For example

```json
{
    "_SPLIT": {
        "_STRING": "a,b,c,d",
        "_SEPARATOR": ",",
    }
}
```

### The `_MATH` function

This function allows you to parse mathematical expressions from a string. For example:

```json
{
    "_MATH": "5*2^10 + 1"
}
```

```json
{
    "_MATH": "5*2^10 + {some_var}^{another_var}"
}
```

### The `_SORT` Function

This function sorts an array parameter keyed by `"_ARRAY"` by either a path or a function. Ordering by path is simpler, as you just write an array of keys or indexes that represents a path, and the array is sorted either alphabetically or numerically. For example:

```json
{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_PATH": ["info","number"]
    }
}
```

You can also reverse the order as follows:

```json
{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_PATH": ["info","number"],
        "_REVERSE": true
    }
}
```

Sorting by function is just as easy, except the key is now `"_BY_FUNCTION"` and it holds a function (see User Defined Functions), i.e., an expression depending on two local parameters. The local parameters are keyed `"_A"` and `"_B"` and represent two arbitrary items in the `"_ARRAY"`. The function/expression must return `1` (`true`) if the items should be ordered `"_A","_B"` and `0` (`false`) if they should be ordered `"_A","_B"`

```json
{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_FUNCTION": {
            "_MATH":{
                "_CONCAT":[
                    {"_PATH":["_A","info","number"]},
                    "<",
                    {"_PATH":["_B","info","number"]}
                ]
            }
        }
    }
}
```

### Filtering Arrays Using `_LOOP`

It is important to know that `null` values get removed entirely from computed objects. Thus one could for example filter out every other element of an array by simply doing:

```json
{
    "_LOOP": {
        "_ARRAY": /* ... */,
        "_EACH" : {
            "_CONDITIONAL": {
                "_IF":{"_MATH":"{_INDEX}%2==0"},
                "_THEN":null,
                "_ELSE":"{_ITEM}"
            }
        }
    }
}
```

### The `_WEB_CALLBACK` Function (Web View for User Input or Authentication)

It is possible to pop up a webview when a relevant card is loaded, be it for user authentication, or any other form of user input. This is achieved with the `_WEB_CALLBACK` function. This function will return either when the user dismisses the webview manually, or when a certain base URL is loaded as a result of navigation. This functions looks like this:

```json
{
    "_WEB_CALLBACK": {
        "_ADDRESS":"http://relevant.ai/",
        "_CALLBACK_BASE":"relevant.ai/callback"
    }
}
```

Notice that the callback base URL **does not contain the protocol http://**. If the user dismisses the webview, then the function returns `null`. Otherwise it returs an object of the form:

```json
{
    "url":/* URL ABSOLUTE STRING */,
    "body":{
        /* URL PARAMETERS AND VALUES */
    },
    "method":/* "GET" OR "POST" OR OTHER */,
    "headers":{
        /* HEADER TITLES AND VALUES */
    }
}
```

Notice that this return only contains information about the request that was made. This is because the webview is dismissed before its content is loaded.

## Templates (Card Appearance):

Let us go back to the first example of a fully functional card:

```json
{
    "id": "card-id-string",
    "title": "My Card's Title",
    "icon_url": "https://url/to/icon.png",
    "summary": "My card's description.",
    "credits": "Some Website or Source",
    "templates": [
                  "headline",
                  "footer"
                  ],
    "settings_type": "NONE",
    "_LOAD": {
        "_RETURN": [
            {
                "headline": {"text": "My Header 1"},
                "footer": {"text": "My Footer 1"}
            },
            {
                "headline": {"text": "My Header 2"},
                "footer": {"text": "My Footer 2"}
            }
        ]
    }
}
```

This card has two templates `headline` and `footer`, in that order. This is why the `_LOAD` object returns an array of objects of the form `{"headline":{...},"footer":{...}}`. In general, if a card is using templates `template_1`, `template_2`, ..., `template_n`, then the `_LOAD` object must return an array of objects of the form `{"template_1":{...},...,"template_n":{...}}`, where the `{...}`'s hold parameters that skin each template's labels and images. Below is a list of all available templates and their parameters:

```json
{
    "title":{"text":/*...*/},
    "footer":{"text":/*...*/},
    "large-image-square":{"image":/*...*/},
    "description":{"title":/*...*/,"subtitle":/*...*/,"thumb":/*...*/},
    "title-two-lines":{"text":/*...*/},
    "profile":{"title":/*...*/,"subtitle":/*...*/},
    "banner":{"image":/*...*/},
    "headline":{"text":/*...*/},
    "large-stats":{
        "title-1":/*...*/,
        "title-2":/*...*/,
        "title-3":/*...*/,
        "title-4":/*...*/,
        "title-5":/*...*/,
        "value-1":/*...*/,
        "value-2":/*...*/,
        "value-3":/*...*/,
        "value-4":/*...*/,
        "value-5":/*...*/
    },
    "stats":{
        "title-1":/*...*/,
        "title-2":/*...*/,
        "title-3":/*...*/,
        "value-1":/*...*/,
        "value-2":/*...*/,
        "value-3":/*...*/
    },
    "transit":{
        "number":/*...*/,
        "place":/*...*/,
        "address":/*...*/,
        "time-1":/*...*/,
        "time-2":/*...*/,
        "time-3":/*...*/
    },
    "image-stats":{"stats":/*...*/,"image":/*...*/},
    "large-image-3x4":{"image":/*...*/},
    "short-description":{"text":/*...*/},
    "profile-image":{"image":/*...*/},
    "match":{
        "team1": {
            "image":/*...*/,
            "name":/*...*/,
            "score":/*...*/
        },
        "team2": {
            "image":/*...*/,
            "name":/*...*/,
            "score":/*...*/
        },
        "middle":/*...*/,
        "time":/*...*/
    },
    "thumbnail":{"image":/*...*/},
    "long-description":{"text":/*...*/},
    "nearby":{
        "image":/*...*/,
        "place":/*...*/,
        "address":/*...*/,
        "latitude":/*...*/,
        "longitude":/*...*/
    },
    "thumb-scalar":{
        "number":/*...*/,
        "units":/*...*/,
        "title":/*...*/,
        "subtitle":/*...*/,
        "image":/*...*/
    }
}
```

## Actions:

Relevant cards allow you to perform actions by holding down your finger on the card. The default actions are *refresh*, *share* and *zoom-in*. You can add more actions to the library using the `actions` template. For this you don't need to add `"actions"` to your `"templates metadata"`. You simply need to add an `"actions"` key to each element of the array returned by the `_LOAD` object. The value of this key will be an array of added actions. All of the elements of this array look as follows:

```json
{
    "_ACTION":/*...*/,
    "_ICON":/*...*/
}
```

The value of `"_ICON"` can be `"done-icon"`, `"eye-icon"`, `"fav-icon"`, `"heart-icon"`, `"repost-icon"`, `"source-icon"`, `"downvote-icon"`, `"map-icon"`, or `"upvote-icon"`. The action must be one of the following currently available functions:

```json
{
    "_WEBVIEW":/* SOME SOURCE URL */
}
```

```json
{
    "_MAPVIEW":{
        "title":/*...*/,
        "latitude":/*...*/,
        "longitude":/*...*/
    }
}
```






