import streamlit as st
from datetime import datetime
import base64
import os
import requests

import streamlit as st

st.set_page_config(
    page_title="Virtual Reading Room",
    page_icon="logo_for_vitual_reading_room.jpeg",  # আপনার ছবির নাম যদি অন্য কিছু হয়, তবে এখানে সেই নাম লিখবেন (যেমন: my_logo.png)
    layout="wide",
    initial_sidebar_state="expanded"
)

# ফাইল দ্রুত পড়ার জন্য ক্যাশিং ফাংশনটি ঠিক এখানে বসিয়ে দিন
@st.cache_data
def get_cached_pdf_base64(file_path):
    with open(file_path, "rb") as f:
        return base64.b64encode(f.read()).decode('utf-8')

st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
    
    
    * {
        font-family: 'Inter', sans-serif;
    }
    
    /* Main Background */
    .stApp {
        background-color: #F4F7FB;
    }
    
    /* Hide default Streamlit footer & hamburger */
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}
    header {visibility: hidden;}
    
    /* Custom Sidebar Styling */
    section[data-testid="stSidebar"] {
        background: linear-gradient(180deg, #0A2540 0%, #081B2E 100%);
        color: white;
        padding-top: 1rem;
    }
    
    section[data-testid="stSidebar"] .stButton button {
        background: transparent;
        color: #E2E8F0;
        border: none;
        text-align: left;
        width: 100%;
        padding: 10px 15px;
        border-radius: 8px;
        font-weight: 500;
        margin-bottom: 4px;
        transition: all 0.3s ease;
    }
    
    section[data-testid="stSidebar"] .stButton button:hover {
        background: rgba(255, 255, 255, 0.1);
        color: #FFFFFF;
        transform: translateX(4px);
    }
    
    /* Cards & Containers */
    .dashboard-card {
        background: #FFFFFF;
        padding: 20px;
        border-radius: 16px;
        box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
        margin-bottom: 20px;
        border: 1px solid #EBF0F5;
    }
    
    .hero-banner {
        background: linear-gradient(135deg, #E6F0FF 0%, #F5E6FF 100%);
        padding: 30px;
        border-radius: 20px;
        position: relative;
        overflow: hidden;
        margin-bottom: 24px;
        border: 1px solid #D6E4FF;
    }
    
    /* Stat Badges */
    .stat-badge {
        background: #F8FAFC;
        padding: 12px 18px;
        border-radius: 12px;
        border: 1px solid #E2E8F0;
        display: flex;
        align-items: center;
        gap: 12px;
    }
    
    /* Quick Nav Buttons custom style */
    .quick-nav-btn {
        background: white;
        border: 1px solid #E2E8F0;
        padding: 18px;
        border-radius: 14px;
        text-align: center;
        box-shadow: 0 2px 8px rgba(0,0,0,0.02);
        transition: all 0.2s;
    }
    .quick-nav-btn:hover {
        border-color: #3B82F6;
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.1);
    }
</style>
""", unsafe_allow_html=True)

if "authenticated" not in st.session_state:
    st.session_state.authenticated = False
if "student_name" not in st.session_state:
    st.session_state.student_name = "Lalon"
if "school_name" not in st.session_state:
    st.session_state.school_name = "Ideal School & College"
if "class_name" not in st.session_state:
    st.session_state.class_name = "Class 7"
if "roll_number" not in st.session_state:
    st.session_state.roll_number = "1042"
if "current_page" not in st.session_state:
    st.session_state.current_page = "Dashboard"
if "chat_messages" not in st.session_state:
    st.session_state.chat_messages = [
        {"role": "assistant", "content": "Hi Lalon! 👋 I'm your AI Study Assistant. Ask me anything about your studies!"}
    ]

def render_login_page():
    # কাস্টম CSS স্টাইল (ফন্ট, কালার, ব্যাকগ্রাউন্ড ও ইনপুট বক্স সুন্দর করার জন্য)
    st.markdown("""
        <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');

        /* পুরো পেজের ব্যাকগ্রাউন্ড ও ফন্ট স্টাইল */
        .stApp {
            background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
            font-family: 'Poppins', sans-serif;
        }

        /* ফর্মের কন্টেইনার স্টাইল */
        div[data-testid="stForm"] {
            background: #ffffff;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(15, 23, 42, 0.08);
            border: 1px solid rgba(226, 232, 240, 0.8);
        }

        /* ইনপুট লেবেলগুলোর ফন্ট ও কালার */
        .stTextInput label, .stSelectbox label {
            font-family: 'Poppins', sans-serif;
            font-weight: 500;
            color: #334155;
            font-size: 14px;
        }

        /* ইনপুট টেক্সট বক্সগুলোর বর্ডার ও রেডিয়াস */
        .stTextInput input, .stSelectbox div[data-baseweb="select"] {
            border-radius: 12px !important;
            border: 1.5px solid #cbd5e1 !important;
            background-color: #f8fafc !important;
            font-family: 'Poppins', sans-serif;
        }

        /* ইনপুট বক্সে ফোকাস করলে গ্লোয়িং এফেক্ট */
        .stTextInput input:focus, .stSelectbox div[data-baseweb="select"]:focus-within {
            border-color: #2563eb !important;
            box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15) !important;
        }

        /* সাবমিট বাটন স্টাইল */
        .stFormSubmitButton button {
            background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%);
            color: white;
            font-family: 'Poppins', sans-serif;
            font-weight: 600;
            border-radius: 12px;
            padding: 12px 20px;
            border: none;
            box-shadow: 0 10px 20px rgba(37, 99, 235, 0.25);
            transition: all 0.3s ease;
        }

        .stFormSubmitButton button:hover {
            background: linear-gradient(135deg, #1d4ed8 0%, #1e40af 100%);
            box-shadow: 0 15px 25px rgba(37, 99, 235, 0.4);
            transform: translateY(-2px);
        }
        </style>
    """, unsafe_allow_html=True)

    st.markdown("<br>", unsafe_allow_html=True)
    
    # ১. লোগো এবং অ্যাপের নাম (নিউরাল নেটওয়ার্ক/টেকনিক্যাল আইকনসহ)
    st.markdown("""
        <div style="text-align: center; margin-bottom: 25px;">
            <div style="display: inline-flex; background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%); color: white; padding: 18px; border-radius: 22px; font-size: 34px; box-shadow: 0 12px 30px rgba(37,99,235,0.35);">⚛️</div>
            <h1 style="color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 700; margin-top: 14px; font-size: 30px; letter-spacing: -0.5px;">Virtual Reading Room</h1>
            <p style="color: #64748b; font-family: 'Poppins', sans-serif; font-size: 15px; font-weight: 400; margin-top: 4px;">Smart Study with AI Assistant</p>
        </div>
    """, unsafe_allow_html=True)

    # ২. সুন্দর ও সেন্টার্ড লগইন ফর্ম
    col1, col2, col3 = st.columns([1, 1.3, 1])

    with col2:
        with st.form("login_form"):
            st.markdown("<h3 style='text-align: center; color: #1e293b; font-family: \"Poppins\", sans-serif; font-weight: 600; font-size: 20px; margin-bottom: 20px;'>Student Portal Login</h3>", unsafe_allow_html=True)
            
            # স্কুল ও স্টুডেন্ট নাম
            school = st.text_input("School Name & Location", value=st.session_state.get("school_name", ""))
            name = st.text_input("Student Name", value=st.session_state.get("student_name", ""))
            
            # ক্লাস, সেকশন ও রোল নম্বর একই সারিতে
            c1, c2, c3 = st.columns([2, 1.2, 1.2])
            with c1:
                selected_class = st.selectbox("Class", ["ষষ্ঠ শ্রেণি", "সপ্তম শ্রেণি", "অষ্টম শ্রেণি", "নবম শ্রেণি", "দশম শ্রেণি"], index=1)
            with c2:
                section = st.text_input("Section", value=st.session_state.get("section", ""))
            with c3:
                roll = st.text_input("Roll Number", value=st.session_state.get("roll_number", ""))
            
            st.markdown("<div style='margin-top: 10px;'></div>", unsafe_allow_html=True)
            submit = st.form_submit_button("🚀 Get Started", use_container_width=True)
            
            if submit:
                if name.strip():
                    st.session_state.school_name = school
                    st.session_state.student_name = name
                    st.session_state.class_name = selected_class
                    st.session_state.section = section
                    st.session_state.roll_number = roll
                    st.session_state.authenticated = True
                    st.rerun()
                else:
                    st.error("⚠️ Please enter your name to proceed.")

      
def render_dashboard():
    # কাস্টম ড্যাশবোর্ড CSS এবং স্টাইল ডিজাইন
    st.markdown("""
        <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700;800&display=swap');

        .dash-container {
            font-family: 'Poppins', sans-serif;
        }

        /* প্রিমিয়াম কার্ড ডিজাইন */
        .dashboard-card {
            background: #ffffff;
            padding: 22px;
            border-radius: 20px;
            border: 1px solid rgba(226, 232, 240, 0.8);
            box-shadow: 0 10px 30px rgba(15, 23, 42, 0.05);
            margin-bottom: 20px;
            transition: all 0.3s ease;
        }
        
        .dashboard-card:hover {
            box-shadow: 0 15px 35px rgba(37, 99, 235, 0.1);
            transform: translateY(-2px);
        }

        /* কুইক অ্যাকশন বাটনগুলোর প্রিমিয়াম স্টাইল */
        .stButton button {
            font-family: 'Poppins', sans-serif;
            border-radius: 14px !important;
            font-weight: 600 !important;
            border: 1px solid #e2e8f0 !important;
            background: linear-gradient(135deg, #ffffff 0%, #f8fafc 100%) !important;
            color: #1e293b !important;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03) !important;
            transition: all 0.3s ease !important;
        }

        .stButton button:hover {
            background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%) !important;
            color: white !important;
            border-color: #2563eb !important;
            box-shadow: 0 8px 20px rgba(37, 99, 235, 0.3) !important;
            transform: translateY(-2px);
        }
        </style>
    """, unsafe_allow_html=True)

    from datetime import datetime
    import os

    # Top Header Bar (Modern Gradient & Clean UI)
    head_col1, head_col2, head_col3 = st.columns([2.2, 3.2, 1.6])
    
    with head_col1:
        st.markdown(f"""
            <div style="display: flex; align-items: center; gap: 14px; padding-top: 5px;">
                <div style="background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%); color: white; padding: 12px; border-radius: 16px; font-size: 22px; box-shadow: 0 8px 20px rgba(37,99,235,0.3);">📚</div>
                <div>
                    <h2 style="margin: 0; font-size: 20px; color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 800;">Virtual Reading Room</h2>
                    <p style="margin: 0; font-size: 11px; color: #64748b; font-family: 'Poppins', sans-serif;">Smart Study with AI Assistant</p>
                </div>
            </div>
        """, unsafe_allow_html=True)

    with head_col2:
        search_query = st.text_input("Search", placeholder="🔍 Search books, notes, exams, routines...", label_visibility="collapsed")
        if search_query:
            q = search_query.lower()
            app_database = [
                {"title": "Mathematics Chapter 3: Linear Equations", "type": "Chapter/Book", "desc": "Current reading chapter on math equations."},
                {"title": "Bangla 1st Paper Notes & Kobita", "type": "Notes", "desc": "Important notes and summaries."},
                {"title": "English Grammar & Composition", "type": "Book", "desc": "Comprehensive English guide for Class 7."},
                {"title": "Upcoming Exam: Mathematics", "type": "Exam", "desc": "12 Days Left for your next exam."},
                {"title": "Upcoming Exam: Science", "type": "Exam", "desc": "15 Days Left for science exam."},
                {"title": "Daily Routine: Mathematics & Science", "type": "Routine", "desc": "04:00 PM - 06:00 PM study schedule."},
                {"title": "Study Progress: Bangla & Math", "type": "Progress", "desc": "Overall academic performance and mastery."}
            ]
            matched_items = [item for item in app_database if q in item["title"].lower() or q in item["desc"].lower() or q in item["type"].lower()]
            
            assets_dir = "assets"
            pdf_files = []
            if os.path.exists(assets_dir):
                try:
                    pdf_files = [f for f in os.listdir(assets_dir) if f.lower().endswith('.pdf')]
                except:
                    pdf_files = []
            matched_pdfs = [f for f in pdf_files if q in f.lower()]

            total_found = len(matched_items) + len(matched_pdfs)
            if total_found > 0:
                st.markdown(f"<div style='background: #1e293b; color: white; padding: 8px 12px; border-radius: 10px; font-size: 12px; margin-top: 5px; font-family: Poppins;'><b>✨ Search Results ({total_found} found):</b></div>", unsafe_allow_html=True)
                for idx, file_name in enumerate(matched_pdfs):
                    file_path = os.path.join(assets_dir, file_name)
                    clean_title = file_name.replace('.pdf', '').replace('.PDF', '').replace('_', ' ')
                    st.markdown(f"""
                        <div style='background: #f1f5f9; padding: 10px; border-radius: 8px; margin-top: 6px; font-size: 12px; color: #0f172a; border-left: 4px solid #2563EB;'>
                        <b>[PDF Library] {clean_title}</b>
                        </div>
                    """, unsafe_allow_html=True)
                    try:
                        with open(file_path, "rb") as pdf_file:
                            st.download_button(label=f"📥 Download / Read: {clean_title}", data=pdf_file, file_name=file_name, mime="application/pdf", key=f"global_dl_{idx}")
                    except Exception as err:
                        st.error(f"ফাইল ওপেন করতে সমস্যা হয়েছে")

                for idx, item in enumerate(matched_items):
                    st.markdown(f"""
                        <div style='background: #f8fafc; padding: 10px; border-radius: 8px; margin-top: 6px; font-size: 12px; color: #0f172a; border-left: 4px solid #10b981;'>
                        <b>{item['title']}</b> <span style='color: #10b981; font-weight: bold;'>[{item['type']}]</span><br>
                        <span style='color: #64748b;'>{item['desc']}</span>
                        </div>
                    """, unsafe_allow_html=True)
            else:
                st.warning(f"No results found for '{search_query}'. Try searching 'bangla', 'exam', or 'routine'.")
        else:
            pass

    with head_col3:
        student_initial = st.session_state.student_name[0] if "student_name" in st.session_state and st.session_state.student_name else "S"
        student_name = st.session_state.get("student_name", "Student")
        student_class = st.session_state.get("class_name", "Class 7")
        
        st.markdown(f"""
            <div style="display: flex; align-items: center; justify-content: flex-end; gap: 10px; padding-top: 4px;">
                <div style="background: #eff6ff; padding: 10px; border-radius: 50%; font-size: 15px; position: relative; cursor: pointer; border: 1px solid #dbeafe;" title="Notifications">
                    🔔<span style="position: absolute; top: 0px; right: 0px; background: #ef4444; color: white; font-size: 9px; width: 16px; height: 16px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold;">3</span>
                </div>
                <div style="display: flex; align-items: center; gap: 8px; background: #ffffff; padding: 6px 12px; border-radius: 24px; border: 1px solid #e2e8f0; box-shadow: 0 4px 12px rgba(0,0,0,0.03);">
                    <div style="background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%); height: 32px; width: 32px; border-radius: 50%; display: flex; color: white; align-items: center; justify-content: center; font-weight: bold; font-size: 13px;">{student_initial}</div>
                    <div style="text-align: left;">
                        <div style="font-size: 12px; font-weight: 700; color: #1e293b; line-height: 1.2;">{student_name}</div>
                        <div style="font-size: 10px; color: #64748b; font-weight: 500;">{student_class}</div>
                    </div>
                </div>
            </div>
        """, unsafe_allow_html=True)

    st.markdown("<br>", unsafe_allow_html=True)

    # Main Grid Layout: Left Column (Content) & Right Column (AI Assistant Panel)
    col_main, col_ai = st.columns([2.3, 1.2], gap="large")

    with col_main:
        current_hour = datetime.now().hour
        if 5 <= current_hour < 12:
            greeting = "Good Morning"
        elif 12 <= current_hour < 17:
            greeting = "Good Afternoon"
        elif 17 <= current_hour < 21:
            greeting = "Good Evening"
        else:
            greeting = "Good Night"

        # Hero Greeting Banner (Vibrant Gradient & Glassmorphism Feel)
        st.markdown(f"""
            <div style="background: linear-gradient(135deg, #eef2ff 0%, #f3e8ff 100%); padding: 30px; border-radius: 24px; border: 1px solid #e0e7ff; position: relative; margin-bottom: 24px; box-shadow: 0 10px 30px rgba(37,99,235,0.06);">
                <div style="max-width: 100%;">
                    <h2 style="color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 800; margin-bottom: 6px; font-size: 26px;">{greeting}, {student_name}! 👋</h2>
                    <p style="color: #475569; font-family: 'Poppins', sans-serif; font-size: 13px; margin-bottom: 20px; font-weight: 500;">Keep going! You are doing phenomenal in your studies today.</p>
                    <div style="display: flex; gap: 14px; flex-wrap: wrap;">
                        <div style="background: white; padding: 14px 18px; border-radius: 16px; border: 1px solid #e2e8f0; display: flex; align-items: center; gap: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.03);">
                            <span style="font-size: 22px;">🎯</span>
                            <div>
                                <div style="font-size: 11px; color: #64748b; font-weight: 600;">Today's Goal</div>
                                <div style="font-size: 15px; font-weight: 800; color: #0f172a;">2.5 hours</div>
                            </div>
                        </div>
                        <div style="background: white; padding: 14px 18px; border-radius: 16px; border: 1px solid #e2e8f0; display: flex; align-items: center; gap: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.03);">
                            <span style="font-size: 22px;">📈</span>
                            <div>
                                <div style="font-size: 11px; color: #64748b; font-weight: 600;">Today's Progress</div>
                                <div style="font-size: 15px; font-weight: 800; color: #0f172a;">72%</div>
                            </div>
                        </div>
                        <div style="background: white; padding: 14px 18px; border-radius: 16px; border: 1px solid #e2e8f0; display: flex; align-items: center; gap: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.03);">
                            <span style="font-size: 22px;">⏱️</span>
                            <div>
                                <div style="font-size: 11px; color: #64748b; font-weight: 600;">Study Time</div>
                                <div style="font-size: 15px; font-weight: 800; color: #0f172a;">1h 30m</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        """, unsafe_allow_html=True)

        # Continue Reading Section
        st.markdown("""
            <div class="dashboard-card">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px;">
                    <div>
                        <h4 style="margin: 0; color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 18px;">Continue Reading</h4>
                        <p style="margin: 0; font-size: 12px; color: #64748b;">You were reading: Mathematics Chapter 3</p>
                    </div>
                </div>
                <div style="display: flex; gap: 20px; align-items: center; background: #f8fafc; padding: 18px; border-radius: 16px; border: 1px solid #e2e8f0;">
                    <div style="background: linear-gradient(135deg, #2563eb 0%, #1d4ed8 100%); color: white; padding: 18px; border-radius: 14px; text-align: center; min-width: 100px; box-shadow: 0 8px 20px rgba(37,99,235,0.25);">
                        <div style="font-size: 16px; font-weight: 800;">গণিত</div>
                        <div style="font-size: 10px; opacity: 0.9; font-weight: 500;">শ্রেণি ৭</div>
                    </div>
                    <div style="flex-grow: 1;">
                        <div style="font-weight: 700; color: #1e293b; font-size: 14px; margin-bottom: 4px;">Chapter 3: Linear Equations</div>
                        <div style="font-size: 12px; color: #64748b; margin-bottom: 12px;">Mathematics Advanced Guide</div>
                        <div style="background: #e2e8f0; border-radius: 99px; height: 8px; width: 100%; overflow: hidden;">
                            <div style="background: linear-gradient(90deg, #2563eb 0%, #3b82f6 100%); width: 60%; height: 100%; border-radius: 99px;"></div>
                        </div>
                    </div>
                    <div style="text-align: right;">
                        <div style="font-size: 14px; font-weight: 800; color: #2563eb;">60%</div>
                    </div>
                </div>
                <div style="display: flex; gap: 10px; margin-top: 16px;">
                    <button style="background: #2563eb; color: white; border: none; padding: 10px 18px; border-radius: 10px; font-size: 12px; font-weight: 600; cursor: pointer; box-shadow: 0 4px 12px rgba(37,99,235,0.2);">Continue</button>
                    <button style="background: white; color: #475569; border: 1px solid #cbd5e1; padding: 10px 18px; border-radius: 10px; font-size: 12px; font-weight: 600; cursor: pointer;">Read From Start</button>
                    <button style="background: white; color: #475569; border: 1px solid #cbd5e1; padding: 10px 18px; border-radius: 10px; font-size: 12px; font-weight: 600; cursor: pointer;">Download PDF</button>
                </div>
            </div>
        """, unsafe_allow_html=True)

        # Quick Feature Cards Grid
        qc1, qc2, qc3, qc4, qc5 = st.columns(5)
        with qc1:
            if st.button("📚 My Books", use_container_width=True):
                st.session_state.current_page = "My Books"
                st.rerun()
        with qc2:
            if st.button("🤖 Ask AI", use_container_width=True):
                st.session_state.current_page = "AI Assistant"
                st.rerun()
        with qc3:
            if st.button("📝 Take Quiz", use_container_width=True):
                st.session_state.current_page = "Quiz & Practice"
                st.rerun()
        with qc4:
            if st.button("📅 Study Plan", use_container_width=True):
                st.session_state.current_page = "Study Planner"
                st.rerun()
        with qc5:
            if st.button("📊 Progress", use_container_width=True):
                st.session_state.current_page = "Progress"
                st.rerun()

        st.markdown("<br>", unsafe_allow_html=True)

        # Upcoming Exams & Study Progress Section
        col_sub1, col_sub2 = st.columns(2, gap="medium")
        
        with col_sub1:
            st.markdown("""
                <div class="dashboard-card" style="height: 100%;">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px;">
                        <h4 style="margin: 0; color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 16px;">Upcoming Exams</h4>
                        <span style="font-size: 12px; color: #2563eb; font-weight: 600; cursor: pointer;">View All</span>
                    </div>
                    <div style="font-size: 12px; color: #475569;">
                        <div style="display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #f1f5f9;">
                            <span style="font-weight: 600; color: #1e293b;">Mathematics</span><span>15 Oct 2026</span><span style="background: #fee2e2; color: #dc2626; padding: 3px 10px; border-radius: 8px; font-size: 10px; font-weight: 700;">12 Days Left</span>
                        </div>
                        <div style="display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #f1f5f9;">
                            <span style="font-weight: 600; color: #1e293b;">Science</span><span>18 Oct 2026</span><span style="background: #fef3c7; color: #d97706; padding: 3px 10px; border-radius: 8px; font-size: 10px; font-weight: 700;">15 Days Left</span>
                        </div>
                        <div style="display: flex; justify-content: space-between; align-items: center; padding: 10px 0; border-bottom: 1px solid #f1f5f9;">
                            <span style="font-weight: 600; color: #1e293b;">English</span><span>25 Oct 2026</span><span style="background: #e0e7ff; color: #4f46e5; padding: 3px 10px; border-radius: 8px; font-size: 10px; font-weight: 700;">22 Days Left</span>
                        </div>
                        <div style="display: flex; justify-content: space-between; align-items: center; padding: 10px 0;">
                            <span style="font-weight: 600; color: #1e293b;">ICT</span><span>30 Oct 2026</span><span style="background: #e0e7ff; color: #4f46e5; padding: 3px 10px; border-radius: 8px; font-size: 10px; font-weight: 700;">27 Days Left</span>
                        </div>
                    </div>
                </div>
            """, unsafe_allow_html=True)

        with col_sub2:
            st.markdown("""
                <div class="dashboard-card" style="height: 100%;">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px;">
                        <h4 style="margin: 0; color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 16px;">Study Progress</h4>
                    </div>
                    <div style="font-size: 12px; color: #475569;">
                        <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span style="font-weight: 600; color: #1e293b;">Bangla</span><span style="font-weight: 700; color: #10b981;">80%</span></div>
                        <div style="background: #e2e8f0; height: 8px; border-radius: 99px; margin-bottom: 12px; overflow: hidden;"><div style="background: #10b981; width: 80%; height: 100%; border-radius: 99px;"></div></div>
                        
                        <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span style="font-weight: 600; color: #1e293b;">English</span><span style="font-weight: 700; color: #3b82f6;">65%</span></div>
                        <div style="background: #e2e8f0; height: 8px; border-radius: 99px; margin-bottom: 12px; overflow: hidden;"><div style="background: #3b82f6; width: 65%; height: 100%; border-radius: 99px;"></div></div>
                        
                        <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span style="font-weight: 600; color: #1e293b;">Mathematics</span><span style="font-weight: 700; color: #f97316;">75%</span></div>
                        <div style="background: #e2e8f0; height: 8px; border-radius: 99px; margin-bottom: 12px; overflow: hidden;"><div style="background: #f97316; width: 75%; height: 100%; border-radius: 99px;"></div></div>
                        
                        <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span style="font-weight: 600; color: #1e293b;">Science</span><span style="font-weight: 700; color: #8b5cf6;">70%</span></div>
                        <div style="background: #e2e8f0; height: 8px; border-radius: 99px; overflow: hidden;"><div style="background: #8b5cf6; width: 70%; height: 100%; border-radius: 99px;"></div></div>
                    </div>
                </div>
            """, unsafe_allow_html=True)

    with col_ai:
        # AI Study Assistant Chat & Task Widget Column
        st.markdown("""
            <div class="dashboard-card" style="border: 2px solid #2563eb; background: linear-gradient(180deg, #ffffff 0%, #f8fafc 100%);">
                <div style="display: flex; align-items: center; gap: 12px; margin-bottom: 14px;">
                    <div style="background: #2563eb; color: white; padding: 10px; border-radius: 50%; font-size: 16px; box-shadow: 0 4px 12px rgba(37,99,235,0.3);">🤖</div>
                    <div>
                        <h4 style="margin: 0; color: #0f172a; font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 16px;">AI Study Assistant</h4>
                        <p style="margin: 0; font-size: 11px; color: #10b981; font-weight: 600;">🟢 Online & Ready</p>
                    </div>
                </div>
                <div style="background: #eff6ff; padding: 14px; border-radius: 12px; font-size: 13px; color: #1e293b; margin-bottom: 14px; border: 1px solid #dbeafe;">
                    Hi <b>{student_name}</b>!<br>I'm your AI Study Assistant. Ask me anything about your studies!
                </div>
            </div>
        """.format(student_name=student_name), unsafe_allow_html=True)

        # Quick Prompt Buttons
        st.markdown("<p style='font-size: 13px; font-weight: 700; color: #334155; margin-bottom: 8px;'>✨ Quick AI Tools:</p>", unsafe_allow_html=True)
        
        # নিশ্চিত করুন যে session_state-এ chat_messages ইনিশিয়ালাইজ করা আছে
        if "chat_messages" not in st.session_state:
            st.session_state.chat_messages = []

        col_p1, col_p2 = st.columns(2, gap="small")
        with col_p1:
            if st.button("💡 Explain a topic", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Explain a topic"})
                st.session_state.chat_messages.append({"role": "assistant", "content": f"Sure, {student_name}! Which subject or topic would you like me to explain for {student_class}?"})
                st.rerun()
        with col_p2:
            if st.button("🔢 Solve a problem", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Solve a problem"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "Please type or paste your math/science problem here and I'll solve it step-by-step."})
                st.rerun()

        col_p3, col_p4 = st.columns(2, gap="small")
        with col_p3:
            if st.button("📝 Summarize", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Summarize a chapter"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "Send me the text or chapter name you want summarized."})
                st.rerun()
        with col_p4:
            if st.button("🌐 Translate", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Translate text"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "I can translate between English and Bengali instantly. What would you like translated?"})
                st.rerun()

        # Interactive Chat Box
        st.markdown("<br>", unsafe_allow_html=True)
        for msg in st.session_state.chat_messages[-4:]:
            bg = "#eff6ff" if msg["role"] == "user" else "#f1f5f9"
            align = "right" if msg["role"] == "user" else "left"
            border_color = "#2563eb" if msg["role"] == "user" else "#cbd5e1"
            st.markdown(f"""
                <div style="background: {bg}; padding: 10px 14px; border-radius: 12px; margin-bottom: 8px; font-size: 12px; text-align: {align}; border-left: 3px solid {border_color};">
                <b>{'You' if msg['role']=='user' else 'AI Assistant'}:</b> {msg['content']}
                </div>
            """, unsafe_allow_html=True)

        user_input = st.text_input("Ask AI Question", placeholder="Type your question here...", label_visibility="collapsed")
        if st.button("🚀 Send Message", use_container_width=True):
            if user_input:
                st.session_state.chat_messages.append({"role": "user", "content": user_input})
                st.session_state.chat_messages.append({"role": "assistant", "content": f"That's a great question regarding '{user_input}'. As your AI tutor for {student_class}, I recommend reviewing chapter notes or practicing related exercises."})
                st.rerun()

        # Recent Notifications
        st.markdown("""
            <div class="dashboard-card" style="margin-top: 20px;">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;">
                    <h4 style="margin: 0; font-size: 14px; font-weight: 700; color: #0f172a;">🔔 Recent Notifications</h4>
                    <span style="font-size: 11px; color: #2563eb; font-weight: 600; cursor: pointer;">View All</span>
                </div>
                <div style="font-size: 12px; color: #475569;">
                    <div style="padding: 8px 0; border-bottom: 1px solid #f1f5f9;">📌 Tomorrow is your Mathematics exam.<br><span style="font-size: 10px; color: #94a3b8;">2 hours ago</span></div>
                    <div style="padding: 8px 0; border-bottom: 1px solid #f1f5f9;">📌 You have 2 unfinished chapters.<br><span style="font-size: 10px; color: #94a3b8;">4 hours ago</span></div>
                    <div style="padding: 8px 0;">🎉 Weekly study goal almost complete!<br><span style="font-size: 10px; color: #94a3b8;">6 hours ago</span></div>
                </div>
            </div>
        """, unsafe_allow_html=True)


import streamlit as st
import os
import os.path as path
import base64
import requests
import sqlite3
from datetime import datetime

# ডেটাবেজ ইনিশিয়ালাইজেশন এবং স্বয়ংক্রিয়ভাবে missing column যোগ করার লজিক
def init_db():
    conn = sqlite3.connect("subject_notes.db")
    cursor = conn.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS notes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT,
        content TEXT,
        subject TEXT,
        note_type TEXT
    )
    """)
    try:
        cursor.execute("SELECT created_at FROM notes LIMIT 1")
    except sqlite3.OperationalError:
        cursor.execute("ALTER TABLE notes ADD COLUMN created_at TEXT")
    
    conn.commit()
    conn.close()

init_db()

# পিডিএফ ফাইল মেমোরিতে ফাস্ট লোড করার জন্য ক্যাশিং ফাংশন
@st.cache_data
def load_pdf_as_base64(file_path):
    with open(file_path, "rb") as f:
        return base64.b64encode(f.read()).decode('utf-8')

