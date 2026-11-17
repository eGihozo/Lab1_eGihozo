# Lab 1: FSD, but on Kigali Roads

A Full Self Driving (FSD) car drives from ALU Rwanda to the Kigali Convention Centre. The road is a Python list, and my code decides how the car reacts to each sign.



## Table of contents

1. How the road works
2. Repository structure
3. Requirements
4. How to run
5. What I implemented
6. Validation and edge cases
8. Shell script add-on: log analyser
9. Notes
## How the road works

The road is a list called 'road' in 'road_simulation.py'. Each item is one 100 m segment. Index `0` is ALU Rwanda and the last index is the Kigali Convention Centre.

| Item in `road` | Meaning |
|----------------|---------|
| `"20"` to `"100"` | Speed limit sign: the new limit for the car |
| `"zebra"` | Zebra crossing: the car stops, then continues |
| `"bus"` | Bus in the right lane: the car moves to the left lane |
| `"road"` | Plain road: nothing happens |

Rules for the data:
- Every sign is stored as **text**, even speed limits (for example `"80"`, not `80`).
- Only `"bus"`, `"zebra"` or a number from 20 to 100 may be added to the road.
- The road can change while the program runs: signs can be inserted or replaced by index.

## Repository structure

```
lab1_eGihozo/
├── road_simulation.py     # My work: the 10 completed functions
├── engine.py              # Provided, not edited
├── transit_program.py     # Provided, not edited
├── index.html             # Provided, not edited
├── analyze_log.sh         # Bash add-on: log analyser
├── transit_session.log    # Written automatically by each run
└── README.md
```

## Requirements

- Python 3
- A web browser
- Bash (only for the log analyser)

No extra libraries need to be installed.

## How to run

1. Open a terminal in the `lab1_eGihozo` folder.
2. Start the program:
   ```bash
   python road_simulation.py
   ```
   (or `python3 road_simulation.py`)
3. Enter your driver name, then use the road editor menu.
4. Choose **5. Start driving**, then open the link shown (for example `http://127.0.0.1:8008`).
5. Go back to the terminal and press **Enter**. The car starts to drive.

## What I implemented

All 10 functions are in `road_simulation.py`.

### Part 1: Check the sign

| Function | What it does |
|----------|--------------|
| `is_valid_sign(sign)` | Returns `True` only for text that is `"bus"`, `"zebra"` or a number from 20 to 100 written as text |
| `get_sign_type(sign)` | Returns `"bus"`, `"zebra"`, `"speed"` or `"road"` |

### Part 2: Take action

| Function | What it does |
|----------|--------------|
| `get_speed_limit(sign, current_limit)` | Returns the sign's number as the new limit, or keeps the current limit |
| `get_target_speed(limit, stop_timer)` | Returns `0` while stopped at a zebra, otherwise the limit |
| `adjust_speed(speed, target)` | Moves speed one step towards the target: **+5** when below, **-20** when above, never overshooting |
| `count_signs(road_list)` | Uses a loop to return `(speed_signs, zebras, buses)` |

### Part 3: Input validation and journey summary

| Function | What it does |
|----------|--------------|
| `ask_for_number(question, low, high)` | Keeps asking until the input is a whole number in range, using `try/except` |
| `insert_sign(road_list, sign, position)` | Inserts a sign at `position` (0 to `len(road_list)`); returns `False` if out of bounds |
| `replace_sign(road_list, sign, index)` | Replaces the sign at `index` (0 to `len(road_list) - 1`); returns `False` if out of bounds |
| `get_journey_summary(road_list)` | Counts the signs passed, works out the road length (segments x 100 m), and returns the start and end names |

## Validation and edge cases

| Situation | How the program handles it |
|-----------|----------------------------|
| Text typed where a number is expected (e.g. `abc`) | `try/except` catches the error, prints a message and asks again |
| Number outside the allowed range | A clear message is printed and the user is asked again |
| Insert position below 0 or past the end of the road | `insert_sign` returns `False`; the road is unchanged |
| Replace index below 0 or past the last segment | `replace_sign` returns `False`; the road is unchanged |
| Sign that is not `"bus"`, `"zebra"` or `"20"`-`"100"` | `is_valid_sign` returns `False` |
| Sign that is not text (e.g. the integer `60`) | `is_valid_sign` returns `False` |
| Unknown or invalid item in the road | `get_sign_type` treats it as plain `"road"`, so nothing crashes |
| Missing log file (shell script) | Prints an error and exits with code `1` |

## Examples I tested

I ran these against my code and they gave the results shown.

| Call | Result |
|------|--------|
| `is_valid_sign("80")` | `True` |
| `is_valid_sign("19")`, `is_valid_sign("101")` | `False` |
| `is_valid_sign(60)` (not text) | `False` |
| `is_valid_sign("abc")`, `is_valid_sign("")` | `False` |
| `get_sign_type("bus")` / `"zebra"` / `"80"` / `"road"` | `"bus"` / `"zebra"` / `"speed"` / `"road"` |
| `get_speed_limit("80", 60)` | `80` |
| `get_speed_limit("bus", 60)` | `60` (limit kept) |
| `get_target_speed(60, 3)` | `0` (stopped at zebra) |
| `get_target_speed(60, 0)` | `60` |
| `adjust_speed(0, 60)` | `5` (accelerates by 5) |
| `adjust_speed(58, 60)` | `60` (does not overshoot) |
| `adjust_speed(80, 60)` | `60` (brakes by 20, stops at target) |
| `adjust_speed(65, 60)` | `60` (does not undershoot) |
| `count_signs(road)` for the default road | `(4, 2, 2)` |
| `get_journey_summary(road)` for the default road | `(8, 3400, "ALU Rwanda", "Convention Center")` |
| `insert_sign(road, "x", len(road) + 1)` | `False` |
| `replace_sign(road, "x", len(road))` | `False` |

## Shell script add-on: log analyser

Each run of the simulation appends its events to `transit_session.log`. `analyze_log.sh` reads that log and prints a summary. Run the simulation at least once first so the log exists.

```bash
bash analyze_log.sh                  # uses transit_session.log
bash analyze_log.sh other_file.log   # uses a file you choose
```

It prints:
- the number of sessions (runs) in the log
- the number of zebra crossing stops
- the number of lane changes
- the driver name and top speed of the latest trip

If the file does not exist, it prints an error and exits with code `1`.

## Notes

- I did not edit `engine.py`, `transit_program.py` or `index.html`.
- The page only shows what the Python code sends, so if something goes wrong, check the terminal error first.