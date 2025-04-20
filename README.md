# GenIAVectorizer

A lightweight Streamlit application designed to vectorize documents (PDF, TXT, Markdown, JSON) and store them in AstraDB for Retrieval-Augmented Generation (RAG) workflows. This project provides a user-friendly interface to upload documents, process them into vector embeddings, and store them with metadata for efficient retrieval.

Inspired by the sleek and modern design of GenIAlab, this application leverages the power of Streamlit for its frontend and AstraDB for scalable vector storage, making it an ideal tool for preparing documents for RAG-based applications.

## Table of Contents

- Features
- What is Streamlit?
- What is AstraDB?
- Prerequisites
- Installation
- Functionality
- Usage
- Deployment on Streamlit Community Cloud
- Troubleshooting
- Contributing
- License
- Contact

## Features

- **Multi-format Document Support**: Upload and process PDF, TXT, Markdown, and JSON files.
- **Duplicate Detection**: Uses SHA-256 hashing to skip previously vectorized files.
- **Chunking and Embedding**: Splits documents into manageable chunks (2500 characters, 250-character overlap) and generates embeddings using `sentence-transformers/all-MiniLM-L6-v2`.
- **Vector Storage**: Stores embeddings and metadata (`filename`, `upload_date`, `file_type`, `hash`) in AstraDB.
- **User-Friendly Interface**: Built with Streamlit, featuring a clean, modern UI with progress tracking and feedback.
- **Error Handling**: Robust error management for file processing and vector storage.

## What is Streamlit?

Streamlit is an open-source Python framework that enables data scientists and developers to create interactive web applications with minimal code. It is particularly popular for building data-driven apps, dashboards, and prototypes due to its simplicity and Pythonic API. Streamlit allows you to transform Python scripts into shareable web apps by adding a few commands, such as `st.title()`, `st.file_uploader()`, and `st.progress()`, which are used in this project to create the GenIAVectorizer interface.

**Key Streamlit Features Used in GenIAVectorizer**:

- **File Uploader**: Enables users to upload multiple documents via `st.file_uploader()`.
- **Progress Bar**: Displays processing progress with `st.progress()`.
- **Dynamic UI**: Provides real-time feedback (success, warning, error messages) using `st.success()`, `st.warning()`, and `st.error()`.
- **Secrets Management**: Securely handles sensitive credentials (e.g., AstraDB tokens) via `st.secrets`.

For more details, refer to the Streamlit Documentation.

## What is AstraDB?

AstraDB is a fully managed, cloud-native database-as-a-service built on Apache Cassandra, optimized for scalability and performance. It provides a vector storage capability, making it ideal for AI and machine learning workloads like RAG, where high-dimensional embeddings need to be stored and queried efficiently. AstraDB's vector store integrates seamlessly with embedding models, allowing for similarity searches and metadata storage.

**Key AstraDB Features Used in GenIAVectorizer**:

- **Vector Store**: Stores document embeddings and metadata in a collection using `AstraDBVectorStore`.
- **Similarity Search**: Checks for duplicate documents by comparing file hashes via `vectorstore.similarity_search()`.
- **Scalability**: Handles large volumes of document vectors with low latency.
- **Secure Authentication**: Uses API endpoints and application tokens for secure access.

For more details, refer to the AstraDB Documentation.

## Prerequisites

To run or deploy this project, you need:

- **Python 3.8+**: Download from python.org.
- **Git**: For cloning the repository.
- **AstraDB Account**: Sign up at astra.datastax.com and create a vector database. Obtain your API endpoint, application token, and namespace.
- **Streamlit Community Cloud Account**: For deployment (optional). Sign up at share.streamlit.io.
- **Dependencies**: Libraries listed in `requirements.txt` (e.g., `streamlit`, `langchain`, `langchain-astradb`, `langchain-huggingface`, `PyPDF2`, `unstructured`).

## Installation

Follow these steps to set up the project locally:

1. **Clone the Repository**:

   ```bash
   git clone https://github.com/dimopage/streamlit-astradb-rag.git
   cd streamlit-astradb-rag
   ```

2. **Create a Virtual Environment**:

   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install Dependencies**:

   ```bash
   pip install -r requirements.txt
   ```

4. **Set Up Secrets**: Create a `.streamlit/secrets.toml` file in the project root:

   ```bash
   mkdir .streamlit
   nano .streamlit/secrets.toml
   ```

   Add the following, replacing placeholders with your AstraDB credentials:

   ```toml
   [secrets]
   ASTRA_DB_API_ENDPOINT = "https://your-astra-db-api-endpoint"
   ASTRA_DB_APPLICATION_TOKEN = "AstraCS:your-token"
   ASTRA_DB_NAMESPACE = "your_namespace"
   ```

## Functionality

The GenIAVectorizer application processes uploaded documents and prepares them for RAG workflows. Here's a detailed breakdown of its functionality:

1. **Document Upload**:

   - Users upload files (PDF, TXT, MD, JSON) via Streamlit's file uploader.
   - Multiple files can be uploaded simultaneously.

2. **Duplicate Detection**:

   - Each file is hashed using SHA-256 to generate a unique identifier.
   - The hash is checked against existing vectors in AstraDB using `vectorstore.similarity_search()`.
   - If a match is found, the file is skipped to avoid redundant processing.

3. **Document Loading**:

   - Appropriate loaders are selected based on file type:
     - `PyPDFLoader` for PDFs.
     - `TextLoader` for TXT and Markdown.
     - `UnstructuredFileLoader` for JSON.
   - Files are temporarily saved to disk using `tempfile` for processing.

4. **Metadata Enrichment**:

   - Each document is tagged with metadata:
     - `filename`: Original file name.
     - `upload_date`: ISO-formatted timestamp.
     - `file_type`: MIME type (e.g., `application/pdf`).
     - `hash`: SHA-256 hash.

5. **Text Splitting**:

   - Documents are split into chunks using `RecursiveCharacterTextSplitter` (chunk size: 2500 characters, overlap: 250 characters).
   - This ensures manageable chunks for embedding and retrieval.

6. **Embedding Generation**:

   - Chunks are converted to vector embeddings using the `sentence-transformers/all-MiniLM-L6-v2` model via `HuggingFaceEmbeddings`.
   - Embeddings capture semantic meaning for similarity searches.

7. **Vector Storage**:

   - Embeddings and metadata are stored in an AstraDB collection (named `default` by default).
   - The `AstraDBVectorStore` class handles storage and retrieval.

8. **User Feedback**:

   - A progress bar tracks file processing.
   - Success, warning, or error messages inform the user of outcomes (e.g., number of chunks stored, skipped files, or processing errors).

## Usage

1. **Run the Application Locally**:

   ```bash
   streamlit run streamlit_app.py
   ```

   The app will open in your browser at `http://localhost:8501`.

2. **Upload Documents**:

   - Use the file uploader to select one or more PDF, TXT, Markdown, or JSON files.
   - The app processes files in sequence, displaying a progress bar.
   - For each file:
     - The app checks for duplicates using the file's hash.
     - If not a duplicate, the file is loaded, split into chunks, embedded, and stored in AstraDB.
     - Metadata (filename, upload date, file type, hash) is attached to each chunk.
   - Upon completion, a success message shows the number of chunks stored (e.g., "Vectorized and stored 42 chunks in collection 'default'").
   - Skipped files (duplicates) or errors are reported via warnings or error messages.

3. **Verify in AstraDB**:

   - Log in to astra.datastax.com.
   - Navigate to your database and namespace.
   - Check the collection (e.g., `default`) to confirm that vectors and metadata are stored correctly.
   - Use AstraDB's query tools to inspect embeddings or perform similarity searches.

## Deployment on Streamlit Community Cloud

To deploy the app publicly:

1. **Push to GitHub**: Ensure all files are committed and pushed to a public repository:

   ```bash
   git add .
   git commit -m "Prepare for Streamlit Cloud deployment"
   git push origin main
   ```

2. **Create a Streamlit Cloud App**:

   - Log in to Streamlit Community Cloud.

   - Click **New App** and select your GitHub repository.

   - Set the main file to `streamlit_app.py`.

   - In **Advanced Settings**, add the secrets from your `.streamlit/secrets.toml`:

     ```toml
     ASTRA_DB_API_ENDPOINT = "https://your-astra-db-api-endpoint"
     ASTRA_DB_APPLICATION_TOKEN = "AstraCS:your-token"
     ASTRA_DB_NAMESPACE = "your_namespace"
     ```

3. **Deploy**:

   - Click **Deploy** and wait for the app to build.
   - Access the app via the provided URL (e.g., `https://your-app-name.streamlit.app`).
   - Test by uploading documents and verifying success messages.

## Troubleshooting

- **AstraDB Connection Error**: Ensure your API endpoint, token, and namespace are correct in `secrets.toml` or Streamlit Cloud secrets. Check the AstraDB Documentation for setup details.
- **File Processing Errors**: Verify that files are valid (e.g., non-corrupted PDFs, well-formed JSON). Unsupported file types will trigger a warning.
- **Dependency Issues**: Ensure `requirements.txt` matches the required versions. Run `pip install -r requirements.txt` in a clean virtual environment.
- **Streamlit UI Issues**: Clear the browser cache or try a different browser if the app doesn't load. Ensure you're using port 8501 (Streamlit's default).

## Contributing

Contributions are welcome! To contribute:

- Open issues or submit pull requests on GitHub.
- Suggest new features, such as additional file types or RAG retrieval capabilities.
- Follow the Contributing Guidelines.

## License

This project is licensed under the MIT License. See the LICENSE file for details.

## Contact

For questions or support, open an issue on GitHub or email \[hi@genialab.space\].

---

Built with ❤️ for document processing and RAG workflows by GenIAlab.Space.