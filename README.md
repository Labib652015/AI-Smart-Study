# AI-Smart-Study
import streamlit as st
import pandas as pd
import os
import urllib.request
import urllib.parse
from datetime import datetime

# Page Configuration
st.set_page_config(
    page_title="AI Smart Study",
    page_icon="📚",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Initialize Session State variables
if 'logged_in' not in st.session_state:
    st.session_state.logged_in = False

if 'user_info' not in st.session_state:
    st.session_state.user_info = {
        "school": "",
        "location": "",
        "name": "",
        "class": "",
        "branch": "",
        "roll": ""
    }

if 'uploaded_books_data' not in st.session_state:
    st.session_state.uploaded_books_data = {}

if 'notes' not in st.session_state:
    st.session_state.notes = ""

# Create temporary directories locally
os.makedirs("temp_books", exist_ok=True)
os.makedirs("syllabus", exist_ok=True)

# Function to check internet connection
def check_internet():
    try:
        urllib.request.urlopen('http://google.com', timeout=2)
        return True
    except:
        return False


# --- 1. LOGIN PAGE ---
def login_page():
    st.markdown("<h1 style='text-align: center; color: #4A90E2;'>🤖 AI Smart Study</h1>", unsafe_allow_html=True)
    st.markdown("<h3 style='text-align: center; color: gray;'>আপনার তথ্য দিয়ে লগইন করুন</h3>", unsafe_allow_html=True)
    
    st.markdown("---")
    
    col1, col2, col3 = st.columns([1, 2, 1])
    with col2:
        with st.form("login_form"):
            school = st.text_input("স্কুলের নাম ও লোকেশন")
            name = st.text_input("শিক্ষার্থীর নাম")
            student_class = st.text_input("শ্রেণি")
            branch = st.text_input("শাখা")
            roll = st.text_input("রোল")
            
            submit_button = st.form_submit_button(label="পরবর্তী ধাপে যাও")
            
            if submit_button:
                if school and name and student_class and roll:
                    st.session_state.user_info = {
                        "school": school,
                        "location": school,
                        "name": name,
                        "class": student_class,
                        "branch": branch,
                        "roll": roll
                    }
                    st.session_state.logged_in = True
                    st.rerun()
                else:
                    st.error("দয়া করে সব প্রয়োজনীয় তথ্য পূরণ করুন!")


# --- 2. HOME PAGE & DASHBOARD ---
def home_page():
    col_l, col_r = st.columns([4, 1])
    with col_l:
        st.markdown("## 🤖 AI Smart Study Dashboard")
    with col_r:
        st.markdown(f"👤 **{st.session_state.user_info['name']}**<br><small>রোল: {st.session_state.user_info['roll']} | শ্রেণি: {st.session_state.user_info['class']}</small>", unsafe_allow_html=True)
    
    st.markdown("---")
    
    c1, c2, c3 = st.columns(3)
    with c1:
        st.info("📊 **Study Progress**\n\nপড়ার অগ্রগতি: ৬০% সম্পন্ন")
    with c2:
        st.warning("🔔 **Upcoming Exams**\n\nআগামীকাল গণিত পরীক্ষা")
    with c3:
        st.success("🎯 **Today's Study Plan**\n\nবাংলা ১ম পত্র ও ইংরেজি গ্রামার")

    st.markdown("### আজকের পড়াশোনা ও সময়")
    st.write(f"🏫 **স্কুল:** {st.session_state.user_info['school']}")
    st.write("⏱ আজকের নির্ধারিত পড়ার সময়: ৩ ঘণ্টা")


# --- 3. MY BOOKS & READING ---
def my_books_page():
    st.markdown("## 📖 My Books & Reading")
    st.markdown("এখানে আপনার পাঠ্যবই এবং সহায়ক বইগুলো আপলোড করতে পারবেন এবং পড়তে পারবেন।")
    
    tab1, tab2 = st.tabs(["📚 বই আপলোড করো", "📖 পড়া শুরু করো"])
    
    with tab1:
        st.markdown("### বই আপলোড সেকশন")
        book_type = st.selectbox("বইয়ের ধরণ নির্বাচন করুন", ["বোর্ড বই (Textbook)", "সহায়ক বা গাইড বই", "অন্যান্য বই"])
        uploaded_file = st.file_uploader("PDF ফাইল আপলোড করুন", type=["pdf"])
        
        if uploaded_file is not None:
            if st.button("বই সংরক্ষণ করুন"):
                file_path = os.path.join("temp_books", uploaded_file.name)
                with open(file_path, "wb") as f:
                    f.write(uploaded_file.getbuffer())
                
                st.session_state.uploaded_books_data[uploaded_file.name] = {
                    "type": book_type,
                    "path": file_path
                }
                st.success(f"'{uploaded_file.name}' সফলভাবে আপলোড ও সংরক্ষিত হয়েছে!")
                
    with tab2:
        st.markdown("### পড়া শুরু করুন")
        if st.session_state.uploaded_books_data:
            book_names = list(st.session_state.uploaded_books_data.keys())
            selected_book = st.selectbox("পড়ার জন্য একটি বই নির্বাচন করুন", book_names)
            
            if selected_book:
                st.markdown(f"--- \n### 📚 {selected_book} - রিডিং মোড")
                
                file_path = st.session_state.uploaded_books_data[selected_book]["path"]
                
                with open(file_path, "rb") as pdf_file:
                    PDFbyte = pdf_file.read()
                
                if check_internet():
                    st.download_button(
                        label="📖 বইটি পড়তে এখানে ক্লিক করুন",
                        data=PDFbyte,
                        file_name=selected_book,
                        mime='application/pdf'
                    )
                else:
                    st.warning("⚠️ ইন্টারনেট সংযোগ করুন")
                
                st.markdown("---")
                sample_text = st.text_area("বইয়ের কোনো লাইন বুঝতে না পারলে এখানে কপি-পেস্ট করুন:", "এখানে টেক্সট দিন...")
                
                if st.button("🤖 এআই অ্যাসিস্ট্যান্টের সাহায্য নাও"):
                    if sample_text and sample_text != "এখানে টেক্সট দিন...":
                        st.markdown("💡 **নিচের বক্স থেকে প্রম্পটটি কপি করুন:**")
                        st.code(sample_text, language="text")
                        
                        st.markdown(
                            """
                            <a href="https://gemini.google.com/app" target="_blank">
                                <button style="background-color: #4A90E2; color: white; padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; margin-top: 10px;">
                                    🚀 সরাসরি জেমিনি চ্যাটবটে ওপেন করুন
                                </button>
                            </a>
                            """,
                            unsafe_allow_html=True
                        )
                        st.success("ওপরের বক্সের ডান পাশের কোণায় ক্লিক করে সহজে টেক্সট কপি করুন, এরপর নীল বাটনে ক্লিক করে জেমিনিতে পেস্ট করে দিন!")
                    else:
                        st.warning("দয়া করে বোঝার জন্য টেক্সট দিন।")
        else:
            st.warning("কোনো বই আপলোড করা হয়নি। দয়া করে প্রথমে 'বই আপলোড করো' ট্যাব থেকে পিডিএফ বই আপলোড করুন।")


# --- 4. SYLLABUS ---
def syllabus_page():
    st.markdown("## 📑 সিলেবাস তৈরি ও ব্যবস্থাপনা")
    
    tab_create, tab_view = st.tabs(["✍️ নতুন সিলেবাস তৈরি", "📂 সংরক্ষিত সিলেবাসসমূহ"])
    
    with tab_create:
        subject = st.selectbox("বিষয় নির্বাচন করুন", ["বাংলা প্রথম পত্র", "বাংলা দ্বিতীয় পত্র", "ইংরেজি প্রথম পত্র", "ইংরেজি দ্বিতীয় পত্র", "গণিত", "বিজ্ঞান", "আইসিটি"], key="syllabus_subject")
        chapters = st.text_area("অধ্যায় বা টপিকগুলোর নাম লিখুন (প্রতি লাইনে একটি)")
        
        if st.button("সিলেবাস তৈরি করুন"):
            if chapters:
                timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
                file_name = f"{subject.replace(' ', '_')}_{timestamp}.txt"
                file_path = os.path.join("syllabus", file_name)
                
                with open(file_path, "w", encoding="utf-8") as f:
                    f.write(f"বিষয়: {subject}\nতারিখ: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\nঅধ্যায় ও টপিকসমূহ:\n{chapters}")
                
                st.success(f"'{subject}' বিষয়ের সিলেবাস 'syllabus' ফোল্ডারে সফলভাবে সংরক্ষিত হয়েছে!")
            else:
                st.warning("দয়া করে অধ্যায় বা টপিকগুলোর নাম লিখুন।")
                
    with tab_view:
        st.markdown("### 📂 সিলেবাস বইয়ের তালিকা")
        saved_syllabi = sorted([f for f in os.listdir("syllabus") if f.endswith('.txt')], reverse=True)
        
        if saved_syllabi:
            selected_syllabus = st.selectbox(
                "পড়ার জন্য একটি সিলেবাস নির্বাচন করুন", 
                saved_syllabi,
                format_func=lambda x: f"📖 {x.replace('_', ' ').replace('.txt', '')}"
            )
            
            if selected_syllabus:
                file_path = os.path.join("syllabus", selected_syllabus)
                with open(file_path, "r", encoding="utf-8") as f:
                    content = f.read()
                
                display_name = selected_syllabus.replace('_', ' ').replace('.txt', '')
                st.markdown(f"--- \n### 📖 সিলেবাস বুক: {display_name}")
                st.text_area("সিলেবাসের বিস্তারিত বিবরণ:", content, height=200, key="view_syllabus_box")
                
                st.download_button(
                    label="📥 সিলেবাস ফাইল ডাউনলোড করুন",
                    data=content,
                    file_name=selected_syllabus,
                    mime="text/plain"
                )
        else:
            st.info("এখনো কোনো সিলেবাস সংরক্ষণ করা হয়নি। 'নতুন সিলেবাস তৈরি' ট্যাব থেকে সিলেবাস তৈরি করুন।")


# --- 5. ROUTINE ---
def routine_page():
    st.markdown("## ⏰ পড়ার রুটিন")
    day = st.selectbox("বার নির্বাচন করুন", ["শনিবার", "রবিবার", "সোমবার", "মঙ্গলবার", "বুধবার", "বৃহস্পতিবার", "শুক্রবার"])
    schedule_details = st.text_area("সময়ের তালিকা ও বিষয়ের নাম লিখুন")
    if st.button("রুটিন সেভ করুন"):
        st.success(f"{day}-এর রুটিন সফলভাবে সেভ করা হয়েছে!")


# --- 6. EXAM & ASSIGNMENT ---
def exam_page():
    st.markdown("## 📝 Exam & Assignment")
    st.info("আগামীকাল সকাল ১০:০০ টায় গণিত পরীক্ষা রয়েছে।")


# --- 7. NOTES & AI ASSISTANT ---
def notes_page():
    st.markdown("## 📝 Notes & AI Assistant")
    col1, col2 = st.columns(2)
    with col1:
        st.markdown("### নোট খাতা")
        user_note = st.text_area("এখানে আপনার নোট লিখুন বা পেস্ট করুন:", value=st.session_state.notes, height=300)
        if st.button("নোট সেভ করুন"):
            st.session_state.notes = user_note
            st.success("নোট সফলভাবে সংরক্ষিত হয়েছে!")
    with col2:
        st.markdown("### 🤖 AI Assistant")
        ai_query = st.text_input("নোট বা পড়া সম্পর্কিত কোনো প্রশ্ন থাকলে এখানে লিখুন:")
        
        if st.button("জিজ্ঞেস করুন"):
            if ai_query:
                st.markdown("💡 **নিচের বক্সের ওপর ক্লিক করে বা কপি বাটনে প্রেস করে প্রশ্নটি কপি করুন:**")
                st.code(ai_query, language="text")
                
                gemini_url = "https://gemini.google.com/app"
                st.markdown(
                    f"""
                    <a href="{gemini_url}" target="_blank">
                        <button style="background-color: #4A90E2; color: white; padding: 10px 20px; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; margin-top: 10px;">
                            🚀 সরাসরি জেমিনি চ্যাটবটে ওপেন করুন
                        </button>
                    </a>
                    """,
                    unsafe_allow_html=True
                )
                st.success("সফল! কোড বক্সের কর্নারে থাকা 'Copy' আইকনে ক্লিক করে প্রশ্নটি কপি করে নিন, এরপর নীল বাটনে ক্লিক করে জেমিনিতে পেস্ট করে দিন।")
            else:
                st.warning("দয়া করে কোনো প্রশ্ন লিখুন।")


# --- 8. QUIZ & PRACTICE ---
def quiz_page():
    st.markdown("## 🧠 Quiz & Practice")
    q_subject = st.selectbox("কুইজের বিষয়", ["বাংলা", "ইংরেজি", "গণিত", "বিজ্ঞান"])
    if st.button("কুইজ শুরু করো"):
        st.write(f"**{q_subject}** বিষয়ের উপর কুইজ শুরু হয়েছে...")
        st.radio("১. সঠিক উত্তরটি নির্বাচন করুন:", ["অপশন ক", "অপশন খ", "অপশন গ", "অপশন ঘ"])
        if st.button("উত্তর জমা দিন"):
            st.success("সঠিক উত্তর! আপনার স্কোর: ১/১")


# --- 9. SETTINGS ---
def settings_page():
    st.markdown("## ⚙️ Settings")
    theme = st.selectbox("থিম নির্বাচন করুন", ["লাইট মোড", "ডার্ক মোড"])
    app_name = st.text_input("অ্যাপের নাম পরিবর্তন", value="AI Smart Study")
    if st.button("সেটিংস সেভ করুন"):
        st.success("সেটিংস সফলভাবে আপডেট করা হয়েছে!")


# --- MAIN APP ROUTING (SIDEBAR NAVIGATION) ---
if not st.session_state.logged_in:
    login_page()
else:
    st.sidebar.markdown(f"### 🤖 AI Smart Study")
    st.sidebar.markdown(f"স্বাগতম, **{st.session_state.user_info['name']}**")
    st.sidebar.markdown("---")
    
    menu = st.sidebar.radio(
        "মেনু নির্বাচন করুন",
        [
            "🏠 ড্যাশবোর্ড", 
            "📚 My Books & Reading", 
            "📑 সিলেবাস", 
            "⏰ রুটিন", 
            "📝 Exam & Assignment", 
            "📝 Notes & AI Assistant", 
            "🧠 Quiz & Practice", 
            "⚙️ Settings"
        ]
    )
    
    st.sidebar.markdown("---")
    if st.sidebar.button("লগআউট"):
        st.session_state.logged_in = False
        st.rerun()

    (
        home_page() if menu == "🏠 ড্যাশবোর্ড" else
        my_books_page() if menu == "📚 My Books & Reading" else
        syllabus_page() if menu == "📑 সিলেবাস" else
        routine_page() if menu == "⏰ রুটিন" else
        exam_page() if menu == "📝 Exam & Assignment" else
        notes_page() if menu == "📝 Notes & AI Assistant" else
        quiz_page() if menu == "🧠 Quiz & Practice" else
        settings_page()
    )
