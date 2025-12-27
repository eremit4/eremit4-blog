---
title: "La Micelia e El Mago: Tracing a Spanish-Speaking Credential Theft Operation"
date: 2025-12-04
draft: false
featuredImage: "toad-cover.png"
region: ["LATAM", "EU"]
tags:
  [
    "phishing",
    "outlook",
    "c2",
    "command-control",
    "telegram",
    "discord",
    "credential-theft",
    "javascript",
  ]
categories: ["Adversary Tracking"]
---

## Introduction

In this scroll, I record the beginning of a hunt that took shape in August 2025, when early signs of a Spanish-speaking phishing kit surfaced from obscurity. What followed revealed clear targeting patterns, operational fingerprints, and subtle indicators of AI-assisted development, along with the use of Telegram and Discord as command and control channels — a trend increasingly common in the wild.

## Where the Trail Begins

I often invest part of my free time observing threats that rise above the background noise, and during one of these sessions I encountered yet another kit targeting Microsoft Outlook users — this one clearly shaped for Hispanic victims, complete with a replica of the Outlook login panel.

{{< figure
    src="img/hispanic-toad-0.webp"
    alt="hispanic-toad-0"
    figclass="text-center"
    class="mx-auto"
>}}

While inspecting the kit statically, a particular signature caught my attention: four toad emojis inserted inside the string “0UTL”, likely a stylized abbreviation of Outlook. This small but intentional marker appeared within the DOM and served as a signature left by the operators.

{{< figure
    src="img/hispanic-toad-1.webp"
    alt="hispanic-toad-0"
    figclass="text-center"
    class="mx-auto"
>}}

Signatures like this are extremely valuable for pivoting. They allow the creation of reliable indicators to search for other deployments of the same kit and to map its presence across the wild. Before expanding the investigation, I structured the hunt around a few guiding questions:

- Is this an isolated deployment or evidence of a coordinated campaign operated by the same actor?
- What data does the kit capture, and what mechanisms does it use to exfiltrate it?
- What infrastructure receives the stolen data, and can it be linked to other activity by the operator?



## Tracking the Toad

To answer the first question, I needed reliable signals to track the toads in the wild. I began by creating a saved search in URLScan to monitor submissions where the string “🍄🍄0UTL🍄🍄” appeared within page content. The results were immediately revealing.

{{< figure
    src="img/hispanic-toad-2.webp"
    alt="hispanic-toad-0"
    figclass="text-center"
    class="mx-auto"
>}}

Based on the data available, the earliest observable activity dates back to March 2025. This does not necessarily indicate the true start of the operation — only the limits of current visibility. Nevertheless, URLScan alone returned 75 distinct results containing this signature, which is sufficient to partially answer the first guiding question:

> **1) Is this an isolated deployment or evidence of a coordinated campaign operated by the same actor?**
>> Answer: The evidence strongly suggests a coordinated campaign targeting Microsoft Outlook users. However, at this stage, there is not enough information to confidently attribute all deployments to a single threat actor.

With that initial assessment in place, the next logical step was to analyze each identified deployment in greater depth, focusing on code structure, exfiltration mechanisms, and infrastructure reuse. This deeper inspection revealed several notable patterns.

First, despite sharing the same toad-based signature, the campaign consists of two distinct phishing kit variants:

- One variant exfiltrates stolen credentials via Telegram, using a script named **tlgram.js**.
- The other variant relies on Discord webhooks for exfiltration, implemented through a script named **disBLOCK.js**.

Second, the overall structure and coding style across both variants strongly suggest AI-assisted development. The logic is verbose, modular, and unusually consistent for low-level phishing kits, with repetitive patterns that resemble machine-generated scaffolding rather than handcrafted code.

  {{< figure
      src="img/hispanic-toad-3.webp"
      alt="hispanic-toad-3"
      figclass="text-center"
      class="mx-auto"
  >}}
  {{< figure
      src="img/hispanic-toad-4.webp"
      alt="hispanic-toad-4"
      figclass="text-center"
      class="mx-auto"
  >}}

Finally, both variants implement the same victim profiling techniques. Each kit collects the victim’s public IP address and geolocation data by querying **ipify** and **ipinfo**, enriching the stolen credentials with contextual metadata before exfiltration.

This convergence of signatures, tooling choices, and data collection methods reinforces the hypothesis of a shared operational lineage — even if definitive attribution remains out of reach for now.

So after that, I improved the query with these two exfiltration scripts and 
