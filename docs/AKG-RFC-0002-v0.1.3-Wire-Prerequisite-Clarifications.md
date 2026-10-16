# AKG-RFC-0002 v0.1.3 — Wire Prerequisite Clarifications

**Status:** Normative amendment to AKG-RFC-0002 v0.1.3
**Escopo:** Sete (7) cláusulas que resolvem os blockers B02-01 a B02-12 do Appendix B.0.2, sem introduzir nova semântica.
**Base semântica:** AKG-RFC-0002 v0.1.3 — *Semantically Closed for PoC*
**Disciplina:** Esta emenda transforma semântica congelada em bytes. Não usa os bytes como lugar para escolher uma nova semântica.


## Cláusula 1 — `policy_ref`: Identidade é o Digest

**Blockers resolvidos:** B02-01

**Regra:** O digest de conteúdo é a identidade normativa de uma política. `policy_id` e `policy_version` humanos são metadados não-identitários.

**WireProjection_v1_PolicyRef:**
```text
policy_ref = {
  digest : bstr32   ; BLAKE3-256(DomainPolicy || AKG-CBOR-v1(PolicyCore))
}
```

**Definições:**
- `PolicyCore` é a projeção não-assinada do documento de política, excluindo assinatura e envelope.
- `DomainPolicy` é a string literal `"AKG-POLICY-v1"`.
- Duas `policy_ref` com o mesmo digest são a mesma política. Duas com digests diferentes são políticas diferentes, independentemente de `policy_id`/`version` humanos.

**Fonte normativa:** SCITT *Canonical Payload Binding*: "a canonical form of structured content, a content-addressed identifier derived from that form". `aa-policy`: "A content digest is unambiguous in both cases and needs no trusted version field".

**Status:** `RESOLVED`


## Cláusula 2 — `identity_ref`: Uma Única Projeção

**Blockers resolvidos:** B02-02

**Regra:** `identity_ref` tem uma única representação wire canónica. A variante é determinada exclusivamente pelo campo `scheme`, nunca inferida do formato do `value`.

**WireProjection_v1_IdentityRef:**
```text
identity_ref = {
  scheme : text,   ; "did", "uri", "key", "spize", etc. (1..64 bytes, NFC)
  value  : text    ; identificador canónico (1..1024 bytes, NFC)
}
```

**Definições:**
- `scheme` é obrigatório e case-insensitive-normalizado para lowercase NFC.
- `value` é NFC normalizado.
- Nenhuma tagged union alternativa é permitida em v1. A forma `{scheme, value}` é a projeção única.

**Fonte normativa:** DID Core CBOR: "All properties of the DID document represented in CBOR MUST be included in the root map (major type 5)". A forma `{scheme, value}` é compatível com o modelo de dados CBOR map com chaves textuais.

**Status:** `RESOLVED`


## Cláusula 3 — `property_ref`: Importação Literal da RFC-0001

**Blockers resolvidos:** B02-03

**Regra:** `property_ref` é importado literalmente da RFC-0001. A projeção wire é a que a RFC-0001 congela.

**WireProjection_v1_PropertyRef (condicional à verificação literal):**
```text
property_ref = {
  name    : text_name,   ; NFC, 1..256 bytes UTF-8
  version : uint32       ; versão exacta
}
```

**Definições:**
- Igualdade: `name_A = name_B ∧ version_A = version_B`.
- Nenhuma compatibilidade entre versões é inferida em v1.
- Se a RFC-0001 contiver literalmente esta estrutura, a importação é pura e esta cláusula fecha sem emenda.

**Fonte normativa:** RFC-0001 §5.2 (verificação literal pendente).

**Status:** `CLOSE AFTER LITERAL RFC-0001 VERIFICATION`


## Cláusula 4 — `scope_expr`: Importação Literal da RFC-0001

**Blockers resolvidos:** B02-04

**Regra:** `scope_expr` é importado literalmente da RFC-0001 como tagged union.

**WireProjection_v1_ScopeExpr (condicional à verificação literal):**
```text
scope_expr = tagged union sobre "kind":

  { "kind": "component",    "value": text_scope }
| { "kind": "artifact_set", "values": [text_scope] }  ; semantic set
| { "kind": "symbol_set",   "values": [text_scope] }  ; semantic set
```

