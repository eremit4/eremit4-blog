---
title: "What Comes After Cards: A Brazilian Magecart Operation Moves into Instant Payments (PIX)"
date: 2026-04-21
draft: false
featuredImage: "br-magecart-pix-cover.jpg"
images:
 - "br-magecart-pix-cover.jpg"
region: ["Brazil"]
tags:
  [
    "digital-skimming",
    "magecart",
    "fraud",
    "PHP",
    "discord",
    "PIX"
  ]
categories: ["Adversary Tracking"]
---

## Introduction
This research examines how a Brazilian Magecart operation is extending beyond traditional card skimming to also target PIX, Brazil’s instant payment system, within e-commerce environments. While Magecart activity has historically focused on credit card data exfiltration, this shift suggests a move toward real-time monetization by manipulating payment flows, aligning with the growing adoption of PIX in online purchases. This remains an uncommon TTP for Magecart, but one that may become more prevalent as the Brazilian payment landscape evolves. As in previous research, the focus here is on techniques, patterns, and infrastructure rather than exposing affected merchants or victims. **All infrastructure analyzed in this research has already been publicly flagged as malicious in widely accessible platforms such as VirusTotal.**

## The First Reach of a Known Threat
In mid-October 2025, I identified activity consistent with a known Magecart pattern, where Brazilian e-commerce platforms, primarily running PHP frameworks such as Magento, were compromised to enable credit card skimming during checkout. Once the associated domains were flagged as malicious, parts of the infrastructure went offline and affected merchants remediated the intrusion. For a time, the activity appeared to fade, with no further signals linking back to the campaign.

