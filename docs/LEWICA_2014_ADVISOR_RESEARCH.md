# Lewica 2014 Advisor Research And Implementation Notes

This document defines the first playable advisor layer for the SLD-only 2014-2015 skeleton. The design goal is to copy the original Social Democracy advisor logic at the systems level: named political figures sit above the random card deck, each has multiple special actions, all actions share a six-month `advisor_action_timer`, and the roster itself is limited and politically meaningful.

The content goal is narrower: only code advisors who make sense for the Polish institutional left in the January 2014 to October 2015 slice. Later Lewica figures, Razem government-era figures, Wiosna-era consolidation, and post-2023 ministers are intentionally excluded until the early skeleton is playable end to end.

## Historical Baseline

The 2014 player identity is SLD under Leszek Miller. The party is represented in the Sejm but is no longer the hegemonic force of the Polish left. It has an old organizational apparatus, a thinner younger-modernizer layer, the nearby liberal-left debris of Twój Ruch / Europa Plus, and small socialist partner circles such as PPS/UP. This is the only playable identity in the current skeleton.

Key electoral constraints:

- PKW's 2014 European Parliament results give SLD-UP 667,319 votes, 9.44%, and 5 seats. This is a survivable but not breakthrough position.
- PKW's 2015 Sejm results give Zjednoczona Lewica 1,147,102 votes, 7.55%, and 0 seats because it ran as a coalition under the 8% threshold. Razem won 550,349 votes, 3.62%, also 0 seats.
- These two facts define the skeleton's strategic problem: SLD has just enough inherited structure to matter, but a coalition mistake or faction split can erase the left from parliament.

## System Model

Advisor state is stored as boolean roster flags:

- `miller_advisor`
- `gawkowski_advisor`
- `senyszyn_advisor`
- `czarzasty_advisor`
- `trela_advisor`
- `jonski_advisor`
- `biedron_advisor`
- `wanda_nowicka_advisor`
- `barbara_nowacka_advisor`
- `ikonowicz_advisor`
- `kwasniewski_advisor`
- `belka_advisor`
- `cimoszewicz_advisor`

The starting roster is intentionally conservative:

- Leszek Miller
- Krzysztof Gawkowski
- Joanna Senyszyn

Roster size is capped by `max_advisors = 3`. Roster changes use `advisor_roster_timer = 6` and spend the monthly action. Advisor actions use the original-style shared `advisor_action_timer = 6`.

## Faction Coupling

Every advisor is mapped to one or more faction pressures:

- SLD apparatus: Miller, Czarzasty, Kwasniewski, partly Gawkowski.
- SLD modernizers: Gawkowski, Trela, Jonski, Senyszyn.
- Gawkowski secretariat network: Gawkowski's younger organizational layer, sensitive to both Miller centralization and Czarzasty succession maneuvering.
- Twój Ruch / liberal-left residue: Biedron, Wanda Nowicka, Barbara Nowacka, Kwasniewski.
- PPS/UP and class-left edge: Ikonowicz, partly Senyszyn and Nowacka.
- Elder technocratic / state credibility: Belka, Cimoszewicz, Kwasniewski.

The implementation uses 0-100 `*_tension` as the active faction-crisis scale. Existing 0-10 `*_dissent` values remain as a compatibility layer for older cards. If a faction's tension passes 100, `post_event` routes to `lewica_faction_crisis.faction_split` and removes that faction's strength from the active left.

## Advisor Roster

### Leszek Miller

Faction: SLD apparatus / old governing left.

Reasoning: Miller is the party chair in 2014 and the player cannot understand SLD's survival logic without him. He represents old networks, prime-ministerial credibility, hierarchical control, and the danger of overcentralization.

Implemented actions:

- `Enforce Apparatus Discipline`: lowers apparatus tension, raises modernizer/TR tension, increases Miller authority.
- `Emergency Fundraising`: adds resources like the original Wels emergency fundraising pattern; repeated use raises broad faction tension.
- `Responsible Governing Left`: improves PO/PSL channels and older-voter credibility while softening ideological distinction.

### Krzysztof Gawkowski

Faction: organizer-modernizer bridge / secretariat network.

Reasoning: Gawkowski was SLD secretary-general and a younger face of the party machine. In game terms he is a clean bridge between the apparatus card economy and later modernization mechanics.

Implemented actions:

