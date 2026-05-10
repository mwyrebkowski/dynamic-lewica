# Lewica 2014-2027 Implementation Blueprint

Status: planning document only. Do not start live mechanics implementation until this blueprint is reviewed.

Fact-model lock date: 2026-05-07.

This document adapts the proposed "Lewica 2014-2027: A Dynamic Political Simulation" into a concrete implementation plan for the `dynamic-lewica` fork. It is intentionally detailed and implementation-facing. It tells a future implementer what to change, what not to change, how to translate DSD mechanics, how to name state variables, how to stage the work, and what needs to be tested before gameplay can be considered valid.

The current codebase is still the DSD-derived Weimar game. This document is the bridge from design concept to code.

## Non-Negotiable Design Boundary

The project must preserve the DSD architecture:

- Card-driven turns.
- Centralized monthly resolver.
- Global `Q` state.
- Internal faction strength and dissent.
- Advisor pinned actions with shared cooldowns.
- Coalition partner dissent.
- Demographic election simulation.
- Scheduled and conditional events.
- Multiple endings.
- Qualitative qdisplay feedback over raw-number full transparency.

The project must adapt the political domain:

- Germany 1928-1934 becomes Poland 2014-2027.
- SPD becomes the institutional Polish left: SLD, United Left, Wiosna, Razem alliance, Nowa Lewica, and possible successor blocs.
- Paramilitary street force becomes civil-society movement force.
- Civil-war/coup risk becomes constitutional-crisis risk.
- Prussia as strategic state fortress becomes a Polish institutional/regional subgame.
- Weimar party relations become modern rival-party faction modeling.
- Foreign-policy pressure becomes EU/NATO/US/Russia/Ukraine/Vatican pressure.

The game should not become a generic election simulator. It should remain a pressure-cooker political survival game where every solution damages another subsystem.

## Historical And Speculative Boundary

Everything through 2026-05-07 should be treated as historical if it is documented. Everything after 2026-05-07 should be treated as playable counterfactual unless it is already institutionally scheduled.

Implementation rule:

- If an event is historical through 2026-05-07, use exact date and exact baseline outcome.
- If an event is scheduled but not yet resolved by 2026-05-07, create it as a scheduled future event with branching outcomes.
- If an event is plausible but uncertain, mark it as probabilistic and write narrative text that frames it as speculation.

Examples:

- 2015 left wipeout is historical and should be exact.
- 2025 presidential result is historical and should be exact.
- 2027 Sejm election is future and must be simulated.
- Kaczynski death/incapacitation is plausible, not scripted certainty.
- Tusk departure is plausible, not scripted certainty.
- Konfederacja split is plausible, not scripted certainty.
- Ukraine ceasefire/settlement is plausible, not scripted certainty.

## Source Anchors For Hard Facts

Use these as anchor references during implementation. They are not a full bibliography, but they lock the most important numeric beats.

- 2015 Sejm: PKW official page shows Zjednoczona Lewica 7.55 percent and Razem 3.62 percent, both outside Sejm.  
  https://parlament2015.pkw.gov.pl/349_Wyniki_Sejm.html
- 2019 Sejm: Lewica/SLD list returned with 12.56 percent and 49 seats.  
  https://www.wprost.pl/wybory-parlamentarne-2019/10261045/wybory-parlamentarne-2019-lista-poslow-sld-razem-i-wiosny.html
- 2023 Sejm: Nowa Lewica 8.61 percent and 26 seats; KO, TD, Lewica, and Konfederacja seat totals from PKW-reported results.  
  https://www.rp.pl/wybory/art39279141-wybory-2023-znamy-ostateczny-podzial-mandatow-w-sejmie
- 2024 Razem split: PAP reports Magdalena Biejat, Anna Gorska, Joanna Wicha, Dorota Olko, and Daria Gosek-Popiolek leaving Razem while remaining in the Lewica club.  
  https://www.pap.pl/aktualnosci/rozlam-w-partii-razem-piec-parlamentarzystek-odchodzi
- 2025 presidential first round: Biejat 4.23 percent; Zandberg 4.86 percent; Trzaskowski and Nawrocki advance.  
  https://ewybory.eu/wybory-prezydenckie-2025-polska/1-tura/
- 2025 presidential runoff: PAP/PKW-reported result Nawrocki 50.89 percent, Trzaskowski 49.11 percent.  
  https://biznes.pap.pl/wiadomosci/gospodarka/karol-nawrocki-wygrywa-wybory-prezydenckie-z-wynikiem-5089-proc-rafal-0
- 2025 Sejm Marshal: Sejm official communication reports Czarzasty elected Marshal of the Sejm on 2025-11-18, 236 for, 209 against, 2 abstentions.  
  https://www.sejm.gov.pl/sejm10.nsf/komunikat.xsp?documentId=E7F57A11C3CEAA02C1258D46003D2E77&symbol=M_WYDARZENIA_KOMUNIKAT

## First Implementation Principle

Do not start by editing every card.

Start by implementing the new state model, calendar, action economy, and a thin playable skeleton. Then add period events in chronological layers.

Correct order:

1. Mechanical shell.
2. State initialization.
3. Election model.
4. Party factions.
5. Movement/constitutional-crisis model.
6. Minimal card decks.
7. Major historical events.
8. Government and ministry content.
9. Rival-party faction modeling.
10. Endings.
11. Polish text polish and media.

Wrong order:

1. Rewrite all narrative first.
2. Keep old Weimar mechanics hidden underneath.
3. Patch around broken state after the fact.

## Repo-Level Strategy

The current repo has:

- `source/scenes/root.scene.dry`
- `source/scenes/post_event.scene.dry`
- `source/scenes/election_simulation.scene.dry`
- `source/scenes/election_algorithm.scene.dry`
- `source/scenes/main.scene.dry`
- many `party_affairs`, `government_affairs`, `advisors`, `events`
- `source/qdisplays`

Implementation should keep this layout, but replace content in waves.

Recommended new documentation:

- `docs/MECHANICS_GAMEDESIGN.md`: existing DSD mechanics analysis.
- `docs/LEWICA_IMPLEMENTATION_BLUEPRINT.md`: this document.
- `docs/LEWICA_CONTENT_MATRIX.md`: later, a spreadsheet-like content tracker for every event/card.
- `docs/LEWICA_FACT_NOTES.md`: later, source-backed historical notes.

