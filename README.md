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
