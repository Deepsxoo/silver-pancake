# silver-pancake


# from langchain.llms import OpenAI
from matplotlib import pyplot as plt
from pandasai import SmartDataframe
from pandasai import PandasAI
from pandasai.llm import OpenAI,AzureOpenAI
from pandasai.callbacks import StdoutCallback
from pandasai.llm import OpenAI
import streamlit as st
import pandas as pd
import os
from pandasai_app.components.faq import faq
#,encoding= 'unicode_escape',low_memory=False
file_formats = {
    "csv": pd.read_csv,
    "xls": pd.read_excel,
    "xlsx": pd.read_excel,
    "xlsm": pd.read_excel,
    "xlsb": pd.read_excel,
}

def clear_submit():
    """
    Clear the Submit Button State
    Returns:

    """
    st.session_state["submit"] = False


@st.cache_data(ttl="2h")
def load_data(uploaded_file):
    try:
        ext = os.path.splitext(uploaded_file.name)[1][1:].lower()
    except:
        ext = uploaded_file.split(".")[-1]
    if ext in file_formats:
        return file_formats[ext](uploaded_file,encoding= 'unicode_escape',low_memory=False)
    else:
        st.error(f"Unsupported file format: {ext}")
        return None


st.set_page_config(page_title="PandasAI ", page_icon="🐼")
st.title("🐼 PandasAI: Chat with CSV")

uploaded_file = st.file_uploader(
    "Upload a Data file",
    type=list(file_formats.keys()),
    help="Various File formats are Support",
    on_change=clear_submit,
)

if uploaded_file:
    data = load_data(uploaded_file)
    data = data[data['Qty'].notnull()]
    data = data[data['Category'].notnull()]
    data = data[data['Net_Price'].notnull()]
    #data = data[data['PO Value'].notnull()]
    data = data[data['Sales_Price'].notnull()]
    data = data[data['Discount_Percent'].notnull()]
    #data = data[data['No of months'].notnull()]
    data = data[data['Customer_Type'].notnull()]
    data = data[data['Classification'].notnull()]
    data = data[data['PO_Date'].notnull()]

    data = data[data['Quarter'].notnull()]
    data.loc[data['Quarter'] == 'Q1-21', 'Quarter'] = '2021Q1'
    data.loc[data['Quarter'] == 'Q1-22', 'Quarter'] = '2022Q1'
    data.loc[data['Quarter'] == 'Q1-22.23', 'Quarter'] = '2023Q1'
    data.loc[data['Quarter'] == 'Q1-23.24', 'Quarter'] = '2024Q1'
    data.loc[data['Quarter'] == 'Q2-21', 'Quarter'] = '2021Q2'
    data.loc[data['Quarter'] == 'Q2-22', 'Quarter'] = '2022Q2'
    data.loc[data['Quarter'] == 'Q2.22-23', 'Quarter'] = '2023Q2'
    data.loc[data['Quarter'] == 'Q3-21', 'Quarter'] = '2021Q3'
    data.loc[data['Quarter'] == 'Q3-22', 'Quarter'] = '2022Q3'
    data.loc[data['Quarter'] == 'Q3.22-23', 'Quarter'] = '2023Q3'
    data.loc[data['Quarter'] == 'Q4-21', 'Quarter'] = '2021Q4'
    data.loc[data['Quarter'] == 'Q4-22', 'Quarter'] = '2022Q4'
    data.loc[data['Quarter'] == 'Q4.22-23', 'Quarter'] = '2023Q4'



    data['date'] = pd.PeriodIndex(data['Quarter'], freq='Q').strftime('%m-%Y')
    data = data.sort_values(by='Quarter', ascending=True)
    data = data[data['date'].notnull()]
    # data['Qty'] = pd.to_numeric(data['Qty'])
    # df_grouped = data.groupby(['Quarter', 'Type', 'Region'])['Qty'].sum().reset_index()
    # print(df_grouped)

openai_api_key = st.sidebar.text_input("OpenAI API Key",
                                        type="password",
                                        placeholder="Paste your OpenAI API key here (sk-...)")

with st.sidebar:
        st.markdown("---")
        st.markdown(
            "## How to use\n"
            "1. Enter your [OpenAI API key](https://platform.openai.com/account/api-keys) below🔑\n"  # noqa: E501
            "2. Upload a csv file with data📄\n"
            "3. A csv file is read as Pandas Dataframe📄\n"
            "4. Ask a question about to make dataframe conversational💬\n"
        )

        st.markdown("---")
        st.markdown("# About")
        st.markdown(
            "📖Pandasai App allows you to ask questions about your "
            "csv / dataframe and get accurate answers"
        )
        st.markdown(
            "Pandasai is in active development so is this tool."
            "You can contribute to the project on [GitHub]() "  # noqa: E501
            "with your feedback and suggestions💡"
        )
        st.markdown("Made by [DR. AMJAD RAZA](https://www.linkedin.com/in/amjadraza/)")
        st.markdown("---")

        faq()