Recommended new source organization:

- `source/scenes/lewica_init.scene.dry`: optional helper called by root for large initialization blocks.
- `source/scenes/lewica_post_event.scene.dry`: optional helper called by `post_event` if Dendry call structure permits.
- `source/scenes/events_2014/*.scene.dry`
- `source/scenes/events_2015/*.scene.dry`
- `source/scenes/events_2016_2018/*.scene.dry`
- `source/scenes/events_2019_2021/*.scene.dry`
- `source/scenes/events_2022_2025/*.scene.dry`
- `source/scenes/events_2026_2027/*.scene.dry`

If Dendry imports are awkward, keep existing `source/scenes/events` but prefix files:

- `2014_ep_election.scene.dry`
- `2015_presidential_election.scene.dry`
- `2015_sejm_catastrophe.scene.dry`
- `2016_black_protest.scene.dry`
- `2024_razem_split.scene.dry`

## Naming Convention

Use lowercase snake case for all `Q` variables.

Use stable party ids:

- `left` for the currently playable left bloc.
- `sld`
- `wiosna`
- `razem`
- `ko`
- `po`
- `pis`
- `td`
- `pl2050`
- `psl`
- `konf`
- `korwin`
- `kukiz`
- `other`

Use display variables for names:

- `Q.left_name`
- `Q.ko_name`
- `Q.td_name`
- `Q.konf_name`

Use flat names for faction data:

- `Q.sld_apparatus_strength`
- `Q.sld_apparatus_dissent`
- `Q.wiosna_founders_strength`
- `Q.razem_strength`
- `Q.razem_dissent`

Use flat names for rival-party factions:

- `Q.ko_tusk_strength`
- `Q.ko_trzaskowski_strength`
- `Q.pis_kaczynski_strength`
- `Q.pis_morawiecki_strength`
- `Q.konf_mentzen_strength`
- `Q.konf_braun_strength`

Use movement prefix plus two variables:

- `Q.strajk_kobiet_strength`
- `Q.strajk_kobiet_mobilization`
- `Q.kod_strength`
- `Q.kod_mobilization`
- `Q.mw_onr_strength`
- `Q.mw_onr_mobilization`
- `Q.climate_strike_strength`
- `Q.climate_strike_mobilization`
- `Q.ip_union_strength`
- `Q.ip_union_mobilization`

Avoid ambiguous variables like `left_left` or `party_left`. The playable party is `left`; ideological left faction can be `radical_left`.

## Calendar And Tempo

Default start:

- `Q.year = 2014`
- `Q.month = 1`
- `Q.time = 0`
- `Q.end_year = 2027`
- `Q.end_month = 12`
- total months = 168

Action economy:

- Hard: hand size 3, actions per month 1.
- Normal: hand size 3, actions per month 2.
- Easy: hand size 4, actions per month 3.

Implementation:

In `root.scene.dry`, set:

```js
Q.hand_size = 3;
Q.actions_per_month = 2;
Q.advisor_action_timer = 0;
```

Difficulty setup:

```js
if (Q.difficulty == -1) {
    Q.hand_size = 4;
    Q.actions_per_month = 3;
}
if (Q.difficulty == 0) {
    Q.hand_size = 3;
    Q.actions_per_month = 2;
}
if (Q.difficulty == 1) {
    Q.hand_size = 3;
    Q.actions_per_month = 1;
}
```

In `post_event.scene.dry`, replace hardcoded monthly threshold:

```js
if (Q.month_actions >= Q.actions_per_month) {
    advanceMonth();
}
```

Do not keep Rubicon four-actions-per-month as-is. There is no direct Rubicon mode in the Lewica game. If a constitutional crisis mode is later added, it should be a separate `Q.crisis_mode` with its own action threshold.

## Core State Master List

### Time And Mode

- `year`
- `month`
- `time`
- `month_actions`
- `actions_per_month`
- `hand_size`
- `started`
- `historical_mode`
- `advanced_start`
- `start_scenario`
- `crisis_mode`
- `final_election_seen`
- `game_over`

### Player Identity

- `left_name`
- `party_identity`
- `party_leader`
- `left_in_sejm`
- `left_in_government`
- `left_in_opposition`
- `left_bloc_type`
- `left_brand_strength`
- `left_threshold_safety`

Recommended values:

- `party_identity = "SLD"` in January 2014.
- `left_name = "SLD"` in January 2014.
- `party_leader = "Miller"` in January 2014.
- `left_bloc_type = "sld"` in January 2014.

### Player Resources

- `party_resources` from 0 to 10.
- `budget` from -5 to +10.
- `media_reach` from 0 to 10.
- `civil_society_capital` from 0 to 10.

Initial January 2014:

- `party_resources = 5`
- `budget = 0`
- `media_reach = 4`
- `civil_society_capital = 3`

Notes:

- `budget` matters only in government or when influencing policy through coalition.
- `party_resources` replaces DSD `resources`, but keeping `resources` as alias may reduce initial rewrite cost.
- First implementation can set `Q.resources = Q.party_resources` for compatibility, then migrate cards.

### Government

- `government_party`
- `prime_minister`
- `president`
- `left_in_government`
- `left_confidence_support`
- `coalition_type`
- `coalition_cohesion`
- `ko_partner_dissent`
- `td_partner_dissent`
- `psl_partner_dissent`
- `pl2050_partner_dissent`
- `razem_partner_dissent`

Ministry ownership:

- `premiership_party`
- `deputy_pm_party`
- `foreign_minister_party`
- `interior_minister_party`
- `defense_minister_party`
- `justice_minister_party`
- `treasury_minister_party`
- `finance_minister_party`
- `industry_energy_minister_party`
- `climate_environment_minister_party`
- `digital_minister_party`
- `family_labor_social_policy_minister_party`
- `health_minister_party`
- `education_science_minister_party`
- `culture_minister_party`
- `agriculture_minister_party`
- `infrastructure_minister_party`
- `sport_tourism_minister_party`
- `equality_minister_party`

Initial January 2014:

- `prime_minister = "Tusk"`
- `president = "Komorowski"`
- `government_party = "PO-PSL"`
- `left_in_government = 0`
- ministry parties set to `PO`, `PSL`, or `I` as needed for flavor, but no player access.

### Regime And Institutions

