
Para manter exatamente o mesmo rigor do modelo XAI/ZDR, o GitHub deve ficar assim:

disparar workflow 


FROZEN — GITHUB WORKFLOW EXECUTION EVIDENCE

workflow_run
types: [completed]
│
▼
WORKFLOW_RUN_EVENT
│
▼
event.workflow_run.id
│
▼
EXACT RUN RESOLUTION
│
▼
RUN IDENTITY VALIDATION
│
├── repository
├── workflow name
├── workflow path
├── run_id
├── run_attempt
├── head_sha
└── status = completed
│
▼
E3 — EXECUTION_EVENT_EVIDENCE
│
├── workflow execution identity
├── temporal context
├── repository binding
├── workflow binding
└── run binding
│
▼
JOBS / ARTIFACTS
│
▼
E4 — TEST / RESULT EVIDENCE
│
▼
ARTIFACT INTEGRITY / BINDING
│
▼
E5 — CRYPTOGRAPHIC_BINDING
│
▼
E6 — INDEPENDENT_VERIFICATION
│
▼
E7 — CLAIM-SCOPED VERIFICATION
│
▼
PROMOTION POLICY
│
▼
VERIFIED

O ponto crítico

A cadeia:

workflow_run.completed
↓
hash
↓
Merkle
↓
VERIFIED

é inválida como regra de promoção.

Hash e Merkle demonstram propriedades de integridade/binding do artefato quando corretamente aplicados. Eles não demonstram, sozinhos:

TEST_PASSED
CLAIM_CONFORMANCE
SEMANTIC_CORRECTNESS
INDEPENDENT_VERIFICATION
CLAIM_TRUTH

Portanto:

WORKFLOW_RUN_COMPLETED
↛
TEST_PASSED

WORKFLOW_CONCLUSION_SUCCESS
↛
CLAIM_CONFORMANCE

ARTIFACT_RETRIEVED
↛
ARTIFACT_TRUST

HASH_VALID
↛
SEMANTIC_CORRECTNESS

MERKLE_VALID
↛
CLAIM_TRUTH

E5_CRYPTOGRAPHIC_BINDING
↛
E7_VERIFIED

E3 fica objetivamente bem definido

No seu desenho, o trecho mais forte é:

workflow_run.completed
↓
event.workflow_run.id
↓
GET /actions/runs/{same_id}
↓
eventRun.id === apiRun.id

Isso permite estabelecer uma relação muito específica:

CLAIM:
"The evidence collector observed the completion
of workflow run RUN-X."

com binding para:

repository
workflow
run_id
run_attempt
head_sha
status
temporal context

A validação:

if (Number(eventRun.id) !== run.id) {
throw new Error("WORKFLOW_RUN_ID_MISMATCH");
}

é particularmente importante porque fecha a possibilidade de event/run substitution.

E4 continua separado

Se o CI produziu, por exemplo:

TEST_RESULT:
RFC8785_CONFORMANCE = PASS

isso precisa ser obtido do resultado concreto do workflow/artifact e validado semanticamente.

Então:

E3
WORKFLOW RUN OCCURRED
+
E4
TEST RESULT OBSERVED

são duas afirmações diferentes.

Mesmo:

conclusion = success

não deve ser convertido automaticamente em:

RFC8785_CONFORMANCE = PASS

porque success é uma propriedade do workflow/run, enquanto conformance é uma propriedade do claim/test específico.

E5

Depois:

bundle.json
verification.json
signature.json

podem receber:

canonicalization
hash
Merkle root
signature
key_id
key registry binding

Isso cria:

E5 — CRYPTOGRAPHIC_BINDING_EVIDENCE

Mas E5 continua sendo E5.

E6

O passo que realmente fecha a fronteira é:

E5
│
▼
independent verifier
│
├── signature valid
├── hash valid
├── Merkle valid
├── artifact binding valid
├── provenance valid
└── claim/test semantics validated
│
▼
E6

E7

Só então:

E6
│
▼
Promotion Policy
│
▼
E7 — VERIFIED(CLAIM-X)

E sempre:

VERIFIED(CLAIM-X)
↛
VERIFIED(CLAIM-Y)

