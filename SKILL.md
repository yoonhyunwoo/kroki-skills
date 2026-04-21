---
name: "kroki"
description: "Generate diagrams from plain text using Kroki HTTP API. Use when the user asks to create, draw, or generate any kind of diagram — UML, flowchart, sequence, architecture, network, ER, Gantt, mindmap, wireframe, timing, BPMN, C4, data visualization, etc."
argument-hint: "[diagram-description]"
allowed-tools: "Bash(python3*), Bash(cat*), View, Write"
---

# Kroki Diagram Generator

Generate diagrams from plain text descriptions using the Kroki HTTP API at `https://kroki.io`.

## Workflow

### Phase 1: Recommend — ALWAYS do this before generating

When a user asks for a diagram, **do NOT immediately generate it**. Instead:

1. Analyze the user's intent — what are they trying to visualize?
2. Select the **top 3 most suitable diagram types** from the Supported Types table below.
3. For each candidate, write a **concise example** (3–8 lines) showing how the user's concept would look in that syntax.
4. Present the recommendations and ask which one to use (or if they want a different type).

Output format for recommendations:

```
## Recommended Diagram Types

### 1. {Type Name} — {one-line reason}
{example code block}

### 2. {Type Name} — {one-line reason}
{example code block}

### 3. {Type Name} — {one-line reason}
{example code block}
```

Only proceed to Phase 2 after the user confirms their choice.

### Phase 2: Generate

Once the user has chosen a diagram type:

1. Write the full diagram source code.
2. Send it to Kroki using POST plain-text via `python3` one-liner.
3. Save the output to a file and report the path.

#### Generation command

```bash
python3 -c "
import urllib.request, sys, json

source = sys.stdin.read()
data = json.dumps({
    'diagram_source': source,
    'diagram_type': '{TYPE}',
    'output_format': '{FORMAT}'
}).encode('utf-8')

req = urllib.request.Request('https://kroki.io/{TYPE}/{FORMAT}', data=data, headers={'Content-Type': 'application/json'})
resp = urllib.request.urlopen(req, timeout=30)
sys.stdout.buffer.write(resp.read())
" < {input_file} > {output_file}
```

Alternatively for simple diagrams, inline the source:

```bash
python3 -c "
import urllib.request, json
src = '''{DIAGRAM_SOURCE}'''
data = json.dumps({'diagram_source': src, 'diagram_type': '{TYPE}', 'output_format': '{FORMAT}'}).encode()
req = urllib.request.Request('https://kroki.io/{TYPE}/{FORMAT}', data=data, headers={'Content-Type': 'application/json'})
resp = urllib.request.urlopen(req, timeout=30)
open('{OUTPUT_FILE}', 'wb').write(resp.read())
print('Saved: {OUTPUT_FILE}')
"
```

#### Output format selection

- Default: `svg` (vector, best for docs)
- Use `png` when user explicitly wants raster
- Use `pdf` when user wants print-ready output

#### File naming

Save to the current working directory: `diagram.{svg|png|pdf}` unless the user specifies a path.

---

## Supported Types

| Endpoint | Type | Best for |
|---|---|---|
| `graphviz` | GraphViz / DOT | Directed graphs, dependency graphs, tree structures, network topology |
| `mermaid` | Mermaid | Flowcharts, sequence diagrams, class diagrams, Gantt charts, state diagrams, ER diagrams, pie charts, mindmaps |
| `plantuml` | PlantUML | All UML diagrams (class, sequence, use case, activity, state, component, deployment, object, timing), wireframes |
| `c4plantuml` | C4 with PlantUML | Software architecture — Context, Container, Component, Dynamic, Deployment diagrams |
| `structurizr` | Structurizr DSL | C4 model with workspace, system landscape, views |
| `d2` | D2 | Modern declarative diagrams, architecture, flowcharts with auto-layout |
| `bpmn` | BPMN | Business process modeling |
| `blockdiag` | BlockDiag | Block diagrams |
| `seqdiag` | SeqDiag | Sequence diagrams (simple) |
| `actdiag` | ActDiag | Activity diagrams (simple) |
| `nwdiag` | NwDiag | Network diagrams |
| `packetdiag` | PacketDiag | Network packet structure |
| `rackdiag` | RackDiag | Server rack diagrams |
| `erd` | ERD | Entity-relationship diagrams |
| `ditaa` | Ditaa | ASCII art to image |
| `svgbob` | SvgBob | ASCII diagrams to SVG |
| `excalidraw` | Excalidraw | Hand-drawn style diagrams |
| `goat` | GoAT | ASCII line diagrams |
| `nomnoml` | Nomnoml | UML class diagrams (simple) |
| `vega` | Vega | Complex data visualizations (bar, line, scatter, heatmap, geographic) |
| `vegalite` | Vega-Lite | Concise data visualizations |
| `wavedrom` | WaveDrom | Digital timing diagrams / waveforms |
| `bytefield` | Bytefield | Binary protocol / byte field diagrams |
| `wireviz` | WireViz | Cable and connector wiring diagrams |
| `symbolator` | Symbolator | HDL component diagrams |
| `umlet` | UMLet | UML diagrams (free-form) |
| `diagramsnet` | diagrams.net | General purpose (experimental) |

