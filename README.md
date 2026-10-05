import streamlit as st
import pandas as pd
import os
import fitz  # PyMuPDF
import google.generativeai as genai
from datetime import datetime

# --- PAGE CONFIGURATION ---
st.set_page_config(
    page_title="AI Smart Study",
    page_icon="🤖",
    layout="wide",
    initial_sidebar_state="expanded"
)

# --- CUSTOM CSS FOR STYLING & ALIGNMENT ---
st.markdown("""
    <style>
        .school-box {
            background-color: #1e1e1e;
            padding: 20px;
            border-radius: 10px;
            border-left: 5px solid #4A90E2;
            height: 100%;
        }
        .profile-box {
            background-color: #1e1e1e;
            padding: 20px;
            border-radius: 10px;
            border-left: 5px solid #28a745;
            height: 100%;
        }
    </style>
""", unsafe_allow_html=True)

# --- DIRECTORY SETUP ---
OS_FOLDERS = ["study_materials/board_pdfs", "study_materials/practice_pdfs"]
for folder in OS_FOLDERS:
    os.makedirs(folder, exist_ok=True)

# --- GEMINI AI API CONFIGURATION ---
GEMINI_API_KEY = "YOUR_GEMINI_API_KEY_HERE"
if GEMINI_API_KEY != "YOUR_GEMINI_API_KEY_HERE":
    genai.configure(api_key=GEMINI_API_KEY)