- `constitutional_crisis_risk` 0 to 10.
- `rule_of_law_restoration` 0 to 10.
- `coalition_cohesion` 0 to 10.
- `secularization_tide` 0 to 10.
- `church_institutional_power` 0 to 10.
- `pis_post_kaczynski_fracture` 0 to 10.
- `konfederacja_split_likelihood` 0 to 10.
- `populist_realignment_pressure` 0 to 10.
- `presidential_veto_pressure` 0 to 10.

Initial January 2014:

- `constitutional_crisis_risk = 1`
- `rule_of_law_restoration = 8`
- `coalition_cohesion = 0`
- `secularization_tide = 4`
- `church_institutional_power = 7`
- `pis_post_kaczynski_fracture = 0`
- `konfederacja_split_likelihood = 2`
- `populist_realignment_pressure = 3`
- `presidential_veto_pressure = 0`

### External Pressure

- `eu_institutional_alignment`
- `us_alignment`
- `german_bilateral`
- `french_bilateral`
- `ukrainian_alignment`
- `russian_threat_level`
- `vatican_episcopate_alignment`
- `trump_admin_modifier`
- `eu_funds_unlocked`
- `kpo_funds_status`

Initial January 2014:

- `eu_institutional_alignment = 9`
- `us_alignment = 8`
- `german_bilateral = 8`
- `french_bilateral = 7`
- `ukrainian_alignment = 5`
- `russian_threat_level = 4`
- `vatican_episcopate_alignment = 7`
- `trump_admin_modifier = 0`

### Elections

- `next_sejm_election_year`
- `next_sejm_election_month`
- `next_presidential_election_year`
- `next_presidential_election_month`
- `next_ep_election_year`
- `next_ep_election_month`
- `next_local_election_year`
- `next_local_election_month`
- `in_election_campaign`
- `presidential_candidate`
- `sejm_list_strategy`
- `threshold_type`

Thresholds:

- Party list: 5 percent.
- Coalition list: 8 percent.
- Minority list: not relevant for Lewica player but can be modeled for `other`.

Use `threshold_type`:

- `party`
- `coalition`
- `exempt`

### Telemetry

- `party_support_records`
- `institutional_records`
- `movement_records`
- `economic_records`
- `election_records`

The old `economic_records` can stay, but it should eventually track:

- inflation
- unemployment
- cost_of_living_pressure
- budget

## Demographic Election Model

The user's demographic list is intentionally cross-cutting. Urban professionals, women under 50, under-30 voters, and religiously unaffiliated voters overlap in real life. DSD's election algorithm expects class-like segments, but it can support weighted political signal pools if each segment is internally normalized.

Therefore implement these as voter segments, not literal mutually exclusive census classes.

Use:

```js
Q.voter_segments = [
  "urban_professionals",
  "urban_working_class",
  "rural_east",
  "rural_west",
  "catholic_practicing",
  "catholic_nonpracticing",
  "unaffiliated",
  "under_30",
  "age_30_50",
  "age_50_65",
  "age_65_plus",
  "women_under_50",
  "women_over_50",
  "lgbtq",
  "ukrainian_refugees",
  "belarusian_eastern_minority"
];
```

Each segment has a weight:

- `urban_professionals_weight`
- `rural_east_weight`
- etc.

Weights do not need to sum to 100. They define electoral salience. The election algorithm should normalize final party support.

Parties in the election model should use canonical ids:

```js
Q.parties = ["left", "ko", "pis", "td", "psl", "pl2050", "konf", "razem", "other"];
```

For each segment-party pair:

- `urban_professionals_left`
- `urban_professionals_ko`
- `urban_professionals_pis`
- etc.

Era-specific parties:

- Pre-2015: `left` display name SLD; `ko` display name PO; `td` inactive; `pl2050` inactive; `konf` maps to Korwin/right-libertarian; `razem` inactive until 2015.
- 2015: `razem` active; `left` can be SLD/United Left depending list strategy.
- 2018: Wiosna can be represented as `left_wiosna` internally or as a player-identity modifier. Simpler first pass: Wiosna affects `left` if player chose Wiosna, otherwise appears as an event pressure and vote drain.
- 2019 onward: `left` display name Lewica/Nowa Lewica; Razem may be inside or outside depending route.
- 2024 onward: `razem` becomes external if split occurs.

Implementation warning:

Do not overfit the first build to perfect polling. The election model only needs plausible relative movement at first. Exact historical outcomes can be scripted for anchor elections if the player is in historical mode.

## Baseline Voter Segment Weights

Use approximate political salience weights, not census shares:

- `urban_professionals_weight = 12`
- `urban_working_class_weight = 10`
- `rural_east_weight = 10`
- `rural_west_weight = 9`
- `catholic_practicing_weight = 11`
- `catholic_nonpracticing_weight = 10`
- `unaffiliated_weight = 7`
- `under_30_weight = 8`
- `age_30_50_weight = 11`
- `age_50_65_weight = 10`
- `age_65_plus_weight = 10`
- `women_under_50_weight = 9`
- `women_over_50_weight = 8`
- `lgbtq_weight = 3`
- `ukrainian_refugees_weight = 0` before 2022, then 3
- `belarusian_eastern_minority_weight = 1`

These are starting balance values. Tune after the first playable election pass.

## Baseline January 2014 Affinities

These are rough first-pass values for the election simulation. They should be tuned to reproduce the 2014-2015 left weakness when the player makes historical choices.

Format:

`segment`: left / ko / pis / td / psl / pl2050 / konf / razem / other

Inactive parties get zero until their activation events.

- Urban professionals: 12 / 45 / 20 / 0 / 3 / 0 / 8 / 0 / 12
- Urban working class: 18 / 25 / 33 / 0 / 5 / 0 / 6 / 0 / 13
- Rural east: 5 / 15 / 55 / 0 / 15 / 0 / 4 / 0 / 6
- Rural west: 9 / 30 / 35 / 0 / 15 / 0 / 4 / 0 / 7
- Catholic practicing: 4 / 20 / 55 / 0 / 12 / 0 / 3 / 0 / 6
- Catholic nonpracticing: 14 / 42 / 22 / 0 / 5 / 0 / 6 / 0 / 11
- Unaffiliated: 28 / 38 / 8 / 0 / 2 / 0 / 8 / 0 / 16
- Under 30: 16 / 30 / 22 / 0 / 4 / 0 / 12 / 0 / 16
- Age 30-50: 12 / 36 / 28 / 0 / 6 / 0 / 6 / 0 / 12
- Age 50-65: 13 / 32 / 35 / 0 / 8 / 0 / 3 / 0 / 9
- Age 65 plus: 10 / 28 / 43 / 0 / 9 / 0 / 2 / 0 / 8
- Women under 50: 16 / 38 / 24 / 0 / 5 / 0 / 5 / 0 / 12
- Women over 50: 11 / 34 / 36 / 0 / 8 / 0 / 2 / 0 / 9
- LGBTQ: 35 / 45 / 2 / 0 / 1 / 0 / 2 / 0 / 15
- Ukrainian refugees: inactive weight before 2022
- Belarusian/eastern minority: 18 / 35 / 20 / 0 / 2 / 0 / 5 / 0 / 20

