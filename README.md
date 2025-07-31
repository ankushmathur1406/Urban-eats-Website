# Urban-eats-Website

Urban Eats is deployed and can be accessed using the following link:

[Urban Eats Deployment](https://food-delivery-app-client-m6rc.onrender.com/)



%%%
import streamlit as st
import pandas as pd

st.set_page_config(page_title="Parts Availability Checker", layout="wide")
st.title("Upload & Process Files")


llp_table = st.file_uploader("Upload LLP Table (Excel or CSV)", type=['xlsx', 'csv'])
details_table = st.file_uploader("Upload Part Details Table (Excel or CSV)", type=['xlsx', 'csv'])

if llp_table and details_table:
    try:
       
        df_llp = pd.read_excel(llp_table) if llp_table.name.endswith('.xlsx') else pd.read_csv(llp_table)
        df_parts = pd.read_excel(details_table) if details_table.name.endswith('.xlsx') else pd.read_csv(details_table)

       
        df_llp.columns = df_llp.columns.str.strip().str.lower()
        df_parts.columns = df_parts.columns.str.strip().str.lower()

       
        llp_row = df_llp[df_llp['a/c'].str.lower().str.contains('llp')].iloc[0].drop('a/c')
        llp_row = pd.to_numeric(llp_row, errors='coerce').fillna(0)

        
        df_llp_clean = df_llp[~df_llp['a/c'].str.lower().str.contains('llp')].copy()
        for col in df_llp_clean.columns:
            if col != 'a/c':
                df_llp_clean[col] = pd.to_numeric(df_llp_clean[col], errors='coerce').fillna(0)

        
        results = []
        for _, row in df_parts.iterrows():
            ac = row['a/c']
            desc_raw = row['desc'].strip().lower()

            
            part_desc = desc_raw.split('(')[0].strip()
            part_desc = part_desc.replace('/', '').replace('\\', '').replace('-', '').strip()

            matched_col = None
            for col in llp_row.index:
                if part_desc in col:
                    matched_col = col
                    break

            required = int(llp_row.get(matched_col, 0))
            match_row = df_llp_clean[df_llp_clean['a/c'] == ac]
            available = int(match_row.iloc[0][matched_col]) if not match_row.empty and matched_col in match_row.columns else 0

            if available == required:
                status = "ok"
            elif available > required:
                status = "more"
            else:
                status = "short"

            results.append({
                "a/c": ac,
                "part desc": desc_raw,
                "required": required,
                "available": available,
                "result": status  # lowercase key
            })

        
        result_df = pd.DataFrame(results)
        result_df.columns = result_df.columns.str.strip().str.lower()

        st.session_state['merged_df'] = result_df
        st.success("✅ Data processed successfully!")
        st.dataframe(result_df)

    except Exception as e:
        st.error(f"❌ Error: {e}")




%%%%full
import streamlit as st
import pandas as pd

st.set_page_config(page_title="Parts Availability Checker", layout="wide")
st.title("Upload & Process Files")


llp_table = st.file_uploader("Upload LLP Table (Excel or CSV)", type=['xlsx', 'csv'])
details_table = st.file_uploader("Upload Part Details Table (Excel or CSV)", type=['xlsx', 'csv'])

if llp_table and details_table:
    try:
       
        df_llp = pd.read_excel(llp_table) if llp_table.name.endswith('.xlsx') else pd.read_csv(llp_table)
        df_parts = pd.read_excel(details_table) if details_table.name.endswith('.xlsx') else pd.read_csv(details_table)

       
        df_llp.columns = df_llp.columns.str.strip().str.lower()
        df_parts.columns = df_parts.columns.str.strip().str.lower()

       
        llp_row = df_llp[df_llp['a/c'].str.lower().str.contains('llp')].iloc[0].drop('a/c')
        llp_row = pd.to_numeric(llp_row, errors='coerce').fillna(0)

        
        df_llp_clean = df_llp[~df_llp['a/c'].str.lower().str.contains('llp')].copy()
        for col in df_llp_clean.columns:
            if col != 'a/c':
                df_llp_clean[col] = pd.to_numeric(df_llp_clean[col], errors='coerce').fillna(0)

        
        results = []
        for _, row in df_parts.iterrows():
            ac = row['a/c']
            desc_raw = row['desc'].strip().lower()

            
            part_desc = desc_raw.split('(')[0].strip()
            part_desc = part_desc.replace('/', '').replace('\\', '').replace('-', '').strip()

            matched_col = None
            for col in llp_row.index:
                if part_desc in col:
                    matched_col = col
                    break

            required = int(llp_row.get(matched_col, 0))
            match_row = df_llp_clean[df_llp_clean['a/c'] == ac]
            available = int(match_row.iloc[0][matched_col]) if not match_row.empty and matched_col in match_row.columns else 0

            if available == required:
                status = "ok"
            elif available > required:
                status = "more"
            else:
                status = "short"

            results.append({
                "a/c": ac,
                "part desc": desc_raw,
                "required": required,
                "available": available,
                "result": status  # lowercase key
            })

        
        result_df = pd.DataFrame(results)
        result_df.columns = result_df.columns.str.strip().str.lower()

        st.session_state['merged_df'] = result_df
        st.success("✅ Data processed successfully!")
        st.dataframe(result_df)

    except Exception as e:
        st.error(f"❌ Error: {e}")


%%%imperfect
import streamlit as st
import pandas as pd
from io import BytesIO
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph
from reportlab.lib import colors
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet

st.title("Imperfect Records: Short / More")

def generate_pdf(df, title):
    buffer = BytesIO()
    doc = SimpleDocTemplate(buffer, pagesize=A4)
    styles = getSampleStyleSheet()
    elements = [Paragraph(title, styles['Heading2'])]

    data = [df.columns.tolist()] + df.values.tolist()
    table = Table(data, hAlign='LEFT')
    table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.lightgrey),
        ('GRID', (0, 0), (-1, -1), 0.5, colors.black),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
        ('FONTSIZE', (0, 0), (-1, -1), 9),
        ('ALIGN', (0, 0), (-1, -1), 'LEFT'),
        ('VALIGN', (0, 0), (-1, -1), 'MIDDLE'),
    ]))
    elements.append(table)
    doc.build(elements)
    pdf = buffer.getvalue()
    buffer.close()
    return pdf