# --- PREDEFINED SYLLABUS DATA (CLASS 6 TO 8) ---
PREDEFINED_SYLLABUS = {
    6: {
        "বাংলা ১ম": [
            "১. গদ্য - সততার পুরস্কার", "২. গদ্য - মিনু", "৩. গদ্য - নীল নদ আর পিরামিডের দেশ", 
            "৪. গদ্য - তোলপাড়", "৫. গদ্য - অমর একুশে", "৬. গদ্য - পিছু ডাক", 
            "৭. গদ্য - কতকাল ধরে", "৮. গদ্য - মাদার তেরেসা", "৯. গদ্য - লিখন", 
            "১০. পদ্য - জন্মভূমি", "১১. পদ্য - তিঁতিঁড়", "১২. পদ্য - মানুষ জাতি", 
            "১৩. পদ্য - ঝিঙে ফুল", "১৪. পদ্য - বাঁশী", "১৫. পদ্য - বাঁচতে দাও", 
            "১৬. পদ্য - সুখ", "১৭. পদ্য - আমার বাড়ি", "১৮. পদ্য - ফাগুন মাস", 
            "১৯. আনন্দপাঠ - রবিনসন ক্রুসো", "২০. আনন্দপাঠ - মার্চেন্ট অব ভেনিস", 
            "২১. আনন্দপাঠ - রিপ ভ্যান উইঙ্কল", "২২. আনন্দপাঠ - দ্য জঙ্গল বুক", "২৩. আনন্দপাঠ - কালো হীরা"
        ],
        "বাংলা ২য়": ["১. ভাষা ও বাংলা ভাষা", "২. ধ্বনিতত্ত্ব", "৩. রূপতত্ত্ব"],
        "English 1st": ["1. Going to a new school", "2. Congratulations!", "3. At a railway station"],
        "English 2nd": ["1. Parts of Speech", "2. Tense", "3. Articles"],
        "গণিত": ["১. স্বাভাবিক সংখ্যা ও ভগ্নাংশ", "২. অনুপাত ও শতাংশ", "৩. বীজগণিতীয় রাশি"],
        "বিজ্ঞান": ["১. বৈজ্ঞানিক প্রক্রিয়া ও পরিমাপ", "২. জীবজগত", "৩. উদ্ভিদ ও প্রাণীর কোষীয় সংগঠন"],
        "আইসিটি": ["১. তথ্য ও যোগাযোগ প্রযুক্তি পরিচিতি", "২. কম্পিউটার নেটওয়ার্ক"],
        "বাংলাদেশ ও বিশ্বপরিচয়": ["১. বাংলাদেশের মুক্তিযুদ্ধ", "২. বাংলাদেশ ও বাংলাদেশের নাগরিক"],
        "ধর্ম ও নৈতিক শিক্ষা": ["১. আকাইদ", "২. ইবাদাত", "৩. আখলাক"]
    },
    7: {
        "বাংলা ১ম": ["১. কাবুলিওয়ালা", "২. লখার ২১শে", "৩. শ্রাবণে"],
        "বাংলা ২য়": ["১. ভাষা", "২. ধ্বনি ও বর্ণ", "৩. শব্দ ও পদ"],
        "English 1st": ["1. Attention, please", "2. My daily life", "3. What are friends for?"],
        "English 2nd": ["1. Nouns & Pronouns", "2. Right form of verbs", "3. Prepositions"],
        "গণিত": ["১. মূলদ ও অমূলদ সংখ্যা", "২. অনুপাত ও সমানুপাত", "৩. বীজগণিতীয় সূত্রাবলী"],
        "বিজ্ঞান": ["১. নিম্নশ্রেণীর জীব", "২. উদ্ভিদের বাহ্যিক বৈশিষ্ট্য", "৩. পদার্থের গঠন"],
        "আইসিটি": ["১. ব্যক্তি জীবনে তথ্য ও যোগাযোগ প্রযুক্তি", "২. কম্পিউটার সংশ্লিষ্ট যন্ত্রপাতি"],
        "বাংলাদেশ ও বিশ্বপরিচয়": ["১. বাংলাদেশের স্বাধীনতা সংগ্রাম", "২. বাংলাদেশের সংস্কৃতি"],
        "ধর্ম ও নৈতিক শিক্ষা": ["১. তাওহীদ ও আকাইদ", "২. সালাত ও সাওম", "৩. নৈতিক মূল্যবোধ"]
    },
    8: {
        "বাংলা ১ম": ["১. অতিথি স্মৃতি", "২. ভাব ও কাজ", "৩. রূপসী বাংলা"],
        "বাংলা ২য়": ["১. ভাষা ও ব্যাকরণ", "২. সন্ধি", "৩. সমাস"],
        "English 1st": ["1. Our folk songs", "2. Nakshi Kantha", "3. Good food"],
        "English 2nd": ["1. Transformation of Sentences", "2. Voice Change", "3. Idioms & Phrases"],
        "গণিত": ["১. প্যাটার্ন", "২. মুনাফা", "৩. বীজগণিতীয় সূত্রাবলী ও প্রয়োগ"],
        "বিজ্ঞান": ["১. উন্নত ও অনুন্নত জীব", "২. জীবের বৃদ্ধি ও বংশগতি", "৩. ব্যাপন, অভিস্রবণ ও প্রশ্বেদন"],
        "আইসিটি": ["১. তথ্য ও যোগাযোগ প্রযুক্তির গুরুত্ব", "২. কম্পিউটার নেটওয়ার্ক"],
        "বাংলাদেশ ও বিশ্বপরিচয়": ["১. ঔপনিবেশিক যুগ ও বাংলার স্বাধীনতা সংগ্রাম", "২. বাংলাদেশের অর্থনীতি"],
        "ধর্ম ও নৈতিক শিক্ষা": ["১. আকাইদ ও বিশ্বাস", "২. শরিয়তের উৎস", "৩. ইবাদত"]
    }
}

# --- INITIALIZE SESSION STATE ---
if 'logged_in' not in st.session_state:
    st.session_state.logged_in = False

if 'setup_complete' not in st.session_state:
    st.session_state.setup_complete = False

if 'user_info' not in st.session_state:
    st.session_state.user_info = {}

if 'selected_syllabus' not in st.session_state:
    st.session_state.selected_syllabus = {}

if 'weekly_routine' not in st.session_state:
    st.session_state.weekly_routine = {}

if 'downloaded_materials' not in st.session_state:
    st.session_state.downloaded_materials = {}