**Definições:**
- `kind` é sempre obrigatório.
- `values` nunca é vazio; semantic set ordenado por UTF-8 NFC bytes.
- Nenhuma variante nova pode ser inventada em B.0.2.

**Fonte normativa:** RFC-0001 §8.1 (verificação literal pendente). CBOR tagged unions: "alternatives look structurally the same in terms of CBOR data items, but still need to be distinguished" através de um discriminador explícito.

**Status:** `CLOSE AFTER LITERAL RFC-0001 VERIFICATION`


## Cláusula 5 — `Finding` e `FindingDisposition`: Separação Factual/Política

**Blockers resolvidos:** B02-05

**Regra:** `Finding` é a observação factual imutável do verificador. `FindingDisposition` é a classificação derivada pela política. São objetos distintos, com identidades distintas.

**WireProjection_v1_Finding:**
```text
finding = {
  code          : text_id,        ; código factual do verificador
  category      : enum text,      ; MALFORMED_INPUT, EVIDENCE_MISSING, etc.
  subject_ref   : subject_ref,    ; intent, claim, evidence, external
  evidence_refs : [EvidenceRef],  ; semantic set
  details_ref   : digest?         ; opcional
  ; severity NÃO pertence ao finding factual
}
```

**WireProjection_v1_FindingDisposition:**
```text
finding_disposition = {
  finding_ref   : digest,         ; FindingDigest = H(DomainFinding || CanonicalCBOR(Finding))
  disposition   : enum text,      ; INFO | WARNING | BLOCKING
  policy_ref    : policy_ref,     ; política que produziu a disposição
  rationale_ref : digest?         ; razão documentada
}
```

**Definições:**
- `FindingDigest` substitui qualquer `finding_id` textual.
- `BLOCKING` é uma disposição, não um atributo do finding.
- `Finding ≠ FindingDisposition`.

**Fonte normativa:** Separação factual/política estabelecida no hardening da RFC-0002. `gemma.msl.ubc.ca`: "A curator's ruling on ONE finding inside an audit".

**Status:** `RESOLVED` (requer clarificação semântica explícita na RFC-0002)


## Cláusula 6 — `EnvironmentDescriptor`: Perfil CoRIM

**Blockers resolvidos:** B02-06

**Regra:** `EnvironmentDescriptor` é uma composição de `EnvironmentProfile` e componentes content-addressed, usando measurements CoRIM-style.

**WireProjection_v1_EnvironmentDescriptor:**
```text
environment_descriptor = {
  environment_version : uint16,
  os                  : identity_ref?,
  architecture        : identity_ref?,
  runtime             : identity_ref?,
  toolchain           : identity_ref?,
  tee_type            : enum text?,    ; INTEL_SGX, INTEL_TDX, AMD_SEV_SNP, ARM_TRUSTZONE, GPU_TEE, NONE
  measurements        : [measurement], ; array de measurements
  container_image     : digest?,
  dependencies_digest : digest?,
  config_digest       : digest?,
  environment_digest  : digest         ; BLAKE3-256(DomainEnv || CanonicalCBOR(env_core))
}

measurement = {
  mkey        : text_id,      ; measurement key
  mval        : digest,       ; measurement value
  algorithm   : enum text,    ; SHA-256, BLAKE3, etc.
  authorized_by : identity_ref?
}
```

**Definições:**
- `environment_digest` é externo ao core que identifica.
- Campos irrelevantes podem ser omitidos.
- Ausência de informação necessária para determinar equivalência ⇒ `INCONCLUSIVE`.

**Fonte normativa:** CoRIM: "mkey (index 0): An optional measurement key... mval (index 1): The measurements associated with the environment". SLSA Provenance v0.2 define o campo `environment`.

**Status:** `RESOLVED`


## Cláusula 7 — `SignatureEnvelope` e `BranchAttestation`

**Blockers resolvidos:** B02-11, B02-12

### 7.1 — `SignatureEnvelope`

**Regra:** A assinatura é calculada sobre `SignatureDomain || ObjectDigest`, conforme M12. COSE_Sign1 não é usado na PoC.

