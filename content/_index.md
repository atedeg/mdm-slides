+++
title = "MDM 🧀"
description = "Mambelli Domain Model"
outputs = ["Reveal"]

[reveal_hugo]
  transition = "slide"
  transition_speed = "fast"
  custom_theme = "custom-theme.scss"
  custom_theme_compile = "true"
  slide_number = 'c/t'

[reveal_hugo.custom_theme_options]
  targetPath = "css/custom-theme.css"
  enableSourceMap = true
+++

<img class="mdm-logo" src="img/logo.svg" alt="MDM"/>

<p class="mdm-title">Mambelli Domain Model</p>

<p class="avatar-container">
  <img class="avatar" src="https://avatars.githubusercontent.com/u/20598369?v=4" alt="giacomo-avatar">
  <img class="avatar" src="https://avatars.githubusercontent.com/u/4855008?v=4" alt="nicolo-avatar">
  <img class="avatar" src="https://avatars.githubusercontent.com/u/11615611?v=4" alt="nicolas-avatar">
  <img class="avatar" src="https://avatars.githubusercontent.com/u/62563336?v=4" alt="linda-avatar">
</p>

---

# Introduction

We decided to model the domain of the [_Mambelli_](https://mambelli.com) cheese factory:  
a small family business that produces and sells cheese.

We chose this domain because:

- It is quite complex
- There are domain experts that could help us
- We ❤️ cheese

---

{{% section %}}

# Domain Modeling

<img class="stretch" src="img/event-storming-session.gif" alt="event storming session">

{{% note %}}
Descrivi qui i sottodomini e cosa fanno
{{% /note %}}

---

## Event Storming

<img class="stretch no-border" src="img/event-storming.svg" alt="event storming diagram">

{{% note %}}
La stima del latte (milk planning) viene al momento fatta a mano basandosi su dati storici e intuito;
è big bet perché porterebbe grandi vantaggi con una predizione accurata del latte da ordinare
{{% /note %}}

---

## Core Domain Chart

<img class="stretch no-border" src="img/core-domain-chart.svg" alt="core domain chart">

---

## Context Map

<img class="stretch no-border" src="img/context-map-light.svg" alt="context map">

{{% /section %}}

---

{{% section %}}

# Architecture

The architecture of each bounded context follows the Clean Architecture's structure

<img class="stretch no-border" src="img/clean-architecture.svg" alt="clean architecture">

{{% /section %}}

---

{{% section %}}

# DevOps

---

## DVCS Workflow

We adopt a `git-flow`-like workflow with a `beta` branch for pre-releases and a `main` branch for the stable ones

{{< mermaid >}}
%%{
  init: {
    'theme':'base',
    'themeVariables': {
      'git0': '#ff3c3e',
      'git1': '#f9b59c',
      'git2': '#8de0d8',
      'git3': '#fca55f',
      'git4': '#55303e',
      'gitBranchLabel0': '#000000',
      'gitBranchLabel1': '#000000',
      'gitBranchLabel2': '#000000',
      'gitBranchLabel3': '#000000',
      'gitBranchLabel4': '#ffffff'
    }
}
}%%
gitGraph
  commit id: "chore: ..."
  commit id: "build: ..."
  commit id: "chore: ..."
  commit id: "ci: ..."
  branch beta
  commit tag: "1.0.0-beta.1"
  branch feat/feature-1
  checkout feat/feature-1
  commit id: "feat: feature 1"
  commit id: "feat: feature 2"
  checkout beta
  merge feat/feature-1 tag: "1.0.0-beta.2"
  branch feat/feature-2
  commit id: "feat: feature 3"
  checkout beta
  merge feat/feature-2 tag: "1.0.0-beta.3"
  checkout main
  merge beta tag: "1.0.0"
  branch fix/fix-2
  commit id: "fix: fix1"
  commit id: "fix: fix2"
  checkout main
  merge fix/fix-2 tag: "1.0.1"
{{< /mermaid >}}

---

## Conventional Commit

To enforce conventional `conventional commit` we developed
a [_gradle plugin_](https://github.com/nicolasfara/conventional-commits) and an
[_sbt plugin_](https://github.com/nicolasfara/sbt-conventional-commits).
The plugin is handy since:

- It creates a git hook as soon as the project is imported  
  (one can not forget to set it up!)
- It can also be configured through ad-hoc plug-in keys

{{% note %}}

Perché usare `conventional commit`?

- Determinare automaticamente il version bump
- Avviare build e publishing del progetto in automatico
- Generare automaticamenet CHANGELOGs
- Storia dei commit più facile da capire

- Husky e commitlint non ci piacevano perché uno se ne può dimenticare
  dato che richiedono intervento manuale

{{% /note %}}

---

## Semantic Release

[`semantic-release`](https://semantic-release.gitbook.io/semantic-release/)
automates the whole package release workflow:

- Determines the next version number
- Generates the release notes
- Publishes the package

The use of __Conventional Commits__ combined with __Semantic Release__ helped us to automate the release process

---

## Quality Assurance

Code quality is an important aspect, so the following tools have been used:

- [Wartremover](https://www.wartremover.org/)
- [Scalafix](https://scalacenter.github.io/scalafix/)
- [Scalafmt](https://scalameta.org/scalafmt/)

Each of these tools is used in CI to prevent the merging of bad code

---

## Continuous Integration and Deployment

The following workflow is the result of a pipeline optimization that seeks to take full advantage of the parallelism between jobs:

{{< mermaid >}}
%%{init: {'theme':'base'}}%%
flowchart LR
  SFMT(Scalafmt) --> B
  SFX(Scalafix) --> B
  WRM(Wartremover) --> B
  T(Unit test) --> Cov(Coverage Report)
  Cov --> B
  A(Documentation site)-->B(Publish)
  B --> C(Publish site)
{{< /mermaid >}}

To simplify the `Publish` job, the [`scala-release`](https://github.com/atedeg/scala-release) action was developed

---

## Project Management

- We first defined a product backlog and biweekly sprint backlogs
- All backlog items are tracked by linked GitHub issues
- Closing PRs and issues automatically advances the project status

{{% /section %}}

---

{{% section%}}

# Development Choices

---

## Domain Modeling Approach

All core domain concepts are modelled using simple ADTs to keep
them as simple and adherent to the expert's definition as possible:

```scala{}
enum Batch:
  case Aging(id: BatchID, cheeseType: CheeseType, readyFrom: LocalDateTime)
  case ReadyForQualityAssurance(id: BatchID, cheeseType: CheeseType)
```

instead of the possibly confusing:

```scala{}
final case class Batch(
  id: BatchID,
  cheeseType: CheeseType,
  isAging: Bool,
  readyFrom: Option[LocalDateTime], // only defined if isAging 
)
```

{{% note %}}

- La prima esprime chiaramente i possibili stati in cui si può trovare un lotto
- Non ci possono essere stati incoerenti
- La definizione è più semplice e lineare da leggere anche per un esperto di dominio

{{% /note %}}

---

All primitive types are not only wrapped in appropriate value objects but also enriched
with compile-time-checked predicates:

```scala{}
type PositiveNumber = Int Refined Positive
final case class InStockQuantity(n: PositiveNumber)
```

instead of:

```scala{}
final case class InStockQuantity private(n: Int)
object InStockQuantity:
  def apply(n: Int): Option[InStockQuantity] =
    if n < 0 then None else Some(InStockQuantity(n))
```

{{% note %}}

- Modellazione più chiara, facile da leggere anche per un esperto di dominio
- Esprime chiaramente delle invarianti importanti a livello di type signature
- Non c'è bisogno di andare a leggere il codice del costruttore
- Meno test da scrivere perché non c'è un costruttore la cui implementazione potrebbe
  erroneamente cambiare nel tempo
- Più errori catturati  compile-time

{{% /note %}}

---

## Side-effect Encoding via Monads

All core domain actions use a monadic encoding of side-effects: all side effects are reified
(using an [`mtl`](https://typelevel.org/cats-mtl/getting-started.html)-style encoding)
and expressed in the type signature of the function

```scala{|1|1,5|1,7|}
def labelProduct[M[_]: CanRaise[WeightNotInRange]: CanEmit[ProductStocked]: Monad]
  (...): M[LabelledProduct] =
for
  ...
  product <- optionalProduct.ifMissingRaise(WeightNotInRange(...): WeightNotInRange)
  labelledProduct = LabelledProduct(product, AvailableQuantity(1), batch.id)
  _ <- emit(ProductStocked(labelledProduct): ProductStocked)
yield labelledProduct
```

{{% note %}}

- Modellazione più chiara, isola ed esprime chiaramente i side effect
- DSL monadico per scrivere codice che sia più facilmente comprensibile
- Reificando i side effect diventa molto semplice testare le diverse azioni isolandole fra loro

{{% /note %}}

---

## DTOs

DTOs play a fundamental role in transferring data to and from a bounded context.
All the repetitive code needed to generate a `DTO` instance is automatically generated
taking advantage of Scala 3's metaprogramming and inlining

```scala{}
final case class Client(code: ClientCode, name: String, vatNumber: VATNumber)
final case class ClientDTO(code: String, name: String, vatNumber: String)

// Automatic derivation with compile-time checks!
given DTO[Client, ClientDTO] = productTypeDTO
```

---

## HTTP API

We used the [tapir](https://tapir.softwaremill.com/en/latest/) library to define the endpoints.
It proved to be useful in many ways:

- The endpoints are defined declaratively
- All the endpoints definitions are type-checked
- It automatically generates the OpenAPI specification and serves it through Swagger

{{% note %}}

Demo con il serverino swagger

{{% /note %}}

---

## Documentation

- All ubiquitous language concepts are mirrored by a corresponding ADT in the code
- Keeping the code and the ubiquitous language up-to-date can be a difficult task
- The code should be the only source of truth (the code _is_ the ubiquitous language)

We needed a way to automatically generate documentation pages containing the
ubiquitous language definitions coming from the scaladoc

---

## Ubidoc

A configurable plugin we developed to create markdown tables from ubiquitous language
definitions from the scaladoc

```scala
/**
 * It defines, for each product, the [[StockedQuantity quantity available in stock]].
 */
final case class Stock(...)
/**
 * A quantity of a stocked product, it may also be zero.
 */
final case class StockedQuantity(...)
```

is turned into this table:

|Term | Definition |
|-----|-------------|
| Stock | It defines, for each product, the [quantity available in stock](https://atedeg.dev/mdm/dev/atedeg/mdm/restocking/StockedQuantity.html#). |
| Stocked Quantity | A quantity of a stocked product, it may also be zero. |

{{% /section %}}

---

# License

The open licence that best suited our needs was the
[Apache Licence 2.0](https://www.apache.org/licenses/LICENSE-2.0):  
our codebase could contain refences to the Mambelli trademark
and this licence does not grant permission to use it  
_"...except as required for reasonable and customary use_  
_in describing the origin of the Work..."_
