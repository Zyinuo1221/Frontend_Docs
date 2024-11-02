<div align="center">

# 🚀 ETL Pipeline Documentation: The Journey from Input to LangGraph Prompt 🚀

</div>

## 📑 Table of Contents
- [Overview](#overview)
- [Pipeline Database Descriptions](#pipeline-database-descriptions)

---

## Overview

This pipeline is designed to facilitate seamless and dynamic handling of user input data gathered from the MVC (Model-View-Controller) interface. It performs several critical operations, from validation to extraction and matching, ensuring that each user’s unique data is accurately integrated into pre-defined prompt templates stored within the Data Mart. This pipeline is essential for generating fully-formed LangGraph prompts that can be utilized in downstream applications.

## Pipeline Database Structure

This section details each table within the ETL pipeline, outlining its purpose, columns, and role in storing key data at each stage of the extraction, validation, and prompt-assembly process.

#### 1. 💾_Source Table
**Purpose:** Stores metadata required for MVC user input validation, including request verification, authorization, and required variables for prompts.

- **Columns:**
  - **💾_Source (PK)**: Storing source name, will serving as a reference in `🧑‍💻_MVC_Input`.
  - **📥_Operation_Request**: Links to corresponding operation requests table in the datamart and could be retrive from the front-end URL, such as `Create_Project`.
  - **💾_Authorization**: Stores authorization codes for validation.
  - **💻_Required_Object**: Holds prompt-required variables for input.
  - **🧑‍💻_MVC_Input**: Facilitates one-to-many relationships with user input, enabling data flow between tables.

#### 2. 🧑‍💻_MVC_Input Table
**Purpose:** Captures and validates MVC input, manages validation status, and tracks validation process progression.

- **Columns:**
  - **🧑‍💻_MVC_Input (PK)**: Storing input name, will serving as a reference in `🖊️_Input_Extracted`.
  - **🧑‍💻_PK**: Sequence identifier tracking user input through the pipeline.
  - **💾_Source**: Links to `💾 _Source` for validation against the source’s authorization and required object metadata.
  - **🧑‍💻_INPUT**: Stores raw user JSON data.
  - **🧑‍💻_Filter**: Binary Boolean field reflecting validation result.
  - **🧑‍💻_Status**: HTML status codes indicating validation progress.
  - **🧑‍💻_Error_Message**: Logs any errors encountered during validation.
  - **🧑‍💻_Return_to_MVC**: Return message conveying validation status to MVC.
  - **🖊️_Input_Extract_1️⃣**: Field facilitating direct data transfer for extraction.

#### 3. 🖊️_Input_Extracted Table
**Purpose:** Stores results from the extraction stage, including extracted data fields and combined information for downstream processing.

- **Columns:**
  - **🖊️_Extract (PK)**: Storing extraction process name, will serving as a reference in `🖊️_Input_Extracted`
  - **🧑‍💻_Input_1️⃣**: Link field in order to receive data from `🧑‍💻_MVC_Input`.
  - **1️⃣_🧑‍💻_PK & 1️⃣ _🧑‍💻_INPUT**: Lookup fields retriving sequence and input data.
  - **🖊️_Process_Status**: Automation control field activating extraction scripts.
  - **🖊️_Profile_Extract_⚡️, 🖊️_Operation_Extract_⚡️, 🖊️_Input_Extract_⚡️**: Extracted information columns.
  - **🖊️_User_Info**: Merges profile and input extractions for further use.
  - **⚙️ DM_Process**: Field facilitating direct data transfer for further processing.


