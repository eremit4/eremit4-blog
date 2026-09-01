---
title: "What Comes After Cards: A Brazilian Magecart Operation Moves into Instant Payments (PIX)"
date: 2026-09-01
draft: false
featuredImage: "br-magecart-pix-cover.png"
images:
 - "br-magecart-pix-cover.jpg"
region: ["Brazil"]
tags:
  [
    "digital-skimming",
    "magecart",
    "AI-assisted",
    "PHP",
    "javascript",
    "magento",
    "discord",
    "PIX",
    "QR-code"
  ]
categories: ["Adversary Tracking"]
---

## Introduction
This research examines how a Brazilian Magecart operation is extending beyond traditional card skimming to also target PIX, Brazil’s instant payment system, within e-commerce environments. While Magecart activity has historically focused on credit card data exfiltration, this shift suggests a move toward real-time monetization by manipulating payment flows, aligning with the growing adoption of PIX in online purchases. This remains an uncommon TTP for Digital Skimming groups, but one that may become more prevalent as the Brazilian payment landscape evolves. As in previous research, the focus here is on techniques, patterns, and infrastructure rather than exposing affected merchants or victims. All infrastructure analyzed in this research has already been publicly flagged as malicious in widely accessible platforms such as VirusTotal.

## The First Reach
Activity consistent with a skimming pattern was identified affecting Brazilian e-commerce platforms, predominantly running PHP frameworks, which were compromised to enable credit card skimming during checkout. The infrastructure identified during this period included dedicated C2 domains used for exfiltration, later flagged as malicious by multiple vendors on platforms such as VirusTotal. Following this detection, part of the infrastructure was taken offline and affected merchants remediated the intrusion, resulting in a temporary decline in activity indicators associated with the campaign.

