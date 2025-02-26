# Amazon Product Co-purchasing Network Analysis with ArangoDB, LangChain, and Gradio/Streamlit

This Jupyter Notebook (`arangoo.ipynb`) demonstrates an end-to-end workflow for analyzing the Amazon product co-purchasing network, leveraging ArangoDB for graph storage and retrieval, LangChain for intelligent querying with a Cohere large language model (LLM), and Gradio (or Streamlit, with alternative cells provided) for a user-friendly web interface.  The project downloads, parses, analyzes, and provides interactive exploration of Amazon's product relationships.

**Key Features and Functionality:**

1.  **Dependency Management:**
    *   The notebook begins by installing necessary Python packages, including:
        *   `python-arango`: For interacting with ArangoDB.
        *   `networkx`: For graph manipulation and analysis.
        *   `pandas`: For data handling and manipulation.
        *   `gdown`: (Initially used, later replaced with `requests`) For downloading data from URLs.
        *   `requests`: For making HTTP requests to download datasets.
        *   `tqdm`: For displaying progress bars.
        *   `langchain`: For building LLM-powered applications.
        *   `langchain-openai` and `langchain-community`: Specific langchain components, with transition instructions.
        *   `openai` or `langchain_cohere`: For accessing language models (either OpenAI or Cohere, with a focus on Cohere).
        *   `gradio` or `streamlit`: For creating the interactive web application.
        *   `matplotlib`: For graph visualization.
        *   `python-dotenv`: (Optional) for loading environment variables.
        *    `community` (python-louvain): For community detection (with fallback to connected components if installation fails).
        *   `pyArango`

    *   The installation process is made robust with error handling and checks for existing installations.

2.  **Data Acquisition and Preprocessing:**
    *   **Dataset Download:** The notebook downloads Amazon SNAP datasets directly from the Stanford SNAP website using `requests`. The datasets include:
        *   `metadata.json.gz`: Contains product metadata (ASIN, title, description, etc.).
        *   `amazon0302.txt.gz`, `amazon0312.txt.gz`, `amazon0505.txt.gz`, `amazon0601.txt.gz`:  Co-purchase network data for different time periods.
    *   **Data Parsing:** Custom functions (`parse_amazon_metadata` and `parse_amazon_copurchase`) are defined to handle:
        *   Reading data from gzipped files (`.gz`).
        *   Parsing JSON metadata, handling potential multi-line JSON objects and JSONDecodeErrors.
        *   Parsing the co-purchase network data, extracting source and target product IDs.
        *   Converting the parsed data into pandas DataFrames.
        *   Saving the parsed data to CSV files for faster reloading.
    *   **Error Handling:** Includes checks for existing CSV files, handles potential `JSONDecodeError` during metadata parsing, and provides informative error messages if data loading or parsing fails.

3.  **Graph Construction and Analysis:**
    *   **Graph Creation:** A directed graph (`networkx.DiGraph`) is constructed from the co-purchase data, representing products as nodes and co-purchases as edges.
    *   **Metadata Integration:** If available, product metadata (from `metadata.json.gz`) is added as node attributes to the graph.  The code handles cases where the metadata DataFrame might be empty or the ASIN column is missing.
    *   **Graph Analysis:** The `analyze_graph` function performs:
        *   Basic graph statistics: Number of nodes, number of edges, average degree, maximum degree.
        *   Identification of top nodes (products) by degree (number of connections).
        *   Connected component analysis: Finds the largest weakly connected component.
        *   Subgraph sampling: Creates a smaller subgraph for visualization and detailed analysis.
    *   **Community Detection:** The `detect_communities` function attempts to use the Louvain algorithm (from the `community` package, also known as `python-louvain`) for community detection.  If `python-louvain` is not installed, it falls back to using NetworkX's connected components algorithm.  It also handles large graphs by sampling nodes for community detection.

4.  **ArangoDB Integration:**
    *   **Connection Setup:** The `setup_arangodb` function establishes a connection to an ArangoDB instance, handling retries and potential connection errors. It also creates the specified database and graph if they don't exist. It uses root credentials, and the database name, graph name, host, username, and password are hardcoded.
    *   **Graph Persistence:** The `persist_amazon_graph` function stores the NetworkX graph in ArangoDB.  It creates vertex and edge collections and inserts nodes and edges in batches for efficiency. It uses overwrite mode 'update' to avoid duplicates.
    *   **Graph Loading:** The `load_graph_from_arangodb` function retrieves a graph from ArangoDB and reconstructs it as a NetworkX `DiGraph`.
    * **Data Loading Strategy**: The code prioritizes loading the graph directly from ArangoDB if it exists, falling back to downloading and processing from the original data files only if the graph is not found in the database.

