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

import json
import random
import sqlite3
import time
from datetime import datetime
import streamlit as st


# --- ডেটাবেজ ইনিশিয়ালাইজেশন ---
def init_databases():
    conn1 = sqlite3.connect("subject_notes.db")
    cursor1 = conn1.cursor()
    cursor1.execute("""
        CREATE TABLE IF NOT EXISTS notes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT,
            content TEXT,
            subject TEXT,
            note_type TEXT
        )
    """)
    conn1.commit()
    conn1.close()

    conn2 = sqlite3.connect("smart_school_syllabus.db")
    cursor2 = conn2.cursor()
    cursor2.execute("""
        CREATE TABLE IF NOT EXISTS model_test_history (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            test_title TEXT,
            test_date TEXT,
            total_marks INTEGER,
            allocated_time INTEGER,
            obtained_score INTEGER,
            correct_count INTEGER,
            wrong_count INTEGER,
            details_json TEXT
        )
    """)
    conn2.commit()
    conn2.close()


init_databases()


def render_quiz_practice():
    # --- কাস্টম স্টাইল ও ডিজাইন ---
    st.markdown(
        """
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@400;500;600;700&family=Outfit:wght@400;600;800&display=swap');

    .practice-magic-wrapper {
        font-family: 'Outfit', 'Hind Siliguri', sans-serif;
        background: linear-gradient(135deg, #fef3c7 0%, #fae8ff 50%, #e0f2fe 100%);
        padding: 30px;
        border-radius: 24px;
        border: 2px solid #f43f5e;
        box-shadow: 0 15px 35px rgba(244, 63, 94, 0.1);
        color: #1e293b;
        margin-bottom: 25px;
    }

    .practice-banner {
        background: linear-gradient(135deg, #f43f5e 0%, #fb7185 50%, #e11d48 100%);
        padding: 22px;
        border-radius: 18px;
        color: white;
        text-align: center;
        margin-bottom: 22px;
        box-shadow: 0 8px 20px rgba(244, 63, 94, 0.25);
    }

    .practice-quote-box {
        background: #ffffff;
        padding: 15px 20px;
        border-radius: 14px;
        border-left: 6px solid #f43f5e;
        border: 1px solid #fbcfe8;
        margin-bottom: 22px;
        box-shadow: 0 5px 15px rgba(244, 63, 94, 0.06);
    }

    .stTabs [data-baseweb="tab-list"] {
        gap: 10px;
    }

    .stTabs [data-baseweb="tab"] {
        height: 48px;
        background-color: #ffffff;
        border-radius: 12px;
        color: #e11d48;
        font-weight: 700;
        border: 1px solid #fbcfe8;
        padding: 8px 18px;
    }

    .stTabs [aria-selected="true"] {
        background: linear-gradient(135deg, #f43f5e 0%, #e11d48 100%) !important;
        color: white !important;
    }

    .stButton button {
        font-family: 'Outfit', 'Hind Siliguri', sans-serif !important;
        border-radius: 10px !important;
        font-weight: 700 !important;
        background: linear-gradient(135deg, #f43f5e 0%, #e11d48 100%) !important;
        color: white !important;
        border: none !important;
        box-shadow: 0 4px 12px rgba(244, 63, 94, 0.25) !important;
        transition: all 0.3s ease !important;
    }

    .stButton button:hover {
        background: linear-gradient(135deg, #e11d48 0%, #be123c 100%) !important;
        transform: translateY(-2px);
        box-shadow: 0 6px 16px rgba(244, 63, 94, 0.35) !important;
    }
    </style>
    """,
        unsafe_allow_html=True,
    )

    st.markdown("<div class='practice-magic-wrapper'>", unsafe_allow_html=True)

    st.markdown(
        """
    <div class="practice-banner">
        <h1 style="margin: 0; font-size: 26px; font-weight: 800;">🎯 স্মার্ট ইংলিশ প্র্যাকটিস ও মডেল টেস্ট হাব</h1>
        <p style="margin: 5px 0 0 0; font-size: 14px; font-weight: 500; opacity: 0.95;">
            ক্লাস ৬, ৭ ও ৮ এর ইংরেজি ১ম ও ২য় পত্রের পূর্ণাঙ্গ সিলেবাসভিত্তিক প্রস্তুতি প্ল্যাটফর্ম!
        </p>
    </div>
    """,
        unsafe_allow_html=True,
    )

    st.markdown(
        """
    <div class="practice-quote-box">
        <div style="display: flex; align-items: center; gap: 12px;">
            <span style="font-size: 26px;">💡</span>
            <div>
                <h4 style="margin: 0; color: #be123c; font-size: 14px; font-weight: 700;">শিক্ষার্থীদের জন্য টিপস:</h4>
                <p style="margin: 3px 0 0 0; color: #334155; font-size: 13px; font-weight: 500;">
                    "নিয়মিত ভোকাবুলারি ও গ্রামার প্র্যাকটিস করলে ইংরেজি ভীতি দূর হয়ে আত্মবিশ্বাস বহুগুণ বেড়ে যায়।"
                </p>
            </div>
        </div>
    </div>
    """,
        unsafe_allow_html=True,
    )

    if "user_progress" not in st.session_state:
        st.session_state["user_progress"] = []
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

    # --- পূর্ণাঙ্গ প্রশ্ন ব্যাংক ডিকশনারি (ক্লাস ৬, ৭, ৮ - ইংরেজি ১ম ও ২য় পত্র) ---
    library_question_bank = {
        "Class 6": {
            "English 1st Paper": {
                "MCQ (Reading Comprehension)": [
                    {
                        "q": "What is the main topic of the lesson 'Fabiha's First Day at School'?",
                        "options": [
                            "a) Excitement and nervousness of a new school",
                            "b) Playing football",
                            "c) Shopping in a market",
                            "d) Visiting a museum",
                        ],
                        "answer": "a) Excitement and nervousness of a new school",
                    },
                    {
                        "q": "Where did the story 'At the Airport' take place?",
                        "options": [
                            "a) Railway station",
                            "b) International Airport",
                            "c) Bus terminal",
                            "d) Launch ghat",
                        ],
                        "answer": "b) International Airport",
                    },
                    {
                        "q": "Who is the central character in the lesson about 'A Clean Village'?",
                        "options": [
                            "a) Ratan",
                            "b) Mina",
                            "c) Raju",
                            "d) Nobibur",
                        ],
                        "answer": "b) Mina",
                    },
                    {
                        "q": "What do we learn from the story of 'The Shoemaker and the Elves'?",
                        "options": [
                            "a) Laziness",
                            "b) Hard work and kindness",
                            "c) Greed",
                            "d) Dishonesty",
                        ],
                        "answer": "b) Hard work and kindness",
                    },
                    {
                        "q": "International Mother Language Day is observed on—",
                        "options": [
                            "a) 21st February",
                            "b) 26th March",
                            "c) 16th December",
                            "d) 1st May",
                        ],
                        "answer": "a) 21st February",
                    },
                ],
                "Fill in the Blanks": [
                    {
                        "q": "Fill in the blank: Honesty is the best ____.",
                        "options": [
                            "a) policy",
                            "b) habit",
                            "c) duty",
                            "d) wealth",
                        ],
                        "answer": "a) policy",
                    },
                    {
                        "q": "Fill in the blank: Birds can fly ____ the sky.",
                        "options": ["a) in", "b) on", "c) at", "d) by"],
                        "answer": "a) in",
                    },
                    {
                        "q": "Fill in the blank: Trees provide us with oxygen to ____.",
                        "options": [
                            "a) sleep",
                            "b) breathe",
                            "c) eat",
                            "d) walk",
                        ],
                        "answer": "b) breathe",
                    },
                    {
                        "q": "Fill in the blank: The Padma is a big ____.",
                        "options": [
                            "a) mountain",
                            "b) river",
                            "c) ocean",
                            "d) desert",
                        ],
                        "answer": "b) river",
                    },
                ],
                "Vocabulary": [
                    {
                        "q": "What is the synonym of the word 'Courage'?",
                        "options": [
                            "a) Fear",
                            "b) Bravery",
                            "c) Cowardice",
                            "d) Weakness",
                        ],
                        "answer": "b) Bravery",
                    },
                    {
                        "q": "What is the antonym of the word 'Ancient'?",
                        "options": [
                            "a) Old",
                            "b) Modern",
                            "c) Historic",
                            "d) Aged",
                        ],
                        "answer": "b) Modern",
                    },
                    {
                        "q": "What does the word 'Hospitality' mean?",
                        "options": [
                            "a) Treating guests with warmth and generosity",
                            "b) Cruelty to animals",
                            "c) Fighting enemies",
                            "d) Ignoring people",
                        ],
                        "answer": "a) Treating guests with warmth and generosity",
                    },
                ],
            },
            "English 2nd Paper": {
                "Right Forms of Verbs": [
                    {
                        "q": "Choose the correct form: 'Water (boil) at 100 degree centigrade.'",
                        "options": [
                            "a) boil",
                            "b) boils",
                            "c) boiled",
                            "d) is boiling",
                        ],
                        "answer": "b) boils",
                    },
                    {
                        "q": "Choose the correct form: 'Yesterday, he (go) to Dhaka.'",
                        "options": ["a) goes", "b) went", "c) gone", "d) go"],
                        "answer": "b) went",
                    },
                    {
                        "q": "Choose the correct form: 'Look! The boy (play) football.'",
                        "options": [
                            "a) play",
                            "b) plays",
                            "c) is playing",
                            "d) played",
                        ],
                        "answer": "c) is playing",
                    },
                ],
                "Parts of Speech": [
                    {
                        "q": "Identify the part of speech of the underlined word: 'She runs *fast*.'",
                        "options": [
                            "a) Noun",
                            "b) Adjective",
                            "c) Adverb",
                            "d) Verb",
                        ],
                        "answer": "c) Adverb",
                    },
                    {
                        "q": "Identify the noun in the sentence: 'Honesty is the best policy.'",
                        "options": [
                            "a) Honesty",
                            "b) Best",
                            "c) Policy",
                            "d) Both a and c",
                        ],
                        "answer": "d) Both a and c",
                    },
                ],
                "Changing Sentences & Narration": [
                    {
                        "q": "Make it negative: 'He is an honest man.'",
                        "options": [
                            "a) He is not a dishonest man.",
                            "b) He is not an honest man.",
                            "c) He never honest.",
                            "d) None",
                        ],
                        "answer": "a) He is not a dishonest man.",
                    },
                    {
                        "q": "Change into Indirect: Rahim said, 'I am reading a book.'",
                        "options": [
                            "a) Rahim said that he was reading a book.",
                            "b) Rahim said that I am reading a book.",
                            "c) Rahim told that he is reading.",
                            "d) Rahim says he reads.",
                        ],
                        "answer": "a) Rahim said that he was reading a book.",
                    },
                ],
            },
        },
        "Class 7": {
            "English 1st Paper": {
                "MCQ (Reading Comprehension)": [
                    {
                        "q": "Begum Rokeya fought all her life for—",
                        "options": [
                            "a) Women's rights and education",
                            "b) Political power",
                            "c) Wealth",
                            "d) Sports development",
                        ],
                        "answer": "a) Women's rights and education",
                    },
                    {
                        "q": "Where did the Tagore's family live?",
                        "options": [
                            "a) Jorasanko, Kolkata",
                            "b) Sylhet",
                            "c) Chattogram",
                            "d) Rajshahi",
                        ],
                        "answer": "a) Jorasanko, Kolkata",
                    },
                    {
                        "q": "What is 'Heritage'?",
                        "options": [
                            "a) Traditions and qualities passed down through generations",
                            "b) Modern gadgets",
                            "c) Foreign cultures",
                            "d) Plastic goods",
                        ],
                        "answer": (
                            "a) Traditions and qualities passed down through"
                            " generations"
                        ),
                    },
                ],
                "Fill in the Blanks": [
                    {
                        "q": "Fill in the blank: Prevention is ____ than cure.",
                        "options": [
                            "a) better",
                            "b) worse",
                            "c) good",
                            "d) best",
                        ],
                        "answer": "a) better",
                    },
                    {
                        "q": "Fill in the blank: The sun sets in the ____.",
                        "options": ["a) East", "b) West", "c) North", "d) South"],
                        "answer": "b) West",
                    },
                ],
                "Vocabulary": [
                    {
                        "q": "Synonym of 'Enormous' is—",
                        "options": [
                            "a) Tiny",
                            "b) Huge / Massive",
                            "c) Little",
                            "d) Short",
                        ],
                        "answer": "b) Huge / Massive",
                    },
                    {
                        "q": "Antonym of 'Transparent' is—",
                        "options": [
                            "a) Clear",
                            "b) Opaque",
                            "c) Bright",
                            "d) Clean",
                        ],
                        "answer": "b) Opaque",
                    },
                ],
            },
            "English 2nd Paper": {
                "Right Forms of Verbs": [
                    {
                        "q": "Choose the correct form: 'If you come, I (go) with you.'",
                        "options": ["a) went", "b) will go", "c) go", "d) gone"],
                        "answer": "b) will go",
                    },
                    {
                        "q": "Choose the correct form: 'He speaks as though he (know) everything.'",
                        "options": [
                            "a) knows",
                            "b) knew",
                            "c) had known",
                            "d) has known",
                        ],
                        "answer": "b) knew",
                    },
                ],
                "Punctuation & Capitalization": [
                    {
                        "q": "Which punctuation mark is used at the end of an interrogative sentence?",
                        "options": [
                            "a) Full stop (.)",
                            "b) Question mark (?)",
                            "c) Exclamation (!)",
                            "d) Comma (,)",
                        ],
                        "answer": "b) Question mark (?)",
                    }
                ],
            },
        },
        "Class 8": {
            "English 1st Paper": {
                "MCQ (Reading Comprehension)": [
                    {
                        "q": "What is ethnic people's culture mostly based on?",
                        "options": [
                            "a) Agriculture and nature",
                            "b) Industrial technology",
                            "c) Space science",
                            "d) Corporate business",
                        ],
                        "answer": "a) Agriculture and nature",
                    },
                    {
                        "q": "Shat Gambuj Mosque is located in—",
                        "options": [
                            "a) Bagerhat",
                            "b) Sylhet",
                            "c) Cumilla",
                            "d) Cox's Bazar",
                        ],
                        "answer": "a) Bagerhat",
                    },
                ],
                "Fill in the Blanks": [
                    {
                        "q": "Fill in the blank: Industry is the key to ____.",
                        "options": [
                            "a) success",
                            "b) failure",
                            "c) poverty",
                            "d) laziness",
                        ],
                        "answer": "a) success",
                    }
                ],
                "Vocabulary": [
                    {
                        "q": "Synonym of 'Significant' is—",
                        "options": [
                            "a) Important",
                            "b) Trivial",
                            "c) Minor",
                            "d) Useless",
                        ],
                        "answer": "a) Important",
                    }
                ],
            },
            "English 2nd Paper": {
                "Right Forms of Verbs": [
                    {
                        "q": "Choose the correct form: 'It is high time we (change) our bad habits.'",
                        "options": [
                            "a) change",
                            "b) changed",
                            "c) changing",
                            "d) have changed",
                        ],
                        "answer": "b) changed",
                    }
                ],
                "Narration": [
                    {
                        "q": "Change into Indirect: 'What is your name?' he said to me.",
                        "options": [
                            "a) He asked me what my name was.",
                            "b) He told me what is my name.",
                            "c) He asked what is your name.",
                            "d) None",
                        ],
                        "answer": "a) He asked me what my name was.",
                    }
                ],
            },
        },
    }

    # --- ৪টি সুন্দর ট্যাব তৈরি ---
    tab1, tab2, tab3, tab4 = st.tabs([
        "📖 প্র্যাকটিস ও রিভিশন",
        "📝 কুইজ ও প্র্যাকটিস টেস্ট",
        "🏆 মডেল টেস্ট কনফিগার ও পরীক্ষা",
        "📊 পরীক্ষার ইতিহাস ও রিটেস্ট",
    ])

    # ================= TAB 1: প্র্যাকটিস ও রিভিশন =================
    with tab1:
        st.markdown(
            "### 📖 ক্লাস ও টপিকভিত্তিক ইংরেজি অনুশীলন (Vocabulary, Grammar & MCQ)"
        )

        c1, c2, c3 = st.columns(3, gap="medium")
        with c1:
            p_class = st.selectbox(
                "শ্রেণি বেছে নিন:",
                list(library_question_bank.keys()),
                key="p_class_sel",
            )
        with c2:
            p_books = list(library_question_bank[p_class].keys())
            p_book = st.selectbox(
                "বই বা পেপার বেছে নিন:", p_books, key="p_book_sel"
            )
        with c3:
            p_chapters = list(library_question_bank[p_class][p_book].keys())
            p_chap = st.selectbox(
                "টপিক / ক্যাটাগরি বেছে নিন:", p_chapters, key="p_chap_sel"
            )

        st.markdown(
            "<hr style='margin: 15px 0; border: 0; border-top: 2px dashed"
            " #fbcfe8;'>",
            unsafe_allow_html=True,
        )

        q_list = library_question_bank[p_class][p_book][p_chap]
        for idx, item in enumerate(q_list, 1):
            with st.expander(f"📌 প্রশ্ন {idx}: {item['q']}"):
                st.write("**বিকল্পসমূহ:**")
                for opt in item["options"]:
                    st.write(f"- {opt}")
                st.success(f"**সঠিক উত্তর:** {item['answer']}")

    # ================= TAB 2: কুইজ ও প্র্যাকটিস টেস্ট =================
    with tab2:
        st.markdown("### 📝 ইন্টারঅ্যাক্টিভ প্র্যাকটিস টেস্ট জেনারেটর")

        q_c1, q_c2, q_c3 = st.columns(3, gap="medium")
        with q_c1:
            q_sel_class = st.selectbox(
                "শ্রেণি বেছে নিন:",
                list(library_question_bank.keys()),
                key="quiz_class_sel",
            )
        with q_c2:
            q_available_books = list(
                library_question_bank[q_sel_class].keys()
            )
            q_selected_book = st.selectbox(
                "পেপার বেছে নিন:", q_available_books, key="quiz_book_sel"
            )
        with q_c3:
            q_available_chapters = list(
                library_question_bank[q_sel_class][q_selected_book].keys()
            )
            q_selected_chapter = st.selectbox(
                "টপিক বেছে নিন:", q_available_chapters, key="quiz_chap_sel"
            )

        st.markdown(
            "<hr style='margin: 15px 0; border: 0; border-top: 2px dashed"
            " #fbcfe8;'>",
            unsafe_allow_html=True,
        )

        qc1, qc2 = st.columns(2, gap="medium")
        with qc1:
            question_count = st.number_input(
                "প্রশ্ন সংখ্যা নির্ধারণ করুন:",
                min_value=1,
                max_value=50,
                value=3,
                step=1,
                key="quiz_q_count_input",
            )
        with qc2:
            study_hours = st.number_input(
                "পড়াশোনার সময় (ঘণ্টা):",
                min_value=0.5,
                max_value=5.0,
                value=1.0,
                step=0.5,
                key="study_hours_input",
            )

        st.markdown("<br>", unsafe_allow_html=True)
        if st.button("🚀 প্র্যাকটিস টেস্ট শুরু করুন", use_container_width=True):
            st.session_state["quiz_active"] = True
            st.session_state["show_review"] = False

            raw_questions = library_question_bank[q_sel_class][
                q_selected_book
            ][q_selected_chapter]
            selected_subset = random.sample(
                raw_questions, min(len(raw_questions), int(question_count))
            )
            st.session_state["generated_quiz"] = selected_subset
            st.rerun()

        if st.session_state.get(
            "quiz_active", False
        ) and "generated_quiz" in st.session_state:
            st.markdown("---")
            st.markdown(
                f"### 📋 প্র্যাকটিস সেশন: {q_sel_class} - {q_selected_book} ({q_selected_chapter})"
            )

            if st.session_state.get("show_review", False):
                st.markdown("### 📊 ফলাফল ও সঠিক উত্তরসমূহ:")
                for idx, item in enumerate(
                    st.session_state["generated_quiz"], 1
                ):
                    user_ans = st.session_state.get(f"lib_q_{idx}")
                    correct_ans = item["answer"]
                    st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                    if user_ans == correct_ans:
                        st.success(f"আপনার উত্তর: {user_ans} (সঠিক আছে)")
                    else:
                        st.error(f"আপনার উত্তর: {user_ans} (ভুল ❌)")
                        st.info(f"সঠিক উত্তর হবে: **{correct_ans}**")
                    st.markdown("---")

                if st.button(
                    "🔄 আবার নতুন করে প্র্যাকটিস শুরু করুন",
                    use_container_width=True,
                ):
                    st.session_state["quiz_active"] = False
                    st.session_state["show_review"] = False
                    st.rerun()
            else:
                for idx, item in enumerate(
                    st.session_state["generated_quiz"], 1
                ):
                    st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                    st.radio(
                        f"উত্তর নির্বাচন করুন (প্রশ্ন {idx}):",
                        item["options"],
                        key=f"lib_q_{idx}",
                    )
                    st.markdown("")

                if st.button(
                    "📥 উত্তর জমা দিন ও প্রগ্রেস সেভ করুন",
                    key="submit_quiz_btn",
                    use_container_width=True,
                ):
                    correct_count = 0
                    total_q = len(st.session_state["generated_quiz"])

                    for idx, item in enumerate(
                        st.session_state["generated_quiz"], 1
                    ):
                        u_ans = st.session_state.get(f"lib_q_{idx}")
                        c_ans = item["answer"]
                        if u_ans == c_ans:
                            correct_count += 1

                    percentage = (
                        (correct_count / total_q) * 100 if total_q > 0 else 0
                    )
                    score_str = f"{correct_count}/{total_q} ({percentage:.0f}%)"
                    performance = (
                        "Excellent (চমৎকার)"
                        if percentage >= 80
                        else (
                            "Very Good (খুব ভালো)"
                            if percentage >= 50
                            else "Good (চেষ্টা চালিয়ে যাও)"
                        )
                    )

                    progress_record = {
                        "class": q_sel_class,
                        "subject": q_selected_book,
                        "chapter": q_selected_chapter,
                        "mode": "Topic Practice",
                        "study_hours": study_hours,
                        "score": score_str,
                        "performance": performance,
                    }
                    st.session_state["user_progress"].append(progress_record)

                    st.balloons()
                    st.success(
                        f"অভিনন্দন! আপনার স্কোর: **{score_str}** | পারফরম্যান্স:"
                        f" **{performance}**"
                    )
                    st.session_state["show_review"] = True
                    st.rerun()

    # ================= TAB 3: মডেল টেস্ট কনফিগার ও পরীক্ষা =================
    with tab3:
        st.markdown(
            "### 🏆 কাস্টম মডেল টেস্ট ও লাইভ কাউন্টডাউন পরীক্ষা (ইংরেজি)"
        )

        if not st.session_state.exam_started:
            with st.form("model_test_custom_config_form"):
                st.markdown("#### পরীক্ষার কনফিগারেশন সেটআপ করুন")
                sel_class = st.selectbox(
                    "ক্লাস নির্বাচন করুন:",
                    list(library_question_bank.keys()),
                    key="m_class_select",
                )
                available_subjects = list(
                    library_question_bank[sel_class].keys()
                )
                sel_subjects = st.multiselect(
                    "পেপার নির্বাচন করুন:",
                    available_subjects,
                    default=available_subjects[:1],
                    key="m_subjects_select",
                )

                all_available_chapters = []
                for sub in sel_subjects:
                    for ch in library_question_bank[sel_class][sub].keys():
                        all_available_chapters.append(f"{sub} -> {ch}")

                sel_chapters = st.multiselect(
                    "টপিকসমূহ নির্বাচন করুন:",
                    all_available_chapters,
                    default=all_available_chapters,
                    key="m_chapters_select",
                )
                test_name_input = st.text_input(
                    "মডেল টেস্টের শিরোনাম:",
                    value="ইংরেজি মডেল টেস্ট পরীক্ষা",
                    key="m_test_name_input_val",
                )

                col_el, col_e2 = st.columns(2)
                with col_el:
                    custom_marks = st.number_input(
                        "প্রশ্ন সংখ্যা / পূর্ণমান:",
                        min_value=1,
                        max_value=100,
                        value=5,
                        step=1,
                        key="m_custom_marks_input",
                    )
                with col_e2:
                    custom_time = st.number_input(
                        "নির্ধারিত সময় (মিনিট):",
                        min_value=1,
                        max_value=100,
                        value=5,
                        step=1,
                        key="m_custom_time_input",
                    )

                start_exam_btn = st.form_submit_button(
                    "🚀 মডেল টেস্ট শুরু করুন",
                    use_container_width=True,
                    type="primary",
                )

                if start_exam_btn:
                    if not sel_subjects or not sel_chapters:
                        st.error(
                            "দয়া করে কমপক্ষে একটি পেপার এবং টপিক সিলেক্ট করুন।"
                        )
                    else:
                        pool_questions = []
                        for item in sel_chapters:
                            sub_name, ch_name = item.split(" -> ")
                            ch_q_list = library_question_bank[sel_class][
                                sub_name
                            ].get(ch_name, [])
                            pool_questions.extend(ch_q_list)

                        if not pool_questions:
                            st.error(
                                "নির্বাচিত টপিকগুলোতে পর্যাপ্ত প্রশ্ন পাওয়া"
                                " যায়নি।"
                            )
                        else:
                            num_q_needed = min(
                                len(pool_questions), int(custom_marks)
                            )
                            selected_qs = random.sample(
                                pool_questions, num_q_needed
                            )

                            st.session_state.exam_questions = selected_qs
                            st.session_state.exam_started = True
                            st.session_state.exam_submitted = False
                            st.session_state.is_saved_to_db = False
                            st.session_state.user_answers = {}
                            st.session_state.exam_title_str = test_name_input
                            st.session_state.exam_total_marks = int(
                                custom_marks
                            )
                            st.session_state.exam_time_allocated = int(
                                custom_time
                            )
                            st.session_state.exam_deadline = time.time() + (
                                int(custom_time) * 60
                            )
                            st.rerun()
        else:
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
            <div style="background: linear-gradient(135deg, #1e3a8a 0%, #2563eb 100%); color: white; padding: 15px 20px; border-radius: 12px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
                <div style="font-size: 16px; font-weight: bold;">
                    পরীক্ষা চলছে: <span style="color: #fef08a;">{st.session_state.exam_title_str}</span>
                </div>
                <div style="background-color: #ef4444; color: white; padding: 6px 14px; border-radius: 8px; font-size: 20px; font-weight: bold; font-family: monospace;">
                    {time_str_display}
                </div>
            </div>
            """,
                unsafe_allow_html=True,
            )

            with st.form("active_custom_exam_form"):
                for idx, q_data in enumerate(st.session_state.exam_questions):
                    st.markdown(f"**প্রশ্ন {idx + 1}: {q_data['q']}**")
                    options = q_data["options"]
                    ans_key = f"q_ans_live_{idx}"
                    user_choice = st.radio(
                        f"উত্তর সিলেক্ট করুন ({idx + 1})",
                        options,
                        key=ans_key,
                        index=None,
                    )
                    st.session_state.user_answers[idx] = user_choice
                    st.markdown(
                        "<hr style='margin: 10px 0; border: 0.5px solid"
                        " #e2e8f0;'>",
                        unsafe_allow_html=True,
                    )

                submit_test = st.form_submit_button(
                    "📩 পরীক্ষা সম্পন্ন ও খাতা জমা দিন",
                    use_container_width=True,
                    type="primary",
                )
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
                current_date_str = datetime.now().strftime(
                    "%Y-%m-%d %I:%M %p"
                )
                details_data = json.dumps(
                    st.session_state.exam_questions, ensure_ascii=False
                )

                conn = sqlite3.connect("smart_school_syllabus.db")
                cursor = conn.cursor()
                cursor.execute(
                    """
                    INSERT INTO model_test_history (test_title, test_date, total_marks, allocated_time, obtained_score, correct_count, wrong_count, details_json)
                    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                """,
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

            st.success("🎉 আপনার মডেল টেস্ট সফলভাবে জমা দেওয়া হয়েছে!")
            st.markdown("### পরীক্ষার ফলাফল ও মূল্যায়ন")
            st.success(
                f"**পূর্ণমান:** {st.session_state.exam_total_marks} | **প্রাপ্ত"
                f" স্কোর:** {score} | **সঠিক:** {correct_cnt} টি |"
                f" **ভুল:** {wrong_cnt} টি ({percentage:.1f}%)"
            )

            with st.expander("📖 উত্তরপত্র ও সঠিক উত্তরসমূহ দেখে নিন", expanded=True):
                for idx, q_data in enumerate(st.session_state.exam_questions):
                    user_ans = st.session_state.user_answers.get(idx)
                    correct_ans = q_data["answer"]
                    if user_ans == correct_ans:
                        st.markdown(
                            f"✅ **প্রশ্ন {idx + 1}:** সঠিক (আপনার উত্তর:"
                            f" {user_ans})"
                        )
                    else:
                        st.markdown(
                            f"❌ **প্রশ্ন {idx + 1}:** ভুল (আপনার উত্তর:"
                            f" {user_ans if user_ans else 'উত্তর দেননি'})।"
                            f" সঠিক উত্তর: **{correct_ans}**"
                        )

            if st.button(
                "🔄 নতুন মডেল টেস্টে ফিরে যান",
                use_container_width=True,
                type="primary",
            ):
                st.session_state.exam_started = False
                st.session_state.exam_submitted = False
                st.session_state.is_saved_to_db = False
                st.session_state.exam_questions = []
                st.session_state.user_answers = {}
                st.rerun()

    # ================= TAB 4: পরীক্ষার ইতিহাস ও রিটেস্ট =================
    with tab4:
        st.markdown("### 📊 সংরক্ষিত মডেল টেস্টের ইতিহাস ও রিটেস্ট")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute(
            "SELECT id, test_title, test_date, total_marks, allocated_time,"
            " obtained_score, correct_count, wrong_count, details_json FROM"
            " model_test_history ORDER BY id DESC"
        )
        past_tests = cursor.fetchall()
        conn.close()

        if past_tests:
            for pt in past_tests:
                (
                    t_id,
                    t_title,
                    t_date,
                    t_total,
                    t_time,
                    t_score,
                    t_correct,
                    t_wrong,
                    tjson,
                ) = pt
                with st.expander(
                    f"📂 {t_title} | তারিখ: {t_date} | প্রাপ্ত স্কোর:"
                    f" {t_score}/{t_total}"
                ):
                    col_hs1, col_hs2 = st.columns(2)
                    with col_hs1:
                        st.markdown(f"- **পূর্ণমান:** {t_total}")
                        st.markdown(f"- **নির্ধারিত সময়:** {t_time} মিনিট")
                    with col_hs2:
                        st.markdown(f"- **সঠিক উত্তর:** {t_correct} টি")
                        st.markdown(f"- **ভুল উত্তর:** {t_wrong} টি")

                    col_h1, col_h2 = st.columns(2)
                    with col_h1:
                        if st.button(
                            "🔄 পুনরায় এই টেস্টটি দিন (Retest)",
                            key=f"retest_btn_{t_id}",
                            use_container_width=True,
                            type="primary",
                        ):
                            st.session_state.exam_questions = json.loads(tjson)
                            st.session_state.exam_started = True
                            st.session_state.exam_submitted = False
                            st.session_state.is_saved_to_db = False
                            st.session_state.user_answers = {}
                            st.session_state.exam_title_str = (
                                f"পুনরায় পরীক্ষা: {t_title}"
                            )
                            st.session_state.exam_total_marks = t_total
                            st.session_state.exam_time_allocated = t_time
                            st.session_state.exam_deadline = time.time() + (
                                t_time * 60
                            )
                            st.rerun()
                    with col_h2:
                        if st.button(
                            "🗑️ হিস্ট্রি ডিলেট করুন",
                            key=f"del_history_btn_{t_id}",
                            use_container_width=True,
                        ):
                            con_del = sqlite3.connect(
                                "smart_school_syllabus.db"
                            )
                            cur_del = con_del.cursor()
                            cur_del.execute(
                                "DELETE FROM model_test_history WHERE id=?",
                                (t_id,),
                            )
                            con_del.commit()
                            con_del.close()
                            st.success("হিস্ট্রি সফলভাবে মুছে ফেলা হয়েছে!")
                            st.rerun()
        else:
            st.info(
                "ℹ️ এখনও কোনো মডেল টেস্ট দেওয়া হয়নি। ৩ নম্বর ট্যাব থেকে"
                " কাস্টম কনফিগার করে আপনার প্রথম মডেল টেস্ট শুরু করুন।"
            )

    st.markdown("</div>", unsafe_allow_html=True)

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

import sqlite3
import json
import streamlit as st

def render_study_planner():
    st.markdown("""
    <div style="background: linear-gradient(135deg, #1E3A8A 0%, #3882F6 100%); padding: 25px; border-radius: 12px; color: white; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
        <h2 style="margin: 0; color: white;">ক্লাস সেভেন স্মার্ট স্কুল সিলেবাস ও ডিজিটাল বই হাব</h2>
        <p style="margin: 5px 0 0 0; font-size: 16px; opacity: 0.9;">একাধিক বিষয় ও অধ্যায় সিলেক্ট করে আপনার সুশৃঙ্খল ডিজিটাল সিলেবাস তৈরি করুন!</p>
    </div>
    """, unsafe_allow_html=True)

    tab1, tab2, tab3 = st.tabs(["📚 সিলেবাস ক্রিয়েটর ও ডিজিটাল বুক", "⏰ স্মার্ট স্টাডি প্লান", "📝 মডেল টেস্ট প্রস্তুতি"])

    # --- ক্লাস সেভেনের সমন্বিত সিলেবাস ডিকশনারি (Original Structure) ---
    unified_syllabus_dictionary = {
        "বাংলা প্রথম পত্র": {
            "গদ্যাংশ": [
                "কাবুলিওয়ালা", "লখার একুশে", "মরু-ভাস্কর", "শব্দ থেকে কবিতা", 
                "পাখি", "পিতৃ পুরুষের গল্প", "ছবির রং", "সেই ছেলেটি", "বহু জাতিসত্তার দেশ বাংলাদেশ"
            ],
            "কবিতাংশ": [
                "নতুন দেশ", "কুলি-মজুর", "সাম্য", "মেলা", "এই অক্ষরে", "সিঁথি", "কোকিল", "কিং লিয়ার", "যুদ্ধক্ষেত্রে পিতাপুত্র", "নাটিকা: জাগো সুন্দর"
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
                "পত্র / দরখাস্ত (ছাত্রাবাস জীবনের অভিজ্ঞতা জানিয়ে মাকে পত্র, জীবনের লক্ষ্য জানিয়ে বড় ভাইকে চিঠি, অনুপস্থিতি ছুটি ও জরিমানা মওকুফের আবেদন, পাঠাগার স্থাপনের জন্য আবেদন, লোকজ উৎসবের বর্ণনা দিয়ে প্রবাসী বন্ধুকে পত্র ইত্যাদি)",
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
                "The Life of a Farmer", "Our National Flag", "My Hobby", "Deforestation", "A Street Hawker"
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
                "তৃতীয় অধ্যায়: নিরাপদ ও নৈতিক ব্যবহার", "চতুর্থ অধ্যায়: ওয়ার্ড প্রসেসিং", "পঞ্চম অধ্যায়: শিক্ষায় ইন্টারনেটের ব্যবহার"
            ]
        }
    }

    with tab1:
        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS school_syllabus_multi (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                student_class TEXT,
                exam_term TEXT,
                syllabus_json TEXT
            )
        """)
        conn.commit()

        cursor.execute("SELECT id, student_class, exam_term, syllabus_json FROM school_syllabus_multi ORDER BY id DESC LIMIT 1")
        saved_record = cursor.fetchone()
        conn.close()

        if "edit_mode" not in st.session_state:
            st.session_state.edit_mode = False

        if not saved_record or st.session_state.edit_mode:
            st.markdown("### 📝 ক্লাস সেভেন মাল্টি-সাবজেক্ট সিলেবাস ইনপুট দিন")
            
            default_class = "সপ্তম শ্রেণি (Class 7)"
            default_term = "সাপ্তাহিক কুইজ"
            parsed_saved_data = {}

            if saved_record and st.session_state.edit_mode:
                default_class = saved_record[1]
                default_term = saved_record[2]
                parsed_saved_data = json.loads(saved_record[3])

            col1, col2 = st.columns(2)
            with col1:
                student_class = st.selectbox("শ্রেণি নির্বাচন করুন:", ["সপ্তম শ্রেণি (Class 7)"], index=0, key="sel_class_multi")
                exam_terms_list = ["সাপ্তাহিক কুইজ", "অর্ধ-বার্ষিক", "বার্ষিক", "মডেল টেস্ট", "প্রি-টেস্ট", "সিটি", "অন্যান্য মূল্যায়ন"]
                term_idx = exam_terms_list.index(default_term) if default_term in exam_terms_list else 0
                exam_term = st.selectbox("পরীক্ষার নাম:", exam_terms_list, index=term_idx, key="sel_term_multi")
                
            with col2:
                available_subjects = list(unified_syllabus_dictionary.keys())
                default_subjs = list(parsed_saved_data.keys()) if parsed_saved_data else []
                selected_subjects = st.multiselect("একাধিক বিষয় নির্বাচন করুন:", available_subjects, default=default_subjs, key="sel_subjects_multi")

            selected_syllabus_data = {}
            if selected_subjects:
                st.markdown("#### নির্বাচিত বিষয়গুলোর অধ্যায়সমূহ নির্ধারণ করুন:")
                for subj in selected_subjects:
                    with st.expander(f"📖 বিষয়: {subj}", expanded=True):
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
                btn_text = "💾 ফাইল আপডেট করুন" if st.session_state.edit_mode else "নির্ধারিত সিলেবাস ছক আকারে সেভ করুন"
                if st.button(btn_text, use_container_width=True, type="primary"):
                    if selected_syllabus_data:
                        syllabus_str = json.dumps(selected_syllabus_data, ensure_ascii=False)
                        
                        conn = sqlite3.connect("smart_school_syllabus.db")
                        cursor = conn.cursor()
                        cursor.execute("DELETE FROM school_syllabus_multi")
                        cursor.execute("""
                            INSERT INTO school_syllabus_multi (student_class, exam_term, syllabus_json)
                            VALUES (?, ?, ?)
                        """, (student_class, exam_term, syllabus_str))
                        conn.commit()
                        conn.close()
                        
                        st.session_state.edit_mode = False
                        st.success("সিলেবাস সফলভাবে সেভ করা হয়েছে!")
                        st.rerun()
                    else:
                        st.error("দয়া করে অন্তত একটি বিষয় এবং তার অধ্যায় সিলেক্ট করুন।")

            if st.session_state.edit_mode:
                with col_btn_s2:
                    if st.button("❌ Cancel (বাতিল)", use_container_width=True):
                        st.session_state.edit_mode = False
                        st.rerun()

        else:
            rec_id, r_class, r_term, r_json = saved_record
            parsed_syllabus = json.loads(r_json)

            st.markdown(f"### 📋 আপনার সাজানো ডিজিটাল সিলেবাস ছক ({r_class} - {r_term})")

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
            table_html = f'<table style="width:100%; border-collapse: collapse; margin-top: 10px; font-family: sans-serif;"><thead><tr style="background-color: #1E3A8A; color: white; text-align: left;"><th style="padding: 12px; border: 1px solid #cbd5e1;">বিষয় (Subject)</th><th style="padding: 12px; border: 1px solid #cbd5e1;">বিভাগ (Category)</th><th style="padding: 12px; border: 1px solid #cbd5e1;">নির্বাচিত অধ্যায়সমূহ (Selected Topics)</th></tr></thead><tbody>{all_rows}</tbody></table>'
            
            st.markdown(table_html, unsafe_allow_html=True)
            st.markdown("<br>", unsafe_allow_html=True)

            col_btn1, col_btn2 = st.columns(2)
            with col_btn1:
                if st.button("✏️ এডিট করুন", use_container_width=True):
                    st.session_state.edit_mode = True
                    st.rerun()
            with col_btn2:
                if st.button("🗑️ ডিলেট করুন", use_container_width=True):
                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("DELETE FROM school_syllabus_multi")
                    conn.commit()
                    conn.close()
                    st.session_state.edit_mode = False
                    st.success("সিলেবাস সফলভাবে বাতিল ও মুছে ফেলা হয়েছে! পুনরায় সিলেবাস তৈরি করুন।")
                    st.rerun()


