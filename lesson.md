# Streamlit for Data Science

## **Overview**

- **Total Duration:** 3h 20m to 3h 45m (plus optional bonus section)
- **Context:** Coaching Session / Blended Learning
- **Target Audience:** Data science learners with Python, NumPy, and Pandas knowledge
- **Prerequisites:**
  - Python basics
  - NumPy fundamentals
  - Pandas DataFrames and basic operations
  - Basic command line/terminal usage
  - Conda/Miniconda installed (from Module 1 onboarding)
- **Learning Objectives:**
  - LO1: Understand what Streamlit is and create basic interactive apps
  - LO2: Display data and visualizations using Streamlit components
  - LO3: Implement user inputs and filtering logic
  - LO4: Optimize app performance with caching
  - LO5: Deploy and share Streamlit apps publicly
  - LO6: (Bonus) Add basic authentication to protect apps

---

## **Section 1: Getting Started (15 min)**

### **Introduction**

Streamlit is an open-source Python framework that turns data scripts into shareable web apps in minutes. **No frontend experience required** - if you can write Python, you can build web apps.

> Note: in practice, you would do EDA first (covered in Lessons 1.8 and 1.9) - understanding distributions, spotting outliers, deciding on cleaning steps. A dashboard is what you build after you understand the data, to make findings explorable for others. The assumption here is that the dataset has already gone through EDA and basic cleaning, and this dataset is ready for building a dashboard on top of it.

### **Why Streamlit for Data Science?**

- **Pure Python:** No HTML, CSS, or JavaScript needed
- **Fast prototyping:** Go from script to app in < 10 lines of code
- **Data-centric:** Built-in support for Pandas, NumPy, matplotlib, plotly
- **Interactive widgets:** Sliders, buttons, file uploads with minimal code
- **Free deployment:** Streamlit Cloud for public/private sharing

### **Common Use Cases**

- Data exploration dashboards
- Machine learning model demos
- Interactive data visualizations
- Internal tools for data teams
- Portfolio projects

### **Installation & Environment Setup**

We'll use **conda** with an `environment.yml` file — the same workflow you've been using throughout the course. Starting from the yml file means you always have a clean, reproducible record of your project's dependencies.

**Step 1: Create a project folder**

```bash
mkdir my-streamlit-app
cd my-streamlit-app
```

**Step 2: Create `environment.yml`**

Create the file `environment.yml` in your project folder:

```yaml
name: streamlit_app
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.11.9
  - pandas=2.2.2
  - pip=24.0
  - pip:
      - streamlit==1.36.0
      - plotly==5.22.0
```

Streamlit and Plotly are not readily available on conda channels, so they go under the `pip:` section. Everything else is installed via conda.

**Step 3: Create the environment**

```bash
conda env create -f environment.yml
```

**Step 4: Activate the environment**

```bash
conda activate streamlit_app
```

You should see `(streamlit_app)` in your terminal prompt.

**Step 5: Verify installation**

```bash
streamlit --version
```

**Pro tip:** Verify you're in the right environment:

```bash
conda env list          # The active environment is marked with *
which python            # Should show a path inside your conda envs
```

**To deactivate when done:**

```bash
conda deactivate
```

---

### **Your First App**

**Make sure your conda environment is activated!** You should see `(streamlit_app)` in your prompt.

Create a file `app.py`:

```python
import streamlit as st

st.title("Hello, Streamlit!")
st.write("This is my first Streamlit app.")
```

**Run it:**

```bash
streamlit run app.py
```

Your browser will open automatically at `http://localhost:8501`

`st.title()` displays a large heading. `st.write()` is a general-purpose function that can display text, dataframes, charts, and more. We'll explore more display and input components next.

**Troubleshooting:**

- If `streamlit: command not found` - make sure your conda environment is activated (`conda activate streamlit_app`)
- If import errors - check that you created the environment from the `environment.yml` file

---

## **Section 2: Code-Along Start - Display Components (25 min)**

This is a full code-along lesson. Everyone builds one usable dashboard together using `./data/resale_data.csv`.

### **2.1 What each display function does**

- `st.title("...")`: page title (largest text), usually once at the top.
- `st.header("...")`: major section heading.
- `st.subheader("...")`: subsection heading inside a section.
- `st.write(...)`: flexible output for text, numbers, DataFrames, charts.
- `st.caption("...")`: helper text or notes in smaller font.
- `st.markdown("...")`: rich text formatting (bold, lists, links, code).
- `st.metric(label, value, delta)`: KPI cards for dashboard summaries.
- `st.dataframe(df)`: interactive table (scroll/sort/filter in UI).
- `st.table(df)`: static table, best for small outputs.

