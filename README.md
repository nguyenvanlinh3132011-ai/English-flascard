# English-flascard
import streamlit as st
import random
import time
import base64

# --- 1. CẤU HÌNH & CSS (THÊM ANIMATION) ---
st.set_page_config(page_title="GRAMMARARENA FX", layout="wide", initial_sidebar_state="collapsed")

st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=Lexend:wght@400;700;900&display=swap');
    * { font-family: 'Lexend', sans-serif; }
    .stApp { background-color: #fdfaf6; }

    /* Hiệu ứng Rung lắc khi sai */
    @keyframes shake {
        0% { transform: translateX(0); }
        25% { transform: translateX(-10px); }
        50% { transform: translateX(10px); }
        75% { transform: translateX(-10px); }
        100% { transform: translateX(0); }
    }
    .shake-effect { animation: shake 0.4s ease-in-out; border-color: #ff4b4b !important; }

    /* Hiệu ứng Pulse khi đúng */
    @keyframes pulse {
        0% { transform: scale(1); }
        50% { transform: scale(1.02); }
        100% { transform: scale(1); }
    }
    .pulse-effect { animation: pulse 0.4s ease-in-out; border-color: #58cc02 !important; }

    .neu-card {
        border: 4px solid #000; border-radius: 15px; padding: 25px;
        box-shadow: 6px 6px 0px 0px #000; background: white; margin-bottom: 15px;
    }
    
    .header-bar {
        display: flex; justify-content: space-between; align-items: center;
        padding: 15px 25px; border: 4px solid #000; border-radius: 15px;
        background: white; margin-bottom: 25px; box-shadow: 5px 5px 0px 0px #000;
    }

    .welcome-banner {
        background: #d8b4fe; border: 4px solid #000; border-radius: 15px;
        padding: 25px; margin-bottom: 25px; box-shadow: 8px 8px 0px 0px #000;
        display: flex; justify-content: space-between; align-items: center;
    }

    .stButton>button {
        width: 100%; border: 4px solid #000 !important;
        box-shadow: 4px 4px 0px 0px #000 !important;
        background-color: #fff !important; color: #000 !important;
        font-weight: 900 !important; border-radius: 12px !important;
        height: 55px; text-transform: uppercase; transition: 0.1s;
    }
    .stButton>button:active { transform: translate(4px, 4px); box-shadow: 0px 0px 0px 0px #000 !important; }
</style>
""", unsafe_allow_html=True)

# --- 2. HÀM HỖ TRỢ HIỆU ỨNG (JS & AUDIO) ---
def trigger_fx(status):
    if status == "success":
        # Confetti + Ting sound
        st.balloons()
        html_code = """
            <audio autoplay><source src="https://www.soundjay.com/misc/sounds/bell-ringing-05.mp3" type="audio/mpeg"></audio>
        """
    else:
        # Buzz sound
        html_code = """
            <audio autoplay><source src="https://www.soundjay.com/misc/sounds/fail-trombone-01.mp3" type="audio/mpeg"></audio>
        """
    st.markdown(html_code, unsafe_allow_html=True)

# --- 3. DATA ENGINE (5 THÌ - INFINITY) ---
SUBJECTS = [
    {"n": "The Pilot", "p": "s"}, {"n": "My cousins", "p": "p"}, {"n": "She", "p": "s"}, 
    {"n": "Engineers", "p": "p"}, {"n": "I", "p": "i"}, {"n": "The robot", "p": "s"}
]
VERBS = [
    {"inf": "repair", "s": "repairs", "ed": "repaired", "ing": "repairing"},
    {"inf": "design", "s": "designs", "ed": "designed", "ing": "designing"},
    {"inf": "travel", "s": "travels", "ed": "traveled", "ing": "traveling"},
    {"inf": "eat", "s": "eats", "ed": "ate", "ing": "eating"},
    {"inf": "sleep", "s": "sleeps", "ed": "slept", "ing": "sleeping"}
]
ADVERBS = {
    "present_simple": ["usually", "every Sunday", "frequently"],
    "present_cont": ["at the moment", "right now", "currently"],
    "future_simple": ["tomorrow", "next week", "soon"],
    "past_simple": ["yesterday", "two hours ago", "last year"],
    "past_cont": ["at 9 PM last night", "while I was cooking"]
}

def generate_q(tense):
    sub = random.choice(SUBJECTS)
    v = random.choice(VERBS)
    if tense == "Present Simple":
        ans = v['s'] if sub['p'] == 's' else v['inf']
        q = f"{sub['n']} ________ ({v['inf']}) {random.choice(ADVERBS['present_simple'])}."
        opts = [v['s'], v['inf'], v['ing']]
    elif tense == "Present Continuous":
        be = "is" if sub['p'] == 's' else ("am" if sub['p'] == 'i' else "are")
        ans = f"{be} {v['ing']}"
        q = f"Look! {sub['n']} ________ ({v['inf']}) {random.choice(ADVERBS['present_cont'])}."
        opts = [ans, f"{be} {v['ed']}", v['s']]
    elif tense == "Past Simple":
        ans = v['ed']
        q = f"{sub['n']} ________ ({v['inf']}) the house {random.choice(ADVERBS['past_simple'])}."
        opts = [v['ed'], v['inf'], f"was {v['ing']}"]
    elif tense == "Past Continuous":
        be = "was" if (sub['p'] == 's' or sub['p'] == 'i') else "were"
        ans = f"{be} {v['ing']}"
        q = f"{sub['n']} ________ ({v['inf']}) {random.choice(ADVERBS['past_cont'])}."
        opts = [ans, v['ed'], f"is {v['ing']}"]
    elif tense == "Future Simple":
        ans = f"will {v['inf']}"
        q = f"Maybe {sub['n']} ________ ({v['inf']}) {random.choice(ADVERBS['future_simple'])}."
        opts = [ans, f"is {v['ing']}", v['ed']]
    return q, ans, list(set(opts))

# --- 4. SESSION STATE ---
if 'xp' not in st.session_state: st.session_state.xp = 0
if 'game_mode' not in st.session_state: st.session_state.game_mode = "Map"
if 'fx_type' not in st.session_state: st.session_state.fx_type = ""
if 'progress' not in st.session_state:
    st.session_state.progress = {t: 0 for t in ["Present Simple", "Present Continuous", "Past Simple", "Past Continuous", "Future Simple"]}
if 'flash_idx' not in st.session_state: st.session_state.flash_idx = 0

# --- 5. UI RENDER ---
# Header
st.markdown("""<div class="header-bar">
    <div style="font-weight:900; font-size:1.6rem;">🦉 GRAMMAR<span style="color:#9333ea;">ARENA</span></div>
    <div style="display:flex; gap:15px; align-items:center;">
        <span style="background:#fed7aa; border:3px solid #000; padding:5px 12px; border-radius:8px; font-weight:900;">🔥 1 DAYS</span>
        <div style="width:40px; height:40px; background:#fbbf24; border:3px solid #000; border-radius:50%; display:flex; align-items:center; justify-content:center; font-weight:900;">P1</div>
    </div>
</div>""", unsafe_allow_html=True)

if st.session_state.game_mode == "Map":
    # Banner
    st.markdown(f"""<div class="welcome-banner">
        <div><h1 style="margin:0; font-weight:900; font-size:2.2rem;">Welcome back, Player 1! 👋</h1><p style="margin:0; font-weight:700;">Rank: Rookie Master</p></div>
        <div style="background:white; border:4px solid #000; width:80px; height:80px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-weight:900; font-size:1.5rem; box-shadow:4px 4px 0px #000;">{st.session_state.xp}XP</div>
    </div>""", unsafe_allow_html=True)

    # Quest Map
    st.markdown("<h3 style='font-weight:900;'>QUEST MAP</h3>", unsafe_allow_html=True)
    quest_config = [
        ("Present Simple", "#bef264"), ("Present Continuous", "#7dd3fc"),
        ("Past Simple", "#fbbf24"), ("Past Continuous", "#ff90e8"),
        ("Future Simple", "#ff6b6b")
    ]
    col1, col2 = st.columns(2)
    for i, (name, color) in enumerate(quest_config):
        with (col1 if i % 2 == 0 else col2):
            st.markdown(f"""<div class="neu-card" style="background:{color};">
                <div style="font-weight:900; font-size:1.4rem;">{name}</div>
                <div style="font-weight:700;">Progress: {st.session_state.progress[name]}%</div>
                <div style="background:white; border:3px solid #000; height:15px; border-radius:10px; margin-top:10px;">
                    <div style="width:{st.session_state.progress[name]}%; background:#000; height:100%;"></div>
                </div>
            </div>""", unsafe_allow_html=True)
            if st.button(f"PLAY {name.upper()}", key=name):
                st.session_state.current_tense = name
                st.session_state.game_mode = "Quiz"
                st.rerun()

    # Flashcards & Leaderboard (CÂN ĐỐI)
    st.markdown("<br>", unsafe_allow_html=True)
    c_fl, c_lb = st.columns([2, 1])
    with c_fl:
        st.markdown(f"""<div class="neu-card">
            <h3 style="margin:0; font-weight:900;">🎴 FLASHCARDS</h3>
            <div style="font-size:2rem; font-weight:900; color:#9333ea; margin:15px 0;">Vocabulary #{st.session_state.flash_idx}</div>
            <div style="display:flex; gap:10px;">
                <button style="flex:1; padding:10px; border:3px solid #000; border-radius:10px; font-weight:900; background:#eee; cursor:pointer;">TOPIC: WORK</button>
                <button style="flex:1; padding:10px; border:3px solid #000; border-radius:10px; font-weight:900; background:#eee; cursor:pointer;">TOPIC: TECH</button>
            </div>
        </div>""", unsafe_allow_html=True)
        if st.button("NEXT CARD 🔄"): st.session_state.flash_idx += 1; st.rerun()

    with c_lb:
        st.markdown(f"""<div class="neu-card" style="background:#fde047;">
            <h3 style="margin:0; font-weight:900;">🏆 LIVE RANK</h3>
            <div style="font-weight:900; margin-top:10px;">#1 You ({st.session_state.xp} XP)</div>
            <div style="font-weight:800; color:#555;">#2 Bot_Genius (1,200 XP)</div>
            <hr style="border:1px solid #000">
            <div style="background:#ff4b4b; color:white; padding:5px; text-align:center; font-weight:900; border-radius:8px;">RESET ⚠️</div>
        </div>""", unsafe_allow_html=True)

elif st.session_state.game_mode == "Quiz":
    tense = st.session_state.current_tense
    if 'active_q' not in st.session_state:
        st.session_state.active_q = generate_q(tense)
    
    q_txt, q_ans, q_opts = st.session_state.active_q
    
    # Render Question with FX class
    fx_class = ""
    if st.session_state.fx_type == "success": fx_class = "pulse-effect"
    elif st.session_state.fx_type == "fail": fx_class = "shake-effect"

    st.markdown(f"<h1 style='text-align:center; font-weight:900;'>{tense.upper()}</h1>", unsafe_allow_html=True)
    st.markdown(f"""<div class="neu-card {fx_class}" style="text-align:center; padding:60px 20px;">
        <h2 style="font-weight:900; font-size:2.2rem;">{q_txt}</h2>
    </div>""", unsafe_allow_html=True)
    
    ans_cols = st.columns(len(q_opts))
    for i, opt in enumerate(q_opts):
        if ans_cols[i].button(opt, key=f"opt_{i}"):
            if opt == q_ans:
                st.session_state.fx_type = "success"
                st.session_state.xp += 50
                st.session_state.progress[tense] = min(100, st.session_state.progress[tense] + 5)
                trigger_fx("success")
            else:
                st.session_state.fx_type = "fail"
                trigger_fx("fail")
            
            time.sleep(0.8)
            st.session_state.fx_type = "" # Reset FX
            del st.session_state.active_q
            st.rerun()

    if st.button("⬅️ BACK TO MAP"):
        st.session_state.game_mode = "Map"
        if 'active_q' in st.session_state: del st.session_state.active_q
        st.rerun()
