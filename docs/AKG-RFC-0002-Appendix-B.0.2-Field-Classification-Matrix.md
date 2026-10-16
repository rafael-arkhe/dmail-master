# AKG-RFC-0002 — Appendix B.0.2
## Field Classification Matrix — Freeze Candidate

**Profile:** `AKG-CBOR-v1`
**Depends on:** B.0.1 General Wire Rules
**Semantic baseline:** `AKG-RFC-0002 v0.1.3 + Wire Prerequisite Clarifications`
**Status:** `FREEZE CANDIDATE`

A regra de autoridade deste appendix é:

\[
B.0.2: SemanticType \rightarrow UniqueWireProjection
\]

e nunca:

\[
WireProjection \rightarrow NewSemanticType
\]

---

## B.0.2-0 — Regras wire transversais

B.0.2 herda de B.0.1:

| Classe | Regra v1 |
|---|---|
| object | CBOR map |
| field key | exact text key definida pelo schema |
| unknown field | `REJECT` |
| digest | `bstr(32)` = BLAKE3-256 |
| optional | ausência do key; `null` não substitui ausência |
| semantic set | CBOR array canonicamente ordenado; duplicata ⇒ malformed |
| ordered sequence | array cuja ordem é preservada |
| enum | literal text fechado; unknown ⇒ reject |
| text | NFC antes do encoding |
| self-digest | proibido dentro do core identificado |
| signature | fora do digest preimage |
| signature message | `SignatureDomain || ObjectDigest` |
| unknown union discriminator | `REJECT` |

Bounds globais continuam os de B.0.1 e não são redefinidos aqui.

---

# B.0.2-1 — `PolicyRef`

A semântica preservada da RFC-0001 é:

\[
PolicyRef=\{policy\_id,version\}
\]

| Field | Wire type | Required | Collection | Normalization | Bounds |
|---|---|---:|---|---|---|
| `policy_id` | text | yes | scalar | NFC | `text_name` |
| `version` | uint | yes | scalar | — | `uint32` |

Equality:

\[
PolicyRef_A=PolicyRef_B \iff policy\_id_A=policy\_id_B \land version_A=version_B
\]

`uint32` é uma restrição do profile wire, não semântica importada da RFC-0001.

---

# B.0.2-2 — `PolicyDigest` binding

`PolicyDigest` é separado de `PolicyRef`:

| Field/Value | Wire type | Required |
|---|---|---:|
| `policy_digest` | bstr32 | yes whenever the referenced policy affects protected evaluation |

Não há self-contained `PolicyDigest` object necessário: o valor é o digest criptográfico do `PolicyCore` segundo o domain já normativamente definido.

Regra geral:

\[
SecurityRelevantPolicyRef \Rightarrow PolicyDigestPresent
\]

e:

\[
Resolve(PolicyRef)\rightarrow d
\]

\[
d\neq PolicyDigest \Rightarrow REJECT
\]

**Importante:** B.0.2 preserva `PolicyRef`; não o converte em `{digest}`.

---

# B.0.2-3 — `IdentityRef`

| Field | Wire type | Required | Bounds | Rule |
|---|---|---:|---|---|
| `scheme` | text | yes | 1..64 UTF-8 bytes | canonical v1 scheme |
| `value` | text | yes | 1..1024 UTF-8 bytes | opaque under scheme |

Para evitar transformação silenciosa de identidade, eu congelaria uma regra um pouco mais segura do que “lowercase on decode”:

\[
NonCanonicalScheme \Rightarrow REJECT
\]

Em v1, `scheme` MUST já chegar na forma canónica definida pelo profile; o decoder não modifica o identificador antes de verificar assinatura/digest.

`value` é NFC, mas qualquer normalização específica do esquema pertence ao esquema correspondente.

---

# B.0.2-4 — `PropertyRef`

A semântica `{name, version}` está confirmada diretamente no schema da RFC-0001.