5.  **LLM-Powered Insights (LangChain + Cohere):**
    *   **LangChain Setup:** The `setup_langchain_cohere` function configures LangChain to use the Cohere LLM (`ChatCohere` class). A prompt template is defined to provide the LLM with graph analysis results and a user query.  The API key is expected to be in the `COHERE_API_KEY` environment variable.
    *   **Agentic Querying:** The `agentic_query` function takes a user's question, the graph analysis results, and community detection results, and uses the configured LangChain LLMChain to generate an answer.  This allows users to ask natural language questions about the graph and receive insights.

6.  **Gradio/Streamlit Web Interface:**
    *   **User Interface:** The notebook includes cells to build an interactive web application using either Gradio or Streamlit (with Streamlit as the preferred/more developed option).
        *   **Input:**
            *   A slider to control the maximum number of nodes to display in the visualization (for performance).
            *   A dropdown to select the Amazon dataset to analyze.
            *   A text input for the user to enter a query for the LLM.
            *   A checkbox to control whether to persist the graph to ArangoDB.
        *   **Output:**
            *   A text box displaying a summary of the graph statistics.
            *   An image showing a visualization of a sample of the graph.
            *   A text box showing the LLM's response to the user's query.
            *   A text box showing the status of the processing.
    *   **Interaction:** The user interface allows users to:
        *   Select a dataset.
        *   Adjust the maximum number of nodes for visualization.
        *   Ask natural language questions about the graph.
        *   See the graph analysis results and LLM responses.
        *   Optionally persist the graph to ArangoDB.
    *   **Streamlit Specifics:** The Streamlit implementation uses `st.columns` for layout, `st.spinner` for loading indicators, and `st.image` to display the graph visualization. Session state (`st.session_state`) is used to store results and maintain interactivity.

7.  **Example Queries:**  The notebook demonstrates example LLM queries, such as:
    *   "What is the most influential product?"
    *   "How many communities are there?"
    *   "What is the structure of the network?"
    *   "Give me a product recommendation."

**Execution Flow:**

1.  **Dependencies:** Install necessary packages.
2.  **Configuration:** Load API keys and ArangoDB connection details.
3.  **Data Loading:**
    *   Attempt to load the graph from ArangoDB.
    *   If not found in ArangoDB, download and parse the Amazon SNAP datasets.
    *   Create a NetworkX graph from the co-purchase data.
4.  **Graph Analysis:**
    *   Calculate graph statistics.
    *   Perform community detection.
5.  **ArangoDB Persistence (optional):** Store the graph in ArangoDB if enabled.
6.  **LangChain Setup:** Configure LangChain with the Cohere LLM.
7.  **Gradio/Streamlit Interface:** Launch the web application for interactive analysis.  The user interacts with the interface, triggering the `process_data` function, which handles the data loading, analysis, and LLM querying.

**Improvements and Considerations:**

*   **Error Handling:** The notebook has good error handling for common issues (missing API keys, file download errors, JSON parsing errors, ArangoDB connection problems).
*   **Modularity:** The code is well-structured with functions for each major step, improving readability and maintainability.
*   **Scalability:** The use of batch processing for ArangoDB insertion and sampling for visualization and community detection helps handle large datasets.
*   **Clarity:** The comments and print statements provide good explanations of the code's functionality.
*   **ArangoDB Credentials:** The notebook uses hardcoded credentials for ArangoDB. In a production environment, these should be stored securely (e.g., using environment variables or a secrets management system).
* **Streamlit vs. Gradio**: the Streamlit cells are much more up-to-date, well-formatted and use modern best practices.
* **Colab Compatibility**: The notebook includes workarounds specifically for running in Google Colab, like the handling of Streamlit session state, and using the `!{sys.executable} -m pip install ...` to correctly install packages.

This notebook provides a solid foundation for analyzing the Amazon product co-purchasing network.  It combines data processing, graph analysis, LLM querying, and a user-friendly interface, demonstrating a practical application of graph databases and large language models.

# Amazon Product Co-purchasing Network Analysis: Code Walkthrough

This document provides a detailed, step-by-step walkthrough of the code in `arangoo.ipynb`. It explains the purpose and functionality of each code cell, highlighting key features and improvements. We'll focus on the Streamlit implementation, which is the more complete and recommended version for interactive exploration.

## Notebook Structure

The notebook is organized into the following main sections:

1. **Setup and Dependencies:** Installing necessary libraries, handling imports, and setting up environment variables.
2. **Data Acquisition:** Downloading, parsing, and preparing the Amazon SNAP datasets.
3. **Graph Construction and Analysis:** Creating a NetworkX graph, adding metadata, and performing basic graph analysis (degree, connected components).
4. **Community Detection:** Identifying communities within the graph.
5. **ArangoDB Integration:** Connecting to ArangoDB, storing the graph, and loading it back.
6. **LLM Interaction (LangChain):** Setting up LangChain with Cohere and querying the graph with natural language.
7. **Streamlit Web Interface:** Building an interactive web application for exploring the graph and results.

## Detailed Code Walkthrough

### Cell 1: Initial Package Installations (Corrected)

```python
# Install the required packages correctly
import sys
!{sys.executable} -m pip install python-arango networkx pandas gdown
!{sys.executable} -m pip install langchain langchain-openai langchain-community openai
```

* **Purpose:** Install essential Python packages reliably.
* **Explanation:**
  * `import sys`: Imports the `sys` module for access to the Python executable path.
  * `!{sys.executable} -m pip install ...`: This is the *correct* way to install within Jupyter, especially in Colab.
    * `!`: Executes a shell command.
    * `{sys.executable}`: Gets the path to the *current* Python interpreter. This avoids environment conflicts.
    * `-m pip`: Uses the `pip` module directly (more reliable than `pip` alone).
  * Installs two sets of packages:
    1. Core: `python-arango`, `networkx`, `pandas`, `gdown` (later, `gdown` is superseded by `requests`).
    2. LangChain: `langchain`, `langchain-openai`, `langchain-community`, `openai`. These are for LLM integration.

* **Key Improvement:** Using `!{sys.executable} -m pip install` ensures correct installation within the notebook's environment.

### Cell 2: Library Imports
```python
import sys
import subprocess
import importlib

import networkx as nx
from arango import ArangoClient
# Update these langchain imports
from langchain_openai import OpenAI  # Or use this
# Alternatively, you might need to use:
# from langchain_community.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain
import pandas as pd
import json
import gdown
```
- **Purpose:** Import necessary libraries for data manipulation, graph analysis, LLM interaction, and database connection.

### Cell 3: Install and Import Function

```python
# ... (Code for install_and_import, and initial package installations) ...
```
* **Purpose:** *Deprecated* in favor of more robust solution in later cells, but contained an install and import function.

### Cell 4: Dependency Installation and Imports (Streamlit Version - Centralized)

```python
# ... (Code for install_and_import function, package lists, installation logic, and imports) ...
```

* **Purpose:** This is the *main* cell for managing dependencies. It combines installation and import in a robust and organized way.
* **Explanation:**
  * **`install_and_import(package)` Function:**
    * Tries to import a package. If it fails, it attempts to install the package using `subprocess.check_call` (a safer way to run shell commands than `os.system`).
    * Includes error handling for both import and installation failures.
  * **Package Lists:**
    * `required_packages`: Non-LangChain packages.
    * `langchain_packages`: LangChain-related packages.
    * `all_packages`: Combined list.
  * **Installation Logic:**
    * Checks if all packages are *already* installed.
    * If *any* are missing, gathers them into `missing_packages`.
    * Installs *all* missing packages in a *single* `pip install` command (efficient).
    * *Double-checks* that each installed package can now be imported.
    * Sets `installed_all_packages` to `False` if any installation or import fails.
  * **Imports:** Imports all necessary libraries *after* the installation attempts.

* **Key Improvements:**
  * **Combined Install and Import:** `install_and_import` streamlines the process.
  * **Batch Installation:** More efficient than installing one-by-one.
  * **Double-Check Imports:** Ensures packages are importable after installation.
  * **`installed_all_packages` Flag:** A clear indicator of success/failure.
  * **Clearer Output:** Informative messages about installation status.

### Cell 5: Data Download and Parsing (Revised)

```python
# ... (Code for downloading and parsing Amazon SNAP datasets) ...
```

