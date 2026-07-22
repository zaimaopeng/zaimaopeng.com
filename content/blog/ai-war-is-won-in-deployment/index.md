---
title: "When Models Leave the Lab, the Real Competition Begins"
date: 2026-07-22T22:35:46+10:00
lastmod: 2026-07-22T22:48:24+10:00
description: "Model capability is becoming abundant. What remains scarce is the ability to connect models to data, responsibility, and daily work—and turn one deployment into a reusable product. That is why FDEs matter, and where FitMed needs to position itself."
tags: ["AI", "FDE", "Enterprise AI", "Medical Devices", "FitMed", "Startups"]
categories: ["Opinion"]
showTableOfContents: true
slug: "ai-war-is-won-in-deployment"
draft: false
---

Over the past few days, I have been mapping the real delivery chain behind patient-matched implants for FitMed.

On the surface, the question seems to be whether AI can generate a candidate design faster. Once I laid out a case from imaging input through professional review, version approval, manufacturing handoff, and product release, the question changed. A faster algorithm improves only one part of the chain. If the data does not connect, approval authority is unclear, or nobody owns exceptions, the local speed gain disappears into waiting and rework.

That exercise changed how I understand the recent rise of Forward Deployed Engineers, or FDEs.

FDE is often treated as a new job title for engineers who can work with customers. That description is not wrong, but it is too small. FDE is better understood as a mechanism for finishing product development in the customer's real environment. A team enters the work itself, identifies a problem worth changing, connects the data and systems, defines what people and machines are allowed to decide, proves the result, and carries what it learns back into the product.

My view of enterprise AI is therefore straightforward: models still matter, but the centre of value is moving toward deployment. The scarce capability is no longer just building a stronger model. It is turning a model into a system that can run, be governed, produce evidence, and become easier to deploy for the second customer.

## Stronger models make deployment harder, not easier

Traditional software usually arrives with predefined functions and operating boundaries. Customers adapt through configuration. General-purpose models work in the opposite direction. They appear able to do almost anything, but they do not know the context unique to each organisation: who owns the budget, which data may be used, which exception requires escalation, who has final approval, what counts as a better result, and who takes over when the system fails.

These questions are often called the “last mile,” as if implementation were a short section added after the product is complete. That metaphor now looks misleading. As a model moves closer to real action, permissions, governance, and responsibility become more complex. The more general the capability, the more important the organisation's unwritten rules become.

The overlooked variable is not model size. It is whether the organisation can connect the technology to daily operations.

Palantir's public work with Trinity Industries is a useful example. Trinity needed to review large volumes of maintenance records and invoices generated across its railcar fleet. A chatbot that answers questions would not solve the operational problem. The real work was to organise data, audit rules, work queues, and human review into an invoice-auditing process that could run every day. [The case](https://www.youtube.com/watch?v=D5t6384lqoE) shows the distance between model capability and a business result.

