# HazırOl — Household Disaster Preparedness Assistant

**Live demo:** https://aybkc.github.io/hazirol/en/ (English) · https://aybkc.github.io/hazirol/ (Türkçe)

HazırOl ("Be Ready") is a generative-AI assistant that turns a short description of a household into a **personalised disaster plan** and a **quantity-calculated emergency bag list** for the natural hazards Türkiye actually faces: earthquake, flood, wildfire, landslide and extreme heat.

It also asks a second question: **if households had followed this kind of guidance, how much of the loss in recent Turkish disasters could have been prevented?** The answer is shown as a transparent, adjustable range rather than a single headline number.

> Built for the Samsung Innovation Campus *Generative AI for the SDGs* hackathon, October 2026.
>
> **Team — Group 13:** Fatma Çevik · Ayberk Koç · Hüseyin Aksoy · Adil Yer · Mehmet Tarık Özcan

## SDG link

| Goal | Target | How HazırOl contributes |
|---|---|---|
| **SDG 11** Sustainable cities | **11.5** Reduce deaths and people affected by disasters, with a focus on people in vulnerable situations | Household-specific actions; mandatory items for elderly, babies, disabled people, people on medication and pets |
| **SDG 13** Climate action | **13.1** Strengthen resilience and adaptive capacity to climate-related hazards | Flood, wildfire and heat guidance tied to current events (14–15 Oct 2026 İstanbul–Kocaeli flood, 2025 wildfire season) |

## Why now

- **14–15 October 2026, İstanbul–Kocaeli flood:** 6 deaths, 3,055 people rescued. Most deaths involved entering flood water, electrocution in a flooded park and being swept away in the Ayazağa stream or in a car.
- **23 April 2025, İstanbul (Silivri offshore) M6.2:** no deaths, but **151 injuries, all from people jumping out of buildings in panic** (İstanbul Governorate).
- **2025 wildfire season:** 3,224 fires and 81,473 ha burned; about 90% of fires with a known cause were human-caused. 10 responders died at Seyitgazi.
- **6 February 2023 Kahramanmaraş earthquakes:** 53,537 deaths and 107,213 injured.

Many of these losses are **behavioural** — what people did in the first minutes. That is the part a preparedness assistant can influence.

## How it works

```
Household form ──► 1. Rule engine (deterministic)
                     • bag quantities (water = people × 4 L × 3 days …)
                     • mandatory safety priorities (DASK, furniture anchoring,
                       flood exit route, carrying plan for vulnerable members)
                 ──► 2. Generative AI plan (LLM, JSON)
                     • role + context + hidden step-by-step reasoning
                     • schema-first output + one-shot example
                 ──► 3. Two-layer safety check
                     • second LLM call audits the plan and flags issues
                     • regex sanitiser removes dangerous advice and every
                       phone number except 112
                 ──► Merged plan: priorities · hazard phases · per-person
                     advice · family communication plan · emergency bag
```

If no model is available, steps 2–3a are skipped and the user still gets the safe rule-based plan, clearly labelled.

### Models

- **GitHub Pages version (this repo):** Google **Gemini 2.5 Flash** through the REST API. Each user pastes their own free API key from [Google AI Studio](https://aistudio.google.com/apikey). The key stays in the browser tab; it is never stored or sent anywhere except Google's API.
- **claude.ai artifact version:** the same page calls **Claude** through the artifact `sample` capability; no key is needed.

### Prompt design

See [`prompts.md`](prompts.md) for the full prompts and the reasoning behind each technique.

| Technique | Where |
|---|---|
| Role & context framing | Plan prompt: "a calm disaster-preparedness advisor who knows AFAD guidance" |
| Chain-of-thought (hidden) | Plan prompt: 4 numbered reasoning steps, "do not write these steps" |
| Few-shot (one-shot) | Plan prompt: a complete Hatay household input → output example |
| Schema-first structured output | Plan and validator prompts return strict JSON |
| Negative constraints | No building-safety verdicts, no numbers except 112, no outdated advice |
| Zero-shot critique / feedback loop | Validator prompt: a second call audits the first output with a 5-point checklist |
| Deterministic guardrail | `sanitize()` and the `FORBIDDEN` regex list run after both model calls |

## "How much could have been prevented?"

For each event, the loss is split by **cause**. Only causes a household can change by behaviour are counted:

```
preventable = behavioural loss × adoption rate × behaviour-change effect
```

Both factors are sliders. With the defaults (25% adoption, mid-point effect). Results are expected values; a fraction of a life means, for example, 0.7 = about 7 lives across 10 similar events, so the app shows lives as a whole number with the expected value beside it:

| Event | Behavioural loss counted | Estimated preventable |
|---|---|---|
| İstanbul–Kocaeli flood, Oct 2026 | 6 deaths (all behavioural) | **~1 life** (expected value 0.7, range 0.5–0.9: about 7 lives per 10 similar floods) |
| Silivri M6.2, Apr 2025 | 151 injuries from jumping | **~19 injuries** (11–26) |
| Sındırgı M6.1, Aug & Oct 2025 | none (structural) | **0** |
| 2025 wildfires | human-caused share of 81,473 ha | **~917 ha** (367–1,467) |
| Kahramanmaraş, Feb 2023 | ~50% of injuries non-structural | **~2,680 injuries** (1,340–4,021) |

Building-collapse deaths are **deliberately excluded**: an app cannot make a building stronger. It can only point people to a licensed performance assessment.

Each event card lists its sources and caveats in the app.

## Ethics

| Risk | Mitigation |
|---|---|
| Hallucinated life-threatening advice | Rule-based safety list always included; forbidden-advice regex; validator call |
| Invented phone numbers or sources | Only 112 is allowed; every other number is removed |
| False sense of safety ("your building is safe") | The model is forbidden from judging buildings; pre-2000, unknown or cracked buildings always trigger a licensed-assessment recommendation |
| Vulnerable groups overlooked | Mandatory items for elderly, babies, disabled people, people on medication and pets; no dosing advice ("ask your doctor") |
| Privacy | No name, address or phone is asked; district is optional; nothing is stored on a server |
| Overclaiming impact | Only behavioural losses counted, results shown as ranges, all assumptions visible and adjustable |

HazırOl is **not** an official warning system. In an emergency call **112**. Follow AFAD, the Turkish State Meteorological Service and governorate announcements.

## Run locally

It is a single static file with no build step.

```bash
git clone https://github.com/aybkc/hazirol.git
cd hazirol
python3 -m http.server 8000
# open http://localhost:8000
```

Click **"Örnek hane: Ayazağa"** to load a sample household, optionally paste a Gemini API key, then **"Planımı hazırla"**.

## Repository

```
index.html   the whole app (HTML + CSS + JS, no dependencies besides Google Fonts)
en/index.html  the same app in English (English UI, prompts and safety patterns)
prompts.md   full prompt texts and design notes
README.md    this file
```

## Sources

İstanbul Governorate and AFAD statements; BirGün, ANKA, Karar, Memurlar.net (Oct 2026 flood); Yetkin Report and İTÜ Mines (2025 earthquakes); TBMM reports and Bloomberg HT (2025 wildfires); Presidency Strategy and Budget Office (SBB) 2023 earthquake report; TEPAV; ReliefWeb; US National Weather Service (flood deaths in vehicles); Global Commission on Adaptation (2019). Links are listed under each event in the app.

## License

MIT
