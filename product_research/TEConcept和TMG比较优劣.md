
Here is the English translation:

---

Here is a practical comparison of the two offers. My brief conclusion upfront: **TEConcept is clearer, more concrete, and easier to budget in the short term**, while **TMG TE appears more flexible and scalable, but also more complex and associated with additional costs**.

## 1) Quick Overview

| Criterion     | TMG TE GmbH                                                                | TEConcept                                             |
| ------------- | -------------------------------------------------------------------------- | ----------------------------------------------------- |
| Offer date    | April 27, 2026                                                             | May 20, 2025                                          |
| Offer number  | 20026-3                                                                    | AN-1878-3201-5230                                     |
| Nature        | Modular portfolio with many options                                        | Concrete, bundled package                             |
| Pricing model | No fixed total price                                                       | Fixed total price available                           |
| Fixed amounts | e.g., Stack SDL €20,000, FDL €30,000, In-Design SDL €25,000, support extra | Net €27,200, gross €32,368                            |
| Production    | Additional production licenses required                                    | Manufacturing License already included                |
| Maintenance   | 18% p.a. of license value, no technical support included                   | 1 year of maintenance updates included                |
| Support       | Extra, €1,800/day                                                          | Only partially specified, T&M for additional services |

---

## 2) TMG TE GmbH – Pros and Cons

### Advantages

**1. Highly flexible and scalable.**
The offer is modular: development licenses, in-design variants, gateway apps for PROFINET / EtherNet/IP / EtherCAT, production license packages, test tools, and conformance services can all be combined. This is beneficial if you have not yet finalized your architecture, fieldbuses, or production volumes.

**2. Well suited for product families and growth.**
With SDL/FDL and tiered production licenses up to a flat-rate model, the structure is geared toward scaling and product variants. If you plan multiple device types or higher volumes, this can be strategically advantageous.

**3. Broader IO-Link ecosystem.**
Beyond the master stack, additional components are available such as gateway apps, test devices, device tools, conformance testing, and physical layer testing. This is helpful if you need more than just a software stack.

**4. In-design and source code options available.**
If you want deeper control or hardware-level customization, TMG TE provides structured options for that.

### Disadvantages

**1. No fixed total price — harder to budget.**
Unlike TEConcept, there is **no clear final price**. Depending on licenses, production volumes, gateways, maintenance, and support, the total project cost can increase significantly.

**2. Support is not included.**
Importantly, **licenses and maintenance do not include technical support**. Support is charged separately at **€1,800 per day**, which can become expensive during integration, debugging, and commissioning.

**3. Ongoing additional costs.**
Maintenance costs **18% of license value per year**, and **production licenses (PROD)** must be purchased separately. This increases total cost of ownership.

**4. More effort required on the customer side.**
The offer indicates that hardware adaptations and BSP adjustments must be handled by the customer. This can be a risk if your embedded team is limited.

**5. Higher cost to resume maintenance.**
If maintenance is interrupted, re-entry incurs additional fees, which can be contractually inconvenient.

---

## 3) TEConcept – Pros and Cons

### Advantages

**1. Clear and complete pricing structure with fixed total.**
TEConcept specifies four concrete items:

* Manufacturing License: **€12,800**
* Basic Package with Source Code: **€6,600**
* MAX14819/MAX14819A driver: **€2,800**
* Porting to STM32F1xx: **€5,000**
  Total: **€27,200 net / €32,368 gross**
  This makes internal approval and budgeting much easier.

**2. More product-oriented package.**
Compared to the catalog-like TMG offer, this feels tailored: stack, production license, PHY driver, and MCU porting are already included.

**3. 1 year of maintenance updates included.**
This reduces short-term follow-up costs.

**4. Source code included in multiple components.**
Licenses are provided for stack, driver, and porting — beneficial for integration, debugging, and long-term control.

**5. Manufacturing license already covered.**
Production rights are included, reducing the risk of missing a critical licensing component later.

### Disadvantages

**1. Less flexible than TMG TE.**
The offer is more specific: tied to **MAX14819/MAX14819A** and **STM32F1xx**. Changes in hardware or product strategy may lead to additional costs.

**2. Certain components explicitly excluded.**
A **driver for volatile memory is not included**, which may require additional work depending on your architecture.

**3. Additional services billed separately.**
Time-based services are charged on a **T&M basis**, meaning extra costs may arise for issues or changes.

**4. Offer is non-binding.**
It is explicitly described as **non-binding**, which is weaker from a procurement perspective.

**5. Less visible extensibility.**
Compared to TMG TE, fewer additional modules, testing services, and scaling options are evident in this offer.

---

## 4) Direct Comparison by Decision Criteria

### If **predictability and fast start** are important

Then **TEConcept** is stronger. You get a clearly priced, ready-to-use package.

### If **flexibility and scalability** are important

Then **TMG TE** is stronger. It acts like a modular toolkit for multiple product lines and scaling.

### If **follow-up costs** are important

**TEConcept has a short-term advantage** due to included maintenance and transparent pricing.
**TMG TE** involves ongoing costs for maintenance, support, and production licenses.

### If **technical support during integration** is important

