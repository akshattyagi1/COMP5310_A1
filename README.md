# COMP5310 Assignment 1 - NSW Train Occupancy

## Purpose

This project assesses three candidate datasets and prepares the NSW Train
Occupancy dataset for exploratory analysis and Assignment 2. The stakeholder is
an NSW train operations team deciding where and when to prioritise capacity or
timetable review.

The exploratory question is: **which departure times, stop stations, and
service line-direction combinations show the greatest risk of crowding?** A
record is labelled `Crowded` when its occupancy status is
`FEW_SEATS_AVAILABLE` or `STANDING_ROOM_ONLY`.

## Software requirements

- Python 3.10 or later
- Jupyter Notebook or JupyterLab
- pandas
- NumPy
- matplotlib

Install the Python packages, if needed:

```bash
python -m pip install pandas numpy matplotlib jupyter
```

No random sampling or random modelling is used in this assignment, so no random
seed is required.

## Project structure

```text
COMP5310_A1/
├── data/
│   ├── raw/
│   │   ├── train_occupancy.csv
│   │   ├── airline_delay.csv
│   │   └── hotel_bookings.csv
│   └── processed/
│       ├── train_occupancy_clean_full.csv
│       └── train_occupancy_clean.csv
├── figures/
│   ├── 01_crowding_by_departure_hour.png
│   ├── 02_crowding_by_station.png
│   └── 03_crowding_by_line_direction.png
├── notebooks/
│   ├── 01_dataset_audition.ipynb
│   └── 02_data_readiness.ipynb
└── README.md
```

## Run order

Open Jupyter with `notebooks/` as the working folder, because both notebooks
use the relative path `../data/...`.

1. Run `01_dataset_audition.ipynb` from top to bottom. It compares the Train
   Occupancy, Air Travel Delay, and Hotel Booking datasets and explains the
   choice of Train Occupancy.
2. Run `02_data_readiness.ipynb` from top to bottom. It loads
   `../data/raw/train_occupancy.csv`, audits and cleans it, recreates the
   processed CSV files, and generates the three EDA figures.

from the project root:

```bash
cd notebooks
jupyter notebook
```

## Expected outputs

Running `02_data_readiness.ipynb` recreates:

- `data/processed/train_occupancy_clean_full.csv`: the master cleaned dataset
  (49,927 rows). It retains records with an unavailable arrival or departure
  timestamp, because those rows can still support analyses that do not require
  both timestamps.
- `data/processed/train_occupancy_clean.csv`: the timestamp-complete subset
  (49,880 rows), for analyses requiring both parsed timestamps.
- The three PNG figures in `figures/`.

Cleaning removes 250 exact duplicates and 73 records with missing
`Occupancy Status`; trims whitespace in selected categorical fields;
standardises three inconsistent occupancy labels; and parses valid timestamp
formats. Placeholder or unusable timestamps are retained as missing rather
than guessed.