### **2.2 Start `app.py` with a visible page structure**

```python
import streamlit as st

# Sets the page configuration
# You can set the page title and layout here
st.set_page_config(page_title="HDB Resale Dashboard", layout="wide")

st.title("Singapore HDB Resale Dashboard")
st.caption("Code-along: building a usable dashboard from real resale transactions.")

st.header("Dashboard Overview")
st.subheader("What this app will show")
st.markdown("""
- Transaction volume after filtering
- Average resale price
- Median floor area
- Town and flat type trends
""")
```

- Note: `st.set_page_config` must be called before any other Streamlit commands. It sets the page title (shown in browser tab) and layout (wide vs centered).

---

## **Section 3: Load Real Data and Show First Output (25 min)**

We now connect the page to the actual data file: `./data/resale_data.csv`.

### **3.1 Load directly first (no caching yet)**

```python
import streamlit as st
import pandas as pd

DATA_PATH = "./data/resale_data.csv"

df = pd.read_csv(DATA_PATH)
# Lesson assumption:
# this dataset has already gone through EDA and basic cleaning.
# Here we focus on dashboard building, not data cleaning.
# We still set the datetime dtype explicitly for reliable filtering and charting.
df["month"] = pd.to_datetime(df["month"])
```

### **3.2 First data display**

```python
st.write(f"Rows loaded: {len(df):,} | Columns: {len(df.columns)}")
st.dataframe(df.head(20), width="stretch")
```

What you should observe:

- At this stage, data is loaded directly (without caching) so you can later compare rerun behavior.
- This is your initial table preview before adding filters in Section 4.

---

## **Section 4: Add Sidebar Filters and KPIs (30 min)**

### **4.1 Sidebar controls**

Streamlit has a built-in sidebar for inputs. Anything inside `st.sidebar` appears there. Typically, filters go in the sidebar.

Here we will add the following filters:

- Town (multi-select) -> use `st.sidebar.multiselect`
- Flat Type (multi-select) -> use `st.sidebar.multiselect`
- Resale Price Range (slider) -> use `st.sidebar.slider`
- Month Range (date input) -> use `st.sidebar.date_input`

Steps to set up the sidebar filters

1. Get unique values for towns and flat types

To get the unique values for towns and flat types from the DataFrame, you can use the `unique()` method provided by pandas. Here's how you can do it:

```python
unique_towns = sorted(df["town"].dropna().unique())
unique_flat_types = sorted(df["flat_type"].dropna().unique())
```

We use `dropna()` to ensure that any missing values are excluded from the unique lists. The `sorted()` function is used to sort the unique values alphabetically for better user experience in the multi-select widgets.

2. Create multi-selects for towns and flat types

The `multiselect` widget allows users to select multiple options from a list. It accepts parameters such as the label, options, and default selected values.

Here's how you can create multi-select widgets for towns and flat types in the sidebar:

```python
selected_towns = st.sidebar.multiselect("Town", unique_towns, default=[])
selected_flat_types = st.sidebar.multiselect("Flat Type", unique_flat_types, default=[])
```

3. Create slider for resale price range

First we get the min and max resale prices from the DataFrame:

```python
min_price = int(df["resale_price"].min())
max_price = int(df["resale_price"].max())
```

Then we create the slider widget in the sidebar:

```python
price_range = st.sidebar.slider(
    "Resale Price Range",
    min_value=min_price,
    max_value=max_price,
    value=(min_price, max_price),
    step=10000,
)
```

4. Create date input for month range

First, get the minimum and maximum dates from the "month" column:

```python
date_min = df["month"].min().date()
date_max = df["month"].max().date()
```

Then create the date input widget in the sidebar:

```python
date_range = st.sidebar.date_input("Month Range", value=(date_min, date_max))
```

The code for the entire sidebar section looks like this:

```python
st.sidebar.header("Filters")

unique_towns = sorted(df["town"].dropna().unique())
unique_flat_types = sorted(df["flat_type"].dropna().unique())

selected_towns = st.sidebar.multiselect("Town", unique_towns, default=[])
selected_flat_types = st.sidebar.multiselect("Flat Type", unique_flat_types, default=[])

min_price = int(df["resale_price"].min())
max_price = int(df["resale_price"].max())
price_range = st.sidebar.slider(
    "Resale Price Range",
    min_value=min_price,
    max_value=max_price,
    value=(min_price, max_price),
    step=10000,
)

date_min = df["month"].min().date()
date_max = df["month"].max().date()
date_range = st.sidebar.date_input("Month Range", value=(date_min, date_max))
```