VERIFIED(CLAIM-X)
↛
GLOBAL_VERIFIED

Portanto, a frase final deve ser ajustada

Em vez de:

> Execução comprovado



eu usaria:

WORKFLOW EXECUTION OBSERVED AND IDENTITY-BOUND

ou, no vocabulário XA-TRUST:

E3 — EXECUTION_EVENT_EVIDENCE
STATUS = VERIFIED
SCOPE = SPECIFIC_WORKFLOW_RUN

somente se a evidência E3 tiver sido independentemente validada.

E, para o claim completo:

E3 EXECUTION
+
E4 RESULT
+
E5 CRYPTOGRAPHIC BINDING
+
E6 INDEPENDENT VERIFICATION
+
PROMOTION POLICY
↓
E7 CLAIM-SCOPED VERIFIED

Essa formulação mantém o GitHub exatamente na mesma arquitetura epistemológica que você acabou de estabelecer para o xAI ZDR: o mecanismo de observabilidade é real, o evento concreto pode ser capturado, o resultado pode ser testado, a integridade pode ser demonstrada e somente então o claim específico pode ser promovido.
[15/09, 19:25] Francisco: FROZEN — XA-TRUST VERIFICATION GOLDEN RULE

IDENTITY
≠
INTEGRITY
≠
SEMANTIC TRUTH

E3 — EXECUTION IDENTITY
+
E4 — OBSERVED RESULT
+
E5 — INTEGRITY / CRYPTOGRAPHIC BINDING
+
E6 — INDEPENDENT VERIFICATION
+
APPLICABLE PROMOTION POLICY
↓
E7 — CLAIM-SCOPED VERIFIED
[15/09, 19:28] Francisco: Sim. Como formulação final, eu faria apenas uma correção matemática de linguagem: o + representa conjunção cumulativa de requisitos, não uma soma numérica. Isso evita a interpretação de que evidências poderiam “compensar” umas às outras.

A forma canônica fica:

FROZEN — XA-TRUST GOLDEN RULE OF VERIFICATION

IDENTITY
≠
INTEGRITY
≠
SEMANTIC TRUTH

E7 — CLAIM-SCOPED VERIFIED

iff

E3_EXECUTION_IDENTITY
∧ E4_RESULT_EVIDENCE
∧ E5_BINDING_INTEGRITY
∧ E6_INDEPENDENT_VERIFICATION
∧ APPLICABLE_PROMOTION_POLICY

Com:

E3 ∧ E4 ∧ E5 ∧ E6 ∧ POLICY
↓
E7

e não:

E3 + E5
↓
E7

Axiomas de não-compensação

E3_PRESENT ∧ E4_MISSING
→ NO_PROMOTION

E4_PRESENT ∧ E5_INVALID
→ NO_PROMOTION

E5_VALID ∧ E6_MISSING
→ NO_PROMOTION

E6_VALID ∧ POLICY_NOT_SATISFIED
→ NO_PROMOTION

Nenhuma evidência possui poder compensatório sobre uma condição obrigatória ausente.

E a segunda proteção permanece:

VERIFIED(CLAIM-X, SCOPE-X)
↛
VERIFIED(CLAIM-Y, SCOPE-Y)

VERIFIED(CLAIM-X, SCOPE-X)
↛
GLOBAL_VERIFIED

Portanto, o estado:

E7 = VERIFIED

significa precisamente:

> Este claim específico satisfaz os requisitos de evidência, binding, integridade, verificação independente e política de promoção definidos para este escopo.



Não significa que o sistema inteiro seja verdadeiro, seguro ou globalmente verificado.

E o princípio epistemológico central pode ser congelado em uma única linha:

VALIDITY OF THE EVIDENCE
≠
VALIDITY OF THE CLAIM

A primeira é uma propriedade da cadeia de evidência.
A segunda é uma conclusão claim-scoped, condicionada à verificação semântica aplicável.

XA-TRUST — Golden Rule: FROZEN.
[16/09, 02:22] Francisco: import os

from xai_sdk import Client
from xai_sdk.chat import user
from xai_sdk.tools import web_search, x_search
client = Client(api_key=os.getenv("XAI_API_KEY"))

