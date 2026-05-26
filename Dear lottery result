import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
from pypdf import PdfReader
import re

# Page Setup
st.set_page_config(page_title="Nagaland Dear Lottery Analyzer", layout="wide", page_icon="🎟️")

st.title("🎯 Nagaland State Lottery Forensics Analyzer")
st.caption("Extract, analyze, and spot structural patterns from official Dear Lottery PDF sheets.")
st.markdown("---")

# --- CORE PROCESSING ENGINE: PDF TEXT SCRAPER ---
def parse_nagaland_pdf(pdf_file):
    """Extracts 4-digit numbers and series letters from official Nagaland PDF results."""
    try:
        reader = PdfReader(pdf_file)
        full_text = ""
        for page in reader.pages:
            text = page.extract_text()
            if text:
                full_text += text + "\n"
        
        # 1. Extract all 4-digit numbers (mainly looking for the 5th prize block of 100 numbers)
        # Using regex to find standalone 4-digit blocks
        all_4_digits = re.findall(r'\b\d{4}\b', full_text)
        
        # 2. Extract series alphabets found in top tier prizes (e.g., 52C, 88A, 99L)
        # Looking for 2 numbers followed by a capital letter (A, B, C, D, E, G, H, J, K, L)
        series_matches = re.findall(r'\b\d{2}[A-L]\b', full_text)
        letters = [match[-1] for match in series_matches]
        
        return all_4_digits, letters
    except Exception as e:
        st.error(f"Error parsing PDF file: {e}")
        return [], []

# --- SIDEBAR: LIVE FILE UPLOADER ---
with st.sidebar:
    st.header("📁 Upload Center")
    st.markdown("Download daily **1 PM, 6 PM, or 8 PM** PDF result sheets from the official portal and upload them here.")
    
    uploaded_files = st.file_uploader(
        "Upload Official Results (PDF Format)", 
        type=["pdf"], 
        accept_multiple_files=True
    )

# --- DATA GENERATION (LIVE OR DEMO fallback) ---
all_extracted_numbers = []
all_extracted_letters = []

if uploaded_files:
    # Process files uploaded by user
    with st.spinner("Analyzing uploaded PDF layouts..."):
        for file in uploaded_files:
            nums, lets = parse_nagaland_pdf(file)
            all_extracted_numbers.extend(nums)
            all_extracted_letters.extend(lets)
    
    if all_extracted_numbers:
        st.sidebar.success(f"Successfully scanned {len(uploaded_files)} PDF(s)!")
        # Create Dataframe from live data
        last_two_digits = [num[-2:] for num in all_extracted_numbers]
        df_prizes = pd.DataFrame({"4_digit": all_extracted_numbers, "last_2": last_two_digits})
        
        # Count alphabets
        letter_counts = pd.Series(all_extracted_letters).value_counts().to_dict() if all_extracted_letters else {'A':10, 'B':8}
    else:
        st.sidebar.warning("Uploaded file read successfully, but no matching 4-digit numbers found. Showing demo patterns.")
        uploaded_files = None # Fallback to demo if parsing found nothing

# FALLBACK DEMO DATA ENGINE (If no files uploaded)
if not uploaded_files:
    np.random.seed(42)
    # Generate mock 5th prize numbers (representing multiple days of draws)
    mock_prizes = [f"{np.random.randint(0, 10000):04d}" for _ in range(1200)]
    last_two = [num[-2:] for num in mock_prizes]
    df_prizes = pd.DataFrame({"4_digit": mock_prizes, "last_2": last_two})
    
    alphabets = ['A', 'B', 'C', 'D', 'E', 'G', 'H', 'J', 'K', 'L']
    letter_counts = {letter: int(np.random.randint(20, 70)) for letter in alphabets}
    
    st.info("💡 Showing **Demonstration Mock Data Patterns**. Drop your real Nagaland Result PDFs in the sidebar to switch to live analysis.")

# --- COMPACT MATRIX PREVIEW ---
with st.expander("👁️ View Processed Data Matrix Rows", expanded=False):
    col_v1, col_v2 = st.columns(2)
    with col_v1:
        st.subheader("Captured 4-Digit Combinations")
        st.dataframe(df_prizes, use_container_width=True, height=200)
    with col_v2:
        st.subheader("Series Letter Breakdown")
        st.json(letter_counts)

# --- INTERACTIVE DASHBOARD TABS ---
tab1, tab2, tab3 = st.tabs([
    "📊 4-Digit Ending Patterns", 
    "❄️ Series & Alphabet Analysis", 
    "🎲 Dear Smart Generator"
])