### **4.2 Apply filters**

To handle filtering based on the selected sidebar inputs, we will create a new DataFrame called `filtered_df` that starts as a copy of the original DataFrame `df`. We will then apply each filter conditionally based on whether the user has made any selections.

```python
filtered_df = df.copy()

if selected_towns:
    filtered_df = filtered_df[filtered_df["town"].isin(selected_towns)]

if selected_flat_types:
    filtered_df = filtered_df[filtered_df["flat_type"].isin(selected_flat_types)]

filtered_df = filtered_df[
    filtered_df["resale_price"].between(price_range[0], price_range[1])
]

if len(date_range) == 2:
    start_date, end_date = date_range
    filtered_df = filtered_df[filtered_df["month"].between(
        pd.to_datetime(start_date), pd.to_datetime(end_date)
    )]
```

### **4.3 Update the table to use `filtered_df`**

Now update the table code from Section 3.2 so it uses the filtered DataFrame.
Important: move this table block to a position **after** `filtered_df` is created in Section 4.2.
If you place it earlier, `filtered_df` is not defined yet and the app will error.

```python
st.header("Filtered Results")
st.write(f"Matching rows: {len(filtered_df):,} | Columns: {len(filtered_df.columns)}")
st.dataframe(filtered_df.head(20), width="stretch")
```

### **4.4 KPI row**

The KPI row shows key metrics at a glance. We will use `st.metric` inside `st.columns` to create a row of four metrics: total transactions, average price, median price, and median floor area.

```python
st.header("Key Metrics")
# Create four columns for the metrics and unpack them
# We can then use each column to place a metric
col1, col2, col3, col4 = st.columns(4)

# Each col provides a .metric() method that takes a label and a value
col1.metric("Transactions", f"{len(filtered_df):,}")
col2.metric("Average Price", f"${filtered_df['resale_price'].mean():,.0f}")
col3.metric("Median Price", f"${filtered_df['resale_price'].median():,.0f}")
col4.metric("Median Floor Area", f"{filtered_df['floor_area_sqm'].median():.1f} sqm")
```

---

## **Section 5: Add Charts and Data Views (30 min)**

We will add three main visualizations to analyze trends in the filtered data here:

- Average resale price by town (bar chart)
- Transactions by flat type (bar chart)
- Monthly median resale price trend (line chart)

For complex filtering and grouping operations, it is common to prototype first in Jupyter notebooks. Notebooks are useful for step-by-step exploration, checking intermediate outputs, and testing chart ideas quickly, including interactive Plotly charts. Once the logic is stable, move it into `.py` functions and call those functions from your Streamlit app.

### **5.1 Main visuals**

**Two ways to use `st.columns`:**

Earlier in section 4.4, we used the **direct method** to add metrics to columns:

```python
col1, col2, col3, col4 = st.columns(4)
col1.metric("Transactions", f"{len(filtered_df):,}")
col2.metric("Average Price", f"${filtered_df['resale_price'].mean():,.0f}")
```

This works well when you're adding **one element per column**. However, when you need to add **multiple elements** to the same column (like a subheader, chart, and caption), using the **context manager** approach is cleaner:

```python
with col_left:
    st.subheader("Title")
    st.plotly_chart(fig)
    st.caption("Note")
```

The context manager tells Streamlit "put everything in this block into this column". This allows you to use regular `st.` functions instead of repeating `col_left.` before each element.

Both methods work the same way. Use the direct method for single elements, and the context manager for multiple elements.

Now, in the main visuals section, we will use the context manager approach for better readability and flexibility to display:

- Average Resale Price by Town (with a subheader and caption)
- Transactions by Flat Type (with a subheader and caption)