if 'test_scores' not in st.session_state:
    st.session_state.test_scores = {}

if 'completed_chapters' not in st.session_state:
    st.session_state.completed_chapters = []

if 'chat_history' not in st.session_state:
    st.session_state.chat_history = []

if 'notes' not in st.session_state:
    st.session_state.notes = ""

if 'profile_pic' not in st.session_state:
    st.session_state.profile_pic = None

# --- 1. LOGIN PAGE ---
def login_page():
    st.markdown("<h1 style='text-align: center; color: #4A90E2;'>🤖 AI Smart Study - Login</h1>", unsafe_allow_html=True)
    st.markdown("<h3 style='text-align: center; color: gray;'>প্রয়োজনীয় তথ্য দিয়ে প্রবেশ করুন</h3>", unsafe_allow_html=True)
    st.markdown("---")
    
    col1, col2, col3 = st.columns([1, 2, 1])
    with col2:
        with st.form("login_form"):
            school = st.text_input("স্কুলের নাম ও ঠিকানা")
            name = st.text_input("শিক্ষার্থীর নাম")
            student_class = st.selectbox("শ্রেণি নির্বাচন করুন", [6, 7, 8])
            branch = st.text_input("শাখা (Section)")
            roll = st.text_input("রোল নম্বর")
            
            submit_button = st.form_submit_button(label="Login ➔")
            
            if submit_button:
                if school and name and student_class and roll:
                    st.session_state.user_info = {
                        "school": school,
                        "name": name,
                        "class": student_class,
                        "branch": branch,
                        "roll": roll
                    }
                    st.session_state.logged_in = True
                    st.rerun()
                else:
                    st.error("সবগুলো প্রয়োজনীয় তথ্য সঠিকভাবে পূরণ করুন!")

# --- 2. SETUP WIZARD (ONBOARDING) ---
def setup_wizard_page():
    st.markdown("## 📖 AI Smart Study - সিলেবাস ও রুটিন সেটআপ")
    
    st.info(f"হাই! **{st.session_state.user_info['name']}**! class {st.session_state.user_info['class']} এর জন্য নির্ধারিত বিষয়সমূহ নিচে উল্লেখ করা হলো। এখান থেকে আপনার প্রয়োজনমতো সিলেবাস এবং রুটিন কাস্টমাইজ করে নিন।")
    
    st.markdown("---")
    st.markdown("### ১. সিলেবাস তৈরি করুন")
    
    student_cls = st.session_state.user_info['class']
    class_syllabus = PREDEFINED_SYLLABUS.get(student_cls, {})
    
    user_selected_syllabus = {}
    
    for sub, chapters in class_syllabus.items():
        with st.expander(f"📘 {sub}"):
            selected_chaps = st.multiselect(
                f"{sub} বিষয় থেকে যেসব অধ্যায় পড়তে চান:",
                options=chapters,
                default=chapters,
                key=f"setup_{sub}"
            )
            user_selected_syllabus[sub] = selected_chaps

    st.markdown("---")
    st.markdown("### ২. সাপ্তাহিক রুটিন তৈরি করুন")
    
    days = ["শনিবার", "রবিবার", "সোমবার", "মঙ্গলবার", "বুধবার", "বৃহস্পতিবার", "শুক্রবার"]
    weekly_routine = {}
    
    for day in days:
        with st.expander(f"📅 {day} - এর সময়সূচি"):
            day_schedule = {}
            for sub in user_selected_syllabus.keys():
                if len(user_selected_syllabus[sub]) > 0:
                    hours = st.number_input(
                        f"{sub} (ঘণ্টায়)", 
                        min_value=0.0, 
                        max_value=6.0, 
                        value=1.0, 
                        step=0.5, 
                        key=f"{day}_{sub}"
                    )
                    if hours > 0:
                        day_schedule[sub] = hours
            weekly_routine[day] = day_schedule

    st.markdown("---")
    if st.button("পরবর্তী পেইজে যান ➔"):
        st.session_state.selected_syllabus = user_selected_syllabus
        st.session_state.weekly_routine = weekly_routine
        st.session_state.setup_complete = True
        st.success("আপনার সিলেবাস এবং রুটিন সফলভাবে সেভ হয়েছে!")
        st.rerun()