{{< figure
src="img/domain-flagged.webp"
alt="VirusTotal detection report"
caption="Malicious domain associated with the campaign. The infrastructure was identified and flagged as malicious by multiple security vendors on VirusTotal."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

That assumption broke in December 2025, when one of my detection rules triggered on a new merchant interacting with previously identified infrastructure. At first glance, it looked like a simple continuation of the same operation. However, a closer analysis revealed a subtle but meaningful shift in behavior.

{{< figure
src="img/urlscan-saved-search.webp"
alt="URLScan saved search detections"
caption="Detections from a saved search rule used to identify new victims of the campaign. The spikes in the graph represent periods when the malicious script was found active on different e-commerce platforms."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

The actors were no longer limited to harvesting credit card data. Instead, they had introduced a new capability: manipulating the payment flow itself by replacing the QR Code presented at checkout when PIX was selected.
At that point, this was no longer just a classic Magecart operation.

### PIX and the Evolution of Magecart Monetization
To understand the impact of this shift, it is necessary to briefly contextualize PIX within the Brazilian payment ecosystem. PIX is an instant payment system introduced by the Central Bank of Brazil, widely adopted across both individuals and businesses, where transactions are processed in real time. In PIX transactions, payments are typically completed by scanning a QR Code or using a copy-and-paste payment string.

{{< figure
src="img/pix-qrcode.webp"
alt="Example of PIX QR Code checkout payment flow"
caption="Example of a PIX QR Code payment flow during checkout. Source: Modern Treasury (https://www.moderntreasury.com/learn/pix-payments)"
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

Observed activity indicates the use of PIX hijacking, a technique in which the malicious script monitors user interactions during checkout and replaces the legitimate QR code and PIX copy-and-paste string with attacker-controlled values. This manipulation redirects the payment flow to an account under the attacker’s control, typically without raising suspicion, as the victim proceeds to scan the QR code or paste the PIX copy-and-paste string within their banking app on a smartphone, believing it is part of the normal transaction.

Instead of relying on stolen card data that must be monetized later, the attacker can now receive funds immediately. The victim completes the payment believing it is legitimate, and the transfer is executed in real time to an attacker-controlled account, with limited chances of recovery.

{{< figure
src="img/pix-hijacking-flow.webp"
alt="Flow diagram of PIX hijacking showing real-time QR code replacement and payment redirection during checkout."
caption="Flow diagram illustrating PIX hijacking, where a malicious script replaces the original QR Code and copy-and-paste payment string in real time, redirecting funds to an attacker-controlled account."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

For the merchant, the impact is characterized by a visibility gap. Since the order is successfully created in the system but the funds are diverted, the transaction appears as a 'pending' or 'abandoned' payment. This discrepancy often persists until customer complaints trigger a manual investigation, providing the actor with a significant window of opportunity.

This misalignment can persist for days until the issue is identified and properly investigated. During this window, the attacker continues to operate and extract value. In high-traffic periods in Brazilian retail, such as Mother’s Day, Easter, or other seasonal events, even a short delay can translate into significant financial losses.

For a deeper look into Magecart operations and how web skimming has evolved across Latin America, I previously covered this in [When the Checkout Bleeds: An Analysis of Skimming Campaigns in LATAM](https://sagehollow.world/posts/adversary-tracking/when-the-checkout-bleeds/
). This context is important because PIX changes the economics of these operations.

## Following the Tentacles
The campaign operates through an organized and modular infrastructure, built on a cluster of interconnected command and control domains. A clear division of labor is present across the codebase, with distinct modules handling credit card exfiltration and a dedicated component responsible for real-time PIX hijacking.

A key characteristic is the reuse of infrastructure across domains. Scripts hosted on one malicious node frequently exfiltrate credit card data to endpoints hosted on another campaign domain, establishing cross-domain communication between C2 nodes and forming a triangulated exfiltration pattern rather than isolated one-to-one flows. In contrast, the PIX hijacking module appears more stable, remaining tied to the same C2 domain and not following this cross-domain exfiltration behavior.

To better understand this behavior, an interactive graph was generated to map relationships between command and control domains and anonymized merchants. The visualization also highlights infrastructure rotation over time, likely in response to domains being flagged or taken down.

<div class="sage-graph"> <iframe src="https://eremit4.github.io/eremit4-graphs/brazilian-magecart-and-pix-hijacking/bmaph-graph.html" loading="lazy" style="width:100%; height:650px; border:none;"> </iframe> </div>

Not all nodes or domains listed are directly connected at the infrastructure level, due to the lack of observable infrastructure reuse in exfiltration paths and limited historical and temporal visibility across merchants in open-source data, preventing direct linkage to other domains. However, they share consistent code structure, obfuscation and exfiltration techniques, as well as file naming patterns, providing strong confidence that they are part of the same campaign.

### Exfiltration Logic and Infrastructure Rotation
The majority of variants rely on standard POST requests to attacker-controlled endpoints, which serves as the stable baseline for their data collection. However, we observed possible experimentations involving the use of Discord webhooks for data exfiltration. This tactic allows the actor to route stolen information through a high-reputation API, effectively bypassing basic domain blacklisting and blending malicious traffic with trusted HTTPS communication.

Crucially, the use of Discord Webhooks provides a "write-only" exfiltration channel. Since Discord webhooks do not grant read permissions to the endpoint itself, the actors eliminate the risk of researchers or law enforcement accessing the webhook to dump previously stolen data. This creates a one-way bridge that protects the attacker’s harvested data even if the specific script or webhook URL is discovered.

{{< figure
src="img/discord-exfil.webp"
alt="Evidence of a malicious script exfiltrating credit card data via a Discord webhook."
caption="Evidence of a malicious script exfiltrating credit card data via a Discord webhook."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

In the case of PIX hijacking, the scripts act as an active proxy rather than a passive collector. The skimmer extracts the transaction value from DOM elements within the e-commerce page and converts the total amount into cents. This standardized value is sent to the command-and-control server, which responds with a JSON payload containing both a custom-generated QR code and a corresponding copy-and-paste PIX payload.

{{< figure
src="img/pix-hijacking.webp"
alt="Example of attacker-controlled API response returning a base64-encoded QR Code and a corresponding PIX “copy and paste” key used to replace the original payment."
caption="Example of attacker-controlled API response returning a base64-encoded QR Code and a corresponding PIX “copy and paste” key used to replace the original payment."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}


### Anti-Analysis and Lightweight Obfuscation
The campaign employs a “just-enough” obfuscation strategy, primarily relying on string array indirection, hex-style indexing, and Base64-decoded resources to conceal key elements such as exfiltration endpoints and operational logic. Across the cluster, samples prioritize execution reliability through techniques such as extensive DOM polling, multiple selector fallbacks, and monkey patching of native checkout functions to ensure consistent data capture across different implementations. To hinder analysis, several scripts implement anti-analysis measures including console interference, overriding functions like `console.log`, `console.warn`, and related methods to reduce visibility into runtime behavior during inspection.

{{< figure
src="img/anti-analysis-console.webp"
alt="Evidence of anti-analysis implementation via console method hooking, limiting visibility during runtime inspection."
caption="Evidence of anti-analysis implementation via console method hooking, limiting visibility during runtime inspection."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

## Campaign Overview: Diamond Model

{{< figure
src="img/diamond-model.webp"
alt="Diamond Model summarizing the adversary, infrastructure, capabilities, and victim relationships observed in the digital skimming campaign."
caption="Diamond Model summarizing the adversary, infrastructure, capabilities, and victim relationships observed in the digital skimming campaign."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

## Conclusion
This activity reflects a shift in monetization, not in initial compromise. The actors retain traditional Magecart techniques for access and data collection, but introduce a mechanism to redirect PIX payments at the moment of checkout.

**There is no vulnerability in PIX itself.** The attacker does not interfere with the payment system, but with the client-side flow that presents the payment details to the user. By replacing the QR Code or “copy and paste” key, the payment is silently redirected to an attacker-controlled account.

The effectiveness of this approach relies on user trust in the checkout interface, not on weaknesses in the underlying payment infrastructure.
