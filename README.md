import streamlit as st
import streamlit.components.v1 as components
import base64
import os
import tempfile

# --- Page Configuration ---
st.set_page_config(page_title="AI Resume Builder & ATS Optimizer", page_icon="📄", layout="wide")

# --- Custom CSS Styling ---
st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Playfair+Display:wght@400;500;600;700&display=swap');

    .stApp {
        background: linear-gradient(135deg, #0f0c29 0%, #1a1a3e 40%, #24243e 100%);
    }

    /* Hide default Streamlit header and footer */
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}

    /* ===== HERO SECTION ===== */
    .hero-container {
        text-align: center;
        padding: 60px 20px 40px;
        position: relative;
        overflow: hidden;
    }
    .hero-container::before {
        content: '';
        position: absolute;
        top: -50%;
        left: -50%;
        width: 200%;
        height: 200%;
        background: radial-gradient(circle at 30% 50%, rgba(99, 102, 241, 0.15) 0%, transparent 50%),
                    radial-gradient(circle at 70% 50%, rgba(168, 85, 247, 0.12) 0%, transparent 50%);
        animation: float-bg 15s ease-in-out infinite;
    }
    @keyframes float-bg {
        0%, 100% { transform: translateY(0) rotate(0deg); }
        50% { transform: translateY(-20px) rotate(3deg); }
    }
    .hero-badge {
        display: inline-block;
        background: linear-gradient(135deg, rgba(99,102,241,0.3), rgba(168,85,247,0.3));
        border: 1px solid rgba(99,102,241,0.4);
        border-radius: 30px;
        padding: 8px 24px;
        font-size: 0.85rem;
        color: #c4b5fd;
        font-family: 'Inter', sans-serif;
        font-weight: 500;
        letter-spacing: 1.5px;
        text-transform: uppercase;
        margin-bottom: 20px;
        position: relative;
    }
    .hero-title {
        font-family: 'Inter', sans-serif;
        font-size: 3.8rem;
        font-weight: 900;
        background: linear-gradient(135deg, #ffffff 0%, #c4b5fd 50%, #818cf8 100%);
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        background-clip: text;
        line-height: 1.15;
        margin-bottom: 18px;
        position: relative;
    }
    .hero-subtitle {
        font-family: 'Inter', sans-serif;
        font-size: 1.2rem;
        color: rgba(203, 213, 225, 0.8);
        max-width: 650px;
        margin: 0 auto 35px;
        line-height: 1.7;
        position: relative;
    }

    /* ===== FEATURE CARDS ===== */
    .feature-grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 24px;
        max-width: 1000px;
        margin: 10px auto 40px;
        padding: 0 20px;
    }
    .feature-card {
        background: rgba(255,255,255,0.04);
        backdrop-filter: blur(20px);
        border: 1px solid rgba(255,255,255,0.08);
        border-radius: 20px;
        padding: 32px 24px;
        text-align: center;
        transition: all 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        position: relative;
        overflow: hidden;
    }
    .feature-card::before {
        content: '';
        position: absolute;
        top: 0; left: 0; right: 0;
        height: 3px;
        background: linear-gradient(90deg, transparent, rgba(99,102,241,0.6), transparent);
        opacity: 0;
        transition: opacity 0.4s;
    }
    .feature-card:hover {
        transform: translateY(-8px);
        border-color: rgba(99,102,241,0.3);
        background: rgba(255,255,255,0.07);
        box-shadow: 0 20px 60px rgba(99,102,241,0.15);
    }
    .feature-card:hover::before { opacity: 1; }
    .feature-icon {
        font-size: 2.8rem;
        margin-bottom: 16px;
        display: block;
    }
    .feature-title {
        font-family: 'Inter', sans-serif;
        font-size: 1.15rem;
        font-weight: 700;
        color: #e2e8f0;
        margin-bottom: 10px;
    }
    .feature-desc {
        font-family: 'Inter', sans-serif;
        font-size: 0.88rem;
        color: rgba(148,163,184,0.9);
        line-height: 1.6;
    }

    /* ===== STATS BAR ===== */
    .stats-bar {
        display: flex;
        justify-content: center;
        gap: 60px;
        padding: 30px 20px;
        max-width: 800px;
        margin: 0 auto 30px;
    }
    .stat-item { text-align: center; position: relative; }
    .stat-number {
        font-family: 'Inter', sans-serif;
        font-size: 2.2rem;
        font-weight: 800;
        background: linear-gradient(135deg, #818cf8, #c084fc);
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        background-clip: text;
    }
    .stat-label {
        font-family: 'Inter', sans-serif;
        font-size: 0.82rem;
        color: rgba(148,163,184,0.7);
        text-transform: uppercase;
        letter-spacing: 1.5px;
        margin-top: 4px;
    }

    /* ===== TEMPLATE SELECTOR ===== */
    .template-section-title {
        font-family: 'Inter', sans-serif;
        font-size: 1.8rem;
        font-weight: 700;
        color: #e2e8f0;
        text-align: center;
        margin: 20px 0 8px;
    }
    .template-section-sub {
        font-family: 'Inter', sans-serif;
        font-size: 1rem;
        color: rgba(148,163,184,0.8);
        text-align: center;
        margin-bottom: 25px;
    }

    /* ===== FORM STYLING ===== */
    .section-header {
        font-family: 'Inter', sans-serif;
        font-size: 1.3rem;
        color: #c4b5fd;
        font-weight: 700;
        margin-top: 25px;
        margin-bottom: 15px;
        padding-bottom: 8px;
        border-bottom: 2px solid rgba(99,102,241,0.3);
        letter-spacing: 0.5px;
    }

    /* Style inputs */
    .stTextInput > div > div > input,
    .stTextArea > div > div > textarea {
        background: rgba(255,255,255,0.92) !important;
        border: 1px solid rgba(255,255,255,0.3) !important;
        border-radius: 12px !important;
        color: #000000 !important;
        font-family: 'Inter', sans-serif !important;
        font-size: 1rem !important;
        font-weight: 500 !important;
    }
    .stTextInput > div > div > input::placeholder,
    .stTextArea > div > div > textarea::placeholder {
        color: rgba(165,200,255,0.5) !important;
    }
    .stTextInput > div > div > input:focus,
    .stTextArea > div > div > textarea:focus {
        border-color: rgba(129,140,248,0.6) !important;
        box-shadow: 0 0 0 3px rgba(129,140,248,0.2) !important;
        background: rgba(255,255,255,0.1) !important;
    }
    .stTextInput label, .stTextArea label, .stFileUploader label, .stSelectbox label, .stRadio label {
        color: #e0e7ff !important;
        font-family: 'Inter', sans-serif !important;
        font-weight: 600 !important;
        font-size: 0.95rem !important;
    }

    /* Buttons */
    div.stButton > button:first-child {
        background: linear-gradient(135deg, #6366f1, #8b5cf6) !important;
        color: white !important;
        border-radius: 12px !important;
        padding: 12px 28px !important;
        font-size: 1.1rem !important;
        font-weight: 700 !important;
        font-family: 'Inter', sans-serif !important;
        width: 100% !important;
        border: none !important;
        transition: all 0.3s ease !important;
        letter-spacing: 0.5px !important;
        box-shadow: 0 4px 20px rgba(99,102,241,0.3) !important;
    }
    div.stButton > button:first-child:hover {
        transform: translateY(-3px) !important;
        box-shadow: 0 8px 30px rgba(99,102,241,0.5) !important;
    }

    /* Sidebar */
    section[data-testid="stSidebar"] {
        background: linear-gradient(180deg, #1e1b4b 0%, #1a1a3e 100%) !important;
        border-right: 1px solid rgba(99,102,241,0.2);
    }
    section[data-testid="stSidebar"] .stMarkdown {
        color: #e2e8f0;
    }

    /* Download buttons */
    .stDownloadButton > button {
        background: linear-gradient(135deg, #059669, #10b981) !important;
        color: white !important;
        border-radius: 12px !important;
        font-weight: 600 !important;
        font-family: 'Inter', sans-serif !important;
        border: none !important;
        transition: all 0.3s ease !important;
    }
    .stDownloadButton > button:hover {
        transform: translateY(-2px) !important;
        box-shadow: 0 6px 20px rgba(16,185,129,0.3) !important;
    }

    /* Radio buttons for template selection */
    .stRadio > div {
        display: flex;
        flex-wrap: wrap;
        gap: 12px;
    }
    .stRadio > div > label {
        background: rgba(255,255,255,0.04) !important;
        border: 1px solid rgba(255,255,255,0.1) !important;
        border-radius: 12px !important;
        padding: 12px 20px !important;
        transition: all 0.3s ease !important;
        cursor: pointer !important;
        color: #ffffff !important;
    }
    .stRadio > div > label:hover {
        border-color: rgba(99,102,241,0.4) !important;
        background: rgba(99,102,241,0.1) !important;
    }

    /* Expander */
    .streamlit-expanderHeader {
        font-family: 'Inter', sans-serif !important;
        color: #c4b5fd !important;
        font-weight: 600 !important;
    }

    /* Success, info, warning boxes */
    .stAlert {
        border-radius: 12px !important;
    }

    /* Form submit button */
    .stFormSubmitButton > button {
        background: linear-gradient(135deg, #6366f1, #8b5cf6, #a855f7) !important;
        color: white !important;
        border-radius: 14px !important;
        padding: 14px 32px !important;
        font-size: 1.15rem !important;
        font-weight: 700 !important;
        font-family: 'Inter', sans-serif !important;
        width: 100% !important;
        border: none !important;
        letter-spacing: 0.5px !important;
        box-shadow: 0 6px 30px rgba(99,102,241,0.35) !important;
        transition: all 0.3s ease !important;
    }
    .stFormSubmitButton > button:hover {
        transform: translateY(-3px) !important;
        box-shadow: 0 10px 40px rgba(99,102,241,0.5) !important;
    }
</style>
""", unsafe_allow_html=True)


# --- Sidebar ---
with st.sidebar:
    st.markdown("""
    <div style="text-align:center;padding:20px 0;">
        <div style="font-size:3rem;margin-bottom:10px;">📄</div>
        <div style="font-family:'Inter',sans-serif;font-size:1.3rem;font-weight:800;color:#c4b5fd;letter-spacing:1px;">
            RESUME BUILDER
        </div>
        <div style="font-family:'Inter',sans-serif;font-size:0.78rem;color:rgba(148,163,184,0.7);margin-top:4px;letter-spacing:2px;text-transform:uppercase;">
            AI-Powered • ATS Optimized
        </div>
    </div>
    <hr style="border:none;height:1px;background:linear-gradient(90deg,transparent,rgba(99,102,241,0.3),transparent);margin:0 0 20px;">
    """, unsafe_allow_html=True)

    st.markdown("""
    <div style="background:rgba(99,102,241,0.1);border:1px solid rgba(99,102,241,0.2);border-radius:14px;padding:20px;margin-bottom:20px;">
        <div style="font-family:'Inter',sans-serif;font-size:0.95rem;font-weight:700;color:#c4b5fd;margin-bottom:12px;">⚡ Quick Tips</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.82rem;color:rgba(203,213,225,0.8);line-height:1.8;">
            • Use action verbs for impact<br>
            • Quantify achievements<br>
            • Match job keywords<br>
            • Keep it concise (1-2 pages)<br>
            • Proofread carefully
        </div>
    </div>
    """, unsafe_allow_html=True)

    st.markdown("""
    <div style="background:rgba(16,185,129,0.1);border:1px solid rgba(16,185,129,0.2);border-radius:14px;padding:20px;">
        <div style="font-family:'Inter',sans-serif;font-size:0.95rem;font-weight:700;color:#6ee7b7;margin-bottom:12px;">🎯 ATS Score Boosters</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.82rem;color:rgba(203,213,225,0.8);line-height:1.8;">
            • Standard section headings<br>
            • No tables or graphics<br>
            • Simple, clean formatting<br>
            • Industry-specific keywords<br>
            • Consistent date formats
        </div>
    </div>
    """, unsafe_allow_html=True)


# ===================================================================
# HERO SECTION — Landing Page
# ===================================================================
st.markdown("""
<div class="hero-container">
    <div class="hero-badge">✨ AI-Powered Resume Builder</div>
    <div class="hero-title">Build Your Dream Resume<br>in Minutes</div>
    <div class="hero-subtitle">
        Create stunning, ATS-optimized professional resumes with our AI-powered builder.
        Choose from 5 premium templates designed to land you interviews.
    </div>
</div>
""", unsafe_allow_html=True)

# --- Stats Bar ---
st.markdown("""
<div class="stats-bar">
    <div class="stat-item">
        <div class="stat-number">5</div>
        <div class="stat-label">Pro Templates</div>
    </div>
    <div class="stat-item">
        <div class="stat-number">ATS</div>
        <div class="stat-label">Optimized</div>
    </div>
    <div class="stat-item">
        <div class="stat-number">AI</div>
        <div class="stat-label">Powered</div>
    </div>
    <div class="stat-item">
        <div class="stat-number">∞</div>
        <div class="stat-label">Downloads</div>
    </div>
</div>
""", unsafe_allow_html=True)

# --- Feature Cards ---
st.markdown("""
<div class="feature-grid">
    <div class="feature-card">
        <span class="feature-icon">🤖</span>
        <div class="feature-title">AI-Enhanced Content</div>
        <div class="feature-desc">Our Gemini AI rewrites your resume with powerful action verbs and industry keywords for maximum impact.</div>
    </div>
    <div class="feature-card">
        <span class="feature-icon">🎨</span>
        <div class="feature-title">5 Premium Templates</div>
        <div class="feature-desc">Choose from Executive, Modern, Corporate, Creative, and Elegant styles — each professionally designed.</div>
    </div>
    <div class="feature-card">
        <span class="feature-icon">📊</span>
        <div class="feature-title">ATS-Optimized</div>
        <div class="feature-desc">Every template is built to pass Applicant Tracking Systems used by 98% of Fortune 500 companies.</div>
    </div>
</div>
""", unsafe_allow_html=True)

st.markdown("<br>", unsafe_allow_html=True)

# ===================================================================
# TEMPLATE SELECTION
# ===================================================================
st.markdown("""
<div class="template-section-title">Choose Your Template</div>
<div class="template-section-sub">Select a professional template that matches your industry and style</div>
""", unsafe_allow_html=True)

# Template cards as visual display
st.markdown("""
<div style="display:grid;grid-template-columns:repeat(5,1fr);gap:16px;max-width:1100px;margin:0 auto 30px;padding:0 20px;">
    <div style="background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);border-radius:16px;padding:20px 14px;text-align:center;transition:all 0.3s;">
        <div style="width:100%;height:90px;background:linear-gradient(135deg,#1e3a5f,#2563eb);border-radius:10px;margin-bottom:12px;display:flex;align-items:center;justify-content:center;">
            <div style="color:white;font-family:'Playfair Display',serif;font-weight:700;font-size:0.9rem;">Executive</div>
        </div>
        <div style="font-family:'Inter',sans-serif;font-size:0.85rem;font-weight:700;color:#e2e8f0;">Executive Classic</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.72rem;color:rgba(148,163,184,0.7);margin-top:4px;">Senior & Management</div>
    </div>
    <div style="background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);border-radius:16px;padding:20px 14px;text-align:center;transition:all 0.3s;">
        <div style="width:100%;height:90px;background:linear-gradient(135deg,#f8f9fa,#e9ecef);border-radius:10px;margin-bottom:12px;display:flex;align-items:center;justify-content:center;">
            <div style="color:#1a1a2e;font-family:'Inter',sans-serif;font-weight:700;font-size:0.9rem;">Minimal</div>
        </div>
        <div style="font-family:'Inter',sans-serif;font-size:0.85rem;font-weight:700;color:#e2e8f0;">Modern Minimalist</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.72rem;color:rgba(148,163,184,0.7);margin-top:4px;">Tech & Startups</div>
    </div>
    <div style="background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);border-radius:16px;padding:20px 14px;text-align:center;transition:all 0.3s;">
        <div style="width:100%;height:90px;background:linear-gradient(135deg,#1a1a2e,#2d2d5e);border-radius:10px;margin-bottom:12px;display:flex;align-items:center;justify-content:center;">
            <div style="color:#d4af37;font-family:'Playfair Display',serif;font-weight:700;font-size:0.9rem;">Corporate</div>
        </div>
        <div style="font-family:'Inter',sans-serif;font-size:0.85rem;font-weight:700;color:#e2e8f0;">Corporate Pro</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.72rem;color:rgba(148,163,184,0.7);margin-top:4px;">Finance & Consulting</div>
    </div>
    <div style="background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);border-radius:16px;padding:20px 14px;text-align:center;transition:all 0.3s;">
        <div style="width:100%;height:90px;background:linear-gradient(135deg,#667eea,#764ba2);border-radius:10px;margin-bottom:12px;display:flex;align-items:center;justify-content:center;">
            <div style="color:white;font-family:'Inter',sans-serif;font-weight:800;font-size:0.9rem;">Creative</div>
        </div>
        <div style="font-family:'Inter',sans-serif;font-size:0.85rem;font-weight:700;color:#e2e8f0;">Creative Bold</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.72rem;color:rgba(148,163,184,0.7);margin-top:4px;">Design & Marketing</div>
    </div>
    <div style="background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);border-radius:16px;padding:20px 14px;text-align:center;transition:all 0.3s;">
        <div style="width:100%;height:90px;background:linear-gradient(135deg,#0a0a0a,#1a1a1a);border-radius:10px;margin-bottom:12px;display:flex;align-items:center;justify-content:center;">
            <div style="color:#d4af37;font-family:'Playfair Display',serif;font-weight:700;font-size:0.9rem;">Elegant</div>
        </div>
        <div style="font-family:'Inter',sans-serif;font-size:0.85rem;font-weight:700;color:#e2e8f0;">Elegant Formal</div>
        <div style="font-family:'Inter',sans-serif;font-size:0.72rem;color:rgba(148,163,184,0.7);margin-top:4px;">Legal & Academia</div>
    </div>
</div>
""", unsafe_allow_html=True)

# Actual template selection
template_choice = st.radio(
    "🎨 Select Template Style",
    ["📋 Executive Classic", "✨ Modern Minimalist", "🏢 Corporate Professional", "🎨 Creative Bold", "🖤 Elegant Formal"],
    horizontal=True
)

st.markdown("<br>", unsafe_allow_html=True)

# ===================================================================
# FORM INPUTS
# ===================================================================
with st.form("resume_builder"):
    st.markdown('<div class="section-header">👤 Personal Details</div>', unsafe_allow_html=True)

    profile_pic = st.file_uploader("Upload Profile Photo (Optional)", type=["jpg", "png", "jpeg"])

    col1, col2 = st.columns(2)
    with col1:
        full_name = st.text_input("Full Name", placeholder="e.g. John Doe")
        email = st.text_input("Email Address", placeholder="e.g. john@example.com")
    with col2:
        phone = st.text_input("Phone Number", placeholder="e.g. +1 234 567 890")
        linkedin = st.text_input("LinkedIn / Portfolio URL", placeholder="e.g. linkedin.com/in/johndoe")

    st.markdown('<div class="section-header">🎯 Objective / Summary</div>', unsafe_allow_html=True)
    objective = st.text_area("Write a brief objective or summary",
                             placeholder="Briefly describe your career goals and what you bring to the table...")

    st.markdown('<div class="section-header">🎓 Education</div>', unsafe_allow_html=True)
    education = st.text_area("Enter your education details",
                             placeholder="e.g. B.S. in Computer Science, University of XYZ, 2020-2024, GPA: 3.8")

    st.markdown('<div class="section-header">💼 Internship / Work Experience</div>', unsafe_allow_html=True)
    internship = st.text_area("Enter your internships or past work experience",
                              placeholder="e.g. Software Engineer Intern at ABC Corp. (June 2023 - August 2023).")

    st.markdown('<div class="section-header">🛠️ Skills & Technical Expertise</div>', unsafe_allow_html=True)
    col3, col4 = st.columns(2)
    with col3:
        technical_skills = st.text_area("Technical Skills", placeholder="e.g. Python, Java, React, SQL, AWS")
    with col4:
        soft_skills = st.text_area("Soft Skills", placeholder="e.g. Leadership, Communication, Problem Solving")
        
    languages = st.text_input("Languages Known", placeholder="e.g. English (Native), Spanish (Fluent)")

    st.markdown('<div class="section-header">📜 Declaration</div>', unsafe_allow_html=True)
    declaration = st.text_area("Declaration (Optional)", 
                               placeholder="e.g. I hereby declare that the information provided above is true to the best of my knowledge.")

    st.markdown("<br>", unsafe_allow_html=True)
    submit_button = st.form_submit_button("🚀 Generate My Professional Resume")


# =====================================================
# 5 PROFESSIONAL RESUME TEMPLATES
# =====================================================

def get_photo_html(photo_file, style=""):
    """Generate photo HTML from uploaded file."""
    if photo_file is None:
        return ""
    b64 = base64.b64encode(photo_file.getvalue()).decode("utf-8")
    mime = photo_file.type
    default_style = "width:120px;height:120px;border-radius:50%;object-fit:cover;"
    return f'<img src="data:{mime};base64,{b64}" style="{default_style}{style}">'


def get_contact_str(em, ph, link, sep=" &nbsp;|&nbsp; ", icons=True):
    """Generate contact string."""
    parts = []
    if em:
        parts.append(f"{'📧 ' if icons else ''}{em}")
    if ph:
        parts.append(f"{'📞 ' if icons else ''}{ph}")
    if link:
        parts.append(f"{'🔗 ' if icons else ''}{link}")
    return sep.join(parts)


def make_list_items(text):
    """Convert multiline text to list items."""
    if not text:
        return ""
    items = ""
    for line in text.strip().split("\n"):
        line = line.strip()
        if line:
            items += f"<li>{line}</li>"
    return items


def make_badge_items(text, bg, color, border_color):
    """Convert text to badges."""
    if not text:
        return ""
    badges = ""
    for item in text.replace("\n", ",").split(","):
        item = item.strip()
        if item:
            badges += f'<span style="background:{bg};color:{color};padding:7px 16px;border-radius:20px;font-size:0.85rem;font-weight:500;border:1px solid {border_color};display:inline-block;margin:4px;">{item}</span>'
    return badges


# ─── TEMPLATE 1: Executive Classic ─────────────────────────────
def template_executive(name, em, ph, link, obj, edu, intern, tech_skills, soft_skills, langs, declaration, photo_file):
    photo_html = get_photo_html(photo_file, "border:4px solid rgba(255,255,255,0.7);box-shadow:0 4px 20px rgba(0,0,0,0.2);margin-bottom:15px;")
    contact_str = get_contact_str(em, ph, link)

    def section(icon, title, content):
        if not content:
            return ""
        return f'''<div style="margin-bottom:28px;">
            <div style="font-family:'Playfair Display',serif;font-size:1.15rem;font-weight:700;color:#1e3a5f;text-transform:uppercase;letter-spacing:2px;border-bottom:3px solid #2563eb;padding-bottom:10px;margin-bottom:16px;">{icon} {title}</div>
            {content}
        </div>'''

    summary = f'<p style="color:#374151;line-height:1.8;font-size:0.95rem;font-family:\'Inter\',sans-serif;">{obj}</p>' if obj else ""
    edu_items = make_list_items(edu)
    exp_items = make_list_items(intern)
    tech_badges = make_badge_items(tech_skills, "linear-gradient(135deg,#EEF2FF,#DBEAFE)", "#1E40AF", "#BFDBFE")
    soft_badges = make_badge_items(soft_skills, "linear-gradient(135deg,#F5F3FF,#EDE9FE)", "#5B21B6", "#DDD6FE")
    lang_badges = make_badge_items(langs, "linear-gradient(135deg,#F0FDF4,#DCFCE7)", "#166534", "#BBF7D0")
    decl_html = f'<p style="color:#4b5563;line-height:1.6;font-size:0.88rem;font-style:italic;padding-top:10px;">"{declaration}"</p>' if declaration else ""

    return f'''
    <style>
        .exec-card ul {{ list-style:none; padding:0; margin:0; }}
        .exec-card ul li {{
            position:relative; padding-left:22px; margin-bottom:10px;
            color:#374151; line-height:1.7; font-size:0.93rem; font-family:'Inter',sans-serif;
        }}
        .exec-card ul li::before {{ content:'▹'; position:absolute; left:0; color:#2563eb; font-weight:bold; }}
    </style>
    <div class="exec-card" style="font-family:'Inter',sans-serif;max-width:820px;margin:20px auto;background:#fff;border-radius:0;box-shadow:0 10px 50px rgba(0,0,0,0.1);overflow:hidden;border:none;">
        <div style="background:linear-gradient(135deg,#1e3a5f 0%,#2563eb 100%);color:white;padding:50px 40px;text-align:center;">
            {photo_html}
            <div style="font-family:'Playfair Display',serif;font-size:2.4rem;font-weight:700;letter-spacing:3px;margin-bottom:6px;">{name}</div>
            <div style="font-size:0.88rem;opacity:0.85;margin-top:12px;letter-spacing:1px;">{contact_str}</div>
        </div>
        <div style="padding:35px 40px;">
            {section("🎯", "Professional Summary", summary)}
            {section("🎓", "Education", f"<ul>{edu_items}</ul>")}
            {section("💼", "Professional Experience", f"<ul>{exp_items}</ul>")}
            {section("💻", "Technical Skills", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{tech_badges}</div>')}
            {section("🤝", "Soft Skills", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{soft_badges}</div>')}
            {section("🌐", "Languages", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{lang_badges}</div>')}
            {section("📜", "Declaration", decl_html)}
        </div>
    </div>'''


# ─── TEMPLATE 2: Modern Minimalist ──────────────────────────────
def template_modern(name, em, ph, link, obj, edu, intern, tech_skills, soft_skills, langs, declaration, photo_file):
    photo_html = get_photo_html(photo_file, "border:3px solid #e5e7eb;margin-bottom:15px;")
    contact_str = get_contact_str(em, ph, link, " · ", False)

    def section(title, content):
        if not content:
            return ""
        return f'''<div style="margin-bottom:30px;">
            <div style="font-family:'Inter',sans-serif;font-size:0.78rem;font-weight:700;color:#9ca3af;text-transform:uppercase;letter-spacing:3px;margin-bottom:14px;">{title}</div>
            {content}
        </div>'''

    summary = f'<p style="color:#4b5563;line-height:1.8;font-size:0.94rem;font-family:\'Inter\',sans-serif;">{obj}</p>' if obj else ""
    edu_items = make_list_items(edu)
    exp_items = make_list_items(intern)
    tech_badges = make_badge_items(tech_skills, "#f3f4f6", "#374151", "#e5e7eb")
    soft_badges = make_badge_items(soft_skills, "#f3f4f6", "#374151", "#e5e7eb")
    lang_badges = make_badge_items(langs, "#f3f4f6", "#374151", "#e5e7eb")
    decl_html = f'<p style="color:#6b7280;line-height:1.6;font-size:0.85rem;font-style:italic;border-left:3px solid #cbd5e1;padding-left:12px;">{declaration}</p>' if declaration else ""

    return f'''
    <style>
        .modern-card ul {{ list-style:none; padding:0; margin:0; }}
        .modern-card ul li {{
            position:relative; padding-left:16px; margin-bottom:10px;
            color:#4b5563; line-height:1.7; font-size:0.93rem; font-family:'Inter',sans-serif;
        }}
        .modern-card ul li::before {{ content:'—'; position:absolute; left:0; color:#d1d5db; font-weight:300; }}
    </style>
    <div class="modern-card" style="font-family:'Inter',sans-serif;max-width:820px;margin:20px auto;background:#ffffff;border-radius:4px;box-shadow:0 1px 3px rgba(0,0,0,0.06);overflow:hidden;border:1px solid #e5e7eb;">
        <div style="padding:50px 45px 30px;text-align:center;border-bottom:1px solid #f3f4f6;">
            {photo_html}
            <div style="font-size:2.2rem;font-weight:800;color:#111827;letter-spacing:-0.5px;margin-bottom:8px;">{name}</div>
            <div style="font-size:0.88rem;color:#9ca3af;letter-spacing:1px;">{contact_str}</div>
        </div>
        <div style="padding:35px 45px;">
            {section("About", summary)}
            {section("Education", f"<ul>{edu_items}</ul>")}
            {section("Experience", f"<ul>{exp_items}</ul>")}
            {section("Technical Skills", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{tech_badges}</div>')}
            {section("Soft Skills", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{soft_badges}</div>')}
            {section("Languages", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{lang_badges}</div>')}
            {section("Declaration", decl_html)}
        </div>
    </div>'''


# ─── TEMPLATE 3: Corporate Professional ─────────────────────────
def template_corporate(name, em, ph, link, obj, edu, intern, tech_skills, soft_skills, langs, declaration, photo_file):
    photo_html = get_photo_html(photo_file, "border:3px solid #d4af37;box-shadow:0 4px 15px rgba(0,0,0,0.3);margin-bottom:18px;")

    contact_parts = []
    if em:
        contact_parts.append(f"<div style='margin-bottom:6px;font-size:0.85rem;'>📧 {em}</div>")
    if ph:
        contact_parts.append(f"<div style='margin-bottom:6px;font-size:0.85rem;'>📞 {ph}</div>")
    if link:
        contact_parts.append(f"<div style='margin-bottom:6px;font-size:0.85rem;'>🔗 {link}</div>")
    contact_html = "".join(contact_parts)

    def section_right(icon, title, content):
        if not content:
            return ""
        return f'''<div style="margin-bottom:28px;">
            <div style="font-family:'Inter',sans-serif;font-size:1.05rem;font-weight:700;color:#1a1a2e;text-transform:uppercase;letter-spacing:2px;border-left:4px solid #d4af37;padding-left:14px;margin-bottom:14px;">{icon} {title}</div>
            {content}
        </div>'''

    summary = f'<p style="color:#374151;line-height:1.8;font-size:0.93rem;">{obj}</p>' if obj else ""
    edu_items = make_list_items(edu)
    exp_items = make_list_items(intern)
    decl_html = f'<p style="color:#4b5563;line-height:1.6;font-size:0.85rem;font-style:italic;margin-top:10px;">{declaration}</p>' if declaration else ""

    # Sidebar skills (Tech + Soft combo)
    all_skills = ""
    if tech_skills or soft_skills:
        items = ""
        combined_skills = []
        if tech_skills: combined_skills.extend([s.strip() for s in tech_skills.replace("\\n", ",").split(",") if s.strip()])
        if soft_skills: combined_skills.extend([s.strip() for s in soft_skills.replace("\\n", ",").split(",") if s.strip()])
        
        for s in combined_skills:
            items += f'<div style="background:rgba(212,175,55,0.15);border-radius:8px;padding:6px 12px;margin-bottom:6px;font-size:0.82rem;color:#e2e8f0;text-align:center;">{s}</div>'
        all_skills = f'''<div style="margin-top:25px;">
            <div style="font-size:0.78rem;font-weight:700;color:#d4af37;text-transform:uppercase;letter-spacing:2px;margin-bottom:12px;">Expertise</div>
            {items}
        </div>'''

    sidebar_langs = ""
    if langs:
        items = ""
        for l in langs.replace("\\n", ",").split(","):
            l = l.strip()
            if l:
                items += f'<div style="background:rgba(255,255,255,0.08);border-radius:8px;padding:6px 12px;margin-bottom:6px;font-size:0.82rem;color:#e2e8f0;text-align:center;">{l}</div>'
        sidebar_langs = f'''<div style="margin-top:25px;">
            <div style="font-size:0.78rem;font-weight:700;color:#d4af37;text-transform:uppercase;letter-spacing:2px;margin-bottom:12px;">Languages</div>
            {items}
        </div>'''

    return f'''
    <style>
        .corp-card ul {{ list-style:none; padding:0; margin:0; }}
        .corp-card ul li {{
            position:relative; padding-left:22px; margin-bottom:10px;
            color:#374151; line-height:1.7; font-size:0.93rem; font-family:'Inter',sans-serif;
        }}
        .corp-card ul li::before {{ content:'◆'; position:absolute; left:0; color:#d4af37; font-size:0.6rem; top:6px; }}
    </style>
    <div class="corp-card" style="font-family:'Inter',sans-serif;max-width:880px;margin:20px auto;display:flex;box-shadow:0 10px 50px rgba(0,0,0,0.12);overflow:hidden;border-radius:0;">
        <div style="width:280px;background:linear-gradient(180deg,#1a1a2e 0%,#16213e 100%);color:#e2e8f0;padding:40px 25px;text-align:center;flex-shrink:0;">
            {photo_html}
            <div style="font-family:'Playfair Display',serif;font-size:1.3rem;font-weight:700;color:#ffffff;margin-bottom:8px;">{name}</div>
            <div style="width:40px;height:2px;background:#d4af37;margin:12px auto;"></div>
            <div style="margin-top:15px;text-align:left;">
                {contact_html}
            </div>
            {all_skills}
            {sidebar_langs}
        </div>
        <div style="flex:1;background:#ffffff;padding:35px 40px;">
            {section_right("🎯", "Professional Summary", summary)}
            {section_right("🎓", "Education", f"<ul>{edu_items}</ul>")}
            {section_right("💼", "Experience", f"<ul>{exp_items}</ul>")}
            {section_right("📜", "Declaration", decl_html)}
        </div>
    </div>'''


# ─── TEMPLATE 4: Creative Bold ──────────────────────────────────
def template_creative(name, em, ph, link, obj, edu, intern, tech_skills, soft_skills, langs, declaration, photo_file):
    photo_html = get_photo_html(photo_file, "border:4px solid rgba(255,255,255,0.9);box-shadow:0 8px 30px rgba(0,0,0,0.3);margin-bottom:18px;")
    contact_str = get_contact_str(em, ph, link, " &bull; ", True)

    def section(icon, title, content):
        if not content:
            return ""
        return f'''<div style="margin-bottom:30px;">
            <div style="font-family:'Inter',sans-serif;font-size:1.1rem;font-weight:800;color:#4c1d95;letter-spacing:1px;margin-bottom:14px;display:flex;align-items:center;gap:10px;">
                <span style="background:linear-gradient(135deg,#667eea,#764ba2);color:white;width:32px;height:32px;border-radius:8px;display:inline-flex;align-items:center;justify-content:center;font-size:0.9rem;">{icon}</span>
                {title}
            </div>
            {content}
        </div>'''

    summary = f'<p style="color:#4b5563;line-height:1.8;font-size:0.94rem;font-family:\'Inter\',sans-serif;background:linear-gradient(135deg,#faf5ff,#ede9fe);padding:16px 20px;border-radius:12px;border-left:4px solid #8b5cf6;">{obj}</p>' if obj else ""
    edu_items = make_list_items(edu)
    exp_items = make_list_items(intern)
    tech_badges = make_badge_items(tech_skills, "linear-gradient(135deg,#e0e7ff,#c7d2fe)", "#4338ca", "#a5b4fc")
    soft_badges = make_badge_items(soft_skills, "linear-gradient(135deg,#ede9fe,#ddd6fe)", "#5b21b6", "#c4b5fd")
    lang_badges = make_badge_items(langs, "linear-gradient(135deg,#fce7f3,#fbcfe8)", "#9d174d", "#f9a8d4")
    decl_html = f'<p style="color:#6b7280;line-height:1.6;font-size:0.85rem;font-style:italic;background:#f9fafb;padding:12px;border-radius:8px;">{declaration}</p>' if declaration else ""

    return f'''
    <style>
        .creative-card ul {{ list-style:none; padding:0; margin:0; }}
        .creative-card ul li {{
            position:relative; padding-left:22px; margin-bottom:12px;
            color:#4b5563; line-height:1.7; font-size:0.93rem; font-family:'Inter',sans-serif;
        }}
        .creative-card ul li::before {{ content:'→'; position:absolute; left:0; color:#8b5cf6; font-weight:bold; }}
    </style>
    <div class="creative-card" style="font-family:'Inter',sans-serif;max-width:820px;margin:20px auto;background:#fff;border-radius:20px;box-shadow:0 15px 60px rgba(102,126,234,0.15);overflow:hidden;border:none;">
        <div style="background:linear-gradient(135deg,#667eea 0%,#764ba2 50%,#f093fb 100%);color:white;padding:50px 40px;text-align:center;position:relative;overflow:hidden;">
            <div style="position:absolute;top:-50px;right:-50px;width:200px;height:200px;background:rgba(255,255,255,0.08);border-radius:50%;"></div>
            <div style="position:absolute;bottom:-30px;left:-30px;width:120px;height:120px;background:rgba(255,255,255,0.05);border-radius:50%;"></div>
            {photo_html}
            <div style="font-size:2.5rem;font-weight:900;letter-spacing:1px;margin-bottom:8px;position:relative;">{name}</div>
            <div style="font-size:0.88rem;opacity:0.9;margin-top:12px;position:relative;">{contact_str}</div>
        </div>
        <div style="padding:35px 40px;">
            {section("🎯", "ABOUT ME", summary)}
            {section("🎓", "EDUCATION", f"<ul>{edu_items}</ul>")}
            {section("💼", "EXPERIENCE", f"<ul>{exp_items}</ul>")}
            {section("💻", "TECH SKILLS", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{tech_badges}</div>')}
            {section("⚡", "SOFT SKILLS", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{soft_badges}</div>')}
            {section("🌍", "LANGUAGES", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{lang_badges}</div>')}
            {section("✍️", "DECLARATION", decl_html)}
        </div>
    </div>'''


# ─── TEMPLATE 5: Elegant Formal ─────────────────────────────────
def template_elegant(name, em, ph, link, obj, edu, intern, tech_skills, soft_skills, langs, declaration, photo_file):
    photo_html = get_photo_html(photo_file, "border:3px solid #d4af37;box-shadow:0 4px 20px rgba(0,0,0,0.15);margin-bottom:18px;")
    contact_str = get_contact_str(em, ph, link, " &nbsp;⬩&nbsp; ", False)

    def section(title, content):
        if not content:
            return ""
        return f'''<div style="margin-bottom:30px;">
            <div style="font-family:'Playfair Display',serif;font-size:1.1rem;font-weight:700;color:#1a1a1a;text-transform:uppercase;letter-spacing:3px;margin-bottom:6px;">{title}</div>
            <div style="width:50px;height:2px;background:#d4af37;margin-bottom:16px;"></div>
            {content}
        </div>'''

    summary = f'<p style="color:#374151;line-height:1.9;font-size:0.94rem;font-family:\'Inter\',sans-serif;font-style:italic;">{obj}</p>' if obj else ""
    edu_items = make_list_items(edu)
    exp_items = make_list_items(intern)
    tech_badges = make_badge_items(tech_skills, "#f8f5f0", "#44403c", "#d6d3d1")
    soft_badges = make_badge_items(soft_skills, "#f8f5f0", "#44403c", "#d6d3d1")
    lang_badges = make_badge_items(langs, "#f8f5f0", "#44403c", "#d6d3d1")
    decl_html = f'<p style="color:#4b5563;line-height:1.7;font-size:0.9rem;font-style:italic;">"{declaration}"</p>' if declaration else ""

    return f'''
    <style>
        .elegant-card ul {{ list-style:none; padding:0; margin:0; }}
        .elegant-card ul li {{
            position:relative; padding-left:22px; margin-bottom:10px;
            color:#374151; line-height:1.8; font-size:0.93rem; font-family:'Inter',sans-serif;
        }}
        .elegant-card ul li::before {{ content:'■'; position:absolute; left:0; color:#d4af37; font-size:0.5rem; top:7px; }}
    </style>
    <div class="elegant-card" style="font-family:'Inter',sans-serif;max-width:820px;margin:20px auto;background:#fffffe;border-radius:0;box-shadow:0 5px 30px rgba(0,0,0,0.08);overflow:hidden;border:2px solid #1a1a1a;">
        <div style="background:#1a1a1a;color:#fffffe;padding:50px 45px;text-align:center;">
            {photo_html}
            <div style="font-family:'Playfair Display',serif;font-size:2.4rem;font-weight:700;letter-spacing:5px;margin-bottom:8px;text-transform:uppercase;">{name}</div>
            <div style="width:60px;height:2px;background:#d4af37;margin:14px auto;"></div>
            <div style="font-size:0.85rem;color:rgba(255,255,255,0.7);letter-spacing:1.5px;margin-top:12px;">{contact_str}</div>
        </div>
        <div style="padding:40px 45px;">
            {section("Professional Summary", summary)}
            {section("Education", f"<ul>{edu_items}</ul>")}
            {section("Professional Experience", f"<ul>{exp_items}</ul>")}
            {section("Technical Skills", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{tech_badges}</div>')}
            {section("Soft Skills", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{soft_badges}</div>')}
            {section("Languages", f'<div style="display:flex;flex-wrap:wrap;gap:8px;">{lang_badges}</div>')}
            {section("Declaration", decl_html)}
        </div>
        <div style="height:4px;background:linear-gradient(90deg,#1a1a1a,#d4af37,#1a1a1a);"></div>
    </div>'''


# =====================================================
# MARKDOWN GENERATOR (shared across templates)
# =====================================================
def generate_download_md(name, em, ph, link, obj, edu, intern, tech_skills, soft_skills, langs, declaration):
    """Generate a clean Markdown version for download."""
    parts = [f"# {name}\n"]
    ci = []
    if em: ci.append(f"Email: {em}")
    if ph: ci.append(f"Phone: {ph}")
    if link: ci.append(f"LinkedIn: {link}")
    if ci:
        parts.append(" | ".join(ci) + "\n")
    parts.append("---\n")
    if obj:
        parts.append("## PROFESSIONAL SUMMARY\n")
        parts.append(f"{obj}\n")
    if edu:
        parts.append("## EDUCATION\n")
        for line in edu.strip().split("\n"):
            if line.strip():
                parts.append(f"- {line.strip()}")
        parts.append("")
    if intern:
        parts.append("## EXPERIENCE / INTERNSHIP\n")
        for line in intern.strip().split("\n"):
            if line.strip():
                parts.append(f"- {line.strip()}")
        parts.append("")
    if tech_skills:
        parts.append("## TECHNICAL SKILLS\n")
        for s in tech_skills.replace("\n", ",").split(","):
            if s.strip():
                parts.append(f"- {s.strip()}")
        parts.append("")
    if soft_skills:
        parts.append("## SOFT SKILLS\n")
        for s in soft_skills.replace("\n", ",").split(","):
            if s.strip():
                parts.append(f"- {s.strip()}")
        parts.append("")
    if langs:
        parts.append("## LANGUAGES\n")
        for l in langs.replace("\n", ",").split(","):
            if l.strip():
                parts.append(f"- {l.strip()}")
        parts.append("")
    if declaration:
        parts.append("## DECLARATION\n")
        parts.append(f"_{declaration}_\n")
    return "\n".join(parts)


# =====================================================
# TEMPLATE DISPATCHER
# =====================================================
TEMPLATE_MAP = {
    "📋 Executive Classic": template_executive,
    "✨ Modern Minimalist": template_modern,
    "🏢 Corporate Professional": template_corporate,
    "🎨 Creative Bold": template_creative,
    "🖤 Elegant Formal": template_elegant,
}


# =====================================================
# MAIN LOGIC — Generate Resume on Submit
# =====================================================
if submit_button:
    if not full_name or not objective or not education:
        st.warning("⚠️ Please fill in at least your Name, Objective, and Education details.")
    else:
        with st.spinner("⏳ Crafting your professional resume..."):

            ai_resume = None

            # Try AI generation (optional)
            try:
                import google.generativeai as genai
                genai.configure(api_key="AIzaSyB5T839e5RNfqi0KiTcESYimSvRsL71-ZA")
                model = genai.GenerativeModel('gemini-2.0-flash')
                prompt = f"""You are an Expert ATS Analyst and Senior Professional Resume Writer.
Generate a highly professional, ATS-optimized resume from these inputs.
Rewrite content to sound professional with strong action verbs and keywords.

- Name: {full_name}
- Email: {email}
- Phone: {phone}
- LinkedIn/Portfolio: {linkedin}
- Objective: {objective}
- Education: {education}
- Internship/Experience: {internship}
- Technical Skills: {technical_skills}
- Soft Skills: {soft_skills}
- Languages: {languages}
- Declaration: {declaration}

RULES: Return ONLY Markdown. No conversational text. Use standard headings
(SUMMARY, EDUCATION, EXPERIENCE, TECHNICAL SKILLS, SOFT SKILLS). Use bullet points. Clean and readable. Include the Declaration verbatim at the end if provided."""
                response = model.generate_content(prompt)
                ai_resume = response.text
            except Exception:
                ai_resume = None

        if ai_resume:
            # AI succeeded — render inside selected template wrapper
            if profile_pic is not None:
                b64_img = base64.b64encode(profile_pic.getvalue()).decode("utf-8")
                mime_type = profile_pic.type
                img_tag = f'<div style="text-align:center;"><img src="data:{mime_type};base64,{b64_img}" width="150" height="150" style="border-radius:50%;object-fit:cover;border:4px solid #DBEAFE;margin-bottom:20px;"></div>'
                ai_resume = img_tag + "\n\n" + ai_resume

            st.success("✅ AI Optimization Complete! Your resume is ready below.")
            st.balloons()
            st.markdown('<div class="section-header">✨ Your ATS-Optimized Resume</div>', unsafe_allow_html=True)
            ai_html_wrapped = f'''
            <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Playfair+Display:wght@400;500;600;700&display=swap" rel="stylesheet">
            <div style="font-family:'Inter',sans-serif;background:#fff;padding:35px;border-radius:12px;box-shadow:0 4px 20px rgba(0,0,0,0.08);border:1px solid #E5E7EB;">
                {ai_resume}
            </div>
            '''
            components.html(ai_html_wrapped, height=800, scrolling=True)

            st.download_button(
                label="📥 Download Resume (.md)",
                data=ai_resume,
                file_name=f"{full_name.replace(' ', '_')}_Optimized_CV.md",
                mime="text/markdown",
                use_container_width=True
            )

            # PNG Download
            try:
                from html2image import Html2Image
                with tempfile.TemporaryDirectory() as tmpdir:
                    hti = Html2Image(output_path=tmpdir, size=(900, 1200))
                    img_file = f"{full_name.replace(' ', '_')}_Resume.png"
                    hti.screenshot(html_str=ai_html_wrapped, save_as=img_file)
                    img_path = os.path.join(tmpdir, img_file)
                    with open(img_path, "rb") as f:
                        img_bytes = f.read()
                    st.download_button(
                        label="🖼️ Download Resume as PNG Image",
                        data=img_bytes,
                        file_name=img_file,
                        mime="image/png",
                        use_container_width=True
                    )
            except Exception:
                st.caption("💡 To enable PNG download, run: pip install html2image")
        else:
            # FALLBACK: Built-in premium HTML template (always works, no API needed)
            st.info("📝 Generating your resume using the premium built-in template.")

            # Get the selected template function
            template_fn = TEMPLATE_MAP.get(template_choice, template_executive)

            resume_html = template_fn(
                full_name, email, phone, linkedin,
                objective, education, internship,
                technical_skills, soft_skills, languages, declaration, profile_pic
            )

            # Wrap with Google Fonts for iframe rendering
            resume_html_full = f'''
            <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Playfair+Display:wght@400;500;600;700&display=swap" rel="stylesheet">
            {resume_html}
            '''

            download_md = generate_download_md(
                full_name, email, phone, linkedin,
                objective, education, internship,
                technical_skills, soft_skills, languages, declaration
            )

            st.success("✅ Resume generated successfully!")
            st.balloons()
            st.markdown(f'<div class="section-header">✨ Your Professional Resume — {template_choice}</div>', unsafe_allow_html=True)
            components.html(resume_html_full, height=900, scrolling=True)

            st.download_button(
                label="📥 Download Resume (.md)",
                data=download_md,
                file_name=f"{full_name.replace(' ', '_')}_Resume.md",
                mime="text/markdown",
                use_container_width=True
            )

            # PNG Download
            try:
                from html2image import Html2Image
                with tempfile.TemporaryDirectory() as tmpdir:
                    hti = Html2Image(output_path=tmpdir, size=(900, 1200))
                    img_file = f"{full_name.replace(' ', '_')}_Resume.png"
                    hti.screenshot(html_str=resume_html_full, save_as=img_file)
                    img_path = os.path.join(tmpdir, img_file)
                    with open(img_path, "rb") as f:
                        img_bytes = f.read()
                    st.download_button(
                        label="🖼️ Download Resume as PNG Image",
                        data=img_bytes,
                        file_name=img_file,
                        mime="image/png",
                        use_container_width=True
                    )
            except Exception:
                st.caption("💡 To enable PNG download, run: pip install html2image")