- `Secretary's Map`: regional rebuild, lower apparatus/modernizer tension.
- `Procedural Modernization`: media reach and internal democracy; apparatus tension rises.
- `Draft A Unity Calendar`: improves relations with non-SLD left actors while raising apparatus concern.

### Joanna Senyszyn

Faction: secular SLD veteran / modernizer pressure.

Reasoning: Senyszyn gives the 2014 SLD a visible anticlerical identity, especially after her 2009-2014 European Parliament term. She is useful for secular voters but costly with cautious apparatus and Catholic-adjacent voters.

Implemented actions:

- `Anticlerical Speech`: increases secularization, unaffiliated and LGBTQ+ support, lowers Vatican/Episcopate alignment, raises apparatus tension.
- `Professor On Television`: raises media reach and urban attention.
- `Rebuke The Leadership From The Left`: releases reformist pressure but increases anti-Miller pressure.

### Wlodzimierz Czarzasty

Faction: apparatus operator / post-Miller positioning.

Reasoning: Czarzasty becomes SLD chair in 2016, immediately after this skeleton's 2015 collapse. In 2014-2015 he should exist as an optional advisor who can stabilize the machine and prepare succession, not as the default leader.

Implemented actions:

- `Media Backchannel`: media reach and apparatus strength.
- `Hardball With Regional Chairs`: strong apparatus discipline, higher reformer tension.
- `Prepare The Post-Miller Option`: raises Czarzasty positioning and anti-Miller pressure.

### Tomasz Trela

Faction: regional/mid-generation SLD.

Reasoning: Trela becomes deputy mayor of Lodz after the 2014 local elections and is a useful model for local-government survival politics. He is not a national savior; he is a regional capacity advisor.

Implemented actions:

- `Lodz Machine`: regional rebuild and urban working-class support.
- `Mid-Generation Bridge`: lowers apparatus and modernizer tension.
- `Municipal Services Offer`: small-town/western Poland and local services credibility.

### Dariusz Jonski

Faction: SLD spokesman / media combat.

Reasoning: Jonski was a visible SLD spokesman in 2014. He gives the game a fast-cycle media and scandal-pressure tool, useful during the PO tape-affair environment.

Implemented actions:

- `Daily Attack Line`: media reach and anti-incumbent attention.
- `Scandal Press Conference`: stronger PO attack, damages KO relation.
- `Local Candidate Protection`: lowers regional revolt pressure.

### Robert Biedron

Faction: liberal-left / LGBTQ+ / local renewal.

Reasoning: Biedron's 2014 Slupsk mayoral win is one of the few left-progressive successes in this period. He should not be an SLD insider, but as an advisor he lets the player attempt to graft his civic-progressive style onto the SLD campaign.

Implemented actions:

- `Slupsk Campaign Model`: urban, unaffiliated, under-30 and media gains; apparatus tension rises.
- `LGBTQ+ Mobilization`: strong identity gains and clerical backlash.
- `Civic Localism`: city reform image and mild PO compatibility.

### Wanda Nowicka

Faction: feminist / Twój Ruch parliamentary bridge.

Reasoning: Nowicka was deputy speaker of the Sejm in the 2011-2015 term and an important women's-rights figure. She connects SLD to feminist politics and the Palikot-era liberal-left orbit.

Implemented actions:

- `Feminist Parliamentary Bridge`: women under 50 and civil society.
- `Reproductive Rights Frame`: secular/feminist framing with clerical backlash.
- `Liberal-Left Channel`: lowers TR residue tension.

### Barbara Nowacka

Faction: 2015 United Left public face / feminist progressive.

Reasoning: Nowacka becomes the public leader of Zjednoczona Lewica in the 2015 campaign. She is gated to 2015 recruitment because she is a coalition-era solution, not the default January 2014 SLD structure.

Implemented actions:

- `United Left Public Face`: 2015-only; brand and partner cohesion, apparatus tension.
- `Young Progressive Media`: under-30 and women-under-50 attention.
- `List Parity Bargain`: lowers partner tension but raises apparatus tension.

### Piotr Ikonowicz

Faction: socialist / tenant-defense / class-left edge.

Reasoning: Ikonowicz is outside SLD but historically connected to PPS and social-justice activism. He gives the player a way to build a class-left identity that is not simply nostalgia for SLD government.

Implemented actions:

