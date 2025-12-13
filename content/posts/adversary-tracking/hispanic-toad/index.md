---
title: "Unmasking ToadKit: Exposing a Hispanic Credential Theft Operation"
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

In this scroll, I record the beginning of a hunt that took shape in August 2025, when early signs of a Hispanic-focused phishing kit surfaced from obscurity. What followed revealed clear targeting patterns, operational fingerprints, and subtle indicators of AI-assisted development, along with the use of Telegram and Discord as command-and-control channels — a trend increasingly common in the wild.

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

The answer the first question, we must use some signals to track the toads in the wild. So I created a Saved Search in URLScan to monitor the submission of URLs with the string "🍄🍄0UTL🍄🍄" being part of the their content, and the result was very interesting.

{{< figure
    src="img/hispanic-toad-2.webp"
    alt="hispanic-toad-0"
    figclass="text-center"
    class="mx-auto"
>}}

We wont attribute that the operator starts in March 2025 due to our limited visibility, but it is the first evidence that URLScan gave me and returns 75 results since them. This first query is sufficient to answer partially the first question:

> **1) Is this an isolated deployment or evidence of a coordinated campaign operated by the same actor?**
>> A: This is coordinated campaign, targeting Outlook users, but we don't have sufficient evidences that indicates that we are talking about the same threat actor (TA).

As a natural next step, I checked all the results of this campaign to evaluate their code, infrastructure, and other evidences that could create others relantioship and answer tother qustions as well. Inspecting the code, some points catch my mind: 
- We have two different kits with the same toad signature: one use telegram through **tlgram.js** script to exfiltration, and another use discord through **disBLOCK.js** as exfiltration infraestructure;
- The structure of the code seems that was developed with AI assistance;

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

- both versions use ipify and ipinfo to collect IP address and geolocation data from the victims

{{< figure
    src="img/hispanic-toad-3.webp"
    alt="hispanic-toad-3"
    figclass="text-center"
    class="mx-auto"
>}}