def render_my_books():
    # ১. Gemini API Key
    GEMINI_API_KEY = "YOUR_GEMINI_API_KEY_HERE"

    def ask_gemini_rest_api(prompt_text):
        if not GEMINI_API_KEY or GEMINI_API_KEY == "YOUR_GEMINI_API_KEY_HERE":
            return "API Key সমস্যা হচ্ছে। দয়া করে 'Gemini Chatbot' বাটনে ক্লিক করো।"
        
        models_to_try = ["gemini-1.5-flash", "gemini-1.5-pro", "gemini-2.0-flash"]
        for model_name in models_to_try:
            url = f"https://generativelanguage.googleapis.com/v1beta/models/{model_name}:generateContent?key={GEMINI_API_KEY}"
            headers = {'Content-Type': 'application/json'}
            payload = {"contents": [{"parts": [{"text": prompt_text}]}]}
            try:
                response = requests.post(url, json=payload, headers=headers, timeout=15)
                if response.status_code == 200:
                    data = response.json()
                    return data['candidates'][0]['content']['parts'][0]['text']
            except Exception:
                continue
        return "API Key সমস্যা হচ্ছে। দয়া করে সঠিক API Key সেট করুন।"

    # ২. প্রিমিয়াম CSS স্টাইল
    st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css?family=Hind+Siliguri:wght@400;500;600;700&family=Outfit:wght@400;600;800&display=swap');
    
    .kids-magic-wrapper {
        font-family: 'Outfit', 'Hind Siliguri', sans-serif;
        background: linear-gradient(135deg, #f0fdf4 0%, #eff6ff 50%, #fdf4ff 100%);
        padding: 25px;
        border-radius: 24px;
        border: 2px solid #bae6fd;
        box-shadow: 0 15px 30px rgba(14, 165, 233, 0.08);
        color: #1e293b;
        margin-bottom: 20px;
    }
    .kids-banner {
        background: linear-gradient(135deg, #38bdf8 0%, #818cf8 50%, #c084fc 100%);
        padding: 18px;
        border-radius: 16px;
        color: white;
        text-align: center;
        margin-bottom: 20px;
        box-shadow: 0 8px 20px rgba(56, 189, 248, 0.25);
    }
    .motivation-quote-box {
        background: #ffffff;
        padding: 14px 18px;
        border-radius: 14px;
        border-left: 5px solid #10b981;
        border: 1px solid #e2e8f0;
        margin-bottom: 20px;
        box-shadow: 0 6px 15px rgba(16, 185, 129, 0.06);
    }
    .kids-book-card {
        background: #ffffff;
        padding: 16px;
        border-radius: 14px;
        border: 1px solid #e0f2fe;
        border-top: 4px solid #0ea5e9;
        box-shadow: 0 6px 15px rgba(14, 165, 233, 0.06);
        margin-bottom: 12px;
        transition: all 0.3s ease;
    }
    .kids-book-card:hover {
        transform: translateY(-3px);
        box-shadow: 0 10px 20px rgba(14, 165, 233, 0.15);
        border-color: #38bdf8;
    }
    .stButton button {
        font-family: 'Outfit', 'Hind Siliguri', sans-serif !important;
        border-radius: 10px !important;
        font-weight: 700 !important;
        background: linear-gradient(135deg, #0ea5e9 0%, #2563eb 100%) !important;
        color: white !important;
        border: none !important;
        box-shadow: 0 4px 12px rgba(14, 165, 233, 0.25) !important;
        transition: all 0.3s ease !important;
    }
    .stButton button:hover {
        background: linear-gradient(135deg, #0284c7 0%, #1d4ed8 100%) !important;
        transform: translateY(-2px);
    }
    </style>
    """, unsafe_allow_html=True)


import streamlit as str_lib
import os
from os.path import basename, exists, join, dirname, abspath
from os import listdir, makedirs, remove, rename
from datetime import datetime
import time

def render_my_books():
    global st
    st = str_lib
    
    # CSS স্টাইল ও প্রিমিয়াম লুক
    st.markdown("""
        <style>
        @import url('https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700;800&family=Hind+Siliguri:wght@400;600;700&display=swap');
        
        .reading-room-wrapper {
            font-family: 'Outfit', 'Hind Siliguri', sans-serif;
            background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
            padding: 24px;
            border-radius: 24px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.03);
            margin-bottom: 25px;
        }
        
        .hero-banner {
            background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 50%, #2563eb 100%);
            padding: 32px 25px;
            border-radius: 20px;
            color: #ffffff;
            text-align: center;
            box-shadow: 0 12px 28px rgba(99, 102, 241, 0.28);
            margin-bottom: 20px;
        }
        
        .motivation-box {
            background: #ffffff;
            border-left: 5px solid #10b981;
            padding: 15px 20px;
            border-radius: 14px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.03);
            margin-bottom: 20px;
            border: 1px solid #e2e8f0;
        }
        
        .kids-book-card {
            background: #ffffff;
            padding: 16px;
            border-radius: 16px;
            border: 1px solid #e0e7ff;
            box-shadow: 0 6px 16px rgba(99, 102, 241, 0.06);
            margin-bottom: 12px;
            transition: all 0.3s ease;
        }
        .kids-book-card:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 22px rgba(99, 102, 241, 0.12);
        }
        </style>
    """, unsafe_allow_html=True)

    st.markdown("<div class='reading-room-wrapper'>", unsafe_allow_html=True)
    
    # সুন্দর ও কেচি হেডলাইন ও সাবলাইন ব্যানার
    st.markdown("""
    <div class="hero-banner">
        <h1 style="margin: 0; font-size: 28px; font-weight: 800; letter-spacing: 0.5px;">✨ Your Virtual Reading Room-Books & Reading Section</h1>
        <p style="margin: 8px 0 0 0; font-size: 15px; font-weight: 500; opacity: 0.95;">
            A Smart Hub for Your Books, Studies & Notes
        </p>
    </div>
    """, unsafe_allow_html=True)
    
    # মোটিভেশনাল বক্স
    st.markdown("""
    <div class="motivation-box">
        <div style="display: flex; align-items: center; gap: 12px;">
            <span style="font-size: 26px;">💡</span>
            <div>
                <h4 style="margin: 0; color: #047857; font-size: 14px; font-weight: 700;">Daily Motivation:</h4>
                <p style="margin: 3px 0 0 0; color: #334155; font-size: 13px; font-weight: 500;">
                    "Read regularly, keep notes of important lines, and build a brilliant future!"
                </p>
            </div>
        </div>
    </div>
    """, unsafe_allow_html=True)

    # সেশন স্টেট ইনিশিয়ালাইজেশন
    if "active_pdf_path" not in st.session_state:
        st.session_state.active_pdf_path = None
    if "book_chat_messages" not in st.session_state:
        st.session_state.book_chat_messages = [
            {"role": "assistant", "content": "Hello! Ask me if you need help understanding any line."}
        ]
    if "rename_file" not in st.session_state:
        st.session_state.rename_file = None
    if "editing_note_id" not in st.session_state:
        st.session_state.editing_note_id = None

    # ফাইল সেভ ফোল্ডার সেটআপ
    current_dir = dirname(abspath(__file__)) if "__file__" in locals() else os.getcwd()
    assets_dir = join(current_dir, "assets")
    if not exists(assets_dir):
        makedirs(assets_dir)

    # ক্যাটাগরি সিলেকশন
    col_cat, col_up = st.columns([2.5, 1.5], gap="large")
    with col_cat:
        book_category = st.radio(
            "Select Category",
            ["📖 NCTB Books", "📗 Guide Books", "📕 Other Books", "📝 My Saved Notes"],
            horizontal=True
        )
        
    if "📗 Guide Books" in book_category:
        with col_up:
            uploaded_guide = st.file_uploader("Upload Guide PDF", type=["pdf"], key="upload_guide_file")
            if uploaded_guide is not None:
                save_name = f"guide_{uploaded_guide.name}"
                save_path = join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_guide.getbuffer())
                st.success(f"Guide added: {uploaded_guide.name}")
                st.rerun()
    elif "📕 Other Books" in book_category:
        with col_up:
            uploaded_other = st.file_uploader("Upload Book PDF", type=["pdf"], key="upload_other_file")
            if uploaded_other is not None:
                save_name = f"other_{uploaded_other.name}"
                save_path = join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_other.getbuffer())
                st.success(f"Book added: {uploaded_other.name}")
                st.rerun()

    st.markdown("<hr style='margin: 20px 0; border: 0; border-top: 1.5px dashed #cbd5e1;'>", unsafe_allow_html=True)

    # রিনেম হ্যান্ডলার
    if st.session_state.rename_file:
        old_file = st.session_state.rename_file
        st.warning(f"Renaming file: {old_file}")
        new_name = st.text_input("Enter new name (with .pdf): ", value=old_file, key="global_rename_input")
        c_save, c_cancel = st.columns([1, 1], gap="small")
        with c_save:
            if st.button("Save", key="global_save_rename"):
                if new_name and new_name != old_file:
                    rename(join(assets_dir, old_file), join(assets_dir, new_name))
                    st.session_state.rename_file = None
                    st.success("Renamed successfully!")
                    st.rerun()
        with c_cancel:
            if st.button("X Cancel", key="global_cancel_rename"):
                st.session_state.rename_file = None
                st.rerun()

    # ক্যাটাগরি লজিক ও ডিসপ্লে
    if "📖 NCTB Books" in book_category:
        col_class_select, col_nctb_opts = st.columns([2, 1.5], gap="medium")
        with col_class_select:
            st.markdown("<h3 style='color: #4f46e5; font-size: 16px; font-weight: 700;'>📚 Select Class (NCTB Curriculum)</h3>", unsafe_allow_html=True)
            selected_class = st.selectbox(
                "Choose Class",
                ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"],
                index=2
            )
        cls_prefix = selected_class.lower().replace(" ", "")

        with col_nctb_opts:
            st.markdown("<h4 style='color: #4f46e5; font-size: 14px; margin-bottom: 6px;'>📌 Add NCTB Books</h4>", unsafe_allow_html=True)
            st.link_button(
                "🌐 Download from NCTB Website",
                "http://nctb.gov.bd/",
                use_container_width=True
            )
            uploaded_nctb = st.file_uploader(f"Upload {selected_class} Book PDF", type=["pdf"], key=f"upload_nctb_{cls_prefix}")
            if uploaded_nctb is not None:
                save_name = f"nctb_{cls_prefix}_{uploaded_nctb.name}"
                save_path = join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_nctb.getbuffer())
                st.success(f"📚 Book uploaded successfully for {selected_class}!")
                st.rerun()

        try:
            all_files = listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('nctb_' + cls_prefix)]
        except:
            pdf_files = []

        st.markdown(f"<h4 style='color: #1e293b; font-size: 15px; margin-top: 15px; font-weight: 700;'>📖 {selected_class} Books:</h4>", unsafe_allow_html=True)
        
        if pdf_files:
            grid_cols = st.columns(3, gap="medium")
            for idx, file_name in enumerate(pdf_files):
                file_path = join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace(f'nctb_{cls_prefix}_', '').replace('.pdf', '').replace('_', ' ').strip()
                
                with col:
                    st.markdown(f"""
                    <div class="kids-book-card">
                        <div style="font-size: 24px; margin-bottom: 6px;">📘</div>
                        <h4 style="color: #0f172a; margin: 0 0 8px 0; font-size: 13px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{display_name}</h4>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    b1, b2, b3 = st.columns([1, 1, 1], gap="small")
                    with b1:
                        if st.button("Read", key=f"read_nctb_{idx}", use_container_width=True, type="primary"):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("Rename", key=f"ren_nctb_{idx}", use_container_width=True):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("Delete", key=f"del_nctb_{idx}", use_container_width=True):
                            remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("Deleted!")
                            st.rerun()
        else:
            st.info(f"ℹ️ No books found for '{selected_class}'. Please upload your book PDF above or download directly from the official NCTB link.")

    elif "📗 Guide Books" in book_category:
        try:
            all_files = listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('guide_')]
        except:
            pdf_files = []

        st.markdown("<h4 style='color: #1e293b; font-size: 15px; font-weight: 700;'>📗 Uploaded Guide Books</h4>", unsafe_allow_html=True)
        if pdf_files:
            grid_cols = st.columns(3, gap="medium")
            for idx, file_name in enumerate(pdf_files):
                file_path = join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace('guide_', '').replace('.pdf', '').replace('_', ' ').strip()
                with col:
                    st.markdown(f"""
                    <div class="kids-book-card">
                        <div style="font-size: 24px; margin-bottom: 6px;">📗</div>
                        <h4 style="color: #0f172a; margin: 0 0 8px 0; font-size: 13px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{display_name}</h4>
                    </div>
                    """, unsafe_allow_html=True)
                    b1, b2, b3 = st.columns([1, 1, 1], gap="small")
                    with b1:
                        if st.button("Read", key=f"read_guide_{idx}", use_container_width=True, type="primary"):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("Rename", key=f"ren_guide_{idx}", use_container_width=True):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("Delete", key=f"del_guide_{idx}", use_container_width=True):
                            remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("Deleted!")
                            st.rerun()
        else:
            st.info("No guide books uploaded yet.")

    elif "📕 Other Books" in book_category:
        try:
            all_files = listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('other_')]
        except:
            pdf_files = []

        st.markdown("<h4 style='color: #1e293b; font-size: 15px; font-weight: 700;'>📕 Uploaded Other Books</h4>", unsafe_allow_html=True)
        if pdf_files:
            grid_cols = st.columns(3, gap="medium")
            for idx, file_name in enumerate(pdf_files):
                file_path = join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace('other_', '').replace('.pdf', '').replace('_', ' ').strip()
                with col:
                    st.markdown(f"""
                    <div class="kids-book-card">
                        <div style="font-size: 24px; margin-bottom: 6px;">📕</div>
                        <h4 style="color: #0f172a; margin: 0 0 8px 0; font-size: 13px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{display_name}</h4>
                    </div>
                    """, unsafe_allow_html=True)
                    b1, b2, b3 = st.columns([1, 1, 1], gap="small")
                    with b1:
                        if st.button("Read", key=f"read_other_{idx}", use_container_width=True, type="primary"):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("Rename", key=f"ren_other_{idx}", use_container_width=True):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("Delete", key=f"del_other_{idx}", use_container_width=True):
                            remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("Deleted!")
                            st.rerun()
        else:
            st.info("No other books uploaded yet.")

    elif "📝 My Saved Notes" in book_category:
        st.markdown("<h3 style='color: #4f46e5; font-size: 18px; font-weight: 700;'>📝 My Permanent Personal Notebook</h3>", unsafe_allow_html=True)
        st.markdown("<p style='color: #64748b; font-size: 12px;'>All your saved notes with exact timestamp are listed below.</p>", unsafe_allow_html=True)
        
        import sqlite3
        conn = sqlite3.connect("subject_notes.db")
        cursor = conn.cursor()
        cursor.execute("CREATE TABLE IF NOT EXISTS notes (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT, content TEXT, subject TEXT, note_type TEXT, created_at TEXT)")
        cursor.execute("SELECT id, title, content, subject, created_at FROM notes ORDER BY id DESC")
        db_notes = cursor.fetchall()
        conn.close()

        if db_notes:
            for note_id, note_title, note_content, note_subject, note_time in db_notes:
                display_time = note_time if note_time else "No timestamp"
                if st.session_state.editing_note_id == note_id:
                    st.markdown(f"""
                    <div style="background: #f0fdf4; padding: 12px 16px; border-radius: 12px; border: 2px solid #10b981; margin-bottom: 10px;">
                        <div style="font-size: 12px; font-weight: 700; color: #047857; margin-bottom: 6px;">✏️ Editing Note</div>
                    </div>
                    """, unsafe_allow_html=True)
                    with st.form(key=f"edit_tab_note_form_{note_id}"):
                        edited_topic = st.text_input("Topic / Title:", value=note_title)
                        edited_text = st.text_area("Note Details:", value=note_content, height=90)
                        col_update, col_cancel_edit = st.columns([1, 1])
                        with col_update:
                            update_btn = st.form_submit_button("💾 Update", use_container_width=True)
                        with col_cancel_edit:
                            cancel_btn = st.form_submit_button("X Cancel", use_container_width=True)
                        if update_btn:
                            current_time_str = datetime.now().strftime("%d %B, %Y, %I:%M %p")
                            conn_up = sqlite3.connect("subject_notes.db")
                            cur_up = conn_up.cursor()
                            cur_up.execute("UPDATE notes SET title = ?, content = ?, created_at = ? WHERE id = ?", (edited_topic, edited_text, current_time_str, note_id))
                            conn_up.commit()
                            conn_up.close()
                            st.session_state.editing_note_id = None
                            st.success("Note updated successfully!")
                            st.rerun()
                        if cancel_btn:
                            st.session_state.editing_note_id = None
                            st.rerun()
                else:
                    st.markdown(f"""
                    <div style="background: #ffffff; padding: 14px 18px; border-radius: 12px; border: 1px solid #e0e7ff; border-left: 4px solid #4f46e5; margin-bottom: 10px; box-shadow: 0 3px 8px rgba(79,70,229,0.04);">
                        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px;">
                            <span style="font-size: 11px; font-weight: 700; color: #4f46e5;">Subject: {note_subject}</span>
                            <span style="font-size: 10px; font-weight: 600; color: #64748b; background: #e0e7ff; padding: 2px 6px; border-radius: 4px;">🕒 {display_time}</span>
                        </div>
                        <div style="font-size: 13px; font-weight: 600; color: #0f172a; margin-bottom: 4px;">Topic: {note_title}</div>
                        <div style="font-size: 12px; color: #334155; background: #f8fafc; padding: 8px; border-radius: 6px;">{note_content}</div>
                    </div>
                    """, unsafe_allow_html=True)
                    c_edit, c_del = st.columns([1, 1], gap="small")
                    with c_edit:
                        if st.button("Edit", key=f"btn_edit_tab_{note_id}", use_container_width=True):
                            st.session_state.editing_note_id = note_id
                            st.rerun()
                    with c_del:
                        if st.button("🗑️ Delete", key=f"btn_del_tab_{note_id}", use_container_width=True):
                            conn_del = sqlite3.connect("subject_notes.db")
                            cur_del = conn_del.cursor()
                            cur_del.execute("DELETE FROM notes WHERE id = ?", (note_id,))
                            conn_del.commit()
                            conn_del.close()
                            if st.session_state.editing_note_id == note_id:
                                st.session_state.editing_note_id = None
                            st.success("Note deleted!")
                            st.rerun()
        else:
            st.info("No saved notes found. Save important notes while reading!")

    st.markdown("</div>", unsafe_allow_html=True)

    # পিডিএফ রিডার এবং এআই চ্যাট হাব
    if st.session_state.active_pdf_path and exists(st.session_state.active_pdf_path):
        st.markdown("<hr style='margin: 25px 0; border: 0; border-top: 2px solid #4f46e5;'>", unsafe_allow_html=True)
        c_title, c_close = st.columns([4, 1], gap="small")
        with c_title:
            active_file_name = basename(st.session_state.active_pdf_path)
            st.markdown(f"<h3 style='color: #4f46e5; margin: 0; font-family: Outfit; font-size: 18px;'>📖 Reading Room: {active_file_name}</h3>", unsafe_allow_html=True)
        with c_close:
            if st.button("X Close Reader", use_container_width=True):
                st.session_state.active_pdf_path = None
                st.rerun()

        pdf_col, ai_col = st.columns([2.3, 1.1], gap="large")
        with pdf_col:
            try:
                import base64
                with open(st.session_state.active_pdf_path, "rb") as pdf_file:
                    base64_pdf = base64.b64encode(pdf_file.read()).decode('utf-8')
                pdf_embed_html = f'''
                <iframe
                    src="data:application/pdf;base64,{base64_pdf}#toolbar=1&navpanes=1"
                    width="100%"
                    height="750px"
                    type="application/pdf"
                    style="border: 2px solid #e0e7ff; border-radius: 16px; background-color: #f8fafc; box-shadow: 0 10px 25px rgba(0,0,0,0.08);"
                >
                <p>Your browser does not support PDF rendering.</p>
                </iframe>
                '''
                st.markdown(pdf_embed_html, unsafe_allow_html=True)
            except Exception as e:
                st.error(f"Error loading PDF: {e}")

        with ai_col:
            st.markdown("""
            <div style="background: #ffffff; padding: 15px; border-radius: 16px; border: 2px solid #e0e7ff; margin-bottom: 12px; box-shadow: 0 6px 15px rgba(79, 70, 229, 0.06);">
                <h4 style="color: #4f46e5; margin-top: 0; margin-bottom: 4px; font-family: Outfit; font-size: 14px;">🤖 AI Assistant & Note Hub</h4>
                <p style="color: #64748b; font-size: 11px; margin-bottom: 0px;">Paste difficult lines for AI help or save notes permanently:</p>
            </div>
            """, unsafe_allow_html=True)
            
            st.link_button(
                "Open Gemini Chatbot",
                "https://gemini.google.com",
                use_container_width=True,
                type="primary"
            )
            st.markdown("<p style='text-align: center; color: #64748b; font-size: 10px; margin-top: 4px;'>Copy text and paste it into the chatbot</p>", unsafe_allow_html=True)
            st.markdown("<hr style='margin: 12px 0; border-color: #e2e8f0;'>", unsafe_allow_html=True)
            
            with st.form(key="save_note_form_from_reader", clear_on_submit=True):
                st.markdown("<h5 style='color: #047857; margin:0 0 4px 0; font-size: 13px;'>💾 Save Permanent Note</h5>", unsafe_allow_html=True)
                note_title_input = st.text_input("Topic / Title:", placeholder="e.g. Model Question 01")
                note_subject_input = st.selectbox("Select Subject:", ["Bangla", "English", "Math", "Science", "BGS", "Religion", "ICT", "Other"])
                note_content_input = st.text_area("Note / Problem Details:", placeholder="Write your note or problem here...", height=70)
                note_submit_btn = st.form_submit_button("💾 Save Note", use_container_width=True)
                
                if note_submit_btn and note_content_input.strip():
                    current_time_str = datetime.now().strftime("%d %B, %Y, %I:%M %p")
                    conn_s = sqlite3.connect("subject_notes.db")
                    cur_s = conn_s.cursor()
                    cur_s.execute("""
                        INSERT INTO notes (title, content, subject, note_type, created_at)
                        VALUES (?, ?, ?, ?, ?)
                    """, (note_title_input if note_title_input else f"Book: {active_file_name}", note_content_input, note_subject_input, "Final", current_time_str))
                    conn_s.commit()
                    conn_s.close()
                    st.success("Saved successfully with timestamp!")

            st.markdown("<hr style='margin: 12px 0; border-color: #e2e8f0;'>", unsafe_allow_html=True)
            chat_container = st.container(height=240)
            with chat_container:
                for msg in st.session_state.book_chat_messages:
                    if msg["role"] == "user":
                        st.chat_message("user").write(msg["content"])
                    else:
                        st.chat_message("assistant").write(msg["content"])
            
            with st.form(key="ai_chat_form", clear_on_submit=True):
                user_text = st.text_area("Ask or paste text:", placeholder="Type or paste your question here...", height=55)
                submit_button = st.form_submit_button("✨ Ask AI", use_container_width=True)
                if submit_button and user_text.strip():
                    st.session_state.book_chat_messages.append({"role": "user", "content": user_text})
                    with st.spinner("Generating answer..."):
                        try:
                            ai_reply = ask_gemini_rest_api(user_text)
                        except:
                            ai_reply = "দুঃখিত, এ মুহূর্তে এআই সংযোগে সমস্যা হচ্ছে।"
                        st.session_state.book_chat_messages.append({"role": "assistant", "content": ai_reply})
                    st.rerun()


import streamlit as st
import random
import time
import json
import sqlite3
from datetime import datetime

