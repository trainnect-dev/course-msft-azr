### Lab 1.5: Understanding Healthcare Data Fragmentation

**Module Alignment:** Section 1: Introduction to Microsoft Fabric in Healthcare

**Objective:**
*   Analyze a fictional hospital’s data systems to identify fragmentation.
*   Understand the challenges posed by data fragmentation for analytics.
*   Conceptualize how Microsoft Fabric can address these challenges.

**Scenario:**
Valley General Hospital, a mid-sized healthcare provider, uses several distinct IT systems to manage its operations and patient care:
1.  **Electronic Health Record (EHR) System:** Contains patient demographics, clinical encounter details, prescribed medications, allergies, and problem lists.
2.  **Pharmacy System:** Manages medication dispensing, prescription fill history, and inventory. Linked to the EHR for prescriptions but operates as a separate database for dispensing records.
3.  **Laboratory Information System (LIS):** Stores results from all lab tests conducted, including blood work, pathology reports, and microbiology. Results are sent to the EHR but detailed metadata and raw outputs might remain in the LIS.
4.  **Outpatient Portal System:** Allows patients to schedule appointments, view summaries of their visits, and communicate with providers. Appointment data and patient-entered information are stored here.

**Prerequisites:**
*   Understanding of basic healthcare data types (demographics, encounters, medications, lab results).
*   Familiarity with the concept of data silos.

**Tools to be Used:**
*   This is a conceptual lab, primarily requiring analytical thinking and discussion. No specific Fabric tools are used for execution, but knowledge of Fabric's purpose is beneficial.

**Estimated Time:** 30 minutes

**Tasks:**

This lab is discussion-based. Consider the following questions based on the scenario:

**Questions & Detailed Answers:**

**1. List which key data elements are likely stored in each system at Valley General Hospital.**

*   **EHR System:**
    *   `Patient_ID`, `Patient_Name`, `Date_of_Birth`, `Gender`, `Address`, `Contact_Info`
    *   `Encounter_ID`, `Encounter_Date`, `Encounter_Type` (e.g., Inpatient, Outpatient, Emergency)
    *   `Provider_ID`, `Attending_Physician`
    *   `Diagnosis_Codes` (e.g., ICD-10), `Problem_List`
    *   `Medication_Orders` (prescribed medications, dosage, frequency)
    *   `Allergies_List`, `Adverse_Reactions`
    *   `Vital_Signs` (Height, Weight, Blood Pressure, Temperature)
    *   `Clinical_Notes` (Progress notes, consultation notes - often unstructured)
    *   `Immunization_Records`
    *   Pointers to lab results and radiology reports.

*   **Pharmacy System:**
    *   `Prescription_ID` (linked to EHR order)
    *   `Patient_ID`
    *   `Medication_NDC_Code` (National Drug Code)
    *   `Dispense_Date`, `Dispense_Quantity`, `Days_Supply`
    *   `Fill_Number`, `Refills_Remaining`
    *   `Pharmacist_ID`
    *   `Cost_Information`, `Insurance_Formulary_Status` (potentially)
    *   Inventory levels of medications.

*   **Laboratory Information System (LIS):**
    *   `Lab_Order_ID` (linked to EHR order)
    *   `Patient_ID`
    *   `Specimen_ID`, `Specimen_Type`, `Collection_Date_Time`
    *   `Test_Code` (e.g., LOINC), `Test_Name`
    *   `Result_Value` (quantitative or qualitative)
    *   `Reference_Range`, `Abnormal_Flags`
    *   `Performing_Lab_ID`, `Technician_ID`
    *   Pathology reports, microbiology culture details (can be extensive and semi-structured).

*   **Outpatient Portal System:**
    *   `Patient_ID`
    *   `Appointment_ID`, `Scheduled_Date_Time`, `Appointment_Status` (Scheduled, Confirmed, Cancelled, Completed)
    *   `Provider_ID`, `Clinic_Location`
    *   `Reason_for_Visit` (patient-stated)
    *   Secure messages exchanged between patient and provider.
    *   Patient-entered data (e.g., pre-visit questionnaires, symptom checkers).
    *   View logs (which patient viewed what information).

**2. What are the challenges if Valley General Hospital wants to develop a predictive model for patient readmission within 30 days of discharge?**

*   **Data Silos & Integration Complexity:**
    *   Crucial data for readmission prediction (e.g., discharge medications from EHR, actual dispensing from Pharmacy, post-discharge lab results from LIS, follow-up appointment adherence from Outpatient Portal) reside in separate systems.
    *   Combining this data requires complex, often manual, and potentially error-prone integration efforts. Each system might use different patient identifiers or data formats, requiring sophisticated mapping.

*   **Data Timeliness & Accessibility:**
    *   Real-time or near real-time data access is difficult. If the Pharmacy system updates dispensing records daily via batch, the model might not have the latest medication adherence information.
    *   Accessing data from multiple systems often involves different APIs, query languages, or export formats, increasing development time.

*   **Inconsistent Data Definitions & Standards:**
    *   The definition of "medication adherence" or "completed follow-up" might differ or not be explicitly captured across systems.
    *   While standards like HL7 or FHIR might be used for some interfaces, the internal storage and granularity can vary.

*   **Lack of a Unified Patient View (Patient 360):**
    *   Without a consolidated view, it's hard to see the complete patient journey leading up to a potential readmission. For example, did the patient pick up their discharge medications? Did they attend their follow-up appointment? Were there critical lab value changes post-discharge?

*   **Feature Engineering Difficulty:**
    *   Creating meaningful features for the model (e.g., number of prior admissions, specific medication classes, social determinants of health if captured) becomes challenging when data is fragmented. For instance, calculating the "number of hospitalizations in the last 6 months" requires querying and joining data that might span EHR and potentially older archived systems.

*   **Data Quality Issues:**
    *   Discrepancies between systems (e.g., a medication prescribed in EHR but never dispensed according to Pharmacy records) can lead to inaccurate model inputs.
    *   Missing data in one system that is critical for a feature can reduce the model's predictive power.

*   **Scalability & Performance:**
    *   Querying multiple disparate systems for large patient cohorts to train a model can be slow and resource-intensive.

**3. How can Microsoft Fabric’s OneLake and Data Factory help address these challenges?**

*   **OneLake as a Unified Data Store:**
    *   **Centralization:** OneLake acts as a single, logical data lake for the entire organization. Data from the EHR, Pharmacy, LIS, and Outpatient Portal can be ingested and stored in OneLake, eliminating physical silos.
    *   **Single Source of Truth:** By processing and conforming data into Bronze, Silver, and Gold layers within OneLake, Valley General can create a unified, reliable source for analytics and model training. This enables a true Patient 360 view.
    *   **Open Data Formats:** Storing data in open formats like Delta Parquet within OneLake means it's accessible by various compute engines in Fabric (Spark, SQL, Power BI via DirectLake) without data movement or duplication.

*   **Data Factory for Ingestion and Orchestration:**
    *   **Connectors:** Data Factory provides a wide range of connectors to pull data from various sources, including SQL databases (for EHR, LIS, Pharmacy backends), APIs (potentially for the Outpatient Portal or modern EHRs using FHIR), and file systems.
    *   **Data Integration Pipelines:** Data Factory can be used to build robust pipelines to extract, transform (if needed at a basic level for ingestion), and load (ETL/ELT) data from these disparate systems into the Bronze layer of OneLake.
    *   **Scheduling & Automation:** These pipelines can be scheduled to run at regular intervals (e.g., hourly, daily), ensuring that OneLake is consistently updated with the latest information, improving data timeliness for the readmission model.
    *   **Data Transformation (with Dataflows Gen2 or Notebooks):** While Data Factory orchestrates, it integrates seamlessly with Dataflows Gen2 (for low-code transformations) or Notebooks (for complex transformations using Spark/Python) to cleanse, standardize, and conform the ingested data into the Silver and Gold layers. This is where data from different sources can be joined and harmonized.

*   **Addressing Specific Challenges with Fabric:**
    *   **Integration Complexity:** Fabric provides the tools (Data Factory, Notebooks) to build these integrations once and then automate them.
    *   **Data Timeliness:** Scheduled pipelines ensure fresher data. Real-Time Analytics in Fabric could even handle streaming data if some sources support it.
    *   **Unified Patient View:** The Gold layer in OneLake, built using Fabric tools, would house the comprehensive data needed for the readmission model, effectively creating that Patient 360.
    *   **Feature Engineering:** With all relevant data in OneLake, data scientists can use Fabric Notebooks (with Spark or Python) to easily access and process this data to engineer complex features for the model.
    *   **Scalability & Performance:** Fabric's compute engines (Spark for Notebooks, SQL engine for the Warehouse) are designed for performance and can scale to handle large datasets for model training and scoring.

**Expected Outcome / Deliverables:**
*   A clear understanding of how data fragmentation in healthcare impacts analytical capabilities.
*   An appreciation for the role of a unified data platform like Microsoft Fabric in overcoming these challenges.

### Lab 2.9: Setting Up Your First Fabric Environment and Ingesting Sample Data

**Module Alignment:** Section 2: Setting Up the Environment

**Objective:**
*   Familiarize with the Microsoft Fabric portal and workspace creation.
*   Create a Lakehouse and ingest sample CSV data.
*   Explore the ingested data using a Fabric Notebook.
*   Understand basic role assignments within a workspace.

**Scenario:**
You are a data engineer at Valley General Hospital, newly onboarded to Microsoft Fabric. Your first task is to set up a development workspace for an upcoming cardiology analytics project and perform a test ingestion of sample patient data.

**Prerequisites:**
*   Access to a Microsoft Fabric enabled Microsoft 365 tenant.
*   Permissions to create workspaces in Fabric (typically Fabric Administrator or Power BI Administrator to enable Fabric for the tenant, then users with appropriate capacity permissions can create workspaces).
*   A sample CSV file with patient data. We will create one in the lab.

**Tools to be Used:**
*   Microsoft Fabric Portal
*   Fabric Workspace
*   Fabric Lakehouse
*   Fabric Notebook (PySpark)

**Estimated Time:** 45 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Create a Workspace**

1.  **Navigate to Microsoft Fabric:** Open your browser and go to `app.fabric.microsoft.com`.
2.  **Create a New Workspace:**
    *   In the left navigation pane, click on **Workspaces**.
    *   Click the **+ New workspace** button in the top right.
    *   **Name:** Enter `DEV_CardiologyAnalytics`.
    *   **Description:** (Optional) Enter "Development workspace for cardiology analytics projects".
    *   **Domain:** (Optional) Assign to a relevant domain if your organization uses them.
    *   **Capacity:** Assign the workspace to a Fabric capacity (e.g., a trial capacity or a provisioned F-SKU). This is crucial for using Fabric features.
    *   Click **Apply**.

**Part 2: Create a Lakehouse**

1.  **Open Your Workspace:** From the Workspaces list, click on `DEV_CardiologyAnalytics` to open it.
2.  **Create a New Lakehouse:**
    *   Within the workspace, click the **+ New** button.
    *   Select **Lakehouse** from the options.
    *   **Name:** Enter `CardiologyLakehouse`.
    *   Click **Create**.
    *   The Lakehouse explorer view will open. You'll see sections for `Tables` and `Files`.

**Part 3: Prepare and Upload Sample CSV Data**

1.  **Create Sample CSV Data:**
    *   Open a plain text editor (like Notepad or VS Code) on your local machine.
    *   Copy and paste the following data into the editor:
        ```csv
        PatientID,FirstName,LastName,DateOfBirth,Gender,LastVisitDate,Diagnosis
        P001,John,Doe,1985-06-15,Male,2023-01-10,Hypertension
        P002,Jane,Smith,1992-03-22,Female,2023-02-20,Diabetes Type 2
        P003,Robert,Jones,1978-11-05,Male,2022-12-05,Asthma
        P004,Emily,Brown,2001-07-30,Female,2023-03-15,Migraine
        P005,Michael,Davis,1965-09-12,Male,2023-01-25,Coronary Artery Disease
        ```
    *   Save the file as `sample_patients.csv` on your local machine.

2.  **Upload Data to Lakehouse (Files section):**
    *   In the `CardiologyLakehouse` explorer view, under the `Files` section, click the three dots (**...**) next to `Files` (or an existing folder if you prefer).
    *   Select **Upload** -> **Upload files**.
    *   Browse to your locally saved `sample_patients.csv` and select it.
    *   Confirm the upload. You should see `sample_patients.csv` appear under the `Files` section.

3.  **Load CSV to a Delta Table (using UI):**
    *   In the Lakehouse explorer, find the `sample_patients.csv` file under `Files`.
    *   Click the three dots (**...**) next to `sample_patients.csv`.
    *   Select **Load to table** -> **New table**.
    *   **Table name:** Enter `bronze_patients`.
    *   Fabric will infer the schema. Review it and click **Load**.
    *   A notification will appear once the table is created. You should see `bronze_patients` under the `Tables` section.

**Part 4: Explore Data with a Notebook**

1.  **Create a New Notebook:**
    *   Go back to your `DEV_CardiologyAnalytics` workspace view.
    *   Click **+ New** -> **Notebook**.
    *   A new notebook will open.

2.  **Attach Lakehouse to Notebook:**
    *   In the notebook interface, on the left side, you should see an "Explorer" pane.
    *   Click on **Add Lakehouse**.
    *   Select your `CardiologyLakehouse` and click **Add**. This makes the tables and files in your Lakehouse accessible to the notebook.

3.  **Write and Run PySpark Code:**
    *   In the first cell of the notebook, ensure the language is set to **PySpark (Python)**.
    *   Enter the following code to load and display the `bronze_patients` table:

        ```python
        # Read the Delta table from the Lakehouse
        df_patients = spark.read.table("CardiologyLakehouse.bronze_patients") # Or just "bronze_patients" if default lakehouse is set

        # Display the DataFrame schema and some data
        df_patients.printSchema()
        display(df_patients.limit(5)) # display() is a Fabric-specific function for rich table rendering

        # Perform a simple count
        patient_count = df_patients.count()
        print(f"Total number of patients: {patient_count}")
        ```
    *   Click the **Run cell** button (or Shift+Enter) to execute the code.

**Part 5: Assign Roles in Workspace (Conceptual - Requires Admin to actually assign)**

1.  **Navigate to Workspace Access Management:**
    *   Go to your `DEV_CardiologyAnalytics` workspace.
    *   In the top right corner of the workspace view, click on **Manage access** (or an icon representing access).
2.  **Add Users and Assign Roles:**
    *   You would typically see a list of current members. Click **+ Add people or groups**.
    *   Enter the email address of a team member.
    *   Assign a role:
        *   **Admin:** Full control (e.g., Data Engineering Lead).
        *   **Member:** Can view, edit, and publish content (e.g., another Data Engineer or Power BI Developer).
        *   **Contributor:** Can create items and publish reports but can't manage access or workspace settings (e.g., Data Analyst).
        *   **Viewer:** Can only view content (e.g., Clinical stakeholder).
    *   Click **Add**.
    *   *(Note: For this lab, you might not have other users to add, but understand the process.)*

**Expected Outcome / Deliverables:**
*   A Fabric workspace named `DEV_CardiologyAnalytics` is created.
*   A Lakehouse named `CardiologyLakehouse` exists within the workspace.
*   The `sample_patients.csv` data is uploaded to the Lakehouse `Files` section and loaded into a Delta table named `bronze_patients`.
*   A Fabric Notebook successfully reads and displays data from the `bronze_patients` table.
*   Understanding of how to conceptually assign different roles within a Fabric workspace.

**Questions from Manual & Answers - LINK TO HTML???:**

*   **Q1: What level of access should a data analyst be granted if they primarily need to build reports and perform data exploration but not manage the workspace or core data engineering pipelines?**
    *   **A1:** A **Contributor** role is often suitable. They can create notebooks, dataflows, and Power BI reports using existing data in the Lakehouse. If they only need to consume existing reports, **Viewer** would be more appropriate. If they need to publish reports and manage datasets they create, **Member** might be considered, but Contributor is a good starting point for report building without full workspace admin rights.

*   **Q2: Which Fabric tool is best for building visual, low-code/no-code ETL pipelines for ingesting data from various sources into the Lakehouse?**
    *   **A2:** **Dataflows Gen2** is the primary tool in Fabric for visual, low-code/no-code ETL/ELT pipeline creation. Data Factory pipelines orchestrate activities, which can include running Dataflows Gen2.

*   **Q3: How does OneLake enhance collaboration between different roles (e.g., Data Engineers, Data Scientists, BI Analysts) working on the same data?**
    *   **A3:** OneLake provides a **single, unified, and logical copy of the data** (stored in Delta Parquet format).
        *   **No Data Duplication:** Different roles and their preferred tools (Spark for Data Scientists/Engineers, SQL endpoint for Analysts, DirectLake for Power BI) can access the same underlying data in OneLake without creating multiple copies. This ensures everyone is working from the same version of truth.
        *   **Interoperability:** Data written by a Spark notebook is immediately queryable via the SQL endpoint and accessible in Power BI via DirectLake.
        *   **Shortcuts:** Data can be virtualized from other storage accounts or even other Fabric domains/workspaces, further enhancing collaboration without physically moving data.
        *   **Simplified Governance:** Managing security and governance is easier on a single data store.

### Lab 3.8: Ingesting Patient Encounter Data from HL7 and FHIR

**Module Alignment:** Section 3: Data Ingestion and Integration

**Objective:**
*   Build ingestion pipelines for patient encounter data from two distinct formats: HL7 v2.x batch files and FHIR JSON resources.
*   Parse and process these different data structures using PySpark in a Fabric Notebook.
*   Store the raw, processed data from each source into separate "Bronze" layer tables in the Lakehouse.
*   Conform and integrate the data from both sources into a unified `Silver_Unified_Encounters` table, providing a consolidated view of patient encounters for downstream Gold layer modeling.