# --- 3. HOME PAGE & DASHBOARD ---
def home_page():
    col_left, col_right = st.columns([1, 1], gap="medium")
    
    with col_left:
        st.markdown('<div class="school-box">', unsafe_allow_html=True)
        st.markdown("#### 🏫 প্রতিষ্ঠানের তথ্য")
        school_info = st.session_state.user_info.get('school', 'স্কুলের নাম ও ঠিকানা দেওয়া হয়নি')
        st.markdown(f"**স্কুল ও ঠিকানা:**<br><br>{school_info}", unsafe_allow_html=True)
        st.markdown('</div>', unsafe_allow_html=True)
        
    with col_right:
        st.markdown('<div class="profile-box">', unsafe_allow_html=True)
        st.markdown("#### 👤 Student Profile")
        
        uploaded_file = st.file_uploader("প্রোফাইল ছবি আপলোড করুন", type=["jpg", "jpeg", "png"], key="dash_profile_upload")
        if uploaded_file is not None:
            st.session_state.profile_pic = uploaded_file
            
        cols = st.columns([1, 2], gap="small")
        with cols[0]:
            if st.session_state.profile_pic is not None:
                st.image(st.session_state.profile_pic, width=100)
            else:
                st.markdown("📷 *(ছবি নেই)*")
                
        with cols[1]:
            name = st.session_state.user_info.get('name', 'নাম')
            cls = st.session_state.user_info.get('class', '-')
            branch = st.session_state.user_info.get('branch', '-')
            roll = st.session_state.user_info.get('roll', '-')
            
            st.markdown(
                f"**নাম:** {name}<br>"
                f"**শ্রেণি:** {cls}<br>"
                f"**শাখা:** {branch}<br>"
                f"**রোল নম্বর:** {roll}", 
                unsafe_allow_html=True
            )
        st.markdown('</div>', unsafe_allow_html=True)
            
    st.markdown("---")
    st.markdown("## 🏠 AI Smart Study-Dashboard")
    
    total_chapters = sum(len(chaps) for chaps in st.session_state.selected_syllabus.values())
    completed_chaps = len(st.session_state.completed_chapters)
    completion_rate = int((completed_chaps / total_chapters * 100)) if total_chapters > 0 else 0

    c1, c2, c3 = st.columns(3, gap="medium")
    with c1:
        st.info(f"📊 **Syllabus Progress**\n\nসিলেবাস সম্পন্ন: **{completion_rate}%** ({completed_chaps}/{total_chapters} পাঠ)")
    with c2:
        st.warning("🔔 **Today's Target**\n\nআজকের নির্ধারিত অধ্যায়ের পড়া শেষ করুন ও কুইজ দিন")
    with c3:
        day_map = {
            "Saturday": "শনিবার", "Sunday": "রবিবার", "Monday": "সোমবার", 
            "Tuesday": "মঙ্গলবার", "Wednesday": "বুধবার", "Thursday": "বৃহস্পতিবার", "Friday": "শুক্রবার"
        }
        today_bn = day_map.get(datetime.now().strftime("%A"), "শনিবার")
        today_routine = st.session_state.weekly_routine.get(today_bn, {})
        total_today_hours = sum(today_routine.values())
        
        st.success(f"🎯 **Today's Routine ({today_bn})**\n\nআজকের টার্গেট: **{total_today_hours} ঘণ্টা**")

    st.markdown("---")
    st.markdown("### 📅 আজকের অধ্যয়নের সময়সূচী")
    if today_routine:
        for sub, hrs in today_routine.items():
            st.markdown(f"- **{sub}:** {hrs} ঘণ্টা")
    else:
        st.write("আজ কোনো পড়াশোনার বিষয় রুটিনে যুক্ত করা নেই।")