* **Purpose:** Downloads, parses, and saves the Amazon SNAP datasets.
* **Explanation:**
  * **`download_file(url, filename)`:**
    * Downloads from `url` to `filename` using `requests.get(..., stream=True)` for efficiency.
    * Uses `response.raise_for_status()` to check for HTTP errors.
    * Displays a `tqdm` progress bar.
    * Handles `requests.exceptions.RequestException`.
  * **`parse_amazon_metadata(gz_file)`:**
    * Parses `metadata.json.gz`.
    * Uses `gzip.open(..., 'rt', encoding='utf-8')`.
    * Reads and parses JSON line by line, accumulating lines to handle multiline objects, includes error handling with `json.JSONDecodeError`.
    * Creates a pandas DataFrame, handles missing 'ASIN' column.
    * Saves to CSV.
  * **`parse_amazon_copurchase(gz_file, max_edges=None)`:**
    * Parses co-purchase files (e.g., `amazon0302.txt.gz`).
    * Uses `gzip.open(..., 'rt', encoding='latin1')`.
    * Reads lines, skips comments (`#`), splits into source/target.
    * Optional `max_edges` parameter for limiting data.
    * Creates a pandas DataFrame, saves to CSV.
  * **`amazon_datasets` Dictionary:** Defines dataset URLs and types.
  * **Data Directory:** Creates `amazon_data` to store files.
  * **Download/Processing Loop:**
    * Iterates through `amazon_datasets`.
    * Checks for existing CSV files.
    * If CSV exists, reads it. If not, downloads (if needed) and parses.
    * Stores DataFrames in the `datasets` dictionary.

* **Key Improvements:**
  * **`requests` for Downloads:** More robust than `gdown`.
  * **Correct JSON and Text Parsing:** Handles multi-line JSON and correct file encoding.
  * **`max_edges`:** Limits data for testing.
  * **Clear Structure:** `amazon_datasets` dictionary organizes information.
  * **CSV Caching:** Speeds up subsequent runs.

### Cell 6: Graph Preparation

```python
# ... (Code for creating the NetworkX graph) ...
```

* **Purpose:** Creates a `networkx.DiGraph` object from the parsed data.
* **Explanation:**
  * **`prepare_amazon_graph(copurchase_df, metadata_df=None)`:**
    * Creates a `nx.DiGraph()` (directed graph).
    * **Adding Edges:** Iterates through `copurchase_df`, adding edges with `G.add_edge(source, target)`. Handles `None` or empty `copurchase_df`. Handles `KeyError` on missing columns.
    * **Adding Node Attributes:** If `metadata_df` is valid, adds product metadata (title, description, etc.) as node attributes. Includes checks for:
      * `metadata_df` having an 'ASIN' column.
      * Non-empty ASIN values.
      * Node existence in the graph.
      * Non-NaN metadata values.
    * Returns the graph.
  * **Dataset Selection:** `dataset_name = "amazon0601"` (can be changed).
  * **Graph Creation:** Loads data, calls `prepare_amazon_graph`, stores the graph in `datasets`. Handles missing/empty dataset cases.

* **Key Improvements:**
  * **`nx.DiGraph()`:** Represents the directed nature of co-purchases.
  * **Metadata Integration:** Enriches the graph with product information.
  * **Robustness:** Handles missing/empty data, `KeyError`, missing ASINs, NaN values.
  * **String Node IDs:** Avoids type issues.
  * **Error Handling:** Sets `amazon_graph` to `None` on failure.

### Cell 7: Basic Graph Analysis

```python
# ... (Code for calculating graph statistics) ...
```

* **Purpose:** Calculates and presents key graph metrics.
* **Explanation:**
  * **`analyze_graph(G: Optional[nx.DiGraph]) -> Optional[dict]:`:**
    * Handles `None` input graph.
    * Calculates:
      * `num_nodes`, `num_edges`, `avg_degree`, `max_degree`.
      * `top_nodes_by_degree`: Top 10 most connected nodes.
      * `largest_cc_size`, `largest_cc_percentage`: Largest connected component.
      * `sample_subgraph`: A smaller subgraph (for visualization).
      * `sample_subgraph_size`.
    * **Sampling:** Creates a `sample_subgraph` for large graphs, starting from a highly connected node and adding neighbors.
    * Returns a dictionary of metrics.
    * Includes error handling.
  * **Calling and Output:** Calls `analyze_graph`, checks for errors, and prints results clearly.

* **Key Improvements:**
  * **`Optional[nx.DiGraph]`:** Explicitly allows `None` input.
  * **`None` Handling:** Deals with missing input graph.
  * **Comprehensive Metrics:** Calculates a good range of statistics.
  * **Sampling:** Handles large graphs practically.
  * **Clear Output:** Readable presentation of results.
  * **Error Handling:** `try-except` for robustness.

### Cell 8: Community Detection (Corrected)

```python
# ... (Code for community detection) ...
```