```python
import plotly.express as px

st.header("Visual Analysis")

col_left, col_right = st.columns(2)

# Tells Streamlit to put the following content in the left column
with col_left:
    st.subheader("Average Resale Price by Town")
    avg_price_by_town = (
        filtered_df.groupby("town", as_index=False)["resale_price"]
        .mean()
        .sort_values("resale_price", ascending=False)
        .head(10) # Top 10 towns only for clarity
    )
    # Create a Plotly bar chart with towns on x-axis and average resale price on y-axis
    fig_town = px.bar(avg_price_by_town, x="town", y="resale_price")
    # Display the Plotly chart in Streamlit
    st.plotly_chart(fig_town, width="stretch")

# Tells Streamlit to put the following content in the right column
with col_right:
    st.subheader("Transactions by Flat Type")
    tx_by_flat = (
        filtered_df.groupby("flat_type", as_index=False)
        .size()
        .rename(columns={"size": "transactions"})
        .sort_values("transactions", ascending=False)
    )
    # Create a Plotly bar chart with flat types on x-axis and transaction counts on y-axis
    fig_flat = px.bar(tx_by_flat, x="flat_type", y="transactions")
    # Display the Plotly chart in Streamlit
    st.plotly_chart(fig_flat, width="stretch")
```

For Average Resale Price by Town:

1. Group `filtered_df` by "town" and calculate the mean resale price.
2. Sort the results in descending order of resale price.
3. Create a bar chart using Plotly Express and display it.

For Transactions by Flat Type:

1. Group `filtered_df` by "flat_type" and count the number of transactions.
2. Sort the results in descending order of transaction counts.
3. Create a bar chart using Plotly Express and display it.

### **5.2 Monthly trend**

For Monthly Median Resale Price trend, we will create a line chart showing how the median resale price changes over time.

```python
st.subheader("Monthly Median Resale Price")
trend = (
    filtered_df.groupby("month", as_index=False)["resale_price"]
    .median()
    .sort_values("month")
)
# Create a Plotly line chart with month on x-axis and median resale price on y-axis
fig_trend = px.line(trend, x="month", y="resale_price", markers=True)
# Display the Plotly chart in Streamlit
st.plotly_chart(fig_trend, width="stretch")
```

### **5.3 Optional: Detailed table + download**

This is an optional section for users who want to inspect more rows and export the filtered results.
Unlike the main table in Section 4.3 (always visible for quick feedback), this one is collapsed in an expander to keep the dashboard clean. There is also a download button to export the filtered data as CSV.

```python
with st.expander("View Filtered Transactions"):
    st.dataframe(filtered_df, width="stretch", height=350)
    csv = filtered_df.to_csv(index=False).encode("utf-8")
    st.download_button(
        "Download filtered CSV",
        data=csv,
        file_name="filtered_resale_data.csv",
        mime="text/csv",
    )
```

---

## **Section 6: The Rerun Model in Streamlit (25 min)**

In this section, you will observe Streamlit's rerun behavior using your current dashboard. A rerun means the entire script executes again from top to bottom. This is a core concept in Streamlit that affects how you structure your code and manage state.

### **6.1 Demonstrate reruns**

```python
from datetime import datetime
print(f"🟢 Rerun at: {datetime.now()}")
```

Change any sidebar filter and see that the script reruns top-to-bottom.

Now change any sidebar filter and watch your terminal. You should see a new timestamp each time, which means the script ran again from top to bottom.

### **6.2 What triggers reruns and what does not**

Rerun triggers:

- Changing widget values (`multiselect`, `slider`, `date_input`, `button`)

Frontend-only actions do not trigger reruns because they are handled entirely in the browser without needing to execute Python code again:

- Sorting inside `st.dataframe`
- Opening/closing `st.expander`

### **6.3 Add caching and compare performance**

Because this app reads a large CSV, without caching the file is re-read each rerun. Now refactor to a cached loader:

```python
@st.cache_data
def load_data(path):
    df = pd.read_csv(path)
    df["month"] = pd.to_datetime(df["month"])
    return df

df = load_data(DATA_PATH)
```

After this change, try the same filters again and compare responsiveness.

---

## **Section 7: Complete Code-Along Version (20 min)**

### **7.1 Final Dashboard Checklist**

By this point, learners should have one usable dashboard with:

- Real data loading from `./data/resale_data.csv`
- Sidebar filters
- KPI metrics
- Plotly charts
- Detailed table + download
- Rerun understanding

### **7.2 Run the Final App**

Use this structure as your final `app.py` and run:

```bash
streamlit run app.py
```

---

## **Section 8: Deployment & Sharing (20 min)**

### **8.1 Introduction to Streamlit Cloud**

Streamlit Cloud is a **free** platform to deploy, manage, and share your Streamlit apps.

**Benefits:**

- Free for public apps
- Direct GitHub integration
- Automatic updates on push
- HTTPS included
- Built-in authentication for private apps