# --- HELPER FUNCTION FOR SIMULATED SYSTEM PDF DOWNLOAD ---
def get_system_file_bytes(sub, chap, file_type):
    folder = "study_materials/board_pdfs" if file_type == "board" else "study_materials/practice_pdfs"
    safe_chap = chap.replace(" ", "_").replace(".", "")
    file_path = os.path.join(folder, f"{sub}_{safe_chap}_{file_type}.pdf")
    
    if not os.path.exists(file_path):
        with open(file_path, "w", encoding="utf-8") as f:
            f.write(f"--- AI SMART STUDY OFFICIAL MATERIAL ---\nSubject: {sub}\nChapter: {chap}\nType: {file_type.upper()}")
            
    with open(file_path, "rb") as f:
        return f.read(), os.path.basename(file_path)

# --- 4. MY BOOKS & READING ---
def my_books_page():
    st.markdown("## 📖 My Books & Reading")
    st.caption("আমাদের সিস্টেমে আগে থেকে সংরক্ষিত বোর্ড বই ও মডেল টেস্ট ফাইল প্রয়োজন অনুযায়ী ডাউনলোড করে পড়ুন।")
    
    tab1, tab2 = st.tabs(["📥 বোর্ড ও প্র্যাকটিস ফাইল ডাউনলোড", "📖 রিডিং মোড ও AI অ্যাসিস্ট্যান্ট"])
    
    with tab1:
        st.markdown("### 📚 আপনার নির্বাচিত সিলেবাসের প্রস্তুতকৃত ফাইলসমূহ")
        for sub, chaps in st.session_state.selected_syllabus.items():
            if chaps:
                with st.expander(f"📘 {sub}"):
                    for chap in chaps:
                        st.markdown(f"**পাঠ/অধ্যায়:** {chap}")
                        c1, c2 = st.columns(2)
                        
                        with c1:
                            board_bytes, file_name_b = get_system_file_bytes(sub, chap, "board")
                            if st.download_button(
                                label=f"📄 {chap} - বোর্ড PDF ডাউনলোড",
                                data=board_bytes,
                                file_name=file_name_b,
                                mime="text/plain",
                                key=f"board_{sub}_{chap}"
                            ):
                                st.session_state.downloaded_materials[f"{sub} - {chap} (Board)"] = board_bytes
                                st.toast(f"'{chap}' বোর্ড বই ডাউনলোড সম্পন্ন!")

                        with c2:
                            prac_bytes, file_name_p = get_system_file_bytes(sub, chap, "practice")
                            if st.download_button(
                                label=f"📝 {chap} - মডেল ও MCQ PDF",
                                data=prac_bytes,
                                file_name=file_name_p,
                                mime="text/plain",
                                key=f"prac_{sub}_{chap}"
                            ):
                                st.session_state.downloaded_materials[f"{sub} - {chap} (Practice)"] = prac_bytes
                                st.toast(f"'{chap}' প্র্যাকটিস ফাইল ডাউনলোড সম্পন্ন!")
                        
                        st.markdown("---")

    with tab2:
        st.markdown("### 📖 রিডিং স্পেস ও AI হেল্পার")
        
        if not st.session_state.downloaded_materials:
            st.warning("⚠️ আপনি এখনো কোনো ফাইল ডাউনলোড করেননি! প্রথমে ফাইল ডাউনলোড করে নিন।")
        else:
            col_read, col_ai = st.columns([3, 2], gap="medium")
            
            with col_read:
                st.markdown("#### 📄 পাঠ্য সম্বলিত উইন্ডো")
                selected_file_key = st.selectbox(
                    "পড়ার জন্য ডাউনলোডকৃত ফাইল বাছুন:", 
                    list(st.session_state.downloaded_materials.keys())
                )
                
                file_data = st.session_state.downloaded_materials[selected_file_key]
                text_content = file_data.decode("utf-8", errors="ignore")
                
                st.text_area(
                    "পড়ার অংশ:", 
                    value=text_content, 
                    height=350,
                    key="reading_box"
                )

            with col_ai:
                st.markdown("#### 🤖 AI চ্যাটবট")
                user_prompt = st.text_area("আপনার প্রশ্ন বা না বোঝা অংশ:", height=120)
                
                if st.button("AI-কে জিজ্ঞাসা করুন"):
                    if user_prompt:
                        try:
                            model = genai.GenerativeModel('gemini-pro')
                            response = model.generate_content(
                                f"তুমি একজন সহায়ক গৃহশিক্ষক। এই পড়াটি সহজ ভাষায় বুঝিয়ে দাও: {user_prompt}"
                            )
                            st.session_state.chat_history.append(("user", user_prompt))
                            st.session_state.chat_history.append(("ai", response.text))
                        except Exception as e:
                            st.error("Gemini API কানেক্ট করা যায়নি। API Key পরীক্ষা করুন।")
                    else:
                        st.warning("দয়া করে কিছু প্রশ্ন লিখুন।")
                
                st.markdown("---")
                st.markdown("##### 💬 উত্তরসমূহ:")
                for role, msg in st.session_state.chat_history[-4:]:
                    if role == "user":
                        st.markdown(f"**প্রশ্ন:** {msg}")
                    else:
                        st.success(f"**AI উত্তর:** {msg}")

