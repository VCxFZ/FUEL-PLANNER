# FUEL-PLANNER
PLANNING FUEL 
import streamlit as st
from geopy.distance import great_circle
import pandas as pd

# --- Airport Coordinates DB (minimal for now, can be expanded)
airport_coords = {
    "OMDB": (25.2532, 55.3657),
    "OTHH": (25.2731, 51.6086),
    "VIDP": (28.5562, 77.1000),
    "EGLL": (51.4700, -0.4543),
    "KJFK": (40.6413, -73.7781),
    "EDDF": (50.0379, 8.5622),
    "VABB": (19.0896, 72.8656),
    "RJTT": (35.5494, 139.7798),
    "ZBAA": (40.0799, 116.6031)
}

# --- Aircraft data
aircraft_data = {
    "B737-800 NG": {"fuel_burn": 2700, "speed": 450},
    "B737 MAX 8": {"fuel_burn": 2500, "speed": 460},
    "B737 MAX 9": {"fuel_burn": 2550, "speed": 460},
}

# --- Fuel add-ons by category (example values)
def calculate_extra_fuel(risks):
    fuel = 0
    notes = []

    if risks['weather'] == "Moderate":
        fuel += 300
        notes.append("+300 kg for moderate weather")
    elif risks['weather'] == "Severe":
        fuel += 600
        notes.append("+600 kg for severe weather")

    if risks['atc_delay']:
        fuel += 400
        notes.append("+400 kg for expected ATC delay")

    if risks['terrain']:
        fuel += 300
        notes.append("+300 kg for mountainous/remote area")

    if risks['tech_issue']:
        fuel += 500
        notes.append("+500 kg for MEL/CDL limitations")

    if risks['night_op']:
        fuel += 200
        notes.append("+200 kg for night operations")

    return fuel, notes

# --- Streamlit UI
st.title("Airline Extra Fuel Planner")
st.markdown("Estimate extra fuel required for dispatch based on route and operational risk factors.")

# --- Inputs
col1, col2 = st.columns(2)
aircraft = col1.selectbox("Select Aircraft Type", list(aircraft_data.keys()))
dep = col1.text_input("Departure ICAO", "OMDB")
arr = col2.text_input("Destination ICAO", "VIDP")

# --- Risk Inputs
st.header("Operational Risk Factors")
weather = st.selectbox("Destination Weather Forecast", ["None", "Moderate", "Severe"])
atc_delay = st.checkbox("ATC Congestion Expected?")
terrain = st.checkbox("Mountainous / Remote Area?")
tech_issue = st.checkbox("MEL/CDL Technical Issues Present?")
night_op = st.checkbox("Night Time Operation?")

# --- Estimate Base Fuel
if dep in airport_coords and arr in airport_coords:
    distance_nm = great_circle(airport_coords[dep], airport_coords[arr]).nautical
    speed = aircraft_data[aircraft]['speed']
    fuel_burn = aircraft_data[aircraft]['fuel_burn']
    est_time_hr = distance_nm / speed
    base_fuel = est_time_hr * fuel_burn

    # Extra Fuel Calculation
    risks = {
        'weather': weather,
        'atc_delay': atc_delay,
        'terrain': terrain,
        'tech_issue': tech_issue,
        'night_op': night_op
    }
    extra_fuel, reasons = calculate_extra_fuel(risks)

    total_fuel = base_fuel + extra_fuel

    st.header("\ud83d\udcca Fuel Summary")
    st.markdown(f"**Route Distance:** {distance_nm:.0f} NM")
    st.markdown(f"**Estimated Flight Time:** {est_time_hr:.2f} hrs")
    st.markdown(f"**Base Fuel Required:** {base_fuel:.0f} kg")
    st.markdown(f"**Extra Fuel Added:** {extra_fuel:.0f} kg")
    st.success(f"**Total Recommended Fuel:** {total_fuel:.0f} kg")

    with st.expander("Show Reasoning"):
        for reason in reasons:
            st.markdown(f"- {reason}")
else:
    st.warning("Please enter valid ICAO codes present in the database.")
    st.info(f"Available: {', '.join(airport_coords.keys())}")

