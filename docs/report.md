# Distribution Network Optimisation — Report

## Introduction

This report develops and solves a linear programming model to help a
manufacturing organisation minimise its distribution costs while respecting
capacity and demand constraints. The company operates 4 factories and
distributes 3 products to 6 retailers, either directly or via 4 warehouses.

## Model Formulation

### Decision variables

- `a_frp` — quantity of product `p` transported from factory `f` to retailer `r`
- `b_fwp` — quantity of product `p` transported from factory `f` to warehouse `w`
- `c_wrp` — quantity of product `p` transported from warehouse `w` to retailer `r`

for all `f ∈ {1,2,3,4}`, `p ∈ {1,2,3}`, `w ∈ {1,2,3,4}`, `r ∈ {1,2,3,4,5,6}`.

### Objective function

Minimise total distribution cost across all three transport legs:
factory → warehouse, factory → retailer, and warehouse → retailer.

$$
\text{Total Cost} = \sum_{f=1}^{4}\sum_{w=1}^{4}\sum_{p=1}^{3} \text{Cost}_{fwp} \cdot b_{fwp}
\;+\; \sum_{f=1}^{4}\sum_{r=1}^{6}\sum_{p=1}^{3} \text{Cost}_{frp} \cdot a_{frp}
\;+\; \sum_{w=1}^{4}\sum_{r=1}^{6}\sum_{p=1}^{3} \text{Cost}_{wrp} \cdot c_{wrp}
$$

Where:
- `Cost_fwp` — unit cost of transporting product `p` from factory `f` to warehouse `w`
- `Cost_frp` — unit cost of transporting product `p` from factory `f` to retailer `r`
- `Cost_wrp` — unit cost of transporting product `p` from warehouse `w` to retailer `r`

### Constraints

**Factory capacity constraint** — total product shipped from each factory
(direct + via warehouses) cannot exceed that factory's capacity:

$$
\sum_{w=1}^{4} b_{fwp} + \sum_{r=1}^{6} a_{frp} \;\le\; C_f \qquad \forall f \in \{1,2,3,4\},\; \forall p \in \{1,2,3\}
$$

**Retailer demand constraint** — total product received by each retailer
(direct + via warehouses) must meet or exceed demand:

$$
\sum_{f=1}^{4} a_{frp} + \sum_{w=1}^{4} c_{wrp} \;\ge\; C_r \qquad \forall r \in \{1,2,3,4,5,6\},\; \forall p \in \{1,2,3\}
$$

**Warehouse balance constraint** — inflow to each warehouse must equal
outflow:

$$
\sum_{f=1}^{4} b_{fwp} \;=\; \sum_{r=1}^{6} c_{wrp} \qquad \forall w \in \{1,2,3,4\},\; \forall p \in \{1,2,3\}
$$

**Warehouse capacity constraint** — inbound shipments to a warehouse cannot
exceed its capacity:

$$
\sum_{f=1}^{4} b_{fwp} \;\le\; C_w \qquad \forall w \in \{1,2,3,4\},\; \forall p \in \{1,2,3\}
$$

**Non-negativity constraint** — all decision variables are non-negative:

$$
a_{frp} \ge 0, \quad b_{fwp} \ge 0, \quad c_{wrp} \ge 0
$$

## Results

The optimisation yielded a total objective cost of **£1,640,500**. All
constraints were satisfied: factory capacities were respected, warehouse
inflow/outflow stayed balanced and within capacity, and every retailer's
demand was fully met.

### Optimal routes

**Factories → Warehouses** (total cost £183,250)
- Factory 3 was the primary supplier, moving 215,000 units overall.
- Factory 3 → Warehouse 1 was the standout route: 32,500 units each of
  Product 2 and Product 3.

**Warehouses → Retailers** (total cost £846,500)
- Warehouse 4 was the most-used warehouse, shipping 92,500 units in total.
- Warehouse 2 → Retailer 4 was the most efficient route: 25,000 units of
  Product 2 and 25,000 units of Product 3.

