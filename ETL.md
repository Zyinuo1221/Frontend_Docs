<div align="center">

# 🚀 ETL Pipeline Documentation: The Journey from Input to LangGraph Prompt 🚀

</div>

## 📑 Table of Contents
- [Overview](#overview)
- [Pipeline Database Structure](#pipeline-database-structure)
- [Prompt Template Guidelines](#prompt-template-guidelines)
---

## Overview

This pipeline is designed to facilitate seamless and dynamic handling of user input data gathered from the MVC (Model-View-Controller) interface. It performs several critical operations, from validation to extraction and matching, ensuring that each user’s unique data is accurately integrated into pre-defined prompt templates stored within the Data Mart. This pipeline is essential for generating fully-formed LangGraph prompts that can be utilized in downstream applications.

---
## Pipeline Database Structure

This section details each table within the ETL pipeline, outlining its purpose, columns, and role in storing key data at each stage of the extraction, validation, and prompt-assembly process.

#### 1. 💾_Source Table
**Purpose:** Stores metadata required for MVC user input validation, including request verification, authorization, and required variables for prompts.

- **Columns:**
  - **💾_Source**: *Primary Key (PK)* — Holds the source name, serving as a reference in the `🧑‍💻_MVC_Input` table.
  - **📥_Operation_Request**: *Linked Record* — Connects to the corresponding operation request in the Data Mart, retrievable from the front-end URL (e.g., `Create_Project`).
  - **💾_Authorization**: *Text Field* — Stores authorization codes for validating the request origin.
  - **💻_Required_Object**: *Text Field* — Specifies essential variables that need to be collected from the user input for prompt generation.
  - **🧑‍💻_MVC_Input**: *Linked Record* — Establishes a one-to-many relationship with user inputs, enabling data flow between tables.

#### 2. 🧑‍💻_MVC_Input Table
**Purpose:** Captures and validates MVC inputs, managing validation status and tracking progression throughout the ETL pipeline.

- **Columns:**
  - **🧑‍💻_MVC_Input**: *Primary Key (PK)* — Identifies the input instance, serving as a reference in the `🖊️_Input_Extracted` table.
  - **🧑‍💻_PK**: *Sequence Identifier* — Tracks the position of the user input within the processing pipeline.
  - **💾_Source**: *Linked Record* — Connects to `💾_Source` to validate input against source metadata, with associated lookup fields:
    - **💻_Required_Object**: retrieve the essential variables for user input validation.
    - **💾_Authorization**: retrieve the code value for user source validation
  - **🧑‍💻_INPUT**: *JSON Field* — Stores the raw JSON data from the user.
  - **🧑‍💻_Filter**: *Boolean Field* — Reflects the validation result, with `TRUE` or `FALSE` values.
  - **🧑‍💻_Status**: *Single-Select Field* — Displays HTML status codes (e.g., `200`, `400`) that represent the validation progress.
  - **🧑‍💻_Error_Message**: *Long Text Field* — Logs any errors encountered during validation.
  - **🧑‍💻_Return_to_MVC**: *Long Text Field* — Stores return messages sent to the MVC, indicating the validation status.
  - **🖊️_Input_Extract_1️⃣**: *Linked Record* — Facilitates data transfer to the extraction stage.

#### 3. 🖊️_Input_Extracted Table
**Purpose:** Stores extraction results, including specific extracted data fields and combined information, for further downstream processing.

- **Columns:**
  - **🖊️_Extract**: *Primary Key (PK)* — Serves as an identifier for the extraction instance, allowing direct reference in the next processing stage.
  - **🧑‍💻_Input_1️⃣**: *Linked Record* — Receives data from `🧑‍💻_MVC_Input` for extraction, with associated lookup fields:
    - **1️⃣_🧑‍💻_PK**: *Lookup Field* — Retrieves the input sequence ID.
    - **1️⃣_🧑‍💻_INPUT**: *Lookup Field* — Retrieves raw input data.
  - **🖊️_Process_Status**: *Single-Select Field* — Controls automation and triggers extraction scripts.
  - **🖊️_Profile_Extract_⚡️, 🖊️_Operation_Extract_⚡️, 🖊️_Input_Extract_⚡️**: *Text Fields* — Store the extracted data elements result for specific fields.
  - **🖊️_User_Info**: *Long Text Field* — Merges extracted profile and input information for further processing.
  - **⚙️_DM_Process**: *Linked Record* — Directs data flow to subsequent ETL stages.
  
#### 4. ⚙️_Process_Extracted_to_PromptContent Table
**Purpose:** Controls the prompt matching and input-filling processes, aligning user inputs with prompt templates from the Data Mart for accurate prompt generation.

- **Columns:**
  - **⚙️_Process_Extracted_Info**: *Primary Key (PK)* — Identifies the specific process instance, directing records to the transformation stage.
  - **🖊️_Input_Extract_1️⃣**: *Linked Record* — Retrieves data from the previous extraction stage, ensuring continuity. Includes:
    - **1️⃣_🧑‍💻_PK**: *Lookup Field* — Tracks the unique input sequence for consistency across pipeline stages.
    - **🖊️_Extracted_Operation_⚡️**: *Lookup Field* — Used to match with Data Mart records for identifying the appropriate prompt template.
  - **⚙️_Matching_Process_Status**: *Single-Select Field* — Handles error states during the matching process (e.g., Success, Pending, Error).
  - **📥_Prompt_Template**: *Linked Record* — Holds the matched prompt template, with a roll-up field to aggregate template details.
    - **📄_Live_Prompt_Content**: *Roll-up Field* — Contains the actual template content used for input injection.
  - **⚙️_User_Template_Combination**: *Long Text Field* — Stores the final filled template, containing user input values.
  - **🔧_Transform_the_Process**: *Linked Record* — Directs data to the subsequent transformation stage, maintaining an efficient ETL flow.

#### 5. 🔧_Transform_to_JSON Table
**Purpose:** Facilitates the transformation of prompt content into JSON format, a structured output for downstream processing in the ETL pipeline.

- **Columns:**
  - **🔧_JSON_Transform**: *Primary Key (PK)* — Acts as an identifier for the transformation instance, directing the JSON output toward the Output stage.
  - **⚙️_Process_Extracted_Input**: *Linked Record* — Retrieves processed data from the previous ETL stage to ensure data continuity. Includes:
    - **1️⃣_🧑‍💻 _PK**: *Lookup Field* — Tracks the unique sequence identifier of each user input, providing traceability across pipeline stages.
    - **⚙️_User_Template_Combination**: *Lookup Field* — Extracts values from the LangGraph prompt to facilitate accurate JSON transformation.
  - **🔧_Transform_Status**: *Single-Select Field* — Manages error handling states (e.g., In Progress, Completed, Failed) to monitor the transformation process status.
  - **🔧_JSON_Format_Prompt**: *Long Text Field* — Stores the generated JSON format output after transformation.
  - **💡_Transform_Prompt_Output**: *Linked Record* — Passes the transformed JSON output directly to the next pipeline stage, maintaining efficient data flow across the ETL process.

#### 6. 💡_Transform_Prompt_Output Table
**Purpose:** Captures the final, usable LangGraph prompt output, preparing it for the loading stage in the ETL pipeline.

- **Columns:**
  - **💡_Prompt_Output**: *Primary Key (PK)* — Stores the name of the prompt output, which will be referenced in the subsequent loading process.
  - **🔧_Transform_to_JSON**: *Linked Record* — Retrieves the transformation results from the previous stage. Includes lookup fields:
    - **1️⃣_🧑‍💻_PK**: *Lookup Field* — Tracks the unique sequence identifier of the input through all stages.
    - **🔧_JSON_Format_Prompt**: *Lookup Field* — Retrieves the JSON-formatted LangGraph prompt.
  - **📤_Output_Load**: *Linked Record* — Facilitates the direct data transfer to the loading stage, ensuring the final prompt output is ready for deployment.
  
#### 7. 📤_Output_Load Table
**Purpose:** Stores the final output along with the target loading source. As the specific loading source is currently unspecified, this field remains empty for future assignment.

- **Columns:**
  - **📤_Output_JSON_Load**: *Primary Key (PK)* — Holds the name of the output, providing a unique identifier for each JSON load instance.
  - **💡_Transformed_Prompt_Output**: *Linked Record* — Retrieves the JSON output from the `💡_Transform_Prompt_Output` table, ensuring continuity in data flow across pipeline stages. Includes:
    - **1️⃣_🧑‍💻_PK**: *Lookup Field* — Tracks the unique sequence identifier to maintain traceability.
    - **🔧_JSON_Format_Prompt**: *Lookup Field* — Accesses the JSON-formatted prompt for validation and loading.
  - **📤_Output_Loading**: *Long Text Field* — Stores the processed and cast version of the `🔧_JSON_Format_Prompt`, enhancing loading performance and enabling compatibility with various data targets.

---
## Prompt Template Guidelines

This section outlines the structure, conventions, and labeling used within each prompt template to ensure consistency and clarity in prompt generation.

### Template Structure
- **Prompt Format**: Each prompt is structured as a JSON object `{ }`, designed to act as a function that performs a specific role. This standardized format facilitates easy parsing, processing, and transformation throughout the ETL pipeline.
- **Steps as Top-Level Keys**: Each prompt consists of several steps, where each step title serves as a top-level key within the JSON output. These top-level keys define distinct stages or segments of the prompt’s intended functionality.

- **Prompt Classes**: Within each step, multiple attributes (referred to as "prompt classes") contribute to the final prompt content. These classes are labeled with a square bracket `[...]` to clearly indicate each attribute or element, allowing for organized representation and easier referencing.

### Placeholder Variables
- **Global Variable Syntax**: In the prompt template content, values that require dynamic input are enclosed within angle brackets `<...>`. Each `<global variable>` represents a specific placeholder that is replaced with user input values during the transformation stage, ensuring tailored and accurate prompt generation.
  
By following these conventions, the prompt templates maintain a clear and predictable structure, enabling efficient data injection and processing across the ETL pipeline.