**Scenario:**
Valley General Hospital needs to consolidate patient encounter information for comprehensive analytics. A significant portion of their historical and ongoing encounter data comes from legacy systems that output HL7 ADT (Admit, Discharge, Transfer) messages as batch files. Newer systems and integrations are increasingly using FHIR (Fast Healthcare Interoperability Resources) and expose encounter data as JSON resources, simulating an API feed. Your task is to ingest both types of data, process them, and create a unified view.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics` created in Lab 2.9).
*   Microsoft Fabric Lakehouse (e.g., `CardiologyLakehouse` created in Lab 2.9).
*   Familiarity with creating and running Fabric Notebooks.
*   Basic understanding of HL7 v2.x message structure (segments like MSH, PID, PV1) and FHIR JSON resource structure (specifically the `Encounter` resource).

**Tools to be Used:**
*   Microsoft Fabric Lakehouse
*   Microsoft Fabric Notebook (PySpark)

**Estimated Time:** 75 - 90 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Prepare and Upload Sample HL7 Data**

1.  **Create Sample HL7 ADT File:**
    *   On your local machine, open a plain text editor.
    *   Copy and paste the following sample HL7 data. This represents a batch file with two ADT messages.
        ```hl7
        MSH|^~\&|HIS_System|ValleyGeneral|LIS_System|ValleyGeneral|20230401100000||ADT^A01|MSG00001|P|2.3
        PID|1||PATID12345^^^ValleyGeneral^MR||Doe^John||19850615|M|||123 Main St^^Anytown^CA^90210||(555)123-4567
        PV1|1|I|EmergencyRoom^ER101^RoomA^^^ValleyGeneral||||AssignedDoctorID^Smith^Robert|||ADM|20230401094500
        MSH|^~\&|HIS_System|ValleyGeneral|LIS_System|ValleyGeneral|20230402113000||ADT^A03|MSG00002|P|2.3
        PID|1||PATID67890^^^ValleyGeneral^MR||Smith^Jane||19920322|F|||456 Oak Ave^^Anytown^CA^90210||(555)987-6543
        PV1|1|O|CardiologyClinic^CC202^RoomB^^^ValleyGeneral||||AssignedDoctorID^Jones^Emily|||DIS|20230402111500|20230402121500
        ```
    *   Save the file as `sample_adt_batch.hl7` on your local machine.

2.  **Upload HL7 File to Lakehouse:**
    *   Navigate to your `CardiologyLakehouse` in the `DEV_CardiologyAnalytics` workspace.
    *   Under the `Files` section, click the three dots (**...**) next to `Files`.
    *   Select **Upload** -> **Upload files**.
    *   Browse to your `sample_adt_batch.hl7` file and upload it.

**Part 2: Create a Notebook and Attach Lakehouse**

1.  **Create a New Notebook:**
    *   In your `DEV_CardiologyAnalytics` workspace, click **+ New** -> **Notebook**.
    *   Name it something like `Encounter_Ingestion_And_Conformation`.
2.  **Attach Lakehouse:**
    *   In the notebook's left "Explorer" pane, click **Add Lakehouse**.
    *   Select your `CardiologyLakehouse` and click **Add**.

**Part 3: Ingest and Process HL7 Data**

1.  **Add a new PySpark cell and enter the following code to read and parse the HL7 file:**
    ```python
    # Read the raw HL7 file as text
    hl7_file_path = "Files/sample_adt_batch.hl7" # Relative path in Lakehouse
    raw_hl7_rdd = spark.sparkContext.textFile(hl7_file_path)

    # Function to parse a single HL7 message (assuming messages are single lines for simplicity here,
    # or that MSH indicates a new message if they span lines - real HL7 parsing is more complex)
    # This is a very basic parser. Production HL7 parsing often uses dedicated libraries.
    def parse_hl7_message(message_str):
        segments = message_str.split('\n') # If messages are multi-line and segments are on new lines
        if not segments:
            segments = [message_str] # if it's a single line message string

        parsed_data = {}
        patient_id = None
        visit_id = None
        encounter_class = None
        admission_date = None
        discharge_date = None
        source_system = "HL7_HIS_System" # Example source

        for line in segments: # Assuming each line is a segment for this basic parser
            fields = line.split('|')
            segment_type = fields[0]

            if segment_type == "MSH":
                # MSH-2: Encoding Characters, MSH-3: Sending Application, MSH-9: Message Type
                parsed_data['msh_sending_app'] = fields[2] if len(fields) > 2 else None
                parsed_data['msh_message_type'] = fields[8] if len(fields) > 8 else None
            elif segment_type == "PID":
                # PID-3: Patient Identifier List (taking the first ID)
                if len(fields) > 3 and fields[3]:
                    patient_id_complex = fields[3].split('^')
                    patient_id = patient_id_complex[0]
                # PID-5: Patient Name (LastName^FirstName)
                parsed_data['patient_name'] = fields[5] if len(fields) > 5 else None
                # PID-7: Date/Time of Birth
                parsed_data['dob'] = fields[7] if len(fields) > 7 else None
                # PID-8: Administrative Sex
                parsed_data['gender'] = fields[8] if len(fields) > 8 else None
            elif segment_type == "PV1":
                # PV1-1: Set ID - PV1 (can be used as part of a visit identifier)
                visit_id_segment = fields[1] if len(fields) > 1 else "UNKNOWN_VISIT"
                visit_id = f"HL7_{patient_id}_{visit_id_segment}" if patient_id else f"HL7_UNKNOWN_{visit_id_segment}"

                # PV1-2: Patient Class (I=Inpatient, O=Outpatient, E=Emergency)
                encounter_class = fields[2] if len(fields) > 2 else None
                # PV1-18: Patient Type (not always populated, but an example)
                # PV1-44: Admission Date/Time
                admission_date = fields[44] if len(fields) > 44 else None
                # PV1-45: Discharge Date/Time
                discharge_date = fields[45] if len(fields) > 45 else None
        
        if patient_id and visit_id: # Only return if we have key identifiers
            return (visit_id, patient_id, encounter_class, admission_date, discharge_date, source_system, message_str)
        return None

    # Group lines by message (MSH indicates start of a new message)
    # A more robust way would be to read the file content and split by MSH
    file_content = ""
    for item in raw_hl7_rdd.collect():
        file_content += item + "\n" # Assuming each item in RDD is a line

    messages = file_content.strip().split('MSH|^~\\&|') # Split by MSH, handling the first one
    parsed_messages = []
    for msg_body in messages:
        if msg_body.strip(): # If not an empty string
            full_message = "MSH|^~\\&|" + msg_body.strip() # Add back the MSH prefix
            parsed_result = parse_hl7_message(full_message)
            if parsed_result:
                parsed_messages.append(parsed_result)

    # Create DataFrame
    if parsed_messages:
        hl7_columns = ["encounter_id", "patient_id", "encounter_class_raw", "admission_date_raw", "discharge_date_raw", "source_system", "raw_message"]
        df_hl7_bronze = spark.createDataFrame(parsed_messages, hl7_columns)
        
        df_hl7_bronze.printSchema()
        display(df_hl7_bronze)

        # Write to Bronze table in Lakehouse
        df_hl7_bronze.write.format("delta").mode("overwrite").saveAsTable("CardiologyLakehouse.bronze_hl7_encounters")
        print("bronze_hl7_encounters table created successfully.")
    else:
        print("No HL7 messages were successfully parsed.")

    ```
2.  **Run the cell.** This code reads the HL7 file, attempts a basic parse of PID and PV1 segments for each message, and stores the extracted fields along with the raw message into a Delta table named `bronze_hl7_encounters`.
    *   *Note: Production-grade HL7 parsing is significantly more complex and typically uses dedicated libraries like `hl7apy` or `hapi-fhir` (if converting to FHIR). This parser is simplified for the lab's scope.*

**Part 4: Prepare and Ingest Sample FHIR Data**

1.  **Create Sample FHIR Encounter JSON Data:**
    *   In a new PySpark cell, define a Python list containing sample FHIR Encounter JSON strings:
    ```python
    fhir_encounter_json_list = [
        """
        {
          "resourceType": "Encounter",
          "id": "FHIRENC001",
          "status": "finished",
          "class": {
            "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
            "code": "AMB",
            "display": "ambulatory"
          },
          "subject": {
            "reference": "Patient/PATID12345",
            "display": "John Doe"
          },
          "period": {
            "start": "2023-05-10T10:00:00Z",
            "end": "2023-05-10T11:00:00Z"
          },
          "serviceProvider": {
            "reference": "Organization/ValleyGeneral"
          }
        }
        """,
        """
        {
          "resourceType": "Encounter",
          "id": "FHIRENC002",
          "status": "finished",
          "class": {
            "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
            "code": "IMP",
            "display": "inpatient encounter"
          },
          "subject": {
            "reference": "Patient/PATID99001",
            "display": "Alice Wonderland"
          },
          "period": {
            "start": "2023-05-12T14:30:00Z",
            "end": "2023-05-15T10:00:00Z"
          },
           "serviceProvider": {
            "reference": "Organization/ValleyGeneral"
          }
        }
        """,
        """
        {
          "resourceType": "Encounter",
          "id": "FHIRENC003",
          "status": "in-progress",
          "class": {
            "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
            "code": "EMER",
            "display": "emergency"
          },
          "subject": {
            "reference": "Patient/PATID67890",
            "display": "Jane Smith"
          },
          "period": {
            "start": "2023-05-18T08:15:00Z"
          },
           "serviceProvider": {
            "reference": "Organization/ValleyGeneral"
          }
        }
        """
    ]
    ```

2.  **Process FHIR JSON and Store in Bronze Table:**
    *   In the same cell or a new one, add the following PySpark code:
    ```python
    from pyspark.sql.functions import col, from_json, explode
    from pyspark.sql.types import StringType, StructType, StructField, ArrayType

    # Create an RDD from the list of JSON strings
    fhir_rdd = spark.sparkContext.parallelize(fhir_encounter_json_list)

    # Convert RDD of JSON strings to a DataFrame with a single column "raw_json"
    df_raw_fhir = fhir_rdd.map(lambda x: (x,)).toDF(["raw_json"])

    # Define a schema for the FHIR Encounter resource (simplified for key fields)
    fhir_encounter_schema = StructType([
        StructField("id", StringType(), True),
        StructField("status", StringType(), True),
        StructField("class", StructType([
            StructField("code", StringType(), True),
            StructField("display", StringType(), True)
        ]), True),
        StructField("subject", StructType([
            StructField("reference", StringType(), True)
        ]), True),
        StructField("period", StructType([
            StructField("start", StringType(), True),
            StructField("end", StringType(), True)
        ]), True)
    ])

    # Parse the JSON strings
    df_fhir_parsed = df_raw_fhir.withColumn("parsed_json", from_json(col("raw_json"), fhir_encounter_schema))

    # Select and flatten the required fields
    df_fhir_bronze = df_fhir_parsed.select(
        col("parsed_json.id").alias("encounter_id_fhir"),
        col("parsed_json.status").alias("status"),
        col("parsed_json.class.code").alias("encounter_class_code"),
        col("parsed_json.class.display").alias("encounter_class_display"),
        col("parsed_json.subject.reference").alias("patient_id_fhir"),
        col("parsed_json.period.start").alias("admission_date_fhir"),
        col("parsed_json.period.end").alias("discharge_date_fhir"),
        col("raw_json").alias("raw_message")
    ).withColumn("source_system", lit("FHIR_API_Sim"))


    df_fhir_bronze.printSchema()
    display(df_fhir_bronze)

    # Write to Bronze table in Lakehouse
    df_fhir_bronze.write.format("delta").mode("overwrite").saveAsTable("CardiologyLakehouse.bronze_fhir_encounters")
    print("bronze_fhir_encounters table created successfully.")
    ```
3.  **Run the cell.** This code parses the JSON strings, extracts relevant fields, and stores them in a Delta table named `bronze_fhir_encounters`.

**Part 5: Conform HL7 and FHIR Encounter Data into a Unified Silver Table**

1.  **Define Common Schema and Transform Data:**
    *   In a new PySpark cell, add the following code:
    ```python
    from pyspark.sql.functions import col, lit, when, regexp_replace, to_timestamp, substring

    # Load the bronze tables
    df_hl7 = spark.read.table("CardiologyLakehouse.bronze_hl7_encounters")
    df_fhir = spark.read.table("CardiologyLakehouse.bronze_fhir_encounters")

    # --- Transform HL7 Data ---
    # Clean patient_id (example: PATID12345^^^ValleyGeneral^MR -> PATID12345)
    # HL7 dates are often YYYYMMDDHHMMSS, convert to ISO 8601 format timestamp
    df_hl7_transformed = df_hl7.select(
        col("encounter_id").alias("encounter_id"),
        regexp_replace(col("patient_id"), r"\^\^\^.*", "").alias("patient_id"), # Basic cleaning
        when(col("encounter_class_raw") == "I", "inpatient")
            .when(col("encounter_class_raw") == "O", "outpatient")
            .when(col("encounter_class_raw") == "E", "emergency")
            .otherwise(lower(col("encounter_class_raw"))).alias("encounter_class"),
        # Convert YYYYMMDDHHMMSS to YYYY-MM-DD HH:MM:SS for to_timestamp
        when(length(col("admission_date_raw")) >= 8, 
             to_timestamp(
                concat(
                    substring(col("admission_date_raw"), 1, 4), lit("-"),
                    substring(col("admission_date_raw"), 5, 2), lit("-"),
                    substring(col("admission_date_raw"), 7, 2), lit(" "),
                    substring(col("admission_date_raw"), 9, 2), lit(":"),
                    substring(col("admission_date_raw"), 11, 2), lit(":"),
                    substring(col("admission_date_raw"), 13, 2)
                ), "yyyy-MM-dd HH:mm:ss")
            ).otherwise(None).alias("admission_date"),
        when(length(col("discharge_date_raw")) >= 8,
             to_timestamp(
                concat(
                    substring(col("discharge_date_raw"), 1, 4), lit("-"),
                    substring(col("discharge_date_raw"), 5, 2), lit("-"),
                    substring(col("discharge_date_raw"), 7, 2), lit(" "),
                    substring(col("discharge_date_raw"), 9, 2), lit(":"),
                    substring(col("discharge_date_raw"), 11, 2), lit(":"),
                    substring(col("discharge_date_raw"), 13, 2)
                ), "yyyy-MM-dd HH:mm:ss")
            ).otherwise(None).alias("discharge_date"),
        col("source_system"),
        col("raw_message") # Keep raw message for lineage/auditing
    )

    # --- Transform FHIR Data ---
    # Clean patient_id (example: Patient/PATID12345 -> PATID12345)
    # FHIR dates are ISO 8601 (YYYY-MM-DDTHH:MM:SSZ), convert to timestamp
    df_fhir_transformed = df_fhir.select(
        col("encounter_id_fhir").alias("encounter_id"),
        regexp_replace(col("patient_id_fhir"), "Patient/", "").alias("patient_id"), # Basic cleaning
        lower(col("encounter_class_display")).alias("encounter_class"), # Using display, or map code
        to_timestamp(col("admission_date_fhir")).alias("admission_date"),
        to_timestamp(col("discharge_date_fhir")).alias("discharge_date"),
        col("source_system"),
        col("raw_message") # Keep raw message for lineage/auditing
    )
    
    # --- Union the transformed DataFrames ---
    df_unioned_encounters = df_hl7_transformed.unionByName(df_fhir_transformed, allowMissingColumns=True)

    # Add a unique surrogate key for the silver table, rename columns for Gold layer compatibility, and add audit timestamp
    from pyspark.sql.functions import monotonically_increasing_id, current_timestamp

    df_silver_unified_encounters = df_unioned_encounters.withColumn(
        "EncounterSK", monotonically_increasing_id()
    ).select(
        col("EncounterSK"),
        col("encounter_id").alias("SourceSystemEncounterID"),
        col("patient_id").alias("PatientIdentifier"),
        col("encounter_class").alias("EncounterType"),
        col("admission_date").alias("AdmissionDateTime"),
        col("discharge_date").alias("DischargeDateTime"),
        lit(None).cast("string").alias("PrimaryDiagnosisCode"), # Placeholder - to be populated if parsing is enhanced
        lit(None).cast("string").alias("AttendingProviderIdentifier"), # Placeholder - to be populated if parsing is enhanced
        col("source_system").alias("SourceSystem"),
        col("raw_message").alias("RawMessage"),
        current_timestamp().alias("SilverLoadTimestamp")
    )

    # Optional: Deduplication logic can be added here if necessary
    # For this lab, we'll assume basic union is sufficient.
    
    df_silver_unified_encounters.printSchema()
    display(df_silver_unified_encounters)

    # Write to Silver table in Lakehouse
    df_silver_unified_encounters.write.format("delta").mode("overwrite").option("overwriteSchema", "true").saveAsTable("CardiologyLakehouse.Silver_Unified_Encounters")
    print("Silver_Unified_Encounters table created successfully.")
    ```
2.  **Run the cell.** This code loads the bronze tables, applies transformations to standardize column names and data types, maps encounter class codes, unions the data, adds a surrogate key and audit timestamp, and renames/adds columns to align with downstream Gold layer expectations. The result is saved to `Silver_Unified_Encounters`.

**Expected Outcome / Deliverables:**
*   A `bronze_hl7_encounters` Delta table in `CardiologyLakehouse` containing parsed data from the HL7 file.
*   A `bronze_fhir_encounters` Delta table in `CardiologyLakehouse` containing parsed data from the FHIR JSON.
*   A `Silver_Unified_Encounters` Delta table in `CardiologyLakehouse` containing a unified and conformed view of patient encounters. This table should have key columns like `EncounterSK`, `SourceSystemEncounterID`, `PatientIdentifier`, `EncounterType`, `AdmissionDateTime`, `DischargeDateTime`, `PrimaryDiagnosisCode` (placeholder), `AttendingProviderIdentifier` (placeholder), `SourceSystem`, `RawMessage`, and `SilverLoadTimestamp`.

**Questions from Manual & Answers: - LINK TO HTML???**

*   **Q1: What are some common fields in HL7 (PID, PV1 segments) and FHIR (Encounter resource) that represent patient identity and encounter details?**
    *   **A1:**
        *   **HL7:**
            *   `PID-3` (Patient Identifier List): Contains Medical Record Numbers (MRN), account numbers.
            *   `PID-5` (Patient Name): Last Name, First Name.
            *   `PID-7` (Date/Time of Birth).
            *   `PV1-2` (Patient Class): e.g., 'I' for Inpatient, 'O' for Outpatient, 'E' for Emergency.
            *   `PV1-19` (Visit Number): An identifier for the visit.
            *   `PV1-44` (Admit Date/Time).
            *   `PV1-45` (Discharge Date/Time).
        *   **FHIR Encounter Resource:**
            *   `Encounter.id`: Logical ID for the encounter resource.
            *   `Encounter.identifier`: Business identifiers for the encounter (e.g., Visit Number).
            *   `Encounter.status`: e.g., planned, arrived, in-progress, finished, cancelled.
            *   `Encounter.class`: Similar to HL7 PV1-2 (e.g., AMB, IMP, EMER).
            *   `Encounter.subject`: Reference to the Patient resource (contains patient ID).
            *   `Encounter.period.start`: Admission/start date-time of the encounter.
            *   `Encounter.period.end`: Discharge/end date-time of the encounter.

*   **Q2: Why do we typically keep a "Bronze" layer (raw or minimally processed data) even after transforming data into a "Silver" (conformed) layer?**
    *   **A2:**
        *   **Auditability and Traceability:** The Bronze layer serves as an immutable record of the data as it was received from the source system. This is crucial for auditing, debugging data quality issues, and understanding data lineage. If discrepancies are found in the Silver or Gold layers, you can always trace back to the original raw data.
        *   **Reprocessing Capabilities:** If transformation logic changes or errors are discovered in the Silver layer processing, the Bronze layer allows you to reprocess the data from its original state without needing to re-ingest from the source systems, which might be slow, costly, or no longer possible for historical data.
        *   **Schema Evolution of Source Systems:** Source systems might change their schema or output format. The Bronze layer captures this raw data, and transformations to Silver can be adjusted accordingly over time, preserving historical raw formats.
        *   **Compliance and Data Retention:** For certain regulatory requirements, organizations might need to store data in its original received format for a specific period.
        *   **Exploratory Analysis:** Data scientists or analysts might want to explore the raw data to understand nuances or discover new patterns that might be lost or altered during transformation to the Silver layer.

*   **Q3: What governance logging would you ideally apply when ingesting data from external sources like HL7 batch files or FHIR APIs?**
    *   **A3:**
        *   **Source Information:**
            *   For files: File name, file path, file size, file hash (checksum like MD5 or SHA256) to ensure integrity.
            *   For APIs: API endpoint URL, request parameters (if any, being mindful of PII in URLs).
        *   **Timestamp of Ingestion:** The exact date and time the ingestion process started and finished for each batch or API call.
        *   **User/Service Principal Identity:** The user account or service principal that initiated or performed the ingestion.
        *   **Data Volume:** Number of messages, records, or bytes processed/ingested.
        *   **Status of Ingestion:** Success, failure, or partial success. If failed, include error messages or codes.
        *   **Job/Pipeline Run ID:** A unique identifier for the specific execution run of the ingestion pipeline for traceability.
        *   **Schema Version (if applicable):** If the source data schema is versioned, log the version ingested.
        *   **Data Classification/Sensitivity (if known at ingestion):** Tagging the data with its sensitivity level (e.g., PHI) as early as possible.
        *   **Lineage Metadata:** Information linking the raw ingested data (Bronze) to its source system and any subsequent transformations (to Silver/Gold). Microsoft Purview can help automate some ofthis.

---
**This lab is more involved, especially the parsing and transformation logic. The provided PySpark code is illustrative and would need to be more robust for production (e.g., better error handling, more comprehensive HL7 parsing, dynamic schema handling for FHIR).**

### Lab 4.8: Modeling Patient Encounters for Analytics (Gold Layer)

**Module Alignment:** Section 4: Data Modeling and Transformation

**Objective:**
*   Design and implement the transformation of patient and encounter data from Bronze/Silver layers into a Gold layer dimensional model.
*   Create `Dim_Patient` from a `Bronze_Patients_Raw` table, including de-duplication and surrogate key generation.
*   Create a simplified `Dim_Date` table.
*   Create `Fact_Encounter` by joining `Silver_Unified_Encounters` with the new `Dim_Patient` and `Dim_Date`.
*   (Optional) Create a SQL view (`Patient_Visit_Analytics_View`) on the Gold layer tables for BI consumption.
*   Utilize PySpark in a Fabric Notebook for these transformations.

**Scenario:**
Valley General Hospital needs a robust dimensional model in their Gold layer for advanced analytics and reporting on patient encounters. This involves creating dimension tables for Patients and Dates, and a Fact table for Encounters, sourcing data from previously established Bronze and Silver layers. This lab focuses on building these foundational Gold layer components.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `HealthDataLH_YourName` - replace `YourName` as appropriate).
*   `Bronze_Patients_Raw` table (sample data creation is included as part of this lab's notebook).
*   `Silver_Unified_Encounters` table (assumed to be created from a prior lab, e.g., Lab 3.8, with columns like `SourceSystemEncounterID`, `PatientIdentifier`, `AdmissionDateTime`, `DischargeDateTime`, `EncounterType`, `EncounterSK`).
*   Familiarity with creating and running Fabric Notebooks.
*   Basic understanding of dimensional modeling concepts (facts, dimensions, surrogate keys).

**Tools to be Used:**
*   Microsoft Fabric Lakehouse
*   Microsoft Fabric Notebook (PySpark)
*   SQL Analytics Endpoint (for optional view creation)

**Note:** This lab's hands-on exercises are performed using the Jupyter Notebook: `labs/gem-Labs-Section-4-Lab-4_8_Modeling_Patient_Encounters_for_Analytics.ipynb`. The following sections in this manual describe the concepts and steps undertaken in the notebook.

**Estimated Time:** 75 - 90 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Prepare Bronze Patient Data and Verify Silver Encounters**

1.  **Create Sample `Bronze_Patients_Raw` Data (if not already present):**
    *   The notebook `labs/gem-Labs-Section-4-Lab-4_8_Modeling_Patient_Encounters_for_Analytics.ipynb` includes a cell to create sample patient data in `Bronze_Patients_Raw`. This simulates data typically ingested from an EHR or FHIR Patient resources.
    *   **Code Snippet (from Notebook):**
    ```python
    from pyspark.sql import Row
    import datetime

    lakehouse_name = "HealthDataLH_YourName" # Replace YourName
    bronze_patients_table = f"{lakehouse_name}.Bronze_Patients_Raw"

    # Sample patient data
    patient_data = [
        Row(PatientSourceID="PATID12345", MRN="MRN001", SSN_Token="TOKEN_SSN1", FirstName="John", LastName="Doe", DateOfBirth=datetime.date(1980, 1, 1), Gender="M", Race="WH", Ethnicity="N", ZipCode="90210", SourceSystem="EHR_SystemA", IngestionTimestamp=datetime.datetime.now()),
        Row(PatientSourceID="PATID67890", MRN="MRN002", SSN_Token="TOKEN_SSN2", FirstName="Jane", LastName="Roe", DateOfBirth=datetime.date(1975, 3, 15), Gender="F", Race="AS", Ethnicity="N", ZipCode="90211", SourceSystem="EHR_SystemA", IngestionTimestamp=datetime.datetime.now()),
        Row(PatientSourceID="PATFHIR001", MRN="MRN003", SSN_Token="TOKEN_SSN3", FirstName="Walter", LastName="White", DateOfBirth=datetime.date(1965, 9, 7), Gender="M", Race="WH", Ethnicity="N", ZipCode="87101", SourceSystem="FHIR_API", IngestionTimestamp=datetime.datetime.now()),
        Row(PatientSourceID="EXTRALONGPATIENTID004", MRN="MRN004", SSN_Token="TOKEN_SSN4", FirstName="Sarah", LastName="Connor", DateOfBirth=datetime.date(1985, 5, 10), Gender="F", Race="WH", Ethnicity="H", ZipCode="90001", SourceSystem="EHR_SystemB", IngestionTimestamp=datetime.datetime.now()),
        Row(PatientSourceID="PATID12345", MRN="MRN001", SSN_Token="TOKEN_SSN1", FirstName="John", LastName="Doe", DateOfBirth=datetime.date(1980, 1, 1), Gender="M", Race="WH", Ethnicity="N", ZipCode="90210", SourceSystem="EHR_SystemA_OldRecord", IngestionTimestamp=datetime.datetime.now()-datetime.timedelta(days=10)) # Duplicate for testing dedupe
    ]
    patients_df = spark.createDataFrame(patient_data)
    patients_df.write.format("delta").mode("overwrite").saveAsTable(bronze_patients_table)
    print(f"Sample data written to {bronze_patients_table}")
    # spark.read.table(bronze_patients_table).show() # Display in notebook
    ```
2.  **Verify `Silver_Unified_Encounters` Table:**
    *   Ensure the `Silver_Unified_Encounters` table (from Lab 3.8 or equivalent) exists in your `HealthDataLH_YourName` Lakehouse. This table is crucial input for `Fact_Encounter`.

**Part 2: Create `Dim_Patient` Dimension Table (Gold Layer)**

*   This dimension table will store unique, de-duplicated patient information with a surrogate key.
1.  **Generate `Dim_Patient`:**
    *   In a PySpark cell (as per the notebook `Create_Dim_Patient_Gold_YourName` section):
    ```python
    from pyspark.sql.functions import col, lit, current_timestamp, expr, monotonically_increasing_id, md5, concat_ws, year, month, dayofmonth, datediff, current_date, coalesce
    from pyspark.sql.window import Window # Corrected import from notebook
    import datetime

    lakehouse_name = "HealthDataLH_YourName" # Replace YourName
    bronze_patients_table = f"{lakehouse_name}.Bronze_Patients_Raw"
    dim_patient_table = f"{lakehouse_name}.Dim_Patient" # Gold Layer Table

    raw_patients_df = spark.read.table(bronze_patients_table)

    # --- De-duplicate patient records ---
    window_spec_patient = Window.partitionBy("MRN").orderBy(col("IngestionTimestamp").desc()) # Corrected Window import

    deduplicated_patients_df = raw_patients_df.withColumn("row_num", expr("row_number() OVER (PARTITION BY MRN ORDER BY IngestionTimestamp DESC)")) \
        .filter(col("row_num") == 1) \
        .drop("row_num")

    # --- Select and Transform for Dimension Table ---
    dim_patient_df = deduplicated_patients_df.select(
        col("MRN").alias("PatientNaturalKey"),
        col("PatientSourceID").alias("SourcePatientID"),
        col("FirstName"),
        col("LastName"),
        col("DateOfBirth").cast("date"),
        col("Gender"),
        col("Race"),
        col("Ethnicity"),
        col("ZipCode"),
        col("SourceSystem").alias("PatientSourceSystem")
    ).withColumn(
        "PatientKey", md5(col("PatientNaturalKey"))
    ).withColumn(
        "FullName", expr("concat(FirstName, ' ', LastName)")
    ).withColumn(
        "Age", expr("floor(datediff(current_date(), DateOfBirth) / 365.25)").cast("int")
    ).withColumn(
        "GoldLoadTimestamp", current_timestamp()
    ).withColumn(
        "EffectiveStartDate", lit(datetime.date(1900, 1, 1)).cast("date")
    ).withColumn(
        "EffectiveEndDate", lit(None).cast("date")
    ).withColumn(
        "IsCurrent", lit(True).cast("boolean")
    )

    final_dim_patient_df = dim_patient_df.select(
        "PatientKey", "PatientNaturalKey", "SourcePatientID", "FirstName", "LastName", "FullName", "DateOfBirth", "Age", "Gender",
        "Race", "Ethnicity", "ZipCode", "PatientSourceSystem",
        "EffectiveStartDate", "EffectiveEndDate", "IsCurrent", "GoldLoadTimestamp"
    )

    final_dim_patient_df.write.format("delta").mode("overwrite").saveAsTable(dim_patient_table)
    print(f"Successfully created/updated {dim_patient_table}")
    # final_dim_patient_df.show(truncate=False) # Display in notebook
    ```
2.  **Run the cell.** This creates a `Dim_Patient` table with a unique surrogate key (`PatientKey`) for each distinct `MRN`. It includes demographic attributes and SCD Type 2 columns.

**Part 3: Create `Fact_Encounter` Fact Table (Gold Layer)**

*   This fact table will link to dimension tables and store measures related to patient encounters.
1.  **Generate `Fact_Encounter` (and simplified `Dim_Date` if needed):**
    *   In a PySpark cell (as per the notebook `Create_Fact_Encounter_Gold_YourName` section):
    ```python
    from pyspark.sql.functions import col, lit, current_timestamp, expr, to_date, datediff, year, month, dayofmonth, coalesce, when, explode
    import datetime

    lakehouse_name = "HealthDataLH_YourName" # Replace YourName
    silver_unified_encounters_table = f"{lakehouse_name}.Silver_Unified_Encounters"
    dim_patient_table = f"{lakehouse_name}.Dim_Patient"
    fact_encounter_table = f"{lakehouse_name}.Fact_Encounter"
    dim_date_table = f"{lakehouse_name}.Dim_Date"

    # --- (Optional) Create a simple Dim_Date if not existing ---
    try:
        spark.read.table(dim_date_table).limit(1).collect()
        print(f"{dim_date_table} already exists.")
    except:
        print(f"Creating simplified {dim_date_table}...")
        date_range_df = spark.sql("SELECT sequence(to_date('2020-01-01'), to_date('2030-12-31'), interval 1 day) as date_array")
        dates_df = date_range_df.select(explode(col("date_array")).alias("FullDate"))
        simple_dim_date_df = dates_df.select(
            expr("replace(cast(FullDate as string), '-', '')").cast("int").alias("DateKey"),
            col("FullDate").cast("date"),
            year(col("FullDate")).alias("Year"),
            month(col("FullDate")).alias("Month"),
            dayofmonth(col("FullDate")).alias("Day")
        )
        simple_dim_date_df.write.format("delta").mode("overwrite").saveAsTable(dim_date_table)
        print(f"Simplified {dim_date_table} created.")

    encounters_df = spark.read.table(silver_unified_encounters_table)
    patients_dim_df = spark.read.table(dim_patient_table).select("PatientKey", "PatientNaturalKey")
    date_dim_df = spark.read.table(dim_date_table).select("DateKey", "FullDate")

    fact_df_intermediate = encounters_df.join(
        patients_dim_df,
        encounters_df.PatientIdentifier == patients_dim_df.PatientNaturalKey,
        "left_outer"
    ).select(encounters_df["*"], patients_dim_df["PatientKey"])

    fact_df_intermediate = fact_df_intermediate.join(
        date_dim_df.alias("dim_admission_date"),
        to_date(fact_df_intermediate.AdmissionDateTime) == col("dim_admission_date.FullDate"),
        "left_outer"
    ).select(fact_df_intermediate["*"], col("dim_admission_date.DateKey").alias("AdmissionDateKey"))

    fact_df_intermediate = fact_df_intermediate.join(
        date_dim_df.alias("dim_discharge_date"),
        to_date(fact_df_intermediate.DischargeDateTime) == col("dim_discharge_date.FullDate"),
        "left_outer"
    ).select(fact_df_intermediate["*"], col("dim_discharge_date.DateKey").alias("DischargeDateKey"))

    fact_encounter_df = fact_df_intermediate.withColumn(
        "LengthOfStayInDays",
        when(col("DischargeDateTime").isNotNull() & col("AdmissionDateTime").isNotNull(),
             datediff(to_date(col("DischargeDateTime")), to_date(col("AdmissionDateTime")))
        ).otherwise(None).cast("int")
    ).select(
        col("EncounterSK").alias("EncounterKey"),
        col("PatientKey"),
        col("AdmissionDateKey"),
        col("DischargeDateKey"),
        col("SourceSystemEncounterID").alias("EncounterNaturalKey"),
        col("EncounterType"),
        col("PrimaryDiagnosisCode"), # Assuming this exists in Silver_Unified_Encounters
        col("AttendingProviderIdentifier"), # Assuming this exists
        col("LengthOfStayInDays"),
        col("SourceSystem"),
        current_timestamp().alias("GoldLoadTimestamp")
    ).filter(col("PatientKey").isNotNull())

    fact_encounter_df.write.format("delta").mode("overwrite").saveAsTable(fact_encounter_table)
    print(f"Successfully created/updated {fact_encounter_table}")
    # fact_encounter_df.show(truncate=False) # Display in notebook
    ```
2.  **Run the cell.** This creates `Fact_Encounter` by joining `Silver_Unified_Encounters` with `Dim_Patient` and `Dim_Date`, and calculates `LengthOfStayInDays`.

**Part 4: (Optional) Create `Patient_Visit_Analytics_View` (Gold Layer View)**

*   This view provides a denormalized perspective for easier BI reporting.
1.  **Create SQL View:**
    *   Using the SQL Analytics Endpoint for `HealthDataLH_YourName`, run the following SQL:
    ```sql
    -- Ensure you are in the context of your Lakehouse, e.g., USE HealthDataLH_YourName;
    -- Or fully qualify table names if needed: HealthDataLH_YourName.dbo.Fact_Encounter

    CREATE OR ALTER VIEW Patient_Visit_Analytics_View AS
    SELECT
        p.PatientNaturalKey AS PatientMRN,
        p.FullName AS PatientFullName,
        p.DateOfBirth AS PatientDateOfBirth,
        p.Age AS PatientAge,
        p.Gender AS PatientGender,
        fe.EncounterNaturalKey,
        fe.EncounterType,
        fe.PrimaryDiagnosisCode,
        adm_dt.FullDate AS AdmissionDate,
        dis_dt.FullDate AS DischargeDate,
        fe.LengthOfStayInDays,
        fe.AttendingProviderIdentifier,
        fe.SourceSystem AS EncounterSourceSystem
        -- Add more fields from dimensions (Dim_Provider, Dim_Diagnosis) as needed
    FROM
        HealthDataLH_YourName.dbo.Fact_Encounter fe
    JOIN
        HealthDataLH_YourName.dbo.Dim_Patient p ON fe.PatientKey = p.PatientKey
    LEFT JOIN
        HealthDataLH_YourName.dbo.Dim_Date adm_dt ON fe.AdmissionDateKey = adm_dt.DateKey
    LEFT JOIN
        HealthDataLH_YourName.dbo.Dim_Date dis_dt ON fe.DischargeDateKey = dis_dt.DateKey
    ;

    -- Test the view
    -- SELECT * FROM Patient_Visit_Analytics_View LIMIT 10;
    ```

**Expected Outcome / Deliverables:**
*   A `Dim_Patient` Delta table in `HealthDataLH_YourName` with patient surrogate keys, natural keys, demographic attributes, and SCD Type 2 columns.
*   A simplified `Dim_Date` Delta table in `HealthDataLH_YourName`.
*   A `Fact_Encounter` Delta table in `HealthDataLH_YourName` linking to dimension tables via surrogate keys and containing measures like `LengthOfStayInDays`.
*   (Optional) A SQL view `Patient_Visit_Analytics_View` providing a denormalized look at the data.
*   An understanding of how these Gold layer tables form a star schema for analytics.

**Discussion Questions:**

1.  **In `Dim_Patient`, why is it generally better to use a system-generated `PatientKey` (surrogate key) as the primary key instead of using the `PatientIdentifier` (natural key from the source system) directly?**
    *   **Stability:** Natural keys (like MRN) can sometimes change, be reassigned, or have errors in the source system. Surrogate keys are controlled within the data warehouse and remain stable, protecting the fact tables from these source system changes.
    *   **Performance:** Surrogate keys are typically integers (or a fixed-length hash like in our example), which are more efficient for joins in database systems compared to potentially long or composite string-based natural keys.
    *   **Decoupling:** Using surrogate keys decouples the data warehouse model from the operational source system's keying structure. This allows the source system to evolve its keys without breaking the warehouse.
    *   **Handling Slowly Changing Dimensions (SCDs):** Surrogate keys are essential for implementing SCD Type 2, where you need to track historical changes to dimension attributes. A new surrogate key is assigned for each version of a patient's record.
    *   **Integration of Multiple Sources:** If patients come from multiple source systems with different natural key formats or potential overlaps, a surrogate key provides a single, unambiguous way to identify a unique patient entity in the warehouse.

2.  **How can Microsoft Purview (discussed in later sections) help in understanding the lineage of data as it moves from `Bronze_HL7_Encounters_Raw` (or `Bronze_Patients_Raw`) to `Silver_Unified_Encounters` and finally into `Fact_Encounter` and `Dim_Patient`?**
    *   **Automated Lineage Tracking:** Microsoft Purview can scan Fabric items (Lakehouses, Notebooks, Data Factory pipelines) and automatically capture data lineage. It visualizes how data flows from source tables through transformation processes (e.g., the Spark Notebooks) to the final analytical tables and even into Power BI reports.
    *   **Impact Analysis:** If there's a change proposed to a Bronze table or a transformation rule, lineage helps identify all downstream tables, reports, and processes that will be affected.
    *   **Troubleshooting & Root Cause Analysis:** If data quality issues are found in Gold tables, lineage allows data stewards or engineers to trace back to the origin of the data and the transformations applied at each step, helping to pinpoint where the issue was introduced.
    *   **Compliance and Governance:** For regulations like HIPAA, demonstrating data provenance and how sensitive data elements are processed and transformed is critical. Purview's lineage provides this auditable trail.
    *   **Data Discovery:** Business users or analysts can use lineage to understand where the data in their reports or analytical models originates, increasing trust and understanding of the data.

3.  **If a new field, like `DischargeDisposition`, needs to be added to the `Silver_Unified_Encounters` table later, how does using Delta Lake format for your Lakehouse tables make this process easier?**
    *   **Schema Evolution:** Delta Lake supports schema evolution, which means you can add new columns (like `DischargeDisposition`) to an existing Delta table without rewriting the entire table or breaking existing queries that don't use the new column.
    *   The Spark Notebook or Dataflow Gen2 performing the transformation to `Silver_Unified_Encounters` can be updated to include the new `DischargeDisposition` field.
    *   When this updated job runs and writes to the Delta table, the new column will be added to the table's schema automatically (if `mergeSchema` option is used or if the write mode supports it).
    *   **Schema Enforcement:** While allowing evolution, Delta Lake also offers schema enforcement. This prevents accidental schema changes due to bad data, but for intentional additions, schema evolution handles it gracefully.
    *   **No Downtime for Readers:** Existing queries and downstream processes that read from `Silver_Unified_Encounters` but do not yet reference the new `DischargeDisposition` column will continue to work without modification.
    *   **Data Backfill:** After adding the column, historical records will have null values for `DischargeDisposition`. You can then run a separate job to backfill this new column for existing records if the data is available.

### Lab 5.9: Configure Governance for a New Dataset with PHI

**Module Alignment:** Section 5: Security, Compliance, and Governance

**Objective:**
*   Understand and apply sensitivity labels to datasets containing Protected Health Information (PHI) within Microsoft Fabric.
*   (Conceptual) Understand how Data Loss Prevention (DLP) rules can be associated with sensitivity labels to protect data.
*   Explore how to enable and review audit tracking for activities on sensitive data.
*   (Conceptual) Understand how Microsoft Purview can be used to visualize data lineage.
*   Configure basic role-based access control (RBAC) on a Fabric item (e.g., a Lakehouse table).

**Scenario:**
Valley General Hospital has just ingested a new dataset containing patient appointment details and sensitive visit notes into the `CardiologyLakehouse`. This data is highly confidential and subject to HIPAA regulations. As a data engineer with a focus on governance, your task is to apply appropriate security and compliance measures to this dataset.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `CardiologyLakehouse`).
*   A sample table in the Lakehouse representing the new sensitive dataset. We will create one.
*   Permissions to manage sensitivity labels in your Microsoft 365 compliance center (this might require Global Admin or Compliance Admin roles for initial setup of labels. For applying existing labels in Fabric, appropriate workspace permissions are needed).
*   Permissions to manage access to items within your Fabric workspace.
*   (Optional but Recommended) Familiarity with the Microsoft Purview compliance portal.

**Tools to be Used:**
*   Microsoft Fabric Lakehouse
*   Microsoft Fabric Workspace settings
*   (Conceptual/UI Exploration) Microsoft Purview compliance portal (for understanding where labels and DLP are configured)
*   Fabric Notebook (for creating sample data)

**Estimated Time:** 60 minutes (Actual application of labels and DLP might depend on M365 admin configurations. This lab will focus on the Fabric side and conceptual understanding of Purview integration.)

**Tasks / Step-by-Step Instructions:**

**Part 1: Create Sample Sensitive Dataset**

1.  **Open or Create a Notebook:**
    *   In your `DEV_CardiologyAnalytics` workspace, open an existing notebook or create a new one (e.g., `SensitiveData_Governance_Setup`).
    *   Ensure your `CardiologyLakehouse` is attached.

2.  **Generate Sample Data and Table:**
    *   In a PySpark cell, create a sample DataFrame and save it as a table in your Lakehouse. This table will represent the new sensitive dataset.
    ```python
    from pyspark.sql import Row
    from pyspark.sql.functions import col, lit
    from pyspark.sql.types import StructType, StructField, StringType, DateType, TimestampType

    # Define schema for the sensitive appointments data
    schema = StructType([
        StructField("AppointmentID", StringType(), False),
        StructField("PatientID", StringType(), False),
        StructField("AppointmentDate", DateType(), True),
        StructField("ProviderName", StringType(), True),
        StructField("VisitNotes_PHI", StringType(), True), # This column contains sensitive PHI
        StructField("LastUpdated", TimestampType(), True)
    ])

    # Sample data including PHI in VisitNotes_PHI
    data = [
        Row(AppointmentID="APP001", PatientID="P001", AppointmentDate="2023-06-01", ProviderName="Dr. Smith", VisitNotes_PHI="Patient reports chest pain. ECG ordered. Discussed family history of heart disease.", LastUpdated="2023-06-01T10:30:00"),
        Row(AppointmentID="APP002", PatientID="P002", AppointmentDate="2023-06-01", ProviderName="Dr. Jones", VisitNotes_PHI="Follow-up for hypertension. BP 140/90. Medication adjusted.", LastUpdated="2023-06-01T11:15:00"),
        Row(AppointmentID="APP003", PatientID="P003", AppointmentDate="2023-06-02", ProviderName="Dr. Smith", VisitNotes_PHI="Routine check-up. Patient feels well. No new complaints. Advised on diet and exercise.", LastUpdated="2023-06-02T09:00:00")
    ]

    df_sensitive_appointments = spark.createDataFrame(data, schema)

    df_sensitive_appointments.printSchema()
    display(df_sensitive_appointments)

    # Write to Gold table in Lakehouse (assuming it's curated sensitive data)
    table_name = "CardiologyLakehouse.gold_sensitive_appointments"
    df_sensitive_appointments.write.format("delta").mode("overwrite").saveAsTable(table_name)
    print(f"Table '{table_name}' created successfully with sensitive PHI.")
    ```
    *   Run the cell to create the `gold_sensitive_appointments` table.

**Part 2: Apply Sensitivity Labels (in Microsoft Fabric)**

*   **Note:** The availability and names of sensitivity labels are configured by an administrator in the Microsoft Purview compliance portal. For this lab, we'll assume a label like "Confidential - PHI" or "HIPAA-HIGH" exists. If not, you might only be able to see default labels or none.

1.  **Navigate to the Lakehouse Table:**
    *   Go to your `CardiologyLakehouse`.
    *   Under the `Tables` section, find the `gold_sensitive_appointments` table.
2.  **Set Sensitivity Label:**
    *   Click the three dots (**...**) next to the `gold_sensitive_appointments` table.
    *   Select **Settings**.
    *   In the settings pane that opens on the right, look for a section related to **Sensitivity label** (the exact UI might vary slightly).
    *   Click **Edit** or the current label (if one is set, e.g., "None").
    *   A dialog will appear listing available sensitivity labels. Choose an appropriate label that signifies highly confidential PHI (e.g., "Confidential - PHI", "Highly Confidential", or a custom "HIPAA-HIGH" if configured).
    *   Click **Save** or **Apply**.
    *   The table in the Lakehouse explorer might now show an icon or text indicating the applied sensitivity label.

    *   **Alternative (if labeling at dataset level for Power BI):**
        *   If you create a Power BI dataset from this table, you can also set sensitivity labels at the dataset level within the Power BI service settings for that dataset. Fabric aims to inherit these.

**Part 3: Understand Data Loss Prevention (DLP) - Conceptual**

*   DLP policies are configured in the **Microsoft Purview compliance portal** by compliance administrators, not directly within the Fabric workspace by a typical data engineer.
*   These policies are linked to sensitivity labels. For example, a DLP policy could state: "If a file or dataset is labeled 'Confidential - PHI', then block download to unmanaged devices, block sharing with external users, and audit the activity."

1.  **How it Works (Conceptual):**
    *   When you applied the "Confidential - PHI" label to your `gold_sensitive_appointments` table (or a Power BI dataset built on it):
        *   Fabric (and other M365 services) become aware of this classification.
        *   If a DLP policy exists for this label, it will automatically be enforced.
        *   For example, if a user tries to export data from a Power BI report built on this labeled dataset to an Excel file on a personal device, the DLP policy might block the action or generate an alert for administrators.

2.  **Where to Configure (for Admins):**
    *   Microsoft Purview compliance portal (`compliance.microsoft.com`).
    *   Navigate to "Data loss prevention" -> "Policies".
    *   Create a policy, define conditions (e.g., content contains sensitivity label "Confidential - PHI"), and define actions (e.g., restrict access, block activities, send notifications).

**Part 4: Enable and Review Audit Tracking**

*   Auditing is generally enabled by default for most activities in Microsoft Fabric and Microsoft 365. Audit logs capture user and admin activities.

1.  **Where to Review Audit Logs (Typically for Admins/Compliance Officers):**
    *   **Microsoft Purview compliance portal:**
        *   Navigate to `compliance.microsoft.com`.
        *   Go to the **Audit** section.
        *   You can search the audit log for specific activities, users, date ranges, and workloads (e.g., "Power BI", "Microsoft Fabric").
        *   Activities related to accessing or modifying your `gold_sensitive_appointments` table (e.g., viewing it in a notebook, querying it via SQL, changing its label) would be logged here.
    *   **Fabric Monitoring Hub:**
        *   Within Fabric, the Monitoring Hub provides insights into pipeline runs, Spark applications, and other activities. While not the primary audit log for compliance, it's useful for operational monitoring.

2.  **What to Look For:**
    *   Access to the `gold_sensitive_appointments` table (who read it, when).
    *   Changes to the sensitivity label.
    *   Attempts to share or export data from reports built on this table.
    *   Failed access attempts.

**Part 5: Understand Data Lineage with Microsoft Purview - Conceptual**

*   Microsoft Purview automatically scans your Fabric workspaces (if configured by an admin) and maps the relationships between data assets.

1.  **How it Works (Conceptual):**
    *   If Purview has scanned your `CardiologyLakehouse`:
        *   It would identify the `gold_sensitive_appointments` table.
        *   If this table was created by a Notebook (as in Part 1), Purview would show this relationship (lineage from the notebook job to the table).
        *   If a Power BI dataset and report are then created on top of `gold_sensitive_appointments`, Purview would extend the lineage to show `Notebook -> gold_sensitive_appointments_table -> Power_BI_Dataset -> Power_BI_Report`.
2.  **Viewing Lineage (in Purview Data Catalog):**
    *   Users with access to Purview can search for the `gold_sensitive_appointments` asset.
    *   The asset details page in Purview would have a "Lineage" tab showing an interactive graph of its upstream sources and downstream consumers.
    *   This is crucial for understanding data flow, impact analysis, and compliance reporting.

**Part 6: Configure Role-Based Access Control (RBAC) on the Table/Lakehouse**

*   While workspace roles (Admin, Member, Contributor, Viewer) provide broad access, you might want more granular control, especially for highly sensitive data. Fabric allows sharing individual items with specific permissions. SQL-level permissions can also be set on tables via the SQL Analytics Endpoint.

1.  **Sharing a Specific Lakehouse Table (Item-Level Sharing):**
    *   Navigate to your `CardiologyLakehouse`.
    *   Find the `gold_sensitive_appointments` table.
    *   Click the three dots (**...**) next to the table name.
    *   Select **Share**.
    *   In the "Share" dialog:
        *   Enter the name or email of a user or group (e.g., `Cardiology_Analysts_Group`).
        *   Choose the permission level:
            *   **Read:** Allows querying the table (e.g., via SQL endpoint or Notebook).
            *   **ReadData:** (SQL Permissions) Allows `SELECT` on the table data.
            *   **ReadWriteData:** (SQL Permissions) Allows `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
            *   **Build:** (For Power BI datasets) Allows users to build reports on top of this data.
        *   You can also specify if they can re-share.
        *   Add a message (optional).
        *   Click **Grant access**.
    *   This provides more granular access to this specific table beyond the general workspace role.

