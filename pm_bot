import streamlit as st
from streamlit_chat import message
from streamlit_extras.add_vertical_space import add_vertical_space

from langchain.chat_models import ChatOpenAI
from langchain import LLMChain
from langchain.llms import BaseLLM
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain.prompts import ChatPromptTemplate, SystemMessagePromptTemplate, MessagesPlaceholder, HumanMessagePromptTemplate
from langchain.callbacks import get_openai_callback

import os
# from dotenv import load_dotenv
# load_dotenv()

os.environ["OPENAI_API_KEY"] = st.secrets["OPENAI_API_KEY"]

# defining LLM and memory
llm = ChatOpenAI(model = "gpt-4", temperature = 0.5)
memory = ConversationBufferMemory(return_messages = True)

### helper functions and classes ###

# creating a class to create conversation chains based on description
class CharCreationChain(ConversationChain):

  @classmethod
  def from_description(cls, description ) -> ConversationChain:
    prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template(description),
    MessagesPlaceholder(variable_name = "history"),
    HumanMessagePromptTemplate.from_template("{input}")
])
    
    return cls(prompt=prompt, llm=llm, memory = memory)

def get_initial_message():
    messages = [
        {"role" : "user", "content" : "Can I you interview me for the PM role?"},
        {"role" : "assistant", "content" : "Of course! Let's begin"}
    ]
    return messages

def update_chat(messages, role, content):
    messages.append({"role" : role, "content": content})
    return messages

def remove_last_name(full_name):
    name_parts = full_name.split(' ')
    return ' '.join(name_parts[:-1])

### helper functions and classes - end ###

# creating interviewer persona
char_descriptions = {
    "pm_interviewer_1": "You are Vivian Reyes, a sharp-witted product management guru with years of experience at the world's most innovative companies. Your striking candor and sharp intellect leave interviewees feeling both challenged and inspired. With a tongue-in-cheek wit and a knack for offering incisive observations, your interviews are as enlightening as they are memorable.",
    "pm_interviewer_2" : "You are William Thompson, a seasoned product management expert with hands-on experience in both established corporations and agile startups. Your empathetic nature and genuine curiosity make you an accessible, relatable figure for interviewees. Bill's patient demeanor encourages candidates to think deeply and articulate their thought processes effectively. Armed with his own experiences, he loves to share anecdotes and resonate with candidates on their journey into product management.",
    "pm_interviewer_3" : "You are Dr. Priya Nair, a data-driven product leader with a background in artificial intelligence and machine learning. As an accomplished researcher-turned-product manager, Priya brings her incisive analytical skills to bear during her interviews, questioning assumptions and delving into candidates' problem-solving abilities. She has an unparalleled rigor in her approach to product management interviews, coupled with a soft-spoken and calm demeaner, often drawing candidates into engaging discussions that push the boundaries of their knowledge."
}

interviewer_details = {
    "pm_interviewer_1": {
        "name": "Vivian Reyes",
        "role": "Director of Product",
        "company": "Google"
    },
    "pm_interviewer_2": {
        "name": "William Thompson",
        "role": "Senior Product Manager",
        "company": "Airbnb"
    },
    "pm_interviewer_3": {
        "name": "Dr. Priya Nair",
        "role": "CPO",
        "company": "Scale AI"
    }
}

def format_func(option):
    return interviewer_details[option]["name"] + ", " + interviewer_details[option]["role"] + " at " + interviewer_details[option]["company"]


pm_interviewer_1 = CharCreationChain.from_description(char_descriptions['pm_interviewer_1'])
pm_interviewer_2 = CharCreationChain.from_description(char_descriptions['pm_interviewer_2'])
pm_interviewer_3 = CharCreationChain.from_description(char_descriptions['pm_interviewer_3'])



### Streamlit app ###
st.title("Interview Bot!🤖")

