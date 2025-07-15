import streamlit as st
import pytesseract
from PIL import Image, ImageEnhance, ImageFilter
from pdf2image import convert_from_path
from docx import Document
from fpdf import FPDF
from io import BytesIO
import tempfile
import os
from langdetect import detect_langs

pytesseract.pytesseract.tesseract_cmd = r"C:\\Program Files\\Tesseract-OCR\\tesseract.exe"

LANG_CODE_MAP = {
    'ml': 'mal',
    'en': 'eng',
    # extend as needed
}

st.set_page_config(page_title="📑 OCR with Malayalam Support", layout="wide")
st.title("📑 Multilingual OCR Tool (with Malayalam Support)")

# File upload
uploaded_file = st.file_uploader("Upload PDF, DOCX, or Image", type=["pdf", "docx", "png", "jpg", "jpeg"])

# Enhancement options
enhance_col1, enhance_col2, enhance_col3 = st.columns(3)
with enhance_col1:
    grayscale = st.checkbox("Grayscale", value=True)
    sharpen = st.checkbox("Sharpen", value=True)
with enhance_col2:
    contrast = st.checkbox("Enhance Contrast", value=True)
    brightness = st.checkbox("Enhance Brightness", value=False)
with enhance_col3:
    denoise = st.checkbox("Denoise (Filter)", value=False)

# Save format
save_format = st.selectbox("Save OCR Output As", ["TXT", "DOCX", "PDF"])

# Utility functions
def enhance_image(img):
    if grayscale:
        img = img.convert('L')
    else:
        img = img.convert('RGB')
    if sharpen:
        img = img.filter(ImageFilter.SHARPEN)
    if contrast:
        img = ImageEnhance.Contrast(img).enhance(2)
    if brightness:
        img = ImageEnhance.Brightness(img).enhance(1.5)
    if denoise:
        img = img.filter(ImageFilter.MedianFilter(size=3))
    return img

def detect_languages(text):
    try:
        return [LANG_CODE_MAP.get(d.lang, 'eng') for d in detect_langs(text)]
    except:
        return ['eng']

def save_output(text, fmt):
    output = BytesIO()
    if fmt == "TXT":
        output.write(text.encode("utf-8"))
        return output.getvalue(), "output.txt"
    elif fmt == "DOCX":
        doc = Document()
        for line in text.split("\n"):
            doc.add_paragraph(line)
        temp_path = tempfile.mktemp(suffix=".docx")
        doc.save(temp_path)
        with open(temp_path, "rb") as f:
            return f.read(), "output.docx"
    elif fmt == "PDF":
        pdf = FPDF()
        pdf.add_page()
        pdf.set_font("Arial", size=12)
        for line in text.split("\n"):
            pdf.multi_cell(0, 10, line)
        temp_path = tempfile.mktemp(suffix=".pdf")
        pdf.output(temp_path)
        with open(temp_path, "rb") as f:
            return f.read(), "output.pdf"
    return None, None

# OCR logic
if uploaded_file:
    ext = os.path.splitext(uploaded_file.name)[1].lower()
    ocr_text = ""

    if ext == ".pdf":
        st.info("Processing PDF...")
        with tempfile.NamedTemporaryFile(delete=False, suffix=".pdf") as tmp:
            tmp.write(uploaded_file.read())
            tmp_path = tmp.name
        pages = convert_from_path(tmp_path, dpi=300)
        for i, page in enumerate(pages):
            st.write(f"Processing page {i+1}/{len(pages)}")
            img = enhance_image(page)
            text = pytesseract.image_to_string(img, lang="mal+eng")
            ocr_text += text + "\n"

    elif ext == ".docx":
        st.info("Reading DOCX file...")
        doc = Document(uploaded_file)
        ocr_text = "\n".join([p.text for p in doc.paragraphs])

    elif ext in [".png", ".jpg", ".jpeg"]:
        img = Image.open(uploaded_file)
        img = enhance_image(img)
        st.image(img, caption="Enhanced Image", use_column_width=True)
        ocr_text = pytesseract.image_to_string(img, lang="mal+eng")

    else:
        st.error("Unsupported file type.")

    if ocr_text:
        st.subheader("📄 OCR Result")
        st.text_area("Recognized Text:", ocr_text, height=300)

        file_bytes, filename = save_output(ocr_text, save_format)
        if file_bytes:
            st.download_button("💾 Download Output", file_bytes, file_name=filename)
    else:
        st.warning("No text extracted.")