| Field | Wire type | Required | Bounds |
|---|---|---:|---|
| `name` | text | yes | `text_name` |
| `version` | uint | yes | `uint32` |

\[
PropertyRef_A=PropertyRef_B \iff name_A=name_B\land version_A=version_B
\]

No v1 não existe compatibilidade implícita entre versões.

---

# B.0.2-5 — `ScopeExpr`

A RFC-0001 congela a union semântica:

\[
scope\_expr := artifact\_set \mid symbol\_set \mid component(name)
\]

B.0.2 materializa essa union com discriminador explícito:

| `kind` | Fields | Semantics |
|---|---|---|
| `component` | `value:text` | one component |
| `artifact_set` | `values:[text]` | semantic set |
| `symbol_set` | `values:[text]` | semantic set |

Para `artifact_set.values` e `symbol_set.values`:

\[
SortKey(x)=UTF8(NFC(x))
\]

ordem lexicográfica por bytes.

Empty set ⇒ malformed.
Duplicate normalized value ⇒ malformed.
Nenhuma quarta variante é permitida em v1.

Esse é um caso legítimo de:

\[
RFC0001.Semantics + B002.WireDiscriminator
\]

---

# B.0.2-6 — `Finding`

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `code` | text | yes | scalar |
| `category` | enum text | yes | scalar |
| `subject_ref` | `SubjectRef` | yes | scalar |
| `evidence_refs` | `[EvidenceRef]` | yes | semantic set |
| `details_ref` | bstr32 | no | scalar |

`severity` MUST NOT occur.

\[
severity\in Finding \Rightarrow Malformed
\]

`FindingDigest` é derivado externamente:

\[
FindingDigest= BLAKE3\text{-}256( DomainFinding \parallel CanonicalBytes(Finding) )
\]

---

# B.0.2-7 — `FindingDisposition`

| Field | Wire type | Required |
|---|---|---:|
| `finding_ref` | bstr32 | yes |
| `disposition` | enum text | yes |
| `policy_ref` | `PolicyRef` | yes |
| `policy_digest` | bstr32 | yes |
| `rationale_ref` | bstr32 | no |

PoC disposition enum:

\[
INFO\mid WARNING\mid BLOCKING
\]

Unknown ⇒ reject.

A regra crítica é:

\[
FindingDisposition.policy\_ref
\]

e:

\[
FindingDisposition.policy\_digest
\]

formarem o mesmo policy binding.

---

# B.0.2-8 — `EnvironmentComponent`

A emenda fechou semanticamente:

\[
EnvironmentDescriptor = EnvironmentProfile+Components
\]

A projeção mínima fica:

| Field | Wire type | Required |
|---|---|---:|
| `kind` | text_id | yes |
| `identity` | `IdentityRef` | yes |
| `artifact_digest` | bstr32 | no |
| `configuration_digest` | bstr32 | no |

Cada component core pode ter identidade content-addressed derivada externamente:

\[
ComponentDigest= H( DomainEnvironmentComponent \parallel CanonicalBytes(ComponentCore) )
\]

**Não** incluir `component_digest` dentro do próprio core.

---

# B.0.2-9 — `EnvironmentDescriptor`

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `profile_ref` | bstr32 | yes | scalar |
| `components` | `[EnvironmentComponent]` | yes | semantic set |

Sort:

\[
SortKey(Component)=ComponentDigest
\]

`EnvironmentDigest` é externo:

\[
EnvironmentDigest= H( DomainEnvironment \parallel CanonicalBytes(EnvironmentDescriptor) )
\]

O descriptor não recebe campos privilegiados como `os`, `runtime`, `tee_type`, etc.; esses valores são expressos por `EnvironmentComponent.kind`.
Isso preserva a generalidade da emenda e evita hard-code de tecnologias no kernel.

---

# B.0.2-10 — `StrongStateCommutationEvidence`

PoC projection:

| Field | Wire type | Required |
|---|---|---:|
| `subject_a` | bstr32 | yes |
| `subject_b` | bstr32 | yes |
| `state_profile_ref` | bstr32 | yes |
| `preconditions_ref` | bstr32 | no |
| `proof_digest` | bstr32 | yes |

Não existem na PoC: `verifier`, `verification_method`, `policy_ref`, `equivalence_ref`, `obs_p_ref`, `result`.

A equivalência é fixa normativamente:

\[
ExactCanonicalStateEquality
\]

e não precisa ocupar bytes no payload.

Isso é importante: não serializamos constantes semânticas que o profile já fixa, salvo quando necessárias para version dispatch.

---

# B.0.2-11 — `EvidenceRef`

| Field | Wire type | Required |
|---|---|---:|
| `evidence_class` | text_id | yes |
| `content_digest` | bstr32 | yes |
| `media_type` | text | no |
| `location_ref` | text | no |
| `producer_ref` | `IdentityRef` | no |

Em semantic sets:

\[
SortKey(E) = (E.content\_digest,\ CanonicalBytes(E))
\]

`location_ref` nunca participa da identidade do artefato.

---

# B.0.2-12 — `VerificationResult`

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `verdict` | enum text | yes | scalar |
| `findings` | `[Finding]` | yes | semantic set |

Verdicts:

\[
PASS\mid FAIL\mid INCONCLUSIVE
\]

Sort de findings:

\[
SortKey(F)=FindingDigest(F)
\]

O `VerificationResult` contém findings factuais, nunca policy dispositions.

---

# B.0.2-13 — `VerifierIdentity`

| Field | Wire type | Required |
|---|---|---:|
| `principal` | `IdentityRef` | yes |
| `agent` | `IdentityRef` | no |
| `authority_domain` | text_id | yes |
| `verifier_class` | text_id | yes |
| `key_ref` | `IdentityRef` | yes |

Ausência de `agent`: `omit` não `null`.

---

# B.0.2-14 — `EvaluationContext`

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `branch_ref` | `BranchId` | yes | scalar |
| `base_frontier_digest` | bstr32 | yes | scalar |
| `composed_intents` | `[bstr32]` | yes | semantic set |
| `dependency_closure` | `[bstr32]` | yes | semantic set |
| `property_refs` | `[PropertyRef]` | yes | semantic set |
| `scopes` | `[ScopeExpr]` | yes | semantic set |
| `policy_ref` | `PolicyRef` | yes | scalar |
| `policy_digest` | bstr32 | yes | scalar |
| `freshness_policy_ref` | `PolicyRef` | no | scalar |
| `freshness_policy_digest` | bstr32 | conditional | scalar |
| `environment_ref` | bstr32 | conditional | scalar |
| `verifier_inputs_digest` | bstr32 | yes | scalar |

Conditional pair:

\[
freshness\_policy\_ref\ present \iff freshness\_policy\_digest\ present
\]

Environment rule:

\[
MethodRequiresEnvironment \Rightarrow environment\_ref\ present
\]
\[
MethodClaimsEnvironmentIndependent \Rightarrow environment\_ref\ MAY\ be\ absent
\]

`BranchId` precisa ter sua projeção final v1 explicitada antes do freeze.

---

# B.0.2-15 — `VerificationRecordCore`

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `record_version` | uint | yes | scalar |
| `intent_digest` | bstr32 | yes | scalar |
| `claim_refs` | `[ClaimRef]` | yes | semantic set |
| `verifier` | `VerifierIdentity` | yes | scalar |
| `verification_method` | `VerificationMethod` | yes | scalar |
| `evidence` | `[EvidenceRef]` | yes | semantic set |
| `result` | `VerificationResult` | yes | scalar |
| `evaluation_context` | `EvaluationContext` | yes | scalar |
| `issued_at` | uint | yes | scalar |
| `policy_ref` | `PolicyRef` | yes | scalar |
| `policy_digest` | bstr32 | yes | scalar |

