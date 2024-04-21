+++
title = 'Pino Sns Transport'
date = 2024-04-20T21:19:24+05:00
draft = true
series = "Application Monitoring"
tags = ["pino", "application-monitoring", "open-source"]
+++

## Usage

{{< tabs tabTotal="3" >}}

{{% tab tabName="Quick Start" %}}

```typescript
import pino, { TransportTargetOptions } from 'pino';
import type { SnsTransportOptions } from 'pino-sns-transport';

const transportTargets: TransportTargetOptions[] = [
  {
    target: 'pino-sns-transport',
    options: {
      topic: process.env.TOPIC,
    } as SnsTransportOptions,
    level: 'warn',
  },
];

const transport = pino.transport({
  targets: transportTargets,
});

const logger = pino(
  {
    /**
     * Set this to trace or the minimum from the logging
     * levels of the transports so that all logs are
     * forwarded to transports, each transport carries its
     * own level and therefore can decide whether it wants
     * to log or not
     */
    level: 'trace',
  },
  transport,
)
```

{{% /tab %}}

{{% tab tabName="All Options" %}}

```typescript
import { SNSClientConfig } from "@aws-sdk/client-sns";

export type LogFilter = {
  key: string;
  pattern: RegExp,
}

export type SnsTransportOptions = {
  snsClientConfig?: SNSClientConfig;
  topic?: string;
  topicArn?: string;
  beautify?: boolean;
  beautifyOptions?: {
    indentSize?: number;
    maxWidth?: number;
  };
  excludeKeys?: string[];
  keyExamineDepth?: number;
  includeLogs?: LogFilter[];
  excludeLogs?: LogFilter[];
}
```

{{< /tab >}}

{{% tab tabName="Longer Example" %}}

```typescript
const transportTargets: TransportTargetOptions[] = [
  {
    target: 'pino-sns-transport',
    options: {
      topicArn: process.env.TOPIC_ARN,
      excludeKeys: [
        'pid',
        'hostname',
        'res.headers',
        'req.headers',
        'req.remoteAddress',
        'req.remotePort',
      ],
      excludeLogs: [
        {
          key: 'msg',
          pattern: /Request (Completed|Errored)/,
        },
        {
          key: 'context',
          pattern: /ExceptionsHandler/,
        },
      ],
    } as SnsTransportOptions,
    level: 'warn',
  },
];
```

{{< /tab >}}

{{< /tabs >}}