2.  **SQL Permissions via SQL Analytics Endpoint (More Advanced):**
    *   Open the SQL Analytics Endpoint for your `CardiologyLakehouse`.
    *   You can use T-SQL `GRANT`, `DENY`, `REVOKE` statements to manage permissions on tables for specific users or roles (AAD groups).
        ```sql
        -- Example (run in SQL Analytics Endpoint, not PySpark notebook):
        -- GRANT SELECT ON CardiologyLakehouse.gold_sensitive_appointments TO [user_or_group_email@domain.com];
        -- GRANT SELECT ON CardiologyLakehouse.gold_sensitive_appointments TO [AAD_Group_Name];
        ```
    *   This is powerful for fine-grained control, especially for users accessing data via SQL tools.

**Expected Outcome / Deliverables:**
*   The `gold_sensitive_appointments` table in `CardiologyLakehouse` has an appropriate sensitivity label applied.
*   A conceptual understanding of how DLP policies (configured in Purview) would interact with this label.
*   Knowledge of where to look for audit logs related to activities on sensitive data.
*   A conceptual understanding of how Purview provides data lineage.
*   Experience with sharing a specific Fabric item (the table) with granular permissions, demonstrating the principle of least privilege.

**Questions from Manual & Answers LINK TO HTML?:**

