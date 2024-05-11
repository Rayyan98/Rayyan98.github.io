+++
title = 'Serverless Easy Env'
date = 2024-04-20T21:18:58+05:00
series = "Serverless"
tags = ["serverless", "serverless-plugins", "open-source"]
Summary = "random summary"
+++

## Abstract

Conditionals in serverless are unidiomatic and often based on stage. They are hard to read and cumbersome to write. This plugin exists to make it easy to source variables based on stage in a lazy manner.

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

For the native serverless approach, it is 

## Usage

{{< tabs tabTotal="3" >}}

{{% tab tabName="Serverless TS" %}}

```typescript
import type { AWS } from '@serverless/typescript';

const createServerlessConfiguration: () => Promise<AWS> = async () => {
  return {
    service: 'sample-service',
    custom: {
      'serverless-easy-env': {
        envResolutions: {
          apiKeys: {
            prod: ['ssm:apiKeyOnSsm', 'ssm:apiKeyOnSsm2'],
            default: ['env:API_KEY']
          },
          datadogEnabled: {
            prod: true,
            stg: true,
            dev: false,
            local: false,
          },
          someValueLikeSecurityGroups: {
            local: ['random-name'],
            default: ['ssm:abc', 'ssm:def']
          },
        },
      },
    },
    provider: {
      name: 'aws',
      runtime: 'nodejs18.x',
      apiGateway: {
        apiKeys: '${easyenv:apiKeys}' as never,
      },
      vpc: {
        securityGroupIds: '${easyenv:someValueLikeSecurityGroups}',
      } as never,
    },
    plugins: [
      'serverless-easy-env',
      'serverless-offline',
    ],
    package: {
      patterns: ['!**/*', 'src/**/*', '.env.easy*'],
    },
    functions: {
      main: {
        handler: 'src/main.handler',
        events: [
          {
            http: {
              method: 'GET',
              path: '/',
              cors: true,
            },
          },
        ],
      },
    },
  };
};

module.exports = createServerlessConfiguration();
```

{{% /tab %}}

{{% tab tabName="Serverless YAML" %}}

```yaml
service: sample-service
custom:
  serverless-easy-env:
    envResolutions:
      apiKeys:
        prod:
          - ssm:apiKeyOnSsm
          - ssm:apiKeyOnSsm2
        default:
          - env:API_KEY
      datadogEnabled:
        prod: true
        stg: true
        dev: false
        local: false
      someValueLikeSecurityGroups:
        local:
          - random-name
        default:
          - ssm:abc
          - ssm:def
provider:
  name: aws
  runtime: nodejs18.x
  apiGateway:
    apiKeys:
      - ${easyenv:apiKeys}
  vpc:
    securityGroupIds: ${easyenv:someValueLikeSecurityGroups}
plugins:
  - serverless-easy-env
  - serverless-offline
package:
  patterns:
    - '!**/*'
    - src/**/*
    - .env.easy*
functions:
  main:
    handler: src/main.handler
    events:
      - http:
          method: GET
          path: /
          cors: true
```

{{< /tab >}}

{{% tab tabName="Resolved YAML" %}}

```yaml
service: sample-service
custom:
  serverless-easy-env:
    envResolutions:
      apiKeys:
        prod:
          - ssm:apiKeyOnSsm
          - ssm:apiKeyOnSsm2
        default:
          - env:API_KEY
      datadogEnabled:
        prod: true
        stg: true
        dev: false
        local: false
      someValueLikeSecurityGroups:
        local:
          - random-name
        default:
          - ssm:abc
          - ssm:def
provider:
  name: aws
  runtime: nodejs18.x
  apiGateway:
    apiKeys:
      - some-api-key-that-exported-in-env
  vpc:
    securityGroupIds:
      - random-name
  stage: dev
  region: us-east-1
  versionFunctions: true
plugins:
  - serverless-easy-env
  - serverless-offline
package:
  patterns:
    - '!**/*'
    - src/**/*
    - .env.easy*
  artifactsS3KeyDirname: serverless/sample-service/local/code-artifacts
functions:
  main:
    handler: src/main.handler
    events:
      - http:
          method: GET
          path: /
          cors: true
    name: sample-service-local-main
```

{{< /tab >}}

{{< /tabs >}}
