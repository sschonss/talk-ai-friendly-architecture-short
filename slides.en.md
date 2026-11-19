---
marp: true
theme: aifa
paginate: true
lang: en
title: AI-Friendly Architecture
description: Preparing real systems for AI agents
author: Luiz Schons
footer: 'AI-Friendly Architecture · luizschons.com'

---

<!-- _class: cover -->
<!-- _paginate: false -->
<!-- _footer: '' -->

# AI-Friendly Architecture

Putting AI to work like a developer, and faster

<br>

**Luiz Schons**

<!--
Talk based on the 9-article series at luizschons.com/series/ai-friendly-architecture.
Transition: open with the new-hire analogy. The personal intro comes right after.
-->

---

<!-- _class: statement -->

> AI is the team's **new hire**.

<!--
The analogy that guides the whole talk.
- It doesn't know the domain, the services, past decisions, or where the answers live.
- With well-documented knowledge, it finds answers on its own and gains autonomy.
- When it depends on a specific person, informal context, or someone's memory, it works worse.
Hold on to this line: we'll come back to it at the end.
Transition: "Before we go on, let me introduce myself."
-->

---

<div class="about">
<div>

# Luiz Schons

<p class="role">Senior Software Engineer @ <b>PicPay</b></p>

<div class="chips">
<span>Guarapuava, Paraná, Brazil</span>
<span>DevParaná</span>
<span>UTFPR</span>
<span>PHP</span>
</div>

<p class="site">luizschons.com</p>

</div>
<img class="photo" src="assets/luiz.jpg" alt="Luiz Schons">
</div>

<!--
Quick intro (30 seconds): who I am, where I work, where I'm from.
Transition: "Three acts, very practical."
-->

---

## The path

<div class="cols3">
<div class="card"><p class="muted small">Act 1</p><h3>Document</h3></div>
<div class="card"><p class="muted small">Act 2</p><h3>Deterministic tests</h3></div>
<div class="card"><p class="muted small">Act 3</p><h3>In practice</h3></div>
</div>

<!--
A short, practical talk: how to put AI to work like a developer, and faster.
- Act 1: documentation is the agent's context. Without it, the agent guesses.
- Act 2: AI is probabilistic; the safety net is deterministic (tests, contracts, CI, guardrails).
- Act 3: all of it together in a real task, from the ticket link to deploy.
Single example: payments domain, Payment Service, partial refund.
-->

---

<!-- _class: divider -->

<div class="kicker">Act 1 · Document</div>

# Documentation is context

<!--
The agent is the new hire who shows up every day. What it doesn't find, it invents. Documentation stops being bureaucracy and becomes working material.
-->

---

## The problem isn't always the model

<div class="compare">
<div class="compare-row">
<div class="compare-in"><span>Swap the model</span><span>Rewrite the prompt</span></div>
<span class="compare-arrow">→</span>
<div class="compare-ctx bad">Bad context</div>
<span class="compare-arrow">→</span>
<div class="compare-out bad">Weak decision</div>
</div>
<div class="compare-row">
<div class="compare-in"><span>Same model</span><span>Same prompt</span></div>
<span class="compare-arrow">→</span>
<div class="compare-ctx good">Good context</div>
<span class="compare-arrow">→</span>
<div class="compare-out good">Confident decision</div>
</div>
</div>

<p class="muted">Often, the problem is the <b>system</b>.</p>

<!--
How to read the diagram: in the top row, we swap the model and rewrite the prompt, but the context stays bad, and the decision stays weak. In the bottom row, we keep the same model and the same prompt, we only improve the context, and the decision becomes confident. The bottleneck is the context system, not the model.

How we got here:
1. We learned to scale systems.
2. We learned to modularize code.
3. We separated responsibilities, distributed services, organized teams by context.
4. Now: build software that AI agents can also understand, not only people.

The usual explanation for AI mistakes is "the model isn't good enough". Sometimes that's true.
But systems are designed for people who ALREADY have context. An in-house dev knows:
- where the relevant documentation lives;
- which decisions sit in old ADRs;
- which service owns each responsibility;
- where to look in an incident;
- which metrics point to a real problem.
The agent knows none of that by default. Context that is scattered, hidden, or inconsistent tanks the quality of the answer.
-->

---

## Incident: checkout latency