### **8.2 Preparing for Deployment**

Streamlit Community Cloud supports both `environment.yml` and `requirements.txt`, but `requirements.txt` is strongly recommended for deployment because the free tier has a 1GB memory limit, and conda's environment resolution often runs out of memory and fails to deploy.

Because this app reads a local CSV (`./data/resale_data.csv`), that file must also be pushed to GitHub.  
If the data file is missing from the repo, deployment succeeds but the app will fail at runtime when it tries to read the file.

#### **Creating requirements.txt**

The simplest approach is to manually write it with only what your app uses:

```txt
streamlit==1.36.0
pandas==2.2.2
plotly==5.22.0
```

**Why manually?**

- Cleaner, no unnecessary dependencies
- Faster deployment
- You know exactly what your app needs

> **Tip:** You can also auto-generate it from your conda environment with `pip list --format=freeze > requirements.txt`, but you'll want to clean it up to include only the packages your app actually imports.

#### **Your Project Structure**

```
my-streamlit-app/
├── app.py              # Your Streamlit app
├── data/
│   └── resale_data.csv # Data source used by app.py
├── environment.yml     # Local development (conda)
├── requirements.txt    # Deployment (Streamlit Cloud)
└── .gitignore         # Ignore unnecessary files
```

You'll have **two** dependency files serving different purposes: `environment.yml` is your source of truth for local development, and `requirements.txt` is what Streamlit Cloud uses to install packages. Keep them in sync — whenever you add a package to `environment.yml`, add it to `requirements.txt` too.

For this lesson dataset size (~21MB), committing `data/resale_data.csv` is usually fine for Streamlit Community Cloud.  
If your dataset becomes much larger, consider external storage (database, object storage, or API) instead of keeping big files in Git.

#### **Important: .gitignore**

Use the official GitHub Python template as your base:

- `https://github.com/github/gitignore/blob/main/Python.gitignore`

Then add Streamlit-specific entries if needed, especially:

```gitignore
.streamlit/secrets.toml
```

Keep your lesson data file tracked in Git (`data/resale_data.csv`) since the deployed app reads it directly.

### **8.3 Deployment Steps**

#### **Step 1: Prepare your project**

Ensure you have:

- `app.py` (your Streamlit app)
- `data/resale_data.csv` (data source used by the app)
- `requirements.txt` (dependencies)
- `.gitignore`

```bash
# Final check
ls -la
ls -la data
# You should see: app.py, requirements.txt, .gitignore, and data/resale_data.csv
```

#### **Step 2: Push to GitHub**

```bash
# Initialize Git (if not already done)
git init

# Add files
git add .

# Commit
git commit -m "Initial commit"

# Create a new repo on GitHub, then:
git branch -M main
git remote add origin https://github.com/username/repo.git
git push -u origin main
```

#### **Step 3: Deploy on Streamlit Cloud**