*   **Q1: Why is it important to apply the "principle of least privilege" when granting viewer roles or any access to sensitive datasets like PHI?**
    *   **A1:** The principle of least privilege dictates that users should only be granted the minimum levels of access – or permissions – necessary to perform their job duties. For sensitive datasets like PHI:
        *   **Reduces Risk of Unauthorized Disclosure:** Limiting access minimizes the attack surface. If an account is compromised, the potential for PHI exposure is lessened if that account only had viewer access to a small subset of data.
        *   **Minimizes Accidental Data Modification/Deletion:** Viewer roles prevent users from unintentionally altering or deleting critical PHI.
        *   **Compliance with Regulations:** HIPAA and other privacy regulations mandate strict access controls. Least privilege is a core tenet of demonstrating due diligence in protecting sensitive information.
        *   **Prevents Data Misuse:** Ensures that users cannot access or use PHI for purposes outside their legitimate job functions.
        *   **Simplifies Auditing:** When access is tightly controlled, auditing user activity and investigating incidents becomes more straightforward.

*   **Q2: How does Microsoft Purview support investigations in case of a potential data breach involving the `gold_sensitive_appointments` dataset?**
    *   **A2:** Microsoft Purview provides several capabilities crucial for breach investigations:
        *   **Audit Logs:** Purview's unified audit log captures detailed records of activities across Microsoft 365 services, including Fabric. Investigators can search for who accessed the `gold_sensitive_appointments` dataset, when they accessed it, what actions they performed (e.g., read, export, label change), their IP address, etc. This helps pinpoint the scope and timeline of a breach.
        *   **Data Lineage:** The lineage graph in Purview can show where the `gold_sensitive_appointments` data originated and where it flowed (e.g., to which Power BI reports or other downstream processes). This helps understand the potential blast radius of a breach – what other assets might be compromised if this dataset was breached.
        *   **Sensitivity Labels & Data Classification:** Knowing the dataset was labeled as containing PHI helps prioritize the investigation. Purview can also show other assets with the same label that might be at risk or related.
        *   **Activity Alerts:** If alerts were configured (e.g., for unusual access patterns or attempts to download large volumes of PHI-labeled data), these alerts would provide early indicators and valuable forensic information.
        *   **Data Discovery:** Purview's data map can help identify all locations where similar sensitive data (or copies of the breached data) might exist within the organization's data estate.

*   **Q3: What typically triggers a Data Loss Prevention (DLP) policy to block an action like downloading a report containing data from `gold_sensitive_appointments`?**
    *   **A3:** A DLP policy block is typically triggered by a combination of conditions defined within the policy. Common triggers include:
        1.  **Sensitivity Label Detection:** The primary trigger is often the presence of a specific sensitivity label (e.g., "Confidential - PHI") on the content (the report or its underlying dataset, `gold_sensitive_appointments`).
        2.  **Content Inspection (Keywords/Patterns):** DLP policies can also be configured to inspect content for specific keywords (e.g., "patient record," "diagnosis"), regular expressions (e.g., patterns matching Social Security Numbers or MRNs), or built-in sensitive information types. If the report content matches these patterns, the policy can be triggered even without an explicit label, though label-based is more common for structured data.
        3.  **Context of the Action:**
            *   **Destination:** Attempting to download to an unmanaged device (personal laptop), a non-trusted IP address range, or a USB drive.
            *   **Recipient:** Attempting to share the report or data with external users (outside the organization) or specific unauthorized internal groups.
            *   **Application:** Attempting to copy data to an unsanctioned application (e.g., personal cloud storage).
        4.  **User Attributes:** Policies can sometimes consider the user's role, department, or group membership.
        5.  **Quantity of Sensitive Data:** Some policies might trigger if a large volume of sensitive data is involved in the action.

        When these conditions are met, the DLP policy's defined action (e.g., block the download, encrypt the content, notify an administrator, require justification) is enforced.

This lab covers critical governance aspects. The actual implementation of sensitivity labels and DLP policies heavily depends on the M365/Purview setup done by administrators, but data engineers in Fabric need to understand how to apply labels and how these higher-level governance tools interact with their work.

### Lab 6.8: Build a Readmissions Dashboard with Row-Level Security (RLS)

**Module Alignment:** Section 6: Analytics and Reporting