- `Tenant Intervention`: urban working-class gains and PPS/UP tension relief.
- `Social Justice Office`: costs resources, raises civil-society and union capacity.
- `Working-Class Authenticity Pitch`: economic-left clarity; mild urban-professional cost.

### Aleksander Kwasniewski

Faction: elder statesman / EU and presidential legitimacy.

Reasoning: Kwasniewski is the most valuable left-associated legitimacy asset, but he is not a normal faction organizer. His strongest endorsement is one-time by design.

Implemented actions:

- `Senior Statesman Endorsement`: one-time major brand boost.
- `EU Accession Legacy`: European credibility.
- `Liberal-Left Alliance Signal`: helps bridge SLD and the post-Palikot orbit.

### Marek Belka

Faction: technocratic elder / macroeconomic credibility.

Reasoning: As NBP president in 2014 and former SLD-linked prime minister/finance minister, Belka represents competent state management. He helps with voters who fear fiscal chaos but irritates the socialist edge.

Implemented actions:

- `Macroeconomic Authority`: urban-professional and governing credibility.
- `Budget Credibility`: lower finance pressure and older-voter trust.
- `Centrist Reassurance`: PO-compatible credibility with an economic-left cost.

### Wlodzimierz Cimoszewicz

Faction: independent elder-left / foreign-policy credibility.

Reasoning: Cimoszewicz is an elder left figure with prime-ministerial, justice, foreign-ministry and Senate credentials. In 2015 he can validate a fresh public face without simply restoring Millerism.

Implemented actions:

- `Diplomatic Gravitas`: EU/US/Ukraine credibility.
- `Endorse A New Face`: one-time 2015 unity boost, weakens Miller authority.
- `Anti-PiS Constitutional Frame`: improves KO relation and institutional credibility.

## Design Exclusions For This Skeleton

The following figures are intentionally not coded yet:

- Adrian Zandberg and Razem leaders: Razem appears as an external relationship in 2015, not as playable/advisor content yet.
- Robert Biedron's later Wiosna network: Wiosna is a 2018 mechanic, not a 2014-2015 mechanic.
- Agnieszka Dziemianowicz-Bak, Magdalena Biejat and post-2019 Lewica figures: not period-appropriate for this slice.
- 2023-2027 ministers: outside the first playable skeleton.

## Source Notes

- PKW 2014 European Parliament results: [SLD-UP committee page](https://pe2014.pkw.gov.pl/pl/wyniki/komitety/view/38.html) and [PKW 2014 index](https://pe2014.pkw.gov.pl/pl/index.html).
- PKW 2015 Sejm results: [national Sejm results](https://parlament2015.pkw.gov.pl/349_Wyniki_Sejm/0/1.html).
- Krzysztof Gawkowski as SLD secretary-general: [Polityka profile](https://www.polityka.pl/tygodnikpolityka/kraj/1526697%2C1%2Cnajmlodszy-sekretarz.read) and [Gov.pl biography](https://www.gov.pl/web/premier/krzysztof-gawkowski).
- Czarzasty later SLD leadership and KRRiT background: [PAP on 2016 SLD leadership](https://www.pap.pl/aktualnosci/news%2C460309%2Cwlodzimierz-czarzasty-nowym-szefem-sld.html) and [Sejm biography page](https://www.sejm.gov.pl/sejm10.nsf/page.xsp/biografia).
- Biedron's 2014 Slupsk win: [DW report](https://www.dw.com/en/biedron-wins-in-slupsk-becomes-first-gay-poland-mayor/a-18104097).
- Barbara Nowacka as United Left leader: [PAP English report](https://www.pap.pl/en/news/news%2C412876%2Cunited-left-presents-leader.html).
- Joanna Senyszyn's 2009-2014 EP term: [European Parliament member page](https://www.europarl.europa.eu/meps/pl/96789).
- Wanda Nowicka's parliamentary background: [Sejm X biography noting VII and IX terms](https://www.sejm.gov.pl/sejm10.nsf/posel.xsp?id=266).
- Leszek Miller's 2014 left-unity line: [PAP Samorzadowy](https://samorzad.pap.pl/kategoria/wybory/sld-z-lewej-razem-mamy-szanse-pokonac-prawice-twierdzi-leszek-miller).
- Cimoszewicz endorsement of Nowacka: [PAP English report](https://www.pap.pl/en/news/news%2C412876%2Cunited-left-presents-leader.html).
