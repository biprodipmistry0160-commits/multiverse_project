# multiverse_project

multiverse-project/
├─ src/
│  ├─ core/
│  │  ├─ universe_base.py
│  │  └─ simulator.py
│  ├─ universes/
│  │  ├─ earth.json
│  │  └─ mars.json
│  └─ app.py
├─ tests/
│  └─ test_universe.py
└─ README.md
////
# src/core/universe_base.py

from dataclasses import dataclass


@dataclass
class Universe:
    """
    A simple representation of a 'universe' in the multiverse.

    Attributes
    ----------
    name : str
        Name of the universe (e.g., 'Earth', 'Mars').
    gravity : float
        Gravitational acceleration in m/s^2.
    day_length_hours : float
        Duration of a full day in this universe, in hours.
    description : str
        Short human-readable description.
    """
    name: str
    gravity: float
    day_length_hours: float
    description: str

    def jump_height(self, initial_velocity: float) -> float:
        """
        Compute max jump height for a given initial vertical velocity
        using basic kinematics: h = v^2 / (2g).

        Parameters
        ----------
        initial_velocity : float
            Initial vertical velocity in m/s.

        Returns
        -------
        float
            Maximum height reached in meters.
        """
        if self.gravity <= 0:
            raise ValueError("Gravity must be positive.")
        return (initial_velocity ** 2) / (2 * self.gravity)
# src/core/simulator.py

import json
from pathlib import Path
from typing import List

from .universe_base import Universe


class MultiverseSimulator:
    """
    Loads multiple universes from JSON config files and runs
    simple comparisons between them.
    """

    def __init__(self, universe_dir: str | Path) -> None:
        self.universe_dir = Path(universe_dir)
        self.universes: List[Universe] = []

    def load_universes(self) -> None:
        """
        Load all *.json files in the universe_dir as Universe objects.
        """
        if not self.universe_dir.exists():
            raise FileNotFoundError(f"Universe directory not found: {self.universe_dir}")

        for json_file in self.universe_dir.glob("*.json"):
            with json_file.open("r", encoding="utf-8") as f:
                data = json.load(f)

            universe = Universe(
                name=data["name"],
                gravity=data["gravity"],
                day_length_hours=data["day_length_hours"],
                description=data.get("description", "")
            )
            self.universes.append(universe)

    def compare_jump_height(self, velocity: float) -> list[dict]:
        """
        For a given initial velocity, compute the jump height in each universe.

        Returns a list of dicts that can be printed or converted to a table.
        """
        results: list[dict] = []
        for u in self.universes:
            height = u.jump_height(velocity)
            results.append(
                {
                    "universe": u.name,
                    "gravity": u.gravity,
                    "day_length_hours": u.day_length_hours,
                    "jump_height_m": round(height, 3),
                }
            )
        return results
{
  "name": "Earth",
  "gravity": 9.81,
  "day_length_hours": 24.0,
  "description": "Baseline human universe with standard Earth gravity."
}
{
  "name": "Mars",
  "gravity": 3.71,
  "day_length_hours": 24.6,
  "description": "Lower gravity red planet. Good for big jumps, bad for breathing."
}
# src/app.py

from pathlib import Path

from core.simulator import MultiverseSimulator


def pretty_print_table(rows: list[dict]) -> None:
    if not rows:
        print("No data to display.")
        return

    # Simple text table (no external libs)
    headers = list(rows[0].keys())
    col_widths = {h: max(len(h), *(len(str(row[h])) for row in rows)) for h in headers}

    def fmt_row(row: dict | None = None, header: bool = False) -> str:
        if header:
            return " | ".join(f"{h:<{col_widths[h]}}" for h in headers)
        else:
            return " | ".join(f"{str(row[h]):<{col_widths[h]}}" for h in headers)

    print(fmt_row(header=True))
    print("-+-".join("-" * col_widths[h] for h in headers))
    for r in rows:
        print(fmt_row(r))


def main() -> None:
    base_dir = Path(__file__).resolve().parent
    universe_dir = base_dir / "universes"

    sim = MultiverseSimulator(universe_dir=universe_dir)
    sim.load_universes()

    print("🌌 Multiverse Simulator")
    print(f"Loaded {len(sim.universes)} universes.\n")

    # You can change this velocity to see different results
    initial_velocity = 3.0  # m/s (~small jump)

    results = sim.compare_jump_height(initial_velocity)
    print(f"Initial jump velocity: {initial_velocity} m/s\n")
    pretty_print_table(results)

    print("\nTip: Add more *.json files in src/universes/ to create new universes.")


if __name__ == "__main__":
    main()
# tests/test_universe.py

import math
from src.core.universe_base import Universe


def test_jump_height_positive():
    earth = Universe(
        name="Earth",
        gravity=9.81,
        day_length_hours=24,
        description="Test universe"
    )
    v = 3.0
    h = earth.jump_height(v)
    expected = (v ** 2) / (2 * 9.81)
    assert math.isclose(h, expected, rel_tol=1e-6)
# Multiverse Project

A tiny **Multiverse Simulator**: each universe is defined by its own physics
(gravity, day length, description), and we compare simple behaviours across them.

## How it works

- Universes are defined as JSON files in `src/universes/`.
- Each universe has:
  - `name`
  - `gravity` (m/s²)
  - `day_length_hours`
  - `description`
- The simulator loads all universes and calculates how high a jump would go
  in each universe for the same initial velocity.

## Run locally

```bash
cd multiverse-project
python -m src.app

---

## 9. What you do now

1. Create the folders/files exactly as above.
2. Paste the code into each file.
3. From repo root:

```bash
git add .
git commit -m "Add basic multiverse simulator"
git push origin main
