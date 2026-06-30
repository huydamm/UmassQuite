# UmassQuite

**Find a quiet place to study at UMass — before you walk across campus to find out it's packed.**

Every student knows the pain: you trek to the library, climb to the "quiet floor," and it's somehow louder than the dining hall. UmassQuite fixes that. It's a crowdsourced study-spot finder that maps every building and study space on campus and tells you, right now, where it's actually quiet.

## What it does

- 🗺️ **Maps the whole campus** — buildings and study spots pulled from real UMass geodata.
- 🙋 **Crowdsourced reports** — students drop quick reports on how noisy and how crowded a spot is.
- 📊 **Live quiet scores** — every location gets a 0–100 quiet score based on the last 7 days of reports.
- 🏆 **Rankings** — see the quietest spots on campus right now, or filter by hour of day to find where it's calm at 9pm vs. noon.
- 🔮 **It predicts the future** — a machine-learning model forecasts how quiet a spot *will* be an hour from now, so you can plan ahead instead of guessing.

## How it works

The backend is a **FastAPI** service backed by **PostgreSQL**. Students submit reports (`noise_level`, `occupancy_estimate`) for a location, and the API turns the recent history into a quiet score:

```
quiet_score = 100 − (avg_noise × 15 + avg_occupancy × 0.5)
```

On top of that, a **Random Forest** model (scikit-learn) learns the patterns — which location, what hour, what day of the week — and predicts quiet scores into the near future. That powers the `/forecast` and `/quiet-now` endpoints so the app can recommend the best place to go *next*, not just where was quiet an hour ago.

## API at a glance

| Endpoint | What it gives you |
|----------|-------------------|
| `GET /locations` | All study spots (filter by `type`) |
| `POST /report` | Submit a noise + occupancy report |
| `GET /locations/{id}/quiet` | A spot's quiet score over the last 7 days |
| `GET /rankings?hour=` | Ranked quietest spots (optionally for a given hour) |
| `GET /forecast?location_id=&hours_ahead=` | ML-predicted quiet score, up to 12h out |
| `GET /quiet-now` | Every spot ranked by predicted quiet score right now |

## Tech stack

FastAPI · SQLModel / SQLAlchemy · PostgreSQL · scikit-learn (Random Forest) · pandas · deployed on Render, frontend on Vercel.

## Running it locally

```bash
cd backend
pip install -r requirements.txt

# point at your Postgres database
export DATABASE_URL="postgresql://campus_user:campus_pass@localhost:5432/campus_db"

# seed locations + train the forecast model, then:
uvicorn main:app --reload
```

Visit `http://localhost:8000/docs` for the interactive API explorer.

---

*Built for students who'd rather study than hunt for a seat.*
