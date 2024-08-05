import streamlit as st from api.api_datagen.mainmod_datagen import bucket_exists,main_call_gen,zip_and_download_gcs_files,copy_gcs_folder
import time
import datetime
import re
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

st.title("CCAI Synthetic Call Generator")
current_timestamp = datetime.datetime.now().strftime("%Y%m%d%H%M%S")

company_name = st.text_input("Company Name", value = "Ulta Beauty")
company_website = st.text_input("Company Website", value = "https://www.ulta.com/")
company_reviews = st.text_input("Company Reviews", value = "https://www.trustpilot.com/review/www.ulta.com")
num_log_files = st.number_input(
    "Number of log files:",
    min_value=1,
    max_value=4001,
    value=3,  # Default value
    step=1,    # Only allow integer increments
)
temperature = st.slider("Temperature", min_value=0.0, max_value=1.0, value=0.8, step=0.1)
gcs_bucket = st.text_input("[OPTIONAL]GCS bucket in your project where data will be copied")
st.info(f"""ðŸ‘† Detailed instructions on GCS bucket creation: [ccai_bucket_creation](https://docs.google.com/document/d/1Q7DQUOagJbPQc7WoRizr023JtCwJAJFdqEnxoqlBBhk/edit?usp=drive_link&resourcekey=0-20oP1QsHihwj6eLYMHEzbQ) \n""")

if not(num_log_files is not None and 1 <= num_log_files <= 4001):
    st.error(f"Num_Log_Files can be between 1 and 4001 only")

ccai_call_gen = st.button("Generate Synthetic Calls")

if ccai_call_gen and company_website and company_reviews and num_log_files:
    if gcs_bucket:
        if not bucket_exists(gcs_bucket):
            st.error(f"Please read instructions and create bucket in your project")
            st.stop()
    if num_log_files > 10:
        msg = f"Seriously you want to generate {num_log_files} conversations!!! I will start spinning some data.."
    else:
        msg = f"Just spinning up some data.."
    with st.spinner(msg):
        uploaded_files = main_call_gen(num_log_files=num_log_files,company_name=company_name,company_website=company_website,company_reviews=company_reviews,temperature=temperature)

        if uploaded_files and not gcs_bucket:
            logger.info(f"Data will be copied to {gcs_bucket}")
            with st.spinner("Zipping files..."):
                time.sleep(5)
                zip_data = zip_and_download_gcs_files(uploaded_files)
            st.download_button(
                label="Download Zip",
                data=zip_data,
                file_name=f"ccai_downloaded_files_{current_timestamp}.zip")
            
        elif uploaded_files and gcs_bucket:
            source_blob = uploaded_files[0]
            logger.info(f"Source blob{source_blob}")
            match = re.search(r'/(.*?)/', source_blob)
            source_folder = match.group(1)
            logger.info(f"Source folder{source_folder}")

            if copy_gcs_folder(source_folder,gcs_bucket):
                st.success(f"Check {source_folder} folder in bucket {gcs_bucket} in your org for the generated data")
            else:
                st.error(f"Things do break..contact @ankurwahi with your bucket name")
        else:
            st.error(f"Process failed, check logs")
