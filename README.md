# text-summarizer
import streamlit as st
from transformers import pipeline
import PyPDF2
import docx
import io

# Load summarization model
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")

# Extract text from PDF
def read_pdf(file):
    pdf_reader = PyPDF2.PdfReader(file)
    text = ""
    for page in pdf_reader.pages:
        text += page.extract_text()
    return text

# Extract text from DOCX
def read_docx(file):
    doc = docx.Document(file)
    return "\n".join([para.text for para in doc.paragraphs])

# App UI
st.title("📝 Text Summarizer")

uploaded_file = st.file_uploader("Upload a document (TXT, PDF, DOCX)", type=["txt", "pdf", "docx"])

text = ""
if uploaded_file is not None:
    if uploaded_file.type == "text/plain":
        stringio = io.StringIO(uploaded_file.getvalue().decode("utf-8"))
        text = stringio.read()
    elif uploaded_file.type == "application/pdf":
        text = read_pdf(uploaded_file)
    elif uploaded_file.type == "application/vnd.openxmlformats-officedocument.wordprocessingml.document":
        text = read_docx(uploaded_file)

if text:
    st.subheader("📄 Original Text")
    st.write(text)

    if st.button("Summarize"):
        with st.spinner("Generating summary..."):
            summary = summarizer(text, max_length=150, min_length=40, do_sample=False)[0]['summary_text']
        st.subheader("🧠 Summary")
        st.success(summary)
