---
aliases:
  - "streamlit_notes"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Streamlit/streamlit_notes.md"
imported: 2026-09-24
---

# Streamlit Notes

## Contents

- [[#Introduction]]
- [[#Installation and Basic Features]]
- [[#Widgets]]
- [[#Layouts]]
- [[#Visualizations]]
- [[#File Upload]]
- [[#State Management]]
- [[#Performance Optimization]]
- [[#Dashboards]]
- [[#Deployment]]
- [[#Authentication]]
- [[#Project]]

## Introduction
- Data visualization is the graphical representation of information and data. It transforms raw data into charts, graphs, and dashboards, making patterns, trends, and correlations easier to understand.
- Visualization is essential in the field of data engineering for several reasons:
    - Simplifies complex data: Turns large datasets into digestible visuals.
    - Reveals patterns & trends: Highlights relationships not obvious in raw numbers.
    - Improves decision-making: Visual data is quicker to interpret.
    - Increases engagement: People respond better to interactive visuals.
    - Facilitates storytelling: Helps communicate insights effectively.
- Popular Visualization Tools:
    - Python Libraries: Matplotlib, Seaborn, Plotly, Altair, Bokeh
    - JavaScript Libraries: D3.js, Chart.js, ECharts
    - BI & Dashboard Tools: Tableau, Power BI, Looker
    - Low-code Tools: Google Data Studio, Flourish
    - Streamlit Integrations: Works seamlessly with Plotly, Altair, Matplotlib, Deck.gl, Vega-Lite
- Streamlit is a Python library that lets you create interactive web apps quickly without HTML, CSS, or JavaScript. You write Python code, run it, and instantly get a sharable web app that updates as your code changes. It is a good visualization tool because it's at the intersection of data science and web development — it allows you to:
    - Turn Python scripts into shareable web apps instantly without any front end coding.
    - Combine data processing and visualization in the same environment.
    - Use interactive widgets (sliders, dropdowns, filters) to update charts in real-time.
    - Integrate with multiple visualization libraries without extra boilerplate code.
    - Deploy easily without needing HTML/CSS/JS skills.
- Streamlit is best used for:
    - Data scientists who want to present findings interactively.
    - Quick internal dashboards.
    - Prototypes for analytics products.
    - Real-time analytics apps.

## Installation and Basic Features
- If a virtual Python environment doesn't already exist, run the following commands to install Streamlit:
    ```python
    python3 -m venv .venv
    source .venv/bin/activate
    python -m pip install --upgrade pip
    pip install streamlit
    streamlit hello
    ```
- Create an `app.py` file for your Streamlit app with the following code:
    ```python
    import pandas as pd
    import streamlit as st

    from PIL import Image

    # Setup
    st.title("Hello Streamlit!")
    st.write("This is my first Streamlit app.")

    # Displaying Text
    st.title("Streamlit Basics")
    st.header("This is a header")
    st.subheader("This is a subheader")
    st.text("This is plain text.")
    st.markdown("**Markdown** _is_ supported!")  # Bold and italic

    # Displaying Data
    data = {
        "Name": ["Alice", "Bob", "Charlie"],
        "Age": [25, 30, 35]
    }

    df = pd.DataFrame(data)

    st.dataframe(df)  # Scrollable, interactive table
    st.table(df)      # Static table

    # Displaying Media (images, audio, and video)
    image = Image.open("my_image.jpg")
    st.image(image, caption="Sample Image", use_column_width=True)

    st.audio("sample_audio.mp3") # Relative path to the file.

    st.video("sample_video.mp4")
    ```
    - Use `st.markdown` for rich formatting (links, lists, colors, etc.).
- To run the app, run the command `streamlit run app.py`.

## Widgets
- Widgets are interactive UI elements that allow users to input data, make selections, and trigger actions. Examples include:
    - Button - Runs code when clicked.
    - Slider - Lets users pick a number or range.
    - Select Box - Dropdown menu to choose an option.
    - Text Input - Enter text.
- **Every widget automatically reruns the script when the user interacts with it — this is how Streamlit updates the app instantly**.
- The following app uses common Streamlit widgets:
    ```python
    import streamlit as st

    # Button
    if st.button("Click Me!"): # Code block only executes when button is clicked.
        st.write("Button was clicked!")
    
    # Text Input
    name = st.text_input("Enter your name:")
    if name:
        st.write(f"Hello, {name}!")

    # Slider
    age = st.slider("Select your age:", 0, 100, 25)
    st.write(f"You are {age} years old.")

    # Number Input
    number = st.number_input("Enter a number:", min_value=0, max_value=10, value=5)
    st.write(f"You chose: {number}")

    # Select Box
    option = st.selectbox(
        "Choose your favorite fruit:",
        ("Apple", "Banana", "Cherry")
    )

    st.write(f"You selected: {option}")

    # Multi-Select
    choices = st.multiselect(
        "Select your favorite colors:",
        ["Red", "Green", "Blue", "Yellow"],
        default=["Red"]
    )

    st.write(f"You chose: {choices}")

    # File Uploader
    uploaded_file = st.file_uploader("Choose a CSV file", type="csv")
    
    if uploaded_file is not None:
        import pandas as pd
        df = pd.read_csv(uploaded_file)
        st.write(df.head())
    ```

## Layouts
- A Streamlit app can quickly become cluttered if all elements are stacked vertically. Proper layout ensures easier navigation, cleaner design, and better user experience.
- Sidebars are perfect for filters, menus, or app settings.
- Columns let you place items side-by-side.
- Expandable and collapsible sections are great for optional or detailed info.
- Streamlit allows an app's basic theme to be configured using a `config.toml` file or app settings. The `config.toml` file should be created in the `.streamlit/` directory of a project, if it doesn't already exist. **The .streamlit directory should be automatically created in the folder of the main user on your Mac. You don't create the folder yourself**. Typical settings for the `config.toml` file include:
    ```python
    [theme]
    primaryColor = "#F63366"
    backgroundColor = "#0E1117"
    secondaryBackgroundColor = "#262730"
    textColor = "#FAFAFA"
    ```
- Example App:
    ```sql
    import streamlit as st

    st.title("Main Area")
    st.write("This is the main content of the app.")

    # Sidebar
    st.sidebar.title("Sidebar Menu")
    username = st.sidebar.text_input("Enter your name:")
    option = st.sidebar.selectbox("Choose a category:", ["News", "Sports", "Tech"])

    # Columns
    col1, col2, col3 = st.columns(3)
    # col1, col2, col3 = st.columns([3, 1, 2]) is an alternative way to declare columns and specify their width at the same time.

    with col1:
        st.header("Column 1")
        st.write("Content for column 1.")

    with col2:
        st.header("Column 2")
        st.write("Content for column 2.")

    with col3:
        st.header("Column 3")
        st.write("Content for column 3.")

    col1, col2, col3 = st.columns([3, 1, 2]) # Specifies column widths.

    # Expanders
    with st.expander("See more details"):
        st.write("Here you can add extra information.")
    ```

## Visualizations
- Notable Streamlit Visualization Properties:
    - Charts update in real time when users change filters.
    - You can combine data processing + visualization in one script.
    - Streamlit supports multiple visualization libraries out-of-the-box.
- Pandas Data Frame Streamlit Charting Functions:
    ```python
    import streamlit as st
    import pandas as pd
    import numpy as np

    # Sample data
    chart_data = pd.DataFrame(
        np.random.randn(20, 3), # Creates a 2D array of random numbers with 20 rows and 3 columns.
        columns=["A", "B", "C"]
    )

    st.line_chart(chart_data)
    st.area_chart(chart_data)
    st.bar_chart(chart_data)
    ```
- Matplotlib in Streamlit
    ```python
    import matplotlib.pyplot as plt

    fig, ax = plt.subplots()
    ax.plot([1, 2, 3, 4], [10, 20, 25, 30]) # x-axis values, then y-axis values
    ax.set_title("Matplotlib Line Chart")
    st.pyplot(fig)
    ```
- Plotly in Streamlit (Interactive Charts):
    ```python
    import plotly.express as px

    df = pd.DataFrame({
        "Fruit": ["Apple", "Banana", "Orange", "Apple", "Banana", "Orange"],
        "Amount": [4, 1, 2, 2, 4, 5],
        "City": ["NY", "NY", "NY", "LA", "LA", "LA"]
    })

    fig = px.bar(df, x="Fruit", y="Amount", color="City", barmode="group")
    st.plotly_chart(fig)
    ```
- Altair in Streamlit:
    ```python
    import altair as alt

    df = pd.DataFrame({
        'x': range(1, 11),
        'y': [x**2 for x in range(1, 11)]
    })

    chart = alt.Chart(df).mark_line().encode(
        x='x',
        y='y'
    )
    st.altair_chart(chart, use_container_width=True) # Useful for plotting statistical data.
    ```
- Mini Project
    ```python
    import streamlit as st
    import pandas as pd
    import plotly.express as px
    import requests

    st.title("COVID-19 Live Cases Dashboard")

    @st.cache_data # Caches data retrieved from URL.
    def load_data():
        url = "https://disease.sh/v3/covid-19/historical/all?lastdays=all"
        response = requests.get(url)

        data = response.json()

        df = pd.DataFrame({
            "date": list(data["cases"].keys()),
            "cases": list(data["cases"].values()),
            "deaths": list(data["deaths"].values()),
            "recovered": list(data["recovered"].values())
        })

        return df

    df = load_data()

    # Plot cases over time
    fig = px.line(df, x="date", y=["cases", "deaths", "recovered"],
                title="Global COVID-19 Cases Over Time")
    st.plotly_chart(fig, use_container_width=True)
    ```

## File Upload
- Loading CSV Files:
    ```python
    import streamlit as st
    import pandas as pd

    df = pd.read_csv("sample.csv")
    st.write(df.head())  # Display first 5 rows
    ```
- Loading Excel Files
    ```python
    import streamlit as st
    import pandas as pd

    df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
    st.write(df)
    ```
    - Requires installing openpyxl with: `pip install openpyxl`
- Loading API Data:
    ```python
    import requests
    import streamlit as st

    def get_bitcoin_price():
        url = "https://api.coingecko.com/api/v3/simple/price"
        params = {"ids": "bitcoin", "vs_currencies": "usd"}
        resp = requests.get(url, params=params, timeout=10)

        if resp.ok:
            data = resp.json()
            return data["bitcoin"]["usd"]
        else:
            st.error(f"API error: {resp.status_code}")
            return None

    st.subheader("💰 Live Bitcoin Price")

    price = get_bitcoin_price()

    if price:
        st.success(f"Current BTC Price: ${price:,.2f}")
    ```
- User-Uploaded File:
    ```python
    import streamlit as st
    import pandas as pd

    uploaded_file = st.file_uploader("Choose a CSV file", type=["csv", "xlsx"])

    if uploaded_file is not None:
        if uploaded_file.name.endswith("csv"):
            df = pd.read_csv(uploaded_file)
        else:
            df = pd.read_excel(uploaded_file)

        st.write(df.head())
    
    age = st.selectbox("Select Category", df['Age'].unique())
    filtered_df = df[df['Age'] == age]
    st.write(filtered_df)
    ```
- Filtering Data:
    ```python
    # Example: Filter by category
    category = st.selectbox("Select Category", df["Category"].unique())

    filtered_df = df[df["Category"] == category]

    st.write(filtered_df)
    ```
- User-Downloaded Data:
    ```python
    csv = df.to_csv(index=False).encode("utf-8")

    st.download_button(
        label="Download data as CSV",
        data=csv,
        file_name="data.csv",
        mime="text/csv",
    )
    ```
- Mini Project
    ```python
    import streamlit as st
    import pandas as pd
    import yfinance as yf

    st.title("Stock Price Viewer")

    # User selects stock
    ticker = st.text_input("Enter Stock Ticker (e.g., AAPL, MSFT):", "AAPL")

    if st.button("Get Data"):
        stock = yf.Ticker(ticker)
        df = stock.history(period="1mo")  # 1 month data
        st.line_chart(df["Close"])

        # Download option
        csv = df.to_csv().encode("utf-8")
        st.download_button(
            label="Download CSV",
            data=csv,
            file_name=f"{ticker}_data.csv",
            mime="text/csv"
        )
    ```
    - Requires installing yfinance with: `pip install yfinance`

## State Management
- Streamlit reruns your script top-to-bottom whenever a widget changes. Without state management, your app will "forget" everything each time this happens. For example, consider a messaging app where the previous message disappears every time you send a new one, instead of being able to see your entire chat history.
- A built-in dictionary that persists values across reruns helps mitigate this issue. `st.session_state` acts as a global "memory box" for your app. With this function you can:
    - Initialize Values: `st.session_state["counter"] = 0`
    - Re-Read Them: `st.session_state.counter`
    - Update Them on Interaction: `st.session_state.count -= 1`
- Example App:
    ```python
    import streamlit as st

    st.title("Counter Demo")

    # Initialize state
    if "count" not in st.session_state:
        st.session_state.count = 0

    # Buttons to modify existing state
    if st.button("Increment"):
        st.session_state.count += 1

    if st.button("Decrement"):
        st.session_state.count -= 1

    # Re-read state after modification
    st.write("Current Count:", st.session_state.count)
    ```
    - The counter won't reset each time a button is clicked, like a normal variable would. This is because it's stored in the state dictionary.
- Using State with Widgets:
    ```python
    import streamlit as st

    st.title("Session State with Widgets")

    st.text_input("Enter your name:", key="username")
    st.slider("Select age:", 0, 100, key="age")

    st.write("Hello,", st.session_state.username, "!")
    st.write("Your age is:", st.session_state.age)
    ```
    - Here, `username` and `age` values are initialized and updated in the session state using the `key` parameter. These values stay persistent even if you interact with other widgets.
- Callbacks with State:
    ```python
    import streamlit as st

    st.title("Callback Example")

    def reset_age():
        st.session_state.age = 18

    st.number_input("Enter age:", min_value=0, max_value=100, key="age")
    st.button("Reset Age", on_click=reset_age) # Resets age value to 19

    st.write("Current Age:", st.session_state.age)
    ```
    - Callbacks let you run a function when a widget changes. Here, the `on_click` parameter of the button allows you to reset the value of `age` stored in `session_state` to 18 when the button is clicked.
- Forms with State:
    ```sql
    import streamlit as st

    st.title("Form with State Example")

    if "age" not in st.session_state:
        st.session_state.age = 18

    with st.form("age_form"):
        age = st.number_input("Enter age:", min_value=0, max_value=100, value=st.session_state.age)
        submitted = st.form_submit_button("Submit")

        if submitted:
            st.session_state.age = age

    st.write("Current Age:", st.session_state.age)
    ```
    - Forms group widgets together and prevent re-runs until the form's "submit" button is clicked. The above code demonstrates stable input handling when using session state.
- Chat App Example:
    ```sql
    import streamlit as st

    st.title("Simple Chat App")

    # Initialize messages list in session_state
    if "messages" not in st.session_state:
        st.session_state.messages = []

    # Chat input form
    with st.form("chat_form"):
        msg = st.text_input("Enter a message:")
        send = st.form_submit_button("Send")

    # Append new message if submitted
    if send and msg:
        st.session_state.messages.append(msg)

    # Display chat history
    st.subheader("Chat History")

    for i, m in enumerate(st.session_state.messages, 1): # Automatically adds a counter variable (starting at 1) when iterating through the list.
        st.write(f"{i}. {m}")
    ```

## Performance Optimization
- When Streamlit apps become very large, they can encounter performance issues due to heavy data loads, recomputing expensive operations, and redundant re-runs.
- Streamlit offers several tools to optimize an app's performance, including:
    - Caching Data with `@st.cache_data`:
        ```python
        import streamlit as st
        import pandas as pd

        @st.cache_data # Acts as an annotation for the function, caching the return value. Not a stand-alone method call.
        def load_data():
            url = "https://people.sc.fsu.edu/~jburkardt/data/csv/airtravel.csv"
            return pd.read_csv(url)

        df = load_data()  # Now it's cached
        st.write(df.head())
        ```
        - When the data is cached, it doesn't need to be reloaded for each re-run.
    - Caching Computations with `@st.cache_resource`:
        ```python
        import streamlit as st
        import time

        @st.cache_resource
        def expensive_function(x):
            time.sleep(3)  # simulate heavy computation
            return x * x

        st.write("Square:", expensive_function(10))
        ```
        - Annotating the `expensive_function` method prevents timely recomputation for each re-run of the app.
- Combining Forms, Callbacks, and Caching:
    ```sql
    import streamlit as st
    import pandas as pd

    @st.cache_data
    def load_data():
        url = "https://people.sc.fsu.edu/~jburkardt/data/csv/airtravel.csv"
        return pd.read_csv(url)

    df = load_data() # Data retrieved from API once, then retrieved from cache in subsequent runs.

    st.title("Flight Data Search")

    with st.form("search_form"):
        month = st.selectbox("Select Month", df["Month"].unique())
        submit = st.form_submit_button("Search")

    if submit: # Only filters the data frame if the "submit" button is clicked.
        st.write("Flights in", month)
        st.dataframe(df[df["Month"] == month]) # Displays filtered data frame as an interactive table.
    ```
- To-Do List App Example:
    ```sql
    import streamlit as st

    st.title("To-Do List")

    # Initialize state
    if "tasks" not in st.session_state:
        st.session_state.tasks = []

    # Form to add tasks
    with st.form("task_form"):
        task = st.text_input("New Task")
        add = st.form_submit_button("Add Task")

    if add and task:
        st.session_state.tasks.append(task)

    # Display tasks with delete buttons
    for i, t in enumerate(st.session_state.tasks): # Index must start as 0 (default) because it's being used to pop values from the list of tasks.
        col1, col2 = st.columns([4,1]) # Specifies column width for each column

        col1.write(t)

        if col2.button("X", key=f"del_{i}"):
            st.session_state.tasks.pop(i) # Removes the task from the list.
            st.rerun()
    ```

## Dashboards
- Dashboards are visual interfaces used to explore data interactively. They allow you to:
    - Monitor KPIs in real time.
    - Explore trends with filters.
    - Combine multiple data sources into one view.
    - Share insights with non-technical users.
- Streamlit simplifies dashboard creation by:
    - Allowing widgets to be used as filters.
    - Caching data to quickly update a dashboard.
    - Laying out controls to make a dashboard clean and responsive.
    - Integrating with Plotly/Altair/Matplotlib seamlessly.
- Sales Dashboard Example:
    ```sql
    import streamlit as st
    import pandas as pd
    import plotly.express as px

    st.set_page_config(page_title="Sales Dashboard", layout="wide") # Title of the tab/window.

    # Sample dataset (replace with real sales data)
    @st.cache_data
    def load_data():
        url = "https://raw.githubusercontent.com/yannie28/Global-Superstore/master/Global_Superstore(CSV).csv"
        df = pd.read_csv(url)
        
        # Convert to datetime
        df["Order Date"] = pd.to_datetime(df["Order Date"], dayfirst=False)
        
        return df

    df = load_data()

    # Sidebar filters
    st.sidebar.header("Filters")
    year = st.sidebar.selectbox("Select Year", sorted(df['Order Date'].dt.year.unique()))
    region = st.sidebar.multiselect("Select Region", df['Region'].unique(), default=df['Region'].unique()) # Selectd all regions by default.

    # Filter data
    filtered_df = df[(df['Order Date'].dt.year == year) & (df['Region'].isin(region))]

    # KPIs
    total_sales = int(filtered_df["Sales"].sum())
    total_profit = int(filtered_df["Profit"].sum())
    total_customers = filtered_df["Customer ID"].nunique() # Number of unique customers.

    st.title(f"Sales Dashboard - {year}") # Title that is displayed in the webpage.

    kpi1, kpi2, kpi3 = st.columns(3) # 3 columns of equal length.

    kpi1.metric("Total Sales", f"${total_sales:,.2f}") # Separates each number with a comma and limits the precision to two decimal places.
    kpi2.metric("Total Profit", f"${total_profit:,.2f}")
    kpi3.metric("Unique Customers", total_customers)

    st.markdown("---")

    # Charts
    col1, col2 = st.columns([2, 1])

    with col1: # Line graph for cumulative monthly sales.
        sales_trend = (
            filtered_df
                .groupby(filtered_df["Order Date"].dt.to_period("M"))["Sales"]
                .sum()
                .reset_index()
        )

        sales_trend["Order Date"] = sales_trend["Order Date"].astype(str)
        fig = px.line(sales_trend, x="Order Date", y="Sales", title="Monthly Sales Trend")
        st.plotly_chart(fig, use_container_width=True)

    with col2: # Horizontal bar chart for product quantities sold.
        top_products = filtered_df.groupby("Product Name")["Sales"].sum().nlargest(5).reset_index()
        fig = px.bar(top_products, x="Sales", y="Product Name", orientation="h", title="Top 5 Products")
        st.plotly_chart(fig, use_container_width=True)
    ```
    - This interactive dashboard allows users to:
        - Select Year and Region.
        - View KPIs such as total sales, profits, and number of unique customers.
    
## Deployment
- When you develop and test a Streamlit app on your local computer, only you can access it. When you deploy a Streamlit app, others have access to it as well. This is helpful for developing shared dashboards, internal tools, portfolios, and client demos.
- Deployment Options:
    - Streamlit Community Cloud (Free & Easy):
        - No server setup is required.
        1. Push code to github.
        1. Go to Streamlit Cloud.
        1. Connect to GitHub, Select Repo, and Deploy.
        1. Get a shareable, public URL.
    - Other Cloud Providers (More Power/Private App):
        - Heroku – simple, free tier available.
        - Render – easy deployment, free tier.
        - AWS / GCP / Azure – enterprise-grade, but setup-heavy.
        - Docker – package your app and deploy anywhere.
- Preparing For Deployment:
    - Add a `requirements.txt` file so that the server knows which dependencies to install:
        ```python
        streamlit
        pandas
        plotly
        altair
        matplotlib
        seaborn
        ```
    - Make sure to include `.streamlit/config.toml` in the repo if you use custom themes.
- Deploying to Streamlit Cloud Example:
    1. Push code to GitHub (repo must be public for free).
    1. Go to Streamlit Cloud.
    1. Click New App → Connect to GitHub.
    1. Select main branch, file: app.py.
    1. Click Deploy.
    1. Done – your app is live at: https://your-username-your-repo.streamlit.app/


## Authentication
- Streamlit apps do not have any built-in, default authentication methods or user access controls. There are several ways to implement these restrictions so that an app is ready for production use and can handle security, APIs, and performance efficiently.
1. Streamlit Authenticator (Easiest):
    ```python
    import streamlit as st
    import streamlit_authenticator as stauth

    names = ["Alice", "Bob"]
    usernames = ["alice", "bob"]
    passwords = ["123", "456"]

    hashed_pw = stauth.Hasher(passwords).generate()

    authenticator = stauth.Authenticate(
        names, usernames, hashed_pw,
        "app_cookie", "random_signature_key", cookie_expiry_days=30
    )
    name, authentication_status, username = authenticator.login("Login", "main")

    if authentication_status:
        st.success(f"Welcome {name}!")
    elif authentication_status is False:
        st.error("Invalid username or password")
    ```
    - OAuth (Google, GitHub) can be implemented using external libraries (e.g., Auth0 + Streamlit) when enterprise-grade security is required.
    - Installing streamlit-authenticator on your local computer is required.
1. Handling Secrets & Environment Variables:
    - Never use hard-coded API keys in your Streamlit dashboard code. Instead, create a `.streamlit/secrets.toml` file to store these keys:
        ```python
        [api_keys]
        openai = "your_openai_api_key"
        snowflake_user = "user123"
        snowflake_pass = "securepass"
        ```
    - To use these secrets in your code:
        ```python
        import streamlit as st

        api_key = st.secrets["api_keys"]["openai"]
        st.write("Loaded API key securely!")
        ```
1. Connecting to External APIs:
    ```python
    import streamlit as st
    import requests

    st.title("Free Weather App (No API Key)")

    city = st.text_input("Enter city:")

    if city:
        # Add User-Agent header (required by Nominatim)
        geo_url = f"https://nominatim.openstreetmap.org/search?city={city}&format=json"
        headers = {"User-Agent": "streamlit-weather-app"}

        geo_response = requests.get(geo_url, headers=headers).json()

        if geo_response:
            lat = geo_response[0]["lat"]
            lon = geo_response[0]["lon"]

            # Call Open-Meteo API
            weather_url = (
                f"https://api.open-meteo.com/v1/forecast?"
                f"latitude={lat}&longitude={lon}&current_weather=true"
            )

            weather_response = requests.get(weather_url).json()

            if "current_weather" in weather_response:
                current = weather_response["current_weather"]

                st.metric("Temperature (°C)", current["temperature"])
                st.write("Wind Speed:", current["windspeed"], "km/h")
                st.write("Time:", current["time"])

            else:
                st.error("Weather data not available.")
        else:
            st.error("City not found.")
    ```
1. Integrating ML Models:
    ```python
    from textblob import TextBlob
    import streamlit as st

    text = st.text_area("Enter text:")

    if text:
        sentiment = TextBlob(text).sentiment.polarity

        if sentiment > 0:
            st.success("Positive")
        elif sentiment < 0:
            st.error("Negative")
        else:
            st.info("Neutral"
    ```

## Project

[[02_technical_prep/Streamlit Example Project]]