No signature inside core.

Digest:

\[
VerificationRecordDigest = BLAKE3\text{-}256( DomainVerify \parallel CanonicalBytes(VerificationRecordCore) )
\]

Attestation external:

\[
VerificationRecord= \{ core,\ attestation \}
\]

conceptually, per M12.

---

# B.0.2-16 — `SignatureEnvelope`

PoC permits exactly one profile.

| Field | Wire type | Required | Exact rule |
|---|---|---:|---|
| `algorithm_profile` | text enum | yes | `"AKG-SIG-ED25519-v1"` |
| `key_ref` | `IdentityRef` | yes | signing key |
| `signature_bytes` | bstr | yes | exactly 64 bytes |

Any other profile:

\[
UnknownAlgorithmProfile\Rightarrow REJECT
\]

Signature:

\[
Ed25519.Sign( PrivateKey, SignatureDomain\parallel ObjectDigest )
\]

Public key material for the profile is exactly 32 bytes.
B.0.2 MUST NOT encode an algorithm-selection field that accepts arbitrary future names.
The enum is closed in v1.

---

# B.0.2-17 — `OrderingEdge`

| Field | Wire type | Required |
|---|---|---:|
| `source` | bstr32 | yes |
| `target` | bstr32 | yes |
| `kind` | enum text | yes |
| `reason_ref` | bstr32 | yes |

Semantic-set sort:

\[
(source,target,kind_{UTF8},reason\_ref)
\]

Self-edge:

\[
source=target \Rightarrow Malformed
\]

Cycle detection is semantic validation rather than canonicalization:

\[
Cycle(OrderingGraph)\Rightarrow REJECT
\]

---

# B.0.2-18 — `BranchIntegrationStateCore`

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `branch_id` | `BranchId` | yes | scalar |
| `state_version` | uint | yes | scalar |
| `integrated_intents` | `[bstr32]` | yes | semantic set |
| `dependency_graph` | `[DependencyEdge]` | yes | semantic set |
| `explicit_ordering_edges` | `[OrderingEdge]` | yes | semantic set |
| `integration_frontier` | `[bstr32]` | yes | semantic set |
| `policy_ref` | `PolicyRef` | yes | scalar |
| `policy_digest` | bstr32 | yes | scalar |

Não existem: `state_digest`, `signature`, `integrator` dentro do core.

\[
BranchDigest= BLAKE3\text{-}256( DomainBranch \parallel CanonicalBytes(BranchIntegrationStateCore) )
\]

---

# B.0.2-19 — `BranchAttestation`

| Field | Wire type | Required |
|---|---|---:|
| `branch_digest` | bstr32 | yes |
| `signer` | `IdentityRef` | yes |
| `signature` | `SignatureEnvelope` | yes |

O signed object digest é `branch_digest`.
A presence da attestation não altera o `BranchDigest`.

---

# B.0.2-20 — `AssessmentDimension`

O schema congelado anterior era:

\[
AssessmentDimension= \{ verdict, findings, evidence\_refs \}
\]

Mantendo agora a separação factual/policy:

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `verdict` | enum text | yes | scalar |
| `finding_refs` | `[bstr32]` | yes | semantic set |
| `evidence_refs` | `[EvidenceRef]` | yes | semantic set |

Eu prefiro `finding_refs` em vez de findings inline aqui, porque o assessment refere fatos já emitidos pelo verifier e aplica policy sobre eles.
Isso preserva `VerifierCreatesFinding` e `Integrator/PolicyReferencesFinding` e evita duplicar o factual object dentro do assessment.

### Consequência normativa de propagação

Todos os campos históricos chamados `findings` em objetos de policy/integration precisam ser classificados como uma destas duas coisas: `FindingRef` ou `FindingDisposition`. Nunca um híbrido severity-bearing `Finding`.
Esse é um **audit obrigatório antes do freeze**, não uma nova decisão de arquitetura.