# TAB 1: ENDING PATTERNS
with tab1:
    st.subheader("Frequency Map of Drawn Numbers")
    st.markdown("This matrix tracks the distribution of the **Last 2 Digits (00 to 99)** across parsed prize pools to pinpoint repeating clusters.")
    
    # Process counts for X axis
    freq_df = df_prizes['last_2'].value_counts().sort_index().reset_index()
    freq_df.columns = ['Last Two Digits (00-99)', 'Total Times Drawn']
    
    # Generate Plotly Chart matching the blueprint color scheme
    fig = px.bar(
        freq_df, 
        x='Last Two Digits (00-99)', 
        y='Total Times Drawn',
        color='Total Times Drawn',
        color_continuous_scale='viridis',
    )
    fig.update_layout(xaxis_tickangle=-90, height=480, margin=dict(t=15, b=15, l=0, r=0))
    st.plotly_chart(fig, use_container_width=True)

# TAB 2: SERIES & ALPHABET ANALYSIS
with tab2:
    st.subheader("Alphabet Prefix Frequency")
    st.markdown("Monitors which structural Series Letters appear most frequently across high-tier prize allocations.")
    
    df_alpha = pd.DataFrame(list(letter_counts.items()), columns=['Series Letter', 'Draw Count']).sort_values(by='Draw Count', ascending=False)
    
    col_g1, col_g2 = st.columns()
    with col_g1:
        fig_alpha = px.bar(df_alpha, x='Series Letter', y='Draw Count', color='Draw Count', color_continuous_scale='plasma')
        fig_alpha.update_layout(height=350, margin=dict(t=10, b=10))
        st.plotly_chart(fig_alpha, use_container_width=True)
    with col_g2:
        st.markdown("### 📈 Analytical Highlights")
        if not df_alpha.empty:
            st.metric(label="Hottest Series Letter", value=df_alpha.iloc['Series Letter'], delta=f"Drawn {df_alpha.iloc['Draw Count']}x")
            st.metric(label="Coldest Series Letter", value=df_alpha.iloc[-1]['Series Letter'], delta=f"Drawn {df_alpha.iloc[-1]['Draw Count']}x", delta_color="inverse")

# TAB 3: SMART COMBINATION GENERATOR
with tab3:
    st.subheader("Dear-Format Ticket Generator")
    st.markdown("Assembles full ticket structures matching the authentic Nagaland serial layout based on chosen logic rules.")
    
    col_in1, col_in2 = st.columns(2)
    with col_in1:
        strategy = st.selectbox("Frequency Weighting Rule", ["Weighted Hot Endings", "Cold/Overdue Endings", "Pure Unbiased Random"])
    with col_in2:
        count = st.slider("Tickets to generate", 1, 5, 3)
        
    st.markdown("#### Suggested Combinations")
    
    letters_pool = list(letter_counts.keys()) if letter_counts else ['A', 'B', 'C', 'D']
    
    for i in range(count):
        # Generate standard Nagaland prefix: 2-digit number (usually between 50-99 or 10-49 depending on scheme)
        rand_prefix = np.random.randint(50, 100)
        rand_letter = np.random.choice(letters_pool)
        
        # Determine the 5-digit sequence
        if strategy == "Weighted Hot Endings" and not df_prizes.empty:
            # Pick a highly frequent ending 2-digit combo
            hot_end = df_prizes['last_2'].value_counts().index[i % len(df_prizes['last_2'].unique())]
            rand_front = f"{np.random.randint(0, 1000):03d}"
            five_digit = f"{rand_front}{hot_end}"
        elif strategy == "Cold/Overdue Endings" and not df_prizes.empty:
            # Pick a low-frequency ending 2-digit combo
            cold_end = df_prizes['last_2'].value_counts().index[-(i + 1) % len(df_prizes['last_2'].unique())]
            rand_front = f"{np.random.randint(0, 1000):03d}"
            five_digit = f"{rand_front}{cold_end}"
        else:
            five_digit = f"{np.random.randint(0, 100000):05d}"
            
        # Display as clear code lines resembling printed tickets
        st.code(f"🎟️ DEAR TICKET RECOMMENDATION: {rand_prefix}{rand_letter} {five_digit}", language="text")

# --- FOOTNOTE LEGAL DISCLAIMER ---
st.markdown("---")
st.warning(
    "⚠️ **Forensic Disclaimer:** This tool processes descriptive statistics for historical data tracking. "
    "It does not predict future independent random events or modify underlying lottery house edge profiles."
)