**WireProjection_v1_SignatureEnvelope:**
```text
signature_envelope = {
  algorithm_profile : enum text,    ; AKG-SIG-ED25519-v1
  key_ref           : identity_ref,
  signature_bytes   : bstr          ; Sign(key, SignatureDomain || ObjectDigest)
}
```

**Definições:**
- Bounds: máximo 4096 bytes para `signature_bytes`.
- Ed25519: assinatura de 64 bytes, chave pública de 32 bytes, determinística.
- COSE_Sign1 pode tornar-se um signature profile posteriormente, mas requer especificação explícita de como `Domain || Digest` entra na `Sig_structure`.

**Fonte normativa:** M12: `SignedMessage = SignatureDomain || ObjectDigest`. RFC 9052: COSE_Sign1 assina `Sig_structure = ["Signature1", protected, external_aad, payload]`.

### 7.2 — `BranchAttestation`

**Regra:** O core do `BranchIntegrationState` não inclui assinatura. A atestação é um objeto separado, content-addressed.

**WireProjection_v1_BranchIntegrationStateCore:**
```text
branch_integration_state_core = {
  branch_id               : text,
  state_version           : uint16,
  integrated_intents      : [digest],           ; semantic set
  dependency_graph        : [dependency_edge],  ; semantic set
  explicit_ordering_edges : [ordering_edge],    ; semantic set
  integration_frontier    : [digest],           ; semantic set
  policy_ref              : policy_ref
}
```

**WireProjection_v1_BranchAttestation:**
```text
branch_attestation = {
  branch_digest : digest,           ; BranchDigest = H(DomainBranch || CanonicalCBOR(BranchCore))
  signer        : identity_ref,
  signature     : signature_envelope
}
```

**Definições:**
- O mesmo `BranchDigest` não muda apenas porque um deployment resolveu atestá-lo.
- A atestação é opcional.

**Fonte normativa:** M12 (digest externo ao core). Content-addressing: "This is a BLAKE3 content-addressed hash chain, NOT an Ed25519-signed OutcomeChain... so the chain is tamper-evident by content-addressing".

**Status:** `RESOLVED`


## Resumo das Emendas

| Cláusula | Blockers | Tipo | Status |
|:---|:---|:---|:---|
| 1 | B02-01 | `policy_ref` | `RESOLVED` |
| 2 | B02-02 | `identity_ref` | `RESOLVED` |
| 3 | B02-03 | `property_ref` | `CLOSE AFTER RFC-0001 VERIFICATION` |
| 4 | B02-04 | `scope_expr` | `CLOSE AFTER RFC-0001 VERIFICATION` |
| 5 | B02-05 | `Finding`/`FindingDisposition` | `RESOLVED` (clarificação) |
| 6 | B02-06 | `EnvironmentDescriptor` | `RESOLVED` |
| 7 | B02-11, B02-12 | `SignatureEnvelope`/`BranchAttestation` | `RESOLVED` |

**Blockers já fechados por M6/M12:** B02-07, B02-08, B02-09, B02-10.

**Preservação crítica:** `DomainAssess ≠ DomainAssessment`. Em domain separation, um byte diferente significa um protocolo diferente.


## Freeze Gate para B.0.2

B.0.2 recebe `FROZEN` somente quando:

```
∀ b ∈ B02: ExactlyOneNormativeSource(b)
         ∧ ExactlyOneSemanticDefinition(b)
         ∧ ExactlyOneWireProjection(b)
         ∧ NoContradictoryAlternative(b)
         ∧ NoWireDecisionCreatesSemantics(b)
```

E adicionalmente:

```
AllDigestPreimagesDefined
∧ AllSignatureMessagesDefined
∧ AllSemanticSetSortKeysDefined
∧ AllOptionalityRulesDefined
∧ AllBoundsDefined
```

Depois de verificar literalmente `property_ref` e `scope_expr` na RFC-0001:

```
B.0.2 = FROZEN
B.1 → B.2
```


## Selos Atuais

```
AKG-RFC-0002 v0.1.3 = SEMANTICALLY CLOSED FOR PoC
M6 = CLOSED
B.0.2 = DESIGN CONVERGED; NORMATIVE PREREQUISITES PENDING
B.1 = BLOCKED ON B.0.2 FREEZE
```