Historical anchors should override or adjust these at election moments if historical mode is enabled.

## Internal Factions By Era

The faction system should be dynamic. Active factions change by era.

Use an `active_left_factions` array:

```js
Q.active_left_factions = ["sld_apparatus", "sld_modernizers", "tr_residue", "pps_up"];
```

### 2014-2017: SLD-Dominant Phase

Factions:

- `sld_apparatus`
- `sld_modernizers`
- `tr_residue`
- `pps_up`

Initial strengths:

- `sld_apparatus_strength = 5`
- `sld_modernizers_strength = 2`
- `tr_residue_strength = 2`
- `pps_up_strength = 1`

Initial dissent:

- `sld_apparatus_dissent = 2`
- `sld_modernizers_dissent = 4`
- `tr_residue_dissent = 5`
- `pps_up_dissent = 3`

Preferences:

- SLD apparatus: coalition survival, list safety, pragmatic KO cooperation, low radicalism.
- SLD modernizers: rebrand, younger candidates, more social progressivism.
- TR residue: anticlerical urban liberalism, high media tactics, weak organization.
- PPS/UP: labor, old socialist identity, trade union ties.

### 2015-2019: Razem External Pole

Razem is not initially a faction inside the player party unless the player chooses Razem start.

Variables:

- `razem_external = 1`
- `razem_relation`
- `razem_competition_pressure`
- `razem_state_funding`
- `razem_parliamentary_presence`

If playing Razem start:

- `party_identity = "Razem"`
- `active_left_factions = ["razem_programmatic", "razem_activist", "razem_pragmatic"]`

### 2018-2021: Wiosna And Lewica Coalition Phase

Add:

- `wiosna_founders`
- `razem_allied`
- `sld_apparatus`
- `pps_wrn_heirs`

On Wiosna founding:

- If player is SLD: Wiosna becomes external pressure and may drain urban-progressive support.
- If player chooses Wiosna route: Wiosna becomes player identity.
- If negotiated alliance succeeds: Wiosna becomes active faction.

### 2021-2024: Nowa Lewica Era

Factions:

- `sld_faction`
- `wiosna_faction`
- `razem_allied`
- `labor_left`

Critical event:

- Nowa Lewica formation on 2021-10-09.
- Dual leadership compromise.
- Leadership friction and frakcje should be active variables.

### 2025-2027: Post-Frakcja Phase

Factions:

- `czarzasty_gawkowski_mainstream`
- `dziemianowicz_bak_reformers`
- `trela_regional`
- `wiosna_residual`
- `razem_external_pole`

Use:

- `razem_external_pole_strength` if Razem is outside but strategically relevant.
- `razem_external_pole_dissent` should represent difficulty of reunification rather than internal party dissent.

## Faction Dissent Rules

Each faction has strength 0-10 and dissent 0-10.

Overall dissent:

```js
Q.left_dissent = weightedAverage(activeFactionDissentByStrength);
```

Effects:

- `left_dissent >= 7`: ultimatum event for the highest-dissent faction.
- `left_dissent >= 9`: split, public resignation, or leadership challenge.
- faction-specific dissent >= 8: faction event can trigger even if overall dissent is lower.

Global efficiency:

```js
Q.party_efficiency = Math.max(0.1, 1 - (Q.left_dissent / 10));
```

Use `party_efficiency` for:

- campaign gains
- organizing gains
- media gains
- relationship repair

Do not use it for:

- direct crisis consequences
- hard election thresholds
- government ministry ownership

## Rival Party Faction Modeling

This is a major adaptation beyond base DSD.

Rival party factions should update monthly in `post_event`, not only during events.

### KO

Variables:

- `ko_tusk_strength`
- `ko_schetyna_residual_strength`
- `ko_trzaskowski_strength`
- `ko_pelczynska_nalecz_strength`
- `ko_regional_barons_strength`
- `ko_nowoczesna_residual_strength`
- `ko_inicjatywa_polska_strength`
- `ko_succession_pressure`

Core mechanics:

- Strong Lewica with stable coalition performance strengthens `ko_trzaskowski_strength`.
- Chaotic Lewica strengthens `ko_schetyna_residual_strength`.
- Public attacks on Tusk strengthen `ko_tusk_strength` defensively and can harden KO against Lewica.
- Tusk departure event checks `ko_succession_pressure` and faction strengths.

### PiS

Variables:

- `pis_kaczynski_strength`
- `pis_morawiecki_strength`
- `pis_security_state_strength`
- `pis_suwerenna_polska_strength`
- `pis_regional_machines_strength`
- `pis_younger_reformers_strength`
- `kaczynski_health`
- `pis_fracture_pressure`

Core mechanics:

- `kaczynski_health` declines slowly after 2025 unless historical mode locks it.
- If Kaczynski health crisis triggers, set `pis_post_kaczynski_fracture` active.
- Corruption exposure weakens court faction, may strengthen Morawiecki if framed as cleanup.
- Strong Konfederacja weakens PiS radical wing vote share.

### Trzecia Droga

Variables:

- `pl2050_holownia_loyalists_strength`
- `pl2050_provincial_activists_strength`
- `pl2050_defection_pressure`
- `psl_kosiniak_strength`
- `psl_old_guard_strength`
- `psl_rural_machine_strength`
- `td_cohesion`

Core mechanics:

- PL2050 declines after 2024 unless actively stabilized by government success.
- Defecting progressives can move to KO or Lewica.
- PSL blocks or softens abortion/equality policy depending dissent and pressure.

### Konfederacja

Variables:

- `konf_mentzen_strength`
- `konf_braun_strength`
- `konf_korwin_old_guard_strength`
- `konf_young_right_strength`
- `konf_split_likelihood`