---

## Quick Syntax Reference (for examples in Phase 1)

### GraphViz (DOT)
```dot
digraph G {
    rankdir=LR;
    A -> B -> C;
    A -> C [label="skip"];
}
```

### Mermaid
```mermaid
graph TD
    A[Start] --> B{Decision}
    B -->|Yes| C[Action]
    B -->|No| D[End]
```

```mermaid
sequenceDiagram
    Alice->>Bob: Hello
    Bob-->>Alice: Hi
```

```mermaid
classDiagram
    class Animal {
        +String name
        +makeSound()
    }
    class Dog {
        +fetch()
    }
    Animal <|-- Dog
```

### PlantUML
```plantuml
@startuml
Alice -> Bob: Hello
Bob --> Alice: Hi
@enduml
```

```plantuml
@startuml
class Animal {
  + name: String
  + makeSound()
}
class Dog {
  + fetch()
}
Animal <|-- Dog
@enduml
```

### D2
```d2
x -> y: hello
y -> z: world
```

### C4 with PlantUML
```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml
Person(user, "User", "A user of the system")
System(system, "System", "The main system")
Rel(user, system, "Uses")
@enduml
```

### BPMN
```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL">
  <process id="process1" isExecutable="false">
    <startEvent id="start" name="Start"/>
    <task id="task1" name="Do Work"/>
    <endEvent id="end" name="End"/>
    <sequenceFlow sourceRef="start" targetRef="task1"/>
    <sequenceFlow sourceRef="task1" targetRef="end"/>
  </process>
</definitions>
```

### ERD
```
[Person]
*name
height
weight
+birth_location_id

[Location]
*id
city
state
country

Person *--1 Location
```

### SvgBob
```
  +---------+
  |  Hello  |---> World
  +---------+
```

### WaveDrom
```json
{ "signal": [
    { "name": "clk",  "wave": "p......." },
    { "name": "data", "wave": "x..2345x" }
]}
```

### Vega-Lite
```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "data": {"values": [{"x": 1, "y": 3}, {"x": 2, "y": 7}, {"x": 3, "y": 5}]},
  "mark": "bar",
  "encoding": {
    "x": {"field": "x", "type": "quantitative"},
    "y": {"field": "y", "type": "quantitative"}
  }
}
```

### PlantUML Built-in Icon Libraries (stdlib)

PlantUML stdlib provides built-in icon libraries accessible via `!include <lib/...>`. No external URLs needed.

#### Kubernetes Icons (`<k8s>`)

```plantuml
@startuml
!include <k8s/Common>
!include <k8s/Simplified>
!include <k8s/OSS/all>

left to right direction

actor "User" as user
Cluster_Boundary(c, "Kubernetes Cluster") {
  Namespace_Boundary(ns, "my-app") {
    KubernetesSvc(svc, "backend service", "8080/TCP")
    KubernetesPod(pod, "backend pod", "Spring Boot")
  }
}

user --> svc : request
svc --> pod : route
@enduml
```

Key includes:
- `!include <k8s/Common>` — base functions/shared definitions
- `!include <k8s/Simplified>` — simplified (less detailed) icons
- `!include <k8s/OSS/all>` — all Kubernetes OSS icons

