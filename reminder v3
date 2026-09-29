from __future__ import annotations

from datetime import date, timedelta

import streamlit as st
from supabase import Client, create_client


# -----------------------------------------------------------------------------
# Configuration
# -----------------------------------------------------------------------------
WEAR_INTERVAL_DAYS = 3
FIRST_WEAR_DATE = "2025-02-14"  # Use the YYYY-MM-DD format.
RECORDS_TABLE = "retainer_records"


@st.cache_resource
def get_supabase() -> Client:
    """Create one Supabase client per Streamlit process."""
    return create_client(
        st.secrets["SUPABASE_URL"],
        st.secrets["SUPABASE_ANON_KEY"],
    )


def get_schedule_config() -> tuple[date, int]:
    """Validate and return the configured first wear date and interval."""
    if WEAR_INTERVAL_DAYS < 1:
        st.error("WEAR_INTERVAL_DAYS must be a positive integer.")
        st.stop()

    try:
        first_wear_date = date.fromisoformat(FIRST_WEAR_DATE)
    except ValueError:
        st.error(
            "FIRST_WEAR_DATE must use the YYYY-MM-DD format. "
            f"Current value: {FIRST_WEAR_DATE}"
        )
        st.stop()

    return first_wear_date, WEAR_INTERVAL_DAYS


def load_records(client: Client) -> set[date]:
    """Load all saved wear dates from Supabase."""
    try:
        response = client.table(RECORDS_TABLE).select("date").execute()
        return {
            date.fromisoformat(record["date"])
            for record in response.data
            if record.get("date")
        }
    except Exception as error:
        st.error(f"Could not load records from Supabase: {error}")
        st.stop()


def add_record(client: Client, wear_date: date) -> None:
    """Save a wear date. The date column must be unique or a primary key."""
    try:
        client.table(RECORDS_TABLE).upsert(
            {"date": wear_date.isoformat()},
            on_conflict="date",
        ).execute()
    except Exception as error:
        st.error(f"Could not save the record: {error}")
        st.stop()


def is_scheduled_wear_date(
    candidate: date, first_wear_date: date, interval_days: int
) -> bool:
    """Return whether a date falls on the configured wear schedule."""
    days_since_first_wear = (candidate - first_wear_date).days
    return (
        days_since_first_wear >= 0
        and days_since_first_wear % interval_days == 0
    )


def get_next_wear_date(
    today: date,
    recorded_dates: set[date],
    first_wear_date: date,
    interval_days: int,
) -> date:
    """Return the next scheduled date that has not already been recorded.

    Today is returned when it is a scheduled date and has not yet been recorded.
    """
    if today <= first_wear_date:
        candidate = first_wear_date
    else:
        days_since_first_wear = (today - first_wear_date).days
        completed_intervals = (days_since_first_wear + interval_days - 1) // interval_days
        candidate = first_wear_date + timedelta(
            days=completed_intervals * interval_days
        )

    while candidate in recorded_dates:
        candidate += timedelta(days=interval_days)

    return candidate


def render_next_wear_message(next_wear_date: date, today: date) -> None:
    """Display a context-appropriate reminder."""
    days_until = (next_wear_date - today).days

    if days_until == 0:
        st.error("Your retainer is scheduled for today. Remember to record it after wearing it.")
    elif days_until == 1:
        st.warning("Your next scheduled wear date is tomorrow.")
    elif days_until <= 3:
        st.warning(f"Your next scheduled wear date is in {days_until} days.")
    else:
        st.info(
            f"Next scheduled wear date: **{next_wear_date.isoformat()}** "
            f"({days_until} days from now)"
        )


def main() -> None:
    st.set_page_config(page_title="Retainer Wear Reminder", page_icon="🦷")
    st.title("Retainer Wear Reminder")
    st.caption("Your records are stored in Supabase and can be viewed across devices.")

    first_wear_date, interval_days = get_schedule_config()
    client = get_supabase()
    today = date.today()
    recorded_dates = load_records(client)

    st.subheader(f"Today: {today.isoformat()}")
    st.info(f"Recorded wear days: **{len(recorded_dates)}**")

    if today in recorded_dates:
        st.success("Today has already been recorded.")
    elif st.button("I wore my retainer today", type="primary", use_container_width=True):
        add_record(client, today)
        st.success("Your wear record has been saved.")
        st.rerun()

    st.divider()
    st.subheader("Next Scheduled Wear Date")
    next_wear_date = get_next_wear_date(
        today,
        recorded_dates,
        first_wear_date,
        interval_days,
    )
    render_next_wear_message(next_wear_date, today)
    st.caption(
        f"Schedule: every {interval_days} days · "
        f"First wear date: {first_wear_date.isoformat()}"
    )

    with st.expander(f"Wear History ({len(recorded_dates)} days)"):
        if recorded_dates:
            for recorded_date in sorted(recorded_dates, reverse=True)[:60]:
                st.write(recorded_date.isoformat())
        else:
            st.write("No wear dates have been recorded yet.")

    st.caption("Data is stored in your Supabase project.")


if __name__ == "__main__":
    main()