**Objective:**
*   Connect Power BI to Microsoft Fabric Lakehouse data (from revised Lab 4.8) using the DirectLake connection mode.
*   Build an interactive Power BI dashboard to visualize hospital readmission data.
*   Implement Row-Level Security (RLS) in Power BI to restrict data visibility based on user roles (e.g., a provider seeing only their patients' readmission data).
*   Apply a sensitivity label to the Power BI dataset and report.
*   Publish the Power BI report to a Fabric workspace.

**Scenario:**
Valley General Hospital's quality improvement team needs a dashboard to track 30-day patient readmissions. The dashboard should allow analysis by diagnosis and provider. Crucially, when a specific healthcare provider views the dashboard, they should only see readmission data pertaining to patients under their care to maintain privacy and relevance.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `HealthDataLH_YourName` - consistent with revised Lab 4.8).
*   Gold-layer tables created in the revised Lab 4.8: `Fact_Encounter`, `Dim_Patient`, `Dim_Date`.
    *   `Fact_Encounter` should have `PatientKey`, `AdmissionDateKey`, `LengthOfStayInDays`, and will be augmented with `provider_id` and `is_readmission`.
    *   `Dim_Patient` should have `PatientKey` and `PatientNaturalKey`.
    *   `Dim_Date` should have `DateKey` and date attributes like `FullDate`.
    *   *For this lab, we will augment the `Fact_Encounter` table with a `provider_id` and an `is_readmission` flag.*
*   Power BI Desktop (optional, as much can be done in Fabric, but useful for complex modeling). For this lab, we'll primarily use the Fabric portal experience.
*   Permissions to create and publish Power BI content in the Fabric workspace.
*   Sample user email addresses for testing RLS (you can use your own or test accounts if available).
*   Sensitivity labels (e.g., "Confidential - PHI") configured in Microsoft Purview and available in Fabric.

**Tools to be Used:**
*   Microsoft Fabric Lakehouse (SQL Analytics Endpoint)
*   Microsoft Fabric Power BI (creating datasets and reports directly in the service)
*   (Conceptual) Power BI Desktop for RLS definition if preferred.

**Estimated Time:** 75 - 90 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Augment Gold Layer Data (If Necessary)**

1.  **Open or Create a Notebook:**
    *   In your `DEV_CardiologyAnalytics` workspace, open the notebook used for Lab 4.8 (e.g., `Create_Fact_Encounter_Gold_YourName`) or create a new one (e.g., `Augment_Fact_Encounter_Lab6_8`).
    *   Ensure your `HealthDataLH_YourName` Lakehouse is attached.

2.  **Add `provider_id` and `is_readmission` to `Fact_Encounter`:**
    *   If your `Fact_Encounter` table from Lab 4.8 doesn't have `provider_id` and `is_readmission`, run the following PySpark code to add them.
    ```python
    from pyspark.sql.functions import col, when, lit, rand
    
    lakehouse_name = "HealthDataLH_YourName" # Ensure this matches your Lakehouse name
    fact_encounter_table_name = f"{lakehouse_name}.Fact_Encounter"

    try:
        df_fact_encounter_existing = spark.read.table(fact_encounter_table_name)
        
        existing_cols = df_fact_encounter_existing.columns
        needs_provider = "provider_id" not in existing_cols
        needs_readmission_flag = "is_readmission" not in existing_cols

        if needs_provider:
            # Simulate provider_id - assign one of three providers randomly
            df_fact_encounter_existing = df_fact_encounter_existing.withColumn("provider_id",
                when(rand() < 0.33, "DrSmith@valleygeneral.org")
                .when(rand() < 0.66, "DrJones@valleygeneral.org")
                .otherwise("DrBrown@valleygeneral.org")
            )
            print("Added simulated provider_id column to Fact_Encounter.")

        if needs_readmission_flag:
            # Simulate is_readmission flag - ~20% readmission rate
            df_fact_encounter_existing = df_fact_encounter_existing.withColumn("is_readmission",
                when(rand() < 0.2, True).otherwise(False)
            )
            print("Added simulated is_readmission column to Fact_Encounter.")

        if needs_provider or needs_readmission_flag:
            df_fact_encounter_existing.write.format("delta").mode("overwrite").option("overwriteSchema", "true").saveAsTable(fact_encounter_table_name)
            print(f"{fact_encounter_table_name} table updated with provider_id and is_readmission.")
        else:
            print(f"{fact_encounter_table_name} table already has provider_id and is_readmission.")
            
        df_fact_encounter_augmented = spark.read.table(fact_encounter_table_name)
        display(df_fact_encounter_augmented.select("EncounterNaturalKey", "provider_id", "is_readmission").limit(10))

    except Exception as e:
        print(f"Error loading or augmenting {fact_encounter_table_name}: {e}. Please ensure it was created in Lab 4.8.")
        # Fallback for minimal Fact_Encounter if it doesn't exist (ensure schema matches Lab 4.8 output)
        if "Table or view not found" in str(e) or "NoSuchTableException" in str(e): # Adjusted exception check
            print(f"Creating a minimal {fact_encounter_table_name} for this lab.")
            # Schema based on revised Lab 4.8 Fact_Encounter
            data = [
                ("ENCNAT001", "PATKEY001", 20230101, 20230105, "INPATIENT", "DX01", "PROV001", 4, "EHR_SystemA", "DrSmith@valleygeneral.org", False, "EK001"),
                ("ENCNAT002", "PATKEY002", 20230105, 20230110, "INPATIENT", "DX02", "PROV002", 5, "EHR_SystemB", "DrJones@valleygeneral.org", True, "EK002")
            ]
            columns = [
                "EncounterNaturalKey", "PatientKey", "AdmissionDateKey", "DischargeDateKey", 
                "EncounterType", "PrimaryDiagnosisCode", "AttendingProviderIdentifier", 
                "LengthOfStayInDays", "SourceSystem", "provider_id", "is_readmission", "EncounterKey"
            ]
            df_minimal_fact = spark.createDataFrame(data, columns)
            # Cast to appropriate types if necessary, matching Fact_Encounter schema from Lab 4.8
            # Example: df_minimal_fact = df_minimal_fact.withColumn("LengthOfStayInDays", col("LengthOfStayInDays").cast("int"))
            df_minimal_fact.write.format("delta").mode("overwrite").saveAsTable(fact_encounter_table_name)
            print(f"Minimal {fact_encounter_table_name} created.")
    ```
    *   Run the cell.

**Part 2: Create a Power BI Dataset in Fabric (DirectLake)**

1.  **Navigate to your Lakehouse:** Open `HealthDataLH_YourName`.
2.  **Create a New Power BI Dataset:**
    *   In the Lakehouse explorer view, on the top ribbon, click **New Power BI dataset**.
    *   A dialog "New Power BI dataset" will appear.
    *   Select the tables you need for the readmission dashboard (names from revised Lab 4.8):
        *   `Fact_Encounter`
        *   `Dim_Patient`
        *   `Dim_Date`
    *   Click **Confirm**.
    *   Fabric will create a new Power BI dataset connected to your Lakehouse tables in DirectLake mode. It will open the dataset modeling view.
3.  **Manage Relationships (If Necessary):**
    *   Go to the **Model view**.
    *   Verify or create relationships using the new key names:
        *   `Fact_Encounter.PatientKey` to `Dim_Patient.PatientKey` (Many-to-One, single filter direction from Dim to Fact)
        *   `Fact_Encounter.AdmissionDateKey` to `Dim_Date.DateKey` (Many-to-One, single filter direction)
4.  **Rename the Dataset:**
    *   In the dataset view, go to **File -> Rename**.
    *   Rename it to `ReadmissionAnalytics_Dataset`.
    *   Save your changes.

**Part 3: Implement Row-Level Security (RLS)**

*   We will create a role that filters the `Fact_Encounter` table based on the `provider_id` column.

1.  **Define RLS Roles and Rules:**
    *   In the Power BI dataset modeling view, on the top ribbon, click **Manage roles**.
    *   In the "Manage roles" dialog:
        *   Click **Create**.
        *   **Role name:** Enter `ProviderView`.
        *   Select the `Fact_Encounter` table.
        *   In the DAX filter expression box for `Fact_Encounter`, enter:
            ```dax
            'Fact_Encounter'[provider_id] = USERPRINCIPALNAME()
            ```
        *   Click **Save**.
2.  **Test the Role (Optional but Recommended):**
    *   In the "Manage roles" dialog or by clicking "View as" on the ribbon:
        *   Select the `ProviderView` role.
        *   Optionally enter a user's email (UPN) to simulate their view.
        *   Click **OK** or **Apply**.
        *   The data in `Fact_Encounter` should be filtered.
        *   Click "Stop viewing" on the yellow banner to revert.

**Part 4: Create the Readmissions Power BI Report**

1.  **Create a New Report:**
    *   With `ReadmissionAnalytics_Dataset` open, click **Create report -> Start from scratch**.
2.  **Add Visualizations:**
    *   **Slicer for Encounter Type:**
        *   Add a Slicer visual. Drag `Fact_Encounter[EncounterType]` to the "Field" well.
    *   **Bar Chart: Readmissions by Provider:**
        *   Add a "Stacked bar chart" visual.
        *   **Y-axis:** Drag `Fact_Encounter[provider_id]`
        *   **X-axis:** Drag `Fact_Encounter[EncounterNaturalKey]` (Right-click, select "Count (Distinct)")
        *   **Legend:** Drag `Fact_Encounter[is_readmission]`
    *   **KPI Card: Overall Readmission Rate:**
        *   Create a new measure in `Fact_Encounter`:
            ```dax
            Readmission Rate =
            DIVIDE(
                CALCULATE(COUNTROWS('Fact_Encounter'), 'Fact_Encounter'[is_readmission] = TRUE()),
                COUNTROWS('Fact_Encounter')
            )
            ```
        *   Add a "Card" visual. Drag `[Readmission Rate]` to "Fields". Format as percentage.
    *   **Table: Readmission Details:**
        *   Add a "Table" visual.
        *   Add fields: `Dim_Patient[PatientNaturalKey]`, `Dim_Date[FullDate]` (related via `Fact_Encounter[AdmissionDateKey]`), `Fact_Encounter[provider_id]`, `Fact_Encounter[is_readmission]`, `Fact_Encounter[LengthOfStayInDays]`.
3.  **Arrange and Format:** Improve titles, colors, text sizes.
4.  **Save the Report:**
    *   Click **File -> Save**.
    *   Name: `Hospital Readmission Dashboard`.
    *   Save to your `DEV_CardiologyAnalytics` workspace.

**Part 5: Apply Sensitivity Label to Dataset and Report**

1.  **Label the Dataset:**
    *   Navigate to your `DEV_CardiologyAnalytics` workspace.
    *   Find `ReadmissionAnalytics_Dataset`. Click **... -> Settings**.
    *   Find **Sensitivity label**. Select "Confidential - PHI" or "HIPAA-HIGH". Click **Apply**.
2.  **Label the Report (Often Inherited):**
    *   Open `Hospital Readmission Dashboard`.
    *   Go to **File -> Sensitivity label** and apply the same label if not inherited.

**Part 6: Publish and Test RLS**

1.  **Report is Already in Workspace.**
2.  **Assign Users to RLS Roles:**
    *   Go to `DEV_CardiologyAnalytics` workspace.
    *   Find `ReadmissionAnalytics_Dataset`. Click **... -> Security**.
    *   Next to `ProviderView`, type a test user's email (e.g., `DrSmith@valleygeneral.org`).
    *   Click **Add**, then **Save**.
3.  **Test RLS:**
    *   Log in to Fabric with the test user's account.
    *   Open the `Hospital Readmission Dashboard`. Data should be filtered by `provider_id`.

**Expected Outcome / Deliverables:**
*   An augmented `Fact_Encounter` table in `HealthDataLH_YourName` with `provider_id` and `is_readmission` columns.
*   A Power BI dataset (`ReadmissionAnalytics_Dataset`) in Fabric using DirectLake mode, connected to the revised Gold layer tables.
*   An RLS role (`ProviderView`) defined on the dataset.
*   An interactive Power BI report (`Hospital Readmission Dashboard`).
*   The dataset and report classified with a sensitivity label.

**Questions from Manual & Answers: - LINK TO HTML?**

*   **Q1: Why is DirectLake the preferred connection mode for Power BI reports built on top of Fabric Lakehouse data for live analytics?**
    *   **A1:** DirectLake is preferred because:
        *   **Performance:** It reads Delta Parquet files directly from OneLake without needing to query a SQL endpoint or import/duplicate data into a separate Power BI proprietary format. This significantly reduces latency and improves query performance, especially for large datasets, making it feel like "Import" mode speed with "DirectQuery" data freshness.
        *   **No Data Duplication:** Data remains in OneLake. This avoids data silos, reduces storage costs, and ensures Power BI reports are always querying the single source of truth.
        *   **Real-time Analytics:** Changes made to the Delta tables in the Lakehouse (e.g., by Spark jobs or SQL) are immediately reflected in Power BI reports without needing a dataset refresh schedule (for the data itself, though model changes might need a refresh).
        *   **Simplified Architecture:** It streamlines the data flow from Lakehouse to Power BI.

*   **Q2: What is the primary purpose of Row-Level Security (RLS) in a healthcare analytics context?**
    *   **A2:** The primary purpose of RLS in healthcare analytics is to **restrict data access at the row level based on the identity or role of the user viewing the report or querying the dataset.** This ensures that users only see the data they are authorized to see, which is critical for:
        *   **HIPAA Compliance and Patient Privacy:** Preventing unauthorized access to Protected Health Information (PHI). For example, a doctor should only see their own patients' data, not data for all patients in the hospital.
        *   **Data Minimization:** Adhering to the principle of "minimum necessary" access.
        *   **Relevance:** Providing users with a view of the data that is most relevant to their specific role or department, reducing information overload.
        *   **Security:** Protecting sensitive data from internal threats or accidental exposure.

*   **Q3: How does applying a sensitivity label (e.g., "HIPAA-HIGH") to a Power BI report and its underlying dataset help in protecting the data?**
    *   **A3:** Applying a sensitivity label helps protect data in several ways:
        *   **Visual Indication:** It provides a clear visual marking on the report (and in the service) indicating the data's sensitivity level, reminding users to handle it appropriately.
        *   **Downstream Protection:** Labels can be inherited. If data is exported from a labeled report to Excel or PowerPoint, the label (and its associated protections, if any) can persist in those files.
        *   **Integration with DLP Policies:** Sensitivity labels are a key trigger for Data Loss Prevention (DLP) policies configured in Microsoft Purview. A DLP policy might:
            *   Block or audit attempts to share the labeled report with external users.
            *   Prevent downloading the report to unmanaged devices.
            *   Block printing or copying content from the report.
        *   **Access Control (Conditional Access):** In conjunction with Azure AD Conditional Access policies, access to reports with specific sensitivity labels can be further restricted (e.g., requiring MFA, or blocking access from non-compliant devices).
        *   **Auditing and Reporting:** Activities on labeled content are often audited with more scrutiny, and compliance reports can be generated based on data classifications.
        *   **User Awareness:** It raises user awareness about the nature of the data they are handling.

This lab combines data modeling, Power BI report creation, and crucial security features like RLS and sensitivity labeling. It's a fairly comprehensive one that touches on many aspects of using Fabric for analytics.

### Lab 7.8: Build a Readmission Risk Model with MLflow Tracking

**Module Alignment:** Section 7: Machine Learning and AI Integration

**Objective:**
*   Develop a machine learning model to predict 30-day hospital readmission risk using patient data from the revised Gold layer tables (Lab 4.8).
*   Utilize a Microsoft Fabric Notebook with PySpark and scikit-learn for model development.
*   Perform basic feature engineering, data cleaning, and model training.
*   Integrate MLflow to track experiment parameters, metrics, and the trained model.
*   Persist model predictions back to a Gold layer table in the Lakehouse.
*   (Conceptual) Understand how these predictions could be visualized in Power BI.

**Scenario:**
Valley General Hospital wants to proactively identify patients at high risk of readmission within 30 days of discharge. As a data scientist/engineer, you are tasked with building a classification model using historical patient and encounter data. The model's performance and artifacts need to be tracked using MLflow for reproducibility and governance.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `HealthDataLH_YourName` - consistent with revised Lab 4.8).
*   Gold-layer tables from revised Lab 4.8: `Fact_Encounter` (augmented in Lab 6.8 to include `is_readmission` and `provider_id`) and `Dim_Patient`.
    *   `Fact_Encounter` needs columns like `PatientKey`, `AdmissionDateKey`, `LengthOfStayInDays`, `EncounterType`, `is_readmission` (boolean target variable), and other potential features.
    *   `Dim_Patient` needs `PatientKey` and demographic features like `Age`, `Gender`.
    *   *Ensure `Fact_Encounter` has the `is_readmission` column from Lab 6.8.*
*   Familiarity with creating and running Fabric Notebooks.
*   Basic understanding of machine learning concepts (classification, feature engineering, train-test split, evaluation metrics like AUC, precision, recall).
*   Basic knowledge of Python, PySpark, and scikit-learn.

**Tools to be Used:**
*   Microsoft Fabric Lakehouse
*   Microsoft Fabric Notebook (PySpark, Python, scikit-learn)
*   MLflow (integrated within Fabric)

**Estimated Time:** 90 - 120 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Prepare Data for Modeling**

1.  **Open or Create a Notebook:**
    *   In your `DEV_CardiologyAnalytics` workspace, create a new Notebook (e.g., `Readmission_Risk_Modeling_Lab7_8`) or open an existing one.
    *   Ensure your `HealthDataLH_YourName` Lakehouse is attached.

2.  **Load and Prepare Feature Set:**
    *   In a PySpark cell, load data from `Fact_Encounter` and `Dim_Patient`. Join them and select/engineer features.
    ```python
    from pyspark.sql.functions import col, mean # Added mean import

    lakehouse_name = "HealthDataLH_YourName" # Ensure this matches your Lakehouse name
    fact_encounter_table = f"{lakehouse_name}.Fact_Encounter"
    dim_patient_table = f"{lakehouse_name}.Dim_Patient"

    # Load tables
    df_fact_encounter = spark.read.table(fact_encounter_table)
    df_dim_patient = spark.read.table(dim_patient_table)

    # Join fact and dimension tables
    # Ensure 'is_readmission' column exists in df_fact_encounter (from Lab 6.8 augmentation)
    df_model_input = df_fact_encounter.join(
        df_dim_patient,
        df_fact_encounter.PatientKey == df_dim_patient.PatientKey, # Join on correct surrogate key
        "inner"
    ).select(
        df_fact_encounter.EncounterNaturalKey, # Using Natural Key for identification
        df_fact_encounter.PatientKey, # Keep for potential future use or joining predictions
        df_dim_patient.Age.alias("age"), # Using Age from Dim_Patient
        df_dim_patient.Gender.alias("gender"), # Using Gender from Dim_Patient
        df_fact_encounter.EncounterType.alias("encounter_type"),
        df_fact_encounter.LengthOfStayInDays.alias("length_of_stay_days"),
        # Add more features if available, e.g., number of prior visits, specific diagnosis codes (would require Dim_Diagnosis)
        # For simplicity, we'll use these.
        # Ensure the target variable 'is_readmission' is present and boolean or 0/1
        col("is_readmission").cast("boolean").alias("label") # Our target variable
    )

    # Handle missing values (simple imputation for this lab)
    # For length_of_stay_days, fill with mean or median if appropriate, or 0 if it makes sense
    # For categorical, fill with a specific category like 'Unknown'
    mean_los_val = df_model_input.select(mean(col("length_of_stay_days"))).first()
    mean_los = mean_los_val[0] if mean_los_val and mean_los_val[0] is not None else 0


    df_model_input = df_model_input.fillna({
        "length_of_stay_days": mean_los, 
        "age": df_model_input.select(mean(col("age"))).first()[0] or 0, # Fill age with mean or 0
        "gender": "Unknown",
        "encounter_type": "Unknown"
    })
    
    # Ensure label column does not have nulls for training
    df_model_input = df_model_input.na.drop(subset=["label"])

    # Convert to Pandas DataFrame for scikit-learn
    pandas_df = df_model_input.toPandas()

    print(f"Prepared dataset for modeling with {pandas_df.shape[0]} rows and {pandas_df.shape[1]} columns.")
    display(pandas_df.head())
    # pandas_df.info() # Display in notebook
    ```
    *   Run the cell. This creates a Pandas DataFrame ready for scikit-learn.

**Part 2: Feature Engineering and Preprocessing (scikit-learn)**

1.  **Encode Categorical Features and Split Data:**
    *   In a new Python cell:
    ```python
    import pandas as pd
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import OneHotEncoder, StandardScaler
    from sklearn.compose import ColumnTransformer
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.metrics import accuracy_score, precision_score, recall_score, roc_auc_score, f1_score, confusion_matrix
    import mlflow
    import mlflow.sklearn
    import numpy as np

    # Ensure pandas_df is available from the previous PySpark cell
    if 'pandas_df' not in locals() or pandas_df.empty:
        print("pandas_df is not defined or empty. Please run the previous cell to generate it.")
        # Fallback dummy data reflecting new column names
        data_dummy = {
            'EncounterNaturalKey': [f'E{i}' for i in range(100)],
            'PatientKey': [f'PK{i}' for i in range(100)],
            'age': np.random.randint(20, 85, 100),
            'gender': np.random.choice(['M', 'F', 'Unknown'], 100),
            'encounter_type': np.random.choice(['INPATIENT', 'OUTPATIENT', 'EMERGENCY', 'Unknown'], 100),
            'length_of_stay_days': np.random.randint(0, 15, 100),
            'label': np.random.choice([True, False], 100, p=[0.2, 0.8])
        }
        pandas_df = pd.DataFrame(data_dummy)
        pandas_df['length_of_stay_days'] = pandas_df['length_of_stay_days'].astype(float)
        pandas_df['age'] = pandas_df['age'].astype(float)
        print("Created a dummy pandas_df for testing.")

    # Separate features (X) and target (y)
    X = pandas_df.drop(columns=['label', 'EncounterNaturalKey', 'PatientKey']) # Drop identifiers and target
    y = pandas_df['label'].astype(int) # Ensure target is integer (0 or 1)

    # Identify categorical and numerical features
    categorical_features = ['gender', 'encounter_type'] # 'age' is now numerical
    numerical_features = ['age', 'length_of_stay_days']

    # Create preprocessor
    preprocessor = ColumnTransformer(
        transformers=[
            ('cat', OneHotEncoder(handle_unknown='ignore'), categorical_features),
            ('num', StandardScaler(), numerical_features)
        ],
        remainder='passthrough'
    )

    # Split data into training and testing sets
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42, stratify=y)

    # Apply preprocessing
    X_train_processed = preprocessor.fit_transform(X_train)
    X_test_processed = preprocessor.transform(X_test)

    print("Data preprocessing complete.")
    # print(f"X_train_processed shape: {X_train_processed.shape}") # Display in notebook
    # print(f"X_test_processed shape: {X_test_processed.shape}") # Display in notebook
    ```
    *   Run the cell.

**Part 3: Train Model and Track with MLflow**

1.  **Train a Classifier and Log with MLflow:**
    *   In a new Python cell:
    ```python
    # MLflow experiment setup
    # mlflow.set_experiment("ReadmissionRiskExperiment_Lab7_8") # Optional

    with mlflow.start_run() as run:
        run_id = run.info.run_id
        print(f"MLflow Run ID: {run_id}")

        model = RandomForestClassifier(n_estimators=100, random_state=42, class_weight='balanced')
        model.fit(X_train_processed, y_train)

        y_pred_test = model.predict(X_test_processed)
        y_pred_proba_test = model.predict_proba(X_test_processed)[:, 1]

        test_accuracy = accuracy_score(y_test, y_pred_test)
        precision = precision_score(y_test, y_pred_test, zero_division=0)
        recall = recall_score(y_test, y_pred_test, zero_division=0)
        f1 = f1_score(y_test, y_pred_test, zero_division=0)
        auc = roc_auc_score(y_test, y_pred_proba_test) if len(np.unique(y_test)) > 1 else 0.5

        print(f"Test Accuracy: {test_accuracy:.4f}")
        # ... print other metrics ...

        mlflow.log_param("model_type", model.__class__.__name__)
        mlflow.log_params(model.get_params())
        mlflow.log_metric("test_accuracy", test_accuracy)
        mlflow.log_metric("precision", precision)
        mlflow.log_metric("recall", recall)
        mlflow.log_metric("f1_score", f1)
        mlflow.log_metric("auc", auc)

        mlflow.sklearn.log_model(sk_model=model, artifact_path="readmission_risk_model", serialization_format=mlflow.sklearn.SERIALIZATION_FORMAT_PICKLE)
        mlflow.sklearn.log_model(sk_model=preprocessor, artifact_path="preprocessor")
        print("Model, preprocessor, parameters, and metrics logged to MLflow.")
        
        try:
            feature_names_out = preprocessor.get_feature_names_out()
            mlflow.log_text("\n".join(feature_names_out), "feature_names.txt")
        except Exception as e:
            print(f"Could not log feature names: {e}")

    print(f"MLflow Run completed.")
    ```
    *   Run the cell.

**Part 4: Persist Predictions to Lakehouse**

1.  **Make Predictions on the Full Dataset and Save:**
    *   In a new Python cell:
    ```python
    # Get the latest run ID for the current notebook's experiment (or use specific run_id)
    # This assumes the notebook name is used as the experiment name by default in Fabric
    notebook_path = mlflow.get_run(run_id=None).data.tags['mlflow.source.name']
    current_experiment = mlflow.get_experiment_by_name(notebook_path)
    
    logged_run_id_to_load = run_id # Use run_id from the training cell above for simplicity in this combined script
    # If running separately, you might use:
    # if current_experiment:
    #    latest_run = mlflow.search_runs(experiment_ids=[current_experiment.experiment_id], order_by=["start_time DESC"], max_results=1).iloc[0]
    #    logged_run_id_to_load = latest_run.run_id
    # else:
    #    print("Experiment not found. Cannot load model.")
    #    logged_run_id_to_load = None


    if logged_run_id_to_load:
        print(f"Using MLflow Run ID: {logged_run_id_to_load} to load model and preprocessor.")
        logged_model_path = f"runs:/{logged_run_id_to_load}/readmission_risk_model"
        logged_preprocessor_path = f"runs:/{logged_run_id_to_load}/preprocessor"

        loaded_model = mlflow.sklearn.load_model(logged_model_path)
        loaded_preprocessor = mlflow.sklearn.load_model(logged_preprocessor_path)

        X_full = pandas_df.drop(columns=['label', 'EncounterNaturalKey', 'PatientKey'])
        X_full_processed = loaded_preprocessor.transform(X_full)

        full_predictions = loaded_model.predict(X_full_processed)
        full_prediction_probabilities = loaded_model.predict_proba(X_full_processed)[:, 1]

        pandas_df_with_predictions = pandas_df.copy()
        pandas_df_with_predictions['predicted_readmission_label'] = full_predictions
        pandas_df_with_predictions['predicted_readmission_probability'] = full_prediction_probabilities
        
        df_predictions_output = pandas_df_with_predictions[['EncounterNaturalKey', 'PatientKey', 'label', 'predicted_readmission_label', 'predicted_readmission_probability']]
        
        spark_df_predictions = spark.createDataFrame(df_predictions_output)
        
        # display(spark_df_predictions.limit(10)) # Display in notebook

        output_table_name = f"{lakehouse_name}.gold_patient_readmission_predictions"
        spark_df_predictions.write.format("delta").mode("overwrite").saveAsTable(output_table_name)
        print(f"{output_table_name} table created successfully.")
    else:
        print("Could not determine MLflow run ID to load the model.")
    ```
    *   Run the cell.

**Part 5: Conceptual - Visualize Predictions in Power BI**

*   The `gold_patient_readmission_predictions` table can now be used in Power BI.
*   You would create/update a Power BI dataset to include this table.
*   In a Power BI report, you could:
    *   Show patients with high `predicted_readmission_probability`.
    *   Visualize distribution of prediction probabilities.
    *   Compare `label` with `predicted_readmission_label`.

**Expected Outcome / Deliverables:**
*   A trained machine learning model.
*   An MLflow experiment run with logged parameters, metrics, model, and preprocessor.
*   A Delta table `gold_patient_readmission_predictions` in `HealthDataLH_YourName` with predictions.
*   Understanding of the end-to-end ML workflow within Fabric.
*   Understanding of the end-to-end ML workflow within Fabric using Notebooks and MLflow.

**Questions from Manual & Answers: - LINK TO HTML?**

*   **Q1: What makes a machine learning model "explainable," and why is this particularly important in healthcare decision-making?**
    *   **A1:**
        *   **Explainable Model:** An explainable AI (XAI) model is one whose internal workings and decision-making processes can be understood by humans. Instead of being a "black box," it provides insights into *why* it made a particular prediction or decision. Techniques like SHAP (SHapley Additive exPlanations) or LIME (Local Interpretable Model-agnostic Explanations) can help identify which input features contributed most to a prediction.
        *   **Importance in Healthcare:**
            1.  **Clinical Trust and Adoption:** Clinicians are more likely to trust and use AI-driven recommendations if they understand the reasoning behind them. A black box model that simply outputs a risk score without explanation is less likely to be adopted.
            2.  **Patient Safety and Accountability:** If a model makes an incorrect prediction leading to an adverse patient outcome, explainability is crucial for understanding the failure, debugging the model, and determining accountability.
            3.  **Bias Detection and Fairness:** Explainability techniques can help uncover if a model is relying on sensitive attributes (like race or gender, even if indirectly) in a biased way, which is ethically and legally problematic in healthcare.
            4.  **Regulatory Compliance:** Regulatory bodies (like the FDA for medical devices incorporating AI) are increasingly emphasizing the need for model transparency and explainability.
            5.  **Improving Models:** Understanding which features are driving predictions can provide insights back to data scientists to improve feature engineering or identify data quality issues.
            6.  **Personalized Medicine:** Understanding why a model predicts a certain risk for an *individual* patient can help tailor interventions more effectively.

*   **Q2: How do tools like SHAP (SHapley Additive exPlanations) values assist clinicians in understanding and trusting model predictions?**
    *   **A2:** SHAP values assist clinicians by providing **feature-level importance for individual predictions**. For a specific patient predicted to be at high risk of readmission, SHAP can show:
        *   Which specific factors (e.g., "length of previous stay = 10 days," "age_group = 70+", "number of chronic conditions = 5") contributed most to increasing that risk score.
        *   Which factors might have decreased the risk score.
        *   The magnitude of each feature's contribution.
        This allows a clinician to see if the model's reasoning aligns with their clinical judgment. If the top reasons make clinical sense, trust in the prediction increases. If a seemingly irrelevant feature is driving the prediction, it might indicate a model issue or an unexpected correlation worth investigating. This transparency moves beyond just a risk score to a more actionable insight.

*   **Q3: Why is model lineage (tracking data, code, parameters, and versions used to train a model) important for compliance and reproducibility in healthcare AI?**
    *   **A3:** Model lineage is critical for:
        *   **Reproducibility:** If you need to retrain the model or reproduce a previous result (e.g., for validation or to understand a past prediction), lineage ensures you can use the exact same dataset version, code version, environment, and hyperparameters. MLflow helps capture much of this.
        *   **Auditing and Compliance:** Regulatory bodies or internal auditors may require proof of how a model was developed, validated, and what data it was trained on. Complete lineage provides this audit trail, demonstrating due diligence and adherence to development standards (e.g., for HIPAA security rule compliance regarding data integrity and access).
        *   **Debugging and Error Analysis:** If a model starts performing poorly or making unexpected predictions, lineage helps trace back to changes in data, code, or dependencies that might have caused the issue.
        *   **Model Versioning and Management:** As models are updated or retrained, lineage helps track different versions, their performance, and the data they were trained on, allowing for rollback if a new version underperforms.
        *   **Impact Analysis:** If an issue is found in an upstream data source, lineage helps identify all models trained on that data that might be affected and require retraining or revalidation.
        *   **Transparency and Trust:** Documented lineage contributes to the overall transparency of the AI system, building trust with stakeholders, including clinicians and patients.

This lab provides a foundational end-to-end machine learning example within Fabric. Real-world ML projects would involve more sophisticated feature engineering, hyperparameter tuning, cross-validation, and potentially more complex model architectures, but this covers the core workflow and MLflow integration.


### Lab 8.7: Diagnose and Optimize a Slow Power BI Report

**Module Alignment:** Section 8: Performance Optimization

**Objective:**
*   Identify performance bottlenecks in a Power BI report using the Performance Analyzer.
*   Apply DAX optimization techniques by converting calculated columns to measures, referencing the updated Gold layer tables from revised Lab 4.8.
*   Implement data model best practices by reducing unnecessary fields in visuals.
*   Understand how Lakehouse table optimization (partitioning, `OPTIMIZE` command) can contribute to better Power BI performance when using DirectLake with tables from revised Lab 4.8.

**Scenario:**
Valley General Hospital's cardiology department uses a Power BI report to track patient encounter trends. Recently, clinicians have reported that the main page of this report is loading very slowly. Your task is to diagnose the performance issues and implement optimizations, using the data model established in the revised Lab 4.8.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `HealthDataLH_YourName` - consistent with revised Lab 4.8).
*   Gold-layer tables from revised Lab 4.8: `Fact_Encounter`, `Dim_Patient`, `Dim_Date`.
    *   `Fact_Encounter` should have a variety of columns, including dates and numerical values.
*   A Power BI report built on these tables within the Fabric workspace. We will create a sample "slow" report for this lab.
*   Permissions to edit Power BI reports and datasets in the Fabric workspace.
*   (Optional but helpful) Power BI Desktop for more in-depth DAX editing or model viewing.

**Tools to be Used:**
*   Microsoft Fabric Power BI (report and dataset editing in the service)
*   Power BI Performance Analyzer
*   Microsoft Fabric Notebook (for Lakehouse table optimization)
*   Microsoft Fabric Lakehouse (SQL Analytics Endpoint - conceptual for verifying table optimizations)

**Estimated Time:** 75 - 90 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Create a Sample "Slow" Power BI Report**

1.  **Create/Open a Notebook to Prepare Data (If Needed):**
    *   Ensure your `Fact_Encounter`, `Dim_Patient`, and `Dim_Date` tables (from revised Lab 4.8) are populated. If `Fact_Encounter` is small, let's add more rows to simulate a larger dataset.
    ```python
    from pyspark.sql.functions import col, lit, concat # Added concat

    lakehouse_name = "HealthDataLH_YourName" # Ensure this matches your Lakehouse name
    fact_encounter_table = f"{lakehouse_name}.Fact_Encounter"

    df_fact_encounter = spark.read.table(fact_encounter_table)
    current_rows = df_fact_encounter.count()
    print(f"Current rows in {fact_encounter_table}: {current_rows}")

    if current_rows < 10000: 
        num_multiples = (50000 // current_rows) + 1 if current_rows > 0 else 50000
        
        dfs_to_union = []
        if current_rows > 0:
            for i in range(num_multiples):
                df_new_iteration = df_fact_encounter.withColumn("EncounterNaturalKey", concat(col("EncounterNaturalKey"), lit(f"_copy{i}")))
                # Ensure other unique keys are handled if necessary for union, or select distinct later
                dfs_to_union.append(df_new_iteration)
            
            if dfs_to_union:
                df_fact_encounter_large = dfs_to_union[0]
                for i in range(1, len(dfs_to_union)):
                    df_fact_encounter_large = df_fact_encounter_large.unionByName(dfs_to_union[i], allowMissingColumns=True)
                
                # Overwrite the original Fact_Encounter table with the larger version
                df_fact_encounter_large.write.format("delta").mode("overwrite").option("overwriteSchema", "true").saveAsTable(fact_encounter_table)
                print(f"Updated {fact_encounter_table} with approximately {df_fact_encounter_large.count()} rows.")
            else:
                print("No data to multiply.")
        else:
            print(f"{fact_encounter_table} is empty, cannot multiply rows.")
    else:
        print(f"{fact_encounter_table} is already large enough.")
    ```
    *   Run the cell.

2.  **Create a New Power BI Dataset and Report in Fabric:**
    *   Navigate to `HealthDataLH_YourName` Lakehouse.
    *   Click **New Power BI dataset**. Select `Fact_Encounter`, `Dim_Patient`, and `Dim_Date`. Click **Confirm**.
    *   Rename the dataset to `EncounterTrends_Slow_Dataset_v2`.
    *   Verify relationships in the Model view (e.g., `Fact_Encounter.PatientKey` to `Dim_Patient.PatientKey`, `Fact_Encounter.AdmissionDateKey` to `Dim_Date.DateKey`).
    *   Click **Create report -> Start from scratch**.
3.  **Design the "Slow" Report Page:**
    *   **Add a Calculated Column (Bad Practice for this scenario):**
        *   Go to the Data view in the dataset. Select `Fact_Encounter`.
        *   Click **New column**.
        *   DAX for `AdmissionDateFromDim` (using `Dim_Date` and `Fact_Encounter` keys):
            ```dax
            AdmissionFullDate_CC = LOOKUPVALUE(Dim_Date[FullDate], Dim_Date[DateKey], 'Fact_Encounter'[AdmissionDateKey])
            ```
        *   Then another calculated column in `Fact_Encounter`:
            ```dax
            EncounterYearMonth_CC = FORMAT([AdmissionFullDate_CC], "YYYY-MM")
            ```
    *   **Visual 1: Table with Many Columns and the Calculated Column**
        *   Add a "Table" visual.
        *   Drag fields:
            *   `Dim_Patient[PatientNaturalKey]`
            *   `Dim_Patient[Gender]`
            *   `Dim_Patient[Age]`
            *   `Fact_Encounter[EncounterNaturalKey]`
            *   `Fact_Encounter[EncounterType]`
            *   `Fact_Encounter[LengthOfStayInDays]`
            *   `Fact_Encounter[EncounterYearMonth_CC]`
            *   `Fact_Encounter[provider_id]` (if added in Lab 6.8)
            *   `Fact_Encounter[is_readmission]` (if added in Lab 6.8)
            *   `Dim_Date[FullDate]` (related via `AdmissionDateKey`, rename in visual to `AdmissionFullDateFromDimDate`)
    *   **Visual 2: Matrix with High Cardinality Fields**
        *   Add a "Matrix" visual.
        *   **Rows:** `Dim_Patient[PatientNaturalKey]`
        *   **Columns:** `Fact_Encounter[EncounterYearMonth_CC]`
        *   **Values:** `Fact_Encounter[EncounterNaturalKey]` (Count)
    *   **Visual 3: Card with Complex Measure (using the CC)**
        *   Create a new measure in `Fact_Encounter`:
            ```dax
            ComplexCount_CC_v2 = COUNTROWS(FILTER('Fact_Encounter', 'Fact_Encounter'[EncounterYearMonth_CC] = "2023-01")) 
            ```
            *(Adjust "2023-01" if your sample data's year/month is different)*
        *   Add a "Card" visual with this `[ComplexCount_CC_v2]` measure.
    *   Save the report as `EncounterTrends_Slow_Report_v2`.

**Part 2: Diagnose with Performance Analyzer**
    (Steps remain the same as original lab: Start recording, refresh visuals, analyze DAX Query times.)

**Part 3: Implement Optimizations**

1.  **Optimization 1: Convert Calculated Column to Measure / Use `Dim_Date`**
    *   **Step 3.1.1: Add YearMonth to `Dim_Date` (if not already there from revised Lab 4.8):**
        *   The revised Lab 4.8's `Dim_Date` (simplified version) might not have a `YearMonth` text column. If it doesn't, you can add it in the Power BI data model or ideally back in the Notebook that creates `Dim_Date`. For this lab, let's assume we'll use `Dim_Date[Month]` and `Dim_Date[Year]` columns that should exist.
        *   Alternatively, if `Dim_Date` has `FullDate`, you can create a `YearMonth` column in Power BI's Data View for `Dim_Date`:
            ```dax
            YearMonth = FORMAT('Dim_Date'[FullDate], "YYYY-MM")
            ```
    *   **Step 3.1.2: Modify Report Visuals:**
        *   Open `EncounterTrends_Slow_Report_v2` for editing.
        *   Remove `Fact_Encounter[EncounterYearMonth_CC]` from all visuals.
        *   Delete the calculated columns `[AdmissionFullDate_CC]` and `[EncounterYearMonth_CC]` from `Fact_Encounter`.
        *   Drag `Dim_Date[YearMonth]` (if created in 3.1.1) or use `Dim_Date[Year]` and `Dim_Date[Month]` in visuals. For the Matrix columns, use `Dim_Date[YearMonth]`.
    *   **Step 3.1.3: Modify Complex Measure:**
        *   Edit the `ComplexCount_CC_v2` measure. Filter on `Dim_Date[YearMonth]` (or `Dim_Date[Year]` and `Dim_Date[Month]`):
            ```dax
            ComplexCount_Optimized_v2 =
            CALCULATE(
                COUNTROWS('Fact_Encounter'),
                'Dim_Date'[YearMonth] = "2023-01" // Or KEEPFILTERS(Dim_Date[Year]=2023 && Dim_Date[Month]=1)
            )
            ```
        *   Replace the old measure in the Card visual.

2.  **Optimization 2: Reduce Unnecessary Fields in Visuals**
    (Steps remain the same: Review table visual, remove non-essential columns.)

3.  **Optimization 3: Use Measures Instead of Implicit Measures or Summarized Columns**
    *   For the Matrix visual's "Values" field, create an explicit measure:
        ```dax
        Total Encounters_v2 = COUNTROWS('Fact_Encounter')
        ```
    *   Use `[Total Encounters_v2]` in the Matrix values.

4.  **Re-test with Performance Analyzer:**
    (Steps remain the same: Save, clear recording, start again, refresh, compare timings.)

**Part 4: Lakehouse Table Optimization (Conceptual for Power BI Impact)**
    (Concept remains the same, but ensure table names are correct if running the OPTIMIZE commands.)
1.  **Partitioning (Conceptual)**
2.  **Run `OPTIMIZE` and `VACUUM` on Lakehouse Tables:**
    *   Open a Fabric Notebook:
    ```python
    lakehouse_name = "HealthDataLH_YourName"
    # Run OPTIMIZE on key tables
    spark.sql(f"OPTIMIZE {lakehouse_name}.Fact_Encounter")
    spark.sql(f"OPTIMIZE {lakehouse_name}.Dim_Patient")
    spark.sql(f"OPTIMIZE {lakehouse_name}.Dim_Date")
    print("OPTIMIZE command completed for relevant tables.")

    # Run VACUUM (be cautious with retention period in production)
    # spark.sql(f"VACUUM {lakehouse_name}.Fact_Encounter RETAIN 168 HOURS") 
    # print("VACUUM command completed for Fact_Encounter.")
    ```

**Expected Outcome / Deliverables:**
*   An understanding of how to use Power BI Performance Analyzer to identify bottlenecks.
*   An optimized version of the `EncounterTrends_Slow_Report` with:
    *   Calculated columns replaced by measures or dimension table attributes.
    *   Reduced number of fields in some visuals.
*   Demonstrably faster load times for the report page (as shown by Performance Analyzer).
*   Knowledge of Lakehouse table maintenance commands (`OPTIMIZE`) that contribute to overall query performance.

**Questions from Manual & Answers: LINK TO HTML?**

*   **Q1: Why are DAX Measures generally preferred over Calculated Columns for aggregations or dynamic calculations in Power BI, especially for performance?**
    *   **A1:**
        *   **Calculation Timing & Storage:**
            *   **Calculated Columns:** Are computed row by row during data refresh and stored in the model. This consumes memory and increases model size. For every row, the DAX expression is evaluated and its result materialized.
            *   **Measures:** Are calculated at query time, only when they are used in a visual. They are not stored in the model, so they don't increase model size or refresh time directly.
        *   **Context Transition:** Measures are evaluated in the context of the visual or filter they are placed in. Calculated columns are evaluated in the row context of their table and do not inherently respond to report filters in the same dynamic way without further context transition in measures that use them.
        *   **Performance:**
            *   For large tables, calculated columns can significantly slow down data refresh and increase memory footprint.
            *   While complex measures can also be slow if poorly written, they are generally more efficient for aggregations because they operate on aggregated data based on the current filter context, rather than pre-calculating for every row.
            *   Calculated columns can sometimes inhibit query optimization techniques that the Power BI engine (VertiPaq) uses.
        *   **Flexibility:** Measures are more flexible as they dynamically respond to filters and slicers in the report.

*   **Q2: What are common causes of slow file scans or query performance against tables in a Fabric Lakehouse when queried by Power BI in DirectLake mode?**
    *   **A2:**
        *   **Too Many Small Files:** Delta Lake tables can accumulate many small Parquet files, especially after frequent small appends or updates. Querying many small files is less efficient than querying fewer, larger files. The `OPTIMIZE` command helps compact these.
        *   **Lack of or Ineffective Partitioning:** If large tables are not partitioned, or partitioned on columns with very high cardinality or columns not frequently used in filters, queries might have to scan much more data than necessary.
        *   **Schema Complexity:** Very wide tables (hundreds of columns) can lead to more data being read, even if only a few columns are selected, depending on how Parquet files store column stripes.
        *   **Data Skew in Partitions:** If data is partitioned, but one partition is vastly larger than others, queries hitting that partition will be slow.
        *   **Outdated Table Statistics:** The query optimizer relies on statistics about the data distribution. If statistics are stale, it might generate suboptimal query plans. Running `ANALYZE TABLE ... COMPUTE STATISTICS` can help.
        *   **Complex Predicates/Filters in Power BI:** Even with DirectLake, if the DAX queries generated by Power BI visuals involve very complex filtering logic that doesn't translate well to efficient Delta table scans (e.g., filtering on computationally intensive derived values not present in the table), performance can suffer.
        *   **Insufficient Fabric Capacity:** If the Fabric capacity allocated to the workspace is under-provisioned for the query load, queries will queue or run slowly due to resource contention.

*   **Q3: How can caching strategies, either within Power BI or at other layers, improve report rendering speed for frequently accessed reports?**
    *   **A3:**
        *   **Power BI Service Query Caching:** The Power BI service automatically caches query results for visuals. When a user opens a report, if the same query (with the same filter context) has been executed recently and the underlying data hasn't changed significantly (or the cache hasn't expired), Power BI can serve the result from its cache instead of re-querying the data source (even DirectLake). This dramatically speeds up report loading for subsequent users or visits. Cache duration varies (e.g., typically up to an hour, can be influenced by dataset refresh).
        *   **Browser Caching:** Browsers cache static assets of the Power BI report (like images, report structure), which helps in rendering the report shell faster on subsequent visits.
        *   **DirectLake and OneLake Caching:** OneLake itself might have caching layers for frequently accessed Delta/Parquet file footers or metadata, speeding up the "query" part of DirectLake. Fabric capacities also manage memory for caching data read from OneLake.
        *   **Materialized Views (in SQL Warehouse/Lakehouse):** For very complex or common aggregations that are still slow even with DirectLake, you could pre-calculate them and store them in materialized views (if using a Warehouse) or aggregated Gold tables in the Lakehouse. Power BI would then query these pre-aggregated results, which is a form of manual caching/pre-computation.
        *   **Power BI Premium Per User (PPU) / Premium Capacity Features:** These capacities offer more control over caching and performance, including potentially larger cache sizes and more aggressive caching.
        *   **Dashboard Tile Caching:** Tiles pinned to Power BI dashboards have their own caching mechanism and refresh schedule, which can provide quick views of key metrics.

        It's important to note that for DirectLake, the primary benefit is already avoiding the import model's refresh latency. Caching then further optimizes the query execution part for repeated views.

This performance optimization lab gives a taste of common issues and solutions. Real-world performance tuning can be a deep and iterative process.

### Lab 9.8: Configure Collaboration Settings for a New Project Workspace

**Module Alignment:** Section 9: Collaboration and Sharing

**Objective:**
*   Create a new Microsoft Fabric workspace tailored for a specific project.
*   Assign different workspace roles (Admin, Member, Contributor, Viewer) to team members based on their project responsibilities, demonstrating the principle of least privilege.
*   Understand how to share specific Fabric items (e.g., a Power BI report) with appropriate permissions.
*   Apply a sensitivity label to a shared item to govern its usage.
*   (Conceptual) Understand how access reviews can be configured for compliance.

**Scenario:**
Valley General Hospital is initiating a new project to analyze Emergency Department (ED) efficiency. A cross-functional team has been assembled, including data engineers, data analysts, and ED clinical leads. You need to set up a dedicated Fabric workspace for this project, ensuring each team member has the appropriate level of access to collaborate effectively while adhering to data governance policies.

**Prerequisites:**
*   Microsoft Fabric enabled Microsoft 365 tenant.
*   Permissions to create workspaces in Fabric.
*   A few sample user email addresses (you can use your own, test accounts, or colleagues' emails if they are part of your M365 tenant and you have permission to add them for testing purposes).
*   A sample Power BI report (can be a simple one created for this lab or one from a previous lab).
*   Sensitivity labels (e.g., "Confidential - Internal Use" or "HIPAA-HIGH") configured in Microsoft Purview and available in Fabric.

**Tools to be Used:**
*   Microsoft Fabric Portal (Workspace creation and management)
*   Power BI (for sharing a report item)

**Estimated Time:** 45 - 60 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Create a New Project Workspace**

1.  **Navigate to Microsoft Fabric:** Open `app.fabric.microsoft.com`.
2.  **Create New Workspace:**
    *   Click on **Workspaces** in the left navigation pane.
    *   Click **+ New workspace**.
    *   **Name:** `ED_Efficiency_Project_Q2` (or a similar descriptive name indicating project and timeframe).
    *   **Description:** (Optional) "Workspace for the Q2 Emergency Department Efficiency Analysis Project. Contains ED patient flow data, staffing data, and performance dashboards."
    *   **Domain:** (Optional) Assign to a relevant domain (e.g., "Clinical Analytics," "Operational Improvement").
    *   **Capacity:** Assign the workspace to a Fabric capacity.
    *   Click **Apply**.
    *   Your new workspace `ED_Efficiency_Project_Q2` is now created.

**Part 2: Assign Workspace Roles to Team Members**

*   For this part, you'll simulate adding team members with different roles. Replace the placeholder email addresses with actual test user emails if you have them.

1.  **Access Workspace Management:**
    *   Open the `ED_Efficiency_Project_Q2` workspace.
    *   In the top right corner of the workspace view, click on **Manage access**.
2.  **Assign Roles:**
    *   **Data Engineering Lead (Admin):**
        *   Click **+ Add people or groups**.
        *   **Enter name or email:** `data.engineer.lead@valleygeneral.org` (replace with a real test email or your own).
        *   **Role:** Select **Admin**. (Admins have full control, can manage content, settings, and access).
        *   Click **Add**.
    *   **Data Analyst (Contributor):**
        *   Click **+ Add people or groups**.
        *   **Enter name or email:** `data.analyst.ed@valleygeneral.org` (replace).
        *   **Role:** Select **Contributor**. (Contributors can create, edit, and delete content like reports, datasets, notebooks, and dataflows. They can publish reports. They cannot manage workspace settings or access for others).
        *   Click **Add**.
    *   **ED Clinical Lead / Manager (Viewer):**
        *   Click **+ Add people or groups**.
        *   **Enter name or email:** `ed.manager@valleygeneral.org` (replace).
        *   **Role:** Select **Viewer**. (Viewers can view and interact with reports and dashboards but cannot edit content or see underlying datasets/dataflows unless explicitly shared with build permissions).
        *   Click **Add**.
    *   **(Optional) Another Data Engineer (Member):**
        *   Click **+ Add people or groups**.
        *   **Enter name or email:** `junior.data.engineer@valleygeneral.org` (replace).
        *   **Role:** Select **Member**. (Members can do most things Contributor can, plus publish Power BI apps and share content. They still cannot manage workspace access).
        *   Click **Add**.
3.  **Review Assigned Roles:**
    *   In the "Manage access" pane, you should now see the list of users and their assigned roles for the workspace.

**Part 3: Share a Specific Power BI Report with Row-Level Security (RLS)**

*   Assume you have a Power BI report in this workspace (e.g., `ED_Performance_Dashboard`) that has RLS configured (e.g., a "DepartmentalView" role that filters data by department). If you don't have one, you can quickly create a dummy report or use one from a previous lab and imagine RLS is set up.

1.  **Create or Identify a Sample Report:**
    *   If you don't have a report in `ED_Efficiency_Project_Q2`, quickly create one:
        *   Go to the workspace, click **+ New -> Report**.
        *   Choose "Pick a published dataset" (if you have one) or "Paste or manually enter data" to create a very simple report with one visual.
        *   Save it as `ED_Department_Summary_Report`.
2.  **Share the Report with Specific Permissions:**
    *   In the `ED_Efficiency_Project_Q2` workspace, find your `ED_Department_Summary_Report`.
    *   Click the three dots (**...**) next to the report name.
    *   Select **Share**.
    *   In the "Share report" dialog:
        *   **Enter name or email:** `specific.stakeholder@valleygeneral.org` (a user who is NOT already a member of the workspace with full view access, or a Viewer who needs build permissions on this specific report's dataset).
        *   **Allow recipients to share this report:** Uncheck this for tighter control.
        *   **Allow recipients to build content with the data associated with this report:** Check this if you want this specific stakeholder to be able to create their own reports using the underlying dataset (grants "Build" permission on the dataset). For a clinical lead who might want to explore, this could be useful.
        *   **Send an email notification:** Optional.
        *   Click **Grant access**.
    *   This demonstrates sharing a specific item, potentially with different permissions than the user's general workspace role.

3.  **Apply Sensitivity Label to the Report:**
    *   Open the `ED_Department_Summary_Report`.
    *   Go to **File -> Sensitivity label**.
    *   Choose an appropriate label, for example, "Confidential - Internal Use" or "HIPAA-HIGH" if it contains PHI (even aggregated).
    *   Click **Apply**.

**Part 4: (Conceptual) Configure Quarterly Access Reviews**

*   Access reviews are typically configured in **Microsoft Entra ID (Azure AD) Privileged Identity Management (PIM)** or through **Entra ID Access Reviews** features, often by an Identity Administrator or Compliance Administrator, not directly within the Fabric workspace UI by a data engineer.

1.  **Understanding the Process:**
    *   **Purpose:** To regularly review who has access to sensitive resources (like your Fabric workspace or specific roles within it) and ensure that access is still necessary and appropriate. This is a key compliance control.
    *   **Setup (Admin Task in Entra ID):**
        *   An administrator would go to the Microsoft Entra admin center.
        *   Navigate to "Identity Governance" -> "Access Reviews."
        *   Create a new access review.
        *   **Scope:** Define what is being reviewed (e.g., members of an AAD group that has been granted access to the Fabric workspace, or direct assignments to the workspace if supported for review).
        *   **Reviewers:** Assign who will perform the review (e.g., the workspace owner like `data.engineer.lead@valleygeneral.org`, or their manager).
        *   **Frequency:** Set it to occur quarterly.
        *   **Actions upon completion:** Define what happens if access is denied during the review (e.g., access is automatically removed).
    *   **Performing the Review:**
        *   When the review period starts, the assigned reviewers receive notifications.
        *   They go to the "My Access" portal or Entra ID to approve or deny access for each user/group in scope.

2.  **Your Role as Workspace Admin/Data Engineer:**
    *   You might be assigned as a reviewer for your workspace.
    *   You need to understand the importance of these reviews and participate diligently to maintain compliance.

**Expected Outcome / Deliverables:**
*   A new Fabric workspace named `ED_Efficiency_Project_Q2` is created.
*   Simulated team members are assigned appropriate workspace roles (Admin, Contributor, Viewer).
*   A Power BI report within the workspace is shared with a specific user, potentially with different permissions than their workspace role.
*   The shared Power BI report has a sensitivity label applied.
*   A conceptual understanding of how quarterly access reviews are set up and their importance for governance.

**Questions from Manual & Answers:**

*   **Q1: Why is it generally better to assign analysts the "Contributor" role rather than the "Member" role if their primary job is to create reports and datasets but not manage workspace access or publish apps?**
    *   **A1:** The "Contributor" role aligns more closely with the principle of least privilege for analysts whose main tasks are content creation and editing.
        *   **Contributors can:** Create, edit, and delete content they have access to (reports, datasets, dataflows, notebooks), and publish reports to the workspace. This is usually sufficient for an analyst's development tasks.
        *   **Contributors cannot:** Manage workspace access for other users, modify workspace settings, or publish/manage Power BI Apps for the workspace.
        *   **Members can** do everything a Contributor can, PLUS they can publish Power BI apps and share content more broadly within the workspace context (e.g., update an app).
        *   By assigning "Contributor," you limit the potential for analysts to inadvertently change workspace settings, manage permissions incorrectly, or impact the distribution of content via Apps if that's not part of their designated responsibilities. It provides a safer scope for their work.

*   **Q2: Which Microsoft Purview tool or Fabric feature is primarily used to verify who has accessed or modified specific data items or reports within a workspace?**
    *   **A2:** The **Microsoft Purview Audit log** (accessed via the Microsoft Purview compliance portal) is the primary tool for verifying who has accessed or modified specific data items (like Lakehouse tables, datasets) or reports within a Fabric workspace.
        *   Fabric activities are logged to the unified audit log. Administrators or compliance officers can search these logs for activities related to specific Fabric items, users, and timeframes.
        *   While the Fabric Monitoring Hub shows operational logs for pipeline runs and Spark jobs, the Purview Audit log is the authoritative source for compliance-related access and modification tracking.

*   **Q3: How do Sensitivity Labels affect the ability to share Fabric content (like a Power BI report) externally or download it?**
    *   **A3:** Sensitivity Labels themselves are primarily for classification and visual marking. Their direct effect on sharing and downloading is determined by **Data Loss Prevention (DLP) policies** and **Conditional Access policies** that are configured (usually in Microsoft Purview and Azure AD) to act upon these labels:
        *   **Trigger for DLP Policies:** If a DLP policy is in place that targets a specific sensitivity label (e.g., "HIPAA-HIGH"), it can:
            *   **Block external sharing:** Prevent users from sharing reports or datasets labeled "HIPAA-HIGH" with users outside the organization.
            *   **Block download:** Prevent users from downloading the report or its data to unmanaged/personal devices.
            *   **Audit actions:** Log attempts to share or download, even if not blocked.
            *   **Display policy tips:** Warn users about the sensitivity of the data when they attempt certain actions.
        *   **Inform Conditional Access Policies:** Azure AD Conditional Access policies can potentially use information about the sensitivity of data being accessed (though this is more common for SharePoint/OneDrive currently) to enforce stricter access controls (e.g., requiring MFA, blocking access from untrusted networks if a user is trying to access a "Highly Confidential" report).
        *   **User Awareness:** The label itself makes users more aware of the data's sensitivity, potentially making them more cautious about sharing or downloading it.
        *   **Inheritance:** Labels can be inherited from datasets to reports, and if content is exported (e.g., to Excel), the label and any associated encryption/protection can persist.

        So, while the label itself is a tag, its power to restrict sharing/download comes from the associated governance policies that reference it.

This lab covers the practical aspects of setting up a collaborative environment in Fabric, emphasizing role-based access and the application of governance features like sensitivity labels.

### Lab 10.8: Customize a Workspace and Integrate External Data via OneLake Shortcuts

**Module Alignment:** Section 10: Advanced Features and Customization

**Objective:**
*   Create a OneLake shortcut to an external data source (simulated as another path within your OneLake, or conceptually an Azure Data Lake Storage Gen2 account).
*   Ingest and transform data accessed via the shortcut into the Silver layer of a Fabric Lakehouse.
*   Build a Power BI report using this integrated data, applying custom branding/themes.
*   Apply a sensitivity label to the dataset and report.
*   (Conceptual) Understand how webhooks could be used to notify upon new data arrival in the external source.

**Scenario:**
Valley General Hospital's oncology department collaborates with an external research institute that stores de-identified clinical trial data in their own Azure Data Lake Storage Gen2 (ADLS Gen2) account. The oncology team needs to analyze this trial data alongside their internal patient data. You are tasked with creating a OneLake shortcut to this external data, integrating it into the `CardiologyLakehouse` (we'll use this existing Lakehouse for simplicity, though in reality, an `OncologyLakehouse` might be more appropriate), and creating a themed Power BI report.

**Prerequisites:**
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `CardiologyLakehouse`).
*   Permissions to create shortcuts in your Lakehouse.
*   (For a real ADLS Gen2 shortcut) An actual ADLS Gen2 account with a container and sample data, and appropriate credentials (e.g., account key, SAS token, or service principal with permissions).
    *   **For this lab, we will simulate the "external" source by creating data in a different folder path within your existing OneLake/Lakehouse to avoid needing external Azure resources. The shortcut creation process is similar.**
*   A sample Power BI report or the ability to create one.
*   Sensitivity labels available in Fabric.

**Tools to be Used:**
*   Microsoft Fabric Lakehouse (Shortcuts, SQL Analytics Endpoint)
*   Microsoft Fabric Notebook (PySpark for data preparation and transformation)
*   Microsoft Fabric Power BI (for report creation and theming)

**Estimated Time:** 75 - 90 minutes

**Tasks / Step-by-Step Instructions:**

**Part 1: Simulate and Prepare "External" Data Source in OneLake**

1.  **Open or Create a Notebook:**
    *   In your `DEV_CardiologyAnalytics` workspace, open an existing notebook or create a new one (e.g., `ExternalData_Integration_Oncology`).
    *   Ensure your `CardiologyLakehouse` is attached.

2.  **Create Sample "External" Clinical Trial Data in a Separate Lakehouse Path:**
    *   This simulates data residing in an external ADLS Gen2. We'll write it to a distinct folder within the `Files` section of your `CardiologyLakehouse`.
    ```python
    from pyspark.sql import Row
    from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DateType, DoubleType

    # Define schema for the simulated clinical trial data
    schema_trial_data = StructType([
        StructField("TrialID", StringType(), False),
        StructField("ExternalPatientID", StringType(), False), # De-identified patient ID from research institute
        StructField("EnrollmentDate", DateType(), True),
        StructField("TreatmentArm", StringType(), True), # e.g., 'StandardCare', 'InvestigationalDrugA'
        StructField("AgeAtEnrollment", IntegerType(), True),
        StructField("ResponseMetric", DoubleType(), True), # e.g., tumor size reduction %
        StructField("AdverseEventReported", StringType(), True) # 'Yes' or 'No'
    ])

    # Sample data
    trial_data = [
        Row(TrialID="ONC001", ExternalPatientID="RSRCH_PAT_101", EnrollmentDate="2022-01-15", TreatmentArm="InvestigationalDrugA", AgeAtEnrollment=65, ResponseMetric=0.25, AdverseEventReported="No"),
        Row(TrialID="ONC001", ExternalPatientID="RSRCH_PAT_102", EnrollmentDate="2022-02-01", TreatmentArm="StandardCare", AgeAtEnrollment=70, ResponseMetric=0.10, AdverseEventReported="Yes"),
        Row(TrialID="ONC001", ExternalPatientID="RSRCH_PAT_103", EnrollmentDate="2022-01-20", TreatmentArm="InvestigationalDrugA", AgeAtEnrollment=58, ResponseMetric=0.35, AdverseEventReported="No"),
        Row(TrialID="ONC002", ExternalPatientID="RSRCH_PAT_201", EnrollmentDate="2022-03-10", TreatmentArm="InvestigationalDrugB", AgeAtEnrollment=62, ResponseMetric=0.15, AdverseEventReported="Yes")
    ]

    df_simulated_external_trial_data = spark.createDataFrame(trial_data, schema_trial_data)

    # Define a path within your Lakehouse Files to simulate the external location
    # This path will be the target for our OneLake shortcut later.
    simulated_external_path = "Files/SimulatedExternalResearchData/ClinicalTrials/oncology_trial_data_source" # Using Parquet format

    df_simulated_external_trial_data.write.format("parquet").mode("overwrite").save(simulated_external_path)
    
    print(f"Simulated external clinical trial data saved to: {simulated_external_path}")
    display(df_simulated_external_trial_data)
    ```    *   Run the cell. This creates Parquet files in the specified `Files` path within your `CardiologyLakehouse`. This path (`SimulatedExternalResearchData/ClinicalTrials/oncology_trial_data_source`) will act as our "external ADLS Gen2" source for the shortcut.

**Part 2: Create a OneLake Shortcut to the "External" Data**

1.  **Navigate to your Lakehouse:** Open `CardiologyLakehouse`.
2.  **Create a New Shortcut:**
    *   In the Lakehouse explorer view, under `Tables` or `Files` (location doesn't strictly matter for where you initiate, but it will appear as a folder-like item), click the three dots (**...**) next to `Files` (or the root of the Lakehouse name on the left pane).
    *   Select **New shortcut**.
3.  **Configure the Shortcut:**
    *   In the "New shortcut" dialog:
        *   **Select data source type:** Choose **Microsoft OneLake**.
            *   *(If this were a real external ADLS Gen2, you would select "Azure Data Lake Storage Gen2". The configuration steps would then ask for Account URL, authentication method (Account Key, SAS, Service Principal, Org Account), etc.)*
        *   **Shortcut name:** `External_Oncology_Trials`
        *   **Connection:** Since we chose "Microsoft OneLake", it will ask for the path within OneLake.
            *   You need to navigate to the path where you saved the simulated external data.
            *   Click **Browse**.
            *   Select your current workspace (`DEV_CardiologyAnalytics`).
            *   Select your Lakehouse (`CardiologyLakehouse`).
            *   Navigate into the `Files` directory, then `SimulatedExternalResearchData`, then `ClinicalTrials`.
            *   Select the folder `oncology_trial_data_source` (this is the folder containing the Parquet files).
            *   Click **Select**.
        *   The "Path" field should now be populated (e.g., `DEV_CardiologyAnalytics.CardiologyLakehouse/Files/SimulatedExternalResearchData/ClinicalTrials/oncology_trial_data_source`).
    *   Click **Create**.
    *   You should now see `External_Oncology_Trials` listed in your Lakehouse explorer (likely under `Files` or as a top-level item depending on where you initiated it). It will have a different icon indicating it's a shortcut.

**Part 3: Load and Transform Data from the Shortcut into Silver Layer**

1.  **Access Shortcut Data in Notebook:**
    *   Go back to your `ExternalData_Integration_Oncology` notebook (or create a new one).
    *   In a PySpark cell, read the data from the shortcut. The shortcut path in Fabric is typically `[LakehouseName]/[ShortcutName]`.
    ```python
    # Path to the shortcut within the Lakehouse context
    # The shortcut itself points to the 'oncology_trial_data_source' folder which contains parquet files.
    shortcut_path_in_lakehouse = "CardiologyLakehouse.External_Oncology_Trials" 
    # Note: When reading, Spark needs the path to the actual data files, not just the shortcut name if it's a folder.
    # OneLake paths for shortcuts are usually like: /<WorkspaceName>/<LakehouseName>.Lakehouse/<ShortcutName>
    # For direct Spark access, you might need the full OneLake path:
    # For this lab, since the shortcut points to a folder of parquet files, we can read it as such.
    # If the shortcut was to a specific file, the path would be direct to that file.
    # If the shortcut was to a table in another Lakehouse, you'd use spark.read.table("OtherLakehouse.ShortcutToTable")

    # Let's try reading the shortcut as if it's a directory of parquet files
    # The path for Spark will be relative to the Lakehouse root if the shortcut is at the root,
    # or relative to Files/Tables if it's under them.
    # OneLake path for the shortcut (if shortcut is at Lakehouse root):
    # /External_Oncology_Trials (this is the folder of parquet files)
    
    # Correct path for reading data via shortcut (assuming shortcut is at Lakehouse root or under Files)
    # The table access via SQL endpoint would be `CardiologyLakehouse`.`External_Oncology_Trials` if it was a table shortcut.
    # For a file/folder shortcut, we use the path.
    # The shortcut "External_Oncology_Trials" in the Lakehouse explorer points to the folder containing Parquet files.
    # So, we can read this folder.
    
    try:
        # The shortcut 'External_Oncology_Trials' itself represents the folder containing parquet files.
        df_trial_data_from_shortcut = spark.read.format("parquet").load(f"CardiologyLakehouse/External_Oncology_Trials")
        
        df_trial_data_from_shortcut.printSchema()
        display(df_trial_data_from_shortcut.limit(5))

        # Perform transformations (e.g., rename columns, add audit columns)
        df_silver_oncology_trials = df_trial_data_from_shortcut.select(
            col("TrialID").alias("trial_id"),
            col("ExternalPatientID").alias("external_patient_id"),
            col("EnrollmentDate").alias("enrollment_date"),
            col("TreatmentArm").alias("treatment_arm"),
            col("AgeAtEnrollment").alias("age_at_enrollment"),
            col("ResponseMetric").alias("response_metric"),
            col("AdverseEventReported").alias("adverse_event_reported"),
            lit("OncologyResearchShortcut").alias("source_system"),
            current_timestamp().alias("silver_load_timestamp")
        )

        df_silver_oncology_trials.printSchema()
        display(df_silver_oncology_trials.limit(5))

        # Write to a Silver layer table in your CardiologyLakehouse
        df_silver_oncology_trials.write.format("delta").mode("overwrite").saveAsTable("CardiologyLakehouse.silver_oncology_trials")
        print("silver_oncology_trials table created successfully from shortcut data.")

    except Exception as e:
        print(f"Error reading from shortcut or processing data: {e}")
        print("Ensure the shortcut 'External_Oncology_Trials' points to the Parquet files directory.")
        print("The path for spark.read.load() should be like 'LakehouseName/ShortcutName' if the shortcut is a folder of files.")

    ```
    *   Run the cell. This reads the data *through the shortcut* (without copying it into your primary Lakehouse storage for the shortcut itself), transforms it, and saves it as a new Delta table `silver_oncology_trials` in your `CardiologyLakehouse`.

**Part 4: Build a Power BI Report with Custom Theming**

1.  **Create a New Power BI Dataset:**
    *   In `CardiologyLakehouse`, click **New Power BI dataset**.
    *   Select the new `silver_oncology_trials` table. You might also select `dim_patient` if you plan to (conceptually) link internal patients to external trial IDs later. For now, just `silver_oncology_trials` is fine.
    *   Click **Confirm**.
    *   Rename the dataset to `OncologyTrialAnalytics_Dataset`.
2.  **Create a New Report:**
    *   With `OncologyTrialAnalytics_Dataset` open, click **Create report -> Start from scratch**.
3.  **Add Visuals:**
    *   **Bar Chart: Average Response Metric by Treatment Arm:**
        *   X-axis: `silver_oncology_trials[treatment_arm]`
        *   Y-axis: `silver_oncology_trials[response_metric]` (set aggregation to Average)
    *   **Pie Chart: Adverse Event Reported:**
        *   Values: `silver_oncology_trials[external_patient_id]` (Count)
        *   Legend: `silver_oncology_trials[adverse_event_reported]`
    *   **Table: Trial Details:**
        *   Include columns like `trial_id`, `external_patient_id`, `enrollment_date`, `treatment_arm`, `response_metric`.
4.  **Apply Custom Theming:**
    *   In the Power BI report editing view (within Fabric):
        *   Go to the **View** tab on the ribbon.
        *   In the "Themes" group, click the dropdown arrow.
        *   You can select a built-in theme.
        *   **To customize further (or import a theme JSON):**
            *   Click **Browse for themes**. If you have a JSON theme file (often created in Power BI Desktop or downloaded), you can import it.
            *   Alternatively, click **Customize current theme**.
            *   In the "Customize theme" dialog:
                *   **Name and colors:** Change primary colors to match Valley General Hospital's branding (e.g., a specific blue and green).
                *   **Text:** Adjust font families, sizes, and colors for titles, cards, KPIs, and tab headers.
                *   **Visuals:** Customize background, border, header, and tooltip settings for visuals.
                *   **Page:** Set page background.
                *   **Filter pane:** Customize the look of the filter pane.
            *   Click **Apply**.
    *   Observe how your report visuals update with the new theme.
5.  **Save the Report:**
    *   Click **File -> Save**.
    *   Name: `Oncology Clinical Trial Insights`.
    *   Ensure it's saved to your `DEV_CardiologyAnalytics` workspace.

**Part 5: Apply Sensitivity Label**

1.  **Label the Dataset and Report:**
    *   Navigate to `OncologyTrialAnalytics_Dataset` in your workspace, go to **Settings**, and apply an appropriate sensitivity label (e.g., "Confidential - Research Data" or "HIPAA-HIGH" if it contains any re-identifiable linked data, though this scenario implies de-identified external IDs).
    *   Open the `Oncology Clinical Trial Insights` report and ensure the label is applied or apply it via **File -> Sensitivity label**.

**Part 6: (Conceptual) Webhook for New Data Notification**

*   If the external research institute's ADLS Gen2 had an Azure Event Grid subscription, it could publish an event whenever new trial data files are added to their `oncology_trial_data_source` folder.
*   **How it would work:**
    1.  **Event Grid on External ADLS Gen2:** The research institute configures Event Grid on their storage account to monitor for "Blob Created" events in the specific data path.
    2.  **Event Subscription:** They create an event subscription that sends these events to an endpoint you control. This endpoint could be:
        *   An **Azure Function** or **Logic App** in your Azure subscription.
    3.  **Action upon Event:**
        *   The Azure Function/Logic App receives the event (which includes the path to the new file).
        *   It could then:
            *   Trigger your Fabric Data Factory pipeline (via its REST API) that refreshes the `silver_oncology_trials` table by re-reading the shortcut.
            *   Send a notification (e.g., email, Teams message) to the oncology analytics team that new data is available.
            *   Log the event for auditing.
*   This creates an event-driven architecture, automating the refresh process when new external data arrives. The setup of the Event Grid and the consuming Function/Logic App is outside Fabric itself but integrates with Fabric pipelines.

**Expected Outcome / Deliverables:**
*   A OneLake shortcut (`External_Oncology_Trials`) created in `CardiologyLakehouse` pointing to the simulated external data path.
*   A Silver layer table (`silver_oncology_trials`) in `CardiologyLakehouse` populated with data read through the shortcut.
*   A Power BI report (`Oncology Clinical Trial Insights`) built on this data, featuring custom theming to match organizational branding.
*   The Power BI dataset and report are classified with an appropriate sensitivity label.
*   A conceptual understanding of how webhooks and event-driven architecture could be used to automate data refresh from external sources.

**Questions from Manual & Answers: LINK TO HTML?**

*   **Q1: What is the key benefit of using a OneLake shortcut to access data in an external storage system (like Azure Data Lake Gen2 or Amazon S3) compared to directly ingesting and copying all the data into your primary Lakehouse storage?**
    *   **A1:** The key benefit is **accessing data in place without data duplication or movement.**
        *   **No Data Duplication:** The shortcut acts as a symbolic link or pointer to the data in its original location. The data is not copied into your Fabric workspace's primary OneLake storage. This saves storage costs and avoids managing multiple copies of the same data.
        *   **Single Source of Truth:** Analytics are performed on the data residing in the external system, ensuring users are always working with the latest version from the source (unless a refresh/ingestion to a silver table is done, but the shortcut itself points to live external data).
        *   **Simplified Data Governance (for the source):** The source system maintains control and governance over its data. The shortcut respects the permissions set on the source.
        *   **Faster Access for Exploration:** Users can quickly start exploring and analyzing data from external systems via shortcuts without waiting for lengthy ETL processes to copy data.
        *   **Reduced ETL Complexity for Some Scenarios:** For direct querying or ad-hoc analysis on external data, shortcuts can simplify the initial setup. (Note: For performance or complex transformations, you might still ingest from the shortcut into Silver/Gold tables within your Lakehouse).

*   **Q2: Why is Git integration important for managing versions of Fabric items like Notebooks or Power BI report definitions (as PBIX/PBIP files)?**
    *   **A2:** Git integration is important for:
        *   **Version Control:** Tracks changes to code (Notebooks) and report definitions over time. You can see who changed what, when, and why. This is crucial for understanding the evolution of an asset.
        *   **Rollback Capabilities:** If a recent change introduces errors or undesirable behavior, you can easily revert to a previous stable version of the Notebook or report definition.
        *   **Collaboration:** Facilitates teamwork by allowing multiple developers/analysts to work on the same items, merge their changes, and resolve conflicts in a structured way using branches and pull requests.
        *   **Auditing and History:** Provides a complete history of all modifications, which is valuable for compliance, debugging, and understanding the development lifecycle.
        *   **CI/CD (Continuous Integration/Continuous Deployment):** Enables automated testing and deployment of Fabric items. Changes committed to Git can trigger pipelines that deploy Notebooks or Power BI reports to different environments (Dev, Test, Prod).
        *   **Code Reviews:** Pull request mechanisms in Git platforms (like GitHub, Azure DevOps) allow for peer review of code and report changes before they are merged into the main branch, improving quality.
        *   **Reproducibility:** Ensures that you can recreate a specific version of an analytical asset or ML model training script from a particular point in time.

*   **Q3: How can webhooks or event-driven architectures (e.g., using Azure Event Grid with Fabric) support automation and real-time responsiveness in a healthcare data platform?**
    *   **A3:** Webhooks and event-driven architectures can support automation and real-time responsiveness by:
        *   **Automated Pipeline Triggers:** When new data arrives in a source system (e.g., a new HL7 file lands in a storage account, a new FHIR resource is created, an IoT device sends a critical alert), an event can be published. A Fabric Data Factory pipeline can subscribe to this event and automatically trigger an ingestion or processing job. This eliminates the need for polling or fixed schedule-based triggers, making data available faster.
        *   **Real-time Notifications and Alerts:** Critical events (e.g., a patient's lab result exceeding a dangerous threshold, an AI model predicting high risk for a patient, a system failure) can trigger webhooks that send immediate notifications to clinicians, care teams, or IT support via email, SMS, or Teams messages.
        *   **Dynamic Resource Scaling:** Events indicating high load or processing demand could potentially trigger automation to scale up Fabric capacities or other Azure resources.
        *   **Synchronizing Systems:** When data is updated in one system, an event can trigger processes to update related data in other downstream systems or analytical models, ensuring consistency.
        *   **Triggering AI Model Retraining:** The arrival of a significant batch of new data (signaled by an event) could automatically trigger a pipeline to retrain relevant machine learning models.
        *   **Enhanced Monitoring:** Events related to pipeline failures, data quality issues, or security alerts can be routed to monitoring dashboards or incident management systems for immediate attention.
        This shifts the paradigm from batch-oriented processing to a more reactive and near real-time data ecosystem.

This lab covers some powerful advanced features. The shortcut mechanism is a key differentiator for OneLake, and theming helps with user adoption and branding of BI solutions.

This lab will guide the learner to apply the concepts and architectures discussed in the case studies to a new, specific problem. It will involve more design thinking and less prescriptive coding, but will still require using Fabric components.

---

### Lab 11.8: Apply Case Study Architecture to Address Appointment No-Shows

**Module Alignment:** Section 11: Case Studies and Real-World Applications

**Objective:**
*   Analyze a common healthcare problem (appointment no-shows) through the lens of the architectures and solutions presented in the course case studies.
*   Design a high-level Microsoft Fabric solution to ingest relevant data, build a predictive model for no-show risk, and visualize insights for clinic schedulers.
*   Identify key Fabric components (Data Factory, Lakehouse, Notebooks, Power BI, Purview) and their roles in the proposed solution.
*   Consider data governance, collaboration, and potential challenges in implementing such a solution.

**Scenario:**
Valley General Hospital's Cardiology Clinic is experiencing a high rate of patient no-shows for scheduled appointments. This leads to wasted provider time, underutilized resources, and potential delays in patient care. The clinic manager wants to implement a data-driven solution using Microsoft Fabric to predict which appointments are at high risk of being a no-show and to understand the contributing factors.

**Prerequisites:**
*   Completion and understanding of previous labs and sections, especially those covering:
    *   Data ingestion (Lab 3.8)
    *   Data modeling (Lab 4.8)
    *   Machine learning (Lab 7.8)
    *   Power BI reporting (Lab 6.8)
    *   Collaboration and Governance (Labs 5.9, 9.8)
*   Microsoft Fabric Workspace (e.g., `DEV_CardiologyAnalytics`).
*   Microsoft Fabric Lakehouse (e.g., `CardiologyLakehouse`).
*   Ability to create and conceptually design pipelines, notebooks, and reports.

**Tools to be Used (Conceptual Design & Partial Implementation):**
*   Microsoft Fabric Data Factory (for data ingestion design)
*   Microsoft Fabric Lakehouse (for data storage and modeling design)
*   Microsoft Fabric Notebook (for ML model development design and potential feature engineering)
*   Microsoft Fabric Power BI (for dashboard design)
*   Microsoft Purview (for governance considerations)

**Estimated Time:** 90 - 120 minutes (This is a design and partial implementation lab)

**Tasks / Step-by-Step Instructions:**

**Part 1: Problem Definition and Data Discovery (Design Thinking)**

1.  **Understand the Goal:**
    *   Primary Goal: Reduce appointment no-shows in the Cardiology Clinic.
    *   Secondary Goals: Understand factors contributing to no-shows, optimize scheduling, improve resource utilization.
2.  **Identify Key Data Sources and Elements (Brainstorming):**
    *   What data would be relevant to predict no-shows? List them out.
        *   *Example Answer:*
            *   **Appointment System:** `Appointment_ID`, `Patient_ID`, `Scheduled_DateTime`, `Appointment_Type` (New, Follow-up), `Provider_ID`, `Clinic_Location`, `Lead_Time_Days` (days between booking and appointment), `Day_Of_Week`.
            *   **EHR/Patient Master:** `Patient_ID`, `Age`, `Gender`, `Zip_Code` (for distance/socioeconomic factors), `Communication_Preferences` (Email, SMS), `Insurance_Type`.
            *   **Historical Appointment Data:** `Patient_ID`, `Appointment_DateTime`, `Actual_Status` (Attended, No-Show, Cancelled), `Cancellation_Reason` (if available). This is crucial for the target variable and historical features.
            *   **(Optional) External Data:** Weather forecasts for appointment day, public transit information.
3.  **Define the Target Variable for ML:**
    *   What are you trying to predict?
        *   *Example Answer:* A binary variable `Is_NoShow` (True/False or 1/0) for each future scheduled appointment.
4.  **Consider Potential Features for the ML Model:**
    *   From the data elements above, which ones could be good predictors?
        *   *Example Answer:* History of no-shows for the patient, lead time, appointment type, day of the week, patient age, distance to clinic (derived from zip code).

**Part 2: Design the Fabric Solution Architecture (High-Level)**

1.  **Sketch a Medallion Architecture for this problem:**
    *   **Bronze Layer:** Where will raw data from the Appointment System and EHR land? What format?
        *   *Example Answer:* `bronze_appointments_raw` (from scheduling system API/DB), `bronze_patient_demographics_raw` (from EHR DB). Stored as Delta tables.
    *   **Silver Layer:** What conformed, cleansed tables will you create?
        *   *Example Answer:* `silver_appointments` (cleaned, with lead time calculated), `silver_patients` (relevant demographics), `silver_historical_attendance` (derived from past appointments, including no-show flags).
    *   **Gold Layer:** What table will be the input for your ML model and BI dashboard?
        *   *Example Answer:* `gold_appointment_features_for_ml` (aggregated features per patient/appointment), `gold_no_show_predictions` (output from ML model), `dim_provider_clinic_schedule` (for BI).
2.  **Identify Fabric Components and their Roles:**
    *   **Data Ingestion:** How will data get from source systems to Bronze? (Data Factory Pipelines, Notebooks for custom APIs).
    *   **Data Transformation (Bronze -> Silver -> Gold):** (Notebooks with PySpark, Dataflows Gen2).
    *   **Machine Learning:** (Notebooks with PySpark/Python & scikit-learn, MLflow).
    *   **Reporting/Visualization:** (Power BI).
    *   **Governance:** (Purview for lineage/classification, Fabric RBAC).
    *   **Orchestration:** (Data Factory Pipelines).

**Part 3: Partial Implementation - Data Ingestion and Silver Layer (Focus on Appointments)**

1.  **Create Sample "Appointment System" Data (Notebook):**
    *   In a Fabric Notebook (e.g., `NoShow_Prediction_Project`), simulate raw appointment data and save it to a Bronze table.
    ```python
    from pyspark.sql import Row
    from pyspark.sql.functions import col, lit, to_date, current_timestamp, datediff, expr
    from pyspark.sql.types import StructType, StructField, StringType, TimestampType, IntegerType, BooleanType

    # Schema for simulated raw appointment data
    schema_raw_appointments = StructType([
        StructField("RawAppointmentID", StringType(), False),
        StructField("PatientSystemID", StringType(), False),
        StructField("ScheduledTimestamp", TimestampType(), True),
        StructField("AppointmentTypeRaw", StringType(), True), # e.g., "NewPat", "FollowUpVisit"
        StructField("ProviderCode", StringType(), True),
        StructField("BookingTimestamp", TimestampType(), True),
        StructField("HistoricalStatus", StringType(), True) # 'Attended', 'NoShow', 'Cancelled', 'Scheduled' (for future)
    ])

    # Sample raw data
    raw_app_data = [
        Row(RawAppointmentID="APP_XYZ_001", PatientSystemID="P001", ScheduledTimestamp="2023-07-10T10:00:00", AppointmentTypeRaw="NewPat", ProviderCode="DR_SMITH", BookingTimestamp="2023-06-01T14:00:00", HistoricalStatus="Attended"),
        Row(RawAppointmentID="APP_XYZ_002", PatientSystemID="P002", ScheduledTimestamp="2023-07-10T11:00:00", AppointmentTypeRaw="FollowUpVisit", ProviderCode="DR_JONES", BookingTimestamp="2023-06-15T09:00:00", HistoricalStatus="NoShow"),
        Row(RawAppointmentID="APP_XYZ_003", PatientSystemID="P001", ScheduledTimestamp="2023-07-11T09:30:00", AppointmentTypeRaw="FollowUpVisit", ProviderCode="DR_SMITH", BookingTimestamp="2023-07-01T16:00:00", HistoricalStatus="Attended"),
        Row(RawAppointmentID="APP_XYZ_004", PatientSystemID="P003", ScheduledTimestamp="2023-07-12T14:00:00", AppointmentTypeRaw="NewPat", ProviderCode="DR_BROWN", BookingTimestamp="2023-05-20T10:00:00", HistoricalStatus="Scheduled"), # Future
        Row(RawAppointmentID="APP_XYZ_005", PatientSystemID="P002", ScheduledTimestamp="2023-08-01T15:00:00", AppointmentTypeRaw="FollowUpVisit", ProviderCode="DR_JONES", BookingTimestamp="2023-07-10T11:30:00", HistoricalStatus="Scheduled")  # Future
    ]
    df_bronze_appointments = spark.createDataFrame(raw_app_data, schema_raw_appointments)
    df_bronze_appointments.write.format("delta").mode("overwrite").saveAsTable("CardiologyLakehouse.bronze_appointments_raw")
    print("bronze_appointments_raw table created.")
    display(df_bronze_appointments)
    ```
2.  **Transform to `silver_appointments` (Notebook):**
    *   Clean data, calculate lead time, define the `Is_NoShow` target variable for historical data.
    ```python
    df_bronze = spark.read.table("CardiologyLakehouse.bronze_appointments_raw")

    df_silver_appointments = df_bronze.withColumn("appointment_id", col("RawAppointmentID")) \
        .withColumn("patient_id", col("PatientSystemID")) \
        .withColumn("scheduled_datetime", col("ScheduledTimestamp")) \
        .withColumn("appointment_type", when(col("AppointmentTypeRaw") == "NewPat", "New Patient")
                                     .when(col("AppointmentTypeRaw") == "FollowUpVisit", "Follow-Up")
                                     .otherwise("Other")) \
        .withColumn("provider_id", col("ProviderCode")) \
        .withColumn("booking_datetime", col("BookingTimestamp")) \
        .withColumn("lead_time_days", datediff(to_date(col("ScheduledTimestamp")), to_date(col("BookingTimestamp")))) \
        .withColumn("day_of_week", date_format(col("ScheduledTimestamp"), "EEEE")) \
        .withColumn("is_historical_no_show", when(col("HistoricalStatus") == "NoShow", True).otherwise(False)) \
        .withColumn("is_historical_attended", when(col("HistoricalStatus") == "Attended", True).otherwise(False)) \
        .withColumn("is_future_appointment", when(col("HistoricalStatus") == "Scheduled", True).otherwise(False)) \
        .withColumn("silver_load_timestamp", current_timestamp()) \
        .select("appointment_id", "patient_id", "scheduled_datetime", "appointment_type", 
                "provider_id", "booking_datetime", "lead_time_days", "day_of_week",
                "is_historical_no_show", "is_historical_attended", "is_future_appointment", "silver_load_timestamp")

    df_silver_appointments.write.format("delta").mode("overwrite").option("overwriteSchema","true").saveAsTable("CardiologyLakehouse.silver_appointments")
    print("silver_appointments table created.")
    display(df_silver_appointments)
    ```

**Part 4: Design the ML Model and Prediction Output (Conceptual)**

1.  **Feature Engineering for `gold_appointment_features_for_ml`:**
    *   What features would you create from `silver_appointments` and a (hypothetical) `silver_patients` table?
        *   *Example:* `patient_past_no_show_rate`, `patient_total_appointments`, `is_first_appointment_with_provider`, `avg_lead_time_for_provider_type`.
2.  **Model Choice:**
    *   What type of model? (e.g., Logistic Regression, RandomForest, Gradient Boosting).
3.  **Prediction Output Table (`gold_no_show_predictions`):**
    *   What columns should it have? (`Appointment_ID`, `Patient_ID`, `Scheduled_DateTime`, `NoShow_Risk_Score` (probability), `Predicted_Is_NoShow` (binary flag), `Prediction_Timestamp`).

**Part 5: Design the Power BI Dashboard (Conceptual)**

1.  **Key Visuals and KPIs:**
    *   What information should schedulers see?
        *   *Example:* List of upcoming appointments color-coded by no-show risk score, overall predicted no-show rate for next week, top factors contributing to risk (if model is explainable), trend of no-show rates.
2.  **Interactivity:**
    *   How can users filter or drill down? (By provider, date range, appointment type).
3.  **Actions:**
    *   Could the dashboard link to actions? (e.g., trigger a reminder workflow for high-risk appointments).

**Part 6: Governance and Collaboration Considerations**

1.  **Sensitivity Labels:** What labels for Bronze, Silver, Gold tables and the Power BI report?
    *   *Example:* `bronze_appointments_raw` (Confidential - PHI), `silver_appointments` (Confidential - PHI), `gold_no_show_predictions` (Confidential - PHI), Power BI Report (Confidential - Internal Use).
2.  **RBAC:** Who needs what access to the workspace and specific items?
    *   Data Engineers (Admin/Member), Data Scientists (Contributor for ML notebooks/models), Schedulers/Clinic Managers (Viewer for Power BI report, potentially with RLS if providers are also viewers).
3.  **Data Quality Monitoring:** How would you monitor the quality of incoming appointment data?
4.  **Model Monitoring:** How would you track the no-show prediction model's performance over time? (MLflow metrics, regular re-evaluation).

**Expected Outcome / Deliverables:**
*   A documented high-level solution design for the no-show prediction system using Fabric.
*   Partially implemented Bronze (`bronze_appointments_raw`) and Silver (`silver_appointments`) tables in the Lakehouse.
*   Clear articulation of the features and structure for the Gold ML input table and prediction output table.
*   A conceptual design for the Power BI dashboard, including key visuals.
*   Considerations for governance, collaboration, and operationalizing the solution.

**Questions from Manual & Answers (Adapted for this Lab's Context):  LINK TO HTML?**

*   **Q1: In the context of predicting no-shows, what makes real-world dashboards (like the one you designed) effective for clinic schedulers or managers?**
    *   **A1:** Effective dashboards for clinic schedulers/managers would be:
        *   **Actionable:** Provide clear risk scores or flags for upcoming appointments, enabling staff to take proactive steps (e.g., targeted reminders, overbooking considerations).
        *   **Timely:** Reflect the latest predictions based on up-to-date appointment schedules.
        *   **User-Friendly and Intuitive:** Easy to understand at a glance, with clear KPIs (e.g., overall predicted no-show rate) and visualizations (e.g., lists of high-risk patients).
        *   **Contextual:** Allow filtering by date, provider, clinic, or appointment type to narrow down focus.
        *   **Explainable (if possible):** If the ML model provides reason codes for high risk, displaying these can help staff understand *why* an appointment is flagged.
        *   **Integrated (Ideally):** If possible, insights should be embeddable or accessible within their primary scheduling tools.

*   **Q2: How do OneLake shortcuts and shared workspaces support scaling a solution like no-show prediction if, for example, you needed to incorporate data from a separate hospital department's scheduling system that resides in a different Fabric workspace or even a different ADLS Gen2 account?**
    *   **A2:**
        *   **OneLake Shortcuts:** If another department's scheduling data is in a different ADLS Gen2 or another Fabric Lakehouse, a shortcut can be created in the `ED_Efficiency_Project_Q2` Lakehouse to access that data *in place*. This avoids data duplication and complex ETL to copy it over. The no-show prediction model could then join internal data with this shortcutted external data to build a more comprehensive feature set.
        *   **Shared Workspaces & Cross-Workspace Referencing:** While direct cross-workspace table querying is evolving in Fabric, data sharing can be facilitated. If the other department also uses Fabric, they could share their processed Silver/Gold tables. The no-show project could then potentially create shortcuts to these shared tables within its own Lakehouse. This allows different teams to manage their own data domains but share curated datasets for broader analytics.
        *   **Scalability:** This approach allows the solution to scale by easily incorporating new data sources without fundamentally re-architecting the central project's Lakehouse each time. The core logic for feature engineering and modeling can be adapted to consume data from these new shortcutted sources.

*   **Q3: What ethical risks or biases must be carefully mitigated when developing and deploying an AI model to predict patient appointment no-shows in a healthcare setting?**
    *   **A3:**
        *   **Socioeconomic Bias:** The model might inadvertently learn that patients from certain zip codes, with specific insurance types, or from particular demographic groups (which can be proxies for socioeconomic status or race) have higher no-show rates. If the model then flags these patients more often, it could lead to them receiving excessive (potentially annoying) reminders or even being implicitly deprioritized for appointments, creating inequities in care access.
        *   **Bias Amplification:** If historical data reflects existing biases in how certain patient groups were treated or managed regarding appointments, the model can learn and amplify these biases.
        *   **Lack of Access to Technology:** If reminder systems heavily rely on technology (smartphones, email) that certain patient populations lack access to, a no-show model might unfairly penalize them if "response to digital reminder" becomes a feature.
        *   **Over-reliance and Deskilling:** Staff might become over-reliant on the model's predictions and reduce their own critical thinking or personal outreach efforts that might be more effective for certain patients.
        *   **Transparency and Explainability:** If the model is a "black box," it's hard to identify or address these biases. Patients flagged as high-risk might not understand why, leading to frustration.
        *   **Consequences of Misprediction:**
            *   **False Positives (predicting no-show, but patient attends):** Could lead to unnecessary interventions or overbooking strategies that inconvenience patients.
            *   **False Negatives (predicting attendance, but patient is a no-show):** The original problem of wasted slots persists.
        *   **Mitigation Strategies:** Include fairness assessments during model development (e.g., using tools like Fairlearn), ensuring diverse training data, regularly auditing model predictions for disparate impact across demographic groups, providing model explainability, and implementing policies that ensure equitable treatment regardless of risk score (e.g., using risk scores to offer *more* support rather than to penalize).

This lab is more design-oriented, pushing the learner to think about applying the course's content to solve a new problem end-to-end. It's a good way to synthesize knowledge.

### Lab 12.X: Capstone Challenge - Building a Unified Sepsis Surveillance and Prediction System

**Module Alignment:** Section 12: Labs and Exercises (Capstone Project integrating concepts from Sections 1-11)

**Objective:**
*   Apply knowledge and skills gained throughout the Microsoft Fabric for Healthcare course to design and partially implement a comprehensive solution for a critical healthcare problem: sepsis surveillance and early prediction.
*   Integrate data from multiple simulated sources (vitals, labs, patient demographics, encounter history).
*   Develop a data model (Bronze, Silver, Gold) suitable for both real-time surveillance and predictive analytics.
*   Build a proof-of-concept machine learning model to predict sepsis onset risk.
*   Design an interactive Power BI dashboard for clinicians to monitor at-risk patients and sepsis-related KPIs.
*   Incorporate considerations for data governance, security (including RLS and sensitivity labels), performance, and collaboration.

**Scenario:**
Valley General Hospital is launching an initiative to improve early detection and management of sepsis, a life-threatening condition. They want to leverage Microsoft Fabric to create a unified system that:
1.  Ingests and integrates relevant patient data in near real-time.
2.  Provides clinicians with a dashboard to monitor patients for early warning signs based on established criteria (e.g., SIRS, SOFA scores - simplified for this lab).
3.  Employs a machine learning model to predict the likelihood of sepsis onset for ICU patients.

**Prerequisites:**
*   Successful completion and thorough understanding of all previous labs and course content (Labs 1.5 through 11.8).
*   Proficiency in using Fabric Lakehouse, Notebooks (PySpark/Python), Data Factory (conceptual design), Power BI, and understanding of MLflow and Purview concepts.
*   Ability to work with less prescriptive instructions and make informed design choices.
*   Microsoft Fabric Workspace and necessary permissions.

**Tools to be Used (Design and Partial Implementation):**
*   Microsoft Fabric Lakehouse
*   Microsoft Fabric Notebooks (PySpark, Python, scikit-learn)
*   MLflow
*   Microsoft Fabric Power BI
*   (Conceptual) Data Factory, Real-Time Analytics (KQL for streaming if extending), Microsoft Purview

**Estimated Time:** 3-4 hours (or longer, depending on depth of implementation)

**High-Level Tasks & Design Considerations:**

**Phase 1: Data Source Identification and Ingestion Strategy (Design & Simulate)**

1.  **Identify Key Data Sources:**
    *   **Patient Vitals:** Heart Rate, Respiratory Rate, Temperature, Blood Pressure (simulated as streaming or frequent batch).
    *   **Lab Results:** White Blood Cell count (WBC), Lactate levels, Creatinine (simulated batch).
    *   **Patient Demographics & History:** Age, existing comorbidities, recent surgeries (from `dim_patient` or a new `silver_patient_history` table).
    *   **Encounter Data:** ICU admission/discharge times, current location (from `fact_encounter` or a new `silver_icu_stays` table).
2.  **Design Ingestion Pipelines (Conceptual for Data Factory/Real-Time Analytics):**
    *   How would you ingest streaming vitals? (e.g., Event Hub -> Real-Time Analytics KQL -> Lakehouse Bronze table).
    *   How would you ingest batch lab results? (e.g., Data Factory pipeline from LIS DB/files -> Lakehouse Bronze table).
3.  **Simulate Bronze Layer Data (Notebook):**
    *   Create PySpark DataFrames representing raw data from these sources and save them as Delta tables in a `SepsisBronze` schema/folder in your Lakehouse (e.g., `bronze_patient_vitals_stream`, `bronze_lab_results_batch`). Include timestamps.

**Phase 2: Data Modeling (Silver & Gold Layers - Implement in Notebook)**

1.  **Design and Create Silver Layer Tables:**
    *   `silver_patient_vitals_processed`: Cleaned, validated vitals with patient and encounter context.
    *   `silver_lab_results_processed`: Cleaned, validated labs with patient and encounter context, potentially flagging abnormal results.
    *   `silver_icu_patient_episodes`: Consolidates ICU stay information, linking demographics, vitals, and labs for a given patient during an ICU stay.
2.  **Design and Create Gold Layer Tables:**
    *   `gold_sepsis_surveillance_hourly` (or other appropriate frequency): Aggregated table per patient per hour (or shift) including latest vitals, key lab results, and calculated simplified Sepsis Indicators (e.g., if 2 out of 3 SIRS criteria are met – Temp >38C or <36C, HR >90, RR >20. *This is a simplification; real SOFA/qSOFA is more complex*).
    *   `gold_sepsis_ml_features`: Feature-engineered table specifically for training the sepsis prediction model (may include trends, deltas in vitals/labs over time, historical data).
    *   `gold_sepsis_predictions`: Table to store predictions from the ML model.

**Phase 3: Machine Learning Model Development (Implement in Notebook)**

1.  **Feature Engineering:** From `gold_sepsis_ml_features`, create relevant features.
2.  **Define Target Variable:** `Sepsis_Onset_Next_6_Hours` (boolean – this would require careful labeling of historical data, which you'll need to simulate or define based on criteria).
3.  **Model Selection, Training, and Evaluation:**
    *   Choose a suitable classification model (e.g., Logistic Regression, Random Forest, XGBoost).
    *   Perform train-test split.
    *   Train the model.
    *   Evaluate using appropriate metrics (AUC, Precision, Recall, F1-score, Specificity).
4.  **MLflow Tracking:** Log parameters, metrics, and the model artifact.
5.  **Prediction:** Generate predictions on a test set or new data and store them in `gold_sepsis_predictions`.

**Phase 4: Power BI Dashboard Design and Implementation**

1.  **Create Power BI Dataset (DirectLake):** Connect to your Gold layer tables.
2.  **Design "Sepsis Surveillance Dashboard":**
    *   **Patient List View:** Table showing current ICU patients, their latest vitals, key labs, calculated Sepsis Indicators, and the ML prediction score/risk level. Color-code high-risk patients.
    *   **Individual Patient Drill-Through:** Allow clicking on a patient to see trends of their vitals and labs over the last 24-48 hours.
    *   **KPIs:** Overall number of patients at high risk (by indicator), overall number of patients at high risk (by ML model), average time to intervention (if this data were available).
    *   **Filters:** By ICU unit, shift, risk level.
3.  **Implement RLS:**
    *   If different ICU units should only see their patients, design and implement RLS.
4.  **Apply Sensitivity Labels:** Label the dataset and report appropriately (e.g., "HIPAA-HIGH - Clinical Decision Support").

**Phase 5: Governance, Performance, and Collaboration Considerations (Discussion/Documentation)**

1.  **Data Governance:**
    *   How will Microsoft Purview be used for lineage and classification?
    *   What are the key audit requirements?
2.  **Security:**
    *   Detail RBAC for the workspace and specific sensitive items.
    *   How will data be protected at rest and in transit?
3.  **Performance:**
    *   What are potential performance bottlenecks for the streaming ingestion, ML scoring, and Power BI dashboard?
    *   What optimization techniques would you consider (e.g., table partitioning, `OPTIMIZE`, DAX optimization)?
4.  **Collaboration:**
    *   How will data engineers, data scientists, and clinicians collaborate within this Fabric workspace?
    *   What are the communication and hand-off points?
5.  **Operationalization:**
    *   How would the ML model be retrained and deployed?
    *   How would the "real-time" aspect of the surveillance dashboard be maintained?

**Deliverables for this Capstone Lab:**

1.  **Fabric Notebook(s):**
    *   Code for simulating Bronze data.
    *   Code for transforming data into Silver and Gold layer tables.
    *   Code for ML model training, evaluation, MLflow tracking, and prediction.
2.  **Fabric Lakehouse:**
    *   Bronze, Silver, and Gold layer Delta tables as designed.
3.  **Power BI Report:**
    *   A functional Power BI dashboard connected to the Gold tables, implementing key visuals and RLS (if applicable).
    *   Sensitivity label applied.
4.  **Short Design Document (e.g., Markdown in Notebook or separate document):**
    *   Briefly outlining the architecture, data flow, ML model approach, dashboard design, and considerations for governance, performance, and collaboration.

**Guidance for Learners:**

*   This is a challenge lab. You are expected to draw upon all previous learnings.
*   Focus on a feasible scope for the implementation parts. The design document can cover more aspirational aspects.
*   Make reasonable assumptions where specific data or business rules are not provided (and document them).
*   Simplicity in ML model choice and feature engineering is acceptable; the focus is on the end-to-end Fabric workflow.
*   Prioritize creating a functional, albeit simplified, version of each component.

---

This capstone challenge provides a substantial project for learners to apply their skills. It allows for creativity and problem-solving while reinforcing the core concepts of using Microsoft Fabric in a healthcare context.
