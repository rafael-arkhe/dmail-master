# AKG-RFC-0002 v0.1.3 — Wire Closure Patch v3 (Final Preconditions)

## WC3-1 — VerificationMethod subordinate types
`profile_ref`, `obligation_ref`, `projection_ref`, `ruleset_ref`, `expected_result` e `actual_result` MUST possuir definição normativa própria antes de qualquer wire projection. `expected_result` e `actual_result` MUST NOT ser substituídos por `result_value` sem amendment explícito. O `ContractualVerifier` existente realmente os trata como tipos nominalmente distintos.

Além disso:
\[
result\_digest \Rightarrow ExactResultDigestPreimage
\]
antes do freeze.

## WC3-2 — PolicyDigest
`PolicyDigest` MUST bind the exact resolved policy artifact. Sua computação MUST ser definida pelo `PolicyResolutionProfile`. B.0.2 MUST NOT presumir a existência de `PolicyCore`.

Então:
\[
ResolvedPolicyDigest \neq BoundPolicyDigest \Rightarrow REJECT
\]
continua intacto, mas sem preimage fictício.

## WC3-3 — ClaimRef
Wire v1: `ClaimRef = text_id`. Nenhuma regra nova de namespace é criada por B.0.2. O significado da referência permanece determinado pelo contexto normativo que contém a request.

## WC3-4 — Snapshot requirement
Antes dos propagation audits:
\[
NormativeSnapshot = RFC0001_{frozen} + RFC0002_{v0.1.3} + WirePrerequisiteClarifications + WireClosurePatches
\]
com todo texto marcado `historical`, `superseded`, `draft`, `research note` ou versão anterior excluído da superfície normativa. Então os audits executam **somente** sobre `NormativeSnapshot`.

## Freeze gate final revisado
B.0.2 = FROZEN se e somente se:
\[
VerificationMethodSubtypesClosed \land ResultDigestPreimageDefined \land PolicyDigestComputationDefined \land NormativeSnapshotBuilt \land FindingPropagationAudit(snapshot) = PASS \land PolicyDigestPropagationAudit(snapshot) = PASS \land AllAKGDomainsByteExact \land AllFieldMatrixRowsComplete
\]

Então:
\[
B.0.2 = FROZEN
\]
\[
B.1 = UNBLOCKED
\]

## Selo actual
\[
B.0.2 = WIRE CLOSURE NEAR COMPLETE; NORMATIVE SNAPSHOT REQUIRED
\]
\[
B.1 = BLOCKED
\]
