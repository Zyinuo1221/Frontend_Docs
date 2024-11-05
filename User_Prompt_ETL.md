<div align="center">

# 🚀 ETL Pipeline Documentation: The Journey from Input to LangGraph Prompt 🚀

</div>

## 📑 Table of Contents
- [Overview](#overview)
- [Pipeline Database Structure](#pipeline-database-structure)
- [Prompt Template Guidelines](#prompt-template-guidelines)
- [IPO Process (Input-Process-Output)](#ipo-process-input-process-output)
  - [Input Validation](#input-validation)
  - [Template Processing](#template-processing)
  - [Output Formatting](#output-formatting)
- [ETL Pipeline](#etl-pipeline)
  - [Extraction](#extraction)
  - [Transformation](#transformation)
  - [Loading](#loading)

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

---
## IPO Process (Input-Process-Output)

The **IPO Process** (Input-Process-Output) defines the data flow structure of our ETL pipeline, focusing on validating incoming data, processing it to match pre-defined templates, and formatting the final output. This systematic approach ensures that the pipeline operates efficiently, accurately, and reliably from initial input capture through to the final output formatting. The IPO Process comprises three main stages:

## Input Validation

The **Input Validation** stage verifies the incoming JSON data from the MVC for correctness, completeness, and authorization. This ensures all required fields and values are in place before the data proceeds to further ETL stages.

### Process Flow and Code Breakdown

1. **Record Retrieval and Field Initialization**
   - Retrieve the triggered record ID and select the fields required for validation, including `userInputJSON`, `authorizationField`, and `requiredKeysArray`. These fields determine the core elements that need validation.

   ```javascript
   let inputConfig = input.config();
   let recordId = inputConfig.recordId;
   let table = base.getTable('🧑‍💻_MVC_Input');
   let record = await table.selectRecordAsync(recordId);

   let userInputJSON = record.getCellValue('🧑‍💻_INPUT');
   let authorizationField = record.getCellValue('💾_Authorization') ? record.getCellValue('💾_Authorization')[0] : null;
   let requiredKeysArray = record.getCellValue('💻_Required_Key') || [];
   let requiredKeys = requiredKeysArray.join("\n").split("\n").map(key => key.trim()).join(", ");
   let errorLog = '';
   ```
2. **Empty Input Check**
  - If userInputJSON is empty, log an error, update the Return_to_MVC field with a re-entry prompt, and exit the function.
  
  ```javascript
  if (!userInputJSON) {
    errorLog += "🧑‍💻_INPUT field is empty, unable to continue processing.\n";
    let returnMessage = errorLog + "Please re-enter your choice";
    await table.updateRecordAsync(recordId, {
        'fldV5JSf0VaggGYjp': {name: '400 Bad Request'},
        'fldp1jrAxRnjgEATC': {name: 'FALSE'},
        '🧑‍💻_Error_Message': errorLog,
        '🧑‍💻_Return_to_MVC': returnMessage
    });
    return;
  }
  ```
3. **JSON Parsing and Cleaning**
  - Clean userInputJSON of invalid characters and parse it into an object. If parsing fails, log an error and provide feedback to MVC for re-entry.
    
  ```javascript
  let cleanedJSON = userInputJSON.replace(/\\_/g, '_')
                               .replace(/\\/g, '')
                               .replace(/[\u200B-\u200D\uFEFF\s]+/g, ' ')
                               .trim();
  let userInput;
  try {
      userInput = JSON.parse(cleanedJSON);
  } catch (error) {
      errorLog += "JSON parsing failed: " + error.message + "\n";
      let returnMessage = errorLog + "Please re-enter your choice";
      await table.updateRecordAsync(recordId, {
          'fldV5JSf0VaggGYjp': {name: '400 Bad Request'},
          'fldp1jrAxRnjgEATC': {name: 'FALSE'},
          '🧑‍💻_Error_Message': errorLog,
          '🧑‍💻_Return_to_MVC': returnMessage
      });
      return;
  }
  ```

4. **Authorization Validation**
  - Confirm the presence of an Authorization key in userInput and check if its value matches authorizationField. Log any discrepancies to prevent unauthorized access.

  ```javascript
  if (!userInput.Authorization) {
    errorLog += "Authorization key is missing in the input JSON.\n";
} else if (userInput.Authorization !== authorizationField) {
    errorLog += "Authorization mismatch. Possible unauthorized access attempt.\n";
}
  ```
5. **Required Top-Level Key Check**
  - Verify the presence of mandatory top-level keys (user_profile, operation_request, and user_input). Missing keys are logged for error reporting.

  ```javascript
    if (!userInput.Authorization) {
      errorLog += "Authorization key is missing in the input JSON.\n";
  } else if (userInput.Authorization !== authorizationField) {
      errorLog += "Authorization mismatch. Possible unauthorized access attempt.\n";
  }
  ```
6. **Dynamic Nested Key Validation**
  - Using the list of requiredKeys, dynamically search for each required path within user_input. The searchKeyInJSON helper function performs a recursive search for each required key, logging any missing paths.

  ```javascript
  let requiredKeysFromSource = requiredKeys.split(',').map(key => key.trim());
  let missingPaths = [];
  
  requiredKeysFromSource.forEach(requiredKeyPath => {
      if (!searchKeyInJSON(userInput, requiredKeyPath)) {
          missingPaths.push(requiredKeyPath);
      }
  });
  
  if (missingPaths.length > 0) {
      errorLog += `Missing required prompt objects: ${missingPaths.join(', ')}.\n`;
  }
  
  function searchKeyInJSON(obj, keyToFind) {
      if (typeof obj !== 'object' || obj === null) return false;
      for (let key in obj) {
          if (key === keyToFind) return true;
          if (typeof obj[key] === 'object' && searchKeyInJSON(obj[key], keyToFind)) return true;
      }
      return false;
  }
  ```

7. **Validation Result Update**
  - Based on validation results, update the 🧑‍💻_Status, 🧑‍💻_Filter, 🧑‍💻_Error_Message, and 🧑‍💻_Return_to_MVC fields. If validation is successful, the status is set to 200 Passed; otherwise, it is set to 400 Bad Request with an error message.
  ```javascript
  let isValid = errorLog.length === 0;
  let status = isValid ? {name: '200 Passed'} : {name: '400 Bad Request'};
  let filterValue = isValid ? {name: 'TRUE'} : {name: 'FALSE'};
  let returnMessage = isValid ? "All inputs are valid" : (errorLog + "Please re-enter your choice");
  
  await table.updateRecordAsync(recordId, {
      'fldV5JSf0VaggGYjp': status,
      'fldp1jrAxRnjgEATC': filterValue,
      '🧑‍💻_Error_Message': errorLog,
      '🧑‍💻_Return_to_MVC': returnMessage
  });
  ```

### Summary
The **Input Validation** process performs a structured verification of each incoming record by:
- Checking JSON integrity and completeness.
- Validating required fields and authorization.
- Logging errors for immediate feedback, ensuring inputs meet all criteria before progressing further in the ETL pipeline.

## Template Processing

The **Template Processing** stage aligns extracted user data with pre-defined templates from the Data Mart. This process ensures that each prompt output is accurately tailored with relevant user data. The process is split into two primary steps: **Template Matching** and **Dynamic Value Injection**.

### Process Flow and Code Breakdown
#### 1. Template Matching and Error Handling
The Template Matching process validates and aligns the user’s extracted input with a corresponding prompt template stored in the Data Mart. This ensures that each user request is matched with an appropriate prompt template for further processing.

1. **Retrieve Extracted Operation**: Fetches the extracted operation information from the `🖊️_Extracted_Operation_⚡️` field to locate the relevant template.
2. **Initialize Matching Status**: Sets `⚙️_Matching_Process_Status` to “In-Progress” to track the matching progress.
3. **Error Handling**: If a matching template is not found or fails validation, `⚙️_Matching_Process_Status` is updated to “Failed” and the process is terminated to maintain data integrity.

```javascript
// Retrieve the triggered record ID
let inputConfig = input.config();
let recordId = inputConfig.recordId;

// Initialize the status to In-Progress
await assemblePromptTable.updateRecordAsync(recordId, {
    '⚙️_Matching_Process_Status': { name: 'In-Progress' }
});

// Retrieve and clean extracted operation field
let extractedOperationLookup = record.getCellValue('🖊️_Extracted_Operation_⚡️');
if (!extractedOperationLookup || !Array.isArray(extractedOperationLookup) || extractedOperationLookup.length === 0) {
    console.log("🖊️_Extracted_Operation_⚡️ field is empty or not valid.");
    await assemblePromptTable.updateRecordAsync(recordId, {
        '⚙️_Matching_Process_Status': { name: 'Failed' } 
    });
    return;
}

let cleanedOperation = extractedOperationLookup[0].trim().toLowerCase();

// Match extracted operation with the 📥_Operation_Request table
let matchedOperation = operationRecords.records.find(r => {
    let operationValue = r.getCellValue('📥_Operation_Request');
    return operationValue && operationValue.trim().toLowerCase() === cleanedOperation;
});

// Update prompt template and matching status if a match is found
if (matchedOperation) {
    await assemblePromptTable.updateRecordAsync(recordId, {
        '📥_Prompt_Template': [{id: matchedOperation.id}],
        '⚙️_Matching_Process_Status': { name: 'Completed' }
    });
} else {
    await assemblePromptTable.updateRecordAsync(recordId, {
        '⚙️_Matching_Process_Status': { name: 'Failed' }
    });
}
```
##### Output:
- **Matched Template**: If a match is found, the record in `⚙️_Process_Extracted_to_AssemblePrompt` is linked with the appropriate prompt template from the `📥_Operation_Request` table, based on the operation extracted from user input.
- **Matching Status**: Updates `⚙️_Matching_Process_Status` to “Completed” if the match is successful. If no match is found or an error occurs, the status is set to “Failed” to halt further processing.

#### 2. Dynamic Value Injection into Prompt Template
This stage dynamically injects user-specific information into the matched template, preparing it for final output as a LangGraph prompt.

1. **Retrieve Template Content**: Loads the selected template and the user’s JSON data to insert values.
2. **Placeholder Replacement**: Identifies <...> placeholders and replaces them with actual values from the userJson.
3. **Special Handling for project_objective**: Format goal and timeframe as a single string for user clarity.

```javascript
// Extract placeholders and replace with user values
let regex = /<([^>]+)>/g;
for (let match of promptTemplate.matchAll(regex)) {
    let placeholder = match[0];
    let key = match[1];

    if (key.toLowerCase() === "project_objective") {
        let projectObjective = findNestedValue(userJson, key);
        if (projectObjective) {
            let goal = formatNestedObject(findNestedValue(projectObjective, "goal"));
            let timeframe = formatNestedObject(findNestedValue(projectObjective, "timeframe"));
            promptTemplate = promptTemplate.replace(placeholder, `${goal} in ${timeframe}`);
        }
    } else {
        let value = findNestedValue(userJson, key);
        promptTemplate = promptTemplate.replace(placeholder, `${key}: ${value}`);
    }
}

await assemblePromptTable.updateRecordAsync(recordId, {
    '⚙️_User_Template_Combination': promptTemplate
});
```

##### Output:
- **Usable Prompt**: After dynamic value injection, the prompt template is customized with relevant user data, creating a complete and usable LangGraph prompt stored in `⚙️_User_Template_Combination`. This prompt is now ready for the next stage in the ETL pipeline.

#### 3. Output Formatting

The **Output Formatting** stage finalizes the JSON prompt output by formatting and storing it in a designated field within the `📤_Ouput_Load` table. This ensures the generated prompt content is fully prepared for downstream usage or integration with external systems.

1. **Retrieve and Verify Record**: Checks if the triggered record exists within the `📤_Ouput_Load` table. If the record is missing, the process is terminated.
2. **Extract JSON Prompt Content**: Accesses the `🔧_JSON_Format_Prompt` field, which contains the formatted JSON prompt. If the field is empty or invalid, the process exits to prevent downstream errors.
3. **Store JSON Content**: Copies the validated JSON prompt into the `📤_Output_Loading` field, which acts as the final repository for the structured prompt output.

```javascript
// Get the triggered record ID from the config
let inputConfig = input.config();
let recordId = inputConfig.recordId;

// Get the Output_Load table
let outputLoadTable = base.getTable('📤_Ouput_Load');

// Select the record from the Output_Load table using the record ID
let record = await outputLoadTable.selectRecordAsync(recordId);

// Check if the record exists
if (!record) {
    console.log(`No record found with ID ${recordId}`);
    return;  // Exit if the record does not exist
}

// Get the value of the 🔧_JSON_Format_Prompt lookup field
let jsonFormatLookup = record.getCellValue('🔧_JSON_Format_Prompt');

// Check if the lookup field contains a valid value
if (!jsonFormatLookup || !Array.isArray(jsonFormatLookup) || jsonFormatLookup.length === 0) {
    console.log("🔧_JSON_Format_Prompt field is empty or invalid.");
    return;  // Exit if no valid data is found
}

// Extract the first value from the lookup array
let jsonFormattedContent = jsonFormatLookup[0];

// Update the 📤_Output_Loading field with the extracted value
await outputLoadTable.updateRecordAsync(recordId, {
    '📤_Output_Loading': jsonFormattedContent  // Store the JSON content in the long-text field
});

console.log("JSON content successfully copied to 📤_Output_Loading.");
```
##### Output:
- **Formatted JSON Prompt**:The 📤_Output_Loading field in 📤_Ouput_Load now contains the finalized JSON prompt, ready for use in the ETL pipeline’s final stages or external applications.


