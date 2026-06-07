# EU5 Guide

This guide is meant to become a broad Europa Universalis V manual over time. The current pass focuses on the economy: wealth, tax base, control, markets, trade, estates, and the parameters a player can use to reason about economic strategy.

Generated from the installed EU5 files at:

`/mnt/d/Program Files/Steam/steamapps/common/Europa Universalis V`

Local build identifiers seen on 2026-06-06:

- `caesar_branch.txt`: `u26q2/release/next`
- `caesar_rev.txt` / `binaries/checksum.txt`: `4be9dd2e34196e66a72855407a347f5235e3bd27c2c34fc63a0e2980cc2e8149eed42774`

Source policy: installed game files are authoritative for this guide. Public sources are useful cross-checks, but they can lag behind the local build. Source links and reusable search paths live in the Codex skill `eu5-game-guide`.

## Economy: Quick Mental Model

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

## Economy: Core Equations

Use UI percentages as decimals in the equations. For example, 75% control is `0.75`.

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

Installed `tax_per_pop` weights:

| Estate | `tax_per_pop` | Strategic meaning |
|---|---:|---|
| Crown | 0 | Crown power affects trade-income split and tax caps, but crown pops do not take local tax-base share. |
| Nobles | 100 | Dominant per-pop claim on wealth; noble-heavy locations shift tax base to nobles. |
| Clergy | 25 | Meaningful tax-base share, below burghers and nobles. |
| Burghers | 40 | Strong urban/commercial claim; often tied to trade and production modifiers. |
| Peasants | 1 | Large numbers can matter, but each pop has low weight. |
| Dhimmi | 1 | Same low per-pop tax-base weight as peasants in this build. |
| Tribes | 0.01 | Very low per-pop taxable share. |
| Cossacks | 0.02 | Very low per-pop taxable share. |

Peasant enfranchisement decides how much of the Peasants estate's wealth share peasants actually keep. The rest is transferred to Nobles:

```text
peasants_kept_share = peasant_share * peasant_enfranchisement_final
nobles_gain_from_peasants = peasant_share * (1 - peasant_enfranchisement_final)
```

Example: if the final displayed peasant enfranchisement is `25%`, Peasants keep `25%` of their Peasants estate share and Nobles receive the other `75%`. Higher enfranchisement shifts income away from Nobles and toward Peasants; lower enfranchisement does the opposite. Whether this raises treasury income depends on which estate can actually be taxed at useful rates and caps.

Local Crown Power from buildings is exposed as `local_crown_estate_power`. It is a location-scope modifier, not the same lever as country-scope `global_crown_estate_power` or `crown_power_from_population`. It does not add wealth, production, price, or control by itself. It raises the Crown estate's local power, so its income effect is an extraction effect through estate/Crown mechanics rather than a production effect:

```text
more local_crown_estate_power in one location
  -> more local Crown estate power
  -> less relative room for non-Crown estate dominance in that location
  -> better state capture only if local allocation, estate taxability, or downstream Crown/estate effects improve
```

National Crown Power uses country-scope modifiers:

```text
effective_crown_power_from_population =
    1.0
  + 0.5 * (average_country_control - 0.50)
  + other_crown_power_from_population_modifiers

national_crown_power_shape ~=
    base Crown contribution
  * (1 + base_crown_estate_power_modifier)
  + population contribution * effective_crown_power_from_population
  + global_crown_estate_power
  - relative pressure from non-Crown estate power
```

The `average_country_control` line is exact from static modifiers: `average_control_50` applies `crown_power_from_population = 0.5` around a 50% control midpoint. Example: `28.36%` average control gives roughly `(0.2836 - 0.50) * 0.5 = -10.82%` Crown Power from Population, matching the observed UI. The full national Crown Power aggregation remains a shape model because the final political-power aggregation/order is engine-side.

Control also applies a local Crown Power penalty through `inverse_control`:

```text
inverse_control_factor = 1 - local_control

local_crown_power_from_control =
    -1.0 * inverse_control_factor
```

Example: `60.54%` local control gives `-39.46%` Local Crown Power. This is separate from the tax-base equation, so control has multiple outputs: taxable capture, local Crown Power, and a national average-control effect on Crown Power from Population.

The practical difference is:

| Lever | Scope | Main income meaning |
|---|---|---|
| `local_crown_estate_power` | One location | Can improve capture of existing local tax base; strongest in rich, estate-dominated locations. |
| `global_crown_estate_power` | Entire country | Raises country-wide Crown Power/Crown estate power; important for national estate pressure, tax caps/effects, and trade-income treasury share. |
| `crown_power_from_population` | Entire country | Changes how much Crown estate power is generated from population across the country. |
| `base_crown_estate_power_modifier` | Entire country | Modifies base Crown Power from Crown-estate political power. |

The visible control penalty is not a direct zero-sum transfer to a named estate. Low control reduces local Crown Power and reduces the national Crown population multiplier, but the static modifiers do not show a matching automatic `local_nobles_estate_power`, `local_burghers_estate_power`, or other estate-power gain from that same control penalty. Non-Crown estates still gain power through their own pops, wealth, privileges, laws, and local modifiers.

The exact local conversion is partly engine-side, and Crown has `tax_per_pop = 0` and `power_per_pop = 0`, so do not model this as the Crown receiving a normal pop-based tax-base share like Nobles or Burghers. Evaluate local Crown Power buildings in rich locations first. A `local_crown_estate_power` building in a poor location has little income to improve; in a rich location it can matter because less estate dominance can mean more money reaching the treasury or fewer estate-power penalties. If the same building also gives `local_max_control`, production, capacity, or trade modifiers, those separate modifiers can create additional income through their own branches.

Evidence level: estate weights are exact from `estates/00_default.txt`; peasant transfer is exact from the peasant-enfranchisement tooltip and Peasants estate `disenfranchise_to = nobles_estate`; local/global Crown Power modifier categories are exact from modifier definitions and localization; the `inverse_control` and `average_control_50` paths are exact from static modifiers; the final Crown/estate political-power aggregation is a strategy model because exact engine ordering is not fully exposed.

Estate enrichment is a reduction layer on estate tax-base income. The concept text confirms it applies, but exact ordering is engine-side, so the guide treats it separately from source wealth.

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

Normal player trade routes expose a buy/sell/cost breakdown in the trade UI:

```text
normal_trade_route_profit =
    sell_value
  - buy_cost
  - merchant_maintenance
  - sound_toll_fee_if_any

profit_per_merchant_capacity =
    normal_trade_route_profit / merchant_capacity_used

displayed_trade_income_from_route =
    profit_per_merchant_capacity * trade_income
```

For a possible route, the UI exposes the same shape before the route exists:

```text
possible_trade_profit =
    possible_sell_price
  - possible_buy_cost
  - possible_maintenance
  - possible_sound_toll_if_any

possible_profit_per_merchant_capacity =
    possible_trade_profit / desired_merchant_capacity
```

Evidence level: mixed. The GUI exposes `Trade.GetBuy`, `Trade.GetSell`, `Trade.GetMaintenance`, `Trade.GetSoundTollFee`, `Trade.GetProfit`, `Trade.GetProfitPerMerchantCapacity`, and `Trade.GetAssignedMerchantCapacityModifiedPlayer`; for possible routes it exposes `PossibleTrade.GetBuyCost`, `PossibleTrade.GetSellPrice`, `PossibleTrade.GetMaintenance`, `PossibleTrade.GetProfit`, and `PossibleTrade.GetDesiredMerchantCapacity`. The exact engine internals behind buy/sell/maintenance are not fully script-exposed.

Burgher trade is not the same route object:

```text
burgher_trade_capacity =
    burgher_population
  * (local_trades_per_burgher + global_trades_per_burgher + other_burgher_trade_modifiers)
```

