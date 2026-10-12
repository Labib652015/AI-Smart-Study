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

import os
import requests
import base64
import streamlit as st

# পিডিএফ ফাইল মেমোরিতে ফাস্ট লোড ও হ্যাং বন্ধ করার জন্য ক্যাশিং ফাংশন
@st.cache_data
def load_pdf_as_base64(file_path):
    with open(file_path, "rb") as f:
        return base64.b64encode(f.read()).decode('utf-8')

def render_my_books():
    # ১. Gemini API Key (যদি ইন-অ্যাপ চ্যাট ব্যবহার করতে চান)
    GEMINI_API_KEY = "YOUR_GEMINI_API_KEY_HERE"
    
    def ask_gemini_rest_api(prompt_text):
        if not GEMINI_API_KEY or GEMINI_API_KEY == "YOUR_GEMINI_API_KEY_HERE":
            return "⚠️ API Key সমস্যা হচ্ছে। দয়া করে 'Gemini Chatbot' বাটনে ক্লিক করো।"
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
        return "⚠️ API Key সমস্যা হচ্ছে। দয়া করে 'Gemini Chatbot' বাটনে ক্লিক করো।"

    # ২. বাচ্চাদের জন্য আকর্ষণীয়, কালারফুল ও প্রিমিয়াম CSS স্টাইল (আলাদা ফন্ট ও নতুন মাত্রা)
    st.markdown("""
        <style>
        @import url('https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;500;600;700&family=Outfit:wght@400;600;800&display=swap');

        .kids-magic-wrapper {
            font-family: 'Outfit', 'Hind Siliguri', sans-serif;
            background: linear-gradient(135deg, #f0fdf4 0%, #eff6ff 50%, #fdf4ff 100%);
            padding: 30px;
            border-radius: 28px;
            border: 2px solid #bae6fd;
            box-shadow: 0 20px 40px rgba(14, 165, 233, 0.1);
            color: #1e293b;
            margin-bottom: 25px;
        }

        .kids-banner {
            background: linear-gradient(135deg, #38bdf8 0%, #818cf8 50%, #c084fc 100%);
            padding: 22px;
            border-radius: 20px;
            color: white;
            text-align: center;
            margin-bottom: 25px;
            box-shadow: 0 10px 25px rgba(56, 189, 248, 0.3);
        }

        .motivation-quote-box {
            background: #ffffff;
            padding: 16px 20px;
            border-radius: 16px;
            border-left: 6px solid #10b981;
            border-top: 1px solid #e2e8f0;
            border-right: 1px solid #e2e8f0;
            border-bottom: 1px solid #e2e8f0;
            margin-bottom: 25px;
            box-shadow: 0 8px 20px rgba(16, 185, 129, 0.08);
        }

        .kids-book-card {
            background: #ffffff;
            padding: 20px;
            border-radius: 18px;
            border: 1px solid #e0f2fe;
            border-top: 5px solid #0ea5e9;
            box-shadow: 0 10px 25px rgba(14, 165, 233, 0.08);
            margin-bottom: 16px;
            transition: all 0.3s ease;
        }

        .kids-book-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 30px rgba(14, 165, 233, 0.2);
            border-color: #38bdf8;
        }

        /* আকর্ষণীয় ও কালারফুল বাটন ডিজাইন */
        .stButton button {
            font-family: 'Outfit', 'Hind Siliguri', sans-serif !important;
            border-radius: 12px !important;
            font-weight: 700 !important;
            background: linear-gradient(135deg, #0ea5e9 0%, #2563eb 100%) !important;
            color: white !important;
            border: none !important;
            box-shadow: 0 6px 15px rgba(14, 165, 233, 0.3) !important;
            transition: all 0.3s ease !important;
        }

        .stButton button:hover {
            background: linear-gradient(135deg, #0284c7 0%, #1d4ed8 100%) !important;
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(14, 165, 233, 0.4) !important;
        }
        </style>
    """, unsafe_allow_html=True)

    # মূল কালারফুল কন্টেইনার শুরু
    st.markdown("<div class='kids-magic-wrapper'>", unsafe_allow_html=True)
    
    # আকর্ষণীয় ব্যানার ও লোগো
    st.markdown("""
        <div class="kids-banner">
            <h1 style="margin: 0; font-size: 28px; font-weight: 800; font-family: 'Outfit', sans-serif;">🌟 ভার্চুয়াল রিডিং রুম - পড়ার জাদুকরী দুনিয়া 🌟</h1>
            <p style="margin: 5px 0 0 0; font-size: 14px; font-weight: 500; opacity: 0.95;">ষষ্ঠ থেকে দশম শ্রেণির সোনামনিদের স্মার্ট পড়াশোনা, জ্ঞানার্জন ও চরিত্র গঠনের ঘর</p>
        </div>
    """, unsafe_allow_html=True)

    # আজকের নীতিকথা ও অনুপ্রেরণামূলক বার্তা (বাচ্চাদের উৎসাহ বাড়াতে)
    st.markdown("""
        <div class="motivation-quote-box">
            <div style="display: flex; align-items: center; gap: 12px;">
                <span style="font-size: 28px;">💡</span>
                <div>
                    <h4 style="margin: 0; color: #047857; font-size: 15px; font-weight: 700;">আজকের অনুপ্রেরণা ও নীতিকথা:</h4>
                    <p style="margin: 4px 0 0 0; color: #334155; font-size: 13px; font-weight: 500;">
                        "আজকের অর্জিত জ্ঞান ও ভালো অভ্যাসই আগামী দিনে তোমাকে একজন আদর্শ দেশপ্রেমিক ও সফল মানুষ হিসেবে গড়ে তুলবে। নিয়মিত বই পড়ো, দেশকে ভালোবাসো!" 🇧🇩✨
                    </p>
                </div>
            </div>
        </div>
    """, unsafe_allow_html=True)

    # ৩. সেশন স্টেট ইনিশিয়ালাইজেশন[cite: 18]
    if "active_pdf_path" not in st.session_state:
        st.session_state.active_pdf_path = None
    if "book_chat_messages" not in st.session_state:
        st.session_state.book_chat_messages = [
            {"role": "assistant", "content": "✨ হ্যালো বন্ধু! বইয়ের কোনো লাইন বুঝতে সমস্যা হলে আমাকে জানাও, আমি বুঝিয়ে দেব।"}
        ]
    if "rename_file" not in st.session_state:
        st.session_state.rename_file = None

    # ৪. ফাইল সেভ করার ফোল্ডার সেটআপ[cite: 18]
    current_dir = os.path.dirname(os.path.abspath(__file__)) if "__file__" in locals() else os.getcwd()
    assets_dir = os.path.join(current_dir, "assets")
    if not os.path.exists(assets_dir):
        os.makedirs(assets_dir)

    # ৫. ক্যাটাগরি সিলেকশন[cite: 18]
    col_cat, col_up = st.columns([2.5, 1.5], gap="large")
    with col_cat:
        book_category = st.radio(
            "Select Category",
            ["📖 এনসিটিবি পাঠ্যবই (NCTB)", "📘 সহায়ক গাইডবুক", "📂 অন্যান্য বই"],
            horizontal=True
        )
    with col_up:
        if "সহায়ক গাইডবুক" in book_category:
            uploaded_guide = st.file_uploader("Upload Guide Book PDF", type=["pdf"], key="upload_guide_file")
            if uploaded_guide is not None:
                save_name = f"guide_{uploaded_guide.name}"
                save_path = os.path.join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_guide.getbuffer())
                st.success(f"🎉 গাইড বই সফলভাবে যুক্ত হয়েছে: {uploaded_guide.name}")
                st.rerun()
        elif "অন্যান্য বই" in book_category:
            uploaded_other = st.file_uploader("Upload Other Book PDF", type=["pdf"], key="upload_other_file")
            if uploaded_other is not None:
                save_name = f"other_{uploaded_other.name}"
                save_path = os.path.join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_other.getbuffer())
                st.success(f"🎉 বই সফলভাবে যুক্ত হয়েছে: {uploaded_other.name}")
                st.rerun()

    st.markdown("<hr style='margin: 25px 0; border: 0; border-top: 2px dashed #bae6fd;'>", unsafe_allow_html=True)

    # ৬. ক্যাটাগরি অনুযায়ী বই ফিল্টার ও প্রদর্শন লজিক[cite: 19, 20, 21, 22]
    if "এনসিটিবি পাঠ্যবই" in book_category:
        st.markdown("<h3 style='color: #0284c7; font-size: 18px; font-weight: 700;'>🎯 তোমার শ্রেণি নির্বাচন করো (NCTB Curriculum)</h3>", unsafe_allow_html=True)
        selected_class = st.selectbox(
            "Choose Class",
            ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"],
            index=1
        )
        cls_prefix = selected_class.lower().replace(" ", "")
        
        try:
            all_files = os.listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and cls_prefix in f.lower()]
        except:
            pdf_files = []

        st.markdown(f"<h4 style='color: #334155; font-size: 15px; margin-top: 15px;'>📚 {selected_class} এর উপলব্ধ বইসমূহ:</h4>", unsafe_allow_html=True)
        
        if pdf_files:
            grid_cols = st.columns(3, gap="medium")
            for idx, file_name in enumerate(pdf_files):
                file_path = os.path.join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                clean_title = file_name.replace(cls_prefix, '').replace('.pdf', '').replace('.PDF', '').replace('_', ' ').strip()
                if not clean_title:
                    clean_title = file_name.replace('.pdf', '')
                
                with col:
                    st.markdown(f"""
                        <div class="kids-book-card">
                            <div style="font-size: 28px; margin-bottom: 8px;">📖</div>
                            <h4 style="color: #0f172a; margin: 0 0 10px 0; font-size: 14px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{selected_class} - {clean_title}</h4>
                        </div>
                    """, unsafe_allow_html=True)
                    if st.button("🚀 বই পড়ো (Read)", key=f"read_nctb_{cls_prefix}_{idx}", use_container_width=True):
                        st.session_state.active_pdf_path = file_path
                        st.rerun()
        else:
            st.info(f"ℹ️ '{selected_class}' এর জন্য ফোল্ডারে কোনো বই পাওয়া যায়নি। উপরে পাঠ্যবই যুক্ত করুন।")

    elif "সহায়ক গাইডবুক" in book_category:
        try:
            all_files = os.listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('guide_')]
        except:
            pdf_files = []

        if st.session_state.rename_file:
            old_file = st.session_state.rename_file
            st.warning(f"⚠️ ফাইলের নাম পরিবর্তন হচ্ছে: {old_file}")
            new_name = st.text_input("নতুন নাম লিখুন (.pdf সহ):", value=old_file)
            c_save, c_cancel = st.columns([1, 1], gap="small")
            with c_save:
                if st.button("💾 সংরক্ষণ"):
                    if new_name and new_name != old_file:
                        os.rename(os.path.join(assets_dir, old_file), os.path.join(assets_dir, new_name))
                        st.session_state.rename_file = None
                        st.success("✅ নাম সফলভাবে পরিবর্তন হয়েছে!")
                        st.rerun()
            with c_cancel:
                if st.button("❌ বাতিল"):
                    st.session_state.rename_file = None
                    st.rerun()

        st.markdown("<h4 style='color: #334155; font-size: 15px;'>📘 আপলোডকৃত সহায়ক গাইডবুকসমূহ</h4>", unsafe_allow_html=True)
        if pdf_files:
            grid_cols = st.columns(3, gap="medium")
            for idx, file_name in enumerate(pdf_files):
                file_path = os.path.join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace('guide_', '').replace('.pdf', '')
                with col:
                    st.markdown(f"""
                        <div class="kids-book-card">
                            <div style="font-size: 28px; margin-bottom: 8px;">📘</div>
                            <h4 style="color: #0f172a; margin: 0 0 10px 0; font-size: 14px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{display_name}</h4>
                        </div>
                    """, unsafe_allow_html=True)
                    b1, b2, b3 = st.columns([1.2, 1, 1], gap="small")
                    with b1:
                        if st.button("📖 পড়ো", key=f"read_guide_{idx}", use_container_width=True):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("✏️ নাম", key=f"ren_guide_{idx}", use_container_width=True):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("🗑️ ডিলিট", key=f"del_guide_{idx}", use_container_width=True):
                            os.remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("✅ মুছে ফেলা হয়েছে!")
                            st.rerun()
        else:
            st.info("ℹ️ কোনো গাইড বই আপলোড করা হয়নি। উপরে 'Upload Guide Book PDF' থেকে আপলোড করুন।")

    else:
        try:
            all_files = os.listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('other_')]
        except:
            pdf_files = []

        if st.session_state.rename_file:
            old_file = st.session_state.rename_file
            st.warning(f"⚠️ ফাইলের নাম পরিবর্তন হচ্ছে: {old_file}")
            new_name = st.text_input("নতুন নাম লিখুন (.pdf সহ):", value=old_file)
            c_save, c_cancel = st.columns([1, 1], gap="small")
            with c_save:
                if st.button("💾 সংরক্ষণ"):
                    if new_name and new_name != old_file:
                        os.rename(os.path.join(assets_dir, old_file), os.path.join(assets_dir, new_name))
                        st.session_state.rename_file = None
                        st.success("✅ নাম সফলভাবে পরিবর্তন হয়েছে!")
                        st.rerun()
            with c_cancel:
                if st.button("❌ বাতিল"):
                    st.session_state.rename_file = None
                    st.rerun()

        st.markdown("<h4 style='color: #334155; font-size: 15px;'>📂 আপলোডকৃত অন্যান্য বইসমূহ</h4>", unsafe_allow_html=True)
        if pdf_files:
            grid_cols = st.columns(3, gap="medium")
            for idx, file_name in enumerate(pdf_files):
                file_path = os.path.join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace('other_', '').replace('.pdf', '')
                with col:
                    st.markdown(f"""
                        <div class="kids-book-card">
                            <div style="font-size: 28px; margin-bottom: 8px;">📂</div>
                            <h4 style="color: #0f172a; margin: 0 0 10px 0; font-size: 14px; font-weight: 700; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{display_name}</h4>
                        </div>
                    """, unsafe_allow_html=True)
                    b1, b2, b3 = st.columns([1.2, 1, 1], gap="small")
                    with b1:
                        if st.button("📖 পড়ো", key=f"read_other_{idx}", use_container_width=True):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("✏️ নাম", key=f"ren_other_{idx}", use_container_width=True):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("🗑️ ডিলিট", key=f"del_other_{idx}", use_container_width=True):
                            os.remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("✅ মুছে ফেলা হয়েছে!")
                            st.rerun()
        else:
            st.info("ℹ️ কোনো অন্যান্য বই আপলোড করা হয়নি। উপরে 'Upload Other Book PDF' থেকে আপলোড করুন।")

    st.markdown("</div>", unsafe_allow_html=True)

    # ৭. ফুল-ফিচার্ড পিডিএফ রিডার উইন্ডো এবং ডানপাশে Gemini AI[cite: 23, 24, 25]
    if st.session_state.active_pdf_path and os.path.exists(st.session_state.active_pdf_path):
        st.markdown("<hr style='margin: 30px 0; border: 0; border-top: 2px solid #0ea5e9;'>", unsafe_allow_html=True)
        c_title, c_close = st.columns([4, 1], gap="small")
        with c_title:
            active_file_name = os.path.basename(st.session_state.active_pdf_path)
            st.markdown(f"<h3 style='color: #0284c7; margin: 0; font-family: Outfit;'>📖 পড়ার কক্ষ - {active_file_name}</h3>", unsafe_allow_html=True)
        with c_close:
            if st.button("❌ রিডার বন্ধ করো", use_container_width=True):
                st.session_state.active_pdf_path = None
                st.rerun()

        pdf_col, ai_col = st.columns([2.3, 1.1], gap="large")
        
        with pdf_col:
            try:
                base64_pdf = load_pdf_as_base64(st.session_state.active_pdf_path)
                pdf_embed_html = f'''
                    <iframe
                    src="data:application/pdf;base64,{base64_pdf}#toolbar=1&navpanes=1"
                    width="100%"
                    height="780px"
                    type="application/pdf"
                    style="border: 2px solid #bae6fd; border-radius: 18px; background-color: #f8fafc; box-shadow: 0 15px 35px rgba(0,0,0,0.1);"
                    >
                    <p>আপনার ব্রাউজার PDF প্রদর্শন করতে পারছে না।</p>
                    </iframe>
                '''
                st.markdown(pdf_embed_html, unsafe_allow_html=True)
            except Exception as e:
                st.error(f"Error loading PDF: {e}")

        with ai_col:
            st.markdown("""
                <div style="background: #ffffff; padding: 18px; border-radius: 18px; border: 2px solid #bae6fd; margin-bottom: 15px; box-shadow: 0 10px 25px rgba(14,165,233,0.08);">
                    <h4 style="color: #0284c7; margin-top: 0; margin-bottom: 6px; font-family: Outfit; font-size: 16px;">🤖 এআই স্টাডি অ্যাসিস্ট্যান্ট</h4>
                    <p style="color: #64748b; font-size: 12px; margin-bottom: 0px;">বইয়ের যেকোনো কঠিন লাইন কপি করে এখানে বা জেমিনি চ্যাটবটে পেস্ট করো:</p>
                </div>
            """, unsafe_allow_html=True)
            
            st.link_button(
                "🚀 জেমিনি চ্যাটবট ওপেন করো",
                "https://gemini.google.com",
                use_container_width=True,
                type="primary"
            )
            st.markdown("<p style='text-align: center; color: #64748b; font-size: 11px; margin-top: 6px;'>টেক্সট কপি করার পর বাটনে ক্লিক করে পেস্ট করুন</p>", unsafe_allow_html=True)
            
            st.markdown("<hr style='margin: 15px 0; border-color: #e2e8f0;'>", unsafe_allow_html=True)
            
            chat_container = st.container(height=320)
            with chat_container:
                for msg in st.session_state.book_chat_messages:
                    if msg["role"] == "user":
                        st.chat_message("user").write(msg["content"])
                    else:
                        st.chat_message("assistant").write(msg["content"])
            
            with st.form(key="ai_chat_form", clear_on_submit=True):
                user_text = st.text_area(
                    "প্রশ্ন বা টেক্সট পেস্ট করো:",
                    placeholder="এখানে তোমার প্রশ্ন লিখ বা পেস্ট কর...",
                    height=70
                )
                submit_button = st.form_submit_button("✨ এআই-এর সাহায্য নাও", use_container_width=True)
                if submit_button and user_text.strip():
                    st.session_state.book_chat_messages.append({"role": "user", "content": user_text})
                    with st.spinner("উত্তর তৈরি হচ্ছে..."):
                        ai_reply = ask_gemini_rest_api(user_text)
                    st.session_state.book_chat_messages.append({"role": "assistant", "content": ai_reply})
                    st.rerun()


# Practice & Prepation Option এর কোডগুলো এখান থেকে শুরু...

import streamlit as st
import random
import time
import json
import sqlite3
from datetime import datetime

def render_quiz_practice():


    # .....নিচের কোডগুলো আধুনিক, আকর্ষণীয় ও প্রিমিয়াম কাস্টম স্টাইল, কালার ও ডিজাইন এর....
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
            border-top: 1px solid #f1f5f9;
            border-right: 1px solid #f1f5f9;
            border-bottom: 1px solid #f1f5f9;
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
            border-top: 1px solid #f1f5f9;
            border-right: 1px solid #f1f5f9;
            border-bottom: 1px solid #f1f5f9;
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

# .....উপর থেকে এই পর্যন্ত কোডগুলো আধুনিক, আকর্ষণীয় ও প্রিমিয়াম কাস্টম স্টাইল, কালার ও ডিজাইন এর....

    st.markdown("<div class='practice-magic-wrapper'>", unsafe_allow_html=True)
    st.markdown(
        """
        <div class="practice-banner">
            <h1 style="margin: 0; font-size: 28px; font-weight: 800; letter-spacing: 0.5px;">Smart Practice & Model Test Hub</h1>
            <p style="margin: 8px 0 0 0; font-size: 15px; font-weight: 500; opacity: 0.95;">
                ৬ষ্ঠ থেকে ১০ম শ্রেণির স্মার্ট প্র্যাকটিস, কুইজ টেস্ট এবং লাইভ মডেল টেস্ট প্ল্যাটফর্ম!
            </p>
        </div>
        """,
        unsafe_allow_html=True
    )

    # সেশন স্টেট ইনিশিয়ালাইজেশন
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
                    "কাবুলিওয়ালা": {
                        "MCQ": [
                            {"q": "'কাবুলিওয়ালা' গল্পে কাবুলিওয়ালার আসল নাম কী ছিল?", "options": ["ক) রহমত", "খ) ছোট খোকা", "গ) নিয়ামত", "ঘ) মিনু"], "answer": "ক) রহমত"},
                            {"q": "কাবুলিওয়ালা গল্পের কথক পেশায় কী ছিলেন?", "options": ["ক) শিক্ষক", "খ) ব্যবসায়ী", "গ) লেখক/উপন্যাসিক", "ঘ) চিকিৎসক"], "answer": "গ) লেখক/উপন্যাসিক"}
                        ],
                        "CQ": [
                            {
                                "stem": "উদ্দীপক: জীবিকার তাগিদে সুদূর পার্বত্য অঞ্চল থেকে সমতলের শহরে আসেন এক হকার। শহরের ব্যস্ততার মাঝেও পাশের বাড়ির একটি ছোট্ট খুকিকে দেখলেই তার নিজের ছোট মেয়ের কথা মনে পড়ে যায়।",
                                "ka": "কাবুলিওয়ালা গল্পের মিনুর বয়স কত ছিল?",
                                "kha": "কাবুলিওয়ালা কেন জেলে গিয়েছিল?",
                                "ga": "উদ্দীপকের হকারের সাথে 'কাবুলিওয়ালা' গল্পের রহমতের সাদৃশ্য কোথায়—ব্যাখ্যা কর।",
                                "gha": "\"পিতৃহৃদয়ের টানে দূর পরবাসেও হৃদয়ের সম্পর্ক গড়ে ওঠে।\"—'কাবুলিওয়ালা' গল্পের আলোকে মূল্যায়ন কর।"
                            }
                        ],
                        "Written": [
                            {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "'মিনু ও রহমতের বন্ধুত্ব বয়স ও দেশের সীমানা ছাড়িয়ে গিয়েছিল'—ব্যাখ্যা কর।"},
                            {"title": "সারাংশ লিখন", "desc": "দাঁড়াও তবে খাটো করে নয়... (পাঠ্যবইয়ের সংশ্লিষ্ট সারাংশ অনুশীলন করুন)।"}
                        ]
                    }
                }
            },
            "বাংলা দ্বিতীয় পত্র": {
                            "গদ্যাংশ": {
                                "কাবুলিওয়ালা": {
                                    "MCQ": [
                                        {"q": "'কাবুলিওয়ালা' গল্পে কাবুলিওয়ালার আসল নাম কী ছিল?", "options": ["ক) রহমত", "খ) ছোট খোকা", "গ) নিয়ামত", "ঘ) মিনু"], "answer": "ক) রহমত"},
                                        {"q": "কাবুলিওয়ালা গল্পের কথক পেশায় কী ছিলেন?", "options": ["ক) শিক্ষক", "খ) ব্যবসায়ী", "গ) লেখক/উপন্যাসিক", "ঘ) চিকিৎসক"], "answer": "গ) লেখক/উপন্যাসিক"}
                                    ],
                                    "CQ": [
                                        {
                                            "stem": "উদ্দীপক: জীবিকার তাগিদে সুদূর পার্বত্য অঞ্চল থেকে সমতলের শহরে আসেন এক হকার। শহরের ব্যস্ততার মাঝেও পাশের বাড়ির একটি ছোট্ট খুকিকে দেখলেই তার নিজের ছোট মেয়ের কথা মনে পড়ে যায়।",
                                            "ka": "কাবুলিওয়ালা গল্পের মিনুর বয়স কত ছিল?",
                                            "kha": "কাবুলিওয়ালা কেন জেলে গিয়েছিল?",
                                            "ga": "উদ্দীপকের হকারের সাথে 'কাবুলিওয়ালা' গল্পের রহমতের সাদৃশ্য কোথায়—ব্যাখ্যা কর।",
                                            "gha": "\"পিতৃহৃদয়ের টানে দূর পরবাসেও হৃদয়ের সম্পর্ক গড়ে ওঠে।\"—'কাবুলিওয়ালা' গল্পের আলোকে মূল্যায়ন কর।"
                                        }
                                    ],
                                    "Written": [
                                        {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "'মিনু ও রহমতের বন্ধুত্ব বয়স ও দেশের সীমানা ছাড়িয়ে গিয়েছিল'—ব্যাখ্যা কর।"},
                                        {"title": "সারাংশ লিখন", "desc": "দাঁড়াও তবে খাটো করে নয়... (পাঠ্যবইয়ের সংশ্লিষ্ট সারাংশ অনুশীলন করুন)।"}
                                    ]
                                }
                            }
                        },
                        "গণিত": {
        "পাটিগণিত": {
            "স্বাভাবিক সংখ্যা ও ভগ্নাংশ": {
                "MCQ": [
                    {"q": "সবচেয়ে ছোট মৌলিক সংখ্যা কোনটি?", "options": ["ক) ১", "খ) ২", "গ) ৩", "ঘ) ৪"], "answer": "খ) ২"},
                    {"q": "নিচের কোনটি মৌলিক সংখ্যা?", "options": ["ক) ৯", "খ) ১১", "গ) ১৫", "ঘ) ২১"], "answer": "খ) ১১"}
                ],
                "CQ": [
                    {
                        "stem": "উদ্দীপক: ২৪ এবং ৩৬ দুটি সংখ্যা। সংখ্যা দুটির সাধারণ উৎপাদক এবং লসাগু ও গসাগু নির্ণয় করতে হবে।",
                        "ka": "২৪ এর সকল উৎপাদক নির্ণয় কর।",
                        "kha": "২৪ ও ৩৬ এর গসাগু নির্ণয় কর।",
                        "ga": "সংখ্যা দুটির লসাগু ও গসাগু এর গুণফল সংখ্যা দুটির গুণফলের সমান কিনা—যাচাই কর।",
                        "gha": "\"মৌলিক উৎপাদকের সাহায্যে গসাগু নির্ণয় প্রক্রিয়া সহজতর হয়।\"—উক্তিটি মূল্যায়ন কর।"
                    }
                ],
                "Written": [
                    {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "১ থেকে ১০০ পর্যন্ত কয়টি মৌলিক সংখ্যা আছে এবং কী কী—ব্যাখ্যা কর।"},
                    {"title": "গাণিতিক সমস্যা", "desc": "তিনটি ঘণ্টা একত্রে বাজার পর যথাক্রমে ৯, ১২ ও ১৫ সেকেন্ড অন্তর বাজতে লাগল। কতক্ষণ পর ঘণ্টাগুলো আবার একত্রে বাজবে?"}
                ]
            }
        },
        "বীজগণিত": {
            "বীজগণিতীয় রাশি": {
                "MCQ": [
                    {"q": "৩x + ৫y রাশিতে x এর সহগ কত?", "options": ["ক) ৩", "খ) ৫", "গ) ৮", "ঘ) ১৫"], "answer": "ক) ৩"},
                    {"q": "বীজগণিতীয় রাশিতে যোগ, বিয়োগ, গুণ ও ভাগ—এই কয়টি প্রক্রিয়া চিহ্ন ব্যবহার করা হয়?", "options": ["ক) ২টি", "খ) ৩টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "গ) ৪টি"}
                ],
                "CQ": [
                    {
                        "stem": "উদ্দীপক: a = ৩, b = ২ এবং c = ১ হলে, বীজগণিতীয় রাশিটির মান নির্ণয় কর: ৪a² - ৫ab + ৩c²",
                        "ka": "a ও b এর মান বসিয়ে (a + b) এর মান নির্ণয় কর।",
                        "kha": "উদ্দীপকের রাশির মান নির্ণয় কর।",
                        "ga": "দেখাও যে, (a + b)² = a² + ২ab + b²।",
                        "gha": "\"বীজগণিতীয় রাশির মান নির্ণয়ে চলকের সাংখ্যিক মান প্রতিস্থাপন গুরুত্বপূর্ণ ভূমিকা রাখে।\"—বিশ্লেষণ কর।"
                    }
                ],
                "Written": [
                    {"title": "সংক্ষিপ্ত প্রশ্ন", "desc": "বীজগণিতীয় রাশি ও সমীকরণের মধ্যে প্রধান পার্থক্য কী—লিখে দেখাও।"}
                ]
            }
        }
    },
            "ইংরেজি প্রথম পত্র": {
                "Reading & Composition": {
                    "Unit 1: Going on a foreign trip": {
                        "MCQ": [
                            {"q": "What is the main topic of Unit 1 in Class 6 English?", "options": ["a) Preparing for a journey", "b) Playing football", "c) Cooking food", "d) Gardening"], "answer": "a) Preparing for a journey"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "Paragraph Writing", "desc": "Write a paragraph on 'A Journey by Train'."},
                            {"title": "Email/Letter Writing", "desc": "Write an email to your friend thanking him/her for a nice birthday gift."}
                        ]
                    }
                }
            },
            "ইংরেজি দ্বিতীয় পত্র": {
                "Grammar & Composition": {
                    "Parts of Speech": {
                        "MCQ": [
                            {"q": "Identify the verb in: 'The boy runs fast.'", "options": ["a) boy", "b) fast", "c) runs", "d) The"], "answer": "c) runs"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "Application Writing", "desc": "Write an application to the Headmaster for a Morning School facility or Fee Concession."},
                            {"title": "Dialogues", "desc": "Write a dialogue between two friends about the importance of reading newspaper."}
                        ]
                    }
                }
            }, 
            "বিজ্ঞান": {
        "অভিজ্ঞতাভিত্তিক": {
            "জলবায়ু ও পরিবেশ": {
                "MCQ": [
                    {"q": "বায়ুমণ্ডলে কোন গ্যাসের পরিমাণ সবচেয়ে বেশি?", "options": ["ক) অক্সিজেন", "খ) নাইট্রোজেন", "গ) কার্বন ডাই-অক্সাইড", "ঘ) আর্গন"], "answer": "খ) নাইট্রোজেন"},
                    {"q": "উদ্ভিদ শালোকসংশ্লেষণ প্রক্রিয়ায় কোন গ্যাস গ্রহণ করে?", "options": ["ক) অক্সিজেন", "খ) নাইট্রোজেন", "গ) কার্বন ডাই-অক্সাইড", "ঘ) হাইড্রোজেন"], "answer": "গ) কার্বন ডাই-অক্সাইড"}
                ],
                "CQ": [
                    {
                        "stem": "উদ্দীপক: গ্রামের একটি উন্মুক্ত মাঠে নির্বিচারে গাছ কাটা হচ্ছে এবং কলকারখানার কালো ধোঁয়া বাতাসে মিশছে। ফলে ধীরে ধীরে এলাকার তাপমাত্রা বৃদ্ধি পাচ্ছে এবং জীববৈচিত্র্য হুমকির মুখে পড়েছে।",
                        "ka": "বায়ুমণ্ডলের প্রধান গ্যাসগুলোর নাম কী কী?",
                        "kha": "উদ্ভিদ কীভাবে পরিবেশের ভারসাম্য রক্ষা করে—ব্যাখ্যা কর।",
                        "ga": "উদ্দীপকে বর্ণিত সমস্যার কারণে পরিবেশের ওপর কী ধরনের নেতিবাচক প্রভাব পড়ছে—বিশ্লেষণ কর।",
                        "gha": "\"পরিবেশ রক্ষায় বৃক্ষরোপণের বিকল্প নেই।\"—উক্তিটির যথার্থতা মূল্যায়ন কর।"
                    }
                ],
                "Written": [
                    {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "গ্রিনহাউস প্রতিক্রিয়া বলতে কী বোঝো—ব্যাখ্যা কর।"},
                    {"title": "সংক্ষিপ্ত প্রশ্ন", "desc": "বায়ু দূষণ রোধের তিনটি উপায় লেখো।"}
                ]
            }
        }
    },

    "বাংলাদেশ ও বিশ্বপরিচয়": {
        "আমাদের ইতিহাস ও সংস্কৃতি": {
            "বাংলাদেশের মুক্তিযুদ্ধ ও ইতিহাস": {
                "MCQ": [
                    {"q": "কত সালে বাংলাদেশের মহান মুক্তিযুদ্ধ সংঘটিত হয়?", "options": ["ক) ১৯৬৬", "খ) ১৯৬৯", "গ) ১৯৭০", "ঘ) ১৯৭১"], "answer": "ঘ) ১৯৭১"},
                    {"q": "মুজিবনগর সরকার কত তারিখে শপথ গ্রহণ করে?", "options": ["ক) ১০ এপ্রিল ১৯৭১", "খ) ১৭ এপ্রিল ১৯৭১", "গ) ২৬ মার্চ ১৯৭১", "ঘ) ১৬ ডিসেম্বর ১৯৭১"], "answer": "খ) ১৭ এপ্রিল ১৯৭১"}
                ],
                "CQ": [
                    {
                        "stem": "উদ্দীপক: ১৯৭১ সালে এদেশের বীর জনতা অন্যায়ের বিরুদ্ধে রুখে দাঁড়িয়েছিল। দীর্ঘ ৯ মাসের রক্তক্ষয়ী সংগ্রামের পর তারা একটি স্বাধীন ও সার্বভৌম দেশ অর্জন করে।",
                        "ka": "বাংলাদেশের স্বাধীনতা দিবস কবে?",
                        "kha": "মুজিবনগর সরকার গঠনের প্রয়োজনীয়তা কী ছিল—ব্যাখ্যা কর।",
                        "ga": "উদ্দীপকের বর্ণনার সাথে আমাদের মুক্তিযুদ্ধের কোন্ দিকটির সাদৃশ্য রয়েছে—ব্যাখ্যা কর।",
                        "gha": "\"জনগণের একতা ও আত্মত্যাগই ছিল ১৯৭১ সালের বিজয়ের মূল চাবিকাঠি।\"—উক্তিটি মূল্যায়ন কর।"
                    }
                ],
                "Written": [
                    {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "মুক্তিযুদ্ধে সাধারণ মানুষের অবদান সংক্ষেপে আলোচনা কর।"},
                    {"title": "সংক্ষিপ্ত প্রশ্ন", "desc": "মুক্তিযুদ্ধের যেকোনো তিনটি প্রধান সেক্টর ও পরিচালন ব্যবস্থা সংক্ষেপে লেখো।"}
                ]
            }
        }
    },
"ইসলাম ও নৈতিক শিক্ষা": {
        "আকাঈদ ও নৈতিক জীবন": {
            "তাওহীদ ও নৈতিকতা": {
                "MCQ": [
                    {"q": "'তাওহীদ' শব্দের অর্থ কী?", "options": ["ক) একত্ববাদ", "খ) বিশ্বাস", "গ) ইবাদত", "ঘ) আনুগত্য"], "answer": "ক) একত্ববাদ"},
                    {"q": "আল্লাহ তায়ালার পরিচয় ও গুণাবলী সংক্ষেপে প্রকাশের মাধ্যম কোনটি?", "options": ["ক) ঈমানে মুফাসসাল", "খ) ঈমানে মুজমাল", "গ) কালেমা শাহাদাত", "ঘ) আসমাউল হুসনা"], "answer": "খ) ঈমানে মুজমাল"}
                ],
                "CQ": [
                    {
                        "stem": "উদ্দীপক: সাকিব সবসময় সত্য কথা বলে এবং মনে করে আল্লাহ তায়ালা সবকিছু দেখছেন ও শুনছেন। তাই সে গোপনেও কোনো অন্যায় কাজ করে না।",
                        "ka": "ইসলামের প্রথম ও মৌলিক ভিত্তি কোনটি?",
                        "kha": "তাওহীদে বিশ্বাস মানুষের চরিত্রে কী প্রভাব ফেলে—ব্যাখ্যা কর।",
                        "ga": "উদ্দীপকে সাকিবের আচরণের মাধ্যমে আকাঈদের কোন বিষয়টি প্রতিফলিত হয়েছে—ব্যাখ্যা কর।",
                        "gha": "\"আল্লাহ তায়ালার সার্বক্ষণিক নজরদারির অনুভূতিই মানুষকে অন্যায় থেকে মুক্ত রাখে।\"—উক্তিটি সাকিবের জীবনবোধের আলোকে মূল্যায়ন কর।"
                    }
                ],
                "Written": [
                    {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "আখিরাতে বিশ্বাসের গুরুত্ব সংক্ষেপে আলোচনা কর।"},
                    {"title": "সংক্ষিপ্ত প্রশ্ন", "desc": "দৈনন্দিন জীবনে নৈতিক মূল্যবোধ চর্চার তিনটি উপায় লেখো।"}
                ]
            }
        }
    },
    "তথ্য ও যোগাযোগ প্রযুক্তি": {
        "তথ্য ও যোগাযোগ প্রযুক্তি পরিচিতি": {
            "হার্ডওয়্যার ও সফটওয়্যার": {
                "MCQ": [
                    {"q": "নিচের কোনটি একটি ইনপুট ডিভাইস?", "options": ["ক) মনিটর", "খ) প্রিন্টার", "গ) কিবোর্ড", "ঘ) স্পিকার"], "answer": "গ) কিবোর্ড"},
                    {"q": "কম্পিউটারের 'মস্তিষ্ক' বলা হয় কোন অংশকে?", "options": ["ক) র্যাম (RAM)", "খ) হার্ডডিস্ক", "গ) প্রসেসর (CPU)", "ঘ) মাদারবোর্ড"], "answer": "গ) প্রসেসর (CPU)"}
                ],
                "CQ": [
                    {
                        "stem": "উদ্দীপক: তাহসিন তার কম্পিউটারে ছবি আঁকার জন্য একটি মাউস ও কিবোর্ড ব্যবহার করে। কাজ শেষ করে সে তার আঁকা ছবিটি প্রিন্টারের মাধ্যমে কাগজে ছেপে বের করে নেয়।",
                        "ka": "ICT এর পূর্ণরূপ কী?",
                        "kha": "ইনপুট ডিভাইস ও আউটপুট ডিভাইসের প্রধান পার্থক্য কী—ব্যাখ্যা কর।",
                        "ga": "তাহসিনের ব্যবহৃত ডিভাইসগুলোর মধ্যে কোনগুলো ইনপুট এবং কোনটি আউটপুট তা চিহ্নিত করে কাজ ব্যাখ্যা কর।",
                        "gha": "\"আধুনিক শিক্ষাব্যবস্থায় তথ্য ও যোগাযোগ প্রযুক্তির ভূমিকা অপরিসীম।\"—উক্তিটির যথার্থতা মূল্যায়ন কর।"
                    }
                ],
                "Written": [
                    {"title": "অনুধাবনমূলক প্রশ্ন", "desc": "সফটওয়্যার ছাড়া হার্ডওয়্যার কেন অচল—ব্যাখ্যা কর।"},
                    {"title": "সংক্ষিপ্ত প্রশ্ন", "desc": "ইন্টারনেট ব্যবহারের যেকোনো তিনটি সুফল সংক্ষেপে লেখো।"}
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
                        "CQ": [],
                        "Written": [
                            {"title": "ভাবসম্প্রসারণ", "desc": "দাঁড়াও, একটু দাড়াও... মানুষের কল্যাণেই মানুষ।"},
                            {"title": "চিঠি বা দরখাস্ত", "desc": "শিক্ষা সফরে যাওয়ার অনুমতি চেয়ে প্রধান শিক্ষকের নিকট আবেদনপত্র।"}
                        ]
                    }
                }
            },
            "বাংলা ২য় পত্র": {
      "ব্যাকরণ ও নির্মিতি": {
        "শব্দ, পদ ও বাক্যপ্রকরণ": {
          "MCQ": [
            {"q": "বাংলা ভাষার মূল উৎস কোনটি?", "options": ["ক) বৈদিক ভাষা", "খ) ইন্দো-ইউরোপীয়", "গ) প্রাকৃত", "ঘ) অপভ্রংশ"], "answer": "খ) ইন্দো-ইউরোপীয়"},
            {"q": "সাধু ভাষার প্রধান বৈশিষ্ট্য কোনটি?", "options": ["ক) পরিবর্তনশীল", "খ) গুরুগম্ভীর ও তৎসম শব্দবহুল", "গ) আঞ্চলিকতা দোষে দুষ্ট", "ঘ) সংক্ষিপ্ত ও চটুল"], "answer": "খ) গুরুগম্ভীর ও তৎসম শব্দবহুল"},
            {"q": "কোন শব্দটিতে উপসর্গ যুক্ত হয়েছে?", "options": ["ক) উপহার", "খ) ফলপ্রসূ", "গ) বইপত্র", "ঘ) হাঁস-মুরগি"], "answer": "ক) উপহার"},
            {"q": "ণ-ত্ব ও ষ-ত্ব বিধান কোন ভাষার শব্দের ক্ষেত্রে প্রযোজ্য?", "options": ["ক) তদ্ভব", "খ) তৎসম", "গ) দেশি", "ঘ) বিদেশি"], "answer": "খ) তৎসম"},
            {"q": "নিচের কোনটি স্বরসন্ধির উদাহরণ?", "options": ["ক) বিদ্যালয়", "খ) দিগন্ত", "গ) পরিচ্ছদ", "ঘ) সংবাদ"], "answer": "ক) বিদ্যালয়"},
            {"q": "'আকাশ' শব্দের সমার্থক শব্দ কোনটি?", "options": ["ক) বসুন্ধরা", "খ) গগন", "গ) পাবক", "ঘ) সলিল"], "answer": "খ) গগন"},
            {"q": "'অগ্নি' এর বিপরীত শব্দ কোনটি?", "options": ["ক) বাতাস", "খ) জল", "গ) আলো", "ঘ) ধোঁয়া"], "answer": "খ) জল"},
            {"q": "বিশেষ্যের পরিবর্তে যে শব্দ ব্যবহৃত হয় তাকে কী বলে?", "options": ["ক) বিশেষণ", "খ) সর্বনাম", "গ) অব্যয়", "ঘ) ক্রিয়া"], "answer": "খ) সর্বনাম"},
            {"q": "বাক্যের মূল উপাদান কী?", "options": ["ক) বর্ণ", "খ) শব্দ", "গ) পদ", "ঘ) ধ্বনি"], "answer": "গ) পদ"},
            {"q": "ক্রিয়ার মূল অংশকে কী বলা হয়?", "options": ["ক) প্রত্যয়", "খ) উপসর্গ", "গ) ধাতু", "ঘ) বিভক্তি"], "answer": "গ) ধাতু"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সুমাইয়া প্রতিদিন সকালে উঠে ব্যাকরণের নিয়ম মেনে শুদ্ধভাবে কথা বলার চর্চা করে এবং লেখার সময় বানান ও বিভক্তির প্রতি বিশেষ নজর রাখে। ফলে তার লেখার মান দিন দিন উন্নত হচ্ছে।",
              "ka": "ভাষা কাকে বলে?",
              "kha": "সাধু ও চলিত ভাষার দুটি মূল পার্থক্য ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সুমাইয়ার অনুশীলনে বাংলা ব্যাকরণ চর্চার কোন দিকটি প্রকাশ পেয়েছে—ব্যাখ্যা কর।",
              "gha": "\"ব্যাকরণ পাঠের মাধ্যমেই ভাষার শুদ্ধরূপ অর্জন সম্ভব।\"—উদ্দীপক ও পাঠ্যবইয়ের আলোকে পর্যালোচনা কর।"
            },
            {
              "stem": "উদ্দীপক ২: বাংলা ভাষায় এমন কিছু শব্দ ব্যবহৃত হয় যা সরাসরি সংস্কৃত থেকে পরিবর্তিত হয়ে কিংবা অপরিবর্তিত অবস্থায় আসে। আবার অনেক শব্দ উপসর্গ ও প্রত্যয় যোগ করে নতুন অর্থের সৃষ্টি করে।",
              "ka": "বাংলা ব্যাকরণের প্রধান আলোচ্য বিষয় কয়টি?",
              "kha": "উপসর্গ কীভাবে নতুন শব্দ গঠন করে—উদাহরণসহ বুঝিয়ে লেখ।",
              "ga": "উদ্দীপকের আলোকে শব্দ গঠনের প্রক্রিয়াটি বাংলা ব্যাকরণের কোন অংশকে নির্দেশ করে—আলোচনা কর।",
              "gha": "\"শব্দ গঠন ও ব্যাকরণগত নিয়মকানুন ভাষার মূল ভিত্তি।\"—উক্তিটি বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "English 1st Paper": {
      "Reading & Composition": {
        "Unit 1: Environment & Stories": {
          "MCQ": [
            {"q": "What class does Rina study in?", "options": ["a) class six", "b) class seven", "c) class eight", "d) class nine"], "answer": "b) class seven"},
            {"q": "Why does Rina want to become a doctor?", "options": ["a) To earn money", "b) To serve poor villagers", "c) To travel the world", "d) To live in a city"], "answer": "b) To serve poor villagers"},
            {"q": "Where did Hasan go last winter?", "options": ["a) Cox's Bazar", "b) Sreemangal", "c) Sylhet", "d) Sundarbans"], "answer": "b) Sreemangal"},
            {"q": "What did Hasan and his family see in Sreemangal?", "options": ["a) Sea beach", "b) Green tea gardens", "c) Desert", "d) High mountains"], "answer": "b) Green tea gardens"},
            {"q": "What is the synonym of the word 'Serene'?", "options": ["a) Calm", "b) Noisy", "c) Rough", "d) Busy"], "answer": "a) Calm"},
            {"q": "What does early rising give a person?", "options": ["a) Enough time to perform daily duties", "b) Laziness", "c) Sickness", "d) Fatigue"], "answer": "a) Enough time to perform daily duties"},
            {"q": "How does early rising keep the mind?", "options": ["a) Dull and sad", "b) Fresh and cheerful", "c) Angry and upset", "d) Tired"], "answer": "b) Fresh and cheerful"},
            {"q": "Choose the correct past form of 'visit':", "options": ["a) visit", "b) visits", "c) visited", "d) visiting"], "answer": "c) visited"},
            {"q": "Which one is an article?", "options": ["a) And", "b) But", "c) An", "d) In"], "answer": "c) An"},
            {"q": "What does a healthy body lead to?", "options": ["a) A sound mind", "b) Bad habits", "c) Weakness", "d) Illness"], "answer": "a) A sound mind"}
          ],
          "CQ": [
            {
              "stem": "Read the passage: Rina is a student of class seven. She gets up early in the morning and attends her school regularly. She dreams of becoming a doctor to serve poor villagers.",
              "ka": "What class does Rina study in?",
              "kha": "Why does Rina want to become a doctor?",
              "ga": "Describe Rina's daily routine based on the stem using appropriate Present Simple tenses.",
              "gha": "\"Determination and hard work lead to success.\" Evaluate Rina's mindset in light of the context."
            },
            {
              "stem": "Read the passage: Last winter, Hasan visited Sreemangal with his family. They saw green tea gardens and enjoyed the serene beauty of nature.",
              "ka": "Where did Hasan go last winter?",
              "kha": "How did Hasan and his family feel during the visit?",
              "ga": "Identify the Past Tense verbs used in the stem and explain their functions.",
              "gha": "\"Travelling broadens our horizon and refreshes our mind.\" Justify this statement with Hasan's experience."
            }
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
        "মূলদ-অমূলদ সংখ্যা ও মূলদ সমীকরণ": {
          "MCQ": [
            {"q": "নিচের কোনটি মৌলিক সংখ্যা?", "options": ["ক) ৪", "খ) ৯", "গ) ১৩", "ঘ) ১৫"], "answer": "গ) ১৩"},
            {"q": "সবচেয়ে ছোট জোড় মৌলিক সংখ্যা কোনটি?", "options": ["ক) ০", "খ) ১", "গ) ২", "ঘ) ৪"], "answer": "গ) ২"},
            {"q": "√২৫ সংখ্যাটি কী ধরনের সংখ্যা?", "options": ["ক) অমূলদ", "খ) মূলদ", "গ) ঋণাত্মক", "ঘ) ভগ্নাংশ"], "answer": "খ) মূলদ"},
            {"q": "x + ৫ = ১২ সমীকরণে x এর মান কত?", "options": ["ক) ৫", "খ) ৭", "গ) ১২", "ঘ) ১৭"], "answer": "খ) ৭"},
            {"q": "৩a²b রাশিতে a এর ঘাত (Power) কত?", "options": ["ক) ১", "খ) ২", "গ) ৩", "ঘ) ৪"], "answer": "খ) ২"},
            {"q": "(a + b)² এর সঠিক প্রসারিত রূপ কোনটি?", "options": ["ক) a² + b²", "খ) a² + 2ab + b²", "গ) a² - 2ab + b²", "ঘ) a² - b²"], "answer": "খ) a² + 2ab + b²"},
            {"q": "বর্গের ক্ষেত্রফল = ?", "options": ["ক) দৈর্ঘ্য × প্রস্থ", "খ) (বাহু)²", "গ) ২ × (দৈর্ঘ্য + প্রস্থ)", "ঘ) বাহু × ৪"], "answer": "খ) (বাহু)²"},
            {"q": "সমকোণী ত্রিভুজের বৃহত্তম বাহুকে কী বলে?", "options": ["ক) লম্ব", "খ) ভূমি", "গ) অতিভুজ", "ঘ) উচ্চতা"], "answer": "গ) অতিভুজ"},
            {"q": "১ মিটার = কত সেন্টিমিটার?", "options": ["ক) ১০ সেন্টিমিটার", "খ) ১০০ সেন্টিমিটার", "গ) ১০০০ সেন্টিমিটার", "ঘ) ১০,০০০ সেন্টিমিটার"], "answer": "খ) ১০০ সেন্টিমিটার"},
            {"q": "৫০ এর ২০% কত?", "options": ["ক) ৫", "খ) ১০", "গ) ১৫", "ঘ) ২০"], "answer": "খ) ১০"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: তিনটি ক্রমিক স্বাভাবিক সংখ্যার যোগফল ৩৬।",
              "ka": "ক্ষুদ্রতম সংখ্যাটি x হলে অন্য সংখ্যা দুটিকে x এর মাধ্যমে লেখ[cite: 13]।",
              "kha": "উদ্দীপকের তথ্য অনুসারে গঠিত সমীকরণটির সমাধান করে সংখ্যা তিনটি নির্ণয় কর[cite: 13]।",
              "ga": "সংখ্যা তিনটির গড় ও বৃহত্তম সংখ্যার বর্গের অন্তর নির্ণয় কর[cite: 13]।",
              "gha": "\"বীজগণিতীয় সমীকরণ গঠন পাটিগণিতীয় জটিল হিসাবকে সহজতর করে।\"—প্রমাণ কর[cite: 14]."
            },
            {
              "stem": "উদ্দীপক ২: একটি আয়তাকার বাগানের দৈর্ঘ্য প্রস্থের চেয়ে ৫ মিটার বেশি। বাগানটির পরিসীমা ৫০ মিটার।",
              "ka": "প্রস্থ x মিটার ধরে বাগানের দৈর্ঘ্য x এর মাধ্যমে লেখ[cite: 14]।",
              "kha": "বাগানটির দৈর্ঘ্য ও প্রস্থ নির্ণয় কর[cite: 14]।",
              "ga": "বাগানটির ক্ষেত্রফল নির্ণয় কর এবং প্রতি বর্গমিটারে ৫০ টাকা হিসেবে ঘাস লাগাতে কত খরচ হবে হিসাব কর[cite: 14]।",
              "gha": "পরিসীমা অপরিবর্তিত রেখে বাগানটিকে বর্গাকার করলে এর ক্ষেত্রফলের কী পরিবর্তন হবে—বিশ্লেষণ কর[cite: 14]."
            }
          ]
        }
      }
    },
    "বিজ্ঞান": {
      "ভৌত ও জীববিজ্ঞান": {
        "কোষ ও তাপ-তাপমাত্রা": {
          "MCQ": [
            {"q": "জীবদেহের গঠন ও কার্যকারিতার একককে কী বলে?", "options": ["ক) কলা", "খ) কোষ", "গ) অঙ্গ", "ঘ) তন্ত্র"], "answer": "খ) কোষ"},
            {"q": "উদ্ভিদ কোষের কোন অঙ্গাণু সালোকসংশ্লেষণে সাহায্য করে?", "options": ["ক) মাইটোকন্ড্রিয়া", "খ) ক্লোরোপ্লাস্ট", "গ) রাইবোসোম", "ঘ) গলগি বডি"], "answer": "খ) ক্লোরোপ্লাস্ট"},
            {"q": "কোষের 'পাওয়ার হাউস' কাকে বলা হয়?", "options": ["ক) নিউক্লিয়াস", "খ) লাইসোসোম", "গ) মাইটোকন্ড্রিয়া", "ঘ) সেন্ট্রোসোম"], "answer": "গ) মাইটোকন্ড্রিয়া"},
            {"q": "তাপমাত্রার এসআই (SI) একক কোনটি?", "options": ["ক) সেলসিয়াস", "খ) ফারেনহাইট", "গ) কেলভিন", "ঘ) ক্যালরি"], "answer": "গ) কেলভিন"},
            {"q": "মানবদেহের স্বাভাবিক তাপমাত্রা কত ডিগ্রি ফারেনহাইট?", "options": ["ক) ৯৭.৪° F", "খ) ৯৮.৪° F", "গ) ৯৯.৪° F", "ঘ) ১০০° F"], "answer": "খ) ৯৮.৪° F"},
            {"q": "পদার্থের কোন অবস্থায় অণুগুলোর মধ্যে আন্তঃআণবিক আকর্ষণ সবচেয়ে বেশি?", "options": ["ক) কঠিন", "খ) তরল", "গ) বায়বীয়", "ঘ) প্লাজমা"], "answer": "ক) কঠিন"},
            {"q": "নিচের কোনটি ভৌত পরিবর্তন?", "options": ["ক) লোহায় মরিচা ধরা", "খ) বরফ গলে পানি হওয়া", "গ) দুধ থেকে দই তৈরি", "ঘ) কাঠ পোড়ানো"], "answer": "খ) বরফ গলে পানি হওয়া"},
            {"q": "উদ্ভিদ মূলরোমের সাহায্যে মাটি থেকে কোন প্রক্রিয়ায় পানি শোষণ করে?", "options": ["ক) ব্যাপন", "খ) অভিস্রবণ", "গ) প্রস্বেদন", "ঘ) শসন"], "answer": "খ) অভিস্রবণ"},
            {"q": "শব্দ সঞ্চালনের জন্য কিসের প্রয়োজন হয়?", "options": ["ক) মাধ্যমের", "খ) আলোর", "গ) শুধু শূন্যস্থানের", "ঘ) বিদ্যুতের"], "answer": "ক) মাধ্যমের"},
            {"q": "আলোর প্রতিফলনের প্রধান নিয়ম কয়টি?", "options": ["ক) ১টি", "খ) ২টি", "গ) ৩টি", "ঘ) ৪টি"], "answer": "খ) ২টি"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: রহিম একটি পাত্রে কিছু বরফ রেখে তাপ দিতে শুরু করল। কিছু সময় পর বরফ গলে পানি হলো এবং আরও তাপ দেওয়ায় পানি ফুটতে শুরু করল।",
              "ka": "তাপমাত্রা কী?[cite: 15]",
              "kha": "তাপ ও তাপমাত্রার দুটি মৌলিক পার্থক্য ব্যাখ্যা কর[cite: 15].",
              "ga": "উদ্দীপকে সংঘটিত অবস্থান্তর প্রক্রিয়ায় অণুগুলোর কাঠামোগত পরিবর্তন ব্যাখ্যা কর[cite: 15].",
              "gha": "\"তাপ প্রয়োগে পদার্থের ভৌত অবস্থার পরিবর্তন ঘটে হলেও রাসায়নিক গঠন অপরিবর্তিত থাকে।\"—যাচাই কর[cite: 15]."
            },
            {
              "stem": "উদ্দীপক ২: অণুবীক্ষণ যন্ত্রের নিচে একটি উদ্ভিদ কোষ ও প্রাণীকোষ পর্যবেক্ষণ করা হলো। দেখা গেল, উদ্ভিদ কোষে একটি শক্ত প্রাচীর ও সবুজ অঙ্গাণু বিদ্যমান যা প্রাণীকোষে নেই।",
              "ka": "কোষ প্রাচীর কী দিয়ে গঠিত?[cite: 15]",
              "kha": "প্লাস্টিডকে উদ্ভিদের খাদ্য তৈরির কারখানা বলা হয় কেন?[cite: 15]",
              "ga": "উদ্দীপকের আলোকে উদ্ভিদকোষ ও প্রাণীকোষের প্রধান বৈসাদৃশ্যগুলো তুলনা কর[cite: 15, 16].",
              "gha": "\"সবুজ প্লাস্টিডের অনুপস্থিতিতে সমগ্র জীবজগৎ স্তব্ধ হয়ে পড়বে।\"—উক্তিটি বিশ্লেষণ কর[cite: 16]."
            }
          ]
        }
      }
    },
    "বাংলাদেশ ও বিশ্বপরিচয়": {
      "ইতিহাস ও সমাজ": {
        "মুক্তিযুদ্ধ ও পরিবেশ": {
          "MCQ": [
            {"q": "বাংলাদেশের স্বাধীনতা সংগ্রাম কত সালে সংঘটিত হয়?", "options": ["ক) ১৯৬৬", "খ) ১৯৬৯", "গ) ১৯৭০", "ঘ) ১৯৭১"], "answer": "ঘ) ১৯৭১"},
            {"q": "মুজিবনগর সরকার কবে গঠিত হয়?", "options": ["ক) ৭ মার্চ ১৯৭১", "খ) ১০ এপ্রিল ১৯৭১", "গ) ১৭ এপ্রিল ১৯৭১", "ঘ) ১৬ ডিসেম্বর ১৯৭১"], "answer": "খ) ১০ এপ্রিল ১৯৭১"},
            {"q": "মুক্তিযুদ্ধে কত নম্বর সেক্টর কোনো আঞ্চলিক কমান্ডার ছিল না (নৌ-কমান্ডো)?", "options": ["ক) ১ নম্বর", "খ) ৭ নম্বর", "গ) ১০ নম্বর", "ঘ) ১১ নম্বর"], "answer": "গ) ১০ নম্বর"},
            {"q": "জাতিসংঘের বর্তমান সদস্য সংখ্যা কত?", "options": ["ক) ১৯০", "খ) ১৯২", "গ) ১৯৩", "ঘ) ১৯৫"], "answer": "গ) ১৯৩"},
            {"q": "সার্ক (SAARC) কত সালে প্রতিষ্ঠিত হয়?", "options": ["ক) ১৯৮০", "খ) ১৯৮৫", "গ) ১৯৯০", "ঘ) ২০০০"], "answer": "খ) ১৯৮৫"},
            {"q": "জলবায়ু পরিবর্তনের প্রধান মানবসৃষ্ট কারণ কোনটি?", "options": ["ক) নির্বিচারে বৃক্ষনিধন", "খ) সুনামি", "গ) আগ্নেয়গিরির অগ্ন্যুৎপাত", "ঘ) ভূমিকম্প"], "answer": "ক) নির্বিচারে বৃক্ষনিধন"},
            {"q": "সামাজিকীকরণের প্রথম ও প্রধান বাহন কোনটি?", "options": ["ক) বিদ্যালয়", "খ) পরিবার", "গ) খেলার সাথী", "ঘ) গণমাধ্যম"], "answer": "খ) পরিবার"},
            {"q": "এশিয়ার সবচেয়ে বৃহত্তম মহাদেশ কোনটি?", "options": ["ক) ইউরোপ", "খ) এশিয়া", "গ) আফ্রিকা", "ঘ) অস্ট্রেলিয়া"], "answer": "খ) এশিয়া"},
            {"q": "কত বছর বয়স পর্যন্ত শিশুদের শিশু বলা হয় (বাংলাদেশ শিশু নীতি অনুযায়ী)?", "options": ["ক) ১৪ বছর", "খ) ১৬ বছর", "গ) ১৮ বছর", "ঘ) ২১ বছর"], "answer": "গ) ১৮ বছর"},
            {"q": "আন্তর্জাতিক নারী দিবস কবে পালিত হয়?", "options": ["ক) ৮ মার্চ", "খ) ২১ ফেব্রুয়ারি", "গ) ১ মে", "ঘ) ১০ ডিসেম্বর"], "answer": "ক) ৮ মার্চ"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ১৯৭১ সালে বাংলাদেশের সাধারণ কৃষক, শ্রমিক ও ছাত্রসমাজ একজোট হয়ে দখলদার বাহিনীর বিরুদ্ধে প্রতিরোধ গড়ে তোলে এবং দীর্ঘ ৯ মাস রক্তক্ষয়ী সংগ্রামের পর ১৬ই ডিসেম্বর চূড়ান্ত বিজয় অর্জন করে।",
              "ka": "মুক্তিযুদ্ধের সর্বাধিনায়ক কে ছিলেন?[cite: 17]",
              "kha": "মুজিবনগর সরকার শপথ গ্রহণ করে কোথায়—ব্যাখ্যা কর[cite: 17].",
              "ga": "উদ্দীপকে উল্লিখিত সর্বস্তরের মানুষের অংশগ্রহণের দিকটি আমাদের মুক্তিযুদ্ধে কী ভূমিকা রেখেছিল—ব্যাখ্যা কর[cite: 17].",
              "gha": "\"জনগণের ঐক্য ও জাতীয়তাবোধই ছিল ১৯৭১ সালের স্বাধীনতার মূল শক্তি।\"—মূল্যায়ন কর[cite: 17]."
            },
            {
              "stem": "উদ্দীপক ২: উপকূলীয় একটি গ্রামে অনিয়ন্ত্রিতভাবে বনাঞ্চল কেটে ফেলার ফলে সমুদ্রের জলোচ্ছ্বাস ও ঝড়-ঝঞ্ঝার তীব্রতা বৃদ্ধি পেয়ে ঘরবাড়ি ধ্বংস হয়ে যাচ্ছে।",
              "ka": "গ্রিনহাউস গ্যাস কী?[cite: 17]",
              "kha": "জলবায়ু পরিবর্তনের ফলে বাংলাদেশের বৈশ্বিক ঝুঁকি ব্যাখ্যা কর[cite: 17].",
              "ga": "উদ্দীপকের এলাকার প্রাকৃতিক বিপর্যয়ের কারণ পাঠ্যবইয়ের পরিবেশ অধ্যায়ের আলোকে বর্ণনা কর[cite: 17].",
              "gha": "\"টেকসই উন্নয়ন ও বৃক্ষরোপণই কেবল উপকূলীয় জীবন রক্ষা করতে পারে।\"—মতামত দাও[cite: 17]."
            }
          ]
        }
      }
    },
    "ইসলাম ও নৈতিক শিক্ষা": {
      "আকাঈদ ও আখলাক": {
        "ঈমান ও সামাজিক চরিত্র": {
          "MCQ": [
            {"q": "'আকাঈদ' শব্দটি কোন শব্দের বহুবচন?", "options": ["ক) আকীদাহ", "খ) ঈমান", "গ) ইসলাম", "ঘ) তাওহীদ"], "answer": "ক) আকীদাহ"},
            {"q": "ইসলামের প্রথম ও মৌলিক ভিত্তি কোনটি?", "options": ["ক) সালাত", "খ) তাওহীদ", "গ) যাকাত", "ঘ) হজ"], "answer": "খ) তাওহীদ"},
            {"q": "'আসমাউল হুসনা' বলতে কী বোঝায়?", "options": ["ক) সুন্দর নামসমূহ", "খ) নফসের পবিত্রতা", "গ) শ্রেষ্ঠ নবীদের তালিকা", "ঘ) কিয়ামতের আলামত"], "answer": "ক) সুন্দর নামসমূহ"},
            {"q": "দৈনন্দিন জীবনে সালাত দিনে কতবার ফরজ করা হয়েছে?", "options": ["ক) ৩ বার", "খ) ৪ বার", "গ) ৫ বার", "ঘ) ৭ বার"], "answer": "গ) ৫ বার"},
            {"q": "কুরআন মজিদের প্রধান সুরক্ষার দায়িত্ব কার?", "options": ["ক) ফেরেশতাদের", "খ) মানুষের", "গ) মহান আল্লাহর", "ঘ) নবীদের"], "answer": "গ) মহান আল্লাহর"},
            {"q": "আখলাক প্রধানত কয় ভাগে বিভক্ত?", "options": ["ক) ২ ভাগে", "খ) ৩ ভাগে", "গ) ৪ ভাগে", "ঘ) ৫ ভাগে"], "answer": "ক) ২ ভাগে"},
            {"q": "আমলে সালেহ বলতে কী বোঝায়?", "options": ["ক) সৎকাজ", "খ) অন্যায় কাজ", "গ) সময় অপচয়", "ঘ) অহংকার"], "answer": "ক) সৎকাজ"},
            {"q": "সত্যবাদিতাকে আরবিতে কী বলা হয়?", "options": ["ক) সিদক", "খ) কিজব", "গ) খিয়ানত", "ঘ) আমানত"], "answer": "ক) সিদক"},
            {"q": "কাউকে গালি দেওয়া বা মন্দ নামে ডাকা ইসলামে-", "options": ["ক) মুস্তাহাব", "খ) হারাম/হালকা গুনাহ", "গ) কবিরা গুনাহ ও নিষিদ্ধ", "ঘ) জায়েজ"], "answer": "গ) কবিরা গুনাহ ও নিষিদ্ধ"},
            {"q": "পরিবেশের ভারসাম্য রক্ষায় গাছ লাগানো ইসলামে-", "options": ["ক) নফল ইবাদত ও সাদকাহ", "খ) নিরুৎসাহিত", "গ) ফরজ নয় ও অপ্রয়োজনীয়", "ঘ) মাকরুহ"], "answer": "ক) নফল ইবাদত ও সাদকাহ"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সায়েম তার ব্যবসায় কখনো মাপে কম দেয় না এবং খদ্দেরের সাথে সততার পরিচয় দেয়। সে বিশ্বাস করে যে তার প্রতিটি কাজের হিসাব শেষ বিচারে আল্লাহর কাছে দিতে হবে।",
              "ka": "আখিরাত শব্দের অর্থ কী?[cite: 18]",
              "kha": "তাওহীদে বিশ্বাস মানুষের মনে কীভাবে অন্যায়ের ভয় দূর করে—ব্যাখ্যা কর[cite: 18, 19].",
              "ga": "উদ্দীপকের সায়েমের কর্মের মাধ্যমে ঈমানের কোন স্তম্ভটির প্রতিফলন ঘটেছে—ব্যাখ্যা কর[cite: 19].",
              "gha": "\"পরকালে জবাবদিহিতার অনুভূতিই ব্যবসায়িক ও সামাজিক সততার মূল ভিত্তি।\"—বিশ্লেষণ কর[cite: 19]."
            },
            {
              "stem": "উদ্দীপক ২: রফিক ক্লাসে কোনো সহপাঠী বিপদে পড়লে সাহায্য করে এবং প্রতিবেশীদের সাথে সবসময় হাসিমুখে কথা বলে ভালো ব্যবহার করে।",
              "ka": "আখলাক-এ হামিদা কী?[cite: 19]",
              "kha": "ইসলামে ভালো ব্যবহারের গুরুত্ব ব্যাখ্যা কর[cite: 19].",
              "ga": "উদ্দীপকে রফিকের আচরণে সৎচরিত্রের কোন গুণাবলি ফুটে উঠেছে—ব্যাখ্যা কর[cite: 19].",
              "gha": "\"উত্তম চরিত্রই একজন সুনাগরিক ও আদর্শ মুসলিম গঠনের হাতিয়ার।\"—উক্তিটি মূল্যায়ন কর[cite: 19]."
            }
          ]
        }
      }
    },
    "তথ্য ও যোগাযোগ প্রযুক্তি": {
      "ICT基礎 & নেটওয়ার্ক": {
        "কম্পিউটার ও ইন্টারনেট": {
          "MCQ": [
            {"q": "কম্পিউটারের প্রধান মেমোরি কোনটি?", "options": ["ক) র্যাম (RAM)", "খ) পেনড্রাইভ", "গ) সিডি", "ঘ) হার্ডডিস্ক"], "answer": "ক) র্যাম (RAM)"},
            {"q": "নিচের কোনটি একটি আউটপুট ডিভাইস?", "options": ["ক) মাউস", "খ) কিবোর্ড", "গ) মনিটর", "ঘ) স্ক্যানার"], "answer": "গ) মনিটর"},
            {"q": "কম্পিউটারের ব্রেইন বা মস্তিষ্ক বলা হয় কাকে?", "options": ["ক) CPU", "খ) RAM", "গ) ROM", "ঘ) Monitor"], "answer": "ক) CPU"},
            {"q": "১ বাইট (Byte) কত বিটের (Bit) সমান?", "options": ["ক) ৪ বিট", "খ) ৮ বিট", "গ) ১৬ বিট", "ঘ) ৩২ বিট"], "answer": "খ) ৮ বিট"},
            {"q": "ই-মেইল (E-mail) এর পূর্ণরূপ কী?", "options": ["ক) Electrical Mail", "খ) Electronic Mail", "গ) Easy Mail", "ঘ) Engine Mail"], "answer": "খ) Electronic Mail"},
            {"q": "ওয়েবসাইট দেখার জন্য কোন সফটওয়্যার ব্যবহৃত হয়?", "options": ["ক) ওয়েব ব্রাউজার", "খ) ওয়ার্ড প্রসেসর", "গ) মিডিয়া প্লেয়ার", "ঘ) অ্যান্টিভাইরাস"], "answer": "ক) ওয়েব ব্রাউজার"},
            {"q": "নিচের কোনটি একটি সার্চ ইঞ্জিন?", "options": ["ক) Facebook", "খ) Google", "গ) WhatsApp", "ঘ) Windows"], "answer": "খ) Google"},
            {"q": "নেটওয়ার্ক সংযোগে ব্যবহৃত তারকে কী বলা হয়?", "options": ["ক) ক্যাবল", "খ) প্রসেসর", "গ) পোর্ট", "ঘ) হাব"], "answer": "ক) ক্যাবল"},
            {"q": "পাসওয়ার্ড ব্যবহারের প্রধান উদ্দেশ্য কী?", "options": ["ক) কম্পিউটার দ্রুত করা", "খ) তথ্য ও অ্যাকাউন্টের নিরাপত্তা প্রদান", "গ) মেমোরি বাড়ানো", "ঘ) ছবি প্রদর্শন"], "answer": "খ) তথ্য ও অ্যাকাউন্টের নিরাপত্তা প্রদান"},
            {"q": "সাইবার স্পেসে অন্যায়ভাবে অন্যের তথ্য চুরি বা ক্ষতি করাকে কী বলে?", "options": ["ক) হ্যাকিং/সাইবার অপরাধ", "খ) ব্রাউজিং", "গ) ডাউনলোড", "ঘ) প্রোগ্রামিং"], "answer": "ক) হ্যাকিং/সাইবার অপরাধ"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সিয়াম ইন্টারনেটে পরীক্ষার ফলাফল দেখার জন্য কম্পিউটারে সার্চ ব্রাউজার ব্যবহার করে রোল নম্বর প্রদান করল এবং কয়েক সেকেন্ডের মধ্যে মনিটরে রেজাল্ট শিট দেখতে পেয়ে প্রিন্টারের সাহায্যে তা ছেপে নিল।",
              "ka": "ইন্টারনেট কী?[cite: 20]",
              "kha": "ইনপুট ও আউটপুট ডিভাইসের প্রধান দুটি পার্থক্য লেখ[cite: 20].",
              "ga": "উদ্দীপকে সিয়ামের ব্যবহৃত ডিভাইসগুলোর মধ্যে কোনগুলো ইনপুট এবং কোনটি আউটপুট তা চিহ্নিত করে কাজ ব্যাখ্যা কর[cite: 20].",
              "gha": "\"তথ্য প্রযুক্তির দ্রুত বিকাশ শিক্ষা ব্যবস্থাকে সহজ ও প্রবেশযোগ্য করেছে।\"—উদ্দীপকের আলোকে মূল্যায়ন কর[cite: 20]."
            },
            {
              "stem": "উদ্দীপক ২: একটি ব্যাংকে বেশ কয়েকটি কম্পিউটার একসাথে যুক্ত করে একটি নেটওয়ার্ক তৈরি করা হয়েছে যাতে কর্মকর্তাগণ সহজেই এক স্থান থেকে অন্য স্থানে গোপন ফাইল আদান-প্রদান ও ডেটা সংরক্ষণ করতে পারেন।",
              "ka": "LAN এর পূর্ণরূপ কী?[cite: 20]",
              "kha": "কম্পিউটার নেটওয়ার্ক ব্যবহারের দুটি প্রধান সুবিধা ব্যাখ্যা কর[cite: 21].",
              "ga": "উদ্দীপকের ব্যাংকিং ব্যবস্থায় নেটওয়ার্ক প্রযুক্তির ভূমিকা কীভাবে কাজ করছে তা আলোচনা কর[cite: 21].",
              "gha": "\"নেটওয়ার্কিং বিশ্বকে একটি বিশ্বপল্লীতে (Global Village) রূপান্তর করেছে।\"—উক্তিটির যথার্থতা বিচার কর[cite: 21]."
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
                            {"q": "ষোলোই ডিসেম্বর বাংলাদেশে কী হিসেবে পালিত হয়?", "options": ["ক) স্বাধীনতা দিবস", "খ) বিজয় দিবস", "গ) শহীদ দিবস", "ঘ) বাংলা নববর্ষ"], "answer": "খ) বিজয় দিবস"}
                        ],
                        "CQ": [],
                        "Written": [
                            {"title": "সারমর্ম", "desc": "ক্ষুদ্র ক্ষুদ্র বালু কণা... বিন্দু বিন্দু জল।"}
                        ]
                    }
                }
            },
            "বাংলা ২য় পত্র": {
      "ব্যাকরণ ও নির্মিতি": {
        "ভাষা, ব্যাকরণ ও নির্মিতি": {
          "MCQ": [
            {"q": "বাংলা ভাষার মূল ভিত্তি বা উৎস কোনটি?", "options": ["ক) ইন্দো-ইউরোপীয় ভাষা", "খ) ফারসি ভাষা", "গ) ড্রাবিড় ভাষা", "ঘ) প্রাকৃত ভাষা"], "answer": "ক) ইন্দো-ইউরোপীয় ভাষা"},
            {"q": "কোন ভাষা রূপটি পরিবর্তনশীল ও অঞ্চলভেদে ভিন্ন হয়?", "options": ["ক) সাধু ভাষা", "খ) চলিত ভাষা", "গ) কথ্য বা উপভাষা", "ঘ) প্রমিত ভাষা"], "answer": "গ) কথ্য বা উপভাষা"},
            {"q": "সন্ধি ব্যাকরণের কোন অংশে আলোচিত হয়?", "options": ["ক) রূপতত্ত্ব", "খ) ধ্বনিতত্ত্ব", "গ) বাক্যতত্ত্ব", "ঘ) অর্থতত্ত্ব"], "answer": "খ) ধ্বনিতত্ত্ব"},
            {"q": "শব্দের ক্ষুদ্রতম একককে কী বলে?", "options": ["ক) অক্ষর", "খ) ধ্বনি", "গ) পদ", "ঘ) বাক্য"], "answer": "খ) ধ্বনি"},
            {"q": "নিচের কোনটি তৎসম শব্দের উদাহরণ?", "options": ["ক) হাত", "খ) চন্দ্র", "গ) ডিঙি", "ঘ) চেয়ার"], "answer": "খ) চন্দ্র"},
            {"q": "'অভাব' অর্থে কোন উপসর্গটি ব্যবহৃত হয়েছে?", "options": ["ক) অজানা", "খ) গরহাজির", "গ) সুদিন", "ঘ) সুনীতি"], "answer": "খ) গরহাজির"},
            {"q": "ধাতুর পর যে বর্ণ বা বর্ণসমষ্টি যুক্ত হয়ে নতুন শব্দ গঠন করে তাকে কী বলে?", "options": ["ক) প্রত্যয়", "খ) উপসর্গ", "গ) সমাস", "ঘ) অনুসর্গ"], "answer": "ক) প্রত্যয়"},
            {"q": "সমাসের প্রধান কাজ কী?", "options": ["ক) পদের সংকোচন", "খ) পদের সম্প্রসারণ", "গ) বাক্যের দীর্ঘায়ন", "ঘ) অর্থ পরিবর্তন"], "answer": "ক) পদের সংকোচন"},
            {"q": "বাক্যের অপরিহার্য অংশ কয়টি?", "options": ["ক) ১টি", "খ) ২টি", "গ) ৩টি", "ঘ) ৪টি"], "answer": "খ) ২টি"},
            {"q": "সবচেয়ে বেশি ব্যবহৃত বাংলা উপসর্গ কয়টি?", "options": ["ক) ২১টি", "খ) ২০টি", "গ) ১৯টি", "ঘ) ২২টি"], "answer": "ক) ২১টি"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: রফিক সাহেব তার লেখায় সবসময় সহজ-সরল ভাষায় মনের ভাব প্রকাশ করেন, যা সব অঞ্চলের মানুষ সহজেই বুঝতে পারে। অন্যদিকে তার বন্ধু প্রথাগত জটিল ও প্রাচীন শব্দ ব্যবহারে স্বাচ্ছন্দ্য বোধ করেন।",
              "ka": "চলিত ভাষার প্রবর্তক কে?",
              "kha": "সাধু ও চলিত ভাষার দুটি মৌলিক পার্থক্য লেখ।",
              "ga": "উদ্দীপকের রফিক সাহেবের ভাষারীতি পাঠ্যবইয়ের ব্যাকরণগত কোন রূপটিকে নির্দেশ করে—ব্যাখ্যা কর।",
              "gha": "\"সর্বজনগ্রাহ্য ও সাবলীল যোগাযোগের জন্য চলিত/প্রমিত ভাষারীতিই সর্বোত্তম।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: বাংলা ভাষায় শব্দ গঠনের জন্য ধ্বনি, উপসর্গ, প্রত্যয় এবং সমাসের ব্যাপক ব্যবহার রয়েছে। সঠিক নিয়ম মেনে এগুলো প্রয়োগ করলে ভাষার ভাব প্রকাশ নির্ভুল হয়।",
              "ka": "ব্যাকরণের ধ্বনিতত্ত্বে কী নিয়ে আলোচনা করা হয়?",
              "kha": "উপসর্গ কীভাবে নতুন শব্দ গঠনে সাহায্য করে—উদাহরণসহ লেখ।",
              "ga": "উদ্দীপকের আলোকে শব্দ গঠনের ব্যাকরণিক প্রক্রিয়াগুলো ব্যাখ্যা কর।",
              "gha": "\"ব্যাকরণ চর্চাই ভাষার রূপ ও শুদ্ধতা রক্ষায় প্রধান হাতিয়ার।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "English 1st Paper": {
      "Reading & Composition": {
        "Unit 1: Craft & Habits": {
          "MCQ": [
            {"q": "What is Nakshi Kantha?", "options": ["a) A kind of embroidered quilt", "b) A type of folk song", "c) A traditional dance", "d) A local sweet"], "answer": "a) A kind of embroidered quilt"},
            {"q": "Where was the name 'Nakshi Kantha' taken from?", "options": ["a) An English word", "b) The Bengali word 'naksha'", "c) An Arabic word", "d) A Hindi word"], "answer": "b) The Bengali word 'naksha'"},
            {"q": "What does the word 'naksha' mean?", "options": ["a) A traditional song", "b) Artistic pattern", "c) Colorful design", "d) Handmade cloth"], "answer": "b) Artistic pattern"},
            {"q": "What is early rising?", "options": ["a) The habit of getting up early from bed", "b) Sleeping late at night", "c) Waking up in the afternoon", "d) Working overnight"], "answer": "a) The habit of getting up early from bed"},
            {"q": "How does early rising help a person?", "options": ["a) Gives enough time to perform daily duties", "b) Makes a person lazy", "c) Causes tiredness", "d) Wastes time"], "answer": "a) Gives enough time to perform daily duties"},
            {"q": "How does early rising keep the mind?", "options": ["a) Dull and sad", "b) Fresh and cheerful", "c) Tense and worried", "d) Tired"], "answer": "b) Fresh and cheerful"},
            {"q": "What does a healthy body lead to?", "options": ["a) A sound mind", "b) Sickness", "c) Weakness", "d) Fatigue"], "answer": "a) A sound mind"},
            {"q": "Choose the correct preposition: 'He is proficient ___ English.'", "options": ["a) in", "b) at", "c) on", "d) with"], "answer": "a) in"},
            {"q": "What is the synonym of the word 'Traditional'?", "options": ["a) Modern", "b) Conventional/Customary", "c) New", "d) Strange"], "answer": "b) Conventional/Customary"},
            {"q": "Which sentence is in Present Simple Tense?", "options": ["a) He plays football.", "b) He played football.", "c) He will play football.", "d) He is playing football."], "answer": "a) He plays football."}
          ],
          "CQ": [
            {
              "stem": "Read the text: Nakshi Kantha is a kind of embroidered quilt. The name was taken from the Bengali word 'naksha' which means artistic pattern. It is a traditional craft of Bangladesh[cite: 24].",
              "ka": "Where does the name 'Nakshi Kantha' come from?[cite: 24]",
              "kha": "Why is Nakshi Kantha called a traditional craft?[cite: 24]",
              "ga": "Rewrite the main ideas of the text using simple present passive structures[cite: 24].",
              "gha": "\"Cultural crafts like Nakshi Kantha represent our national identity globally.\" Justify the statement[cite: 24]."
            },
            {
              "stem": "Read the text: Early rising is the habit of getting up early from bed. It gives a person enough time to perform daily duties and keeps the mind fresh and cheerful[cite: 24].",
              "ka": "What is early rising?[cite: 24]",
              "kha": "How does early rising benefit human physical health?[cite: 24]",
              "ga": "Transform the positive aspects mentioned in the stem using appropriate linking words[cite: 24].",
              "gha": "\"A healthy body leads to a sound mind.\" Evaluate the significance of early rising in light of this proverb[cite: 24]."
            }
          ]
        }
      }
    },
    "English 2nd Paper": {
      "Grammar & Composition": {
        "Grammar Basics & Transformation": {
          "MCQ": [
            {"q": "Which one is an Assertive sentence?", "options": ["a) What a beautiful bird!", "b) Bangladesh is our motherland.", "c) Do you know him?", "d) May Allah bless you."], "answer": "b) Bangladesh is our motherland."},
            {"q": "Change the voice: 'He plays cricket.'", "options": ["a) Cricket was played by him.", "b) Cricket is played by him.", "c) Cricket has been played by him.", "d) Cricket played by him."], "answer": "b) Cricket is played by him."},
            {"q": "What is the superlative degree of 'Good'?", "options": ["a) Better", "b) Best", "c) More good", "d) Gooder"], "answer": "b) Best"},
            {"q": "Choose the correct preposition: 'He is proficient in English.'", "options": ["a) in", "b) at", "c) on", "d) with"], "answer": "a) in"},
            {"q": "Identify the conjunction in: 'Do or die.'", "options": ["a) Do", "b) or", "c) die", "d) None"], "answer": "b) or"},
            {"q": "Which sentence is in Past Perfect Tense?", "options": ["a) He had finished the task.", "b) He finished the task.", "c) He was finishing the task.", "d) He has finished the task."], "answer": "a) He had finished the task."},
            {"q": "What is the noun form of 'Believe'?", "options": ["a) Believable", "b) Belief", "c) Believing", "d) Believer"], "answer": "b) Belief"},
            {"q": "Choose the correct sentence:", "options": ["a) One of my friends are rich.", "b) One of my friends is rich.", "c) One of my friend is rich.", "d) One of my friends were rich."], "answer": "b) One of my friends is rich."},
            {"q": "Synonym of 'Obey' is-", "options": ["a) Disregard", "b) Follow", "c) Refuse", "d) Ignore"], "answer": "b) Follow"},
            {"q": "Antonym of 'Ancient' is-", "options": ["a) Old", "b) Modern", "c) Traditional", "d) Past"], "answer": "b) Modern"}
          ],
          "CQ": [
            {
              "stem": "Transform the following sentences as directed: (a) Cricket is a very popular game. (Exclamatory) (b) Everybody knows this. (Negative) (c) Who does not love cricket? (Assertive) (d) It is an exciting game. (Interrogative)",
              "ka": "What is an assertive sentence?",
              "kha": "How do you change an affirmative sentence into a negative one without changing its meaning—explain with an example.",
              "ga": "Transform the sentences (a), (b), (c) and (d) according to the directions given in the stem.",
              "loan": "",
              "gha": "\"Sentence transformation tests a student's grammatical capability and flexibility in language use.\"—Evaluate the statement."
            },
            {
              "stem": "Read the following dialogue and apply appropriate grammatical rules: Rahim said to Karim, \"Where are you going now? I need your help to complete my grammar assignment.\"",
              "ka": "What is direct speech?",
              "kha": "State the basic rules of changing direct speech into indirect speech.",
              "ga": "Change the narration of the sentences given in the stem.",
              "gha": "\"Proper understanding of narration and voice change makes written communication error-free.\"—Analyze based on the context."
            }
          ]
        }
      }
    },
    "গণিত": {
      "পাটিগণিত ও বীজগণিত": {
        "প্যাটার্ন, মুনাফা ও বীজগণিতীয় সূত্রাবলী": {
          "MCQ": [
            {"q": "৩, ৬, ৯, ১২, ... প্যাটার্নটির পরবর্তী সংখ্যাটি কত?", "options": ["ক) ১৩", "খ) ১৪", "গ) ১৫", "ঘ) ১৬"], "answer": "গ) ১৫"},
            {"q": "সরল মুনাফা I = ?", "options": ["ক) Pnr", "খ) P/nr", "গ) Pr/n", "ঘ) Pn/r"], "answer": "ক) Pnr"},
            {"q": "(a + b)² - (a - b)² = ?", "options": ["ক) 2(a² + b²)", "খ) 4ab", "গ) a² + b²", "ঘ) 2ab"], "answer": "খ) 4ab"},
            {"q": "১ থেকে ১০ পর্যন্ত মৌলিক সংখ্যা কয়টি?", "options": ["ক) ৩টি", "খ) ৪টি", "গ) ৫টি", "ঘ) ২টি"], "answer": "খ) ৪টি"},
            {"q": "(x + a)(x + b) এর প্রসারিত রূপ কোনটি?", "options": ["ক) x² + (a+b)x + ab", "খ) x² + abx + a", "গ) x² + (a-b)x - ab", "ঘ) x² - ab"], "answer": "ক) x² + (a+b)x + ab"},
            {"q": "চক্রবৃদ্ধি মুনাফা নির্ণয়ের সূত্র C = ?", "options": ["ক) P(1 + r)^n", "খ) P(1 - r)^n", "গ) P(1 + nr)", "ঘ) Prn"], "answer": "ক) P(1 + r)^n"},
            {"q": "একটি বর্গক্ষেত্রের বাহুর দৈর্ঘ্য a হলে তার কর্ণের দৈর্ঘ্য কত?", "options": ["ক) a²", "খ) √2a", "গ) 2a", "ঘ) a/√2"], "answer": "খ) √2a"},
            {"q": "সামান্তরিকের ক্ষেত্রফল = ?", "options": ["ক) ভূমি × উচ্চতা", "খ) ১/২ × ভূমি × উচ্চতা", "গ) দৈর্ঘ্য × প্রস্থ", "ঘ) বাহু²"], "answer": "ক) ভূমি × উচ্চতা"},
            {"q": "x + 1/x = 3 হলে, x² + 1/x² এর মান কত?", "options": ["ক) ৭", "খ) ৯", "গ) ১১", "ঘ) ৬"], "answer": "ক) ৭"},
            {"q": "বৃত্তের পরিধি ও ব্যাসের অনুপাতকে কী বলা হয়?", "options": ["ক) পাই (π)", "খ) থিটা (θ)", "গ) আলফা (α)", "ঘ) ব্যাসার্ধ"], "answer": "ক) পাই (π)"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: কোনো আসল ৩ বছরে মুনাফা-আসলে ১৫২০০ টাকা এবং ৫ বছরে মুনাফা-আসলে ১৭৬০০ টাকা হয়[cite: 25, 26]।",
              "ka": "২ বছরের মুনাফা কত তা নির্ণয় কর[cite: 26].",
              "kha": "আসল ও বার্ষিক মুনাফার হার নির্ণয় কর[cite: 26].",
              "ga": "উক্ত আসলে একই হারে ৫ বছরের জন্য সঞ্চয় করলে চক্রবৃদ্ধি মুনাফা কত হবে?[cite: 26]",
              "gha": "\"নির্দিষ্ট হারে সরল মুনাফার চেয়ে চক্রবৃদ্ধি মুনাফায় আমানতকারীর লাভ বেশি।\"—গাণিতিকভাবে বিশ্লেষণ কর[cite: 26]."
            },
            {
              "stem": "উদ্দীপক ২: a + 1/a = 4 এবং x² - √5x + 1 = 0 দুটি বীজগণিতীয় সমীকরণ[cite: 26].",
              "ka": "x + 1/x এর মান কত?[cite: 26]",
              "kha": "প্রমাণ কর যে, a² + 1/a² = 14[cite: 26].",
              "ga": "দেখাও যে, a⁴ + 1/a⁴ = 194[cite: 26].",
              "gha": "\"বীজগণিতীয় অনুসিদ্ধান্ত ব্যবহার করে জটিল মান সহজে বের করা সম্ভব।\"—দ্বিতীয় সমীকরণের মান প্রয়োগ করে বিশ্লেষণ কর[cite: 26]."
            }
          ]
        }
      }
    },
    "বাংলাদেশ ও বিশ্বপরিচয়": {
      "ইতিহাস ও বিশ্বসমাজ": {
        "মুক্তিযুদ্ধ ও সামাজিক পরিবর্তন": {
          "MCQ": [
            {"q": "১৯৭১ সালের ৭ই মার্চ বঙ্গবন্ধু কোথায় ভাষণ দেন?", "options": ["ক) রেসকোর্স ময়দান", "খ) পল্টন ময়দান", "গ) শহীদ মিনার", "ঘ) বাহাদুর শাহ পার্ক"], "answer": "ক) রেসকোর্স ময়দান"},
            {"q": "বাংলাদেশের মুক্তিযুদ্ধে বীরপ্রতীক খেতাবপ্রাপ্ত একমাত্র বিদেশী নাগরিক কে?", "options": ["ক) সাইমন ড্রিং", "খ) ডব্লিউ এ এস ওডারল্যান্ড", "গ) জর্জ হ্যারিসন", "ঘ) রবি শংকর"], "answer": "খ) ডব্লিউ এ এস ওডারল্যান্ড"},
            {"q": "মুক্তিযুদ্ধের সময় সমগ্র দেশকে কয়টি সেক্টরে ভাগ করা হয়েছিল?", "options": ["ক) ৮টি", "খ) ১০টি", "গ) ১১টি", "ঘ) ৬৪টি"], "answer": "গ) ১১টি"},
            {"q": "জাতিসংঘ শিশু তহবিল (UNICEF) এর সদর দফতর কোথায় অবস্থিত?", "options": ["ক) জেনেভা", "খ) নিউ ইয়র্ক", "গ) প্যারিস", "ঘ) লন্ডন"], "answer": "খ) নিউ ইয়র্ক"},
            {"q": "বাঙালি জাতীয়তাবাদের মূল ভিত্তি কোনটি ছিল?", "options": ["ক) ধর্মীয় চেতনা", "খ) ভাষা ও সংস্কৃতির চেতনা", "গ) অর্থনৈতিক বৈষম্য", "ঘ) ভৌগোলিক অবস্থান"], "answer": "খ) ভাষা ও সংস্কৃতির চেতনা"},
            {"q": "বাংলাদেশের সংবিধানে মূলনীতি কয়টি?", "options": ["ক) ৩টি", "খ) ৪টি", "গ) ৫টি", "ঘ) ৭টি"], "answer": "খ) ৪টি"},
            {"q": "এশিয়ার বৃহত্তম অর্থনীতির দেশ কোনটি?", "options": ["ক) জাপান", "খ) চীন", "গ) ভারত", "ঘ) দক্ষিণ কোরিয়া"], "answer": "খ) চীন"},
            {"q": "বাংলাদেশে দুর্যোগ ব্যবস্থাপনার প্রধান উদ্দেশ্য কী?", "options": ["ক) ক্ষয়ক্ষতি কমানো", "খ) সাহায্য বন্ধ করা", "গ) জনসংখ্যা বাড়ানো", "ঘ) বনাঞ্চল কাটা"], "answer": "ক) ক্ষয়ক্ষতি কমানো"},
            {"q": "সামাজিকীকরণের সবচেয়ে শক্তিশালী মাধ্যম কোনটি?", "options": ["ক) পরিবার", "খ) খেলার মাঠ", "গ) গণমাধ্যম", "ঘ) ক্লাব"], "answer": "ক) পরিবার"},
            {"q": "কত সালে 'ঐতিহাসিক আগরতলা মামলা' দায়ের করা হয়?", "options": ["ক) ১৯৬৬", "খ) ১৯৬৮", "গ) ১৯৬৯", "ঘ) ১৯৭০"], "answer": "খ) ১৯৬৮"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ১৯৭১ সালে পাকিস্তানি বাহিনীর গণহত্যা ও নির্যাতনের বিরুদ্ধে বাঙালি জাতি দল-মত নির্বিশেষে রুখে দাঁড়ায়। সাধারণ মানুষ ও যৌথ বাহিনীর সম্মিলিত আক্রমণের ফলে অর্জিত হয় কাঙিক্ষত স্বাধীনতা[cite: 29].",
              "ka": "মুজিবনগর সরকার কত তারিখে শপথ গ্রহণ করে?[cite: 29]",
              "kha": "অপারেশন সার্চলাইট বলতে কী বোঝায়—ব্যাখ্যা কর।[cite: 29]",
              "ga": "উদ্দীপকে বর্ণিত মুক্তি সংগ্রামে সর্বস্তরের জনগণের ভূমিকার কারণ আলোচনা কর[cite: 29].",
              "gha": "\"বাঙালির জাতীয় ঐক্য ও আত্মত্যাগই ছিল ১৯৭১ সালের বিজয়ের মূল চাবিকাঠি।\"—মূল্যায়ন কর[cite: 29]."
            },
            {
              "stem": "উদ্দীপক ২: জলবায়ু পরিবর্তনের ফলে সুন্দরবনের ম্যানগ্রোভ বনাঞ্চল হুমকির মুখে পড়েছে এবং উপকূলীয় এলাকায় ঘন ঘন ঘূর্ণিঝড় ও জলোচ্ছ্বাসের প্রাদুর্ভাব বাড়ছে[cite: 29, 30].",
              "ka": "গ্রিনহাউস প্রতিক্রিয়া কী?[cite: 30]",
              "kha": "ভূগোল ও পরিবেশের ভারসাম্য রক্ষায় বনাঞ্চলের গুরুত্ব লেখ[cite: 30].",
              "ga": "উদ্দীপকে বর্ণিত ঝুঁকির কারণ পাঠ্যবইয়ের পরিবেশ ও দুর্যোগ অধ্যায়ের আলোকে ব্যাখ্যা কর[cite: 30].",
              "gha": "\"পরিবেশগত দুর্যোগ প্রশমনে বৈশ্বিক সহযোগিতা ও স্থানীয় সচেতনতা অপরিহার্য।\"—উক্তিটি বিশ্লেষণ কর[cite: 30]."
            }
          ]
        }
      }
    },
    "বিজ্ঞান": {
      "পদার্থ ও জীববিজ্ঞান": {
        "প্রাণীজগতের শ্রেণীবিন্যাস ও মৌল-রাসায়নিক বিক্রিয়া": {
          "MCQ": [
            {"q": "প্রাণীজগতের সর্ববৃহৎ পর্ব কোনটি?", "options": ["ক) Annelida", "খ) Arthropoda", "গ) Mollusca", "ঘ) Chordata"], "answer": "খ) Arthropoda"},
            {"q": "শ্রেণীবিন্যাসের জনক কাকে বলা হয়?", "options": ["ক) এরিস্টটল", "খ) ক্যারোলাস লিনিয়াস", "গ) থিওফ্রাস্টাস", "ঘ) জন রে"], "answer": "খ) ক্যারোলাস লিনিয়াস"},
            {"q": "পরমাণুর নিউক্লিয়াসে কোন কণাগুলো থাকে?", "options": ["ক) ইলেকট্রন ও প্রোটন", "খ) প্রোটন ও নিউট্রন", "গ) ইলেকট্রন ও নিউট্রন", "ঘ) কেবল ইলেকট্রন"], "answer": "খ) প্রোটন ও নিউট্রন"},
            {"q": "হাইড্রোজেনের প্রতীক কোনটি?", "options": ["ক) He", "খ) H", "গ) Hg", "ঘ) Na"], "answer": "খ) H"},
            {"q": "কোষ বিভাজনের কোন ধাপে ক্রোমোজোমগুলো বিষুবীয় অঞ্চলে অবস্থান করে?", "options": ["ক) প্রোফেজ", "খ) মেটাফেজ", "গ) এনাফেজ", "ঘ) টেলোফেজ"], "answer": "খ) মেটাফেজ"},
            {"q": "মানুষের লোহিত রক্তকণিকার গড় আয়ু কত দিন?", "options": ["ক) ৬০ দিন", "খ) ১২০ দিন", "গ) ১৫০ দিন", "ঘ) ১৮০ দিন"], "answer": "খ) ১২০ দিন"},
            {"q": "এসিড নীল লিটমাসকে কোন রঙে পরিণত করে?", "options": ["ক) লাল", "খ) সবুজ", "গ) হলুদ", "ঘ) বেগুনি"], "answer": "ক) লাল"},
            {"q": "পৃথিবীর কেন্দ্রে অভিকর্ষজ ত্বরণ (g) এর মান কত?", "options": ["ক) ৯.৮ m/s²", "খ) ০ m/s²", "গ) ৯.৮১ m/s²", "ঘ) ১০ m/s²"], "answer": "খ) ০ m/s²"},
            {"q": "উদ্ভিদের বৃদ্ধি ও কোষ বিভাজনে সাহায্যকারী প্রধান অঙ্গাণু কোনটি?", "options": ["ক) মাইটোকন্ড্রিয়া", "খ) নিউক্লিয়াস", "গ) গলগি বডি", "ঘ) লাইসোসোম"], "answer": "খ) নিউক্লিয়াস"},
            {"q": "নিচের কোনটি সংযোজন বিক্রিয়ার উদাহরণ?", "options": ["ক) C + O₂ → CO₂", "খ) CaCO₃ → CaO + CO₂", "গ) Zn + H₂SO₄ → ZnSO₄ + H₂", "ঘ) HCl + NaOH → NaCl + H₂O"], "answer": "ক) C + O₂ → CO₂"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সিয়াম পুকুর থেকে কেঁচো, চিংড়ি এবং একটি মাছ সংগ্রহ করল। শিক্ষক বললেন, প্রাণী তিনটির দৈহিক গঠন ও মেরুদণ্ডের উপস্থিতি ভিন্ন হওয়ার কারণে এরা আলাদা পর্বের অন্তর্ভুক্ত[cite: 27].",
              "ka": "শ্রেণীবিন্যাসের সর্বনিম্ন ধাপ কোনটি?[cite: 27]",
              "kha": "অর্থোপোডা পর্বের প্রাণীদের দুটি প্রধান বৈশিষ্ট্য লেখ[cite: 27].",
              "ga": "উদ্দীপকের সংগৃহীত কেঁচো ও চিংড়ির বৈশিষ্ট্যগত পার্থক্য ব্যাখ্যা কর[cite: 27].",
              "gha": "\"প্রাণীজগতের বৈচিত্র্য অনুধাবনে শ্রেণীবিন্যাসের ভূমিকা অপরিসীম।\"—উদ্দীপকের প্রাণীদের আলোকে মূল্যায়ন কর[cite: 27]."
            },
            {
              "stem": "উদ্দীপক ২: একটি মৌলের পারমাণবিক সংখ্যা ১১ এবং ভরসংখ্যা ২৩। মৌলটি পানির সাথে বিক্রিয়া করে হাইড্রোজেন গ্যাস উৎপন্ন করে[cite: 27, 28].",
              "ka": "পারমাণবিক সংখ্যা কাকে বলে?[cite: 27]",
              "kha": "আইসোটোপ বলতে কী বোঝায়—উদাহরণসহ লেখ[cite: 28].",
              "ga": "উদ্দীপকের মৌলটির ইলেকট্রন, প্রোটন ও নিউট্রন সংখ্যা নির্ণয় কর[cite: 28].",
              "gha": "\"নিউক্লিয়াসের প্রোটন ও নিউট্রন সংখ্যাই পরমাণুর রাসায়নিক পরিচয় ও বৈশিষ্ট্য নির্ধারণ করে।\"—বিশ্লেষণ কর[cite: 28]."
            }
          ]
        }
      }
    },
    "ইসলাম ও নৈতিক শিক্ষা": {
      "আকাঈদ ও আখলাক": {
        "ঈমান, ইবাদত ও নৈতিকতা": {
          "MCQ": [
            {"q": "'আকাঈদ' শব্দটি কোন শব্দ থেকে এসেছে?", "options": ["ক) আকীদাহ", "খ) আমল", "গ) ইসলাম", "ঘ) তাওহীদ"], "answer": "ক) আকীদাহ"},
            {"q": "ইসলামের পঞ্চস্তম্ভের দ্বিতীয় স্তম্ভ কোনটি?", "options": ["ক) সাওম", "খ) সালাত", "গ) যাকাত", "ঘ) হজ"], "answer": "খ) সালাত"},
            {"q": "নেফাক শব্দের অর্থ কী?", "options": ["ক) কাপট্য/প্রতারণা", "খ) সত্যবাদিতা", "গ) আমানতদারী", "ঘ) ধৈর্য"], "answer": "ক) কাপট্য/প্রতারণা"},
            {"q": "যে ব্যক্তি নেফাক করে তাকে কী বলা হয়?", "options": ["ক) কাফির", "খ) মুনাফিক", "গ) মুশরিক", "ঘ) ফাসিক"], "answer": "খ) মুনাফিক"},
            {"q": "কুরআন মজিদ নাজিল হতে কত বছর সময় লেগেছিল?", "options": ["ক) ২০ বছর", "খ) ২২ বছর", "গ) ২৩ বছর", "ঘ) ২৫ বছর"], "answer": "গ) ২৩ বছর"},
            {"q": "আখলাক-এ যমীমাহ বলতে কী বোঝায়?", "options": ["ক) সৎ স্বভাব", "খ) নিন্দনীয়/অসৎ চরিত্র", "গ) দানশীলতা", "ঘ) সত্যবাদিতা"], "answer": "খ) নিন্দনীয়/অসৎ চরিত্র"},
            {"q": "গীবত শব্দের অর্থ কী?", "options": ["ক) পরচর্চা/পরনিন্দা", "খ) হিংসা", "গ) অহংকার", "ঘ) গালি দেওয়া"], "answer": "ক) পরচর্চা/পরনিন্দা"},
            {"q": "হাদিস অনুসারে 'উত্তম চরিত্রের অধিকারী ব্যক্তি' কে?", "options": ["ক) যার ধনসম্পদ বেশি", "খ) যার নৈতিক চরিত্র সবচেয়ে সুন্দর", "গ) যে অনেক বেশি কথা বলে", "ঘ) যে বেশি ভ্রমণ করে"], "answer": "খ) যার নৈতিক চরিত্র সবচেয়ে সুন্দর"},
            {"q": "যাকাত প্রদানের মাধ্যমে ধনীদের সম্পদ-", "options": ["ক) কমে যায়", "খ) পবিত্র ও বৃদ্ধি পায়", "গ) নষ্ট হয়", "ঘ) অপরিবর্তিত থাকে"], "answer": "খ) পবিত্র ও বৃদ্ধি পায়"},
            {"q": "পরিবেশ পরিচ্ছন্ন রাখা ইসলামে কোনটির অন্তর্ভুক্ত?", "options": ["ক) ঈমানের অঙ্গ", "খ) ফরজ নয়", "গ) অপ্রয়োজনীয় কাজ", "ঘ) কেবল সামাজিক দায়িত্ব"], "answer": "ক) ঈমানের অঙ্গ"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: জামিল সাহেব সবসময় মুখে মিষ্টি কথা বলেন, কিন্তু কাজের বেলায় ওয়াদা বরখেলাপ করেন এবং বন্ধুদের গোপন কথা অন্যের কাছে ফাঁস করে দেন। অন্যদিকে তার ভাই খালিদ কথা দিলে তা রক্ষা করেন এবং সততার সাথে জীবনযাপন করেন।",
              "ka": "মুনাফিকের নিদর্শন কয়টি?",
              "kha": "নিফাক মানুষের আখিরাতের জীবনের জন্য চরম ক্ষতিকর কেন—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের জামিল সাহেবের আচরণের মধ্যে কোন সামাজিক অপরাধটি পরিলক্ষিত হয়—ব্যাখ্যা কর।",
              "gha": "\"খালিদের মতো আমানতদারী ও ওয়াদা পালনের মনোভাবই সমাজে শান্তি প্রতিষ্ঠা করতে পারে।\"—পাঠ্যবইয়ের আলোকে বক্তব্যটি পর্যালোচনা কর।"
            },
            {
              "stem": "উদ্দীপক ২: আরিয়ান তার উপার্জিত অর্থের একটি নির্দিষ্ট অংশ প্রতি বছর দরিদ্র স্বজন ও এলাকার অসহায়দের মধ্যে বণ্টন করে দেয়। এতে তার সম্পত্তির পরিমাণ কমে না গিয়ে বরকত ও পবিত্রতা বৃদ্ধি পায়।",
              "ka": "যাকাত শব্দের অর্থ কী?",
              "kha": "ইসলামে যাকাত ব্যবস্থার অর্থনৈতিক গুরুত্ব বুঝিয়ে লেখ।",
              "ga": "উদ্দীপকের আরিয়ানের কাজে ইসলামের কোন ফরজ বিধানটি পালিত হয়েছে—ব্যাখ্যা কর।",
              "gha": "\"যাকাত ভিত্তিক অর্থনীতি সমাজে ধনী ও দরিদ্রের বৈষম্য দূর করতে ভূমিকা রাখে।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "তথ্য ও যোগাযোগ প্রযুক্তি": {
      "ই-লার্নিং ও বিশ্বগ্রাম": {
        "তথ্যপ্রযুক্তির আধুনিক প্রয়োগ": {
          "MCQ": [
            {"q": "বিশ্বগ্রাম (Global Village) ধারণার প্রবক্তা কে?", "options": ["ক) বিল গেটস", "খ) মার্শাল ম্যাকলুহান", "গ) স্টিভ জবস", "ঘ) মার্ক জুকারবার্গ"], "answer": "খ) মার্শাল ম্যাকলুহান"},
            {"q": "ই-লার্নিং এর মূল ভিত্তি কোনটি?", "options": ["ক) ইন্টারনেট ও কম্পিউটার প্রযুক্তি", "খ) কেবল বই", "গ) রেডিও", "ঘ) টেলিভিশন"], "answer": "ক) ইন্টারনেট ও কম্পিউটার প্রযুক্তি"},
            {"q": "নিচের কোনটি ই-কমার্স (E-commerce) এর উদাহরণ?", "options": ["ক) অনলাইনে পণ্য কেনাবেচা", "খ) ই-মেইল পাঠানো", "গ) গান শোনা", "ঘ) গেম খেলা"], "answer": "ক) অনলাইনে পণ্য কেনাবেচা"},
            {"q": "Wi-Fi এর পূর্ণরূপ কী?", "options": ["ক) Wireless Fidelity", "খ) Wired Filter", "গ) Wide Field", "ঘ) Web Finance"], "answer": "ক) Wireless Fidelity"},
            {"q": "কম্পিউটার নেটওয়ার্কের সবচেয়ে বড় ক্ষেত্র কোনটি?", "options": ["ক) PAN", "খ) LAN", "গ) MAN", "ঘ) WAN"], "answer": "ঘ) WAN"},
            {"q": "অপটিক্যাল ফাইবারের মধ্য দিয়ে কী প্রবাহিত হয়?", "options": ["ক) বিদ্যুৎ সংকেত", "খ) আলোর সংকেত", "গ) শব্দ তরঙ্গ", "ঘ) চুম্বক ক্ষেত্র"], "answer": "খ) আলোর সংকেত"},
            {"q": "ভিডিও কনফারেন্সিং এর প্রধান সুবিধা কী?", "options": ["A) দূর থেকে দেখা ও কথা বলা", "খ) কম খরচে প্রিন্ট করা", "গ) দ্রুত টাইপ করা", "ঘ) হিসাব করা"], "answer": "A) দূর থেকে দেখা ও কথা বলা"},
            {"q": "ইন্টারনেটে ক্ষতিকর সফটওয়্যার বা ম্যালওয়্যার থেকে রক্ষা পেতে কোনটি ব্যবহার করা হয়?", "options": ["ক) অ্যান্টিভাইরাস", "খ) এমএস ওয়ার্ড", "গ) ব্রাউজার", "ঘ) ক্যালকুলেটর"], "answer": "ক) অ্যান্টিভাইরাস"},
            {"q": "তথ্য ও যোগাযোগ প্রযুক্তির প্রধান হাতিয়ার কোনটি?", "options": ["ক) কম্পিউটার", "খ) সাইকেল", "গ) ঘড়ি", "ঘ) রেডিও"], "answer": "ক) কম্পিউটার"},
            {"q": "নিচের কোনটি আউটপুট ডিভাইস?", "options": ["ক) কিবোর্ড", "খ) মাউস", "গ) প্রিন터", "ঘ) স্ক্যানার"], "answer": "গ) প্রিন터"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: রনি ঘরে বসেই ল্যাপটপ ও ইন্টারনেটের সাহায্যে বিদেশি একটি বিশ্ববিদ্যালয়ের অনলাইন কোর্সে অংশ নিচ্ছে, পরীক্ষা দিচ্ছে এবং ডিজিটাল লাইব্রেরি থেকে বই পড়ছে।",
              "ka": "ই-লার্নিং কী?",
              "kha": "শিক্ষাক্ষেত্রে ইন্টারনেটের দুটি সুফল ব্যাখ্যা কর।",
              "ga": "উদ্দীপকে রনির শিক্ষাগ্রহণের মাধ্যমটি পাঠ্যবইয়ের ই-লার্নিং ধারণার সাথে কীভাবে সম্পর্কিত—ব্যাখ্যা কর।",
              "gha": "\"ই-লার্নিং ঐতিহ্যগত শিক্ষা ব্যবস্থাকে আরও সহজ ও গতিশীল করেছে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: রহিম সাহেব ঢাকায় বসে ইন্টারনেটের মাধ্যমে দেশের বাইরে অবস্থানকারী ব্যবসায়িক অংশীদারদের সাথে লাইভ মিটিং করলেন এবং মুহূর্তের মধ্যে গুরুত্বপূর্ণ ফাইল ও চুক্তিপত্র আদান-প্রদান করলেন।",
              "ka": "গ্লোবাল ভিলেজ বা বিশ্বগ্রাম কী?",
              "kha": "ভিডিও কনফারেন্সিং এর প্রয়োজনীয়তা ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের যোগাযোগ ব্যবস্থায় আইসিটি কীভাবে ভূমিকা রাখছে তা বর্ণনা কর।",
              "gha": "\"তথ্য ও যোগাযোগ প্রযুক্তির প্রসারে পৃথিবী আজ একটি বিশ্বপল্লীতে পরিণত হয়েছে।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
        },

  "নবম শ্রেণি": {
    "বাংলা ১ম পত্র": {
      "সাহিত্য": {
        "গদ্য ও পদ্য": {
          "MCQ": [
            {"q": "'শুভা' গল্পে শুভার প্রকৃত নাম কী ছিল?", "options": ["ক) সুভাষিণী", "খ) সুকেশিনী", "গ) সুহাসিনী", "ঘ) সুচরিতা"], "answer": "ক) সুভাষিণী"},
            {"q": "বই পড়া অনুশীলনের জন্য কিসের প্রয়োজন সবচেয়ে বেশি?", "options": ["ক) লাইব্রেরি", "খ) বিদ্যালয়", "গ) বিশ্ববিদ্যালয়", "ঘ) গৃহ"], "answer": "ক) লাইব্রেরি"},
            {"q": "'বঙ্গবাণী' কবিতায় কবি কাদের প্রতি ক্ষোভ প্রকাশ করেছেন?", "options": ["ক) যারা বাংলাদেশে জন্মে বাংলা ভাষাকে অবজ্ঞা করে", "খ) যারা বিদেশি ভাষা জানে না", "গ) যারা পড়াশোনা করে না", "ঘ) যারা কাব্যচর্চা করে না"], "answer": "ক) যারা বাংলাদেশে জন্মে বাংলা ভাষাকে অবজ্ঞা করে"},
            {"q": "কপোতাক্ষ নদ কবিতার মূল সুর কোনটি?", "options": ["ক) স্বদেশপ্রেম", "খ) প্রকৃতিপ্রীতি", "গ) রোমান্টিকতা", "ঘ) স্মৃতিচারণ"], "answer": "ক) স্বদেশপ্রেম"},
            {"q": "'মানুষ' কবিতায় সাম্যবাদের বার্তা কে প্রচার করেছেন?", "options": ["ক) কাজী নজরুল ইসলাম", "খ) রবীন্দ্রনাথ ঠাকুর", "গ) জসীমউদ্দীন", "ঘ) জীবনানন্দ দাশ"], "answer": "ক) কাজী নজরুল ইসলাম"},
            {"q": "মমতাদি কত টাকা মাইনে আশা করেছিল?", "options": ["ক) ১২ টাকা", "খ) ১৫ টাকা", "গ) ২০ টাকা", "ঘ) ২৫ টাকা"], "answer": "ক) ১২ টাকা"},
            {"q": "আম আঁটির ভেঁপু গল্পের মূল চরিত্র কোনটি?", "options": ["ক) অপু ও দুর্গা", "খ) হরিহর", "গ) সর্বজয়া", "ঘ) রায়পিসি"], "answer": "ক) অপু ও দুর্গা"},
            {"q": "'পল্লীসাহিত্য' প্রবন্ধে কোন বিষয়টি সংরক্ষণের ওপর জোর দেওয়া হয়েছে?", "options": ["ক) লোকসাহিত্য ও লোকসংস্কৃতি", "খ) শহুরে সাহিত্য", "গ) উপন্যাস", "ঘ) নাট্যচর্চা"], "answer": "ক) লোকসাহিত্য ও লোকসংস্কৃতি"},
            {"q": "'জীবন-সংগীত' কবিতার মূল বার্তা কী?", "options": ["ক) কর্মময় জীবনের গুরুত্ব", "খ) অলস জীবনযাপন", "গ) বৈরাগ্য সাধন", "ঘ) সংসারের মায়া ত্যাগ"], "answer": "ক) কর্মময় জীবনের গুরুত্ব"},
            {"q": "'একাত্তরের দিনগুলি' কোন জাতীয় রচনা?", "options": ["ক) দিনলিপি/স্মৃতিচারণ", "খ) উপন্যাস", "গ) নাটক", "ঘ) ভ্রমণকাহিনী"], "answer": "ক) দিনলিপি/স্মৃতিচারণ"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সলিমুল্লাহ গ্রামে এক প্রতিবন্ধী শিশুকে গভীর মমতায় আগলে রাখেন। সমাজের মানুষ তাকে অবহেলা করলেও তিনি মনে করেন ভালোবাসাই সকল মানুষের অধিকার।",
              "ka": "শুভার বাকপ্রতিবন্ধী দুই সখীর নাম কী?",
              "kha": "শুভা প্রকৃতির মাঝে আশ্রয় খুঁজে পেয়েছিল কেন?",
              "ga": "উদ্দীপকের সলিমুল্লাহর মনোভাব 'শুভা' গল্পের কোন দিকটি ফুটিয়ে তোলে—ব্যাখ্যা কর।",
              "gha": "\"বিশেষ চাহিদাসম্পন্ন শিশুদের প্রতি সহানুভূতিশীল দৃষ্টিভঙ্গিই মানবিক সমাজ গঠনের পূর্বশর্ত।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: প্রবাসে দীর্ঘদিন অবস্থান করেও সজল তার নিজ গ্রামের নদী ও মাটির গন্ধ ভুলতে পারেন না। সুযোগ পেলেই তিনি মাতৃভূমির স্মৃতিচারণ করেন।",
              "ka": "'কপোতাক্ষ নদ' কোন ধরণের কবিতা?",
              "kha": "কবি কপোতাক্ষ নদকে স্নেহের ডোরে বাঁধার কথা বলেছেন কেন?",
              "ga": "উদ্দীপকের সজলের অনুভূতির সাথে 'কপোতাক্ষ নদ' কবিতার কবির মানসিকতার তুলনা কর।",
              "gha": "\"স্বদেশপ্রীতি ও স্বদেশের স্মৃতিকাতরতা মানুষের হৃদয়ে চির অমলিন থাকে।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "বাংলা ২য় পত্র": {
      "ব্যাকরণ ও নির্মিতি": {
        "ধ্বনিতত্ত্ব ও সমাস": {
          "MCQ": [
            {"q": "বাংলা ভাষায় মৌলিক স্বরধ্বনি কয়টি?", "options": ["ক) ৭টি", "খ) ১১টি", "গ) ৩৯টি", "ঘ) ২৫টি"], "answer": "ক) ৭টি"},
            {"q": "ণ-ত্ব ও ষ-ত্ব বিধান ব্যাকরণের কোন অংশে আলোচিত হয়?", "options": ["ক) ধ্বনিতত্ত্ব", "খ) রূপতত্ত্ব", "গ) বাক্যতত্ত্ব", "ঘ) অর্থতত্ত্ব"], "answer": "ক) ধ্বনিতত্ত্ব"},
            {"q": "পরপদের অর্থ প্রধান হয় কোন সমাসে?", "options": ["ক) তৎপুরুষ সমাস", "খ) দ্বন্দ্ব সমাস", "গ) বহুব্রীহি সমাস", "ঘ) অব্যয়ীভাব সমাস"], "answer": "ক) তৎপুরুষ সমাস"},
            {"q": "'রাজর্ষি' শব্দটির সঠিক ব্যাসবাক্য কোনটি?", "options": ["ক) যিনি রাজা তিনিই ঋষি", "খ) রাজার ঋষি", "গ) রাজা ও ঋষি", "ঘ) রাজার ন্যায় ঋষি"], "answer": "ক) যিনি রাজা তিনিই ঋষি"},
            {"q": "স্বরসন্ধির উদাহরণ কোনটি?", "options": ["ক) বিদ্যালয় (বিদ্যা + আলয়)", "খ) দিগন্ত", "গ) সংবাদ", "ঘ) নিচ্ছল"], "answer": "ক) বিদ্যালয় (বিদ্যা + আলয়)"},
            {"q": "কৃৎ প্রত্যয় সাধিত শব্দকে কী বলে?", "options": ["ক) কৃদন্ত পদ", "খ) তদ্ধিতান্ত পদ", "গ) সমাসবদ্ধ পদ", "ঘ) উপসর্গ"], "answer": "ক) কৃদন্ত পদ"},
            {"q": "বিশেষ্যের পরিবর্তে যে পদ ব্যবহৃত হয় তাকে কী বলে?", "options": ["ক) সর্বনাম পদ", "খ) বিশেষণ পদ", "গ) অব্যয় পদ", "ঘ) ক্রিয়াপদ"], "answer": "ক) সর্বনাম পদ"},
            {"q": "বাক্যের অর্থগত শ্রেণিবিভাগ কয়টি?", "options": ["ক) ৫টি", "খ) ৩টি", "গ) ৪টি", "ঘ) ২টি"], "answer": "ক) ৫টি"},
            {"q": "'হাতে খড়ি' কোন সমাসের উদাহরণ?", "options": ["ক) বহুব্রীহি সমাস (অলুক বহুব্রীহি)", "খ) তৎপুরুষ সমাস", "গ) কর্মধারয় সমাস", "ঘ) দ্বিগু সমাস"], "answer": "ক) বহুব্রীহি সমাস (অলুক বহুব্রীহি)"},
            {"q": "ণ-ত্ব বিধান অনুযায়ী কোন যুক্তবর্ণে 'ণ' হয়?", "options": ["ক) ট-বর্গের পূর্বে", "খ) ত-বর্গের পূর্বে", "গ) ক-বর্গের পূর্বে", "ঘ) চ-বর্গের পূর্বে"], "answer": "ক) ট-বর্গের পূর্বে"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ব্যাকরণগত সঠিক শব্দ প্রয়োগ ও সঠিক পদ সংযোগ ভাষার সৌন্দর্য ও ভাব প্রকাশের স্পষ্টতা বাড়ায়। সমাস ও প্রত্যয় সাধিত শব্দ ভাষাকে সংক্ষেপ করে।",
              "ka": "সমাস শব্দের অর্থ কী?",
              "kha": "দ্বন্দ্ব সমাস ও বহুব্রীহি সমাসের দুটি প্রধান পার্থক্য লেখ।",
              "ga": "উদ্দীপকের আলোকে পদসংক্ষেপণে সমাসের ভূমিকা ব্যাখ্যা কর।",
              "gha": "\"শব্দ গঠনের প্রক্রিয়া জানা ছাড়া প্রমিত ভাষারীতি অনুসরণ করা সম্ভব নয়।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: ধ্বনির সঠিক উচ্চারণ ও যুক্তবর্ণের নিয়ম না জানলে লেখার ক্ষেত্রে ভুল হওয়ার সম্ভাবনা থাকে। ণ-ত্ব ও ষ-ত্ব বিধান তৎসম শব্দ ব্যবহারের সঠিক নিয়ম নির্ধারণ করে।",
              "ka": "ষ-ত্ব বিধান বলতে কী বোঝায়?",
              "kha": "কোন কোন ক্ষেত্রে ণ-ত্ব বিধান প্রযোজ্য হয় না—উদাহরণসহ লেখ।",
              "ga": "উদ্দীপকে উল্লিখিত বিধান দুটি বাংলা বানানের শুদ্ধতায় কী ভূমিকা রাখে তা ব্যাখ্যা কর।",
              "gha": "\"বাংলা ভাষার প্রমিত রূপ বজায় রাখতে ব্যাকরণের নিয়ম মানা অপরিহার্য।\"—উক্তিটির সত্যতা যাচাই কর।"
            }
          ]
        }
      }
    },
    "English 1st Paper": {
      "Reading & Writing": {
        "Unit 1: Good Citizens & Environment": {
          "MCQ": [
            {"q": "What is the primary responsibility of a good citizen?", "options": ["a) Obeying laws and helping others", "b) Accumulating wealth", "c) Living in isolation", "d) Ignoring society"], "answer": "a) Obeying laws and helping others"},
            {"q": "What causes environmental pollution significantly?", "options": ["a) Industrial waste and deforestation", "b) Tree plantation", "c) Clean energy", "d) Organic farming"], "answer": "a) Industrial waste and deforestation"},
            {"q": "Which one is a renewable energy source?", "options": ["a) Solar power", "b) Coal", "c) Petroleum", "d) Natural gas"], "answer": "a) Solar power"},
            {"q": "What is the synonym of 'Responsibility'?", "options": ["a) Duty", "b) Neglect", "c) Freedom", "d) Avoidance"], "answer": "a) Duty"},
            {"q": "Antonym of 'Destroy' is-", "options": ["a) Create", "b) Damage", "c) Ruin", "d) Harm"], "answer": "a) Create"},
            {"q": "Global warming is mainly caused by-", "options": ["a) Greenhouse gases", "b) Oxygen", "c) Fresh water", "d) Reforestation"], "answer": "a) Greenhouse gases"},
            {"q": "Which organization works for global peace and cooperation?", "options": ["a) UNO", "b) FIFA", "c) ICC", "d) WTO"], "answer": "a) UNO"},
            {"q": "What does 'E-learning' stand for?", "options": ["a) Electronic Learning", "b) Easy Learning", "c) Early Learning", "d) Essential Learning"], "answer": "a) Electronic Learning"},
            {"q": "Choose the correct sentence:", "options": ["a) Honesty is the best policy.", "b) Honesty is a best policy.", "c) Honesty is best policy.", "d) Honesty is an best policy."], "answer": "a) Honesty is the best policy."},
            {"q": "What is the key factor to success according to good habits?", "options": ["a) Perseverance and discipline", "b) Laziness", "c) Luck only", "d) Procrastination"], "answer": "a) Perseverance and discipline"}
          ],
          "CQ": [
            {
              "stem": "Read the text: To be a good citizen, a person needs to acquire knowledge, skill, and a good attitude towards society. Good citizens respect laws and contribute to environmental conservation.",
              "ka": "What qualities make a person a good citizen?",
              "kha": "Why is obeying laws essential for social harmony?",
              "ga": "Summarize the key duties of a good citizen based on the passage.",
              "gha": "\"Individual responsibility leads to national growth.\" Evaluate the statement in light of the text."
            },
            {
              "stem": "Read the text: Deforestation and excessive carbon emissions are threatening global climate stability. Planting trees and using renewable energy can save our planet.",
              "ka": "What is the main cause of climate change?",
              "kha": "How can tree plantation help combat global warming?",
              "ga": "Explain the importance of adopting renewable energy sources.",
              "gha": "\"Environmental protection is not an option, but a necessity for human survival.\" Justify."
            }
          ]
        }
      }
    },
    "English 2nd Paper": {
      "Grammar & Composition": {
        "Grammar Rules & Transformation": {
          "MCQ": [
            {"q": "Change the voice: 'She writes a letter.'", "options": ["a) A letter is written by her.", "b) A letter was written by her.", "c) A letter has been written by her.", "d) A letter is writing by her."], "answer": "a) A letter is written by her."},
            {"q": "Which one is a Complex sentence?", "options": ["a) Though he is poor, he is honest.", "b) He is poor but honest.", "c) He is a poor and honest man.", "d) Being poor, he is honest."], "answer": "a) Though he is poor, he is honest."},
            {"q": "Identify the tag question: 'Let us go for a walk, ___?'", "options": ["a) shall we", "b) don't we", "c) won't we", "d) can we"], "answer": "a) shall we"},
            {"q": "Fill in the blank: 'He is senior ___ me.'", "options": ["a) to", "b) than", "c) from", "d) with"], "answer": "a) to"},
            {"q": "Which tense expresses an action completed in the recent past?", "options": ["a) Present Perfect", "b) Past Simple", "c) Present Continuous", "d) Future Simple"], "answer": "a) Present Perfect"},
            {"q": "Choose the correct indirect speech: He said, 'I am ill.'", "options": ["a) He said that he was ill.", "b) He said that I was ill.", "c) He says that he is ill.", "d) He said that he is ill."], "answer": "a) He said that he was ill."},
            {"q": "Noun form of the verb 'Know' is-", "options": ["a) Knowledge", "b) Known", "c) Knowing", "d) Knowable"], "answer": "a) Knowledge"},
            {"q": "Identify the degree: 'No other boy in the class is as good as Rahi.'", "options": ["a) Positive", "b) Comparative", "c) Superlative", "d) None"], "answer": "a) Positive"},
            {"q": "Choose the correct right form of verb: 'Neither he nor his friends ___ present.'", "options": ["a) were", "b) was", "c) is", "d) has"], "answer": "a) were"},
            {"q": "What is the antonym of 'Optimistic'?", "options": ["a) Pessimistic", "b) Hopeful", "c) Confident", "d) Positive"], "answer": "a) Pessimistic"}
          ],
          "CQ": [
            {
              "stem": "Transform the following sentences as directed: (a) Bangladesh is a beautiful country. (Exclamatory) (b) It is richer in natural beauty than most other countries. (Superlative) (c) Who does not love his motherland? (Assertive) (d) We should protect our nature. (Passive)",
              "ka": "What is a compound sentence?",
              "kha": "Explain the rule of converting Comparative degree into Superlative degree.",
              "ga": "Transform sentences (a), (b), (c), and (d) according to instructions.",
              "gha": "\"Mastery over sentence transformation improves overall writing clarity.\" Analyze."
            },
            {
              "stem": "Complete the passage with suitable verbs in correct forms: Education (a) ___ (enlighten) the mind. It (b) ___ (help) us distinguish between right and wrong. A person who (c) ___ (be) educated contributes to national progress.",
              "ka": "What is Subject-Verb Agreement?",
              "kha": "State the rule for using Present Perfect Tense with 'recently' or 'already'.",
              "ga": "Supply the correct form of verbs for (a), (b), and (c).",
              "gha": "\"Grammatical correctness is the foundation of effective language skills.\" Discuss."
            }
          ]
        }
      }
    },
    "গণিত": {
      "বীজগণিত ও জ্যামিতি": {
        "বীজগণিতীয় রাশি ও সেটের ধারণা": {
          "MCQ": [
            {"q": "A = {1, 2, 3} হলে সেট A-এর উপসেট (Subset) সংখ্যা কত?", "options": ["ক) ৮টি", "খ) ৬টি", "গ) ৪টি", "ঘ) ২টি"], "answer": "ক) ৮টি"},
            {"q": "(a + b)³ এর সঠিক সূত্র কোনটি?", "options": ["ক) a³ + 3a²b + 3ab² + b³", "খ) a³ - 3a²b + 3ab² - b³", "গ) a³ + b³ + ab", "ঘ) a³ + 3ab + b³"], "answer": "ক) a³ + 3a²b + 3ab² + b³"},
            {"q": "ফাঁকা সেটকে (Empty Set) কোন প্রতীক দিয়ে প্রকাশ করা হয়?", "options": ["ক) ∅", "খ) ∈", "গ) ⊂", "ঘ) ∪"], "answer": "ক) ∅"},
            {"q": "log₁₀ 100 এর মান কত?", "options": ["ক) ২", "খ) ১০", "গ) ১", "ঘ) ০"], "answer": "ক) ২"},
            {"q": "2x + 3y = 12 এবং x - y = 1 সমীকরণের সমাধান কোনটি?", "options": ["ক) (3, 2)", "খ) (2, 3)", "গ) (4, 1)", "ঘ) (1, 4)"], "answer": "ক) (3, 2)"},
            {"q": "একটি ত্রিভুজের তিন কোণের যোগফল কত ডিগ্রি?", "options": ["ক) ১৮০°", "খ) ৯০°", "গ) ৩৬০°", "ঘ) ২৭০°"], "answer": "ক) ১৮০°"},
            {"q": "বৃত্তের বৃহত্তম জ্যা-কে কী বলা হয়?", "options": ["ক) ব্যাস", "খ) ব্যাসার্ধ", "গ) পরিধি", "ঘ) চাপ"], "answer": "ক) ব্যাস"},
            {"q": "x² - 5x + 6 = 0 সমীকরণের মূলদ্বয় কোনটি?", "options": ["ক) 2, 3", "খ) -2, -3", "গ) 1, 6", "ঘ) -1, -6"], "answer": "ক) 2, 3"},
            {"q": "বৃত্তস্থ চতুর্ভুজের বিপরীত কোণদ্বয়ের যোগফল কত?", "options": ["ক) ১৮০°", "খ) ৯০°", "গ) ৩৬০°", "ঘ) ২৭০°"], "answer": "ক) ১৮০°"},
            {"q": "a⁰ এর মান কত (যেখানে a ≠ 0)?", "options": ["ক) ১", "খ) ০", "গ) a", "ঘ) অসংজ্ঞায়িত"], "answer": "ক) ১"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: U = {1, 2, 3, 4, 5, 6, 7}, A = {2, 4, 6} এবং B = {3, 5, 6} তিনটি সেট।",
              "ka": "A ∩ B নির্ণয় কর।",
              "kha": "A ∪ B এবং (A ∪ B)′ নির্ণয় কর।",
              "ga": "ডি মরগানের নিয়মঃ (A ∪ B)′ = A′ ∩ B′ সত্যতা যাচাই কর।",
              "gha": "\"ভেনচিত্র ও সেট তত্ত্ব গাণিতিক বিন্যাসকে সহজতর করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: x + 1/x = 3 একটি বীজগণিতীয় সমীকরণ।",
              "ka": "x - 1/x এর মান কত?",
              "kha": "প্রমাণ কর যে, x³ + 1/x³ = 18।",
              "ga": "x⁴ + 1/x⁴ এর মান নির্ণয় কর।",
              "gha": "\"বীজগণিতীয় সূত্রের সঠিক রূপান্তর জটিল রাশিমালা সমাধানে সহায়ক।\"—গাণিতিকভাবে প্রমাণ কর।"
            }
          ]
        }
      }
    },
    "বিজ্ঞান": {
      "পদার্থ ও জীববিজ্ঞান": {
        "বল, শক্তি ও জীবনের গঠন": {
          "MCQ": [
            {"q": "নিউটন-এর গতির দ্বিতীয় সূত্র থেকে কিসের পরিমাপ পাওয়া যায়?", "options": ["ক) বল", "খ) ভরবেগ", "গ) কাজ", "ঘ) ত্বরণ"], "answer": "ক) বল"},
            {"q": "কাজ করার সামর্থ্যকে কী বলে?", "options": ["ক) শক্তি", "খ) ক্ষমতা", "গ) বল", "ঘ) বেগ"], "answer": "ক) শক্তি"},
            {"q": "উদ্ভিদের শালোকসংশ্লেষণ প্রক্রিয়ায় উৎপন্ন প্রধান উপাদান কোনটি?", "options": ["ক) গ্লুকোজ ও অক্সিজেন", "খ) নাইট্রোজেন", "গ) কার্বন ডাই অক্সাইড", "ঘ) মিথেন"], "answer": "ক) গ্লুকোজ ও অক্সিজেন"},
            {"q": "মানবদেহের প্রধান রেচন অঙ্গ কোনটি?", "options": ["ক) বৃক্ক (Kidney)", "খ) যকৃৎ", "গ) ফুসফুস", "ঘ) ত্বক"], "answer": "ক) বৃক্ক (Kidney)"},
            {"q": "মহাকর্ষীয় ধ্রুবক (G) এর একক কোনটি?", "options": ["ক) N m² kg⁻²", "খ) N m kg⁻¹", "গ) N m/s", "ঘ) m/s²"], "answer": "ক) N m² kg⁻²"},
            {"q": "কোষের বংশগতির ধারক ও বাহক কাকে বলা হয়?", "options": ["ক) ক্রোমোজোম/DNA", "খ) আরএনএ", "গ) প্লাস্টিড", "ঘ) সাইটোপ্লাজম"], "answer": "ক) ক্রোমোজোম/DNA"},
            {"q": "শব্দ সঞ্চালনের বেগ সবচেয়ে বেশি কোন মাধ্যমে?", "options": ["ক) কঠিন", "খ) তরল", "গ) বায়বীয়", "ঘ) শূন্যস্থান"], "answer": "ক) কঠিন"},
            {"q": "নিচের কোনটি পরিপাকতন্ত্রের অংশ?", "options": ["ক) পাকস্থলী", "খ) ধমনি", "গ) নিউরন", "ঘ) ট্রাকিয়া"], "answer": "ক) পাকস্থলী"},
            {"q": "শক্তির এসআই (SI) একক কোনটি?", "options": ["ক) জুল (Joule)", "খ) ওয়াট", "গ) পাউণ্ড", "ঘ) নিউটন"], "answer": "ক) জুল (Joule)"},
            {"q": "নিউক্লিয়াস আবিষ্কার করেন কে?", "options": ["ক) রবার্ট ব্রাউন", "খ) রবার্ট হুক", "গ) ডাল্টন", "ঘ) মেনডেল"], "answer": "ক) রবার্ট ব্রাউন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ১০ কেজি ভরের একটি বস্তুকে ৫ m/s² ত্বরণে ধাক্কা দেওয়া হলো। বস্তুটি ২০ মিটার দূরত্ব অতিক্রম করে থেমে গেল।",
              "ka": "বল কাকে বলে?",
              "kha": "নিউটন ও ডাইনের মধ্যে সম্পর্ক ব্যাখ্যা কর।",
              "ga": "প্রযুক্ত বলের মান হিসাব কর।",
              "gha": "\"শক্তির রূপান্তর ও কাজের পরিমাণ প্রযুক্ত বলের ওপর নির্ভরশীল।\"—উদ্দীপকের তথ্যের আলোকে গাণিতিক বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: উদ্ভিদ সূর্যালোক ও কার্বন ডাই অক্সাইড ব্যবহার করে শর্করা জাতীয় খাদ্য তৈরি করে। অপরদিকে মানুষ শ্বসনের মাধ্যমে সেই খাদ্য ভেঙে শক্তি লাভ করে।",
              "ka": "শালোকসংশ্লেষণ কী?",
              "kha": "শ্বসন ও শালোকসংশ্লেষণের প্রধান পার্থক্যগুলো কী কী?",
              "ga": "উদ্দীপকে বর্ণিত শক্তি উৎপাদন প্রক্রিয়াটি উদ্ভিদদেহে কীভাবে ঘটে ব্যাখ্যা কর।",
              "gha": "\"শালোকসংশ্লেষণ ও শ্বসনের পারস্পরিক সামঞ্জস্যের ওপরই পৃথিবীর জীবজগতের অস্তিত্ব টিকে রয়েছে।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "বাংলাদেশ ও বিশ্বপরিচয়": {
      "ইতিহাস ও সমাজ ব্যবস্থা": {
        "রাষ্ট্র, সংবিধান ও সমাজচিত্র": {
          "MCQ": [
            {"q": "বাংলাদেশের স্বাধীনতার ঘোষণা প্রথম কে পাঠ করেন/প্রদান করেন?", "options": ["ক) বঙ্গবন্ধু শেখ মুজিবুর রহমান", "খ) তাজউদ্দীন আহমদ", "গ) সৈয়দ নজরুল ইসলাম", "ঘ) এম এ জি ওসমানী"], "answer": "ক) বঙ্গবন্ধু শেখ মুজিবুর রহমান"},
            {"q": "বাংলাদেশের সংবিধান কার্যকর হয় কত তারিখে?", "options": ["ক) ১৬ই ডিসেম্বর ১৯৭২", "খ) ২৬শে মার্চ ১৯৭২", "গ) ১৭ই এপ্রিল ১৯৭১", "ঘ) ২৬শে নভেম্বর ১৯৭২"], "answer": "ক) ১৬ই ডিসেম্বর ১৯৭২"},
            {"q": "রাষ্ট্রের উপাদান কয়টি?", "options": ["ক) ৪টি", "খ) ৩টি", "গ) ৫টি", "ঘ) ৬টি"], "answer": "ক) ৪টি"},
            {"q": "জাতিসংঘের নিরাপত্তা পরিষদের স্থায়ী সদস্য সংখ্যা কত?", "options": ["ক) ৫টি", "খ) ১০টি", "গ) ১৫টি", "ঘ) ৭টি"], "answer": "ক) ৫টি"},
            {"q": "বাঙালি সংস্কৃতিতে 'পহেলা বৈশাখ' উদযাপনের প্রবর্তক কে?", "options": ["ক) সম্রাট আকবর", "খ) লক্ষণ সেন", "গ) আলাউদ্দিন হোসেন শাহ", "ঘ) সিরাজউদ্দৌলা"], "answer": "ক) সম্রাট আকবর"},
            {"q": "বাংলাদেশের সর্বোচ্চ আইন কোনটি?", "options": ["ক) সংবিধান", "খ) দণ্ডবিধি", "গ) দেওয়ানি আইন", "ঘ) প্রথাগত আইন"], "answer": "ক) সংবিধান"},
            {"q": "সামাজিক নিয়মনীতি লঙ্ঘনের ফলে সমাজে কী সৃষ্টি হয়?", "options": ["ক) সামাজিক অবক্ষয় ও বিশৃঙ্খলা", "খ) উন্নয়ন", "গ) প্রবৃদ্ধি", "ঘ) সমতা"], "answer": "ক) সামাজিক অবক্ষয় ও বিশৃঙ্খলা"},
            {"q": "টেকসই উন্নয়ন লক্ষ্যমাত্রা (SDG) এর লক্ষ্য সংখ্যা কতটি?", "options": ["ক) ১৭টি", "খ) ৮টি", "গ) ১২টি", "ঘ) ২০টি"], "answer": "ক) ১৭টি"},
            {"q": "বাংলাদেশের জাতীয় সংসদের আসন সংখ্যা কতটি?", "options": ["ক) ৩৫০টি", "খ) ৩০০টি", "গ) ৪০০টি", "ঘ) ৩২০টি"], "answer": "ক) ৩৫০টি"},
            {"q": "অর্থনৈতিক উন্নয়নের অন্যতম প্রধান সূচক কোনটি?", "options": ["ক) মাথাপিছু আয়", "খ) মোট জনসংখ্যা", "গ) জমির পরিমাণ", "ঘ) আমদানির পরিমাণ"], "answer": "ক) মাথাপিছু আয়"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ১৯৭২ সালে প্রণীত বাংলাদেশের সংবিধানের অন্যতম মূলনীতি হলো গণতন্ত্র, সমাজতন্ত্র, ধর্মনিরপেক্ষতা ও জাতীয়তাবাদ। এই সংবিধান আইনের শাসন প্রতিষ্ঠায় মূল ভিত্তি হিসেবে কাজ করছে।",
              "ka": "বাংলাদেশের সংবিধান প্রণয়ন কমিটির প্রধান কে ছিলেন?",
              "kha": "সংবিধানকে রাষ্ট্রের সর্বোচ্চ দলিল বলা হয় কেন?",
              "ga": "উদ্দীপকে বর্ণিত চার মূলনীতির আলোকে আমাদের গণতান্ত্রিক ব্যবস্থার ব্যাখ্যা দাও।",
              "gha": "\"সংবিধান রক্ষা করা এবং সংবিধানের অনুশাসন মেনে চলা প্রতিটি নাগরিকের মৌলিক দায়িত্ব।\"—আলোচনা কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি সমাজে যুবসমাজের একটি অংশ উপযুক্ত শিক্ষার অভাবে বিভিন্ন সামাজিক অপরাধ ও মাদকে জড়িয়ে পড়ছে। স্থানীয় সচেতন নাগরিকেরা তাদের পুনর্বাসনে উদ্যোগ গ্রহণ করেছেন।",
              "ka": "সামাজিক ব্যাধি কী?",
              "kha": "কিশোর অপরাধের প্রধান কারণগুলো ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সমস্যার জন্য সমাজ কাঠামোর কোন ঘাটতি দায়ী তা আলোচনা কর।",
              "gha": "\"সামাজিক সচেতনতা ও সঠিক শিক্ষা প্রয়োগই অপরাধমুক্ত সমাজ গঠনের পথ দেখাতে পারে।\"—মতামত দাও।"
            }
          ]
        }
      }
    },
    "রসায়ন": {
      "বিজ্ঞান শাখা": {
        "পদার্থের গঠন ও রাসায়নিক বন্ধন": {
          "MCQ": [
            {"q": "সোডিয়াম (Na) এর পারমাণবিক সংখ্যা কত?", "options": ["ক) ১১", "খ) ১২", "গ) ১৩", "ঘ) ১৯"], "answer": "ক) ১১"},
            {"q": "নিচের কোনটি নিষ্ক্রিয় গ্যাস?", "options": ["ক) আর্গন (Ar)", "খ) অক্সিজেন (O₂)", "গ) ক্লোরিন (Cl₂)", "ঘ) নাইট্রোজেন (N₂)"], "answer": "ক) আর্গন (Ar)"},
            {"q": "আয়নিক বন্ধন গঠিত হয় কাদের মধ্যে?", "options": ["ক) ধাতু ও অধাতু", "খ) দুই অধাতু", "গ) দুই ধাতু", "ঘ) নিষ্ক্রিয় গ্যাস"], "answer": "ক) ধাতু ও অধাতু"},
            {"q": "পানি (H₂O) অনুতে কোন ধরণের বন্ধন বিদ্যমান?", "options": ["ক) সমযোজী বন্ধন", "খ) আয়নিক বন্ধন", "গ) ধাতু বন্ধন", "ঘ) হাইড্রোজেন বন্ধন"], "answer": "ক) সমযোজী বন্ধন"},
            {"q": "কার্বনের যোজনী সাধারণ অবস্থায় কত?", "options": ["ক) ৪", "খ) ২", "গ) ৩", "ঘ) ১"], "answer": "ক) ৪"},
            {"q": "১ মোল পদার্থে কতটি অণু থাকে?", "options": ["ক) ৬.০২৩ × ১০²³", "খ) ৩.০১ × ১০²³", "গ) ১.৬৬ × ১০⁻²⁴", "ঘ) ৯.৮ × ১০⁸"], "answer": "ক) ৬.০২৩ × ১০²³"},
            {"q": "পর্যায় সারণির মূল ভিত্তি কোনটি?", "options": ["ক) পারমাণবিক সংখ্যা (ইলেকট্রন বিন্যাস)", "খ) ভর সংখ্যা", "গ) গলনাঙ্ক", "ঘ) ঘনত্ব"], "answer": "ক) পারমাণবিক সংখ্যা (ইলেকট্রন বিন্যাস)"},
            {"q": "ক্ষার ধাতুর সমূহের গ্রুপ নম্বর কত?", "options": ["ক) গ্রুপ ১", "খ) গ্রুপ ২", "গ) গ্রুপ ১৭", "ঘ) গ্রুপ ১৮"], "answer": "ক) গ্রুপ ১"},
            {"q": "মিথেন (CH₄) গ্যাসে কার্বনের শতকরা পরিমাণ কত?", "options": ["ক) ৭৫%", "খ) ২৫%", "গ) ৫০%", "ঘ) ৮০%"], "answer": "ক) ৭৫%"},
            {"q": "নিচের কোনটি এসিডের জলীয় দ্রবণে উৎপন্ন হয়?", "options": ["ক) H⁺ আয়ন", "খ) OH⁻ আয়ন", "গ) Na⁺ আয়ন", "ঘ) Cl⁻ আয়ন"], "answer": "ক) H⁺ আয়ন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: মৌল A (পারমাণবিক সংখ্যা ১১) এবং মৌল B (পারমাণবিক সংখ্যা ১৭) পরস্পর যুক্ত হয়ে AB যৌগ গঠন করে।",
              "ka": "যোজনী কাকে বলে?",
              "kha": "নিষ্ক্রিয় গ্যাসগুলো রাসায়নিকভাবে নিষ্ক্রিয় কেন?",
              "ga": "A ও B মৌলের মধ্যে বন্ধন গঠন প্রক্রিয়া ইলেকট্রন বিন্যাসের সাহায্যে ব্যাখ্যা কর।",
              "gha": "\"গঠিত AB যৌগটি পানিতে দ্রবণীয় এবং জলীয় দ্রবণে বিদ্যুৎ পরিবহন করে।\"—যাচাই কর।"
            },
            {
              "stem": "উদ্দীপক ২: গবেষণাগারে ৫ গ্রাম কার্বনকে প্রয়োজনীয় অক্সিজেনে দহন করে কার্বন ডাই অক্সাইড (CO₂) তৈরি করা হলো।",
              "ka": "মোলার ভর কী?",
              "kha": "এসটিপি (STP)-তে ১ মোল গ্যাসের আয়তন কত?",
              "ga": "উদ্দীপকে উৎপন্ন CO₂ এর মোলার ভর ও অনুর সংখ্যা হিসাব কর।",
              "gha": "\"ভরের সংরক্ষণ সূত্র রাসায়নিক বিক্রিয়ার সমতা রক্ষা করে।\"—বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "পদার্থবিজ্ঞান": {
      "বিজ্ঞান শাখা": {
        "গতি, বল ও শক্তি": {
          "MCQ": [
            {"q": "পরন্ত বস্তুর সূত্র প্রদান করেন কে?", "options": ["ক) গ্যালিলিও", "খ) নিউটন", "গ) আইন্সটাইন", "ঘ) আর্কিমিডিস"], "answer": "ক) গ্যালিলিও"},
            {"q": "ত্বরণের একক কোনটি?", "options": ["ক) m/s²", "খ) m/s", "গ) N/m", "ঘ) kg·m/s"], "answer": "ক) m/s²"},
            {"q": "ভরবেগের একক কোনটি?", "options": ["ক) kg·m/s", "খ) N·m", "গ) J/s", "ঘ) m/s²"], "answer": "ক) kg·m/s"},
            {"q": "১ অশ্বক্ষমতা (HP) = কত ওয়াট?", "options": ["ক) ৭৪৬ ওয়াট", "খ) ১০০০ ওয়াট", "গ) ৫০০ ওয়াট", "ঘ) ৭০০ ওয়াট"], "answer": "ক) ৭৪৬ ওয়াট"},
            {"q": "শব্দ একটি কী ধরণের তরঙ্গ?", "options": ["ক) অনুদৈর্ঘ্য তরঙ্গ", "খ) অনুপ্রস্থ তরঙ্গ", "গ) তড়িৎচুম্বকীয় তরঙ্গ", "ঘ) স্থিতিস্থাপক তরঙ্গ নয়"], "answer": "ক) অনুদৈর্ঘ্য তরঙ্গ"},
            {"q": "অবতল দর্পণে ফোকাস দূরত্ব ব্যাসার্ধের কত গুণ?", "options": ["ক) অর্ধেক (r/2)", "খ) দ্বিগুণ", "গ) সমান", "ঘ) তিনগুণ"], "answer": "ক) অর্ধেক (r/2)"},
            {"q": "স্থির অবস্থান থেকে বিনামূল্যে পড়ন্ত বস্তুর বেগ সময়ের সাথে-", "options": ["ক) সমানুপাতিক", "খ) ব্যস্তানুপাতিক", "গ) বর্গের সমানুপাতিক", "ঘ) অপরিবর্তিত"], "answer": "ক) সমানুপাতিক"},
            {"q": "তাপমাত্রার পরিমাপক স্কেল কেলভিনে সেলসিয়াসের সম্পর্ক কোনটি?", "options": ["ক) K = C + ২৭৩.১৫", "খ) K = C - ২৭৩", "গ) K = ২৭৩ - C", "ঘ) C = K + ২৭৩"], "answer": "ক) K = C + ২৭৩.১৫"},
            {"q": "কোন বলের কারণে গ্রহগুলো সূর্যের চারদিকে ঘোরে?", "options": ["ক) মহাকর্ষ বল", "খ) নিউক্লীয় বল", "গ) তড়িৎ বল", "ঘ) ঘর্ষণ বল"], "answer": "ক) মহাকর্ষ বল"},
            {"q": "কাজ করার হারকে কী বলে?", "options": ["ক) ক্ষমতা", "খ) শক্তি", "গ) বেগ", "ঘ) ভরবেগ"], "answer": "ক) ক্ষমতা"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ৫ কেজি ভরের একটি বস্তুকে ৫০ মিটার উঁচু দালানের ছাদ থেকে নিচে ফেলে দেওয়া হলো। (g = 9.8 m/s²)",
              "ka": "মুক্তভাবে পড়ন্ত বস্তু কী?",
              "kha": "গতিশক্তি ও বিভব শক্তির মধ্যে সম্পর্ক লেখ।",
              "ga": "ভূমি স্পর্শ করার পূর্বমুহূর্তে বস্তুটির বেগ নির্ণয় কর।",
              "gha": "\"পড়ন্ত বস্তুর ক্ষেত্রে যান্ত্রিক শক্তির সংরক্ষণশীলতা নীতি সংরক্ষিত হয়।\"—গাণিতিক প্রমাণের সাহায্যে বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি গাড়ির বেগ ১০ সেকেন্ডে স্থির অবস্থান থেকে বৃদ্ধি পেয়ে ২০ m/s হলো। গাড়িটির ভর ১০০০ কেজি।",
              "ka": "ত্বরণ কাকে বলে?",
              "kha": "ঘর্ষণ বল কীভাবে গতির বিপরীত কাজ করে—ব্যাখ্যা কর।",
              "ga": "গাড়িটির ত্বরণ এবং প্রযুক্ত বলের মান হিসাব কর।",
              "gha": "\"ত্বরণ ও ভরের গুণফলই প্রযুক্ত বল নির্দেশ করে।\"—নিউটনীয় গতির আলোকে মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "জীববিজ্ঞান": {
      "বিজ্ঞান শাখা": {
        "কোষ ও অঙ্গসংস্থান": {
          "MCQ": [
            {"q": "কোষের শক্তিঘর (Powerhouse) কাকে বলে?", "options": ["ক) মাইটোকন্ড্রিয়া", "খ) গলগি বডি", "গ) নিউক্লিয়াস", "ঘ) লাইসোসোম"], "answer": "ক) মাইটোকন্ড্রিয়া"},
            {"q": "ডিএনএ (DNA) এর দ্বিসূত্রক মডেল আবিষ্কার করেন কে?", "options": ["ক) ওয়াটসন ও ক্রিক", "খ) রবার্ট হুক", "গ) মেনডেল", "ঘ) ল্যামার্ক"], "answer": "ক) ওয়াটসন ও ক্রিক"},
            {"q": "উদ্ভিদ কোষের প্রধান বৈশিষ্ট্য কোনটি?", "options": ["ক) কোষ প্রাচীরের উপস্থিতি", "খ) সেন্টিওল", "গ) ছোট কোষগহ্বর", "ঘ) লাইসোসোমের সংখ্যাধিক্য"], "answer": "ক) কোষ প্রাচীরের উপস্থিতি"},
            {"q": "মানুষের লোহিত রক্তকণিকায় কোনটি অনুপস্থিত?", "options": ["ক) নিউক্লিয়াস", "খ) হিমোগ্লোবিন", "গ) প্লাজমা", "ঘ) ঝিল্লি"], "answer": "ক) নিউক্লিয়াস"},
            {"q": "কোন বিভাজনের মাধ্যমে জননকোষ গঠিত হয়?", "options": ["ক) মিওসিস", "খ) অ্যামাইটোসিস", "গ) মাইটোসিস", "ঘ) সাইটোকাইনেসিস"], "answer": "ক) মিওসিস"},
            {"q": "সালোকসংশ্লেষণের প্রধান উপজাত দ্রব্য কোনটি?", "options": ["ক) অক্সিজেন", "খ) কার্বন ডাই অক্সাইড", "গ) নাইট্রোজেন", "ঘ) হাইড্রোজেন"], "answer": "ক) অক্সিজেন"},
            {"q": "উদ্ভিদের পানি পরিহনের জন্য দায়ী কলা কোনটি?", "options": ["ক) জাইলেম", "খ) ফ্লোয়েম", "গ) প্যারেনকাইমা", "ঘ) স্ক্লেরেনকাইমা"], "answer": "ক) জাইলেম"},
            {"q": "মানবদেহের বৃহত্তম গ্রন্থি কোনটি?", "options": ["ক) যকৃৎ (Liver)", "খ) অগ্ন্যাশয়", "গ) থাইরয়েড", "ঘ) পিটুইটারি"], "answer": "ক) যকৃৎ (Liver)"},
            {"q": "কোন ভিটামিনের অভাবে রিকেটস রোগ হয়?", "options": ["ক) ভিটামিন D", "খ) ভিটামিন A", "গ) ভিটামিন C", "ঘ) ভিটামিন K"], "answer": "ক) ভিটামিন D"},
            {"q": "অনুচক্রিকার মূল কাজ কোনটি?", "options": ["ক) রক্ত তঞ্চনে (জমাট বাঁধতে) সাহায্য করা", "খ) অক্সিজেন বহন", "গ) জীবাণু ধ্বংস", "ঘ) তাপ নিয়ন্ত্রণ"], "answer": "ক) রক্ত তঞ্চনে (জমাট বাঁধতে) সাহায্য করা"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: কোষ বিভাজনের দুটি চিত্র লক্ষ্য করা হলো: 'A' প্রক্রিয়ায় দেহের বৃদ্ধি ঘটে এবং 'B' প্রক্রিয়ায় অপত্য কোষে ক্রোমোজোম সংখ্যা অর্ধেক হয়ে যায়।",
              "ka": "অ্যামাইটোসিস বিভাজন কাকে বলে?",
              "kha": "মাইটোসিসকে সমীকরণিক বিভাজন বলা হয় কেন?",
              "ga": "উদ্দীপকের 'A' ও 'B' প্রক্রিয়ার বৈসাদৃশ্য তুলনা কর।",
              "gha": "\"জীবের ধারাবাহিকতা ও প্রজাতি রক্ষায় 'B' প্রক্রিয়ার গুরুত্ব অপরিসীম।\"—উক্তিটি মূল্যায়ন কর।"
            },
            {
              "stem": "উদ্দীপক ২: রফিক পর্যবেক্ষণ করল যে, গাছ মূলের সাহায্যে পানি শোষণ করে পাতায় পাঠায় এবং সূর্যালোকের সাহায্যে শর্করা তৈরি করে।",
              "ka": "প্রস্বেদন কী?",
              "kha": "জাইলেম ও ফ্লোয়েমের কাজ ব্যাখ্যা কর।",
              "ga": "উদ্দীপকে বর্ণিত প্রক্রিয়ায় উদ্ভিদের খাদ্য তৈরি ও পরিবহন প্রক্রিয়া আলোচনা কর।",
              "gha": "\"উদ্ভিদের প্রস্বেদন প্রক্রিয়াকে প্রয়োজনীয় ক্ষতিকর প্রক্রিয়া (Necessary evil) বলা হয়।\"—বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "উচ্চতর গণিত": {
      "বিজ্ঞান শাখা": {
        "সেট, ফাংশন ও জ্যামিতি": {
          "MCQ": [
            {"q": "f(x) = 2x + 1 হলে f(2) এর মান কত?", "options": ["ক) ৫", "খ) ৪", "গ) ৩", "ঘ) ১"], "answer": "ক) ৫"},
            {"q": "ত্রিভুজের পরিকেন্দ্র, ভরকেন্দ্র ও লম্ববিন্দু সম্পর্কিত উপপাদ্য কোনটি?", "options": ["ক) ইউলারের রেখা", "খ) পিথাগোরাসের উপপাদ্য", "গ) অ্যাপোলোনিয়াসের উপপাদ্য", "ঘ) টলেমির উপপাদ্য"], "answer": "ক) ইউলারের রেখা"},
            {"q": "aˣ = b হলে x = কত?", "options": ["ক) logₐ b", "খ) log_b a", "গ) a/b", "ঘ) b/a"], "answer": "ক) logₐ b"},
            {"q": "দ্বিপদী বিস্তৃতি (1 + x)ⁿ এর পদসংখ্যা কত?", "options": ["ক) n + 1", "খ) n", "গ) n - 1", "ঘ) 2n"], "answer": "ক) n + 1"},
            {"q": "একটি ঘনকের ধারের দৈর্ঘ্য a হলে তার আয়তন কত?", "options": ["ক) a³", "খ) 6a²", "গ) √3a", "ঘ) 3a²"], "answer": "ক) a³"},
            {"q": "sin²θ + cos²θ = কত?", "options": ["ক) ১", "খ) ০", "গ) ২", "ঘ) -১"], "answer": "ক) ১"},
            {"q": "(2, 3) ও (5, 7) বিন্দুদ্বয়ের মধ্যবর্তী দূরত্ব কত?", "options": ["ক) ৫", "খ) ২৫", "গ) ৭", "ঘ) ৩"], "answer": "ক) ৫"},
            {"q": "সমকোণী ত্রিভুজের ক্ষেত্রে অ্যাপোলোনিয়াসের উপপাদ্য কিসের বর্ধিত রূপ?", "options": ["ক) পিথাগোরাসের উপপাদ্য", "খ) ইউক্লিডের পদ্ধতি", "গ) টলেমির সূত্র", "ঘ) ভেনচিত্র"], "answer": "ক) পিথাগোরাসের উপপাদ্য"},
            {"q": "Vector A = 2i + 3j হলে এর মান (Magnitude) কত?", "options": ["ক) √13", "খ) ৫", "গ) ১৩", "ঘ) √5"], "answer": "ক) √13"},
            {"q": "tan 45° এর মান কত?", "options": ["ক) ১", "খ) ০", "গ) √৩", "ঘ) ১/√৩"], "answer": "ক) ১"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: F(x) = (2x + 2) / (x - 1) একটি ফাংশন (x ≠ 1)।",
              "ka": "ফাংশনের ডোমেন কাকে বলে?",
              "kha": "উক্ত ফাংশনের ডোমেন ও রেঞ্জ নির্ণয় কর।",
              "ga": "F⁻¹(x) অর্থাৎ বিপরীত ফাংশন নির্ণয় কর।",
              "gha": "\"এক-এক ও অনটূ ফাংশন হওয়ার ওপর বিপরীত ফাংশনের অস্তিত্ব নির্ভর করে।\"—যাচাই কর।"
            },
            {
              "stem": "উদ্দীপক ২: ΔABC এর AD একটি মধ্যমা। AB = 6 cm, AC = 8 cm এবং BC = 10 cm।",
              "ka": "অ্যাপোলোনিয়াসের উপপাদ্যের বাক্যটি লেখ।",
              "kha": "অ্যাপোলোনিয়াসের উপপাদ্য ব্যবহার করে মধ্যমা AD এর দৈর্ঘ্য নির্ণয় কর।",
              "ga": "প্রমাণ কর যে, AB² + AC² = 2(AD² + BD²)।",
              "gha": "\"জ্যামিতিক উপপাদ্য ভেক্টরের সাহায্যেও প্রমাণ করা সম্ভব।\"—উক্তিটির সত্যতা বিচার কর।"
            }
          ]
        }
      }
    },
    "হিসাববিজ্ঞান": {
      "ব্যবসায় শিক্ষা শাখা": {
        "হিসাব সমীকরণ ও জাবেদা": {
          "MCQ": [
            {"q": "হিসাব সমীকরণের মূল উপাদান কোনটি?", "options": ["ক) A = L + E", "খ) A = L - E", "গ) E = A + L", "ঘ) L = A + E"], "answer": "ক) A = L + E"},
            {"q": "হিসাববিজ্ঞানের প্রথম ও প্রাথমিক বই কোনটি?", "options": ["ক) জাবেদা (Journal)", "খ) খতিয়ান (Ledger)", "গ) রেওয়ামিল", "ঘ) নগদান বই"], "answer": "ক) জাবেদা (Journal)"},
            {"q": "খতিয়ানকে বলা হয় হিসাবের-", "options": ["ক) পাকা বই/রাজা", "খ) সহকারী বই", "গ) প্রাথমিক বই", "ঘ) সাময়িক বই"], "answer": "ক) পাকা বই/রাজা"},
            {"q": "ক্যাশ মেমো কে তৈরি করেন?", "options": ["ক) বিক্রেতা", "খ) ক্রেতা", "গ) ব্যাংক", "ঘ) ম্যানেজার"], "answer": "ক) বিক্রেতা"},
            {"q": "সম্পদ বাড়লে ডেবিট এবং কমলে কী হয়?", "options": ["ক) ক্রেডিট", "খ) ডেবিট", "গ) অপরিবর্তিত", "ঘ) সমতা"], "answer": "ক) ক্রেডিট"},
            {"q": "ব্যবসায়িক লেনদেনের লিখিত প্রমাণপত্রকে কী বলে?", "options": ["ক) ভাউচার/দলিল", "খ) খতিয়ান", "গ) রেওয়ামিল", "ঘ) চালান"], "answer": "ক) ভাউচার/দলিল"},
            {"q": "হিসাববিজ্ঞানের জনক কে?", "options": ["ক) লুকা প্যাসিওলি", "খ) অ্যাডাম স্মিথ", "গ) ফিলিপ কোটলার", "ঘ) মার্শাল"], "answer": "ক) লুকা প্যাসিওলি"},
            {"q": "উৎপাদন ব্যয় বা বেতন প্রদান কোন ধরণের হিসাব?", "options": ["ক) নামিক/ব্যয় হিসাব", "খ) সম্পত্তি হিসাব", "গ) দায় হিসাব", "ঘ) মালিকানা স্বত্ব"], "answer": "ক) নামিক/ব্যয় হিসাব"},
            {"q": "দুপাformat জমা পদ্ধতির মূল ভিত্তি কোনটি?", "options": ["ক) ডেবিট ও ক্রেডিট সমতা", "খ) একক হিসাব", "গ) নগদ গ্রহণ", "ঘ) অবচয়"], "answer": "ক) ডেবিট ও ক্রেডিট সমতা"},
            {"q": "রেওয়ামিলের মূল উদ্দেশ্য কী?", "options": ["ক) হিসাবের গাণিতিক শুদ্ধতা যাচাই", "খ) মুনাফা নির্ণয়", "গ) কর নির্ধারণ", "ঘ) দেনা পরিশোধ"], "answer": "ক) হিসাবের গাণিতিক শুদ্ধতা যাচাই"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: জনাব রফিক ২০২৪ সালের ১ জানুয়ারি তারিখে ৫০,০০০ টাকা নগদ ও ২০,০০০ টাকার আসবাবপত্র নিয়ে ব্যবসা শুরু করলেন। উক্ত মাসে নগদ লেনদেন: পণ্য ক্রয় ১৫,০০০ টাকা, ধারে পণ্য বিক্রয় ২৫,০০০ টাকা।",
              "ka": "প্রারম্ভিক মূলধনের পরিমাণ কত?",
              "kha": "জানুয়ারি মাসের লেনদেনগুলোর জাবেদা দাখিলা দেখাও।",
              "ga": "লেনদেনগুলোর প্রভাব হিসাব সমীকরণে (A = L + E) ছকে দেখাও।",
              "gha": "\"সঠিক জাবেদা ভুক্তির ওপর নির্ভুল খতিয়ান প্রস্তুত নির্ভর করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: ব্যবসায়ী সাজিদের ব্যবসায়িক খতিয়ান উদ্বৃত্ত: নগদ তহবিল ১০,০০০ টাকা, ব্যাংক জমা ২০,০০০ টাকা, দেনাদার ১৫,০০০ টাকা, পাওনাদার ১০,০০০ টাকা এবং মূলধন ৩৫,০০০ টাকা।",
              "ka": "রেওয়ামিল কী?",
              "kha": "কোন কোন ভুল রেওয়ামিলে ধরা পড়ে না—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের উদ্বৃত্তগুলো নিয়ে একটি শুদ্ধ রেওয়ামিল প্রস্তুত কর।",
              "gha": "\"রেওয়ামিল প্রস্তুতের মাধ্যমে আর্থিক বিবরণী তৈরির পথ সুগম হয়।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "ব্যবসায় পরিচিতি": {
      "ব্যবসায় শিক্ষা শাখা": {
        "ব্যবসায়ের পরিবেশ ও উৎপত্তি": {
          "MCQ": [
            {"q": "ব্যবসায়ের প্রধান ও মূল উদ্দেশ্য কোনটি?", "options": ["ক) মুনাফা অর্জন", "খ) জনকল্যাণ", "গ) পণ্য উৎপাদন", "ঘ) সেবা প্রদান"], "answer": "ক) মুনাফা অর্জন"},
            {"q": "উৎপাদনের মূল উপাদান কয়টি?", "options": ["ক) ৪টি (ভূমি, শ্রম, মূলধন, সংগঠন)", "খ) ২টি", "গ) ৩টি", "ঘ) ৫টি"], "answer": "ক) ৪টি (ভূমি, শ্রম, মূলধন, সংগঠন)"},
            {"q": "শিল্পকে মূলত প্রধানত কয় ভাগে ভাগ করা যায়?", "options": ["ক) ৫ ভাগে", "খ) ৩ ভাগে", "গ) ৪ ভাগে", "ঘ) ৬ ভাগে"], "answer": "ক) ৫ ভাগে"},
            {"q": "বাণিজ্য (Commerce) বলতে কী বোঝায়?", "options": ["ক) পণ্য বণ্টন সংক্রান্ত কাজ", "খ) পণ্য উৎপাদন", "গ) কাঁচামাল সংগ্রহ", "ঘ) যন্ত্রপাতি স্থাপন"], "answer": "ক) পণ্য বণ্টন সংক্রান্ত কাজ"},
            {"q": "একমালিকানা ব্যবসায়ের বড় সুবিধা কোনটি?", "options": ["ক) দ্রুত সিদ্ধান্ত গ্রহণ ও একক মালিকানা", "খ) প্রচুর মূলধন", "গ) স্থায়ী স্থায়িত্ব", "ঘ) যৌথ ঝুঁকি"], "answer": "ক) দ্রুত সিদ্ধান্ত গ্রহণ ও একক মালিকানা"},
            {"q": "ব্যবসায়িক পরিবেশের প্রাকৃতিক উপাদান কোনটি?", "options": ["ক) জলবায়ু ও ভূ-প্রকৃতি", "খ) আইন-কানুন", "গ) শিক্ষা", "ঘ) প্রযুক্তি"], "answer": "ক) জলবায়ু ও ভূ-প্রকৃতি"},
            {"q": "পণ্যসামগ্রী দূর-দূরান্তে পৌঁছানোর বাধা দূর করে কোনটি?", "options": ["ক) পরিবহন", "খ) গুদামজাতকরণ", "গ) ব্যাংক", "ঘ) বীমা"], "answer": "ক) পরিবহন"},
            {"q": "ঝুঁকিজনিত বাধা দূর করে ব্যবসায়ের কোন অঙ্গ?", "options": ["ক) বীমা (Insurance)", "খ) পরিবহন", "গ) বিজ্ঞাপন", "ঘ) ব্যাংক"], "answer": "ক) বীমা (Insurance)"},
            {"q": "অংশীদারি ব্যবসায়ের মূল ভিত্তি কোনটি?", "options": ["ক) চুক্তি", "খ) নিবন্ধন", "গ) মূলধন", "ঘ) অংশীদার সংখ্যা"], "answer": "ক) চুক্তি"},
            {"q": "পাবলিক লিমিটেড কোম্পানির সর্বনিম্ন সদস্য সংখ্যা কত?", "options": ["ক) ৭ জন", "খ) ২ জন", "গ) ৫০ জন", "ঘ) ২০ জন"], "answer": "ক) ৭ জন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সজল সাহেব অল্প পুঁজি নিয়ে একা একটি মুদি দোকান চালু করেন। নিজের বুদ্ধিমত্তায় দ্রুত সিদ্ধান্ত নিয়ে তিনি ভালো মুনাফা করছেন। তবে ব্যবসার সমস্ত ঝুঁকি তাকে একাই বহন করতে হয়।",
              "ka": "একমালিকানা ব্যবসা কী?",
              "kha": "একমালিকানা ব্যবসা সবচেয়ে জনপ্রিয় কেন—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সজলের ব্যবসার বৈশিষ্ট্যগুলো পাঠ্যবইয়ের আলোকে আলোচনা কর।",
              "gha": "\"সীমাবদ্ধ ঝুঁকি ও অসীম দায় থাকা সত্ত্বেও একমালিকানা ব্যবসা যুগের পর যুগ টিকে আছে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: তিন বন্ধু মিলে লিখিত চুক্তির ভিত্তিতে একটি ফলের আড়ত শুরু করলেন। ব্যবসায় অর্জিত লাভ-ক্ষতি তারা চুক্তির শর্তানুযায়ী ভাগ করে নেন।",
              "ka": "অংশীদারি কারবার কী?",
              "kha": "\"চুক্তিই অংশীদারি কারবারের মূল ভিত্তি\"—বুঝিয়ে লেখ।",
              "ga": "উদ্দীপকে বর্ণিত ব্যবসার ধরণ ও পরিচালনার নিয়ম ব্যাখ্যা কর।",
              "gha": "\"নিবন্ধন না থাকলেও অংশীদারি ব্যবসা বৈধ, তবে নিবন্ধিত হওয়া অধিক নিরাপদ।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "ব্যবসায় উদ্যোগ": {
      "ব্যবসায় শিক্ষা শাখা": {
        "উদ্যোগ ও ব্যবসায়িক পরিকল্পনা": {
          "MCQ": [
            {"q": "যেকোনো কাজের ঝুঁকি নিয়ে পথ চলাই হলো-", "options": ["ক) উদ্যোগ", "খ) চাকরি", "গ) অলসতা", "ঘ) চুক্তি"], "answer": "ক) উদ্যোগ"},
            {"q": "ব্যবসায় উদ্যোগের প্রধান গুণ কোনটি?", "options": ["ক) উদ্ভাবনী শক্তি ও ঝুঁকি নেওয়ার সাহস", "খ) অধিক মূলধন থাকা", "গ) কায়িক শ্রম", "ঘ) উচ্চ শিক্ষা"], "answer": "ক) ব্যবসায় উদ্যোগের প্রধান গুণ কোনটি?"},
            {"q": "SWOT বিশ্লেষণের 'O' অক্ষরটি কী নির্দেশ করে?", "options": ["ক) Opportunity (সুযোগ)", "খ) Organization", "গ) Operation", "ঘ) Ownership"], "answer": "ক) Opportunity (সুযোগ)"},
            {"q": "ক্ষুদ্র ও কুটির শিল্পের বড় সুবিধা কী?", "options": ["ক) কম মূলধন ও স্থানীয় কর্মসংস্থান", "খ) ভারী যন্ত্রপাতি ব্যবহার", "গ) বিদেশি বিনিয়োগ", "ঘ) উচ্চ কর প্রদান"], "answer": "ক) কম মূলধন ও স্থানীয় কর্মসংস্থান"},
            {"q": "ব্যবসায়িক পরিকল্পনার প্রথম ধাপ কোনটি?", "options": ["ক) প্রকল্প নির্বাচন/ধারণা চিহ্নিতকরণ", "খ) ঋণ গ্রহণ", "গ) বিজ্ঞাপন প্রদান", "ঘ) কর্মী নিয়োগ"], "answer": "ক) প্রকল্প নির্বাচন/ধারণা চিহ্নিতকরণ"},
            {"q": "ব্যবসায়িক ঝুঁকি হ্রাসের উপায় কোনটি?", "options": ["ক) বীমা করা ও সঠিক পরিকল্পনা", "খ) বেশি দামে বিক্রি", "গ) বিজ্ঞাপন বন্ধ করা", "ঘ) ঋণ না নেওয়া"], "answer": "ক) বীমা করা ও সঠিক পরিকল্পনা"},
            {"q": "সফল উদ্যোক্তার অন্যতম মানদণ্ড কোনটি?", "options": ["ক) অধ্যবসায় ও সততা", "খ) দ্রুত কাজ বন্ধ করা", "গ) অন্যকে অনুকরণ করা", "ঘ) ঝুঁকি না নেওয়া"], "answer": "ক) অধ্যবসায় ও সততা"},
            {"q": "বিপণনের (Marketing) প্রথম কাজ কোনটি?", "options": ["ক) ক্রয়", "খ) বিক্রয়", "গ) গুদামজাতকরণ", "ঘ) পরিবহন"], "answer": "ক) ক্রয়"},
            {"q": "পণ্য ঢাকাকে কী বলা হয়?", "options": ["ক) মোড়কীকরণ (Packaging)", "খ) প্রমিতকরণ", "গ) পর্যায়িতকরণ", "ঘ) পরিবহন"], "answer": "ক) মোড়কীকরণ (Packaging)"},
            {"q": "বাংলাদেশ ক্ষুদ্র ও কুটির শিল্প সংস্থা কোনটি?", "options": ["ক) BSCIC", "খ) EPB", "গ) BGMEA", "ঘ) FBCCI"], "answer": "ক) BSCIC"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: তরুণ উদ্যোক্তা তৌহিদ তার এলাকায় প্লাস্টিক বর্জ্য প্রক্রিয়াজাত করে নতুন পণ্য তৈরির কারখানা গড়ে তোলেন। নতুন এই চিন্তায় প্রথম দিকে ঝুঁকি থাকলেও তিনি এখন একজন সফল ব্যবসায়ী।",
              "ka": "ব্যবসায় উদ্যোক্তা কে?",
              "kha": "ঝুঁকি গ্রহণ ও ব্যবসায়ের সাফল্যের সম্পর্ক ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের তৌহিদের উদ্যোগটি কীভাবে স্থানীয় অর্থনীতিতে প্রভাব ফেলছে—ব্যাখ্যা কর।",
              "gha": "\"উদ্ভাবনী ক্ষমতাই একজন সফল উদ্যোক্তাকে সাধারণ ব্যবসায়ী থেকে আলাদা করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: রিনা আক্তার একটি হস্তশিল্প প্রতিষ্ঠান চালুর পূর্বে বাজারের চাহিদা, মূলধনের উৎস ও কাঁচামালের সহজলভ্যতা যাচাই করে একটি নীল নকশা তৈরি করলেন।",
              "ka": "ব্যবসায়িক পরিকল্পনা কী?",
              "kha": "SWOT বিশ্লেষণ ব্যবসায়ের জন্য কেন গুরুত্বপূর্ণ?",
              "ga": "উদ্দীপকে রিনা আক্তারের গৃহীত পদক্ষেপটি কেন প্রয়োজনীয় তা আলোচনা কর।",
              "gha": "\"পূর্ব পরিকল্পনা ছাড়া ব্যবসায় সফল হওয়া অসম্ভব বললেই চলে।\"—উক্তিটির যথার্থতা মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "বাংলাদেশের ইতিহাস ও বিশ্বসভ্যতা": {
      "মানবিক শাখা": {
        "প্রাচীন সভ্যতা ও প্রাচীন বাংলা": {
          "MCQ": [
            {"q": "ইতিহাসের জনক কাকে বলা হয়?", "options": ["ক) হেরোডোটাস", "খ) থুসিডাইডিস", "গ) এরিস্টটল", "ঘ) ইবনে খালদুন"], "answer": "ক) হেরোডোটাস"},
            {"q": "সিন্ধু সভ্যতা কোন যুগের সভ্যতা?", "options": ["ক) তাম্র-ব্রোঞ্জ যুগ", "খ) লৌহ যুগ", "গ) প্রস্তর যুগ", "ঘ) আধুনিক যুগ"], "answer": "ক) তাম্র-ব্রোঞ্জ যুগ"},
            {"q": "প্রাচীন বাংলার প্রথম স্বাধীন নরপতি কে ছিলেন?", "options": ["ক) শশাঙ্ক", "খ) ধর্মপাল", "গ) গোপাল", "ঘ) লক্ষণ সেন"], "answer": "ক) শশাঙ্ক"},
            {"q": "পাল বংশের প্রতিষ্ঠাতা কে ছিলেন?", "options": ["ক) গোপাল", "খ) ধর্মপাল", "গ) দেবপাল", "ঘ) মহিপাল"], "answer": "ক) গোপাল"},
            {"q": "মৎস্যন্যায় বলতে কোন সময়কে বোঝায়?", "options": ["ক) শশাঙ্কের মৃত্যুর পর ১০০ বছরের অরাজকতা", "খ) পাল শাসনের স্বর্ণযুগ", "গ) সেন বংশের সময়", "ঘ) সুলতানি আমল"], "answer": "ক) শশাঙ্কের মৃত্যুর পর ১০০ বছরের অরাজকতা"},
            {"q": "মিশরীয় সভ্যতার সবচেয়ে বড় অবদান কোনটি?", "options": ["ক) পিরামিড ও হায়ারোগ্লিফিক লিপি", "খ) চাকা আবিষ্কার", "গ) পাত্র তৈরি", "ঘ) ভাস্কর্য"], "answer": "ক) পিরামিড ও হায়ারোগ্লিফিক লিপি"},
            {"q": "প্রাচীন জনপদ 'গৌড়' এর অন্তর্ভুক্ত অঞ্চল কোনটি ছিল?", "options": ["ক) চাঁপাইনবাবগঞ্জ ও মুর্শিদাবাদ", "খ) চট্টগ্রাম", "গ) বরিশাল", "ঘ) সিলেট"], "answer": "ক) চাঁপাইনবাবগঞ্জ ও মুর্শিদাবাদ"},
            {"q": "সেন বংশের শেষ রাজা কে ছিলেন?", "options": ["ক) লক্ষণ সেন", "খ) বিজয় সেন", "গ) বল্লাল সেন", "ঘ) সামন্ত সেন"], "answer": "ক) লক্ষণ সেন"},
            {"q": "কত সালে ইখতিয়ার উদ্দিন মুহম্মদ বিন বখতিয়ার খলজি বাংলা জয় করেন?", "options": ["ক) ১২০৪ সালে", "খ) ১২০৬ সালে", "গ) ১১৯৯ সালে", "ঘ) ১২৫৮ সালে"], "answer": "ক) ১২০৪ সালে"},
            {"q": "পাহাড়পুরের সোমপুর মহাবিহার কে নির্মাণ করেন?", "options": ["ক) ধর্মপাল", "খ) দেবপাল", "গ) গোপাল", "ঘ) শশাঙ্ক"], "answer": "ক) ধর্মপাল"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: শশাঙ্কের মৃত্যুর পর বাংলাজুড়ে তীব্র অরাজকতা ও বিশৃঙ্খলা দেখা দেয়, যেখানে শক্তিশালী ব্যক্তি দুর্বলদের ওপর অত্যাচার করত। দীর্ঘ ১০০ বছর পর জনগণ মিলে গোপালকে রাজা নির্বাচিত করে শান্তি ফিরিয়ে আনে।",
              "ka": "মৎস্যন্যায় কী?",
              "kha": "পাল রাজত্বকে বাংলার স্বর্ণযুগ বলা হয় কেন?",
              "ga": "উদ্দীপকে বর্ণিত ঐতিহাসিক প্রেক্ষাপট ও পাল বংশের শাসনের সূচনা আলোচনা কর।",
              "gha": "\"জনগণের সর্বসম্মত সিদ্ধান্তই বাংলায় প্রথম স্থায়ী ও দীর্ঘস্থায়ী বংশের ভিত্তি গড়ে দিয়েছিল।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: সিন্ধু নদের অববাহিকায় গড়ে ওঠা প্রাচীন নগর পরিকল্পনা ও নর্দমা ব্যবস্থা আধুনিক সভ্যতার প্রকৌশলীদের বিস্মিত করে।",
              "ka": "হায়ারোগ্লিফিক কী?",
              "kha": "সিন্ধু সভ্যতার নগর পরিকল্পনার দুটি বৈশিষ্ট্য লেখ।",
              "ga": "উদ্দীপকের আলোকে প্রাচীন সভ্যতার অবদান ও আধুনিক জীবনে তার প্রভাব ব্যাখ্যা কর।",
              "gha": "\"ইতিহাস পাঠ আমাদের অতীত ঐতিহ্যকে চিনে ভবিষ্যতের সিদ্ধান্ত নিতে সাহায্য করে।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "পৌরনীতি ও নাগরিকতা": {
      "মানবিক শাখা": {
        "রাষ্ট্র, সরকার ও নাগরিকতা": {
          "MCQ": [
            {"q": "পৌরনীতির ইংরেজি প্রতিশব্দ 'Civics' এসেছে কোন ভাষা থেকে?", "options": ["ক) ল্যাটিন ভাষা (Civis ও Civitas)", "খ) গ্রিক ভাষা", "গ) ফরাসি ভাষা", "ঘ) ইংরেজি"], "answer": "ক) ল্যাটিন ভাষা (Civis ও Civitas)"},
            {"q": "পরিবারের প্রধানতম সামাজিক কাজ কোনটি?", "options": ["ক) সন্তান লালন-পালন ও বংশ রক্ষা", "খ) বিনোদন", "গ) অর্থনৈতিক লেনদেন", "ঘ) আইন প্রণয়ন"], "answer": "ক) সন্তান লালন-পালন ও বংশ রক্ষা"},
            {"q": "নাগরিকতা অর্জনের প্রধান পদ্ধতি কয়টি?", "options": ["ক) ২টি (জন্মসূত্র ও অনুমোদন সূত্র)", "খ) ৩টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ২টি (জন্মসূত্র ও অনুমোদন সূত্র)"},
            {"q": "রাষ্ট্রের অপরিহার্য উপাদান কোনটি?", "options": ["ক) সার্বভৌমত্ব", "খ) লিখিত সংবিধান", "গ) রাজনৈতিক দল", "ঘ) বিচার বিভাগ"], "answer": "ক) সার্বভৌমত্ব"},
            {"q": "গণতন্ত্রের মূল ভিত্তি কোনটি?", "options": ["ক) জনগণের মতামত ও ভোটাধিকার", "খ) সামরিক শাসন", "গ) একনায়কত্ব", "ঘ) রাজতন্ত্র"], "answer": "ক) জনগণের মতামত ও ভোটাধিকার"},
            {"q": "আইনের প্রধান উৎস কোনটি?", "options": ["ক) প্রথা, ধর্ম ও আইনসভা", "খ) ব্যক্তির আদেশ", "গ) শক্তি প্রয়োগ", "ঘ) চুক্তি"], "answer": "ক) প্রথা, ধর্ম ও আইনসভা"},
            {"q": "মানবাধিকার সনদের সর্বজনীন ঘোষণা জাতিসংঘ কবে দেয়?", "options": ["ক) ১০ ডিসেম্বর ১৯৪৮", "খ) ২৪ অক্টোবর ১৯৪৫", "গ) ২৬ মার্চ ১৯৭১", "ঘ) ১ মে ১৮৮৬"], "answer": "ক) ১০ ডিসেম্বর ১৯৪৮"},
            {"q": "সরকারের অঙ্গ কয়টি?", "options": ["ক) ৩টি (আইন, শাসন ও বিচার বিভাগ)", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি (আইন, শাসন ও বিচার বিভাগ)"},
            {"q": "সুনাগরিকের প্রধান গুণ কয়টি?", "options": ["ক) ৩টি (বুদ্ধি, বিবেক, আত্মসংযম)", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি (বুদ্ধি, বিবেক, আত্মসংযম)"},
            {"q": "দ্বৈত নাগরিকতা বলতে কী বোঝায়?", "options": ["ক) একই সাথে দুই দেশের নাগরিক হওয়া", "খ) দুই রাজ্যের অধিবাসী হওয়া", "গ) জন্মসূত্রে নাগরিক হওয়া", "ঘ) অনুমোদন সূত্রে নাগরিক হওয়া"], "answer": "ক) একই সাথে দুই দেশের নাগরিক হওয়া"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সায়ান একজন বিবেকবান নাগরিক। সে কর প্রদান করে, সৎ প্রার্থীকে ভোট দেয় এবং রাষ্ট্রের সম্পদ রক্ষা করে। সে মনে করে নাগরিকের অধিকার ও কর্তব্য একে অপরের পরিপূরক।",
              "ka": "সুনাগরিক কাকে বলে?",
              "kha": "অধিকার ও কর্তব্যের পারস্পরিক সম্পর্ক ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সায়ানের আচরণের মাধ্যমে সুনাগরিকের কোন গুণাবলি প্রকাশ পেয়েছে—ব্যাখ্যা কর।",
              "gha": "\"সুনাগরিকই একটি দেশের টেকসই উন্নয়ন ও সুশাসনের মূল চালিকাশক্তি।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: ক দেশে আইনসভার মাধ্যমে গঠিত সরকার জনগণের নিকট দায়ী থাকে। দেশের সার্বভৌম ক্ষমতার মালিক জনগণ এবং সেখানে নিয়মিত নির্বাচন অনুষ্ঠিত হয়।",
              "ka": "সরকার কী?",
              "kha": "একনায়কতন্ত্র ও গণতন্ত্রের প্রধান পার্থক্য লেখ।",
              "ga": "উদ্দীপকের 'ক' দেশে কোন ধরণের সরকার ব্যবস্থা বিদ্যমান পাঠ্যবইয়ের আলোকে আলোচনা কর।",
              "gha": "\"জনগণের অংশগ্রহণ ও সুশাসন নিশ্চিত করাই আধুনিক গণতান্ত্রিক রাষ্ট্রের প্রধান লক্ষ্য।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "অর্থনীতি": {
      "মানবিক শাখা": {
        "মৌলিক অর্থনৈতিক ধারণা ও বাজার": {
          "MCQ": [
            {"q": "অর্থনীতির জনক কাকে বলা হয়?", "options": ["ক) অ্যাডাম স্মিথ", "খ) লর্ড রবিন্স", "গ) আলফ্রেড মার্শাল", "ঘ) কেয়ন্স"], "answer": "ক) অ্যাডাম স্মিথ"},
            {"q": "অর্থনৈতিক প্রধান সমস্যা দুটি কী কী?", "options": ["ক) দুষ্প্রাপ্যতা ও অসীম অভাব", "খ) মূলধন ও জমি", "গ) আয় ও ব্যয়", "ঘ) আমদানি ও রপ্তানি"], "answer": "ক) দুষ্প্রাপ্যতা ও অসীম অভাব"},
            {"q": "চাহিদার বিধি অনুযায়ী দাম বাড়লে চাহিদার পরিমাণ-", "options": ["ক) কমে", "খ) বাড়ে", "গ) স্থির থাকে", "ঘ) শূন্য হয়"], "answer": "ক) কমে"},
            {"q": "সম্পূর্ণ প্রতিযোগিতামূলক বাজারে পণ্যের দাম কীভাবে নির্ধারিত হয়?", "options": ["ক) চাহিদা ও যোগানের পারস্পরিক প্রভাবে", "খ) বিক্রেতার ইচ্ছায়", "গ) ক্রেতার ইচ্ছায়", "ঘ) সরকারের নির্দেশে"], "answer": "ক) চাহিদা ও যোগানের পারস্পরিক প্রভাবে"},
            {"q": "উৎপাদনের উপাদানের খাজনা কার বিনিময় মূল্য?", "options": ["ক) জমি", "খ) শ্রম", "গ) মূলধন", "ঘ) সংগঠন"], "answer": "ক) জমি"},
            {"q": "জাতীয় আয় পরিমাপের পদ্ধতি কয়টি?", "options": ["ক) ৩টি", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি"},
            {"q": "যে বাজারে একজনমাত্র বিক্রেতা থাকে তাকে কী বলে?", "options": ["ক) একচেটিয়া বাজার (Monopoly)", "খ) প্রতিযোগিতা বাজার", "গ) পাইকারি বাজার", "ঘ) শেয়ার বাজার"], "answer": "ক) একচেটিয়া বাজার (Monopoly)"},
            {"q": "অর্থনীতির প্রধান অর্থনৈতিক ব্যবস্থা কয়টি?", "options": ["ক) ৪টি (ধনতান্ত্রিক, সমাজতান্ত্রিক, মিশ্র, ইসলামী)", "খ) ৩টি", "গ) ২টি", "ঘ) ৫টি"], "answer": "ক) ৪টি (ধনতান্ত্রিক, সমাজতান্ত্রিক, মিশ্র, ইসলামী)"},
            {"q": "যোগান বিধি অনুযায়ী দাম ও যোগানের সম্পর্ক কেমন?", "options": ["ক) সমমুখী/প্রত্যক্ষ", "খ) বিপরীতমুখী", "গ) কোনো সম্পর্ক নেই", "ঘ) স্থির"], "answer": "ক) সমমুখী/প্রত্যক্ষ"},
            {"q": "জিডিপি (GDP) এর পূর্ণরূপ কী?", "options": ["ক) Gross Domestic Product", "খ) Gross Development Product", "গ) General Domestic Price", "ঘ) Global Domestic Product"], "answer": "ক) Gross Domestic Product"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: বাজারে আলুর দাম প্রতি কেজি ২০ টাকা থেকে বেড়ে ৩০ টাকা হলে চাহিদার পরিমাণ ৫০ কেজি থেকে কমে ৩০ কেজি হয়। আবার দাম কমলে ক্রেতারা বেশি আলু কেনেন।",
              "ka": "চাহিদা কাকে বলে?",
              "kha": "দুষ্প্রাপ্যতা বলতে কী বোঝায়?",
              "ga": "উদ্দীপকের তথ্য থেকে একটি চাহিদা রেখা অঙ্কন কর।",
              "gha": "\"দাম ও চাহিদার বিপরীতমুখী সম্পর্কই চাহিদা বিধির ভিত্তি।\"—উদ্দীপকের তথ্যের সাহায্যে মূল্যায়ন কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি দেশে সরকারি ও বেসরকারি উভয় খাতের যৌথ অংশগ্রহণে শিল্পকারখানা পরিচালিত হয় এবং মুক্ত বাজার ব্যবস্থার সাথে সামাজিক কল্যাণ রক্ষা করা হয়।",
              "ka": "মিশ্র অর্থব্যবস্থা কী?",
              "kha": "সমাজতান্ত্রিক অর্থব্যবস্থার দুটি মূল বৈশিষ্ট্য লেখ।",
              "ga": "উদ্দীপকের দেশটি কোন ধরণের অর্থনৈতিক ব্যবস্থা অনুসরণ করছে তা ব্যাখ্যা কর।",
              "gha": "\"জনকল্যাণ ও অর্থনৈতিক প্রবৃদ্ধির সমতা রক্ষায় মিশ্র অর্থব্যবস্থা অত্যন্ত কার্যকর।\"—বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "ইসলাম ও নৈতিক শিক্ষা": {
      "ধর্মীয় বিষয়": {
        "ঈমান, শরিয়ত ও নৈতিকতা": {
          "MCQ": [
            {"q": "ইসলাম শব্দের অর্থ কী?", "options": ["ক) শান্তি ও আত্মসমর্পণ", "খ) ইবাদত", "গ) বিশ্বাস", "ঘ) অনুসরণ"], "answer": "ক) শান্তি ও আত্মসমর্পণ"},
            {"q": "শরিয়তের প্রধান উৎস কোনটি?", "options": ["ক) কুরআন মজিদ", "খ) হাদিস", "গ) ইজমা", "ঘ) কিয়াস"], "answer": "ক) কুরআন মজিদ"},
            {"q": "কুরআন মজিদের আয়াত সংখ্যা কত?", "options": ["ক) ৬৬৬৬টি (প্রচলিত মত)", "খ) ৬২৩৬টি", "গ) ৭০০০টি", "ঘ) ৫০০০টি"], "answer": "ক) ৬৬৬৬টি (প্রচলিত মত)"},
            {"q": "সাদাকাহ শব্দের অর্থ কী?", "options": ["ক) দান/দয়া", "খ) নামাজ", "গ) রোজা", "ঘ) ফরজ"], "answer": "ক) দান/দয়া"},
            {"q": "শরিয়তের দ্বিতীয় উৎস কোনটি?", "options": ["ক) সুন্নাহ/হাদিস", "খ) ইজমা", "গ) কিয়াস", "ঘ) ফিকহ"], "answer": "ক) সুন্নাহ/হাদিস"},
            {"q": "আখলাক-এ হামিদা কী?", "options": ["ক) প্রশংসনীয় চরিত্র", "খ) নিন্দনীয় চরিত্র", "গ) অহংকার", "ঘ) প্রতারণা"], "answer": "ক) প্রশংসনীয় চরিত্র"},
            {"q": "জাহিলিয়াত শব্দের অর্থ কী?", "options": ["ক) অজ্ঞতা ও বর্বরতা", "খ) জ্ঞান", "গ) সভ্যতা", "ঘ) শান্তি"], "answer": "ক) অজ্ঞতা ও বর্বরতা"},
            {"q": "হুদায়বিয়ার সন্ধি কত হিজরিতে স্বাক্ষরিত হয়?", "options": ["ক) ৬ হিজরিতে", "খ) ৮ হিজরিতে", "গ) ৫ হিজরিতে", "ঘ) ১০ হিজরিতে"], "answer": "ক) ৬ হিজরিতে"},
            {"q": "বিদায় হজের ভাষণ মহানবী (সা.) কোথায় প্রদান করেন?", "options": ["ক) আরাফাতের ময়দানে", "খ) মিনার ময়দানে", "গ) কাবা প্রাঙ্গণে", "ঘ) মদিনায়"], "answer": "ক) আরাফাতের ময়দানে"},
            {"q": "সর্বশ্রেষ্ঠ ও সর্বশেষ আসমানি কিতাব কোনটি?", "options": ["ক) আল-কুরআন", "খ) তাওরাত", "গ) ইঞ্জিল", "ঘ) যবুর"], "answer": "ক) আল-কুরআন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: শাহেদ সামাজিক কাজে নিয়োজিত থাকে। সে প্রতিবেশীর বিপদে সাহায্য করে এবং মিথ্যা বলা থেকে বিরত থাকে। সে বিশ্বাস করে প্রতিটি কাজের হিসাব হাশরের মাঠে দিতে হবে।",
              "ka": "আখিরাত শব্দের অর্থ কী?",
              "kha": "শরিয়তের বিধিবিধান পালনে ঈমানের ভূমিকা ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের শাহেদের আচরণের মধ্যে ভালো চরিত্রের কোন দিকটি ফুটে উঠেছে—ব্যাখ্যা কর।",
              "gha": "\"পরকালে বিশ্বাসই মানুষকে ইহকালে সত্য ও নৈতিকতার পথে পরিচালিত করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: মহানবী (সা.) শত্রুর সাথে হুদায়বিয়ার অসম চুক্তিতে আবদ্ধ হয়ে শান্তিকামী মানসিকতার পরিচয় দিয়েছিলেন, যা পরবর্তীতে মক্কা বিজয়ের পথ সহজ করেছিল।",
              "ka": "হুদায়বিয়ার সন্ধি কত বছরের জন্য হয়েছিল?",
              "kha": "রাসূল (সা.) এর দূরদর্শিতা কীভাবে শান্তি এনেছিল—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের ঐতিহাসিক ঘটনাটির গুরুত্ব শরিয়ত ও জীবনের আলোকে বর্ণনা কর।",
              "gha": "\"মহানবী (সা.) এর শান্তি ও ক্ষমার আদর্শই আধুনিক বিশ্বের সংঘাত নিরসনের উপায়।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "তথ্য ও যোগাযোগ প্রযুক্তি": {
      "আবশ্যিক বিষয়": {
        "ডিজিটাল বাংলাদেশ ও সিকিউরিটি": {
          "MCQ": [
            {"q": "তথ্যপ্রযুক্তির প্রসারে 'ডিজিটাল বাংলাদেশ' রূপকল্প কত সালে ঘোষিত হয়?", "options": ["ক) ২০০৮ সালে", "খ) ২০১০ সালে", "গ) ২০১৫ সালে", "ঘ) ২০২০ সালে"], "answer": "ক) ২০০৮ সালে"},
            {"q": "ই-পাসপোর্ট ও ই-গভর্ন্যান্সের প্রধান লক্ষ্য কী?", "options": ["ক) সেবা সহজ ও দ্রুত করা", "খ) খরচ বাড়ানো", "গ) কাগজের ব্যবহার বাড়ানো", "ঘ) সময় নষ্ট করা"], "answer": "ক) সেবা সহজ ও দ্রুত করা"},
            {"q": "কম্পিউটার ভাইরাস হলো এক ধরণের-", "options": ["ক) ক্ষতিকারক প্রোগ্রাম", "খ) হার্ডওয়্যার ডিভাইস", "গ) নেটওয়ার্ক ক্যাবল", "ঘ) মেমোরি চিপ"], "answer": "ক) ক্ষতিকারক প্রোগ্রাম"},
            {"q": "দ্বিমাত্রিক বারকোডকে সংক্ষেপে কী বলে?", "options": ["ক) QR Code", "খ) PIN", "গ) URL", "ঘ) IP"], "answer": "ক) QR Code"},
            {"q": "কম্পিউটার সুরক্ষার জন্য কোনটি ব্যবহার করা উচিত?", "options": ["ক) অ্যান্টিভাইরাস সফটওয়্যার", "খ) কেবল গেম", "গ) নতুন মনিটর", "ঘ) উচ্চগতির প্রসেসর"], "answer": "ক) অ্যান্টিভাইরাস সফটওয়্যার"},
            {"q": "সামাজিক যোগাযোগ মাধ্যম কোনটি?", "options": ["ক) Facebook", "খ) Google Search", "গ) MS Word", "ঘ) Photoshop"], "answer": "ক) Facebook"},
            {"q": "ইন্টারনেটে নিরাপদে থাকার প্রধান নিয়ম কোনটি?", "options": ["ক) শক্তিশালী পাসওয়ার্ড ব্যবহার ও তথ্য গোপন রাখা", "খ) সবাইকে পাসওয়ার্ড দেওয়া", "গ) অচেনা লিংকে ক্লিক করা", "ঘ) অ্যান্টিভাইরাস বন্ধ রাখা"], "answer": "ক) শক্তিশালী পাসওয়ার্ড ব্যবহার ও তথ্য গোপন রাখা"},
            {"q": "ডিজিটাল কনটেন্ট কত প্রকার হতে পারে?", "options": ["ক) ৪ প্রকার (টেক্সট, ছবি, অডিও, ভিডিও)", "খ) ২ প্রকার", "গ) ৩ প্রকার", "ঘ) ৫ প্রকার"], "answer": "ক) ৪ প্রকার (টেক্সট, ছবি, অডিও, ভিডিও)"},
            {"q": "ই-বই বা ইলেকট্রনিক বুক পড়ার জনপ্রিয় ডিভাইস কোনটি?", "options": ["ক) ই-বুক রিডার (Kindle)", "খ) প্রিন্টার", "গ) প্রজেক্টর", "ঘ) স্ক্যানার"], "answer": "ক) ই-বুক রিডার (Kindle)"},
            {"q": "অন্যের কম্পিউটার বা সিস্টেমে অবৈধ প্রবেশকে কী বলে?", "options": ["ক) হ্যাকিং (Hacking)", "খ) ব্রাউজিং", "গ) আপলোডিং", "ঘ) প্রোগ্রামিং"], "answer": "ক) হ্যাকিং (Hacking)"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: তানভীর অনলাইন ব্যাংকিং ব্যবহার করে ঘরে বসেই স্কুলের ফি প্রদান করল। তবে একদিন সে একটি ভুয়া ই-মেইল লিংকে ক্লিক করে তার অ্যাকাউন্টের পাসওয়ার্ড হারিয়ে বিপদে পড়ল।",
              "ka": "সাইবার অপরাধ কী?",
              "kha": "পাসওয়ার্ড শক্তিশালী করার দুটি উপায় লেখ।",
              "ga": "উদ্দীপকে তানভীরের অসাবধানতার কারণ আইসিটি নিরাপত্তার আলোকে ব্যাখ্যা কর।",
              "gha": "\"সচেতনতা ও সতর্কতা অবলম্বনই সাইবার জগতে নিরাপদ থাকার প্রধান চাবিকাঠি।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি সরকারি দপ্তরে ফাইলপত্র আগে ম্যানুয়ালি সংরক্ষণ করা হতো। বর্তমানে ই-সেবা ও ই-ফাইলিং চালুর ফলে নাগরিকেরা দ্রুত সেবা পাচ্ছেন।",
              "ka": "ই-গভর্ন্যান্স কী?",
              "kha": "ই-সেবার মাধ্যমে নাগরিক সুবিধা বৃদ্ধির দুটি উদাহরণ দাও।",
              "ga": "উদ্দীপকের সেবা ব্যবস্থার পরিবর্তনের ফলে কীভাবে সময় ও অর্থ সাশ্রয় হচ্ছে তা আলোচনা কর।",
              "gha": "\"ডিজিটাল প্রযুক্তি প্রবর্তন একটি স্বচ্ছ ও দুর্নীতিমুক্ত সমাজ গঠনে সহায়ক।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    }
  },
  
  "দশম শ্রেণি": {
    "বাংলা ১ম পত্র": {
      "সাহিত্য": {
        "গদ্য ও পদ্য": {
          "MCQ": [
            {"q": "'শুভা' গল্পে শুভার প্রকৃত নাম কী ছিল?", "options": ["ক) সুভাষিণী", "খ) সুকেশিনী", "গ) সুহাসিনী", "ঘ) সুচরিতা"], "answer": "ক) সুভাষিণী"},
            {"q": "বই পড়া অনুশীলনের জন্য কিসের প্রয়োজন সবচেয়ে বেশি?", "options": ["ক) লাইব্রেরি", "খ) বিদ্যালয়", "গ) বিশ্ববিদ্যালয়", "ঘ) গৃহ"], "answer": "ক) লাইব্রেরি"},
            {"q": "'বঙ্গবাণী' কবিতায় কবি কাদের প্রতি ক্ষোভ প্রকাশ করেছেন?", "options": ["ক) যারা বাংলাদেশে জন্মে বাংলা ভাষাকে অবজ্ঞা করে", "খ) যারা বিদেশি ভাষা জানে না", "গ) যারা পড়াশোনা করে না", "ঘ) যারা কাব্যচর্চা করে না"], "answer": "ক) যারা বাংলাদেশে জন্মে বাংলা ভাষাকে অবজ্ঞা করে"},
            {"q": "কপোতাক্ষ নদ কবিতার মূল সুর কোনটি?", "options": ["ক) স্বদেশপ্রেম", "খ) প্রকৃতিপ্রীতি", "গ) রোমান্টিকতা", "ঘ) স্মৃতিচারণ"], "answer": "ক) স্বদেশপ্রেম"},
            {"q": "'মানুষ' কবিতায় সাম্যবাদের বার্তা কে প্রচার করেছেন?", "options": ["ক) কাজী নজরুল ইসলাম", "খ) রবীন্দ্রনাথ ঠাকুর", "গ) জসীমউদ্দীন", "ঘ) জীবনানন্দ দাশ"], "answer": "ক) কাজী নজরুল ইসলাম"},
            {"q": "মমতাদি কত টাকা মাইনে আশা করেছিল?", "options": ["ক) ১২ টাকা", "খ) ১৫ টাকা", "গ) ২০ টাকা", "ঘ) ২৫ টাকা"], "answer": "ক) ১২ টাকা"},
            {"q": "আম আঁটির ভেঁপু গল্পের মূল চরিত্র কোনটি?", "options": ["ক) অপু ও দুর্গা", "খ) হরিহর", "গ) সর্বজয়া", "ঘ) রায়পিসি"], "answer": "ক) অপু ও দুর্গা"},
            {"q": "'পল্লীসাহিত্য' প্রবন্ধে কোন বিষয়টি সংরক্ষণের ওপর জোর দেওয়া হয়েছে?", "options": ["ক) লোকসাহিত্য ও লোকসংস্কৃতি", "খ) শহুরে সাহিত্য", "গ) উপন্যাস", "ঘ) নাট্যচর্চা"], "answer": "ক) লোকসাহিত্য ও লোকসংস্কৃতি"},
            {"q": "'জীবন-সংগীত' কবিতার মূল বার্তা কী?", "options": ["ক) কর্মময় জীবনের গুরুত্ব", "খ) অলস জীবনযাপন", "গ) বৈরাগ্য সাধন", "ঘ) সংসারের মায়া ত্যাগ"], "answer": "ক) কর্মময় জীবনের গুরুত্ব"},
            {"q": "'একাত্তরের দিনগুলি' কোন জাতীয় রচনা?", "options": ["ক) দিনলিপি/স্মৃতিচারণ", "খ) উপন্যাস", "গ) নাটক", "ঘ) ভ্রমণকাহিনী"], "answer": "ক) দিনলিপি/স্মৃতিচারণ"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সলিমুল্লাহ গ্রামে এক প্রতিবন্ধী শিশুকে গভীর মমতায় আগলে রাখেন। সমাজের মানুষ তাকে অবহেলা করলেও তিনি মনে করেন ভালোবাসাই সকল মানুষের অধিকার।",
              "ka": "শুভার বাকপ্রতিবন্ধী দুই সখীর নাম কী?",
              "kha": "শুভা প্রকৃতির মাঝে আশ্রয় খুঁজে পেয়েছিল কেন?",
              "ga": "উদ্দীপকের সলিমুল্লাহর মনোভাব 'শুভা' গল্পের কোন দিকটি ফুটিয়ে তোলে—ব্যাখ্যা কর।",
              "gha": "\"বিশেষ চাহিদাসম্পন্ন শিশুদের প্রতি সহানুভূতিশীল দৃষ্টিভঙ্গিই মানবিক সমাজ গঠনের পূর্বশর্ত।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: প্রবাসে দীর্ঘদিন অবস্থান করেও সজল তার নিজ গ্রামের নদী ও মাটির গন্ধ ভুলতে পারেন না। সুযোগ পেলেই তিনি মাতৃভূমির স্মৃতিচারণ করেন।",
              "ka": "'কপোতাক্ষ নদ' কোন ধরণের কবিতা?",
              "kha": "কবি কপোতাক্ষ নদকে স্নেহের ডোরে বাঁধার কথা বলেছেন কেন?",
              "ga": "উদ্দীপকের সজলের অনুভূতির সাথে 'কপোতাক্ষ নদ' কবিতার কবির মানসিকতার তুলনা কর।",
              "gha": "\"স্বদেশপ্রীতি ও স্বদেশের স্মৃতিকাতরতা মানুষের হৃদয়ে চির অমলিন থাকে।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "বাংলা ২য় পত্র": {
      "ব্যাকরণ ও নির্মিতি": {
        "ধ্বনিতত্ত্ব ও সমাস": {
          "MCQ": [
            {"q": "বাংলা ভাষায় মৌলিক স্বরধ্বনি কয়টি?", "options": ["ক) ৭টি", "খ) ১১টি", "গ) ৩৯টি", "ঘ) ২৫টি"], "answer": "ক) ৭টি"},
            {"q": "ণ-ত্ব ও ষ-ত্ব বিধান ব্যাকরণের কোন অংশে আলোচিত হয়?", "options": ["ক) ধ্বনিতত্ত্ব", "খ) রূপতত্ত্ব", "গ) বাক্যতত্ত্ব", "ঘ) অর্থতত্ত্ব"], "answer": "ক) ধ্বনিতত্ত্ব"},
            {"q": "পরপদের অর্থ প্রধান হয় কোন সমাসে?", "options": ["ক) তৎপুরুষ সমাস", "খ) দ্বন্দ্ব সমাস", "গ) বহুব্রীহি সমাস", "ঘ) অব্যয়ীভাব সমাস"], "answer": "ক) তৎপুরুষ সমাস"},
            {"q": "'রাজর্ষি' শব্দটির সঠিক ব্যাসবাক্য কোনটি?", "options": ["ক) যিনি রাজা তিনিই ঋষি", "খ) রাজার ঋষি", "গ) রাজা ও ঋষি", "ঘ) রাজার ন্যায় ঋষি"], "answer": "ক) যিনি রাজা তিনিই ঋষি"},
            {"q": "স্বরসন্ধির উদাহরণ কোনটি?", "options": ["ক) বিদ্যালয় (বিদ্যা + আলয়)", "খ) দিগন্ত", "গ) সংবাদ", "ঘ) নিচ্ছল"], "answer": "ক) বিদ্যালয় (বিদ্যা + আলয়)"},
            {"q": "কৃৎ প্রত্যয় সাধিত শব্দকে কী বলে?", "options": ["ক) কৃদন্ত পদ", "খ) তদ্ধিতান্ত পদ", "গ) সমাসবদ্ধ পদ", "ঘ) উপসর্গ"], "answer": "ক) কৃদন্ত পদ"},
            {"q": "বিশেষ্যের পরিবর্তে যে পদ ব্যবহৃত হয় তাকে কী বলে?", "options": ["ক) সর্বনাম পদ", "খ) বিশেষণ পদ", "গ) অব্যয় পদ", "ঘ) ক্রিয়াপদ"], "answer": "ক) সর্বনাম পদ"},
            {"q": "বাক্যের অর্থগত শ্রেণিবিভাগ কয়টি?", "options": ["ক) ৫টি", "খ) ৩টি", "গ) ৪টি", "ঘ) ২টি"], "answer": "ক) ৫টি"},
            {"q": "'হাতে খড়ি' কোন সমাসের উদাহরণ?", "options": ["ক) বহুব্রীহি সমাস (অলুক বহুব্রীহি)", "খ) তৎপুরুষ সমাস", "গ) কর্মধারয় সমাস", "ঘ) দ্বিগু সমাস"], "answer": "ক) বহুব্রীহি সমাস (অলুক বহুব্রীহি)"},
            {"q": "ণ-ত্ব বিধান অনুযায়ী কোন যুক্তবর্ণে 'ণ' হয়?", "options": ["ক) ট-বর্গের পূর্বে", "খ) ত-বর্গের পূর্বে", "গ) ক-বর্গের পূর্বে", "ঘ) চ-বর্গের পূর্বে"], "answer": "ক) ট-বর্গের পূর্বে"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ব্যাকরণগত সঠিক শব্দ প্রয়োগ ও সঠিক পদ সংযোগ ভাষার সৌন্দর্য ও ভাব প্রকাশের স্পষ্টতা বাড়ায়। সমাস ও প্রত্যয় সাধিত শব্দ ভাষাকে সংক্ষেপ করে।",
              "ka": "সমাস শব্দের অর্থ কী?",
              "kha": "দ্বন্দ্ব সমাস ও বহুব্রীহি সমাসের দুটি প্রধান পার্থক্য লেখ।",
              "ga": "উদ্দীপকের আলোকে পদসংক্ষেপণে সমাসের ভূমিকা ব্যাখ্যা কর।",
              "gha": "\"শব্দ গঠনের প্রক্রিয়া জানা ছাড়া প্রমিত ভাষারীতি অনুসরণ করা সম্ভব নয়।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: ধ্বনির সঠিক উচ্চারণ ও যুক্তবর্ণের নিয়ম না জানলে লেখার ক্ষেত্রে ভুল হওয়ার সম্ভাবনা থাকে। ণ-ত্ব ও ষ-ত্ব বিধান তৎসম শব্দ ব্যবহারের সঠিক নিয়ম নির্ধারণ করে।",
              "ka": "ষ-ত্ব বিধান বলতে কী বোঝায়?",
              "kha": "কোন কোন ক্ষেত্রে ণ-ত্ব বিধান প্রযোজ্য হয় না—উদাহরণসহ লেখ।",
              "ga": "উদ্দীপকে উল্লিখিত বিধান দুটি বাংলা বানানের শুদ্ধতায় কী ভূমিকা রাখে তা ব্যাখ্যা কর।",
              "gha": "\"বাংলা ভাষার প্রমিত রূপ বজায় রাখতে ব্যাকরণের নিয়ম মানা অপরিহার্য।\"—উক্তিটির সত্যতা যাচাই কর।"
            }
          ]
        }
      }
    },
    "English 1st Paper": {
      "Reading & Writing": {
        "Unit 1: Good Citizens & Environment": {
          "MCQ": [
            {"q": "What is the primary responsibility of a good citizen?", "options": ["a) Obeying laws and helping others", "b) Accumulating wealth", "c) Living in isolation", "d) Ignoring society"], "answer": "a) Obeying laws and helping others"},
            {"q": "What causes environmental pollution significantly?", "options": ["a) Industrial waste and deforestation", "b) Tree plantation", "c) Clean energy", "d) Organic farming"], "answer": "a) Industrial waste and deforestation"},
            {"q": "Which one is a renewable energy source?", "options": ["a) Solar power", "b) Coal", "c) Petroleum", "d) Natural gas"], "answer": "a) Solar power"},
            {"q": "What is the synonym of 'Responsibility'?", "options": ["a) Duty", "b) Neglect", "c) Freedom", "d) Avoidance"], "answer": "a) Duty"},
            {"q": "Antonym of 'Destroy' is-", "options": ["a) Create", "b) Damage", "c) Ruin", "d) Harm"], "answer": "a) Create"},
            {"q": "Global warming is mainly caused by-", "options": ["a) Greenhouse gases", "b) Oxygen", "c) Fresh water", "d) Reforestation"], "answer": "a) Greenhouse gases"},
            {"q": "Which organization works for global peace and cooperation?", "options": ["a) UNO", "b) FIFA", "c) ICC", "d) WTO"], "answer": "a) UNO"},
            {"q": "What does 'E-learning' stand for?", "options": ["a) Electronic Learning", "b) Easy Learning", "c) Early Learning", "d) Essential Learning"], "answer": "a) Electronic Learning"},
            {"q": "Choose the correct sentence:", "options": ["a) Honesty is the best policy.", "b) Honesty is a best policy.", "c) Honesty is best policy.", "d) Honesty is an best policy."], "answer": "a) Honesty is the best policy."},
            {"q": "What is the key factor to success according to good habits?", "options": ["a) Perseverance and discipline", "b) Laziness", "c) Luck only", "d) Procrastination"], "answer": "a) Perseverance and discipline"}
          ],
          "CQ": [
            {
              "stem": "Read the text: To be a good citizen, a person needs to acquire knowledge, skill, and a good attitude towards society. Good citizens respect laws and contribute to environmental conservation.",
              "ka": "What qualities make a person a good citizen?",
              "kha": "Why is obeying laws essential for social harmony?",
              "ga": "Summarize the key duties of a good citizen based on the passage.",
              "gha": "\"Individual responsibility leads to national growth.\" Evaluate the statement in light of the text."
            },
            {
              "stem": "Read the text: Deforestation and excessive carbon emissions are threatening global climate stability. Planting trees and using renewable energy can save our planet.",
              "ka": "What is the main cause of climate change?",
              "kha": "How can tree plantation help combat global warming?",
              "ga": "Explain the importance of adopting renewable energy sources.",
              "gha": "\"Environmental protection is not an option, but a necessity for human survival.\" Justify."
            }
          ]
        }
      }
    },
    "English 2nd Paper": {
      "Grammar & Composition": {
        "Grammar Rules & Transformation": {
          "MCQ": [
            {"q": "Change the voice: 'She writes a letter.'", "options": ["a) A letter is written by her.", "b) A letter was written by her.", "c) A letter has been written by her.", "d) A letter is writing by her."], "answer": "a) A letter is written by her."},
            {"q": "Which one is a Complex sentence?", "options": ["a) Though he is poor, he is honest.", "b) He is poor but honest.", "c) He is a poor and honest man.", "d) Being poor, he is honest."], "answer": "a) Though he is poor, he is honest."},
            {"q": "Identify the tag question: 'Let us go for a walk, ___?'", "options": ["a) shall we", "b) don't we", "c) won't we", "d) can we"], "answer": "a) shall we"},
            {"q": "Fill in the blank: 'He is senior ___ me.'", "options": ["a) to", "b) than", "c) from", "d) with"], "answer": "a) to"},
            {"q": "Which tense expresses an action completed in the recent past?", "options": ["a) Present Perfect", "b) Past Simple", "c) Present Continuous", "d) Future Simple"], "answer": "a) Present Perfect"},
            {"q": "Choose the correct indirect speech: He said, 'I am ill.'", "options": ["a) He said that he was ill.", "b) He said that I was ill.", "c) He says that he is ill.", "d) He said that he is ill."], "answer": "a) He said that he was ill."},
            {"q": "Noun form of the verb 'Know' is-", "options": ["a) Knowledge", "b) Known", "c) Knowing", "d) Knowable"], "answer": "a) Knowledge"},
            {"q": "Identify the degree: 'No other boy in the class is as good as Rahi.'", "options": ["a) Positive", "b) Comparative", "c) Superlative", "d) None"], "answer": "a) Positive"},
            {"q": "Choose the correct right form of verb: 'Neither he nor his friends ___ present.'", "options": ["a) were", "b) was", "c) is", "d) has"], "answer": "a) were"},
            {"q": "What is the antonym of 'Optimistic'?", "options": ["a) Pessimistic", "b) Hopeful", "c) Confident", "d) Positive"], "answer": "a) Pessimistic"}
          ],
          "CQ": [
            {
              "stem": "Transform the following sentences as directed: (a) Bangladesh is a beautiful country. (Exclamatory) (b) It is richer in natural beauty than most other countries. (Superlative) (c) Who does not love his motherland? (Assertive) (d) We should protect our nature. (Passive)",
              "ka": "What is a compound sentence?",
              "kha": "Explain the rule of converting Comparative degree into Superlative degree.",
              "ga": "Transform sentences (a), (b), (c), and (d) according to instructions.",
              "gha": "\"Mastery over sentence transformation improves overall writing clarity.\" Analyze."
            },
            {
              "stem": "Complete the passage with suitable verbs in correct forms: Education (a) ___ (enlighten) the mind. It (b) ___ (help) us distinguish between right and wrong. A person who (c) ___ (be) educated contributes to national progress.",
              "ka": "What is Subject-Verb Agreement?",
              "kha": "State the rule for using Present Perfect Tense with 'recently' or 'already'.",
              "ga": "Supply the correct form of verbs for (a), (b), and (c).",
              "gha": "\"Grammatical correctness is the foundation of effective language skills.\" Discuss."
            }
          ]
        }
      }
    },
    "গণিত": {
      "বীজগণিত ও জ্যামিতি": {
        "বীজগণিতীয় রাশি ও সেটের ধারণা": {
          "MCQ": [
            {"q": "A = {1, 2, 3} হলে সেট A-এর উপসেট (Subset) সংখ্যা কত?", "options": ["ক) ৮টি", "খ) ৬টি", "গ) ৪টি", "ঘ) ২টি"], "answer": "ক) ৮টি"},
            {"q": "(a + b)³ এর সঠিক সূত্র কোনটি?", "options": ["ক) a³ + 3a²b + 3ab² + b³", "খ) a³ - 3a²b + 3ab² - b³", "গ) a³ + b³ + ab", "ঘ) a³ + 3ab + b³"], "answer": "ক) a³ + 3a²b + 3ab² + b³"},
            {"q": "ফাঁকা সেটকে (Empty Set) কোন প্রতীক দিয়ে প্রকাশ করা হয়?", "options": ["ক) ∅", "খ) ∈", "গ) ⊂", "ঘ) ∪"], "answer": "ক) ∅"},
            {"q": "log₁₀ 100 এর মান কত?", "options": ["ক) ২", "খ) ১০", "গ) ১", "ঘ) ০"], "answer": "ক) ২"},
            {"q": "2x + 3y = 12 এবং x - y = 1 সমীকরণের সমাধান কোনটি?", "options": ["ক) (3, 2)", "খ) (2, 3)", "গ) (4, 1)", "ঘ) (1, 4)"], "answer": "ক) (3, 2)"},
            {"q": "একটি ত্রিভুজের তিন কোণের যোগফল কত ডিগ্রি?", "options": ["ক) ১৮০°", "খ) ৯০°", "গ) ৩৬০°", "ঘ) ২৭০°"], "answer": "ক) ১৮০°"},
            {"q": "বৃত্তের বৃহত্তম জ্যা-কে কী বলা হয়?", "options": ["ক) ব্যাস", "খ) ব্যাসার্ধ", "গ) পরিধি", "ঘ) চাপ"], "answer": "ক) ব্যাস"},
            {"q": "x² - 5x + 6 = 0 সমীকরণের মূলদ্বয় কোনটি?", "options": ["ক) 2, 3", "খ) -2, -3", "গ) 1, 6", "ঘ) -1, -6"], "answer": "ক) 2, 3"},
            {"q": "বৃত্তস্থ চতুর্ভুজের বিপরীত কোণদ্বয়ের যোগফল কত?", "options": ["ক) ১৮০°", "খ) ৯০°", "গ) ৩৬০°", "ঘ) ২৭০°"], "answer": "ক) ১৮০°"},
            {"q": "a⁰ এর মান কত (যেখানে a ≠ 0)?", "options": ["ক) ১", "খ) ০", "গ) a", "ঘ) অসংজ্ঞায়িত"], "answer": "ক) ১"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: U = {1, 2, 3, 4, 5, 6, 7}, A = {2, 4, 6} এবং B = {3, 5, 6} তিনটি সেট।",
              "ka": "A ∩ B নির্ণয় কর।",
              "kha": "A ∪ B এবং (A ∪ B)′ নির্ণয় কর।",
              "ga": "ডি মরগানের নিয়মঃ (A ∪ B)′ = A′ ∩ B′ সত্যতা যাচাই কর।",
              "gha": "\"ভেনচিত্র ও সেট তত্ত্ব গাণিতিক বিন্যাসকে সহজতর করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: x + 1/x = 3 একটি বীজগণিতীয় সমীকরণ।",
              "ka": "x - 1/x এর মান কত?",
              "kha": "প্রমাণ কর যে, x³ + 1/x³ = 18।",
              "ga": "x⁴ + 1/x⁴ এর মান নির্ণয় কর।",
              "gha": "\"বীজগণিতীয় সূত্রের সঠিক রূপান্তর জটিল রাশিমালা সমাধানে সহায়ক।\"—গাণিতিকভাবে প্রমাণ কর।"
            }
          ]
        }
      }
    },
    "বিজ্ঞান": {
      "পদার্থ ও জীববিজ্ঞান": {
        "বল, শক্তি ও জীবনের গঠন": {
          "MCQ": [
            {"q": "নিউটন-এর গতির দ্বিতীয় সূত্র থেকে কিসের পরিমাপ পাওয়া যায়?", "options": ["ক) বল", "খ) ভরবেগ", "গ) কাজ", "ঘ) ত্বরণ"], "answer": "ক) বল"},
            {"q": "কাজ করার সামর্থ্যকে কী বলে?", "options": ["ক) শক্তি", "খ) ক্ষমতা", "গ) বল", "ঘ) বেগ"], "answer": "ক) শক্তি"},
            {"q": "উদ্ভিদের শালোকসংশ্লেষণ প্রক্রিয়ায় উৎপন্ন প্রধান উপাদান কোনটি?", "options": ["ক) গ্লুকোজ ও অক্সিজেন", "খ) নাইট্রোজেন", "গ) কার্বন ডাই অক্সাইড", "ঘ) মিথেন"], "answer": "ক) গ্লুকোজ ও অক্সিজেন"},
            {"q": "মানবদেহের প্রধান রেচন অঙ্গ কোনটি?", "options": ["ক) বৃক্ক (Kidney)", "খ) যকৃৎ", "গ) ফুসফুস", "ঘ) ত্বক"], "answer": "ক) বৃক্ক (Kidney)"},
            {"q": "মহাকর্ষীয় ধ্রুবক (G) এর একক কোনটি?", "options": ["ক) N m² kg⁻²", "খ) N m kg⁻¹", "গ) N m/s", "ঘ) m/s²"], "answer": "ক) N m² kg⁻²"},
            {"q": "কোষের বংশগতির ধারক ও বাহক কাকে বলা হয়?", "options": ["ক) ক্রোমোজোম/DNA", "খ) আরএনএ", "গ) প্লাস্টিড", "ঘ) সাইটোপ্লাজম"], "answer": "ক) ক্রোমোজোম/DNA"},
            {"q": "শব্দ সঞ্চালনের বেগ সবচেয়ে বেশি কোন মাধ্যমে?", "options": ["ক) কঠিন", "খ) তরল", "গ) বায়বীয়", "ঘ) শূন্যস্থান"], "answer": "ক) কঠিন"},
            {"q": "নিচের কোনটি পরিপাকতন্ত্রের অংশ?", "options": ["ক) পাকস্থলী", "খ) ধমনি", "গ) নিউরন", "ঘ) ট্রাকিয়া"], "answer": "ক) পাকস্থলী"},
            {"q": "শক্তির এসআই (SI) একক কোনটি?", "options": ["ক) জুল (Joule)", "খ) ওয়াট", "গ) পাউণ্ড", "ঘ) নিউটন"], "answer": "ক) জুল (Joule)"},
            {"q": "নিউক্লিয়াস আবিষ্কার করেন কে?", "options": ["ক) রবার্ট ব্রাউন", "খ) রবার্ট হুক", "গ) ডাল্টন", "ঘ) মেনডেল"], "answer": "ক) রবার্ট ব্রাউন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ১০ কেজি ভরের একটি বস্তুকে ৫ m/s² ত্বরণে ধাক্কা দেওয়া হলো। বস্তুটি ২০ মিটার দূরত্ব অতিক্রম করে থেমে গেল।",
              "ka": "বল কাকে বলে?",
              "kha": "নিউটন ও ডাইনের মধ্যে সম্পর্ক ব্যাখ্যা কর।",
              "ga": "প্রযুক্ত বলের মান হিসাব কর।",
              "gha": "\"শক্তির রূপান্তর ও কাজের পরিমাণ প্রযুক্ত বলের ওপর নির্ভরশীল।\"—উদ্দীপকের তথ্যের আলোকে গাণিতিক বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: উদ্ভিদ সূর্যালোক ও কার্বন ডাই অক্সাইড ব্যবহার করে শর্করা জাতীয় খাদ্য তৈরি করে। অপরদিকে মানুষ শ্বসনের মাধ্যমে সেই খাদ্য ভেঙে শক্তি লাভ করে।",
              "ka": "শালোকসংশ্লেষণ কী?",
              "kha": "শ্বসন ও শালোকসংশ্লেষণের প্রধান পার্থক্যগুলো কী কী?",
              "ga": "উদ্দীপকে বর্ণিত শক্তি উৎপাদন প্রক্রিয়াটি উদ্ভিদদেহে কীভাবে ঘটে ব্যাখ্যা কর।",
              "gha": "\"শালোকসংশ্লেষণ ও শ্বসনের পারস্পরিক সামঞ্জস্যের ওপরই পৃথিবীর জীবজগতের অস্তিত্ব টিকে রয়েছে।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "বাংলাদেশ ও বিশ্বপরিচয়": {
      "ইতিহাস ও সমাজ ব্যবস্থা": {
        "রাষ্ট্র, সংবিধান ও সমাজচিত্র": {
          "MCQ": [
            {"q": "বাংলাদেশের স্বাধীনতার ঘোষণা প্রথম কে পাঠ করেন/প্রদান করেন?", "options": ["ক) বঙ্গবন্ধু শেখ মুজিবুর রহমান", "খ) তাজউদ্দীন আহমদ", "গ) সৈয়দ নজরুল ইসলাম", "ঘ) এম এ জি ওসমানী"], "answer": "ক) বঙ্গবন্ধু শেখ মুজিবুর রহমান"},
            {"q": "বাংলাদেশের সংবিধান কার্যকর হয় কত তারিখে?", "options": ["ক) ১৬ই ডিসেম্বর ১৯৭২", "খ) ২৬শে মার্চ ১৯৭২", "গ) ১৭ই এপ্রিল ১৯৭১", "ঘ) ২৬শে নভেম্বর ১৯৭২"], "answer": "ক) ১৬ই ডিসেম্বর ১৯৭২"},
            {"q": "রাষ্ট্রের উপাদান কয়টি?", "options": ["ক) ৪টি", "খ) ৩টি", "গ) ৫টি", "ঘ) ৬টি"], "answer": "ক) ৪টি"},
            {"q": "জাতিসংঘের নিরাপত্তা পরিষদের স্থায়ী সদস্য সংখ্যা কত?", "options": ["ক) ৫টি", "খ) ১০টি", "গ) ১৫টি", "ঘ) ৭টি"], "answer": "ক) ৫টি"},
            {"q": "বাঙালি সংস্কৃতিতে 'পহেলা বৈশাখ' উদযাপনের প্রবর্তক কে?", "options": ["ক) সম্রাট আকবর", "খ) লক্ষণ সেন", "গ) আলাউদ্দিন হোসেন শাহ", "ঘ) সিরাজউদ্দৌলা"], "answer": "ক) সম্রাট আকবর"},
            {"q": "বাংলাদেশের সর্বোচ্চ আইন কোনটি?", "options": ["ক) সংবিধান", "খ) দণ্ডবিধি", "গ) দেওয়ানি আইন", "ঘ) প্রথাগত আইন"], "answer": "ক) সংবিধান"},
            {"q": "সামাজিক নিয়মনীতি লঙ্ঘনের ফলে সমাজে কী সৃষ্টি হয়?", "options": ["ক) সামাজিক অবক্ষয় ও বিশৃঙ্খলা", "খ) উন্নয়ন", "গ) প্রবৃদ্ধি", "ঘ) সমতা"], "answer": "ক) সামাজিক অবক্ষয় ও বিশৃঙ্খলা"},
            {"q": "টেকসই উন্নয়ন লক্ষ্যমাত্রা (SDG) এর লক্ষ্য সংখ্যা কতটি?", "options": ["ক) ১৭টি", "খ) ৮টি", "গ) ১২টি", "ঘ) ২০টি"], "answer": "ক) ১৭টি"},
            {"q": "বাংলাদেশের জাতীয় সংসদের আসন সংখ্যা কতটি?", "options": ["ক) ৩৫০টি", "খ) ৩০০টি", "গ) ৪০০টি", "ঘ) ৩২০টি"], "answer": "ক) ৩৫০টি"},
            {"q": "অর্থনৈতিক উন্নয়নের অন্যতম প্রধান সূচক কোনটি?", "options": ["ক) মাথাপিছু আয়", "খ) মোট জনসংখ্যা", "গ) জমির পরিমাণ", "ঘ) আমদানির পরিমাণ"], "answer": "ক) মাথাপিছু আয়"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ১৯৭২ সালে প্রণীত বাংলাদেশের সংবিধানের অন্যতম মূলনীতি হলো গণতন্ত্র, সমাজতন্ত্র, ধর্মনিরপেক্ষতা ও জাতীয়তাবাদ। এই সংবিধান আইনের শাসন প্রতিষ্ঠায় মূল ভিত্তি হিসেবে কাজ করছে।",
              "ka": "বাংলাদেশের সংবিধান প্রণয়ন কমিটির প্রধান কে ছিলেন?",
              "kha": "সংবিধানকে রাষ্ট্রের সর্বোচ্চ দলিল বলা হয় কেন?",
              "ga": "উদ্দীপকে বর্ণিত চার মূলনীতির আলোকে আমাদের গণতান্ত্রিক ব্যবস্থার ব্যাখ্যা দাও।",
              "gha": "\"সংবিধান রক্ষা করা এবং সংবিধানের অনুশাসন মেনে চলা প্রতিটি নাগরিকের মৌলিক দায়িত্ব।\"—আলোচনা কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি সমাজে যুবসমাজের একটি অংশ উপযুক্ত শিক্ষার অভাবে বিভিন্ন সামাজিক অপরাধ ও মাদকে জড়িয়ে পড়ছে। স্থানীয় সচেতন নাগরিকেরা তাদের পুনর্বাসনে উদ্যোগ গ্রহণ করেছেন।",
              "ka": "সামাজিক ব্যাধি কী?",
              "kha": "কিশোর অপরাধের প্রধান কারণগুলো ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সমস্যার জন্য সমাজ কাঠামোর কোন ঘাটতি দায়ী তা আলোচনা কর।",
              "gha": "\"সামাজিক সচেতনতা ও সঠিক শিক্ষা প্রয়োগই অপরাধমুক্ত সমাজ গঠনের পথ দেখাতে পারে।\"—মতামত দাও।"
            }
          ]
        }
      }
    },
    "রসায়ন": {
      "বিজ্ঞান শাখা": {
        "পদার্থের গঠন ও রাসায়নিক বন্ধন": {
          "MCQ": [
            {"q": "সোডিয়াম (Na) এর পারমাণবিক সংখ্যা কত?", "options": ["ক) ১১", "খ) ১২", "গ) ১৩", "ঘ) ১৯"], "answer": "ক) ১১"},
            {"q": "নিচের কোনটি নিষ্ক্রিয় গ্যাস?", "options": ["ক) আর্গন (Ar)", "খ) অক্সিজেন (O₂)", "গ) ক্লোরিন (Cl₂)", "ঘ) নাইট্রোজেন (N₂)"], "answer": "ক) আর্গন (Ar)"},
            {"q": "আয়নিক বন্ধন গঠিত হয় কাদের মধ্যে?", "options": ["ক) ধাতু ও অধাতু", "খ) দুই অধাতু", "গ) দুই ধাতু", "ঘ) নিষ্ক্রিয় গ্যাস"], "answer": "ক) ধাতু ও অধাতু"},
            {"q": "পানি (H₂O) অনুতে কোন ধরণের বন্ধন বিদ্যমান?", "options": ["ক) সমযোজী বন্ধন", "খ) আয়নিক বন্ধন", "গ) ধাতু বন্ধন", "ঘ) হাইড্রোজেন বন্ধন"], "answer": "ক) সমযোজী বন্ধন"},
            {"q": "কার্বনের যোজনী সাধারণ অবস্থায় কত?", "options": ["ক) ৪", "খ) ২", "গ) ৩", "ঘ) ১"], "answer": "ক) ৪"},
            {"q": "১ মোল পদার্থে কতটি অণু থাকে?", "options": ["ক) ৬.০২৩ × ১০²³", "খ) ৩.০১ × ১০²³", "গ) ১.৬৬ × ১০⁻²⁴", "ঘ) ৯.৮ × ১০⁸"], "answer": "ক) ৬.০২৩ × ১০²³"},
            {"q": "পর্যায় সারণির মূল ভিত্তি কোনটি?", "options": ["ক) পারমাণবিক সংখ্যা (ইলেকট্রন বিন্যাস)", "খ) ভর সংখ্যা", "গ) গলনাঙ্ক", "ঘ) ঘনত্ব"], "answer": "ক) পারমাণবিক সংখ্যা (ইলেকট্রন বিন্যাস)"},
            {"q": "ক্ষার ধাতুর সমূহের গ্রুপ নম্বর কত?", "options": ["ক) গ্রুপ ১", "খ) গ্রুপ ২", "গ) গ্রুপ ১৭", "ঘ) গ্রুপ ১৮"], "answer": "ক) গ্রুপ ১"},
            {"q": "মিথেন (CH₄) গ্যাসে কার্বনের শতকরা পরিমাণ কত?", "options": ["ক) ৭৫%", "খ) ২৫%", "গ) ৫০%", "ঘ) ৮০%"], "answer": "ক) ৭৫%"},
            {"q": "নিচের কোনটি এসিডের জলীয় দ্রবণে উৎপন্ন হয়?", "options": ["ক) H⁺ আয়ন", "খ) OH⁻ আয়ন", "গ) Na⁺ আয়ন", "ঘ) Cl⁻ আয়ন"], "answer": "ক) H⁺ আয়ন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: মৌল A (পারমাণবিক সংখ্যা ১১) এবং মৌল B (পারমাণবিক সংখ্যা ১৭) পরস্পর যুক্ত হয়ে AB যৌগ গঠন করে।",
              "ka": "যোজনী কাকে বলে?",
              "kha": "নিষ্ক্রিয় গ্যাসগুলো রাসায়নিকভাবে নিষ্ক্রিয় কেন?",
              "ga": "A ও B মৌলের মধ্যে বন্ধন গঠন প্রক্রিয়া ইলেকট্রন বিন্যাসের সাহায্যে ব্যাখ্যা কর।",
              "gha": "\"গঠিত AB যৌগটি পানিতে দ্রবণীয় এবং জলীয় দ্রবণে বিদ্যুৎ পরিবহন করে।\"—যাচাই কর।"
            },
            {
              "stem": "উদ্দীপক ২: গবেষণাগারে ৫ গ্রাম কার্বনকে প্রয়োজনীয় অক্সিজেনে দহন করে কার্বন ডাই অক্সাইড (CO₂) তৈরি করা হলো।",
              "ka": "মোলার ভর কী?",
              "kha": "এসটিপি (STP)-তে ১ মোল গ্যাসের আয়তন কত?",
              "ga": "উদ্দীপকে উৎপন্ন CO₂ এর মোলার ভর ও অনুর সংখ্যা হিসাব কর।",
              "gha": "\"ভরের সংরক্ষণ সূত্র রাসায়নিক বিক্রিয়ার সমতা রক্ষা করে।\"—বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "পদার্থবিজ্ঞান": {
      "বিজ্ঞান শাখা": {
        "গতি, বল ও শক্তি": {
          "MCQ": [
            {"q": "পরন্ত বস্তুর সূত্র প্রদান করেন কে?", "options": ["ক) গ্যালিলিও", "খ) নিউটন", "গ) আইন্সটাইন", "ঘ) আর্কিমিডিস"], "answer": "ক) গ্যালিলিও"},
            {"q": "ত্বরণের একক কোনটি?", "options": ["ক) m/s²", "খ) m/s", "গ) N/m", "ঘ) kg·m/s"], "answer": "ক) m/s²"},
            {"q": "ভরবেগের একক কোনটি?", "options": ["ক) kg·m/s", "খ) N·m", "গ) J/s", "ঘ) m/s²"], "answer": "ক) kg·m/s"},
            {"q": "১ অশ্বক্ষমতা (HP) = কত ওয়াট?", "options": ["ক) ৭৪৬ ওয়াট", "খ) ১০০০ ওয়াট", "গ) ৫০০ ওয়াট", "ঘ) ৭০০ ওয়াট"], "answer": "ক) ৭৪৬ ওয়াট"},
            {"q": "শব্দ একটি কী ধরণের তরঙ্গ?", "options": ["ক) অনুদৈর্ঘ্য তরঙ্গ", "খ) অনুপ্রস্থ তরঙ্গ", "গ) তড়িৎচুম্বকীয় তরঙ্গ", "ঘ) স্থিতিস্থাপক তরঙ্গ নয়"], "answer": "ক) অনুদৈর্ঘ্য তরঙ্গ"},
            {"q": "অবতল দর্পণে ফোকাস দূরত্ব ব্যাসার্ধের কত গুণ?", "options": ["ক) অর্ধেক (r/2)", "খ) দ্বিগুণ", "গ) সমান", "ঘ) তিনগুণ"], "answer": "ক) অর্ধেক (r/2)"},
            {"q": "স্থির অবস্থান থেকে বিনামূল্যে পড়ন্ত বস্তুর বেগ সময়ের সাথে-", "options": ["ক) সমানুপাতিক", "খ) ব্যস্তানুপাতিক", "গ) বর্গের সমানুপাতিক", "ঘ) অপরিবর্তিত"], "answer": "ক) সমানুপাতিক"},
            {"q": "তাপমাত্রার পরিমাপক স্কেল কেলভিনে সেলসিয়াসের সম্পর্ক কোনটি?", "options": ["ক) K = C + ২৭৩.১৫", "খ) K = C - ২৭৩", "গ) K = ২৭৩ - C", "ঘ) C = K + ২৭৩"], "answer": "ক) K = C + ২৭৩.১৫"},
            {"q": "কোন বলের কারণে গ্রহগুলো সূর্যের চারদিকে ঘোরে?", "options": ["ক) মহাকর্ষ বল", "খ) নিউক্লীয় বল", "গ) তড়িৎ বল", "ঘ) ঘর্ষণ বল"], "answer": "ক) মহাকর্ষ বল"},
            {"q": "কাজ করার হারকে কী বলে?", "options": ["ক) ক্ষমতা", "খ) শক্তি", "গ) বেগ", "ঘ) ভরবেগ"], "answer": "ক) ক্ষমতা"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: ৫ কেজি ভরের একটি বস্তুকে ৫০ মিটার উঁচু দালানের ছাদ থেকে নিচে ফেলে দেওয়া হলো। (g = 9.8 m/s²)",
              "ka": "মুক্তভাবে পড়ন্ত বস্তু কী?",
              "kha": "গতিশক্তি ও বিভব শক্তির মধ্যে সম্পর্ক লেখ।",
              "ga": "ভূমি স্পর্শ করার পূর্বমুহূর্তে বস্তুটির বেগ নির্ণয় কর।",
              "gha": "\"পড়ন্ত বস্তুর ক্ষেত্রে যান্ত্রিক শক্তির সংরক্ষণশীলতা নীতি সংরক্ষিত হয়।\"—গাণিতিক প্রমাণের সাহায্যে বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি গাড়ির বেগ ১০ সেকেন্ডে স্থির অবস্থান থেকে বৃদ্ধি পেয়ে ২০ m/s হলো। গাড়িটির ভর ১০০০ কেজি।",
              "ka": "ত্বরণ কাকে বলে?",
              "kha": "ঘর্ষণ বল কীভাবে গতির বিপরীত কাজ করে—ব্যাখ্যা কর।",
              "ga": "গাড়িটির ত্বরণ এবং প্রযুক্ত বলের মান হিসাব কর।",
              "gha": "\"ত্বরণ ও ভরের গুণফলই প্রযুক্ত বল নির্দেশ করে।\"—নিউটনীয় গতির আলোকে মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "জীববিজ্ঞান": {
      "বিজ্ঞান শাখা": {
        "কোষ ও অঙ্গসংস্থান": {
          "MCQ": [
            {"q": "কোষের শক্তিঘর (Powerhouse) কাকে বলে?", "options": ["ক) মাইটোকন্ড্রিয়া", "খ) গলগি বডি", "গ) নিউক্লিয়াস", "ঘ) লাইসোসোম"], "answer": "ক) মাইটোকন্ড্রিয়া"},
            {"q": "ডিএনএ (DNA) এর দ্বিসূত্রক মডেল আবিষ্কার করেন কে?", "options": ["ক) ওয়াটসন ও ক্রিক", "খ) রবার্ট হুক", "গ) মেনডেল", "ঘ) ল্যামার্ক"], "answer": "ক) ওয়াটসন ও ক্রিক"},
            {"q": "উদ্ভিদ কোষের প্রধান বৈশিষ্ট্য কোনটি?", "options": ["ক) কোষ প্রাচীরের উপস্থিতি", "খ) সেন্টিওল", "গ) ছোট কোষগহ্বর", "ঘ) লাইসোসোমের সংখ্যাধিক্য"], "answer": "ক) কোষ প্রাচীরের উপস্থিতি"},
            {"q": "মানুষের লোহিত রক্তকণিকায় কোনটি অনুপস্থিত?", "options": ["ক) নিউক্লিয়াস", "খ) হিমোগ্লোবিন", "গ) প্লাজমা", "ঘ) ঝিল্লি"], "answer": "ক) নিউক্লিয়াস"},
            {"q": "কোন বিভাজনের মাধ্যমে জননকোষ গঠিত হয়?", "options": ["ক) মিওসিস", "খ) অ্যামাইটোসিস", "গ) মাইটোসিস", "ঘ) সাইটোকাইনেসিস"], "answer": "ক) মিওসিস"},
            {"q": "সালোকসংশ্লেষণের প্রধান উপজাত দ্রব্য কোনটি?", "options": ["ক) অক্সিজেন", "খ) কার্বন ডাই অক্সাইড", "গ) নাইট্রোজেন", "ঘ) হাইড্রোজেন"], "answer": "ক) অক্সিজেন"},
            {"q": "উদ্ভিদের পানি পরিহনের জন্য দায়ী কলা কোনটি?", "options": ["ক) জাইলেম", "খ) ফ্লোয়েম", "গ) প্যারেনকাইমা", "ঘ) স্ক্লেরেনকাইমা"], "answer": "ক) জাইলেম"},
            {"q": "মানবদেহের বৃহত্তম গ্রন্থি কোনটি?", "options": ["ক) যকৃৎ (Liver)", "খ) অগ্ন্যাশয়", "গ) থাইরয়েড", "ঘ) পিটুইটারি"], "answer": "ক) যকৃৎ (Liver)"},
            {"q": "কোন ভিটামিনের অভাবে রিকেটস রোগ হয়?", "options": ["ক) ভিটামিন D", "খ) ভিটামিন A", "গ) ভিটামিন C", "ঘ) ভিটামিন K"], "answer": "ক) ভিটামিন D"},
            {"q": "অনুচক্রিকার মূল কাজ কোনটি?", "options": ["ক) রক্ত তঞ্চনে (জমাট বাঁধতে) সাহায্য করা", "খ) অক্সিজেন বহন", "গ) জীবাণু ধ্বংস", "ঘ) তাপ নিয়ন্ত্রণ"], "answer": "ক) রক্ত তঞ্চনে (জমাট বাঁধতে) সাহায্য করা"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: কোষ বিভাজনের দুটি চিত্র লক্ষ্য করা হলো: 'A' প্রক্রিয়ায় দেহের বৃদ্ধি ঘটে এবং 'B' প্রক্রিয়ায় অপত্য কোষে ক্রোমোজোম সংখ্যা অর্ধেক হয়ে যায়।",
              "ka": "অ্যামাইটোসিস বিভাজন কাকে বলে?",
              "kha": "মাইটোসিসকে সমীকরণিক বিভাজন বলা হয় কেন?",
              "ga": "উদ্দীপকের 'A' ও 'B' প্রক্রিয়ার বৈসাদৃশ্য তুলনা কর।",
              "gha": "\"জীবের ধারাবাহিকতা ও প্রজাতি রক্ষায় 'B' প্রক্রিয়ার গুরুত্ব অপরিসীম।\"—উক্তিটি মূল্যায়ন কর।"
            },
            {
              "stem": "উদ্দীপক ২: রফিক পর্যবেক্ষণ করল যে, গাছ মূলের সাহায্যে পানি শোষণ করে পাতায় পাঠায় এবং সূর্যালোকের সাহায্যে শর্করা তৈরি করে।",
              "ka": "প্রস্বেদন কী?",
              "kha": "জাইলেম ও ফ্লোয়েমের কাজ ব্যাখ্যা কর।",
              "ga": "উদ্দীপকে বর্ণিত প্রক্রিয়ায় উদ্ভিদের খাদ্য তৈরি ও পরিবহন প্রক্রিয়া আলোচনা কর।",
              "gha": "\"উদ্ভিদের প্রস্বেদন প্রক্রিয়াকে প্রয়োজনীয় ক্ষতিকর প্রক্রিয়া (Necessary evil) বলা হয়।\"—বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "উচ্চতর গণিত": {
      "বিজ্ঞান শাখা": {
        "সেট, ফাংশন ও জ্যামিতি": {
          "MCQ": [
            {"q": "f(x) = 2x + 1 হলে f(2) এর মান কত?", "options": ["ক) ৫", "খ) ৪", "গ) ৩", "ঘ) ১"], "answer": "ক) ৫"},
            {"q": "ত্রিভুজের পরিকেন্দ্র, ভরকেন্দ্র ও লম্ববিন্দু সম্পর্কিত উপপাদ্য কোনটি?", "options": ["ক) ইউলারের রেখা", "খ) পিথাগোরাসের উপপাদ্য", "গ) অ্যাপোলোনিয়াসের উপপাদ্য", "ঘ) টলেমির উপপাদ্য"], "answer": "ক) ইউলারের রেখা"},
            {"q": "aˣ = b হলে x = কত?", "options": ["ক) logₐ b", "খ) log_b a", "গ) a/b", "ঘ) b/a"], "answer": "ক) logₐ b"},
            {"q": "দ্বিপদী বিস্তৃতি (1 + x)ⁿ এর পদসংখ্যা কত?", "options": ["ক) n + 1", "খ) n", "গ) n - 1", "ঘ) 2n"], "answer": "ক) n + 1"},
            {"q": "একটি ঘনকের ধারের দৈর্ঘ্য a হলে তার আয়তন কত?", "options": ["ক) a³", "খ) 6a²", "গ) √3a", "ঘ) 3a²"], "answer": "ক) a³"},
            {"q": "sin²θ + cos²θ = কত?", "options": ["ক) ১", "খ) ০", "গ) ২", "ঘ) -১"], "answer": "ক) ১"},
            {"q": "(2, 3) ও (5, 7) বিন্দুদ্বয়ের মধ্যবর্তী দূরত্ব কত?", "options": ["ক) ৫", "খ) ২৫", "গ) ৭", "ঘ) ৩"], "answer": "ক) ৫"},
            {"q": "সমকোণী ত্রিভুজের ক্ষেত্রে অ্যাপোলোনিয়াসের উপপাদ্য কিসের বর্ধিত রূপ?", "options": ["ক) পিথাগোরাসের উপপাদ্য", "খ) ইউক্লিডের পদ্ধতি", "গ) টলেমির সূত্র", "ঘ) ভেনচিত্র"], "answer": "ক) পিথাগোরাসের উপপাদ্য"},
            {"q": "Vector A = 2i + 3j হলে এর মান (Magnitude) কত?", "options": ["ক) √13", "খ) ৫", "গ) ১৩", "ঘ) √5"], "answer": "ক) √13"},
            {"q": "tan 45° এর মান কত?", "options": ["ক) ১", "খ) ০", "গ) √৩", "ঘ) ১/√৩"], "answer": "ক) ১"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: F(x) = (2x + 2) / (x - 1) একটি ফাংশন (x ≠ 1)।",
              "ka": "ফাংশনের ডোমেন কাকে বলে?",
              "kha": "উক্ত ফাংশনের ডোমেন ও রেঞ্জ নির্ণয় কর।",
              "ga": "F⁻¹(x) অর্থাৎ বিপরীত ফাংশন নির্ণয় কর।",
              "gha": "\"এক-এক ও অনটূ ফাংশন হওয়ার ওপর বিপরীত ফাংশনের অস্তিত্ব নির্ভর করে।\"—যাচাই কর।"
            },
            {
              "stem": "উদ্দীপক ২: ΔABC এর AD একটি মধ্যমা। AB = 6 cm, AC = 8 cm এবং BC = 10 cm।",
              "ka": "অ্যাপোলোনিয়াসের উপপাদ্যের বাক্যটি লেখ।",
              "kha": "অ্যাপোলোনিয়াসের উপপাদ্য ব্যবহার করে মধ্যমা AD এর দৈর্ঘ্য নির্ণয় কর।",
              "ga": "প্রমাণ কর যে, AB² + AC² = 2(AD² + BD²)।",
              "gha": "\"জ্যামিতিক উপপাদ্য ভেক্টরের সাহায্যেও প্রমাণ করা সম্ভব।\"—উক্তিটির সত্যতা বিচার কর।"
            }
          ]
        }
      }
    },
    "হিসাববিজ্ঞান": {
      "ব্যবসায় শিক্ষা শাখা": {
        "হিসাব সমীকরণ ও জাবেদা": {
          "MCQ": [
            {"q": "হিসাব সমীকরণের মূল উপাদান কোনটি?", "options": ["ক) A = L + E", "খ) A = L - E", "গ) E = A + L", "ঘ) L = A + E"], "answer": "ক) A = L + E"},
            {"q": "হিসাববিজ্ঞানের প্রথম ও প্রাথমিক বই কোনটি?", "options": ["ক) জাবেদা (Journal)", "খ) খতিয়ান (Ledger)", "গ) রেওয়ামিল", "ঘ) নগদান বই"], "answer": "ক) জাবেদা (Journal)"},
            {"q": "খতিয়ানকে বলা হয় হিসাবের-", "options": ["ক) পাকা বই/রাজা", "খ) সহকারী বই", "গ) প্রাথমিক বই", "ঘ) সাময়িক বই"], "answer": "ক) পাকা বই/রাজা"},
            {"q": "ক্যাশ মেমো কে তৈরি করেন?", "options": ["ক) বিক্রেতা", "খ) ক্রেতা", "গ) ব্যাংক", "ঘ) ম্যানেজার"], "answer": "ক) বিক্রেতা"},
            {"q": "সম্পদ বাড়লে ডেবিট এবং কমলে কী হয়?", "options": ["ক) ক্রেডিট", "খ) ডেবিট", "গ) অপরিবর্তিত", "ঘ) সমতা"], "answer": "ক) ক্রেডিট"},
            {"q": "ব্যবসায়িক লেনদেনের লিখিত প্রমাণপত্রকে কী বলে?", "options": ["ক) ভাউচার/দলিল", "খ) খতিয়ান", "গ) রেওয়ামিল", "ঘ) চালান"], "answer": "ক) ভাউচার/দলিল"},
            {"q": "হিসাববিজ্ঞানের জনক কে?", "options": ["ক) লুকা প্যাসিওলি", "খ) অ্যাডাম স্মিথ", "গ) ফিলিপ কোটলার", "ঘ) মার্শাল"], "answer": "ক) লুকা প্যাসিওলি"},
            {"q": "উৎপাদন ব্যয় বা বেতন প্রদান কোন ধরণের হিসাব?", "options": ["ক) নামিক/ব্যয় হিসাব", "খ) সম্পত্তি হিসাব", "গ) দায় হিসাব", "ঘ) মালিকানা স্বত্ব"], "answer": "ক) নামিক/ব্যয় হিসাব"},
            {"q": "দুপাformat জমা পদ্ধতির মূল ভিত্তি কোনটি?", "options": ["ক) ডেবিট ও ক্রেডিট সমতা", "খ) একক হিসাব", "গ) নগদ গ্রহণ", "ঘ) অবচয়"], "answer": "ক) ডেবিট ও ক্রেডিট সমতা"},
            {"q": "রেওয়ামিলের মূল উদ্দেশ্য কী?", "options": ["ক) হিসাবের গাণিতিক শুদ্ধতা যাচাই", "খ) মুনাফা নির্ণয়", "গ) কর নির্ধারণ", "ঘ) দেনা পরিশোধ"], "answer": "ক) হিসাবের গাণিতিক শুদ্ধতা যাচাই"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: জনাব রফিক ২০২৪ সালের ১ জানুয়ারি তারিখে ৫০,০০০ টাকা নগদ ও ২০,০০০ টাকার আসবাবপত্র নিয়ে ব্যবসা শুরু করলেন। উক্ত মাসে নগদ লেনদেন: পণ্য ক্রয় ১৫,০০০ টাকা, ধারে পণ্য বিক্রয় ২৫,০০০ টাকা।",
              "ka": "প্রারম্ভিক মূলধনের পরিমাণ কত?",
              "kha": "জানুয়ারি মাসের লেনদেনগুলোর জাবেদা দাখিলা দেখাও।",
              "ga": "লেনদেনগুলোর প্রভাব হিসাব সমীকরণে (A = L + E) ছকে দেখাও।",
              "gha": "\"সঠিক জাবেদা ভুক্তির ওপর নির্ভুল খতিয়ান প্রস্তুত নির্ভর করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: ব্যবসায়ী সাজিদের ব্যবসায়িক খতিয়ান উদ্বৃত্ত: নগদ তহবিল ১০,০০০ টাকা, ব্যাংক জমা ২০,০০০ টাকা, দেনাদার ১৫,০০০ টাকা, পাওনাদার ১০,০০০ টাকা এবং মূলধন ৩৫,০০০ টাকা।",
              "ka": "রেওয়ামিল কী?",
              "kha": "কোন কোন ভুল রেওয়ামিলে ধরা পড়ে না—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের উদ্বৃত্তগুলো নিয়ে একটি শুদ্ধ রেওয়ামিল প্রস্তুত কর।",
              "gha": "\"রেওয়ামিল প্রস্তুতের মাধ্যমে আর্থিক বিবরণী তৈরির পথ সুগম হয়।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "ব্যবসায় পরিচিতি": {
      "ব্যবসায় শিক্ষা শাখা": {
        "ব্যবসায়ের পরিবেশ ও উৎপত্তি": {
          "MCQ": [
            {"q": "ব্যবসায়ের প্রধান ও মূল উদ্দেশ্য কোনটি?", "options": ["ক) মুনাফা অর্জন", "খ) জনকল্যাণ", "গ) পণ্য উৎপাদন", "ঘ) সেবা প্রদান"], "answer": "ক) মুনাফা অর্জন"},
            {"q": "উৎপাদনের মূল উপাদান কয়টি?", "options": ["ক) ৪টি (ভূমি, শ্রম, মূলধন, সংগঠন)", "খ) ২টি", "গ) ৩টি", "ঘ) ৫টি"], "answer": "ক) ৪টি (ভূমি, শ্রম, মূলধন, সংগঠন)"},
            {"q": "শিল্পকে মূলত প্রধানত কয় ভাগে ভাগ করা যায়?", "options": ["ক) ৫ ভাগে", "খ) ৩ ভাগে", "গ) ৪ ভাগে", "ঘ) ৬ ভাগে"], "answer": "ক) ৫ ভাগে"},
            {"q": "বাণিজ্য (Commerce) বলতে কী বোঝায়?", "options": ["ক) পণ্য বণ্টন সংক্রান্ত কাজ", "খ) পণ্য উৎপাদন", "গ) কাঁচামাল সংগ্রহ", "ঘ) যন্ত্রপাতি স্থাপন"], "answer": "ক) পণ্য বণ্টন সংক্রান্ত কাজ"},
            {"q": "একমালিকানা ব্যবসায়ের বড় সুবিধা কোনটি?", "options": ["ক) দ্রুত সিদ্ধান্ত গ্রহণ ও একক মালিকানা", "খ) প্রচুর মূলধন", "গ) স্থায়ী স্থায়িত্ব", "ঘ) যৌথ ঝুঁকি"], "answer": "ক) দ্রুত সিদ্ধান্ত গ্রহণ ও একক মালিকানা"},
            {"q": "ব্যবসায়িক পরিবেশের প্রাকৃতিক উপাদান কোনটি?", "options": ["ক) জলবায়ু ও ভূ-প্রকৃতি", "খ) আইন-কানুন", "গ) শিক্ষা", "ঘ) প্রযুক্তি"], "answer": "ক) জলবায়ু ও ভূ-প্রকৃতি"},
            {"q": "পণ্যসামগ্রী দূর-দূরান্তে পৌঁছানোর বাধা দূর করে কোনটি?", "options": ["ক) পরিবহন", "খ) গুদামজাতকরণ", "গ) ব্যাংক", "ঘ) বীমা"], "answer": "ক) পরিবহন"},
            {"q": "ঝুঁকিজনিত বাধা দূর করে ব্যবসায়ের কোন অঙ্গ?", "options": ["ক) বীমা (Insurance)", "খ) পরিবহন", "গ) বিজ্ঞাপন", "ঘ) ব্যাংক"], "answer": "ক) বীমা (Insurance)"},
            {"q": "অংশীদারি ব্যবসায়ের মূল ভিত্তি কোনটি?", "options": ["ক) চুক্তি", "খ) নিবন্ধন", "গ) মূলধন", "ঘ) অংশীদার সংখ্যা"], "answer": "ক) চুক্তি"},
            {"q": "পাবলিক লিমিটেড কোম্পানির সর্বনিম্ন সদস্য সংখ্যা কত?", "options": ["ক) ৭ জন", "খ) ২ জন", "গ) ৫০ জন", "ঘ) ২০ জন"], "answer": "ক) ৭ জন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সজল সাহেব অল্প পুঁজি নিয়ে একা একটি মুদি দোকান চালু করেন। নিজের বুদ্ধিমত্তায় দ্রুত সিদ্ধান্ত নিয়ে তিনি ভালো মুনাফা করছেন। তবে ব্যবসার সমস্ত ঝুঁকি তাকে একাই বহন করতে হয়।",
              "ka": "একমালিকানা ব্যবসা কী?",
              "kha": "একমালিকানা ব্যবসা সবচেয়ে জনপ্রিয় কেন—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সজলের ব্যবসার বৈশিষ্ট্যগুলো পাঠ্যবইয়ের আলোকে আলোচনা কর।",
              "gha": "\"সীমাবদ্ধ ঝুঁকি ও অসীম দায় থাকা সত্ত্বেও একমালিকানা ব্যবসা যুগের পর যুগ টিকে আছে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: তিন বন্ধু মিলে লিখিত চুক্তির ভিত্তিতে একটি ফলের আড়ত শুরু করলেন। ব্যবসায় অর্জিত লাভ-ক্ষতি তারা চুক্তির শর্তানুযায়ী ভাগ করে নেন।",
              "ka": "অংশীদারি কারবার কী?",
              "kha": "\"চুক্তিই অংশীদারি কারবারের মূল ভিত্তি\"—বুঝিয়ে লেখ।",
              "ga": "উদ্দীপকে বর্ণিত ব্যবসার ধরণ ও পরিচালনার নিয়ম ব্যাখ্যা কর।",
              "gha": "\"নিবন্ধন না থাকলেও অংশীদারি ব্যবসা বৈধ, তবে নিবন্ধিত হওয়া অধিক নিরাপদ।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "ব্যবসায় উদ্যোগ": {
      "ব্যবসায় শিক্ষা শাখা": {
        "উদ্যোগ ও ব্যবসায়িক পরিকল্পনা": {
          "MCQ": [
            {"q": "যেকোনো কাজের ঝুঁকি নিয়ে পথ চলাই হলো-", "options": ["ক) উদ্যোগ", "খ) চাকরি", "গ) অলসতা", "ঘ) চুক্তি"], "answer": "ক) উদ্যোগ"},
            {"q": "ব্যবসায় উদ্যোগের প্রধান গুণ কোনটি?", "options": ["ক) উদ্ভাবনী শক্তি ও ঝুঁকি নেওয়ার সাহস", "খ) অধিক মূলধন থাকা", "গ) কায়িক শ্রম", "ঘ) উচ্চ শিক্ষা"], "answer": "ক) ব্যবসায় উদ্যোগের প্রধান গুণ কোনটি?"},
            {"q": "SWOT বিশ্লেষণের 'O' অক্ষরটি কী নির্দেশ করে?", "options": ["ক) Opportunity (সুযোগ)", "খ) Organization", "গ) Operation", "ঘ) Ownership"], "answer": "ক) Opportunity (সুযোগ)"},
            {"q": "ক্ষুদ্র ও কুটির শিল্পের বড় সুবিধা কী?", "options": ["ক) কম মূলধন ও স্থানীয় কর্মসংস্থান", "খ) ভারী যন্ত্রপাতি ব্যবহার", "গ) বিদেশি বিনিয়োগ", "ঘ) উচ্চ কর প্রদান"], "answer": "ক) কম মূলধন ও স্থানীয় কর্মসংস্থান"},
            {"q": "ব্যবসায়িক পরিকল্পনার প্রথম ধাপ কোনটি?", "options": ["ক) প্রকল্প নির্বাচন/ধারণা চিহ্নিতকরণ", "খ) ঋণ গ্রহণ", "গ) বিজ্ঞাপন প্রদান", "ঘ) কর্মী নিয়োগ"], "answer": "ক) প্রকল্প নির্বাচন/ধারণা চিহ্নিতকরণ"},
            {"q": "ব্যবসায়িক ঝুঁকি হ্রাসের উপায় কোনটি?", "options": ["ক) বীমা করা ও সঠিক পরিকল্পনা", "খ) বেশি দামে বিক্রি", "গ) বিজ্ঞাপন বন্ধ করা", "ঘ) ঋণ না নেওয়া"], "answer": "ক) বীমা করা ও সঠিক পরিকল্পনা"},
            {"q": "সফল উদ্যোক্তার অন্যতম মানদণ্ড কোনটি?", "options": ["ক) অধ্যবসায় ও সততা", "খ) দ্রুত কাজ বন্ধ করা", "গ) অন্যকে অনুকরণ করা", "ঘ) ঝুঁকি না নেওয়া"], "answer": "ক) অধ্যবসায় ও সততা"},
            {"q": "বিপণনের (Marketing) প্রথম কাজ কোনটি?", "options": ["ক) ক্রয়", "খ) বিক্রয়", "গ) গুদামজাতকরণ", "ঘ) পরিবহন"], "answer": "ক) ক্রয়"},
            {"q": "পণ্য ঢাকাকে কী বলা হয়?", "options": ["ক) মোড়কীকরণ (Packaging)", "খ) প্রমিতকরণ", "গ) পর্যায়িতকরণ", "ঘ) পরিবহন"], "answer": "ক) মোড়কীকরণ (Packaging)"},
            {"q": "বাংলাদেশ ক্ষুদ্র ও কুটির শিল্প সংস্থা কোনটি?", "options": ["ক) BSCIC", "খ) EPB", "গ) BGMEA", "ঘ) FBCCI"], "answer": "ক) BSCIC"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: তরুণ উদ্যোক্তা তৌহিদ তার এলাকায় প্লাস্টিক বর্জ্য প্রক্রিয়াজাত করে নতুন পণ্য তৈরির কারখানা গড়ে তোলেন। নতুন এই চিন্তায় প্রথম দিকে ঝুঁকি থাকলেও তিনি এখন একজন সফল ব্যবসায়ী।",
              "ka": "ব্যবসায় উদ্যোক্তা কে?",
              "kha": "ঝুঁকি গ্রহণ ও ব্যবসায়ের সাফল্যের সম্পর্ক ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের তৌহিদের উদ্যোগটি কীভাবে স্থানীয় অর্থনীতিতে প্রভাব ফেলছে—ব্যাখ্যা কর।",
              "gha": "\"উদ্ভাবনী ক্ষমতাই একজন সফল উদ্যোক্তাকে সাধারণ ব্যবসায়ী থেকে আলাদা করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: রিনা আক্তার একটি হস্তশিল্প প্রতিষ্ঠান চালুর পূর্বে বাজারের চাহিদা, মূলধনের উৎস ও কাঁচামালের সহজলভ্যতা যাচাই করে একটি নীল নকশা তৈরি করলেন।",
              "ka": "ব্যবসায়িক পরিকল্পনা কী?",
              "kha": "SWOT বিশ্লেষণ ব্যবসায়ের জন্য কেন গুরুত্বপূর্ণ?",
              "ga": "উদ্দীপকে রিনা আক্তারের গৃহীত পদক্ষেপটি কেন প্রয়োজনীয় তা আলোচনা কর।",
              "gha": "\"পূর্ব পরিকল্পনা ছাড়া ব্যবসায় সফল হওয়া অসম্ভব বললেই চলে।\"—উক্তিটির যথার্থতা মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "বাংলাদেশের ইতিহাস ও বিশ্বসভ্যতা": {
      "মানবিক শাখা": {
        "প্রাচীন সভ্যতা ও প্রাচীন বাংলা": {
          "MCQ": [
            {"q": "ইতিহাসের জনক কাকে বলা হয়?", "options": ["ক) হেরোডোটাস", "খ) থুসিডাইডিস", "গ) এরিস্টটল", "ঘ) ইবনে খালদুন"], "answer": "ক) হেরোডোটাস"},
            {"q": "সিন্ধু সভ্যতা কোন যুগের সভ্যতা?", "options": ["ক) তাম্র-ব্রোঞ্জ যুগ", "খ) লৌহ যুগ", "গ) প্রস্তর যুগ", "ঘ) আধুনিক যুগ"], "answer": "ক) তাম্র-ব্রোঞ্জ যুগ"},
            {"q": "প্রাচীন বাংলার প্রথম স্বাধীন নরপতি কে ছিলেন?", "options": ["ক) শশাঙ্ক", "খ) ধর্মপাল", "গ) গোপাল", "ঘ) লক্ষণ সেন"], "answer": "ক) শশাঙ্ক"},
            {"q": "পাল বংশের প্রতিষ্ঠাতা কে ছিলেন?", "options": ["ক) গোপাল", "খ) ধর্মপাল", "গ) দেবপাল", "ঘ) মহিপাল"], "answer": "ক) গোপাল"},
            {"q": "মৎস্যন্যায় বলতে কোন সময়কে বোঝায়?", "options": ["ক) শশাঙ্কের মৃত্যুর পর ১০০ বছরের অরাজকতা", "খ) পাল শাসনের স্বর্ণযুগ", "গ) সেন বংশের সময়", "ঘ) সুলতানি আমল"], "answer": "ক) শশাঙ্কের মৃত্যুর পর ১০০ বছরের অরাজকতা"},
            {"q": "মিশরীয় সভ্যতার সবচেয়ে বড় অবদান কোনটি?", "options": ["ক) পিরামিড ও হায়ারোগ্লিফিক লিপি", "খ) চাকা আবিষ্কার", "গ) পাত্র তৈরি", "ঘ) ভাস্কর্য"], "answer": "ক) পিরামিড ও হায়ারোগ্লিফিক লিপি"},
            {"q": "প্রাচীন জনপদ 'গৌড়' এর অন্তর্ভুক্ত অঞ্চল কোনটি ছিল?", "options": ["ক) চাঁপাইনবাবগঞ্জ ও মুর্শিদাবাদ", "খ) চট্টগ্রাম", "গ) বরিশাল", "ঘ) সিলেট"], "answer": "ক) চাঁপাইনবাবগঞ্জ ও মুর্শিদাবাদ"},
            {"q": "সেন বংশের শেষ রাজা কে ছিলেন?", "options": ["ক) লক্ষণ সেন", "খ) বিজয় সেন", "গ) বল্লাল সেন", "ঘ) সামন্ত সেন"], "answer": "ক) লক্ষণ সেন"},
            {"q": "কত সালে ইখতিয়ার উদ্দিন মুহম্মদ বিন বখতিয়ার খলজি বাংলা জয় করেন?", "options": ["ক) ১২০৪ সালে", "খ) ১২০৬ সালে", "গ) ১১৯৯ সালে", "ঘ) ১২৫৮ সালে"], "answer": "ক) ১২০৪ সালে"},
            {"q": "পাহাড়পুরের সোমপুর মহাবিহার কে নির্মাণ করেন?", "options": ["ক) ধর্মপাল", "খ) দেবপাল", "গ) গোপাল", "ঘ) শশাঙ্ক"], "answer": "ক) ধর্মপাল"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: শশাঙ্কের মৃত্যুর পর বাংলাজুড়ে তীব্র অরাজকতা ও বিশৃঙ্খলা দেখা দেয়, যেখানে শক্তিশালী ব্যক্তি দুর্বলদের ওপর অত্যাচার করত। দীর্ঘ ১০০ বছর পর জনগণ মিলে গোপালকে রাজা নির্বাচিত করে শান্তি ফিরিয়ে আনে।",
              "ka": "মৎস্যন্যায় কী?",
              "kha": "পাল রাজত্বকে বাংলার স্বর্ণযুগ বলা হয় কেন?",
              "ga": "উদ্দীপকে বর্ণিত ঐতিহাসিক প্রেক্ষাপট ও পাল বংশের শাসনের সূচনা আলোচনা কর।",
              "gha": "\"জনগণের সর্বসম্মত সিদ্ধান্তই বাংলায় প্রথম স্থায়ী ও দীর্ঘস্থায়ী বংশের ভিত্তি গড়ে দিয়েছিল।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: সিন্ধু নদের অববাহিকায় গড়ে ওঠা প্রাচীন নগর পরিকল্পনা ও নর্দমা ব্যবস্থা আধুনিক সভ্যতার প্রকৌশলীদের বিস্মিত করে।",
              "ka": "হায়ারোগ্লিফিক কী?",
              "kha": "সিন্ধু সভ্যতার নগর পরিকল্পনার দুটি বৈশিষ্ট্য লেখ।",
              "ga": "উদ্দীপকের আলোকে প্রাচীন সভ্যতার অবদান ও আধুনিক জীবনে তার প্রভাব ব্যাখ্যা কর।",
              "gha": "\"ইতিহাস পাঠ আমাদের অতীত ঐতিহ্যকে চিনে ভবিষ্যতের সিদ্ধান্ত নিতে সাহায্য করে।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "পৌরনীতি ও নাগরিকতা": {
      "মানবিক শাখা": {
        "রাষ্ট্র, সরকার ও নাগরিকতা": {
          "MCQ": [
            {"q": "পৌরনীতির ইংরেজি প্রতিশব্দ 'Civics' এসেছে কোন ভাষা থেকে?", "options": ["ক) ল্যাটিন ভাষা (Civis ও Civitas)", "খ) গ্রিক ভাষা", "গ) ফরাসি ভাষা", "ঘ) ইংরেজি"], "answer": "ক) ল্যাটিন ভাষা (Civis ও Civitas)"},
            {"q": "পরিবারের প্রধানতম সামাজিক কাজ কোনটি?", "options": ["ক) সন্তান লালন-পালন ও বংশ রক্ষা", "খ) বিনোদন", "গ) অর্থনৈতিক লেনদেন", "ঘ) আইন প্রণয়ন"], "answer": "ক) সন্তান লালন-পালন ও বংশ রক্ষা"},
            {"q": "নাগরিকতা অর্জনের প্রধান পদ্ধতি কয়টি?", "options": ["ক) ২টি (জন্মসূত্র ও অনুমোদন সূত্র)", "খ) ৩টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ২টি (জন্মসূত্র ও অনুমোদন সূত্র)"},
            {"q": "রাষ্ট্রের অপরিহার্য উপাদান কোনটি?", "options": ["ক) সার্বভৌমত্ব", "খ) লিখিত সংবিধান", "গ) রাজনৈতিক দল", "ঘ) বিচার বিভাগ"], "answer": "ক) সার্বভৌমত্ব"},
            {"q": "গণতন্ত্রের মূল ভিত্তি কোনটি?", "options": ["ক) জনগণের মতামত ও ভোটাধিকার", "খ) সামরিক শাসন", "গ) একনায়কত্ব", "ঘ) রাজতন্ত্র"], "answer": "ক) জনগণের মতামত ও ভোটাধিকার"},
            {"q": "আইনের প্রধান উৎস কোনটি?", "options": ["ক) প্রথা, ধর্ম ও আইনসভা", "খ) ব্যক্তির আদেশ", "গ) শক্তি প্রয়োগ", "ঘ) চুক্তি"], "answer": "ক) প্রথা, ধর্ম ও আইনসভা"},
            {"q": "মানবাধিকার সনদের সর্বজনীন ঘোষণা জাতিসংঘ কবে দেয়?", "options": ["ক) ১০ ডিসেম্বর ১৯৪৮", "খ) ২৪ অক্টোবর ১৯৪৫", "গ) ২৬ মার্চ ১৯৭১", "ঘ) ১ মে ১৮৮৬"], "answer": "ক) ১০ ডিসেম্বর ১৯৪৮"},
            {"q": "সরকারের অঙ্গ কয়টি?", "options": ["ক) ৩টি (আইন, শাসন ও বিচার বিভাগ)", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি (আইন, শাসন ও বিচার বিভাগ)"},
            {"q": "সুনাগরিকের প্রধান গুণ কয়টি?", "options": ["ক) ৩টি (বুদ্ধি, বিবেক, আত্মসংযম)", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি (বুদ্ধি, বিবেক, আত্মসংযম)"},
            {"q": "দ্বৈত নাগরিকতা বলতে কী বোঝায়?", "options": ["ক) একই সাথে দুই দেশের নাগরিক হওয়া", "খ) দুই রাজ্যের অধিবাসী হওয়া", "গ) জন্মসূত্রে নাগরিক হওয়া", "ঘ) অনুমোদন সূত্রে নাগরিক হওয়া"], "answer": "ক) একই সাথে দুই দেশের নাগরিক হওয়া"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: সায়ান একজন বিবেকবান নাগরিক। সে কর প্রদান করে, সৎ প্রার্থীকে ভোট দেয় এবং রাষ্ট্রের সম্পদ রক্ষা করে। সে মনে করে নাগরিকের অধিকার ও কর্তব্য একে অপরের পরিপূরক।",
              "ka": "সুনাগরিক কাকে বলে?",
              "kha": "অধিকার ও কর্তব্যের পারস্পরিক সম্পর্ক ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের সায়ানের আচরণের মাধ্যমে সুনাগরিকের কোন গুণাবলি প্রকাশ পেয়েছে—ব্যাখ্যা কর।",
              "gha": "\"সুনাগরিকই একটি দেশের টেকসই উন্নয়ন ও সুশাসনের মূল চালিকাশক্তি।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: ক দেশে আইনসভার মাধ্যমে গঠিত সরকার জনগণের নিকট দায়ী থাকে। দেশের সার্বভৌম ক্ষমতার মালিক জনগণ এবং সেখানে নিয়মিত নির্বাচন অনুষ্ঠিত হয়।",
              "ka": "সরকার কী?",
              "kha": "একনায়কতন্ত্র ও গণতন্ত্রের প্রধান পার্থক্য লেখ।",
              "ga": "উদ্দীপকের 'ক' দেশে কোন ধরণের সরকার ব্যবস্থা বিদ্যমান পাঠ্যবইয়ের আলোকে আলোচনা কর।",
              "gha": "\"জনগণের অংশগ্রহণ ও সুশাসন নিশ্চিত করাই আধুনিক গণতান্ত্রিক রাষ্ট্রের প্রধান লক্ষ্য।\"—মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "অর্থনীতি": {
      "মানবিক শাখা": {
        "মৌলিক অর্থনৈতিক ধারণা ও বাজার": {
          "MCQ": [
            {"q": "অর্থনীতির জনক কাকে বলা হয়?", "options": ["ক) অ্যাডাম স্মিথ", "খ) লর্ড রবিন্স", "গ) আলফ্রেড মার্শাল", "ঘ) কেয়ন্স"], "answer": "ক) অ্যাডাম স্মিথ"},
            {"q": "অর্থনৈতিক প্রধান সমস্যা দুটি কী কী?", "options": ["ক) দুষ্প্রাপ্যতা ও অসীম অভাব", "খ) মূলধন ও জমি", "গ) আয় ও ব্যয়", "ঘ) আমদানি ও রপ্তানি"], "answer": "ক) দুষ্প্রাপ্যতা ও অসীম অভাব"},
            {"q": "চাহিদার বিধি অনুযায়ী দাম বাড়লে চাহিদার পরিমাণ-", "options": ["ক) কমে", "খ) বাড়ে", "গ) স্থির থাকে", "ঘ) শূন্য হয়"], "answer": "ক) কমে"},
            {"q": "সম্পূর্ণ প্রতিযোগিতামূলক বাজারে পণ্যের দাম কীভাবে নির্ধারিত হয়?", "options": ["ক) চাহিদা ও যোগানের পারস্পরিক প্রভাবে", "খ) বিক্রেতার ইচ্ছায়", "গ) ক্রেতার ইচ্ছায়", "ঘ) সরকারের নির্দেশে"], "answer": "ক) চাহিদা ও যোগানের পারস্পরিক প্রভাবে"},
            {"q": "উৎপাদনের উপাদানের খাজনা কার বিনিময় মূল্য?", "options": ["ক) জমি", "খ) শ্রম", "গ) মূলধন", "ঘ) সংগঠন"], "answer": "ক) জমি"},
            {"q": "জাতীয় আয় পরিমাপের পদ্ধতি কয়টি?", "options": ["ক) ৩টি", "খ) ২টি", "গ) ৪টি", "ঘ) ৫টি"], "answer": "ক) ৩টি"},
            {"q": "যে বাজারে একজনমাত্র বিক্রেতা থাকে তাকে কী বলে?", "options": ["ক) একচেটিয়া বাজার (Monopoly)", "খ) প্রতিযোগিতা বাজার", "গ) পাইকারি বাজার", "ঘ) শেয়ার বাজার"], "answer": "ক) একচেটিয়া বাজার (Monopoly)"},
            {"q": "অর্থনীতির প্রধান অর্থনৈতিক ব্যবস্থা কয়টি?", "options": ["ক) ৪টি (ধনতান্ত্রিক, সমাজতান্ত্রিক, মিশ্র, ইসলামী)", "খ) ৩টি", "গ) ২টি", "ঘ) ৫টি"], "answer": "ক) ৪টি (ধনতান্ত্রিক, সমাজতান্ত্রিক, মিশ্র, ইসলামী)"},
            {"q": "যোগান বিধি অনুযায়ী দাম ও যোগানের সম্পর্ক কেমন?", "options": ["ক) সমমুখী/প্রত্যক্ষ", "খ) বিপরীতমুখী", "গ) কোনো সম্পর্ক নেই", "ঘ) স্থির"], "answer": "ক) সমমুখী/প্রত্যক্ষ"},
            {"q": "জিডিপি (GDP) এর পূর্ণরূপ কী?", "options": ["ক) Gross Domestic Product", "খ) Gross Development Product", "গ) General Domestic Price", "ঘ) Global Domestic Product"], "answer": "ক) Gross Domestic Product"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: বাজারে আলুর দাম প্রতি কেজি ২০ টাকা থেকে বেড়ে ৩০ টাকা হলে চাহিদার পরিমাণ ৫০ কেজি থেকে কমে ৩০ কেজি হয়। আবার দাম কমলে ক্রেতারা বেশি আলু কেনেন।",
              "ka": "চাহিদা কাকে বলে?",
              "kha": "দুষ্প্রাপ্যতা বলতে কী বোঝায়?",
              "ga": "উদ্দীপকের তথ্য থেকে একটি চাহিদা রেখা অঙ্কন কর।",
              "gha": "\"দাম ও চাহিদার বিপরীতমুখী সম্পর্কই চাহিদা বিধির ভিত্তি।\"—উদ্দীপকের তথ্যের সাহায্যে মূল্যায়ন কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি দেশে সরকারি ও বেসরকারি উভয় খাতের যৌথ অংশগ্রহণে শিল্পকারখানা পরিচালিত হয় এবং মুক্ত বাজার ব্যবস্থার সাথে সামাজিক কল্যাণ রক্ষা করা হয়।",
              "ka": "মিশ্র অর্থব্যবস্থা কী?",
              "kha": "সমাজতান্ত্রিক অর্থব্যবস্থার দুটি মূল বৈশিষ্ট্য লেখ।",
              "ga": "উদ্দীপকের দেশটি কোন ধরণের অর্থনৈতিক ব্যবস্থা অনুসরণ করছে তা ব্যাখ্যা কর।",
              "gha": "\"জনকল্যাণ ও অর্থনৈতিক প্রবৃদ্ধির সমতা রক্ষায় মিশ্র অর্থব্যবস্থা অত্যন্ত কার্যকর।\"—বিশ্লেষণ কর।"
            }
          ]
        }
      }
    },
    "ইসলাম ও নৈতিক শিক্ষা": {
      "ধর্মীয় বিষয়": {
        "ঈমান, শরিয়ত ও নৈতিকতা": {
          "MCQ": [
            {"q": "ইসলাম শব্দের অর্থ কী?", "options": ["ক) শান্তি ও আত্মসমর্পণ", "খ) ইবাদত", "গ) বিশ্বাস", "ঘ) অনুসরণ"], "answer": "ক) শান্তি ও আত্মসমর্পণ"},
            {"q": "শরিয়তের প্রধান উৎস কোনটি?", "options": ["ক) কুরআন মজিদ", "খ) হাদিস", "গ) ইজমা", "ঘ) কিয়াস"], "answer": "ক) কুরআন মজিদ"},
            {"q": "কুরআন মজিদের আয়াত সংখ্যা কত?", "options": ["ক) ৬৬৬৬টি (প্রচলিত মত)", "খ) ৬২৩৬টি", "গ) ৭০০০টি", "ঘ) ৫০০০টি"], "answer": "ক) ৬৬৬৬টি (প্রচলিত মত)"},
            {"q": "সাদাকাহ শব্দের অর্থ কী?", "options": ["ক) দান/দয়া", "খ) নামাজ", "গ) রোজা", "ঘ) ফরজ"], "answer": "ক) দান/দয়া"},
            {"q": "শরিয়তের দ্বিতীয় উৎস কোনটি?", "options": ["ক) সুন্নাহ/হাদিস", "খ) ইজমা", "গ) কিয়াস", "ঘ) ফিকহ"], "answer": "ক) সুন্নাহ/হাদিস"},
            {"q": "আখলাক-এ হামিদা কী?", "options": ["ক) প্রশংসনীয় চরিত্র", "খ) নিন্দনীয় চরিত্র", "গ) অহংকার", "ঘ) প্রতারণা"], "answer": "ক) প্রশংসনীয় চরিত্র"},
            {"q": "জাহিলিয়াত শব্দের অর্থ কী?", "options": ["ক) অজ্ঞতা ও বর্বরতা", "খ) জ্ঞান", "গ) সভ্যতা", "ঘ) শান্তি"], "answer": "ক) অজ্ঞতা ও বর্বরতা"},
            {"q": "হুদায়বিয়ার সন্ধি কত হিজরিতে স্বাক্ষরিত হয়?", "options": ["ক) ৬ হিজরিতে", "খ) ৮ হিজরিতে", "গ) ৫ হিজরিতে", "ঘ) ১০ হিজরিতে"], "answer": "ক) ৬ হিজরিতে"},
            {"q": "বিদায় হজের ভাষণ মহানবী (সা.) কোথায় প্রদান করেন?", "options": ["ক) আরাফাতের ময়দানে", "খ) মিনার ময়দানে", "গ) কাবা প্রাঙ্গণে", "ঘ) মদিনায়"], "answer": "ক) আরাফাতের ময়দানে"},
            {"q": "সর্বশ্রেষ্ঠ ও সর্বশেষ আসমানি কিতাব কোনটি?", "options": ["ক) আল-কুরআন", "খ) তাওরাত", "গ) ইঞ্জিল", "ঘ) যবুর"], "answer": "ক) আল-কুরআন"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: শাহেদ সামাজিক কাজে নিয়োজিত থাকে। সে প্রতিবেশীর বিপদে সাহায্য করে এবং মিথ্যা বলা থেকে বিরত থাকে। সে বিশ্বাস করে প্রতিটি কাজের হিসাব হাশরের মাঠে দিতে হবে।",
              "ka": "আখিরাত শব্দের অর্থ কী?",
              "kha": "শরিয়তের বিধিবিধান পালনে ঈমানের ভূমিকা ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের শাহেদের আচরণের মধ্যে ভালো চরিত্রের কোন দিকটি ফুটে উঠেছে—ব্যাখ্যা কর।",
              "gha": "\"পরকালে বিশ্বাসই মানুষকে ইহকালে সত্য ও নৈতিকতার পথে পরিচালিত করে।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: মহানবী (সা.) শত্রুর সাথে হুদায়বিয়ার অসম চুক্তিতে আবদ্ধ হয়ে শান্তিকামী মানসিকতার পরিচয় দিয়েছিলেন, যা পরবর্তীতে মক্কা বিজয়ের পথ সহজ করেছিল।",
              "ka": "হুদায়বিয়ার সন্ধি কত বছরের জন্য হয়েছিল?",
              "kha": "রাসূল (সা.) এর দূরদর্শিতা কীভাবে শান্তি এনেছিল—ব্যাখ্যা কর।",
              "ga": "উদ্দীপকের ঐতিহাসিক ঘটনাটির গুরুত্ব শরিয়ত ও জীবনের আলোকে বর্ণনা কর।",
              "gha": "\"মহানবী (সা.) এর শান্তি ও ক্ষমার আদর্শই আধুনিক বিশ্বের সংঘাত নিরসনের উপায়।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    },
    "তথ্য ও যোগাযোগ প্রযুক্তি": {
      "আবশ্যিক বিষয়": {
        "ডিজিটাল বাংলাদেশ ও সিকিউরিটি": {
          "MCQ": [
            {"q": "তথ্যপ্রযুক্তির প্রসারে 'ডিজিটাল বাংলাদেশ' রূপকল্প কত সালে ঘোষিত হয়?", "options": ["ক) ২০০৮ সালে", "খ) ২০১০ সালে", "গ) ২০১৫ সালে", "ঘ) ২০২০ সালে"], "answer": "ক) ২০০৮ সালে"},
            {"q": "ই-পাসপোর্ট ও ই-গভর্ন্যান্সের প্রধান লক্ষ্য কী?", "options": ["ক) সেবা সহজ ও দ্রুত করা", "খ) খরচ বাড়ানো", "গ) কাগজের ব্যবহার বাড়ানো", "ঘ) সময় নষ্ট করা"], "answer": "ক) সেবা সহজ ও দ্রুত করা"},
            {"q": "কম্পিউটার ভাইরাস হলো এক ধরণের-", "options": ["ক) ক্ষতিকারক প্রোগ্রাম", "খ) হার্ডওয়্যার ডিভাইস", "গ) নেটওয়ার্ক ক্যাবল", "ঘ) মেমোরি চিপ"], "answer": "ক) ক্ষতিকারক প্রোগ্রাম"},
            {"q": "দ্বিমাত্রিক বারকোডকে সংক্ষেপে কী বলে?", "options": ["ক) QR Code", "খ) PIN", "গ) URL", "ঘ) IP"], "answer": "ক) QR Code"},
            {"q": "কম্পিউটার সুরক্ষার জন্য কোনটি ব্যবহার করা উচিত?", "options": ["ক) অ্যান্টিভাইরাস সফটওয়্যার", "খ) কেবল গেম", "গ) নতুন মনিটর", "ঘ) উচ্চগতির প্রসেসর"], "answer": "ক) অ্যান্টিভাইরাস সফটওয়্যার"},
            {"q": "সামাজিক যোগাযোগ মাধ্যম কোনটি?", "options": ["ক) Facebook", "খ) Google Search", "গ) MS Word", "ঘ) Photoshop"], "answer": "ক) Facebook"},
            {"q": "ইন্টারনেটে নিরাপদে থাকার প্রধান নিয়ম কোনটি?", "options": ["ক) শক্তিশালী পাসওয়ার্ড ব্যবহার ও তথ্য গোপন রাখা", "খ) সবাইকে পাসওয়ার্ড দেওয়া", "গ) অচেনা লিংকে ক্লিক করা", "ঘ) অ্যান্টিভাইরাস বন্ধ রাখা"], "answer": "ক) শক্তিশালী পাসওয়ার্ড ব্যবহার ও তথ্য গোপন রাখা"},
            {"q": "ডিজিটাল কনটেন্ট কত প্রকার হতে পারে?", "options": ["ক) ৪ প্রকার (টেক্সট, ছবি, অডিও, ভিডিও)", "খ) ২ প্রকার", "গ) ৩ প্রকার", "ঘ) ৫ প্রকার"], "answer": "ক) ৪ প্রকার (টেক্সট, ছবি, অডিও, ভিডিও)"},
            {"q": "ই-বই বা ইলেকট্রনিক বুক পড়ার জনপ্রিয় ডিভাইস কোনটি?", "options": ["ক) ই-বুক রিডার (Kindle)", "খ) প্রিন্টার", "গ) প্রজেক্টর", "ঘ) স্ক্যানার"], "answer": "ক) ই-বুক রিডার (Kindle)"},
            {"q": "অন্যের কম্পিউটার বা সিস্টেমে অবৈধ প্রবেশকে কী বলে?", "options": ["ক) হ্যাকিং (Hacking)", "খ) ব্রাউজিং", "গ) আপলোডিং", "ঘ) প্রোগ্রামিং"], "answer": "ক) হ্যাকিং (Hacking)"}
          ],
          "CQ": [
            {
              "stem": "উদ্দীপক ১: তানভীর অনলাইন ব্যাংকিং ব্যবহার করে ঘরে বসেই স্কুলের ফি প্রদান করল। তবে একদিন সে একটি ভুয়া ই-মেইল লিংকে ক্লিক করে তার অ্যাকাউন্টের পাসওয়ার্ড হারিয়ে বিপদে পড়ল।",
              "ka": "সাইবার অপরাধ কী?",
              "kha": "পাসওয়ার্ড শক্তিশালী করার দুটি উপায় লেখ।",
              "ga": "উদ্দীপকে তানভীরের অসাবধানতার কারণ আইসিটি নিরাপত্তার আলোকে ব্যাখ্যা কর।",
              "gha": "\"সচেতনতা ও সতর্কতা অবলম্বনই সাইবার জগতে নিরাপদ থাকার প্রধান চাবিকাঠি।\"—বিশ্লেষণ কর।"
            },
            {
              "stem": "উদ্দীপক ২: একটি সরকারি দপ্তরে ফাইলপত্র আগে ম্যানুয়ালি সংরক্ষণ করা হতো। বর্তমানে ই-সেবা ও ই-ফাইলিং চালুর ফলে নাগরিকেরা দ্রুত সেবা পাচ্ছেন।",
              "ka": "ই-গভর্ন্যান্স কী?",
              "kha": "ই-সেবার মাধ্যমে নাগরিক সুবিধা বৃদ্ধির দুটি উদাহরণ দাও।",
              "ga": "উদ্দীপকের সেবা ব্যবস্থার পরিবর্তনের ফলে কীভাবে সময় ও অর্থ সাশ্রয় হচ্ছে তা আলোচনা কর।",
              "gha": "\"ডিজিটাল প্রযুক্তি প্রবর্তন একটি স্বচ্ছ ও দুর্নীতিমুক্ত সমাজ গঠনে সহায়ক।\"—উক্তিটি মূল্যায়ন কর।"
            }
          ]
        }
      }
    }
  }
}

    

    # --- ৪টি নির্ধারিত ট্যাব ---
    tab1, tab2, tab3, tab4 = st.tabs([
        "প্র্যাকটিস ও রিভিশন",
        "কুইজ টেস্ট",
        "মডেল টেস্ট",
        "পরীক্ষার ইতিহাস ও রিটেস্ট"
    ])

    # ================= TAB 1: প্র্যাকটিস ও রিভিশন =================
    with tab1:
        st.markdown("### প্র্যাকটিস ও রিভিশন হাব (শ্রেণি, বিষয় ও অধ্যায়ভিত্তিক)")
        st.info("এখানে আপনি পছন্দমতো শ্রেণি, বিষয়, বিভাগ ও অধ্যায় বেছে নিয়ে প্র্যাকটিস করতে পারবেন।")
        
        c1, c2 = st.columns(2, gap="medium")
        with c1:
            p_class = st.selectbox("শ্রেণি বেছে নিন:", list(library_question_bank.keys()), key="p_class_sel")
        with c2:
            p_books = list(library_question_bank[p_class].keys())
            p_book = st.selectbox("বিষয় বেছে নিন:", p_books, key="p_book_sel")

        c3, c4 = st.columns(2, gap="medium")
        with c3:
            p_sections = list(library_question_bank[p_class][p_book].keys())
            p_sec = st.selectbox("বিভাগ বেছে নিন:", p_sections, key="p_sec_sel")
        with c4:
            p_chapters = list(library_question_bank[p_class][p_book][p_sec].keys())
            p_chap = st.selectbox("অধ্যায়/বিষয়বস্তু বেছে নিন:", p_chapters, key="p_chap_sel")

        st.markdown("<hr style='margin: 18px 0; border: 0; border-top: 2px dashed #cbd5e1;'>", unsafe_allow_html=True)
        
        chap_data = library_question_bank[p_class][p_book][p_sec][p_chap]
        
        # প্র্যাকটিসের ধরন নির্বাচন (৩টি অপশন: MCQ, CQ, Written Part)
        practice_type = st.radio(
            "প্র্যাকটিসের ধরন নির্বাচন করুন:",
            ["বহুনির্বাচনি (MCQ) প্র্যাকটিস", "সৃজনশীল প্রশ্ন (CQ) প্র্যাকটিস", "রিটেন পার্ট প্র্যাকটিস"],
            horizontal=True
        )

        if "MCQ" in practice_type:
            st.markdown("#### বহুনির্বাচনি (MCQ) প্র্যাকটিস জেনারেটর")
            mcq_list = chap_data.get("MCQ", [])
            total_available = len(mcq_list)
            
            if total_available > 1:
                q_count_choice = st.slider(
                    "কতটি প্রশ্ন প্র্যাকটিস করতে চান?",
                    1,
                    max_value=total_available,
                    value=min(5, total_available),
                    step=1
                )
            elif total_available == 1:
                q_count_choice = 1
                st.info("এই অধ্যায়ে ১টি প্রশ্ন উপলব্ধ রয়েছে।")
            else:
                q_count_choice = 0

            if total_available > 0:
                if st.button("প্র্যাকটিস শুরু করুন", key="start_mcq_prac"):
                    st.session_state["active_prac_mcqs"] = random.sample(mcq_list, q_count_choice)
                    st.session_state["prac_submitted"] = False

                if "active_prac_mcqs" in st.session_state:
                    st.markdown("---")
                    for idx, item in enumerate(st.session_state["active_prac_mcqs"], 1):
                        st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                        st.radio(f"উত্তর দিন (প্রশ্ন {idx}):", item["options"], key=f"prac_q_{idx}", index=None)
                        st.markdown("")

                    if st.button("উত্তরগুলো যাচাই করুন", key="check_prac_ans"):
                        st.session_state["prac_submitted"] = True
                        st.rerun()

                    if st.session_state.get("prac_submitted", False):
                        st.markdown("### প্র্যাকটিস ফলাফল ও সঠিক উত্তর:")
                        for idx, item in enumerate(st.session_state["active_prac_mcqs"], 1):
                            u_ans = st.session_state.get(f"prac_q_{idx}")
                            c_ans = item["answer"]
                            if u_ans == c_ans:
                                st.success(f"প্রশ্ন {idx}: সঠিক! (আপনার উত্তর: {u_ans})")
                            else:
                                st.error(f"প্রশ্ন {idx}: ভুল। (আপনার উত্তর: {u_ans if u_ans else 'উত্তর দেননি'}) | সঠিক উত্তর: **{c_ans}**")
            else:
                st.warning("এই অধ্যায়ে প্র্যাকটিসের জন্য কোনো MCQ প্রশ্ন পাওয়া যায়নি।")
                
        elif "সৃজনশীল" in practice_type:
            st.markdown("#### সৃজনশীল প্রশ্ন (CQ) প্র্যাকটিস ও সমাধান গাইডলাইন")
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
            else:
                st.info("এই অধ্যায়ে কোনো সৃজনশীল প্রশ্ন যুক্ত করা হয়নি।")
                
        else:
            st.markdown("#### রিটেন বা নির্মিত অংশ প্র্যাকটিস (ব্যাকরণ, লেটার, অ্যাপ্লিকেশন, ভাবসম্প্রসারণ ইত্যাদি)")
            written_list = chap_data.get("Written", [])
            if written_list:
                for idx, w_item in enumerate(written_list, 1):
                    st.markdown(
                        f"""
                        <div class="written-card">
                            <h4 style="color: #0284c7; margin-top: 0;">লিখন ও নির্মিতি টপিক - {idx}: {w_item['title']}</h4>
                            <p style="font-size: 15px; font-weight: 500; color: #334155;"><b>নির্দেশনা/বিষয়:</b> {w_item['desc']}</p>
                        </div>
                        """,
                        unsafe_allow_html=True
                    )
                st.info("💡 টিপস: এই বিষয়গুলোর ওপর নিজ খাতায় উত্তর লিখে বা প্র্যাকটিস করে আপনি আপনার দক্ষতা যাচাই করতে পারেন।")
            else:
                st.info("এই বিষয়ের জন্য নির্দিষ্ট কোনো রিটেন টপিক যুক্ত করা হয়নি।")

    # ================= TAB 2: কুইজ টেস্ট =================
    with tab2:
        st.markdown("### কুইজ টেস্ট (ছোট পরীক্ষা ও স্কোরিং)")
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
                st.markdown("### কুইজ ফলাফল ও সঠিক উত্তরসমূহ:")
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

                if st.button("কুইজ জমা দিন ও হিস্ট্রি সেভ করুন", key="submit_quiz_btn", use_container_width=True):
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
                            (id INTEGER PRIMARY KEY AUTOINCREMENT, quiz_title TEXT, quiz_date TEXT, 
                            total_marks INT, obtained_score INT, correct_count INT, wrong_count INT, details_json TEXT)"""
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

    # ================= TAB 3: মডেল টেস্ট =================
    with tab3:
        st.markdown("### পূর্ণাঙ্গ মডেল টেস্ট (লাইভ কাউন্টডাউন ও অটো-সাবমিট)")
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
                    user_choice = st.radio(f"উত্তর সিলেক্ট করুন ({idx + 1}):", options, key=ans_key, index=None)
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

    # ================= TAB 4: পরীক্ষার ইতিহাস =================
    with tab4:
        st.markdown("### সংরক্ষিত পরীক্ষার ইতিহাস (মডেল টেস্ট ও কুইজ টেস্ট)")
        history_tab_choice = st.radio("কোন হিস্ট্রি দেখতে চান?", ["মডেল টেস্ট ইতিহাস", "কুইজ টেস্ট ইতিহাস"], horizontal=True)

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
                            if st.button("হিস্ট্রি ডিলেট করুন", key=f"del_history_btn_{t_id}", use_container_width=True):
                                con_del = sqlite3.connect("smart_school_syllabus.db")
                                cur_del = con_del.cursor()
                                cur_del.execute("DELETE FROM model_test_history WHERE id=?", (t_id,))
                                con_del.commit()
                                con_del.close()
                                st.success("মডেল টেস্ট হিস্ট্রি মুছে ফেলা হয়েছে!")
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

                        if st.button("কুইজ হিস্ট্রি ডিলেট করুন", key=f"del_quiz_btn_{q_id}", use_container_width=True):
                            con_del = sqlite3.connect("smart_school_syllabus.db")
                            cur_del = con_del.cursor()
                            cur_del.execute("DELETE FROM quiz_test_history WHERE id=?", (q_id,))
                            con_del.commit()
                            con_del.close()
                            st.success("কুইজ টেস্ট হিস্ট্রি সফলভাবে মুছে ফেলা হয়েছে!")
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
            <h2 style='color: #38bdf8; font-family: Outfit; font-weight: 800; margin-bottom: 5px; font-size: 26px;'>✨ স্মার্ট নোটস ও ওয়ার্ড এডিটর হাব</h2>
            <p style='color: #94a3b8; font-size: 14px; font-family: Hind Siliguri;'>এখানে তুমি টেক্সট ফরম্যাটিং, টেবিল তৈরি, বিষয়ভিত্তিক নোট সংরক্ষণ, সার্চসহ তোমার নোটটি বিভিন্ন ফরম্যাটে ডাউনলোড করতে পারবে।</p>
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
      placeholder: 'এখানে তোমার চমৎকার নোট লিখো...'
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
        st.markdown("<h3 style='color: #0284c7; font-size: 20px; font-weight: 700;'>📝 নোট এডিটর ওয়ার্কস্পেস</h3>", unsafe_allow_html=True)
        
        c_top1, c_top2, c_top3 = st.columns([2, 1, 1], gap="small")
        with c_top1:
            note_title = st.text_input(
                "নোটের শিরোনাম দাও (Title):",
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
            ["ড্রাফট হিসেবে সেভ করো", "ফাইনাল নোট হিসেবে সেভ করো"],
            horizontal=True,
            key="note_save_type_radio"
        )

        # অত্যন্ত সুন্দর ও স্পষ্ট হাইলাইটেড ইন্সট্রাকশন বক্স
        st.markdown("""
        <div style="background-color: #e0f2fe; border: 1.5px solid #bae6fd; border-left: 6px solid #0284c7; padding: 14px 18px; border-radius: 12px; margin-top: 15px; margin-bottom: 10px; box-shadow: 0 4px 12px rgba(14,165,233,0.06);">
        <p style="color: #0369a1; font-size: 14px; font-weight: 600; margin: 0; line-height: 1.5;">
        💡 নোট হিসেবে সেভ করতে এডিটরে লেখা এখানে পেস্ট করো অথবা তোমার ইচ্ছেমতো লেখা এখানে লিখে সেভ করো:
        </p>
        </div>
        """, unsafe_allow_html=True)

        # ব্যাকআপ টেক্সট বক্স
        backup_body = st.text_area(
            "ব্যাকআপ টেক্সট এডিটর:",
            value=st.session_state["current_content"],
            height=120,
            placeholder="এখানে তোমার লেখার মূল কন্টেন্ট লিখো বা পেস্ট করো...",
            key=f"backup_editor_textarea_{st.session_state['clear_trigger']}"
        )
        typed_body = backup_body

        # সেভ বা আপডেট বাটন
        btn_label = "🔄 নোট আপডেট করো" if st.session_state["edit_id"] is not None else "💾 নোট সেভ করো"
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
            st.markdown("<h3 style='color: #0284c7; font-size: 18px;'>📚 তোমার সেভ করা নোটসমূহ</h3>", unsafe_allow_html=True)
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
        st.markdown("<h3 style='color: #0f172a; font-size: 18px;'>📚 আমার পাঠাগার</h3>", unsafe_allow_html=True)
        st.info("তোমার পছন্দের বই পড়তে বাঁ পাশের মূল মেনু থেকে 'আমার পাঠাগার' অপশনে ক্লিক করো।")
        st.markdown("---")
        st.markdown("<h3 style='color: #0f172a; font-size: 18px;'>🤖 এআই অ্যাসিস্ট্যান্ট</h3>", unsafe_allow_html=True)
        st.markdown("কোনo প্রশ্ন থাকলে নিচে Gemini-কে জিজ্ঞেস করো এবং তোমার নোট খাতায় নোট করে রাখো।")
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
import random
import time
import json
import sqlite3
from datetime import datetime, timedelta
import math
import pandas as pd
from collections import Counter

def render_study_planner():
    st.markdown("""
    <div style="background: linear-gradient(135deg, #1E3A8A 0%, #3882F6 100%);
    padding: 25px; border-radius: 12px; color: white; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
        <h2 style="margin: 0; color: white;">Smart Study & Syllabus</h2>
        <p style="margin: 5px 0 0 0; font-size: 16px; opacity: 0.9;">নিচের তোমার মতো করে তোমার দেওয়া স্কুলের সিলেবাসের আলোকে সিলেবাস ও পড়ার রুটিন তৈরি করো। তারপর তোমার স্মার্ট স্টাডি প্ল্যান বানাও!</p>
    </div>
    """, unsafe_allow_html=True)

    tab1, tab2, tab3 = st.tabs(["📚 Syllabus", "⏰ Routine", "📅 Study Plan"])

    # --- ক্লাস সেভেনের সমন্বিত সিলেবাস ডিকশনারি (Original Structure) ---
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
                "প্রথম অধ্যায়: নিম্নশ্রেণির জীব", "দ্বিতীয় অধ্যায়: উদ্ভিদ ও প্রাণীর কোষীয় সংগঠন",
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

    # --- TAB 1: SYLLABUS ---
    with tab1:
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
        try:
            cursor.execute("ALTER TABLE school_syllabus_multi ADD COLUMN is_default INT DEFAULT 0")
        except:
            pass
        conn.commit()
        
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
            
            form_title = "Edit" if editing_syllabus else "+ নতুন সিলেবাস তৈরি করুন"
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
                student_class = st.selectbox("শ্রেণি নির্বাচন করো:", ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"], index=1, key="sel_class_multi")
                exam_terms_list = ["Weekly Quiz", "Half-Yearly", "Yearly", "Model Test", "Pre-Test", "C.T.", "Tutorial", "Others"]
                term_idx = exam_terms_list.index(default_term) if default_term in exam_terms_list else 0
                exam_term = st.selectbox("পরীক্ষার নাম:", exam_terms_list, index=term_idx, key="sel_term_multi")
            with col2:
                available_subjects = list(unified_syllabus_dictionary.keys())
                default_subjs = [sub for sub in parsed_saved_data.keys() if sub in available_subjects] if parsed_saved_data else []
                selected_subjects = st.multiselect("এক/একাধিক বিষয় নির্বাচন করো:", available_subjects, default=default_subjs, key="sel_subjects_multi")

            selected_syllabus_data = {}
            if selected_subjects:
                st.markdown("#### নির্বাচিত বিষয়/বিষয়সমূহের টপিক বাছাই করো: ")
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
            
            if editing_syllabus or st.session_state.create_new_syllabus:
                with col_btn_s2:
                    if st.button("❌ Cancel", use_container_width=True):
                        st.session_state.create_new_syllabus = False
                        st.session_state.syllabus_edit_id = None
                        st.rerun()

        if all_saved_syllabi and not st.session_state.create_new_syllabus and st.session_state.syllabus_edit_id is None:
            st.markdown("### 🗂️ সংরক্ষিত সিলেবাসসমূহ ও মূল সিলেবাস নির্বাচন")
            
            syllabus_options = {}
            for s in all_saved_syllabi:
                s_id, s_class, s_term, s_json, s_def = s[0], s[1], s[2], s[3], s[4]
                def_tag = " ⭐ [মূল/ডিফল্ট সিলেবাস]" if s_def == 1 else ""
                label_str = f"📌 {s_term} ({s_class}) - ID: {s_id}{def_tag}"
                syllabus_options[label_str] = s

            selected_s_label = st.selectbox("আপনার সংরক্ষিত সিলেবাসসমূহ থেকে বেছে নিন:", list(syllabus_options.keys()), key="select_saved_syllabus_dropdown")
            selected_s_record = syllabus_options[selected_s_label]
            s_id, s_class, s_term, s_json, s_def = selected_s_record

            st.markdown("---")
            col_sel_title, col_sel_action = st.columns([3, 2])
            with col_sel_title:
                st.markdown(f"**বর্তমান সিলেবাস:** `{s_term} ({s_class})`")
                if s_def == 1:
                    st.success("✨ এটি বর্তমানে আপনার **'মূল সিলেবাস'** হিসেবে সেট করা আছে।")
                else:
                    st.info("ℹ️ এটি সাধারণ সংরক্ষিত সিলেবাস। একে মূল সিলেবাস হিসেবে সেট করতে পারেন।")
            with col_sel_action:
                is_currently_default = (s_def == 1)
                make_default_btn = st.checkbox("⭐ এটিকে 'মূল সিলেবাস' হিসেবে টিক দিন", value=is_currently_default, key=f"main_default_check_{s_id}")
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
            st.markdown(f"#### Now, Your Syllabus is here ({s_class} - {s_term})")
            
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

            col_btn1, col_btn2, col_btn3 = st.columns(3)
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
                    st.success("সিলেবাস সফলভাবে বাতিল ও মুছে ফেলা হয়েছে! পুনরায় সিলেবাস তৈরি করো।")
                    st.rerun()
            with col_btn3:
                if st.button("➕ Create New Syllabus", use_container_width=True, type="primary", key="create_new_s_btn"):
                    st.session_state.create_new_syllabus = True
                    st.session_state.syllabus_edit_id = None
                    st.rerun()

    # --- TAB 2: ROUTINE ---
    with tab2:
        st.markdown("### ⏰ সাপ্তাহিক কাস্টম রুটিন প্রস্তুতকারক (Custom Study Routine Builder)")
        st.markdown("প্রথমে তোমার পছন্দের সিলেবাস নির্বাচন করো। এরপর প্রতিদিনের জন্য বিষয়, সময় স্লট, অধ্যায় এবং টার্গেট সেট করে রুটিন সাজাও!")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, student_class, exam_term, syllabus_json, is_default FROM school_syllabus_multi ORDER BY id DESC")
        routine_syllabi = cursor.fetchall()
        conn.close()

        if routine_syllabi:
            syllabus_choices = {}
            default_index = 0
            for idx, s in enumerate(routine_syllabi):
                s_id, s_class, s_term, s_json, s_def = s[0], s[1], s[2], s[3], s[4]
                label_str = f"📌 {s_term} ({s_class}) [ID: {s_id}]"
                if s_def == 1:
                    label_str += " ⭐ (মূল সিলেবাস)"
                    default_index = idx
                syllabus_choices[label_str] = s_json

            selected_routine_s_label = st.selectbox("🎯 সিলেবাস নির্বাচন করো:", list(syllabus_choices.keys()), index=default_index, key="routine_syllabus_dropdown")
            
            # শুধুমাত্র নির্বাচিত সিলেবাসের ভেতরে থাকা বিষয়গুলো রুটিনের জন্য ফিল্টার করা হলো
            routine_selected_syl_dict = json.loads(syllabus_choices[selected_routine_s_label])
            available_subjects_in_syllabus = list(routine_selected_syl_dict.keys())

            if available_subjects_in_syllabus:
                st.markdown("---")
                st.markdown("#### 📅 সপ্তাহের ৭ দিনের পড়ার পরিকল্পনা ও টার্গেট সেটিং (শনিবার - শুক্রবার):")
                
                days_of_week = ["শনিবার", "রবিবার", "সোমবার", "মঙ্গলবার", "বুধবার", "বৃহস্পতিবার", "শুক্রবার"]
                weekly_custom_routine = {}

                for day in days_of_week:
                    with st.expander(f"📆 দিন: {day}", expanded=(day == "শনিবার")):
                        num_subjs_for_day = st.number_input(
                            f"{day} কয়টি বিষয় পড়তে চাও? (যত খুশি দিন)",
                            min_value=1, max_value=25, value=min(2, len(available_subjects_in_syllabus)), step=1,
                            key=f"num_subs_{day}"
                        )
                        
                        day_entries = []
                        for i in range(int(num_subjs_for_day)):
                            st.markdown(f"**🔹 বিষয় {i+1} বিস্তারিত ও টার্গেট সেটিং:**")
                            c_s, c_t, c_c = st.columns([2, 2, 2])
                            with c_s:
                                chosen_sub = st.selectbox(
                                    f"বিষয় নির্বাচন:",
                                    available_subjects_in_syllabus,
                                    key=f"sub_{day}_{i}"
                                )
                            with c_t:
                                time_str = st.text_input(
                                    f"সময় বা স্লট:",
                                    value="06:30 AM - 07:30 AM (সকাল)",
                                    key=f"time_{day}_{i}"
                                )
                            with c_c:
                                chapter_str = st.text_input(
                                    f"টপিক বা অধ্যায়:",
                                    value="অধ্যায় ১.১",
                                    key=f"chap_{day}_{i}"
                                )
                            
                            st.caption("🎯 অধ্যায়ের মোট পরিমাণ ও কন্টেন্ট টার্গেট (ঐচ্ছিক):")
                            col_p1, col_p2, col_p3 = st.columns(3)
                            with col_p1:
                                total_pages = st.number_input(f"মোট পৃষ্ঠা:", min_value=0, max_value=500, value=10, key=f"pages_{day}_{i}")
                            with col_p2:
                                total_mcq = st.number_input(f"মোট MCQ:", min_value=0, max_value=1000, value=30, key=f"mcq_{day}_{i}")
                            with col_p3:
                                total_cq = st.number_input(f"মোট CQ (সৃজনশীল):", min_value=0, max_value=100, value=5, key=f"cq_{day}_{i}")

                            st.caption("✅ এই অধ্যায়ের কি কি সম্পন্ন করা বা লক্ষ্য রয়েছে তা টিক দিন:")
                            col_ck1, col_ck2, col_ck3, col_ck4 = st.columns(4)
                            with col_ck1:
                                is_mainbook = st.checkbox("📖 মূল বই রিডিং", value=True, key=f"mb_{day}_{i}")
                            with col_ck2:
                                is_guide = st.checkbox("📘 গাইড/নোট", value=False, key=f"gb_{day}_{i}")
                            with col_ck3:
                                is_mcq_done = st.checkbox("❓ MCQ প্র্যাকটিস", value=True, key=f"md_{day}_{i}")
                            with col_ck4:
                                is_cq_done = st.checkbox("✍️ CQ চর্চা", value=False, key=f"cd_{day}_{i}")

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
                if st.button("🚀 আপনার রুটিন তৈরি করুন (Generate Your Routine)", use_container_width=True, type="primary"):
                    st.session_state["generated_custom_weekly_routine"] = weekly_custom_routine
                    st.session_state["generated_routine_syllabus_name"] = selected_routine_s_label
                    st.success("সফলভাবে তোমার কাস্টম প্রাত্যহিক রুটিন তৈরি করা হয়েছে!")

                if "generated_custom_weekly_routine" in st.session_state:
                    st.markdown("<br>", unsafe_allow_html=True)
                    st.markdown("### 🏆 আপনার চূড়ান্ত প্রাত্যহিক রুটিন ছক")
                    
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
                                    st.markdown(f"**✏️ এডিট করো ({day_name} - বিষয় {idx+1}):**")
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
                                        new_mb = st.checkbox("📖 মূল বই", value=entry.get("mainbook", True))
                                    with eb2:
                                        new_gb = st.checkbox("📘 গাইড/নোট", value=entry.get("guide", False))
                                    with eb3:
                                        new_md = st.checkbox("❓ MCQ", value=entry.get("mcq_done", True))
                                    with eb4:
                                        new_cd = st.checkbox("✍️ CQ", value=entry.get("cq_done", False))

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
                                    st.markdown(f"<div style='padding-top:8px; font-weight:bold; color:#1e293b;'>📅 {day_name}</div>", unsafe_allow_html=True)
                                with col_t:
                                    st.markdown(f"<div style='padding-top:8px; color:#1e3a8a; font-weight:bold;'>⏰ {entry['time']}</div>", unsafe_allow_html=True)
                                with col_s:
                                    st.markdown(f"<div style='padding-top:5px;'><span style='background-color:#DBEAFE; color:#1D4ED8; padding:5px 12px; border-radius:6px; font-weight:bold; font-size:13px; display:inline-block;'>📘 {entry['subject']}</span></div>", unsafe_allow_html=True)
                                with col_c:
                                    target_info = []
                                    if entry.get("pages", 0) > 0: target_info.append(f"📖 {entry['pages']}টি পৃষ্ঠা")
                                    if entry.get("mcq", 0) > 0: target_info.append(f"❓ {entry['mcq']}টি MCQ")
                                    if entry.get("cq", 0) > 0: target_info.append(f"✍️ {entry['cq']}টি CQ")
                                    target_str = " | ".join(target_info) if target_info else "সাধারন রিভিশন"

                                    done_tags = []
                                    if entry.get("mainbook"): done_tags.append("✅মূলবই")
                                    if entry.get("guide"): done_tags.append("✅গাইড")
                                    if entry.get("mcq_done"): done_tags.append("✅MCQ")
                                    if entry.get("cq_done"): done_tags.append("✅CQ")
                                    done_str = " ".join(done_tags) if done_tags else "⏳পড়া বাকি"

                                    st.markdown(f"""
                                    <div style='padding-top:2px;'>
                                        <div style='color:#1e293b; font-weight:bold; font-size:14px;'>📝 {entry['chapter']}</div>
                                        <div style='color:#475569; font-size:12px; margin-top:2px;'>🎯 {target_str}</div>
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
                    st.info("💡 টিপস: তুমি চাইলে ব্রাউজার থেকে সরাসরি প্রিন্ট (Ctrl + P) করে এই রুটিনটি প্রিন্ট আউট বা পেজে সংরক্ষণ করে রাখতে পারো।")
            else:
                st.warning("⚠️ নির্বাচিত সিলেবাসে কোনো বিষয় পাওয়া যায়নি। দয়া করে সিলেবাস চেক করুন।")
        else:
            st.warning("⚠️ কোনো সংরক্ষিত সিলেবাস পাওয়া যায়নি। দয়া করে প্রথমে 'Syllabus' ট্যাব থেকে একটি সিলেবাস তৈরি ও সেভ করুন।")

    # --- TAB 3: AUTO SMART STUDY PLAN (শুধুমাত্র নির্বাচিত সিলেবাসের বিষয় ও টপিকভিত্তিক) ---
    with tab3:
        st.markdown("### 🤖 Auto Smart Study Plan & Interactive Task Tracker")
        st.markdown("সিলেবাস নির্বাচন ও সময় দেওয়ার পর কেবল **নির্বাচিত সিলেবাসের বিষয় ও টপিকগুলো** নিয়ে অটো স্টাডি প্ল্যান তৈরি হবে। প্রতিটি বিষয়ের পড়ার সময়, স্ট্যাটাস ও কারণ ইনপুট দিয়ে লাইভ প্রোগ্রেস ট্র্যাক করো!")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, student_class, exam_term, syllabus_json, is_default FROM school_syllabus_multi ORDER BY id DESC")
        plan_syllabi = cursor.fetchall()
        conn.close()

        if plan_syllabi:
            p_choices = {}
            p_default_idx = 0
            for idx, s in enumerate(plan_syllabi):
                s_id, s_class, s_term, s_json, s_def = s[0], s[1], s[2], s[3], s[4]
                label_str = f"📌 {s_term} ({s_class}) [ID: {s_id}]"
                if s_def == 1:
                    label_str += " ⭐ (মূল সিলেবাস)"
                    p_default_idx = idx
                p_choices[label_str] = s_json

            selected_plan_s_label = st.selectbox("🎯 যে সিলেবাসটি শেষ করতে চাও তা নির্বাচন করো:", list(p_choices.keys()), index=p_default_idx, key="auto_plan_syllabus_dropdown")
            
            # 🔥 মূল সংশোধন: শুধুমাত্র সিলেক্ট করা সিলেবাসের ডিকশনারি পার্স করা হলো
            selected_syl_dict = json.loads(p_choices[selected_plan_s_label])

            # ১. এনালিটিক্স প্রিভিউ (শুধুমাত্র নির্বাচিত সিলেবাসের ওপর ভিত্তি করে)
            st.markdown("#### 📊 ১. নির্বাচিত সিলেবাসের পূর্ণাঙ্গ এনালিটিক্স প্রিভিউ:")
            total_subjs_count = len(selected_syl_dict)
            total_topics_count = sum([len(t) for cats in selected_syl_dict.values() for t in cats.values()])

            c1, c2, c3, c4 = st.columns(4)
            c1.metric("📚 মোট বিষয়", f"{total_subjs_count} টি")
            c2.metric("📝 মোট অধ্যায়/টপিক", f"{total_topics_count} টি")
            c3.metric("📖 আনুমানিক পৃষ্ঠা", f"~{total_topics_count * 8} টি")
            c4.metric("🎯 মোট MCQ ও CQ", f"~{total_topics_count * 20} MCQ | {total_topics_count * 3} CQ")

            st.markdown("---")

            # ২. সময় ও ঘণ্টা
            st.markdown("#### ⏳ ২. সময় ও দৈনিক পড়ার ঘণ্টা নির্ধারণ করুন:")
            col_t1, col_t2 = st.columns(2)
            with col_t1:
                target_days = st.number_input("সিলেবাস শেষ করার জন্য কত দিন সময় আছে?", min_value=1, max_value=365, value=3, step=1)
            with col_t2:
                daily_study_hours = st.slider("প্রতিদিন গড়ে কত ঘণ্টা পড়তে চাও? (দৈনিক টার্গেট সময়)", 1.0, 12.0, 4.0, 0.5)

            if st.button("✨ অটো স্টাডি প্ল্যান ও টাস্ক ট্র্যাকার তৈরি করুন", use_container_width=True, type="primary"):
                subj_topic_queues = {}
                # 🔥 শুধুমাত্র নির্বাচিত সিলেবাসের বিষয় ও টপিকগুলো সারিবদ্ধ করা হলো
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

                st.session_state["active_interleaved_topics"] = interleaved_topics_list
                st.session_state["active_target_days"] = target_days
                st.session_state["active_daily_hours"] = daily_study_hours
                st.session_state["active_plan_syl_label"] = selected_plan_s_label
                st.success("🎉 আপনার নির্বাচিত সিলেবাসের আলোকে স্মার্ট স্টাডি প্ল্যান প্রস্তুত করা হয়েছে!")

            # --- ৩. ইন্টারেক্টিভ প্ল্যান ও ট্র্যাকার ---
            if "active_interleaved_topics" in st.session_state:
                interleaved_topics_list = st.session_state["active_interleaved_topics"]
                target_days = st.session_state["active_target_days"]
                daily_study_hours = st.session_state["active_daily_hours"]
                
                total_items = len(interleaved_topics_list)
                topics_per_day = math.ceil(total_items / max(1, target_days))

                st.markdown("<br>", unsafe_allow_html=True)
                st.markdown("### 🏆 জেনারেটেড প্রাত্যহিক স্টাডি প্ল্যান ও লাইভ প্রোগ্রেস ট্র্যাকার:")

                # স্টেট ইনিশিয়ালাইজেশন
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

                backlog_items = []
                obstacle_reasons_list = []
                total_tasks_global = 0
                completed_score_global = 0.0

                curr_idx = 0
                for day_num in range(1, target_days + 1):
                    if curr_idx >= total_items:
                        day_tasks = [{"subject": "সমন্বিত রিভিশন", "category": "অল সাবজেক্ট", "topic": "পূর্বের সকল পঠিত বিষয় রিভিশন"}]
                    else:
                        day_tasks = interleaved_topics_list[curr_idx : curr_idx + topics_per_day]
                        curr_idx += topics_per_day

                    day_score = 0.0
                    day_task_count = len(day_tasks)

                    st.markdown(f"""
                    <div style="background: #F8FAFC; border: 2px solid #3B82F6; border-radius: 10px; padding: 15px; margin-bottom: 15px;">
                        <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #CBD5E1; padding-bottom: 8px; margin-bottom: 12px;">
                            <span style="font-size: 16px; font-weight: bold; color: #1E3A8A;">📅 দিন {day_num} (টার্গেট ডে)</span>
                            <span style="background-color: #DBEAFE; color: #1E4ED8; font-weight: bold; padding: 4px 10px; border-radius: 6px; font-size: 13px;">⏱️ নির্ধারিত টার্গেট সময়: {daily_study_hours} ঘণ্টা</span>
                        </div>
                    """, unsafe_allow_html=True)

                    for t_idx, task in enumerate(day_tasks):
                        s_name = task["subject"]
                        cat_name = task["category"]
                        top_name = task["topic"]
                        task_key = f"day_{day_num}_task_{t_idx}_{s_name}"

                        st.markdown(f"**📘 {s_name}** - *[{cat_name}] {top_name}*")
                        
                        col_st, col_hr, col_dn, col_pn = st.columns([1.5, 1.2, 2.3, 2.3])
                        
                        with col_st:
                            current_status = st.selectbox(
                                "পড়ার অবস্থা:",
                                ["✅ সম্পূর্ণ সম্পন্ন", "⏳ আংশিক সম্পন্ন", "❌ একদম পড়তে পারিনি"],
                                index=["✅ সম্পূর্ণ সম্পন্ন", "⏳ আংশিক সম্পন্ন", "❌ একদম পড়তে পারিনি"].index(st.session_state["user_subject_status"].get(task_key, "✅ সম্পূর্ণ সম্পন্ন")),
                                key=f"status_sel_{task_key}"
                            )
                            st.session_state["user_subject_status"][task_key] = current_status

                        with col_hr:
                            subject_hours_txt = st.text_input(
                                "পড়ার সময় (যেমন: ১ ঘণ্টা ১০ মি):",
                                value=str(st.session_state["user_subject_hours_text"].get(task_key, "১.০ ঘণ্টা")),
                                key=f"hrs_txt_{task_key}"
                            )
                            st.session_state["user_subject_hours_text"][task_key] = subject_hours_txt

                        with col_dn:
                            done_text = st.text_input(
                                "আজ কতটুকু পড়েছ লেখো:",
                                value=st.session_state["user_done_notes"].get(task_key, ""),
                                placeholder="যেমন: ১-৫ পৃষ্ঠা পড়েছি...",
                                key=f"done_in_{task_key}"
                            )
                            st.session_state["user_done_notes"][task_key] = done_text

                        with col_pn:
                            pending_text = st.text_input(
                                "কতটুকু বাকি আছে লেখো:",
                                value=st.session_state["user_pending_notes"].get(task_key, ""),
                                placeholder="যেমন: শব্দার্থ পড়া হয়নি...",
                                key=f"pending_in_{task_key}"
                            )
                            st.session_state["user_pending_notes"][task_key] = pending_text

                        obstacle_val = "কোনো সমস্যা নেই"
                        if current_status != "✅ সম্পূর্ণ সম্পন্ন":
                            obstacle_val = st.selectbox(
                                "⚠️ পড়া শেষ না হওয়ার মূল কারণ কী ছিল?",
                                [
                                    "⏳ সময়ের অভাব / ব্যস্ততা",
                                    "🧠 টপিক কঠিন বা বুঝতে সমস্যা হয়েছে",
                                    "🤒 শারীরিক অসুস্থতা / ক্লান্তি",
                                    "📱 অন্যমনস্কতা বা মনোযোগের অভাব",
                                    "🏠 পারিবারিক বা অন্যান্য কাজ"
                                ],
                                index=0,
                                key=f"obstacle_{task_key}"
                            )
                            st.session_state["user_obstacle_reason"][task_key] = obstacle_val
                            obstacle_reasons_list.append(obstacle_val)

                        if current_status == "✅ সম্পূর্ণ সম্পন্ন":
                            day_score += 1.0
                        elif current_status == "⏳ আংশিক সম্পন্ন":
                            day_score += 0.5
                        else:
                            day_score += 0.0

                        if current_status != "✅ সম্পূর্ণ সম্পন্ন" or pending_text.strip() != "":
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

                    st.markdown(f"**📊 দিন দিন {day_num} এর স্টাডি প্রোগ্রেস: `{day_prog_percentage}%` (টার্গেট সময়: {daily_study_hours} ঘণ্টা)**")
                    st.progress(day_prog_percentage / 100)

                    if day_prog_percentage == 100:
                        st.success(f"🎉 অসাধারণ! তুমি নির্দিষ্ট টার্গেটের চেয়ে কম সময়েই সমস্ত পড়া সম্পূর্ণ করে ফেলেছ! অল্প সময়ে সিলেবাস শেষ করতে পারার এই দক্ষতা ও ক্রেডিট সত্যিই প্রশংসনীয়!")
                    elif day_prog_percentage >= 50:
                        st.info(f"👍 দারুন চেষ্টা! তুমি আংশিক পড়া শেষ করেছ। সিলেবাস পুরোপুরি শেষ করতে চাইলে সময় একটু বাড়িয়ে নিতে পারো।")
                    else:
                        st.warning(f"⚠️ আজ পড়ার গতি একটু কম। সিলেবাস নিয়মিত শেষ করতে চাইলে সময় বাড়িয়ে মনোযোগ দিয়ে পড়ার চেষ্টা করো!")

                    st.markdown("</div>", unsafe_allow_html=True)

                # --- ৪. সার্বিক স্টাডি প্রোগ্রেস, বকেয়া রিমাইন্ডার ও এআই রুট কজ ইনসাইটস ---
                global_percentage = int((completed_score_global / max(1, total_tasks_global)) * 100)
                st.markdown("---")
                st.markdown("### 📈 আপনার সার্বিক প্রোগ্রেস রিপোর্ট ও এআই ইনসাইটস (Analytics & Insights)")
                
                col_p_left, col_p_right = st.columns([1, 2])
                with col_p_left:
                    st.metric("🔥 সামগ্রিক সিলেবাস প্রোগ্রেস", f"{global_percentage}%")
                    st.progress(global_percentage / 100)
                    if global_percentage == 100:
                        st.balloons()
                        st.success("🎉 অভিনন্দন! তুমি তোমার নির্বাচিত সিলেবাসের ১০০% টার্গেট পূরণ করেছ!")
                
                with col_p_right:
                    if obstacle_reasons_list:
                        reason_counts = Counter(obstacle_reasons_list)
                        most_common_reason, count = reason_counts.most_common(1)[0]
                        
                        st.markdown(f"""
                        <div style="background-color: #FEF3C7; border-left: 5px solid #D97706; padding: 12px 15px; border-radius: 6px; margin-bottom: 12px;">
                            <h4 style="margin: 0 0 5px 0; color: #92400E;">🤖 AI Learning Behavior Insights</h4>
                            <p style="margin: 0; font-size: 13px; color: #78350F;">
                                আপনার পড়াশোনায় পিছিয়ে পড়ার প্রধান কারণ হলো: <b>{most_common_reason}</b> ({count} বার চিহ্নিত হয়েছে)। এই সমস্যা উত্তরণে সময় ব্যবস্থাপনা বা কনসেপ্ট ক্লিয়ার করার জন্য শিক্ষকের সহায়তা নিতে পারেন।
                            </p>
                        </div>
                        """, unsafe_allow_html=True)

                    if backlog_items:
                        st.warning(f"⚠️ **বকেয়া রিমাইন্ডার (Pending Backlogs):** আপনার মোট {len(backlog_items)}টি বিষয়ে অপূর্ণ পড়া রয়েছে!")
                        with st.expander("🔔 কোন দিন কোন বিষয়ে কি পড়া বাকি আছে ও কারণ দেখে নিন", expanded=True):
                            for b in backlog_items:
                                r_disp = f" | 📌 *কারণ:* {b['reason']}" if b['reason'] != "কোনো সমস্যা নেই" else ""
                                st.markdown(f"• **দিন {b['day']} [{b['subject']}]:** {b['topic']}<br>&nbsp;&nbsp;&nbsp;&nbsp;🔹 *পড়েছ:* {b['done']} | 🔴 *বাকি:* `{b['pending']}` ({b['status']}){r_disp}", unsafe_allow_html=True)
                            st.info("💡 পরামর্শ: আগামী রিভিশন সেশন বা সাপ্তাহিক ছুটির দিনে এই বকেয়া পড়াগুলো অবশ্যই রিকভার করে নিন!")
                    else:
                        st.success("✨ দারুন! আপনার কোনো বকেয়া বা অপূর্ণ পড়া নেই। আপনি একদম পারফেক্ট ট্র্যাকে আছেন!")

        else:
            st.warning("⚠️ কোনো সংরক্ষিত সিলেবাস পাওয়া যায়নি। দয়া করে প্রথমে 'Syllabus' ট্যাব থেকে একটি সিলেবাস তৈরি ও সেভ করুন।")

   
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
        <h2 style='color: #1E3A8A; font-weight: 700; margin-bottom: 5px;'>📋 এক্সাম ও অ্যাসাইনমেন্ট শিডিউল ম্যানেজমেন্ট</h2>
        <p style='color: #475569; font-size: 15px;'>তোমার আসন্ন পরীক্ষা, কুইজ এবং অ্যাসাইনমেন্টের সময়সূচী এখানে সাজিয়ে রাখো এবং এক নজরে সামারি টেবিল দেখে নাও।</p>
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
        st.markdown("### ➕ নতুন পরীক্ষা বা অ্যাসাইনমেন্ট যুক্ত করো")
        col1, col2 = st.columns(2)

        with col1:
            exam_name = st.text_input("পরীক্ষা বা অ্যাসাইনমেন্টের নাম:", placeholder="যেমন: গণিত প্রথম পত্র পরীক্ষা")
            exam_type = st.selectbox("পরীক্ষার ধরন:", ["বার্ষিক পরীক্ষা", "অর্ধ-বার্ষিক পরীক্ষা", "সাপ্তাহিক কুইজ", "মাসিক পরীক্ষা", "অ্যাসাইনমেন্ট"])

        with col2:
            exam_date = st.date_input("পরীক্ষার তারিখ:", date.today())
            description = st.text_input("সিলেবাস বা বিশেষ নোট (ঐচ্ছিক):", placeholder="যেমন: অধ্যায় ১ থেকে ৩ পর্যন্ত")

        submitted = st.form_submit_button("রুটিনে সেভ করো", use_container_width=True, type="primary")

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
            st.markdown("### 🗓️ তোমার সংরক্ষিত রুটিন ও সময়সূচী")
        with col_head2:
            if st.button("📊 সামারি টেবিল দেখাও (Popup)", use_container_width=True, type="primary"):
                show_exam_summary_dialog(exams_list)

        filter_type = st.selectbox("ধরন অনুযায়ী ফিল্টার করো:", ["সকল", "বার্ষিক পরীক্ষা", "অর্ধ-বার্ষিক পরীক্ষা", "সাপ্তাহিক কুইজ", "মাসিক পরীক্ষা", "অ্যাসাইনমেন্ট"])

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
            ("📝 Note Book", "Notes"),
            ("📅 Study Plan", "Study Planner"),
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