---

# B.0.2-21 — `IntegrationAssessmentCore`

A projeção deve incluir:

| Field | Wire type | Required | Collection |
|---|---|---:|---|
| `assessment_version` | uint | yes | scalar |
| `candidate_intent` | bstr32 | yes | scalar |
| `branch_state_digest` | bstr32 | yes | scalar |
| `branch_frontier_digest` | bstr32 | yes | scalar |
| `policy_ref` | `PolicyRef` | yes | scalar |
| `policy_digest` | bstr32 | yes | scalar |
| `local_validity` | `AssessmentDimension` | yes | scalar |
| `dependency_validity` | `AssessmentDimension` | yes | scalar |
| `evidence_validity` | `AssessmentDimension` | yes | scalar |
| `independence_validity` | `AssessmentDimension` | yes | scalar |
| `composition_validity` | `AssessmentDimension` | yes | scalar |
| `finding_dispositions` | `[FindingDisposition]` | yes | semantic set |
| `required_reverification` | `[VerificationRequest]` | yes | semantic set |
| `proposed_ordering_edges` | `[OrderingEdge]` | yes | semantic set |
| `decision` | enum text | yes | scalar |

Não existe `assessment_digest` dentro do core.

\[
AssessmentDigest= BLAKE3\text{-}256( DomainAssess \parallel CanonicalBytes(IntegrationAssessmentCore) )
\]

Preservar **exatamente** `DomainAssess`.

---

# Residual wire-only closure ledger

| ID | Pendência | Classe |
|---|---|---|
| `W02-01` | exact `SubjectRef` projection | wire/import |
| `W02-02` | exact `ClaimRef` representation and sort key | wire/import |
| `W02-03` | exact `BranchId` projection | wire/import |
| `W02-04` | exact `VerificationMethod` variant projections | wire/import |
| `W02-05` | exact `DependencyEdge` projection/sort | wire/import |
| `W02-06` | exact `VerificationRequest` projection/sort | wire/import |
| `W02-07` | closed enums for finding categories, decisions, ordering kinds | wire registry |
| `W02-08` | propagate `Finding` vs `FindingDisposition` through every legacy field | consistency audit |
| `W02-09` | verify every `PolicyRef` affecting protected decisions has paired `PolicyDigest` | consistency audit |
| `W02-10` | verify all object/domain separators against frozen exact bytes | crypto audit |

Isso é exatamente o tipo de blocker que B.0.2 deveria descobrir: **wire-local, finito e auditável**.

## Freeze gate revisado

Eu congelaria B.0.2 quando:

\[
W02\text{-}01..10=CLOSED
\]
e:
\[
EveryField \Rightarrow \begin{cases} ExactWireType\\ ExactOptionality\\ ExactBounds\\ ExactCollectionSemantics\\ ExactSortKey \end{cases}
\]
mais:
\[
EveryDigest \Rightarrow ExactDomain+ExactPreimage
\]
e:
\[
EverySignature \Rightarrow ExactProfile+ExactMessage
\]
e:
\[
LegacyFindingPropagationAudit=PASS
\]
\[
PolicyDigestPropagationAudit=PASS
\]

Então:
\[
B.0.2=FROZEN
\]
seguido de:
\[
B.1=UNBLOCKED
\]

## Selos Atuais

\[
AKG\text{-}RFC\text{-}0002\ v0.1.3 = SEMANTICALLY\ CLOSED\ FOR\ PoC
\]
\[
M6=CLOSED
\]
\[
WIRE\ PREREQUISITE\ CLARIFICATIONS = NORMATIVELY\ CLOSED
\]
\[
B.0.2 = FIELD\ MATRIX\ DRAFTED;\ WIRE\text{-}LOCAL\ CLOSURE\ REQUIRED
\]
\[
B.1=BLOCKED
\]