Core mechanics:

- Braun scandals raise split likelihood.
- Mentzen media success raises Konf vote but may not benefit Braun.
- High anti-Ukrainian sentiment strengthens both Mentzen and Braun.
- Formal split can create separate `konf` and `korona` entities in election model.

## Civil-Society Movement Model

Replace DSD paramilitary force with movement pressure.

Movement variables:

- strength: durable organization, members, networks, funding, legitimacy.
- mobilization: active deployment in protests, campaigns, media cycles.

Movements:

- Strajk Kobiet.
- KOD.
- MW/ONR far-right youth.
- Climate strike.
- Inicjatywa Pracownicza / radical union network.

Effective pressure:

```js
Q.strajk_kobiet_pressure = Q.strajk_kobiet_strength * Q.strajk_kobiet_mobilization / 10;
```

Aggregate blocs:

- `progressive_movement_pressure = strajk_kobiet + climate_strike + ip_union`
- `liberal_democratic_movement_pressure = kod`
- `far_right_movement_pressure = mw_onr`

Monthly effects:

- High progressive pressure raises women-under-50 left affinity, secularization tide, and equality agenda salience.
- High KOD pressure raises KO affinity and rule-of-law agenda salience.
- High far-right pressure raises Konf/PiS affinity in some segments and raises constitutional-crisis risk.
- Very high cross-field mobilization raises constitutional-crisis risk due to protest/repression dynamics.

Movement decay:

- Mobilization decays monthly by 0.2 to 0.5 unless refreshed by events/cards.
- Strength decays slowly only if neglected or publicly discredited.

Design reason:

This preserves DSD's "street force changes politics" loop while matching contemporary Poland. Protest capacity matters, but it does not imply armed conflict.

## Constitutional Crisis Model

Core variable:

- `constitutional_crisis_risk` 0 to 10.

Sources of increase:

- Judicial conflicts.
- Presidential veto standoffs after Nawrocki victory.
- Refusal to swear in officials.
- Constitutional Tribunal disputes.
- Prosecutor-general disputes.
- Mass protest/repression spirals.
- PiS/Konf destabilization campaigns.
- Government attempts to bypass institutions without broad legitimacy.

Sources of decrease:

- Negotiated settlements.
- Rule-of-law restoration.
- EU institutional backing.
- Clear electoral mandate.
- Cross-coalition legal consensus.
- Low movement mobilization after reform success.

Thresholds:

- 4: visible institutional stress, minor crisis events.
- 6: recurring veto/judicial standoff cards.
- 8: major constitutional crisis events.
- 9: catastrophic ending possible.

It should not be a simple "left good, right bad" meter. Player overreach can raise it too.

## Foreign Policy Model

Foreign policy is not background flavor. It changes coalition, media, defense, EU funds, and right-wing mobilization.

Variables:

- `eu_institutional_alignment`
- `us_alignment`
- `german_bilateral`
- `french_bilateral`
- `ukrainian_alignment`
- `russian_threat_level`
- `vatican_episcopate_alignment`
- `trump_admin_modifier`

Rules:

- Ukraine invasion on 2022-02-24 raises Russian threat and Ukrainian alignment.
- Trump second term on 2025-01-20 sets `trump_admin_modifier = 1` and makes US alignment more volatile.
- Nawrocki victory makes US alignment partially routed through presidency.
- EU alignment gates KPO/EU funds and rule-of-law support.
- Vatican/Episcopate alignment interacts with church, education, abortion, civil partnership, and secularization events.

## Economic Model

Keep economy simpler than DSD's Great Depression model, but preserve tradeoffs.

Variables:

- `unemployment`
- `inflation`
- `cost_of_living_pressure`
- `housing_pressure`
- `public_finance_pressure`
- `budget`
- `eu_funds_unlocked`
- `defense_spending_pressure`

Core loops:

- Inflation/cost of living hurts government parties and benefits opposition/populists.
- Housing pressure creates left opportunity among under-30 and urban renters.
- Public finance pressure constrains social spending.
- EU funds improve budget and government competence if rule-of-law restoration succeeds.
- Defense spending after 2022 limits left fiscal room and creates Razem/mainstream tension.

## Ministries And Government Cards

Each government card must be gated by ministry ownership.

Use ministry checks:

- Equality cards require `equality_minister_party == "left"` or strong coalition leverage.
- Labor/family cards require `family_labor_social_policy_minister_party == "left"`.
- Digital cards require `digital_minister_party == "left"`.
- Education/science cards require `education_science_minister_party == "left"` or coalition agreement.
- Justice cards require `justice_minister_party == "left"` or KO cooperation.
- Finance/treasury cards require corresponding ministry or coalition budget agreement.
- Foreign/defense cards usually require KO control but can appear as coalition-position cards if Lewica is in government.

2023 OTL baseline:

- `left_in_government = 1`
- `equality_minister_party = "left"`
- `family_labor_social_policy_minister_party = "left"`
- `digital_minister_party = "left"`
- `education_science_minister_party = "left"`
- deputy PM position via Gawkowski.

Do not overstate Lewica control. Many big institutional reforms are KO/prime-minister/justice-dependent.

## Deck Structure

### Party Affairs

Core categories:

- Congress and leadership.
- Faction discipline.
- Coalition/list negotiations.
- Media strategy.
- Membership recruitment.
- Demographic organizing.
- Civil society linkage.
- Ideological debate.
- Candidate selection.
- Scandal response.
- Campaign conventions.

Card implementation template:

```dry
title: Party Congress
new-page: true
is-card: true
tags: party_affairs
view-if: party_congress_timer <= 0
on-arrival: month_actions += 1; party_congress_timer = 24
```

Effects should hit at least three systems:

- faction strength/dissent
- party resources/media/civil society
- demographic support or future event flag

### Government Affairs

Core categories:

- Coalition management.
- Ministry policy.
- Legislative votes.
- Budget/fiscal policy.
- Rule-of-law restoration.
- Equality/abortion/civil partnership.
- Labor/housing/social policy.
- Foreign/security.
- Education/church/secularization.
- Crisis management.

Card implementation template:

```dry
title: Civil Partnership Bill
new-page: true
is-card: true
tags: govt_affairs, equality
view-if: left_in_government and equality_minister_party = "left" and civil_partnership_bill_timer <= 0
on-arrival: month_actions += 1; civil_partnership_bill_timer = 8
```