# --- 5. PROGRESS ANALYSIS ---
def progress_page():
    st.markdown("## 📈 প্রোগ্রেস অ্যানালিসিস")
    st.markdown("এখানে আপনার সিলেবাস কমপ্লিশন ও টেস্ট পারফরম্যান্স বিশ্লেষণ করা হয়েছে:")
    st.markdown("---")
    
    col1, col2 = st.columns(2, gap="medium")
    total_chaps = sum(len(chaps) for chaps in st.session_state.selected_syllabus.values())
    completed = len(st.session_state.completed_chapters)
    syllabus_percent = (completed / total_chaps * 100) if total_chaps > 0 else 0
    
    with col1:
        st.markdown("### (১) সিলেবাস কমপ্লিট")
        st.progress(syllabus_percent / 100)
        st.write(f"**অগ্রগতি:** {syllabus_percent:.1f}%")
        st.caption("নোট: কোনো অধ্যায়ের টেস্টে পাস (কমপক্ষে ৮০% মার্কস) করলে সেটি 'কমপ্লিট' হিসেবে গণনাকৃত হবে।")
        
    with col2:
        st.markdown("### (২) টেস্ট প্রোগ্রেস")
        scores = st.session_state.test_scores
        if scores:
            avg_score = sum(scores.values()) / len(scores)
            st.metric(label="গড় টেস্ট স্কোর", value=f"{avg_score:.1f}%")
            st.write(f"মোট দেওয়া টেস্ট: **{len(scores)}** টি")
        else:
            st.info("এখনো কোনো টেস্ট দেওয়া হয়নি।")