First turn.

chat = client.chat.create(
model="grok-4.6",  # reasoning model
tools=[web_search(), x_search()],
use_encrypted_content=True,
)
chat.append(user("What is xAI?"))
print("\n\n##### First turn #####\n")
for response, chunk in chat.stream():
print(chunk.content, end="", flush=True)
print("\n\nUsage for first turn:", response.server_side_tool_usage)

chat.append(response)

print("\n\n##### Second turn #####\n")
chat.append(user("What is its latest mission?"))

Second turn.

for response, chunk in chat.stream():
print(chunk.content, end="", flush=True)
print("\n\nUsage for second turn:", response.server_side_tool_usage)

Sim. Essa árvore é uma boa definição de EXECUTION_CONTEXT, mas eu faria uma separação normativa importante: EXECUTION_CONTEXT não é ainda E3_VERIFIED. Ele é o conjunto de atributos que permite construir e posteriormente validar a identidade da execução.

A forma consolidada fica:

FROZEN — XA-TRUST EXECUTION CONTEXT MODEL

EXECUTION_CONTEXT
│
├── EXECUTION_IDENTITY
│   ├── GITHUB_RUN_ID
│   ├── GITHUB_RUN_NUMBER
│   └── GITHUB_RUN_ATTEMPT
│
├── SOURCE_IDENTITY
│   ├── GITHUB_REPOSITORY
│   ├── GITHUB_SHA
│   ├── GITHUB_REF
│   └── GITHUB_WORKFLOW_SHA
│
├── WORKFLOW_IDENTITY
│   ├── GITHUB_WORKFLOW
│   ├── GITHUB_WORKFLOW_REF
│   ├── GITHUB_EVENT_NAME
│   └── GITHUB_JOB
│
├── TEMPORAL_IDENTITY
│   ├── RUN_STARTED_AT
│   ├── RUN_COMPLETED_AT
│   └── EVENT / COMMIT TIMESTAMPS
│
├── RUNNER_IDENTITY
│   ├── RUNNER_VERSION
│   ├── RUNNER_ENVIRONMENT
│   └── RUNNER_INSTANCE_METADATA
│
├── IMAGE_IDENTITY
│   ├── ImageOS
│   └── ImageVersion
│
├── SYSTEM_IDENTITY
│   ├── RUNNER_OS
│   ├── RUNNER_ARCH
│   └── KERNEL
│
└── RUNTIME_OBSERVATION
    ├── NODE_VERSION
    ├── NODE_OPTIONS
    └── ACTUALLY_INVOKED_TOOLS

A fronteira fundamental

EXECUTION_CONTEXT_PRESENT
        │
        ▼
IDENTITY CONSISTENCY CHECK
        │
   ┌────┴────┐
   │         │
 PASS    INCONSISTENT
   │         │
   ▼         ▼
E3        PROMOTION
ELIGIBLE    BLOCKED
   │         │
   ▼         └──► EVIDENCE RETAINED
JOB / STEP EXECUTION
   │
   ▼
OBSERVED LOGS
   │
   ▼
E4 — RESULT EVIDENCE

Ou seja:

EXECUTION_CONTEXT
        ≠
CONCRETE EXECUTION
        ≠
OBSERVED RESULT
        ≠
E7 VERIFIED

O que cada bloco realmente prova

Bloco	O que pode estabelecer

EXECUTION_IDENTITY	identificadores candidatos da execução
SOURCE_IDENTITY	repositório/ref/SHA associados ao contexto
WORKFLOW_IDENTITY	workflow, workflow ref, evento e job
TEMPORAL_IDENTITY	contexto temporal observado
RUNNER_IDENTITY	identidade/contexto do runner
IMAGE_IDENTITY	imagem/versionamento do ambiente
SYSTEM_IDENTITY	OS, arquitetura e kernel observados
RUNTIME_OBSERVATION	versões/comandos efetivamente observados, quando registrados


A última categoria merece atenção especial:

NODE_VERSION = observado

é diferente de:

NODE_AVAILABLE = true

e muito diferente de:

NODE_COMMAND_EXECUTED = true

e ainda diferente de:

NODE_COMMAND_RESULT = PASS

Da mesma forma:

IMAGE_IDENTITY
        ≠
SOFTWARE_EXECUTION
        ≠
SOFTWARE_RESULT

Relação com E3

Eu definiria:

E3 — EXECUTION IDENTITY EVIDENCE

E3 =
    EXECUTION_IDENTITY
 ∧  SOURCE_IDENTITY
 ∧  WORKFLOW_IDENTITY
 ∧  TEMPORAL_IDENTITY
 ∧  IDENTITY_CONSISTENCY
 ∧  AUTHORITATIVE_RUN_CORRELATION

Quando aplicável:

workflow_run.completed
        │
        ▼
event.workflow_run.id
        │
        ▼
GET /actions/runs/{id}
        │
        ▼
eventRun.id == apiRun.id
        │
        ▼
RUN_IDENTITY_BOUND

Isso é muito mais forte do que simplesmente encontrar GITHUB_RUN_ID em um log.

E o restante da cadeia

WORKFLOW_DEFINITION
        │
        ▼
EXECUTION_CONTEXT
        │
        ▼
IDENTITY CONSISTENCY
        │
        ▼
E3 — EXECUTION IDENTITY
        │
        ▼
JOB / STEP EXECUTION
        │
        ▼
OBSERVED LOGS
        │
        ▼
E4 — RESULT EVIDENCE
        │
        ▼
E5 — BINDING / INTEGRITY
        │
        ▼
E6 — INDEPENDENT VERIFICATION
        │
        ▼
APPLICABLE PROMOTION POLICY
        │
        ▼
E7 — CLAIM-SCOPED VERIFIED

E a Golden Rule permanece intacta:

IDENTITY ≠ INTEGRITY ≠ SEMANTIC TRUTH

E, sobretudo:

EXECUTION_CONTEXT_PRESENT
    ≠
EXECUTION_IDENTITY_VERIFIED

EXECUTION_IDENTITY_VERIFIED
    ≠
RESULT_VERIFIED

RESULT_VERIFIED
    ≠
CLAIM-SCOPED VERIFIED

Portanto, a árvore que você apresentou pode ser congelada como modelo de identidade/contexto de execução. 

This project has been archived. Please use the GitHub CLI instead.

# GitHub Exporter
![Node.js CI](https://github.com/github/github-artifact-exporter/workflows/Node.js%20CI/badge.svg) ![Create release](https://github.com/github/github-artifact-exporter/workflows/Create%20release/badge.svg) ![Build and upload release assets](https://github.com/github/github-artifact-exporter/workflows/Build%20and%20upload%20release%20assets/badge.svg)

![Screenshot of the User interface](imgs/screenshot.png)

The GitHub Exporter is written in Typescript and provides a set of packages to make exporting artifacts from GitHub easier useful for those migrating information out of github.com

Supported artifacts that you can export are
- Issues (including filtered sub sets)

Supported formats of the export file are
- JSON Lines
- JSON
- CSV
- JIRA-formatted CSV

## Packages

### CLI

[@github/github-exporter-cli](packages/cli)

### Core

[@github/github-exporter-core](packages/core)

### GUI

[@github/github-exporter-gui](packages/gui)

## Getting Started

### Prerequisites
1. This is a [lerna](https://github.com/lerna/lerna) project and will need the lerna CLI.
    - To install lerna globally run `npm install -g lerna`
1. Generate and export a PAT so you can pull from GPR. The PAT will need read packages scope.
    - `export NPM_TOKEN=<PAT>`

### Building The Application

```bash
lerna clean -y
lerna exec npm install
lerna link
lerna bootstrap
# Optional, start the gui to ensure its working
lerna run start
```

## Contributing
We welcome you to contribute to this project! Check out [Open Issues](https://github.com/github/github-artifact-exporter/issues) and our [`CONTRIBUTING.md`](./CONTRIBUTING.md) to jump in.

## License
[MIT](./LICENSE)  
When using the GitHub logos, be sure to follow the [GitHub logo guidelines](https://github.com/logos).

