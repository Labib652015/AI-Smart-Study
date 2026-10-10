import streamlit as st
from datetime import datetime
import base64
import os
import requests

# Configure Streamlit page settings
st.set_page_config(
    page_title="AI Smart Study & Assistant",
    page_icon="📚",
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
    st.markdown("<br><br>", unsafe_allow_html=True)
    col1, col2, col3 = st.columns([1, 1.2, 1])
    
    with col2:
        st.markdown("""
        <div style="text-align: center; margin-bottom: 24px;">
            <div style="display: inline-flex; background: #2563EB; color: white; padding: 16px; border-radius: 20px; font-size: 32px; box-shadow: 0 10px 25px rgba(37,99,235,0.3);">📚</div>
            <h1 style="color: #0F172A; font-weight: 800; margin-top: 12px; font-size: 28px;">AI Smart Study</h1>
            <p style="color: #64748B; font-size: 14px;">Learn Smarter • Grow Faster</p>
        </div>
        """, unsafe_allow_html=True)
        
        with st.form("login_form"):
            st.markdown("### Student Portal Login")
            school = st.text_input("School Name & Location", value=st.session_state.school_name)
            name = st.text_input("Student Name", value=st.session_state.student_name)
            branch_class = st.selectbox("Branch / Class", ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"], index=1)
            roll = st.text_input("Roll Number", value=st.session_state.roll_number)
            
            submit = st.form_submit_button("🚀 Enter Dashboard", use_container_width=True)
            if submit:
                if name.strip():
                    st.session_state.school_name = school
                    st.session_state.student_name = name
                    st.session_state.class_name = branch_class
                    st.session_state.roll_number = roll
                    st.session_state.authenticated = True
                    st.rerun()
                else:
                    st.error("Please enter your name to proceed.")

def render_dashboard():

    # Top Header Bar (Updated & Interactive)
    head_col1, head_col2, head_col3 = st.columns([2.5, 3, 1.5])
    
    with head_col1:
        st.markdown(f"""
        <div style="display: flex; align-items: center; gap: 12px;">
            <div style="background: #2563EB; color: white; padding: 10px; border-radius: 12px; font-size: 20px; box-shadow: 0 4px 12px rgba(37,99,235,0.2);">📖</div>
            <div>
                <h2 style="margin: 0; font-size: 22px; color: #0F172A; font-weight: 800;">AI Smart Study</h2>
                <p style="margin: 0; font-size: 11px; color: #64748B;">Learn Smarter &bull; Grow Faster</p>
            </div>
        </div>
        """, unsafe_allow_html=True)
        
    with head_col2:
        import os
        
        # রিয়েল-টাইম স্মার্ট সার্চ ইনপুট ফিল্ড (যেকোনো কিওয়ার্ডের জন্য)
        search_query = st.text_input("Search", placeholder="🔍 Search books, notes, exams, routines...", label_visibility="collapsed")
        
        if search_query:
            q = search_query.lower()
            
            # ১. অ্যাপের অন্যান্য ডাটাবেজ বা সেকশন থেকে ম্যাচিং খোঁজার তালিকা
            app_database = [
                {"title": "Mathematics - Chapter 3: Linear Equations", "type": "Chapter/Book", "desc": "Current reading chapter on math equations."},
                {"title": "Bangla 1st Paper - Notes & Kobita", "type": "Notes", "desc": "Important notes and summaries."},
                {"title": "English Grammar & Composition", "type": "Book", "desc": "Comprehensive English guide for Class 7."},
                {"title": "Upcoming Exam: Mathematics (15 Oct 2025)", "type": "Exam", "desc": "12 Days Left for your next exam."},
                {"title": "Upcoming Exam: Science (18 Oct 2025)", "type": "Exam", "desc": "15 Days Left for science exam."},
                {"title": "Daily Routine: Mathematics & Science", "type": "Routine", "desc": "04:00 PM - 06:00 PM study schedule."},
                {"title": "Study Progress: Bangla (80%) & Math (75%)", "type": "Progress", "desc": "Overall academic performance and mastery."}
            ]
            
            # টেক্সট ডাটা ফিল্টার করা
            matched_items = [item for item in app_database if q in item["title"].lower() or q in item["desc"].lower() or q in item["type"].lower()]
            
            # ২. assets ফোল্ডার থেকে পিডিএফ ফাইলগুলো খোঁজা
            assets_dir = "assets"
            pdf_files = []
            if os.path.exists(assets_dir):
                try:
                    pdf_files = [f for f in os.listdir(assets_dir) if f.lower().endswith('.pdf')]
                except:
                    pdf_files = []
            
            matched_pdfs = [f for f in pdf_files if q in f.lower()]
            
            # ফলাফল প্রদর্শন
            total_found = len(matched_items) + len(matched_pdfs)
            
            if total_found > 0:
                st.markdown(f"<div style='background: #1E293B; color: white; padding: 10px; border-radius: 8px; font-size: 12px; margin-top: 5px;'><b>🔍 Search Results ({total_found} found):</b></div>", unsafe_allow_html=True)
                
                # পিডিএফ ফাইলগুলোর ফলাফল ও ডাউনলোড বাটন দেখানো
                for idx, file_name in enumerate(matched_pdfs):
                    file_path = os.path.join(assets_dir, file_name)
                    clean_title = file_name.replace('.pdf', '').replace('.PDF', '').replace('_', ' ')
                    
                    st.markdown(f"""
                    <div style='background: #F1F5F9; padding: 10px; border-radius: 6px; margin-top: 6px; font-size: 12px; color: #0F172A; border-left: 4px solid #2563EB;'>
                        📚 <b>[PDF Library] {clean_title}</b>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    try:
                        with open(file_path, "rb") as pdf_file:
                            st.download_button(
                                label=f"📥 Download / Read: {clean_title}",
                                data=pdf_file,
                                file_name=file_name,
                                mime="application/pdf",
                                key=f"global_dl_{idx}"
                            )
                    except Exception as err:
                        st.error(f"⚠️ ফাইল ওপেন করতে সমস্যা হয়েছে: {err}")
                        
                # অ্যাপের অন্যান্য তথ্য (নোট, রুটিন, এক্সাম ইত্যাদি) দেখানো
                for idx, item in enumerate(matched_items):
                    st.markdown(f"""
                    <div style='background: #F8FAFC; padding: 10px; border-radius: 6px; margin-top: 6px; font-size: 12px; color: #0F172A; border-left: 4px solid #10B981;'>
                        📌 <b>{item['title']}</b> <span style='color: #10B981; font-weight: bold;'>[{item['type']}]</span><br>
                        <span style='color: #64748B;'>{item['desc']}</span>
                    </div>
                    """, unsafe_allow_html=True)
            else:
                st.warning(f"No results found for '{search_query}'. Try searching 'bangla', 'exam', or 'routine'.")
        else:
            st.info("💡 Tip: Type any keyword (e.g., 'math', 'exam', 'bangla') to search everything across the app.")
            
    with head_col3:
        st.markdown(f"""
        <div style="display: flex; align-items: center; justify-content: flex-end; gap: 12px;">
            <div style="background: #F1F5F9; padding: 8px 12px; border-radius: 50%; font-size: 16px; position: relative; cursor: pointer;" title="Notifications">
                🔔<span style="position: absolute; top: -2px; right: -2px; background: #EF4444; color: white; font-size: 9px; width: 16px; height: 16px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold;">3</span>
            </div>
            <div style="display: flex; align-items: center; gap: 8px; background: #F8FAFC; padding: 6px 12px; border-radius: 20px; border: 1px solid #E2E8F0;">
                <div style="background: #3B82F6; color: white; width: 28px; height: 28px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 12px;">{st.session_state.student_name[0]}</div>
                <div style="text-align: left;">
                    <div style="font-size: 12px; font-weight: 600; color: #1E293B;">{st.session_state.student_name}</div>
                    <div style="font-size: 10px; color: #64748B;">{st.session_state.class_name}</div>
                </div>
            </div>
        </div>
        """, unsafe_allow_html=True)

    # Main Grid Layout: Left Column (Content) & Right Column (AI Assistant Panel)
    col_main, col_ai = st.columns([2.3, 1.2], gap="large")

    with col_main:

# রিয়েল-টাইম সময় বের করার জন্য পাইথনের datetime ব্যবহার
        from datetime import datetime
        current_hour = datetime.now().hour
        
        # সময় অনুযায়ী সঠিক গ্রিটিং নির্ধারণ
        if 5 <= current_hour < 12:
            greeting = "Good Morning"
        elif 12 <= current_hour < 17:
            greeting = "Good Afternoon"
        elif 17 <= current_hour < 21:
            greeting = "Good Evening"
        else:
            greeting = "Good Night"

        # Hero Greeting Banner (Real-Time Dynamic)
        st.markdown(f"""
        <div style="background: linear-gradient(135deg, #EEF2FF 0%, #F3E8FF 100%); padding: 25px; border-radius: 20px; border: 1px solid #E0E7FF; position: relative; margin-bottom: 24px; box-shadow: 0 4px 15px rgba(0,0,0,0.02);">
            <div style="max-width: 100%;">
                <h2 style="color: #0F172A; font-weight: 800; margin-bottom: 4px; font-size: 24px;">{greeting}, {st.session_state.student_name}! 👋</h2>
                <p style="color: #475569; font-size: 13px; margin-bottom: 18px;">Keep going! You are doing great in your studies today.</p>
                <div style="display: flex; gap: 12px; flex-wrap: wrap;">
                    <div style="background: white; padding: 12px 16px; border-radius: 12px; border: 1px solid #E2E8F0; display: flex; align-items: center; gap: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.02);">
                        <span style="font-size: 20px;">🎯</span>
                        <div>
                            <div style="font-size: 11px; color: #64748B; font-weight: 500;">Today's Study Goal</div>
                            <div style="font-size: 14px; font-weight: 700; color: #0F172A;">2.5 hours</div>
                        </div>
                    </div>
                    <div style="background: white; padding: 12px 16px; border-radius: 12px; border: 1px solid #E2E8F0; display: flex; align-items: center; gap: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.02);">
                        <span style="font-size: 20px;">🟢</span>
                        <div>
                            <div style="font-size: 11px; color: #64748B; font-weight: 500;">Today's Progress</div>
                            <div style="font-size: 14px; font-weight: 700; color: #0F172A;">72%</div>
                        </div>
                    </div>
                    <div style="background: white; padding: 12px 16px; border-radius: 12px; border: 1px solid #E2E8F0; display: flex; align-items: center; gap: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.02);">
                        <span style="font-size: 20px;">⏱️</span>
                        <div>
                            <div style="font-size: 11px; color: #64748B; font-weight: 500;">Study Time Today</div>
                            <div style="font-size: 14px; font-weight: 700; color: #0F172A;">1h 30m</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        """, unsafe_allow_html=True)
        
        # Continue Reading Section
        st.markdown("""
        <div class="dashboard-card">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;">
                <div>
                    <h4 style="margin: 0; color: #0F172A; font-weight: 700;">📖 Continue Reading</h4>
                    <p style="margin: 0; font-size: 12px; color: #64748B;">You were reading: Mathematics - Chapter 3</p>
                </div>
            </div>
            <div style="display: flex; gap: 20px; align-items: center; background: #F8FAFC; padding: 16px; border-radius: 12px; border: 1px solid #E2E8F0;">
                <div style="background: #2563EB; color: white; padding: 20px; border-radius: 12px; text-align: center; min-width: 100px;">
                    <div style="font-size: 16px; font-weight: 700;">গণিত</div>
                    <div style="font-size: 10px; opacity: 0.9;">শ্রেণি ৭</div>
                </div>
                <div style="flex-grow: 1;">
                    <div style="font-weight: 600; color: #1E293B; font-size: 15px;">Mathematics</div>
                    <div style="font-size: 12px; color: #64748B; margin-bottom: 8px;">Chapter 3: Linear Equations</div>
                    <div style="background: #E2E8F0; border-radius: 99px; height: 8px; width: 100%; overflow: hidden;">
                        <div style="background: #2563EB; width: 60%; height: 100%;"></div>
                    </div>
                </div>
                <div style="text-align: right;">
                    <div style="font-size: 12px; font-weight: 600; color: #2563EB; margin-bottom: 8px;">60%</div>
                </div>
            </div>
            <div style="display: flex; gap: 10px; margin-top: 14px;">
                <button style="background: #2563EB; color: white; border: none; padding: 8px 16px; border-radius: 8px; font-size: 12px; font-weight: 600; cursor: pointer;">Continue</button>
                <button style="background: white; color: #475569; border: 1px solid #CBD5E1; padding: 8px 16px; border-radius: 8px; font-size: 12px; font-weight: 600; cursor: pointer;">Read From Start</button>
                <button style="background: white; color: #475569; border: 1px solid #CBD5E1; padding: 8px 16px; border-radius: 8px; font-size: 12px; font-weight: 600; cursor: pointer;">Download PDF</button>
            </div>
        </div>
        """, unsafe_allow_html=True)

        # Quick Feature Cards Grid
        qc1, qc2, qc3, qc4, qc5 = st.columns(5)
        with qc1:
            if st.button("📚 My Books\n(Classes 6-10)", use_container_width=True):
                st.session_state.current_page = "My Books"
                st.rerun()
        with qc2:
            if st.button("🤖 Ask AI\nGet instant help", use_container_width=True):
                st.session_state.current_page = "AI Assistant"
                st.rerun()
        with qc3:
            if st.button("🎯 Take Quiz\nTest knowledge", use_container_width=True):
                st.session_state.current_page = "Quiz & Practice"
                st.rerun()
        with qc4:
            if st.button("📅 Study Plan\nYour AI Plan", use_container_width=True):
                st.session_state.current_page = "Study Planner"
                st.rerun()
        with qc5:
            if st.button("📈 Progress\nSee growth", use_container_width=True):
                st.session_state.current_page = "Progress"
                st.rerun()

        # Upcoming Exams & Study Progress Section
        col_sub1, col_sub2 = st.columns(2)
        with col_sub1:
            st.markdown("""
            <div class="dashboard-card">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;">
                    <h4 style="margin: 0; color: #0F172A; font-weight: 700;">📅 Upcoming Exams</h4>
                    <span style="font-size: 12px; color: #2563EB; font-weight: 600; cursor: pointer;">View All</span>
                </div>
                <div style="font-size: 12px; color: #475569;">
                    <div style="display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #F1F5F9;">
                        <span style="font-weight: 600;">Mathematics</span><span>15 Oct 2025</span><span style="background: #FEE2E2; color: #DC2626; padding: 2px 8px; border-radius: 6px; font-size: 10px;">12 Days Left</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #F1F5F9;">
                        <span style="font-weight: 600;">Science</span><span>18 Oct 2025</span><span style="background: #FEF3C7; color: #D97706; padding: 2px 8px; border-radius: 6px; font-size: 10px;">15 Days Left</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #F1F5F9;">
                        <span style="font-weight: 600;">English</span><span>25 Oct 2025</span><span style="background: #E0E7FF; color: #4F46E5; padding: 2px 8px; border-radius: 6px; font-size: 10px;">22 Days Left</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; padding: 8px 0;">
                        <span style="font-weight: 600;">ICT</span><span>30 Oct 2025</span><span style="background: #E0E7FF; color: #4F46E5; padding: 2px 8px; border-radius: 6px; font-size: 10px;">27 Days Left</span>
                    </div>
                </div>
            </div>
            """, unsafe_allow_html=True)

        with col_sub2:
            st.markdown("""
            <div class="dashboard-card">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px;">
                    <h4 style="margin: 0; color: #0F172A; font-weight: 700;">📈 Study Progress</h4>
                </div>
                <div style="font-size: 12px; color: #475569;">
                    <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span>Bangla</span><span style="font-weight: 600;">80%</span></div>
                    <div style="background: #E2E8F0; height: 6px; border-radius: 99px; margin-bottom: 10px;"><div style="background: #10B981; width: 80%; height: 100%;"></div></div>
                    
                    <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span>English</span><span style="font-weight: 600;">65%</span></div>
                    <div style="background: #E2E8F0; height: 6px; border-radius: 99px; margin-bottom: 10px;"><div style="background: #3B82F6; width: 65%; height: 100%;"></div></div>
                    
                    <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span>Mathematics</span><span style="font-weight: 600;">75%</span></div>
                    <div style="background: #E2E8F0; height: 6px; border-radius: 99px; margin-bottom: 10px;"><div style="background: #F97316; width: 75%; height: 100%;"></div></div>
                    
                    <div style="margin-bottom: 6px; display: flex; justify-content: space-between;"><span>Science</span><span style="font-weight: 600;">70%</span></div>
                    <div style="background: #E2E8F0; height: 6px; border-radius: 99px; margin-bottom: 10px;"><div style="background: #8B5CF6; width: 70%; height: 100%;"></div></div>
                </div>
            </div>
            """, unsafe_allow_html=True)

    with col_ai:
        # AI Study Assistant Chat & Task Widget Column
        st.markdown("""
        <div class="dashboard-card" style="border: 2px solid #3B82F6;">
            <div style="display: flex; align-items: center; gap: 10px; margin-bottom: 12px;">
                <div style="background: #3B82F6; color: white; padding: 8px; border-radius: 50%; font-size: 16px;">🤖</div>
                <div>
                    <h4 style="margin: 0; color: #0F172A; font-weight: 700;">AI Study Assistant</h4>
                    <p style="margin: 0; font-size: 11px; color: #10B981;">● Online</p>
                </div>
            </div>
            <div style="background: #F8FAFC; padding: 12px; border-radius: 10px; font-size: 13px; color: #334155; margin-bottom: 12px;">
                Hi Lalon! 👋<br>I'm your AI Study Assistant.<br>Ask me anything about your studies!
            </div>
        </div>
        """, unsafe_allow_html=True)

        # Quick Prompt Buttons
        st.markdown("<p style='font-size: 13px; font-weight: 600; color: #475569;'>Quick Tools:</p>", unsafe_allow_html=True)
        col_p1, col_p2 = st.columns(2)
        with col_p1:
            if st.button("💡 Explain a topic", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Explain a topic"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "Sure! Which subject or topic would you like me to explain for Class 7?"})
        with col_p2:
            if st.button("📐 Solve a problem", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Solve a problem"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "Please type or paste your math/science problem here and I'll solve it step-by-step."})
        
        col_p3, col_p4 = st.columns(2)
        with col_p3:
            if st.button("📝 Summarize", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Summarize a chapter"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "Send me the text or chapter name you want summarized."})
        with col_p4:
            if st.button("🌐 Translate", use_container_width=True):
                st.session_state.chat_messages.append({"role": "user", "content": "Translate text"})
                st.session_state.chat_messages.append({"role": "assistant", "content": "I can translate between English and Bengali instantly. What would you like translated?"})

        # Interactive Chat Box
        st.markdown("<br>", unsafe_allow_html=True)
        for msg in st.session_state.chat_messages[-4:]:
            bg = "#EFF6FF" if msg["role"] == "user" else "#F1F5F9"
            align = "right" if msg["role"] == "user" else "left"
            st.markdown(f"""
            <div style="background: {bg}; padding: 10px 14px; border-radius: 12px; margin-bottom: 8px; font-size: 13px; text-align: {align};">
                <b>{ 'You' if msg['role']=='user' else 'AI Assistant' }:</b> {msg['content']}
            </div>
            """, unsafe_allow_html=True)

        user_input = st.text_input("Ask AI Question", placeholder="Type your question here...", label_visibility="collapsed")
        if st.button("Send Message 🚀", use_container_width=True):
            if user_input:
                st.session_state.chat_messages.append({"role": "user", "content": user_input})
                st.session_state.chat_messages.append({"role": "assistant", "content": f"That's a great question regarding '{user_input}'. As your AI tutor for Class 7, I recommend reviewing Chapter notes or practicing related exercises."})
                st.rerun()

        # Recent Notifications
        st.markdown("""
        <div class="dashboard-card" style="margin-top: 20px;">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
                <h4 style="margin: 0; font-size: 14px; font-weight: 700; color: #0F172A;">🔔 Recent Notifications</h4>
                <span style="font-size: 11px; color: #2563EB; font-weight: 600; cursor: pointer;">View All</span>
            </div>
            <div style="font-size: 12px; color: #475569;">
                <div style="padding: 6px 0; border-bottom: 1px solid #F1F5F9;">⚠️ Tomorrow is your Mathematics exam. <br><span style="font-size: 10px; color: #94A3B8;">2 hours ago</span></div>
                <div style="padding: 6px 0; border-bottom: 1px solid #F1F5F9;">📚 You have 2 unfinished chapters.<br><span style="font-size: 10px; color: #94A3B8;">4 hours ago</span></div>
                <div style="padding: 6px 0;">🎉 Weekly study goal almost complete!<br><span style="font-size: 10px; color: #94A3B8;">6 hours ago</span></div>
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
            return "⚠️ ⚠️ API Key সমস্যা হচ্ছে। দয়া করে'Gemini Chatboot'বাটনে ক্লিক করো।"
            
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
                
        return "⚠️ API Key সমস্যা হচ্ছে। দয়া করে'Gemini Chatboot'বাটনে ক্লিক করো।"

    # ২. কাস্টম সিএসএস
    st.markdown("""
    <style>
        .my-books-dark-container {
            background-color: #0F172A;
            padding: 25px;
            border-radius: 20px;
            border: 1px solid #334155;
            box-shadow: 0 10px 25px rgba(0,0,0,0.4);
        }
        .dark-gemini-header {
            color: #F8FAFC !important;
            font-weight: 800;
            font-size: 26px;
        }
        .dark-gemini-subtext {
            color: #94A3B8 !important;
            font-size: 14px;
            margin-bottom: 20px;
        }
        .book-card {
            background: #1E293B;
            padding: 16px;
            border-radius: 12px;
            border: 1px solid #334155;
            border-top: 4px solid #38BDF8;
            margin-bottom: 12px;
        }
    </style>
    """, unsafe_allow_html=True)



    st.markdown("<div class='my-books-dark-container'>", unsafe_allow_html=True)
    st.markdown("<h2 class='dark-gemini-header'>📚 Library & E-Book Management</h2>", unsafe_allow_html=True)
    st.markdown("<p class='dark-gemini-subtext'> তোমার পছন্দের বই পড়, গুরুত্বপূর্ণ লাইনগুলো হাইলাইট কর। কোন কিছু বুঝতে না পারলে কপি কর এবং পাশে Gemini AI থেকে উত্তর নাও।</p>", unsafe_allow_html=True)

    # ৩. সেশন স্টেট
    if "active_pdf_path" not in st.session_state:
        st.session_state.active_pdf_path = None
    if "book_chat_messages" not in st.session_state:
        st.session_state.book_chat_messages = [
            {"role": "assistant", "content": " কপি করে নিচে পেস্ট কর অথবা সরাসরি Gemini Chatboot-এ চলে যাও।"}
        ]
    if "rename_file" not in st.session_state:
        st.session_state.rename_file = None

    # ৪. ফাইল সেভ করার ফোল্ডার সেটআপ
    current_dir = os.path.dirname(os.path.abspath(__file__)) if "__file__" in locals() else os.getcwd()
    assets_dir = os.path.join(current_dir, "assets")
    if not os.path.exists(assets_dir):
        os.makedirs(assets_dir)

# ৫. ক্যাটাগরি সিলেকশন (NCTB Textbooks, Guide Books, Other Books)
    col_cat, col_up = st.columns([2.5, 1.5])
    with col_cat:
        book_category = st.radio(
            "Select Category",
            ["📖 NCTB Textbooks", "📘 Guide Books", "📚 Other Books"],
            horizontal=True
        )
        
    with col_up:
        # ইউজার যে ক্যাটাগরিতে থাকবে, ঠিক সেই ক্যাটাগরি অনুযায়ী আপলোড অপশন দেখাবে
        if "Guide Books" in book_category:
            uploaded_guide = st.file_uploader("Upload Guide Book PDF", type=["pdf"], key="upload_guide_file")
            if uploaded_guide is not None:
                # গাইড বইয়ের জন্য 'guide_' প্রিফিক্স দিয়ে সেভ হবে
                save_name = f"guide_{uploaded_guide.name}"
                save_path = os.path.join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_guide.getbuffer())
                st.success(f"✅ Guide Uploaded: {uploaded_guide.name}")
                st.rerun()
                
        elif "Other Books" in book_category:
            uploaded_other = st.file_uploader("Upload Other Book PDF", type=["pdf"], key="upload_other_file")
            if uploaded_other is not None:
                # অন্যান্য বইয়ের জন্য 'other_' প্রিফিক্স দিয়ে সেভ হবে
                save_name = f"other_{uploaded_other.name}"
                save_path = os.path.join(assets_dir, save_name)
                with open(save_path, "wb") as f:
                    f.write(uploaded_other.getbuffer())
                st.success(f"✅ Other Book Uploaded: {uploaded_guide.name}" if 'uploaded_guide' in locals() else f"✅ Other Book Uploaded: {uploaded_other.name}")
                st.rerun()

    st.markdown("<hr style='margin: 15px 0; border: 0; border-top: 1px solid #334155;'>", unsafe_allow_html=True)

    # ৬. ক্যাটাগরি অনুযায়ী বই ফিল্টার ও প্রদর্শন লজিক
    if "NCTB Textbooks" in book_category:
        st.markdown("<h3 style='color: #38BDF8; font-size: 18px;'>🎒 Select Class Level (NCTB Curriculum)</h3>", unsafe_allow_html=True)
        selected_class = st.selectbox(
            "Choose Class",
            ["Class 6", "Class 7", "Class 8", "Class 9", "Class 10"]
        )
        cls_prefix = selected_class.lower().replace(" ", "")
        
        try:
            all_files = os.listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and cls_prefix in f.lower()]
        except:
            pdf_files = []
            
        st.subheader(f"📑 Available Books for {selected_class}")
        if pdf_files:
            grid_cols = st.columns(3)
            for idx, file_name in enumerate(pdf_files):
                file_path = os.path.join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                clean_title = file_name.replace(cls_prefix, '').replace('.pdf', '').replace('_', ' ').strip()
                if not clean_title:
                    clean_title = file_name.replace('.pdf', '')
                    
                with col:
                    st.markdown(f"""
                    <div class="book-card">
                        <h4 style="color:#F8FAFC; margin:0 0 10px 0; font-size:14px; text-overflow: ellipsis; overflow: hidden; white-space: nowrap;">📖 {selected_class} - {clean_title}</h4>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    if st.button("📖 Read", key=f"read_nctb_{cls_prefix}_{idx}", use_container_width=True):
                        st.session_state.active_pdf_path = file_path
                        st.rerun()
        else:
            st.info(f"💡 '{selected_class}' এর জন্য ফোল্ডারে কোনো বই পাওয়া যায়নি।")
            
    elif "Guide Books" in book_category:
        # শুধু গাইড বইগুলো ফিল্টার করবে (যাদের নামের শুরুতে 'guide_' আছে)
        try:
            all_files = os.listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('guide_')]
        except:
            pdf_files = []

        if st.session_state.rename_file:
            old_file = st.session_state.rename_file
            st.warning(f"Renaming: {old_file}")
            new_name = st.text_input("Enter new file name (with .pdf):", value=old_file)
            c_save, c_cancel = st.columns([1, 1])
            with c_save:
                if st.button("💾 Save Name"):
                    if new_name and new_name != old_file:
                        os.rename(os.path.join(assets_dir, old_file), os.path.join(assets_dir, new_name))
                        st.session_state.rename_file = None
                        st.success("Renamed successfully!")
                        st.rerun()
            with c_cancel:
                if st.button("Cancel"):
                    st.session_state.rename_file = None
                    st.rerun()

        st.subheader("📑 Uploaded Guide Books")
        if pdf_files:
            grid_cols = st.columns(3)
            for idx, file_name in enumerate(pdf_files):
                file_path = os.path.join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace('guide_', '')
                with col:
                    st.markdown(f"""
                    <div class="book-card">
                        <h4 style="color:#F8FAFC; margin:0 0 10px 0; font-size:14px; text-overflow: ellipsis; overflow: hidden; white-space: nowrap;">📘 {display_name}</h4>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    b1, b2, b3 = st.columns([1.2, 1, 1])
                    with b1:
                        if st.button("📖 Read", key=f"read_guide_{idx}"):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("✏️ Rename", key=f"ren_guide_{idx}"):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("🗑️ Delete", key=f"del_guide_{idx}"):
                            os.remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("Deleted!")
                            st.rerun()
        else:
            st.info("কোনো গাইড বই আপলোড করা হয়নি। উপরে 'Upload Guide Book PDF' থেকে আপলোড করুন।")

    else:
        # শুধু অন্যান্য বইগুলো ফিল্টার করবে (যাদের নামের শুরুতে 'other_' আছে)
        try:
            all_files = os.listdir(assets_dir)
            pdf_files = [f for f in all_files if f.lower().endswith('.pdf') and f.lower().startswith('other_')]
        except:
            pdf_files = []

        if st.session_state.rename_file:
            old_file = st.session_state.rename_file
            st.warning(f"Renaming: {old_file}")
            new_name = st.text_input("Enter new file name (with .pdf):", value=old_file)
            c_save, c_cancel = st.columns([1, 1])
            with c_save:
                if st.button("💾 Save Name"):
                    if new_name and new_name != old_file:
                        os.rename(os.path.join(assets_dir, old_file), os.path.join(assets_dir, new_name))
                        st.session_state.rename_file = None
                        st.success("Renamed successfully!")
                        st.rerun()
            with c_cancel:
                if st.button("Cancel"):
                    st.session_state.rename_file = None
                    st.rerun()

        st.subheader("📑 Uploaded Other Books")
        if pdf_files:
            grid_cols = st.columns(3)
            for idx, file_name in enumerate(pdf_files):
                file_path = os.path.join(assets_dir, file_name)
                col = grid_cols[idx % 3]
                display_name = file_name.replace('other_', '')
                with col:
                    st.markdown(f"""
                    <div class="book-card">
                        <h4 style="color:#F8FAFC; margin:0 0 10px 0; font-size:14px; text-overflow: ellipsis; overflow: hidden; white-space: nowrap;">📚 {display_name}</h4>
                    </div>
                    """, unsafe_allow_html=True)
                    
                    b1, b2, b3 = st.columns([1.2, 1, 1])
                    with b1:
                        if st.button("📖 Read", key=f"read_other_{idx}"):
                            st.session_state.active_pdf_path = file_path
                            st.rerun()
                    with b2:
                        if st.button("✏️ Rename", key=f"ren_other_{idx}"):
                            st.session_state.rename_file = file_name
                            st.rerun()
                    with b3:
                        if st.button("🗑️ Delete", key=f"del_other_{idx}"):
                            os.remove(file_path)
                            if st.session_state.active_pdf_path == file_path:
                                st.session_state.active_pdf_path = None
                            st.success("Deleted!")
                            st.rerun()
        else:
            st.info("কোনো অন্যান্য বই আপলোড করা হয়নি। উপরে 'Upload Other Book PDF' থেকে আপলোড করুন।")


    # ৭. ফুল-ফিচার্ড পিডিএফ রিডার উইন্ডো (টুলবার, হাইলাইটার, প্রিন্ট, পেজ জাম্পসহ) এবং ডানপাশে Gemini AI
    if st.session_state.active_pdf_path and os.path.exists(st.session_state.active_pdf_path):
        st.markdown("<hr style='margin: 25px 0; border: 0; border-top: 1px solid #38BDF8;'>", unsafe_allow_html=True)
        
        c_title, c_close = st.columns([4, 1])
        with c_title:
            active_file_name = os.path.basename(st.session_state.active_pdf_path)
            st.markdown(f"<h3 style='color: #38BDF8; margin:0;'>📖 Reading: {active_file_name}</h3>", unsafe_allow_html=True)
        with c_close:
            if st.button("❌ Close Reader", use_container_width=True):
                st.session_state.active_pdf_path = None
                st.rerun()

        pdf_col, ai_col = st.columns([2.3, 1.1], gap="medium")

  # ১. বাম পাশে ফুল-ফিচার ব্রাউজার নেটিভ পিডিএফ ভিউয়ার
        with pdf_col:
            try:
                # ক্যাশ করা ফাংশন ব্যবহার করে খুব দ্রুত Base64 ডেটা লোড করা
                base64_pdf = get_cached_pdf_base64(st.session_state.active_pdf_path)
                
                pdf_embed_html = f'''
                <iframe 
                    src="data:application/pdf;base64,{base64_pdf}#toolbar=1&navpanes=1"
                    width="100%" 
                    height="780px" 
                    type="application/pdf"
                    style="border: 1px solid #334155; border-radius: 12px; background-color: #1E293B;"
                >
                    <p>আপনার ব্রাউজার PDF প্রদর্শন করতে পারছে না।</p>
                </iframe>
                '''
                st.markdown(pdf_embed_html, unsafe_allow_html=True)
            except Exception as e:
                st.error(f"Error loading PDF: {e}")

        # ২. ডানপাশে Gemini AI Assistant Sidebar
        with ai_col:
            st.markdown("""
            <div style="background: #1E293B; padding: 15px; border-radius: 12px; border: 1px solid #334155; margin-bottom: 12px;">
                <h4 style="color:#38BDF8; margin-top:0; margin-bottom:5px;">🤖 Gemini Study Assistant</h4>
                <p style="color:#94A3B8; font-size:12px; margin-bottom:0px;">টেক্সট কপি করে সরাসরি অফিশিয়াল Gemini চ্যাটবটে চলে যাও:</p>
            </div>
            """, unsafe_allow_html=True)

            st.link_button(
                "🚀 Open Gemini Chatboot & Get Your Answer", 
                "https://gemini.google.com", 
                use_container_width=True,
                type="primary"
            )
            
            st.markdown("<p style='text-align:center; color:#64748B; font-size:11px; margin-top:4px;'>কপি করার পর বাটনে ক্লিক করুন ➔ তারপর Ctrl+V দিয়ে পেস্ট করুন</p>", unsafe_allow_html=True)

            st.markdown("<hr style='margin: 12px 0; border-color: #334155;'>", unsafe_allow_html=True)

            chat_container = st.container(height=320)
            with chat_container:
                for msg in st.session_state.book_chat_messages:
                    if msg["role"] == "user":
                        st.chat_message("user").write(msg["content"])
                    else:
                        st.chat_message("assistant").write(msg["content"])

            with st.form(key="ai_chat_form", clear_on_submit=True):
                user_text = st.text_area(
                    "Ask or Paste Question (In-App):",
                    placeholder="কপি করা টেক্সট পেস্ট (Ctrl+V) করুন...",
                    height=70
                )
                submit_button = st.form_submit_button("💬 Ask In-App AI", use_container_width=True)

            if submit_button and user_text.strip():
                st.session_state.book_chat_messages.append({"role": "user", "content": user_text})
                with st.spinner("তোমার উত্তর তৈরি হচ্ছে..."):
                    ai_reply = ask_gemini_rest_api(user_text)
                st.session_state.book_chat_messages.append({"role": "assistant", "content": ai_reply})
                st.rerun()

    st.markdown("</div>", unsafe_allow_html=True)


    


def render_ai_assistant_page():
    st.markdown("## 🤖 AI Study Assistant Hub")
    st.markdown("Your personal 24/7 AI tutor ready to help you solve problems, summarize chapters, and generate practice quizzes.")
    
    query = st.text_area("What do you want to learn or solve today?", placeholder="Type your complex math equation, science question, or essay topic here...")
    col1, col2 = st.columns(2)
    with col1:
        if st.button("✨ Explain Step-by-Step", use_container_width=True):
            if query:
                st.success("AI Explanation:")
                st.write(f"Here is a detailed, easy-to-understand breakdown for: '{query}'. Step 1: Identify key variables. Step 2: Apply formula. Step 3: Conclude result.")
            else:
                st.warning("Please type a topic first.")
    with col2:
        if st.button("📝 Generate Practice Quiz", use_container_width=True):
            if query:
                st.success("Generated 3 Practice Questions:")
                st.markdown("1. What is the fundamental principle behind...?\n2. Calculate the value if x = 5...\n3. True or False...")
            else:
                st.warning("Please enter a subject or topic.")


import streamlit as st

def render_quiz_practice():
    st.markdown("""
    <div style="background: linear-gradient(135deg, #1E3A8A 0%, #3882F6 100%); padding: 25px; border-radius: 12px; color: white; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
        <h2 style="margin: 0; color: white;">📚 Library-Integrated Quiz & Practice Hub</h2>
        <p style="margin: 5px 0 0 0; font-size: 16px; opacity: 0.9;">ভার্চুয়াল লাইব্রেরি থেকে বিষয় নির্বাচন করুন এবং সরাসরি কুইজ ও অনুশীলন করুন!</p>
    </div>
    """, unsafe_allow_html=True)
    
    # প্রগ্রেস সংরক্ষণের জন্য সেশন স্টেট ইনিশিয়ালাইজ করা
    if "user_progress" not in st.session_state:
        st.session_state["user_progress"] = []
        
    # ক্লাস, বিষয়, অধ্যায় এবং নির্দিষ্ট প্রশ্ন সম্বলিত ডিকশনারি
    library_question_bank = {
    "Class 6": {
        "বাংলা (চারুপাঠ)": {
            "অধ্যায় ১: মাদার তেরেসা": [
                {
                    "q": "মাদার তেরেসা কত সালে জন্মগ্রহণ করেন?",
                    "options": ["ক) ১৯১০ সালের ২৬ আগস্ট", "খ) ১৯১২ সালের ২৭ আগস্ট", "গ) ১৯১৪ সালের ২৮ আগস্ট", "ঘ) ১৯১৬ সালের ২৯ আগস্ট"],
                    "answer": "ক) ১৯১০ সালের ২৬ আগস্ট"
                },
                {
                    "q": "মাদার তেরেসার জন্মস্থান কোথায়?",
                    "options": ["ক) ভারত", "খ) যুগোস্লাভিয়ার স্কোপিয়ে", "গ) ফ্রান্স", "ঘ) ইংল্যান্ড"],
                    "answer": "খ) যুগোস্লাভিয়ার স্কোপিয়ে"
                },
                {
                    "q": "মাদার তেরেসার আসল নাম কী ছিল?",
                    "options": ["ক) মার্গারেট", "খ) অ্যাগনেস গোনহেক্সা বোয়াযু", "গ) এলিজাবেথ", "ঘ) ক্যাথরিন"],
                    "answer": "খ) অ্যাগনেস গোনহেক্সা বোয়াযু"
                },
                {
                    "q": "মাদার তেরেসার পারিবারিক পদবি বা মূল নাম অনুযায়ী তার উপাধি কী ছিল?",
                    "options": ["ক) অ্যাগনেস", "খ) গোনহেক্সা", "গ) মাদার", "ঘ) তেরেসা"],
                    "answer": "খ) গোনহেক্সা"
                },
                {
                    "q": "কত বছর বয়সে মাদার তেরেসা ঘরের মায়া ত্যাগ করে নান হওয়ার সিদ্ধান্ত নেন?",
                    "options": ["ক) ১৫ বছর বয়সে", "খ) ১৮ বছর বয়সে", "গ) ২০ বছর বয়সে", "ঘ) ২২ বছর বয়সে"],
                    "answer": "খ) ১৮ বছর বয়সে"
                },
                {
                    "q": "মাদার তেরেসা কোন ধর্মীয় মিশনারিতে যোগ দেন?",
                    "options": ["ক) লরেট সিস্টার্স", "খ) সিস্টার্স অব চ্যারিটি", "গ) সেন্ট মেরি সংঘ", "ঘ) আসিসি সংঘ"],
                    "answer": "ক) লরেট সিস্টার্স"
                },
                {
                    "q": "লরেট সিস্টার্সের প্রধান কার্যালয় কোথায় ছিল?",
                    "options": ["ক) রোমে", "খ) ডাবলিনে", "গ) লন্ডনে", "ঘ) কলকাতায়"],
                    "answer": "খ) ডাবলিনে"
                },
                {
                    "q": "মাদার তেরেসা শিক্ষকতা করার জন্য প্রথম কোন দেশে আসেন?",
                    "options": ["ক) পাকিস্তান", "খ) ভারত", "গ) বাংলাদেশ", "ঘ) মায়ানমার"],
                    "answer": "খ) ভারত"
                },
                {
                    "q": "ভারতের কোন শহরে মাদার তেরেসা লরেট কনভেন্ট স্কুলের শিক্ষিকা হিসেবে যোগ দেন?",
                    "options": ["ক) মুম্বাই", "খ) দিল্লি", "গ) কলকাতা", "ঘ) চেন্নাই"],
                    "answer": "গ) কলকাতা"
                },
                {
                    "q": "কলকাতায় এসে মাদার তেরেসা কোন ভাষা খুব ভালোভাবে আয়ত্ত করেছিলেন?",
                    "options": ["ক) হিন্দি ও ইংরেজি", "খ) বাংলা ও ইংরেজি", "গ) উর্দু ও ফার্সি", "ঘ) মারাঠি ও বাংলা"],
                    "answer": "খ) বাংলা ও ইংরেজি"
                },
                {
                    "q": "মাদার তেরেসা কলকাতার সেন্ট মেরি স্কুলে কত বছর শিক্ষকতা করেন?",
                    "options": ["ক) প্রায় ১০ বছর", "খ) প্রায় ১৫ বছর", "গ) প্রায় ১৭ বছর", "ঘ) প্রায় ২০ বছর"],
                    "answer": "গ) প্রায় ১৭ বছর"
                },
                {
                    "q": "সেন্ট মেরি স্কুলের প্রধান শিক্ষিকা হিসেবে মাদার তেরেসা কত বছর দায়িত্ব পালন করেছিলেন?",
                    "options": ["ক) কয়েক বছর", "খ) অনেক বছর", "গ) নির্দিষ্ট কোনো সময় নেই", "ঘ) কোনো দায়িত্ব পালন করেননি"],
                    "answer": "ক) কয়েক বছর"
                },
                {
                    "q": "কলকাতার কোন ঘটনার পর মাদার তেরেসা কনভেন্ট ছেড়ে সাধারণ মানুষের সেবায় নেমে পড়েন?",
                    "options": ["ক) মহামারী", "খ) ১৯৪৬ সালের ভয়াবহ দাঙ্গা", "গ) বন্যা", "ঘ) দুর্ভিক্ষ"],
                    "answer": "খ) ১৯৪৬ সালের ভয়াবহ দাঙ্গা"
                },
                {
                    "q": "কত সালে মাদার তেরেসা কলকাতার বস্তিবাসীদের সেবার জন্য নতুন সংগঠন গড়ে তোলেন?",
                    "options": ["ক) ১৯৪৮ সালে", "খ) ১৯৫০ সালে", "গ) ১৯৫২ সালে", "ঘ) ১৯৫৫ সালে"],
                    "answer": "খ) ১৯৫০ সালে"
                },
                {
                    "q": "মাদার তেরেসার গড়া সেবামূলক প্রতিষ্ঠানের নাম কী?",
                    "options": ["ক) সেবা সংঘ", "খ) মিশনারিজ অব চ্যারিটি", "গ) চ্যারিটি হোম", "ঘ) নির্মল হৃদয়"],
                    "answer": "খ) মিশনারিজ অব চ্যারিটি"
                }
            ],
            "অধ্যায় ২: চিঠি বিলি": [
                {
                    "q": "'চিঠি বিলি' কবিতার রচয়িতা কে?",
                    "options": ["ক) সুকুমার রায়", "খ) রোকনুজ্জামান খান", "গ) শামসুর রাহমান", "ঘ) জসীমউদ্দীন"],
                    "answer": "খ) রোকনুজ্জামান খান"
                },
                {
                    "q": "রোকনুজ্জামান খান কী নামে বেশি পরিচিত?",
                    "options": ["ক) দাদাভাই", "খ) চাচাভাই", "গ) নানুভাই", "ঘ) খোকাভাই"],
                    "answer": "ক) দাদাভাই"
                },
                {
                    "q": "'চিঠি বিলি' কবিতায় চিঠি বিলি করতে কে ছুটে চলেছে?",
                    "options": ["ক) চিংড়ি মাছ", "খ) ব্যাঙ", "গ) খলসে মাছ", "ঘ) কাতলা মাছ"],
                    "answer": "খ) ব্যাঙ"
                },
                {
                    "q": "চিঠি বিলি করতে যাওয়ার সময় ব্যাঙের মাথায় কী ছিল?",
                    "options": ["ক) টুপি", "খ) ছাতা", "গ) পাতার মুকুট", "ঘ) গামছা"],
                    "answer": "খ) ছাতা"
                },
                {
                    "q": "খেয়ানৌকার মাঝি কে?",
                    "options": ["ক) পুঁটি মাছ", "খ) চিংড়ির বাচ্চা", "গ) খলসে মাছ", "ঘ) ভেটকি মাছ"],
                    "answer": "খ) চিংড়ির বাচ্চা"
                },
                {
                    "q": "খলসে মাছের চোখ ঝলসে যাওয়ার কারণ কী ধরনের আলো?",
                    "options": ["ক) চাঁদের আলো", "খ) সাঁঝের বেলার রোদ", "গ) বিদ্যুতের আলো", "ঘ) আগুনের শিখা"],
                    "answer": "খ) সাঁঝের বেলার রোদ"
                },
                {
                    "q": "চিংড়ির বাচ্চাটিকে 'জবর' বলা হয়েছে কেন?",
                    "options": ["ক) সে খুব দুষ্টু বলে", "খ) সে অত্যন্ত দক্ষ ও কাজের জুরি মেলা ভার বলে", "গ) সে আকারে বড় বলে", "ঘ) সে দেখতে সুন্দর বলে"],
                    "answer": "খ) সে অত্যন্ত দক্ষ ও কাজের জুরি মেলা ভার বলে"
                },
                {
                    "q": "রোকনুজ্জামান খান কত সালে জন্মগ্রহণ করেন?",
                    "options": ["ক) ১৯২৫ সালে", "খ) ১৯২৫ সালের ৯ এপ্রিল", "গ) ১৯৩০ সালে", "ঘ) ১৯৩৫ সালে"],
                    "answer": "খ) ১৯২৫ সালের ৯ এপ্রিল"
                },
                {
                    "q": "রোকনুজ্জামান খান দাদাভাই কোথায় জন্মগ্রহণ করেন?",
                    "options": ["ক) ঢাকা জেলায়", "খ) ফরিদপুর জেলার পাংশায়", "গ) কুষ্টিয়া জেলায়", "ঘ) বরিশাল জেলায়"],
                    "answer": "খ) ফরিদপুর জেলার পাংশায়"
                },
                {
                    "q": "রোকনুজ্জামান খান কোন পত্রিকার কচিকাঁচার মেলার পাতার দায়িত্বে ছিলেন?",
                    "options": ["ক) দৈনিক ইত্তেফাক", "খ) দৈনিক সংবাদ", "গ) দৈনিক আজাদ", "ঘ) দৈনিক প্রগতি"],
                    "answer": "ক) দৈনিক ইত্তেফাক"
                },
                {
                    "q": "শিশুদের সংগঠন 'কচিকাঁচার মেলা' কে প্রতিষ্ঠা করেন?",
                    "options": ["ক) সুকুমার রায়", "খ) রোকনুজ্জামান খান দাদাভাই", "গ) শামসুর রাহমান", "ঘ) কাজী নজরুল ইসলাম"],
                    "answer": "খ) রোকনুজ্জামান খান দাদাভাই"
                },
                {
                    "q": "রোকনুজ্জামান খান দাদাভাই কত সালে মৃত্যুবরণ করেন?",
                    "options": ["ক) ১৯৯০ সালে", "খ) ১৯৯৯ সালের ৩ ডিসেম্বর", "গ) ২০০৫ সালে", "ঘ) ২০১০ সালে"],
                    "answer": "খ) ১৯৯৯ সালের ৩ ডিসেম্বর"
                },
                {
                    "q": "'চিঠি বিলি' কবিতায় মূলত কিসের আনন্দ প্রকাশ পেয়েছে?",
                    "options": ["ক) বর্ষার দিনে কল্পনার রাজ্য ও যোগাযোগের আনন্দ", "খ) শহুরে জীবনের যান্ত্রিকতা", "গ) ব্যবসার প্রসার", "ঘ) কৃষকদের কষ্ট"],
                    "answer": "ক) বর্ষার দিনে কল্পনার রাজ্য ও যোগাযোগের আনন্দ"
                },
                {
                    "q": "বিলের খলসে মাছ চিঠি পাঠিয়েছে কার উদ্দেশ্যে?",
                    "options": ["ক) ব্যাঙের উদ্দেশ্যে", "খ) চিংড়ির উদ্দেশ্যে", "গ) কাতলার উদ্দেশ্যে", "ঘ) ভেটকি মাছের উদ্দেশ্যে"],
                    "answer": "খ) চিংড়ির উদ্দেশ্যে"
                },
                {
                    "q": "চিঠি বিলি করার কাজটি কিসের মধ্য দিয়ে পরিচালিত হয়?",
                    "options": ["ক) বাস্তব জগতের নিয়মে", "খ) কবির কল্পনাজগতের অভিনব খেয়ালে", "গ) মাছের নিজস্ব প্রযুক্তিতে", "ঘ) মানুষের নির্দেশে"],
                    "answer": "খ) কবির কল্পনাজগতের অভিনব খেয়ালে"
                }
            ]
        },
        "গণিত": {
            "অধ্যায় ৫: সরল সমীকরণ": [
                {
                    "q": "সরল সমীকরণে চলকের সর্বোচ্চ ঘাত বা শক্তি কত হয়?",
                    "options": ["ক) ১", "খ) ২", "গ) ৩", "ঘ) ৪"],
                    "answer": "ক) ১"
                },
                {
                    "q": "x + 5 = 10 সমীকরণে x এর মান কত?",
                    "options": ["ক) ৩", "খ) ৪", "গ) ৫", "ঘ) ৬"],
                    "answer": "গ) ৫"
                },
                {
                    "q": "2x = 12 হলে, x এর মান কত?",
                    "options": ["ক) ৪", "খ) ৫", "গ) ৬", "ঘ) ৭"],
                    "answer": "গ) ৬"
                },
                {
                    "q": "কোনো সমীকরণের সমান চিহ্নের বাম দিকের রাশিকে কী বলা হয়?",
                    "options": ["ক) ডানপক্ষ", "খ) বামপক্ষ", "গ) চলক", "ঘ) ধ্রুবক"],
                    "answer": "খ) বামপক্ষ"
                },
                {
                    "q": "সমীকরণের মূল বা বীজ বলতে কী বোঝায়?",
                    "options": ["ক) চলকের মান", "খ) ধ্রুবকের মান", "গ) পক্ষের মান", "ঘ) গুণের মান"],
                    "answer": "ক) চলকের মান"
                },
                {
                    "q": "x - 3 = 7 হলে, x এর মান কত?",
                    "options": ["ক) ৫", "খ) ৮", "গ) ১০", "ঘ) ১২"],
                    "answer": "গ) ১০"
                },
                {
                    "q": "পক্ষান্তর করার সময় পদের কী পরিবর্তন ঘটে?",
                    "options": ["ক) মানের পরিবর্তন হয়", "খ) চিহ্নের পরিবর্তন হয়", "গ) কোনো পরিবর্তন হয় না", "ঘ) পদের অবস্থান অপরিবর্তিত থাকে"],
                    "answer": "খ) চিহ্নের পরিবর্তন হয়"
                },
                {
                    "q": "x / 2 = 4 হলে, x এর মান কত?",
                    "options": ["ক) ২", "খ) ৪", "গ) ৬", "ঘ) ৮"],
                    "answer": "ঘ) ৮"
                },
                {
                    "q": "৫ এর সাথে কোনো সংখ্যা যোগ করলে যোগফল ১২ হয়। সমীকরণটি নিচের কোনটি?",
                    "options": ["ক) x + 5 = 12", "খ) 5x = 12", "গ) x - 5 = 12", "ঘ) 12 + x = 5"],
                    "answer": "ক) x + 5 = 12"
                },
                {
                    "q": "3x - 2 = 10 হলে, 3x এর মান কত?",
                    "options": ["ক) ৮", "খ) ১০", "গ) ১২", "ঘ) ১৪"],
                    "answer": "গ) ১২"
                },
                {
                    "q": "সমীকরণের উভয়পক্ষকে একই সংখ্যা দ্বারা গুণ করলে সমীকরণের মানের কী হয়?",
                    "options": ["ক) পরিবর্তিত হয়", "খ) অপরিবর্তিত থাকে", "গ) অর্ধেক হয়", "ঘ) দ্বিগুণ হয়"],
                    "answer": "খ) অপরিবর্তিত থাকে"
                },
                {
                    "q": "কোন সংখ্যার দ্বিগুণ থেকে ৩ বাদ দিলে ৭ হয়?",
                    "options": ["ক) ৩", "খ) ৪", "গ) ৫", "ঘ) ৬"],
                    "answer": "গ) ৫"
                },
                {
                    "q": "4x + 2 = 18 হলে, x এর মান কত?",
                    "options": ["ক) ৩", "খ) ৪", "গ) ৫", "ঘ) ৬"],
                    "answer": "খ) ৪"
                },
                {
                    "q": "খোলা বাক্যকে কিসে রূপান্তর করা যায়?",
                    "options": ["ক) সমীকরণে", "খ) উপাত্তে", "গ) পরিসরে", "ঘ) গণসংখ্যায়"],
                    "answer": "ক) সমীকরণে"
                },
                {
                    "q": "নিচের কোনটি সরল সমীকরণ?",
                    "options": ["ক) x^2 + x = 5", "খ) 2x + 3 = 9", "গ) xy = 10", "ঘ) x^3 = 8"],
                    "answer": "খ) 2x + 3 = 9"
                }
            ],
            "অধ্যায় ৮: তথ্য ও উপাত্ত": [
                {
                    "q": "উপাত্ত প্রধানত কত প্রকার?",
                    "options": ["ক) ১ প্রকার", "খ) ২ প্রকার", "গ) ৩ প্রকার", "ঘ) ৪ প্রকার"],
                    "answer": "খ) ২ প্রকার"
                },
                {
                    "q": "সরাসরি উৎস থেকে সংগৃহীত উপাত্তকে কী বলা হয়?",
                    "options": ["ক) প্রাথমিক উপাত্ত", "খ) মাধ্যমিক উপাত্ত", "গ) বিন্যস্ত উপাত্ত", "ঘ) অবিন্যস্ত উপাত্ত"],
                    "answer": "ক) প্রাথমিক উপাত্ত"
                },
                {
                    "q": "যে উপাত্তগুলো মানের ক্রমানুসারে সাজানো থাকে না, তাকে কী বলে?",
                    "options": ["ক) বিন্যস্ত উপাত্ত", "খ) অবিন্যস্ত উপাত্ত", "গ) প্রাথমিক উপাত্ত", "ঘ) মাধ্যমিক উপাত্ত"],
                    "answer": "খ) অবিন্যস্ত উপাত্ত"
                },
                {
                    "q": "পরিসর নির্ণয়ের সঠিক সূত্র কোনটি?",
                    "options": ["ক) সর্বোচ্চ মান - সর্বনিম্ন মান", "খ) সর্বোচ্চ মান + সর্বনিম্ন মান", "গ) সর্বোচ্চ মান - সর্বনিম্ন মান + ১", "ঘ) সর্বোচ্চ মান + সর্বনিম্ন মান - ১"],
                    "answer": "গ) সর্বোচ্চ মান - সর্বনিম্ন মান + ১"
                },
                {
                    "q": "কোনো উপাত্তের সর্বোচ্চ মান ২৫ এবং সর্বনিম্ন মান ১০ হলে, পরিসর কত?",
                    "options": ["ক) ১৫", "খ) ১৬", "গ) ১৪", "ঘ) ৩৫"],
                    "answer": "খ) ১৬"
                },
                {
                    "q": "ট্যালি চিহ্ন প্রকাশের সময় ৫ সংখ্যাটি কীভাবে লেখা হয়?",
                    "options": ["ক) পাঁচটি খাড়া দাগ দিয়ে", "খ) চারটি খাড়া ও একটি আড়াআড়ি দাগ দিয়ে", "গ) একটি গোল বৃত্ত দিয়ে", "ঘ) দুটি ক্রস চিহ্ন দিয়ে"],
                    "answer": "খ) চারটি খাড়া ও একটি আড়াআড়ি দাগ দিয়ে"
                },
                {
                    "q": "কোনো উৎস থেকে পূর্বে সংগৃহীত উপাত্তকে কী বলা হয়?",
                    "options": ["ক) প্রাথমিক উপাত্ত", "খ) মাধ্যমিক উপাত্ত", "গ) অবিন্যস্ত উপাত্ত", "ঘ) সারণি উপাত্ত"],
                    "answer": "খ) মাধ্যমিক উপাত্ত"
                },
                {
                    "q": "উপাত্তগুলোকে মানের ঊর্ধ্বক্রম বা অধঃক্রমে সাজালে কোন উপাত্ত পাওয়া যায়?",
                    "options": ["ক) অবিন্যস্ত উপাত্ত", "খ) বিন্যস্ত উপাত্ত", "গ) প্রাথমিক উপাত্ত", "ঘ) কাঁচা উপাত্ত"],
                    "answer": "খ) বিন্যস্ত উপাত্ত"
                },
                {
                    "q": "পরিসংখ্যানের মূল আলোচ্য বিষয় কী?",
                    "options": ["ক) বর্ণমালা", "খ) সংখ্যাভিত্তিক তথ্য বা উপাত্ত", "গ) জ্যামিতিক চিত্র", "ঘ) বীজগণিতীয় সূত্র"],
                    "answer": "খ) সংখ্যাভিত্তিক তথ্য বা উপাত্ত"
                },
                {
                    "q": "গণসংখ্যা নিবেশন সারণি তৈরি করতে প্রথমে কী নির্ণয় করতে হয়?",
                    "options": ["ক) গণসংখ্যা", "খ) ট্যালি চিহ্ন", "গ) পরিসর", "ঘ) শ্রেণি ব্যাপ্তি"],
                    "answer": "গ) পরিসর"
                },
                {
                    "q": "কোনো উপাত্তে সুনির্দিষ্ট মান কতবার উপস্থিত আছে তা কোনটি দ্বারা প্রকাশ করা হয়?",
                    "options": ["ক) পরিসর", "খ) গণসংখ্যা", "গ) চলক", "ঘ) ট্যালি"],
                    "answer": "খ) গণসংখ্যা"
                },
                {
                    "q": "নিচের কোনটি প্রাথমিক উপাত্তের উদাহরণ?",
                    "options": ["ক) বই থেকে সংগৃহীত জনসংখ্যার তথ্য", "খ) ইন্টারনেট থেকে নেওয়া আবহাওয়ার প্রতিবেদন", "গ) নিজে সরাসরি শিক্ষার্থীদের ওজন মেপে সংগ্রহ করা তথ্য", "ঘ) পত্রিকা থেকে কাটা নিবন্ধ"],
                    "answer": "গ) নিজে সরাসরি শিক্ষার্থীদের ওজন মেপে সংগ্রহ করা তথ্য"
                },
                {
                    "q": "কাঁচা উপাত্ত বা অসংগঠিত উপাত্তের অপর নাম কী?",
                    "options": ["ক) বিন্যস্ত উপাত্ত", "খ) অবিন্যস্ত উপাত্ত", "গ) মাধ্যমিক উপাত্ত", "ঘ) সারণিবদ্ধ উপাত্ত"],
                    "answer": "খ) অবিন্যস্ত উপাত্ত"
                },
                {
                    "q": "উপাত্তের মানের বিস্তার বা ব্যবধান বোঝাতে কোনটি ব্যবহার করা হয়?",
                    "options": ["ক) পরিসর", "খ) গণসংখ্যা", "গ) ট্যালি", "ঘ) গড়"],
                    "answer": "ক) পরিসর"
                },
                {
                    "q": "একটি উপাত্তের সর্বোচ্চ সংখ্যাটি হলো ৫০ এবং সর্বনিম্ন সংখ্যাটি হলো ২০; এর পরিসর কত?",
                    "options": ["ক) ৩০", "খ) ৩১", "গ) ২৯", "ঘ) ৭০"],
                    "answer": "খ) ৩১"
                }
            ]
        }
    },
    "Class 7": {
        "গণিত": {
            "অধ্যায় ৩": [
                {
                    "q": "শূন্য (০) কোন ধরনের সংখ্যা?",
                    "options": ["ক) ধনাত্মক সংখ্যা", "খ) ঋণাত্মক সংখ্যা", "গ) অঋণাত্মক পূর্ণসংখ্যা", "ঘ) ভগ্নাংশ সংখ্যা"],
                    "answer": "গ) অঋণাত্মক পূর্ণসংখ্যা"
                },
                {
                    "q": "-৫ এর বিপরীত সংখ্যা কোনটি?",
                    "options": ["ক) -৫", "খ) ৫", "গ) ০", "ঘ) ১/৫"],
                    "answer": "খ) ৫"
                },
                {
                    "q": "সংখ্যাতালিকায় মূলবিন্দুর ডান দিকের সংখ্যাগুলো কেমন হয়?",
                    "options": ["ক) ঋণাত্মক", "খ) ধনাত্মক", "গ) শূন্য", "ঘ) সমান"],
                    "answer": "খ) ধনাত্মক"
                },
                {
                    "q": "(+৫) + (-৩) এর মান কত?",
                    "options": ["ক) -৮", "খ) +৮", "গ) +২", "ঘ) -২"],
                    "answer": "গ) +২"
                },
                {
                    "q": "সবচেয়ে ছোট ধনাত্মক পূর্ণসংখ্যা কোনটি?",
                    "options": ["ক) ১", "খ) ০", "গ) -১", "ঘ) অসীম"],
                    "answer": "ক) ১"
                },
                {
                    "q": "যেকোনো ঋণাত্মক সংখ্যা জিরো (০) এর তুলনায় কেমন?",
                    "options": ["ক) বড়", "খ) ছোট", "গ) সমান", "ঘ) দ্বিগুণ"],
                    "answer": "খ) ছোট"
                },
                {
                    "q": "-১০ এবং -৫ এর মধ্যে কোনটি বড়?",
                    "options": ["ক) -১০", "খ) -৫", "গ) উভয় সমান", "ঘ) বলা যায় না"],
                    "answer": "খ) -৫"
                },
                {
                    "q": "+১২ এর পরম মান কত?",
                    "options": ["ক) -১২", "খ) ১২", "গ) ০", "ঘ) ১"],
                    "answer": "খ) ১২"
                },
                {
                    "q": "দুটি বিপরীত সংখ্যার যোগফল সর্বদা কত হয়?",
                    "options": ["ক) ১", "খ) -১", "গ) ০", "ঘ) ২"],
                    "answer": "গ) ০"
                },
                {
                    "q": "সংখ্যাতালিকার বাম দিকে অগ্রসর হলে সংখ্যার মানের কী পরিবর্তন হয়?",
                    "options": ["ক) মান বাড়ে", "খ) মান কমে", "গ) অপরিবর্তিত থাকে", "ঘ) দ্বিগুণ হয়"],
                    "answer": "খ) মান কমে"
                },
                {
                    "q": "(-৩) × (-২) এর গুণফল কত?",
                    "options": ["ক) -৬", "খ) +৬", "গ) -৫", "ঘ) +৫"],
                    "answer": "খ) +৬"
                },
                {
                    "q": "পূর্ণসংখ্যার সেটকে সাধারণত কোন অক্ষর দ্বারা প্রকাশ করা হয়?",
                    "options": ["ক) N", "খ) Z", "গ) Q", "ঘ) R"],
                    "answer": "খ) Z"
                },
                {
                    "q": "-১৫ এর সাথে কত যোগ করলে যোগফল ০ হবে?",
                    "options": ["ক) -১৫", "খ) ১৫", "গ) ০", "ঘ) ১"],
                    "answer": "খ) ১৫"
                },
                {
                    "q": "(-৮) - (-৩) এর মান কত?",
                    "options": ["ক) -৫", "খ) +৫", "গ) -১১", "ঘ) +১১"],
                    "answer": "ক) -৫"
                },
                {
                    "q": "শূন্য থেকে ৫ একক বামে অবস্থান করলে কোন সংখ্যাটি নির্দেশ করে?",
                    "options": ["ক) +৫", "খ) -৫", "গ) ০", "ঘ) ১০"],
                    "answer": "খ) -৫"
                }
            ],
            "অধ্যায় ৭": [
                {
                    "q": "সমীকরণে অজ্ঞাত রাশি বা অক্ষর প্রতীককে কী বলা হয়?",
                    "options": ["ক) ধ্রুবক", "খ) চলক", "গ) গুণক", "ঘ) লব"],
                    "answer": "খ) চলক"
                },
                {
                    "q": "সরল সমীকরণের চলকের সর্বোচ্চ ঘাত বা মাত্রা কত?",
                    "options": ["ক) ১", "খ) ২", "গ) ৩", "ঘ) ৪"],
                    "answer": "ক) ১"
                },
                {
                    "q": "x - ৪ = ৫ সমীকরণে x এর মান কত?",
                    "options": ["ক) ১", "খ) ৯", "গ) ২০", "ঘ) -১"],
                    "answer": "খ) ৯"
                },
                {
                    "q": "পক্ষান্তর করার নিয়ম অনুযায়ী সমীকরণের এক পক্ষ থেকে অন্য পক্ষে পদ স্থানান্তর করলে কী বদলায়?",
                    "options": ["ক) মান বদলায়", "খ) চিহ্ন বদলায়", "গ) কোনো পরিবর্তন হয় না", "ঘ) চলক বদলায়"],
                    "answer": "খ) চিহ্ন বদলায়"
                },
                {
                    "q": "৩x = ১২ হলে, x এর মান কত?",
                    "options": ["ক) ২", "খ) ৩", "গ) ৪", "ঘ) ৬"],
                    "answer": "গ) ৪"
                },
                {
                    "q": "কোনো সংখ্যার দ্বিগুণের সাথে ৫ যোগ করলে ফল ১৫ হয়। সমীকরণটি নিচের কোনটি?",
                    "options": ["ক) 2x - 5 = 15", "খ) 2x + 5 = 15", "গ) 5x + 2 = 15", "ঘ) x + 5 = 15"],
                    "answer": "খ) 2x + 5 = 15"
                },
                {
                    "q": "সমীকরণের উভয়পক্ষকে একই অসত্য সংখ্যা দ্বারা গুণ করলে কী হয়?",
                    "options": ["ক) মান পরিবর্তিত হয়", "খ) সমীকরণ ঠিক থাকে", "গ) সমাধান থাকে না", "ঘ) শূন্য হয়"],
                    "answer": "খ) সমীকরণ ঠিক থাকে"
                },
                {
                    "q": "x / ৫ = ৩ হলে, x এর মান কত?",
                    "options": ["ক) ৮", "খ) ২", "গ) ১৫", "ঘ) ৫/৩"],
                    "answer": "গ) ১৫"
                },
                {
                    "q": "সমীকরণের সমাধান বলতে কী বোঝায়?",
                    "options": ["ক) চলকের মান নির্ণয়", "খ) ধ্রুবক নির্ণয়", "গ) পক্ষের মান নির্ণয়", "ঘ) গুণফল নির্ণয়"],
                    "answer": "ক) চলকের মান নির্ণয়"
                },
                {
                    "q": "2x + 3 = 9 হলে, 2x এর মান কত?",
                    "options": ["ক) ১২", "খ) ৬", "গ) ৩", "ঘ) ১৮"],
                    "answer": "খ) ৬"
                },
                {
                    "q": "দুটি ক্রমিক স্বাভাবিক সংখ্যার যোগফল ১১ হলে, ছোট সংখ্যাটি কত?",
                    "options": ["ক) ৫", "খ) ৬", "গ) ৪", "ঘ) ৭"],
                    "answer": "ক) ৫"
                },
                {
                    "q": "কোনো বিধিবদ্ধ সত্য বা প্রতিজ্ঞাকে কী বলা হয় যা নির্দিষ্ট শর্তে সত্য?",
                    "options": ["ক) অভেদ", "খ) সমীকরণ", "গ) অনুপাত", "ঘ) অসমতা"],
                    "answer": "খ) সমীকরণ"
                },
                {
                    "q": "5x - 3 = 2x + 9 হলে, x এর মান কত?",
                    "options": ["ক) ২", "খ) ৩", "গ) ৪", "ঘ) ৫"],
                    "answer": "গ) ৪"
                },
                {
                    "q": "খালিঘর বা অজ্ঞাত রাশির মান বের করার প্রক্রিয়াকে কী বলে?",
                    "options": ["ক) সরলীকরণ", "খ) সমীকরণ সমাধান", "গ) উৎপাদক বিশ্লেষণ", "ঘ) লসাগু নির্ণয়"],
                    "answer": "খ) সমীকরণ সমাধান"
                },
                {
                    "q": "নিচের কোনটি সরল সমীকরণ?",
                    "options": ["ক) x^2 + 2x = 1", "খ) 3x - 5 = 7", "গ) xy = 12", "ঘ) x^3 = 8"],
                    "answer": "খ) 3x - 5 = 7"
                }
            ]
        },
        "বাংলা": {
            "পিতৃপুরুষের গল্প": [
                {
                    "q": "'পিতৃপুরুষের গল্প' রচনাটির লেখক কে?",
                    "options": ["ক) হারুন হাবীব", "খ) সেলিনা হোসেন", "গ) শওকত ওসমান", "ঘ) রাবেয়া খাতুন"],
                    "answer": "ক) হারুন হাবীব"
                },
                {
                    "q": "'পিতৃপুরুষের গল্প' মূলত কোন পটভূমিতে রচিত?",
                    "options": ["ক) ভাষা আন্দোলন", "খ) মুক্তিযুদ্ধ", "গ) বায়ান্নর একুশে ফেব্রুয়ারি", "ঘ) বাষ্পীয় বিপ্লব"],
                    "answer": "খ) মুক্তিযুদ্ধ"
                },
                {
                    "q": "গল্পে অন্তু কোথায় বেড়াতে আসে?",
                    "options": ["ক) নানাবাড়িতে", "খ) চাচার বাড়ি বা ঢাকায় খোকন চাচার বাসায়", "গ) গ্রামে নানা বাড়িতে", "ঘ) মামাবাড়িতে"],
                    "answer": "খ) চাচার বাড়ি বা ঢাকায় খোকন চাচার বাসায়"
                },
                {
                    "q": "অন্তু কোথায় বসবাস করত?",
                    "options": ["ক) ঢাকা শহরে", "খ) বিদেশে বা অন্য শহরে (সাধারণত প্রবাস বা বিদেশ)", "গ) গ্রামে", "ঘ) বন্দর নগরীতে"],
                    "answer": "খ) বিদেশে বা অন্য শহরে (সাধারণত প্রবাস বা বিদেশ)"
                },
                {
                    "q": "খোকন চাচা পেশায় কী ছিলেন?",
                    "options": ["ক) শিক্ষক", "খ) সাংবাদিক ও সাহিত্যিক", "গ) ব্যবসায়ী", "ঘ) ডাক্তার"],
                    "answer": "খ) সাংবাদিক ও সাহিত্যিক"
                },
                {
                    "q": "অন্তু মুক্তিযুদ্ধ সম্পর্কে জানতে চাইলে খোকন চাচা তাকে কার কথা বলেন?",
                    "options": ["ক) তার দাদার ও বীর মুক্তিযোদ্ধাদের", "খ) বিদেশি বন্ধুদের", "গ) পিসিমার", "ঘ) প্রতিবেশীদের"],
                    "answer": "ক) তার দাদার ও বীর মুক্তিযোদ্ধাদের"
                },
                {
                    "q": "মুক্তিযুদ্ধে অন্তুর দাদা কীভাবে শহিদ হয়েছিলেন?",
                    "options": ["ক) পাক হানাদার বাহিনীর গুলিতে", "খ) বোমায়", "গ) অসুস্থ হয়ে", "ঘ) দুর্ঘটনায়"],
                    "answer": "ক) পাক হানাদার বাহিনীর গুলিতে"
                },
                {
                    "q": "পাক হানাদার বাহিনী বাঙালিদের ওপর কবে আক্রমণ চালায়?",
                    "options": ["ক) ১৯৭১ সালের ২৬ মার্চ", "খ) ১৯৭১ সালের ২৫ মার্চ কালরাত্রিতে", "গ) ১৯৫২ সালের ২১ ফেব্রুয়ারি", "ঘ) ১৯৭১ সালের ১৬ ডিসেম্বর"],
                    "answer": "খ) ১৯৭১ সালের ২৫ মার্চ কালরাত্রিতে"
                },
                {
                    "q": "মুক্তিযুদ্ধে সাধারণ মানুষ কীভাবে অংশগ্রহণ করেছিল?",
                    "options": ["ক) জীবন বাজি রেখে অস্ত্র হাতে ও নানাভাবে সাহায্য করে", "খ) কেবল ঘরে বসে", "গ) পালিয়ে গিয়ে", "ঘ) চুপ করে থেকে"],
                    "answer": "ক) জীবন বাজি রেখে অস্ত্র হাতে ও নানাভাবে সাহায্য করে"
                },
                {
                    "q": "খোকন চাচার মতে আমাদের পিতৃপুরুষ কারা?",
                    "options": ["ক) যারা দেশের জন্য লড়াই করে শহীদ হয়েছেন এবং মুক্তি এনেছেন", "খ) শুধু বয়স্ক ব্যক্তিরা", "গ) ধনী ব্যক্তিরা", "ঘ) বিদেশি গবেষকরা"],
                    "answer": "ক) যারা দেশের জন্য লড়াই করে শহীদ হয়েছেন এবং মুক্তি এনেছেন"
                },
                {
                    "q": "মুক্তিযুদ্ধে বাঙালিরা কত মাস রক্তক্ষয়ী সংগ্রাম করেছিল?",
                    "options": ["ক) ৬ মাস", "খ) ৯ মাস", "গ) ১২ মাস", "ঘ) ৩ মাস"],
                    "answer": "খ) ৯ মাস"
                },
                {
                    "q": "কত তারিখে বাংলাদেশ স্বাধীনতা লাভ করে?",
                    "options": ["ক) ১৯৭১ সালের ১৬ ডিসেম্বর", "খ) ১৯৭২ সালের ২১ ফেব্রুয়ারি", "গ) ১৯৭১ সালের ২৬ মার্চ", "ঘ) ১৯৭০ সালের ৫ জানুয়ারি"],
                    "answer": "ক) ১৯৭১ সালের ১৬ ডিসেম্বর"
                },
                {
                    "q": "'পিতৃপুরুষের গল্প' গল্পে মূলত কিসের ইতিহাস তুলে ধরা হয়েছে?",
                    "options": ["ক) প্রাচীন আমলের ইতিহাস", "খ) বাঙালির মুক্তি সংগ্রামের গৌরবময় ইতিহাস", "গ) রূপকথার গল্প", "ঘ) শহরের আধুনিকায়ন"],
                    "answer": "খ) বাঙালির মুক্তি সংগ্রামের গৌরবময় ইতিহাস"
                },
                {
                    "q": "অন্তর মনের ভেতর কিসের জন্ম নেয় যখন সে মুক্তিযুদ্ধের কথা শোনে?",
                    "options": ["ক) দেশের প্রতি ভালোবাসা ও শ্রদ্ধাবোধ", "খ) ভয়", "গ) বিরক্তি", "ঘ) কৌতূহলহীনতা"],
                    "answer": "ক) দেশের প্রতি ভালোবাসা ও শ্রদ্ধাবোধ"
                },
                {
                    "q": "নতুন প্রজন্মের কাছে মুক্তিযুদ্ধের চেতনা পৌঁছে দেওয়ার মূল মাধ্যম কোনটি?",
                    "options": ["ক) সঠিক ইতিহাস চর্চা ও গল্প শোনা", "খ) খেলাধুলা", "গ) ভ্রমণ", "ঘ) প্রযুক্তি ব্যবহার"],
                    "answer": "ক) সঠিক ইতিহাস চর্চা ও গল্প শোনা"
                }
            ],
            "স্মৃতি বা সিথি পদ্য": [
                {
                    "q": "স্মৃতি বা প্রকৃতিভিত্তিক পদ্যটিতে মূলত কী ফুটিয়ে তোলা হয়েছে?",
                    "options": ["ক) মানুষের স্মৃতির পাতা ও প্রকৃতির রূপ", "খ) যন্ত্রসভ্যতার জয়গান", "গ) যুদ্ধের নির্মমতা", "ঘ) ব্যবসার হিসাব"],
                    "answer": "ক) মানুষের স্মৃতির পাতা ও প্রকৃতির রূপ"
                },
                {
                    "q": "স্মৃতির টানে মানুষের মনে কী ভেসে ওঠে?",
                    "options": ["ক) অতীত দিনের সুন্দর মুহূর্ত ও প্রিয়জন", "খ) ভয়ের স্বপ্ন", "গ) কার্টুন চরিত্র", "ঘ) জটিল হিসাব"],
                    "answer": "ক) অতীত দিনের সুন্দর মুহূর্ত ও প্রিয়জন"
                },
                {
                    "q": "কবিতায় কোন সময়ের কথা বিশেষভাবে স্মরণ করা হয়েছে?",
                    "options": ["ক) শৈশব ও ফেলে আসা দিনগুলো", "খ) ভবিষ্যতের পরিকল্পনা", "গ) অফিসের কর্মব্যস্ততা", "ঘ) পরীক্ষার হল"],
                    "answer": "ক) শৈশব ও ফেলে আসা দিনগুলো"
                },
                {
                    "q": "প্রকৃতির বিভিন্ন উপাদান আমাদের মনে কিসের সৃষ্টি করে?",
                    "options": ["ক) আনন্দের ও স্মৃতির আলোড়ন", "খ) বিরক্তি", "গ) আতঙ্ক", "ঘ) ক্লান্তি"],
                    "answer": "ক) আনন্দের ও স্মৃতির আলোড়ন"
                },
                {
                    "q": "স্মৃতি জিনিসটি মানুষের জীবনে কেমন ভূমিকা রাখে?",
                    "options": ["ক) জীবনকে সমৃদ্ধ ও আবেগপ্রবণ করে তোলে", "খ) কোনো প্রভাব ফেলে না", "গ) কষ্ট বাড়ায় কেবল", "ঘ) ভুলিয়ে দেয় সবকিছু"],
                    "answer": "ক) জীবনকে সমৃদ্ধ ও আবেগপ্রবণ করে তোলে"
                },
                {
                    "q": "কবিতায় কিসের গন্ধ জড়িয়ে থাকার কথা বলা হয়েছে?",
                    "options": ["ক) মাটির ও শৈশবের সোঁদা গন্ধ", "খ) রাসায়নিক গন্ধ", "গ) ধোঁয়ার গন্ধ", "ঘ) ওষুধের গন্ধ"],
                    "answer": "ক) মাটির ও শৈশবের সোঁদা গন্ধ"
                },
                {
                    "q": "ফেলে আসা দিনগুলোর স্মৃতি মানুষকে কোথায় ফিরিয়ে নিয়ে যায়?",
                    "options": ["ক) অতীত সোনালী অতীতে", "খ) ভবিষ্যতের দিকে", "গ) মহাশূন্যে", "ঘ) অন্ধকারের দিকে"],
                    "answer": "ক) অতীত সোনালী অতীতে"
                },
                {
                    "q": "কবিতার মূল সুর বা ভাব কী?",
                    "options": ["ক) নস্টালজিয়া বা স্মৃতিচারণ", "খ) বিদ্রোহ", "গ) উপহাস", "ঘ) বৈজ্ঞানিক আবিষ্কার"],
                    "answer": "ক) নস্টালজিয়া বা স্মৃতিচারণ"
                },
                {
                    "q": "স্মৃতির পাতায় কাদের মুখ ভেসে ওঠে?",
                    "options": ["ক) মা-বাবা, বন্ধু ও আপনজন", "খ) অচেনা পথচারী", "গ) ঐতিহাসিক চরিত্র", "ঘ) রূপকথার রাজা"],
                    "answer": "ক) মা-বাবা, বন্ধু ও আপনজন"
                },
                {
                    "q": "কবিতায় নদীর রূপ কেমনভাবে ফুটিয়ে তোলা হয়েছে?",
                    "options": ["ক) চিরবহমান ও প্রাণের প্রতীক হিসেবে", "খ) ধ্বংসাত্মক হিসেবে", "গ) শুকনো খাল হিসেবে", "ঘ) কৃত্রিম জলাশয় হিসেবে"],
                    "answer": "ক) চিরবহমান ও প্রাণের প্রতীক হিসেবে"
                },
                {
                    "q": "মানুষের জীবনের সবচেয়ে অমূল্য সম্পদ কোনটি?",
                    "options": ["ক) মধুর স্মৃতি", "খ) অর্থ-কড়ি", "গ) দালানকোঠা", "ঘ) বাহন"],
                    "answer": "ক) মধুর স্মৃতি"
                },
                {
                    "q": "কবির লেখনীতে স্মৃতিচারণ মূলত কিসের প্রতীক?",
                    "options": ["ক) শিকড়ের প্রতি টান", "খ) পলায়নপরতা", "গ) উদাসীনতা", "ঘ) বিচ্ছেদ বেদনা"],
                    "answer": "ক) শিকড়ের প্রতি টান"
                },
                {
                    "q": "স্মৃতি কবিতা বা পদ্য পাঠ করলে পাঠকের মনে কিসের জন্ম হয়?",
                    "options": ["ক) আবেগ ও ভালোবাসা", "খ) রাগ", "গ) হিংসা", "ঘ) লোভ"],
                    "answer": "ক) আবেগ ও ভালোবাসা"
                },
                {
                    "q": "গ্রামবাংলার চিরচেনা রূপ স্মৃতির পাতায় কীভাবে ধরা দেয়?",
                    "options": ["ক) রূপময় ও স্নিগ্ধ রূপে", "খ) ভয়ংকর রূপে", "গ) রুক্ষ ও শুষ্ক রূপে", "ঘ) ঝঞ্ঝাবিক্ষুব্ধ রূপে"],
                    "answer": "ক) রূপময় ও স্নিগ্ধ রূপে"
                },
                {
                    "q": "স্মৃতি মানুষের মনকে কেমন করে রাখে?",
                    "options": ["ক) কোমল ও সংবেদনশীল", "খ) কঠোর", "গ) স্বার্থপর", "ঘ) উদাসীন"],
                    "answer": "ক) কোমল ও সংবেদনশীল"
                }
            ]
        }
    },
    "Class 8": {
        "গণিত": {
            "অধ্যায় ৩ (পরিমাপ)": [
                {
                    "q": "মেট্রিক পদ্ধতিতে আয়তনের একক কী?",
                    "options": ["ক) মিটার", "খ) লিটার", "গ) গ্রাম", "ঘ) বর্গমিটার"],
                    "answer": "খ) লিটার"
                },
                {
                    "q": "১ হেক্টরে কত বর্গমিটার?",
                    "options": ["ক) ১০০ বর্গমিটার", "খ) ১০০০ বর্গমিটার", "গ) ১০০০০ বর্গমিটার", "ঘ) ১ লক্ষ বর্গমিটার"],
                    "answer": "গ) ১০০০০ বর্গমিটার"
                },
                {
                    "q": "ভূমির ক্ষেত্রফল নির্ণয়ের সূত্র কোনটি?",
                    "options": ["ক) দৈর্ঘ্য × প্রস্থ", "খ) দৈর্ঘ্য + প্রস্থ", "গ) ২ × (দৈর্ঘ্য + প্রস্থ)", "ঘ) বাহু × বাহু"],
                    "answer": "ক) দৈর্ঘ্য × প্রস্থ"
                },
                {
                    "q": "১ ঘন সেন্টিমিটার বিশুদ্ধ পানির ওজন কত?",
                    "options": ["ক) ১ মিলিগ্রাম", "খ) ১ গ্রাম", "গ) ১ কেজি", "ঘ) ১ টন"],
                    "answer": "খ) ১ গ্রাম"
                },
                {
                    "q": "সামান্তরিকের ক্ষেত্রফল নির্ণয়ের সূত্র কী?",
                    "options": ["ক) ভূমি × উচ্চতা", "খ) ১/২ × ভূমি × উচ্চতা", "গ) দৈর্ঘ্য × প্রস্থ", "ঘ) বাহু কিউব"],
                    "answer": "ক) ভূমি × উচ্চতা"
                },
                {
                    "q": "সিজিএস (CGS) পদ্ধতিতে দূরত্বের একক কী?",
                    "options": ["ক) মিটার", "খ) সেন্টিমিটার", "গ) কিলোমিটার", "ঘ) মিলিমিটার"],
                    "answer": "খ) সেন্টিমিটার"
                },
                {
                    "q": "একটি আয়তাকার ক্ষেত্রের দৈর্ঘ্য ১০ মিটার এবং প্রস্থ ৫ মিটার হলে, এর ক্ষেত্রফল কত?",
                    "options": ["ক) ১৫ বর্গমিটার", "খ) ৫০ বর্গমিটার", "গ) ৩০ বর্গমিটার", "ঘ) ১০০ বর্গমিটার"],
                    "answer": "খ) ৫০ বর্গমিটার"
                },
                {
                    "q": "ত্রিভুজের ক্ষেত্রফল নির্ণয়ের সূত্র কোনটি?",
                    "options": ["ক) ভূমি × উচ্চতা", "খ) ১/২ × ভূমি × উচ্চতা", "গ) দৈর্ঘ্য × প্রস্থ", "ঘ) ৪ × বাহু"],
                    "answer": "খ) ১/২ × ভূমি × উচ্চতা"
                },
                {
                    "q": "১ লিটার পানি কত কিলোগ্রামের সমান?",
                    "options": ["ক) ১ কিলোগ্রাম", "খ) ০.১ কিলোগ্রাম", "গ) ১০ কিলোগ্রাম", "ঘ) ০.০০১ কিলোগ্রাম"],
                    "answer": "ক) ১ কিলোগ্রাম"
                },
                {
                    "q": "বর্গক্ষেত্রের পরিসীমা নির্ণয়ের সূত্র কোনটি?",
                    "options": ["ক) ৪ × এক বাহুর দৈর্ঘ্য", "খ) বাহু × বাহু", "গ) ২ × (দৈর্ঘ্য + প্রস্থ)", "ঘ) ৬ × বাহু স্কয়ার"],
                    "answer": "ক) ৪ × এক বাহুর দৈর্ঘ্য"
                },
                {
                    "q": "আইএসও (ISO) বা আন্তর্জাতিক পদ্ধতিতে ভরের মূল একক কী?",
                    "options": ["ক) গ্রাম", "খ) কিলোগ্রাম", "গ) টন", "ঘ) কুইন্টাল"],
                    "answer": "খ) কিলোগ্রাম"
                },
                {
                    "q": "একটি বর্গক্ষেত্রের এক বাহুর দৈর্ঘ্য ৪ মিটার হলে, এর ক্ষেত্রফল কত?",
                    "options": ["ক) ৮ বর্গমিটার", "খ) ১২ বর্গমিটার", "গ) ১৬ বর্গমিটার", "ঘ) ৩২ বর্গমিটার"],
                    "answer": "গ) ১৬ বর্গমিটার"
                },
                {
                    "q": "রম্বসের ক্ষেত্রফল নির্ণয়ের সূত্র কোনটি?",
                    "options": ["ক) ১/২ × কর্ণদ্বয়ের গুণফল", "খ) ভূমি × উচ্চতা", "গ) দৈর্ঘ্য × প্রস্থ", "ঘ) বাহু কিউব"],
                    "answer": "ক) ১/২ × কর্ণদ্বয়ের গুণফল"
                },
                {
                    "q": "১ মেট্রিক টনে কত কিলোগ্রাম?",
                    "options": ["ক) ১০০ কেজি", "খ) ১০০০ কেজি", "গ) ১০০০০ কেজি", "ঘ) ১০ কেজি"],
                    "answer": "খ) ১০০০ কেজি"
                },
                {
                    "q": "আয়তাকার ঘনবস্তুর সমগ্র তলের ক্ষেত্রফলের সূত্র কোনটি?",
                    "options": ["ক) ২(ab + bc + ca)", "খ) abc", "গ) ৬a^2", "ঘ) ৪ab"],
                    "answer": "ক) ২(ab + bc + ca)"
                }
            ],
            "অধ্যায় ৯ (পিথাগোরাসের উপপাদ্য)": [
                {
                    "q": "পিথাগোরাসের উপপাদ্যটি কোন ত্রিভুজের ক্ষেত্রে প্রযোজ্য?",
                    "options": ["ক) সমকোণী ত্রিভুজ", "খ) স্থূলকোণী ত্রিভুজ", "গ) সমবাহু ত্রিভুজ", "ঘ) বিসমবাহু ত্রিভুজ"],
                    "answer": "ক) সমকোণী ত্রিভুজ"
                },
                {
                    "q": "সমকোণী ত্রিভুজের সমকোণের বিপরীত বাহুকে কী বলা হয়?",
                    "options": ["ক) লম্ব", "খ) ভূমি", "গ) অতিভুজ", "ঘ) সমদ্বিখণ্ডক"],
                    "answer": "গ) অতিভুজ"
                },
                {
                    "q": "পিথাগোরাসের উপপাদ্য অনুযায়ী সমকোণী ত্রিভুজের ক্ষেত্রে নিচের কোনটি সঠিক?",
                    "options": ["ক) লম্ব^2 + ভূমি^2 = অতিভুজ^2", "খ) লম্ব^2 + অতিভুজ^2 = ভূমি^2", "গ) অতিভুজ^2 + ভূমি^2 = লম্ব^2", "ঘ) লম্ব + ভূমি = অতিভুজ"],
                    "answer": "ক) লম্ব^2 + ভূমি^2 = অতিভুজ^2"
                },
                {
                    "q": "একটি সমকোণী ত্রিভুজের লম্ব ৩ সেমি এবং ভূমি ৪ সেমি হলে, অতিভুজের দৈর্ঘ্য কত?",
                    "options": ["ক) ৫ সেমি", "খ) ৭ সেমি", "গ) ২৫ সেমি", "ঘ) ১২ সেমি"],
                    "answer": "ক) ৫ সেমি"
                },
                {
                    "q": "সমকোণী ত্রিভুজের সবচেয়ে বড় বাহু কোনটি?",
                    "options": ["ক) লম্ব", "খ) ভূমি", "গ) অতিভুজ", "ঘ) কোনোটিই নয়"],
                    "answer": "গ) অতিভুজ"
                },
                {
                    "q": "পিথাগোরাস কোন দেশের প্রাচীন গণিতবিদ ছিলেন?",
                    "options": ["ক) গ্রিস", "খ) মিশর", "গ) ভারত", "ঘ) চীন"],
                    "answer": "ক) গ্রিস"
                },
                {
                    "q": "কোন ত্রিভুজের বাহুগুলোর অনুপাত ৩ : ৪ : ৫ হলে, ত্রিভুজটি কেমন হবে?",
                    "options": ["ক) সমকোণী", "খ) স্থূলকোণী", "গ) সমবাহু", "ঘ) সূক্ষ্মকোণী"],
                    "answer": "ক) সমকোণী"
                },
                {
                    "q": "একটি সমকোণী ত্রিভুজের অতিভুজ ১০ সেমি এবং ভূমি ৮ সেমি হলে, লম্বের দৈর্ঘ্য কত?",
                    "options": ["ক) ৬ সেমি", "খ) ৩ সেমি", "গ) ৫ সেমি", "ঘ) ২ সেমি"],
                    "answer": "ক) ৬ সেমি"
                },
                {
                    "q": "সমকোণী ত্রিভুজের একটি কোণ সমকোণ হলে, বাকি দুটি কোণের সমষ্টি কত?",
                    "options": ["ক) ৯০ ডিগ্রি", "খ) ১৮০ ডিগ্রি", "গ) ৪৫ ডিগ্রি", "ঘ) ৩৬০ ডিগ্রি"],
                    "answer": "ক) ৯০ ডিগ্রি"
                },
                {
                    "q": "পিথাগোরাসের উপপাদ্যের মূল প্রতিপাদ্য বিষয় কীসের সাথে সম্পর্কিত?",
                    "options": ["ক) ক্ষেত্রফল", "খ) পরিসীমা", "গ) আয়তন", "ঘ) কোণ"],
                    "answer": "ক) ক্ষেত্রফল"
                },
                {
                    "q": "একটি সমকোণী ত্রিভুজের লম্ব ৫ সেমি এবং অতিভুজ ১৩ সেমি হলে, ভূমির দৈর্ঘ্য কত?",
                    "options": ["ক) ১২ সেমি", "খ) ৮ সেমি", "গ) ১০ সেমি", "ঘ) ৯ সেমি"],
                    "answer": "ক) ১২ সেমি"
                },
                {
                    "q": "সমকোণী ত্রিভুজের সমকোণ সংলগ্ন বাহু দুটির উপর অঙ্কিত বর্গক্ষেত্রদ্বয়ের ক্ষেত্রফলের সমষ্টি কার সমান?",
                    "options": ["ক) অতিভুজের উপর অঙ্কিত বর্গক্ষেত্রের ক্ষেত্রফলের সমান", "খ) ত্রিভুজের পরিসীমার সমান", "গ) উচ্চতার সমান", "ঘ) ভূমির দ্বিগুণ"],
                    "answer": "ক) অতিভুজের উপর অঙ্কিত বর্গক্ষেত্রের ক্ষেত্রফলের সমান"
                },
                {
                    "q": "কোনটি পিথাগোরাসের ত্রয়ী (Pythagorean triple)?",
                    "options": ["ক) (3, 4, 5)", "খ) (1, 2, 3)", "গ) (2, 3, 4)", "ঘ) (4, 5, 6)"],
                    "answer": "ক) (3, 4, 5)"
                },
                {
                    "q": "সমকোণী সমদ্বিবাহু ত্রিভুজের সমান বাহুদ্বয়ের দৈর্ঘ্য ১ একক হলে, অতিভুজের দৈর্ঘ্য কত?",
                    "options": ["ক) রুট ওভার ২", "খ) ২", "গ) ৩", "ঘ) ১"],
                    "answer": "ক) রুট ওভার ২"
                },
                {
                    "q": "পিথাগোরাসের উপপাদ্যের বিপরীত উপপাদ্য ব্যবহার করে কী প্রমাণ করা যায়?",
                    "options": ["ক) ত্রিভুজটি সমকোণী", "খ) ত্রিভুজটি সমবাহু", "গ) ত্রিভুজটি সর্বসম", "ঘ) ত্রিভুজটির ক্ষেত্রফল"],
                    "answer": "ক) ত্রিভুজটি সমকোণী"
                }
            ]
        },
        "বাংলা": {
            "বাংলা নববর্ষ": [
                {
                    "q": "'বাংলা নববর্ষ' প্রবন্ধটি বা এর মূল ভাব অনুসারে পহেলা বৈশাখ কিসের প্রতীক?",
                    "options": ["ক) বাঙালির আবহমান সংস্কৃতি ও প্রাণের উৎসব", "খ) নতুন ব্যবসার সূচনা", "গ) রাজনৈতিক পরিবর্তন", "ঘ) ঋতু পরিবর্তনের শোক"],
                    "answer": "ক) বাঙালির আবহমান সংস্কৃতি ও প্রাণের উৎসব"
                },
                {
                    "q": "বাংলা সন চালুর পেছনে প্রধান ভূমিকা কার বলে ধরা হয়?",
                    "options": ["ক) সম্রাট আকবর", "খ) সম্রাট শাহজাহান", "গ) মোগল সেনাপতি", "ঘ) হোসেন শাহ"],
                    "answer": "ক) সম্রাট আকবর"
                },
                {
                    "q": "বাংলা সন প্রথম কবে থেকে চালু হয়?",
                    "options": ["ক) ১৫৮৪ বা ১৫৫৬ সালের দিকে (আকবরের সিংহাসন আরোহণের বছর)", "খ) ১২০০ সাল", "গ) ১৬৫০ সাল", "ঘ) ১৭০৯ সাল"],
                    "answer": "ক) ১৫৮৪ বা ১৫৫৬ সালের দিকে (আকবরের সিংহাসন আরোহণের বছর)"
                },
                {
                    "q": "পূর্বে বাংলা নববর্ষে ব্যবসায়ীদের প্রধান আকর্ষণ কী ছিল?",
                    "options": ["ক) হালখাতা উৎসব", "খ) বৈশাখী মেলা", "গ) লাঠিখেলা", "ঘ) ঘুড়ি ওড়ানো"],
                    "answer": "ক) হালখাতা উৎসব"
                },
                {
                    "q": "'হালখাতা' বলতে কী বোঝায়?",
                    "options": ["ক) পুরোনো হিসাব চুকিয়ে নতুন হিসাবের খাতা খোলা", "খ) নতুন বই কেনা", "গ) খাতার দোকান দেওয়া", "ঘ) নতুন খাতা তৈরি করা"],
                    "answer": "ক) পুরোনো হিসাব চুকিয়ে নতুন হিসাবের খাতা খোলা"
                },
                {
                    "q": "পহেলা বৈশাখে ব্যবসায়ীরা গ্রাহকদের কী দিয়ে আপ্যায়ন করতেন?",
                    "options": ["ক) মিষ্টি", "খ) ফলমূল", "গ) পিঠা", "ঘ) চা"],
                    "answer": "ক) মিষ্টি"
                },
                {
                    "q": "মঙ্গল শোভাযাত্রা পহেলা বৈশাখের কোন শহরের উৎসবের প্রধান অঙ্গে পরিণত হয়েছে?",
                    "options": ["ক) ঢাকা", "খ) কলকাতা", "গ) রাজশাহী", "ঘ) চট্টগ্রাম"],
                    "answer": "ক) ঢাকা"
                },
                {
                    "q": "পহেলা বৈশাখের ঐতিহ্যবাহী খাবার হিসেবে কোনটি সবচেয়ে জনপ্রিয়?",
                    "options": ["ক) পান্তা ভাত ও ইলিশ মাছ", "খ) খিচুড়ি ও মাংস", "গ) পোলাও ও রোস্ট", "ঘ) রুটি ও সবজি"],
                    "answer": "ক) পান্তা ভাত ও ইলিশ মাছ"
                },
                {
                    "q": "বাংলা নববর্ষ উদ্যাপনের মধ্য দিয়ে বাঙালির কোন পরিচয় প্রকাশ পায়?",
                    "options": ["ক) অসাম্প্রদায়িক ও বাঙালি জাতীয়তাবাদী পরিচয়", "খ) ধর্মীয় গোঁড়ামি", "গ) রাজনৈতিক দলীয় পরিচয়", "ঘ) আঞ্চলিক পরিচয়"],
                    "answer": "ক) অসাম্প্রদায়িক ও বাঙালি জাতীয়তাবাদী পরিচয়"
                },
                {
                    "q": "বৈশাখ মাসের প্রথম দিনটি বাংলাদেশে কীভাবে পালিত হয়?",
                    "options": ["ক) জাতীয় উৎসব হিসেবে সর্বস্তরের মানুষের অংশগ্রহণে", "খ) শুধু সরকারি ছুটির দিন হিসেবে", "গ) শুধু শিশুদের উৎসব হিসেবে", "ঘ) ব্যবসায়ীদের ঘরোয়া অনুষ্ঠান হিসেবে"],
                    "answer": "ক) জাতীয় উৎসব হিসেবে সর্বস্তরের মানুষের অংশগ্রহণে"
                },
                {
                    "q": "বাংলা সনের প্রথম মাস কোনটি?",
                    "options": ["ক) বৈশাখ", "খ) চৈত্র", "গ) ফাল্গুন", "ঘ) আষাঢ়"],
                    "answer": "ক) বৈশাখ"
                },
                {
                    "q": "বর্ষবরণে রমনা বটমূলে ছায়ানটের গান পরিবেশন কত সালের দশক থেকে শুরু হয়?",
                    "options": ["ক) ১৯৬০-এর দশক থেকে", "খ) ১৯৫০-এর দশক", "গ) ১৯৭০-এর দশক", "ঘ) ১৯৮০-এর দশক"],
                    "answer": "ক) ১৯৬০-এর দশক থেকে"
                },
                {
                    "q": "বাংলা নববর্ষের উৎসবে গ্রামীণ মেলায় কীসের সমাগম ঘটে?",
                    "options": ["ক) বিভিন্ন কুটির শিল্প, হস্তশিল্প ও খেলনা", "খ) আধুনিক ইলেকট্রনিক্স সামগ্রী", "গ) বিদেশি পণ্য", "ঘ) মোটর গাড়ি"],
                    "answer": "ক) বিভিন্ন কুটির শিল্প, হস্তশিল্প ও খেলনা"
                },
                {
                    "q": "বাঙালির সর্বজনীন উৎসব কোনটি?",
                    "options": ["ক) পহেলা বৈশাখ বা বাংলা নববর্ষ", "খ) বিজয় দিবস", "গ) একুশে ফেব্রুয়ারি", "ঘ) স্বাধীনতা দিবস"],
                    "answer": "ক) পহেলা বৈশাখ বা বাংলা নববর্ষ"
                },
                {
                    "q": "বাংলা নববর্ষ পালনের মাধ্যমে আমাদের মনে কিসের বিস্তার ঘটে?",
                    "options": ["ক) নিজস্ব ঐতিহ্য ও সংস্কৃতির প্রতি ভালোবাসা", "খ) আধুনিক প্রযুক্তির জ্ঞান", "গ) ব্যবসার প্রসার", "ঘ) বিদেশপ্রীতি"],
                    "answer": "ক) নিজস্ব ঐতিহ্য ও সংস্কৃতির প্রতি ভালোবাসা"
                }
            ],
            "রুপাই পদ্য": [
                {
                    "q": "'রুপাই' কবিতাটি কোন কবির রচনা?",
                    "options": ["ক) জসীমউদ্দীন", "খ) কাজী নজরুল ইসলাম", "গ) রবীন্দ্রনাথ ঠাকুর", "ঘ) সুফিয়া কামাল"],
                    "answer": "ক) জসীমউদ্দীন"
                },
                {
                    "q": "জসীমউদ্দীন কী হিসেবে সমধিক পরিচিত?",
                    "options": ["ক) পল্লীকবি", "খ) বিদ্রোহী কবি", "গ) রোমান্টিক কবি", "ঘ) আধুনিক কবি"],
                    "answer": "ক) পল্লীকবি"
                },
                {
                    "q": "'রুপাই' কবিতাটি কবির কোন কাব্যগ্রন্থ থেকে সংকলিত হয়েছে?",
                    "options": ["ক) নকশী কাথার মাঠ (বা রাখালী কাব্যগ্রন্থ)", "খ) সোজন বাদিয়ার ঘাট", "গ) বালুচর", "ঘ) ধানক্ষেত"],
                    "answer": "ক) নকশী কাথার মাঠ (বা রাখালী কাব্যগ্রন্থ)"
                },
                {
                    "q": "রুপাইয়ের শরীরের রঙ কেমন ছিল?",
                    "options": ["ক) গায়ের বর্ণ প্রদীপের মতো উজ্জ্বল বা শ্যামল", "খ) ফর্সা", "গ) কালচে", "ঘ) লালচে"],
                    "answer": "ক) গায়ের বর্ণ প্রদীপের মতো উজ্জ্বল বা শ্যামল"
                },
                {
                    "q": "রুপাই কিসের কাজ করতে সবচেয়ে পটু ছিল?",
                    "options": ["ক) কৃষিকাজ ও ক্ষেতে ধান বোনা বা কাটার কাজ", "খ) মাছ ধরা", "গ) নৌকা চালানো", "ঘ) রাখালগিরি"],
                    "answer": "ক) কৃষিকাজ ও ক্ষেতে ধান বোনা বা কাটার কাজ"
                },
                {
                    "q": "রুপাইয়ের হাতের কাজকে কবি কিসের সাথে তুলনা করেছেন?",
                    "options": ["ক) সোনার খনি বা সোনা ফলানো হাত", "খ) কারিগরের শিল্পকর্ম", "গ) ফুলের মালা", "ঘ) রূপার থালা"],
                    "answer": "ক) সোনার খনি বা সোনা ফলানো হাত"
                },
                {
                    "q": "গ্রামবাংলার খেটে খাওয়া মানুষের রূপ কোন কবিতায় ফুটে উঠেছে?",
                    "options": ["ক) রুপাই", "খ) বিদ্রোহী", "গ) বঙ্গবাণী", "ঘ) কপোতাক্ষ নদ"],
                    "answer": "ক) রুপাই"
                },
                {
                    "q": "রুপাইয়ের মুখের হাসি কেমন ছিল?",
                    "options": ["ক) স্নিগ্ধ ও সরল চাঁদের আলোর মতো", "খ) কৃত্রিম", "গ) ভীতিপ্রদ", "ঘ) গম্ভীর"],
                    "answer": "ক) স্নিগ্ধ ও সরল চাঁদের আলোর মতো"
                },
                {
                    "q": "রুপাইয়ের চরিত্রের প্রধান বৈশিষ্ট্য কী?",
                    "options": ["ক) পরিশ্রমী, সরল ও দেশপ্রেমিক প্রাণ", "খ) অলসতা", "গ) চতুরতা", "ঘ) অহংকার"],
                    "answer": "ক) পরিশ্রমী, সরল ও দেশপ্রেমিক প্রাণ"
                },
                {
                    "q": "পল্লীকবি জসীমউদ্দীন কোথায় জন্মগ্রহণ করেন?",
                    "options": ["ক) ফরিদপুর জেলার তাম্বুলখানা গ্রামে", "খ) ঢাকা জেলায়", "গ) বরিশাল জেলায়", "ঘ) কুষ্টিয়া জেলায়"],
                    "answer": "ক) ফরিদপুর জেলার তাম্বুলখানা গ্রামে"
                }
            ]
        }
    }
}

    st.markdown("### 🔍 লাইব্রেরি থেকে শ্রেণি, বই ও অধ্যায় নির্বাচন করুন")
    col_11, col_12, col_13 = st.columns(3)
    
    with col_11:
        selected_class = st.selectbox("শ্রেণি বেছে নিন:", list(library_question_bank.keys()), key="quiz_class_sel")
        
    with col_12:
        available_books = list(library_question_bank[selected_class].keys())
        selected_book = st.selectbox("বই বা বিষয় বেছে নিন:", available_books, key="quiz_book_sel")
        
    with col_13:
        available_chapters = list(library_question_bank[selected_class][selected_book].keys())
        selected_chapter = st.selectbox("অধ্যায় বা টপিক বেছে নিন:", available_chapters, key="quiz_chap_sel")
        
    st.markdown("---")
    
    # প্র্যাকটিস কনফিগারেশন
    col_cl, col_c2 = st.columns(2)
    with col_cl:
        practice_mode = st.selectbox("অনুশীলনের মোড:", ["📝 MCQ কুইজ টেস্ট", "💡 ফ্ল্যাশকার্ড রিভিশন"], key="practice_mode_sel")
    with col_c2:
        study_hours = st.number_input("পড়াশোনার সময় (ঘণ্টা):", min_value=0.5, max_value=5.0, value=1.0, step=0.5, key="study_hours_input")
        
    generate_btn = st.button("🚀 নির্বাচিত অধ্যায় থেকে কুইজ শুরু করুন", use_container_width=True)
    if generate_btn:
        st.session_state["quiz_active"] = True
        st.session_state["show_review"] = False  # নতুন কুইজ শুরু হলে আগের রিভিউ রিসেট হবে
        st.session_state["generated_quiz"] = library_question_bank[selected_class][selected_book][selected_chapter]
        
    # কুইজ সচল থাকলে প্রশ্ন ও অপশন দেখাবে
    if st.session_state.get("quiz_active", False) and "generated_quiz" in st.session_state:
        st.markdown("---")
        st.markdown(f"### 📋 কুইজ সেশন: {selected_class} - {selected_book}")
        st.warning(f"নির্বাচিত অধ্যায়: **{selected_chapter}** | মোড: **{practice_mode}**")
        
        # যদি সাবমিট করা হয়ে থাকে কিন্তু রিভিউ মোড চালু থাকে
        if st.session_state.get("show_review", False):
            st.markdown("### 📊 কুইজ ফলাফল ও ভুল উত্তরের সংশোধন:")
            for idx, item in enumerate(st.session_state["generated_quiz"], 1):
                user_ans = st.session_state.get(f"lib_q_{idx}")
                correct_ans = item["answer"]
                
                st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                if user_ans == correct_ans:
                    st.success(f"আপনার উত্তর: {user_ans} (সঠিক আছে ✅)")
                else:
                    st.error(f"আপনার উত্তর: {user_ans} (ভুল ❌)")
                    st.info(f"💡 সঠিক উত্তর হবে: **{correct_ans}**")
                st.markdown("---")
            
            if st.button("🔄 আবার কুইজ শুরু করুন"):
                st.session_state["quiz_active"] = False
                st.session_state["show_review"] = False
                st.rerun()
        else:
            for idx, item in enumerate(st.session_state["generated_quiz"], 1):
                st.markdown(f"**প্রশ্ন {idx}:** {item['q']}")
                st.radio(f"উত্তর নির্বাচন করুন (প্রশ্ন {idx}):", item["options"], key=f"lib_q_{idx}")
                st.markdown("")
                
            if st.button("✅ উত্তর জমা দিন ও প্রগ্রেস সেভ করুন", key="submit_quiz_btn"):
                correct_count = 0
                total_q = len(st.session_state["generated_quiz"])
                
                for idx, item in enumerate(st.session_state["generated_quiz"], 1):
                    u_ans = st.session_state.get(f"lib_q_{idx}")
                    c_ans = item["answer"]
                    if u_ans == c_ans:
                        correct_count += 1
                        
                percentage = (correct_count / total_q) * 100 if total_q > 0 else 0
                if percentage >= 80:
                    performance = "★ Excellent (চমৎকার)"
                elif percentage >= 50:
                    performance = "Very Good (খুব ভালো)"
                else:
                    performance = "Good (চেষ্টা চালিয়ে যান)"
                    
                score_str = f"{correct_count}/{total_q} ({percentage:.0f}%)"
                
                # প্রগ্রেস ডেটা ড্যাশবোর্ড ও প্রগ্রেস পেজের জন্য সেভ করা
                progress_record = {
                    "class": selected_class,
                    "subject": selected_book,
                    "chapter": selected_chapter,
                    "mode": practice_mode,
                    "study_hours": study_hours,
                    "score": score_str,
                    "performance": performance
                }
                st.session_state["user_progress"].append(progress_record)
                
                st.balloons()
                st.success(f"অভিনন্দন! আপনার স্কোর: **{score_str}** | পারফরম্যান্স: **{performance}**")
                st.info("এই ফলাফলটি আপনার ড্যাশবোর্ড এবং প্রগ্রেস পেজে স্বয়ংক্রিয়ভাবে জমা হয়ে গেছে!")
                
                # রিভিউ মোড অন করে পেজ রিফ্রেশ করা
                st.session_state["show_review"] = True
                st.rerun()

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
    # কাস্টম CSS দিয়ে সমস্ত ইনপুট বক্স, টেক্সট এরিয়া ও বাটন আরও স্পষ্ট এবং সুন্দর করা হলো
    st.markdown("""
        <style>
            /* টেক্সট ইনপুট ও সিলেক্ট বক্সের বর্ডার স্পষ্ট করা */
            .stTextInput input, .stSelectbox select {
                border: 2px solid #cbd5e1 !important;
                border-radius: 6px !important;
                background-color: #ffffff !important;
            }
            .stTextInput input:focus, .stSelectbox select:focus {
                border-color: #1E3A8A !important;
                box-shadow: 0 0 0 2px rgba(30, 58, 138, 0.2) !important;
            }
            /* মূল টেক্সট এরিয়া বক্সের বর্ডার এবং লুক */
            textarea {
                border: 2px solid #94a3b8 !important;
                border-radius: 8px !important;
                background-color: #ffffff !important;
                padding: 10px !important;
            }
            textarea:focus {
                border-color: #1E3A8A !important;
                box-shadow: 0 0 0 3px rgba(30, 58, 138, 0.15) !important;
            }
        </style>
        <h2 style='color: #1E3A8A; font-weight: 700; margin-bottom: 5px;'> নিচে তোমার জন্য একটি নোট এডিটর দেওয়া হলো </h2>
        <p style='color: #475569; font-size: 15px;'>এখানে তুমি টেক্সট ফরম্যাটিং, টেবিল তৈরি, বিষয়ভিত্তিক নোট সংরক্ষণ, সার্চসহ তোমার নোটটি বিভিন্ন ফরম্যাটে ডাউনলোড করতে পারবে।</p>
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
    body { font-family: 'Calibri', sans-serif; background-color: #f8fafc; padding: 5px; margin: 0; }
    #editor-container { height: 280px; background: white; border-bottom-left-radius: 8px; border-bottom-right-radius: 8px; border: 1px solid #cbd5e1; }
    .ql-toolbar { background: #f1f5f9; border-top-left-radius: 8px; border-top-right-radius: 8px; border-color: #cbd5e1 !important; display: flex; flex-wrap: wrap; gap: 4px; align-items: center; }
    .ql-container { border-color: #cbd5e1 !important; font-size: 16px; }
    .ql-editor table { border-collapse: collapse !important; width: 100% !important; margin: 12px 0 !important; table-layout: fixed !important; }
    .ql-editor th, .ql-editor td { border: 1px solid #94a3b8 !important; padding: 10px 12px !important; word-break: break-word !important; vertical-align: top !important; }
    .ql-editor th { font-weight: bold !important; color: white !important; }
    .table-btn { background: #1e3a8a; color: white; border: none; padding: 5px 10px; border-radius: 4px; cursor: pointer; font-size: 12px; font-weight: bold; }
    .table-btn:hover { background: #3b82f6; }
    </style>
    </head>
    <body>
    <div style="background: #e2e8f0; padding: 8px 12px; border: 1px solid #cbd5e1; border-bottom: none; display: flex; gap: 10px; align-items: center; font-size: 13px; border-top-left-radius: 8px; border-top-right-radius: 8px;">
    <span><b>টেবিল: </b></span>
    <label>রো: <input type="number" id="t-rows" min="1" max="10" value="3" style="width: 40px;"></label>
    <label>কলাম: <input type="number" id="t-cols" min="1" max="10" value="3" style="width: 40px;"></label>
    <label>কালার:
    <select id="t-color" style="padding: 2px;">
    <option value="#1e3a8a">নীল (Blue)</option>
    <option value="#065f46">সবুজ (Green)</option>
    <option value="#991b1b">লাল (Red)</option>
    <option value="#374151">ধূসর (Gray)</option>
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
        placeholder: 'এখানে তোমার নোট লিখো...'
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

    col_editor, col_sidebar = st.columns([2, 1])

    with col_editor:
        st.markdown("### 📝 নোট এডিটর")
        c_top1, c_top2, c_top3 = st.columns([2, 1, 1])

        with c_top1:
            note_title = st.text_input(
                "নোটের শিরোনাম দাও (Title):",
                value=st.session_state["current_title"],
                key=f"rich_note_title_{st.session_state['clear_trigger']}"
            )

        with c_top2:
            subjects_list = [
                "বাংলা", "ইংরেজি", "গণিত", "বিজ্ঞান", 
                "বাংলাদেশ ও বিশ্বপরিচয়", "ধর্ম", "আইসিটি", "অন্যান্য"
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
            if st.button("+ নতুন নোট", use_container_width=True):
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
            <div style="background-color: #f1f5f9; border: 1.5px solid #cbd5e1; border-left: 6px solid #1E3A8A; padding: 14px 18px; border-radius: 8px; margin-top: 15px; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.04);">
                <p style="color: #0f172a; font-size: 15px; font-weight: 600; margin: 0; line-height: 1.5;">
                    📌 নোট হিসেবে সেভ করতে এডিটরে লেখা এখানে পেস্ট করো অথবা তোমার ইচ্ছেমত লেখা এখানে লিখে সেভ করো:
                </p>
            </div>
        """, unsafe_allow_html=True)

        # ব্যাকআপ টেক্সট বক্স
        backup_body = st.text_area(
            "",
            value=st.session_state["current_content"],
            height=120,
            placeholder="এখানে তোমার লেখার মূল কন্টেন্ট লিখো বা পেস্ট করো...",
            key=f"backup_editor_textarea_{st.session_state['clear_trigger']}"
        )

        typed_body = backup_body

        # সেভ বা আপডেট বাটন
        btn_label = "নোট আপডেট করো" if st.session_state["edit_id"] is not None else "নোট সেভ করো"

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
                    st.success("নোটটি সফলভাবে আপডেট হয়েছে!")
                else:
                    cursor.execute("""
                        INSERT INTO notes (title, content, subject, note_type) 
                        VALUES (?, ?, ?, ?)
                    """, (note_title, typed_body, note_subject, status_type))
                    conn.commit()
                    conn.close()
                    st.success(f"তোমার নোটটি '{note_subject}' বিষয়ে সফলভাবে '{status_type}' হিসেবে সংরক্ষিত হয়েছে!")

                st.session_state["current_title"] = ""
                st.session_state["current_content"] = ""
                st.session_state["clear_trigger"] += 1
                st.rerun()
            else:
                st.error("অনুগ্রহ করে নোটের শিরোনাম এবং নোটের বিষয়বস্তু উভয়ই প্রদান করো।")

        # ফাইল ডাউনলোড অপশন
        st.markdown("### 📥 অথবা Save As")
        download_text_data = backup_body if backup_body else st.session_state["current_content"]

        if download_text_data:
            st.info("ফাইলগুলো তোমার কম্পিউটারের **Downloads** ফোল্ডারে সেভ হয়েছে।")
            dl_col1, dl_col2, dl_col3 = st.columns(3)
            safe_filename = note_title.strip() if note_title.strip() else "My_Document"

            with dl_col1:
                st.download_button(
                    label="Save as Text (.txt)",
                    data=download_text_data,
                    file_name=f"{safe_filename}.txt",
                    mime="text/plain",
                    use_container_width=True
                )
            with dl_col2:
                word_html_data = f"<html><head><meta charset='utf-8'></head><body><h2>{note_title} ({note_subject})</h2>{download_text_data}</body></html>"
                st.download_button(
                    label="Save as Word (.doc)",
                    data=word_html_data,
                    file_name=f"{safe_filename}.doc",
                    mime="application/msword",
                    use_container_width=True
                )
            with dl_col3:
                st.download_button(
                    label="Save as HTML/PDF",
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
            st.markdown("### 📚 সেভ করা নোটসমূহ")

            filter_subject = st.selectbox(
                "বিষয় অনুযায়ী নোট ফিল্টার করো:",
                ["সকল বিষয়"] + subjects_list,
                key="filter_subject_selectbox"
            )

            search_query = st.text_input(
                "🔍 নোট খোঁজ (শিরোনাম বা শব্দ দিয়ে সার্চ করো):",
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
                st.info("কোনো নোট পাওয়া যায়নি।")
            else:
                for row in filtered_notes:
                    n_id, n_title, n_content, n_subject, n_type = row
                    with st.expander(f"📖 [{n_subject}] {n_title} — স্ট্যাটাস: {n_type}"):
                        st.markdown(f"**বিষয়:** {n_subject} | **স্ট্যাটাস:** {n_type}")
                        st.markdown("---")
                        st.markdown(n_content, unsafe_allow_html=True)
                        col_e, col_d = st.columns(2)

                        with col_e:
                            if st.button("এডিট করো", key=f"edit_note_{n_id}"):
                                st.session_state["current_title"] = n_title
                                st.session_state["current_content"] = n_content
                                st.session_state["current_subject"] = n_subject
                                st.session_state["edit_id"] = n_id
                                st.success("নোটটি এডিটর বক্সে লোড করা হয়েছে।")
                                st.rerun()

                        with col_d:
                            if st.button("ডিলিট করো", key=f"del_note_{n_id}"):
                                conn = sqlite3.connect("subject_notes.db")
                                cursor = conn.cursor()
                                cursor.execute("DELETE FROM notes WHERE id = ?", (n_id,))
                                conn.commit()
                                conn.close()
                                if st.session_state["edit_id"] == n_id:
                                    st.session_state["edit_id"] = None
                                    st.session_state["current_title"] = ""
                                    st.session_state["current_content"] = ""
                                st.success("নোটটি সফলভাবে ডিলিট করা হয়েছে!")
                                st.rerun()

    with col_sidebar:
        st.markdown("### 📖 আমার পাঠাগার")
        st.info("তোমার পছন্দের বই পড়তে বাঁ পাশের মূল মেনু থেকে 'আমার পাঠাগার' অপশনে ক্লিক করো।")
        st.markdown("---")
        st.markdown("### 🤖 কোনো প্রশ্ন থাকলে নিচে Gemini-কে জিজ্ঞেস করো")
        st.markdown(
            '<a href="https://gemini.google.com" target="_blank"><button style="width: 100%; background-color: #1E3A8A; color: white; padding: 10px; border: none; border-radius: 5px; font-weight: bold; cursor: pointer; margin-bottom: 10px;"> Open Gemini Chatbot</button></a>',
            unsafe_allow_html=True
        )
        
        # সাইডবারের ফাঁকা জায়গা পূরণের জন্য অতিরিক্ত স্টাডি টিপস কার্ড
        st.markdown("---")
        st.markdown("""
            <div style="background-color: #f8fafc; border: 1px solid #e2e8f0; padding: 12px; border-radius: 8px; text-align: center;">
                <p style="color: #1E3A8A; font-weight: bold; margin-bottom: 5px;">💡 পড়ার টিপস</p>
                <p style="color: #64748b; font-size: 13px; margin: 0;">নিয়মিত নোট তৈরি করলে পরীক্ষার প্রস্তুতি আরও সহজ হয়!</p>
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

import streamlit as st
import pandas as pd

import streamlit as st
import pandas as pd

def render_study_planner():
    st.markdown("""
    <div style="background: linear-gradient(135deg, #10B981 0%, #059669 100%); padding: 25px; border-radius: 12px; color: white; margin-bottom: 25px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
        <h2 style="margin: 0; color: white;">📚 AI Advanced Study Planner & Routine Hub</h2>
        <p style="margin: 5px 0 0 0; font-size: 16px; opacity: 0.9;">আপনার সিলেবাস তৈরি করুন, রুটিন সাজান এবং নিজের পছন্দমতো বা এআই এর সাহায্যে স্টাডি প্ল্যান তৈরি করুন!</p>
    </div>
    """, unsafe_allow_html=True)

    # সেশন স্টেট ইনিশিয়ালাইজ করা
    if "saved_syllabi" not in st.session_state:
        st.session_state["saved_syllabi"] = []
    if "syllabus_saved" not in st.session_state:
        st.session_state["syllabus_saved"] = False
    if "syllabus_editing" not in st.session_state:
        st.session_state["syllabus_editing"] = False

    if "weekly_routine_data" not in st.session_state:
        st.session_state["weekly_routine_data"] = {}
    if "routine_saved" not in st.session_state:
        st.session_state["routine_saved"] = False
    if "routine_editing" not in st.session_state:
        st.session_state["routine_editing"] = False

    if "saved_study_plans" not in st.session_state:
        st.session_state["saved_study_plans"] = []
    if "manual_plan_saved" not in st.session_state:
        st.session_state["manual_plan_saved"] = False
    if "manual_plan_editing" not in st.session_state:
        st.session_state["manual_plan_editing"] = False
    if "final_plan_saved_status" not in st.session_state:
        st.session_state["final_plan_saved_status"] = False

    # মাস্টার সিলেবাস ডাটাবেজ
    master_syllabus_db = {
        "Class 6": {
            "বাংলা": ["অধ্যায় ১: মিনু", "অধ্যায় ২: সততার পুরষ্কার", "অধ্যায় ৩: জন্মভূমি", "অধ্যায় ৪: মাদার তেরেসা"],
            "ইংরেজি": ["Unit 1: Attending a new school", "Unit 2: Congratulation well done", "Unit 3: Exploring useful things"],
            "গণিত": ["অধ্যায় ১: স্বাভাবিক সংখ্যা ও ভগ্নাংশ", "অধ্যায় ২: অনুপাত ও শতকরা", "অধ্যায় ৩: বীজগণীয় রাশি"]
        },
        "Class 7": {
            "বাংলা": ["অধ্যায় ১: বাঙালি সংস্কৃতি", "অধ্যায় ২: লখার একুশে", "অধ্যায় ৩: মিনু"],
            "ইংরেজি": ["Unit 1: Voices of change", "Unit 2: Exploring ambitions", "Unit 3: Environment and us"],
            "গণিত": ["অধ্যায় ১: মূলদ ও অমূলদ সংখ্যা", "অধ্যায় ২: অনুপাত ও সমানুপাত", "অধ্যায় ৩: পরিমাপ"]
        },
        "Class 8": {
            "বাংলা": ["অধ্যায় ১: অতিথির স্মৃতি", "অধ্যায় ২: তৈলসর্বস্ব", "অধ্যায় ৩: পাছে লোকে কিছু বলে"],
            "ইংরেজি": ["Unit 1: Tropic of happiness", "Unit 2: Past present future", "Unit 3: Glimpse of culture"],
            "গণিত": ["অধ্যায় ১: প্যাটার্ন", "অধ্যায় ২: মুনাফা", "অধ্যায় ৩: পরিমাপ"]
        }
    }

    # --- ১. সিলেবাস তৈরি বা এডিট করার অংশ ---
    if not st.session_state["syllabus_saved"] or st.session_state["syllabus_editing"]:
        form_title = "✏️ সিলেবাস এডিট করুন" if st.session_state["syllabus_editing"] else "১. শ্রেণি এবং সমস্ত বিষয়ের সিলেবাস নির্বাচন করুন"
        st.markdown(f"### {form_title}")
        
        plan_class = st.selectbox("শ্রেণি নির্বাচন করুন:", list(master_syllabus_db.keys()), key="plan_class_sel")
        
        st.markdown("#### প্রতিটি বিষয়ের অধ্যায়সমূহ সিলেক্ট করুন (আপনার স্কুলের সিলেবাস অনুযায়ী টিক দিন বা বাদ দিন):")
        
        selected_subjects_data = {}
        subjects_in_class = master_syllabus_db[plan_class]
        
        for subj, chapters in subjects_in_class.items():
            default_val = chapters
            if st.session_state["syllabus_editing"] and st.session_state["saved_syllabi"]:
                for item in st.session_state["saved_syllabi"]:
                    if item["বিষয়"] == subj:
                        saved_list = [c.strip() for c in item["নির্বাচিত অধ্যাসমূহ"].split(",")]
                        default_val = [c for c in saved_list if c in chapters]
                        break

            chosen_chaps = st.multiselect(
                f"{subj} বিষয়ের অধ্যাসমূহ:",
                chapters,
                default=default_val,
                key=f"multiselect_{plan_class}_{subj}"
            )
            if chosen_chaps:
                selected_subjects_data[subj] = chosen_chaps

        col_b1, col_b2 = st.columns(2)
        with col_b1:
            if st.button("💾 সিলেবাস সেভ করুন", key="save_all_syllabus_btn"):
                if selected_subjects_data:
                    st.session_state["saved_syllabi"] = [
                        {
                            "শ্রেণি": plan_class,
                            "বিষয়": subj,
                            "নির্বাচিত অধ্যাসমূহ": ", ".join(chaps)
                        }
                        for subj, chaps in selected_subjects_data.items()
                    ]
                    st.session_state["syllabus_saved"] = True
                    st.session_state["syllabus_editing"] = False
                    st.success("সিলেবাস সফলভাবে সেভ করা হয়েছে!")
                    st.rerun()
                else:
                    st.warning("দয়া করে অন্তত একটি বিষয়ের অধ্যায় সিলেক্ট করুন।")
        
        with col_b2:
            if st.session_state["syllabus_editing"]:
                if st.button("❌ Cancel (সিলেবাস)", key="cancel_syllabus_edit"):
                    st.session_state["syllabus_editing"] = False
                    st.rerun()
    else:
        st.markdown("### 📋 আপনার সংরক্ষিত সিলেবাস তালিকা:")
        df_syllabus = pd.DataFrame(st.session_state["saved_syllabi"])
        st.table(df_syllabus)

        col_act1, col_act2 = st.columns(2)
        with col_act1:
            if st.button("✏️ Edit সিলেবাস", key="edit_syllabus_btn"):
                st.session_state["syllabus_editing"] = True
                st.rerun()
        with col_act2:
            if st.button("🗑️ Delete সিলেবাস", key="delete_syllabus_btn"):
                st.session_state["saved_syllabi"] = []
                st.session_state["syllabus_saved"] = False
                st.session_state["syllabus_editing"] = False
                st.success("সিলেবাস ডিলিট করা হয়েছে।")
                st.rerun()

    st.markdown("---")

    # --- ২. সাপ্তাহিক পড়ার রুটিন ও সময় নির্ধারণ ---
    days_of_week = ["শনিবার", "রবিবার", "সোমবার", "মঙ্গলবার", "বুধবার", "বৃহস্পতিবার", "শুক্রবার"]

    if not st.session_state["routine_saved"] or st.session_state["routine_editing"]:
        routine_title = "✏️ সাপ্তাহিক রুটিন এডিট করুন" if st.session_state["routine_editing"] else "২. সাপ্তাহিক পড়ার রুটিন ও সময় নির্ধারণ"
        st.markdown(f"### {routine_title}")
        st.markdown("সপ্তাহের ৭ দিনের জন্য বাংলা, ইংরেজি ও গণিত পড়ার সময় স্লাইডার টেনে নির্ধারণ করুন:")
        
        tab_days = st.tabs(days_of_week)
        temp_routine = {}
        for idx, day in enumerate(days_of_week):
            with tab_days[idx]:
                st.markdown(f"**{day} এর পড়ার সময় নির্ধারণ:**")
                def_b = st.session_state["weekly_routine_data"].get(day, {}).get("বাংলা", 1.0) if st.session_state["routine_editing"] else 1.0
                def_e = st.session_state["weekly_routine_data"].get(day, {}).get("ইংরেজি", 1.0) if st.session_state["routine_editing"] else 1.0
                def_m = st.session_state["weekly_routine_data"].get(day, {}).get("গণিত", 1.0) if st.session_state["routine_editing"] else 1.0

                b_time = st.slider(f"বাংলা ({day}) - ঘণ্টা", 0.0, 5.0, float(def_b), 0.5, key=f"r_b_{idx}")
                e_time = st.slider(f"ইংরেজি ({day}) - ঘণ্টা", 0.0, 5.0, float(def_e), 0.5, key=f"r_e_{idx}")
                m_time = st.slider(f"গণিত ({day}) - ঘণ্টা", 0.0, 5.0, float(def_m), 0.5, key=f"r_m_{idx}")
                temp_routine[day] = {"বাংলা": b_time, "ইংরেজি": e_time, "গণিত": m_time}

        col_rb1, col_rb2 = st.columns(2)
        with col_rb1:
            if st.button("💾 রুটিন সেভ করুন", key="save_routine_btn"):
                st.session_state["weekly_routine_data"] = temp_routine
                st.session_state["routine_saved"] = True
                st.session_state["routine_editing"] = False
                st.success("সাপ্তাহিক রুটিন সফলভাবে সেভ করা হয়েছে!")
                st.rerun()
        with col_rb2:
            if st.session_state["routine_editing"]:
                if st.button("❌ Cancel (রুটিন)", key="cancel_routine_edit"):
                    st.session_state["routine_editing"] = False
                    st.rerun()
    else:
        st.markdown("### 📋 আপনার সংরক্ষিত সাপ্তাহিক রুটিন তালিকা:")
        routine_table_data = []
        for day, hours in st.session_state["weekly_routine_data"].items():
            routine_table_data.append({
                "বার": day,
                "বাংলা (ঘণ্টা)": hours["বাংলা"],
                "ইংরেজি (ঘণ্টা)": hours["ইংরেজি"],
                "গণিত (ঘণ্টা)": hours["গণিত"]
            })
        df_routine_saved = pd.DataFrame(routine_table_data)
        st.table(df_routine_saved)

        col_rat1, col_rat2 = st.columns(2)
        with col_rat1:
            if st.button("✏️ Edit রুটিন", key="edit_routine_btn"):
                st.session_state["routine_editing"] = True
                st.rerun()
        with col_rat2:
            if st.button("🗑️ Delete রুটিন", key="delete_routine_btn"):
                st.session_state["weekly_routine_data"] = {}
                st.session_state["routine_saved"] = False
                st.session_state["routine_editing"] = False
                st.success("রুটিন ডিলিট করা হয়েছে।")
                st.rerun()

    st.markdown("---")

    # --- ৩. প্লান তৈরির মোড সিলেকশন ---
    st.markdown("### ৩. স্টাডি প্লান তৈরির পদ্ধতি নির্বাচন করুন")
    plan_mode = st.radio("পদ্ধতি বেছে নিন:", ["Generate with AI", "Make by yourself"], key="study_plan_mode_radio")

    all_available_chapters = []
    if st.session_state["saved_syllabi"]:
        for item in st.session_state["saved_syllabi"]:
            subj_name = item['বিষয়']
            chaps = [c.strip() for c in item["নির্বাচিত অধ্যাসমূহ"].split(",")]
            for c in chaps:
                all_available_chapters.append(f"{subj_name} - {c}")
    else:
        all_available_chapters = ["দয়া করে আগে সিলেবাস সেভ করুন"]

    # --- অপশন ক: Generate with AI ---
    if plan_mode == "Generate with AI":
        col_p1, col_p2 = st.columns(2)
        with col_p1:
            plan_duration_ai = st.selectbox("প্ল্যানের মেয়াদ বেছে নিন:", ["১ সপ্তাহ", "১৫ দিন", "১ মাস", "২ মাস"], key="ai_dur_sel")
        with col_p2:
            target_syllabus_options = all_available_chapters if st.session_state["saved_syllabi"] else ["সাধারণ রিভিশন ও প্র্যাকটিস"]
            target_syllabus_ai = st.selectbox("টার겟 অধ্যায় নির্বাচন করুন:", target_syllabus_options, key="ai_target_sel")

        if st.button("🚀 Generate Smart Study Plan (AI)", key="gen_ai_plan_btn"):
            with st.spinner("এআই বিশ্লেষণ করছে..."):
                routine_source = st.session_state["weekly_routine_data"] if st.session_state["routine_saved"] else {d: {"বাংলা": 1.0, "ইংরেজি": 1.0, "গণিত": 1.0} for d in days_of_week}
                plan_summary_data = []
                for day, hours in routine_source.items():
                    total_hrs = hours['বাংলা'] + hours['ইংরেজি'] + hours['গণিত']
                    plan_summary_data.append({
                        "বার": day,
                        "বাংলা সময়": f"{hours['বাংলা']} ঘণ্টা",
                        "ইংরেজি সময়": f"{hours['ইংরেজি']} ঘণ্টা",
                        "গণিত সময়": f"{hours['গণিত']} ঘণ্টা",
                        "মোট সময়": f"{total_hrs} ঘণ্টা",
                        "টার겟": target_syllabus_ai,
                        "স্ট্যাটাস": "সম্পূর্ণ 🟢"
                    })
                df_plan = pd.DataFrame(plan_summary_data)
                st.session_state["current_generated_plan"] = df_plan
                st.session_state["final_plan_saved_status"] = False
                st.success("এআই প্লান তৈরি হয়েছে!")

    # --- অপশন খ: Make by yourself ---
    else:
        st.markdown("#### 🛠️ নিজে কাস্টম প্লান তৈরি করুন (প্রতি অধ্যায়ের জন্য সর্বনিম্ন ১.৫ ঘণ্টা প্রয়োজন)")
        
        col_m1, col_m2 = st.columns(2)
        with col_m1:
            manual_duration = st.selectbox("সময়কাল বেছে নিন:", ["৭ দিন", "১৫ দিন", "৩০ দিন", "৬০ দিন"], key="manual_dur_sel")
        with col_m2:
            num_days_map = {"৭ দিন": 7, "১৫ দিন": 15, "৩০ দিন": 30, "৬০ দিন": 60}
            total_days_count = num_days_map[manual_duration]

        if not st.session_state["manual_plan_saved"] or st.session_state["manual_plan_editing"]:
            st.markdown(f"**প্রতিদিনের জন্য আলাদা আলাদা লাইনে থাকা অধ্যায়গুলো সিলেক্ট করুন (মেয়াদ: {manual_duration}):**")
            
            sample_days = [f"দিন {i}" for i in range(1, min(total_days_count + 1, 8))]
            
            selected_chaps_per_day = {}
            for d_name in sample_days:
                selected_chaps_per_day[d_name] = st.multiselect(
                    f"{d_name} এ পড়তে চান এমন অধ্যায়সমূহ:",
                    all_available_chapters,
                    key=f"manual_chaps_{d_name}"
                )

            if st.button("💾 ম্যানুয়াল প্লান সেভ করুন", key="save_manual_plan_btn"):
                avg_daily_hours = 2.5 
                min_chapter_hours = 1.5
                
                evaluated_schedule = []
                carry_over_chapters = []

                for d_name, chaps in selected_chaps_per_day.items():
                    all_assigned_today = chaps + carry_over_chapters
                    carry_over_chapters = [] 
                    
                    required_time = len(all_assigned_today) * min_chapter_hours
                    if required_time > avg_daily_hours and len(all_assigned_today) > 1:
                        completed_today = all_assigned_today[:1] 
                        carry_over_chapters = all_assigned_today[1:] 
                        status_str = "অসম্পূর্ণ (পরের দিনের জন্য রোলওভার 🟡)"
                    else:
                        completed_today = all_assigned_today
                        status_str = "সম্পূর্ণভাবে সম্পন্ন হবে 🟢"

                    evaluated_schedule.append({
                        "দিন": d_name,
                        "নির্ধারিত অধ্যায়সমূহ": ", ".join(completed_today) if completed_today else "কোনো অধ্যায় নেই",
                        "রোলওভার অধ্যায়": ", ".join(carry_over_chapters) if carry_over_chapters else "নেই",
                        "স্ট্যাটাস": status_str
                    })

                df_manual = pd.DataFrame(evaluated_schedule)
                st.session_state["current_generated_plan"] = df_manual
                st.session_state["manual_plan_saved"] = True
                st.session_state["manual_plan_editing"] = False
                st.session_state["final_plan_saved_status"] = False
                st.success("আপনার কাস্টম স্টাডি প্লান সফলভাবে ভ্যালিড ও সেভ হয়েছে!")
                st.rerun()
        else:
            st.markdown("### 📋 আপনার সংরক্ষিত কাস্টম প্লান:")
            if "current_generated_plan" in st.session_state:
                st.table(st.session_state["current_generated_plan"])

            col_m_act1, col_m_act2 = st.columns(2)
            with col_m_act1:
                if st.button("✏️ Edit কাস্টম প্লান", key="edit_manual_plan_btn"):
                    st.session_state["manual_plan_editing"] = True
                    st.rerun()
            with col_m_act2:
                if st.button("🗑️ Delete কাস্টম প্লান", key="delete_manual_plan_btn"):
                    st.session_state["manual_plan_saved"] = False
                    st.session_state["manual_plan_editing"] = False
                    st.session_state["current_generated_plan"] = None
                    st.success("কাস্টম প্লান ডিলিট করা হয়েছে।")
                    st.rerun()

    # --- চূড়ান্ত প্ল্যান সেভ করার বাটন (সেভ করার পর এটি ও ইনপুট অপশনগুলো লুকিয়ে যাবে) ---
    if "current_generated_plan" in st.session_state and st.session_state["current_generated_plan"] is not None and not st.session_state["final_plan_saved_status"]:
        st.markdown("---")
        st.markdown("#### 📥 জেনারেটেড স্টাডি প্লান প্রিভিউ:")
        st.table(st.session_state["current_generated_plan"])
        
        if st.button("📥 চূড়ান্ত প্ল্যান ড্যাশবোর্ডে সেভ করুন", key="save_final_plan_btn"):
            st.session_state["saved_study_plans"].append({
                "পদ্ধতি": plan_mode,
                "প্ল্যানের বিবরণ": st.session_state["current_generated_plan"]
            })
            st.session_state["final_plan_saved_status"] = True
            st.balloons()
            st.success("স্টাডি প্ল্যান সফলভাবে আপনার ড্যাশবোর্ডে সংরক্ষণ করা হয়েছে!")
            st.rerun()

    # ড্যাশবোর্ডে সেভ হওয়া স্টাডি প্ল্যানগুলোর তালিকা প্রদর্শন
    if st.session_state["saved_study_plans"]:
        st.markdown("---")
        st.markdown("### 📂 ড্যাশবোর্ডে সংরক্ষিত স্টাডি প্ল্যানসমূহ:")
        for idx, saved_plan in enumerate(st.session_state["saved_study_plans"], 1):
            with st.expander(f"সংরক্ষিত প্ল্যান #{idx} (পদ্ধতি: {saved_plan['পদ্ধতি']})"):
                st.table(saved_plan["প্ল্যানের বিবরণ"])


def render_exams():
    st.markdown("## 📋 Exam & Assignment Schedule")
    st.markdown("Stay ahead of deadlines with upcoming exams and homework assignments.")
    st.info("⚠️ Next Exam: Mathematics on 15 Oct 2025 (12 Days Left)")
    st.success("✅ Assignment Submitted: Science Lab Report (Status: Graded - A+)")

def render_notifications():
    st.markdown("## 🔔 Notifications Center")
    st.markdown("All alerts, announcements, and teacher remarks.")
    st.markdown("- **[2 hours ago]** Tomorrow is your Mathematics exam.")
    st.markdown("- **[4 hours ago]** You have 2 unfinished chapters in Science.")
    st.markdown("- **[1 day ago]** Science assignment is due tomorrow.")

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
            <h3 style="color: white; margin: 0; font-size: 18px;">AI Smart Study</h3>
            <p style="color: #94A3B8; font-size: 11px;">{st.session_state.school_name}</p>
        </div>
        """, unsafe_allow_html=True)
        
        nav_options = [
            ("📊 ড্যাশবোর্ড", "Dashboard"),
            ("📚 আমার পাঠাগার", "My Books"),
            ("🎯 প্র্যাক্টিস ও প্রিপারেশন", "Quiz & Practice"),
            ("📝 আমার নোট খাতা", "Notes"),
            ("📅 পাঠ পরিকল্পনা", "Study Planner"),
            ("📋 পরীক্ষার রুটিন", "Exam & Assignment"),
            ("🔔 রিমাইন্ডার", "Notifications"),
            ("⚙️ সেটিংস", "Profile / Settings")
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