### Advisor Actions

Pinned cards with 6-month shared cooldown:

- Czarzasty.
- Gawkowski.
- Dziemianowicz-Bak.
- Biedron.
- Zandberg.
- Biejat.
- Zukowska.
- Belka.
- Kwasniewski.
- Lempart.
- Cimoszewicz.

Implementation:

- Reuse `advisor_action_timer`.
- Each advisor card should have 1-3 actions.
- Advisors should also carry faction tags.

## Initial Card List For First Playable Slice

Do not implement all cards at once. First playable slice should include:

Party Affairs:

- Campaign Urban Professionals.
- Campaign Urban Working Class.
- Campaign Unaffiliated Youth.
- Rebuild Regional Structures.
- Media Strategy.
- Ideological Debate.
- Civil Society Linkage.
- Coalition/List Negotiation.
- Party Congress.
- Shuffle Advisors.

Government Affairs:

- Coalition Affairs.
- Civil Partnership Bill.
- Labor Package.
- Housing Package.
- Rule-of-Law Restoration Position.
- Religion In Schools Reform.
- Ukraine Aid Position.

Advisor:

- Czarzasty.
- Biedron.
- Zandberg.
- Dziemianowicz-Bak.
- Belka.

Events:

- EP 2014.
- Local 2014.
- Presidential 2015.
- Sejm 2015.
- Black Protest 2016.
- Wiosna founding 2018.
- EP 2019.
- Sejm 2019.
- Presidential 2020.
- Abortion ruling 2020.
- Nowa Lewica 2021.
- Ukraine invasion 2022.
- Sejm 2023.
- Razem split 2024.
- Presidential 2025.
- Sejm 2027.

This gives a complete spine before detailed flavor.

## Advisor Implementation Matrix

### Wlodzimierz Czarzasty

Faction: apparatus/mainstream.

Actions:

- Apparatus Mobilization: `party_resources += 1`, apparatus strength +1, modernizer/reformer dissent +1.
- Sejm Maneuvering: defer one hostile vote or reduce coalition crisis by 1, costs media/civil society if overused.
- Coalition Hardball: secure portfolio or concession, raises partner dissent.

### Krzysztof Gawkowski

Faction: mainstream/technocratic.

Actions:

- Government Performance: rule-of-law or digital delivery +1, media reach +0.5.
- Technocratic Reform: pass low-salience administrative reform with low coalition dissent.

### Agnieszka Dziemianowicz-Bak

Faction: younger reformers/labor-left.

Actions:

- Next-Generation Face: women under 50, under 30, urban working class left affinity + small gain.
- Reformist Synthesis: reduce mainstream/reformer/Razem dissent if relations are not already broken.
- Labor Delivery: improve labor policy, raises PSL/KO budget concern.

### Robert Biedron

Faction: Wiosna/social-liberal.

Actions:

- Charismatic Rally: urban professionals, LGBTQ, unaffiliated support up; media reach up.
- European Alignment: EU alignment up; right-wing backlash small.
- LGBTQ Mobilization: equality salience up; church opposition up.

### Adrian Zandberg

Faction: Razem/radical-left.

Actions:

- Programmatic Discipline: economic credibility up; coalition dissent if in government.
- Anti-Coalition Speech: Razem cohesion up; KO relation down.
- Working-Class Authenticity Pitch: urban working class support up; media may frame as anti-government.

### Magdalena Biejat

Faction: bridge/pragmatic left.

Actions:

- Pragmatic Bridge: improves Razem relation and reformer/mainstream compromise.
- Senate/Parliamentary Credibility: media reach up, coalition friction down slightly.
- Presidential Campaign Lessons: after 2025, convert presidential defeat into brand learning.

### Anna Maria Zukowska

Faction: parliamentary leadership.

Actions:

- Parliamentary Discipline: reduce current faction dissent; risk media controversy if overused.
- Media Performance: high-variance media reach action.

### Marek Belka

Faction: elder technocratic.

Actions:

- Macroeconomic Authority: reduce budget panic, improve finance credibility.
- Brussels Credibility: EU alignment and funds handling up.

### Aleksander Kwasniewski

Faction: elder statesman.

Actions:

- Senior Statesman Endorsement: rare one-time media and older-voter boost.
- Diplomatic Backchannel: foreign alignment improvement.

### Marta Lempart

Type: civil-society semi-advisor.

Actions:

- Mass Mobilization: Strajk Kobiet mobilization up, women under 50 left support up, constitutional crisis risk up.
- Protest Discipline: maintain movement strength while lowering crisis risk if civil society capital is high.

## Chronological Event Implementation Plan

### 2014

EP Election 2014:

- Purpose: establish player weakness but not catastrophe.
- Choices: establishment campaign, anti-clerical campaign, younger-rebrand campaign.
- Outputs: media reach, left vote share, faction dissent.

Local Election 2014:

- Purpose: regional apparatus baseline.
- Outputs: party resources/yearly dues equivalent, regional machine strength.

Tusk To Brussels / Kopacz Government:

- Purpose: KO/PO faction shift.
- Effects: Tusk loyalists become externalized; Kopacz caretaker pressure.

### 2015

Presidential Candidate Selection:

- Historical Ogorek route should be available and dangerous.
- Alternatives: elder statesman, ideological candidate, no-name safe candidate.
- Outputs: media reach, party embarrassment, faction blame.

Presidential Election 2015:

- If historical: poor left result.
- Effects: brand strength down, apparatus dissent up or modernizer dissent up depending candidate.

Sejm List Strategy:

- United Left coalition: 8 percent threshold, higher combined vote potential, high catastrophe risk.
- SLD alone: 5 percent threshold, lower ceiling, safer survival path.
- Razem separate: state funding and future insurgency.
- Broad progressive list: hard to assemble, possible alternate path.

Sejm Catastrophe:

- If below threshold, `left_in_sejm = 0`.
- Massive party resource loss.
- Unlock wilderness cards.
- If Razem crosses 3 percent, state funding unlocks Razem route.

### 2016

Czarzasty Leadership:

- Choose Czarzasty, modernizer, elder compromise, or insurgent.
- Sets apparatus/reformer balance.

Constitutional Tribunal Crisis:

- Join KOD, lead separately, focus on left issues, or ignore.
- KOD strength and KO relation affected.

Black Protest:

