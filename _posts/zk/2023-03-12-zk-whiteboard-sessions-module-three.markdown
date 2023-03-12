---
layout: post
title: "ZK Whiteboard Sessions - Module Three 정리"
date: "2023-03-12 12:34:56 +0900"
tags: [zk]
---

본 문서는 ZK Whiteboard Sessions - Module Three를 보고 정리한 문서입니다. zk에 대한 이해가 완벽하지 않아서 잘못 이해하고 작성한 부분이 있을 수 있습니다.

[youtube](https://www.youtube.com/watch?v=vxyoPM2m7Yg)

[Google Slide](https://docs.google.com/presentation/d/1fj7_rwSM5m9_70CAw7T7Y0YS7Q4-rTH089mvFw591cg/edit#slide=id.g13ea2f20f32_0_0)

## **Goal: Construct a poly-IOP called a “Plonk”**

- Plonk + PCS ⇒ SNARK (and also a zk-SNARK)

## Useful observation

- for $$0 \neq f \in \mathbb{F}_p^{(\leq d)}[X]$$ <br>
  for $$r \leftarrow \mathbb{F}_p$$: $$Pr[f(r) =0] \leq d/p$$ (\*)
  <br><font size="3">최대 d차를 갖는 다항식 f에 대하여 f(r)=0 일 확률은 d/p보다 낮다.</font><br>
  ⇒ $$p \approx 2^{256}$$ 이고 $$d \leq 2^{40}$$ 이라고 가정했을 때 $$d/p$$는 무시할만큼 낮다. <br>
  ⇒ 임의의 $$r \leftarrow \mathbb{F}_p$$: 에 대하여 $$f(r)=0$$ 이라면 $$f$$는 높은 확률로 zero라고 추정할 수 있다. <br>
  ⇒ polynomial commitment에 대한 간단한 zero 테스트 <br>
  - **SZDL lemma: multivariate polynomial에 대해서도 (\*) 부분의 식이 성립함**
    - ($$d$$ 는 $$f$$의 차수의 합)
  - SNARK에서 주로 사용하는 테크닉

## Related observation

- $$p \approx 2^{256}$$ 이고 $$d \leq 2^{40}$$ 이라고 가정했을 때 $$d/p$$는 무시할만큼 낮다.
  - Let $$f,g \in \mathbb{F}_p^{(\leq d)}[X]$$ <br>
    임의의 $$r \leftarrow \mathbb{F}_p$$에 대해서 $$f(r)=g(r)$$ 이라고 하면 $$f=g$$ 라고 높은 확률로 추정할 수 있다. <br>
    → $$f(r)-g(r)=0$$ ⇒ $$f-g=0$$ <br>
    ⇒ polynomial commitment에 대한 간단한 equal 테스트

## Useful proof gadgets

- Let $$\omega \in \mathbb{F}_p$$ be a primitive $$k$$-th root of unity ($$\omega^k=1$$) <br>
  Set $$H := \{1, \omega, \omega^2 ,...,\omega^{k-1}\} \subseteq \mathbb{F}_p$$ <br>
  Let $$f \in \mathbb{F}_p^{\leq d}[X]$$ and $$b,c \in \mathbb{F}_p$$. ($$d \geq k)$$ <br>
  There are efficient poly-IOPs for the following tasks
  - Zero-test: prove that $$f$$ is identically zero on $$H$$
  - Sum-check: prove that $$\Sigma_{a \in H}f(a) =b$$ (verifier has $$\fbox{f},b$$)
  - Prod-check: prove that $$\Pi_{a\in H}f(a)=c$$ (verifier has $$\fbox{f},c$$)
    <br><font size="3">box씌운 f는 f에대한 commitment입니다.</font><br>

## Zero test on $$H$$

- Prover P($$f,\bot$$)
- Verifier V($$\fbox{f}$$)
- Lemma: $$f$$ is zero on $$H$$ if and only if $$f(X)$$ is divisible by $$X^k-1$$

- Prover는 $$q(X) \leftarrow f(X)/(X^k-1)$$ 를 계산하고 $$q \in \mathbb{F}_p^{(\leq d)}[X]$$ 에 대한 commitment를 Verifier에게 전달한다
- Verifier는 $$r \leftarrow \mathbb{F}_p$$ 를 랜덤하게 선택한다.
- Verifier는 $$r$$ 에 대해서 $$q(X), f(X)$$ 를 Prover에게 질의한다. 이 과정에서 $$q(r), f(r)$$을 획득
- Verifier는 $$f,q$$에 대한 commitment를 가지고 있기 때문에 이를 검증할 수 있다.
- Verifier는 $$f(r)=q(r)(r^k-1)$$ 임을 확인한다.

  - (implies that $$f(X)=q(X)(X^k-1)$$)
  - 이를 만족한다면 Lemma에 의해 $$f$$는 $$H$$에서 zero

- **Thm:** this protocol is complete and sound, assuming $$d/p$$ is negligible
- proof size: one commitment($$\fbox{q}$$) + two open proof($$q(X), f(X)$$)
- Verifier time: $$O(logk)$$ and two eval verify (but can be done in one)

- Prover가 $$q$$에 대한 commitment를 생성하기 전에 $$r$$을 알아버린다면 prover는 실제로 $$f$$가 조건을 만족하지 않아도 point $$r$$에 대해서 조건을 만족하는 $$q$$를 생성해 버릴 수 있다. <br>
  ⇒ commitment를 생성한 이후에 $$r$$을 생성하는것이 매우 중요함

## **Plonk step 1**

**Step 1**: compile circuit to a computation trace (gate fan-in = 2)

![Untitled](https://i.imgur.com/bxHEF4D.png)

- we’ll want to do is prove that the computation trace is valid and that output actually is zero because we’d like to prove that $$C(x,w)=0$$

## **Computation Trace as Polynomial – Intro**

$$\|C\| :=$$ total number of gates in $$C$$

$$\|I\| := \|I_x\| + \|I_w\| =$$ number inputs to $$C$$
<br><font size="3">I_x: statement input, I_w: witness input</font><br>

let $$d := 3\|C\|+\|I\|$$ (in example, $$d=12$$) and $$H := (1, \omega, \omega^2,...,\omega^{d-1})$$

**The plan:** prover interpolates a polynomial $$P \in \mathbb{F}_p^{(\leq d)}[X]$$ that encodes the entire trace

## **Computation Trace as Polynomial – Constraints**

**The plan:** prover interpolates $$P \in \mathbb{F}_p^{(\leq d)}[X]$$ such that

1. $$P$$ encodes all inputs: P($$\omega^{-j}$$) = input number $$j$$ for $$j = 1,..., \|I\|$$
2. $$P$$ encodes all wires: $$\forall l=0,...,\|C\|-1$$
   - P($$\omega^{3l}$$): left input to gate #$$l$$
   - P($$\omega^{3l+1}$$): right input to gate #$$l$$
   - P($$\omega^{3l+2}$$): output to gate #$$l$$

In out example, Prover interpolates $$P(X)$$ such that:

Inputs: $$P(\omega^{-1})=5$$, $$P(\omega^{-2})=6$$, $$P(\omega^{-3})=1$$

gate 0: $$P(\omega^{0})=5$$, $$P(\omega^{1})=6$$, $$P(\omega^{2})=11$$

gate 1: $$P(\omega^{3})=6$$, $$P(\omega^{4})=1$$, $$P(\omega^{5})=7$$

gate 2: $$P(\omega^{6})=11$$, $$P(\omega^{7})=7$$, $$P(\omega^{8})=77$$

degree($$P$$) = 11

Prover uses FFT to compute the coefficients of $$P$$ in time $$dlogd$$

## **Plonk Step 2**

**Step 2:** Prove P is valid computation trace

- Prover P($$S_p,x,w$$)
- Verifier V($$S_v,x)$$

- Prover는 $$P(X) \in \mathbb{F}_p^{(\leq d)}[X]$$ 를 만들고 $$P$$의 commitment를 Verifier에게 보낸다
  - Prover는 $$P$$가 올바른 computation trace임을 증명해야 한다.
    1. $$P$$ encodes the correct inputs
    2. every gate is evaluated correctly
    3. the wiring is implemented correctly
    4. the output of last gate is 0

## **Prove Trace**

Both prover and verifier interpolate a polynomial $$v(X) \in \mathbb{F}_p^{(\leq\|l_x\|)}[X]$$ that encodes the $$x$$-inputs to the circuit:

for $$j=1,...,\|l_x\|$$: $$v(\omega^{-j})$$ = input #$$j$$

In out example: $$v(\omega^{-1})=5$$, $$v(\omega^{-2})=6$$, $$v(\omega^{-3})=1$$. ($$v$$ is quadratic)

constructing $$v(X)$$ takes time proportional to the size of input $$x$$

⇒ verifier has time do this

Let $$H_{inp} := \{\omega^{-1}, \omega^{-2},...,\omega^{-\|l_x\|}\} \subseteq H$$ (points encoding the input)

Prover proves (1) by using zero-test on $$H_{inp}$$to prove that

⇒ $$P(y)-v(y) = 0$$. $$\forall y \in H_{inp}$$

**Idea**: encode gate types using a **_selector_** polynimial $$S(X)$$

define $$S(X) \in \mathbb{F}_p^{(\leq d)}[X]$$ such that $$\forall l=0,...,\|C\|-1$$:

$$S(\omega^{3l})=1$$ if gate #$$l$$ is an addition gate

$$S(\omega^{3l})=0$$ if gate #$$l$$ is an multiplication gate

Observe that, $$\forall y \in H_{gates}:= \{1, \omega^3, \omega^6,...,\omega^{3(\|C\|-1)}\}$$:

$$S(y)*[P(y) + P(\omega y)] + (1-S(y))*P(y)*P(\omega y) = P(\omega^{2}y)$$

$$P(y)$$ = left input

$$P(\omega y)$$ = right input

$$P(\omega^2y)$$ = output

$$Setup(C) \rightarrow S_p:=S$$ and $$S_v:=(\fbox{S})$$

Prover uses zero-test on the set $$H_{gates}$$ to prove that $$\forall y \in H_{gates}$$

$$S(y)*[P(y) + P(\omega y)] + (1-S(y))*P(y)*P(\omega y) - P(\omega^{2}y)=0$$

encode the wires of $$C$$:

![Untitled](https://i.imgur.com/ZmlF4iS.png)

Define a polynomial $$W : H \rightarrow H$$ that implements a rotation

$$W(\omega^{-2},\omega^1,\omega^3) = W(\omega^1, \omega^3, \omega^{-2}), ..$$

**Lemma** : $$\forall y \in H: P(y) = P(W(y))$$ ⇒ wire constraints are satisfied

**Problem** : the constraint $$P(y)=P(W(y))$$ has degree $$d^2$$

⇒ prover would need to manipulate polynomials of degree $$d^2$$

⇒ quadratic time prover (goal: linear time prover)

**Lemma** : $$P(y) = P(W(y))$$ for all $$y \in H$$ if and only if $$L(Y,Z) \equiv 1,$$

where $$L(Y,Z) := \Pi_{x \in H}(P(x)+Y*W(x)+Z)/P(x)+Y*x+Z$$

To prove that $$L(Y,Z) \equiv 1$$ do:

1. verifier chooses random $$y,z \in \mathbb{F}_p$$
2. prover builds $$:L_1(X)$$ such that $$L_1(X)=(P(x)+Y*W(x)+Z)/P(x)+Y*x+Z$$ for all $$x \in H$$
3. run prod check to prove $$\Pi_{x \in H}L_1(x) = 1$$
4. validate $$L_1$$: run zero test on $$H$$ to prove $$L_2(x)=0$$ for all $$x \in H$$,

   where $$L_2(x)=(P(x)+y*x+z)L_1(x) - P(x)+y*W(x)+z)$$

## The Final Plonk poly-IOP

![Untitled](https://i.imgur.com/PbvRODX.png)

---