<p class="muted small">Without context</p>
<div class="flow muted-arrows">
<div class="bad">Wrong docs</div><span>→</span>
<div class="bad">Wrong logs</div><span>→</span>
<div class="bad">Generic hypothesis</div>
</div>

<br>

<p class="muted small">With context</p>
<div class="flow">
<div><code>docs/payments</code></div><span>→</span>
<div>Runbook</div><span>→</span>
<div>Queries</div><span>→</span>
<div class="ok">Focused hypothesis</div>
</div>

<!--
Introduce the domain that will stay with us until the end: payments. Scenario: checkout latency starts to climb (we'll call it INC-2291).
WITHOUT an AI-friendly architecture, the agent depends on luck: it looks for logs in the wrong service, reads dashboards without knowing which metrics matter, wastes time, and suggests generic hypotheses ("add more resources").
WITH organized context the path is direct: it enters the domain docs folder, finds the latency runbook, uses the ready-made queries, and arrives at a focused hypothesis.
In detail:
1. Identifies the affected service (Payment Service)
2. Consults the payments-domain docs
3. Correlates observability signals (provider latency, timeouts, deploys)
4. Finds related architecture decisions (ADRs)
5. Focuses on the most likely hypotheses
It isn't magic: the system was prepared so AI can work with less friction.
Where to start: incidents, onboarding, and changes in critical areas.
-->

---

<!-- _class: statement -->

> `200 OK` does not mean the **business** is working.

<!--
Common mistake: looking only at CPU, memory, error rate, and latency.
- A successful response can hide a business step that is failing.
- A few errors can hit key customers.
- A slow queue can destroy the experience.
Connect business events: approved payments, completed orders, processed documents, transactions that needed manual help.
Not every technical alert is a business incident; not every business problem shows up as a technical error.
After a deploy: which metrics to watch, what behavior is expected, how the old compares to the new. The agent helps write AND verify the change.
-->

---

<div class="dash">
<div class="dash-panel">
<div class="dash-head"><span>Status code · checkout</span><span class="dash-legend"><i class="ok"></i>2xx 99.9% <i class="err"></i>5xx 0</span></div>
<div class="dash-kpis"><div><b class="ok">200 OK</b><span>majority</span></div><div><b class="ok">0</b><span>errors</span></div></div>
<svg viewBox="0 0 720 380" class="dash-chart"><line x1="70" x2="700" y1="20" y2="20" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="98" y2="98" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="175" y2="175" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="252" y2="252" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="330" y2="330" stroke="#eceae4" stroke-width="1.5"/><rect x="73.9" y="92.6" width="18.4" height="237.4" rx="3" fill="#0f9d6b"/><rect x="73.9" y="90.6" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="100.2" y="84.7" width="18.4" height="245.3" rx="3" fill="#0f9d6b"/><rect x="100.2" y="82.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="126.4" y="61.5" width="18.4" height="268.5" rx="3" fill="#0f9d6b"/><rect x="126.4" y="59.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="152.7" y="67.5" width="18.4" height="262.5" rx="3" fill="#0f9d6b"/><rect x="152.7" y="65.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="178.9" y="51.2" width="18.4" height="278.8" rx="3" fill="#0f9d6b"/><rect x="178.9" y="49.2" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="205.2" y="54.5" width="18.4" height="275.5" rx="3" fill="#0f9d6b"/><rect x="205.2" y="52.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="231.4" y="65.3" width="18.4" height="264.7" rx="3" fill="#0f9d6b"/><rect x="231.4" y="63.3" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="257.7" y="61.1" width="18.4" height="268.9" rx="3" fill="#0f9d6b"/><rect x="257.7" y="59.1" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="283.9" y="82.7" width="18.4" height="247.3" rx="3" fill="#0f9d6b"/><rect x="283.9" y="80.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="310.2" y="84.6" width="18.4" height="245.4" rx="3" fill="#0f9d6b"/><rect x="310.2" y="82.6" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="336.4" y="106.0" width="18.4" height="224.0" rx="3" fill="#0f9d6b"/><rect x="336.4" y="104.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="362.7" y="117.0" width="18.4" height="213.0" rx="3" fill="#0f9d6b"/><rect x="362.7" y="115.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="388.9" y="118.2" width="18.4" height="211.8" rx="3" fill="#0f9d6b"/><rect x="388.9" y="116.2" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="415.2" y="114.7" width="18.4" height="215.3" rx="3" fill="#0f9d6b"/><rect x="415.2" y="112.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="441.4" y="134.7" width="18.4" height="195.3" rx="3" fill="#0f9d6b"/><rect x="441.4" y="132.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="467.7" y="130.7" width="18.4" height="199.3" rx="3" fill="#0f9d6b"/><rect x="467.7" y="128.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="493.9" y="115.3" width="18.4" height="214.7" rx="3" fill="#0f9d6b"/><rect x="493.9" y="113.3" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="520.2" y="98.6" width="18.4" height="231.4" rx="3" fill="#0f9d6b"/><rect x="520.2" y="96.6" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="546.4" y="96.7" width="18.4" height="233.3" rx="3" fill="#0f9d6b"/><rect x="546.4" y="94.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="572.7" y="88.9" width="18.4" height="241.1" rx="3" fill="#0f9d6b"/><rect x="572.7" y="86.9" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="598.9" y="62.5" width="18.4" height="267.5" rx="3" fill="#0f9d6b"/><rect x="598.9" y="60.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="625.2" y="75.0" width="18.4" height="255.0" rx="3" fill="#0f9d6b"/><rect x="625.2" y="73.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="651.4" y="47.0" width="18.4" height="283.0" rx="3" fill="#0f9d6b"/><rect x="651.4" y="45.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="677.7" y="56.9" width="18.4" height="273.1" rx="3" fill="#0f9d6b"/><rect x="677.7" y="54.9" width="18.4" height="2" fill="#d6455d" opacity="0"/><g class="ax"><text x="70" y="364" text-anchor="start">14:00</text><text x="228" y="364" text-anchor="middle">14:15</text><text x="385" y="364" text-anchor="middle">14:30</text><text x="542" y="364" text-anchor="middle">14:45</text><text x="700" y="364" text-anchor="end">15:00</text><text x="56" y="28" text-anchor="end">2k</text><text x="56" y="330" text-anchor="end">0</text></g></svg>
</div>
<div class="dash-panel">
<div class="dash-head"><span>Latency · checkout</span><span class="dash-legend"><i class="p75"></i>p75 <i class="p95"></i>p95 <i class="p99"></i>p99</span></div>
<div class="dash-kpis"><div><b class="warn">1.2 s</b><span>p75</span></div><div><b class="warn">2.2 s</b><span>p95</span></div><div><b class="bad">3.7 s</b><span>p99</span></div></div>
<svg viewBox="0 0 720 380" class="dash-chart"><line x1="70" x2="700" y1="20" y2="20" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="98" y2="98" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="175" y2="175" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="252" y2="252" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="330" y2="330" stroke="#eceae4" stroke-width="1.5"/>
<line x1="70" x2="700" y1="268" y2="268" stroke="#7a7d86" stroke-width="2" stroke-dasharray="8 8"/>
<text x="78" y="258" class="slo">SLO 800 ms</text>
<polyline points="70.0,316.9 96.2,316.9 122.5,316.5 148.8,315.3 175.0,316.8 201.2,315.9 227.5,315.7 253.8,316.3 280.0,315.9 306.2,317.1 332.5,316.1 358.8,313.6 385.0,309.7 411.2,306.9 437.5,303.1 463.8,298.0 490.0,293.3 516.2,288.2 542.5,281.1 568.8,275.1 595.0,269.5 621.2,261.6 647.5,254.4 673.8,245.8 700.0,238.0" fill="none" stroke="#e8b14a" stroke-width="4" stroke-linejoin="round"/>
<polyline points="70.0,306.0 96.2,303.3 122.5,306.7 148.8,305.5 175.0,304.2 201.2,306.5 227.5,305.2 253.8,307.0 280.0,304.5 306.2,304.2 332.5,303.0 358.8,297.9 385.0,294.7 411.2,286.7 437.5,279.4 463.8,270.9 490.0,261.9 516.2,250.0 542.5,238.4 568.8,228.3 595.0,214.9 621.2,203.9 647.5,187.3 673.8,172.8 700.0,156.0" fill="none" stroke="#c27803" stroke-width="4" stroke-linejoin="round"/>
<polyline points="70.0,287.7 96.2,291.0 122.5,290.4 148.8,288.7 175.0,292.7 201.2,289.9 227.5,291.8 253.8,292.1 280.0,292.4 306.2,285.1 332.5,283.1 358.8,274.2 385.0,263.4 411.2,248.8 437.5,240.7 463.8,223.9 490.0,207.6 516.2,188.5 542.5,170.8 568.8,151.3 595.0,134.6 621.2,112.3 647.5,90.3 673.8,63.6 700.0,38.9" fill="none" stroke="#d6455d" stroke-width="6" stroke-linejoin="round"/>
<g class="ax"><text x="70" y="364" text-anchor="start">14:00</text><text x="228" y="364" text-anchor="middle">14:15</text><text x="385" y="364" text-anchor="middle">14:30</text><text x="542" y="364" text-anchor="middle">14:45</text><text x="700" y="364" text-anchor="end">15:00</text><text x="56" y="28" text-anchor="end">4s</text><text x="56" y="183" text-anchor="end">2s</text><text x="56" y="330" text-anchor="end">0</text></g></svg>
</div>
</div>

<p class="muted small">illustrative example</p>

<!--
Illustrative example (fictional numbers) right after the line "200 OK does not mean the business is working".
Left: the panel everyone looks at. Almost everything is 200, zero 5xx. On the technical dashboard, it's all green.
Right: latency for the same checkout. p75 and p95 climb, p99 spikes and blows well past the 800 ms SLO.
The customer doesn't see an error: they see the checkout spinning, give up, and the payment never happens. The business is broken with a 200 OK.
It's the same for the agent: if it only looks at error rate, it concludes "it's healthy". It needs to know which signals matter (latency by percentile, approved payments) and what is normal.
-->

---

## Context Architecture

<div class="cols">
<div class="card bad">

### It isn't
- A giant document
- Everything in a vector DB

</div>
<div class="card good">

### It is
- Paths between sources
- Less effort to find things

</div>
</div>

<br>

<p class="muted">Having information is <b>not</b> having context.</p>

<!--
To understand a system, the new hire needs: code, documentation, architecture decisions, tickets and epics, past incidents, dashboards and metrics, conversations with the team.
In most companies the information EXISTS, but it is DISCONNECTED: code in a repo, decisions in a wiki, tickets in another system, incidents in the ops platform, and the key context in a few people's heads.

Definition: a way to organize and CONNECT the knowledge sources that explain a system.
Questions it should answer:
- What is this service responsible for?
- Who depends on it?
- Why was it built this way?
- Which business rule applies?
- How do we know it works?
- What happened the last time this changed?
It doesn't remove the reasoning: it reduces the effort of finding the right information.
-->

---

## A network, not a place

<svg class="network" viewBox="-10 0 1100 440" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Network of connected context sources">
<defs><marker id="arr" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker></defs>
<line x1="500" y1="110" x2="730" y2="40" stroke="#c3c8d4" stroke-width="2"/>
<line x1="500" y1="110" x2="730" y2="220" stroke="#c3c8d4" stroke-width="2"/>
<line x1="500" y1="110" x2="960" y2="110" stroke="#c3c8d4" stroke-width="2"/>
<line x1="500" y1="330" x2="730" y2="220" stroke="#c3c8d4" stroke-width="2"/>
<line x1="500" y1="330" x2="730" y2="400" stroke="#c3c8d4" stroke-width="2"/>
<line x1="500" y1="330" x2="960" y2="330" stroke="#c3c8d4" stroke-width="2"/>
<line x1="730" y1="40" x2="730" y2="220" stroke="#c3c8d4" stroke-width="2"/>
<line x1="730" y1="220" x2="730" y2="400" stroke="#c3c8d4" stroke-width="2"/>
<line x1="730" y1="40" x2="960" y2="110" stroke="#c3c8d4" stroke-width="2"/>
<line x1="730" y1="400" x2="960" y2="330" stroke="#c3c8d4" stroke-width="2"/>
<line x1="960" y1="220" x2="960" y2="110" stroke="#c3c8d4" stroke-width="2"/>
<line x1="960" y1="220" x2="960" y2="330" stroke="#c3c8d4" stroke-width="2"/>
<line x1="730" y1="220" x2="960" y2="220" stroke="#c3c8d4" stroke-width="2"/>
<line x1="157" y1="220" x2="199" y2="220" stroke="#3b5bdb" stroke-width="2.5" marker-end="url(#arr)"/>
<line x1="340" y1="190" x2="432" y2="144" stroke="#3b5bdb" stroke-width="2.5" marker-end="url(#arr)"/>
<line x1="357" y1="220" x2="649" y2="220" stroke="#3b5bdb" stroke-width="2.5" marker-end="url(#arr)"/>
<line x1="340" y1="250" x2="432" y2="296" stroke="#3b5bdb" stroke-width="2.5" marker-end="url(#arr)"/>
<rect x="-2" y="192.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="80" y="228" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Task</text>
<rect x="198" y="192.0" width="164" height="56" rx="14" fill="#3b5bdb" stroke="#3b5bdb" stroke-width="1.5"/>
<text x="280" y="228" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="700" fill="#ffffff">Domain</text>
<rect x="418" y="82.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="500" y="118" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Services</text>
<rect x="418" y="302.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="500" y="338" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Docs</text>
<rect x="648" y="12.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="730" y="48" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Decisions</text>
<rect x="648" y="192.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="730" y="228" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Code</text>
<rect x="648" y="372.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="730" y="408" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Tickets</text>
<rect x="878" y="82.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="960" y="118" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Dashboards</text>
<rect x="878" y="192.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="960" y="228" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Incidents</text>
<rect x="878" y="302.0" width="164" height="56" rx="14" fill="#ffffff" stroke="#e4e1da" stroke-width="1.5"/>
<text x="960" y="338" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#1d1f24">Runbooks</text>
</svg>

<!--
A single page called "How the company works" goes stale fast.
Each source stays where it works best. What changes: CLEAR LINKS between them.
In the diagram: the task leads to the domain, and the domain is the entry point to services, code, and docs. From there, everything references everything else: the decision explains the code, the dashboard points to the runbook, the incident ties the code to the dashboard, the ticket cites the decision. Nothing is centralized; everything is reachable.
"The goal is not to centralize knowledge. It is to make it easy to find."
-->

---

<!-- _class: divider -->

<div class="kicker">Act 2 · Deterministic tests</div>

# AI proposes. The test decides.

<!--
Context improves the odds of a good answer, but it doesn't guarantee one. What guarantees it is whatever gives the same answer every time.
-->

---

<!-- _class: statement -->

> AI is **probabilistic**. Your system needs to be **deterministic**.

<br>

<p class="muted">Same input, different outputs.</p>

<!--
An idea that matters more every day.
- The model is probabilistic: same prompt, same context, and the answer can change on every run. Sometimes it changes a little, sometimes it changes the decision.
- Example: ask the agent twice to implement the partial refund. Once it uses the public contract; the other time it imports an internal Orders class. Both "look right".
- This is not a defect to be fixed with a better prompt: it is the nature of the tool. Temperature zero doesn't fix a different context, a new model version, or a tool that answered something else.
- So the question changes: it is not "how do I make the model always get it right?", it is "what in my system checks, the same way every time, what the model produced?".
Transition: "On one side, what varies. On the other, what cannot vary."
-->

---

## Probabilistic × Deterministic

<div class="cols">
<div class="card warn">

### Probabilistic
- Generates
- Suggests
- Varies on every run

</div>
<div class="card good">

### Deterministic
- Tests
- Contracts and schemas
- Types and CI
- Fitness functions
- Guardrails and permissions

</div>
</div>

<p class="muted">The model proposes. The deterministic layer <b>decides</b>.</p>

<!--
Left: the model. It generates code, suggests hypotheses, writes queries. It's great at that, but each run can come out differently.
Right: everything that gives the SAME answer for the SAME input, every time:
- Tests: the acceptance criterion becomes code (the refund never exceeds the amount paid).
- Contracts and schemas: the v2 refund API and the events validate the format, not the intent.
- Types and CI: the pipeline runs for every PR, from a human or from an agent.
- Fitness functions (e.g. deptrac): "Payments does not import Orders internals" is a rule in code, checked on every PR, even months later.
- Guardrails and permissions: they check the action, the arguments, and the environment BEFORE execution; what is not allowed simply does not run.
The deterministic layer turns "looks right" into "proven right". It doesn't trust the agent: it checks the agent.
This is not new: it is engineering we already did. The difference is that it stopped being a good practice and became a requirement, because the volume of generated code is higher and the generator is not predictable.
A prompt guides, it does not protect: a rule that lives only in the prompt is not a barrier. The skill guides; the guardrail enforces.
Transition: "And the prompt? It is not a barrier."
-->

---

<!-- _class: statement -->

<p class="kicker-dark">Guardrails</p>

> A prompt **guides**. It does not **protect**.

<br>

<p class="muted">The skill guides. The guardrail enforces.</p>


<!--
"Use the production API for reads only." "Never hit admin endpoints."
Rules in the prompt are not a barrier: the agent receives new docs, combines sources, reads the code; the text may have conflicting instructions.
Checks inside the skill or the MCP have the same weakness.
MCP connects agents to systems, but it is not a security boundary. Authorization lives in the service that protects the resource.
The skill GUIDES. The guardrail ENFORCES.
The skill should say: what is read-only, what needs approval, which data can be exposed, when the evidence is insufficient, when to hand off to a human.
BUT limits cannot exist only as an instruction. Fixed guardrails check permissions, arguments, environment, and impact BEFORE the action.
Never assume the agent will follow a security rule on its own.
How to start: pick a recurring problem and watch the expert. Which questions come first? Which sources? How do they interpret? Which hypotheses do they discard? When do they escalate? What output is useful?
Transition: "And then comes the temptation: use AI itself to validate. That's not the best path."
-->

---

## Don't use AI as the test

<div class="cols">
<div class="card bad">

### AI replays the scenario
- Pays tokens every time
- Slow
- The result varies

</div>
<div class="card good">

### AI writes the test, CI runs it
- Near-zero cost
- Fast
- Always the same

</div>
</div>

<p class="muted">AI writes the test. It is <b>not</b> the test.</p>

<!--
Common mistake: asking the agent to "test" by opening the system, simulating the flow, reading logs, and saying whether it worked. Every time.
Problems:
- It costs: each validation burns tokens again.
- It's slow: the agent reasons, calls tools, waits.
- It varies: the same request can say "passed" today and "failed" tomorrow. A probabilistic validator is not proof.
Better: use AI ONCE to write the test or the script (concurrency, idempotency, a refund never larger than the amount paid). After that it runs in CI on every PR, for free, fast, and always the same.
The agent is still useful: it writes the test, interprets the failure, proposes the fix. The verdict comes from the test.
Transition to Act 3: "Let's see all of this in a real task."
-->

---

<!-- _class: divider -->

<div class="kicker">Act 3 · In practice</div>

# From the ticket link to deploy

<!--
Put it all together in a real task: a partial refund in the Payment Service.
-->

---

## In practice: one link, all the context

<svg class="network" viewBox="0 0 1640 620" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="From the task to the domain page, and from there to every context source">
<defs><marker id="arrc" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker></defs>
<line x1="323" y1="300" x2="525" y2="300" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="45" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="147" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="249" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="351" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="453" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="555" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<rect x="60.0" y="260.0" width="260" height="80" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="190" y="311" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">PAY-1432</text>
<rect x="535.0" y="260.0" width="330" height="80" rx="16" fill="#3b5bdb" stroke="#3b5bdb" stroke-width="2"/>
<text x="700" y="311" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#ffffff">Payment Service</text>
<rect x="1255.0" y="10.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="56" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Repository</text>
<rect x="1255.0" y="112.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="158" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">ADR-012</text>
<rect x="1255.0" y="214.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="260" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Contract v2</text>
<rect x="1255.0" y="316.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="362" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Runbook</text>
<rect x="1255.0" y="418.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="464" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Dashboard</text>
<rect x="1255.0" y="520.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="566" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">INC-2291</text>
<text x="190" y="384" text-anchor="middle" font-family="Inter, sans-serif" font-size="24" font-weight="700" letter-spacing="2" fill="#7a7d86">INPUT</text>
<text x="700" y="384" text-anchor="middle" font-family="Inter, sans-serif" font-size="24" font-weight="700" letter-spacing="2" fill="#7a7d86">ENTRY POINT</text>
</svg>

<p class="muted">The input was a link. The context came from the <b>architecture</b>.</p>

<!--
This is the point of the whole talk, in one slide:
- You sent a link (the ticket).
- The ticket pointed to the domain page.
- The page pointed to everything the agent needs: code, decision, contract, runbook, dashboard, incidents.
The agent didn't need a giant prompt or someone explaining the system. It navigated the Context Architecture.
And it doesn't read everything: it follows only the links relevant to the task (here, ADR-012 and the contract).
-->

---

## In practice: from the plan to the PR

<div class="chat">
<div class="msg agent"><span class="who">Agent</span><div class="skills"><span class="skill">architecture</span><span class="skill">testing</span></div>Plan: reuse the transaction and track the refunded total, as in ADR-012. Tests for concurrency, idempotency, and failure after the provider confirms.</div>
<div class="msg agent"><span class="who">Agent</span><div class="skills"><span class="skill">delivery</span></div>PR opened: <a>#482 · Partial refund by item</a>. Tests passing, fitness functions ok. Change-request draft ready.</div>
<div class="msg user"><span class="who">You</span>I sent the PR to the team for approval and I'm lining up the change request. Once everything is approved, I'll ship it and let you know here.</div>
</div>

<!--
- Architecture and testing skills: the plan rests on the recorded decision (ADR-012), not on a new idea. The tests come from the success criteria.
- Delivery skill: the agent PREPARES the PR and the change record. The guardrails still apply: fitness functions run in the pipeline (the Act 2 rule: Payments does not import Orders internals; the PR passed because the agent used the public contract) and a person is the one who approves.
- Preparing ≠ executing: the team reviews the PR and the change request goes through approval. Only after that does the person ship to production through the pipeline.
-->

---

## In practice: deploy and monitoring

<div class="chat">
<div class="msg user"><span class="who">You</span>Change request approved, I shipped to production. Can you watch it for me?</div>
<div class="msg agent"><span class="who">Agent</span><div class="skills"><span class="skill">observability</span></div>Watching the <a>payments dashboard</a>. 30 min after the deploy:
<ul><li>Partial refunds confirmed, no duplicates</li><li>p95 latency stable</li><li>No divergence between provider and database</li></ul>I'll alert if the failure rate goes above 2%.</div>
<div class="msg agent"><span class="who">Agent</span><div class="skills"><span class="skill done">docs updated</span></div>I updated the Payment Service page with the partial-refund runbook.</div>
</div>

<!--
- Observability skill: the agent knows which dashboard to watch because the link is on the domain page. It compares against the baseline and the success criteria defined before the code.
- The alert threshold (2% failure) was defined by the team; the agent watches and alerts, the decision to continue or roll back is human.
- Closing the loop: the documentation is updated, and the next task starts with more context.
END OF THE SIMULATION and of the main content: from the ticket link to production, with the docs updated for the next task.
Transition: "Takeaways."
-->

---

## Takeaways

<div class="cols3">
<div class="card big">Document<div class="small muted">docs and ADRs</div></div>
<div class="card big">Deterministic tests<div class="small muted">CI decides</div></div>
<div class="card big accent">Let AI propose<div class="small muted">and write tests</div></div>
</div>

<br>

<p class="muted">Less guessing. More <b>architecture</b>.</p>

<!--
Three things for Monday:
1. Document: docs with an owner, recorded decisions (ADRs), a domain page, and runbooks. That is the agent's context, and it helps people first.
2. Deterministic tests: tests, contracts, types, CI, fitness functions, and guardrails are the safety net.
3. Let AI propose: it generates code, tests, and hypotheses. What validates them is the deterministic layer, not another round of AI.
Feedback: each task returns knowledge to the docs, and the next one starts with more context.
Transition: the article series and questions.
-->

---

<div class="qr-slide">
<div>

<p class="kicker-dark">This talk is based on my article series</p>

# AI-Friendly Architecture

<p class="muted">9 articles</p>

<p class="site">luizschons.com/series/ai-friendly-architecture</p>

</div>
<figure class="qr">
<img src="assets/qr-serie.svg" alt="QR code for the series">
<figcaption>Scan to read the series</figcaption>
</figure>
</div>

<!--
To go deeper: the talk is a summary of the 9-article series (luizschons.com/series/ai-friendly-architecture). Leave a few seconds for anyone who wants to scan.
Then go to questions.
-->

---

<!-- _paginate: false -->
<!-- _footer: '' -->

<p class="kicker-dark">Thank you!</p>

# Questions?

<div class="contacts">
<div><span class="contact-label">Site</span><span class="contact-value">luizschons.com</span></div>
<div><span class="contact-label">LinkedIn</span><span class="contact-value">linkedin.com/in/luiz-schons</span></div>
<div><span class="contact-label">GitHub</span><span class="contact-value">github.com/sschonss</span></div>
</div>

<!--
Open for questions. Leave this slide up during Q&A.
Contacts: site, LinkedIn, and GitHub.
This short version has no feedback QR: the long talk's form is from a different event.
-->
