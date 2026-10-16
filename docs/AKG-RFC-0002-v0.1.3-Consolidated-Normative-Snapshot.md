# AKG-RFC-0002 v0.1.3
## Consolidated Normative Snapshot — Pre-Freeze Candidate

**Snapshot ID:** `AKG-RFC-0002-v0.1.3-PoC-NORM-1`
**Status:** `PRE-FREEZE NORMATIVE SNAPSHOT`
**Purpose:** fornecer uma única superfície normativa contra a qual B.0.2, `FindingPropagationAudit` e `PolicyDigestPropagationAudit` possam ser executados deterministicamente.

Este snapshot incorpora:

\[
RFC0001_{frozen} + RFC0002_{v0.1.3} + WirePrerequisiteClarifications + WireClosurePatch_{v1..v3}
\]

e exclui explicitamente qualquer texto classificado como:

\[
historical \lor superseded \lor draft \lor research\_note \lor prior\_version
\]

A regra de precedência é:

\[
Snapshot > Clarifications > CurrentRFCText > HistoricalText
\]

---

## 1. Princípio de autoridade

A tese normativa permanece:

> Autor declara. Verificador atesta. Integrador decide. Policy executa.

Os três objetos persistentes fundamentais da RFC-0002 continuam sendo:

\[
VerificationRecord
\]
\[
IntegrationAssessment
\]
\[
BranchIntegrationState
\]

Nenhum wire appendix pode criar uma quarta autoridade persistente apenas porque uma subestrutura precisa ser serializada.

Em particular:
\[
Evidence \neq Authority
\]
\[
ExplicitOrdering \neq Evidence
\]
\[
Finding \neq PolicyDisposition
\]
\[
RecordValidity \neq EvidenceSufficiency \neq IntegrationEligibility
\]

---

## 2. Canonical identity

Para qualquer objeto Arkhe cujo digest seja definido pelo profile:

\[
CanonicalBytes(x) = AKGCBORv1(WireProjection_v1(x))
\]

e:

\[
ObjectDigest(x) = BLAKE3\text{-}256(DomainObject \parallel CanonicalBytes(UnsignedCore(x)))
\]

A projeção unsigned é definida **antes** da serialização.

É proibido:
\[
SerializeFullObject \rightarrow RemoveSignature
\]
como substituto da `UnsignedProjection`.

---

## 3. Policy binding

A RFC-0001 `PolicyRef` permanece:
\[
PolicyRef= \{ policy\_id, version \}
\]

Ela não é redefinida como digest.

A RFC-0002 acrescenta:
\[
PolicyBinding= (PolicyRef, PolicyDigest)
\]

onde:
\[
PolicyDigest = DigestAccordingTo(ResolvedPolicyArtifact, PolicyResolutionProfile)
\]

`PolicyDigest` MUST identificar o artefato de política exato efetivamente avaliado.

---

## 4. IdentityRef

\[
IdentityRef= \{ scheme, value \}
\]

Equality:
\[
IdentityRef_A=IdentityRef_B \iff scheme_A=scheme_B \land value_A=value_B
\]

---

## 5. PropertyRef

A semântica importada da RFC-0001 é:
\[
PropertyRef= \{ name, version \}
\]

---

## 6. ScopeExpr

A union semântica é:
\[
ScopeExpr := artifact\_set \mid symbol\_set \mid component(name)
\]

---

## 7. SubjectRef

`SubjectRef` é somente:
\[
SubjectRef = DigestRef \mid IdRef
\]

Wire semantic form:
\[
\{kind:"digest",value:bstr32\} \mid \{kind:"id",value:text\_id\}
\]

---

## 8. ClaimRef

Para a PoC:
\[
ClaimRef=text\_id
\]

---

## 9. Finding

`Finding` representa observação factual.
\[
Finding= \{ code, category, subject\_ref?, evidence\_refs, details\_ref? \}
\]

Normativamente:
\[
severity \notin Finding
\]

---

## 10. FindingDisposition

`FindingDisposition` é classificação de policy sobre um finding existente:
\[
FindingDisposition= \{ finding\_ref, disposition, policy\_binding, rationale\_ref? \}
\]

Para PoC:
\[
disposition \in \{ INFO, WARNING, BLOCKING \}
\]

Portanto:
\[
Finding \neq FindingDisposition
\]

---

## 11. Elegibilidade e findings

A regra corrente é:
\[
\exists d: d.disposition=BLOCKING \Rightarrow NotEligible
\]

---

## 12. EnvironmentDescriptor

\[
EnvironmentDescriptor= \{ EnvironmentProfile, Components \}
\]

Cada:
\[
EnvironmentComponent= \{ kind, identity, artifact\_digest?, configuration\_digest? \}
\]

---

## 13. VerificationMethod

A union permanece:
\[
VerificationMethod = ContractualVerifier \mid ExecutableVerifier \mid FormalVerifier
\]