- Pivotal movement event.
- Choices: lead, support, absorb activists, keep distance.
- Raises Strajk Kobiet strength/mobilization, secularization tide, women under 50 salience.

### 2017-2018

Judicial Reform Crisis:

- KOD movement and constitutional risk.

Local 2018:

- Recovery test.

Wiosna Founding:

- If player is SLD: negotiate, attack, ignore, or propose alliance.
- If player is Wiosna: launch with media burst and weak organization.

### 2019

Sekielski Documentary:

- Church scandal event.
- Raises secularization tide.
- Anti-clerical positioning opportunity.

EP 2019:

- European Coalition vs Wiosna vs PiS.
- Determines momentum before Sejm.

Sejm 2019:

- Lewica coalition return path.
- If successful, `left_in_sejm = 1`, seats computed.
- Negotiate SLD/Wiosna/Razem internal hierarchy.

### 2020

COVID:

- Cost-of-living, state capacity, media trust.

Presidential 2020:

- Candidate choice.
- Biedron historical result as baseline.
- Trzaskowski runoff support decision.

Abortion Tribunal Ruling:

- Strajk Kobiet mass mobilization.
- Left can lead, support, or mishandle.
- Long-term women-under-50 and secularization effects.

### 2021

Nowa Lewica Formation:

- Merge SLD/Wiosna.
- Leadership structure: dual chair, Czarzasty dominance, Biedron dominance, broader congress.
- Razem relation changes.

### 2022

Russian Invasion Of Ukraine:

- Russian threat to 9 or 10.
- Ukrainian refugees active.
- Defense spending pressure up.
- Ukraine alignment choices.
- Razem/mainstream foreign-policy tension.

Inflation/Cost Of Living:

- Populist realignment pressure.
- PiS spending politics.
- Left economic positioning.

### 2023

Sejm Campaign:

- List composition, KO relationship, Razem candidate placement.

Sejm Election:

- If historical OTL route, Lewica around 8.61 and 26 seats.
- Player can underperform or overperform.

Government Formation:

- Coalition with KO + TD + Lewica.
- Assign ministries.
- Set partner dissent baselines.
- Unlock government deck.

### 2024

Local 2024:

- Government midterm.

EP 2024:

- Morale crisis if poor result.

Civil Partnership And Abortion Votes:

- PSL/TD partner dissent.
- Left faction frustration.

Razem Split:

- Historical split if conditions align.
- Player can soften, prevent, accelerate, or exploit.

### 2025

Presidential Candidate:

- Biejat historical route.
- Alternative: Senyszyn, Zandberg, Biedron, technocratic candidate.

Presidential Election:

- Biejat 4.23 historical anchor.
- Zandberg separate candidacy if Razem split.
- Nawrocki victory historical anchor.
- Set `presidential_veto_pressure`.

Post-Election Coalition Shock:

- Tusk confidence event.
- Coalition renegotiation.
- KO succession pressure rises.

Nowa Lewica Statute / Congress:

- Frakcje abolished or preserved.
- Czarzasty sole leadership or contested transition.

Czarzasty Marshal:

- Historical scheduled event, 2025-11-18.
- If player conditions differ, can be challenged or traded away.

### 2026

Konfederacja Split Possibility:

- Check `konf_split_likelihood >= 7`.
- If split, create election fragmentation.

Kaczynski Health Crisis Possibility:

- Random/conditional.
- If triggered, activate `pis_post_kaczynski_fracture`.

Polska 2050 Collapse:

- Defection pipeline to KO/Lewica.

Ukraine Settlement:

- Probabilistic.
- Effects on refugees, defense, Trump alignment.

Nawrocki Veto Crisis:

- Increases constitutional-crisis risk and government frustration.

### 2027

Pre-Election Cabinet Reshuffle:

- Get campaign-facing ministries or sacrifice them.

Final List Strategy:

- Solo Lewica.
- Lewica + Razem reunification.
- Left-progressive bloc with KO defectors.
- Broad anti-PiS pact.
- Separate ideological list.

Sejm Election 2027:

- Final election.
- Compute vote shares, seats, threshold failures, right fragmentation.

Final Coalition Negotiation:

- Determines ending modifiers.

## Ending Implementation

Use ending score calculation after 2027 election and coalition negotiation.

Core ending variables:

- `left_votes`
- `left_seats`
- `left_led_bloc_votes`
- `democratic_bloc_seats`
- `right_bloc_seats`
- `left_brand_strength`
- `secularization_tide`
- `rule_of_law_restoration`
- `constitutional_crisis_risk`
- `coalition_cohesion`
- `pis_post_kaczynski_fracture`
- `konf_split_realized`
- `razem_status`

Endings:

1. Constitutional Majority.
2. Social Democratic Party Of Government.
3. Major Party.
4. Junior Coalition Partner.
5. Successful Survival.
6. Eliminated.
7. Absorbed By KO.
8. Razem-Dominated Left.
9. Constitutional Crisis / Authoritarian Breakthrough.
10. Right Wave.

Implementation priority:

- Implement `Eliminated`, `Junior Coalition Partner`, `Successful Survival`, and `Right Wave` first.
- Add rare/perfect endings after core election logic works.

## QDisplay Requirements

Create new qdisplays:

- `civil_society.qdisplay.dry`
- `constitutional_crisis.qdisplay.dry`
- `secularization.qdisplay.dry`
- `brand_strength.qdisplay.dry`
- `coalition_partner_dissent.qdisplay.dry`
- `external_alignment.qdisplay.dry`
- `movement_strength.qdisplay.dry`
- `movement_mobilization.qdisplay.dry`
- `threshold_safety.qdisplay.dry`
- `rival_faction_strength.qdisplay.dry`

Example bands:

Constitutional crisis:

- 0-1: settled
- 2-3: strained
- 4-5: unstable
- 6-7: open confrontation
- 8: institutional emergency
- 9-10: constitutional rupture

Secularization tide:

- 0-2: church-dominant
- 3-4: slow erosion
- 5-6: cultural contestation
- 7-8: secular wave
- 9-10: post-clerical realignment

Threshold safety:

- below 0: outside Sejm
- 0-1: knife edge
- 1-3: unsafe
- 3-5: viable
- 5+: safe

## Content Tone Rules

Tone should be:

- Polish-political-journalistic.
- Skeptical.
- Faction-aware.
- Institutional.
- Concrete.

