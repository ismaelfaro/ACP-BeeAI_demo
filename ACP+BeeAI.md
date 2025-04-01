# Agent Communication Protocol + BeeAI platform

## Demo 1 - Agent discovery

### Intro
The first thing that an ACP-powered platform enables is Agent discoverability

---

### Prepare 

- Install BeeAI platform to play with ACP concepts

```bash
brew install i-am-bee/beeai/beeai
```

- Run the Servers + Agents by default

```bash
beeai serve
```

- Setup the platform

```bash
beeai env setup
```

- Inspect the Agents in the platform

```bash
beeai list 
```

- To learn more about BeeAI, visit the web [beeai.dev](http://beeai.dev)
---

### Metadata

```bash
beeai ui
```

-  Each Agent has metadata that allows you and other Agents to quickly understand whether it is a fit for any use case.

```bash
beeai info aider 
```
```
Metadata(
    framework="Custom",
    license="Apache 2.0",
    languages=["Python"],
    githubUrl="https://github.com/...",
    fullDescription="...",
    examples="...",
    ui=UiDefinition(type=UiType.hands_off),
    avgRunTimeSeconds=5.0,
    avgRunTokens=5000,
    ...
)
```
---

### Use Agents from different frameworks

- One of the common tasks I do on a daily basis is competitive research in Quantum. 
- I can see here I have a few options for my Research use case powered by different providers (gpt-researcher and LangGraph)

```bash
beeai run ollama-deep-researcher "what is the most recent quantum error correction code research by IBM?"
```

```bash 
beeai run gpt-researcher "what is the most recent quantum error correction code research by IBM?"
```
---
### Examples of metadata

```yaml

manifestVersion: 1
name: github.com/i-am-bee/beeai/agents/community/something
driver: nodejs
package: git+https://github.com/i-am-bee/beeai/agents/community/something
command: ["something", "--transport", "http"]
serverType: http
mcpTransport: sse

```

### Discussion

- [help to improve ACP Metadata in our open discussion](https://github.com/i-am-bee/beeai/discussions/344)

---
## Demo 2 - Agent Import

### Intro
The other important component is how ACP-powered platform handles Agents 

You can also import Agents to the catalogue from online or local providers

---
### Providers

```bash
beeai provider list
```

You can add providers, from local or remote sources:

- Via Web

[https://github.com/i-am-bee/beeai-provider-starter](https://github.com/i-am-bee/beeai-provider-starter)

- From local 

```bash
beeai provider add file://demo-agent/openai-provider-unmanaged.yaml
```
---
### Example of wrapping your agents

```python
from acp.server.highlevel import Server, Context
from beeai_sdk.providers.agent import run_agent_provider
from beeai_sdk.schemas.metadata import UiDefinition, UiType
from beeai_sdk.schemas.text import TextInput, TextOutput


async def run():
    server = Server("openai-agent")

    @server.agent(
        name="openai-agent",
        description="OpenAI Agent to showcase beeai platform extension",
        input=TextInput,
        output=TextOutput,
        ui=UiDefinition(type=UiType.hands_off, userGreeting="Write a haiku poem about:")
    )
    async def your_agent_example(input: TextInput, ctx: Context) -> TextOutput:

        ###### [Your agent logic]
        
        return TextOutput(result)
    
    await run_agent_provider(server)

def main():
    asyncio.run(run())
```
---

### Example of wrapping your agents - OpenAI Agents Framework example

```python
from acp.server.highlevel import Server, Context
from beeai_sdk.providers.agent import run_agent_provider
from beeai_sdk.schemas.metadata import UiDefinition, UiType
from beeai_sdk.schemas.text import TextInput, TextOutput
from agents import Agent, Runner

async def run():
    server = Server("openai-agent")

    @server.agent(
        name="openai-agent",
        description="OpenAI Agent to showcase beeai platform extension",
        input=TextInput,
        output=TextOutput,
        ui=UiDefinition(type=UiType.hands_off, userGreeting="Write a haiku poem about:")
    )
    async def openai_agent_example(input: TextInput, ctx: Context) -> TextOutput:
        
        agent = Agent(name="Assistant", instructions="You are a helpful assistant")
        result = await Runner.run(agent, f"Write a haiku about {input}")
        return TextOutput(result)

    await run_agent_provider(server)

def main():
    asyncio.run(run())
```
---

## Demo 3 - Agent composition

### Intro
You can compose workflows where connect multiple Agents. [still experimental]

---

###  workflows

How I can concatenate some Agents, creating sequential workflows

```bash
beeai run sequential-workflow \
  '{"input": "Long article text here...",
    "steps": [
      {
        "agent": "gpt-researcher",
        "instruction": "Write an essay about applications of quantum computing"
      },
      {
        "agent": "aider",
        "instruction": "Create a plain HTML & simple CSS styled website based on the provided essay"
      }
    ]
  }' \
  --dump-files ./my-docs
```


```bash
beeai run sequential-workflow --dump-files ./my-docs
```

---

###  workflows

and also via web

[http://localhost:8333/compose](http://localhost:8333/compose)

steps:
- agent: gpt-researcher
  instruction: Write an essay about AI + Quantum.
- agent: aider
  instruction: Create a plain HTML & simple CSS styled website based on the provided essay, beautiful header.

---
###  workflows 

Also we can create more structure and control workflows.

```bash
beeai agent run supervisor '{"text":"Prepare a marketing strategy to sell most selling mobile phones in 2024 in Europe on my eshop. Ensure the strategy is based on top of thorough research of the market.", 
"availableAgents":["gpt-researcher","marketing-strategy"]}'
```
---

###  architecture

ACP reuse in this moment the client-server architecture from MCP via JSON-RPC 2.0

```

                                         < --------------------------- ACP -------------------------- >
                                                                                                                             
TextInput(Input)             +----------------------+         +----------------------+         +----------------------+    
TextOutput(Output)           |                      |         |                      |         |                      |    
MessageInput(Input)          |        Agent         | <-----> |        Agent         | <-----> |        Agent         |    
MessageOutput(Output)        |                      |         |                      |         |                      |    ^
...                          +----------------------+         +----------------------+         +----------------------+    |
                                         |                                |                                |               |
{                                        v                                v                                v               | 
  "id": 1,                   +----------------------+         +----------------------+         +----------------------+    |
  "jsonrpc": "2.0",          |                      |         |                      |         |                      |    
  "method": "methodName",    |       JSON-RPC       |         |       JSON-RPC       |         |       JSON-RPC       |   MCP
  "params": {                |                      |         |                      |         |                      |       
    /* parameters */}        +----------------------+         +----------------------+         +----------------------+    |   
}                                       |                                |                                |                |   
                                        v                                v                                v                |   
HTTP/WebSockets/Stdio        +----------------------+         +----------------------+         +----------------------+    |   
                             |                      |         |                      |         |                      |    v   
                             |        Tools         |         |        Tools         |         |        Tools         |         
                             |                      |         |                      |         |                      |       
                             +----------------------+         +----------------------+         +----------------------+         

```

- [Documentation online](https://docs.beeai.dev/acp/pre-alpha/architecture#messageinput-messageoutput)
- [ACP discusion](https://github.com/i-am-bee/beeai/discussions/284)

---

###  Open Discussions

ACP main concepts:

  - Architecture & Core Protocol Design
  - Deployment & Infrastructure
  - Data Security & Privacy
  - Protocol Implementation & Encoding
  - Testing & Quality Assurance
  - MCP Integration & Compatibility
  - UI & User Experience

How collaborate:

- [ACP Work streams](https://github.com/orgs/i-am-bee/discussions/488)
- [BeeAI initial implementation](https://docs.beeai.dev/acp/spec/concepts/overview)

---

## Thanks

All the BeeAI team and all contributors, and to you, for 
your participation in this event.

