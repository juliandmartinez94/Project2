import streamlit as st
import pandas as pd
import requests
from datetime import datetime
import matplotlib.pyplot as plt

# No-auth list of all possible brewery types
BREWERY_TYPES = [
    "micro", "nano", "regional", "brewpub",
    "large", "planning", "bar", "contract",
    "proprietor", "closed"
]

API_URL = "https://api.openbrewerydb.org/v1/breweries"

st.set_page_config(page_title="🍺 Brewery Explorer", layout="wide")
st.title("🍺 Brewery Explorer")
st.write("""
    Explore US breweries by state, type, and update year.
    Filter, visualize, and map your results—all in one place!
""")

# --- Sidebar controls ---
with st.sidebar:
    st.header("🔍 Filters")

    # 1) State selector
    states = [
        "Alabama","Alaska","Arizona","Arkansas","California","Colorado","Connecticut","Delaware",
        "Florida","Georgia","Hawaii","Idaho","Illinois","Indiana","Iowa","Kansas","Kentucky",
        "Louisiana","Maine","Maryland","Massachusetts","Michigan","Minnesota","Mississippi",
        "Missouri","Montana","Nebraska","Nevada","New Hampshire","New Jersey","New Mexico",
        "New York","North Carolina","North Dakota","Ohio","Oklahoma","Oregon","Pennsylvania",
        "Rhode Island","South Carolina","South Dakota","Tennessee","Texas","Utah","Vermont",
        "Virginia","Washington","West Virginia","Wisconsin","Wyoming"
    ]
    state = st.selectbox("Select state:", states)

    # 2) How many to fetch
    result_count = st.slider(
        "Number of breweries to fetch:",
        min_value=10,
        max_value=50,
        step=10,
        value=20,
    )

    # 3) Brewery-type checkboxes
    st.markdown("### Filter by Brewery Type")
    selected_types = []
    for t in BREWERY_TYPES:
        checked = st.checkbox(
            label=t.capitalize(),
            value=(t in st.session_state.get("type_select", [])),
            key=f"sidebar_type_{t}"
        )
        if checked:
            selected_types.append(t)
    st.session_state["type_select"] = selected_types

    # 4) Fetch button
    fetch = st.button("🔄 Fetch breweries")


# --- Main pane ---
if fetch or "raw_data" in st.session_state:
    # On initial fetch, grab and store the raw JSON
    if fetch:
        st.info("Loading breweries…")
        params = {
            "by_state": state.lower().replace(" ", "_"),
            "per_page": result_count,
        }
        r = requests.get(API_URL, params=params)
        try:
            r.raise_for_status()
            data = r.json()
        except requests.exceptions.RequestException as e:
            st.error(f"Request error: {e}")
            st.stop()
        except ValueError as e:
            st.error(f"Error parsing JSON: {e}")
            st.stop()

        st.session_state["raw_data"] = data
        st.session_state["state"] = state

    # Reload persisted data
    data = st.session_state["raw_data"]
    state = st.session_state["state"]
    df = pd.DataFrame(data)

    # Apply sidebar type filters
    types_to_keep = st.session_state.get("type_select", [])
    if types_to_keep:
        df = df[df["brewery_type"].isin(types_to_keep)]

    # Show status and table
    st.success(f"Found {len(df)} breweries in {state}")
    st.subheader("Interactive Table")
    st.dataframe(df[["name", "brewery_type", "city", "state"]])

    # Auto-show map with hide option
    hide_map = st.checkbox("Hide map of locations", value=False)
    if not hide_map:
        st.subheader("📍 Brewery Locations")
        st.map(df.dropna(subset=["latitude", "longitude"])[["latitude", "longitude"]])

    if len(df) == 0:
        st.stop()

    # Custom-colored charts using matplotlib
    col1, col2 = st.columns(2)

    with col1:
        st.subheader("Count by Brewery Type")
        counts_type = df["brewery_type"].value_counts()
        fig1, ax1 = plt.subplots()
        ax1.bar(counts_type.index, counts_type.values, color="#DDA15E")
        ax1.set_xticklabels(counts_type.index, rotation=45, ha="right")
        ax1.set_ylabel("Count")
        fig1.tight_layout()
        st.pyplot(fig1)

    with col2:
        st.subheader("Breweries by City")
        counts_city = df["city"].value_counts().sort_index()
        fig2, ax2 = plt.subplots()
        ax2.plot(counts_city.index, counts_city.values, color="#DDA15E", marker="o")
        ax2.set_xticklabels(counts_city.index, rotation=45, ha="right")
        ax2.set_ylabel("Count")
        fig2.tight_layout()
        st.pyplot(fig2)

    # Raw JSON expander
    with st.expander("Raw JSON data"):
        st.json(data)

else:
    st.info("Choose filters on the left and click **Fetch breweries** to begin.")