Model companies are now assigning capital and organisational weight to that distance. [OpenAI launched the OpenAI Deployment Company](https://openai.com/index/openai-launches-the-deployment-company/) with more than USD 4 billion in initial investment and an agreement to acquire Tomoro, which would bring approximately 150 FDEs and deployment specialists; at announcement, the transaction remained subject to regulatory approval. [Google](https://blog.google/company-news/inside-google/around-the-globe/google-asia/singapore-government-partnership/) is expanding its FDE team at the Google Cloud Singapore Engineering Center to help companies scale agentic enterprise transformation. IBM has proposed a multi-role Forward Deployed Unit.

The names differ. The direction is the same: these companies are moving closer to customer outcomes.

## FDE is a contest for the position closest to the result

An enterprise AI system usually has to complete three movements before it becomes valuable: build, prove, and reuse.

Building begins with selecting a high-value workflow, then connecting the data, tools, roles, permissions, and exception paths. Connecting an API is only one step. Proving requires a baseline, test set, acceptance threshold, and failure conditions. One successful output is nowhere near enough. Reuse is harder still. The first deployment must leave behind more than custom code. It should produce test structures, configuration templates, integration methods, operating procedures, and product decisions that make the second deployment easier.

Build alone produces a project. Proof gives the customer a reason to buy and renew. Reuse is where the supplier begins to own a product.

This is the most important difference between FDE and conventional implementation. An FDE team does more than install an existing product; it also discovers what the product should become. [OpenAI's description of its Deployment Company](https://openai.com/index/openai-launches-the-deployment-company/) places diagnosis, workflow selection, building, testing, and production deployment inside the same operating model. [IBM](https://www.ibm.com/think/perspectives/forward-deployed-units-ibm-consulting-field-model-scaling-ai-transformation) goes further by grouping customer outcome leaders, domain specialists, architects, engineers, and data roles into a field unit. The customer environment becomes both the delivery site and a feedback instrument for the product and model teams.

But proximity to the result also means proximity to responsibility. [CIO's analysis of the FIS and Anthropic financial-agent work](https://www.cio.com/article/4167981/anthropics-financial-agents-expose-forward-deployed-engineers-as-new-ai-limiting-factor.html) draws an important distinction: being able to audit what a system did does not mean the organisation has correctly decided what the system is authorised to do. Logs, traceability, and monitoring are necessary, but they do not resolve decision rights, human review, exception escalation, or final accountability.

The opportunity and the risk come from the same mechanism. A team that works inside customer operations learns the real problems and can influence workflows, evaluation standards, and product direction. Yet if every engagement requires senior people to rebuild the solution, more customers simply create more cost. FDE can generate product compounding, or it can turn a company into an unbounded services business.

The test is simple: is the second customer easier to serve than the first? If templates are reused while custom code and additional senior hours fall, field work is becoming product. If revenue growth continues to require the same proportion of custom development and founder time, the company is still selling labour.

## Medical devices need evidence-oriented FDE

Patient-matched implants push the deployment problem into a more demanding setting.

Every patient brings different imaging, anatomy, clinical goals, and manufacturing constraints. Software cannot enumerate every case in advance. The work also crosses clinicians, case coordinators, design engineers, senior reviewers, manufacturing, regulatory and quality teams, procurement, and data controllers. A point algorithm may shorten one task without removing waiting, version errors, repeated clarification, or records reconstructed at the end.

More importantly, the output may enter a product responsibility system as a candidate design and as process evidence. Clinical confirmation, engineering verification, manufacturer release, and pre-operative decisions cannot be collapsed into a vague idea of “customer approval.” The closer AI moves to design generation, the more clearly versions, permissions, human review, exception handling, and stop mechanisms must be expressed in the system.

Medical devices therefore need evidence-oriented FDE. Technical deployment has to be designed together with data rights, quality-management-system interfaces, human review, version approval, escalation, and responsibility boundaries. This raises the threshold, but it also creates the long-term value. A general model can be replaced. A point algorithm can be overtaken. A working method embedded in a customer's quality system, capable of organising cases and evidence reliably, is much harder to remove.

China's [Implementation Plan for Digital and Intelligent Transformation of the Pharmaceutical Industry (2025–2030)](https://www.miit.gov.cn/zwgk/zcwj/wjfb/tz/art/2025/art_13998d1c720e41438c5d25a943101f76.html) identifies medical-device design and development management as a representative digitalisation scenario, spanning planning, inputs, outputs, transfer, review, verification, validation, and change. The signal is important: the industry needs more than faster geometry generation. It needs a way to organise design knowledge, process responsibility, and quality evidence as an enterprise capability.

This is how FitMed should understand FDE. We do not need to chase a fashionable title or present ourselves as a general AI consultancy. The more defensible position is a case-design delivery layer above the licensed manufacturer's quality management system. Human-reviewed candidate designs provide the entry point; case states, professional input, effective versions, approvals, manufacturing handoff, and process evidence create the durable system.

FitMed should not take over the authority of clinicians, legal manufacturers, quality systems, manufacturing teams, or product-release owners. Its job is to make those boundaries clear, executable, and traceable as a case moves through delivery.

## FitMed's position cannot stop at “faster design”

“We use AI to design implants faster” is easy to understand, but it is not a durable position. Faster candidate design may open the door. It does not explain why a customer will continue using the system, nor does it ensure that the improvement will appear in acceptance, payment, or unit economics.

FitMed's first real product is neither an isolated algorithm nor a consulting report. It is a productised customer-onboarding method combined with a case control plane.

A cranial-defect reconstruction module gives customers a clear point of entry and a visible candidate-design efficiency benefit. The general case system organises roles, versions, approvals, open issues, and evidence. Enterprise connections place the platform inside customer data environments, manufacturing rules, and quality systems. The early field team learns the workflow, completes onboarding, proves the result, and feeds the lessons back into the product.

The algorithm is the key. The case control plane is the position we are trying to enter.

Every customer engagement must also leave two kinds of result. One belongs to the customer: did active work, waiting, revision, total delivery time, or professional capacity improve? The other belongs to the product: can the case object, responsibility model, test set, exception taxonomy, configuration template, and handover method be reused for the next customer?

Customer outcomes without reusable assets lead to a services company. Reusable assets without outcomes that a customer will accept and pay for lead to a product demonstration. Both have to occur.

The next job is to obtain evidence in the right order. Begin with a completed case and reconstruct how it was actually delivered, including waiting, revision, and responsibility breaks. Then establish a comparable baseline across consecutive cases. Only after data rights, quality interfaces, contracts, security, responsibility, and named human approval are in place should controlled shadow evaluation begin. A paid pilot comes later, and only if the preceding evidence supports it.

This is still a validation path. It is not a signed order, an authorised production pilot, or proven ROI.

External momentum around FDE only tells FitMed where to look for value. It does not prove that FitMed has captured that value. Investment by OpenAI and Google does not establish the profitability of deployment businesses, and vendor case studies cannot replace independent performance evidence. A strategic fit between FitMed's architecture and the FDE mechanism does not mean we have already found customers or achieved product-market fit.

The strategy also has clear failure conditions. It should be revised, and if necessary paused, if the main delay lies in manufacturing, procurement, or clinical steps that FitMed cannot influence; if manufacturers will not pay for coordination and evidence capability; if every account requires large-scale custom development; if the second deployment consumes the same senior support as the first; or if a second implant category cannot reuse the general case system.

The opposite evidence would be equally clear. FitMed begins to turn the trend into a product capability when a first paid statement of work establishes a real baseline, consecutive cases show attributable delivery improvement, a second account reuses configurations and tests, and additional senior hours fall with experience.

The strongest public description of FitMed should therefore explain the work we are prepared to do, rather than claim a place in an AI wave:

> We enter the real delivery environment for patient-matched implants and organise imaging, candidate design, professional review, version approval, manufacturing handoff, and process evidence into an operational, traceable case system. Each controlled delivery then contributes reusable infrastructure for the next customer, the next product category, and the next generation of algorithms.

Cranial reconstruction is the first entry point. The algorithm is the first key. FDE is how we enter the customer environment. The case control plane is the long-term position.

The next question is no longer whether FDE is fashionable. FitMed needs to prove that customers will pay for this position—and that the second delivery becomes easier than the first.

---

## Sources

- [OpenAI launches the OpenAI Deployment Company](https://openai.com/index/openai-launches-the-deployment-company/), OpenAI, 11 May 2026.
- [Accelerating AI impact in Singapore](https://blog.google/company-news/inside-google/around-the-globe/google-asia/singapore-government-partnership/), Google, 20 May 2026.
- [Forward-deployed units: IBM Consulting's field model for scaling AI transformation](https://www.ibm.com/think/perspectives/forward-deployed-units-ibm-consulting-field-model-scaling-ai-transformation), IBM, 18 June 2026.
- [Anthropic's financial agents expose forward-deployed engineers as new AI limiting factor](https://www.cio.com/article/4167981/anthropics-financial-agents-expose-forward-deployed-engineers-as-new-ai-limiting-factor.html), CIO, 6 May 2026.
- [Palantir AI FDE × Trinity Industries](https://www.youtube.com/watch?v=D5t6384lqoE), Palantir Technologies, 2026.
- [Implementation Plan for Digital and Intelligent Transformation of the Pharmaceutical Industry (2025–2030)](https://www.miit.gov.cn/zwgk/zcwj/wjfb/tz/art/2025/art_13998d1c720e41438c5d25a943101f76.html), Ministry of Industry and Information Technology and six other departments.
