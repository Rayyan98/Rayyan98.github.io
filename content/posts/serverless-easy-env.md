+++
title = 'Serverless Easy Env'
date = 2024-04-20T21:18:58+05:00
series = "Serverless"
tags = ["serverless", "serverless-plugins", "open-source"]
Summary = "random summary"
+++

## Abstract

Conditionals in serverless are unidiomatic and often based on stage. They are hard to read and cumbersome to write. [This plugin](https://www.npmjs.com/package/serverless-easy-env) exists to make it easy to source variables based on stage in a lazy manner.

## Serverless Native Conditionals

Handling conditionals especially based on stage has remained more or less the same in native serverless for a long time ([Ref](https://forum.serverless.com/t/conditional-serverless-yml-based-on-stage/1763)). And while some plugins exist that can handle conditionals, an engineer is still left wanting for a sleek way to source variables based on stage with fallbacks and have it scale easily with the number of stages. Bonus points if it can source variables from other places which is often the case in serverless and be lazy about doing so.

{{< tabs tabTotal="3" >}}

{{% tab tabName="Stage Based Conditionals" %}}

```yaml
custom:
  enabled:
    dev: false
    stg: true
    prod: ${ssm:IS_ENABLED}

enabled: ${self:custom.enabled.${sls:stage}}
```

{{< /tab >}}

{{% tab tabName="With Fallback" %}}

```yaml
custom:
  enabled:
    dev: false
    stg: true
    prod: ${ssm:IS_ENABLED}
    other: false

enabled: ${self:custom.enabled.${sls:stage}, self.enabled.other}
```

{{< /tab >}}

{{% tab tabName="With Plugin" %}}

```yaml
plugins:
 - serverless-plugin-utils

enabled : ${ternary(sls:stage, ‘prod’, true, false)}
```

{{< /tab >}}

{{< /tabs >}}

Its easy to see that while the plugin providing the ternary operator is more generic, why it would be difficult or cumbersome to scale it to a larger number of stages.

For the native serverless approach, it is redundant to write `${sls:stage}` every time, not to mention the entire expression is far too verbose especially with fallbacks. It also introduces slight chances of errors as one might add a fallback at one place but forget to add a fallback for the same variable at another place in the file leading to inconsistent behavior.

Additionally, the more savvy among readers might have already noticed that irrespective of what `sls:stage` resolves to (prod or not prod), the expression `${ssm:IS_ENABLED}` will always be evaluated and its value fetched from the parameter store even though it might not be needed. Consequentially, even to run this template with serverless offline (where stage will most likely resolve to `dev`), one will need to have their aws credentials configured.

## The Plugin - Ground Up

### Eliminating Verbosity

We have two goals here, short syntax and not having to specify the stage. We achieve this by having the plugin introduce a new [custom variable source](https://www.serverless.com/framework/docs/guides/plugins/custom-variables) and having the plugin resolve `sls:stage` internally through serverless util functions ([resolveVariable](https://www.serverless.com/framework/docs/guides/plugins/custom-variables))

This reduces our template to

{{< tabs tabTotal="3" >}}

{{% tab tabName="Template" %}}

```yaml
plugins:
 - serverless-easy-env

custom:
  serverless-easy-env:
    envResolutions:
      ENABLED:
        dev: ${env:IS_ENABLED}
        stg: true
        prod: ${ssm:IS_ENABLED}

enabled: ${easyenv:ENABLED}
```

{{% /tab %}}

{{% tab tabName="CLI" %}}

```bash
sls offline start --stage dev
```

{{% /tab %}}

{{% tab tabName="Environment or .env" %}}

```bash
# Shell
export IS_ENABLED=true
```

```.env
# .env file
IS_ENABLED=true
```

{{% /tab %}}

{{% /tabs %}}

### Easy Fallback

Having to re-specify the same value for most of the stages is far too cumbersome

```yaml
        prod: ${ssm:IS_ENABLED}
        dev: ${env:IS_ENABLED}
        qa-team-1: true
        qa-team-2: true
        stg: true
```

Therefore we assume that no one is going to name their stage `default` and when specified consider it as the fallback value if no stage specific value is found for a certain variable. Leaving this value unspecified will mandate defining the value of the variable for each stage, which might also be desirable in some cases.

```yaml
        prod: ${ssm:IS_ENABLED}
        dev: ${env:IS_ENABLED}
        default: true
```

### Lazy Evaluation

Our efforts still do not address the issue that serverless will still attempt to resolve both `${ssm:IS_ENABLED}` and `${env:IS_ENABLED}` irrespective to what stage is being used. We don't want serverless to reach for the parameter store during local development (when stage is `dev`) and we don't to specify `IS_ENABLED` in the environment running serverless deploy to `prod` since its value to be used must come from the parameter store.

To solve this we let go of the enclosers `${` and `}`. This makes serverless treat them as simple strings and to not try and resolve them while the plugin takes on the responsibility to resolve them through ([resolveVariable](https://www.serverless.com/framework/docs/guides/plugins/custom-variables)) as before.

Our template simplifies to

```yaml
plugins:
 - serverless-easy-env

custom:
  serverless-easy-env:
    envResolutions:
      DATADOG_ENABLED:
        # Controlled from param store for prod and staging 
        prod: ssm:DATADOG_ENABLED
        stg: ssm:DATADOG_ENABLED
        # Disabled on all other environments
        default: false
        # Up to developer for local development through env
        dev: env:DATADOG_ENABLED
      API_ENDPOINT_OF_SOME_OTHER_SERVICE:
        # Must be specified for every env explicitly
        # with no fallback
        prod: ssm:API_ENDPOINT_OF_SOME_OTHER_SERVICE
        stg: ssm:API_ENDPOINT_OF_SOME_OTHER_SERVICE
        qa-team-1: ssm:API_ENDPOINT_OF_SOME_OTHER_SERVICE
        qa-team-2: ssm:API_ENDPOINT_OF_SOME_OTHER_SERVICE
        dev: env:API_ENDPOINT_OF_SOME_OTHER_SERVICE

DATADOG_ENABLED: ${easyenv:DATADOG_ENABLED}
API_ENDPOINT_OF_SOME_OTHER_SERVICE: ${easyenv:API_ENDPOINT_OF_SOME_OTHER_SERVICE}
```

Now only strings related to the stage are going to be resolved. Any [variable source](https://www.serverless.com/framework/docs-providers-aws-guide-variables) besides `ssm` and `env` native to serverless or from a plugin can be used as well.

## Additional Features

Succinctly they are

- Overriding the stage to use instead of inferring through `sls:stage`
- Capture groups that map flexible stage names to fixed stage names for variable resolution
- Control whether to read .env file and path to it
- Deep Resolution of variables inside objects and array
- Output values of variable resolutions to .env formatted or json file for possible packing into Lambda

Further details about whats already discussed and about additional features can be found at the [package page](https://www.npmjs.com/package/serverless-easy-env#read-dot-env)