1. Go to [share.streamlit.io](https://share.streamlit.io)
2. Sign in with GitHub
3. Click "New app"
4. Select your repository, branch (`main`), and main file (`app.py`)
5. Click "Deploy"

**Your app will be live at:** `https://username-reponame-main.streamlit.app`

### **8.4 App Settings**

In Streamlit Cloud dashboard:

- **Secrets:** Store API keys and passwords securely
- **Resources:** Upgrade for more memory/CPU
- **Analytics:** View usage statistics
- **Privacy:** Make app private (requires authentication)

### **8.5 Optional: Managing Secrets**

This is optional for this lesson because the app uses a local CSV file.  
Use this section only when your app connects to external services (for example, Supabase/PostgreSQL or third-party APIs).

For sensitive data like API keys:

**In Streamlit Cloud:**

1. App settings -> Secrets
2. Add in TOML format:

```toml
[database]
host = "db.example.com"
user = "myuser"
password = "mypassword"

[api]
key = "your-api-key"
```

**In your code:**

```python
import streamlit as st

# Access secrets
db_host = st.secrets["database"]["host"]
api_key = st.secrets["api"]["key"]
```

**For local development**, create `.streamlit/secrets.toml` (already in `.gitignore`):

```toml
[database]
host = "localhost"
user = "dev"
password = "dev123"
```

### **8.6 Updating Your App**

Simply push changes to GitHub:

```bash
git add .
git commit -m "Update app"
git push
```

Streamlit Cloud auto-detects and redeploys!

---

## **Section 9: Bonus - Adding Authentication (Reference)**

### **9.1 Streamlit Cloud Built-in Auth**

The easiest way to protect your app:

1. In Streamlit Cloud dashboard
2. Settings -> Sharing
3. Toggle "Make app private"
4. Invite viewers by email

**Viewers sign in with:** Google account, GitHub account, or email magic link.

This is sufficient for most use cases (internal tools, class projects, portfolio demos).

### **9.2 Simple Password Protection**

For a quick password gate without any extra libraries:

```python
import streamlit as st

def check_password():
    """Returns True if user entered correct password"""

    def password_entered():
        if st.session_state["password"] == st.secrets["password"]:
            st.session_state["password_correct"] = True
            del st.session_state["password"]  # Don't store password
        else:
            st.session_state["password_correct"] = False

    if "password_correct" not in st.session_state:
        st.text_input(
            "Password",
            type="password",
            on_change=password_entered,
            key="password"
        )
        return False
    elif not st.session_state["password_correct"]:
        st.text_input(
            "Password",
            type="password",
            on_change=password_entered,
            key="password"
        )
        st.error("Password incorrect")
        return False
    else:
        return True

if check_password():
    st.write("Your protected app content here")
```

Store the password in secrets:

```toml
# .streamlit/secrets.toml
password = "your-secure-password"
```

### **9.3 For More Advanced Auth**

If you need usernames, roles, or more control, look into the `streamlit-authenticator` library. See the [official docs](https://github.com/mkhorasani/Streamlit-Authenticator) for setup instructions.

---

## **Section 10: Next Steps (10 min)**

### **10.1 When to Use Streamlit**

**Good for:**

- Quick prototypes and demos
- Internal data tools
- Machine learning demos
- Portfolio projects
- Data exploration dashboards

**Not ideal for:**

- Apps requiring extensive custom frontend behavior (React + Python backend is a common upgrade path)
- Real-time collaborative editing
- Apps with complex authentication needs

Upgrade path: start with Streamlit for fast iteration, then move to a custom frontend stack (for example React + FastAPI) when product requirements outgrow Streamlit's UI model.

### **10.2 Alternatives to Consider**

- **Dash (Plotly):** More customization, steeper learning curve
- **Gradio:** Excellent for ML model demos
- **Panel:** Similar to Streamlit, more flexibility
- **Voila:** Turn Jupyter notebooks into apps

### **10.3 Resources for Further Learning**

**Official:**

- [Streamlit Documentation](https://docs.streamlit.io)
- [Streamlit Gallery](https://streamlit.io/gallery) - Example apps
- [Streamlit Forum](https://discuss.streamlit.io) - Community support

**Component Libraries:**

- [streamlit-extras](https://github.com/arnaudmiribel/streamlit-extras) - Additional components
- [streamlit-aggrid](https://github.com/PablocFonseca/streamlit-aggrid) - Advanced datagrids

### **10.4 Other Project Ideas**

1. **Stock Market Dashboard** - Real-time stock data with technical indicators
1. **Personal Finance Tracker** - Expense analysis and budgeting
1. **Text Analysis Tool** - Sentiment analysis, word clouds
1. **Portfolio Website** - Showcase your data science projects

---

## **Summary**

You've learned:

- What Streamlit is and how to create basic apps
- Core components: text, data, charts, and widgets
- Optimizing with caching and understanding reruns
- Working with real project data from `./data/resale_data.csv`
- Creating professional layouts with columns, tabs, and sidebars
- Building a complete data exploration dashboard
- Deploying to Streamlit Cloud
- Adding authentication to protect your apps
- Choosing when Streamlit is the right tool and where to learn more

**Next Steps:**

1. Build your own dashboard using your data
2. Deploy it to Streamlit Cloud
3. Share with the class!
4. Explore advanced components and libraries

---

## **Quick Reference Card**

```python
# Essential imports
import streamlit as st
import pandas as pd

# Display
st.title("Title")
st.header("Header")
st.write("Anything")
st.dataframe(df)

# Input
st.button("Click")
st.slider("Select", 0, 100)
st.selectbox("Choose", ["A", "B"])
st.multiselect("Choose many", ["A", "B", "C"])

# Layout
col1, col2 = st.columns(2)
st.sidebar.title("Sidebar")
tab1, tab2 = st.tabs(["Tab1", "Tab2"])

# Caching
@st.cache_data
def load_data():
    return pd.read_csv("./data/resale_data.csv")

# Run
# streamlit run app.py
```