**TMG TE is weaker in this offer**, since support is not included.
TEConcept is not fully explicit either, but appears closer to your actual target setup.

---

## 5) Final Recommendation

Choose **TEConcept** if you want a **concrete, quickly implementable solution** with **clear upfront costs**.

Choose **TMG TE** if you want a **strategically flexible platform**, expect **multiple product variants**, or plan **future scaling**, and are willing to manage the added commercial complexity.

**From a purely commercial perspective based on the provided documents**, **TEConcept currently appears more attractive**, as the offer is clearer, more complete in pricing, and closer to a ready-to-use solution.
However, **TMG TE could be the better long-term choice** if you truly need and leverage the additional flexibility.

---

If you want, I can next create a **decision matrix evaluating cost, risk, flexibility, time-to-market, and technical dependencies**.




---
---



Gladly — here is a **practical questionnaire for TEConcept**, formulated **from NEURA’s customer perspective**, so you can clearly clarify open commercial, technical, and project-related points. The questions are derived directly from the offer, especially from the items **Manufacturing License**, **Basic Package with Source Code**, **MAX14819/MAX14819A drivers**, **STM32F1xx porting**, **1 year of maintenance updates**, **T&M services**, as well as the note that a **driver for volatile memory is not included**.

## For the next meeting with TEConcept

### 1. Commercial Questions

These questions help you better secure pricing, license scope, and potential follow-up costs. While the offer clearly lists the four main items and the total price, some services are defined as additional effort or T&M, and the offer is also described as non-binding.

1. **Can you confirm the full scope of the offer in writing and clearly define which services are included in the fixed price and which are billed on a T&M basis?**

2. **Is the offered price of EUR 27,200 net fully reliable for our project, or do you expect additional implementation efforts based on your experience?**

3. **How do you define “1 Product Category” in the Manufacturing License — and how would the price change if we launch multiple product variants or derivatives?**

4. **Does the Manufacturing License remain valid indefinitely, or are there recurring license fees, renewal costs, or product-related extension fees later on?**

5. **Since the offer is described as non-binding: Can you provide us with a binding commercial final version with a fixed price and validity period for ordering?**

---

### 2. Technical Questions

From a technical perspective, the dependency on **MAX14819/MAX14819A** and the porting to **STM32F1xx** are particularly relevant. It is also explicitly stated that the **driver for volatile memory is not included** — this should be clarified before making a decision.

6. **Which functions are specifically included in the IO-Link Master Stack Basic Package, and which are explicitly not part of it?**

7. **Is the provided porting to STM32F1xx production-ready, or are additional adaptations, integration tests, or performance optimizations required on our side?**

8. **What are the concrete implications of the missing driver for volatile memory — what functionality will we lack in practical use?**

9. **Can you offer the missing volatile memory driver as an option, and if so, what would be the additional effort and an approximate price indication?**

10. **Is the MAX14819/MAX14819A driver strictly tied to these PHYs, or can the same stack later be migrated to other PHYs or hardware platforms with reasonable effort?**

11. **What are the real-time, memory, and CPU requirements of the solution on STM32F1xx, and are there known limitations in terms of performance or scalability?**

---

### 3. Questions on Source Code, Ownership, and Maintenance

Since the offer includes source code licensing and one year of maintenance updates, you should clarify the extent of your rights and what happens after the first year.

12. **What exact rights do we receive with the source code license: are we allowed to modify, extend, debug, and reuse the code internally for future product versions?**

13. **Does the source code license apply to only one development environment or to multiple developers within our team?**

14. **What exactly is included in “1 year of maintenance updates free of charge” — only bug fixes, or also minor releases, security updates, and technical support inquiries?**

15. **What are the maintenance and update conditions after the first year — both in terms of pricing and scope?**

16. **Will we receive access to release notes, change logs, and structured documentation of updates?**

---

### 4. Support and Project Questions

Especially if you plan to integrate the stack into a real product, it’s important to understand how TEConcept supports you and how much effort remains on your side. The offer indicates that additional services may be billed based on effort.

17. **What does your typical support during integration and commissioning look like — reactive on demand or via defined support packages?**

18. **Based on your experience, what level of implementation effort should we expect on the customer side for a project with our setup?**

19. **Can you provide a rough project roadmap: delivery, integration, initial commissioning, testing, and production release?**

20. **What typical risks or challenges do you see when integrating the stack on STM32F1xx with MAX14819/MAX14819A?**

21. **Do you offer support for compliance, test strategy, or series production approval if required?**

---

### 5. Strong Closing Question for a Meeting

If you want to ask **just one key closing question**, this one is recommended because it combines technical, cost, and risk aspects:

**“If we place this order today: What additional technical or commercial efforts do you realistically expect might still arise before we have a production-ready solution?”**

---

## My Recommendation

For a customer discussion, I would not send all questions unfiltered but instead bundle them into a **short priority list**:
first **license scope**, then **missing components such as volatile memory**, followed by **maintenance/support after year one**, and finally **project effort until production readiness**.

This way, you will get clarity as quickly as possible on whether the offer is truly complete and reliable.

---

If you’d like, I can turn this into a **professional email to TEConcept in English or German**.