Key macros:
- `Cluster_Boundary(alias, "label")` — cluster boundary
- `Namespace_Boundary(alias, "label")` — namespace boundary
- `KubernetesSvc(alias, "name", "port")` — Service icon
- `KubernetesPod(alias, "name", "description")` — Pod icon
- `KubernetesDeploy(alias, "name", "description")` — Deployment icon
- `KubernetesIng(alias, "name", "description")` — Ingress icon
- `KubernetesCM(alias, "name", "description")` — ConfigMap icon
- `KubernetesSecret(alias, "name", "description")` — Secret icon
- `KubernetesPVC(alias, "name", "description")` — PVC icon
- `Rel(from, to, "label")` or `-->` — connections

#### AWS Icons (`<awslib>` / `<awslib20>`)

```plantuml
@startuml
!include <awslib20/AWSCommon>
!include <awslib20/Compute/all>
!include <awslib20/Storage/all>

AWSConsole(console, "Management Console", "us-east-1")
EC2Instance(ec2, "Web Server", "t3.medium")
S3Bucket(bucket, "Static Assets", "my-bucket")

console --> ec2 : manage
ec2 --> bucket : read/write
@enduml
```

Key includes:
- `!include <awslib20/AWSCommon>` — base colors, styles (required first)
- `!include <awslib20/{Category}/all>` — load all icons in a category
- `!include <awslib20/{Category}/{Service}>` — load a single service icon

Available categories (use `{Category}/all` or individual service):
- `Analytics` — Athena, EMR, Glue, Kinesis, Redshift, etc.
- `ApplicationIntegration` — APIGateway, EventBridge, SQS, SNS, StepFunctions, etc.
- `Compute` — EC2, ECS, EKS, Lambda, Fargate, Batch, etc.
- `Database` — DynamoDB, ElastiCache, RDS, Aurora, DocumentDB, etc.
- `Networking` — VPC, CloudFront, Route53, ALB, NLB, TransitGateway, etc.
- `Security` — IAM, KMS, WAF, Shield, SecretsManager, etc.
- `Storage` — S3, EBS, EFS, FSx, Backup, etc.

Key macros:
- `AWS{Service}(alias, "name", "description")` — generic pattern
- `EC2Instance(alias, "name", "desc")` — EC2 instance icon
- `S3Bucket(alias, "name", "desc")` — S3 bucket icon
- `LambdaFunction(alias, "name", "desc")` — Lambda icon
- `DynamoDB(alias, "name", "desc")` — DynamoDB table icon
- `APIGateway(alias, "name", "desc")` — API Gateway icon

---

## Diagram Options

Pass options via `diagram_options` in the JSON body:

```json
{
    "diagram_source": "...",
    "diagram_type": "graphviz",
    "output_format": "svg",
    "diagram_options": {
        "layout": "neato",
        "theme": "cerulean"
    }
}
```

### Key options by type

| Type | Options |
|---|---|
| **GraphViz** | `layout` (`dot`, `neato`, `fdp`, `sfdp`, `twopi`, `circo`), `graph-attribute-*`, `node-attribute-*`, `edge-attribute-*` |
| **PlantUML** | `theme` (`amiga`, `cerulean`, `cyborg`, `sketchy`, `superhero`, `blueprint`, `plain`, etc.), `no-metadata` |
| **Mermaid** | kebab-case, dot→underscore (e.g. `er_title-top-margin`), cannot set `maxTextSize`, `securityLevel`, `secure`, `startOnLoad` |
| **D2** | `theme` (17 themes including `terminal`, `dark-mauve`, `earth-tones`), `layout` (`dagre`, `elk`), `sketch` (hand-drawn) |
| **SvgBob** | `font-family`, `font-size`, `fill-color`, `scale`, `stroke-width`, `background` |
| **BlockDiag** | `antialias`, `no-transparency`, `size` (`WxH`) |
| **Ditaa** | `no-antialias`, `no-separation`, `round-corners`, `scale`, `no-shadows`, `tabs` |

---

## Error Handling

- If Kroki returns HTTP 400: the diagram source has a syntax error — read the response body, fix the source, retry.
- If Kroki returns HTTP 413: diagram source is too large — simplify.
- If Kroki returns HTTP 503: service unavailable — retry once after a few seconds.
- Timeout: use 30 seconds. If it exceeds, inform the user and suggest simplifying the diagram.

## Language

Always respond in the same language the user writes in. Write diagram source code in English (standard syntax) regardless of user language.
