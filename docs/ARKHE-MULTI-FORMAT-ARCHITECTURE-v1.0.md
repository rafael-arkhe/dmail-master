# ARKHE MULTI-FORMAT ARCHITECTURE v1.0 — FROZEN

## 1. Núcleo Congelado

> **The native core enforces. The WASM verifier independently recomputes verification. AKG-CBOR-v1 defines identity. Payloads are replaceable. UI is untrusted.**

A síntese concetual do Arkhe é: o kernel é invariante; a orquestração é substituível; as transições protegidas exigem evidência.

## 2. A Tríade de Formatos e a Fronteira de Confiança
A arquitetura materializa-se na seguinte combinação de veículos, eliminando a dependência de um único formato:

*   **Veículo do Kernel (`arkhe-core` nativo / CLI):** Exerce a autoridade fiduciária, interage com TEE e gere chaves. As integrações em interfaces como o **Tauri** são seguras por ACL, mas código executado aí tem acesso total; o Tauri atua como interface/veículo do kernel e não como sandbox WASM por omissão.
*   **Veículo de Verificação (`arkhe-verify-wasm`):** Distribuído como pacote **npm** (padrão emergente na indústria para verificadores WASM), efetua a verificação *read-only* determinística de forma soberana e isolada. O Arkhe adota e integra o modelo do **contentor WASM verificável** (inspirado na arquitetura RuVector) para compor crates WASM numa unidade selada que atua como a sua própria trilha de auditoria.
*   **Veículo de Evidência (AKG-CBOR-v1):** Baseado na RFC 8949 (CBOR determinístico). Define a identidade canónica dos objetos e os *digests* criptográficos com rigorosa separação de domínio (*domain separation*).
*   **Veículos de Artefactos Verificados (GGUF / Safetensors):**
    *   **GGUF:** Formato principal para modelos e inferência. Submetido a um **pipeline de 4 gates já especificado e publicado** (sanitização de metadados contra CVEs, verificação de *digests* duplos, assinaturas Sigstore *keyless* e verificação de provas RFC 6962 com quórum de *witnesses*).
    *   **Safetensors:** Incluído explicitamente como formato complementar, seguro, testado e verificado para treino e intercâmbio, suportado na indústria com ferramentas de assinatura (*modelsign* e OpenSSF OMS).

## 3. Invariantes de Freeze (Invariantes Arquiteturais)

As seguintes regras são normativas e encerram a discussão sobre a delegação de autoridade e a identidade dentro do sistema:

\[
\boxed{
StateTransitionEnforcementAuthority \subseteq NativeCore
}
\]

\[
\boxed{
SigningAuthority, RootAuthority \not\subseteq NativeCore
}
\]

\[
\boxed{
PortableVerifier \not\Rightarrow EnforcementAuthority
}
\]

\[
\boxed{
CanonicalBytes(x) = AKGCBORv1(WireProjection\_v1(x))
}
\]

\[
\boxed{
ObjectDigest(x) = H(DomainObject \parallel CanonicalBytes(UnsignedProjection(x)))
}
\]

\[
\boxed{
ImplementationParity \text{ is demonstrated by golden vectors, not inherited from WASM}
}
\]

## 4. Estrutura Não-Circular de Evidência
O Arkhe rejeita o embutimento de evidências (*embedding*) quando este introduz dependências circulares de hashes nos artefactos avaliados. A arquitetura de ligação correta é desacoplada, utilizando *content-addressing* em atestações e formatos *sidecar*:

\[
ModelArtifact \leftrightarrow ArtifactEnvelope \leftrightarrow EvidenceBundle
\]

**(Portable $\neq$ Embedded)**

## 5. TCB Final (Trusted Computing Base)

| Componente | TCB | Autoridade | Função |
| :--- | :--- | :--- | :--- |
| `arkhe-core` nativo | **Sim** | Enforcement | Kernel fiduciário; aplica políticas e exerce contenção |
| signer / HSM / root authority | **Sim** (externo) | Attestation / Authorization | Aplica chaves raiz e guarda segregada |
| `arkhe-verify.wasm` (via npm) | Não necessariamente | Read-only verification | Independently recomputes verification |
| AKG-CBOR-v1 | Protocolo | Define Identidade | Fornece representação canónica e *digests* independentes |
| Evidence producers | Não | Produzem claims | Entidades periféricas sem poder de decisão fiduciário |
| GGUF / Safetensors | Não | Payload | Objetos passivos que são alvo da verificação |
| Tauri / Web / CLI | Não | Interface (Untrusted) | Camada de apresentação e invocações locais |

## 6. Selos Finais

\[
\boxed{
ARKHE\ MULTI\text{-}FORMAT\ ARCHITECTURE\ v1.0 = FROZEN
}
\]

\[
\boxed{
AKG\text{-}RFC\text{-}0002\ SEMANTIC\ FREEZE = BLOCKED\ BY\ M6A + M6B
}
\]
*(Aguardando o fecho normativo final dos blockers `CompositionClosure` determinística e `StrongStateCommutation` substitutiva antes de avançar empiricamente para os vetores de conformidade B.1 e B.2)*
