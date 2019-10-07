---
date: 2017-03-23T17:36:15Z
title: URL Rewriting
menu:
  main:
    parent: "Transform Traffic"
weight: 5 
---

## <a name="overview"></a>Overview

URL rewriting is a very useful feature when translating an outbound API interface to the internal structure of your services.

To rewrite a URL with Tyk, you must specify the components of the URL to capture, and then the order in which to re-assemble the captured components.

Unlike other web servers, Tyk uses a wide match to capture the URL and then a fixed regex to handle the restructuring. So as with other middleware components you must set a path to match on.

Starting from Tyk Gateway 2.5 and Tyk Dashboard 1.5, our rewriting functionality has been significantly extended, allowing you to add conditional rewriting logic, based on multiple rules, checking URL, body, headers or session meta data. See the [Advanced Rewrites](#advanced) section for more details.

## <a name="url-rewrite-with-api"></a> Rewrite a URL with the API Definition

To rewrite a URL with the API Definition, you must add a new object to the `extended_paths` section of an API definition:

```{.copyWrapper}
"url_rewrites": [{
  "path": "match/me",
  "method": "GET",
  "match_pattern": "(\w+)/(\w+)",
  "rewrite_to": "my/service?value1=$1&value2=$2"
}],
```

*   `path`: The path to match, this can contain wildcards, so to match all sub-resources under `match/`, you could use `match/{id}`. The wildcard `{id}` is transformed into a wide regex (`(.*)`) to ensure that everything possible is captured. This is then discarded. The name of the group is irrelevant, it is only for your reference.

*   `method`: The method to match.

*   `match_pattern`: This is the actual capture group to generate. This is a pure regex, in this case we are capturing two word groups.

*   `rewrite_to`: Each capture group you specify in `match_pattern` is designated with an index, and then made available in the `rewrite_to` template. Here `$n` will map against each value found in the capture group, so in the above example, the rewrite will be:

```{.copyWrapper}
my/service?value1=match&value2=me
```

This can also include a new hostname, but you *must* specify the transport, e.g.

```{.copyWrapper}
https://my-new-target-host.com/my/service?value1=match&value2=me
```

## <a name="url-rewrite-with-endpoint-designer"></a>Rewrite a URL with the Endpoint Designer

To rewrite a URL using the Dashboard, you can use the same values are defined in the API Definition options, just set them in the **Endpoint Designer** instead for your path.

### Step 1: Add an Endpoint for the Path

From the **Endpoint Designer** add an endpoint that matches the path you want to rewrite. Select the **URL Rewrite** plugin.

![Endpoint designer][1]

### Step 2: Configure the URL Rewrite Plugin

Add the regex capture groups and the new URL to the relevant sections.

![URL rewrite configuration][2]

### Step 3: Save the API

Use the *save* or *create* buttons to save the changes and make the URL rewrite active.

## <a name="url-rewrite-context-variables"></a>Context Variables

### Context Variables

As of version 2.2 Tyk allows context variables to be injected into the regex using the `$tyk_context.` namespace instead of the numeric index.

For more details see [Context Variables][3]

### Meta Data

As of v2.3 it is possible to inject meta data from a Tyk Session Object linked to a token into your URL Rewrite commands. In a similar way to the context variables, the values are in a reserved namespace: `$tyk_meta.FIELDNAME`. This can be especially useful if you wish to incorporate custom query string parameters into a URL structure.

## <a name="advanced"></a>Advanced Rewriting

There are plenty of cases when path based rewriting is not enough. To cover this, starting from Tyk Gateway 2.5 and Dashboard 1.5, you can define complex conditional rewrites.

To make it work you should set the **triggers** field, defining rules. If there is no trigger match, the rewrite will fallback to the parent `rewrite_to`, but if either of the other two are triggered, the rewrite target is changed.

Additionally, each trigger also sets a context variable for each match it finds. These context vars can then be used in the rewrites. Trigger contexts take the format: `$tyk_context.trigger-{n}-{name}-{i}` where `n` is the trigger index in the array, `name` is the regexp matcher name and `i` is the index of that match (since query strings and headers can be arrays of values).

 > **NOTE**: When using `header_ matches` in the trigger, the name is the normalised form of the header name.


```{.copyWrapper}
{
  "url_rewrites": [
    {
      "path": "/foo/bar/baz",
      "method": "GET",
      "match_pattern": "/foo/bar/baz",
      "rewrite_to": "/foo/bar/baz",
      "triggers": [
        {
          "on": "any",
          "options": {
            "query_val_matches": {
                "culprit": {
                  "match_rx": "kronk"
                }
              }
            }
            "rewrite_to": "/fooble/barble/bazble?victim=$tyk_context.trigger-0-culprit-0"
        }
        {
          "on": "any",
          "options": {
            "query_val_matches": {
              "culprit": {
                "match_rx": "yzma"
              }
            }
          }
            "rewrite_to": "/foozle/barzle/bazzle?victim=$tyk_context.trigger-1-culprit-0"
        }
      ]
    }
  ]
}
```

The Trigger functionality supports:

* Header matches — `header_matches`
* Query string variable/value matches — `query_val_matches`
* Path part matches, i.e. components of the path itself - `path_part_matches`
* Session meta data values — `session_meta_matches`
* Payload matches — `payload_matches`
* Matching by request, by IP Address or JWT scope - `request_context_matches`

All of the triggers above, except `payload_matches`, have the same structure, shown in the example above. `payload_matches` requires defining only with regexp like this: `"payload_matches": { "match_rx": "regexp" }`.

For each trigger, the trigger can either use the on: `any` or on: `all` formatting. For `any`, if any one of the options in the trigger is true, the rewrite rule is fired. for `all`, all the options must be satisfied. This is limited to triggers, not groups of triggers. These will be evaluated one by one.

Additionally you also mix multiple matches in the same trigger. In the example below, it checks if the HTTP request has `X-Enable-Beta` with value `true`, **AND** if user key meta info has `beta_enabled` field set to `true`. If both matches are `true`, it will proxy the user to another upstream, like beta environment.
```{.copyWrapper}
"triggers": [
  {
    "on": "all",
    "options": {
      "header_val_matches": {
        "X-Enable-Beta": {
          "match_rx": "true"
        }
    },
    "session_meta_matches": {
      "beta_enabled": {
        "match_rx": "true"
      }
    }
  }
    "rewrite_to": "https://beta.upstream.com/feture"
  }
]
```

### Using the Endpoint Designer

You can define advanced URL rewrites using the Tyk Dashboard as well, by using the **Create Advanced Trigger** option from the **URL Rewriter** plugin. You will see a screen like this:

![URL rewrite add trigger][4]

When triggers are added, you can edit or remove them inside the **Advanced URL rewrite** section:

![URL rewrite list trigger][5]


[1]: /docs/img/dashboard/system-management/endpoint_design_url_rewrite_1.8.png
[2]: /docs/img/dashboard/system-management/configure_rewrite_2.5.png
[3]: /docs/concepts/context-variables/
[4]: /docs/img/dashboard/system-management/rewriteEndpointDesigner_add_trigger.png
[5]: /docs/img/dashboard/system-management/rewriteEndpointDesigner_trigger_list.png