if 'merged_df' in st.session_state:
    df = st.session_state['merged_df']

    
    if 'result' not in df.columns:
        computed_results = []
        for _, row in df.iterrows():
            available = int(row.get('available', 0))
            required = int(row.get('required', 0))
            if available == required:
                result = 'ok'
            elif available > required:
                result = 'more'
            else:
                result = 'short'
            computed_results.append(result)

        df['result'] = computed_results

    
    filtered = df[df['result'] != 'ok']

    if not filtered.empty:
        st.subheader("Imperfect Records Found")
        st.dataframe(filtered)

       
        pdf = generate_pdf(filtered, "Imperfect Records (Short / More)")
        st.download_button("📄 Download Imperfect Records as PDF", pdf, file_name="imperfect_result.pdf")

        
        csv = filtered.to_csv(index=False).encode('utf-8')
        st.download_button("📄 Download Imperfect Records as CSV", csv, file_name="imperfect_result.csv")
    else:
        st.success("✅ All records are OK. No imperfections found.")
else:
    st.warning("⚠️ Please upload and process data from the Home page.")

%%serach
import streamlit as st
import pandas as pd
from io import BytesIO
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph
from reportlab.lib import colors
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet

st.set_page_config(page_title="Search by A/C")
st.title("Search Aircraft Parts by A/C")

def generate_pdf(df, title):
    buffer = BytesIO()
    doc = SimpleDocTemplate(buffer, pagesize=A4)
    styles = getSampleStyleSheet()
    elements = [Paragraph(title, styles['Heading2'])]
    data = [df.columns.tolist()] + df.values.tolist()
    table = Table(data, hAlign='LEFT')
    table.setStyle(TableStyle([
        ('BACKGROUND', (0, 0), (-1, 0), colors.lightgrey),
        ('GRID', (0, 0), (-1, -1), 0.5, colors.black),
        ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),
    ]))
    elements.append(table)
    doc.build(elements)
    pdf = buffer.getvalue()
    buffer.close()
    return pdf

if 'merged_df' in st.session_state:
    merged_df = st.session_state['merged_df']

    
    merged_df.columns = merged_df.columns.str.strip().str.lower()

    ac_list = merged_df['a/c'].unique().tolist()

    st.subheader("Select Aircraft and Filter")

    selected_ac = st.selectbox("✈️ Select Aircraft", options=ac_list)
    filter_option = st.radio("Show:", ["All", "Only Imperfect"], horizontal=True)

    filtered_df = merged_df[merged_df['a/c'] == selected_ac]
    if filter_option == "Only Imperfect":
        filtered_df = filtered_df[filtered_df['result'] != "ok"]

    if not filtered_df.empty:
        st.subheader(f"Results for A/C: {selected_ac} ({filter_option})")
        st.dataframe(filtered_df)

        pdf = generate_pdf(filtered_df, f"Parts for A/C {selected_ac} ({filter_option})")
        st.download_button("📄 Download as PDF", pdf, file_name=f"{selected_ac}_filtered.pdf")

        csv = filtered_df.to_csv(index=False).encode('utf-8')
        st.download_button("📄 Download as CSV", csv, file_name=f"{selected_ac}_filtered.csv")
    else:
        st.warning("No matching records found.")
else:
    st.warning("Please upload and process data from the Home page first.")