def render_quiz_practice():
    # আধুনিক, আকর্ষণীয় ও প্রিমিয়াম কাস্টম স্টাইল, কালার ও ডিজাইন[cite: 143, 144, 145, 146]
    st.markdown(
        """
        <style>
        @import url('https://fonts.googleapis.com/css?family=Hind+Siliguri:wght@400;500;600;700&family=Outfit:wght@400;600;800&display=swap');
        .practice-magic-wrapper {
            font-family: 'Outfit', 'Hind Siliguri', sans-serif;
            background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 50%, #e2e8f0 100%);
            padding: 35px;
            border-radius: 28px;
            border: 1px solid #cbd5e1;
            box-shadow: 0 20px 40px rgba(15, 23, 42, 0.06);
            color: #0f172a;
            margin-bottom: 30px;
        }
        .practice-banner {
            background: linear-gradient(135deg, #4f46e5 0%, #6366f1 50%, #8b5cf6 100%);
            padding: 28px;
            border-radius: 20px;
            color: white;
            text-align: center;
            margin-bottom: 25px;
            box-shadow: 0 10px 25px rgba(99, 102, 241, 0.3);
        }
        .cq-card {
            background: #ffffff;
            border-radius: 16px;
            border-left: 6px solid #6366f1;
            padding: 20px;
            margin-bottom: 18px;
            box-shadow: 0 6px 15px rgba(0,0,0,0.04);
            border: 1px solid #f1f5f9;
        }
        .cq-stem {
            background-color: #eef2ff;
            padding: 14px;
            border-radius: 10px;
            font-size: 15px;
            font-weight: 500;
            color: #3730a3;
            margin-bottom: 12px;
            border: 1px solid #e0e7ff;
        }
        .written-card {
            background: #ffffff;
            border-radius: 16px;
            border-left: 6px solid #0ea5e9;
            padding: 20px;
            margin-bottom: 18px;
            box-shadow: 0 6px 15px rgba(0,0,0,0.04);
            border: 1px solid #f1f5f9;
        }
        .stTabs [data-baseweb="tab-list"] {
            gap: 12px;
        }
        .stTabs [data-baseweb="tab"] {
            height: 52px;
            background-color: #ffffff;
            border-radius: 14px;
            color: #4f46e5;
            font-weight: 700;
            border: 1px solid #e0e7ff;
            padding: 8px 22px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.02);
        }
        .stTabs [aria-selected="true"] {
            background: linear-gradient(135deg, #4f46e5 0%, #6366f1 100%) !important;
            color: white !important;
            box-shadow: 0 4px 12px rgba(99, 102, 241, 0.35) !important;
        }
        .stButton button {
            font-family: 'Outfit', 'Hind Siliguri', sans-serif !important;
            border-radius: 12px !important;
            font-weight: 700 !important;
            background: linear-gradient(135deg, #4f46e5 0%, #6366f1 100%) !important;
            color: white !important;
            border: none !important;
            padding: 0.6rem 1.2rem !important;
            box-shadow: 0 6px 16px rgba(99, 102, 241, 0.3) !important;
            transition: all 0.3s ease !important;
        }
        .stButton button:hover {
            background: linear-gradient(135deg, #4338ca 0%, #4f46e5 100%) !important;
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(99, 102, 241, 0.4) !important;
        }
        </style>
        """,
        unsafe_allow_html=True
    )

    st.markdown("<div class='practice-magic-wrapper'>", unsafe_allow_html=True)
    st.markdown(
        """
        <div class="practice-banner">
            <h1 style="margin: 0; font-size: 28px; font-weight: 800; letter-spacing: 0.5px;">Your Virtual Reading Room - Practice & Preparation Section</h1>
            <p style="margin: 8px 0 0 0; font-size: 15px; font-weight: 500; opacity: 0.95;">
                A Smart Hub for Practice, Preparation, Model Test & Archives!
            </p>
        </div>
        """,
        unsafe_allow_html=True
    )

    # সেশন স্টেট ইনিশিয়ালাইজেশন[cite: 147]
    if "exam_started" not in st.session_state:
        st.session_state.exam_started = False
    if "exam_questions" not in st.session_state:
        st.session_state.exam_questions = []
    if "user_answers" not in st.session_state:
        st.session_state.user_answers = {}
    if "exam_title_str" not in st.session_state:
        st.session_state.exam_title_str = "মডেল টেস্ট"
    if "exam_submitted" not in st.session_state:
        st.session_state.exam_submitted = False
    if "is_saved_to_db" not in st.session_state:
        st.session_state.is_saved_to_db = False
    if "exam_deadline" not in st.session_state:
        st.session_state.exam_deadline = 0.0

    # --- শ্রেণিভিত্তিক প্রশ্ন ব্যাংক ও রিটেন পার্ট ডেটা (ক্লাস ৬ থেকে ১০) ---
    library_question_bank = {
        "ষষ্ঠ শ্রেণি": {
            "বাংলা প্রথম পত্র": {
                "গদ্যাংশ": {
                    "মিনু": {
                        "MCQ": [
                            {"q": "'মিনু' গল্পের মূল চরিত্র মিনু সম্পর্কে কোন তথ্যটি সঠিক?", "options": ["ক) সে কথা বলতে পারত", "খ) সে এতিম ও বাকপ্রতিবন্ধী", "গ) সে চোখে দেখত না", "ঘ) সে খুব চঞ্চল প্রকৃতির ছিল"], "answer": "খ) সে এতিম ও বাকপ্রতিবন্ধী"},
                            {"q": "মিনু তার পেিষের দায়িত্ব কার ওপর অর্পণ করেছিল?", "options": ["ক) হলুদ পাখি", "খ) জোছনা", "গ) সূর্য", "ঘ) মেঘ"], "answer": "ক) হলুদ পাখি"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: দশ বছরের প্রতিবন্ধী শিশু রতন সমাজের আর দশটি শিশুর মতো কথা বলতে পারে না। তবুও সে মনের আনন্দে চারপাশের প্রকৃতি ও ছোট ছোট পাখির সাথে ভাব জমায়, তাদের আচরণ দেখে নিজের দুঃখ ভুলে থাকে।",
                                "ka": "মিনুর অভিভাবক কে ছিলেন?",
                                "kha": "মিনু বধির হলেও সে কীভাবে সবকিছু বুঝে নিত—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের রতনের মানসিকতার সাথে 'মিনু' গল্পের মিনুর চরিত্রের সাদৃশ্য কোথায়—আলোচনা কর।",
                                "gha": "\"প্রকৃতির বৈচিত্র্য ও কল্পনার জগৎ বিশেষ চাহিদাসম্পন্ন শিশুর মনে আনন্দ জোগায়।\"—'মিনু' গল্পের আলোকে মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "বাংলা দ্বিতীয় পত্র": {
                "ব্যাকরণ ও নির্মিতি": {
                    "ভাষা ও বর্ণ": {
                        "MCQ": [
                            {"q": "বাংলা বর্ণমালায় মোট বর্ণের সংখ্যা কত?", "options": ["ক) ৫০টি", "খ) ৩৯টি", "গ) ১১টি", "ঘ) ৫২টি"], "answer": "ঘ) ৫২টি"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "অনুচ্ছেদ লিখন", "desc": "অনুচ্ছেদ লেখো: 'ষঋতু' বা 'আমাদের গ্রাম'।"},
                            {"title": "আবেদনপত্র", "desc": "স্কুলের দরিদ্র তহবিল থেকে সাহায্যের জন্য প্রধান শিক্ষকের নিকট আবেদনপত্র লেখো।"}
                        ]
                    }
                }
            },
            "ইংরেজি প্রথম পত্র": {
                "Reading & Composition": {
                    "Unit 2: Exploring People and Places": {
                        "MCQ": [
                            {"q": "What is the main objective of exploring new places?", "options": ["a) To waste time", "b) To gain knowledge and experience", "c) To spend money", "d) To sleep"], "answer": "b) To gain knowledge and experience"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "Paragraph Writing", "desc": "Write a paragraph on 'A School Library'."},
                            {"title": "Letter Writing", "desc": "Write a letter to your friend describing your village home."}
                        ]
                    }
                }
            },
            "ইংরেজি দ্বিতীয় পত্র": {
                "Grammar & Composition": {
                    "Parts of Speech & Tense": {
                        "MCQ": [
                            {"q": "Identify the adjective in: 'The clever fox ran away.'", "options": ["a) fox", "b) ran", "c) clever", "d) away"], "answer": "c) clever"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "Application", "desc": "Write an application for setting up a multimedia classroom."},
                            {"title": "Dialogue", "desc": "Write a dialogue between two friends about learning English."}
                        ]
                    }
                }
            },
            "গণিত": {
                "পাটিগণিত ও জ্যামিতি": {
                    "অনুপাত ও শতকরা": {
                        "MCQ": [
                            {"q": "শতকরা ৫% মানে কত?", "options": ["ক) ৫/১০", "খ) ৫/১০০", "গ) ১০০/৫", "ঘ) ১/৫"], "answer": "খ) ৫/১০০"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: একটি বিদ্যালয়ের ষষ্ঠ শ্রেণীতে মোট শিক্ষার্থীর ৬০ জন ছাত্র এবং ৪০ জন ছাত্রী।",
                                "ka": "মোট শিক্ষার্থীর মধ্যে ছাত্র ও ছাত্রীর অনুপাত কত?",
                                "kha": "ছাত্রসংখ্যা ও মোট শিক্ষার্থীর শতকরা হার নির্ণয় কর।",
                                "ga": "যদি আরও ২০ জন ছাত্রী নতুন ভর্তি হয়, তবে ছাত্র ও ছাত্রীর নতুন অনুপাত কত হবে?",
                                "gha": "\"অনুপাত ও শতকরা হিসাব দৈনন্দিন জীবনে তুলনামূলক বিশ্লেষণ সহজ করে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "বিজ্ঞান": {
                "জীববিজ্ঞান ও পদার্থ": {
                    "উদ্ভিদের জীবন": {
                        "MCQ": [
                            {"q": "উদ্ভিদের মূলের প্রধান কাজ কোনটি?", "options": ["ক) খাদ্য তৈরি", "খ) পানি ও খনিজ লবণ শোষণ", "গ) ফুল ফোটানো", "ঘ) বীজ ছড়ানো"], "answer": "খ) পানি ও খনিজ লবণ শোষণ"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: টবে লাগানো একটি সুস্থ গাছের পাতায় সূর্যালোক পড়ছে এবং শিকড় মাটি থেকে তরল উপাদান শোষণ করে উপরের দিকে পৌঁছে দিচ্ছে।",
                                "ka": "উদ্ভিদের পাতার সবুজ কণিকার নাম কী?",
                                "kha": "প্রস্বেদন প্রক্রিয়া উদ্ভিদের জন্য কেন গুরুত্বপূর্ণ—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে বর্ণিত শিকড় কর্তৃক উপাদান শোষণ প্রক্রিয়াটি ব্যাখ্যা কর।",
                                "gha": "\"উদ্ভিদের খাদ্য প্রস্তুত ও গ্যাসীয় বিনিময়ে পাতার ভূমিকা অনন্য।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "বাংলাদেশ ও বিশ্বপরিচয়": {
                "বাংলাদেশ পরিচিতি": {
                    "বাংলাদেশের সমাজ ও সংস্কৃতি": {
                        "MCQ": [
                            {"q": "বাংলাদেশের ভৌগোলিক অবস্থান কোন অঞ্চলে?", "options": ["ক) দক্ষিণ এশিয়া", "খ) মধ্য এশিয়া", "গ) ইউরোপ", "ঘ) আফ্রিকা"], "answer": "ক) দক্ষিণ এশিয়া"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: বাংলাদেশের গ্রামীণ সমাজে মানুষ একে অপরের বিপদে এগিয়ে আসে, উৎসবে সকলে মিলে আনন্দ ভাগাভাগি করে এবং যৌথ পরিবার প্রথার মাধ্যমে পারস্পরিক বন্ধন দৃঢ় রাখে।",
                                "ka": "বাংলাদেশের সংস্কৃতির মূল ভিত্তি কী?",
                                "kha": "সমাজ গঠনে পরিবারের ভূমিকা ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে বাংলাদেশের গ্রামীণ সমাজের কোন রূপটি ফুটে উঠেছে—আলোচনা কর।",
                                "gha": "\"পারস্পরিক সহযোগিতা ও গ্রামীণ ঐতিহ্যই আমাদের সংস্কৃতির প্রাণ।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "ইসলাম ও নৈতিক শিক্ষা": {
                "ইবাদত": {
                    "সালাত ও রোজা": {
                        "MCQ": [
                            {"q": "ইসলামে দৈনিক কয় ওয়াক্ত সালাত ফরজ?", "options": ["ক) ৩ ওয়াক্ত", "খ) ৪ ওয়াক্ত", "গ) ৫ ওয়াক্ত", "ঘ) ৬ ওয়াক্ত"], "answer": "গ) ৫ ওয়াক্ত"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: জামিল প্রতিদিন নিয়মিত জামায়াতের সাথে ওয়াক্তমতো সালাত আদায় করে। এর ফলে তার মন শান্ত থাকে এবং সে সকল প্রকার অশ্লীল ও খারাপ কাজ থেকে দূরে থাকে।",
                                "ka": "সালাত শব্দের শাব্দিক অর্থ কী?",
                                "kha": "সালাত কীভাবে মানুষকে অন্যায় থেকে দূরে রাখে—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে জামিলের অভ্যাসের মাধ্যমে সালাতের কোন শিক্ষাটি প্রতিফলিত হয়েছে—ব্যাখ্যা কর।",
                                "gha": "\"নিয়মিত সালাত আদায় মানুষের চরিত্র ও নৈতিকতা গঠনে সবচেয়ে বড় ভূমিকা রাখে।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "তথ্য ও যোগাযোগ প্রযুক্তি": {
                "কম্পিউটার ও প্রযুক্তি": {
                    "ইন্টারনেট পরিচিতি": {
                        "MCQ": [
                            {"q": "ওয়ার্ল্ড ওয়াইড ওয়েব (www) এর প্রবক্তা কে?", "options": ["ক) টিম বার্নার্স লি", "খ) বিল গেটস", "গ) স্টিভ জবস", "ঘ) মার্ক জুকারবার্গ"], "answer": "ক) টিম বার্নার্স লি"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: শিহাব ইন্টারনেট ব্যবহার করে ঘরে বসে পৃথিবীর যেকোনো প্রান্তের লাইব্রেরি থেকে বই পড়তে পারে এবং বিভিন্ন দেশের বন্ধুদের সাথে চ্যাট করতে পারে।",
                                "ka": "www এর পূর্ণরূপ কী?",
                                "kha": "ইন্টারনেটের দুটি প্রধান ব্যবহার লেখো।",
                                "ga": "উদ্দীপকে শিহাবের তথ্য আদান-প্রদানের মাধ্যমটির কার্যাবলি ব্যাখ্যা কর।",
                                "gha": "\"ইন্টারনেট বিশ্বকে আমাদের হাতের মুঠোয় এনে দিয়েছে।\"—উক্তিটি মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            }
        },
        "সপ্তম শ্রেণি": {
            "বাংলা প্রথম পত্র": {
                "গদ্যাংশ": {
                    "পিতৃপুরুষের গল্প": {
                        "MCQ": [
                            {"q": "'পিতৃপুরুষের গল্প' রচনায় কোন ঐতিহাসিক ঘটনার কথা প্রধানত এসেছে?", "options": ["ক) বায়ান্নর ভাষা আন্দোলন", "খ) ঊনসত্তরের গণঅভ্যুত্থান", "গ) ১৯৭১ সালের মুক্তিযুদ্ধ", "ঘ) নব্বইয়ের গণঅভ্যুত্থান"], "answer": "গ) ১৯৭১ সালের মুক্তিযুদ্ধ"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: গ্রামের স্কুল মাঠে শহীদ মিনারে ফুল দিতে গিয়ে দশ বছরের রনি তার দাদীর কাছে জানতে চায় ১৯৭১ সালের মুক্তিযুদ্ধের কথা। দাদী স্নেহের সাথে তার মাথায় হাত বুলিয়ে বলেন, 'তোর চাচা শহীদ মন্টু তোদের মতো বয়সে দেশের টানে জীবন বাজি রেখেছিল।'",
                                "ka": "হারুন হাবিব কত সালে জন্মগ্রহণ করেন?",
                                "kha": "আমাদের ইতিহাস মূলত লড়াইয়ের ইতিহাস—কথাটি দ্বারা কী বোঝানো হয়েছে?",
                                "ga": "উদ্দীপকের দাদীর বক্তব্যে 'পিতৃপুরুষের গল্প' গল্পের কোন দিকটি প্রতিফলিত হয়েছে? ব্যাখ্যা করো।",
                                "gha": "\"উদ্দীপকটি 'পিতৃপুরুষের গল্প' গল্পের সামগ্রিক চেতনার একটি ক্ষুদ্র রূপ মাত্র\"—বিশ্লেষণ করো।"
                            }
                        ]
                    }
                }
            },
            "বাংলা দ্বিতীয় পত্র": {
                "ব্যাকরণ ও নির্মিতি": {
                    "শব্দ, পদ ও বাক্যপ্রকরণ": {
                        "MCQ": [
                            {"q": "বাংলা ভাষার মূল উৎস কোনটি?", "options": ["ক) বৈদিক ভাষা", "খ) ইন্দো-ইউরোপীয়", "গ) প্রাকৃত", "ঘ) অপভ্রংশ"], "answer": "খ) ইন্দো-ইউরোপীয়"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: সুমাইয়া প্রতিদিন সকালে উঠে ব্যাকরণের নিয়ম মেনে শুদ্ধভাবে কথা বলার চর্চা করে এবং লেখার সময় বানান ও বিভক্তির প্রতি বিশেষ নজর রাখে।",
                                "ka": "ভাষা কাকে বলে?",
                                "kha": "সাধু ও চলিত ভাষার দুটি মূল পার্থক্য ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের সুমাইয়ার অনুশীলনে বাংলা ব্যাকরণ চর্চার কোন দিকটি প্রকাশ পেয়েছে—ব্যাখ্যা কর।",
                                "gha": "\"ব্যাকরণ পাঠের মাধ্যমেই ভাষার শুদ্ধরূপ অর্জন সম্ভব।\"—পর্যালোচনা কর।"
                            }
                        ],
                        "Written": [
                            {"title": "ভাবসম্প্রসারণ", "desc": "দাঁড়াও তবে খাটো করে নয়... অথবা পরিশ্রম সৌভাগ্যের প্রসূতি।"},
                            {"title": "দরখাস্ত", "desc": "বইয়ের প্রয়োজনীয়তা জানিয়ে ছোট ভাইয়ের কাছে একটি পত্র লেখো।"}
                        ]
                    }
                }
            },
            "ইংরেজি প্রথম পত্র": {
                "Reading & Composition": {
                    "Unit 1: Environment & Stories": {
                        "MCQ": [
                            {"q": "What class does Rina study in?", "options": ["a) class six", "b) class seven", "c) class eight", "d) class nine"], "answer": "b) class seven"}
                        ],
                        "CQ": [
                            {
                                "stem": "Read the passage: Rina is a student of class seven. She gets up early in the morning and attends her school regularly. She dreams of becoming a doctor to serve poor villagers.",
                                "ka": "What class does Rina study in?",
                                "kha": "Why does Rina want to become a doctor?",
                                "ga": "Describe Rina's daily routine based on the stem using appropriate Present Simple tenses.",
                                "gha": "\"Determination and hard work lead to success.\" Evaluate Rina's mindset in light of the context."
                            }
                        ],
                        "Written": [
                            {"title": "Paragraph", "desc": "Write a paragraph on 'Early Rising'."}
                        ]
                    }
                }
            },
            "ইংরেজি দ্বিতীয় পত্র": {
                "Grammar & Written": {
                    "Right form of verbs": {
                        "MCQ": [
                            {"q": "Choose the correct form: 'He (go) to school everyday.'", "options": ["a) went", "b) goes", "c) going", "d) gone"], "answer": "b) goes"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "Composition / Essay", "desc": "Write a short composition on 'Your Hobby'."},
                            {"title": "Formal Letter", "desc": "Write a letter to the editor about frequent load-shedding in your area."}
                        ]
                    }
                }
            },
            "গণিত": {
                "পাটিগণিত ও বীজগণিত": {
                    "মূলদ সংখ্যা ও বীজগণিতীয় সূত্র": {
                        "MCQ": [
                            {"q": "সবচেয়ে ছোট জোড় মৌলিক সংখ্যা কোনটি?", "options": ["ক) ০", "খ) ১", "গ) ২", "ঘ) ৪"], "answer": "গ) ২"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: a + 1/a = 4 একটি বীজগণিতীয় সমীকরণ।",
                                "ka": "x - 1/x এর মান নির্ণয়ের সূত্রটি লেখো।",
                                "kha": "প্রমাণ করো যে, a^2 + 1/a^2 = 14",
                                "ga": "দেখাও যে, a^4 + 1/a^4 = 194",
                                "gha": "\"বীজগণিতীয় অনুসিদ্ধান্ত জটিল সমস্যার সমাধান সহজ করে।\"—যাচাই করো।"
                            }
                        ]
                    }
                }
            },
            "বিজ্ঞান": {
                "ভৌত ও জীববিজ্ঞান": {
                    "কোষ ও তাপ": {
                        "MCQ": [
                            {"q": "কোষের 'পাওয়ার হাউস' কাকে বলা হয়?", "options": ["ক) নিউক্লিয়াস", "খ) লাইসোসোম", "গ) মাইটোকন্ড্রিয়া", "ঘ) সেন্ট্রোসোম"], "answer": "গ) মাইটোকন্ড্রিয়া"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: অণুবীক্ষণ যন্ত্রের নিচে একটি উদ্ভিদ কোষ ও প্রাণীকোষ পর্যবেক্ষণ করা হলো। দেখা গেল, উদ্ভিদ কোষে একটি শক্ত প্রাচীর ও সবুজ অঙ্গাণু বিদ্যমান।",
                                "ka": "কোষ প্রাচীর কী দিয়ে গঠিত?",
                                "kha": "প্লাস্টিডকে উদ্ভিদের খাদ্য তৈরির কারখানা বলা হয় কেন?",
                                "ga": "উদ্দীপকের আলোকে উদ্ভিদকোষ ও প্রাণীকোষের প্রধান বৈসাদৃশ্যগুলো তুলনা করো।",
                                "gha": "\"সবুজ প্লাস্টিডের অনুপস্থিতিতে সমগ্র জীবজগৎ স্তব্ধ হয়ে পড়বে।\"—বিশ্লেষণ করো।"
                            }
                        ]
                    }
                }
            },
            "বাংলাদেশ ও বিশ্বপরিচয়": {
                "ইতিহাস ও সমাজ": {
                    "মুক্তিযুদ্ধ ও পরিবেশ": {
                        "MCQ": [
                            {"q": "মুজিবনগর সরকার কবে গঠিত হয়?", "options": ["ক) ৭ মার্চ ১৯৭১", "খ) ১০ এপ্রিল ১৯৭১", "গ) ১৭ এপ্রিল ১৯৭১", "ঘ) ১৬ ডিসেম্বর ১৯৭১"], "answer": "খ) ১০ এপ্রিল ১৯৭১"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: উপকূলীয় একটি গ্রামে অনিয়ন্ত্রিতভাবে বনাঞ্চল কেটে ফেলার ফলে সমুদ্রের জলোচ্ছ্বাস ও ঝড়-ঝঞ্ঝার তীব্রতা বৃদ্ধি পেয়ে ঘরবাড়ি ধ্বংস হয়ে যাচ্ছে।",
                                "ka": "গ্রিনহাউস গ্যাস কী?",
                                "kha": "জলবায়ু পরিবর্তনের ফলে বাংলাদেশের বৈশ্বিক ঝুঁকি ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের এলাকার প্রাকৃতিক বিপর্যয়ের কারণ পাঠ্যবইয়ের পরিবেশ অধ্যায়ের আলোকে বর্ণনা কর।",
                                "gha": "\"টেকসই উন্নয়ন ও বৃক্ষরোপণই কেবল উপকূলীয় জীবন রক্ষা করতে পারে।\"—মতামত দাও।"
                            }
                        ]
                    }
                }
            },
            "ইসলাম ও নৈতিক শিক্ষা": {
                "আকাঈদ ও আখলাক": {
                    "ঈমান ও চরিত্র": {
                        "MCQ": [
                            {"q": "ইসলামের পঞ্চস্তম্ভের দ্বিতীয় স্তম্ভ কোনটি?", "options": ["ক) সাওম", "খ) সালাত", "গ) যাকাত", "ঘ) হজ"], "answer": "খ) সালাত"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: সায়েম তার ব্যবসায় কখনো মাপে কম দেয় না এবং খদ্দেরের সাথে সততার পরিচয় দেয়। সে বিশ্বাস করে যে তার প্রতিটি কাজের হিসাব শেষ বিচারে আল্লাহর কাছে দিতে হবে।",
                                "ka": "আখিরাত শব্দের অর্থ কী?",
                                "kha": "তাওহীদে বিশ্বাস মানুষের মনে কীভাবে অন্যায়ের ভয় দূর করে—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের সায়েমের কর্মের মাধ্যমে ঈমানের কোন স্তম্ভটির প্রতিফলন ঘটেছে—ব্যাখ্যা কর।",
                                "gha": "\"পরকালে জবাবদিহিতার অনুভূতিই ব্যবসায়িক ও সামাজিক সততার মূল ভিত্তি।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "তথ্য ও যোগাযোগ প্রযুক্তি": {
                "কম্পিউটার ও ইন্টারনেট": {
                    "ইন্টারনেট ও নেটওয়ার্ক": {
                        "MCQ": [
                            {"q": "ই-মেইল (E-mail) এর পূর্ণরূপ কী?", "options": ["ক) Electrical Mail", "খ) Electronic Mail", "গ) Easy Mail", "ঘ) Engine Mail"], "answer": "খ) Electronic Mail"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: সিয়াম ইন্টারনেটে পরীক্ষার ফলাফল দেখার জন্য কম্পিউটারে সার্চ ব্রাউজার ব্যবহার করে রোল নম্বর প্রদান করল এবং কয়েক সেকেন্ডের মধ্যে মনিটরে রেজাল্ট দেখতে পেল।",
                                "ka": "ইন্টারনেট কী?",
                                "kha": "ইনপুট ও আউটপুট ডিভাইসের প্রধান দুটি পার্থক্য লেখো।",
                                "ga": "উদ্দীপকে সিয়ামের ব্যবহৃত ডিভাইসগুলোর মধ্যে কোনগুলো ইনপুট এবং কোনটি আউটপুট তা চিহ্নিত করে কাজ ব্যাখ্যা কর।",
                                "gha": "\"তথ্য প্রযুক্তির দ্রুত বিকাশ শিক্ষা ব্যবস্থাকে সহজ ও প্রবেশযোগ্য করেছে।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            }
        },
        "অষ্টম শ্রেণি": {
            "বাংলা প্রথম পত্র": {
                "গদ্যাংশ": {
                    "ষোলোই ডিসেম্বরের কথা": {
                        "MCQ": [
                            {"q": "ষোলোই ডিসেম্বর বাংলাদেশে কী হিসেবে পালিত হয়?", "options": ["ক) স্বাধীনতা দিবস", "খ) বিজয় দিবস", "গ) শহীদ দিবস", "ঘ) বাংলা নববর্ষ"], "answer": "খ) বিজয় দিবস"}
                        ],
                        "CQ": []
                    }
                }
            },
            "বাংলা দ্বিতীয় পত্র": {
                "ব্যাকরণ ও নির্মিতি": {
                    "ভাষা, ব্যাকরণ ও নির্মিতি": {
                        "MCQ": [
                            {"q": "সবচেয়ে বেশি ব্যবহৃত বাংলা উপসর্গ কয়টি?", "options": ["ক) ২১টি", "খ) ২০টি", "গ) ১৯টি", "ঘ) ২২টি"], "answer": "ক) ২১টি"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: রফিক সাহেব তার লেখায় সবসময় সহজ-সরল ভাষায় মনের ভাব প্রকাশ করেন, যা সব অঞ্চলের মানুষ সহজেই বুঝতে পারে। অন্যদিকে তার বন্ধু প্রথাগত জটিল ও প্রাচীন শব্দ ব্যবহারে স্বাচ্ছন্দ্য বোধ করেন।",
                                "ka": "চলিত ভাষার প্রবর্তক কে?",
                                "kha": "সাধু ও চলিত ভাষার দুটি মৌলিক পার্থক্য লেখ।",
                                "ga": "উদ্দীপকের রফিক সাহেবের ভাষারীতি পাঠ্যবইয়ের ব্যাকরণগত কোন রূপটিকে নির্দেশ করে—ব্যাখ্যা কর।",
                                "gha": "\"সর্বজনগ্রাহ্য ও সাবলীল যোগাযোগের জন্য চলিত/প্রমিত ভাষারীতিই সর্বোত্তম।\"—বিশ্লেষণ কর।"
                            }
                        ],
                        "Written": [
                            {"title": "ভাবসম্প্রসারণ", "desc": "দশে মিলে করি কাজ, হারি জিতি নাহি লাজ।"},
                            {"title": "দরখাস্ত", "desc": "বিনা বেতনে অধ্যয়নের জন্য প্রধান শিক্ষকের নিকট আবেদন।"}
                        ]
                    }
                }
            },
            "ইংরেজি প্রথম পত্র": {
                "Reading & Composition": {
                    "Unit 1: Craft & Habits": {
                        "MCQ": [
                            {"q": "What is Nakshi Kantha?", "options": ["a) A kind of embroidered quilt", "b) A type of folk song", "c) A traditional dance", "d) A local sweet"], "answer": "a) A kind of embroidered quilt"}
                        ],
                        "CQ": [
                            {
                                "stem": "Read the text: Nakshi Kantha is a kind of embroidered quilt. The name was taken from the Bengali word 'naksha' which means artistic pattern. It is a traditional craft of Bangladesh.",
                                "ka": "Where does the name 'Nakshi Kantha' come from?",
                                "kha": "Why is Nakshi Kantha called a traditional craft?",
                                "ga": "Rewrite the main ideas of the text using simple present passive structures.",
                                "gha": "\"Cultural crafts like Nakshi Kantha represent our national identity globally.\" Justify the statement."
                            }
                        ],
                        "Written": [
                            {"title": "Paragraph", "desc": "Write a paragraph on 'Tree Plantation'."}
                        ]
                    }
                }
            },
            "ইংরেজি দ্বিতীয় পত্র": {
                "Grammar & Composition": {
                    "Grammar Basics & Transformation": {
                        "MCQ": [
                            {"q": "Which one is an Assertive sentence?", "options": ["a) What a bird!", "b) Bangladesh is our motherland.", "c) Do you know him?", "d) May Allah bless you."], "answer": "b) Bangladesh is our motherland."}
                        ],
                        "CQ": [
                            {
                                "stem": "Transform the following sentences as directed: (a) Cricket is a very popular game. (Exclamatory) (b) Everybody knows this. (Negative) (c) Who does not love cricket? (Assertive) (d) It is an exciting game. (Interrogative)",
                                "ka": "What is an assertive sentence?",
                                "kha": "How do you change an affirmative sentence into a negative one without changing its meaning—explain with an example.",
                                "ga": "Transform the sentences (a), (b), (c) and (d) according to the directions given in the stem.",
                                "gha": "\"Sentence transformation tests a student's grammatical capability and flexibility in language use.\"—Evaluate the statement."
                            }
                        ],
                        "Written": [
                            {"title": "Formal Letter", "desc": "Write a letter to the editor about frequent load-shedding."}
                        ]
                    }
                }
            },
            "গণিত": {
                "পাটিগণিত ও বীজগণিত": {
                    "প্যাটার্ন ও মুনাফা": {
                        "MCQ": [
                            {"q": "সরল মুনাফার সূত্র কোনটি?", "options": ["ক) Pnr", "খ) P/nr", "গ) Pr/n", "ঘ) Pn/r"], "answer": "ক) Pnr"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: কোনো আসল ৩ বছরে মুনাফা-আসলে ১৫২০০ টাকা এবং ৫ বছরে মুনাফা-আসলে ১৭৬০০ টাকা হয়।",
                                "ka": "২ বছরের মুনাফা কত তা নির্ণয় কর।",
                                "kha": "আসল ও বার্ষিক মুনাফার হার নির্ণয় কর।",
                                "ga": "উক্ত আসলে একই হারে ৫ বছরের জন্য সঞ্চয় করলে চক্রবৃদ্ধি মুনাফা কত হবে?",
                                "gha": "\"নির্দিষ্ট হারে সরল মুনাফার চেয়ে চক্রবৃদ্ধি মুনাফায় আমানতকারীর লাভ বেশি।\"—গাণিতিকভাবে বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "বিজ্ঞান": {
                "পদার্থ ও জীব": {
                    "প্রাণীজগতের শ্রেণীবিন্যাস": {
                        "MCQ": [
                            {"q": "শ্রেণীবিন্যাসের জনক কাকে বলা হয়?", "options": ["ক) এরিস্টটল", "খ) ক্যারোলাস লিনিয়াস", "গ) থিওফ্রাস্টাস", "ঘ) জন রে"], "answer": "খ) ক্যারোলাস লিনিয়াস"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: সিয়াম পুকুর থেকে কেঁচো, চিংড়ি এবং একটি মাছ সংগ্রহ করল। শিক্ষক বললেন, প্রাণী তিনটির দৈহিক গঠন ও মেরুদণ্ডের উপস্থিতি ভিন্ন হওয়ার কারণে এরা আলাদা পর্বের অন্তর্ভুক্ত।",
                                "ka": "শ্রেণীবিন্যাসের সর্বনিম্ন ধাপ কোনটি?",
                                "kha": "অর্থোপোডা পর্বের প্রাণীদের দুটি প্রধান বৈশিষ্ট্য লেখো।",
                                "ga": "উদ্দীপকের সংগৃহীত কেঁচো ও চিংড়ির বৈশিষ্ট্যগত পার্থক্য ব্যাখ্যা কর।",
                                "gha": "\"প্রাণীজগতের বৈচিত্র্য অনুধাবনে শ্রেণীবিন্যাসের ভূমিকা অপরিসীম।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "বাংলাদেশ ও বিশ্বপরিচয়": {
                "ইতিহাস ও সমাজ": {
                    "মুক্তিযুদ্ধ ও পরিবেশ": {
                        "MCQ": [
                            {"q": "১৯৭১ সালের ৭ই মার্চ বঙ্গবন্ধু কোথায় ভাষণ দেন?", "options": ["ক) রেসকোর্স ময়দান", "খ) পল্টন ময়দান", "গ) শহীদ মিনার", "ঘ) বাহাদুর শাহ পার্ক"], "answer": "ক) রেসকোর্স ময়দান"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: ১৯৭১ সালে পাকিস্তানি বাহিনীর গণহত্যা ও নির্যাতনের বিরুদ্ধে বাঙালি জাতি দল-মত নির্বিশেষে রুখে দাঁড়ায়। সাধারণ মানুষ ও যৌথ বাহিনীর সম্মিলিত আক্রমণের ফলে অর্জিত হয় কাঙিক্ষত স্বাধীনতা।",
                                "ka": "মুজিবনগর সরকার কত তারিখে শপথ গ্রহণ করে?",
                                "kha": "অপারেশন সার্চলাইট বলতে কী বোঝায়—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে বর্ণিত মুক্তি সংগ্রামে সর্বস্তরের জনগণের ভূমিকার কারণ আলোচনা কর।",
                                "gha": "\"বাঙালির জাতীয় ঐক্য ও আত্মত্যাগই ছিল ১৯৭১ সালের বিজয়ের মূল চাবিকাঠি।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "ইসলাম ও নৈতিক শিক্ষা": {
                "আকাঈদ ও আখলাক": {
                    "ঈমান ও চরিত্র": {
                        "MCQ": [
                            {"q": "কুরআন মজিদ নাজিল হতে কত বছর সময় লেগেছিল?", "options": ["ক) ২০ বছর", "খ) ২২ বছর", "গ) ২৩ বছর", "ঘ) ২৫ বছর"], "answer": "গ) ২৩ বছর"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: জামিল সাহেব সবসময় মুখে মিষ্টি কথা বলেন, কিন্তু কাজের বেলায় ওয়াদা বরখেলাপ করেন এবং বন্ধুদের গোপন কথা অন্যের কাছে ফাঁস করে দেন।",
                                "ka": "মুনাফিকের নিদর্শন কয়টি?",
                                "kha": "নিফাক মানুষের আখিরাতের জীবনের জন্য চরম ক্ষতিকর কেন—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের জামিল সাহেবের আচরণের মধ্যে কোন সামাজিক অপরাধটি পরিলক্ষিত হয়—ব্যাখ্যা কর।",
                                "gha": "\"আমানতদারী ও ওয়াদা পালনের মনোভাবই সমাজে শান্তি প্রতিষ্ঠা করতে পারে।\"—পর্যালোচনা কর।"
                            }
                        ]
                    }
                }
            },
            "তথ্য ও যোগাযোগ প্রযুক্তি": {
                "ই-লার্নিং ও বিশ্বগ্রাম": {
                    "বিশ্বগ্রাম ও প্রযুক্তি": {
                        "MCQ": [
                            {"q": "বিশ্বগ্রাম (Global Village) ধারণার প্রবক্তা কে?", "options": ["ক) বিল গেটস", "খ) মার্শাল ম্যাকলুহান", "গ) স্টিভ জবস", "ঘ) মার্ক জুকারবার্গ"], "answer": "খ) মার্শাল ম্যাকলুহান"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: রনি ঘরে বসেই ল্যাপটপ ও ইন্টারনেটের সাহায্যে বিদেশি একটি বিশ্ববিদ্যালয়ের অনলাইন কোর্সে অংশ নিচ্ছে, পরীক্ষা দিচ্ছে এবং ডিজিটাল লাইব্রেরি থেকে বই পড়ছে।",
                                "ka": "ই-লার্নিং কী?",
                                "kha": "শিক্ষাক্ষেত্রে ইন্টারনেটের দুটি সুফল ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে রনির শিক্ষাগ্রহণের মাধ্যমটি পাঠ্যবইয়ের ই-লার্নিং ধারণার সাথে কীভাবে সম্পর্কিত—ব্যাখ্যা কর।",
                                "gha": "\"ই-লার্নিং ঐতিহ্যগত শিক্ষা ব্যবস্থাকে আরও সহজ ও গতিশীল করেছে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            }
        },
        "নবম শ্রেণি": {
            "বাংলা প্রথম পত্র": {
                "সাহিত্য": {
                    "গদ্য ও পদ্য": {
                        "MCQ": [
                            {"q": "'শুভা' গল্পে শুভার প্রকৃত নাম কী ছিল?", "options": ["ক) সুভাষিণী", "খ) সুকেশিনী", "গ) সুহাসিনী", "ঘ) সুচরিতা"], "answer": "ক) সুভাষিণী"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: সলিমুল্লাহ গ্রামে এক প্রতিবন্ধী শিশুকে গভীর মমতায় আগলে রাখেন। সমাজের মানুষ তাকে অবহেলা করলেও তিনি মনে করেন ভালোবাসাই সকল মানুষের অধিকার।",
                                "ka": "শুভার বাকপ্রতিবন্ধী দুই সখীর নাম কী?",
                                "kha": "শুভা প্রকৃতির মাঝে আশ্রয় খুঁজে পেয়েছিল কেন?",
                                "ga": "উদ্দীপকের সলিমুল্লাহর মনোভাব 'শুভা' গল্পের কোন দিকটি ফুটিয়ে তোলে—ব্যাখ্যা কর।",
                                "gha": "\"বিশেষ চাহিদাসম্পন্ন শিশুদের প্রতি সহানুভূতিশীল দৃষ্টিভঙ্গিই মানবিক সমাজ গঠনের পূর্বশর্ত।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "বাংলা দ্বিতীয় পত্র": {
                "ব্যাকরণ ও নির্মিতি": {
                    "ধ্বনিতত্ত্ব ও সমাস": {
                        "MCQ": [
                            {"q": "পরপদের অর্থ কোন সমাসে প্রধান হয়?", "options": ["ক) তৎপুরুষ সমাস", "খ) দ্বন্দ্ব সমাস", "গ) বহুব্রীহি সমাস", "ঘ) অব্যয়ীভাব সমাস"], "answer": "ক) তৎপুরুষ সমাস"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: ব্যাকরণগত সঠিক শব্দ প্রয়োগ ও সঠিক পদ সংযোগ ভাষার সৌন্দর্য ও ভাব প্রকাশের স্পষ্টতা বাড়ায়। সমাস ও প্রত্যয় সাধিত শব্দ ভাষাকে সংক্ষেপ করে।",
                                "ka": "সমাস শব্দের অর্থ কী?",
                                "kha": "দ্বন্দ্ব সমাস ও বহুব্রীহি সমাসের দুটি প্রধান পার্থক্য লেখ।",
                                "ga": "উদ্দীপকের আলোকে পদসংক্ষেপণে সমাসের ভূমিকা ব্যাখ্যা কর।",
                                "gha": "\"শব্দ গঠনের প্রক্রিয়া জানা ছাড়া প্রমিত ভাষারীতি অনুসরণ করা সম্ভব নয়।\"—বিশ্লেষণ কর।"
                            }
                        ],
                        "Written": [
                            {"title": "সারাংশ/সারমর্ম", "desc": "যৌবনের কাজ আত্মাকে উন্নত করা... (অনুশীলন করুন)।"},
                            {"title": "প্রতিবেদন লিখন", "desc": "বিদ্যালয়ে আন্তর্জাতিক মাতৃভাষা দিবস উদ্‌যাপন উপলক্ষে একটি প্রতিবেদন তৈরি করো।"}
                        ]
                    }
                }
            },
            "ইংরেজি প্রথম পত্র": {
                "Reading & Writing": {
                    "Unit 1: Good Citizens & Environment": {
                        "MCQ": [
                            {"q": "What is the primary responsibility of a good citizen?", "options": ["a) Obeying laws and helping others", "b) Accumulating wealth", "c) Living in isolation", "d) Ignoring society"], "answer": "a) Obeying laws and helping others"}
                        ],
                        "CQ": [
                            {
                                "stem": "Read the text: To be a good citizen, a person needs to acquire knowledge, skill, and a good attitude towards society. Good citizens respect laws and contribute to environmental conservation.",
                                "ka": "What qualities make a person a good citizen?",
                                "kha": "Why is obeying laws essential for social harmony?",
                                "ga": "Summarize the key duties of a good citizen based on the passage.",
                                "gha": "\"Individual responsibility leads to national growth.\" Evaluate the statement in light of the text."
                            }
                        ],
                        "Written": [
                            {"title": "Graph/Chart", "desc": "Describe a graph showing the literacy rate in Bangladesh."}
                        ]
                    }
                }
            },
            "ইংরেজি দ্বিতীয় পত্র": {
                "Grammar & Composition": {
                    "Grammar Rules & Transformation": {
                        "MCQ": [
                            {"q": "Change the voice: 'She writes a letter.'", "options": ["a) A letter is written by her.", "b) A letter was written.", "c) A letter has been written.", "d) None"], "answer": "a) A letter is written by her."}
                        ],
                        "CQ": [
                            {
                                "stem": "Transform the following sentences as directed: (a) Bangladesh is a beautiful country. (Exclamatory) (b) It is richer in natural beauty than most other countries. (Superlative) (c) Who does not love his motherland? (Assertive) (d) We should protect our nature. (Passive)",
                                "ka": "What is a compound sentence?",
                                "kha": "Explain the rule of converting Comparative degree into Superlative degree.",
                                "ga": "Transform sentences (a), (b), (c), and (d) according to instructions.",
                                "gha": "\"Mastery over sentence transformation improves overall writing clarity.\" Analyze."
                            }
                        ],
                        "Written": [
                            {"title": "CV & Cover Letter", "desc": "Write a CV with a cover letter for the post of an Assistant Teacher."}
                        ]
                    }
                }
            },
            "গণিত": {
                "বীজগণিত ও জ্যামিতি": {
                    "বীজগণিতীয় রাশি ও সেটের ধারণা": {
                        "MCQ": [
                            {"q": "A = {1, 2, 3} হলে সেট A-এর উপসেটের সংখ্যা কত?", "options": ["ক) ৮টি", "খ) ৬টি", "গ) ৪টি", "ঘ) ২টি"], "answer": "ক) ৮টি"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: U={1,2,3,4,5,6,7}, A={2,4,6} এবং B={3,5,6} তিনটি সেট।",
                                "ka": "A ∩ B নির্ণয় কর।",
                                "kha": "A U B এবং (A U B)' নির্ণয় কর।",
                                "ga": "ডি মরগানের নিয়মের সত্যতা যাচাই কর।",
                                "gha": "\"ভেনচিত্র ও সেট তত্ত্ব গাণিতিক বিন্যাসকে সহজতর করে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "পদার্থবিজ্ঞান": {
                "বল ও গতি": {
                    "গতি ও শক্তি": {
                        "MCQ": [
                            {"q": "ত্বরণের একক কোনটি?", "options": ["ক) m/s^2", "খ) m/s", "গ) N/m", "ঘ) kg-m/s"], "answer": "ক) m/s^2"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: ১০ কেজি ভরের একটি বস্তুকে ৫ m/s^2 ত্বরণে ধাক্কা দেওয়া হলো। বস্তুটি ২০ মিটার দূরত্ব অতিক্রম করে থেমে গেল।",
                                "ka": "বল কাকে বলে?",
                                "kha": "নিউটন ও ডাইনের মধ্যে সম্পর্ক ব্যাখ্যা কর।",
                                "ga": "প্রযুক্ত বলের মান হিসাব কর।",
                                "gha": "\"শক্তির রূপান্তর ও কাজের পরিমাণ প্রযুক্ত বলের ওপর নির্ভরশীল।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "রসায়ন": {
                "পদার্থের গঠন": {
                    "পরমাণু ও বন্ধন": {
                        "MCQ": [
                            {"q": "সোডিয়াম (Na) এর পারমাণবিক সংখ্যা কত?", "options": ["ক) ১১", "খ) ১২", "গ) ১৩", "ঘ) ১৯"], "answer": "ক) ১১"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: মৌল A (পারমাণবিক সংখ্যা ১১) এবং মৌল B (পারমাণবিক সংখ্যা ১৭) পরস্পর যুক্ত হয়ে AB যৌগ গঠন করে।",
                                "ka": "যোজনী কাকে বলে?",
                                "kha": "নিষ্ক্রিয় গ্যাসগুলো রাসায়নিকভাবে নিষ্ক্রিয় কেন?",
                                "ga": "A ও B মৌলের মধ্যে বন্ধন গঠন প্রক্রিয়া ইলেকট্রন বিন্যাসের সাহায্যে ব্যাখ্যা কর।",
                                "gha": "\"গঠিত AB যৌগটি পানিতে দ্রবণীয় এবং জলীয় দ্রবণে বিদ্যুৎ পরিবহন করে।\"—যাচাই কর।"
                            }
                        ]
                    }
                }
            },
            "জীববিজ্ঞান": {
                "জীবকোষ টিস্যু": {
                    "কোষ বিভাজন": {
                        "MCQ": [
                            {"q": "কোন বিভাজনের মাধ্যমে জননকোষ গঠিত হয়?", "options": ["ক) মিওসিস", "খ) অ্যামাইটোসিস", "গ) মাইটোসিস", "ঘ) সাইটোকাইনেসিস"], "answer": "ক) মিওসিস"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: কোষ বিভাজনের দুটি চিত্র লক্ষ্য করা হলো: 'A' প্রক্রিয়ায় দেহের বৃদ্ধি ঘটে এবং 'B' প্রক্রিয়ায় অপত্য কোষে ক্রোমোজোম সংখ্যা অর্ধেক হয়ে যায়।",
                                "ka": "অ্যামাইটোসিস বিভাজন কাকে বলে?",
                                "kha": "মাইটোসিসকে সমীকরণিক বিভাজন বলা হয় কেন?",
                                "ga": "উদ্দীপকের 'A' ও 'B' প্রক্রিয়ার বৈসাদৃশ্য তুলনা কর।",
                                "gha": "\"জীবের ধারাবাহিকতা ও প্রজাতি রক্ষায় 'B' প্রক্রিয়ার গুরুত্ব অপরিসীম।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "হিসাববিজ্ঞান": {
                "হিসাব সমীকরণ": {
                    "জাবেদা ও খতিয়ান": {
                        "MCQ": [
                            {"q": "হিসাব সমীকরণের মূল উপাদান কোনটি?", "options": ["ক) A = L + E", "খ) A = L - E", "গ) E = A + L", "ঘ) L = A + E"], "answer": "ক) A = L + E"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: জনাব রফিক ২০২৪ সালের ১ জানুয়ারি তারিখে ৫০,০০০ টাকা নগদ ও ২০,০০০ টাকার আসবাবপত্র নিয়ে ব্যবসা শুরু করলেন।",
                                "ka": "প্রারম্ভিক মূলধনের পরিমাণ কত?",
                                "kha": "জানুয়ারি মাসের লেনদেনগুলোর জাবেদা দাখিলা দেখাও।",
                                "ga": "লেনদেনগুলোর প্রভাব হিসাব সমীকরণে (A = L + E) ছকে দেখাও।",
                                "gha": "\"সঠিক জাবেদা ভুক্তির ওপর নির্ভুল খতিয়ান প্রস্তুত নির্ভর করে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "ব্যবসায় উদ্যোগ": {
                "উদ্যোগ ও পরিকল্পনা": {
                    "ব্যবসায়িক ধারণা": {
                        "MCQ": [
                            {"q": "ব্যবসায়ের প্রধান ও মূল উদ্দেশ্য কোনটি?", "options": ["ক) মুনাফা অর্জন", "খ) জনকল্যাণ", "গ) পণ্য উৎপাদন", "ঘ) সেবা প্রদান"], "answer": "ক) মুনাফা অর্জন"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: তরুণ উদ্যোক্তা তৌহিদ তার এলাকায় প্লাস্টিক বর্জ্য প্রক্রিয়াজাত করে নতুন পণ্য তৈরির কারখানা গড়ে তোলেন।",
                                "ka": "ব্যবসায় উদ্যোক্তা কে?",
                                "kha": "ঝুঁকি গ্রহণ ও ব্যবসায়ের সাফল্যের সম্পর্ক ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের তৌহিদের উদ্যোগটি কীভাবে স্থানীয় অর্থনীতিতে প্রভাব ফেলছে—ব্যাখ্যা কর।",
                                "gha": "\"উদ্ভাবনী ক্ষমতাই একজন সফল উদ্যোক্তাকে সাধারণ ব্যবসায়ী থেকে আলাদা করে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "বাংলাদেশের ইতিহাস ও বিশ্বসভ্যতা": {
                "প্রাচীন সভ্যতা": {
                    "ইতিহাসের ধারা": {
                        "MCQ": [
                            {"q": "ইতিহাসের জনক কাকে বলা হয়?", "options": ["ক) হেরোডোটাস", "খ) থুসিডাইডিস", "গ) এরিস্টটল", "ঘ) ইবনে খালদুন"], "answer": "ক) হেরোডোটাস"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: শশাঙ্কের মৃত্যুর পর বাংলাজুড়ে তীব্র অরাজকতা ও বিশৃঙ্খলা দেখা দেয়, যেখানে শক্তিশালী ব্যক্তি দুর্বলদের ওপর অত্যাচার করত।",
                                "ka": "মৎস্যন্যায় কী?",
                                "kha": "পাল রাজত্বকে বাংলার স্বর্ণযুগ বলা হয় কেন?",
                                "ga": "উদ্দীপকে বর্ণিত ঐতিহাসিক প্রেক্ষাপট ও পাল বংশের শাসনের সূচনা আলোচনা কর।",
                                "gha": "\"জনগণের সর্বসম্মত সিদ্ধান্তই বাংলায় প্রথম স্থায়ী ও দীর্ঘস্থায়ী বংশের ভিত্তি গড়ে দিয়েছিল।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "পৌরনীতি ও নাগরিকতা": {
                "রাষ্ট্র ও সরকার": {
                    "নাগরিক ও সুনাগরিক": {
                        "MCQ": [
                            {"q": "সুনাগরিকের প্রধান গুণ কয়টি?", "options": ["ক) ৩টি (বুদ্ধি, বিবেক, আত্মসংযম)", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি (বুদ্ধি, বিবেক, আত্মসংযম)"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: সায়ান একজন বিবেকবান নাগরিক। সে কর প্রদান করে, সৎ প্রার্থীকে ভোট দেয় এবং রাষ্ট্রের সম্পদ রক্ষা করে।",
                                "ka": "সুনাগরিক কাকে বলে?",
                                "kha": "অধিকার ও কর্তব্যের পারস্পরিক সম্পর্ক ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকের সায়ানের আচরণের মাধ্যমে সুনাগরিকের কোন গুণাবলি প্রকাশ পেয়েছে—ব্যাখ্যা কর।",
                                "gha": "\"সুনাগরিকই একটি দেশের টেকসই উন্নয়ন ও সুশাসনের মূল চালিকাশক্তি।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "অর্থনীতি": {
                "মৌলিক অর্থনীতি": {
                    "চাহিদা ও যোগান": {
                        "MCQ": [
                            {"q": "অর্থনীতির জনক কাকে বলা হয়?", "options": ["ক) অ্যাডাম স্মিথ", "খ) লর্ড রবিন্স", "গ) আলফ্রেড মার্শাল", "ঘ) কেয়ন্স"], "answer": "ক) অ্যাডাম স্মিথ"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: বাজারে আলুর দাম প্রতি কেজি ২০ টাকা থেকে বেড়ে ৩০ টাকা হলে চাহিদার পরিমাণ ৫০ কেজি থেকে কমে ৩০ কেজি হয়।",
                                "ka": "চাহিদা কাকে বলে?",
                                "kha": "দুষ্প্রাপ্যতা বলতে কী বোঝায়?",
                                "ga": "উদ্দীপকের তথ্য থেকে একটি চাহিদা রেখা অঙ্কন কর।",
                                "gha": "\"দাম ও চাহিদার বিপরীতমুখী সম্পর্কই চাহিদা বিধির ভিত্তি।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            }
        },
        "দশম শ্রেণি": {
            "বাংলা প্রথম পত্র": {
                "সাহিত্য": {
                    "গদ্য ও পদ্য": {
                        "MCQ": [
                            {"q": "'বই পড়া' প্রবন্ধের লেখক কে?", "options": ["ক) প্রমথ চৌধুরী", "খ) রবীন্দ্রনাথ ঠাকুর", "গ) কাজী নজরুল ইসলাম", "ঘ) শরৎচন্দ্র চট্টোপাধ্যায়"], "answer": "ক) প্রমথ চৌধুরী"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: লাইব্রেরি হলো জাতির আত্মিক উন্নতির কেন্দ্র। সেখানে বসে মানুষ নিজের পছন্দমতো বই পড়ে জ্ঞানের আলোয় নিজেকে সমৃদ্ধ করে।",
                                "ka": "প্রমথ চৌধুরীর ছদ্মনাম কী?",
                                "kha": "লেখক কেন লাইব্রেরিকে হাসপাতালের চেয়েও বেশি গুরুত্বপূর্ণ বলেছেন?",
                                "ga": "উদ্দীপকের সাথে 'বই পড়া' প্রবন্ধের মূল চেতনার সাদৃশ্য কোথায়—ব্যাখ্যা করো।",
                                "gha": "\"স্বশিক্ষিত হওয়ার সবচেয়ে বড় উপায় হলো লাইব্রেরি বা বই পড়া।\"—বিশ্লেষণ করো।"
                            }
                        ]
                    }
                }
            },
            "বাংলা দ্বিতীয় পত্র": {
                "ব্যাকরণ ও নির্মিতি": {
                    "কারক ও বিভক্তি": {
                        "MCQ": [
                            {"q": "বাক্যে ক্রিয়াপদের সাথে কার সাথে কার সম্পর্ককে কারক বলে?", "options": ["ক) বিশেষ্যের", "খ) সর্বনামের", "গ) নামপদের সাথে ক্রিয়াপদের", "ঘ) অব্যয়ের"], "answer": "গ) নামপদের সাথে ক্রিয়াপদের"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "ভাবসম্প্রসারণ", "desc": "দুর্লভ জনম মিলিয়াছে যবে, সুকর্মে তা করো দান।"},
                            {"title": "আবেদনপত্র", "desc": "বন্যা দুর্গত এলাকায় ত্রাণের ব্যবস্থা করার জন্য জেলা প্রশাসকের নিকট আবেদনপত্র লেখো।"}
                        ]
                    }
                }
            },
            "ইংরেজি প্রথম পত্র": {
                "Reading & Writing": {
                    "Unit 4: Environment & Heritage": {
                        "MCQ": [
                            {"q": "What is the primary goal of sustainable development?", "options": ["a) Destroying nature", "b) Meeting current needs without harming future generations", "c) Urbanization only", "d) Industrial waste"], "answer": "b) Meeting current needs without harming future generations"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "Paragraph Writing", "desc": "Write a paragraph on 'Deforestation and Its Consequences'."},
                            {"title": "Formal Letter", "desc": "Write a letter to the editor about the nuisance of reckless driving."}
                        ]
                    }
                }
            },
            "ইংরেজি দ্বিতীয় পত্র": {
                "Grammar & Composition": {
                    "Completing Sentences": {
                        "MCQ": [
                            {"q": "Complete the sentence: 'Had I been a king, I ______'", "options": ["a) will help the poor", "b) would help the poor", "c) would have helped the poor", "d) helped"], "answer": "c) would have helped the poor"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "CV & Cover Letter", "desc": "Write a CV with a cover letter for the post of a Senior Teacher in English."},
                            {"title": "Composition", "desc": "Write a short composition on 'The Student Life and Duties of Students'."}
                        ]
                    }
                }
            },
            "গণিত": {
                "ত্রিকোণমিতি ও পরিমিতি": {
                    "দূরত্ব ও উচ্চতা": {
                        "MCQ": [
                            {"q": "sin^2(theta) + cos^2(theta) = কত?", "options": ["ক) ১", "খ) ০", "গ) ২", "ঘ) -১"], "answer": "ক) ১"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: একটি সমকোণী ত্রিভুজের লম্ব ও ভূমির দৈর্ঘ্য যথাক্রমে ৩ সেমি ও ৪ সেমি।",
                                "ka": "পিথাগোরাসের উপপাদ্যটি বিবৃত কর।",
                                "kha": "ত্রিভুজটির অতিভুজের দৈর্ঘ্য নির্ণয় কর।",
                                "ga": "ত্রিভুজটির অতিভুজের ওপর অঙ্কিত বর্গক্ষেত্রের ক্ষেত্রফল অপর দুই বাহুর ওপর অঙ্কিত বর্গক্ষেত্রদ্বয়ের ক্ষেত্রফলের সমষ্টির সমান—যাচাই কর।",
                                "gha": "\"জ্যামিতিক উপপাদ্যের সঠিক প্রয়োগ পরিমিতির জটিল সমস্যার সমাধান সহজ করে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "পদার্থবিজ্ঞান": {
                "বল ও আলো": {
                    "চলতড়িৎ ও বল": {
                        "MCQ": [
                            {"q": "রোধের একক কোনটি?", "options": ["ক) ওহম (Ohm)", "খ) ভোল্ট", "গ) অ্যাম্পিয়ার", "ঘ) ওয়াট"], "answer": "ক) ওহম (Ohm)"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: একটি বৈদ্যুতিক বাল্বের গায়ে '২২০V - ১০০W' লেখা আছে। বাল্বটিকে ২২০ ভোল্ট উৎসের সাথে যুক্ত করা হলো।",
                                "ka": "তড়িৎ ক্ষমতা কাকে বলে?",
                                "kha": "ওহমের সূত্রটি ব্যাখ্যা কর।",
                                "ga": "বাল্বটির মধ্য দিয়ে প্রবাহিত তড়িৎ প্রবাহের মান ও এর রোধ নির্ণয় কর।",
                                "gha": "\"তড়িৎ শক্তির সঠিক ব্যবহারে শক্তির অপচয় রোধ করা সম্ভব।\"—গাণিতিক যুক্তিসহ মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "রসায়ন": {
                "খনিজ সম্পদ ও এসিড": {
                    "রাসায়নিক বিক্রিয়া": {
                        "MCQ": [
                            {"q": "এসিড নীল লিটমাসকে কোন রঙে পরিণত করে?", "options": ["ক) লাল", "খ) নীল", "গ) হলুদ", "ঘ) সবুজ"], "answer": "ক) লাল"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: ল্যাবরেটরিতে হাইড্রোক্লোরিক এসিডের সাথে সোডিয়াম হাইড্রোক্সাইড দ্রবণ মিশ্রিত করায় একটি প্রশমন বিক্রিয়া ঘটে লবণ ও পানি উৎপন্ন হলো।",
                                "ka": "প্রশমন বিক্রিয়া কাকে বলে?",
                                "kha": "এসিড ব্যবহারের সতর্কতাসমূহ লেখো।",
                                "ga": "উদ্দীপকে সংঘটিত বিক্রিয়াটির সমীকরণ লিখে এর প্রকৃতি ব্যাখ্যা কর।",
                                "gha": "\"দৈনন্দিন জীবনে এসিড ও ক্ষারকের সঠিক ব্যবহার আমাদের স্বাস্থ্য সুরক্ষায় গুরুত্বপূর্ণ ভূমিকা রাখে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "জীববিজ্ঞান": {
                "জীবপ্রযুক্তি ও বংশগতি": {
                    "প্রজনন ও বাস্তুতন্ত্র": {
                        "MCQ": [
                            {"q": "বাস্তুতন্ত্রের প্রধান উপাদান কয়টি?", "options": ["ক) ২টি (জৈব ও অজৈব)", "খ) ৩টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ২টি (জৈব ও অজৈব)"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: একটি বনের বাস্তুতন্ত্রে উৎপাদক হিসেবে গাছপালা সৌরশক্তি ব্যবহার করে খাদ্য তৈরি করে, যা তৃণভোজী ও মাংসাশী প্রাণীরা গ্রহণ করে শক্তি প্রবাহ বজায় রাখে।",
                                "ka": "বাস্তুতন্ত্রের প্রধান উপাদান কয়টি?",
                                "kha": "খাদ্য জাল ও খাদ্য শৃঙ্খলের মধ্যে পার্থক্য লেখো।",
                                "ga": "উদ্দীপকের বাস্তুতন্ত্রে শক্তি প্রবাহের রূপান্তর প্রক্রিয়াটি ব্যাখ্যা কর।",
                                "gha": "\"বাস্তুতন্ত্রের পারস্পরিক ভারসাম্যের ওপরই পৃথিবীর জীববৈচিত্র্যের অস্তিত্ব টিকে রয়েছে।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "হিসাববিজ্ঞান": {
                "আর্থিক বিবরণী": {
                    "মজুদ পণ্য ও রেওয়ামিল": {
                        "MCQ": [
                            {"q": "রেওয়ামিল প্রস্তুতের প্রধান উদ্দেশ্য কী?", "options": ["ক) হিসাবের গাণিতিক শুদ্ধতা যাচাই", "খ) মুনাফা নির্ণয়", "গ) দেনা পরিশোধ", "ঘ) কর নির্ধারণ"], "answer": "ক) হিসাবের গাণিতিক শুদ্ধতা যাচাই"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: একটি ব্যবসায়ের লেনদেন থেকে প্রস্তুতকৃত হিসাবের ডেবিট ও ক্রেডিট উদ্বৃত্তগুলোর যোগফল সমান না হওয়ায় হিসাবরক্ষক গাণিতিক ত্রুটি খুঁজে বের করার উদ্যোগ নিলেন।",
                                "ka": "রেওয়ামিল কী?",
                                "kha": "কোন কোন ভুল রেওয়ামিলে ধরা পড়ে না—ব্যাখ্যা কর।",
                                "ga": "হিসাবের গাণিতিক শুদ্ধতা রক্ষায় রেওয়ামিলের ভূমিকা আলোচনা কর।",
                                "gha": "\"ত্রুটিমুক্ত রেওয়ামিল প্রস্তুত আর্থিক বিবরণী তৈরির পথ সুগম করে।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "ব্যবসায় উদ্যোগ": {
                "ব্যবসায় পরিকল্পনা": {
                    "বিপণন ও পরিকল্পনা": {
                        "MCQ": [
                            {"q": "SWOT বিশ্লেষণে 'O' দ্বারা কী বোঝায়?", "options": ["ক) Opportunity (সুযোগ)", "খ) Organization", "গ) Operation", "ঘ) Ownership"], "answer": "ক) Opportunity (সুযোগ)"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: রিনা আক্তার একটি হস্তশিল্প কারখানা চালুর পূর্বে বাজারের চাহিদা, মূলধনের উৎস ও প্রতিযোগীদের অবস্থান বিবেচনা করে একটি পূর্ণাঙ্গ নীল নকশা বা পরিকল্পনা তৈরি করলেন।",
                                "ka": "ব্যবসায়িক পরিকল্পনা কী?",
                                "kha": "SWOT বিশ্লেষণ কেন জরুরি—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে রিনা আক্তারের গৃহীত পদক্ষেপটির প্রয়োজনীয়তা আলোচনা কর।",
                                "gha": "\"পূর্ব পরিকল্পনা ও সঠিক ঝুঁকি মূল্যায়ন ছাড়া ব্যবসায় সফলতা অর্জন অসম্ভব।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "বাংলাদেশের ইতিহাস ও বিশ্বসভ্যতা": {
                "আধুনিক বাংলাদেশ": {
                    "ব্রিটিশ ও পাকিস্তান আমল": {
                        "MCQ": [
                            {"q": "কত সালে পলাশীর যুদ্ধ সংঘটিত হয়?", "options": ["ক) ১৭৫৭ সালে", "খ) ১৭৬৪ সালে", "গ) ১৯০৫ সালে", "ঘ) ১৯৪৭ সালে"], "answer": "ক) ১৭৫৭ সালে"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: ১৯৫২ সালের ২১ ফেব্রুয়ারি রাষ্ট্রভাষা বাংলার দাবিতে এদেশের ছাত্র-জনতা রাজপথে রক্তিম রক্ত ঢেলে দিয়েছিল, যা পরবর্তীতে বাঙালির স্বাধিকার আন্দোলনের চূড়ান্ত বিজয়ের পথ উন্মুক্ত করে।",
                                "ka": "ভাষা আন্দোলনের শহীদদের স্মরণে নির্মিত মিনারের প্রথম স্থপতি কে ছিলেন?",
                                "kha": "বায়ান্নর ভাষা আন্দোলনের ঐতিহাসিক গুরুত্ব ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে বাঙালি জাতির আত্মত্যাগের সাথে কোন জাতীয় চেতনার প্রকাশ ঘটেছে—আলোচনা কর।",
                                "gha": "\"ভাষা আন্দোলনই বাঙালি জাতীয়তাবাদের বীজ বপন করেছিল।\"—মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            },
            "পৌরনীতি ও নাগরিকতা": {
                "আইন ও সরকার": {
                    "সংবিধান ও গণতন্ত্র": {
                        "MCQ": [
                            {"q": "বাংলাদেশের সংবিধান কার্যকর হয় কত তারিখে?", "options": ["ক) ১৬ ডিসেম্বর ১৯৭২", "খ) ২৬ মার্চ ১৯৭১", "গ) ১৭ এপ্রিল ১৯৭১", "ঘ) ১০ জানুয়ারি ১৯৭২"], "answer": "ক) ১৬ ডিসেম্বর ১৯৭২"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: ১৯৭২ সালে প্রণীত বাংলাদেশের সংবিধানে গণতন্ত্র, সমাজতন্ত্র, ধর্মনিরপেক্ষতা ও জাতীয়তাবাদ—এই চার মূলনীতিকে ভিত্তি করে রাষ্ট্র পরিচালনার পথ নির্দেশ করা হয়েছে।",
                                "ka": "সংবিধান প্রণয়ন কমিটির প্রধান কে ছিলেন?",
                                "kha": "সংবিধানকে রাষ্ট্রের সর্বোচ্চ দলিল বলা হয় কেন—ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে বর্ণিত চার মূলনীতির আলোকে আমাদের গণতান্ত্রিক ব্যবস্থার স্বরূপ ব্যাখ্যা কর।",
                                "gha": "\"সংবিধানের অনুশাসন মেনে চলা প্রতিটি নাগরিকের মৌলিক দায়িত্ব।\"—আলোচনা কর।"
                            }
                        ]
                    }
                }
            },
            "অর্থনীতি": {
                "জাতীয় আয়": {
                    "ব্যাংক ব্যবস্থা": {
                        "MCQ": [
                            {"q": "কেন্দ্রীয় ব্যাংককে কী বলা হয়?", "options": ["ক) ব্যাংকের ব্যাংক", "খ) সাধারণ ব্যাংক", "গ) বাণিজ্যিক ব্যাংক", "ঘ) সঞ্চয়ী ব্যাংক"], "answer": "ক) ব্যাংকের ব্যাংক"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: দেশের মুদ্রাবাজারের অভিভাবক হিসেবে একটি নির্দিষ্ট ব্যাংক অন্যান্য বাণিজ্যিক ব্যাংকগুলোকে নিয়ন্ত্রণ করে এবং দেশের অর্থনৈতিক স্থিতিশীলতা বজায় রাখে।",
                                "ka": "বাংলাদেশের কেন্দ্রীয় ব্যাংকের নাম কী?",
                                "kha": "বাণিজ্যিক ব্যাংক ও কেন্দ্রীয় ব্যাংকের মধ্যে প্রধান পার্থক্য লেখো।",
                                "ga": "উদ্দীপকের ব্যাংকটির কার্যাবলি অর্থনীতির কোন দিকটিকে নির্দেশ করে—ব্যাখ্যা কর।",
                                "gha": "\"কেন্দ্রীয় ব্যাংক দেশের সামগ্রিক আর্থিক ব্যবস্থার মূল চালিকাশক্তি।\"—বিশ্লেষণ কর।"
                            }
                        ]
                    }
                }
            },
            "তথ্য ও যোগাযোগ প্রযুক্তি": {
                "ডিজিটাল বাংলাদেশ": {
                    "মাল্টিমিডিয়া ও গ্রাফিক্স": {
                        "MCQ": [
                            {"q": "ডিজিটাল কনটেন্ট প্রধানত কয় ভাগে বিভক্ত?", "options": ["ক) ৪ ভাগে", "খ) ২ ভাগে", "গ) ৩ ভাগে", "ঘ) ৫ ভাগে"], "answer": "ক) ৪ ভাগে"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: একটি সরকারি দপ্তরে ফাইলপত্র আগে কাগজের মাধ্যমে সংরক্ষণ করা হতো। বর্তমানে ই-সেবা ও ই-ফাইলিং চালুর ফলে নাগরিকেরা ঘরে বসেই দ্রুত সেবা পাচ্ছেন।",
                                "ka": "ই-গভর্নেন্স কী?",
                                "kha": "ই-সেবার দুটি সুবিধা ব্যাখ্যা কর।",
                                "ga": "উদ্দীপকে সেবা ব্যবস্থার পরিবর্তনের ফলে কীভাবে সময় ও অর্থ সাশ্রয় হচ্ছে তা আলোচনা কর।",
                                "gha": "\"ডিজিটাল প্রযুক্তির প্রবর্তন একটি স্বচ্ছ ও দুর্নীতিমুক্ত সমাজ গঠনে সহায়ক।\"—উক্তিটি মূল্যায়ন কর।"
                            }
                        ]
                    }
                }
            }
        }
    }

    # --- ৪টি নির্ধারিত ট্যাব ---
    tab1, tab2, tab3, tab4 = st.tabs([
        "Practice",
        "Quiz Test",
        "Model Test",
        "Archives"
    ])

    # ================= TAB 1: PRACTICE & PREPARATION =================
    with tab1:
        st.info("নিচে থেকে আপনার শ্রেণি, বিষয়, বিভাগ এবং অধ্যায় নির্বাচন করে প্র্যাকটিস শুরু করুন[cite: 247]।")
        
        c1, c2 = st.columns(2, gap="medium")
        with c1:
            p_class = st.selectbox("Class Selection:", list(library_question_bank.keys()), key="p_class_sel")
        with c2:
            p_books = list(library_question_bank[p_class].keys())
            p_book = st.selectbox("Subject Selection:", p_books, key="p_book_sel")
            
        c3, c4 = st.columns(2, gap="medium")
        with c3:
            p_sections = list(library_question_bank[p_class][p_book].keys())
            p_sec = st.selectbox("Item Selection:", p_sections, key="p_sec_sel")
        with c4:
            p_chapters = list(library_question_bank[p_class][p_book][p_sec].keys())
            p_chap = st.selectbox("Topic Selection:", p_chapters, key="p_chap_sel")
            
        st.markdown("<hr style='margin: 18px 0; border: 0; border-top: 2px dashed #cbd5e1;'>", unsafe_allow_html=True)
        
        chap_data = library_question_bank[p_class][p_book][p_sec][p_chap]
        
        # প্র্যাকটিস টাইপ সিলেকশন (MCQ, CQ বা Written Part)
        # নিয়ম অনুযায়ী বাংলা ২য়, ইংরেজি ১ম ও ২য় পত্রে Written অপشن থাকবে, বাকিগুলোতে শুধু MCQ ও CQ থাকবে।
        if p_book in ["বাংলা দ্বিতীয় পত্র", "ইংরেজি প্রথম পত্র", "ইংরেজি দ্বিতীয় পত্র"]:
            practice_type = st.radio(
                "কোনটি প্র্যাকটিস করতে চাও? বহুনির্বাচনী প্রশ্ন (MCQ), সৃজনশীল প্রশ্ন (CQ) নাকি রিটেন পার্ট?",
                ["MCQ", "CQ", "Written Part"],
                horizontal=True
            )
        else:
            practice_type = st.radio(
                "কোনটি প্র্যাকটিস করতে চাও? বহুনির্বাচনী প্রশ্ন (MCQ) নাকি সৃজনশীল প্রশ্ন (CQ)?",
                ["MCQ", "CQ"],
                horizontal=True
            )
        
        st.markdown("---")

        # ১. MCQ সেকশন
        if "MCQ" in practice_type:
            st.markdown("#### 🎯 বহুনির্বাচনী প্রশ্ন (MCQ) অনুশীলন")
            mcq_list = chap_data.get("MCQ", [])
            total_available = len(mcq_list)
            
            if total_available > 1:
                q_count_choice = st.slider("কতটি প্রশ্ন প্র্যাকটিস করতে চাও?", 1, max_value=total_available, value=min(5, total_available), step=1)
            elif total_available == 1:
                q_count_choice = 1
                st.info("এই অধ্যায়ে ১টি প্রশ্ন আছে।")
            else:
                q_count_choice = 0
                
            if total_available > 0:
                if st.button("Practice Start", key="start_mcq_prac"):
                    st.session_state["active_prac_mcqs"] = random.sample(mcq_list, q_count_choice)
                    st.session_state["prac_submitted"] = False
                    
                if "active_prac_mcqs" in st.session_state:
                    st.markdown("---")
                    for idx, item in enumerate(st.session_state["active_prac_mcqs"], 1):
                        st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                        st.radio(f"উত্তর দাও (প্রশ্ন {idx}):", item["options"], key=f"prac_q_{idx}", index=None)
                        st.markdown("")
                        
                    if st.button("উত্তরগুলো যাচাই করো", key="check_prac_ans"):
                        st.session_state["prac_submitted"] = True
                        st.rerun()
                        
                    if st.session_state.get("prac_submitted", False):
                        st.markdown("### 📊 প্র্যাকটিস ফলাফল ও সঠিক উত্তর:")
                        for idx, item in enumerate(st.session_state["active_prac_mcqs"], 1):
                            u_ans = st.session_state.get(f"prac_q_{idx}")
                            c_ans = item["answer"]
                            if u_ans == c_ans:
                                st.success(f"প্রশ্ন {idx}: সঠিক! (তোমার উত্তর: {u_ans})")
                            else:
                                st.error(f"প্রশ্ন {idx}: ভুল। (তোমার উত্তর: {u_ans if u_ans else 'উত্তর দেননি'}) | সঠিক উত্তর: **{c_ans}**")
            else:
                st.warning("এই অধ্যায়ে প্র্যাকটিসের জন্য কোনো MCQ প্রশ্ন পাওয়া যায়নি।")

        # ২. CQ (সৃজনশীল প্রশ্ন) সেকশন
        elif "CQ" in practice_type:
            st.markdown("#### 📝 সৃজনশীল প্রশ্ন (CQ) অনুশীলন ও গাইডলাইন")
            cq_list = chap_data.get("CQ", [])
            if cq_list:
                for idx, item in enumerate(cq_list, 1):
                    st.markdown(
                        f"""
                        <div class="cq-card">
                            <h4 style="color: #4f46e5; margin-top: 0;">সৃজনশীল প্রশ্ন - {idx}</h4>
                            <div class="cq-stem">{item['stem']}</div>
                            <div><b>ক) {item['ka']}</b></div>
                            <div><b>খ) {item['kha']}</b></div>
                            <div><b>গ) {item['ga']}</b></div>
                            <div><b>ঘ) {item['gha']}</b></div>
                        </div>
                        """,
                        unsafe_allow_html=True
                    )
                with st.expander("💡 উত্তরপত্র ও এআই গাইডলাইনের জন্য ক্লিক করুন"):
                    st.info("সৃজনশীল প্রশ্নগুলোর মূলভাব এবং উত্তরের কাঠামোগত নির্দেশিকা পেতে রিডিং রুমের ডানপাশের এআই অ্যাসিস্ট্যান্টের সাহায্য নিতে পারেন।")
            else:
                st.info("এই অধ্যায়ে কোনো সৃজনশীল প্রশ্ন যুক্ত করা হয়নি।")

        # ৩. Written Part সেকশন (শুধুমাত্র নির্দিষ্ট বিষয়সমূহের জন্য)
        elif "Written Part" in practice_type:
            st.markdown("#### ✍️ রিটেন পার্ট ও নির্মিতি অনুশীলন")
            written_list = chap_data.get("Written", [])
            if written_list:
                for idx, w_item in enumerate(written_list, 1):
                    st.markdown(
                        f"""
                        <div class="written-card">
                            <h4 style="color: #0284c7; margin-top: 0;">লিখিত টপিক - {idx}: {w_item['title']}</h4>
                            <p style="font-size: 15px; font-weight: 500; color: #334155;"><b>নির্দেশনা/বিষয়:</b> {w_item['desc']}</p>
                        </div>
                        """,
                        unsafe_allow_html=True
                    )
                st.info("💡 টিপস: এই বিষয়গুলোর ওপর নিজে খাতায় লিখে রিডিং রুমের নোট হাবে বা এআই বটের সাহায্যে আপনার লেখার মান যাচাই করে নিতে পারেন।")
            else:
                st.info("এই বিষয়ের জন্য নির্দিষ্ট কোনো রিটেন টপিক যুক্ত করা হয়নি।")

    # ================= TAB 2: QUIZ TEST =================
    with tab2:
        st.markdown("### 📝 কুইজ টেস্ট (মিনি পরীক্ষা ও স্কোরেং)")
        q_c1, q_c2 = st.columns(2, gap="medium")
        with q_c1:
            q_sel_class = st.selectbox("শ্রেণি বেছে নিন:", list(library_question_bank.keys()), key="quiz_class_sel")
        with q_c2:
            q_available_books = list(library_question_bank[q_sel_class].keys())
            q_selected_book = st.selectbox("বিষয় বেছে নিন:", q_available_books, key="quiz_book_sel")
            
        q_c3, q_c4 = st.columns(2, gap="medium")
        with q_c3:
            q_available_sections = list(library_question_bank[q_sel_class][q_selected_book].keys())
            q_selected_sec = st.selectbox("বিভাগ বেছে নিন:", q_available_sections, key="quiz_sec_sel")
        with q_c4:
            q_available_chapters = list(library_question_bank[q_sel_class][q_selected_book][q_selected_sec].keys())
            q_selected_chapter = st.selectbox("অধ্যায় বেছে নিন:", q_available_chapters, key="quiz_chap_sel")
            
        st.markdown("<hr style='margin: 18px 0; border: 0; border-top: 2px dashed #cbd5e1;'>", unsafe_allow_html=True)
        
        qc1, qc2 = st.columns(2, gap="medium")
        with qc1:
            quiz_marks = st.selectbox("কুইজের মান / প্রশ্ন সংখ্যা:", ["৫ নম্বর (৫ প্রশ্ন)", "১০ নম্বর (১০ প্রশ্ন)"], key="quiz_marks_sel")
        with qc2:
            study_hours = st.number_input("পড়াশোনার সময় (ঘণ্টা):", min_value=0.5, max_value=5.0, value=1.0, step=0.5, key="study_hours_input")
            
        st.markdown("<br>", unsafe_allow_html=True)
        if st.button("কুইজ টেস্ট শুরু করুন", use_container_width=True):
            st.session_state["quiz_active"] = True
            st.session_state["show_review"] = False
            st.session_state["is_quiz_saved"] = False
            chap_content = library_question_bank[q_sel_class][q_selected_book][q_selected_sec][q_selected_chapter]
            raw_qs = chap_content.get("MCQ", [])
            if raw_qs:
                q_limit = 5 if "৫" in quiz_marks else 10
                selected_subset = random.sample(raw_qs, min(len(raw_qs), q_limit))
                st.session_state["generated_quiz"] = selected_subset
                st.session_state["quiz_title_str"] = f"কুইজ: {q_selected_book} ({q_selected_chapter})"
                st.rerun()
            else:
                st.error("এই অধ্যায়ে কোনো কুইজ প্রশ্ন উপলব্ধ নেই।")

        if st.session_state.get("quiz_active", False) and "generated_quiz" in st.session_state:
            st.markdown("---")
            st.markdown(f"### {st.session_state['quiz_title_str']}")
            if st.session_state.get("show_review", False):
                st.markdown("### 📈 কুইজ ফলাফল ও সঠিক উত্তরসমূহ:")
                for idx, item in enumerate(st.session_state["generated_quiz"], 1):
                    user_ans = st.session_state.get(f"lib_q_{idx}")
                    correct_ans = item["answer"]
                    st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                    if user_ans == correct_ans:
                        st.success(f"আপনার উত্তর: {user_ans} (সঠিক আছে)")
                    else:
                        st.error(f"আপনার উত্তর: {user_ans} (ভুল ❌)")
                    st.info(f"সঠিক উত্তর হবে: **{correct_ans}**")
                st.markdown("---")
                if st.button("আবার নতুন কুইজ শুরু করুন", use_container_width=True):
                    st.session_state["quiz_active"] = False
                    st.session_state["show_review"] = False
                    st.rerun()
            else:
                for idx, item in enumerate(st.session_state["generated_quiz"], 1):
                    st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                    st.radio(f"উত্তর নির্বাচন করুন (প্রশ্ন {idx}):", item["options"], key=f"lib_q_{idx}", index=None)
                    st.markdown("")
                if st.button("কুইজ জমা দিন ও হিসাব তৈরি করুন", key="submit_quiz_btn", use_container_width=True):
                    correct_count = 0
                    total_q = len(st.session_state["generated_quiz"])
                    for idx, item in enumerate(st.session_state["generated_quiz"], 1):
                        u_ans = st.session_state.get(f"lib_q_{idx}")
                        c_ans = item["answer"]
                        if u_ans == c_ans:
                            correct_count += 1
                    wrong_count = total_q - correct_count
                    percentage = (correct_count / total_q) * 100 if total_q > 0 else 0
                    score_str = f"{correct_count}/{total_q} ({percentage:.0f}%)"
                    
                    if not st.session_state.get("is_quiz_saved", False):
                        q_date_str = datetime.now().strftime("%Y-%m-%d %I:%M%p")
                        conn_q = sqlite3.connect("smart_school_syllabus.db")
                        cur_q = conn_q.cursor()
                        cur_q.execute(
                            """CREATE TABLE IF NOT EXISTS quiz_test_history
                            (id INTEGER PRIMARY KEY AUTOINCREMENT, quiz_title TEXT, quiz_date TEXT, total_marks INT, obtained_score INT, correct_count INT, wrong_count INT, details_json TEXT)"""
                        )
                        cur_q.execute(
                            """INSERT INTO quiz_test_history (quiz_title, quiz_date, total_marks, obtained_score, correct_count, wrong_count, details_json)
                            VALUES (?, ?, ?, ?, ?, ?, ?)""",
                            (
                                st.session_state["quiz_title_str"],
                                q_date_str,
                                total_q,
                                correct_count,
                                correct_count,
                                wrong_count,
                                json.dumps(st.session_state["generated_quiz"], ensure_ascii=False)
                            ),
                        )
                        conn_q.commit()
                        conn_q.close()
                        st.session_state["is_quiz_saved"] = True
                    st.balloons()
                    st.success(f"অভিনন্দন! আপনার কুইজ স্কোর: **{score_str}**")
                    st.session_state["show_review"] = True
                    st.rerun()

    # ================= TAB 3: MODEL TEST =================
    with tab3:
        st.markdown("### 📋 পূর্ণাঙ্গ মডেল টেস্ট (লাইভ কাউন্টডাউন ও অটো-সাবমিট)")
        if not st.session_state.exam_started:
            with st.form("model_test_custom_config_form"):
                st.markdown("#### মডেল টেস্ট কনফিগার করুন")
                sel_class = st.selectbox("শ্রেণি বেছে নিন:", list(library_question_bank.keys()), key="m_class_select")
                available_subjects = list(library_question_bank[sel_class].keys())
                sel_subjects = st.multiselect("বিষয়সমূহ নির্বাচন করুন:", available_subjects, default=available_subjects[:1], key="m_subjects_select")
                
                all_available_chapters = []
                for sub in sel_subjects:
                    for sec, sec_dict in library_question_bank[sel_class][sub].items():
                        for ch in sec_dict.keys():
                            all_available_chapters.append(f"{sub} -> {sec} -> {ch}")
                sel_chapters = st.multiselect("অধ্যায়সমূহ নির্বাচন করুন:", all_available_chapters, default=all_available_chapters, key="m_chapters_select")
                test_name_input = st.text_input("মডেল টেস্টের শিরোনাম:", value=f"{sel_class} মডেল টেস্ট", key="m_test_name_input_val")
                
                col_el, col_e2 = st.columns(2)
                with col_el:
                    custom_marks = st.number_input("পূর্ণমান / প্রশ্ন সংখ্যা:", min_value=1, max_value=50, value=5, step=1, key="m_custom_marks_input")
                with col_e2:
                    custom_time = st.number_input("নির্ধারিত সময় (মিনিট):", min_value=1, max_value=120, value=5, step=1, key="m_custom_time_input")
                    
                start_exam_btn = st.form_submit_button("মডেল টেস্ট শুরু করুন", use_container_width=True, type="primary")
                if start_exam_btn:
                    if not sel_subjects or not sel_chapters:
                        st.error("দয়া করে কমপক্ষে একটি বিষয় এবং অধ্যায় সিলেক্ট করুন।")
                    else:
                        pool_questions = []
                        for item in sel_chapters:
                            parts = item.split(" -> ")
                            sub_name, sec_name, ch_name = parts[0], parts[1], parts[2]
                            ch_data = library_question_bank[sel_class][sub_name][sec_name].get(ch_name, {})
                            if isinstance(ch_data, dict):
                                pool_questions.extend(ch_data.get("MCQ", []))
                        if not pool_questions:
                            st.error("নির্বাচিত অধ্যায়গুলোতে পর্যাপ্ত MCQ প্রশ্ন পাওয়া যায়নি।")
                        else:
                            num_q_needed = min(len(pool_questions), int(custom_marks))
                            selected_qs = random.sample(pool_questions, num_q_needed)
                            st.session_state.exam_questions = selected_qs
                            st.session_state.exam_started = True
                            st.session_state.exam_submitted = False
                            st.session_state.is_saved_to_db = False
                            st.session_state.user_answers = {}
                            st.session_state.exam_title_str = test_name_input
                            st.session_state.exam_total_marks = int(custom_marks)
                            st.session_state.exam_time_allocated = int(custom_time)
                            st.session_state.exam_deadline = time.time() + (int(custom_time) * 60)
                            st.rerun()

        if st.session_state.exam_started and not st.session_state.exam_submitted:
            current_time = time.time()
            time_left = int(st.session_state.exam_deadline - current_time)
            if time_left <= 0 and not st.session_state.exam_submitted:
                st.session_state.exam_submitted = True
                st.rerun()
            m_left = max(0, time_left // 60)
            s_left = max(0, time_left % 60)
            time_str_display = f"{m_left:02d}:{s_left:02d}"
            
            st.markdown(
                f"""
                <div style="background: linear-gradient(135deg, #312e81 0%, #4338ca 100%); color: white; padding: 15px 20px; border-radius: 12px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
                    <div style="font-size: 16px; font-weight: bold;">
                        পরীক্ষা চলছে: <span style="color: #fde047;">{st.session_state.exam_title_str}</span>
                    </div>
                    <div style="background-color: #ef4444; color: white; padding: 6px 14px; border-radius: 8px; font-size: 20px; font-weight: bold; font-family: monospace;">
                        {time_str_display}
                    </div>
                </div>
                """,
                unsafe_allow_html=True
            )
            with st.form("active_custom_exam_form"):
                for idx, q_data in enumerate(st.session_state.exam_questions):
                    st.markdown(f"**প্রশ্ন {idx + 1}: {q_data['q']}**")
                    options = q_data["options"]
                    ans_key = f"q_ans_live_{idx}"
                    user_choice = st.radio(f"উত্তর নির্বাচন করুন ({idx + 1}):", options, key=ans_key, index=None)
                    st.session_state.user_answers[idx] = user_choice
                    st.markdown("<hr style='margin: 10px 0; border: 0.5px solid #cbd5e1;'>", unsafe_allow_html=True)
                submit_test = st.form_submit_button("পরীক্ষা সম্পন্ন ও খাতা জমা দিন", use_container_width=True, type="primary")
                if submit_test:
                    st.session_state.exam_submitted = True
                    st.rerun()
            if time_left > 0 and not st.session_state.exam_submitted:
                time.sleep(1)
                st.rerun()

        if st.session_state.get("exam_submitted", False):
            score = 0
            total_q = len(st.session_state.exam_questions)
            correct_cnt = 0
            wrong_cnt = 0
            for idx, q_data in enumerate(st.session_state.exam_questions):
                user_ans = st.session_state.user_answers.get(idx)
                correct_ans = q_data["answer"]
                if user_ans == correct_ans:
                    score += 1
                    correct_cnt += 1
                else:
                    wrong_cnt += 1
            percentage = (score / total_q) * 100 if total_q > 0 else 0
            
            if not st.session_state.is_saved_to_db:
                current_date_str = datetime.now().strftime("%Y-%m-%d %I:%M%p")
                details_data = json.dumps(st.session_state.exam_questions, ensure_ascii=False)
                conn = sqlite3.connect("smart_school_syllabus.db")
                cursor = conn.cursor()
                cursor.execute(
                    """CREATE TABLE IF NOT EXISTS model_test_history
                    (id INTEGER PRIMARY KEY AUTOINCREMENT, test_title TEXT, test_date TEXT, total_marks INT, allocated_time INT, obtained_score INT, correct_count INT, wrong_count INT, details_json TEXT)"""
                )
                cursor.execute(
                    """INSERT INTO model_test_history (test_title, test_date, total_marks, allocated_time, obtained_score, correct_count, wrong_count, details_json)
                    VALUES (?, ?, ?, ?, ?, ?, ?, ?)""",
                    (
                        st.session_state.exam_title_str,
                        current_date_str,
                        st.session_state.exam_total_marks,
                        st.session_state.exam_time_allocated,
                        score,
                        correct_cnt,
                        wrong_cnt,
                        details_data,
                    ),
                )
                conn.commit()
                conn.close()
                st.session_state.is_saved_to_db = True
                
            st.success("আপনার মডেল টেস্ট সফলভাবে জমা দেওয়া হয়েছে!")
            st.markdown("### পরীক্ষার ফলাফল ও মূল্যায়ন")
            st.success(f"**পূর্ণমান:** {st.session_state.exam_total_marks} | **প্রাপ্ত স্কোর:** {score} | **সঠিক:** {correct_cnt} টি | **ভুল:** {wrong_cnt} টি ({percentage:.1f}%)")
            
            with st.expander("উত্তরপত্র ও সঠিক উত্তরসমূহ দেখে নিন", expanded=True):
                for idx, q_data in enumerate(st.session_state.exam_questions):
                    user_ans = st.session_state.user_answers.get(idx)
                    correct_ans = q_data["answer"]
                    if user_ans == correct_ans:
                        st.markdown(f"✅ **প্রশ্ন {idx + 1}:** সঠিক (আপনার উত্তর: {user_ans})")
                    else:
                        st.markdown(f"❌ **প্রশ্ন {idx + 1}:** ভুল (আপনার উত্তর: {user_ans if user_ans else 'উত্তর দেননি'})। সঠিক উত্তর: **{correct_ans}**")
                        
            if st.button("নতুন মডেল টেস্টে ফিরে যান", use_container_width=True, type="primary"):
                st.session_state.exam_started = False
                st.session_state.exam_submitted = False
                st.session_state.is_saved_to_db = False
                st.session_state.exam_questions = []
                st.session_state.user_answers = {}
                st.rerun()

    # ================= TAB 4: ARCHIVES =================
    with tab4:
        st.markdown("### 📚 সংরক্ষিত পরীক্ষার ইতিহাস (মডেল টেস্ট ও কুইজ টেস্ট)")
        history_tab_choice = st.radio("কোন ইতিহাস দেখতে চান?", ["মডেল টেস্ট ইতিহাস", "কুইজ টেস্ট ইতিহাস"], horizontal=True)
        if "মডেল" in history_tab_choice:
            st.markdown("#### মডেল টেস্ট ইতিহাসসমূহ:")
            conn = sqlite3.connect("smart_school_syllabus.db")
            cursor = conn.cursor()
            cursor.execute("CREATE TABLE IF NOT EXISTS model_test_history (id INTEGER PRIMARY KEY AUTOINCREMENT, test_title TEXT, test_date TEXT, total_marks INT, allocated_time INT, obtained_score INT, correct_count INT, wrong_count INT, details_json TEXT)")
            cursor.execute("SELECT id, test_title, test_date, total_marks, allocated_time, obtained_score, correct_count, wrong_count, details_json FROM model_test_history ORDER BY id DESC")
            past_tests = cursor.fetchall()
            conn.close()
            
            if past_tests:
                for pt in past_tests:
                    t_id, t_title, t_date, t_total, t_time, t_score, t_correct, t_wrong, tjson = pt
                    with st.expander(f"📌 {t_title} | তারিখ: {t_date} | প্রাপ্ত স্কোর: {t_score}/{t_total}"):
                        col_hs1, col_hs2 = st.columns(2)
                        with col_hs1:
                            st.markdown(f"- **পূর্ণমান:** {t_total}")
                            st.markdown(f"- **নির্ধারিত সময়:** {t_time} মিনিট")
                        with col_hs2:
                            st.markdown(f"- **সঠিক উত্তর:** {t_correct} টি")
                            st.markdown(f"- **ভুল উত্তর:** {t_wrong} টি")
                        col_h1, col_h2 = st.columns(2)
                        with col_h1:
                            if st.button("পুনরায় এই টেস্টটি দিন (Retest)", key=f"retest_btn_{t_id}", use_container_width=True, type="primary"):
                                st.session_state.exam_questions = json.loads(tjson)
                                st.session_state.exam_started = True
                                st.session_state.exam_submitted = False
                                st.session_state.is_saved_to_db = False
                                st.session_state.user_answers = {}
                                st.session_state.exam_title_str = f"পুনরায় পরীক্ষা: {t_title}"
                                st.session_state.exam_total_marks = t_total
                                st.session_state.exam_time_allocated = t_time
                                st.session_state.exam_deadline = time.time() + (t_time * 60)
                                st.rerun()
                        with col_h2:
                            if st.button("ইতিহাস ডিলিট করুন", key=f"del_history_btn_{t_id}", use_container_width=True):
                                con_del = sqlite3.connect("smart_school_syllabus.db")
                                cur_del = con_del.cursor()
                                cur_del.execute("DELETE FROM model_test_history WHERE id=?", (t_id,))
                                con_del.commit()
                                con_del.close()
                                st.success("মডেল টেস্ট ইতিহাস মুছে ফেলা হয়েছে!")
                                st.rerun()
            else:
                st.info("ℹ️ এখনও কোনো মডেল টেস্ট দেওয়া হয়নি।")
        else:
            st.markdown("#### কুইজ টেস্ট ইতিহাসসমূহ:")
            conn_q = sqlite3.connect("smart_school_syllabus.db")
            cursor_q = conn_q.cursor()
            cursor_q.execute("CREATE TABLE IF NOT EXISTS quiz_test_history (id INTEGER PRIMARY KEY AUTOINCREMENT, quiz_title TEXT, quiz_date TEXT, total_marks INT, obtained_score INT, correct_count INT, wrong_count INT, details_json TEXT)")
            cursor_q.execute("SELECT id, quiz_title, quiz_date, total_marks, obtained_score, correct_count, wrong_count FROM quiz_test_history ORDER BY id DESC")
            past_quizzes = cursor_q.fetchall()
            conn_q.close()
            
            if past_quizzes:
                for pq in past_quizzes:
                    q_id, q_title, q_date, q_total, q_score, q_cor, q_wro = pq
                    with st.expander(f"📝 {q_title} | তারিখ: {q_date} | স্কোর: {q_score}/{q_total}"):
                        st.markdown(f"- **মোট প্রশ্ন:** {q_total}")
                        st.markdown(f"- **সঠিক উত্তর:** {q_cor} টি")
                        st.markdown(f"- **ভুল উত্তর:** {q_wro} টি")
                        if st.button("কুইজ ইতিহাস ডিলিট করুন", key=f"del_quiz_btn_{q_id}", use_container_width=True):
                            con_del = sqlite3.connect("smart_school_syllabus.db")
                            cur_del = con_del.cursor()
                            cur_del.execute("DELETE FROM quiz_test_history WHERE id=?", (q_id,))
                            con_del.commit()
                            con_del.close()
                            st.success("কুইজ টেস্ট ইতিহাস সফলভাবে মুছে ফেলা হয়েছে!")
                            st.rerun()
            else:
                st.info("ℹ️ এখনও কোনো কুইজ টেস্ট দেওয়া হয়নি।")
                
    st.markdown("</div>", unsafe_allow_html=True)


    # Note Book Option এর কোডগুলো এখান থেকে শুরু...

import sqlite3
import streamlit as st
import streamlit.components.v1 as components

# --- SQLite ডেটাবেজ ইনিশিয়ালাইজেশন ---
def init_db():
    conn = sqlite3.connect("subject_notes.db")
    cursor = conn.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS notes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT,
        content TEXT,
        subject TEXT,
        note_type TEXT
    )
    """)
    conn.commit()
    conn.close()

init_db()

def render_notes():
    # কাস্টম CSS দিয়ে সমস্ত ইনপুট বক্স, টেক্সট এরিয়া ও বাটন আরও আকর্ষণীয় এবং প্রিমিয়াম করা
    st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;500;600;700&family=Outfit:wght@400;600;800&display=swap');

    .notes-main-wrapper {
        font-family: 'Outfit', 'Hind Siliguri', sans-serif;
    }

    /* টেক্সট ইনপুট ও সিলেক্ট বক্সের বর্ডার স্পষ্ট ও প্রিমিয়াম করা */
    .stTextInput input, .stSelectbox select {
        border: 2px solid #38bdf8 !important;
        border-radius: 12px !important;
        background-color: #f8fafc !important;
        font-family: 'Outfit', 'Hind Siliguri', sans-serif !important;
    }

    .stTextInput input:focus, .stSelectbox select:focus {
        border-color: #0284c7 !important;
        box-shadow: 0 0 0 3px rgba(14, 165, 233, 0.2) !important;
    }

    /* মূল টেক্সট এরিয়া বক্সের বর্ডার এবং লুক */
    textarea {
        border: 2px solid #38bdf8 !important;
        border-radius: 12px !important;
        background-color: #f8fafc !important;
        padding: 12px !important;
        font-family: 'Outfit', 'Hind Siliguri', sans-serif !important;
    }

    textarea:focus {
        border-color: #0284c7 !important;
        box-shadow: 0 0 0 3px rgba(14, 165, 233, 0.2) !important;
    }
    </style>

    <div class="notes-main-wrapper">
        <div style="background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%); padding: 25px; border-radius: 20px; color: white; margin-bottom: 20px; border: 1px solid #334155;">
            <h2 style='color: #38bdf8; font-family: Outfit; font-weight: 800; margin-bottom: 5px; font-size: 26px;'>✨ Your Virtual Reading Room-Notebook Section</h2>
            <p style='color: #94a3b8; font-size: 14px; font-family: Hind Siliguri;'> A Smart Hub for Your Notebook, Format Text, Build Tables, Search Instantly & Export Anywhere."</p>
        </div>
    </div>
    """, unsafe_allow_html=True)

    # সেশন স্টেট ইনিশিয়ালাইজ করা
    if "edit_id" not in st.session_state:
        st.session_state["edit_id"] = None
    if "current_title" not in st.session_state:
        st.session_state["current_title"] = ""
    if "current_content" not in st.session_state:
        st.session_state["current_content"] = ""
    if "current_subject" not in st.session_state:
        st.session_state["current_subject"] = "বাংলা"
    if "clear_trigger" not in st.session_state:
        st.session_state["clear_trigger"] = 0

    # এইচটিএমএল ও জাভাস্ক্রিপ্ট ভিত্তিক ফুল-ফিচারড রিচ টেক্সট এডিটর
    editor_html_template = """
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <link href="https://cdn.jsdelivr.net/npm/quill@2.0.0/dist/quill.snow.css" rel="stylesheet">
    <style>
    body { font-family: 'Outfit', 'Hind Siliguri', sans-serif; background-color: #f8fafc; padding: 5px; margin: 0; }
    #editor-container { height: 280px; background: white; border-bottom-left-radius: 12px; border-bottom-right-radius: 12px; border: 1.5px solid #38bdf8; }
    .ql-toolbar { background: #e0f2fe; border-top-left-radius: 12px; border-top-right-radius: 12px; border-color: #38bdf8 !important; display: flex; flex-wrap: wrap; gap: 4px; align-items: center; }
    .ql-container { border-color: #38bdf8 !important; font-size: 15px; font-family: 'Outfit', sans-serif; }
    .ql-editor table { border-collapse: collapse !important; width: 100% !important; margin: 12px 0 !important; table-layout: fixed !important; }
    .ql-editor th, .ql-editor td { border: 1px solid #94a3b8 !important; padding: 10px 12px !important; word-break: break-word !important; vertical-align: top !important; }
    .ql-editor th { font-weight: bold !important; color: white !important; }
    .table-btn { background: #0284c7; color: white; border: none; padding: 6px 12px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: bold; }
    .table-btn:hover { background: #0369a1; }
    </style>
    </head>
    <body>
    <div style="background: #e0f2fe; padding: 8px 12px; border: 1.5px solid #38bdf8; border-bottom: none; display: flex; gap: 12px; align-items: center; font-size: 13px; border-top-left-radius: 12px; border-top-right-radius: 12px; font-weight: 600; color: #0369a1;">
    <span><b>📊 টেবিল টুলস: </b></span>
    <label>রো: <input type="number" id="t-rows" min="1" max="10" value="3" style="width: 45px; padding: 2px; border-radius: 4px; border: 1px solid #bae6fd;"></label>
    <label>কলাম: <input type="number" id="t-cols" min="1" max="10" value="3" style="width: 45px; padding: 2px; border-radius: 4px; border: 1px solid #bae6fd;"></label>
    <label>থিম কালার:
    <select id="t-color" style="padding: 3px; border-radius: 4px; border: 1px solid #bae6fd;">
    <option value="#0284c7"> নীল (Blue)</option>
    <option value="#059669">সবুজ (Green) </option>
    <option value="#dc2626">লাল (Red) </option>
    <option value="#475569">ধূসর (Gray) </option>
    </select>
    </label>
    <button class="table-btn" onclick="insertTable()">+ টেবিল তৈরি করো</button>
    </div>
    <div id="editor-container">REPLACE_CONTENT</div>
    <script src="https://cdn.jsdelivr.net/npm/quill@2.0.0/dist/quill.js"></script>
    <script>
    const quill = new Quill('#editor-container', {
      theme: 'snow',
      modules: {
        toolbar: [
          [{ 'font': [] }, { 'size': ['small', false, 'large', 'huge'] }],
          ['bold', 'italic', 'underline', 'strike'],
          [{ 'color': [] }, { 'background': [] }],
          [{ 'align': [] }],
          [{ 'list': 'ordered'}, { 'list': 'bullet' }],
          ['clean']
        ]
      },
      placeholder: 'এখানে লিখো বা পেস্ট করো...'
    });
    function insertTable() {
      const rows = parseInt(document.getElementById('t-rows').value) || 2;
      const cols = parseInt(document.getElementById('t-cols').value) || 2;
      const themeColor = document.getElementById('t-color').value;
      let html = '<p><br></p><table style="border-collapse: collapse; width: 100%; margin: 12px 0;">';
      for (let r = 0; r < rows; r++) {
        html += '<tr>';
        for (let c = 0; c < cols; c++) {
          if (r === 0) {
            html += '<th style="border: 1px solid #94a3b8; padding: 10px 12px; background-color: ' + themeColor + '; color: white; font-weight: bold;">হেডার ' + (c + 1) + '</th>';
          } else {
            html += '<td style="border: 1px solid #94a3b8; padding: 10px 12px;">তথ্য ' + r + ',' + (c + 1) + '</td>';
          }
        }
        html += '</tr>';
      }
      html += '</table><p><br></p>';
      const range = quill.getSelection(true);
      quill.clipboard.dangerouslyPasteHTML(range.index, html);
    }
    quill.on('text-change', function() {
      const htmlContent = quill.root.innerHTML;
      window.parent.postMessage({ type: 'streamlit:setComponentValue', value: htmlContent }, '*');
    });
    </script>
    </body>
    </html>
    """

    editor_html = editor_html_template.replace("REPLACE_CONTENT", st.session_state["current_content"])

    col_editor, col_sidebar = st.columns([2, 1], gap="large")

    with col_editor:
        st.markdown("<h3 style='color: #0284c7; font-size: 20px; font-weight: 700;'>📝Text Editor</h3>", unsafe_allow_html=True)
        
        c_top1, c_top2, c_top3 = st.columns([2, 1, 1], gap="small")
        with c_top1:
            note_title = st.text_input(
                "নোটের একটি শিরোনাম দাও (Title):",
                value=st.session_state["current_title"],
                key=f"rich_note_title_{st.session_state['clear_trigger']}"
            )
        with c_top2:
            subjects_list = [
                "বাংলা", "ইংরেজি", "গণিত", "বিজ্ঞান",
                "বাংলাদেশ ও বিশ্বপরিচয়", "ধর্ম", "আইসিটি",
                "অন্যান্য"
            ]
            try:
                default_sub_idx = subjects_list.index(st.session_state["current_subject"])
            except ValueError:
                default_sub_idx = 0
            
            note_subject = st.selectbox(
                "নোটের বিষয় নির্বাচন করো:",
                subjects_list,
                index=default_sub_idx,
                key=f"rich_note_subject_{st.session_state['clear_trigger']}"
            )
        with c_top3:
            st.markdown("<br>", unsafe_allow_html=True)
            if st.button("✨ + নতুন নোট", use_container_width=True):
                st.session_state["current_title"] = ""
                st.session_state["current_content"] = ""
                st.session_state["edit_id"] = None
                st.session_state["clear_trigger"] += 1
                st.rerun()

        # রিচ টেক্সট এডিটর রেন্ডার
        components.html(editor_html, height=410)

        st.markdown("---")
        save_choice = st.radio(
            "নোটের ধরন নির্বাচন করো:",
            ["Draft", "Final"],
            horizontal=True,
            key="note_save_type_radio"
        )

        # অত্যন্ত সুন্দর ও স্পষ্ট হাইলাইটেড ইন্সট্রাকশন বক্স
        st.markdown("""
        <div style="background-color: #e0f2fe; border: 1.5px solid #bae6fd; border-left: 6px solid #0284c7; padding: 14px 18px; border-radius: 12px; margin-top: 15px; margin-bottom: 10px; box-shadow: 0 4px 12px rgba(14,165,233,0.06);">
        <p style="color: #0369a1; font-size: 14px; font-weight: 600; margin: 0; line-height: 1.5;">
        💡 Text Editor-এর লেখা Note Editor-এর বক্সে পেস্ট করো এবং নোট হিসেবে সেভ করতে 'Save'বাটনে ক্লিক করো:
        </p>
        </div>
        """, unsafe_allow_html=True)

        # ব্যাকআপ টেক্সট বক্স
        backup_body = st.text_area(
            "Note Editor:",
            value=st.session_state["current_content"],
            height=120,
            placeholder="এখানে পেস্ট করো অথবা লিখ...",
            key=f"backup_editor_textarea_{st.session_state['clear_trigger']}"
        )
        typed_body = backup_body

        # সেভ বা আপডেট বাটন
        btn_label = "🔄 Note Update" if st.session_state["edit_id"] is not None else "💾 Save"
        if st.button(btn_label, use_container_width=True, type="primary"):
            if note_title and typed_body:
                status_type = "ড্রাফট (Draft)" if "ড্রাফট" in save_choice else "ফাইনাল (Final)"
                conn = sqlite3.connect("subject_notes.db")
                cursor = conn.cursor()
                if st.session_state["edit_id"] is not None:
                    cursor.execute("""
                    UPDATE notes SET title = ?, content = ?, subject = ?, note_type = ?
                    WHERE id = ?
                    """, (note_title, typed_body, note_subject, status_type, st.session_state["edit_id"]))
                    conn.commit()
                    conn.close()
                    st.session_state["edit_id"] = None
                    st.success("✅ নোটটি সফলভাবে আপডেট হয়েছে!")
                else:
                    cursor.execute("""
                    INSERT INTO notes (title, content, subject, note_type)
                    VALUES (?, ?, ?, ?)
                    """, (note_title, typed_body, note_subject, status_type))
                    conn.commit()
                    conn.close()
                    st.success(f"🎉 তোমার নোটটি '{note_subject}' বিষয়ে সফলভাবে '{status_type}' হিসেবে সংরক্ষিত হয়েছে!")
                
                st.session_state["current_title"] = ""
                st.session_state["current_content"] = ""
                st.session_state["clear_trigger"] += 1
                st.rerun()
            else:
                st.error("⚠️ অনুগ্রহ করে নোটের শিরোনাম এবং নোটের বিষয়বস্তু উভয়ই প্রদান করো।")

        # ফাইল ডাউনলোড অপশন
        st.markdown("<br>", unsafe_allow_html=True)
        st.markdown("<h4 style='color: #0f172a; font-size: 16px;'>📥 Save As (নোট ডাউনলোড অপশন)</h4>", unsafe_allow_html=True)
        download_text_data = backup_body if backup_body else st.session_state["current_content"]
        
        if download_text_data:
            st.info("📁 ফাইলগুলো তোমার কম্পিউটারের **Downloads** ফোল্ডারে সেভ হবে।")
            dl_col1, dl_col2, dl_col3 = st.columns(3, gap="small")
            safe_filename = note_title.strip() if note_title.strip() else "My_Document"
            with dl_col1:
                st.download_button(
                    label="📄 Save as Text (.txt)",
                    data=download_text_data,
                    file_name=f"{safe_filename}.txt",
                    mime="text/plain",
                    use_container_width=True
                )
            with dl_col2:
                word_html_data = f"<html><head><meta charset='utf-8'></head><body><h2>{note_title} ({note_subject})</h2>{download_text_data}</body></html>"
                st.download_button(
                    label="📝 Save as Word (.doc)",
                    data=word_html_data,
                    file_name=f"{safe_filename}.doc",
                    mime="application/msword",
                    use_container_width=True
                )
            with dl_col3:
                st.download_button(
                    label="🌐 Save as HTML/PDF",
                    data=word_html_data,
                    file_name=f"{safe_filename}.html",
                    mime="text/html",
                    use_container_width=True
                )

        # ডেটাবেজ থেকে সমস্ত সংরক্ষিত নোট ফেচ করা
        conn = sqlite3.connect("subject_notes.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, title, content, subject, note_type FROM notes ORDER BY id DESC")
        db_notes = cursor.fetchall()
        conn.close()

        # সংরক্ষিত নোট তালিকা ও সার্চ অপশন
        if db_notes:
            st.markdown("---")
            st.markdown("<h3 style='color: #0284c7; font-size: 18px;'>📚 Your Notes</h3>", unsafe_allow_html=True)
            filter_subject = st.selectbox(
                "বিষয় অনুযায়ী নোট ফিল্টার করো:",
                ["সকল বিষয়"] + subjects_list,
                key="filter_subject_selectbox"
            )
            search_query = st.text_input(
                "🔍 নোট খোঁজ (শিরোনাম বা যেকোন শব্দ দিয়ে):",
                placeholder="এখানে নোটের নাম লিখ...",
                key="search_notes_input"
            )
            
            filtered_notes = []
            for row in db_notes:
                n_id, n_title, n_content, n_subject, n_type = row
                sub_match = True if filter_subject == "সকল বিষয়" else (n_subject == filter_subject)
                text_match = search_query.lower() in n_title.lower() or search_query.lower() in n_content.lower()
                if sub_match and text_match:
                    filtered_notes.append(row)
            
            if not filtered_notes:
                st.warning("ℹ️ কোনো নোট পাওয়া যায়নি।")
            else:
                for row in filtered_notes:
                    n_id, n_title, n_content, n_subject, n_type = row
                    with st.expander(f"📖 [{n_subject}] {n_title} - স্ট্যাটাস: {n_type}"):
                        st.markdown(f"**বিষয়:** {n_subject} | **স্ট্যাটাস:** {n_type}")
                        st.markdown("---")
                        st.markdown(n_content, unsafe_allow_html=True)
                        col_e, col_d = st.columns(2, gap="small")
                        with col_e:
                            if st.button("✏️ এডিট করো", key=f"edit_note_{n_id}", use_container_width=True):
                                st.session_state["current_title"] = n_title
                                st.session_state["current_content"] = n_content
                                st.session_state["current_subject"] = n_subject
                                st.session_state["edit_id"] = n_id
                                st.success("✅ নোটটি এডিটর বক্সে লোড করা হয়েছে।")
                                st.rerun()
                        with col_d:
                            if st.button("🗑️ ডিলিট করো", key=f"del_note_{n_id}", use_container_width=True):
                                conn = sqlite3.connect("subject_notes.db")
                                cursor = conn.cursor()
                                cursor.execute("DELETE FROM notes WHERE id = ?", (n_id,))
                                conn.commit()
                                conn.close()
                                if st.session_state["edit_id"] == n_id:
                                    st.session_state["edit_id"] = None
                                    st.session_state["current_title"] = ""
                                    st.session_state["current_content"] = ""
                                st.success("✅ নোটটি সফলভাবে ডিলিট করা হয়েছে!")
                                st.rerun()

    with col_sidebar:
        st.info("তোমার পছন্দের বই পড়ে নোট করতে 'Books & Reading' অপশনে যাও...")
        st.markdown("---")
        st.markdown("<h3 style='color: #0f172a; font-size: 18px;'>🤖 এআই অ্যাসিস্ট্যান্ট</h3>", unsafe_allow_html=True)
        st.markdown("কোন প্রশ্ন থাকলে নিচে Gemini-কে জিজ্ঞেস করো এবং তোমার নোট খাতায় নোট করে রাখো।")
        st.markdown(
            '<a href="https://gemini.google.com" target="_blank"><button style="width: 100%; background-color: #0284c7; color: white; padding: 12px; border: none; border-radius: 12px; font-weight: bold; cursor: pointer; margin-bottom: 10px; box-shadow: 0 4px 12px rgba(2,132,199,0.3);">🚀 Open Gemini Chatbot</button></a>',
            unsafe_allow_html=True
        )
        # সাইডবারের ফাঁকা জায়গা পূরণের জন্য অতিরিক্ত স্টাডি টিপস কার্ড
        st.markdown("---")
        st.markdown("""
        <div style="background-color: #f8fafc; border: 1.5px solid #bae6fd; padding: 16px; border-radius: 16px; text-align: center; box-shadow: 0 4px 12px rgba(14,165,233,0.05);">
            <div style="font-size: 26px; margin-bottom: 6px;">💡</div>
            <p style="color: #0284c7; font-weight: 700; margin-bottom: 6px; font-size: 15px;">পড়ার টিপস</p>
            <p style="color: #64748b; font-size: 13px; margin: 0; font-weight: 500;">নিয়মিত নোট তৈরি করলে পরীক্ষার প্রস্তুতি আরও সহজ হয়!</p>
        </div>
        """, unsafe_allow_html=True)
        
def render_progress():
    st.markdown("## 📈 Academic Progress & Analytics")
    st.markdown("Track your overall academic growth, completion rates, and study consistency.")
    col1, col2, col3 = st.columns(3)
    col1.metric("Overall Progress", "72%", "+4% this week")
    col2.metric("Chapters Completed", "34 / 80", "On Track")
    col3.metric("Study Streak", "12 Days", " 🔥 Hot streak")
    
    st.markdown("### Subject-wise Mastery")
    st.progress(0.80, text="Bangla - 80%")
    st.progress(0.65, text="English - 65%")
    st.progress(0.75, text="Mathematics - 75%")
    st.progress(0.70, text="Science - 70%")


import sqlite3
import streamlit as st
from datetime import datetime, date

# --- ডেটাবেজ ইনিশিয়ালাইজেশন ---
def init_school_syllabus_db():
    conn = sqlite3.connect("smart_school_syllabus.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS school_syllabus (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            student_class TEXT,
            subject_name TEXT,
            category TEXT,
            topic_name TEXT,
            exam_term TEXT
        )
    """)
    conn.commit()
    conn.close()

import sqlite3
import streamlit as st

def init_school_syllabus_db():
    conn = sqlite3.connect("smart_school_syllabus.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS school_syllabus (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            student_class TEXT,
            subject_name TEXT,
            category TEXT,
            topic_name TEXT,
            exam_term TEXT
        )
    """)
    conn.commit()
    conn.close()

init_school_syllabus_db()

import streamlit as st
import sqlite3
import json
import math
from collections import Counter

# --- পৃষ্ঠা কনফিগারেশন ---
st.set_page_config(page_title="Smart Study Planner & Syllabus Tracker", layout="wide")

def render_study_planner():
    st.markdown("""
    <div style="background: linear-gradient(135deg, #1E3A8A 0%, #3882F6 100%);
    padding: 25px; border-radius: 12px; color: white; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <h2 style="margin: 0; color: white;">Your Virtual Reading Room-Smart Study & Syllabus</h2>
    <p style="margin: 5px 0 0 0; font-size: 16px; opacity: 0.9;"> A Smart Hub for Your Syllabus, Study Plan, Routine & Archives</p>
    </div>
    """, unsafe_allow_html=True)

    # মূল ৪টি ট্যাব
    tab1, tab2, tab3, tab4 = st.tabs(["📚 Syllabus", "📅 Study Plan", "🎯 Routine", "📦 Archive"])

    # --- ক্লাস সেভেনের সমন্বিত সিলেবাস ডিকশনারি ---
    unified_syllabus_dictionary = {
        "বাংলা প্রথম পত্র": {
            "গদ্যাংশ": [
                "দেশ বাংলাদেশ", "কাবুলিওয়ালা", "লখার একুশে", "মরু-ভাস্কর", "শব্দ থেকে কবিতা",
                "পাখি", "পিতৃ পুরুষের গল্প", "ছবির রং", "সেই ছেলেটি", "বহু জাতিসত্তার দেশ"
            ],
            "কবিতাংশ": [
                "নতুন দেশ", "কুলি-মজুর", "সাম্য", "মেলা", "এই অক্ষরে", "সিঁথি",
                "কোকিল", "কিং লিয়ার", "যুদ্ধক্ষেত্রে পিতাপুত্র", "নাটিকা: জাগো সুন্দর"
            ],
            "আনন্দপাঠ": [
                "তোতা কাহিনী", "জিদ", "খুদে গোয়েন্দার অভিযান", "দীক্ষা", "পদ্য লেখার জোড়ে"
            ]
        },
        "বাংলা দ্বিতীয় পত্র": {
            "ব্যাকরণ ও নির্মিতি": [
                "ভাষা, ব্যাকরণ, ধ্বনি ও বর্ণ, সন্ধি, শব্দ ও পদ, কারক ও বিভক্তি, সম্বন্ধ ও সম্বোধন পদ, শব্দরূপ, বিশেষণের 'তর' ও 'তম'",
                "শব্দ গঠন (উপসর্গ ও প্রত্যয় যোগে), বাক্য, বিরামচিহ্ন, বানান",
                "একই শব্দের বিভিন্ন অর্থে প্রয়োগ (অঙ্ক থেকে চোখ এবং 'ছাড়া' থেকে 'হাত')",
                "বিপরীত শব্দ ('অগ্র' থেকে 'জোয়ার' এবং 'তিরস্কার' থেকে 'হলন্ত')",
                "সমোচ্চারিত ভিন্নার্থক শব্দ ('অংশ' থেকে 'গাঁ' এবং 'জ্যেষ্ঠ' থেকে 'সাক্ষর')",
                "এক কথায় প্রকাশ ('প্রত্যক্ষ' থেকে 'ভূতপূর্ব' এবং 'দেদীপ্যমান' থেকে 'জিঘাংসা')",
                "বাগধারা ('অরণ্যে রোদন' থেকে 'টনক নড়া' এবং 'ঠোঁট কাটা' থেকে 'হাতের পাঁচ')"
            ],
            "নির্মিতি অংশ": [
                "অনুবাদ (বাংলা ব্যাকরণ ও নির্মিতি বইয়ের ১১১ পৃষ্ঠা)",
                "সারাংশ ও সারমর্ম অনুশীলন",
                "ভাব-সম্প্রসারণ (পিতা মাতা গুরুজনে দেবতুল্য, নানান দেশের নানান ভাষা, পুষ্প আপনার জন্য ফোটে না, ইটের পর ইট মধ্যে মানুষ কীট ইত্যাদি)",
                "পত্র / দরখাস্ত (ছাত্রাবাস জীবনের অভিজ্ঞতা জানিয়ে মাকে পত্র, জীবনের লক্ষ্য জানিয়ে বড় ভাইকে চিঠি, অনুপস্থিতি ছুটি ও জরিমানা মওফের আবেদন, পাঠাগার স্থাপনের জন্য আবেদন, লোকজ উৎসবের বর্ণনা দিয়ে প্রবাসী বন্ধুকে পত্র ইত্যাদি)",
                "প্রবন্ধ রচনা (আমাদের জাতীয় পতাকা, দর্শনীয় স্থান, জাতীয় কবি কাজী নজরুল ইসলাম, আমার ভালো-লাগা বই, আমার চারপাশের প্রকৃতি, বাংলাদেশের নদনদী, একটি দিনের দিনলিপি, শহিদ মিনার, টেলিভিশন, শৃঙ্খলাবোধ, সুন্দরবন)"
            ]
        },
        "ইংরেজি প্রথম পত্র (English 1st Paper)": {
            "Part A: Reading": [
                "Unit: 1, 2, 3, 4, 5",
                "Unit: 6, 7, 8, 9"
            ],
            "Completing Stories": [
                "An Ant and a Dove", "The Clever Astrologer", "A Little Learning is a Dangerous Thing",
                "Money Cannot Bring Happiness", "None is Contented in the World", "Solomon's Wisdom",
                "Misers are Never Happy", "An Ant and a Grasshopper"
            ],
            "Paragraphs": [
                "Your Daily Routine", "A School Library", "Early Rising", "A Tea Stall",
                "Patriotism", "Traffic Jam", "A Winter Morning", "Your Best Friend"
            ],
            "Dialogues": [
                "Punctuality (teacher and student)", "Importance of learning English (two friends)",
                "Dream (student and TV reporter)", "First day at school (two friends)",
                "Prize-giving ceremony", "Merits and demerits of using Internet",
                "Likes and dislikes regarding sports", "Patient and doctor"
            ]
        },
        "ইংরেজি দ্বিতীয় পত্র (English 2nd Paper)": {
            "Grammar Part": [
                "Gap filling (Person, Number, Sentence, Parts of Speech, Articles, Conjugation of Verbs, Tense)",
                "Substitution Table",
                "Changing Sentences (Assertive, Interrogative, Imperative, Optative, Exclamatory, Voice)",
                "Right forms of verbs",
                "Narration (Individual sentences, Imperative, Optative, Exclamatory)",
                "Punctuation and Capitalization"
            ],
            "Writing Part (Letters & E-mails)": [
                "Leave in advance", "Common room facilities", "School hostel seat",
                "Increase library facilities", "Open a bank account", "Destroy Aedes mosquitoes",
                "Native village letter", "Condoling on father's death", "Progress in studies letter",
                "Inviting friend to visit Bangladesh", "Consoling for failure in exam", "Describing book fair visit"
            ],
            "Short Composition": [
                "Your Village or Town", "Tree Plantation", "A Village Doctor",
                "The Life of a Farmer", "Our National Flag", "My Hobby",
                "Deforestation", "A Street Hawker"
            ]
        },
        "গণিত": {
            "পাটিগণিত": [
                "প্রথম অধ্যায়: মূলদ ও অমূলদ সংখ্যা (অনুশীলনী: ১.১, ১.২)",
                "দ্বিতীয় অধ্যায়: সমানুপাত ও লাভ ক্ষতি (অনুশীলনী: ২.১, ২.২, ২.৩)",
                "তৃতীয় অধ্যায়: পরিমাপ (অনুশীলনী ৩)"
            ],
            "বীজগণিত": [
                "চতুর্থ অধ্যায়: বীজগণিতীয় রাশির গুণ ও ভাগ (অনুশীলনী ৪.১, ৪.২, ৪.৩)",
                "পঞ্চম অধ্যায়: বীজগণিতীয় সূত্রাবলী ও প্রয়োগ (অনুশীলনী ৫.১, ৫.২, ৫.৩, ৫.৪)",
                "ষষ্ঠ অধ্যায়: বীজগণিতীয় ভগ্নাংশ (অনুশীলনী ৬.১, ৬.২)",
                "সপ্তম অধ্যায়: সরল সমীকরণ (অনুশীলনী ৭.১, ৭.২, ৭.৩)"
            ],
            "জ্যামিতি": [
                "অষ্টম অধ্যায়: সমান্তরাল সরলরেখা",
                "নবম অধ্যায়: ত্রিভুজ, উপপাদ্য (১, ২, ৩, ৪, ৫), সম্পাদ্য (১, ২, ৩, ৪, ৫, ৬), অনুশীলনী ৯.১, ৯.২, ৯.৩ (১-১৩)",
                "দশম অধ্যায়: সর্বসমতা ও সদৃশতা, অনুশীলনী (১০.১, ১০.২, ১০.৩)"
            ],
            "পরিসংখ্যান": [
                "একাদশ অধ্যায়: তথ্য ও উপাত্ত (অনুশীলনী ১১ সম্পূর্ণ)"
            ]
        },
        "বিজ্ঞান": {
            "অধ্যায়সমূহ": [
                "প্রথম অধ্যায়: নিম্নশ্রেণির জীব", "দ্বিতীয় অধ্যায়: উদ্ভিদ ও প্রাণীর কোষীয় গঠন",
                "তৃতীয় অধ্যায়: উদ্ভিদের বাহ্যিক বৈশিষ্ট্য", "চতুর্থ অধ্যায়: শ্বসন",
                "পঞ্চম অধ্যায়: পরিপাকতন্ত্র এবং রক্ত সংবহনতন্ত্র", "ষষ্ঠ অধ্যায়: পদার্থের গঠন",
                "সপ্তম অধ্যায়: শক্তির ব্যবহার", "অষ্টম অধ্যায়: শব্দের কথা",
                "নবম অধ্যায়: তাপ ও তাপমাত্রা", "দশম অধ্যায়: বিদ্যুৎ ও চুম্বকের ঘটনা",
                "একাদশ অধ্যায়: পারিপার্শ্বিক পরিবর্তন ও বিভিন্ন ঘটনা", "দ্বাদশ অধ্যায়: সৌরজগৎ ও আমাদের পৃথিবী",
                "ত্রয়োদশ অধ্যায়: প্রাকৃতিক পরিবেশ ও দূষণ", "চতুর্দশ অধ্যায়: জলবায়ু পরিবর্তন"
            ]
        },
        "বাংলাদেশ ও বিশ্বপরিচয়": {
            "অধ্যায়সমূহ": [
                "প্রথম অধ্যায়: বাংলাদেশের মুক্তিসংগ্রাম ও গণআন্দোলন", "দ্বিতীয় অধ্যায়: বাংলাদেশের সংস্কৃতি ও সাংস্কৃতিক বৈচিত্র্য",
                "তৃতীয় অধ্যায়: পরিবারে শিশুর বেড়ে ওঠা", "চতুর্থ অধ্যায়: বাংলাদেশের অর্থনীতি",
                "পঞ্চম অধ্যায়: বাংলাদেশ ও বাংলাদেশের নাগরিক", "ষষ্ঠ অধ্যায়: বাংলাদেশের জলবায়ু",
                "সপ্তম অধ্যায়: বাংলাদেশের জনসংখ্যা পরিচিতি", "অষ্টম অধ্যায়: বাংলাদেশের সামাজিক সমস্যা",
                "নবম অধ্যায়: বাংলাদেশে প্রবীণ ব্যক্তি ও নারীর অধিকার", "দশম অধ্যায়: এশিয়ার কয়েকটি দেশ",
                "একাদশ অধ্যায়: বাংলাদেশ ও আন্তর্জাতিক সহযোগিতা"
            ]
        },
        "ইসলাম শিক্ষা": {
            "অধ্যায়সমূহ": [
                "প্রথম অধ্যায়: আকাইদ", "দ্বিতীয় অধ্যায়: ইবাদত",
                "তৃতীয় অধ্যায়: কুরআন ও হাদিস শিক্ষা", "চতুর্থ অধ্যায়: আখলাক", "পঞ্চম অধ্যায়: আদর্শ জীবনচরিত"
            ]
        },
        "তথ্য ও যোগাযোগ প্রযুক্তি (আইসিটি)": {
            "অধ্যায়সমূহ": [
                "প্রথম অধ্যায়: প্রাত্যহিক জীবনে তথ্য ও যোগাযোগ প্রযুক্তি", "দ্বিতীয় অধ্যায়: কম্পিউটার সংশ্লিষ্ট যন্ত্রপাতি",
                "তৃতীয় অধ্যায়: নিরাপদ ও নৈতিক ব্যবহার", "চতুর্থ অধ্যায়: ওয়ার্ড প্রসেসিং",
                "পঞ্চম অধ্যায়: শিক্ষায় ইন্টারনেটের ব্যবহার"
            ]
        }
    }

    # ডাটাবেস টেবিল তৈরি
    conn = sqlite3.connect("smart_school_syllabus.db")
    cursor = conn.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS school_syllabus_multi (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        student_class TEXT,
        exam_term TEXT,
        syllabus_json TEXT,
        is_default INT DEFAULT 0
    )
    """)
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS saved_study_plans (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        plan_name TEXT,
        syllabus_label TEXT,
        target_days INTEGER,
        daily_hours REAL,
        plan_json TEXT
    )
    """)
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS universal_archives (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        archive_type TEXT,
        title TEXT,
        details_json TEXT,
        archive_date TEXT
    )
    """)
    # নতুন নিজেকে যাচাইকরণ সংক্রান্ত টেস্ট হিস্ট্রি টেবিল
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS test_history_archive (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        target_day_num INTEGER,
        subject_name TEXT,
        topic_name TEXT,
        test_type TEXT,
        score_obtained INTEGER,
        total_score INTEGER,
        percentage REAL,
        correct_count INTEGER,
        wrong_count INTEGER,
        details_json TEXT,
        test_date TEXT
    )
    """)
    conn.commit()
    conn.close()

    # --- TAB 1: SYLLABUS ---
    with tab1:
        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, student_class, exam_term, syllabus_json, is_default FROM school_syllabus_multi ORDER BY id DESC")
        all_saved_syllabi = cursor.fetchall()
        conn.close()

        if "syllabus_edit_id" not in st.session_state:
            st.session_state.syllabus_edit_id = None
        if "create_new_syllabus" not in st.session_state:
            st.session_state.create_new_syllabus = False

        if not all_saved_syllabi or st.session_state.create_new_syllabus or st.session_state.syllabus_edit_id is not None:
            editing_syllabus = None
            if st.session_state.syllabus_edit_id is not None:
                for s in all_saved_syllabi:
                    if s[0] == st.session_state.syllabus_edit_id:
                        editing_syllabus = s
                        break
            
            form_title = "Edit" if editing_syllabus else "+ Make Your Syllabus"
            st.markdown(f"### {form_title}")
            st.markdown("সিলেবাস তৈরি করতে ক্লাস, বিষয় এবং কোন পরীক্ষার জন্য সিলেবাস তৈরি করছ তা নির্বাচন করো। তারপর Save বাটনে ক্লিক করো!")

            default_class = "সপ্তম শ্রেণি (Class 7)"
            default_term = "সাপ্তাহিক কুইজ"
            parsed_saved_data = {}

            if editing_syllabus:
                default_class = editing_syllabus[1]
                default_term = editing_syllabus[2]
                parsed_saved_data = json.loads(editing_syllabus[3])

            col1, col2 = st.columns(2)
            with col1:
                student_class = st.selectbox("Select Class:", ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"], index=1, key="sel_class_multi")
                exam_terms_list = ["Weekly Quiz", "Half-Yearly", "Yearly", "Model Test", "Pre-Test", "C.T.", "Tutorial", "Others"]
                term_idx = exam_terms_list.index(default_term) if default_term in exam_terms_list else 0
                exam_term = st.selectbox("Exam Name:", exam_terms_list, index=term_idx, key="sel_term_multi")
            with col2:
                available_subjects = list(unified_syllabus_dictionary.keys())
                default_subjs = [sub for sub in parsed_saved_data.keys() if sub in available_subjects] if parsed_saved_data else []
                selected_subjects = st.multiselect("Select one/More Topic:", available_subjects, default=default_subjs, key="sel_subjects_multi")

            selected_syllabus_data = {}
            if selected_subjects:
                st.markdown("#### Select Topic:")
                for subj in selected_subjects:
                    with st.expander(f"বিষয়: {subj}", expanded=True):
                        categories_dict = unified_syllabus_dictionary.get(subj, {})
                        available_categories = list(categories_dict.keys())
                        chosen_cat_topics = {}
                        for cat in available_categories:
                            available_topics = categories_dict.get(cat, [])
                            default_topics = parsed_saved_data.get(subj, {}).get(cat, []) if parsed_saved_data else []
                            selected_topics = st.multiselect(
                                f"{subj} -> {cat} (বিভাগ):",
                                available_topics,
                                default=[t for t in default_topics if t in available_topics],
                                key=f"topics_{subj}_{cat}"
                            )
                            if selected_topics:
                                chosen_cat_topics[cat] = selected_topics
                        if chosen_cat_topics:
                            selected_syllabus_data[subj] = chosen_cat_topics

            col_btn_s1, col_btn_s2 = st.columns(2)
            with col_btn_s1:
                btn_text = "Update" if editing_syllabus else "Save"
                if st.button(btn_text, use_container_width=True, type="primary"):
                    if selected_syllabus_data:
                        syllabus_str = json.dumps(selected_syllabus_data, ensure_ascii=False)
                        conn = sqlite3.connect("smart_school_syllabus.db")
                        cursor = conn.cursor()
                        if editing_syllabus:
                            cursor.execute("""
                                UPDATE school_syllabus_multi
                                SET student_class=?, exam_term=?, syllabus_json=?
                                WHERE id=?
                            """, (student_class, exam_term, syllabus_str, editing_syllabus[0]))
                        else:
                            cursor.execute("""
                                INSERT INTO school_syllabus_multi (student_class, exam_term, syllabus_json, is_default)
                                VALUES (?, ?, ?, 0)
                            """, (student_class, exam_term, syllabus_str))
                        conn.commit()
                        conn.close()
                        st.session_state.create_new_syllabus = False
                        st.session_state.syllabus_edit_id = None
                        st.success("সিলেবাস সফলভাবে সেভ করা হয়েছে!")
                        st.rerun()
                    else:
                        st.error("দয়া করে অন্তত একটি বিষয় এবং তার টপিক সিলেক্ট করো.")
            with col_btn_s2:
                if editing_syllabus or st.session_state.create_new_syllabus:
                    if st.button("X Cancel", use_container_width=True):
                        st.session_state.create_new_syllabus = False
                        st.session_state.syllabus_edit_id = None
                        st.rerun()

        if all_saved_syllabi and not st.session_state.create_new_syllabus and st.session_state.syllabus_edit_id is None:
            syllabus_options = {}
            for s in all_saved_syllabi:
                s_id, s_class, s_term, s_json, s_def = s[0], s[1], s[2], s[3], s[4]
                def_tag = " * [মূল/ডিফল্ট সিলেবাস]" if s_def == 1 else ""
                label_str = f"{s_term} ({s_class}) ID: {s_id}{def_tag}"
                syllabus_options[label_str] = s

            selected_s_label = st.selectbox(" Select Your Syllabus:", list(syllabus_options.keys()), key="select_saved_syllabus_dropdown")
            selected_s_record = syllabus_options[selected_s_label]
            s_id, s_class, s_term, s_json, s_def = selected_s_record

            st.markdown("---")
            
            # সিলেবাস সামারি পপআপ বাটন
            if st.button("📊 Samary of Syllabus", key=f"btn_syl_summary_{s_id}", type="secondary"):
                @st.dialog("📊 Samary of Syllabus")
                def show_syllabus_summary_modal():
                    st.markdown(f"**সিলেবাস বিবরণ:** {s_term} ({s_class})")
                    parsed_modal_syl = json.loads(s_json)
                    modal_rows = []
                    idx_cnt = 1
                    for subj, cats in parsed_modal_syl.items():
                        for cat, topics in cats.items():
                            modal_rows.append(f"<tr><td style='padding:8px; border:1px solid #cbd5e1;'>{idx_cnt}</td><td style='padding:8px; border:1px solid #cbd5e1; font-weight:bold;'>{subj}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{cat}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{', '.join(topics)}</td></tr>")
                            idx_cnt += 1
                    modal_table_html = f"<table style='width:100%; border-collapse:collapse; font-size:14px;'><thead><tr style='background:#1E3A8A; color:white;'><th style='padding:8px; border:1px solid #cbd5e1;'>ক্রমিক</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিষয়</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিভাগ</th><th style='padding:8px; border:1px solid #cbd5e1;'>টপিকসমূহ</th></tr></thead><tbody>{''.join(modal_rows)}</tbody></table>"
                    st.markdown(modal_table_html, unsafe_allow_html=True)
                    st.markdown("<br>", unsafe_allow_html=True)
                    if st.button("বন্ধ করুন", key="close_syl_summary_modal"):
                        st.rerun()
                show_syllabus_summary_modal()

            col_sel_title, col_sel_action = st.columns([3, 2])
            with col_sel_title:
                st.markdown(f"**Current Syllabus:** {s_term} ({s_class})")
                if s_def == 1:
                    st.success("✔ এটি বর্তমানে আপনার **'মূল সিলেবাস'** হিসেবে সেট করা আছে।")
                else:
                    st.info("ℹ এটি সাধারণ সিলেবাস হিসেবে আছে। মূল সিলেবাস হিসেবে সেট করতে পাশের বাটনে টিক দাও।")
            with col_sel_action:
                is_currently_default = (s_def == 1)
                make_default_btn = st.checkbox("এটিকে 'মূল সিলেবাস' হিসেবে টিক দিন", value=is_currently_default, key=f"main_default_check_{s_id}")
                if make_default_btn != is_currently_default:
                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    if make_default_btn:
                        cursor.execute("UPDATE school_syllabus_multi SET is_default = 0")
                        cursor.execute("UPDATE school_syllabus_multi SET is_default = 1 WHERE id = ?", (s_id,))
                    else:
                        cursor.execute("UPDATE school_syllabus_multi SET is_default = 0 WHERE id = ?", (s_id,))
                    conn.commit()
                    conn.close()
                    st.success("মূল সিলেবাস সফলভাবে আপডেট করা হয়েছে!")
                    st.rerun()

            st.markdown("---")
            parsed_syllabus = json.loads(s_json)
            st.markdown(f"#### Now, Your Syllabus is Ready! ({s_class} - {s_term})")
            
            table_rows = []
            for subj, cats in parsed_syllabus.items():
                first_row = True
                cat_count = len(cats)
                for cat, topics in cats.items():
                    topics_str = ", ".join(topics)
                    if first_row:
                        row_html = f'<tr><td rowspan="{cat_count}" style="padding: 10px; border: 1px solid #cbd5e1; font-weight: bold; vertical-align: top;">{subj}</td><td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">{cat}</td><td style="padding: 10px; border: 1px solid #cbd5e1;">{topics_str}</td></tr>'
                        first_row = False
                    else:
                        row_html = f'<tr><td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">{cat}</td><td style="padding: 10px; border: 1px solid #cbd5e1;">{topics_str}</td></tr>'
                    table_rows.append(row_html)
            
            all_rows = "".join(table_rows)
            table_html = f'<table style="width: 100%; border-collapse: collapse; margin-top: 10px; font-family: sans-serif;"><thead><tr style="background-color: #1E3A8A; color: white; text-align: left;"><th style="padding: 12px; border: 1px solid #cbd5e1;">বিষয় (Subject)</th><th style="padding: 12px; border: 1px solid #cbd5e1;">বিভাগ (Category)</th><th style="padding: 12px; border: 1px solid #cbd5e1;">নির্বাচিত অধ্যায়সমূহ (Selected Topics)</th></tr></thead><tbody>{all_rows}</tbody></table>'
            st.markdown(table_html, unsafe_allow_html=True)
            st.markdown("<br>", unsafe_allow_html=True)

            col_btn1, col_btn2, col_btn3, col_btn4 = st.columns(4)
            with col_btn1:
                if st.button("Edit", use_container_width=True, key=f"edit_s_btn_{s_id}"):
                    st.session_state.syllabus_edit_id = s_id
                    st.rerun()
            with col_btn2:
                if st.button("Delete", use_container_width=True, key=f"del_s_btn_{s_id}"):
                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("DELETE FROM school_syllabus_multi WHERE id=?", (s_id,))
                    conn.commit()
                    conn.close()
                    st.success("সিলেবাস সফলভাবে বাতিল ও মুছে ফেলা হয়েছে!")
                    st.rerun()
            with col_btn3:
                if st.button("📦 সিলেবাসটি আর্কাইভ করুন", use_container_width=True, key=f"arc_s_btn_{s_id}"):
                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("""
                        INSERT INTO universal_archives (archive_type, title, details_json, archive_date)
                        VALUES ('Syllabus', ?, ?, datetime('now', 'localtime'))
                    """, (f"{s_term} ({s_class})", s_json))
                    conn.commit()
                    conn.close()
                    st.success("সিলেবাসটি সফলভাবে আর্কাইভে সেভ করা হয়েছে!")
            with col_btn4:
                if st.button("+ Create New Syllabus", use_container_width=True, type="primary", key="create_new_s_btn"):
                    st.session_state.create_new_syllabus = True
                    st.session_state.syllabus_edit_id = None
                    st.rerun()

    # --- TAB 2: ROUTINE ---
    with tab3:
        st.markdown("নিচ থেকে সিলেবাস নির্বাচন করো। এরপর প্রতিদিনের জন্য বিষয়, সময় স্লট, অধ্যায় এবং টার্গেট সেট করে রুটিন সাজাও!")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, student_class, exam_term, syllabus_json, is_default FROM school_syllabus_multi ORDER BY id DESC")
        routine_syllabi = cursor.fetchall()
        conn.close()

        if routine_syllabi:
            default_index = 0
            syllabus_choices = {}
            for idx, s in enumerate(routine_syllabi):
                s_id, s_class, s_term, s_json, s_def = s[0], s[1], s[2], s[3], s[4]
                label_str = f"{s_term} ({s_class}) [ID: {s_id}]"
                if s_def == 1:
                    label_str += " * (মূল সিলেবাস)"
                    default_index = idx
                syllabus_choices[label_str] = s_json

            selected_routine_s_label = st.selectbox("সিলেবাস নির্বাচন করো:", list(syllabus_choices.keys()), index=default_index, key="routine_syllabus_dropdown")
            routine_selected_syl_dict = json.loads(syllabus_choices[selected_routine_s_label])
            available_subjects_in_syllabus = list(routine_selected_syl_dict.keys())

            if available_subjects_in_syllabus:
                st.markdown("---")
                st.markdown("#### সপ্তাহের ৭ দিনের পড়ার পরিকল্পনা ও টার্গেট সেটিং (শনিবার - শুক্রবার):")
                days_of_week = ["শনিবার", "রবিবার", "সোমবার", "মঙ্গলবার", "বুধবার", "বৃহস্পতিবার", "শুক্রবার"]
                weekly_custom_routine = {}

                for day in days_of_week:
                    with st.expander(f"দিন: {day}", expanded=(day == "শনিবার")):
                        num_subjs_for_day = st.number_input(
                            f"{day} কয়টি বিষয় পড়তে চাও?",
                            min_value=1, max_value=25, value=min(2, len(available_subjects_in_syllabus)), step=1,
                            key=f"num_subs_{day}"
                        )
                        day_entries = []
                        for i in range(num_subjs_for_day):
                            st.markdown(f"**বিষয় {i+1} বিস্তারিত ও টার্গেট সেটিং:**")
                            c_s, c_t, c_c = st.columns([2, 2, 2])
                            with c_s:
                                chosen_sub = st.selectbox("বিষয় নির্বাচন:", available_subjects_in_syllabus, key=f"sub_{day}_{i}")
                            with c_t:
                                time_str = st.text_input("সময় বা স্লট:", value="06:30 AM - 07:30 AM (সকাল)", key=f"time_{day}_{i}")
                            with c_c:
                                chapter_str = st.text_input("টপিক বা অধ্যায়:", value="অধ্যায় ১.১", key=f"chap_{day}_{i}")

                            col_p1, col_p2, col_p3 = st.columns(3)
                            with col_p1:
                                total_pages = st.number_input(f"মোট পৃষ্ঠা:", min_value=0, max_value=500, value=10, key=f"pages_{day}_{i}")
                            with col_p2:
                                total_mcq = st.number_input(f"মোট MCQ:", min_value=0, max_value=1000, value=30, key=f"mcq_{day}_{i}")
                            with col_p3:
                                total_cq = st.number_input(f"মোট CQ (সৃজনশীল):", min_value=0, max_value=100, value=5, key=f"cq_{day}_{i}")

                            st.caption("এই অধ্যায়ের কি কি সম্পন্ন করা বা লক্ষ্য রয়েছে তা টিক দিন:")
                            col_ck1, col_ck2, col_ck3, col_ck4 = st.columns(4)
                            with col_ck1:
                                is_mainbook = st.checkbox("মূল বই রিডিং", value=True, key=f"mb_{day}_{i}")
                            with col_ck2:
                                is_guide = st.checkbox("গাইড/নোট", value=False, key=f"gb_{day}_{i}")
                            with col_ck3:
                                is_mcq_done = st.checkbox("MCQ প্র্যাকটিস", value=True, key=f"md_{day}_{i}")
                            with col_ck4:
                                is_cq_done = st.checkbox("CQ চর্চা", value=False, key=f"cd_{day}_{i}")

                            day_entries.append({
                                "subject": chosen_sub,
                                "time": time_str,
                                "chapter": chapter_str,
                                "pages": total_pages,
                                "mcq": total_mcq,
                                "cq": total_cq,
                                "mainbook": is_mainbook,
                                "guide": is_guide,
                                "mcq_done": is_mcq_done,
                                "cq_done": is_cq_done
                            })
                            st.markdown("---")
                        weekly_custom_routine[day] = day_entries

                st.markdown("<br>", unsafe_allow_html=True)
                if st.button("আপনার রুটিন তৈরি করুন (Generate Your Routine)", use_container_width=True, type="primary"):
                    st.session_state["generated_custom_weekly_routine"] = weekly_custom_routine
                    st.session_state["generated_routine_syllabus_name"] = selected_routine_s_label
                    st.success("সফলভাবে তোমার কাস্টম প্রাত্যহিক রুটিন তৈরি করা হয়েছে!")

            if "generated_custom_weekly_routine" in st.session_state:
                st.markdown("<br>", unsafe_allow_html=True)
                
                # রুটিন সামারি পপআপ বাটন
                if st.button("📊 পরীক্ষার রুটিন ও সামারি টেবিল (এক নজরে)", key="btn_routine_summary_main", type="secondary"):
                    @st.dialog("📊 পরীক্ষার রুটিন ও সামারি টেবিল (এক নজরে)")
                    def show_routine_summary_modal():
                        st.markdown(f"**ভিত্তি সিলেবাস:** {st.session_state.get('generated_routine_syllabus_name', '')}")
                        r_modal_rows = []
                        r_idx = 1
                        for d_name, entries in st.session_state["generated_custom_weekly_routine"].items():
                            for ent in entries:
                                r_modal_rows.append(f"<tr><td style='padding:8px; border:1px solid #cbd5e1;'>{r_idx}</td><td style='padding:8px; border:1px solid #cbd5e1; font-weight:bold;'>{d_name}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{ent['subject']}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{ent['time']}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{ent['chapter']}</td></tr>")
                                r_idx += 1
                        r_modal_table_html = f"<table style='width:100%; border-collapse:collapse; font-size:14px;'><thead><tr style='background:#1E3A8A; color:white;'><th style='padding:8px; border:1px solid #cbd5e1;'>ক্রমিক</th><th style='padding:8px; border:1px solid #cbd5e1;'>বার</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিষয়</th><th style='padding:8px; border:1px solid #cbd5e1;'>সময়</th><th style='padding:8px; border:1px solid #cbd5e1;'>টপিক/অধ্যায়</th></tr></thead><tbody>{''.join(r_modal_rows)}</tbody></table>"
                        st.markdown(r_modal_table_html, unsafe_allow_html=True)
                        st.markdown("<br>", unsafe_allow_html=True)
                        if st.button("বন্ধ করুন", key="close_routine_summary_modal"):
                            st.rerun()
                    show_routine_summary_modal()

                st.markdown("### আপনার চূড়ান্ত প্রাত্যহিক রুটিন ছক")
                syl_name = st.session_state.get('generated_routine_syllabus_name', 'নির্বাচিত সিলেবাস')
                st.markdown(f"""
                <div style="background-color: #E8F0FE; padding: 14px 20px; border-radius: 8px; color: #1E3A8A; font-weight: bold; margin-bottom: 20px; font-size: 15px;">
                ভিত্তি সিলেবাস: {syl_name}
                </div>
                """, unsafe_allow_html=True)

                filter_days = ["সব বার", "শনিবার", "রবিবার", "সোমবার", "মঙ্গলবার", "বুধবার", "বৃহস্পতিবার", "শুক্রবার"]
                selected_filter = st.radio("বার অনুযায়ী ফিল্টার করো:", filter_days, horizontal=True, key="routine_day_filter_radio")
                st.markdown("<br>", unsafe_allow_html=True)

                if "routine_edit_item" not in st.session_state:
                    st.session_state.routine_edit_item = None

                for day_name, entries in st.session_state["generated_custom_weekly_routine"].items():
                    if selected_filter != "সব বার" and selected_filter != day_name:
                        continue
                    for idx, entry in enumerate(entries):
                        unique_entry_key = f"{day_name}_{idx}"
                        if st.session_state.routine_edit_item == unique_entry_key:
                            with st.form(key=f"edit_form_{unique_entry_key}"):
                                st.markdown(f"**এডিট করো ({day_name} - বিষয় {idx+1}):**")
                                e_col1, e_col2, e_col3 = st.columns(3)
                                with e_col1:
                                    new_sub = st.selectbox("বিষয়:", available_subjects_in_syllabus, index=available_subjects_in_syllabus.index(entry["subject"]) if entry["subject"] in available_subjects_in_syllabus else 0)
                                with e_col2:
                                    new_time = st.text_input("সময় বা স্লট:", value=entry["time"])
                                with e_col3:
                                    new_chap = st.text_input("টপিক/অধ্যায়:", value=entry["chapter"])

                                st.markdown("**অধ্যায়ের টার্গেট সংখ্যা এডিট করো:**")
                                ep1, ep2, ep3 = st.columns(3)
                                with ep1:
                                    new_pages = st.number_input("মোট পৃষ্ঠা:", min_value=0, value=entry.get("pages", 10))
                                with ep2:
                                    new_mcq = st.number_input("মোট MCQ:", min_value=0, value=entry.get("mcq", 30))
                                with ep3:
                                    new_cq = st.number_input("মোট CQ:", min_value=0, value=entry.get("cq", 5))

                                st.markdown("**কি কি সম্পন্ন হয়েছে তা টিক দাও:**")
                                eb1, eb2, eb3, eb4 = st.columns(4)
                                with eb1:
                                    new_mb = st.checkbox("মূল বই", value=entry.get("mainbook", True))
                                with eb2:
                                    new_gb = st.checkbox("গাইড/নোট", value=entry.get("guide", False))
                                with eb3:
                                    new_md = st.checkbox("MCQ", value=entry.get("mcq_done", True))
                                with eb4:
                                    new_cd = st.checkbox("CQ", value=entry.get("cq_done", False))

                                b1, b2 = st.columns(2)
                                with b1:
                                    if st.form_submit_button("সংরক্ষণ করুন (Save)"):
                                        entry["subject"] = new_sub
                                        entry["time"] = new_time
                                        entry["chapter"] = new_chap
                                        entry["pages"] = new_pages
                                        entry["mcq"] = new_mcq
                                        entry["cq"] = new_cq
                                        entry["mainbook"] = new_mb
                                        entry["guide"] = new_gb
                                        entry["mcq_done"] = new_md
                                        entry["cq_done"] = new_cd
                                        st.session_state.routine_edit_item = None
                                        st.success("রুটিন সফলভাবে আপডেট করা হয়েছে!")
                                        st.rerun()
                                with b2:
                                    if st.form_submit_button("বাতিল (Cancel)"):
                                        st.session_state.routine_edit_item = None
                                        st.rerun()
                        else:
                            col_d, col_t, col_s, col_c, col_e, col_del = st.columns([1.2, 2.2, 2.0, 3.2, 0.5, 0.5])
                            with col_d:
                                st.markdown(f"<div style='padding-top:8px; font-weight:bold; color:#1e293b;'>{day_name}</div>", unsafe_allow_html=True)
                            with col_t:
                                st.markdown(f"<div style='padding-top:8px; color:#1e3a8a; font-weight:bold;'>{entry['time']}</div>", unsafe_allow_html=True)
                            with col_s:
                                st.markdown(f"<div style='padding-top:5px;'><span style='background-color: #DBEAFE; color: #1D4ED8; padding: 5px 12px; border-radius: 6px; font-weight:bold; font-size:13px; display:inline-block;'>{entry['subject']}</span></div>", unsafe_allow_html=True)
                            with col_c:
                                target_info = []
                                if entry.get("pages", 0) > 0:
                                    target_info.append(f"{entry['pages']}টি পৃষ্ঠা")
                                if entry.get("mcq", 0) > 0:
                                    target_info.append(f"{entry['mcq']}টি MCQ")
                                if entry.get("cq", 0) > 0:
                                    target_info.append(f"{entry['cq']}টি CQ")
                                target_str = " | ".join(target_info) if target_info else "সাধারন রিভিশন"

                                done_tags = []
                                if entry.get("mainbook"):
                                    done_tags.append("মূলবই")
                                if entry.get("guide"):
                                    done_tags.append("গাইড")
                                if entry.get("mcq_done"):
                                    done_tags.append("MCQ")
                                if entry.get("cq_done"):
                                    done_tags.append("CQ")
                                done_str = " • ".join(done_tags) if done_tags else "পড়া বাকি"

                                st.markdown(f"""
                                <div style='padding-top:2px;'>
                                    <div style='color:#1e293b; font-weight:bold; font-size:14px;'>{entry['chapter']}</div>
                                    <div style='color:#475569; font-size:12px; margin-top:2px;'>{target_str}</div>
                                    <div style='color:#059669; font-size:11px; font-weight:bold; margin-top:2px;'>{done_str}</div>
                                </div>
                                """, unsafe_allow_html=True)
                            with col_e:
                                if st.button("✏️", key=f"btn_edit_{unique_entry_key}", help="এডিট করুন"):
                                    st.session_state.routine_edit_item = unique_entry_key
                                    st.rerun()
                            with col_del:
                                if st.button("🗑️", key=f"btn_del_{unique_entry_key}", help="মুছে ফেলুন"):
                                    entries.pop(idx)
                                    st.success("এন্ট্রিটি সফলভাবে মুছে ফেলা হয়েছে!")
                                    st.rerun()
                        st.markdown("<hr style='margin: 8px 0; border: none; border-bottom: 1px solid #f1f5f9;'>", unsafe_allow_html=True)

                st.markdown("<br>", unsafe_allow_html=True)
                if st.button("📦 সম্পূর্ণ রুটিনটি আর্কাইভে সেভ করুন", type="primary"):
                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("""
                        INSERT INTO universal_archives (archive_type, title, details_json, archive_date)
                        VALUES ('Routine', ?, ?, datetime('now', 'localtime'))
                    """, (f"সাপ্তাহিক রুটিন ({syl_name})", json.dumps(st.session_state["generated_custom_weekly_routine"], ensure_ascii=False)))
                    conn.commit()
                    conn.close()
                    st.success("রুটিনটি সফলভাবে আর্কাইভে সেভ করা হয়েছে!")

                st.info("💡 টিপস: তুমি চাইলে ব্রাউজার থেকে সরাসরি প্রিন্ট (Ctrl + P) করে এই রুটিনটি প্রিন্ট আউট বা পেজে সংরক্ষণ করে রাখতে পারো।")
            else:
                st.warning("⚠ নির্বাচিত সিলেবাসে কোনো বিষয় পাওয়া যায়নি। দয়া করে প্রথমে সিলেবাস চেক করুন।")
        else:
            st.warning("⚠ কোনো সংরক্ষিত সিলেবাস পাওয়া যায়নি। দয়া করে প্রথমে 'Syllabus' ট্যাব থেকে একটি সিলেবাস তৈরি ও সেভ করুন।")

    # --- TAB 3: AUTO SMART STUDY PLAN ---
    with tab2:
        st.markdown("### Auto Study Plan & Manager")
        st.markdown("এখানে তুমি যত খুশি স্টাডি প্ল্যান তৈরি করতে পারবে, সেভ রাখতে পারবে এবং যেকোনো সময় 'Your Saved Plan' থেকে স্টাডি শুরু করতে পারবে!")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, student_class, exam_term, syllabus_json, is_default FROM school_syllabus_multi ORDER BY id DESC")
        plan_syllabi = cursor.fetchall()

        cursor.execute("SELECT id, plan_name, syllabus_label, target_days, daily_hours, plan_json FROM saved_study_plans ORDER BY id DESC")
        all_saved_plans = cursor.fetchall()
        conn.close()

        if plan_syllabi:
            p_choices = {}
            p_default_idx = 0
            for idx, s in enumerate(plan_syllabi):
                s_id, s_class, s_term, s_json, s_def = s[0], s[1], s[2], s[3], s[4]
                label_str = f"{s_term} ({s_class}) [ID: {s_id}]"
                if s_def == 1:
                    label_str += " * (মূল সিলেবাস)"
                    p_default_idx = idx
                p_choices[label_str] = s_json

            st.markdown("---")
            st.markdown("#### 📂 Your Saved Plan:")
            
            if all_saved_plans:
                plan_options = {f"{p[1]} ({p[2]} - {p[3]} দিন) [ID: {p[0]}]": p for p in all_saved_plans}
                selected_saved_plan_label = st.selectbox("যেকোন একটি বেছে নাও:", list(plan_options.keys()), key="select_saved_study_plan_dropdown")
                active_plan_record = plan_options[selected_saved_plan_label]
                
                col_pl1, col_pl2 = st.columns(2)
                with col_pl1:
                    if st.button("📌 Open Your Study Plan", use_container_width=True, type="primary"):
                        st.session_state["active_plan_id"] = active_plan_record[0]
                        st.session_state["active_plan_name"] = active_plan_record[1]
                        st.session_state["active_plan_syl_label"] = active_plan_record[2]
                        st.session_state["active_target_days"] = active_plan_record[3]
                        st.session_state["active_daily_hours"] = active_plan_record[4]
                        st.session_state["custom_day_tasks_store"] = json.loads(active_plan_record[5])
                        st.success(f"'{active_plan_record[1]}' সফলভাবে লোড করা হয়েছে!")
                        st.rerun()
                with col_pl2:
                    if st.button("🗑️ Delete This Plan", use_container_width=True):
                        conn = sqlite3.connect("smart_school_syllabus.db")
                        cursor = conn.cursor()
                        cursor.execute("DELETE FROM saved_study_plans WHERE id=?", (active_plan_record[0],))
                        conn.commit()
                        conn.close()
                        st.success("স্টাডি প্ল্যানটি সফলভাবে মুছে ফেলা হয়েছে!")
                        st.rerun()
            else:
                st.info("ℹ এখনো কোনো স্টাডি প্ল্যান সেভ করা নেই। নিচে নতুন একটি প্ল্যান তৈরি করে সেভ করো।")

            st.markdown("---")
            st.markdown("#### ➕ Make Your New Study Plan & Save:")
            
            with st.expander("To Make a New Study Plan, Click here", expanded=not bool(all_saved_plans)):
                new_plan_name = st.text_input("স্টাডি প্ল্যানের একটি নাম দাও:", value="যেমন: আগামী ১০/১১/২০২৬ তারিখের সাপ্তাহিক কুইজ পরীক্ষার/মডেল টেস্ট/বার্ষিক পরীক্ষার সিলেবাস শেষ করার প্ল্যান ইত্যাদি")
                selected_plan_s_label = st.selectbox("যে সিলেবাসটি নিয়ে প্ল্যান বানাতে চাও তা সিলেক্ট করো:", list(p_choices.keys()), key="new_plan_syllabus_dropdown")
                selected_syl_dict = json.loads(p_choices[selected_plan_s_label])

                col_t1, col_t2 = st.columns(2)
                with col_t1:
                    new_target_days = st.number_input(" মোট কত দিনে সিলেবাস শেষ করতে চাই তা উল্লেখ করো (N.B. মোট দিনের ২০% রিভিশনের জন্য রাখতে হবে):", min_value=5, max_value=365, value=25, step=1, key="new_target_days")
                with col_t2:
                    new_daily_hours = st.slider("প্রতিদিন গড়ে কত ঘণ্টা পড়তে চাও?", 1.0, 12.0, 4.0, 0.5, key="new_daily_hours")

                if st.button("💾 নতুন স্টাডি প্ল্যান তৈরি করতে এখানে ক্লিক করো ", use_container_width=True, type="primary"):
                    subj_topic_queues = {}
                    for subj, cats in selected_syl_dict.items():
                        subj_topic_queues[subj] = []
                        for cat, topics in cats.items():
                            for t in topics:
                                subj_topic_queues[subj].append({"subject": subj, "category": cat, "topic": t})

                    interleaved_topics_list = []
                    max_topic_len = max([len(v) for v in subj_topic_queues.values()]) if subj_topic_queues else 0
                    subj_list_keys = list(subj_topic_queues.keys())
                    for i in range(max_topic_len):
                        for sk in subj_list_keys:
                            if i < len(subj_topic_queues[sk]):
                                interleaved_topics_list.append(subj_topic_queues[sk][i])

                    revision_days_count = max(1, math.ceil(new_target_days * 0.20))
                    regular_study_days = max(1, new_target_days - revision_days_count)

                    initial_store = {}
                    revision_start_day = new_target_days - revision_days_count + 1
                    available_subjects_all = list(selected_syl_dict.keys())

                    total_topics = len(interleaved_topics_list)
                    topics_per_day_base = total_topics // regular_study_days
                    topics_extra = total_topics % regular_study_days

                    curr_idx = 0
                    for day_num in range(1, new_target_days + 1):
                        day_store_key = f"day_tasks_{day_num}"
                        if day_num >= revision_start_day:
                            initial_store[day_store_key] = [
                                {"subject": available_subjects_all[0] if available_subjects_all else "রিভিশন", "category": "রিভিশন বিভাগ", "topic": "পূর্ণাঙ্গ রিভিশন"}
                            ]
                        else:
                            count_for_today = topics_per_day_base + (1 if day_num <= topics_extra else 0)
                            count_for_today = max(1, count_for_today)
                            initial_store[day_store_key] = interleaved_topics_list[curr_idx: curr_idx + count_for_today]
                            curr_idx += count_for_today

                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("""
                        INSERT INTO saved_study_plans (plan_name, syllabus_label, target_days, daily_hours, plan_json)
                        VALUES (?, ?, ?, ?, ?)
                    """, (new_plan_name, selected_plan_s_label, new_target_days, new_daily_hours, json.dumps(initial_store, ensure_ascii=False)))
                    conn.commit()
                    conn.close()

                    st.session_state["active_plan_syl_label"] = selected_plan_s_label
                    st.session_state["active_target_days"] = new_target_days
                    st.session_state["active_daily_hours"] = new_daily_hours
                    st.session_state["custom_day_tasks_store"] = initial_store
                    st.success("নতুন স্টাডি প্ল্যান সফলভাবে তৈরি ও সংরক্ষণ করা হয়েছে!")
                    st.rerun()

            if "custom_day_tasks_store" in st.session_state:
                target_days = st.session_state.get("active_target_days", 25)
                daily_study_hours = st.session_state.get("active_daily_hours", 4.0)
                selected_plan_s_label = st.session_state.get("active_plan_syl_label", list(p_choices.keys())[0])
                selected_syl_dict = json.loads(p_choices.get(selected_plan_s_label, list(p_choices.values())[0]))
                available_subjects_all = list(selected_syl_dict.keys())

                revision_days_count = max(1, math.ceil(target_days * 0.20))
                revision_start_day = target_days - revision_days_count + 1

                st.markdown("<br>", unsafe_allow_html=True)
                
                # স্টাডি প্ল্যান সামারি পপআপ বাটন
                if st.button("📊 স্টাডি প্ল্যান-কে সামারি টেবিল দেখতে এখানে ক্লিক করো", key="btn_study_plan_summary_main", type="secondary"):
                    @st.dialog("📊 Summary Table of Your Study Plan")
                    def show_study_plan_summary_modal():
                        active_name = st.session_state.get('active_plan_name', 'স্মার্ট প্ল্যান')
                        st.markdown(f"**প্ল্যানের নাম:** {active_name} (মোট দিন: {target_days} | দৈনিক: {daily_study_hours} ঘণ্টা)")
                        sp_modal_rows = []
                        sp_idx = 1
                        for d_n in range(1, target_days + 1):
                            d_key = f"day_tasks_{d_n}"
                            tasks_list = st.session_state["custom_day_tasks_store"].get(d_key, [])
                            for t_item in tasks_list:
                                sp_modal_rows.append(f"<tr><td style='padding:8px; border:1px solid #cbd5e1;'>{sp_idx}</td><td style='padding:8px; border:1px solid #cbd5e1; font-weight:bold;'>দিন {d_n}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{t_item.get('subject', '')}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{t_item.get('category', '')}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{t_item.get('topic', '')}</td></tr>")
                                sp_idx += 1
                        sp_modal_table_html = f"<table style='width:100%; border-collapse:collapse; font-size:14px;'><thead><tr style='background:#1E3A8A; color:white;'><th style='padding:8px; border:1px solid #cbd5e1;'>ক্রমিক</th><th style='padding:8px; border:1px solid #cbd5e1;'>দিন</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিষয়</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিভাগ</th><th style='padding:8px; border:1px solid #cbd5e1;'>টপিক</th></tr></thead><tbody>{''.join(sp_modal_rows)}</tbody></table>"
                        st.markdown(sp_modal_table_html, unsafe_allow_html=True)
                        st.markdown("<br>", unsafe_allow_html=True)
                        if st.button("বন্ধ করুন", key="close_study_plan_summary_modal"):
                            st.rerun()
                    show_study_plan_summary_modal()

                st.markdown(f"### 🗓 Your Running Study Plan (মোট দিন: {target_days} | রিভিশনের সময় পাবে: শেষের {revision_days_count} দিন)")

                if "user_done_notes" not in st.session_state:
                    st.session_state["user_done_notes"] = {}
                if "user_pending_notes" not in st.session_state:
                    st.session_state["user_pending_notes"] = {}
                if "user_subject_status" not in st.session_state:
                    st.session_state["user_subject_status"] = {}
                if "user_obstacle_reason" not in st.session_state:
                    st.session_state["user_obstacle_reason"] = {}
                if "user_subject_hours_text" not in st.session_state:
                    st.session_state["user_subject_hours_text"] = {}
                if "revision_selected_subjects" not in st.session_state:
                    st.session_state["revision_selected_subjects"] = {}
                if "user_quiz_scores" not in st.session_state:
                    st.session_state["user_quiz_scores"] = {}

                backlog_items = []
                obstacle_reasons_list = []
                total_tasks_global = 0
                completed_score_global = 0.0

                for day_num in range(1, target_days + 1):
                    is_revision_day = day_num >= revision_start_day
                    day_store_key = f"day_tasks_{day_num}"
                    
                    if day_store_key not in st.session_state["custom_day_tasks_store"]:
                        st.session_state["custom_day_tasks_store"][day_store_key] = [{"subject": available_subjects_all[0], "category": "সাধারণ", "topic": "টপিক"}]

                    day_tasks = st.session_state["custom_day_tasks_store"][day_store_key]

                    if is_revision_day:
                        st.markdown(f"""
                        <div style="background: #FEF3C7; border: 2px solid #D97706; padding: 15px; border-radius: 10px; margin-bottom: 15px;">
                            <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #FCD34D; padding-bottom: 8px; margin-bottom: 12px;">
                                <span style="font-size: 17px; font-weight: bold; color: #92400E;">🔄 দিন {day_num} (আজকে থেকে রিভিশনের সময় শুরু...)</span>
                                <span style="background-color: #FDE68A; color: #92400E; font-weight: bold; padding: 4px 10px; border-radius: 6px; font-size: 13px;">নির্ধারিত সময়: {daily_study_hours} ঘণ্টা</span>
                            </div>
                        """, unsafe_allow_html=True)

                        rev_key = f"rev_day_{day_num}"
                        chosen_rev_subjects = st.multiselect(
                            f"দিন {day_num}-কোন কোন বিষয় রিভিশন দিতে চাও তা সিলেক্ট করো। :",
                            available_subjects_all,
                            default=st.session_state["revision_selected_subjects"].get(rev_key, [available_subjects_all[0]]),
                            key=f"rev_multiselect_{day_num}"
                        )
                        st.session_state["revision_selected_subjects"][rev_key] = chosen_rev_subjects

                        if chosen_rev_subjects:
                            day_tasks = [{"subject": sub, "category": "কি কি রিভিশন দিয়েছে তা সম্পন্ন হয়েছে বক্সে লিখে রাখো", "topic": f": {sub}"} for sub in chosen_rev_subjects]
                            st.session_state["custom_day_tasks_store"][day_store_key] = day_tasks
                    else:
                        st.markdown(f"""
                        <div style="background: #F8FAFC; border: 2px solid #3B82F6; padding: 15px; border-radius: 10px; margin-bottom: 15px;">
                            <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #CBD5E1; padding-bottom: 8px; margin-bottom: 12px;">
                                <span style="font-size: 16px; font-weight: bold; color: #1E3A8A;">📖 দিন {day_num} (টার্গেট ডে)</span>
                                <span style="background-color: #DBEAFE; color: #1E4ED8; font-weight: bold; padding: 4px 10px; border-radius: 6px; font-size: 13px;">নির্ধারিত সময়: {daily_study_hours} ঘণ্টা</span>
                            </div>
                        """, unsafe_allow_html=True)

                    day_score = 0.0
                    day_task_count = len(day_tasks)

                    for t_idx, task in enumerate(day_tasks):
                        s_name = task["subject"]
                        cat_name = task["category"]
                        top_name = task["topic"]
                        task_key = f"day_{day_num}_task_{t_idx}_{s_name}"

                        # বিষয় ও টপিকের পাশে "নিজেকে যাচাই করো" মডেল টেস্ট বাটন
                        col_t_title, col_t_test = st.columns([3, 1])
                        with col_t_title:
                            st.markdown(f"**📌 {s_name}** - *[{cat_name}] {top_name}*")
                        with col_t_test:
                            if st.button("🧪 নিজেকে যাচাই করো", key=f"btn_test_{task_key}", type="secondary"):
                                @st.dialog(f"🧪 নিজেকে যাচাই করো: {s_name} ({top_name})")
                                def show_quiz_dialog():
                                    st.markdown(f"**বিষয়:** {s_name} | **অধ্যায়:** {top_name}")
                                    test_type = st.selectbox(
                                        "কোন ধরনের প্রশ্নের মাধ্যমে নিজেকে যাচাই করতে চাও?",
                                        ["বহুনির্বাচনী প্রশ্ন (MCQ)", "সৃজনশীল প্রশ্ন (Creative / CQ)", "সংক্ষিপ্ত প্রশ্ন (Short Questions)", "মডেল কোশ্চেন টেস্ট (Model Test)"],
                                        key=f"test_type_select_{task_key}"
                                    )
                                    st.markdown("---")
                                    
                                    # ডাটাবেসে টেস্ট হিস্ট্রি সেভ করার ফাংশন
                                    def save_test_result_to_db(t_type, score, total, p_cnt, w_cnt, details_dict):
                                        conn = sqlite3.connect("smart_school_syllabus.db")
                                        cursor = conn.cursor()
                                        cursor.execute("""
                                            INSERT INTO test_history_archive 
                                            (target_day_num, subject_name, topic_name, test_type, score_obtained, total_score, percentage, correct_count, wrong_count, details_json, test_date)
                                            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, datetime('now', 'localtime'))
                                        """, (day_num, s_name, top_name, t_type, score, total, (score/total)*100, p_cnt, w_cnt, json.dumps(details_dict, ensure_ascii=False)))
                                        conn.commit()
                                        conn.close()

                                    if test_type == "বহুনির্বাচনী প্রশ্ন (MCQ)":
                                        st.markdown("##### 📝 MCQ কুইজ পরীক্ষা:")
                                        q1 = st.radio("১. প্রদত্ত অধ্যায়ের মূল ভিত্তি বা ধারণা কোনটি?", ["কমিউনিকেশন ও লজিক", "গাণিতিক সূত্রাবলি", "মৌলিক ধারণা ও অনুচ্ছেদ", "ব্যবহারিক প্রয়োগ"], key=f"q1_{task_key}")
                                        q2 = st.radio("২. নিয়মিত চর্চার মাধ্যমে কোনটি অর্জিত হয়?", ["সময় অপচয়", "দক্ষতা ও সঠিকতা", "ভীতি বৃদ্ধি", "কোনোটিই নয়"], key=f"q2_{task_key}")
                                        q3 = st.radio("৩. প্রশ্নের সঠিক উত্তর প্রদানের মূল চাবিকাঠি কী?", ["হুবহু মুখস্থ করা", "মূল বইয়ের ধারণা স্পষ্ট থাকা", "শুধু গাইড পড়া", "বড় করে লেখা"], key=f"q3_{task_key}")
                                        
                                        if st.button("উত্তর জমা দিন (Submit MCQ)", key=f"sub_mcq_{task_key}", type="primary"):
                                            score = 0
                                            q_details = []
                                            
                                            # Q1 Check
                                            ans1_correct = (q1 == "মৌলিক ধারণা ও অনুচ্ছেদ")
                                            if ans1_correct: score += 1
                                            q_details.append({"q": "১. প্রদত্ত অধ্যায়ের মূল ভিত্তি বা ধারণা কোনটি?", "user_ans": q1, "correct_ans": "মৌলিক ধারণা ও অনুচ্ছেদ", "is_correct": ans1_correct})
                                            
                                            # Q2 Check
                                            ans2_correct = (q2 == "দক্ষতা ও সঠিকতা")
                                            if ans2_correct: score += 1
                                            q_details.append({"q": "২. নিয়মিত চর্চার মাধ্যমে কোনটি অর্জিত হয়?", "user_ans": q2, "correct_ans": "দক্ষতা ও সঠিকতা", "is_correct": ans2_correct})
                                            
                                            # Q3 Check
                                            ans3_correct = (q3 == "মূল বইয়ের ধারণা স্পষ্ট থাকা")
                                            if ans3_correct: score += 1
                                            q_details.append({"q": "৩. প্রশ্নের সঠিক উত্তর প্রদানের মূল চাবিকাঠি কী?", "user_ans": q3, "correct_ans": "মূল বইয়ের ধারণা স্পষ্ট থাকা", "is_correct": ans3_correct})

                                            percent = int((score / 3) * 100)
                                            st.session_state["user_quiz_scores"][task_key] = f"MCQ: {percent}%"
                                            
                                            # সেভ টু আর্কাইভ
                                            save_test_result_to_db("বহুনির্বাচনী প্রশ্ন (MCQ)", score, 3, score, 3 - score, {"questions": q_details})

                                            if percent == 100:
                                                st.balloons()
                                                st.success(f"🎉 অসাধারণ! আপনি ১০০% মার্কস পেয়েছেন! টেস্ট রেজাল্ট স্বয়ংক্রিয়ভাবে আর্কাইভে সেভ করা হয়েছে।")
                                            else:
                                                st.info(f"👍 আপনার অর্জিত স্কোর: {percent}% (সঠিক: {score}, ভুল: {3-score})। টেস্টের তথ্য আর্কাইভে সংরক্ষিত হয়েছে।")
                                            st.rerun()

                                    elif test_type == "সৃজনশীল প্রশ্ন (Creative / CQ)":
                                        st.markdown("##### 🎨 সৃজনশীল উদ্দীপক ও প্রশ্ন:")
                                        st.info("💡 উদ্দীপক: রাফি প্রতিদিন তার রুটিন মেনে পড়ার চেষ্টা করে। এতে সে পরীক্ষার পূর্বে প্রতিটি অধ্যায়ের ওপর পূর্ণ আস্থা অর্জন করতে সক্ষম হয়।")
                                        st.markdown("**ক. জ্ঞানমূলক (১ নম্বর):** মূল বিষয়বস্তুর সংজ্ঞা দাও।")
                                        st.markdown("**খ. অনুধাবনমূলক (২ নম্বর):** নিয়মিত অনুশীলনের গুরুত্ব ব্যাখ্যা করো।")
                                        st.markdown("**গ. প্রয়োগমূলক (৩ নম্বর):** উদ্দীপকে রাফির অধ্যবসায় উক্ত অধ্যায়ের কোন দিকটির সাথে মিল রয়েছে? ব্যাখ্যা করো।")
                                        
                                        user_answer = st.text_area("তোমার খসড়া উত্তর বা মূল পয়েন্টগুলো লেখো:", placeholder="এখানে তোমার উত্তর লেখো...", key=f"cq_ans_{task_key}")
                                        if st.button("উত্তর মিলিয়ে যাচাই করো", key=f"sub_cq_{task_key}", type="primary"):
                                            st.success("✔ উত্তর সফলভাবে মূল্যায়নের জন্য সংরক্ষিত হয়েছে! স্ব-মূল্যায়ন স্কোর: ১০/১০")
                                            st.session_state["user_quiz_scores"][task_key] = "CQ: ১০০% (সম্পন্ন)"
                                            
                                            # সেভ টু আর্কাইভ
                                            cq_data = {
                                                "questions": ["ক. জ্ঞানমূলক", "খ. অনুধাবনমূলক", "গ. প্রয়োগমূলক"],
                                                "user_written_answer": user_answer if user_answer else "উত্তর প্রদান করা হয়েছে।"
                                            }
                                            save_test_result_to_db("সৃজনশীল প্রশ্ন (CQ)", 10, 10, 3, 0, cq_data)
                                            
                                            st.balloons()
                                            st.rerun()

                                    elif test_type == "সংক্ষিপ্ত প্রশ্ন (Short Questions)":
                                        st.markdown("##### ⚡ ৩টি সংক্ষিপ্ত প্রশ্ন উত্তরসহ যাচাই:")
                                        st.markdown("১. **প্রশ্ন:** অধ্যায়টির মূল উদ্দেশ্য কী?  \n*উত্তর:* মূল তত্ত্ব ও সূত্রের স্পষ্ট ধারণা নেওয়া।")
                                        st.markdown("২. **প্রশ্ন:** গাণিতিক বা ব্যাকরণিক সমস্যা কীভাবে সমাধান করতে হয়?  \n*উত্তর:* পর্যায়ক্রমিক নিয়ম অনুসরণ করে।")
                                        st.markdown("৩. **প্রশ্ন:** কীভাবে দ্রুত রিভিশন দেওয়া যায়?  \n*উত্তর:* সংক্ষিপ্ত নোটস বা সামারি শীট দেখে।")
                                        
                                        chk_short = st.checkbox("আমি ৩টি সংক্ষিপ্ত প্রশ্নের উত্তর ভালোভাবে বুঝতে পেরেছি", key=f"chk_short_{task_key}")
                                        if st.button("যাচাই সম্পন্ন করুন", key=f"sub_short_{task_key}", type="primary"):
                                            if chk_short:
                                                st.session_state["user_quiz_scores"][task_key] = "সংক্ষিপ্ত: ১০০%"
                                                save_test_result_to_db("সংক্ষিপ্ত প্রশ্ন", 3, 3, 3, 0, {"status": "সকল প্রশ্নের উত্তর সঠিকভাবে যাচাই করা হয়েছে"})
                                                st.balloons()
                                                st.success("🎉 চমৎকার! সংক্ষিপ্ত প্রশ্ন যাচাই সম্পন্ন ও আর্কাইভে সেভ হয়েছে।")
                                                st.rerun()
                                            else:
                                                st.warning("দয়া করে প্রশ্নগুলো পড়ে টিক চিহ্ন দাও।")

                                    elif test_type == "মডেল কোশ্চেন টেস্ট (Model Test)":
                                        st.markdown("##### 🏆 সংক্ষিপ্ত মডেল টেস্ট (১০ নম্বর):")
                                        mt_q1 = st.radio("১. সঠিক উত্তর বেছে নাও:", ["বিকল্প A", "বিকল্প B (সঠিক)", "বিকল্প C"], key=f"mt1_{task_key}")
                                        mt_q2 = st.text_input("২. এক কথায় উত্তর দাও (অধ্যায়ের মূল প্রবক্তা কে?):", key=f"mt2_{task_key}")
                                        
                                        if st.button("মডেল টেস্ট জমা দিন", key=f"sub_mt_{task_key}", type="primary"):
                                            mt_score = 10 if mt_q1 == "বিকল্প B (সঠিক)" else 5
                                            st.session_state["user_quiz_scores"][task_key] = f"মডেল টেস্ট: {(mt_score/10)*100}%"
                                            save_test_result_to_db("মডেল কোশ্চেন টেস্ট", mt_score, 10, 1 if mt_score==10 else 0, 0 if mt_score==10 else 1, {"q1": mt_q1, "q2": mt_q2})
                                            st.balloons()
                                            st.success("🎉 সাবাশ! মডেল টেস্ট সফলভাবে সম্পন্ন ও আর্কাইভে সংরক্ষণ করা হয়েছে!")
                                            st.rerun()

                                show_quiz_dialog()

                        # টেস্ট স্কোরের আপডেট প্রদর্শন
                        if task_key in st.session_state["user_quiz_scores"]:
                            q_score = st.session_state["user_quiz_scores"][task_key]
                            st.caption(f"🏆 সর্বশেষ যাচাই রেজাল্ট: **{q_score}**")

                        col_st, col_hr, col_dn, col_pn = st.columns([1.5, 1.2, 2.3, 2.3])

                        with col_st:
                            current_status = st.selectbox(
                                "পড়ার অবস্থা:",
                                ["✔ সম্পন্ন সম্পন্ন", "⚡ আংশিক সম্পন্ন", "❌ শেষ হয় নাই"],
                                index=["✔ সম্পন্ন সম্পন্ন", "⚡ আংশিক সম্পন্ন", "❌ শেষ হয় নাই"].index(st.session_state["user_subject_status"].get(task_key, "✔ সম্পন্ন সম্পন্ন")),
                                key=f"status_sel_{task_key}"
                            )
                            st.session_state["user_subject_status"][task_key] = current_status

                        with col_hr:
                            subject_hours_txt = st.text_input(
                                "পড়ার সময়:",
                                value=st.session_state["user_subject_hours_text"].get(task_key, "১.০ ঘণ্টা"),
                                key=f"hrs_txt_{task_key}"
                            )
                            st.session_state["user_subject_hours_text"][task_key] = subject_hours_txt

                        with col_dn:
                            done_text = st.text_input(
                                "সম্পন্ন হয়েছে:",
                                value=st.session_state["user_done_notes"].get(task_key, ""),
                                placeholder="যেমন: পড়া শেষ...",
                                key=f"done_in_{task_key}"
                            )
                            st.session_state["user_done_notes"][task_key] = done_text

                        with col_pn:
                            pending_text = st.text_input(
                                "বাকি আছে বা কারণ:",
                                value=st.session_state["user_pending_notes"].get(task_key, ""),
                                placeholder="যেমন: কারণ বা অসম্পূর্ণ অংশ...",
                                key=f"pending_in_{task_key}"
                            )
                            st.session_state["user_pending_notes"][task_key] = pending_text

                        obstacle_val = "কোনো সমস্যা নেই"
                        if current_status != "✔ সম্পন্ন সম্পন্ন":
                            obstacle_val = st.selectbox(
                                "⚠️ পড়া শেষ না হওয়ার মূল কারণ কী ছিল?",
                                ["সময়ের অভাব / ব্যস্ততা", "টপিক কঠিন বা বুঝতে সমস্যা হয়েছে", "শারীরিক অসুস্থতা / ক্লান্তি", "অন্যমনস্কতা বা মনোযোগের অভাব", "পারিবারিক বা অন্যান্য কাজ"],
                                index=0,
                                key=f"obstacle_{task_key}"
                            )
                            st.session_state["user_obstacle_reason"][task_key] = obstacle_val
                            obstacle_reasons_list.append(obstacle_val)

                        # স্কোর গণনায় কুইজ রেজাল্টের বোনাস প্রভাব
                        base_val = 1.0 if current_status == "✔ সম্পন্ন সম্পন্ন" else (0.5 if current_status == "⚡ আংশিক সম্পন্ন" else 0.0)
                        if task_key in st.session_state["user_quiz_scores"]:
                            base_val = min(1.0, base_val + 0.2)

                        day_score += base_val

                        if current_status != "✔ সম্পন্ন সম্পন্ন" or pending_text.strip() != "":
                            backlog_items.append({
                                "day": day_num,
                                "subject": s_name,
                                "topic": top_name,
                                "done": done_text if done_text else "কিছু অংশ পঠিত",
                                "pending": pending_text if pending_text else "বাকি অংশ অসম্পূর্ণ",
                                "status": current_status,
                                "reason": obstacle_val
                            })

                        st.markdown("<hr style='margin: 8px 0; border: none; border-bottom: 1px solid #E2E8F0;'>", unsafe_allow_html=True)

                    total_tasks_global += day_task_count
                    completed_score_global += day_score
                    day_prog_percentage = int((day_score / max(1, day_task_count)) * 100)

                    st.markdown(f"**📈 দিন {day_num} এর প্রোগ্রেস: {day_prog_percentage}% (টার্গেট সময়: {daily_study_hours} ঘণ্টা)**")
                    st.progress(min(1.0, day_prog_percentage / 100))
                    st.markdown("</div>", unsafe_allow_html=True)

                global_percentage = int((completed_score_global / max(1, total_tasks_global)) * 100)
                st.markdown("---")
                st.markdown("### 📊 আপনার সার্বিক প্রোগ্রেস রিপোর্ট ও এআই ইনসাইটস")
                
                col_p_left, col_p_right = st.columns([1, 2])
                with col_p_left:
                    st.metric("সামগ্রিক সিলেবাস প্রোগ্রেস", f"{global_percentage}%")
                    st.progress(min(1.0, global_percentage / 100))
                    if global_percentage >= 100:
                        st.balloons()
                        st.success("🎉 অসাধারণ! ১০০% টার্গেট ও মডেল টেস্ট সফলভাবে পূরণ হয়েছে!")
                with col_p_right:
                    if obstacle_reasons_list:
                        reason_counts = Counter(obstacle_reasons_list)
                        most_common_reason, count = reason_counts.most_common(1)[0]
                        st.info(f"💡 **AI Insights:** পিছিয়ে পড়ার প্রধান কারণ: **{most_common_reason}** ({count} বার)।")
                    if backlog_items:
                        st.warning(f"📌 **বকেয়া রিমাইন্ডার:** মোট {len(backlog_items)}টি অপূর্ণ টপিক রয়েছে।")
                    else:
                        st.success("🎉 দারুণ! কোনো বকেয়া পড়া নেই।")

                # স্টাডি প্ল্যান সমাপ্তি ও সম্পূর্ণ প্ল্যান হুবহু আর্কাইভে সেভ
                st.markdown("---")
                st.markdown("### 🏁 স্টাডি প্ল্যান সমাপ্তি ও আর্কাইভ অপশন")
                st.info("আপনার স্টাডি প্ল্যানের সমস্ত দিন শেষ হলে নিচের অপশনগুলো ব্যবহার করুন:")

                col_end1, col_end2 = st.columns(2)
                with col_end1:
                    if st.button("📦 এই সম্পূর্ণ স্টাডি প্ল্যানটি হুবহু আর্কাইভে সেভ করুন", use_container_width=True, type="primary"):
                        archive_payload = {
                            "target_days": target_days,
                            "daily_hours": daily_study_hours,
                            "tasks_store": st.session_state["custom_day_tasks_store"]
                        }
                        conn = sqlite3.connect("smart_school_syllabus.db")
                        cursor = conn.cursor()
                        cursor.execute("""
                            INSERT INTO universal_archives (archive_type, title, details_json, archive_date)
                            VALUES ('StudyPlan', ?, ?, datetime('now', 'localtime'))
                        """, (st.session_state.get("active_plan_name", "সম্পন্ন স্টাডি প্ল্যান"), json.dumps(archive_payload, ensure_ascii=False)))
                        conn.commit()
                        conn.close()
                        st.success("অভিনন্দন! আপনার সম্পূর্ণ স্টাডি প্ল্যানটি হুবহু ছক অনুযায়ী 'Archive' ট্যাবে সংরক্ষণ করা হয়েছে।")
                
                with col_end2:
                    if backlog_items:
                        if st.button("🔄 বকেয়া পড়াগুলোর জন্য নতুন প্ল্যান তৈরি করুন", use_container_width=True):
                            st.session_state["create_backlog_plan"] = True
                            st.success("বকেয়া টপিকগুলো নিয়ে নতুন প্ল্যান তৈরির জন্য নিচের ফরমটি পূরণ করুন!")
                    else:
                        st.success("আপনার কোনো বকেয়া পড়া নেই! সমস্ত পড়া সম্পূর্ণ হয়েছে।")

                if st.session_state.get("create_backlog_plan", False):
                    st.markdown("---")
                    st.markdown("#### 🛠️ বকেয়া পড়াগুলোর জন্য নতুন স্টাডি প্ল্যান সেটআপ:")
                    backlog_days = st.number_input("বকেয়া পড়া শেষ করার জন্য কত দিন সময় লাগবে?", min_value=1, max_value=60, value=5, step=1, key="backlog_days_input")
                    backlog_hours = st.slider("প্রতিদিন কত ঘণ্টা করে পড়তে চান?", 1.0, 10.0, 3.0, 0.5, key="backlog_hours_input")

                    if st.button("🚀 বকেয়া প্ল্যান জেনারেট করুন", type="primary"):
                        backlog_store = {}
                        topics_per_day_b = math.ceil(len(backlog_items) / backlog_days)
                        b_idx = 0
                        for d in range(1, backlog_days + 1):
                            b_key = f"day_tasks_{d}"
                            chunk = backlog_items[b_idx: b_idx + topics_per_day_b]
                            backlog_store[b_key] = [{"subject": item["subject"], "category": "বকেয়া রিকভারি", "topic": item["topic"]} for item in chunk]
                            b_idx += topics_per_day_b

                        st.session_state["active_target_days"] = backlog_days
                        st.session_state["active_daily_hours"] = backlog_hours
                        st.session_state["custom_day_tasks_store"] = backlog_store
                        st.session_state["create_backlog_plan"] = False
                        st.success("বকেয়া পড়াগুলো নিয়ে নতুন স্টাডি প্ল্যান সফলভাবে তৈরি করা হয়েছে! উপরে প্ল্যান ট্র্যাকার চেক করুন।")
                        st.rerun()
        else:
            st.warning("⚠ কোনো সংরক্ষিত সিলেবাস পাওয়া যায়নি। দয়া করে প্রথমে 'Syllabus' ট্যাব থেকে একটি সিলেবাস তৈরি ও সেভ করুন।")

    # --- TAB 4: ARCHIVE (সার্বজনীন আর্কাইভ সেন্টার - ৪টি সাব-ট্যাবসহ) ---
    with tab4:
        st.markdown("এখানে তোমার সেভ করা সমস্ত সিলেবাস, স্টাডি প্ল্যান, রুটিন এবং নিজেকে যাচাই করো সংক্রান্ত যাবতীয় টেস্ট হিস্ট্রি সংরক্ষিত রয়েছে। তুমি চাইলে এখান থেকে সব কিছু আবার রিস্টোর করতে পারবে। ")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, archive_type, title, details_json, archive_date FROM universal_archives ORDER BY id DESC")
        archived_records = cursor.fetchall()
        
        # টেস্ট হিস্ট্রি লোড করা
        cursor.execute("SELECT id, target_day_num, subject_name, topic_name, test_type, score_obtained, total_score, percentage, correct_count, wrong_count, details_json, test_date FROM test_history_archive ORDER BY id DESC")
        test_history_records = cursor.fetchall()
        conn.close()

        arc_tab1, arc_tab2, arc_tab3, arc_tab4 = st.tabs(["📚 Archived Syllabus", "🎯 Archived Study Plan","📅 Archived Routine",  "🧪 নিজেকে যাচাই করো আর্কাইভ"])

        # ১. আর্কাইভড সিলেবাস
        with arc_tab1:
            syllabus_archives = [r for r in archived_records if r[1] == 'Syllabus']
            if syllabus_archives:
                for arc in syllabus_archives:
                    arc_id, arc_type, title, details_str, arc_date = arc
                    with st.expander(f"📖 {title} — (আর্কাইভ তারিখ: {arc_date})"):
                        if st.button("📊 আর্কাইভ সিলেবাস সামারি (এক নজরে)", key=f"btn_arc_syl_sum_{arc_id}", type="secondary"):
                            @st.dialog("📊 আর্কাইভ সিলেবাস সামারি টেবিল")
                            def show_arc_syl_sum():
                                st.markdown(f"**সিলেবাস:** {title}")
                                p_syl = json.loads(details_str)
                                s_rows = []
                                s_cnt = 1
                                for sb, cats in p_syl.items():
                                    for ct, tps in cats.items():
                                        s_rows.append(f"<tr><td style='padding:8px; border:1px solid #cbd5e1;'>{s_cnt}</td><td style='padding:8px; border:1px solid #cbd5e1; font-weight:bold;'>{sb}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{ct}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{', '.join(tps)}</td></tr>")
                                        s_cnt += 1
                                st.markdown(f"<table style='width:100%; border-collapse:collapse; font-size:14px;'><thead><tr style='background:#1E3A8A; color:white;'><th style='padding:8px; border:1px solid #cbd5e1;'>ক্রমিক</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিষয়</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিভাগ</th><th style='padding:8px; border:1px solid #cbd5e1;'>টপিক</th></tr></thead><tbody>{''.join(s_rows)}</tbody></table>", unsafe_allow_html=True)
                                st.markdown("<br>", unsafe_allow_html=True)
                                if st.button("বন্ধ করুন", key=f"cls_arc_syl_{arc_id}"):
                                    st.rerun()
                            show_arc_syl_sum()

                        parsed_arc_syl = json.loads(details_str)
                        table_rows = []
                        for subj, cats in parsed_arc_syl.items():
                            first_row = True
                            cat_count = len(cats)
                            for cat, topics in cats.items():
                                topics_str = ", ".join(topics)
                                if first_row:
                                    row_html = f'<tr><td rowspan="{cat_count}" style="padding: 10px; border: 1px solid #cbd5e1; font-weight: bold; vertical-align: top;">{subj}</td><td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">{cat}</td><td style="padding: 10px; border: 1px solid #cbd5e1;">{topics_str}</td></tr>'
                                    first_row = False
                                else:
                                    row_html = f'<tr><td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">{cat}</td><td style="padding: 10px; border: 1px solid #cbd5e1;">{topics_str}</td></tr>'
                                table_rows.append(row_html)
                        all_rows = "".join(table_rows)
                        table_html = f'<table style="width: 100%; border-collapse: collapse; margin-top: 10px; font-family: sans-serif;"><thead><tr style="background-color: #1E3A8A; color: white; text-align: left;"><th style="padding: 12px; border: 1px solid #cbd5e1;">বিষয় (Subject)</th><th style="padding: 12px; border: 1px solid #cbd5e1;">বিভাগ (Category)</th><th style="padding: 12px; border: 1px solid #cbd5e1;">নির্বাচিত অধ্যায়সমূহ (Selected Topics)</th></tr></thead><tbody>{all_rows}</tbody></table>'
                        st.markdown(table_html, unsafe_allow_html=True)
                        st.markdown("<br>", unsafe_allow_html=True)

                        col_ar1, col_ar2 = st.columns(2)
                        with col_ar1:
                            if st.button("🔄 মূল সিলেবাস ট্যাবে রিস্টোর করুন", key=f"restore_syl_{arc_id}", use_container_width=True, type="primary"):
                                conn = sqlite3.connect("smart_school_syllabus.db")
                                cursor = conn.cursor()
                                cursor.execute("""
                                    INSERT INTO school_syllabus_multi (student_class, exam_term, syllabus_json, is_default)
                                    VALUES (?, ?, ?, 0)
                                """, ("Class 7", title, details_str))
                                conn.commit()
                                conn.close()
                                st.success("সিলেবাসটি সফলভাবে মূল 'Syllabus' ট্যাবে রিস্টোর করা হয়েছে!")
                                st.rerun()
                        with col_ar2:
                            if st.button("🗑️ আর্কাইভ থেকে মুছে ফেলুন", key=f"del_arc_syl_{arc_id}", use_container_width=True):
                                conn = sqlite3.connect("smart_school_syllabus.db")
                                cursor = conn.cursor()
                                cursor.execute("DELETE FROM universal_archives WHERE id=?", (arc_id,))
                                conn.commit()
                                conn.close()
                                st.success("আর্কাইভ থেকে সিলেবাসটি মুছে ফেলা হয়েছে!")
                                st.rerun()
            else:
                st.info("ℹ এখনো কোনো সিলেবাস আর্কাইভে সেভ করা হয়নি।")

        # ২. আর্কাইভড রুটিন
        with arc_tab2:
            routine_archives = [r for r in archived_records if r[1] == 'Routine']
            if routine_archives:
                for arc in routine_archives:
                    arc_id, arc_type, title, details_str, arc_date = arc
                    with st.expander(f"📅 {title} — (আর্কাইভ তারিখ: {arc_date})"):
                        if st.button("📊 আর্কাইভ রুটিন সামারি (এক নজরে)", key=f"btn_arc_rout_sum_{arc_id}", type="secondary"):
                            @st.dialog("📊 আর্কাইভ রুটিন সামারি টেবিল")
                            def show_arc_rout_sum():
                                st.markdown(f"**রুটিন:** {title}")
                                p_rout = json.loads(details_str)
                                r_rows = []
                                r_cnt = 1
                                for dn, ents in p_rout.items():
                                    for et in ents:
                                        r_rows.append(f"<tr><td style='padding:8px; border:1px solid #cbd5e1;'>{r_cnt}</td><td style='padding:8px; border:1px solid #cbd5e1; font-weight:bold;'>{dn}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{et['subject']}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{et['time']}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{et['chapter']}</td></tr>")
                                        r_cnt += 1
                                st.markdown(f"<table style='width:100%; border-collapse:collapse; font-size:14px;'><thead><tr style='background:#1E3A8A; color:white;'><th style='padding:8px; border:1px solid #cbd5e1;'>ক্রমিক</th><th style='padding:8px; border:1px solid #cbd5e1;'>বার</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিষয়</th><th style='padding:8px; border:1px solid #cbd5e1;'>সময়</th><th style='padding:8px; border:1px solid #cbd5e1;'>টপিক</th></tr></thead><tbody>{''.join(r_rows)}</tbody></table>", unsafe_allow_html=True)
                                st.markdown("<br>", unsafe_allow_html=True)
                                if st.button("বন্ধ করুন", key=f"cls_arc_rout_{arc_id}"):
                                    st.rerun()
                            show_arc_rout_sum()

                        parsed_arc_routine = json.loads(details_str)
                        for day_name, entries in parsed_arc_routine.items():
                            st.markdown(f"**বার: {day_name}**")
                            for entry in entries:
                                target_info = f"{entry.get('pages', 0)}টি পৃষ্ঠা | {entry.get('mcq', 0)}টি MCQ | {entry.get('cq', 0)}টি CQ"
                                st.markdown(f"""
                                <div style="background: #F8FAFC; border-left: 4px solid #3B82F6; padding: 10px; margin-bottom: 8px; border-radius: 4px;">
                                    <b>বিষয়:</b> {entry['subject']} | <b>সময়:</b> {entry['time']}<br>
                                    <b>টপিক/অধ্যায়:</b> {entry['chapter']} ({target_info})
                                </div>
                                """, unsafe_allow_html=True)
                            st.markdown("---")

                        col_ar1, col_ar2 = st.columns(2)
                        with col_ar1:
                            if st.button("🔄 মূল রুটিন ট্যাবে রিস্টোর করুন", key=f"restore_rout_{arc_id}", use_container_width=True, type="primary"):
                                st.session_state["generated_custom_weekly_routine"] = parsed_arc_routine
                                st.session_state["generated_routine_syllabus_name"] = title
                                st.success("রুটিনটি সফলভাবে মূল 'Routine' ট্যাবে রিস্টোর করা হয়েছে!")
                        with col_ar2:
                            if st.button("🗑️ আর্কাইভ থেকে মুছে ফেলুন", key=f"del_arc_rout_{arc_id}", use_container_width=True):
                                conn = sqlite3.connect("smart_school_syllabus.db")
                                cursor = conn.cursor()
                                cursor.execute("DELETE FROM universal_archives WHERE id=?", (arc_id,))
                                conn.commit()
                                conn.close()
                                st.success("আর্কাইভ থেকে রুটিনটি মুছে ফেলা হয়েছে!")
                                st.rerun()
            else:
                st.info("ℹ এখনো কোনো রুটিন আর্কাইভে সেভ করা হয়নি।")

        # ৩. আর্কাইভড স্টাডি প্ল্যান
        with arc_tab3:
            plan_archives = [r for r in archived_records if r[1] == 'StudyPlan']
            if plan_archives:
                for arc in plan_archives:
                    arc_id, arc_type, title, details_str, arc_date = arc
                    with st.expander(f"🎯 {title} — (আর্কাইভ তারিখ: {arc_date})"):
                        if st.button("📊 আর্কাইভ স্টাডি প্ল্যান সামারি (এক নজরে)", key=f"btn_arc_plan_sum_{arc_id}", type="secondary"):
                            @st.dialog("📊 আর্কাইভ স্টাডি প্ল্যান সামারি টেবিল")
                            def show_arc_plan_sum():
                                st.markdown(f"**প্ল্যান:** {title}")
                                p_plan_data = json.loads(details_str)
                                tp_days = p_plan_data.get("target_days", 25)
                                tp_store = p_plan_data.get("tasks_store", {})
                                p_rows = []
                                p_cnt = 1
                                for dn_i in range(1, tp_days + 1):
                                    dk_i = f"day_tasks_{dn_i}"
                                    t_list_i = tp_store.get(dk_i, [])
                                    for t_item_i in t_list_i:
                                        p_rows.append(f"<tr><td style='padding:8px; border:1px solid #cbd5e1;'>{p_cnt}</td><td style='padding:8px; border:1px solid #cbd5e1; font-weight:bold;'>দিন {dn_i}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{t_item_i.get('subject', '')}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{t_item_i.get('category', '')}</td><td style='padding:8px; border:1px solid #cbd5e1;'>{t_item_i.get('topic', '')}</td></tr>")
                                        p_cnt += 1
                                st.markdown(f"<table style='width:100%; border-collapse:collapse; font-size:14px;'><thead><tr style='background:#1E3A8A; color:white;'><th style='padding:8px; border:1px solid #cbd5e1;'>ক্রমিক</th><th style='padding:8px; border:1px solid #cbd5e1;'>দিন</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিষয়</th><th style='padding:8px; border:1px solid #cbd5e1;'>বিভাগ</th><th style='padding:8px; border:1px solid #cbd5e1;'>টপিক</th></tr></thead><tbody>{''.join(p_rows)}</tbody></table>", unsafe_allow_html=True)
                                st.markdown("<br>", unsafe_allow_html=True)
                                if st.button("বন্ধ করুন", key=f"cls_arc_plan_{arc_id}"):
                                    st.rerun()
                            show_arc_plan_sum()

                        parsed_arc_plan = json.loads(details_str)
                        t_days = parsed_arc_plan.get("target_days", 25)
                        d_hours = parsed_arc_plan.get("daily_hours", 4.0)
                        tasks_store = parsed_arc_plan.get("tasks_store", {})

                        st.markdown(f"""
                        <div style="background-color: #E8F0FE; padding: 12px 18px; border-radius: 8px; color: #1E3A8A; font-weight: bold; margin-bottom: 20px;">
                        📌 আর্কাইভকৃত স্টাডি প্ল্যান বিবরণ: মোট দিন: {t_days} | প্রতিদিন নির্ধারিত সময়: {d_hours} ঘণ্টা
                        </div>
                        """, unsafe_allow_html=True)

                        for day_num in range(1, t_days + 1):
                            day_key = f"day_tasks_{day_num}"
                            day_tasks = tasks_store.get(day_key, [])
                            
                            is_rev = day_num > (t_days - max(1, math.ceil(t_days * 0.20)))
                            bg_color = "#FEF3C7" if is_rev else "#F8FAFC"
                            border_color = "#D97706" if is_rev else "#3B82F6"
                            day_title_prefix = "🔄 বিশেষ রিভিশন দিন" if is_rev else "📖 টার্গেট ডে"

                            st.markdown(f"""
                            <div style="background: {bg_color}; border: 2px solid {border_color}; padding: 15px; border-radius: 10px; margin-bottom: 15px;">
                                <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #CBD5E1; padding-bottom: 8px; margin-bottom: 12px;">
                                    <span style="font-size: 16px; font-weight: bold; color: #1E3A8A;">📅 দিন {day_num} ({day_title_prefix})</span>
                                    <span style="background-color: #DBEAFE; color: #1E4ED8; font-weight: bold; padding: 4px 10px; border-radius: 6px; font-size: 13px;">নির্ধারিত সময়: {d_hours} ঘণ্টা</span>
                                </div>
                            """, unsafe_allow_html=True)

                            if day_tasks:
                                for t in day_tasks:
                                    st.markdown(f"""
                                    <div style="background: #FFFFFF; border: 1px solid #E2E8F0; padding: 10px 14px; border-radius: 6px; margin-bottom: 8px;">
                                        <b>📌 বিষয়:</b> {t.get('subject', '')} <br>
                                        <b>বিভাগ/ক্যাটাগরি:</b> [{t.get('category', 'সাধারণ')}] <br>
                                        <b>টপিক বা অধ্যায়:</b> {t.get('topic', '')}
                                    </div>
                                    """, unsafe_allow_html=True)
                            else:
                                st.markdown("<i>কোনো টাস্ক নেই</i>", unsafe_allow_html=True)

                            st.markdown("</div>", unsafe_allow_html=True)

                        col_ar1, col_ar2 = st.columns(2)
                        with col_ar1:
                            if st.button("🔄 মূল স্টাডি প্ল্যান ট্যাবে রিস্টোর করুন", key=f"restore_plan_{arc_id}", use_container_width=True, type="primary"):
                                conn = sqlite3.connect("smart_school_syllabus.db")
                                cursor = conn.cursor()
                                cursor.execute("""
                                    INSERT INTO saved_study_plans (plan_name, syllabus_label, target_days, daily_hours, plan_json)
                                    VALUES (?, ?, ?, ?, ?)
                                """, (title, "মূল সিলেবাস", t_days, d_hours, json.dumps(tasks_store, ensure_ascii=False)))
                                conn.commit()
                                conn.close()
                                
                                st.session_state["active_plan_name"] = title
                                st.session_state["active_target_days"] = t_days
                                st.session_state["active_daily_hours"] = d_hours
                                st.session_state["custom_day_tasks_store"] = tasks_store
                                st.success("স্টাডি প্ল্যানটি সফলভাবে মূল 'Study Plan' ট্যাবে রিস্টোর করা হয়েছে!")
                        with col_ar2:
                            if st.button("🗑️ আর্কাইভ থেকে মুছে ফেলুন", key=f"del_arc_plan_{arc_id}", use_container_width=True):
                                conn = sqlite3.connect("smart_school_syllabus.db")
                                cursor = conn.cursor()
                                cursor.execute("DELETE FROM universal_archives WHERE id=?", (arc_id,))
                                conn.commit()
                                conn.close()
                                st.success("আর্কাইভ থেকে স্টাডি প্ল্যানটি মুছে ফেলা হয়েছে!")
                                st.rerun()
            else:
                st.info("ℹ এখনো কোনো স্টাডি প্ল্যান আর্কাইভে সেভ করা হয়নি।")

        # ৪. নিজেকে যাচাইকরণ সংক্রান্ত টেস্ট হিস্ট্রি আর্কাইভ (NEW FEATURE)
        with arc_tab4:
            st.markdown("#### 🧪 নিজেকে যাচাইকরণ সংক্রান্ত সার্বিক টেস্ট আর্কাইভ ফাইল")
            st.markdown("এখানে আপনার দেওয়া প্রতিটি মডেল টেস্ট, কুইজ, সৃজনশীল ও সংক্ষিপ্ত পরীক্ষার সম্পূর্ণ রেকর্ড রয়েছে।")

            if test_history_records:
                total_tests_given = len(test_history_records)
                avg_percentage = round(sum([r[7] for r in test_history_records]) / total_tests_given, 1)
                total_wrong_answers = sum([r[9] for r in test_history_records])

                col_m1, col_m2, col_m3 = st.columns(3)
                with col_m1:
                    st.metric("মোট সম্পন্ন পরীক্ষা", f"{total_tests_given} টি")
                with col_m2:
                    st.metric("গড় প্রাপ্ত নম্বর (শতাংশ)", f"{avg_percentage}%")
                with col_m3:
                    st.metric("মোট ভুল উত্তর (রিভিউ প্রয়োজন)", f"{total_wrong_answers} টি")

                st.markdown("---")
                st.markdown("##### 📜 পরীক্ষার তালিকা ও বিস্তারিত ফলাফল:")

                for record in test_history_records:
                    rec_id, day_num, sub_name, topic_name, t_type, score, t_score, percent, c_cnt, w_cnt, details_str, t_date = record
                    
                    badge_color = "#059669" if percent >= 80 else ("#D97706" if percent >= 50 else "#DC2626")
                    
                    st.markdown(f"""
                    <div style="background: #FFFFFF; border: 1px solid #E2E8F0; border-left: 5px solid {badge_color}; padding: 15px; border-radius: 8px; margin-bottom: 12px;">
                        <div style="display: flex; justify-content: space-between; align-items: center;">
                            <div>
                                <span style="font-size: 16px; font-weight: bold; color: #1E3A8A;">📅 দিন {day_num} — {sub_name}</span>
                                <span style="background-color: #DBEAFE; color: #1D4ED8; padding: 2px 8px; border-radius: 4px; font-size: 12px; margin-left: 8px;">{t_type}</span>
                                <div style="color: #475569; font-size: 13px; margin-top: 4px;"><b>অধ্যায়:</b> {topic_name}</div>
                            </div>
                            <div style="text-align: right;">
                                <div style="font-size: 18px; font-weight: bold; color: {badge_color};">{percent}%</div>
                                <div style="font-size: 12px; color: #64748B;">প্রাপ্ত নম্বর: {score}/{t_score} (তারিখ: {t_date})</div>
                            </div>
                        </div>
                    </div>
                    """, unsafe_allow_html=True)

                    col_hb1, col_hb2 = st.columns([4, 1])
                    with col_hb1:
                        if st.button("📊 সম্পূর্ণ বিবরণ ও উত্তরপত্র দেখুন", key=f"btn_view_test_{rec_id}", type="secondary"):
                            @st.dialog(f"📊 টেস্ট রিপোর্ট: {sub_name} (দিন {day_num})")
                            def show_test_detail_modal():
                                st.markdown(f"**বিষয়:** {sub_name} | **অধ্যায়:** {topic_name}")
                                st.markdown(f"**পরীক্ষার ধরন:** {t_type} | **তারিখ:** {t_date}")
                                st.markdown(f"**প্রাপ্ত স্কোর:** {score}/{t_score} ({percent}%) | **সঠিক:** {c_cnt} | **ভুল:** {w_cnt}")
                                st.markdown("---")
                                
                                try:
                                    parsed_details = json.loads(details_str)
                                    if "questions" in parsed_details:
                                        st.markdown("##### 📝 প্রশ্ন ও উত্তর বিশ্লেষণ:")
                                        for q_idx, q_item in enumerate(parsed_details["questions"]):
                                            if isinstance(q_item, dict):
                                                q_txt = q_item.get("q", "")
                                                u_ans = q_item.get("user_ans", "")
                                                c_ans = q_item.get("correct_ans", "")
                                                is_c = q_item.get("is_correct", False)
                                                
                                                status_icon = "✔ সঠিক" if is_c else "❌ ভুল"
                                                icon_color = "green" if is_c else "red"
                                                
                                                st.markdown(f"**প্রশ্ন {q_idx+1}:** {q_txt}")
                                                st.markdown(f"- আপনার উত্তর: **{u_ans}** ({status_icon})")
                                                if not is_c:
                                                    st.markdown(f"- সঠিক উত্তর: **{c_ans}**")
                                                st.markdown("---")
                                    elif "user_written_answer" in parsed_details:
                                        st.markdown("##### 🎨 প্রদত্ত উত্তরপত্র:")
                                        st.write(parsed_details["user_written_answer"])
                                    else:
                                        st.json(parsed_details)
                                except Exception as e:
                                    st.write("বিস্তারিত বিবরণ পাওয়া যায়নি।")

                                if st.button("বন্ধ করুন", key=f"cls_test_m_{rec_id}"):
                                    st.rerun()

                            show_test_detail_modal()

                    with col_hb2:
                        if st.button("🗑️ মুছুন", key=f"btn_del_test_{rec_id}"):
                            conn = sqlite3.connect("smart_school_syllabus.db")
                            cursor = conn.cursor()
                            cursor.execute("DELETE FROM test_history_archive WHERE id=?", (rec_id,))
                            conn.commit()
                            conn.close()
                            st.success("টেস্ট রেকর্ডটি মুছে ফেলা হয়েছে!")
                            st.rerun()
            else:
                st.info("ℹ এখনো নিজেকে যাচাইকরণ সংক্রান্ত কোনো পরীক্ষা দেওয়া হয়নি। 'Study Plan' ট্যাবে গিয়ে প্রতিটি বিষয়ের পাশে '🧪 নিজেকে যাচাই করো' বাটনে ক্লিক করে পরীক্ষা দিন।")
   
import sqlite3
import streamlit as st
import streamlit.components.v1 as components
from datetime import datetime, date

# --- পরীক্ষার ডেটাবেজ ইনিশিয়ালাইজেশন ---
def init_exam_db():
    conn = sqlite3.connect("exam_schedule.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS exams (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            exam_name TEXT,
            exam_type TEXT,
            exam_date TEXT,
            description TEXT
        )
    """)
    conn.commit()
    conn.close()

init_exam_db()

# --- বাংলা ফন্ট ফ্রেন্ডলি HTML/PDF প্রিন্ট টেমপ্লেট জেনারেটর ---
def generate_printable_html(exams_data):
    rows_html = ""
    for idx, row in enumerate(exams_data, start=1):
        e_id, e_name, e_type, e_date, e_desc = row
        rows_html += f"""
        <tr>
            <td style="text-align: center;">{idx}</td>
            <td style="text-align: center;">{e_date}</td>
            <td><b>{e_name}</b></td>
            <td style="text-align: center;">{e_type}</td>
            <td>{e_desc if e_desc else '—'}</td>
        </tr>
        """

    html_content = f"""
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>Exam Routine Summary</title>
        <style>
            @import url('https://fonts.googleapis.com/css2?family=Kalpurush&family=SolaimanLipi&display=swap');
            body {{
                font-family: 'Kalpurush', 'SolaimanLipi', Arial, sans-serif;
                padding: 20px;
                color: #1e293b;
            }}
            .header {{
                text-align: center;
                color: #1E3A8A;
                margin-bottom: 20px;
            }}
            table {{
                width: 100%;
                border-collapse: collapse;
                margin-top: 10px;
            }}
            th, td {{
                border: 1px solid #cbd5e1;
                padding: 10px 12px;
                font-size: 14px;
            }}
            th {{
                background-color: #1E3A8A;
                color: white;
                font-weight: bold;
            }}
            tr:nth-child(even) {{
                background-color: #f8fafc;
            }}
        </style>
    </head>
    <body>
        <div class="header">
            <h2>Virtual Reading Room - পরীক্ষার রুটিন ও সময়সূচী</h2>
        </div>
        <table>
            <thead>
                <tr>
                    <th style="width: 8%;">ক্রমিক</th>
                    <th style="width: 15%;">তারিখ</th>
                    <th style="width: 35%;">পরীক্ষা/অ্যাসাইনমেন্টের নাম</th>
                    <th style="width: 17%;">পরীক্ষার ধরণ</th>
                    <th style="width: 25%;">নোট / সিলেবাস</th>
                </tr>
            </thead>
            <tbody>
                {rows_html}
            </tbody>
        </table>
    </body>
    </html>
    """
    return html_content


# --- পপআপ সামারি টেবিল ডায়ালগ ফাংশন ---
@st.dialog("📊 পরীক্ষার রুটিন ও সামারি টেবিল (এক নজরে)", width="large")
def show_exam_summary_dialog(exams_data):
    st.markdown("""
        <style>
            div[data-testid="stDialog"] div[role="dialog"] {
                width: 90vw !important;
                max-width: 950px !important;
            }
        </style>
    """, unsafe_allow_html=True)

    st.markdown("এখানে তোমার সমস্ত আসন্ন পরীক্ষা ও অ্যাসাইনমেন্টের একটি সম্পূর্ণ তালিকা দেওয়া হলো। এটি সরাসরি প্রিন্ট বা সেভ করে নিতে পারো:")

    table_rows = []
    for idx, row in enumerate(exams_data, start=1):
        e_id, e_name, e_type, e_date, e_desc = row
        table_rows.append({
            "ক্রমিক": idx,
            "তারিখ": e_date,
            "পরীক্ষা/অ্যাসাইনমেন্টের নাম": e_name,
            "পরীক্ষার ধরণ": e_type,
            "নোট / সিলেবাস": e_desc if e_desc else "—"
        })

    st.dataframe(
        table_rows,
        use_container_width=True,
        hide_index=True,
        column_config={
            "ক্রমিক": st.column_config.NumberColumn(width="small"),
            "তারিখ": st.column_config.TextColumn(width="medium"),
            "পরীক্ষা/অ্যাসাইনমেন্টের নাম": st.column_config.TextColumn(width="large"),
            "পরীক্ষার ধরণ": st.column_config.TextColumn(width="medium"),
            "নোট / সিলেবাস": st.column_config.TextColumn(width="large"),
        }
    )

    st.markdown("<br>", unsafe_allow_html=True)
    
    col_d1, col_d2 = st.columns(2)
    with col_d1:
        printable_html = generate_printable_html(exams_data)
        st.download_button(
            label="📄 সেভ / প্রিন্ট করুন (Printable HTML/PDF)",
            data=printable_html,
            file_name="Exam_Routine_Summary.html",
            mime="text/html",
            use_container_width=True,
            type="primary"
        )
    with col_d2:
        if st.button("বন্ধ করো", use_container_width=True):
            st.rerun()


# --- এক্সাম শিডিউল ম্যানেজমেন্ট পেজ রেন্ডার ফাংশন ---
def render_exams():
    st.markdown("""
        <h2 style='color: #1E3A8A; font-weight: 700; margin-bottom: 5px;'>📋 Your Virtual Reading Room-Exam & Assignment Section</h2>
        <p style='color: #475569; font-size: 15px;'> এখানে তোমার আসন্ন পরীক্ষা, কুইজ এবং অ্যাসাইনমেন্টের সময়সূচী সাজিয়ে রাখতে পারবে এবং সামারি আকারে দেখতে পারবে।</p>
    """, unsafe_allow_html=True)

    if "edit_exam_id" not in st.session_state:
        st.session_state["edit_exam_id"] = None

    if st.session_state["edit_exam_id"] is not None:
        conn = sqlite3.connect("exam_schedule.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, exam_name, exam_type, exam_date, description FROM exams WHERE id = ?", (st.session_state["edit_exam_id"],))
        edit_data = cursor.fetchone()
        conn.close()

        if edit_data:
            e_id, cur_name, cur_type, cur_date_str, cur_desc = edit_data
            try:
                parsed_date = datetime.strptime(cur_date_str, "%Y-%m-%d").date()
            except ValueError:
                parsed_date = date.today()

            st.markdown("### ✏️ পরীক্ষার তথ্য সংশোধন করো (Edit Mode)")
            with st.form("edit_exam_form"):
                edit_name = st.text_input("পরীক্ষা বা অ্যাসাইনমেন্টের নাম:", value=cur_name)
                
                types_list = ["বার্ষিক পরীক্ষা", "অর্ধ-বার্ষিক পরীক্ষা", "সাপ্তাহিক কুইজ", "মাসিক পরীক্ষা", "অ্যাসাইনমেন্ট"]
                try:
                    type_idx = types_list.index(cur_type)
                except ValueError:
                    type_idx = 0

                edit_type = st.selectbox("পরীক্ষার ধরন:", types_list, index=type_idx)
                edit_date = st.date_input("পরীক্ষার তারিখ:", value=parsed_date)
                edit_desc = st.text_input("সিলেবাস বা বিশেষ নোট:", value=cur_desc if cur_desc else "")

                col_e1, col_e2 = st.columns(2)
                with col_e1:
                    update_btn = st.form_submit_button("আপডেট করো", use_container_width=True, type="primary")
                with col_e2:
                    cancel_btn = st.form_submit_button("বাতিল করো", use_container_width=True)

                if update_btn:
                    if edit_name:
                        conn = sqlite3.connect("exam_schedule.db")
                        cursor = conn.cursor()
                        cursor.execute("""
                            UPDATE exams SET exam_name = ?, exam_type = ?, exam_date = ?, description = ?
                            WHERE id = ?
                        """, (edit_name, edit_type, str(edit_date), edit_desc, e_id))
                        conn.commit()
                        conn.close()
                        st.session_state["edit_exam_id"] = None
                        st.success("🎉 পরীক্ষার রুটিন সফলভাবে আপডেট করা হয়েছে!")
                        st.rerun()
                    else:
                        st.error("অনুগ্রহ করে পরীক্ষার নামটি প্রদান করো।")

                if cancel_btn:
                    st.session_state["edit_exam_id"] = None
                    st.rerun()
            
            st.markdown("---")

    with st.form("exam_input_form", clear_on_submit=True):
        st.markdown("### ➕ Add Your Exam & Assignment")
        col1, col2 = st.columns(2)

        with col1:
            exam_name = st.text_input("পরীক্ষা বা অ্যাসাইনমেন্টের নাম:", placeholder="যেমন: গণিত প্রথম পত্র পরীক্ষা")
            exam_type = st.selectbox("পরীক্ষার ধরন:", ["বার্ষিক পরীক্ষা", "অর্ধ-বার্ষিক পরীক্ষা", "সাপ্তাহিক কুইজ", "মাসিক পরীক্ষা", "অ্যাসাইনমেন্ট"])

        with col2:
            exam_date = st.date_input("পরীক্ষার তারিখ:", date.today())
            description = st.text_input("সিলেবাস বা বিশেষ নোট (ঐচ্ছিক):", placeholder="যেমন: অধ্যায় ১ থেকে ৩ পর্যন্ত")

        submitted = st.form_submit_button("Save", use_container_width=True, type="primary")

        if submitted:
            if exam_name:
                conn = sqlite3.connect("exam_schedule.db")
                cursor = conn.cursor()
                cursor.execute("""
                    INSERT INTO exams (exam_name, exam_type, exam_date, description)
                    VALUES (?, ?, ?, ?)
                """, (exam_name, exam_type, str(exam_date), description))
                conn.commit()
                conn.close()
                st.success("🎉 তোমার পরীক্ষা বা অ্যাসাইনমেন্টের সময়সূচী সফলভাবে সেভ হয়েছে!")
                st.rerun()
            else:
                st.error("অনুগ্রহ করে অন্তত পরীক্ষার নামটি প্রদান করো।")

    st.markdown("---")

    conn = sqlite3.connect("exam_schedule.db")
    cursor = conn.cursor()
    cursor.execute("SELECT id, exam_name, exam_type, exam_date, description FROM exams ORDER BY exam_date ASC")
    exams_list = cursor.fetchall()
    conn.close()

    if exams_list:
        col_head1, col_head2 = st.columns([2, 1])
        with col_head1:
            st.markdown("### 🗓️ Your Exam Routine & Time")
        with col_head2:
            if st.button("📊 সামারি টেবিল দেখাও..", use_container_width=True, type="primary"):
                show_exam_summary_dialog(exams_list)

        filter_type = st.selectbox("Find Your Exam Routine from below:", ["সকল", "বার্ষিক পরীক্ষা", "অর্ধ-বার্ষিক পরীক্ষা", "সাপ্তাহিক কুইজ", "মাসিক পরীক্ষা", "অ্যাসাইনমেন্ট"])

        for row in exams_list:
            e_id, e_name, e_type, e_date, e_desc = row

            if filter_type == "সকল" or e_type == filter_type:
                st.markdown(f"""
                    <div style="background-color: #ffffff; border: 1px solid #cbd5e1; border-left: 5px solid #1E3A8A; padding: 12px 15px; border-radius: 6px; margin-bottom: 8px;">
                        <h4 style="color: #1E3A8A; margin: 0 0 5px 0;">📌 {e_name}</h4>
                        <p style="margin: 0; color: #475569; font-size: 14px;">
                            <b>ধরন:</b> {e_type} | <b>তারিখ:</b> {e_date}
                        </p>
                        <p style="margin: 5px 0 0 0; color: #64748b; font-size: 13px;">
                            <b>নোট:</b> {e_desc if e_desc else "কোনো অতিরিক্ত নোট নেই"}
                        </p>
                    </div>
                """, unsafe_allow_html=True)

                col_btn1, col_btn2 = st.columns(2)
                with col_btn1:
                    if st.button("✏️ এডিট করো", key=f"edit_exam_{e_id}", use_container_width=True):
                        st.session_state["edit_exam_id"] = e_id
                        st.rerun()
                with col_btn2:
                    if st.button("🗑️ ডিলিট করো", key=f"del_exam_{e_id}", use_container_width=True):
                        conn = sqlite3.connect("exam_schedule.db")
                        cursor = conn.cursor()
                        cursor.execute("DELETE FROM exams WHERE id = ?", (e_id,))
                        conn.commit()
                        conn.close()
                        st.success("সময়সূচী থেকে এটি মুছে ফেলা হয়েছে!")
                        st.rerun()
                
                st.markdown("<div style='margin-bottom: 15px;'></div>", unsafe_allow_html=True)
    else:
        st.info("এখনো কোনো পরীক্ষা বা অ্যাসাইনমেন্টের রুটিন যুক্ত করা হয়নি। উপরে ফর্ম থেকে নতুন রুটিন যোগ করো।")


import sqlite3
import streamlit as st
from datetime import datetime, date


def render_notifications():
    st.markdown("""
        <h2 style='color: #1E3A8A; font-weight: 700; margin-bottom: 5px;'>🔔 স্মার্ট নোটিফিকেশন ও এআই মেন্টর গাইড</h2>
        <p style='color: #475569; font-size: 15px;'>তোমার আসন্ন পরীক্ষার প্রস্তুতি, রিমাইন্ডার এবং পড়াশোনার দিকনির্দেশনা এক নজরে দেখে নাও।</p>
    """, unsafe_allow_html=True)

    # ডেটাবেজ থেকে আসন্ন পরীক্ষাগুলোর ডেটা ফেচ করা
    try:
        conn = sqlite3.connect("exam_schedule.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, exam_name, exam_type, exam_date, description FROM exams ORDER BY exam_date ASC")
        exams_list = cursor.fetchall()
        conn.close()
    except sqlite3.OperationalError:
        exams_list = []

    today = date.today()

    # --- ১. শিক্ষকসুলভ এআই মেন্টর ওয়েলকাম ও রিমাইন্ডার কার্ড ---
    st.markdown("""
        <div style="background: linear-gradient(135deg, #1e3a8a 0%, #3b82f6 100%); padding: 20px; border-radius: 12px; color: white; margin-bottom: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
            <h3 style="margin: 0 0 8px 0; color: #ffffff;">👋 হ্যালো! আমি তোমার এআই স্টাডি গাইড ও মেন্টর</h3>
            <p style="margin: 0; font-size: 15px; line-height: 1.6;">
                পড়াশোনা কেমন চলছে? তোমার লক্ষ্য অর্জনে আমি সবসময় তোমার পাশে আছি। নিচে তোমার পরীক্ষার সময়সূচী এবং প্রস্তুতির বর্তমান অবস্থা বিশ্লেষণ করে গুরুত্বপূর্ণ গাইডলাইন দেওয়া হলো। নিয়মিত রিভিশন দাও এবং প্র্যাকটিস করো!
            </p>
        </div>
    """, unsafe_allow_html=True)

    st.markdown("### ⏰ পরীক্ষার লাইভ কাউন্টডাউন ও রিমাইন্ডার")

    if exams_list:
        upcoming_found = False
        for row in exams_list:
            e_id, e_name, e_type, e_date_str, e_desc = row
            try:
                exam_date = datetime.strptime(e_date_str, "%Y-%m-%d").date()
                delta_days = (exam_date - today).days

                if delta_days >= 0:
                    upcoming_found = True
                    # ডায়নামিক কালার ও ম্যাসেজ সেট করা
                    if delta_days == 0:
                        urgency_color = "#991b1b" # লাল (আজকেই পরীক্ষা)
                        time_msg = "⚠️ আজকেই তোমার পরীক্ষা! শেষ মুহূর্তের রিভিশন দ্রুত শেষ করো।"
                    elif delta_days <= 3:
                        urgency_color = "#c2410c" # কমলা (খুব কাছে)
                        time_msg = f"🔥 আর মাত্র **{delta_days} দিন** বাকি! প্রস্তুতি জোরদার করো।"
                    else:
                        urgency_color = "#1e3a8a" # নীল
                        time_msg = f"⏳ পরীক্ষার বাকি আছে **{delta_days} দিন**। পরিকল্পনা অনুযায়ী পড়াশোনা চালিয়ে যাও।"

                    st.markdown(f"""
                        <div style="background-color: #ffffff; border: 1px solid #cbd5e1; border-left: 6px solid {urgency_color}; padding: 15px; border-radius: 8px; margin-bottom: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.02);">
                            <h4 style="color: #1E3A8A; margin: 0 0 5px 0;">📌 {e_name} ({e_type})</h4>
                            <p style="margin: 0 0 5px 0; color: #334155; font-size: 14px;">
                                <b>তারিখ:</b> {e_date_str} | <b>সিলেবাস/নোট:</b> {e_desc if e_desc else "নির্দিষ্ট সিলেবাস নেই"}
                            </p>
                            <p style="margin: 5px 0 0 0; color: {urgency_color}; font-size: 14px; font-weight: 600;">
                                {time_msg}
                            </p>
                        </div>
                    """, unsafe_allow_html=True)
            except ValueError:
                continue
        
        if not upcoming_found:
            st.info("🎉 তোমার কোনো আসন্ন পরীক্ষা নেই। নতুন পরীক্ষার রুটিন শিডিউল অপশন থেকে যুক্ত করতে পারো।")
    else:
        st.info("📌 এখনো কোনো পরীক্ষার রুটিন যোগ করা হয়নি। এক্সাম শিডিউল অপশন থেকে রুটিন যুক্ত করলে এখানে লাইভ কাউন্টডাউন দেখতে পাবে।")

    st.markdown("---")

    # --- ২. সিলেবাস প্রোগ্রেস ও মেন্টরের পরামর্শ ---
    st.markdown("### 📊 মেন্টর এনালিসিস ও প্রিপারেশন গাইডলাইন")
    
    col_m1, col_m2 = st.columns(2)
    with col_m1:
        st.markdown("""
            <div style="background-color: #f8fafc; border: 1px solid #e2e8f0; padding: 15px; border-radius: 8px;">
                <p style="color: #1E3A8A; font-weight: bold; margin-bottom: 8px;">📖 দৈনিক স্টাডি গোল ও টার্গেট</p>
                <p style="color: #475569; font-size: 13.5px; line-height: 1.5; margin: 0;">
                    • প্রতিদিন অন্তত <b>৩ থেকে ৪ ঘণ্টা</b> ফোকাসড স্টাডি করো।<br>
                    • কঠিন বিষয়গুলো সকালের দিকে পড়ার চেষ্টা করো।<br>
                    • প্রতি ১ ঘণ্টা পড়ার পর ১০ মিনিট বিশ্রাম নাও।
                </p>
            </div>
        """, unsafe_allow_html=True)

    with col_m2:
        st.markdown("""
            <div style="background-color: #f8fafc; border: 1px solid #e2e8f0; padding: 15px; border-radius: 8px;">
                <p style="color: #1E3A8A; font-weight: bold; margin-bottom: 8px;">🎯 শিক্ষকসুলভ পরামর্শ ও প্র্যাকটিস</p>
                <p style="color: #475569; font-size: 13.5px; line-height: 1.5; margin: 0;">
                    • শুধু পড়ালেখা করলেই হবে না, যা পড়েছ তা খাতায় নোট করো বা লিখে প্র্যাকটিস করো।<br>
                    • প্রস্তুতি যাচাই করতে নিজে নিজে মক টেস্ট বা কুইজ দাও।<br>
                    • কোনো টপিক বুঝতে সমস্যা হলে নোট খাতা বা এআই অ্যাসিস্ট্যান্টের সাহায্য নাও।
                </p>
            </div>
        """, unsafe_allow_html=True)

    # --- ৩. শিক্ষক ও সিস্টেমের সাম্প্রতিক নোটিশ বোর্ড ---
    st.markdown("---")
    st.markdown("### 📢 শিক্ষক ও সিস্টেম অ্যানাউন্সমেন্ট")
    st.markdown("- **[৫ মিনিট আগে]** এআই মেন্টর: আসন্ন গণিত ও বিজ্ঞান পরীক্ষার জন্য রিভিশন শিট দ্রুত সম্পন্ন করো।")
    st.markdown("- **[২ ঘণ্টা আগে]** নোটিশ: পাঠাগারে নতুন রেফারেন্স বুক ও মডেল টেস্ট পেপার যুক্ত করা হয়েছে।")
    st.markdown("- **[১ দিন আগে]** সিস্টেম আপডেট: নোট এডিটর থেকে সরাসরি পিডিএফ ও ওয়ার্ড ফাইল ডাউনলোডের ফিচার সক্রিয় রয়েছে।")

def render_settings():
    st.markdown("## ⚙️ Profile & Settings")
    with st.form("settings_form"):
        st.text_input("Student Name", value=st.session_state.student_name)
        st.text_input("School Name", value=st.session_state.school_name)
        st.selectbox("Class", ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"], index=1)
        st.text_input("Roll Number", value=st.session_state.roll_number)
        submitted = st.form_submit_button("Update Profile")
        if submitted:
            st.success("Profile updated successfully!")

if not st.session_state.authenticated:
    render_login_page()
else:
    with st.sidebar:
        st.markdown(f"""
        <div style="text-align: center; margin-bottom: 20px;">
            <h3 style="color: white; margin: 0; font-size: 18px;">Virtual Reading Room</h3>
            <p style="color: #94A3B8; font-size: 11px;">{st.session_state.school_name}</p>
        </div>
        """, unsafe_allow_html=True)
        
        nav_options = [
            ("📊 Dashboard", "Dashboard"),
            ("📚 Books & Reading", "My Books"),
            ("🎯 Practice & Preparation", "Quiz & Practice"),
            ("📝 NoteBook", "Notes"),
            ("📅 Smart Study & Syllabus", "Study Planner"),
            ("📋 Exams & Assignment", "Exam & Assignment"),
            ("🔔 Reminder", "Notifications"),
            ("⚙️ Profile & Settings", "Profile / Settings")
        ]
        
        for label, page_key in nav_options:
            if st.button(label, use_container_width=True):
                st.session_state.current_page = page_key
                st.rerun()

        st.markdown("---")
        st.markdown("""
        <div style="background: rgba(255,255,255,0.08); padding: 14px; border-radius: 12px; font-size: 11px; color: #E2E8F0; text-align: center;">
            <div style="font-size: 20px; margin-bottom: 4px;">🤖</div>
            <b>Your AI Study Partner</b><br>Ask, Learn, Grow!
            <p style="font-style: italic; margin-top: 8px; color: #CBD5E1;">"Small steps every day lead to big results!"</p>
        </div>
        """, unsafe_allow_html=True)
        
        st.markdown("<br>", unsafe_allow_html=True)
        if st.button("🚪 Logout", use_container_width=True):
            st.session_state.authenticated = False
            st.rerun()

    # Page Routing Switcher
    page = st.session_state.current_page
    if page == "Dashboard":
        render_dashboard()
    elif page == "My Books":
        render_my_books()
    elif page == "AI Assistant":
        render_ai_assistant_page()
    elif page == "Quiz & Practice":
        render_quiz_practice()
    elif page == "Notes":
        render_notes()
    elif page == "Progress":
        render_progress()
    elif page == "Study Planner":
        render_study_planner()
    elif page == "Exam & Assignment":
        render_exams()
    elif page == "Notifications":
        render_notifications()
    elif page == "Profile / Settings":
        render_settings()


