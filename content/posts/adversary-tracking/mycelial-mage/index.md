---
title: "The Mycelial Mage: Tracing a Spanish-Speaking Credential Theft Operation"
date: 2025-12-04
draft: false
featuredImage: "mycelial-mage-cover.png"
region: ["LATAM", "EU"]
tags:
  [
    "phishing",
    "outlook",
    "c2",
    "command-and-control",
    "telegram",
    "discord",
    "credential-theft",
    "javascript",
    "hypergraph"
  ]
categories: ["Adversary Tracking"]
---

## Introduction
In this scroll, I record the beginning of a hunt that took shape in August 2025, when early signs of a Spanish-speaking phishing kit surfaced from obscurity. What followed revealed clear targeting patterns, operational fingerprints, and subtle indicators of AI-assisted development, along with the use of Telegram and Discord as command and control channels, a trend increasingly common in the wild.

## Where the Trail Begins
I often invest part of my free time observing threats that rise above the background noise, and during one of these sessions I encountered yet another kit targeting Microsoft Outlook users, and this one clearly shaped for Hispanic victims.

{{< figure
    src="img/mycelial-mage-panel.webp"
    alt="mycelial-mage-panel"
    figclass="text-center"
    class="mx-auto"
>}}

While inspecting the kit statically, a particular signature caught my attention: four mushroom emojis inserted inside the string “0UTL”, likely a stylized abbreviation of Outlook. This small but intentional marker appeared within the DOM and served as a signature left by the operators.

```javascript
function _0x1a1c(){const _0x236637=['🍄🍄0UTL🍄🍄\x0a\x0a','gatorio','js1','n\x20correo\x20e' ...]}
```

Signatures like this are extremely valuable for pivoting. They allow the creation of reliable indicators to search for other deployments of the same kit and to map its presence across the wild. Before expanding the investigation, I structured the hunt around a few guiding questions:

- Do the recovered artifacts point to sporadic proliferation or a consolidated operation?
- What data does the kit capture, and what mechanisms does it use to exfiltrate it?
- What infrastructure receives the stolen data?

## Tracking the Mushrooms
To address the first question, I created a saved search in URLScan to monitor submissions where the string `🍄🍄0UTL🍄🍄` appeared within rendered page content. The results were immediately revealing.

{{< figure
    src="img/mycelial-mage-counts.webp"
    alt="mycelial-mage-counts"
    figclass="text-center"
    class="mx-auto"
>}}

Based on the data available, the earliest observable activity dates back to March 2025. This does not necessarily indicate the true start of the operation, only the limits of current visibility. Nevertheless, URLScan returned dozens of distinct results containing this signature.

While this volume confirms active use, raw counts alone are insufficient to distinguish between scattered reuse and a coordinated campaign. To answer that, the investigation shifted from where the kit appeared to how it evolved.

#### The Spore — xjsx.js
In the earliest observed deployments, credential exfiltration is supported by a small auxiliary script named `xjsx.js`. This file contains almost no execution logic. Instead, it serves as a configuration container for the operator’s Telegram Bot Token and Chat ID, assigned to global variables after light obfuscation (array rotation).

At this stage, the architecture is modular. Configuration is separated from execution, suggesting an early focus on operational convenience. Tokens can be rotated without modifying the core phishing logic.

{{< figure
    src="img/mycelial-mage-xjsx.webp"
    alt="mycelial-mage-xjsx"
    figclass="text-center"
    class="mx-auto"
>}}

{{< figure
    src="img/mycelial-mage-xjsx-deobfuscated.webp"
    alt="mycelial-mage-xjsx-deobfuscated"
    figclass="text-center"
    class="mx-auto"
>}}

#### A Hardened Strain — tlgram.js
Later deployments preserve the original separation of responsibilities but significantly harden the Telegram component. The script introduces layered anti-analysis measures to conceal Telegram credentials and critical logic.

These measures include dynamic construction of `debugger` statements triggered when DevTools are open, console method hijacking to suppress runtime visibility (`log`, `warn`, `info`, `error`, etc.), and self-referential regex traps designed to trigger catastrophic backtracking during source inspection. Together, these mechanisms degrade both static analysis and interactive debugging, without altering the underlying exfiltration workflow.

Telegram credentials, input validation, victim profiling, and exfiltration routines remain functionally intact, but are intentionally concealed by anti-analysis mechanisms. References to `xjsx` persist only as naming remnants or comments within `tlgram.js`, indicating an evolutionary progression of the same codebase rather than an independent fork.

{{< figure
    src="img/mycelial-mage-tlgram.webp"
    alt="mycelial-mage-tlgram"
    figclass="text-center"
    class="mx-auto"
>}}

{{< figure
    src="img/mycelial-mage-tlgram-deobfuscated.webp"
    alt="mycelial-mage-tlgram"
    figclass="text-center"
    class="mx-auto"
>}}

