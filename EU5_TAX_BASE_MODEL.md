# EU5 Guide

A working strategy manual for Europa Universalis V, built by reading the installed game files rather than relying on memory or wikis. This edition covers the **economy** end to end: how wealth is created, how much of it becomes taxable, and how it finally reaches the treasury. Military, diplomacy, society, religion, and progression are scaffolded as [planned sections](#planned-sections) and will follow the same file-first method.

**Generated from:**

- Install path: `/mnt/d/Program Files/Steam/steamapps/common/Europa Universalis V`
- Captured: `2026-06-06`
- `caesar_branch.txt`: `u26q2/release/next`
- `caesar_rev.txt` / `binaries/checksum.txt`: `4be9dd2e34196e66a72855407a347f5235e3bd27c2c34fc63a0e2980cc2e8149eed42774`

**Source policy:** installed game files are authoritative. Public sources are useful cross-checks, but can lag the local build. Reusable search paths live in the Codex skill `eu5-game-guide`. For a broader catalog of moddable parameters, see [EU5_GAMEPLAY_PARAMETERS.md](EU5_GAMEPLAY_PARAMETERS.md).

## How to Read This Guide

Two label systems run through every table, so you can separate *how much a lever matters* from *how well the game documents it*.

**Strategy weight** — how much the dependency typically moves the outcome. These are prioritization judgments, not claims that each branch has one stable global coefficient.

| Weight | Meaning |
|---|---|
| `Critical` | Usually the first-order constraint or multiplier. Fix these first. |
| `High` | Strongly affects outcomes in many starts. |
| `Medium` | Often important, but depends on context or a specific branch. |
| `Low` | Mostly secondary for tax base unless combined with other levers. |
| `Conditional` | Can dominate in specific markets, geographies, or estate setups. |

**Evidence level** — how directly the claim is backed by the files.

| Evidence | Meaning |
|---|---|
| `Exact` | Directly exposed as a formula, coefficient, or file value. |
| `Derived` | Algebraically inferred from exact pieces. |
| `Mixed` | Partly file/UI backed, partly engine-side or state-dependent. |
| `Inferred` | Useful strategy model, but not exposed as a direct formula. |

**Reading the equations:** treat UI percentages as decimals (75% control = `0.75`). An `=` line is exact from the files; a `~=` line is a strategy-level approximation of engine-side math.

**Navigation:** the [Table of Contents](#table-of-contents) lists every section. [Critical Dependencies](#critical-dependencies) is the fastest orientation; the [Dependency Graph](#dependency-graph) is the full map, and every subgraph links back to it.

## Table of Contents

**Economy** (current edition)

1. [Quick Mental Model](#quick-mental-model) — the five-layer summary
2. [Critical Dependencies](#critical-dependencies) — the levers that decide most outcomes
3. [Core Equations](#core-equations) — wealth, tax base, estates, prices, trade
4. [Dependency Graph](#dependency-graph) — the full map and every subgraph
5. [Parameter Dictionary](#parameter-dictionary) — vocabulary for files and tooltips
6. [Strategy Checklist](#strategy-checklist) — the order to evaluate an economic action
7. [Common Strategic Patterns](#common-strategic-patterns) — recipes for typical situations

**Planned Sections** (placeholders): [Military](#military) · [Diplomacy and Subjects](#diplomacy-and-subjects) · [Society and Government](#society-and-government) · [Religion and Culture](#religion-and-culture) · [Progression and Events](#progression-and-events)

## Quick Mental Model

EU5's economy is easiest to read in layers:

1. Locations create `wealth` from profitable RGOs, profitable buildings, and sometimes burgher trade.
2. `control` decides how much of that wealth becomes taxable `tax_base`.
3. Estates divide the taxable base by local weighted population shares.
4. Estate tax rates, caps, efficiency, and estate enrichment decide how much reaches the treasury.
5. Markets, trade, storage, roads, ports, laws, privileges, reforms, and production methods move the upstream prices and availability that make wealth rise or fall.

The strategic shortcut:

- High-wealth locations with low control are wasted taxable potential.
- Low-market-access locations struggle to turn production into profit.
- Expensive output goods help producers; expensive input and maintenance goods hurt users of those goods.
- Trade can improve state income directly, but it also affects tax base indirectly by moving prices, supply, and burgher trade wealth.
- Normal player trade routes and autonomous Burgher trade share the same market ecology, but they do not use the same capacity/control surface.
- Estate composition changes who receives the tax base and how hard the state can tax it.

## Critical Dependencies

If you read nothing else, read this. Treasury income is a chain, and it is only as strong as its weakest link: wealth you cannot control is not taxable, and tax base held by an estate you cannot tax is not income.

**The critical path**

```text
location_wealth
   │  × local_control          (control decides how much wealth is taxable)
   ▼
location_tax_base
   │  × estate share           (which estate owns the taxable base)
   ▼
estate_tax_base
   │  × tax rate × efficiency   (only if that estate can be taxed, under caps)
   ▼
treasury income
```

Every `Critical`-weighted lever in the guide sits on this path:

| Lever | Stage | Why it is critical | Detail |
|---|---|---|---|
| `local_control` | Tax base | Direct multiplier on wealth. 100 wealth at 50% control = 50 tax base; at 80% = 80. Also feeds local and national Crown Power. | [Control](#control-subgraph) |
| `location_wealth` | Tax base | The base amount before control. High control over poor land still yields little. | [Location Wealth](#location-wealth-subgraph) |
| Building profit | Wealth | The most scalable wealth branch; drives towns, cities, and industry. | [Building Wealth](#building-wealth-subgraph) |
| Output price/quantity vs. input/maintenance cost | Building profit | Expensive inputs erase output gains and can turn a building unprofitable (and idle). | [Building Wealth](#building-wealth-subgraph) |
| Employment | Building profit | Buildings are efficient in proportion to employed pops; unstaffed buildings sit idle. | [Employees, POP Deficits, and Profit](#employees-pop-deficits-and-profit) |
| Estate taxability (rate and caps) | Extraction | Tax base only becomes income if the estate holding it can actually be taxed. | [Estate Allocation](#estate-allocation-and-tax-extraction-subgraph) |

**The shortlist** — when you can only do a few things:

1. **Raise control where wealth is already high.** It is the cleanest multiplier and compounds into Crown Power. Rich-but-uncontrolled land is usually the highest-value fix.
2. **Protect building profitability.** Keep input and maintenance goods cheap and available, and make sure the building can be staffed.
3. **Check who owns the tax base before celebrating wealth.** An estate you cannot tax converts little of it to treasury.

**The common traps:**

- High-wealth, low-control locations are wasted potential — see [Rich But Uncontrolled Land](#rich-but-uncontrolled-land).
- A high-output building with bad input prices or poor market access adds little taxable wealth.
- Expanding into large but low-control populations can *lower* national Crown Power through the average-control path.

Read this together with the [Strategy Checklist](#strategy-checklist), which walks the same chain in evaluation order.

## Core Equations

The formulas behind the economy, kept together for reference. The [Dependency Graph](#dependency-graph) turns each into a strategy tree with weights, and the [Parameter Dictionary](#parameter-dictionary) defines the symbols. (UI percentages are decimals: 75% control = `0.75`; `=` is exact from the files, `~=` is a strategy approximation.)

### Wealth And Tax Base

```text
location_wealth =
    max(0, RGO_profit)
  + sum(max(0, building_profit))
  + burgher_trade_wealth_if_present

location_tax_base = location_wealth * local_control

province_tax_base = sum(location_tax_base in province)

country_tax_base = sum(owned province_tax_base)
```

Evidence level: `location_tax_base = location_wealth * local_control` is exact from installed concept text. RGO, building, and burgher trade wealth are exposed by UI concepts, but their full profit internals are partly engine-side.

### Source Profit

```text
RGO_profit ~= output_value - operating_costs

employee_fill_ratio =
    building_employed_amount / building_employment_size_amount

realized_building_throughput ~=
    min(1, employee_fill_ratio)
  * input_access_factor
  * market_access_factor
  * establishment_throughput_if_enabled

building_profit ~=
    output_value * realized_building_throughput
  - input_costs * realized_building_throughput
  - maintenance_costs
```

For strategy, treat profit as:

```text
profit rises when:
  output quantity rises
  output sell price rises
  market access improves
  input availability improves
  input or maintenance prices fall
  production efficiency rises
  employment improves
  establishment improves if enabled in the build

profit falls when:
  output price falls
  input or maintenance prices rise
  market access is low
  buildings lack labor or inputs
  market stockpiles cannot cover shortages
  production workers are pulled into other buildings
```

Evidence level: mixed. The installed concepts and GUI expose profit as the source of wealth, buildings as employee-dependent, and employee/full-capacity triggers; exact price, availability, employment-throughput, and trade interactions are engine-side. The current installed define has the establishment system disabled, even though localization and UI strings still describe it.

### Estate Shares And Tax Income

```text
estate_weight_e =
    local_pop_e
  * tax_per_pop_e
  * estate_power_modifier_e

estate_share_e =
    estate_weight_e / sum(estate_weight_all_estates)

peasant_enfranchisement_final =
    Location.GetPeasantEnfranchisment

peasants_final_share =
    estate_share_peasants * peasant_enfranchisement_final

nobles_final_share =
    estate_share_nobles
  + estate_share_peasants * (1 - peasant_enfranchisement_final)

other_final_share_e =
    estate_share_e for estates other than Peasants and Nobles

estate_tax_base_e =
    location_tax_base * final_estate_share_e

tax_income_from_estate_e =
    estate_tax_base_e
  * effective_tax_rate_e
  * (1 + tax_income_efficiency)

location_tax_income = sum(tax_income_from_estate_e)
```

For quick estimates, treat `estate_power_modifier_e` as `1` unless the location/country has explicit `local_<estate>_estate_power`, `global_<estate>_estate_power`, or related estate-power modifiers. The installed concepts say tax-base distribution depends on local population, estate power, and pop type; the exact engine ordering is not fully exposed.

`final_estate_share_e` means the estate share after special transfer rules. The visible peasant value is `Location.GetPeasantEnfranchisment`; the underlying modifier keys use the installed spelling `local_peasant_enfranchisment` and `global_peasant_enfranchisment`.

Installed per-pop weights (`estates/00_default.txt`). `tax_per_pop` weights the tax-base split; `power_per_pop` weights [estate power](#estate-power). The two are independent, so an estate can be rich but politically weak (Burghers) or poor but influential (Cossacks). `power_per_pop` is counted **per 1,000 pops** (Nobles `25` = `0.025` political power per pop):

| Estate | `tax_per_pop` | `power_per_pop` | Strategic meaning |
|---|---:|---:|---|
| Crown | 0 | 0 | Crown pops take neither tax-base share nor per-pop power; Crown Power comes from population, command, cabinet, and modifiers instead. |
| Nobles | 100 | 25 | Dominant claim on both wealth and influence; noble-heavy locations shift tax base and political power to Nobles. |
| Clergy | 25 | 10 | Meaningful tax and power weight, below Nobles. |
| Burghers | 40 | 4 | Strong tax claim but modest per-pop power: commercial wealth without proportional political weight. |
| Peasants | 1 | 0.025 | Huge numbers, tiny per-pop weight; one Nobles pop outweighs ~1,000 Peasants pops in power. |
| Dhimmi | 1 | 0.02 | Low on both. |
| Tribes | 0.01 | 0.01 | Very low on both. |
| Cossacks | 0.02 | 0.5 | Low tax weight but relatively high per-pop power for a non-elite estate. |

Two adjustments shift these shares before the estate is taxed: **peasant enfranchisement** redirects part of the Peasants' share to the Nobles (Peasants keep their enfranchisement %, Nobles get the rest), and **estate enrichment** reduces estate tax-base income. These — along with the full Crown-and-estate-power treatment (how `estate_power` is computed, how `local_crown_estate_power` rolls up into national Crown Power, the `crown_power_from_population` and `inverse_control` paths, the `0.25` high/low-power threshold, and worked examples from the game UI) — are detailed in the [Estate Allocation And Tax Extraction](#estate-allocation-and-tax-extraction-subgraph) subgraph.

Evidence: estate weights are `Exact` from `estates/00_default.txt`; the peasant transfer is `Exact` from the enfranchisement tooltip and `disenfranchise_to = nobles_estate`; the final political-power aggregation and the enrichment ordering are engine-side (`Mixed`).

### Market Price Path

```text
market_price moves toward target_price over time

target_price is driven by:
  effective_supply
  effective_demand
  price_stability

effective_supply = supply with reduced import impact
effective_demand = demand with reduced export impact
```

Evidence level: concept text confirms this shape. The full price function is engine-side.

### Trade Profit And Income

A route buys a good in one market and sells it in another. Profit and the treasury's cut are both in the trade UI:

```text
route_profit =
    sell_value                 # amount sold * sell price, after selling/import/export efficiency
  - buy_cost                   # amount bought * buy price
  - trade_route_maintenance
  - sound_toll_if_any

profit_per_trade_capacity = route_profit / used_trade_capacity
trade_income_to_treasury  = route_profit * trade_income_share
```

`trade_income_share` is the fraction of profit that reaches the treasury; it tracks **Crown Power** (the rest goes to the estates by their estate power — see [National Crown Power](#national-crown-power)).

*Worked example — Saffron, Konstantinoúpolis → Venexia:*

| Leg | Value |
|---|---:|
| Sell 1.30 in Venexia @ 4.61 | `+6.55` |
| Buy 1.30 in Konstantinoúpolis @ 4.42 | `−5.65` |
| Trade route maintenance | `−0.32` |
| Sound toll | `−0.05` |
| **Total profit** | **`+0.52`** |
| Trade income to treasury (`24%` share) | `0.52 × 0.24 =` **`+0.12`** |

The route nets `+0.52`, but only the `24%` `trade_income` share — `+0.12` — reaches the treasury.

**How much you can ship — capacity vs. distance.** Trade capacity converts to goods through a per-unit transport cost that jumps once a route passes your trade range:

```text
amount_shippable        = trade_capacity / transport_cost_per_unit
transport_cost_per_unit = good_base_transport * distance_multiplier
distance_multiplier     = 1                          if route_cost <= trade_range
                          (route_cost - trade_range) * distance_cost_factor   otherwise
```

*In range:* `2.10` capacity ÷ `0.50` per unit of Incense = **`4.20`** shipped. *Out of range* (Varanasi → Venexia): the distance multiplier hits `×10.72`, so per-unit cost = `0.50 × 10.72 = 5.36` and `0.99` capacity ships only `≈ 0.18` Cloves. (Observed factor: `(8273 − 495) × ~0.0012`.)

**Market balance** sets the price pressure a route trades into:

```text
market_balance = effective_supply - effective_demand
   > 0  surplus -> price falls -> good to BUY
   < 0  deficit -> price rises -> good to SELL
```

A stockpile above `50%` capacity adds extra supply (growing until `75%`). Example: Jewelry in Venexia, supply `137.36` − demand `106.13` = **`+31.23`**, of which `+69.4` supply comes from a `1389`-unit stockpile.

Evidence: route profit, profit-per-capacity, trade-income share, capacity→goods, distance cost, and market balance are all `Exact` from the UI (worked above); buy/sell price internals and the exact distance-cost factor are `Mixed`.

**Burgher trade is a separate, autonomous system** — not a player route:

```text
burgher_trade_capacity = burgher_population * (local_trades_per_burgher + global_trades_per_burgher + ...)
```

Burghers trade on their own to satisfy goods needs in their town/city market, adding to location wealth (the exact conversion is engine-side). No other pop type has an autonomous `trades_per_<pop>` capacity in this build.

## Dependency Graph

This is the full map of the economy. Weight and evidence labels are defined in [How to Read This Guide](#how-to-read-this-guide). Start at the top-level graph, then follow any branch into its subgraph; every subgraph links back here. Indentation shows what feeds what.

**Top-level economy graph:**

- [Country Aggregation](#country-aggregation-subgraph) `[Weight: High, Exact]`
- [Location Tax Base](#location-tax-base-subgraph) `[Weight: Critical, Exact]`
  - [Control](#control-subgraph) `[Weight: Critical, Exact/Mixed]`
  - [Location Wealth](#location-wealth-subgraph) `[Weight: Critical, Mixed]`
    - [RGO Wealth](#rgo-wealth-subgraph) `[Weight: High, Mixed]`
    - [Building Wealth](#building-wealth-subgraph) `[Weight: Critical, Mixed]`
    - [Burgher Trade Wealth](#burgher-trade-wealth-subgraph) `[Weight: Conditional, Mixed]`
  - [Market Access](#market-access-subgraph) `[Weight: High, Mixed]`
  - [Goods Demand And Prices](#goods-demand-and-prices-subgraph) `[Weight: High, Mixed]`
  - [Storage And Stockpiles](#storage-and-stockpiles-subgraph) `[Weight: Medium, Mixed]`
- [Estate Allocation And Tax Extraction](#estate-allocation-and-tax-extraction-subgraph) `[Weight: High, Exact/Mixed]`
- [Trade And Naval Economy](#trade-and-naval-economy-subgraph) `[Weight: Conditional, Mixed]`

### Country Aggregation Subgraph

[Back to top graph](#dependency-graph)

```text
country_tax_base
└─ sum owned province_tax_base
   └─ sum location_tax_base in province
      └─ location_wealth * local_control
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| More high-tax-base locations | High | Exact | Expansion, integration, and conquest matter when the added locations have taxable wealth or can be brought under control. |
| Province/location distribution | Medium | Exact | The country value is additive, but local bottlenecks decide whether a location contributes much. |
| Intel/UI visibility | Low | Exact UI | Does not change the economy, but affects what the player can inspect on foreign countries. |

Player rule: when comparing expansion targets, do not only ask "how much wealth exists?" Ask "how much wealth can I control and tax soon?"

### Location Tax Base Subgraph

[Back to top graph](#dependency-graph)

```text
location_tax_base
├─ location_wealth [Critical, Mixed]
│  ├─ RGO profit
│  ├─ building profit
│  └─ burgher trade wealth
└─ local_control [Critical, Exact]
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| `local_control` | Critical | Exact | Direct multiplier. A 100-wealth location at 50% control gives 50 tax base; at 80% control it gives 80. |
| `location_wealth` | Critical | Exact/Mixed | Direct base amount before control. High control on poor land still produces little. |
| Source-wealth mix | High | Mixed | RGO, building, and trade branches respond to different investments, so the best lever depends on what creates the location's wealth. |

Player rule: prioritize control where wealth is already high, and build wealth where control and market access can support it.

### Location Wealth Subgraph

[Back to top graph](#dependency-graph)

```text
location_wealth
├─ max(0, RGO_profit)
├─ sum(max(0, building_profit))
└─ burgher_trade_wealth, if burghers are present
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Positive RGO profit | High | Mixed | Usually important in rural/high-value raw-good locations. |
| Positive building profit | Critical | Mixed | Core driver of towns, cities, industry, and many compounding production chains. |
| Burgher trade wealth | Conditional | Mixed | Matters most in urban/trade-heavy locations with burghers and trade support. |
| Negative profit branches | Low for tax base | Exact/Mixed | Unprofitable sources do not meaningfully add wealth; they may still drain treasury if subsidized. |

Player rule: local wealth is not the same as production volume. A high-output building with bad input prices or bad market access can fail to add much taxable wealth.

### RGO Wealth Subgraph

[Back to top graph](#dependency-graph)

```text
RGO_profit
├─ raw good identity
│  ├─ category = raw_material
│  ├─ method = mining / farming / gathering / forestry / hunting
│  ├─ default_market_price
│  ├─ transport_cost
│  └─ base_production, where defined
├─ RGO level and employment
│  ├─ RGO size
│  ├─ laborers or slaves
│  └─ each level can employ up to 1,000 rgo_pops
├─ output modifiers
│  ├─ local_<good>_output_modifier
│  ├─ global_goods_<good>_output_modifier
│  ├─ local/global max RGO size modifiers
│  └─ method-specific expansion cost/build-time modifiers
├─ market result
│  ├─ market access
│  ├─ market price
│  ├─ demand and supply
│  └─ imports/exports
└─ geography and setup
   ├─ topography
   ├─ vegetation
   ├─ rural rank
   ├─ roads and proximity
   └─ laws, privileges, advances, and town rights
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Raw good price and demand | High | Mixed | Valuable goods such as precious metals, gems, spices, and scarce inputs can justify RGO investment. |
| RGO size and employment | High | Mixed | More levels only matter when they can employ pops and sell output profitably. |
| Market access | High | Mixed | Low access reduces RGO profit and weakens the link between production and wealth. |
| Good-specific output modifiers | Medium/High | Exact/Mixed | Strong for targeted regions or goods; weaker if the good is low value or oversupplied. |
| Terrain/vegetation/rural status | Medium | Exact/Mixed | Affects capacity, build time, proximity, and whether urbanization harms RGO potential. |

Key files:

| Layer | Installed files | Parameters to inspect |
|---|---|---|
| Goods | `game/in_game/common/goods/*.txt` | `category`, `method`, `default_market_price`, `transport_cost`, `base_production`, `demand_add`, `demand_multiply` |
| RGO capacity | `advances`, `laws`, `estate_privileges`, `town_rights`, `location_ranks`, `vegetation`, `pop_types` | `global_max_rgo_size_modifier`, `global_max_rgo_size_modifier_in_rural`, `local_max_rgo_size_modifier` |
| Good output | buildings, advances, laws, privileges, events | `local_<good>_output_modifier`, `global_goods_<good>_output_modifier` |
| Expansion cost/time | goods, advances, laws, town rights, topography, vegetation | `expand_rgo_*_cost_modifier`, `global_rgo_build_time`, `local_rgo_build_time` |

Player rule: expand valuable RGOs when the location can access a market, employ the RGO, and maintain demand. Urbanize low-value rural locations first because some urban ranks reduce RGO potential.

### Building Wealth Subgraph

[Back to top graph](#dependency-graph)

```text
building_profit
├─ building level and capacity
│  ├─ max_levels
│  ├─ employment_size
│  ├─ building cap script values
│  └─ low-market-access cap penalties
├─ labor and establishment
│  ├─ pop_type
│  ├─ employment_size
│  ├─ building_employed_amount
│  ├─ available population / promotion
│  └─ establishment speed/current establishment if enabled
├─ production recipe
│  ├─ input goods
│  ├─ output goods
│  ├─ maintenance demand
│  └─ production method category
├─ output side
│  ├─ output quantity
│  ├─ output-good market price
│  ├─ production efficiency
│  └─ local/global output modifiers
├─ input side
│  ├─ input-good availability
│  ├─ input-good market price
│  ├─ maintenance-good market price
│  └─ stockpile coverage
└─ market access
   ├─ local sell price
   ├─ purchase order / availability
   └─ building output penalty when low
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Output price and output quantity | Critical | Mixed | The main upside of a profitable building. |
| Input and maintenance costs | Critical | Mixed | Expensive inputs can erase output gains. |
| Market access | High | Mixed | Low access harms output and can also reduce building caps. |
| Employment | Critical | Exact/Mixed | Buildings are efficient based on employed pops; missing workers reduce realized output/profit and can leave the building idle. |
| Establishment | Disabled in current build; high if re-enabled | Exact/Mixed | Localization/UI describe ramp-up, but installed defines currently disable the establishment system. |
| Production efficiency | High | Mixed | Directly improves production profitability when the building is already viable. |
| Building caps | Medium/High | Exact/Mixed | Exact cap scripts constrain scaling; impact depends on local development/population/access. |

Installed exact cap pattern:

```text
if market_access < 0.75:
    building_cap *= market_access + 0.25
```

This appears in several building cap script values. It means very low market access can reduce how far some building categories can scale.

#### Employees, POP Deficits, And Profit

Buildings are not just slots. The installed concept text says building efficiency is based on how many pops are employed there. A building type has a supported `pop_type` and an `employment_size`; the building can only support one pop type, and the maximum employed amount depends on the building type and level.

```text
employee_fill_ratio =
    building_employed_amount / building_employment_size_amount

realized_output ~=
    recipe_output
  * min(1, employee_fill_ratio)
  * production_efficiency
  * market_access/input_access factors

realized_profit ~=
    realized_output_value
  - realized_input_costs
  - maintenance_costs
```

This is a strategy model, not an exposed engine formula. The exact UI-backed pieces are `employment_size`, `building_employed_amount`, `building_is_full_capacity`, and the construction tooltip's expected/current available worker values.

Building in a POP deficit does not magically create full profit. It creates unfilled jobs first:

```text
new building with too few eligible pops
  -> missing employees
  -> lower realized output
  -> input use follows engine-side throughput rules
  -> lower building profit and lower location wealth
  -> lower tax base after control
  -> waits for promotion, migration, or freed workers
```

If there is not enough workforce or promotion in the location, the build UI warns that the building might never be able to employ workers. The alert text also says to wait until buildings or RGOs are filled before building more in that location.

If the resulting profit is negative, the installed concept text says the building is considered unprofitable and stops working; the subsidy concept says an unprofitable building will not buy needed goods or function unless the country subsidizes it. Subsidies can keep a strategic chain alive, but the treasury pays the loss.

The important distinction:

| Situation | Profit effect | Strategic read |
|---|---|---|
| Enough eligible unemployed pops | Building can staff quickly; profit mostly depends on price/access/inputs. | Good construction target if forecast profit is positive. |
| Some deficit, positive promotion | Building underperforms at first, then improves as peasants promote or workers migrate. | Acceptable for long-term urbanization or high-value chains; expect delayed ROI. |
| Deficit and no/low promotion | Building may sit understaffed for a long time, possibly never reaching intended output. | Usually bad unless you need the strategic modifier, can subsidize it, or will solve labor soon. |
| Existing profitable buildings are hiring the same pop type | New building can compete for workers and reduce other buildings' staffing. | Avoid overbuilding one labor type in a small location; spread construction or raise promotion/migration. |

Promotion matters because the concept text says peasants promote when there are possible positions in their location, and only pops of at least tolerated culture can promote. Control also affects promotion speed. In practice, a worker-short location needs one or more of these before the forecast becomes real:

- More available pops of the required `pop_type`.
- Faster local/global pop promotion.
- Migration attraction and population growth.
- Higher control, if low control is slowing promotion.
- Less competition from other buildings/RGOs for the same pop type.

Current installed-build caveat: the UI and localization include establishment throughput, establishment ETA, and "only one building at a time gains establishment at maximum efficiency." However, `loading_screen/common/defines/00_defines.txt` currently has `ESTABLISHMENT_SYSTEM_ENABLED = no`, whose comment says buildings open at `100%` throughput with no production-efficiency bonus or crowded-establishment penalty. If that define changes in a later build, unestablished buildings should again be modeled as a temporary throughput penalty below the threshold, then a production-efficiency bonus above it.

Key files:

| Layer | Installed files | Parameters to inspect |
|---|---|---|
| Building type | `game/in_game/common/building_types/*.txt` | `category`, `max_levels`, `employment_size`, `pop_type`, `build_time`, `construction_demand`, `maintenance_demand`, `modifier`, `raw_modifier` |
| Production recipe | `production_methods/*.txt`, `unique_production_methods` inside building types | input goods, output goods, output amount, maintenance goods |
| Capacity | `script_values/building_caps.txt`, `location_ranks/00_default.txt` | cap formulas, rank bonuses, low-market-access penalties |
| Labor and staffing | employment systems, building tooltips, location/province UI | `employment_size`, `building_employed_amount`, `location_unemployed_population_for_building_type`, missing employees, promotion speed |
| Profit modifiers | advances, laws, privileges, town rights, pop types, buildings | `global_production_efficiency`, `local_production_efficiency`, output modifiers, establishment if enabled |

Player rule: build where the output is valuable, inputs are affordable, market access is good, and labor exists or can be created quickly. Industrial goods should often be kept cheap if many domestic buildings consume them as inputs or maintenance.

### Burgher Trade Wealth Subgraph

[Back to top graph](#dependency-graph)

```text
burgher_trade_wealth
├─ burgher pops present
├─ trades per burgher
│  ├─ local_trades_per_burgher
│  └─ global_trades_per_burgher
├─ burgher trade capacity in market
│  ├─ Market.GetBurgherTradeCapacity
│  ├─ Market.GetBurgherTradeInfo
│  └─ burgher food imports/exports, where shown
├─ market conditions shared with normal trade
│  ├─ goods needs in the burghers' town/city market
│  ├─ market supply and demand
│  ├─ market prices
│  ├─ export_impact_on_demand
│  └─ normal trade demand competing in price calculations
├─ ports and maritime support
│  ├─ harbor suitability / harbor capacity
│  ├─ maritime presence
│  └─ port buildings
└─ Burgher estate effects
   ├─ power_per_pop = 4
   ├─ tax_per_pop = 40
   ├─ satisfaction production/merchant bonuses
   ├─ high-power capacity bonuses
   └─ high-power max-tax penalty
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Burgher presence | Critical for this branch | Exact/Mixed | No burghers means no burgher-trade wealth branch. |
| `local/global_trades_per_burgher` | High | Exact/Mixed | Direct knobs for more burgher trade activity. |
| Burgher trade capacity | High | Exact/Mixed | Market UI has a separate Burgher trade capacity value and tooltip. |
| Market prices and needs | High | Mixed | Burgher trade responds to goods needs in its town/city market and can affect price calculations. |
| Normal trade demand | Conditional | Exact/Mixed | `export_impact_on_demand` explicitly references both burgher trade and normal trade demand in price calculations. |
| Player merchant capacity/power | Conditional/Indirect | Mixed | These drive normal player trade routes; they share the market price/supply environment with Burgher trade but are not shown as the direct Burgher trade capacity counter. |
| Burgher estate power | Conditional | Exact/Mixed | Can add trade capacity/production strength but reduce Crown Power and max Burgher tax. |

Burgher trade versus player trade:

| Question | Normal player trade routes | Burgher trade |
|---|---|---|
| Who controls it? | The country/player through manual routes, trade orders, or automatic normal trade routes shown in the trade UI. | Burgher pops/estate activity; the player shapes it indirectly through burgher population, buildings, privileges, laws, and trade capacity modifiers. |
| Main capacity | `merchant_capacity`, shown on route details as used/assigned capacity. | `burgher_trade_capacity`, shown on market summaries and driven by trades per Burgher population. |
| Profit UI | Route UI exposes buy, sell, maintenance, sound toll, total profit, and profit per merchant capacity. | Market UI exposes Burgher trade capacity/info; exact wealth gain is not exposed as the same route-profit object. |
| Direct treasury income | Route profit can become trade income through `trade_income` and Crown/estate extraction. | Adds to location wealth when Burghers are present, then must pass through control, estate share, tax rates, caps, and efficiency to reach the treasury. |
| Shared dependencies | Market prices, supply/demand, exports/imports, goods availability, storage, ports, maritime/trade infrastructure, and `export_impact_on_demand`. | Same market ecology, but different capacity and control surface. |

Other pops and trade:

| Pop role | How it affects markets | Does it have autonomous trade capacity? |
|---|---|---|
| Burghers | Can do their own trades to satisfy needs in their town/city market; can add location wealth. | Yes, through Burgher trade capacity. |
| Peasants, laborers, nobles, clergy, dhimmi, tribes, cossacks, slaves | Create pop-needs demand, provide labor or estate presence, and affect wealth/tax allocation through estate population and power. | No matching `trades_per_<pop>` modifier found in installed files. |
| Building/RGO workers | Produce goods through buildings/RGOs, which then sell to market and create wealth if profitable. | No; production is not the same as autonomous pop trade. |

Burgher power is double-edged:

```text
higher Burgher influence can improve:
  merchant capacity
  merchant power
  production efficiency
  urban trade strength

higher Burgher influence can also hurt treasury extraction by:
  reducing Crown Power
  reducing treasury share of trade income
  lowering Burgher max tax when Burgher power is high
```

Player rule: Burghers are powerful in commercial economies, but commercial power is not identical to state revenue. Watch Crown Power, tax caps, and the treasury share of trade income.

### Goods Demand And Prices Subgraph

[Back to top graph](#dependency-graph)

```text
market_price
└─ target_price over time
   ├─ effective_supply
   │  ├─ local production
   │  ├─ RGO output
   │  ├─ building output
   │  ├─ imports with reduced impact
   │  └─ stockpile contribution
   ├─ effective_demand
   │  ├─ pop needs
   │  ├─ production method inputs
   │  ├─ building maintenance
   │  ├─ construction demand
   │  ├─ road maintenance/construction
   │  ├─ army/navy construction and maintenance
   │  ├─ exports with reduced impact
   │  └─ temporary/event/special demand
   └─ price_stability
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Demand/supply balance | Critical for price | Mixed | Moves target price, which feeds RGO/building/trade profitability. |
| Pop needs | High | Exact/Mixed | Population, development, pop type, and wealth thresholds can create persistent demand. |
| Production inputs | High | Exact/Mixed | Industrial chains create recurring demand and can make upstream goods valuable. |
| Construction and maintenance | Medium/High | Exact/Mixed | Can spike or sustain demand for construction and upkeep goods. |
| Export demand | Conditional/High | Mixed | Exports can raise demand pressure and prices in the source market. |
| Price stability | Medium | Mixed | Dampens or magnifies how far target price can move from base price. |

Demand categories:

| Category | Typical source |
|---|---|
| `pop_needs` | Pop life needs |
| `building_construction` | Constructing buildings |
| `building_maintenance` | Building and production method upkeep/input demand |
| `guild_input`, `workshop_input`, `manufactory_input`, `mills_input` | Production input families |
| `regiment_construction`, `regiment_maintenance` | Army creation/upkeep |
| `ship_construction`, `ship_maintenance` | Navy creation/upkeep |
| `government_activities` | Government demand |
| `special_demands` | Hardcoded, event, or situation demand |

Player rule: prices are not simply "good if high." High prices help producers of that good and hurt consumers of that good. In a production chain, decide whether you are optimizing the upstream seller, the downstream user, or total state income.

### Market Access Subgraph

[Back to top graph](#dependency-graph)

```text
market_access
├─ distance to market center
├─ roads
│  ├─ road network pathing
│  ├─ market access cost modifiers
│  └─ proximity bonuses on better roads
├─ river/sea propagation
├─ maritime presence
├─ harbor capacity
├─ nearby market creation/relocation
└─ local geography and infrastructure
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Local market access | High | Mixed | Affects building output, RGO profit, market attraction, and purchase order. |
| Roads | High | Exact/Mixed | Better roads add proximity and improve routing; this supports control and access. |
| Market center placement | High | Mixed | Creating/relocating markets can shorten distance and raise local access. |
| Maritime presence and ports | Conditional/High | Mixed | Strong in coastal and island economies; also helps control via proximity. |
| Harbor capacity | Conditional | Exact/Mixed | Reduces sea-land transition cost for proximity and trade calculations. |

Installed road examples:

| Road type | Exact values seen |
|---|---|
| `paved_road` | `proximity = 5` |
| `modern_road` | `proximity = 10` |
| `railroad` | `proximity = 15` |

Player rule: access infrastructure multiplies other economic plans. Build roads, ports, and market centers where they unlock valuable production, high-pop urban demand, or control over rich locations.

### Storage And Stockpiles Subgraph

[Back to top graph](#dependency-graph)

```text
market stockpile
├─ rises when supply > demand
├─ falls when demand > supply
├─ helps buildings, constructions, units, and pops fulfill needs
├─ can add extra trade-available supply
├─ bounded by maximum_stockpile_capacity
└─ smooths price and availability shocks
```

Food storage is separate:

```text
food storage
├─ local food production and consumption
├─ provincial food storage
├─ market food buying/selling
├─ starvation prevention
└─ local/global food capacity and decay modifiers
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Stockpile capacity | Medium | Exact/Mixed | Does not create base demand, but lets markets buffer shortages and keep production/trade running. |
| Stockpile drawdown | Medium/High | Mixed | Important during deficits; prevents input shortages from shutting down profitable chains. |
| Food storage | High for stability | Mixed | Protects population and avoids starvation, which protects long-run economic capacity. |
| Warehouse buildings | Medium | Exact/Mixed | `market_warehouse` adds normal-goods stockpile capacity and trade support. |
| Granary buildings | Medium/High | Exact/Mixed | Food storage protects provinces from shortages. |

Player rule: storage is a shock absorber. It usually will not make a poor location rich by itself, but it can protect profitable economies from shortages and price spikes.

### Control Subgraph

[Back to top graph](#dependency-graph)

```text
local_control
├─ current control
├─ max control
│  ├─ proximity to capital
│  ├─ roads
│  ├─ maritime presence
│  ├─ buildings
│  ├─ location rank
│  ├─ laws and privileges
│  └─ terrain and vegetation
└─ monthly movement toward max control
   ├─ local_monthly_control
   ├─ global_monthly_control
   └─ global_monthly_control_decline
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Current `local_control` | Critical | Exact | Direct tax-base multiplier and local Crown Power input. |
| Max control | Critical over time | Exact/Mixed | Sets where current control naturally trends. |
| Proximity to capital | High | Exact/Mixed | Main max-control foundation. Roads and coasts help. |
| Monthly control modifiers | Medium/High | Exact | Important for recovery speed and newly acquired lands. |
| Control buildings | Medium/High | Exact | Strong in selected locations; for example, `bailiff` gives `local_max_control = 0.2`. |
| Terrain/vegetation | Medium | Exact/Mixed | Can hurt proximity and infrastructure efficiency. |
| Average country control | High | Exact/Mixed | Around the 50% midpoint, changes `crown_power_from_population` by `0.5 * (average_control - 0.50)`, affecting national Crown Power. |

Installed exact examples:

| Source | Values |
|---|---|
| `location_ranks/00_default.txt` | town `local_max_control = 0.05`, city `0.1`, megalopolis `0.2` |
| `building_types/rural_buildings.txt` | `bailiff` gives `local_max_control = 0.2` |
| `road_types/00_generic.txt` | paved road `proximity = 5`, modern road `10`, railroad `15` |
| `auto_modifiers/country.txt` | `global_monthly_control_decline = 0.005` |
| `static_modifiers/location.txt` | `inverse_control` is multiplied by `1 - control` and applies `local_crown_estate_power = -1.0`, `local_levy_size_modifier = -1.0`, `local_manpower_modifier = -1.0`, and `local_sailors_modifier = -1.0`. |
| `static_modifiers/country.txt` | `average_control_50` applies `crown_power_from_population = 0.5` around the 50% average-control midpoint. |

Current-build control equations:

```text
location_tax_base =
    location_wealth * local_control

local_crown_power_from_control =
    -1.0 * (1 - local_control)

crown_power_from_population_from_average_control =
    0.5 * (average_country_control - 0.50)
```

Player rule: control investment has compound returns. Raising control in a valuable, populous location does not just increase tax base; it also reduces local Crown Power penalties and can improve national Crown Power from population through the country average. This is why low-control expansion can hurt the treasury even when the conquered population is large.

For poor, sparsely populated locations, build wealth or strategic infrastructure first unless control has non-economic goals.

### Estate Allocation And Tax Extraction Subgraph

[Back to top graph](#dependency-graph)

```text
treasury tax income
├─ location_tax_base
├─ estate wealth share
│  ├─ local estate pop counts
│  ├─ estate tax_per_pop weights
│  ├─ local/global estate power modifiers
│  ├─ pop type / estate power influence
│  ├─ peasant enfranchisement transfer to Nobles
│  └─ local Crown Power / location Crown estate power
├─ estate tax rate
│  ├─ current tax rate
│  ├─ min tax cap
│  └─ max tax cap
├─ tax_income_efficiency
├─ estate enrichment
└─ estate power side effects
   ├─ high/low estate power changes max tax
   ├─ Crown Power affects trade-income share
   ├─ average control changes Crown Power from Population
   └─ privileges/laws/reforms alter caps and efficiency
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Estate share | High | Exact/Derived | Determines which estate owns the taxable base. |
| Non-Crown local estate power | High/Conditional | Exact/Mixed | Shifts local tax-base allocation toward that estate; treasury effect depends on that estate's tax rate and caps. |
| Peasant enfranchisement | Medium/High in peasant-heavy land | Exact/Mixed | Higher values let Peasants keep more of their share; lower values transfer more to Nobles. |
| Local Crown Power | Medium/High in rich locations | Exact/Mixed | Location-scope extraction lever; does not create wealth, and does not act like a Crown pop tax share. |
| National Crown Power | High for trade/state extraction | Exact/Mixed | Country-scope lever that affects Crown/estate pressure, national Crown effects, and the treasury share of trade income. |
| Tax rate and max tax | High | Exact/Mixed | The state cannot extract above caps, even if wealth exists. |
| `tax_income_efficiency` | Medium/High | Exact | Multiplies extracted income once tax base and rate exist. |
| Estate enrichment | Medium | Mixed | Reduces estate tax-base income; exact order is engine-side. |
| Estate power | High/Conditional | Exact/Mixed | Can unlock bonuses while lowering max tax or reducing Crown Power. |

Installed tax cap examples:

| Source | Values |
|---|---|
| `auto_modifiers/country.txt` | `global_estate_max_tax = 0.35`, `global_estate_min_tax = 0.0` |
| Crown estate power block | `global_estate_max_tax = 0.20` |
| Nobles high power | `nobles_estate_max_tax = -1.0` scaled by high-power amount |
| Burghers high power | `burghers_estate_max_tax = -0.5` scaled by high-power amount |

Player rule: a tax-base increase only becomes treasury income if the estate holding that share can be taxed. Estate management is part of economic optimization, not a separate political minigame.

#### Estate Power

Estate Power is an estate's **share of the country's total Political Power** — its clout in government. The concept text is explicit: Political Power "is used to calculate `estate_power`," each pop "exerts a different amount of Political Power dependent on the estate they belong to," and "the total power of the estates reduces the `crown_power` of the country." So estate power is a *relative share*: raising one estate's power lowers everyone else's, and Crown Power is what remains after the estates take their slices.

```text
estate_power_e =
    ( SUM over locations [ SUM over pop types (pop_count / 1000 * power_per_pop_e) ]   # power_per_pop is per 1,000 pops
      * (1 + local_modifier_e) )        # local_<estate>_estate_power, applied per location
    * (1 + national_modifier_e)         # global_<estate>_estate_power, crown_power_from_population, ...

# then shown as a share of the whole pie:
estate_power_share_e =
    estate_power_e / total_power(all estates + Crown)
```

Two consequences follow from the `SUM over locations`:

- Estate power is **national, but built from locations.** Each location contributes `population × power_per_pop`, scaled by that location's `local_<estate>_estate_power`. So local estate-power modifiers (Local Crown Power included) **roll up into the national total**, weighted by how populous the location is — see [Local Crown Power From Buildings](#local-crown-power-from-buildings).
- It is a **zero-sum share.** The in-game Estates hint calls it "a zero-sum distribution" between the estates and the Crown: raising one estate's power lowers everyone else's, and Crown Power is simply the Crown's slice of the same pie.

`power_per_pop` is the dominant input, and it is **separate from the `tax_per_pop`** weight used for the wealth/tax-base split — an estate can be wealthy but politically weak (Burghers: tax 40, power 4) or poor but influential (Cossacks: tax 0.02, power 0.5). `power_per_pop` is counted **per 1,000 pops**, so a Nobles pop (`25` → `0.025` each) wields about 1,000× the political power of a Peasants pop (`0.025` → `0.000025` each). Characters an estate places in the cabinet or in army/navy command add power too — the worked example below shows two cabinet members at `+12.50%` each (underlying defines `BASE_ESTATE_POWER_FROM_CABINET` / `_FROM_COMMAND = 0.25`). See the [per-pop weights table](#estate-shares-and-tax-income) for all estates.

Installed constants (`loading_screen/common/defines/00_defines.txt`):

| Define | Value | Meaning |
|---|---:|---|
| `BASE_ESTATE_POWER_FROM_CABINET` | `0.25` | Power an estate gains per character it has appointed to the cabinet. |
| `BASE_ESTATE_POWER_FROM_COMMAND` | `0.25` | Power an estate gains per character it has commanding an army or navy. |
| `LOW_POWER_THRESHOLD` | `0.25` | The 25% pivot. An estate's `high_power` effects scale with `(power - 0.25)` when above it; its `low_power` effects scale with `(0.25 - power)` when below it. |

The Crown is special: it has `power_per_pop = 0`, so it gains nothing from ordinary pops. Crown Power instead comes from `crown_power_from_population` (base `1.0`, modified by average control via `average_control_50`), `base_crown_estate_power_modifier`, `global_crown_estate_power`, cabinet/command characters, and privileges/laws/advances — see [National Crown Power](#national-crown-power).

Estate power is an *input* to several systems, not just a status bar:

| Consequence | Mechanism |
|---|---|
| Tax-base split | Each location's tax base is split among estates by **population weight** (`tax_per_pop`), *not* by estate power — see [Where the Money Goes](#where-the-money-goes). |
| Trade-income split | Trade income is divided between Crown and estates by estate power; only the Crown Power share reaches the treasury. |
| Max-tax effects | Above the 0.25 threshold an estate's `high_power` block applies (e.g., Nobles `nobles_estate_max_tax = -1.0`, scaled by how far over the threshold they sit); below it the `low_power` block raises max tax (e.g., `+0.5`). |
| Crown penalties | Too much total estate power — usually from over-granting privileges — starves Crown Power, and very low Crown Power triggers country-wide penalties. |
| Estate satisfaction & culture | Primary/accepted culture and matching state religion raise an estate's power and satisfaction; heretic/heathen religion or foreign culture lower both. |

**Worked example (in-game tooltips).** Two Venetian Nobles tooltips show the whole chain.

*Per-location base* (Nobles in Cioxa) states the per-pop rule outright — "Base Political Power of **134 Nobles** (`+25.00 for every 1,000 Pops`): **3.36**," i.e. `134 × 25/1000 = 3.36`. That base is then multiplied by a stack of additive percentage modifiers: estate-wide privileges (`Land Rights +75%`, `Fortification Licenses +75%`, `Feudal Mercenary Contracts +100%`, `Avogadoria de Comùn +50%`), location effects (`Noble Navy +33%`, `Plenty of Opportunities −13.53%`, `Council of Ten −10%`), and **each cabinet character the estate holds** (`Lazzaro Dandolo +12.50%`, `Nicolò Gradenigo +12.50%`).

*National total* (the Patriziato) sums every location's contribution and reports the share: **231.70** of **628.38** total Political Power = **36.87%**, exactly the displayed Estate Power — so the share is a plain own-over-total ratio.

*Threshold effects* use that **national** 36.87% share — the Cioxa tooltip itself labels them "Estate Power of the Patriziato is above 25.00%." Scaled by `(0.3687 − 0.25) = 0.1187`, every line matches to the decimal:

- `nobles_estate_max_tax = -1.0 × 0.1187 =` **−11.87%** maximum tax (the estate resists taxation)
- `levy_combat_efficiency_modifier = +1.0 × 0.1187 =` **+11.87%**
- `fort_maintenance_efficiency = +1.0 × 0.1187 =` **+11.87%**

Evidence level: per-pop weights and the cabinet/command/threshold constants are `Exact` from `estates/00_default.txt` and `00_defines.txt`. The share normalization (own ÷ total Political Power) and the high/low-power `(relative_power − 0.25)` scaling are `Exact`, confirmed to the decimal against the in-game Estate Power tooltip (worked example above). The summation-over-locations form matches the in-game Estates hint and the community wiki. Only the pop-count → base-power scaling factor and the exact ordering of the percentage modifiers remain engine-side (`Mixed`).

#### Where the Money Goes

A common misconception is that all of a location's money is handed out in proportion to estate power. It is not — **two different distributions run in parallel, and only one of them uses estate power.**

**1. Tax base (a location's wealth) → split among estates by population, then taxed.** The in-game concept is explicit: a location's wealth "is distributed among the estates according to their **relative population**... Nobles get a much bigger share per capita than Burghers, which in turn get a much bigger share per capita than Peasants." That per-capita weight is `tax_per_pop`, *not* estate power. The Crown (`tax_per_pop = 0`) takes **none** of it — the estates hold the entire tax base. The treasury then collects income only by **taxing** each estate's share:

```text
estate_tax_income_e =
    estate_wealth_share_e          # population * tax_per_pop, after peasant enfranchisement
  * effective_tax_rate_e           # bounded by that estate's min / max tax
  * (1 + tax_income_efficiency)
  - estate_enrichment
```

As the Estates hint puts it: *"The Estates collect all the tax base from our locations, and we only get as much as we tax out of them."* Estate **power** still matters here, but indirectly — a high-power estate has a lower **max tax**, so you cannot tax its share as hard.

**2. Trade and food profit/cost → split between Crown and estates by estate power.** This is the distribution that *is* proportional to estate power. The Crown's percentage — i.e. **Crown Power** — goes straight to the treasury; the rest is divided among the estates by their estate power. Per the crown-power hint: *"At 50% Crown Power, half of the expenses and profits for trade and food will go directly into the treasury, while the other half will be distributed among the estates, according to their respective estate power."*

So, directly: **money from a location is not simply handed out in proportion to estate power.** Tax base is split by *population weight* and must be *taxed* out of the estates (with estate power setting the tax ceiling); only trade and food income is split *directly* by estate power, with the Crown's share reaching the treasury automatically. Note also that a location at `0%` control generates **no taxes at all**, even though the estates still hold and benefit from its wealth.

#### Peasant Enfranchisement

Peasant enfranchisement is a redirect *within* the wealth-share distribution above: it sets how much of the **Peasants' own share** the Peasants keep, sending the rest to the Nobles. The Peasants estate has `disenfranchise_to = nobles_estate`, and the tooltip says peasants "only receive that percentage of their wealth share. The rest of their income is transferred to the Nobles." It depends mostly on the free-subjects societal value; towns and cities enfranchise more than rural locations, and some buildings shift it.

```text
peasant_enfranchisement_final = Location.GetPeasantEnfranchisment

peasants_kept_income =
    peasant_wealth_share * peasant_enfranchisement_final

nobles_received_from_peasants =
    peasant_wealth_share * (1 - peasant_enfranchisement_final)
```

Strategic meaning:

- Higher peasant enfranchisement weakens the noble siphon on peasant-heavy locations.
- Lower peasant enfranchisement strengthens Nobles, which can be bad for treasury income if Nobles are hard to tax or their power is creating max-tax penalties.
- It is not automatically a pure tax increase. If Peasants have low tax rates, bad caps, or low satisfaction, shifting share from Nobles to Peasants may not immediately raise treasury income.

Installed examples:

| Source | Values |
|---|---|
| Location rank | megalopolis `local_peasant_enfranchisment = 0.5`, city `0.25`, town `0.1`, rural settlement `-0.2` |
| Rural buildings | bailiff `local_peasant_enfranchisment = -0.1`; fishing village and several rural producers `0.01` |
| Societal values | free-subjects direction can add `global_peasant_enfranchisment = 0.33`; opposite direction can apply `-0.33` |

#### Local Crown Power From Buildings

The building modifier is `local_crown_estate_power`, localized as Local Crown Power. In the [estate-power formula](#estate-power) it is the location's `local_modifier` for the Crown — it scales how much that one location's population counts toward Crown Power.

```text
local_crown_estate_power in a location
  -> that location's population counts for more Crown power
  -> because national Crown Power SUMS all locations, the national figure rises too
  -> the gain is proportional to the location's population and pop value
```

**Does it feed national Crown Power? Yes — because they are the same number.** There are not separate "local" and "national" Crown Powers. There is one national Crown Power, and it is *built by adding up a contribution from every location*. "Local Crown Power" is simply one location's input into that sum, so raising it raises the total directly. Step by step, for a building granting `local_crown_estate_power = +0.20` in a city:

1. **National Crown Power is a share** — the Crown's political power ÷ the total political power of all estates + Crown (e.g. the tooltip's `2816.54 / 7286.74 = 38.65%`).
2. **The Crown's political power is a sum over locations** — each location adds `(its population) × crown_power_from_population × (1 + that location's local crown power)`. The national tooltip shows the already-summed result.
3. **The building raises one location's local crown power** by `+0.20`, so that city's term in the sum grows by ~20%.
4. **The gain is proportional to that location's population** — +20% of a big city's contribution is a lot of political power; +20% of an empty village's is almost nothing. (A Nobles pop is worth ~1,000× a Peasants pop, so high-value pops matter too.)
5. **A bigger Crown sum = a bigger Crown share** — the added political power goes into the Crown's numerator while the estates' power is unchanged, so the Crown's fraction rises and every estate's drops by the same total (zero-sum).
6. **A bigger Crown share scales up every Crown effect** — more trade income to the treasury, higher estate max tax, cheaper privileges/policies, better cabinet and parliament — all linearly (see [National Crown Power](#national-crown-power)).

*Numeric sketch.* Country total political power `1000`, Crown holds `380` → Crown Power `38%`. A city contributes `40` of that (its pops, no crown bonus yet). Build a council hall there (`+0.20`): the city's contribution rises to `48` (`+8`). Crown → `388`, total → `1008`, so Crown Power = `388 / 1008 = 38.5%`, and the estates' combined share falls from `62%` to `61.5%`. Build the same hall in a hamlet contributing only `2`, and the `+0.4` it adds is a rounding error. **Same building, very different result — population is the multiplier.**

**How does it affect tax?** Not by giving the Crown a tax-base share — the Crown holds none ([Where the Money Goes](#where-the-money-goes)). It helps the treasury through three indirect channels:

- **Trade and food split:** higher Crown Power sends a larger share of trade/food profit straight to the treasury instead of to the estates.
- **Higher tax ceilings:** Crown Power and estate power are zero-sum, so more Crown Power means less estate power, which **raises the max tax** you can levy on the estates that actually hold the wealth.
- **Governance:** easier laws, better cabinet efficiency, and more parliament base support.

So treat it as an **extraction and centralization lever, not a growth lever** — it does not raise wealth, prices, output, market access, or control on its own.

Control pushes the *other* way through the `inverse_control` static modifier, which applies `local_crown_estate_power = -1.0` scaled by `(1 - local_control)`:

```text
local_crown_power_from_control = -1.0 * (1 - local_control)
```

So a building granting `local_crown_estate_power = 0.25` in a `60%`-control location must first offset roughly `-40%` Local Crown Power from low control. This is also why **raising control in your most populous locations is itself one of the strongest national Crown Power levers** — the crown-power hint names "the control of our most populated locations" as a primary driver.

Installed building examples:

| Building group | Values |
|---|---|
| `council_hall.txt` | early council hall `local_crown_estate_power = 0.10`; council hall `0.2`; modern council hall `0.5` |
| `town_buildings.txt` | counting house `0.33`; minting office `0.25` plus `local_max_control = 0.05` |
| Some estate/fort/trade-company buildings | negative values such as `-0.1` or `-0.2`, meaning less local Crown Power |

#### National Crown Power

National Crown Power is the Crown's slice of the zero-sum estate-power pie — which, by the [estate-power formula](#estate-power), is the **sum over every location** of the Crown's population contribution, scaled by local and national modifiers. So it is built from population, the `local_crown_estate_power` in each location (including the `inverse_control` penalty for low control), the privileges granted to estates (which raise *their* power and so lower the Crown's), and the country-scope modifiers below:

```text
global_crown_estate_power
  -> Crown estate power / Crown Power in the entire country

crown_power_from_population
  -> how much Crown estate power the Crown gets for each pop in the entire country

average_control_50
  -> modifies crown_power_from_population by 0.5 * (average_control - 0.50)

base_crown_estate_power_modifier
  -> modifier to base Crown political power in the entire country
```

In the current build, the national average-control path is:

```text
effective_crown_power_from_population =
    1.0
  + 0.5 * (average_country_control - 0.50)
  + other_crown_power_from_population_modifiers
```

Examples:

| Average country control | Modifier to Crown Power from Population |
|---:|---:|
| `0%` | `-25%` |
| `28.36%` | about `-10.82%` |
| `50%` | `0%` |
| `100%` | `+25%` |

The in-game "Crown Power from Population" line opens its own sub-breakdown that confirms this composition:

- **Base Value for Every Country `+100%`** — the `crown_power_from_population = 1.0` base every country starts with.
- **Average Control in Country** — the `average_control_50` term, `0.5 × (avg_control − 0.50)`. (One save shows `+7.15%` here, implying ~`64.3%` average control; below 50% it goes negative — e.g. the `−26.89%`-net low-control example below.)
- **Economic Base of Subjects** — a small adjustment from the economic base of subjects.

Difficulty, age advances (e.g. `Statecraft`, `Sovereignty`), laws (e.g. `Zonta`), and the ruler's administrative ability then scale the Crown's population power further.

National Crown Power is not a direct production stat either. Its income effect is mainly that it changes how much estate power pushes back against the state, what Crown-power-scaled effects apply, and how much normal trade income reaches the treasury:

```text
normal_trade_profit
  -> trade_income
  -> split by estate_power
  -> treasury receives Crown Power percentage
```

**Worked example (in-game tooltip).** The Venetian Republic (the Crown estate) shows Crown Power forming and paying out. Its share is `79.10 / 628.38 = 12.58%` — the same own-over-total ratio as any estate, against the same `628.38` country total seen in the Patriziato tooltip.

Because the Crown has `power_per_pop = 0`, its power comes from **population via `crown_power_from_population`**: `365K` pops give a base of `74.82`, then modified by `Crown Power from Population −26.89%` (the low-average-control penalty through `average_control_50`, plus laws), `Ruler's Administrative Ability +18%`, the `Doge in command +16.66%`, difficulty (`Very Easy +25%`), and reforms (`Zonta`, `Council of Forty`, `Traditional Distribution`) — netting `79.10`.

The payoff is the key part: unlike estates (which use `high_power`/`low_power` around the `0.25` threshold), the `crown_estate` has a single `power` block whose every line is multiplied **linearly by the Crown's share**. At `12.58%`, each line matches the tooltip to the decimal:

| `crown_estate.power` line | Result = base × 0.1258 |
|---|---|
| `trade_income = 1.0` | **+12.58%** trade income to treasury |
| `revoke_privilege_cost_modifier = -1.0` | **−12.58%** |
| `country_cabinet_efficiency = 0.5` | **+6.29%** |
| `parliament_base_support = 0.5` | **+6.29%** |
| `change_policy_cost_modifier = -0.5` | **−6.29%** |
| `estate_building_destruction_satisfaction_impact = -0.5` | **−6.29%** |
| `global_estate_max_tax = 0.20` | **+2.51%** |
| `power_projection = 5` | **+0.62** |
| `diplomatic_reputation = 2` | **+0.25** |
| `remove_bureaucracy_price_cost_modifier = -0.25` | **−3.14%** |
| `maintain_bureaucracy_price_cost_modifier = -0.05` | **−0.62%** |

This is the exact mechanism behind the trade/food split: `trade_income = 1.0` means **treasury trade share = Crown Power** (here 12.58%; the hint's "50% Crown Power → half of trade" is the same line at a higher share). It is also why more Crown Power raises max tax (`global_estate_max_tax = 0.20 × Crown Power`), cheapens privilege revocation and policy changes, and lifts cabinet efficiency and parliament support — all scaling straight off the Crown's share. (Confirmed again in a separate save at a `38.65%` Crown share: `trade_income → +38.65%`, `global_estate_max_tax → +7.73%`, `parliament_base_support → +19.32%`, and so on — every line still matched, so the linear rule is general, not a one-save coincidence.)

This is why a local Crown Power building and a national Crown Power law help at *different points of the same sum*. The building raises the Crown's contribution from one location (best where pops are many and valuable); the national modifier lifts every location's contribution at once. Both land in the same national Crown Power figure, which governs the trade/food treasury split and estate max-tax ceilings even when income comes from trade rather than local tax base.

Current-build caveat: control now reaches Crown Power through two visible paths. Locally, low control applies an `inverse_control` penalty to Local Crown Power. Nationally, average control modifies the Crown's population-based power. The second path is not shown as a redistribution to a specific rival estate; it changes the Crown's population multiplier. For strategy, treat this as a compound penalty for low-control population rather than as a simple zero-sum estate transfer.

For non-Crown estates, raising their power adds to *their* slice of the same zero-sum pie. It can tilt the local wealth share toward that estate, but its larger and more certain effects are on **taxability and the trade/food split**:

```text
more local_<non_crown_estate>_estate_power
  -> that estate holds more power locally, and (summed) nationally
  -> its max tax falls (powerful estates resist taxation); its trade/food/parliament weight rises
  -> usually a net negative for the treasury unless you can still extract from it
```

Country-level estate power also matters because total estate power reduces Crown Power. Lower Crown Power reduces the treasury share of normal trade income, since installed concepts say only the Crown Power share of trade income reaches the treasury. Crown-specific local power is therefore best read as a local extraction and centralization lever, national Crown Power as a country-wide state-capacity lever, and non-Crown local estate power as a local taxable-share lever.

### Trade And Naval Economy Subgraph

[Back to top graph](#dependency-graph)

```text
trade and naval economy
├─ normal player trade routes
│  ├─ manual, trade-order, or automatic route origin
│  ├─ trade route profit
│  │  ├─ sell value
│  │  ├─ buy cost
│  │  ├─ merchant maintenance
│  │  └─ sound tolls, if any
│  ├─ profit per merchant capacity
│  ├─ assigned merchant capacity
│  ├─ trade_income modifier
│  └─ Crown Power / estate split
├─ burgher trade
│  ├─ burgher population
│  ├─ local/global trades per burgher
│  ├─ market burgher trade capacity
│  ├─ goods needs in burgher town/city markets
│  └─ location wealth contribution
├─ indirect tax-base effects
│  ├─ market prices
│  ├─ input availability
│  ├─ export demand
│  ├─ burgher trade wealth
│  └─ building/RGO profitability
├─ route feasibility and cost
│  ├─ trade range
│  ├─ route distance
│  ├─ merchant capacity
│  ├─ merchant maintenance
│  ├─ sound tolls
│  ├─ trade land/sea efficiency
│  └─ harbor capacity
├─ route priority
│  ├─ merchant power / Trade Advantage
│  ├─ merchant power from maritime presence
│  └─ building/law/reform/privilege modifiers
└─ maritime support
   ├─ ports
   ├─ fleets in sea zones
   ├─ blockades
   ├─ maritime presence decay
   └─ local/global maritime presence modifiers
```

Relative weights:

| Dependency | Weight | Evidence | How it affects strategy |
|---|---|---|---|
| Route price spread | Critical for trade route profit | Mixed | Buy low, sell high; route profit starts with market price difference. |
| Merchant capacity | High | Mixed | More capacity lets more volume move. |
| Merchant power / Trade Advantage | Conditional/High | Mixed | Critical when supply is contested. |
| Trade range | Conditional/High | Mixed | Expands profitable long-distance trade by reducing over-range penalty. |
| Crown Power / trade-income share | High for treasury | Exact/Mixed | Trade profit is not all treasury income; estate split matters. |
| Burgher trade capacity | Conditional/High | Exact/Mixed | Separate from normal merchant capacity; important in urban markets with many Burghers. |
| Maritime presence | Conditional/High | Mixed | Important for coastal control, merchant power, and sea-connected trade. |
| Harbor capacity | Conditional | Exact/Mixed | Reduces sea-land transition cost in proximity and trade calculations. |

The route-profit, capacity, distance, and trade-income equations — with worked examples — are in [Trade Profit And Income](#trade-profit-and-income). The UI exposes them as `Trade.GetSell/GetBuy/GetMaintenance/GetSoundTollFee/GetProfit` (and `PossibleTrade.*` for a route you are considering); those buy/sell values already fold in market prices, quantities, efficiencies, and route conditions.

Normal trade and Burgher trade share the market, not the same control surface:

```text
shared market dependencies
├─ market prices
├─ goods supply and demand
├─ stockpiles and fulfillment
├─ export/import pressure
├─ export_impact_on_demand
├─ ports, sea routes, and trade geography
└─ downstream effect on RGO/building profitability

normal player trade route only
├─ route creation/order/manual control
├─ merchant_capacity used
├─ merchant_power / Trade Advantage competition
├─ trade route buy/sell/maintenance/sound toll profit
└─ direct trade_income extraction

burgher trade only
├─ Burgher pop presence
├─ local_trades_per_burgher
├─ global_trades_per_burgher
├─ market Burgher trade capacity
└─ location wealth contribution before control/tax extraction
```

Trade UI parameter map:

| UI label | Script key / concept | Economic role |
|---|---|---|
| Selling Efficiency | `selling_efficiency` | Raises sale performance in target market. |
| Export Efficiency | `export_efficiency` | Improves export-side trade performance. |
| Import Efficiency | `import_efficiency` | Improves import-side cost/performance. |
| Trade Advantage | `merchant_power`, `global_merchant_power` | Priority for contested export fulfillment. |
| Merchant Maintenance Efficiency | `merchant_maintenance_efficiency` | Reduces route maintenance burden when favorable. |
| Trade Capacity | `merchant_capacity`, `global_merchant_capacity_modifier` | Allows more trade volume in a market. |
| Trade Income Share | `trade_income` | Treasury extraction from trade income, affected by estate/Crown dynamics. |
| Market Attraction | `global_trade_center_power` | Pulls locations into your market. |
| Market Protection | `global_trade_protection_factor` | Resists foreign market attraction. |
| Trade Range | `trade_range`, `trade_range_modifier` | Distance before extra transport penalties. |
| Production Efficiency | `global_production_efficiency` | Improves building profit, feeding wealth. |
| Burgher Trade Capacity | `local_trades_per_burgher`, `global_trades_per_burgher`, `Market.GetBurgherTradeCapacity` | Separate autonomous Burgher trade capacity; affects Burgher trade wealth and market demand/price behavior. |

Player rule: trade capacity is often good even where control is low because trade income does not scale down by local control the way tax base does. The indirect price effects still depend on the market and production network.

## Parameter Dictionary

This is the minimum vocabulary for reading economic modifiers in files and tooltips.

| Parameter / concept | Guide meaning | First place to inspect |
|---|---|---|
| `wealth` / `Location.GetTotalPossibleTaxBase` | Pre-control source wealth. | English game concepts, location UI. |
| `location_tax_base` / `Location.GetTotalTaxBase` | Wealth after local control. | Location UI and script triggers. |
| `country_tax_base` | Sum of owned province/location tax base. | Country UI and script values. |
| `local_control` | Current control multiplier. | Location UI, missions, triggers. |
| `local_max_control` | Local max-control modifier/cap support. | location ranks, buildings, town rights. |
| `global_monthly_control` | Country-wide monthly control movement. | laws, advances, reforms, privileges. |
| `proximity` | Capital network value feeding max control. | roads, topography, vegetation, concepts. |
| `inverse_control` | Location static modifier scaled by `1 - local_control`; applies Local Crown Power, levy, manpower, and sailor penalties in the current build. | `main_menu/common/static_modifiers/location.txt`. |
| `average_control_50` | Country static modifier scaled around the 50% average-control midpoint; modifies `crown_power_from_population`, complacency, and research speed. | `main_menu/common/static_modifiers/country.txt`. |
| `market_access` | Location's access to its market; affects output/profit/purchasing. | market UI, road/geography systems, building caps. |
| `default_market_price` | Baseline good price. | `goods/*.txt`. |
| `transport_cost` | Good transport costliness. | `goods/*.txt`. |
| `demand_add`, `demand_multiply` | Good demand additions/multipliers. | `goods/*.txt`. |
| `production_methods` | Input/output recipe definitions. | `production_methods/*.txt`. |
| `construction_demand` | Goods needed to build. | building types and goods demand files. |
| `maintenance_demand` | Goods needed to maintain buildings/roads/units. | building types, road types, goods demand files. |
| `pop_type` on buildings | The single pop type a building can employ. | `building_types/*.txt`; building concept text. |
| `employment_size` | Maximum supported employees by building type/level. | building types and building trigger localization. |
| `building_employed_amount` | UI/trigger-backed current employee amount. | `trigger_localization/building_triggers.txt`, building UI. |
| `building_employment_size_amount` | UI/trigger-backed employee capacity amount. | `trigger_localization/building_triggers.txt`, building UI. |
| `building_is_full_capacity` | Whether a building is working at full capacity. | building trigger localization and English trigger text. |
| `location_unemployed_population_for_building_type` | Available local population for a given building type. | location attribute columns and triggers. |
| `Location.GetExpectedUnEmployed` / `Location.GetEffectiveUnEmployed` | Build tooltip values for expected/current available workers. | `building_tooltips.gui`. |
| `Location.GetCivilConstructionsNeededPops` | Build tooltip value for employees already demanded by queued constructions. | `building_tooltips.gui`. |
| `Location.GetPromote` | Local promotion speed shown in building-missing-employees alerts. | alerts and construction tooltip UI. |
| `global_pop_promotion_speed`, `global_pop_promotion_speed_modifier` | Country-wide promotion speed values shown by urbanization action tooltip. | government localization, modifiers, laws/reforms. |
| `global_production_efficiency` | Country-wide production profit/output support. | laws, reforms, privileges, advances. |
| `local_production_efficiency` | Location-specific production support. | buildings, pop types, local modifiers. |
| `ESTABLISHMENT_SYSTEM_ENABLED` | Installed define controlling whether building establishment throughput/bonus is active. Current checked value is `no`. | `loading_screen/common/defines/00_defines.txt`. |
| `local_building_establishment_speed`, `global_building_establishment_speed` | Establishment speed modifiers; only income-relevant if establishment is enabled. | modifier definitions, localization, laws, privileges, societal values. |
| `tax_per_pop` | Estate weight for local wealth share. | `estates/00_default.txt`. |
| `power_per_pop` | Estate political-power weight per pop (drives [estate power](#estate-power)); separate from `tax_per_pop`. | `estates/00_default.txt`. |
| `estate_power` | An estate's share of total country Political Power; input to tax-base split, trade-income split, max-tax effects, and Crown Power. | estate UI, `estate_power(estate_type:X)` triggers, game concepts. |
| `BASE_ESTATE_POWER_FROM_CABINET` / `BASE_ESTATE_POWER_FROM_COMMAND` | Estate power per character in the cabinet / commanding units; both `0.25` in this build. | `loading_screen/common/defines/00_defines.txt`. |
| `LOW_POWER_THRESHOLD` | The 25% pivot that decides whether an estate's `high_power` or `low_power` effects apply and how strongly. | `loading_screen/common/defines/00_defines.txt`. |
| `local_<estate>_estate_power` | Location modifier that shifts local estate power; for non-Crown estates this can shift local tax-base allocation. | buildings, town rights, local modifiers. |
| `global_<estate>_estate_power` | Country modifier that shifts estate power more broadly. | laws, privileges, reforms, modifiers. |
| `local_crown_estate_power` | Location modifier localized as Local Crown Power; affects Crown/government estate power in that location only. | council halls, town buildings, forts, estate buildings, town rights. |
| `global_crown_estate_power` | Country-wide Crown Power / Crown estate power modifier; affects the entire country rather than one location. | laws, reforms, advances, privileges, unique buildings. |
| `crown_power_from_population` | Country modifier for how much estate power the Crown gets from each pop in the entire country; current build starts at `1.0` and average control modifies it through `average_control_50`. | auto modifiers, static modifiers, modifier definitions. |
| `base_crown_estate_power_modifier` | Country modifier to the base political power behind national Crown Power. | modifier definitions; check laws/reforms/advances for current usage. |
| `local_peasant_enfranchisment` | Location modifier for the displayed peasant enfranchisement value. | location ranks, rural buildings, estate buildings, town rights. |
| `global_peasant_enfranchisment` | Country-wide modifier for peasant enfranchisement. | societal values, laws, reforms, privileges, advances. |
| `Location.GetPeasantEnfranchisment` | UI-exposed final peasant enfranchisement used for wealth-share transfer. | government localization and UI binding. |
| `tax_income_efficiency` | Treasury tax-income multiplier. | laws, privileges, reforms, advances. |
| `global_estate_max_tax` | General max tax cap. | auto modifiers, estate effects, laws. |
| `*_estate_max_tax` | Estate-specific max tax cap. | estates, privileges, laws, reforms. |
| `local_trades_per_burgher` | Local burgher trade activity. | pop types, market/trade buildings. |
| `global_trades_per_burgher` | Country-wide burgher trade activity. | privileges, laws, values. |
| `burgher_trade` | Autonomous Burgher trade that tries to satisfy goods needs in the Burghers' town/city market. | English concepts, market UI. |
| `Market.GetBurgherTradeCapacity` | Market UI value for Burgher trade capacity. | market summary GUI and market tooltip. |
| `export_impact_on_demand` | How much Burgher trade and normal trade demand affect price calculations in owned markets. | modifier localization, laws/reforms/privileges. |
| `merchant_capacity` | Capacity spent by normal player trade routes. | trade route UI, market/trade buildings, modifiers. |
| `global_merchant_capacity_modifier` | Country trade capacity modifier. | laws, reforms, privileges, buildings. |
| `Trade.GetAssignedMerchantCapacityModifiedPlayer` | Normal route capacity used after player modifiers. | trade details GUI. |
| `global_merchant_power` | Trade Advantage modifier. | trade UI, laws, reforms, privileges. |
| `Trade.GetProfit` / `PossibleTrade.GetProfit` | Existing or possible normal trade-route profit. | trade details GUI, possible trade UI. |
| `Trade.GetBuy` / `Trade.GetSell` | Existing route buy cost and sell value. | trade details GUI. |
| `Trade.GetMaintenance` / `Trade.GetSoundTollFee` | Existing route costs subtracted from trade profit. | trade details GUI. |
| `trade_income` | Trade income extraction/share modifier. | estates, reforms, laws. |
| `trade_range` | Free trade range before extra cost. | advances, laws, reforms, privileges. |
| `harbor_suitability` | Port building contribution to harbor capacity. | port buildings. |
| `maximum_stockpile_capacity` | Normal-goods market storage capacity. | market buildings. |
| `local_food_capacity` | Food storage. | granaries, location ranks, unit/support systems. |

For broader parameter discovery, use `EU5_GAMEPLAY_PARAMETERS.md` as the project-level catalog.

## Strategy Checklist

Walk the treasury chain in order — fix the binding constraint before adding anything upstream of it:

1. **Control first where wealth is high.** `tax_base = wealth × control`, so control is a direct multiplier; on rich, populous land it also raises Crown Power and removes the `inverse_control` penalty.
2. **Identify the wealth source** (RGO / building / burgher trade) — the best lever differs per branch.
3. **Check market access** — low access cuts building output, RGO profit, and even building caps.
4. **Check the margin, not the output.** `profit = output − inputs − maintenance`; a high-output building with dear inputs adds little taxable wealth.
5. **Find who holds the tax base.** It goes to estates by population × `tax_per_pop`, *not* by power — then check that estate's **max tax**, which falls as its estate power rises.
6. **Watch Crown Power.** It is your share of trade/food income and it lifts every estate's max tax. Raise it via control of populous locations, Local Crown Power buildings there, and fewer estate privileges.
7. **Compare trade vs. local tax.** Trade income ignores local control, so in low-control regions a route can beat local tax investment — but only the Crown-Power share reaches the treasury.
8. **Protect the chain with storage** against input shortages and price spikes.

## Common Strategic Patterns

### Rich But Uncontrolled Land

- Raise max control (proximity, roads, ports, control buildings); use monthly-control modifiers to get there faster.
- In very low-control regions, release a subject that propagates its own control, then annex later.
- **Why:** control is the direct `tax_base` multiplier *and*, in your populous locations, a primary driver of Crown Power. Low-control conquest can actually *lower* Crown Power through the average-control path.

### Profitable Rural Raw Goods

- Expand RGO size only where the good is valuable, demand is durable, and the location can employ the pops.
- Fix market access before overbuilding; avoid urbanizing your best rural RGOs (some urban ranks cut RGO potential).
- **Why:** RGO wealth is real only at a good *effective* price (after market access), with workers to staff it.

### Industrial Towns And Cities

- Keep input and maintenance goods cheap and available; build upstream supply to prevent bottlenecks.
- Improve production efficiency, staffing, and promotion; use storage to ride out supply breaks.
- **Why:** buildings are the most scalable wealth branch but margin-sensitive — `profit = output − inputs − maintenance`, at the mercy of prices, access, and labor.

### Trade-Focused Economy

- Grow trade capacity where profitable routes exist; raise Trade Advantage where supply is contested; extend trade range for long routes (past range, per-unit transport cost balloons — see [Trade Profit And Income](#trade-profit-and-income)).
- **Keep Crown Power high:** `trade_income_to_treasury = route_profit × trade_income_share`, and that share tracks Crown Power; the rest leaks to the estates.
- **Why:** trade income ignores local control, so it can beat local tax in low-control regions — but the estate split still gates how much you keep.

### Low Crown Power / Estate-Dominated

- Raise Crown Power: control your most populous locations, place Local Crown Power buildings *there* (the gain is population-weighted), and revoke privileges from over-powerful estates when stability allows.
- Watch peasant enfranchisement (low → share flows to Nobles) and each estate's max tax (it falls as that estate's power rises).
- **Why:** Crown Power linearly sets your trade/food treasury share and lifts every estate's max-tax ceiling — both an income and an extraction multiplier.

## Planned Sections

These are intentional placeholders for future editions. Each will be built with the same rule the economy section followed: inspect installed game files first, then cross-check public sources only as secondary context.

### Military

Future topics:

- manpower and sailor generation
- army and navy maintenance demand
- levies, recruitment, unit types, and combat modifiers
- logistics, supply, attrition, and road/naval infrastructure
- war economy effects on production and control

### Diplomacy And Subjects

Future topics:

- subject types and payments
- market access diplomacy
- trade companies and foreign buildings
- diplomatic costs and country interactions
- annexation speed and cultural influence

### Society And Government

Future topics:

- estate power, satisfaction, privileges, and laws
- government reforms and bureaucracies
- societal values and policy tradeoffs
- control, legitimacy, cabinet actions, and parliament systems

### Religion And Culture

Future topics:

- religion modifiers and religious aspects
- culture groups, languages, integration, assimilation, and conversion
- holy sites and religious schools
- culture/religion effects on demand and stability

### Progression And Events

Future topics:

- advances and institutions
- ages and disasters
- missions and scripted events
- on-actions and recurring economic effects