# --- ২. স্মার্ট স্টাডি প্লান ও টাইমলাইন সেকশন ---
    with tab2:
        st.markdown("### ⏰ স্মার্ট স্টাডি প্লান ও টাইমলাইন")

        # ডাটাবেজ টেবিল নিশ্চিতকরণ
        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS study_plans_multi (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                plan_title TEXT,
                plan_type TEXT,
                start_date TEXT,
                end_date TEXT,
                daily_hours TEXT,
                plan_json TEXT
            )
        """)
        # প্রতিদিনের কাস্টম নোটের জন্য টেবিল
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS daily_study_notes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                plan_id INTEGER,
                day_number INTEGER,
                note_text TEXT,
                status_text TEXT
            )
        """)
        conn.commit()

        # ডাটাবেজ থেকে সমস্ত সেভ করা প্ল্যান ফেচ করা
        cursor.execute("SELECT id, plan_title, plan_type, start_date, end_date, daily_hours, plan_json FROM study_plans_multi ORDER BY id DESC")
        all_saved_plans = cursor.fetchall()
        conn.close()

        if "plan_edit_id" not in st.session_state:
            st.session_state.plan_edit_id = None
        if "create_new_plan" not in st.session_state:
            st.session_state.create_new_plan = False

        # --- ১. নতুন প্ল্যান তৈরি বা বিদ্যমান প্ল্যান এডিট করার ফর্ম ---
        if not all_saved_plans or st.session_state.create_new_plan or st.session_state.plan_edit_id is not None:
            
            editing_record = None
            if st.session_state.plan_edit_id is not None:
                for p in all_saved_plans:
                    if p[0] == st.session_state.plan_edit_id:
                        editing_record = p
                        break

            form_header = "✏️ স্টাডি প্ল্যান এডিট করুন" if editing_record else "➕ নতুন স্মার্ট স্টাডি প্ল্যান তৈরি করুন"
            st.markdown(f"#### {form_header}")
            st.markdown("আপনার সিলেবাস থেকে বিষয় ও অধ্যায়সমূহ বাছাই করে নির্দিষ্ট মেয়াদের সমন্বিত রুটিন তৈরি করুন:")

            conn = sqlite3.connect("smart_school_syllabus.db")
            cursor = conn.cursor()
            cursor.execute("SELECT syllabus_json FROM school_syllabus_multi ORDER BY id DESC LIMIT 1")
            saved_plan_record = cursor.fetchone()
            conn.close()

            saved_syllabus_dict = {}
            if saved_plan_record:
                saved_syllabus_dict = json.loads(saved_plan_record[0])

            if saved_syllabus_dict:
                available_saved_subjects = list(saved_syllabus_dict.keys())
                
                import datetime
                today = datetime.date.today()
                
                default_title = editing_record[1] if editing_record else "আমার সমন্বিত স্টাডি প্ল্যান"
                default_start = datetime.datetime.strptime(editing_record[3], "%Y-%m-%d").date() if editing_record else today
                default_end = datetime.datetime.strptime(editing_record[4], "%Y-%m-%d").date() if editing_record else (today + datetime.timedelta(days=7))
                default_hrs = float(editing_record[5].replace(" ঘণ্টা", "")) if editing_record else 3.0
                
                parsed_edit_topics = []
                if editing_record:
                    edit_json_data = json.loads(editing_record[6])
                    parsed_edit_topics = edit_json_data.get("topics", [])

                plan_title_input = st.text_input("স্টাডি প্ল্যানের নাম/শিরোনাম দিন:", default_title, key="plan_title_in")
                
                col_d1, col_d2, col_d3 = st.columns(3)
                with col_d1:
                    start_date = st.date_input("শুরুর তারিখ:", default_start, key="plan_start_date_in")
                with col_d2:
                    end_date = st.date_input("শেষের তারিখ:", default_end, key="plan_end_date_in")
                with col_d3:
                    daily_hrs = st.slider("প্রতিদিনের পড়ার সময় (ঘণ্টা):", 1.0, 8.0, default_hrs, 0.5, key="daily_hrs_slider_in")

                calc_days = (end_date - start_date).days + 1
                if calc_days == 7:
                    p_type_label = "সাপ্তাহিক প্ল্যান (৭ দিন)"
                elif calc_days in [28, 29, 30, 31]:
                    p_type_label = "মাসিক প্ল্যান (১ মাস)"
                elif calc_days in [14, 15]:
                    p_type_label = "পাক্ষিক প্ল্যান (১৫ দিন)"
                elif calc_days in [180, 182, 183]:
                    p_type_label = "ষান্মাসিক প্ল্যান (৬ মাস)"
                elif calc_days in [365, 366]:
                    p_type_label = "বার্ষিক প্ল্যান (১ বছর)"
                else:
                    p_type_label = f"কাস্টম প্ল্যান ({calc_days} দিন)"

                st.info(f"💡 নির্ণীত মেয়াদ: **{p_type_label}**")

                default_selected_subjs = list(set([t[0] for t in parsed_edit_topics])) if parsed_edit_topics else available_saved_subjects
                target_subjects = st.multiselect("কমপ্লিট করার জন্য বিষয়সমূহ সিলেক্ট করুন:", available_saved_subjects, default=default_selected_subjs, key="target_subjects_multi_in")

                selected_comprehensive_topics = []
                if target_subjects:
                    st.markdown("##### 📚 বিষয় ভিত্তিক অধ্যায় ও টপিকসমূহ নির্ধারণ করুন:")
                    for subj in target_subjects:
                        with st.expander(f"📖 বিষয়: {subj}", expanded=True):
                            categories_dict = saved_syllabus_dict.get(subj, {})
                            for cat, topics in categories_dict.items():
                                default_t_sel = [t[2] for t in parsed_edit_topics if t[0] == subj and t[1] == cat] if parsed_edit_topics else topics
                                chosen_topics = st.multiselect(
                                    f"{subj} -> {cat}:",
                                    topics,
                                    default=[t for t in default_t_sel if t in topics],
                                    key=f"plan_topics_form_{subj}_{cat}"
                                )
                                for t in chosen_topics:
                                    selected_comprehensive_topics.append((subj, cat, t))

                st.markdown("<br>", unsafe_allow_html=True)

                col_b1, col_b2 = st.columns(2)
                with col_b1:
                    save_btn_txt = "💾 স্টাডি প্ল্যান আপডেট করুন" if editing_record else "🚀 স্টাডি প্ল্যান তৈরি ও সংরক্ষণ করুন"
                    if st.button(save_btn_txt, use_container_width=True, type="primary", key="save_plan_action_btn"):
                        if start_date > end_date:
                            st.error("দয়া করে সঠিক তারিখ দিন (শুরুর তারিখ শেষের তারিখের পরে হতে পারে না)।")
                        elif not selected_comprehensive_topics:
                            st.error("দয়া করে অন্তত একটি বিষয় বা টপিক সিলেক্ট করুন।")
                        else:
                            plan_data = {
                                "title": plan_title_input,
                                "type": p_type_label,
                                "start_date": str(start_date),
                                "end_date": str(end_date),
                                "total_days": calc_days,
                                "daily_hours": f"{daily_hrs} ঘণ্টা",
                                "topics": selected_comprehensive_topics
                            }
                            plan_str = json.dumps(plan_data, ensure_ascii=False)

                            conn = sqlite3.connect("smart_school_syllabus.db")
                            cursor = conn.cursor()
                            
                            if editing_record:
                                cursor.execute("""
                                    UPDATE study_plans_multi 
                                    SET plan_title=?, plan_type=?, start_date=?, end_date=?, daily_hours=?, plan_json=?
                                    WHERE id=?
                                """, (plan_title_input, p_type_label, str(start_date), str(end_date), f"{daily_hrs} ঘণ্টা", plan_str, editing_record[0]))
                            else:
                                cursor.execute("""
                                    INSERT INTO study_plans_multi (plan_title, plan_type, start_date, end_date, daily_hours, plan_json)
                                    VALUES (?, ?, ?, ?, ?, ?)
                                """, (plan_title_input, p_type_label, str(start_date), str(end_date), f"{daily_hrs} ঘণ্টা", plan_str))
                            
                            conn.commit()
                            conn.close()

                            st.session_state.create_new_plan = False
                            st.session_state.plan_edit_id = None
                            st.success("স্টাডি প্ল্যান সফলভাবে সংরক্ষণ করা হয়েছে!")
                            st.rerun()
                
                if st.session_state.create_new_plan or st.session_state.plan_edit_id is not None:
                    with col_b2:
                        if st.button("❌ বাতিল (Cancel)", use_container_width=True, key="cancel_form_action_btn"):
                            st.session_state.create_new_plan = False
                            st.session_state.plan_edit_id = None
                            st.rerun()
            else:
                st.warning("⚠️ অনুগ্রহ করে প্রথমে 'সিলেবাস ক্রিয়েটর ও ডিজিটাল বুক' ট্যাব থেকে সিলেবাস ইনপুট ও সেভ করুন, তবেই এখানে স্টাডি প্ল্যান তৈরি করতে পারবেন।")

        # --- ২. যদি সেভ করা প্ল্যান থাকে ---
        else:
            plan_options = {}
            for p in all_saved_plans:
                p_id_val, p_title_val, p_type_val, p_start_val, p_end_val = p[0], p[1], p[2], p[3], p[4]
                label_str = f"📌 {p_title_val} [{p_type_val}] ({p_start_val} হতে {p_end_val})"
                plan_options[label_str] = p

            selected_plan_label = st.selectbox(
                "আপনার সংরক্ষিত স্টাডি প্ল্যানগুলো থেকে বেছে নিন (সবগুলো প্ল্যান সেভ থাকবে):", 
                list(plan_options.keys()), 
                key="select_saved_plan_main_dropdown"
            )
            
            selected_record = plan_options[selected_plan_label]
            p_id, p_title, p_type, p_start, p_end, p_hrs, p_json = selected_record
            plan_data = json.loads(p_json)
            raw_topics_list = plan_data.get("topics", [])
            total_days = plan_data.get("total_days", 7)

            # ডাটাবেজ থেকে এই প্ল্যানের সেভ করা নোটস লোড করা
            conn = sqlite3.connect("smart_school_syllabus.db")
            cursor = conn.cursor()
            cursor.execute("SELECT day_number, note_text, status_text FROM daily_study_notes WHERE plan_id = ?", (p_id,))
            existing_notes_rows = cursor.fetchall()
            conn.close()

            saved_notes_dict = {}
            for row in existing_notes_rows:
                saved_notes_dict[row[0]] = {"note": row[1], "status": row[2]}

            st.markdown(f"### 📋 {p_title} <span style='font-size:14px; background:#e0e7ff; color:#1e3a8a; padding:3px 10px; border-radius:12px; margin-left:10px;'>{p_type}</span>", unsafe_allow_html=True)
            st.markdown(f"📅 **তারিখ:** {p_start} হতে {p_end} (মোট {total_days} দিন) | ⏰ **দৈনিক পড়ার সময়:** {p_hrs}")

            import datetime
            import math
            start_dt = datetime.datetime.strptime(p_start, "%Y-%m-%d").date()

            subject_grouped = {}
            for item in raw_topics_list:
                subj, cat, t = item[0], item[1], item[2]
                if subj not in subject_grouped:
                    subject_grouped[subj] = []
                subject_grouped[subj].append((subj, cat, t))

            interleaved_topics = []
            max_len = max([len(v) for v in subject_grouped.values()]) if subject_grouped else 0
            subj_keys = list(subject_grouped.keys())

            for i in range(max_len):
                for sk in subj_keys:
                    if i < len(subject_grouped[sk]):
                        interleaved_topics.append(subject_grouped[sk][i])

            num_items = len(interleaved_topics)
            items_per_day = max(1, math.ceil(num_items / total_days))
            
            item_index = 0

            # --- সলিড অল-বর্ডার (Solid All Border) সিএসএস স্টাইল ---
            st.markdown("""
            <style>
            .clean-grid-table {
                width: 100%;
                border-collapse: collapse !important;
                border: 2px solid #2563eb !important;
                margin-top: 15px;
                font-family: sans-serif;
            }
            .clean-grid-table th {
                background-color: #1e3a8a;
                color: white;
                padding: 12px 10px;
                border: 2px solid #2563eb !important;
                text-align: left;
                font-size: 14px;
                font-weight: bold;
            }
            .clean-grid-table td {
                border: 2px solid #2563eb !important;
                padding: 8px 10px;
                background-color: #ffffff;
                vertical-align: top;
            }
            </style>
            """, unsafe_allow_html=True)

            # টেবিলের মূল হেডার
            st.markdown("""
            <table class="clean-grid-table">
                <thead>
                    <tr>
                        <th style="width: 12%;">দিন ও তারিখ</th>
                        <th style="width: 19%;">বিষয়সমূহ</th>
                        <th style="width: 39%;">নির্ধারিত পড়ার টপিকসমূহ</th>
                        <th style="width: 18%;">নোট / খসড়া ইনপুট</th>
                        <th style="width: 12%; text-align: center;">স্ট্যাটাস ও সেভ</th>
                    </tr>
                </thead>
            </table>
            """, unsafe_allow_html=True)

            for day in range(1, total_days + 1):
                current_day_date = start_dt + datetime.timedelta(days=day - 1)
                date_str = current_day_date.strftime("%d %b, %Y")

                day_note_info = saved_notes_dict.get(day, {"note": "", "status": "⏳ বাকি আছে"})
                current_note = day_note_info["note"]
                current_status = day_note_info["status"]

                if item_index >= num_items:
                    day_subjs = "রিভিশন ও প্র্যাকটিস"
                    formatted_topics_html = "<span style='color:#059669; font-weight:bold;'>✨ পূর্বে পঠিত সকল টপিকের সম্যক রিভিশন ও মডেল টেস্ট অনুশীলন</span>"
                else:
                    assigned_items = interleaved_topics[item_index : item_index + items_per_day]
                    item_index += items_per_day
                    
                    unique_subjs = list(dict.fromkeys([item[0] for item in assigned_items]))
                    day_subjs = ", ".join(unique_subjs)
                    
                    formatted_topic_list = []
                    for item in assigned_items:
                        s_name, c_name, t_name = item[0], item[1], item[2]
                        badge_html = f"""<div style="margin-bottom: 5px; line-height: 1.4;">
                            <span style="background-color: #e0e7ff; color: #1e3a8a; padding: 2px 6px; border-radius: 4px; font-size: 11px; font-weight: bold; border: 1px solid #c7d2fe;">{s_name}</span> 
                            <span style="font-size: 12px; color: #1e293b; font-weight: 500;">🔹 {t_name}</span>
                        </div>"""
                        formatted_topic_list.append(badge_html)
                    
                    formatted_topics_html = "".join(formatted_topic_list)

                # চারপাশের অল-বর্ডার (All Borders: 2px solid #2563eb) সহ প্রতিটি গ্রিড রো
                st.markdown("<div style='border: 2px solid #2563eb; border-top: none; background-color: #ffffff; margin-bottom: 0px;'>", unsafe_allow_html=True)
                
                col1, col2, col3, col4, col5 = st.columns([1.2, 1.9, 3.9, 1.8, 1.2])

                with col1:
                    st.markdown(f"<div style='border-right: 2px solid #2563eb; padding: 8px; min-height: 90px;'><b>দিন - {day}</b><br><span style='font-size:11px; color:#475569;'>({date_str})</span></div>", unsafe_allow_html=True)

                with col2:
                    st.markdown(f"<div style='border-right: 2px solid #2563eb; padding: 8px; min-height: 90px;'><span style='color:#1e3a8a; font-weight:bold; font-size:13px;'>{day_subjs}</span></div>", unsafe_allow_html=True)

                with col3:
                    st.markdown(f"<div style='border-right: 2px solid #2563eb; padding: 8px; min-height: 90px; background: #ffffff;'>{formatted_topics_html}</div>", unsafe_allow_html=True)

                with col4:
                    st.markdown("<div style='border-right: 2px solid #2563eb; padding: 6px; min-height: 90px;'>", unsafe_allow_html=True)
                    user_entry_note = st.text_input(
                        f"নোট লিখুন (দিন {day}):", 
                        value=current_note, 
                        key=f"direct_note_{p_id}_{day}",
                        label_visibility="collapsed",
                        placeholder="আজকের খসড়া নোট..."
                    )
                    st.markdown("</div>", unsafe_allow_html=True)

                with col5:
                    st.markdown("<div style='padding: 6px; min-height: 90px;'>", unsafe_allow_html=True)
                    status_opts = ["⏳ বাকি আছে", "✅ সম্পন্ন"]
                    s_idx = status_opts.index(current_status) if current_status in status_opts else 0
                    selected_st = st.selectbox(
                        f"স্ট্যাটাস {day}", 
                        status_opts, 
                        index=s_idx, 
                        key=f"direct_status_{p_id}_{day}",
                        label_visibility="collapsed"
                    )
                    
                    if st.button("💾 সেভ", key=f"save_row_btn_{p_id}_{day}", use_container_width=True):
                        conn = sqlite3.connect("smart_school_syllabus.db")
                        cursor = conn.cursor()
                        cursor.execute("DELETE FROM daily_study_notes WHERE plan_id = ? AND day_number = ?", (p_id, day))
                        cursor.execute("""
                            INSERT INTO daily_study_notes (plan_id, day_number, note_text, status_text)
                            VALUES (?, ?, ?, ?)
                        """, (p_id, day, user_entry_note.strip(), selected_st))
                        conn.commit()
                        conn.close()
                        
                        st.toast(f"🎉 দিন - {day}-এর তথ্য সংরক্ষিত হয়েছে!", icon="✅")
                        import time
                        time.sleep(0.5)
                        st.rerun()
                    st.markdown("</div>", unsafe_allow_html=True)

                st.markdown("</div>", unsafe_allow_html=True)

            st.markdown("<br>", unsafe_allow_html=True)

            # --- ৩টি মূল বাটন: এডিট, ডিলেট, নতুন যোগ ---
            col_opt1, col_opt2, col_opt3 = st.columns(3)
            with col_opt1:
                if st.button("✏️ বর্তমান প্ল্যান এডিট করুন", use_container_width=True, key=f"edit_plan_btn_{p_id}"):
                    st.session_state.plan_edit_id = p_id
                    st.rerun()
            with col_opt2:
                if st.button("🗑️ বর্তমান প্ল্যানটি ডিলেট করুন", use_container_width=True, key=f"delete_plan_btn_{p_id}"):
                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("DELETE FROM study_plans_multi WHERE id = ?", (p_id,))
                    cursor.execute("DELETE FROM daily_study_notes WHERE plan_id = ?", (p_id,))
                    conn.commit()
                    conn.close()
                    st.success("নির্বাচিত স্টাডি প্ল্যানটি সফলভাবে মুছে ফেলা হয়েছে!")
                    st.rerun()
            with col_opt3:
                if st.button("➕ নতুন আরেকটি স্টাডি প্ল্যান তৈরি করুন", use_container_width=True, type="primary", key=f"add_another_new_plan_btn_{p_id}"):
                    st.session_state.create_new_plan = True
                    st.session_state.plan_edit_id = None
                    st.rerun()

            st.info("💡 পরামর্শ: আপনি চাইলে ব্রাউজার থেকে সরাসরি প্রিন্ট (Ctrl + P) করে আপনার যেকোনো সংরক্ষিত রুটিন ও নোট কাগজের মতো সংগ্রহ করতে পারেন।")


    