#### The Wizard's Legacy: A Persistent Baseline
The observed defensive posture in previous phases does not appear to be a linear progression. This iteration of `tlgram.js`, characterized by the complete absence of obfuscation and the exposure of configuration values in plain text, was initially considered a regression. However, historical evidence suggests this specific pattern has been present in the wild for nearly a year, predating many of the more "hardened" variants.

The persistence of this unmasked code, alongside the consistent comment signature `//XJSX🧙🏻‍♂️`, supports the hypothesis of a shared development lineage that maintains multiple operational branches. Rather than a step backward, this version likely serves as a persistent baseline, a stable core of the kit that remains in circulation alongside its more complex successors. This longevity possibly indicates that the operator, or those redistributing the toolset, continues to utilize this established strain as a functional component of the broader "Mycelial Mage" infrastructure.

{{< figure
    src="img/mycelial-mage-tlgram-latest.webp"
    alt="mycelial-mage-tlgram"
    figclass="text-center"
    class="mx-auto"
>}}

#### An Exposed Strain — disBLOCK.js
The most recent evolution replaces Telegram-based exfiltration with Discord webhooks.

In `disBLOCK.js`, the underlying logic reappears largely unchanged, but the script is entirely unobfuscated. Functions are clearly named, indentation is consistent, and explanatory comments in Spanish describe each stage of execution. The structure strongly resembles AI-assisted code generation rather than organically evolved tooling.

{{< figure
    src="img/mycelial-mage-disblock.webp"
    alt="mycelial-mage-disblock"
    figclass="text-center"
    class="mx-auto"
>}}

{{< figure
    src="img/mycelial-mage-disblock-discord.webp"
    alt="mycelial-mage-disblock-discord"
    figclass="text-center"
    class="mx-auto"
>}}

#### Echoes Without Record: A Hypothesis on Ephemeral Exfiltration Channels

The apparent tactical migration from Telegram to Discord, or the hardening of existing Telegram infrastructures, may reflect a sophisticated trade-off between operational flexibility and defensive exposure. Historically, Telegram bots often functioned as “glass boxes” for defenders; an exposed token could act as a skeleton key, allowing analysts to query metadata, intercept real-time logs, or, in some cases, recover historical exfiltration due to the API’s message persistence. More recent activity, however, suggests a possible shift toward a model of forensic opacity, in which the exfiltration channel is treated not as a database, but as an ephemeral tunnel.

In current campaigns, this shift may involve the adoption of active remediation techniques intended to complicate post-compromise analysis. Even when valid tokens are recovered, defenders frequently encounter persistently empty queues. This absence is not necessarily indicative of inactivity, but may instead be consistent with high-frequency active consumption. Mechanisms such as aggressive long-polling or immediate message acknowledgment could result in exfiltrated data being consumed and cleared from the Telegram cloud almost as soon as it arrives, creating a race condition where defenders must not only possess valid credentials, but also outpace an automated backend to observe the API state.

This design appears to reach its most opaque form with the use of Discord webhooks, which, from a defensive perspective, function effectively as write-only sinks. Unlike the interactive surface of a Telegram bot, a Discord webhook is architecturally one-directional: once a payload is delivered, the historical record becomes inaccessible to anyone holding only the webhook URL. While a defender might attempt to introduce noise by interacting with the endpoint, previously exfiltrated data remains obscured. Taken together, this shift suggests a threat model that increasingly prioritizes the containment of stolen data over any need for interactive control.

{{< figure
    src="img/mycelial-mage-sniffing.webp"
    alt="mycelial-mage-sniffing"
    figclass="text-center"
    class="mx-auto"
>}}

#### Shared Morphology
Beneath changing obfuscation layers and rotating command-and-control channels, a persistent operational structure remains. Input validation logic, victim data handling, and exfiltration workflows show clear continuity across all observed variants.

This continuity extends to auxiliary enrichment steps. Across multiple samples, victim IP resolution is consistently performed via `api.ipify.org`, followed by geolocation enrichment using `ipapi.co`. The sequencing, data fields collected, and integration into the exfiltration payload remain functionally identical, despite changes in obfuscation and command-and-control backends.

## A Cursed Harvest
By intercepting the exfiltration routine, we observe the theft as it occurs: a calculated, automated extraction captured in real time through network telemetry. The process is mechanical rather than opaque, exposing how credentials are packaged, enriched, and transmitted to the operator.

Upon execution, the kit performs an initial reconnaissance step using api.ipify.org and ipapi.co to resolve the victim’s IP address and geolocation. The collected metadata is appended to the credentials prior to exfiltration, enriching the payload with contextual information.

The traffic capture below freezes the moment of transfer. Regardless of the command-and-control backend, the exfiltration payload follows the same rigid, standardized format. The example shown reflects a Telegram-based capture, but the structure remains consistent across all observed variants.
```http
🍄🍄0UTL🍄🍄CORRE0: [victim_email] PASSWR: [victim_password] 🌎IP: [IP_Address] [City, Country]
```

{{< figure
    src="img/mycelial-mage-traffic.webp"
    alt="mycelial-mage-traffic"
    figclass="text-center"
    class="mx-auto"
>}}

With the mechanism exposed, the mysteries are answered:

> **1) Do the recovered artifacts point to sporadic proliferation or a consolidated operation?**
>> **Answer:** The evidence supports a consolidated and sustained operation rather than sporadic reuse. Across multiple deployments, the kit exhibits stable implementation traits, such as identical exfiltration workflows, consistent payload structure (e.g., fixed field labels and marker strings), and an invariant enrichment sequence using api.ipify.org followed by ipapi.co. Linguistic elements in code and user-facing strings point to deliberate targeting of Spanish-speaking users. 

> **2) What data does the kit capture, and what mechanisms does it use to exfiltrate it?**
>> **Answer:** The kit acts as a parasite, capturing Email and Password while simultaneously enriching them with IP and Geolocation data seized via the reconnaissance APIs. It exfiltrates this digested data via standard HTTPS POST requests, strictly formatted with the operator's fungal emojis.

> **3) What infrastructure receives the stolen data?**
>> **Answer:** The infrastructure abuses legitimate services—Telegram Bots and Discord Webhooks—as carriers for the spores.

## Following the Cursed Roots

With the operational questions answered and the mechanics of the kit laid bare, the investigation shifted away from what the phishing kit does and toward how far its roots extend. The patterns observed across deployments, including shared code structure, consistent enrichment logic, and interchangeable exfiltration backends, align more closely with a phishing-as-a-service model than with isolated or bespoke operations. In this context, individual deployments are better understood as discrete instances of a shared service rather than independent infrastructures.

Rather than pursuing immediate attribution, the next objective became structural mapping. A URLScan saved search created during this investigation was preserved locally and normalized into a structured dataset, consolidating scan results, script variants, domains, hosting indicators, and exfiltration endpoints into a single CSV.

At this stage, the dataset serves an analytical role. Its purpose is to expose structural relationships that are difficult to observe through linear inspection of indicators. After normalization and deduplication, the remaining data represents a reduced but meaningful projection of the operation’s exposed infrastructure surface. From this corpus, a custom graph was built to reflect how phishing infrastructure is provisioned and reused in practice. The model enforces a clear hierarchy of `Autonomous System → Hosting IP → Phishing Domain → Exfiltration Endpoint`, ensuring that correlations emerge from shared artifacts rather than inflated connectivity.

<div class="sage-graph">
  <iframe
    src="https://eremit4.github.io/eremit4-graphs/mycelial-mage/mycelial-mage-infra-graph.html"
    loading="lazy"
    style="width:100%; height:650px; border:none;">
  </iframe>
</div>

When examined under this structure, the infrastructure does not resolve into a single cohesive cluster, nor does it fragment into entirely unrelated components. Distinct groupings emerge at the hosting and network layers, often segmented across autonomous systems and IP ranges, indicating deliberate separation at the deployment level. At the same time, selective convergence is observable at the exfiltration layer. Telegram bot tokens and Discord webhooks bridge infrastructure clusters that otherwise remain isolated in terms of hosting and network context.

This asymmetric pattern is characteristic of service-based phishing ecosystems. Deployment layers are intentionally disposable and compartmentalized, while backend components may be reused for efficiency or operational convenience. The reuse of exfiltration endpoints represents the strongest indicator of backend commonality observed in this dataset, but it remains insufficient for attribution. Shared channels demonstrate shared service components or workflows, not shared operators, centralized control, or unified intent.

Equally important, some clusters remain isolated even at the exfiltration level. These boundaries suggest either distinct tenants within the same service or parallel implementations with no shared backend. Taken together, the graph supports structural correlation without asserting attribution. It highlights patterns of reuse and separation while remaining agnostic to actor identity, reinforcing earlier conclusions about a distributed ecosystem rather than a singular campaign.

## Structural Conclusions

This scroll was not written to name an actor, but to understand a system.

The investigation focused on exposing how a Spanish-speaking phishing kit operates in practice: how credentials are captured, how exfiltration channels evolve, and how infrastructure is provisioned, segmented, and selectively reused. Across multiple deployments, the evidence points to a service-oriented ecosystem rather than isolated or bespoke campaigns.

Disposable hosting layers coexist with reused exfiltration endpoints, a pattern consistent with phishing-as-a-service models. Structural overlap is observable without requiring assumptions about centralized control or shared operators. TThe data supports correlation at the tooling and service level, not attribution of specific actors.

This boundary is intentional. While future work may pivot on strings, linguistic markers, or underground reuse to explore actor identity, this scroll remains confined to mechanics and infrastructure. Understanding how the operation functions is a prerequisite for any attempt to understand who stands behind it.

The roots mapped here do not yet resolve into a single mycelium. They form a network. What grows from it is a question for the next hunt.

## Infrastructure Artifacts

These indicators of compromise (IOCs) were collected during this study and served as the basis for the hypergraph presented in the *Following the Cursed Roots* section.

- [Full dataset](https://raw.githubusercontent.com/eremit4/eremit4-graphs/refs/heads/main/mycelial-mage/mycelial-mage-dataset.csv)
- **Format:** CSV

