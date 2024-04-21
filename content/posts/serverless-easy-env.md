+++
title = 'Serverless Easy Env'
date = 2024-04-20T21:18:58+05:00
series = "Serverless"
tags = ["serverless", "serverless-plugins", "open-source"]
+++

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