{{< figure
src="img/urlscan-saved-search.webp"
alt="Saved search detections"
caption="Detections from a saved search rule used to identify new victims of the campaign. The spikes in the graph represent periods when the malicious script was found active on different e-commerce platforms."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

After a period of reduced activity, renewed indicators surfaced on a new merchant interacting with domains already associated with the campaign. On its own, this would be consistent with a simple continuation of the operation. Analysis of the payload collected from this case, however, revealed a functional change: in addition to the credit card capture routines, the code now contained logic dedicated to manipulating the PIX payment flow by replacing the QR code presented at checkout when PIX was selected as the payment method. This capability was not present in the earlier iteration of the campaign. At that point, this was no longer just a classic Magecart operation.

### PIX and the Evolution of Magecart Monetization

To understand the impact of this shift, it is necessary to briefly contextualize PIX within the Brazilian payment ecosystem. PIX is an instant payment system introduced by the Central Bank of Brazil, now used by most of the adult population and accepted across nearly every online retailer. Transfers settle in seconds and operate around the clock. At checkout, a PIX payment is presented in two interchangeable forms: a QR Code and a copy-and-paste string. Both carry the same information in a standardized format defined by the Central Bank, specifying the recipient's PIX key, the amount, and the merchant name. The QR Code is simply a visual rendering of that string.

Recovery in fraud cases is handled through the Special Return Mechanism (MED), created by the Central Bank in 2021 under Resolution BCB 103, which allows a victim's bank to request a block on the funds in the receiving account while the case is reviewed. Its effectiveness has been constrained by a structural limit: the original version could only act on the first receiving account, and fraudsters routinely moved funds through a chain of intermediary accounts before a contestation could be processed. MED 2.0, introduced by Resolution BCB 493 and mandatory for all PIX participants since February 2026, extends tracing across up to five layers of subsequent transfers and allows returns to be initiated from those downstream accounts, with a window of up to eleven days after contestation. It is still early to assess its impact in practice.

{{< figure
src="img/pix-qrcode.webp"
alt="Example of PIX QR Code checkout payment flow"
caption="Example of a PIX QR Code payment flow during checkout. Source: Modern Treasury (https://www.moderntreasury.com/learn/pix-payments)"
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

This design is what makes PIX hijacking viable. The payment payload is rendered entirely on the client side, and the customer has no reference to compare it against. A malicious script with access to the page can swap the QR Code and the copy-and-paste field for a payload pointing to an account it controls, matching the original amount so nothing looks out of place. The banking app does show the recipient name before confirmation, but customers are following a routine they have completed hundreds of times and rarely stop to read it. The transfer executes, the money is gone, and the exchange looks like a normal purchase.

{{< figure
src="img/pix-hijacking-flow.webp"
alt="Flow diagram of PIX hijacking showing real-time QR code replacement and payment redirection during checkout."
caption="Flow diagram illustrating PIX hijacking, where a malicious script replaces the original QR Code and copy-and-paste payment string in real time, redirecting funds to an attacker-controlled account."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

The economics are considerably better for the actor than card skimming. Stolen card data has to be validated, packaged, and cashed out through a chain of intermediaries, each taking a share and adding exposure, while a hijacked PIX payment arrives in full, in seconds, with no resale required. Timing also works in the attacker's favor, since a MED request only succeeds while funds remain in the receiving account and the victim here has no reason to suspect anything went wrong. 

For the merchant, the order is created normally but the payment confirmation never arrives, leaving the transaction sitting as pending or abandoned alongside genuinely incomplete checkouts. The discrepancy usually surfaces only when a customer calls asking why their paid order was never shipped, and during high-traffic periods in Brazilian retail such as Mother's Day, Easter, or Black Friday, even a short window translates into significant losses.

For a deeper look into Magecart operations and how web skimming has evolved across Latin America, I previously covered this in [When the Checkout Bleeds: An Analysis of Skimming Campaigns in LATAM](https://sagehollow.world/posts/adversary-tracking/when-the-checkout-bleeds/
). This context is important because PIX changes the economics of these operations.

## Mapping the Infrastructure
The campaign operates through an organized and modular infrastructure, built on a cluster of interconnected command and control domains. A clear division of labor is present across the codebase, with distinct modules handling credit card exfiltration and a dedicated component responsible for real-time PIX hijacking.

A key characteristic is the reuse of infrastructure across domains. Scripts hosted on one malicious node frequently exfiltrate credit card data to endpoints hosted on another campaign domain, establishing cross-domain communication between C2 nodes and forming a triangulated exfiltration pattern rather than isolated one-to-one flows. In contrast, the PIX hijacking module appears more stable, remaining tied to the same C2 domain and not following this cross-domain exfiltration behavior.

To better understand this behavior, an interactive graph was generated to map relationships between command and control domains and anonymized merchants. The visualization also highlights infrastructure rotation over time, likely in response to domains being flagged or taken down.

<div class="sage-graph"> <iframe src="https://eremit4.github.io/eremit4-graphs/brazilian-magecart-and-pix-hijacking/bmaph-graph.html" loading="lazy" style="width:100%; height:650px; border:none;"> </iframe> </div>

Not all nodes or domains listed are directly connected at the infrastructure level, due to the lack of observable infrastructure reuse in exfiltration paths and limited historical and temporal visibility across merchants in open-source data, preventing direct linkage to other domains. However, cross-referencing the graph reveals that distinct C2 domains rotated across the same set of merchants over time while retaining an identical code structure and matching signatures within the page body markup, a combination that is highly unlikely to occur by coincidence. This consistency across otherwise unlinked infrastructure provides strong confidence that these nodes belong to the same campaign, even where no direct network-level connection between domains was observed.

### Exfiltration Techniques
The majority of variants rely on standard POST requests to attacker-controlled endpoints, which serves as the stable baseline for credit card data collection. However, we observed possible experimentation involving the use of Discord webhooks for exfiltration. This tactic allows the actor to route stolen information through a high-reputation API, effectively bypassing basic domain blacklisting and blending malicious traffic with trusted HTTPS communication.

This technique has been widely adopted in the wild and was previously covered in detail in [The Mycelial Mage: Tracing a Spanish-Speaking Credential Theft Operation](https://sagehollow.world/posts/adversary-tracking/mycelial-mage/), where a similar migration from Telegram bots to Discord webhooks was observed following the same operational logic. Other exfiltration methods carry inherent risks for the attacker. Telegram bots can introduce misconfigurations that expose the chat where stolen data is sent, and custom API endpoints can offer surfaces vulnerable to hackback by researchers or law enforcement. Discord webhooks eliminate both risks by functioning as a write-only channel. They accept incoming payloads but offer no read access to what was previously sent, effectively functioning as a write-only channel that protects the attacker's harvested data even if the webhook URL is discovered and eliminating an entire class of defensive hackback that has historically been effective against Telegram-based operations.

{{< figure
src="img/discord-exfil.webp"
alt="Evidence of a malicious script exfiltrating credit card data via a Discord webhook."
caption="Evidence of a malicious script exfiltrating credit card data via a Discord webhook."
figclass="text-center"
class="mx-auto"
zoom="true"
>}}

The Discord exfiltration module also shows signs of AI-assisted development. The script contains detailed Portuguese comments describing each stage of execution, decorative section headers, and a consistent formatting structure that closely resembles code generated by large language models. This pattern was also observed in the Mycelial Mage investigation and is increasingly common across phishing kits active in the wild today. The growing accessibility of these models is lowering the barrier for developing and adapting malicious tooling, allowing operators to iterate faster, pivot between exfiltration channels with minimal friction, and produce functional code without deep programming expertise.

### Inside the PIX Hijacking Module

Unlike the credit card skimming component, which passively collects and exfiltrates data, the PIX hijacking module operates as an active proxy that intercepts and manipulates the payment flow in real time. One of the recovered scripts was deobfuscated for closer inspection, revealing the full logic behind this process.

The script begins with a simple but effective gate: it checks whether the current page URL matches the Magento `checkout/onepage/success` path, the standard route for the order confirmation page. If it does not match, the script remains completely inert. The script walks through every text node in the page looking for the string "Obrigado por sua compra!" ("Thank you for your purchase!"), the confirmation message displayed to the customer after an order is placed. Only when this text is found and confirmed to be visible on the screen does the script proceed with the replacement of the QR code. To handle cases where the QR code is rendered asynchronously after the initial page load, the script attaches a listener that reacts to any change in the page structure, re-checking for that trigger text every time an element is added or modified.

Once triggered, the script needs the exact transaction value to generate a matching PIX payment. The script does not parse formatted prices from the visible page, it reads a variable called `transactionTotal` from `window.dataLayer`, a data structure commonly used by Google Tag Manager to track e-commerce events such as purchases, cart additions, and page views. Many Magento stores push the order total into this structure as part of their analytics setup, and the attacker takes advantage of this as a clean, reliable data source. The value is converted to cents and sent to the C2 server, which responds with a JSON payload containing a base64-encoded QR code image and a corresponding copy-and-paste PIX string, both generated for the exact amount of the original transaction.

With the payload in hand, the script targets two specific elements in the page: the QR code image container (`#qrcode-pix`) and the input field for the copy-and-paste PIX string (`#pix-copiar-colar`). These selectors correspond to PIX payment modules commonly used in Brazilian Magento stores. If no image tag is found inside the QR container, the script injects one directly, accounting for implementations that render the QR code using canvas or other methods. The result is a seamless replacement: the customer sees a QR code that looks legitimate, scans it or copies the PIX string, and completes the payment to an attacker-controlled account without any visible indication that the payment destination was altered.

{{< figure
src="img/pix-hijacking.webp"
alt="Example of attacker-controlled API response returning a base64-encoded QR Code and a corresponding PIX copy and paste key used to replace the original payment."
caption="Example of attacker-controlled API response returning a base64-encoded QR Code and a corresponding PIX copy and paste key used to replace the original payment."
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

## Recommendations

Given the modular, reusable infrastructure behind this campaign, mitigation should focus on reducing the initial compromise surface and limiting the impact of a successful skimmer injection.

- Keep CMS and framework versions current. Outdated PHP framework installations, particularly Magento, remain a primary entry point. Apply security patches promptly and retire end of life versions.
- Enforce MFA and rotate credentials regularly. As covered in When the Checkout Bleeds: An Analysis of Skimming Campaigns in LATAM, attackers in this region often use stealer log credentials to access admin panels, including those of technology providers managing multiple stores, turning a single compromise into a supply-chain level incident. MFA on admin, hosting, and deployment access, combined with regular credential rotation, limits this exposure.
- Monitor checkout page integrity. Since manipulation happens client-side, server-side monitoring alone is insufficient. Use file integrity monitoring on checkout templates and periodic DOM-level auditing of the live checkout page.
- Reconcile PIX transactions against order status. Automate reconciliation between PIX confirmations and order records, alerting on orders that stay pending beyond the expected settlement window.
- Educate customers on payment verification. Prompt customers to confirm the recipient name shown in their banking app before completing a PIX transfer, since the attack relies on trust in the interface rather than a flaw in PIX itself.

## Conclusion

This activity reflects a shift in monetization, not in initial compromise. The actors retain traditional Magecart techniques for access and data collection, but introduce a mechanism to redirect PIX payments at the moment of checkout.

**There is no vulnerability in PIX itself**. The attacker does not interfere with the payment system, but with the client-side flow that presents the payment details to the user. By replacing the QR Code or "copy and paste" key, the payment is silently redirected to an attacker-controlled account.

The effectiveness of this approach relies on user trust in the checkout interface, not on weaknesses in the underlying payment infrastructure. As PIX adoption continues to grow across Brazilian e-commerce, this class of attack is likely to become a more common monetization path for actors who previously relied solely on card data resale.