Avoid:

- Western exoticization.
- Meme-only treatment.
- Smearing living people with invented personal misconduct.
- Treating speculative death/health events as certain.
- Making every left action correct.
- Making every right action cartoonishly irrational.

Use Polish terms when they carry political meaning:

- koalicja
- kongres
- frakcja
- klub parlamentarny
- kolo
- prog wyborczy
- betonowy elektorat
- miekki elektorat
- lista
- jedynki
- aparat
- samorzady

If writing English UI text, use Polish terms sparingly and contextually.

## First Implementation Milestone

Goal: A playable January 2014 to October 2015 skeleton.

Must include:

- New title/info.
- January 2014 initial state.
- New action economy.
- New voter segments.
- New election algorithm adapted to `voter_segments`.
- Basic Party Affairs deck.
- EP 2014 event.
- Local 2014 event.
- Presidential 2015 event.
- Sejm 2015 event with threshold logic.
- Eliminated ending if below threshold and no recovery route.

Acceptance criteria:

- Game starts in January 2014.
- No Weimar status labels remain on the main status screen.
- One, two, or three actions advance a month depending on difficulty.
- Player can reach October 2015.
- United Left coalition path can fail at 7.55-style result.
- SLD-alone path can survive if above 5.
- Razem external route can get funding without entering Sejm.
- Results change if player campaigns and manages factions well.

## Second Implementation Milestone

Goal: 2016-2019 return-to-Sejm arc.

Must include:

- Czarzasty leadership.
- Constitutional Tribunal/KOD.
- Black Protest.
- Wiosna founding.
- Sekielski church scandal.
- EP 2019.
- Sejm 2019 return.
- Internal SLD/Wiosna/Razem coalition negotiation.

Acceptance criteria:

- Black Protest creates long-term movement strength, not just one event.
- Wiosna can either drain SLD or become player identity.
- Lewica return to Sejm is possible but not guaranteed.
- Razem relationship matters in 2019 list formation.

## Third Implementation Milestone

Goal: 2020-2025 government path.

Must include:

- COVID.
- 2020 presidential humiliation.
- Abortion ruling/Strajk Kobiet.
- Nowa Lewica formation.
- Ukraine invasion.
- 2023 election and government formation.
- Ministry ownership.
- Government deck.
- Razem split.
- 2025 presidential defeat.
- Nawrocki veto pressure.
- Czarzasty Marshal.

Acceptance criteria:

- Government participation feels constraining, not only rewarding.
- Equality and abortion cards can fail due to coalition partner dissent.
- Razem split can be prevented only with serious tradeoffs.
- 2025 presidential result changes coalition politics.

## Fourth Implementation Milestone

Goal: 2026-2027 endgame.

Must include:

- PiS post-Kaczynski uncertainty.
- Konfederacja split possibility.
- Polska 2050 collapse.
- Tusk succession pressure.
- Ukraine settlement/random foreign policy crisis.
- 2027 final list strategy.
- Sejm 2027.
- Endings.

Acceptance criteria:

- Constitutional Majority is rare and requires multiple aligned successes.
- Right Wave is a real failure path.
- KO absorption is distinct from elimination.
- Razem-dominated ending is possible.

## Testing Plan

Manual scenario tests:

1. Historical SLD path: United Left fails in 2015.
2. Safe SLD path: SLD runs alone and narrowly survives.
3. Razem path: Razem gets funding and becomes future rival.
4. Wiosna path: Wiosna displaces SLD as urban-progressive pole.
5. Pragmatic government path: Lewica joins 2023 government and survives.
6. Razem reconciliation path: split is repaired by 2027.
7. KO absorption path: Lewica loses identity while in government.
8. Constitutional crisis path: Nawrocki veto conflict escalates.
9. Right wave path: PiS + Konfederacja win 2027.
10. Perfect path: Lewica-led constitutional majority.

Regression checks:

- No negative demographic affinities after monthly resolver.
- Every active faction has strength and dissent.
- Faction strengths normalize correctly.
- Every ministry-gated card has a clear unavailable reason.
- Movement mobilization decays.
- Constitutional crisis risk does not only increase.
- Election threshold logic uses 5 or 8 percent correctly.
- Sejm seats sum to 460.
- Endings are mutually exclusive or deterministic by priority.

## Implementation Risk Register

Risk: Too many overlapping voter segments make elections unstable.  
Mitigation: Treat segments as political salience pools and normalize final support.

Risk: Rival-party faction model becomes too complex.  
Mitigation: Implement KO/PiS/Konf first. Add TD/PSL/PL2050 later.

Risk: Living-person content becomes defamatory or too speculative.  
Mitigation: Use public-record actions only. Label speculative health/departure events as rumors, succession pressure, or institutional uncertainty.

Risk: Old Weimar variables remain and distort the game.  
Mitigation: Create a variable audit. Remove or quarantine old `hindenburg`, `reichsbanner`, `prussia`, `nsdap`, `kpd`, `dnvp` variables from visible state.

Risk: Full rewrite stalls.  
Mitigation: Build chronological playable slices rather than perfect full-period content.

Risk: Government years overshadow wilderness years.  
Mitigation: Make 2014-2019 the identity-formation arc and 2023-2027 the delivery arc.

Risk: Secularization becomes an automatic win button.  
Mitigation: Keep church institutional power high and make aggressive secularism raise coalition and backlash risks.

Risk: Civil-society movement model feels passive.  
Mitigation: Movement mobilization should visibly affect cards, public opinion, crisis risk, and event outcomes.

## Implementation Checklist Before Code

- Approve this blueprint.
- Decide whether first playable identity is SLD-only or chooseable SLD/Razem/Wiosna starts.
- Decide whether UI language starts in English or Polish.
- Decide whether election outcomes in historical mode are forced or only strongly nudged.
- Decide whether generated `out/html` will be committed after each milestone.
- Decide whether to preserve old DSD scenes under an archive folder or delete them progressively.

## Immediate Next Step After Approval

Create the first playable 2014-2015 skeleton:

1. Rename game metadata.
2. Replace root initialization.
3. Replace status page.
4. Implement new `actions_per_month`.
5. Implement voter segments.
6. Implement first four Party Affairs cards.
7. Implement 2014 and 2015 election events.
8. Add threshold failure ending.
9. Build and run locally.

No later-period content should be coded before the 2014-2015 skeleton can be played end to end.

