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
        <h2 style='color: #1E3A8A; font-weight: 700; margin-bottom: 5px;'> নিচে তোমার জন্য একটি ওয়ার্ড এডিটর দেওয়া হলো </h2>
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
        st.markdown("###  অথবা ")
        st.markdown("### 📥 Save As (ফানাইল নোট পিডিএফ/ওয়ার্ড ফাইল ইত্যাদি ডাউনলোড করতে চাইল...)")
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
            st.markdown("### 📚 তোমার সেভ করা নোটসমূহ")

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
        st.markdown("### 🤖 কোনো প্রশ্ন থাকলে নিচে Gemini-কে জিজ্ঞেস করো এবং তোমার নোট খাতায় নোট করে রাখো।")
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
            <h2>AI Smart Study - পরীক্ষার রুটিন ও সময়সূচী</h2>
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
            <h3 style="color: white; margin: 0; font-size: 18px;">AI Smart Study</h3>
            <p style="color: #94A3B8; font-size: 11px;">{st.session_state.school_name}</p>
        </div>
        """, unsafe_allow_html=True)
        
        nav_options = [
            ("📊 ড্যাশবোর্ড", "Dashboard"),
            ("📚 আমার পাঠাগার", "My Books"),
            ("🎯 প্র্যাক্টিস অ্যান্ড প্রিপারেশন", "Quiz & Practice"),
            ("📝 আমার নোট খাতা", "Notes"),
            ("📅 সিলেবাস ও পাঠ পরিকল্পনা", "Study Planner"),
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