* **Purpose:** Identifies communities (clusters) within the graph.
* **Explanation:**
  * **`detect_communities(G: Optional[nx.DiGraph], graph_analysis: Optional[dict] = None, max_nodes: int = 5000) -> Optional[dict]:`:**
    * Handles `None` input graph.
    * **Sampling:** If the graph is large (`> max_nodes`), samples a subgraph. Prioritizes using a previously created `sample_subgraph` from `graph_analysis`; otherwise, does random sampling.
    * **Undirected Conversion:** Converts the graph to undirected (`undirected_G`).
    * **Louvain Algorithm (with Fallback):**
      * Tries to import and use the `community` (python-louvain) library for Louvain community detection.
      * If `community` is not installed, falls back to `nx.connected_components` (a simpler method).
    * **Returns Results:** A dictionary with:
      * `algorithm`: "louvain" or "connected_components".
      * `num_communities`, `community_sizes`, `top_communities`.
      * `node_communities`: Maps each node to its community ID.
  * **Library Installation:** Attempts to install `python-louvain`.
  * **Calling and Output:** Calls `detect_communities` and prints the results.

* **Key Improvements:**
  * **Sampling:** Handles large graphs.
  * **Louvain/Fallback:** Tries to use a good algorithm but has a backup.
  * **Clear Output:** Presents results in a structured way.
  * **Error handling**

### Remaining Cells (Cohere API key, LangChain Setup, Gradio/Streamlit Interface)

The remaining cells build on the foundation established by the previous cells. Here's a brief overview:

* **Cohere API Key (Cells 19, and part of 20):** sets the Cohere API key for LLM integration.
* **LangChain Setup (Cell 20 and variations):**
  * **`setup_langchain_cohere(...)`:**
    * Sets up LangChain with the Cohere LLM.
    * Defines a `PromptTemplate` that includes graph analysis results and the user's query.
    * Uses `ChatCohere` (the recommended class) for interaction with Cohere.
    * Returns an `LLMChain`.
  * **`agentic_query(...)`:**
    * Takes a user query, the LLM chain, and analysis results.
    * Formats the input for the LLM.
    * Calls the LLM chain and returns the response.
  * **Example Queries:** Provides example queries to demonstrate interaction with the LLM.
* **Cell 21,22,23,24,25,26,27**: imports, install, and example usages of langchain and streamlit.

* **Gradio/Streamlit Interface (Cells 50, 53) (Focus on Streamlit - Cell 53):**
  * **Imports:** Imports Streamlit (`import streamlit as st`).
  * **`process_data(...)`:** This is the *core function* that's triggered when the user clicks the "Analyze" button. It:
    * Connects to ArangoDB.
    * Tries to load the graph from ArangoDB; if that fails, it downloads and processes the data (as described in earlier cells).
    * Optionally persists the graph to ArangoDB.
    * Performs graph analysis and community detection.
    * Sets up the LangChain LLM chain.
    * Queries the LLM with the user's input.
    * Returns the results in a dictionary.
  * **`main()` function:** This is where the Streamlit UI is defined and the data processing is orchestrated.
    * **`st.title(...)`:** Sets the title of the app.
    * **`st.sidebar`:** Creates the sidebar with:
      * **`st.header(...)`:** Section headers.
      * **`st.slider(...)`:** Slider for `max_nodes_to_display`.
      * **`st.selectbox(...)`:** Dropdown to select the dataset.
      * **`st.text_input(...)`:** Text box for the user's query.
      * **`st.checkbox(...)`:** Checkbox to enable/disable ArangoDB persistence.
      * **`st.button("Analyze")`:** The button that triggers the analysis.
    * **`st.columns(...)`:** Creates two columns for the main content area.
    * **`st.subheader(...)`:** Subheaders for sections.
    * **`st.text(...)` / `st.write(...)`:** Displays text output (graph summary, LLM response, status).
    * **`st.image(...)`:** Displays the graph visualization.
    * **`st.spinner(...)`:** Shows a loading indicator while processing.
    * **Session State (`st.session_state`)**: Stores results so they persist between interactions. This is crucial for Streamlit's re-running behavior. The results of `process_data` are stored in `st.session_state['results']`.
    * **`run_button.click(...)`:** Connects the "Analyze" button to the `process_data` function. This specifies what happens when the button is clicked: `process_data` is called with the current values from the input widgets, and the outputs of `process_data` are used to update the output elements on the page.
    * **`iface.launch(debug=True)` / Streamlit Run:** Starts the Streamlit web application.
  * **Amazon Datasets definition:** Datasets are listed for use by the streamlit app.

**Running the Streamlit App:**

Because you are using Colab, the streamlit app is run by launching the gradio app. You would normally run `streamlit run arangoo.py` (assuming you saved the Streamlit code as `arangoo.py`).