```text
burgher_trade_wealth_gain ~= engine_result_of_burgher_trades_that_satisfy_market_needs
```

Evidence level: mixed/inferred. Installed localization says Burghers attempt to address goods needs in the market of their town or city by doing their own trades. Modifier text says `local_trades_per_burgher` and `global_trades_per_burgher` affect how much trade per population Burghers can do on their own. The exact wealth conversion is engine-side.

Other pops create demand and work in buildings/RGOs, but they do not have an equivalent autonomous `trades_per_<pop>` capacity in the installed files. In this build, the named autonomous pop-trade system is Burgher-specific.

## Economy: Dependency Graph

Weight labels are strategy weights, not a claim that every branch has one stable global coefficient.

- `Critical`: usually the first-order constraint or multiplier.
- `High`: strongly affects outcomes in many starts.
- `Medium`: often important, but depends more on context or specific branch.
- `Low`: mostly secondary for tax base unless combined with other levers.
- `Conditional`: can dominate in specific markets, geographies, or estate setups.

Evidence labels:

- `Exact`: directly exposed as a formula, coefficient, or file value.
- `Derived`: algebraically inferred from exact pieces.
- `Mixed`: partly file/UI backed, partly engine-side or state-dependent.
- `Inferred`: useful strategy model, but not exposed as a direct formula.

Top-level economy graph:

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

