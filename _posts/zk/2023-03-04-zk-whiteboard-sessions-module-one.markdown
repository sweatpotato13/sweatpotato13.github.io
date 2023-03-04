---
layout: post
title: "ZK Whiteboard Sessions - Module One 정리"
date: "2023-03-04 12:34:56 +0900"
tags: [zk]
---

본 문서는 ZK Whiteboard Sessions - Module One을 보고 정리한 문서입니다. zk에 대한 이해가 완벽하지 않아서 잘못 이해하고 작성한 부분이 있을 수 있습니다.

[https://www.youtube.com/watch?v=h-94UhJLeck](https://www.youtube.com/watch?v=h-94UhJLeck)

## Definition: SNARK

SNARK: 어떤 명제가 참임을 간결하게 증명 하는 것

zk-SNARK: 어떤 명제가 참인것에 대해 정보를 공개하지 않고 간결하게 증명할 수 있는 것

## Applications of SNARKs

- Private Transaction on Public Blockchain
  - Tornado cash, Zash …
- Private Dapps
  - Aleo
- Scalability: 유효성을 갖춘 Rollup 시스템

## Arithmetic circuits

- Fix a finite field F = {0, … p-1} for some prime p > 2
- Arithmetic circuit
  - Circuit $$C: F^n \rightarrow F$$
  - 그래프에서 노드가 `+`, `-`, `x` 이며 입력이 1, $$x_1$$,…,$$x_n$$ 인 directed acyclic graph (DAG)
  - 이는 n차 다항식을 정의 합니다.
  - $$\|C\|=$$ number of gates in $$C$$
- Example
  - $$C_{hash}(h,m)$$: Output 0 if SHA256($$m$$) = $$h$$, and $$\neq 0$$ otherwise
  - $$C_{hash}(h,m)=$$ ($$h$$ - SHA256($$m$$)), $$\|C_{hash}\| =$$ 20K gates

## Argument Systems

- Public arithmetic circuit: $$C:(x,w) \rightarrow F$$
  - $$x$$: public statement in $$F^n$$
  - $$w$$: secret witness in $$F^m$$

![Argument Systems](https://i.imgur.com/QuxpsiX.png)

## Non-interactive Preprocessing Argument Systems

- Preprocessing(setup): $$S(c)$$ → public parameters $$(S_p, S_v)$$

![Non-interactive Preprocessing Argument Systems](https://i.imgur.com/uHSTz1i.png)

## Preprocessing Argument System

- A preprocessing argument system is a triple $$(S,P, V)$$
- $$S(C)$$ → public parameters$$(S_p, S_v)$$ for prover and verifier
- $$P(S_p,x,w)$$ → proof $$\pi$$
- $$V(S_{v}  ,x, \pi)$$ → Accept or Reject

## Argument System Requirements

![Argument System Requirements](https://i.imgur.com/bZEhcF3.png)

**Complete**: 모든 $$C(x,w) = 0$$을 만족 시키는 $$x,w$$에 대하여 $$V(S_v,x, P(S_p,x,w))$$ = accept 일 확률은 1이다.

**Knowledge Sound**: Verifier가 Proof를 Accepts 했다면 Prover는 $$C(x,w) = 0$$ 을 만족시키는 $$w$$을 알고 있다고 할 수 있다.

→ $$P$$가 $$w$$를 모를 때 $$V(S_v, x, \pi$)$$ = Accept 일 확률은 무시할만큼 낮다

Optional: **Zero Knowledge:** $$(C, S_p, S_v, x, \pi)$$ 는 $$w$$에 대한 어떠한 정보도 노출하지 않는다.

## Succintness

- SNARK: a Succinct ARgument of Knowledge
  a succint preprocessing argument system is a triple $$(S, P, V)$$
- $$S(C)$$ → public parameters $$(S_p, S_v)$$ for prover and verifier
- $$P(S_p, x,w)$$ → **short** proof $$\pi; \|\pi\|=0(log(\|C\|,\lambda)$$
- $$V(S_v, x, \pi)$$ → **fast to verify : $$time(V) =0(\|x\|, log(\|C\|), \lambda)$$**

SNARK: $$(S, P, V)$$ is complete, knowledge sound, succint

zk-SNARK: $$(S, P, V)$$ is a SNARK and is zero knowledge

## Not a SNARK: Trivial Argument System

- Prover sends w to verifier
- Verifier checks if $$C(x,w) = 0$$ and accpets if so

**Problems with this**

- $$w$$ might be secret: prover는 $$w$$를 공개하길 원하지 않음
- $$w$$ might be long: 간결한 proof를 원함
- computing $$C(x,w)$$ may be hard: 빠르게 검증하기를 원함

## Motivating Question: how is $$O(log n)$$ Possible?

![Motivating Question](https://i.imgur.com/yLj0RF6.png)

## Types of Setup

$$S(C;r)$$ → public parameters $$(S_p, S_v)$$

$$r$$: random bits

### Types of setup

**trusted setup per circuit.:** $$S(C;r)$$ random $$r$$ must be kept secret from prover

prover learns $$r$$ → 잘못된 statement에 대해서도 검증 할 수 있게 됨

**trusted but universal (updatable) setup:** secret $$r$$ is independent of $$C$$

$$S = (S_{init}, S_{index}): S_{init}(\lambda ;r) → pp, S_{index}(pp, C) → (S_p, S_v)$$

**transparent setup :** $$S(C)$$ does not use secret data ( trusted setup 없음 )

## SNARK Landscape

![Untitled](https://i.imgur.com/t6kpnzR.png)

## SNARK Software Ecosystem

![Untitled](https://i.imgur.com/KRWdGTK.png)

## Definition of Knowledge Sound

Goal: If $$V$$ accepts then $$P$$ knows $$w$$ s.t. $$C(x,w) = 0$$

$$w$$를 안다는것이 무슨 의미일까

**Formally**: $$(S, P, V)$$ is knowledge sound for a circuit $$C$$ if

for event poly. time adversary $$A = (A_0, A_1)$$ such that

\\[ S(C) \rightarrow (S_p, S_v), \\]
\\[ (x,state) \leftarrow A_1(S_p), \\]
\\[ pi \leftarrow A_1(S_p, x, state) : \\]
\\[ Pr[V(S_v,x,\pi) = accept] > 1 / 10^6 (non-negligible) \\]

there is an efficient **extractor** $$E$$ (that uses $$A_1$$ as a block box) s.t.

\\[ S(C) \rightarrow (S_p, S_v), \\]
\\[ (x,state) \leftarrow A_0(S_p), \\]
\\[ w \leftarrow E^{A_1}(S_p,x,state)(S_p, x) : \\]
\\[ Pr[C(x,w)=0] > 1 / 10^6 - \epsilon \\] (for a negligible $$\epsilon$$)

⇒ Poly. time adversary $$A$$가 정상적으로 state로 부터 proof $$\pi$$를 생성할 수 있다면 $$C(x,w) = 0$$을 만들 수 있는 유효한 witness를 추출할 수 있는 extractor를 갖고 있다고 할 수 있다.

## Definition of Zero Knowledge

$$(S, P, V)$$ is zero knowledge if for every $$x \in F^n$$, proof $$\pi$$ reveals nothing about $$w$$, other than its existence

$$(S, P, V)$$ is zero knowledge if there is an efficient alg, **Sim** such that $$(S_p, S_v, \pi) ← Sim(C,x)$$ look like the real $$S_p, S_v$$ and $$\pi$$

**Formally:** $$(S, P, V)$$ is zero knowledge for a circuit $$C$$

if there is an efficient simulator **Sim** such that for all $$x \in F^n$$ s.t $$∃ w: C(x,w) =0$$ the distribution :

$$(C, S_p, S_v, x, \pi):$$ where $$(S_p, S_v) ← S(C)$$, $$\pi ← P(S_p, x, w)$$

is indistinguishable form the distribution :

$$(C, S_p, S_v, x, \pi):$$ where $$(S_p, S_v, \pi) ← Sim(C,x)$$

---