# --- 6. QUIZ & PRACTICE ---
def quiz_page():
    st.markdown("## 🧠 Quiz & Practice Test")
    all_subs = list(st.session_state.selected_syllabus.keys())
    if not all_subs:
        st.warning("প্রথমে আপনার সিলেবাস নির্ধারণ করুন!")
        return

    selected_sub = st.selectbox("বিষয় নির্বাচন করুন", all_subs)
    available_chaps = st.session_state.selected_syllabus.get(selected_sub, [])
    
    if available_chaps:
        selected_chap = st.selectbox("অধ্যায়/পাঠ নির্বাচন করুন", available_chaps)
        st.markdown(f"### 📝 {selected_sub} - {selected_chap} এর টেস্ট")
        
        with st.form("quiz_form"):
            q1 = st.radio("১. সঠিক উত্তরটি চিহ্নিত করুন (নমুনা প্রশ্ন):", ["ক. বিকল্প ১", "খ. সঠিক বিকল্প ২", "গ. বিকল্প ৩"])
            submit_quiz = st.form_submit_button("টেস্ট জমা দিন")
            
            if submit_quiz:
                score = 100 if q1 == "খ. সঠিক বিকল্প ২" else 40
                st.session_state.test_scores[f"{selected_sub}_{selected_chap}"] = score
                
                if score >= 80:
                    if selected_chap not in st.session_state.completed_chapters:
                        st.session_state.completed_chapters.append(selected_chap)
                    st.success(f"🎉 অভিনন্দন! তুমি {score}% পেয়ে পাস করেছেন। অধ্যায়টি 'কমপ্লিট' চিহ্নিত করা হয়েছে।")
                else:
                    st.error(f"❌ তুমি পেয়েছেন {score}%। উত্তীর্ণ হতে কমপক্ষে ৮০% দরকার। আবার চেষ্টা করুন।")
    else:
        st.info("এই বিষয়ে কোনো অধ্যায় নির্বাচন করা হয়নি।")

# --- 7. ROUTINE & NOTES ---
def routine_page():
    st.markdown("## ⏰ তোমার সাপ্তাহিক রুটিন")
    for day, schedule in st.session_state.weekly_routine.items():
        with st.expander(f"📅 {day}"):
            if schedule:
                for sub, hrs in schedule.items():
                    st.markdown(f"- **{sub}:** {hrs} ঘণ্টা")
            else:
                st.write("কোনো রুটিন সেট করা নেই।")

def notes_page():
    st.markdown("## 📝 Notes & Quick Memo")
    user_note = st.text_area("এখানে আপনার গুরুত্বপূর্ণ তথ্য বা পড়ার নোট লিখুন:", value=st.session_state.notes, height=250)
    if st.button("নোট সেভ করুন"):
        st.session_state.notes = user_note
        st.success("নোট সফলভাবে সেভ করা হয়েছে!")

def settings_page():
    st.markdown("## ⚙️ Settings")
    st.info("সেটিংস পেজের কাজ পরবর্তীতে আপডেট করা হবে।")

# --- MAIN NAVIGATION & ROUTING LOGIC ---
if not st.session_state.logged_in:
    login_page()
elif not st.session_state.setup_complete:
    setup_wizard_page()
else:
    st.sidebar.markdown("### 🤖 AI Smart Study")
    
    if st.session_state.profile_pic is not None:
        st.sidebar.image(st.session_state.profile_pic, width=80)
    
    st.sidebar.markdown(f"স্বাগতম, **{st.session_state.user_info['name']}**")
    st.sidebar.markdown(f"<small>স্কুল: {st.session_state.user_info['school']}</small>", unsafe_allow_html=True)
    st.sidebar.markdown(f"<small>শ্রেণি: {st.session_state.user_info['class']} | রোল: {st.session_state.user_info['roll']}</small>", unsafe_allow_html=True)
    st.sidebar.markdown("---")
    
    menu = st.sidebar.radio(
        "মেনু নির্বাচন করুন",
        [
            "🏠 Dashboard", 
            "📈 Study Progress",
            "📚 My Books & Reading", 
            "🧠 Quiz & Practice", 
            "⏰ Routine", 
            "📝 Notes",
            "⚙️ Settings"
        ]
    )
    
    st.sidebar.markdown("---")
    if st.sidebar.button("🚪 লগআউট"):
        st.session_state.logged_in = False
        st.session_state.setup_complete = False
        st.session_state.profile_pic = None
        st.rerun()

    # Routing
    if menu == "🏠 Dashboard":
        home_page()
    elif menu == "📈 Study Progress":
        progress_page()
    elif menu == "📚 My Books & Reading":
        my_books_page()
    elif menu == "🧠 Quiz & Practice":
        quiz_page()
    elif menu == "⏰ Routine":
        routine_page()
    elif menu == "📝 Notes":
        notes_page()
    elif menu == "⚙️️ Settings":
        settings_page()
