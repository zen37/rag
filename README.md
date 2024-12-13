
## Requirements

1. User enters document id.
2. Application retrieves all information about the respective id from: ADX(kusto), ITSM, AzureDevOps
3. If id is mentioned in an incident with severity higher than 4, cache the results, for faster retrieval next time
4. Log the searches


## Technical Architecture


### **1. Web Interface (Frontend)**

The web interface will be built to allow users to interact with the system via **natural language queries**. It will allow users to ask questions like “Tell me everything about document ID X,” and receive answers from various sources.

#### Technologies:
- **Frontend Framework**: 
  - **React.js** (or **Vue.js** / **Angular**): A modern JavaScript framework to build a **single-page application (SPA)**. React is commonly used for dynamic web applications that provide seamless user experiences.
  - **TypeScript**: To ensure type safety, which makes the frontend code more robust and maintainable.
  - **Tailwind CSS**: For efficient and responsive design, enabling a clean, customizable UI.
  - **Axios** or **Fetch API**: For making HTTP requests from the web page to the backend APIs.

- **User Interaction**:
  - **Input Field** for natural language queries (e.g., "What do you know about document ID X?")
  - **Response Display**: The response from the AI system (Azure OpenAI) will be displayed on the webpage in a readable format.

---

### **2. Backend (API Layer)**

The backend processes the user's query, retrieves data from multiple sources (Kusto, ITSM, ADO, SharePoint, OneNote), and integrates the results before passing them to the AI model (Azure OpenAI) for natural language generation.

#### Technologies:
- **Backend Framework**: 
  - **FastAPI** (or **Flask** / **Django**): For a Python-based REST API that can handle incoming queries. **FastAPI** is a modern, high-performance framework for building APIs and is ideal for serving machine learning models and data retrieval operations.
  - **Python 3.9+**: Python is preferred for its rich ecosystem for working with AI models, databases, and APIs.

- **API Routes**: 
  - An endpoint `/query` to receive user input and process the query.
  - The backend will orchestrate the querying of Kusto, ITSM, ADO, and SharePoint data, aggregating the results and sending them to the AI model.

---

### **3. Data Retrieval Layer**

The retrieval layer is responsible for interacting with various data sources, such as **Kusto**, **ITSM**, **ADO**, and **SharePoint/OneNote**, and retrieving relevant information based on the user query.

#### Technologies:
- **Kusto (Azure Data Explorer)**: For querying structured data stored in **Kusto tables**.
  - **Kusto SDK**: Use the **Azure Kusto Python SDK** or **Kusto REST API** to connect to Azure Data Explorer and execute **KQL queries**.
  - **Azure KQL**: To query the relevant Kusto tables for data about the `document_id`.

- **ITSM (ServiceNow, etc.)**: To retrieve unstructured data from ITSM systems.
  - **ServiceNow API** or **REST API** to search incidents, problems, or other records where the `document_id` might be mentioned.
  - Use **full-text search** or **semantic search** capabilities to search through description fields or other text fields in incidents or requests.

- **Azure DevOps (ADO)**: To retrieve work items (e.g., bugs, user stories) mentioning the `document_id`.
  - **Azure DevOps REST API**: Allows querying work items, commits, pull requests, and other resources where the `document_id` could be mentioned in the title or description.
  - **Azure DevOps Search** or **Work Item Query Language (WIQL)**: Use these to run queries based on free-text fields.

- **SharePoint and OneNote**: To search for documents or notes that mention the `document_id`.
  - **Microsoft Graph API**: Provides access to SharePoint and OneNote documents.
  - **Full-text search**: Can be used to search document libraries, lists, and note contents.

---

### **4. AI Model Integration Layer (LangChain + Azure OpenAI)**

LangChain simplifies working with large language models (LLMs) like **Azure OpenAI GPT** and handles the orchestration of retrieving and processing data. It enables easy interaction with multiple data sources and helps create a structured approach for **retrieval-augmented generation (RAG)**.

#### Technologies:
- **LangChain**:
  - LangChain is used to **integrate external data sources** (like Kusto, ITSM, ADO) with the **Azure OpenAI** model. 
  - **LangChain’s retrieval system** will allow querying the external sources (Kusto, ITSM, etc.) and then feeding the results into the AI model to generate a natural language response.
  - **LangChain’s prompt templates** will help structure the input data for the AI model, ensuring the AI generates meaningful and contextually relevant answers.
  - **Custom Tools**: LangChain can be used to define custom tools (e.g., a custom Kusto query tool, or ITSM query tool) that retrieve data from specific sources, which is then passed to the OpenAI model.

- **Azure OpenAI**:
  - The **Azure OpenAI GPT model** (or another LLM) will be used for **text generation**.
  - The model will take in **retrieved data** from Kusto, ITSM, ADO, and SharePoint/OneNote, and generate a response that presents the relevant information to the user.
  - **Azure Cognitive Services** for Natural Language Processing (NLP) could also be used to enhance query understanding and context extraction.

---

### **5. Data Aggregation & Formatting**

Once data is retrieved from various sources, it needs to be aggregated and formatted before being passed to the AI model.

- **Data Normalization**: 
  - Structured data (from Kusto) is already in a normalized format (e.g., tabular data), but unstructured data (from ITSM, ADO, OneNote) might need some **preprocessing** like **tokenization**, **named entity recognition (NER)**, or **text summarization** to extract the most relevant information.
  - Use **text cleaning** to remove extraneous data from descriptions, comments, etc., to focus on the `document_id` references.

- **Data Structuring**:
  - For example, ADO work items might need to be presented as **title**, **status**, **description**, while ITSM incidents may need to show **incident ID**, **title**, and **status**.

---

### **6. Backend Infrastructure & Security**

- **Authentication**: 
  - Use **OAuth** or **Azure Active Directory** for authenticating users who access the system.
  - Ensure that sensitive data (e.g., ITSM, ADO) is protected with appropriate access controls and **role-based access control (RBAC)**.

- **Cloud Infrastructure**:
  - The entire system can be hosted on **Azure** using services like **Azure Web Apps** for the frontend and backend APIs, and **Azure Cognitive Services** for the AI model (via **Azure OpenAI**).
  - **Azure Functions** (optional): Can be used for handling serverless functions like querying databases or running ad-hoc processes (e.g., querying Kusto tables).

---

### **7. Architecture Flow Summary**

1. **Frontend (React.js)** sends the user's query to the backend API.
2. **Backend API (FastAPI)** processes the query and determines which data sources to query (Kusto, ITSM, ADO, etc.).
3. **LangChain** orchestrates the data retrieval from Kusto, ITSM, ADO, and SharePoint using APIs or direct queries.
4. The retrieved data is aggregated, cleaned, and passed to **Azure OpenAI GPT** through LangChain for **natural language generation**.
5. **Azure OpenAI** processes the input data and generates a human-readable response.
6. The response is sent back to the **frontend (React.js)** and displayed to the user in a readable format.

---

### **Key Technologies Summary**:
- **Frontend**: React.js (TypeScript), Tailwind CSS
- **Backend**: FastAPI (Python), Azure Functions
- **Data Retrieval**: Azure Kusto SDK, Azure DevOps REST API, ServiceNow API, Microsoft Graph API
- **AI Integration**: LangChain, Azure OpenAI GPT
- **Cloud Infrastructure**: Azure Web Apps, Azure Cognitive Services, Azure Active Directory for authentication

---