if "messages" not in st.session_state or st.sidebar.button("Clear conversation history"):
    st.session_state["messages"] = [{"role": "assistant", "content": "How can I help you?"}]

for msg in st.session_state.messages:
    st.chat_message(msg["role"]).write(msg["content"])

if prompt := st.chat_input(placeholder="What is this data about?"):
    st.session_state.messages.append({"role": "user", "content": prompt})
    st.chat_message("user").write(prompt)

    # if not openai_api_key:
    #     st.info("Please add your OpenAI API key to continue.")
    #     st.stop()

    #PandasAI OpenAI Model
    #llm = OpenAI(api_token=openai_api_key)
    # llm = OpenAI(api_token=openai_api_key)
    #api_base ="https://hclsparcaimodel.openai.azure.com/"
    api_base ="https://hclnewopenai.openai.azure.com/"
    #api_version = "2023-03-15-preview"
    api_version = "2023-05-15"
    #api_key = "bbf9640b5603473b8200e4bb9c0d03d3"
    api_key = "25e42b9ad133452288c6dca62695b8ef"
    deployment_id = "hclnewopenaicricket" #"sparcgpt4" 
    llm = AzureOpenAI(api_base=api_base,api_version = api_version,api_token = api_key, deployment_name = deployment_id)
    # sdf = SmartDataframe(data, config = {"llm": llm,
    #                                     "enable_cache": False,
    #                                     "conversational": True,
    #                                     "callback": StdoutCallback()})
    #sdf = SmartDataframe(data, config={"llm": llm})
    #print(sdf)
    pandas_ai = PandasAI(llm,save_charts=True,save_charts_path ="C:/ReportingApp/pandasai-app/pandasai_app",conversational=False,callback=StdoutCallback(),enable_cache = False)

    with st.chat_message("assistant"):
        x = pandas_ai.run(data, prompt=st.session_state.messages)
        #response = sdf.chat(st.session_state.messages)
        fig = plt.gcf()
        #fig, ax = plt.subplots(figsize=(10, 6))
        plt.tight_layout()
        if fig.get_axes() and fig is not None:
            st.pyplot(fig)
            fig.savefig("plot.png")
        #st.write(x)
        #st.session_state.prompt_history.append(question)
        #response_history.append(x)  # Append the response to the list
        st.session_state.messages.append({"role": "assistant", "content": x})
        st.write(x)# from langchain.llms import OpenAI
from matplotlib import pyplot as plt
from pandasai import SmartDataframe
from pandasai import PandasAI
from pandasai.llm import OpenAI,AzureOpenAI
from pandasai.callbacks import StdoutCallback
from pandasai.llm import OpenAI
import streamlit as st
import pandas as pd
import os
from pandasai_app.components.faq import faq
#,encoding= 'unicode_escape',low_memory=False
file_formats = {
    "csv": pd.read_csv,
    "xls": pd.read_excel,
    "xlsx": pd.read_excel,
    "xlsm": pd.read_excel,
    "xlsb": pd.read_excel,
}

def clear_submit():
    """
    Clear the Submit Button State
    Returns:

    """
    st.session_state["submit"] = False


@st.cache_data(ttl="2h")
def load_data(uploaded_file):
    try:
        ext = os.path.splitext(uploaded_file.name)[1][1:].lower()
    except:
        ext = uploaded_file.split(".")[-1]
    if ext in file_formats:
        return file_formats[ext](uploaded_file,encoding= 'unicode_escape',low_memory=False)
    else:
        st.error(f"Unsupported file format: {ext}")
        return None


st.set_page_config(page_title="PandasAI ", page_icon="🐼")
st.title("🐼 PandasAI: Chat with CSV")

uploaded_file = st.file_uploader(
    "Upload a Data file",
    type=list(file_formats.keys()),
    help="Various File formats are Support",
    on_change=clear_submit,
)