**Direct factory → retailer shipments** (total cost £610,750)
- Factory 1 was the leading direct supplier (125,000 units across all three
  products).
- Factory 1 → Retailer 4 was the best direct route: 32,500 units of
  Product 1 and 7,500 units of Product 2.

## Discussion

Direct shipments proved more cost-effective overall than routing through
warehouses: £610,750 versus £1,029,750 for the combined indirect legs, even
though the indirect route carried a larger total volume (305,000 units). By
skipping the warehouse stage, direct shipments avoid extra handling and
storage costs. Because indirect routes were consistently pricier for some
factory-retailer pairs, certain combinations were dropped entirely — Factories
2 and 3, for instance, made no direct retailer shipments at all. Warehouse 3
was comparatively underused, suggesting spare capacity for future demand
growth.

## Sensitivity Analysis

- **Reduced costs**: zero-shipment routes such as Factory 1 → Warehouse 4
  (Product 2) would need a cost reduction of £2.55 per unit before becoming
  attractive.
- **Capacity headroom**: Factory 3's Product 3 capacity is used at 72,500 of
  100,000 units. An allowable increase of effectively unlimited size (1E+30)
  and a £0 shadow price indicate current capacity is more than sufficient —
  adding more wouldn't reduce cost further.

## Conclusion

The model successfully minimised total transportation cost, with direct
factory-to-retailer shipments as the most economical approach — the solver
consistently assigned volume to the cheapest available routes. A logical next
step would be reducing reliance on multi-leg "chain" routes further, reserving
warehouse routing mainly for smaller, lower-priority demand.

---

## Carbon Policy Model — Incorporating a Carbon Policy

### Approach

To model a carbon policy, a carbon cost is added on top of the existing
transport cost for every route, combining economic and environmental impact
into a single trade-off.

**Assumptions**: emission factors sourced from DEFRA/BEIS (2024) and EPA
(2024); a carbon cost factor of £10 per tonne of CO2, based on IMF (2023) and
PwC (2023) estimates.

$$
\text{Carbon Emissions (kg CO}_2\text{)} = \text{Emission Factor (kg CO}_2\text{/ton-km)} \times \text{Weight (tons)} \times \text{Distance (km)}
$$

$$
\text{Carbon Cost} = \text{Carbon Emissions (tons CO}_2\text{)} \times \text{Carbon Cost Factor (£/ton CO}_2\text{)}
$$

### Updated objective function

$$
\text{Total Cost} = \big(\text{Carbon Emissions (tons CO}_2\text{)} \times \text{£10/ton CO}_2\big) + \text{Transportation Cost}
$$

Capacity, demand, and warehouse-balance constraints are unchanged from the
base model.

### New optimal solution

Total cost rose to **£5,540,000** once carbon cost was included, but the model
remained efficient overall — it simply reweighted routes to reflect the added
environmental cost.

- **Factory → Warehouse**: shipments dropped sharply. Only Factory 3 →
  Warehouse 3 remained active, moving 12,500 units of Product 3; every other
  factory-warehouse route became uneconomical.
- **Factory → Retailer** (cost £5,141,250): direct shipments were strongly
  favoured. Factory 4 → Retailer 6 was the largest route, with 55,000 units of
  Product 3.
- **Warehouse → Retailer** (cost £381,250): minimal activity, limited to
  Warehouse 3 → Retailer 3 (12,500 units of Product 3).

### Key insights

- Direct shipments were favoured for high-demand retailers, reducing overall
  emissions.
- Factory-to-retailer shipments dominated the network, reflecting a shift
  toward simpler, more direct distribution once carbon cost was priced in.
- Route selection balanced *total* cost (transport + carbon) rather than
  emissions alone — some lower-emission factory-to-warehouse routes were still
  dropped because their combined cost was higher than the direct alternative.

Overall, adding a carbon cost produces a more environmentally-conscious
distribution plan while still optimising for cost-efficiency.
