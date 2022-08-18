+++
title = "MDM 🧀"
description = "Mambelli Domain Model"
outputs = ["Reveal"]

[reveal_hugo]
  transition = "slide"
  transition_speed = "fast"
  custom_theme = "custom-theme.scss"
  custom_theme_compile = "true"

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
- There are domain expert that could help us
- We ❤️ cheese

---

{{% section %}}

# Domain Modeling

<img class="stretch" src="img/event-storming-session.gif" alt="event storming session">

---

## Event Storming

<img class="stretch no-border" src="img/event-storming.svg" alt="event storming diagram">

---

## Core Domain Chart

<img class="stretch no-border" src="img/core-domain-chart.png" alt="core domain chart">

---

## Bounded Contexts

TODO: add context map

---

## User Stories

TODO: add user stories

{{% /section %}}

---

{{% section %}}

# Architecture

---

## Clean Architecture

<img class="stretch no-border" src="img/clean-architecture.svg" alt="clean architecture">

{{% /section %}}

---
{{% section %}}

# DevOps

---

## DVCS Workflow

We adopt a `git-flow`-like workflow with a `beta` branch for pre-releases and a `main` branch for the stable ones.

{{< mermaid >}}
%%{init: {'theme':'base'}}%%
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

Why use `conventional commit`?

- Automatically determining a semantic version bump
- Triggering build and publish processes (with external tools)
- Automatically generating CHANGELOGs
- More understandable commit history

How to enforce the use of `conventional commit`?

A [_gradle plugin_](https://github.com/nicolasfara/conventional-commits)
and an [_sbt plugin_](https://github.com/nicolasfara/sbt-conventional-commits)
have been developed.

---

## Semantic Release

`semantic-release` automates the whole package release workflow:

- Determines the next version number
- Generates the release notes
- Publish the package

The use of __Conventional Commits__ combined with __Semantic Release__ helped us to automate the release process.

---

## Quality Assurance

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

To simplify the `Publish` job, the [`scala-release`](https://github.com/atedeg/scala-release) action was developed.

{{% /section %}}

---

{{% section %}}

# Development

---

## Domain Modeling Approach

---

## Side-effect Encoding via Monads

---

## DTOs

---

## HTTP API

---

## Documentation

{{% /section %}}

---

# License