[Back to top graph](#economy-dependency-graph)

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

#### Peasant Enfranchisement

Peasant enfranchisement is a wealth-share redirect. The Peasants estate has `disenfranchise_to = nobles_estate`, and the UI tooltip says peasants receive only their average enfranchisement percentage of their wealth share; the rest goes to Nobles.

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

The building modifier is `local_crown_estate_power`, localized as Local Crown Power. It is a location-category estate-power modifier. The installed localization says it affects the estate power of the government, or Crown Power, in that location.

```text
local_crown_estate_power building
  -> higher Crown estate power in that location
  -> lower relative estate dominance there
  -> better treasury result only through allocation, tax caps, or Crown/estate income splits
```

Use it as an extraction modifier, not a growth modifier. It does not raise `wealth`, prices, output, market access, or control on its own. In a rich but estate-dominated location, it can improve how much existing economy the state captures; in a poor location, there is little existing income to redirect.

Control also contributes to local Crown Power through the `inverse_control` static modifier:

```text
inverse_control_factor = 1 - local_control

local_crown_power_from_control =
    -1.0 * inverse_control_factor
```

So a building with `local_crown_estate_power = 0.25` in a `60%` control location first has to offset about `-40%` Local Crown Power from low control. This is a separate local modifier from any Crown Power granted by the building itself.

Installed building examples:

| Building group | Values |
|---|---|
| `council_hall.txt` | early council hall `local_crown_estate_power = 0.10`; council hall `0.2`; modern council hall `0.5` |
| `town_buildings.txt` | counting house `0.33`; minting office `0.25` plus `local_max_control = 0.05` |
| Some estate/fort/trade-company buildings | negative values such as `-0.1` or `-0.2`, meaning less local Crown Power |

#### National Crown Power

Country-scope Crown Power is built from country modifiers, population contribution, base Crown power, privileges, laws, reforms, advances, average control, and the pressure of non-Crown estates. The installed files expose these key modifiers:

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

National Crown Power is not a direct production stat either. Its income effect is mainly that it changes how much estate power pushes back against the state, what Crown-power-scaled effects apply, and how much normal trade income reaches the treasury:

```text
normal_trade_profit
  -> trade_income
  -> split by estate_power
  -> treasury receives Crown Power percentage
```

This is why a local Crown Power building and a national Crown Power law can both help the state but at different layers. The building fights the local allocation problem in one location. The national modifier changes the country's overall Crown/estate balance and therefore matters even when income comes from trade routes rather than local tax base.

Current-build caveat: control now reaches Crown Power through two visible paths. Locally, low control applies an `inverse_control` penalty to Local Crown Power. Nationally, average control modifies the Crown's population-based power. The second path is not shown as a redistribution to a specific rival estate; it changes the Crown's population multiplier. For strategy, treat this as a compound penalty for low-control population rather than as a simple zero-sum estate transfer.

For non-Crown estates with taxable pops, local power affects income by changing who controls the taxable base, not by creating source wealth by itself:

```text
more local_<non_crown_estate>_estate_power
  -> larger local estate share of tax base
  -> more income only if that estate has a usable tax rate/cap
  -> less useful if the estate is hard to tax or has high-power max-tax penalties
```

Country-level estate power also matters because total estate power reduces Crown Power. Lower Crown Power reduces the treasury share of normal trade income, since installed concepts say only the Crown Power share of trade income reaches the treasury. Crown-specific local power is therefore best read as a local extraction and centralization lever, national Crown Power as a country-wide state-capacity lever, and non-Crown local estate power as a local taxable-share lever.

### Trade And Naval Economy Subgraph

[Back to top graph](#economy-dependency-graph)

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

Normal trade-route equations:

```text
trade_profit =
    Trade.GetSell
  - Trade.GetBuy
  - Trade.GetMaintenance
  - Trade.GetSoundTollFee

trade_profit_per_capacity =
    Trade.GetProfit / Trade.GetAssignedMerchantCapacityModifiedPlayer

route_trade_income_display =
    Trade.GetProfitPerMerchantCapacity * owner.trade_income
```

Possible-trade equations:

```text
possible_trade_profit =
    PossibleTrade.GetSellPrice
  - PossibleTrade.GetBuyCost
  - PossibleTrade.GetMaintenance
  - possible_sound_toll_if_any

possible_profit_per_capacity =
    PossibleTrade.GetProfit / PossibleTrade.GetDesiredMerchantCapacity
```

Use these as UI-level equations. The buy/sell values already include engine-side effects such as market prices, quantities, efficiencies, route conditions, and availability.

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

#### Community Analysis: Trade System Goals

- [On Trade: Trade Routes, Prices, Production, and the Gameplay Goals of a Trade System](https://forum.paradoxplaza.com/forum/threads/on-trade-trade-routes-prices-production-and-the-gameplay-goals-of-a-trade-system.1922327/) — a long community write-up (not an official dev diary) walking through how trade currently resolves (route profit as sell price with efficiencies minus buy price minus transport cost, with transport cost driven by good weight, distance, and trade range) and proposing changes aimed at richer regional price gradients and more sustainable long-distance trade. Concepts worth knowing when reasoning about the model in this guide:
  - **Trade friction** — the author's term for the total cost of moving a good between markets (transport cost, merchant minimum margins, frictions like piracy/tariffs, offset by trade efficiency). Higher friction widens the price gap between producing and consuming markets, which is what creates room for merchant profit; very large trade-range bonuses can flatten that gradient and reduce profitable opportunities.
  - **Selling/import/export efficiency as multipliers on price** can in extreme cases make a route profitable even when the destination price is *lower* than the source price (effectively negative friction), which the post argues equalizes prices across markets and erodes profit margins; it proposes these efficiencies instead multiply the *profit margin* rather than the sale price.
  - **Consumer vs. industrial goods** — shortages of industrial inputs (e.g. tools, charcoal, iron) cascade through dependent production chains far more harshly than shortages of consumer goods, making any industrial deficit an emergency; the post suggests output shouldn't drop until shortages are severe (e.g. ~50%), with buildings instead losing profitability (and scaling down gradually, level by level) before they lose throughput.
  - **Output vs. throughput modifiers** — the post argues throughput modifiers (scaling both inputs and outputs together) make buildings easier to balance against input/output prices than flat output multipliers.
  - **Common vs. rare goods** — common goods (iron, fibers) compete mainly on efficiency/cost advantages, while rare goods (cloves, spices) have a price floor set by production cost plus the transport cost of the cheapest route; the post proposes a "flexible base price" for rare goods that drifts toward production cost plus shipping cost over time, so long-distance trade in rare goods stays viable without requiring enormous trade capacity.
  - **Routing realism** — the post also argues trade paths should hug rivers and coasts more strongly (with real costs for switching between sea/river/land legs) rather than cutting overland through unrelated regions.

  Treat this as one experienced player's design critique and proposal set, useful for understanding *why* the trade levers in this guide (merchant capacity, trade range, trade income, selling/import/export efficiency, market access) behave the way they do and where the community sees friction — not as a description of confirmed or upcoming game behavior. Per this guide's source policy, the installed game files remain authoritative for exact current values.

## Economy: Parameter Dictionary

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
| `power_per_pop` | Estate political-power weight per pop. | `estates/00_default.txt`. |
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

## Economy: Strategy Checklist

When choosing an economic action, walk the chain in order:

1. Is the location's wealth already high?
2. Is control low enough that raising it gives immediate tax-base gain?
3. Is market access suppressing profit or building caps?
4. Is the wealth mostly RGO, building, or burgher trade?
5. Are output prices high enough and input prices low enough for the branch to profit?
6. Will new production create useful demand elsewhere or just oversupply the output good?
7. Which estate will receive the tax base, and can that estate be taxed?
8. Would trade capacity/range/power produce more immediate treasury income than local tax investment?
9. Does storage protect the production chain from shortages?
10. Does the action support future guide goals such as military logistics, control, diplomacy, or expansion?

## Economy: Common Strategic Patterns

### Rich But Uncontrolled Land

Best levers:

- Raise max control with proximity, roads, ports, maritime presence, and control buildings.
- Use monthly control modifiers to speed recovery.
- Consider subjects in very low-control regions if local control would take too long and the subject can propagate its own control.

Why: control is the direct tax-base multiplier. Raising control on rich land is often cleaner than adding more production to already untaxable wealth.

### Profitable Rural Raw Goods

Best levers:

- Expand RGO size where the good is valuable and demand is durable.
- Improve market access before overbuilding.
- Avoid urbanizing the best rural RGO locations unless the city plan is more valuable than lost RGO potential.
- Use targeted output modifiers when the good is a strategic export or input.

Why: RGO wealth can be high, but only if the location can employ workers and sell at a good effective price.

### Industrial Towns And Cities

Best levers:

- Keep input and maintenance goods available and affordable.
- Improve production efficiency, staffing, and promotion speed.
- Build enough upstream production to prevent input bottlenecks.
- Use market access and storage to avoid supply breaks.

Why: buildings are the most scalable wealth branch, but their profit is sensitive to output prices, input costs, market access, and whether the location can actually staff them.

### Trade-Focused Economy

Best levers:

- Increase merchant capacity where profitable routes exist.
- Increase Trade Advantage where supply is contested.
- Increase trade range for long-distance goods.
- Maintain Crown Power and tax caps so commercial wealth reaches the treasury.
- Use ports, harbor capacity, and maritime presence in coastal trade networks.

Why: trade has direct income and indirect price/availability effects. It can outperform local tax investment in low-control regions, but estate splits still matter.

### Estate-Heavy Bottleneck

Best levers:

- Identify which estate owns the local tax base.
- Check that estate's max tax cap and satisfaction/power effects.
- Use laws, reforms, privileges, and societal values that improve extraction without breaking the productive branch.
- Watch peasant enfranchisement because low values move peasant share toward Nobles, while high values let Peasants keep more of it.

Why: tax base is not treasury income until it passes through estate allocation and tax extraction.

## Future Guide Sections

These sections are intentionally placeholders. Add them later using the same rule: inspect installed game files first, then cross-check public sources only as secondary context.

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
