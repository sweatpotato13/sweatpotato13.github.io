---
layout: post
title: "ZK Whiteboard Sessions - Module Two 정리"
date: "2023-03-06 12:34:56 +0900"
tags: [zk]
---

본 문서는 ZK Whiteboard Sessions - Module Two를 보고 정리한 문서입니다. zk에 대한 이해가 완벽하지 않아서 잘못 이해하고 작성한 부분이 있을 수 있습니다.

[youtube](https://www.youtube.com/watch?v=J4pVTamUBvU)

[Google Slide](https://docs.google.com/presentation/d/1AHEedUHtLcaAzu9isuUI9oBiPEu2TnrYbStkPbk__2E/edit)

## Building an efficient SNARK

1. A functional commitment scheme ⇒ cryptographic object
2. A suitable interactive oracle proof (IOP) ⇒ information theoretic object

(1) + (2) ⇒ (zk)SNARK for general circuits

## Commitment

Cryptographic commitment: emulates an envelope

**Algorithms:**

- $$commit(m,r) \rightarrow com$$ ($$r$$ choose at random)
- $$verify(m,com,r) \rightarrow$$ accept / reject

**Properties:**

- binding: commitment에 대해서 2가지의 유효한 \*opening이 존재하지 않는다.
- hiding: commitment는 data에 대해서 아무런 정보도 공개하지 않음

<font size="3">opening: 메시지 전부 혹은 일부를 공개하는 것</font>

## **Committing to a Value**

Fix a hash function $$H: M\times R \rightarrow C$$

- $$commit(m,r): com:= H(m,r)$$
- $$verify(m,com,r):$$ accept if $$com=H(m,r)$$

## **Committing to a Function**

Choose a family of functions $$F = \{ f: X \rightarrow Y \}$$

![Untitled](https://i.imgur.com/sjG8P87.png)

Prover가 Function family에 속하는 $$f$$에 대해서 commit 하려고 한다면 이를 $$com_f$$를 사용하여 commit 할 수 있고 Verifier는 $$com_f$$를 저장한다.

나중에 Verifier가 $$x$$에 대해서 $$f(x)$$값을 요구할때 Prover는 $$y$$와 함께 Proof $$\pi$$값을 주어 Verifier가 이를 검증할 수 있도록 한다.

Proof $$\pi$$는 $$f(x)=y$$ and $$f \in F$$ 임을 검증한다.

## **Committing to a Function: Syntax**

A functional commitment scheme for $$F$$:

- $$setup(\lambda) \rightarrow pp$$ (outputs public parameters $$pp)$$
- $$commit(pp, f,r) \rightarrow com_f$$

  - commitment to $$f \in F$$ with $$r \in R$$ a binding (and \*optionally hiding) commitment scheme for $$F$$

  <font size="3">optionally hiding: ZK일 경우 hiding까지 만족 해야함</font>

- $$eval$$(Prover P, verifier V): 주어진 $$com_f$$ 와 $$x \in X,$$ $$y \in Y$$에 대해서:

  P($$pp, f,x,y,r) \rightarrow$$ short proof $$\pi$$

  V($$pp, com_f, x,y, \pi) \rightarrow$$ accept/reject

- a SNARK for the relation:
  $$f(x) =y$$ and $$f \in F$$ and $$commit(pp,f,r)= com_f$$

## **Examples of Functional Commitments**

**Polynomial commitments:**

- Committing to a univariate polynomial $$f(x)$$ in $$\mathbb{F}_p^{(\leq d)}[X]$$
  - univariate polynomials of degree at most $$d$$

**Multilinear commitments:**

- Commiting to a multilinear polynomial in $$\mathbb{F}_p^{(\leq 1)}[X_1,...,X_k]$$

**Linear commitments:**

- Committing to a linear function $$f_{\overrightarrow{v}} (\overrightarrow{u})=<\overrightarrow{u},\overrightarrow{v}> = \Sigma^n_{i=i}u_iv_i$$

## **Polynomial Commitment Schemes (PCS)**

A PCS is a functional commitment for the family $$F = \mathbb{F}_p^{(\leq d)}[X]$$

⇒ prover 는 단일 미지수 방정식 $$f$$ in $$\mathbb{F}_p^{(\leq d)}[X]$$ 를 commit할 수 있고, 이후에 public $$u,v \in \mathbb{F}_p$$에 대한 $$v = f(u)$$ 를 검증할 수 있다.

## **KZG PCS Setup & Commitment**

KZG (Kate-Zaverucha-Goldberg’2010)

Group $$\mathbb{G} := \{0,G, 2*G,...,(p-1)*G \}$$ of order $$p$$

$$setup(\lambda) \rightarrow pp$$:

- Sample random $$\alpha \in \mathbb{F}p$$
- $$pp=(H_0=G, H_1=\alpha*G, ..., H_d=\alpha^d*G)\in \mathbb{G}^{d+1}$$
- \*delete $$\alpha$$ (trusted setup)

  <font size="3">α를 지우지 않으면 유츌 되었을 때 이를 알고있는 다른 사람들이 가짜 proof를 생성할 수 있음</font>

$$commit(pp,f) \rightarrow com_f$$ where $$com_f := f(\alpha)*G \in \mathbb{G}$$

- $$f(X) = f_0+f_1X+...+f_dX^d \rightarrow com_f = f_0*H_0+...+f_d*H_d$$

- $$= f_0*G+f_1\alpha*G+...=f(\alpha)*G$$

=> <font size="3">α 없이 public parameter만 가지고 commitment를 검증할 수 있음</font>

## **KZG PCS Evaluation**

$$eval$$:

Prover($$pp,f,u,v)$$

Verifier($$pp,com_f,u,v)$$

Goal: prove $$f(u)=v$$

$$f(u)=v \Leftrightarrow u$$
is a root of $$\hat{f} = f-v \Leftrightarrow
 (X-u)$$
divides $$\hat{f}$$

$$\Leftrightarrow$$ exists $$q \in \mathbb{F}_p[X]$$ such that $$q(X)*(X-u) = f(X)-v$$

compute $$q(X)$$ and $$com_q$$ → send $$\pi := com_q \in \mathbb{G}$$ to verifier

accept if $$(\alpha-u)*com_q = com_f-v*G$$

=> <font size="3">Verifier는 α를 알지 못하는데 어떻게 이를 계산하여 검증할까? => pairing을 사용하여 계산할 수 있음</font>

## **KZG PCS Generalizations**

**Generalizations:**

- KZG for committing to k-variate polynomials, buy eval proof size is k group elements
- **Batch proofs**
  - suppose verifier has commitments .$$com_{f1}, ... com_{f_n}$$
  - prover wants to prove $$f_i(u_{i,j})=v_{i,j}$$ for $$i \in [n], j \in [m]$$
    ⇒ batch proof $$\pi$$ is one or two group elements

## **Dory PCS**

Difficulties with KZG: trusted setup for $$pp$$, and $$pp$$ size is linear in $$d$$

**Dory:**

- **transparent setup**: no secret randomness in setup
- $$com_f$$ is a single group element (independent of degree $$d)$$
- evel proof size for $$f \in \mathbb{f}_p^{(\leq d)}[X]$$ is $$O(logd)$$ group elementes
- eval verify time is $$O(logd)$$
- Prover time: $$O(d)$$

## **Polynomial IOP Setup**

Let $$C(x,w)$$ be some arithmetic circuit.

Let $$x \in \mathbb{F}_p^n$$

**Poly-IOP:** a proof system that proves $$\exists w:C(x,w) =0$$ as follows:

- $$setup(C) \rightarrow$$ public parameters $$S_p$$ and $$S_v =(f_0, f_{-1},...,f_{-s})$$

## **Polynomial IOP Evaluation**

![Untitled](https://i.imgur.com/BhgTc9q.png)

## **Polynomial IOP Properties**

- Complete: if $$\exists w:C(x,w)=0$$ thet verifier always accepts
- Knowledge sound: (informal) Let $$x \in \mathbb{F}_p^n$$
  for every $$P^*$$ that convince the verifier with prob. $$\geq 1/10^6$$
  there is an efficient extractor $$E$$ such that
  $$Pr[E(x,f_1,r_1,...r_{t-1},f_t) \rightarrow w]$$ such that $$C(x,w)=0] \geq 1/10^e -\epsilon$$

- Optional: zero-knowledge

## **The Resulting SNARK**

$$(t,q)$$ Poly-IOP: $$t:=$$ number of polys. committed,

$$q :=$$ number of eval queries in verify ( usually $$t,q \leq 3$$ )

The SNARK:

- Prover sends t poly commitments
- During poly-IOP verify: run PCS evel protocol q times
- Use **flat-Shamir transform** to make the proof system non-interactive

- Length of SNARK proof: t poly-commits + q evel proofs
- Verifier time: q \* time(evel verify) + time(IOP-verify)
- Prover time: t \* time(commit) + q \* time(prove) + time(IOP-prover)