# --- ৩. মডেল টেস্ট প্রস্তুতি সেকশন ---
    with tab3:
        st.markdown("### 📝 মডেল টেস্ট ও প্রস্তুতি প্ল্যানার")
        st.markdown("আপনার কাস্টমাইজ করা নম্বর, সময় এবং নিখুঁত অটো-সাবমিট কাউন্টডাউন সিস্টেমের মাধ্যমে মডেল টেস্ট দিন:")

        # ডাটাবেজ টেবিল নিশ্চিতকরণ
        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS model_test_history (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                test_title TEXT,
                test_date TEXT,
                total_marks INTEGER,
                allocated_time INTEGER,
                obtained_score INTEGER,
                correct_count INTEGER,
                wrong_count INTEGER,
                details_json TEXT
            )
        """)
        conn.commit()
        conn.close()

        # প্রশ্ন ব্যাংক ডিকশনারি
        library_question_bank = {
            "Class 6": {
                "বাংলা (চারুপাঠ)": {
                    "অধ্যায় ১: মাদার তেরেসা": [
                        {"q": "মাদার তেরেসা কত সালে জন্মগ্রহণ করেন?", "options": ["ক) ১৯১০ সালের ২৬ আগস্ট", "খ) ১৯১২ সালের ২৭ আগস্ট", "গ) ১৯১৪ সালের ২৮ আগস্ট", "ঘ) ১৯১৬ সালের ২৯ আগস্ট"], "answer": "ক) ১৯১০ সালের ২৬ আগস্ট"},
                        {"q": "মাদার তেরেসার জন্মস্থান কোথায়?", "options": ["ক) ভারত", "খ) যুগোস্লাভিয়ার স্কোপিয়ে", "গ) ফ্রান্স", "ঘ) ইংল্যান্ড"], "answer": "খ) যুগোস্লাভিয়ার স্কোপিয়ে"},
                        {"q": "মাদার তেরেসার আসল নাম কী ছিল?", "options": ["ক) মার্গারেট", "খ) অ্যাগনেস গোনহেক্সা বোয়াযু", "গ) এলিজাবেথ", "ঘ) ক্যাথরিন"], "answer": "খ) অ্যাগনেস গোনহেক্সা বোয়াযু"},
                        {"q": "কত বছর বয়সে মাদার তেরেসা ঘরের মায়া ত্যাগ করে নান হওয়ার সিদ্ধান্ত নেন?", "options": ["ক) ১৫ বছর বয়সে", "খ) ১৮ বছর বয়সে", "গ) ২০ বছর বয়সে", "ঘ) ২২ বছর বয়সে"], "answer": "খ) ১৮ বছর বয়সে"},
                        {"q": "মাদার তেরেসা কোন ধর্মীয় মিশনারিতে যোগ দেন?", "options": ["ক) লরেট সিস্টার্স", "খ) সিস্টার্স অব চ্যারিটি", "গ) সেন্ট মেরি সংঘ", "ঘ) আসিসি সংঘ"], "answer": "ক) লরেট সিস্টার্স"},
                        {"q": "মাদার তেরেসা শিক্ষকতা করার জন্য প্রথম কোন দেশে আসেন?", "options": ["ক) পাকিস্তান", "খ) ভারত", "গ) বাংলাদেশ", "ঘ) মায়ানমার"], "answer": "খ) ভারত"},
                        {"q": "ভারতের কোন শহরে মাদার তেরেসা লরেট কনভেন্ট স্কুলের শিক্ষিকা হিসেবে যোগ দেন?", "options": ["ক) মুম্বাই", "খ) দিল্লি", "গ) কলকাতা", "ঘ) চেন্নাই"], "answer": "গ) কলকাতা"},
                        {"q": "কলকাতায় এসে মাদার তেরেসা কোন ভাষা খুব ভালোভাবে আয়ত্ত করেছিলেন?", "options": ["ক) হিন্দি ও ইংরেজি", "খ) বাংলা ও ইংরেজি", "গ) উর্দু ও ফার্সি", "ঘ) মারাঠি"], "answer": "খ) বাংলা ও ইংরেজি"},
                        {"q": "মাদার তেরেসা কলকাতার সেন্ট মেরি স্কুলে কত বছর শিক্ষকতা করেন?", "options": ["ক) প্রায় ১০ বছর", "খ) প্রায় ১৫ বছর", "গ) প্রায় ১৭ বছর", "ঘ) প্রায় ২০ বছর"], "answer": "গ) প্রায় ১৭ বছর"},
                        {"q": "কলকাতার কোন ঘটনার পর মাদার তেরেসা কনভেন্ট ছেড়ে সাধারণ মানুষের সেবায় নেমে পড়েন?", "options": ["ক) মহামারী", "খ) ১৯৪৬ সালের ভয়াবহ দাঙ্গা", "গ) বন্যা", "ঘ) দুর্ভিক্ষ"], "answer": "খ) ১৯৪৬ সালের ভয়াবহ দাঙ্গা"},
                        {"q": "কত সালে মাদার তেরেসা কলকাতার বস্তিবাসীদের সেবার জন্য নতুন সংগঠন গড়ে তোলেন?", "options": ["ক) ১৯৪৮ সালে", "খ) ১৯৫০ সালে", "গ) ১৯৫২ সালে", "ঘ) ১৯৫৫ সালে"], "answer": "খ) ১৯৫০ সালে"},
                        {"q": "মাদার তেরেসার গড়া সেবামূলক প্রতিষ্ঠানের নাম কী?", "options": ["ক) সেবা সংঘ", "খ) মিশনারিজ অব চ্যারিটি", "গ) চ্যারিটি হোম", "ঘ) নির্মল হৃদয়"], "answer": "খ) মিশনারিজ অব চ্যারিটি"}
                    ],
                    "অধ্যায় ২: চিঠি বিলি": [
                        {"q": "'চিঠি বিলি' কবিতার রচয়িতা কে?", "options": ["ক) সুকুমার রায়", "খ) রোকনুজ্জামান খান", "গ) শামসুর রাহমান", "ঘ) জসীমউদ্দীন"], "answer": "খ) রোকনুজ্জামান খান"},
                        {"q": "রোকনুজ্জামান খান কী নামে বেশি পরিচিত?", "options": ["ক) দাদাভাই", "খ) চাচাভাই", "গ) নানুভাই", "ঘ) খোকাভাই"], "answer": "ক) দাদাভাই"},
                        {"q": "'চিঠি বিলি' কবিতায় চিঠি বিলি করতে কে ছুটে চলেছে?", "options": ["ক) চিংড়ি মাছ", "খ) ব্যাঙ", "গ) খলসে মাছ", "ঘ) কাতলা মাছ"], "answer": "খ) ব্যাঙ"},
                        {"q": "চিঠি বিলি করতে যাওয়ার সময় ব্যাঙের মাথায় কী ছিল?", "options": ["ক) টুপি", "খ) ছাতা", "গ) পাতার মুকুট", "ঘ) গামছা"], "answer": "খ) ছাতা"},
                        {"q": "খেয়ানৌকার মাঝি কে?", "options": ["ক) পুঁটি মাছ", "খ) চিংড়ির বাচ্চা", "গ) খলসে মাছ", "ঘ) ভেটকি মাছ"], "answer": "খ) চিংড়ির বাচ্চা"},
                        {"q": "রোকনুজ্জামান খান দাদাভাই কোথায় জন্মগ্রহণ করেন?", "options": ["ক) ঢাকা জেলায়", "খ) ফরিদপুর জেলার পাংশায়", "গ) কুষ্টিয়া জেলায়", "ঘ) বরিশাল জেলায়"], "answer": "খ) ফরিদপুর জেলার পাংশায়"},
                        {"q": "শিশুদের সংগঠন 'কচিকাঁচার মেলা' কে প্রতিষ্ঠা করেন?", "options": ["ক) সুকুমার রায়", "খ) রোকনুজ্জামান খান দাদাভাই", "গ) শামসুর রাহমান", "ঘ) কাজী নজরুল ইসলাম"], "answer": "খ) রোকনুজ্জামান খান দাদাভাই"}
                    ]
                },
                "গণিত": {
                    "অধ্যায় ৫: সরল সমীকরণ": [
                        {"q": "সরল সমীকরণে চলকের সর্বোচ্চ ঘাত বা শক্তি কত হয়?", "options": ["ক) ১", "খ) ২", "গ) ৩", "ঘ) ৪"], "answer": "ক) ১"},
                        {"q": "x + 5 = 10 সমীকরণে x এর মান কত?", "options": ["ক) ৩", "খ) ৪", "গ) ৫", "ঘ) ৬"], "answer": "গ) ৫"},
                        {"q": "2x = 12 হলে, x এর মান কত?", "options": ["ক) ৪", "খ) ৫", "গ) ৬", "ঘ) ৭"], "answer": "গ) ৬"},
                        {"q": "কোনো সমীকরণের সমান চিহ্নের বাম দিকের রাশিকে কী বলা হয়?", "options": ["ক) ডানপক্ষ", "খ) বামপক্ষ", "গ) চলক", "ঘ) ধ্রুবক"], "answer": "খ) বামপক্ষ"},
                        {"q": "সমীকরণের মূল বা বীজ বলতে কী বোঝায়?", "options": ["ক) চলকের মান", "খ) ধ্রুবকের মান", "গ) পক্ষের মান", "ঘ) গুণের মান"], "answer": "ক) চলকের মান"}
                    ],
                    "অধ্যায় ৮: তথ্য ও উপাত্ত": [
                        {"q": "উপাত্ত প্রধানত কত প্রকার?", "options": ["ক) ১ প্রকার", "খ) ২ প্রকার", "গ) ৩ প্রকার", "ঘ) ৪ প্রকার"], "answer": "খ) ২ প্রকার"},
                        {"q": "সরাসরি উৎস থেকে সংগৃহীত উপাত্তকে কী বলা হয়?", "options": ["ক) প্রাথমিক উপাত্ত", "খ) মাধ্যমিক উপাত্ত", "গ) বিন্যস্ত উপাত্ত", "ঘ) অবিন্যস্ত উপাত্ত"], "answer": "ক) প্রাথমিক উপাত্ত"},
                        {"q": "যে উপাত্তগুলো মানের ক্রমানুসারে সাজানো থাকে না, তাকে কী বলে?", "options": ["ক) বিন্যস্ত উপাত্ত", "খ) অবিন্যস্ত উপাত্ত", "গ) প্রাথমিক উপাত্ত", "ঘ) মাধ্যমিক উপাত্ত"], "answer": "খ) অবিন্যস্ত উপাত্ত"}
                    ]
                }
            },
            "Class 7": {
                "গণিত": {
                    "অধ্যায় ৩": [
                        {"q": "শূন্য (০) কোন ধরনের সংখ্যা?", "options": ["ক) ধনাত্মক সংখ্যা", "খ) ঋণাত্মক সংখ্যা", "গ) অঋণাত্মক পূর্ণসংখ্যা", "ঘ) ভগ্নাংশ সংখ্যা"], "answer": "গ) অঋণাত্মক পূর্ণসংখ্যা"},
                        {"q": "-৫ এর বিপরীত সংখ্যা কোনটি?", "options": ["ক) -৫", "খ) ৫", "গ) ০", "ঘ) ১/৫"], "answer": "খ) ৫"},
                        {"q": "সংখ্যাতালিকায় মূলবিন্দুর ডান দিকের সংখ্যাগুলো কেমন হয়?", "options": ["ক) ঋণাত্মক", "খ) ধনাত্মক", "গ) শূন্য", "ঘ) সমান"], "answer": "খ) ধনাত্মক"}
                    ]
                },
                "বাংলা": {
                    "পিতৃপুরুষের গল্প": [
                        {"q": "'পিতৃপুরুষের গল্প' রচনাটির লেখক কে?", "options": ["ক) হারুন হাবীব", "খ) সেলিনা হোসেন", "গ) শওকত ওসমান", "ঘ) রাবেয়া খাতুন"], "answer": "ক) হারুন হাবীব"},
                        {"q": "'পিতৃপুরুষের গল্প' মূলত কোন পটভূমিতে রচিত?", "options": ["ক) ভাষা আন্দোলন", "খ) মুক্তিযুদ্ধ", "গ) বায়ান্নর একুশে ফেব্রুয়ারি", "ঘ) বাষ্পীয় বিপ্লব"], "answer": "খ) মুক্তিযুদ্ধ"}
                    ]
                }
            }
        }

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

        # --- ১. কনফিগারেশন ফর্ম ---
        if not st.session_state.exam_started:
            
            with st.form("model_test_custom_config_form"):
                st.markdown("#### ⚙️ কাস্টম মডেল টেস্ট কনফিগার করুন")
                
                sel_class = st.selectbox("ক্লাস নির্বাচন করুন:", list(library_question_bank.keys()), key="m_class_select")
                
                available_subjects = list(library_question_bank[sel_class].keys())
                sel_subjects = st.multiselect("বিষয়সমূহ নির্বাচন করুন:", available_subjects, default=available_subjects[:1], key="m_subjects_select")
                
                all_available_chapters = []
                for sub in sel_subjects:
                    for ch in library_question_bank[sel_class][sub].keys():
                        all_available_chapters.append(f"{sub} -> {ch}")

                sel_chapters = st.multiselect("অধ্যায়সমূহ নির্বাচন করুন:", all_available_chapters, default=all_available_chapters, key="m_chapters_select")
                
                test_name_input = st.text_input("মডেল টেস্টের নাম/শিরোনাম দিন:", value="আমার কাস্টম মডেল টেস্ট", key="m_test_name_input_val")

                col_e1, col_e2, col_e3 = st.columns(3)
                with col_e1:
                    exam_type = st.selectbox("পরীক্ষার ধরন:", ["MCQ (বহুনির্বাচনি)", "সৃজনশীল (CQ) - অচিরেই আসছে"], key="m_type_select")
                with col_e2:
                    custom_marks = st.number_input("পূর্ণমান / নম্বর:", min_value=1, max_value=200, value=10, step=1, key="m_custom_marks_input")
                with col_e3:
                    custom_time = st.number_input("নির্ধারিত সময় (মিনিট):", min_value=1, max_value=300, value=1, step=1, key="m_custom_time_input")

                start_exam_btn = st.form_submit_button("🚀 মডেল টেস্ট শুরু করুন", use_container_width=True, type="primary")

                if start_exam_btn:
                    if not sel_subjects or not sel_chapters:
                        st.error("দয়া করে কমপক্ষে একটি বিষয় এবং অধ্যায় সিলেক্ট করুন।")
                    elif "সৃজনশীল" in exam_type:
                        st.warning("সৃজনশীল প্রশ্ন ব্যাংক সংযোজন চলছে। দয়া করে বর্তমানে MCQ পরীক্ষা দিন।")
                    else:
                        import random, time
                        pool_questions = []
                        for item in sel_chapters:
                            sub_name, ch_name = item.split(" -> ")
                            ch_q_list = library_question_bank[sel_class][sub_name].get(ch_name, [])
                            pool_questions.extend(ch_q_list)

                        if not pool_questions:
                            st.error("নির্বাচিত অধ্যায়গুলোতে পর্যাপ্ত প্রশ্ন পাওয়া যায়নি।")
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
                            # সঠিক ডেডলাইন টাইম সেট করা (বর্তমান সময় + মিনিট)
                            st.session_state.exam_deadline = time.time() + (int(custom_time) * 60)
                            st.rerun()

        else:
            import time
            current_time = time.time()
            time_left = int(st.session_state.exam_deadline - current_time)

            # যদি সময় শেষ হয়ে যায় এবং ইতিমধ্যে সাবমিট না হয়ে থাকে
            if time_left <= 0 and not st.session_state.exam_submitted:
                st.session_state.exam_submitted = True
                st.rerun()

            # --- ২. পরীক্ষা চলাকালীন লাইভ কাউন্টডাউন ও কুইজ ইন্টারফেস ---
            st.markdown("---")
            
            # হেডার ও রিয়েল-টাইম কাউন্টডাউন বার
            m_left = max(0, time_left // 60)
            s_left = max(0, time_left % 60)
            time_str_display = f"{m_left:02d}:{s_left:02d}"

            st.markdown(f"""
            <div style="background: linear-gradient(135deg, #1e3a8a 0%, #2563eb 100%); color: white; padding: 15px 20px; border-radius: 10px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1);">
                <div style="font-size: 18px; font-weight: bold;">
                    ⏱️ পরীক্ষা চলছে: <span style="color: #fef08a;">{st.session_state.exam_title_str}</span>
                </div>
                <div style="background-color: #ef4444; color: white; padding: 8px 16px; border-radius: 8px; font-size: 22px; font-weight: bold; font-family: monospace; letter-spacing: 2px;">
                    ⏳ বাকি সময়: {time_str_display}
                </div>
            </div>
            """, unsafe_allow_html=True)

            st.warning(f"⏰ নির্ধারিত সময়: {st.session_state.exam_time_allocated} মিনিট। সময় শেষ হলে খাতা স্বয়ংক্রিয়ভাবে জমা হয়ে যাবে।")

            with st.form("active_custom_exam_form"):
                for idx, q_data in enumerate(st.session_state.exam_questions):
                    st.markdown(f"**প্রশ্ন {idx + 1}: {q_data['q']}**")
                    options = q_data['options']
                    ans_key = f"q_ans_live_{idx}"
                    
                    user_choice = st.radio(f"উত্তর সিলেক্ট করুন ({idx + 1}):", options, key=ans_key, index=None)
                    st.session_state.user_answers[idx] = user_choice
                    st.markdown("<hr style='margin: 10px 0; border: 0.5px solid #e2e8f0;'>", unsafe_allow_html=True)

                submit_test = st.form_submit_button("✅ পরীক্ষা সম্পন্ন ও খাতা জমা দিন", use_container_width=True, type="primary")

                if submit_test:
                    st.session_state.exam_submitted = True
                    st.rerun()

            # অটো-রিফ্রেশ ট্রিগার করার জন্য যাতে টাইমার প্রতি সেকেন্ডে কমে
            if time_left > 0 and not st.session_state.exam_submitted:
                time.sleep(1)
                st.rerun()

            # --- ৩. ফলাফল মূল্যায়ন ও ডাবল সেভ রোধ ---
            if st.session_state.get("exam_submitted", False):
                score = 0
                total_q = len(st.session_state.exam_questions)
                correct_cnt = 0
                wrong_cnt = 0
                
                for idx, q_data in enumerate(st.session_state.exam_questions):
                    user_ans = st.session_state.user_answers.get(idx)
                    correct_ans = q_data['answer']
                    if user_ans == correct_ans:
                        score += 1
                        correct_cnt += 1
                    else:
                        wrong_cnt += 1

                percentage = (score / total_q) * 100 if total_q > 0 else 0
                
                if not st.session_state.is_saved_to_db:
                    import datetime
                    current_date_str = datetime.datetime.now().strftime("%Y-%m-%d %I:%M %p")
                    details_data = json.dumps(st.session_state.exam_questions, ensure_ascii=False)

                    conn = sqlite3.connect("smart_school_syllabus.db")
                    cursor = conn.cursor()
                    cursor.execute("""
                        INSERT INTO model_test_history (test_title, test_date, total_marks, allocated_time, obtained_score, correct_count, wrong_count, details_json)
                        VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                    """, (
                        st.session_state.exam_title_str, 
                        current_date_str, 
                        st.session_state.exam_total_marks, 
                        st.session_state.exam_time_allocated, 
                        score, 
                        correct_cnt, 
                        wrong_cnt, 
                        details_data
                    ))
                    conn.commit()
                    conn.close()
                    
                    st.session_state.is_saved_to_db = True

                if time_left <= 0:
                    st.error("⏰ নির্ধারিত সময় শেষ! আপনার খাতা স্বয়ংক্রিয়ভাবে জমা নেওয়া হয়েছে।")
                else:
                    st.success("✅ আপনার পরীক্ষা সফলভাবে জমা দেওয়া হয়েছে!")

                st.markdown("### 📊 পরীক্ষার ফলাফল ও মূল্যায়ন")
                st.success(f"🎉 **প্রাপ্ত স্কোর:** {score} / {st.session_state.exam_total_marks} | **সঠিক:** {correct_cnt} টি | **ভুল/অনুত্তীর্ণ:** {wrong_cnt} টি ({percentage:.1f}%)")

                with st.expander("🔍 উত্তরপত্র ও সঠিক উত্তরসমূহ দেখে নিন", expanded=True):
                    for idx, q_data in enumerate(st.session_state.exam_questions):
                        user_ans = st.session_state.user_answers.get(idx)
                        correct_ans = q_data['answer']
                        if user_ans == correct_ans:
                            st.markdown(f"✅ **প্রশ্ন {idx + 1}:** সঠিক (আপনার উত্তর: {user_ans})")
                        else:
                            st.markdown(f"❌ **প্রশ্ন {idx + 1}:** ভুল (আপনার উত্তর: {user_ans if user_ans else 'উত্তর দেননি'} | সঠিক উত্তর: **{correct_ans}**)")

                if st.button("🔄 নতুন মডেল টেস্টে ফিরে যান", use_container_width=True, type="primary"):
                    st.session_state.exam_started = False
                    st.session_state.exam_submitted = False
                    st.session_state.is_saved_to_db = False
                    st.session_state.exam_questions = []
                    st.session_state.user_answers = {}
                    st.rerun()

        # --- ৪. সংরক্ষিত ইতিহাস ও রিটেস্ট (Retest) অপশন ---
        st.markdown("---")
        st.markdown("### 📂 সংরক্ষিত মডেল টেস্টের ইতিহাস ও রিটেস্ট (History & Retest)")

        conn = sqlite3.connect("smart_school_syllabus.db")
        cursor = conn.cursor()
        cursor.execute("SELECT id, test_title, test_date, total_marks, allocated_time, obtained_score, correct_count, wrong_count, details_json FROM model_test_history ORDER BY id DESC")
        past_tests = cursor.fetchall()
        conn.close()

        if past_tests:
            for pt in past_tests:
                t_id, t_title, t_date, t_total, t_time, t_score, t_correct, t_wrong, t_json = pt
                
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
                        if st.button(f"🔄 পুনরায় এই টেস্টটি দিন (Retest)", key=f"retest_btn_{t_id}", use_container_width=True, type="primary"):
                            st.session_state.exam_questions = json.loads(t_json)
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
                        if st.button(f"🗑️ হিস্ট্রি ডিলেট করুন", key=f"del_history_btn_{t_id}", use_container_width=True):
                            conn = sqlite3.connect("smart_school_syllabus.db")
                            cursor = conn.cursor()
                            cursor.execute("DELETE FROM model_test_history WHERE id = ?", (t_id,))
                            conn.commit()
                            conn.close()
                            st.success("হিস্ট্রি সফলভাবে মুছে ফেলা হয়েছে!")
                            st.rerun()
        else:
            st.info("💡 এখনও কোনো মডেল টেস্ট দেওয়া হয়নি। উপরে কাস্টম কনফিগার করে আপনার প্রথম মডেল টেস্ট শুরু করুন।")


   
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