if uploaded_file:
    data = load_data(uploaded_file)
    data = data[data['Qty'].notnull()]
    data = data[data['Category'].notnull()]
    data = data[data['Net_Price'].notnull()]
    #data = data[data['PO Value'].notnull()]
    data = data[data['Sales_Price'].notnull()]
    data = data[data['Discount_Percent'].notnull()]
    #data = data[data['No of months'].notnull()]
    data = data[data['Customer_Type'].notnull()]
    data = data[data['Classification'].notnull()]
    data = data[data['PO_Date'].notnull()]

    data = data[data['Quarter'].notnull()]
    data.loc[data['Quarter'] == 'Q1-21', 'Quarter'] = '2021Q1'
    data.loc[data['Quarter'] == 'Q1-22', 'Quarter'] = '2022Q1'
    data.loc[data['Quarter'] == 'Q1-22.23', 'Quarter'] = '2023Q1'
    data.loc[data['Quarter'] == 'Q1-23.24', 'Quarter'] = '2024Q1'
    data.loc[data['Quarter'] == 'Q2-21', 'Quarter'] = '2021Q2'
    data.loc[data['Quarter'] == 'Q2-22', 'Quarter'] = '2022Q2'
    data.loc[data['Quarter'] == 'Q2.22-23', 'Quarter'] = '2023Q2'
    data.loc[data['Quarter'] == 'Q3-21', 'Quarter'] = '2021Q3'
    data.loc[data['Quarter'] == 'Q3-22', 'Quarter'] = '2022Q3'
    data.loc[data['Quarter'] == 'Q3.22-23', 'Quarter'] = '2023Q3'
    data.loc[data['Quarter'] == 'Q4-21', 'Quarter'] = '2021Q4'
    data.loc[data['Quarter'] == 'Q4-22', 'Quarter'] = '2022Q4'
    data.loc[data['Quarter'] == 'Q4.22-23', 'Quarter'] = '2023Q4'



    data['date'] = pd.PeriodIndex(data['Quarter'], freq='Q').strftime('%m-%Y')
    data = data.sort_values(by='Quarter', ascending=True)
    data = data[data['date'].notnull()]
    # data['Qty'] = pd.to_numeric(data['Qty'])
    # df_grouped = data.groupby(['Quarter', 'Type', 'Region'])['Qty'].sum().reset_index()
    # print(df_grouped)

openai_api_key = st.sidebar.text_input("OpenAI API Key",
                                        type="password",
                                        placeholder="Paste your OpenAI API key here (sk-...)")

with st.sidebar:
        st.markdown("---")
        st.markdown(
            "## How to use\n"
            "1. Enter your [OpenAI API key](https://platform.openai.com/account/api-keys) below🔑\n"  # noqa: E501
            "2. Upload a csv file with data📄\n"
            "3. A csv file is read as Pandas Dataframe📄\n"
            "4. Ask a question about to make dataframe conversational💬\n"
        )

        st.markdown("---")
        st.markdown("# About")
        st.markdown(
            "📖Pandasai App allows you to ask questions about your "
            "csv / dataframe and get accurate answers"
        )
        st.markdown(
            "Pandasai is in active development so is this tool."
            "You can contribute to the project on [GitHub]() "  # noqa: E501
            "with your feedback and suggestions💡"
        )
        st.markdown("Made by [DR. AMJAD RAZA](https://www.linkedin.com/in/amjadraza/)")
        st.markdown("---")

        faq()

if "messages" not in st.session_state or st.sidebar.button("Clear conversation history"):
    st.session_state["messages"] = [{"role": "assistant", "content": "How can I help you?"}]

for msg in st.session_state.messages:
    st.chat_message(msg["role"]).write(msg["content"])

if prompt := st.chat_input(placeholder="What is this data about?"):
    st.session_state.messages.append({"role": "user", "content": prompt})
    st.chat_message("user").write(prompt)

    # if not openai_api_key:
    #     st.info("Please add your OpenAI API key to continue.")
    #     st.stop()

    #PandasAI OpenAI Model
    #llm = OpenAI(api_token=openai_api_key)
    # llm = OpenAI(api_token=openai_api_key)
    #api_base ="https://hclsparcaimodel.openai.azure.com/"
    api_base ="https://hclnewopenai.openai.azure.com/"
    #api_version = "2023-03-15-preview"
    api_version = "2023-05-15"
    #api_key = "bbf9640b5603473b8200e4bb9c0d03d3"
    api_key = "25e42b9ad133452288c6dca62695b8ef"
    deployment_id = "hclnewopenaicricket" #"sparcgpt4" 
    llm = AzureOpenAI(api_base=api_base,api_version = api_version,api_token = api_key, deployment_name = deployment_id)
    # sdf = SmartDataframe(data, config = {"llm": llm,
    #                                     "enable_cache": False,
    #                                     "conversational": True,
    #                                     "callback": StdoutCallback()})
    #sdf = SmartDataframe(data, config={"llm": llm})
    #print(sdf)
    pandas_ai = PandasAI(llm,save_charts=True,save_charts_path ="C:/ReportingApp/pandasai-app/pandasai_app",conversational=False,callback=StdoutCallback(),enable_cache = False)

    with st.chat_message("assistant"):
        x = pandas_ai.run(data, prompt=st.session_state.messages)
        #response = sdf.chat(st.session_state.messages)
        fig = plt.gcf()
        #fig, ax = plt.subplots(figsize=(10, 6))
        plt.tight_layout()
        if fig.get_axes() and fig is not None:
            st.pyplot(fig)
            fig.savefig("plot.png")
        #st.write(x)
        #st.session_state.prompt_history.append(question)
        #response_history.append(x)  # Append the response to the list
        st.session_state.messages.append({"role": "assistant", "content": x})
        st.write(x)
