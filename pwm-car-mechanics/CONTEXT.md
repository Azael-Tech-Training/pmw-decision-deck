# PWM Learning Site

A static learning site teaching PWM in automotive electronics: hand-written lessons with interactive widgets plus a Slidev presentation deck, published as a website.

## Language

### Rendering model

**Static output**:
All HTML is fully assembled before deploy; nothing renders per-request or client-side-rewrites the page. Every visitor receives identical files from the CDN. This is the PWM-002 decision term.
_Avoid_: calling this "SSG"

**SSG**:
Reserved term. Only correct once a tool generates HTML at build time from templates or content (a possible PWM-005 outcome). Until then the project has static output, not SSG.
_Avoid_: using SSG to describe plain bundled hand-written HTML

### Artifacts

**Website**:
The multi-page static learning site: landing page, lessons index, and lesson pages. The artifact PWM-002's tooling decision and PWM-005's architecture decision govern.
_Avoid_: "the site" when referring to the deck or its deploy configs

**Deck**:
Any Slidev presentation belonging to this project. Two exist: the Teaching Deck and the Decision Deck.
_Avoid_: using "deck" unqualified when which one matters; treating either deck's deploy configs as website deploy configs

**Teaching Deck**:
The interactive Spanish presentation teaching PWM, built in `pwm-presentation/` in this repository. Whether it is embedded in or linked from the Website is an open PWM-005 question.
_Avoid_: calling it "the deck" when the Decision Deck is in play

**Decision Deck**:
The Spanish presentation explaining and defending the PWM-002 tooling decision, developed in `pwm-decision-deck/` in this repository and published to the standalone public repo `pwm-decision-deck`. Out of PWM-005's scope.
_Avoid_: confusing it with the Teaching Deck; treating its public repo as part of the Website
