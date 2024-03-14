import streamlit as st
from io import BytesIO
import base64
from streamlit_ace import st_ace
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib.units import inch

# Initialization of session state variables
session_state_keys = ['task_list', 'framework_dict', 'language_dict', 'modules_dict', 'text_dict', 'code_dict']
for key in session_state_keys:
    if key not in st.session_state:
        st.session_state[key] = [] if key == 'task_list' else {}


def create_download_link_pdf(pdf_data, download_filename):
    b64 = base64.b64encode(pdf_data).decode()
    href = f'<a href="data:application/pdf;base64,{b64}" download="{download_filename}">Download PDF</a>'
    return href


def gather_user_inputs():
    app_version = st.text_input("App Version:")
    if st.button("Save App Version") and app_version:
        if app_version not in st.session_state.task_list:
            st.session_state.task_list.append(app_version)

    detail_keys = ["Framework", "Languages", "Modules", "Regression Testing Notes"]
    detail_dicts = ["framework_dict", "language_dict", "modules_dict", "text_dict"]

    for detail, dict_key in zip(detail_keys, detail_dicts):
        input_type = st.text_area if detail == "Regression Testing Notes" else st.text_input
        user_input = input_type(f"{detail} being used:")
        save_button = st.button(f"Save {detail}")

        if save_button and user_input:
            st.session_state[dict_key].setdefault(app_version, []).append(
                user_input.split(',') if ',' in user_input else user_input)

    # Handle code input separately to ensure visibility
    st.caption("Enter your Python code here:")
    code_input = st_ace(language="python", theme="monokai", key=f"code_{app_version}")
    if st.button("Save Code") and code_input:
        st.session_state['code_dict'].setdefault(app_version, []).append(code_input)


def display_saved_items():
    st.write("## Saved Items")
    for version in st.session_state.task_list:
        st.write(f"### App Version: {version}")
        for key in ['framework_dict', 'language_dict', 'modules_dict', 'text_dict', 'code_dict']:
            label = key.replace('_dict', '').capitalize()
            items = st.session_state[key].get(version, [])
            if items:
                st.write(f"#### {label}:")
                for item in items:
                    display_content = ', '.join(item) if isinstance(item, list) and key != 'code_dict' else item
                    if key == 'code_dict':
                        st.code(display_content, language='python')
                    else:
                        st.write(f"- {display_content}")


def generate_pdf_content():
    styles = getSampleStyleSheet()
    elements = []
    for version in st.session_state.task_list:
        elements.append(Paragraph(f"App Version: {version}", styles['Heading2']))
        elements.append(Spacer(1, 0.2 * inch))
        for key in ['framework_dict', 'language_dict', 'modules_dict', 'text_dict', 'code_dict']:
            label = key.replace('_dict', '').capitalize()
            items = st.session_state[key].get(version, [])
            if items:
                elements.append(Paragraph(f"{label}:", styles['Heading3']))
                for item in items:
                    # Ensure content does not include HTML tags
                    content = ', '.join(item) if isinstance(item, list) else item
                    content = content.replace('<', '&lt;').replace('>', '&gt;')  # Basic HTML entities
                    style = styles['Code'] if key == 'code_dict' else styles['BodyText']
                    elements.append(Paragraph(content, style))
                elements.append(Spacer(1, 0.2 * inch))
        elements.append(PageBreak())
    return elements


gather_user_inputs()
display_saved_items()

if st.button("Generate PDF"):
    pdf_buffer = BytesIO()
    doc = SimpleDocTemplate(pdf_buffer, pagesize=letter, leftMargin=50, rightMargin=50, topMargin=50, bottomMargin=50)
    elements = generate_pdf_content()
    doc.build(elements)
    pdf_buffer.seek(0)
    pdf_data = pdf_buffer.getvalue()
    st.markdown(create_download_link_pdf(pdf_data, "App_Details.pdf"), unsafe_allow_html=True)