### 13.1 ExecutableVerifier
\[
ExecutableVerifier= \{ tool\_identity, tool\_version, executable\_digest?, target\_scope, expected\_result, actual\_result \}
\]

### 13.2 FormalVerifier
\[
FormalVerifier= \{ system, system\_version, proof\_obligation\_digest, proof\_digest, result \}
\]

### 13.3 ContractualVerifier
\[
ContractualVerifier= \{ verifier\_profile:profile\_ref, obligations\_checked:[+obligation\_ref], input\_projection:projection\_ref, evaluation\_rules:ruleset\_ref, expected\_results:[+expected\_result], actual\_results:[+actual\_result], result\_digest:digest \}
\]

---

## 14. VerificationRequest

\[
VerificationRequest= \{ claims:[+ClaimRef], properties:[+PropertyRef], scopes:[+ScopeExpr], composed\_intents:[+IntentDigest], required\_classes:[+EvidenceClass], reason:FindingCode \}
\]

---

## 15. DependencyEdge

\[
DependencyEdge= \{ source:IntentDigest, target:IntentDigest, kind \}
\]
com:
\[
kind\in \{ STRUCTURAL, SEMANTIC, VERIFICATION \}
\]

---

## 16. StrongStateCommutationEvidence

Para PoC:
\[
StrongStateCommutes(A,B,D) \iff \forall s\in D: B(A(s))=A(B(s))
\]

---

## 17. Ordering

Explicit ordering é constraint:
\[
ExplicitOrdering \notin CompositionEvidence
\]

---

## 18. Composition closure

O closure normativo utiliza base explícita e frontier explícito:
\[
CompositionClosure(I,B,F,R,P)
\]

---

## 19. VerificationRecord

O core lógico inclui: `record_version`, `intent_digest`, `claim_refs`, `verifier`, `verification_method`, `evidence`, `result`, `evaluation_context`, `issued_at`, `PolicyBinding`. A assinatura está fora do digest core.

---

## 20. IntegrationAssessment

O core inclui: candidate intent, exact branch state binding, exact frontier binding, PolicyBinding, required assessment dimensions, factual finding references, policy dispositions, reverification requests, proposed ordering edges, final decision.

---

## 21. BranchIntegrationState

O core é semanticamente: `IntentDAG + ExplicitOrderingEdges + IntegrationFrontier + PolicyBinding`.

---

## 22. BranchAttestation

\[
BranchAttestation= \{ branch\_digest, signer, signature \}
\]

A attestation é externa e opcional salvo quando policy a exige.

---

## 23. Signature profile

A PoC permite somente: `AKG-SIG-ED25519-v1`.

---

## 24. Domain Registry PoC

Domains normativos preservados:
`AKG:VERIFY:v1\0`, `AKG:ASSESS:v1\0`, `AKG:BRANCH:v1\0`, `AKG:FINDING:v1\0`, `AKG:ENV:v1\0`, `AKG:ENVCOMP:v1\0`, `AKG:POLICY:v1\0`.

Signature domains:
`AKG:VERIFY-SIG:v1\0`, `AKG:ASSESS-SIG:v1\0`, `AKG:BRANCH-SIG:v1\0`.

---

## 25. Superseded constructs

O snapshot declara explicitamente como não normativos:
- `Finding.severity`
- `QualifiedClaimRef`
- `result_value`
- `FormalVerifier.domain_ref`
- `PolicyRef={digest}`
- `PolicyCore` inventado para PoC
- signature dentro do próprio digest core
- `assessment_digest` no próprio assessment preimage
- `state_digest` no próprio branch preimage
- `ExplicitOrdering` como CompositionEvidence
- inferência heurística de semantic dependency

---

## 29. Freeze gate

\[
B.0.2=FROZEN
\]
somente quando:
\[
VerificationMethodSubtypesClosed \land ResultDigestPreimageDefined \land PolicyDigestComputationDefined \land NormativeSnapshotMaterialized \land FindingPropagationAudit=PASS \land PolicyDigestPropagationAudit=PASS \land AllAKGDomainsByteExact \land AllFieldMatrixRowsComplete
\]

---

## 30. Selo desta revisão

\[
\boxed{ AKG\text{-}RFC\text{-}0002\ v0.1.3 = SEMANTICALLY\ CLOSED\ FOR\ PoC }
\]
\[
\boxed{ M6=CLOSED }
\]
\[
\boxed{ NORMATIVE\ SNAPSHOT = DRAFTED }
\]
\[
\boxed{ W02\text{-}08 = PASS\ CANDIDATE }
\]
\[
\boxed{ W02\text{-}04,\ W02\text{-}09 = BLOCKING }
\]
\[
\boxed{ B.0.2 = NOT\ FROZEN }
\]
\[
\boxed{ B.1 = BLOCKED }
\]
