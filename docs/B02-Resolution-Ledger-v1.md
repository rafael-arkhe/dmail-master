# B.0.2 Resolution Ledger v1

**Profile:** `AKG-CBOR-v1`
**Semantic baseline:** AKG-RFC-0002 v0.1.3 — `SEMANTICALLY CLOSED FOR PoC`
**Discipline:** B.0.2 classifica a semântica já congelada; não inventa nova semântica.

| Blocker | Normative Source | Chosen Existing Semantic Type | Exact WireProjection | Status |
|:---|:---|:---|:---|:---|
| B02-01 | RFC-0002 §Policy | `PolicyCore` | `{ digest: bstr32 }` | OPEN |
| B02-02 | RFC-0002 §Identity | `IdentityRef` (tagged union) | A importar da RFC-0001 | OPEN |
| B02-03 | RFC-0001 §5.2 | `{name, version}` | map 2 campos | CLOSE after literal RFC-0001 verification |
| B02-04 | RFC-0001 §8.1 | tagged union | map com `kind` | CLOSE after literal RFC-0001 verification |
| B02-05 | RFC-0002 v0.1.3 | separação factual/policy | dois objetos | OPEN |
| B02-06 | RFC-0002 v0.1.3 §? | measurements vs. named fields | indefinida | OPEN |
| B02-07 | M6 | `OrderingEdge` separado | array em Branch | CLOSED |
| B02-08 | M6 + RFC-0002 | mínimo vs. rico | indefinida | OPEN |
| B02-09 | M12 | removido do core | digest externo | CLOSED |
| B02-10 | M12 | removido do core | digest externo | CLOSED |
| B02-11 | M12 + profile | envelope Arkhe | map 3 campos | OPEN |
| B02-12 | M12 | `BranchAttestation` opcional | objeto separado | CLOSED |
