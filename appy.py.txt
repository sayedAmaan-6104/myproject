import os
import streamlit as st
from dotenv import load_dotenv
import google.generativeai as genai

# Load environment variables
load_dotenv("ammo.env")
api_key = os.getenv("GOOGLE_API_KEY")

if not api_key:
    st.error("API key not found. Make sure ammo.env contains GOOGLE_API_KEY.")
else:
    genai.configure(api_key=api_key)

    # Correct model name
    try:
        model = genai.GenerativeModel(model_name="gemini-1.5-pro-latest")  # ✅ Corrected model name
    except Exception as e:
        st.error(f"Failed to initialize Gemini model: {e}")

    # Page config
    st.set_page_config(page_title="Smart Application Tracking System", page_icon=":robot:")

    # Background styling
    page_bg_img = """
    <style>
    [data-testid="stAppViewContainer"] > .main {
        background: linear-gradient(to bottom right, #1a0232, #65013d);
        background-size: 180%;
        background-position: top left;
        background-repeat: no-repeat;
        background-attachment: local;
    }
    [data-testid="stHeader"] {
        background: rgba(0,0,0,0);
    }
    [data-testid="stToolbar"] {
        right: 2rem;
    }
    </style>
    """
    st.markdown(page_bg_img, unsafe_allow_html=True)

    # Prompt template
    input_prompt_template = """
    You are an advanced generative AI that can recommend courses from across the web. The user wants to learn about [TOPIC]. Their knowledge level is [KNOWLEDGE_LEVEL], and they have [TIME] available to study.

    Please provide a list of relevant courses in plain, natural language. For each course, include the following details with clear explanations:
    - Course Certificate Type
    - Course Difficulty
    - Course Icon
    - Course Organization
    - Course Rating
    - Course Skills
    - Number of Enrolled Students
    - Course Title
    - Course URL
    - Course ID
    - Image Name
    - List of Sub-courses
    - summarize content explained in meeting

    After listing the courses, add a section titled 'Feedback' where the user can provide feedback if the courses are not relevant. Please format the response in plain text as if you are explaining the options to a human user.
    """

    # Streamlit UI
    st.title("MEET SUMMARIZER & WEB RESOURCE")
    st.text("FROM A TO Z EVERYTHING")

    topic_input = st.text_area("ENTER THE TOPIC!!")
    knowledge_level = st.text_input("Enter your knowledge level (e.g. Absolute Beginner, Intermediate, etc.)")
    study_time = st.text_input("Enter the time available to study (e.g. 4 weeks, 2 months, etc.)")

    submit = st.button("Submit")

    if submit:
        if not topic_input:
            st.error("Please enter a topic.")
        elif not knowledge_level or not study_time:
            st.error("Please provide both your knowledge level and the available time to study.")
        else:
            input_prompt = (
                input_prompt_template
                .replace("[TOPIC]", topic_input)
                .replace("[KNOWLEDGE_LEVEL]", knowledge_level)
                .replace("[TIME]", study_time)
            )

            st.info("Generating course recommendations... Please wait.")
            try:
                response = model.generate_content(input_prompt)
                st.write(response.text)
            except Exception as e:
                st.error(f"An error occurred while generating content: {e}")
