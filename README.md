# Smart Energy Consumption Analyzer

A console application written in C that helps households and small businesses
track appliance-wise electricity usage, calculate energy (kWh) and cost, and
find ways to reduce their bills.

**Project 21 – Course Outcomes covered:** CO1, CO2, CO3, CO4, CO5.

---

## Features

| Menu | Feature |
|------|---------|
| 1 | Add an appliance (name and power rating in watts) |
| 2 | Edit an appliance (all related entries are recalculated) |
| 3 | List appliances |
| 4 | Add a usage entry (appliance, date, start time, end time, days used) |
| 5 | Edit or delete a usage entry |
| 6 | List all usage entries with kWh per session and total kWh |
| 7 | Set the tariff (cost per kWh) |
| 8 | Monthly report: hours, kWh and cost per appliance, plus totals |
| 9 | Top 3 highest-consuming appliances (with text bars and % share) |
| 10 | Monthly comparison chart (text-based), with month-over-month % change and an optional per-appliance chart |
| 11 | Suggestions to reduce consumption, plus a "replace X hrs of device A with device B" what-if |
| 12 | Save data manually |
| 0 | Save and exit |

Data is loaded automatically at start-up and saved automatically after changes
and on exit.

---

## Build and Run

Requires any C compiler (gcc, clang, MinGW).

```bash
gcc -Wall -Wextra -std=c99 -o energy_analyzer energy_analyzer.c
./energy_analyzer          # Windows: energy_analyzer.exe
```

---

## How energy is calculated

```
hours per session = (end - start) in minutes / 60      (end < start means past midnight)
kWh per session   = watts x hours / 1000
total kWh         = kWh per session x days used
cost              = total kWh x tariff
```

"Days used" is the number of days in that month on which the same
start/end session was repeated. For example, an AC rated 1500 W running
20:00 to 02:00 (6 h) on 25 days gives 1.5 kW x 6 h x 25 = 225 kWh.
An entry is counted in the month of its start date.

---

## Example session

```
Tariff set to 8.50 per kWh.
Added 'Air Conditioner' (1500.0 W).
Logged: 9.000 kWh per session, 225.000 kWh total (cost 1912.50).

=== Top 3 consumers: All time ===
1. Air Conditioner   405.000 kWh  cost 3442.50  (77.1%)  ########################################
2. Fridge            107.925 kWh  cost  917.36  (20.5%)  ###########

=== Monthly comparison (total kWh) ===
Oct 2026 | ##################### 180.00 kWh  (cost 1530.00)
Nov 2026 | ######################################## 345.38 kWh  (cost 2935.69)  [+91.9% vs prev]
```

---

## Input formats and validation

- Dates: `DD-MM-YYYY` (`/` also accepted). Real calendar dates only, including leap years.
- Times: 24-hour `HH:MM`. Start and end cannot be equal.
- Month filters: `MM-YYYY`, or press Enter for all time.
- Watts must be 1 to 20000. Days used cannot exceed the days in that month.
- Duplicate appliance names (case-insensitive) are rejected.
- Every prompt re-asks until the input is valid (no crashes on text typed instead of numbers).

---

## Program design

| Concept | Where it is used |
|---------|------------------|
| **Structures** | `Appliance`, `Usage`, and `Store` (holds both arrays and the tariff) |
| **Dynamic memory** | `malloc` for initial arrays, `realloc` doubling when full (`new_appliance_slot`, `new_usage_slot`), `calloc` for report arrays, `free` on exit |
| **Pointers** | Functions receive `Store *`, `Appliance *`, `Usage *`; new slots are returned as pointers |
| **Functions** | One function per operation: `add_appliance`, `edit_appliance`, `add_usage`, `report_top3`, `report_comparison`, etc. |
| **Strings** | `parse_date`, `parse_time` (sscanf and validation), `trim`, case-insensitive compare |
| **Loops and arrays** | `aggregate()` builds per-appliance kWh/hour arrays; selection sort for the top 3; text bar charts |
| **Decision logic** | Validation, overnight sessions, suggestion rules |
| **File I/O** | `save_data` / `load_data` using a plain text file |

### Suggestion logic
For each of the top 3 appliances in the chosen period the program shows:
1. The saving from cutting runtime by 20%.
2. Rule-based tips (very high wattage, very long runtime, and so on).

The what-if option then lets you pick device A and a lower-power device B
and enter the hours to shift, and it reports the kWh and cost saved. It is
left to you to choose B so that the comparison is realistic (for example,
air conditioner to ceiling fan).

---

## Data file format (`energy_data.txt`)

```
TARIFF 8.5000
APPLIANCES 2
Air Conditioner|1500.0000
Fridge|150.0000
USAGES 2
0 5 11 2026 1200 120 25      <- appliance index, day, month, year, start min, end min, days used
1 1 11 2026 0 1439 30
```

kWh values are recomputed on load, so editing a wattage never leaves stale data.
Delete `energy_data.txt` to start fresh.

---

## Limitations and possible extensions

- Costs use a single flat tariff (no slab or time-of-day rates).
- Deleting appliances is not supported (to keep usage references consistent); edit the name or rating instead.
- Ideas: slab-based tariffs, CSV export, budget alerts, a standby-power estimator.