placeholder_heading = st.empty()
with placeholder_heading.container():
    st.write("🚀 This app lets you engage in lifelike interview simulations, helping you build your confidence and skills.")
    st.write("👉 To get started, choose your interviewer, role and topic from the from. Then, click on the button below to begin your interview!")
        

# creating a placeholder to store the user's preferences
placeholder = st.empty()

# interviewer_options = ["pm_interviewer_1", "pm_interviewer_2", "pm_interviewer_3"]
roles = ["APM", "PM", "Senior PM"]
topics = ["Product Strategy", "User Research", "Feature Development"]

if "submitted" not in st.session_state:
    st.session_state.submitted = False

if "interview_started" not in st.session_state:
    st.session_state.interview_started = False



# creating a form to get user's preferences
if not st.session_state.submitted:
    with placeholder.container():
        with st.form("preferences"):
            st.write("Choose your parameters") 
            st.session_state.user_name = st.text_input("Enter your name:")
            # st.session_state.interviewer = st.selectbox("Choose your interviewer:", interviewer_options)
            st.session_state.interviewer = st.selectbox("Choose your interviewer", options=list(interviewer_details.keys()), format_func=format_func)
            st.session_state.role = st.selectbox("Choose your role:", roles)
            st.session_state.topic = st.selectbox("Choose your topic:", topics)
            st.session_state.submitted = st.form_submit_button("Submit")

# displaying the user's preferences
if st.session_state.submitted:
    st.write("Hello", st.session_state.user_name)
    # st.write("You will interview with ", st.session_state.interviewer)
    # st.write("You will be asked questions as per", st.session_state.role, "role")
    # st.write("Questions will be based on", st.session_state.topic)
    interviewer = interviewer_details[st.session_state.interviewer]
    st.markdown(
        f'''
        - You will now be interviewed by {interviewer['name']}, {interviewer['role']} at {interviewer['company']}.
        - You will be asked questions as per the {st.session_state.role} role and the questions will be based on {st.session_state.topic}.
        - Feel free to ask questions and engage in a conversation with {remove_last_name(interviewer['name'])}.
        '''
    )
    if st.button("Begin Interview"):
        st.session_state.interview_started = True
    placeholder_heading.empty()
    placeholder.empty()

    # choosing the interviewer
    conversation = eval(st.session_state.interviewer)

    initial_question_prompt = f"""Prepare a question on {st.session_state.topic} for a {st.session_state.role} role. """

    initial_question = conversation.predict(input = initial_question_prompt)

    ## past - initial user_message should be customised based on the form inputs, 
    ## generated - initial bot response should be an initiation of the interview
    if "generated" not in st.session_state:
        st.session_state['generated'] = [initial_question]
    if "past" not in st.session_state:
        st.session_state['past'] = ["Begin Interview"]

    if st.session_state.interview_started:
        if "query" not in st.session_state:
            st.session_state.query = ""

        def clear_input():
            st.session_state["query"] = st.session_state.widget
            st.session_state.widget = ""

        query = st.text_input("query", key="widget", on_change = clear_input, placeholder= "type your response", label_visibility= "collapsed")

        # query = st.text_input("user_input", placeholder= "type your response", label_visibility= "collapsed" )

        if "messages" not in st.session_state:
            st.session_state['messages'] = get_initial_message()

        if st.session_state.query:
            with st.spinner("typing..."):
                messages = st.session_state['messages']
                messages = update_chat(messages, "user", st.session_state.query)
                response = conversation.predict(input = st.session_state.query)
                messages = update_chat(messages, "assistant", response)
                st.session_state.past.append(st.session_state.query)
                st.session_state.generated.append(response)

        if st.session_state['generated']:
            for i in range(len(st.session_state['generated'])-1, -1, -1):
                # message(st.session_state['past'][i], is_user= True, key = str(i) + "_user")
                message(st.session_state["generated"][i], key = str(i))
                message(st.session_state['past'][i], is_user= True, key = str(i) + "_user", avatar_style="pixel-art")

            # with st.expander("Show Messages"):
            #     st.write(messages)
            st.divider()



