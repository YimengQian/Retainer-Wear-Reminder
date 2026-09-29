# Retainer Wear Reminder

A personal retainer-wear tracker built with **Streamlit**. It calculates upcoming wear dates from a configurable interval and stores daily records in **Supabase** so they can be viewed across devices.

## Features

- Calculates scheduled wear dates using a fixed interval (every 3 days by default).
- Records that you wore your retainer today with one click.
- Prevents duplicate records for the same day.
- Shows the total number of recorded days, recent history, and the next scheduled wear date.
- Provides reminders when the next wear date is near.
- Syncs records to a Supabase database.

## Tech Stack

- Python
- [Streamlit](https://streamlit.io/)
- [Supabase](https://supabase.com/)

## Getting Started

### 1. Clone the repository and install dependencies

```bash
git clone https://github.com/YimengQian/Reminder.git
cd Reminder
python -m venv .venv
```

Activate the virtual environment:

```bash
# Windows PowerShell
.\.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install the required packages:

```bash
pip install -r requirements.txt
```

### 2. Configure Supabase

Create a Supabase project, then create a `retainer_records` table to store wear dates. The `date` field must be unique; the following is a minimal schema:

```sql
create table public.retainer_records (
  date date primary key
);
```

Create `.streamlit/secrets.toml` in the project root:

```toml
SUPABASE_URL = "https://<your-project-ref>.supabase.co"
SUPABASE_ANON_KEY = "<your-anon-key>"
```

Find these values in your Supabase project's **Settings → API** page. Never commit `secrets.toml` to the repository.

> **Security note:** The current app accesses `retainer_records` with an anonymous key. Before deployment, configure Supabase Row Level Security (RLS) and access policies for your use case. Do not allow unrestricted public access to tables containing sensitive data.

### 3. Set your wear schedule

Open [reminder v3.py](reminder v3.py) and update these values:

```python
WEAR_INTERVAL_DAYS = 3
FIRST_WEAR_DATE = "2025-02-14"
```

- `WEAR_INTERVAL_DAYS`: Number of days between scheduled wear dates. For example, set it to `2` for a two-day interval.
- `FIRST_WEAR_DATE`: Your first actual wear date, in `YYYY-MM-DD` format.

### 4. Run the reminder

```bash
streamlit run reminder v3.py
```

Open the local URL printed in the terminal to use the app.

## Usage

1. Confirm that the first wear date and interval in `app.py` match your schedule.
2. After wearing your retainer, click **“今天戴了！”** (“Wore it today!”). The record is written to Supabase.
3. Use the home page to review your history, total wear days, and next scheduled date.

## Project Structure

```text
.
├── reminder v3.py    # Current Streamlit app with Supabase synchronization
├── reminder.py       # Earlier implementation
├── reminder v1.py    # Earlier implementation
├── reminder v2.py    # Earlier implementation
├── requirements.txt  # Python dependencies
└── .devcontainer/    # Development-container configuration
```

## Dependencies

```text
streamlit
supabase
```

## Privacy and Data

Wear records are stored in the Supabase project that you configure. Keep your credentials private and use database access policies appropriate for your deployment. The app does not currently include user accounts; if it will be used by multiple people, add authentication and per-user data isolation first.

## Contributing

Issues and pull requests are welcome.
