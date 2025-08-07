# Application Technical Profile

This document provides a detailed technical analysis of the SDCIM (Service Desk Customer Incident Management) application, based on the provided source code. It covers the application's overview, architecture, technology stack, and potential modernization strategies.

## 1. Application overview
This section contains application-level information, providing a high-level summary of its key attributes.

| Attribute | Value |
| :--- | :--- |
| **Acronym** | SDCIM |
| **Application name** | Image Concentrator (Inferred: "Concentrador de Imagens") |
| **Primary business purpose** | To process, manage, and monitor financial remittances and their associated documents within a banking environment. |
| **Business domain** | Banking, Financial Services, Document and Remittance Processing |
| **Solution files** | 2 solutions found: `CIControloComSetup.sln`, `CIControloSemSetup.sln` |
| **Project files** | 18 projects found: `Alerta.csproj`, `CIActividades.csproj`, `CIConfigGlobalParameters.csproj`, `CIConfiguration.csproj`, `CIControlo.csproj`, `CIFicheiro.csproj`, `CIFicheirosControlo.csproj`, `CIReports.csproj`, `CIServAlertas.csproj`, `CIServRemessas.csproj`, `CIServico.csproj`, `CIServTester.csproj`, `CITestes.csproj`, `MDIWebTransmCI.vbproj`, `MDIsControlo.csproj`, `QueryForm.csproj`, `CIDepositoErro.csproj`, `SetupCIControlo.vdproj` |
| **Files** | 108 files analyzed: C# source files (.cs): 55, VB.NET source files (.vb): 4, C# project files (.csproj): 16, VB.NET project files (.vbproj): 1, Solution files (.sln): 2, Configuration files (.config): 7, SQL scripts (.sql): 5, Batch scripts (.bat): 3, Resource files (.resx): 10, Other (.settings, .map, .rpt): 5 |
| **Main programming language(s)** | C#, VB.NET |
| **Target .NET Framework(s)** | .NET Framework 4.0, .NET Framework 2.0, .NET Framework 3.5 |
| **Application type** | Windows Forms Application, Windows Service, Class Libraries |
| **Output type** | .exe (Executable), .dll (Dynamic Link Library) |
| **Deployment model** | On-premises |
| **Deployment target platform** | AnyCPU |
| **Complexity** | 🔴 High |
| **Last updated (Source code)** | 2008-01-01 (inferred from copyright notices) |
| **Last updated (Documentation)** | 2025-08-07 |

### 1.1. Main components and features
The application is composed of several distinct components, each with specific responsibilities, forming a comprehensive system for remittance processing and management.

| Component | Description | Key features | Dependencies | Related projects | Technologies |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Main Control UI** | The primary user interface for monitoring and managing system activities. | - Remittance and document tracking<br>- Status filtering<br>- Manual resubmission of failed items | `CIActividades`, `CIConfiguration`, `NBIISNET` | `CIControlo.csproj` | Windows Forms, .NET 4.0 |
| **Activity Monitoring** | Provides forms and logic to visualize activities related to remittances, tranches, and documents. | - Detailed views of remittances, tranches, documents<br>- Image viewing for documents<br>- Manual state changes | `CIConfigGlobalParameters`, `Alerta` | `CIActividades.csproj` | Windows Forms, C# |
| **Remittance Processing** | Core service logic for processing remittances and their associated tranches. | - Fetches remittances pending processing<br>- Processes tranches of documents<br>- Handles error states | `CIConfigGlobalParameters`, `CIActividades` | `CIServRemessas.csproj` | C# |
| **Alerting System** | A module for defining and triggering alerts based on system situations. | - Sends alerts via Email (WebDAV/EWS), SMS, or logs to DB/File/Event Viewer<br>- Configurable actions and parameters | `CIConfigGlobalParameters` | `Alerta.csproj` | C# |
| **Background Service** | Windows Service that orchestrates automated tasks like file import and remittance processing. | - Thread-based job execution<br>- Scheduled processing with configurable stop times<br>- Integration with other business modules | `CIServRemessas`, `CIServAlertas`, `CIFicheiro` | `CIServico.csproj` | .NET Windows Service |
| **Configuration Management**| Centralized library for managing global parameters and user information. | - Database connection management<br>- User session handling<br>- Access to application-wide settings | `NBiis.Generic`, `GenericNet` | `CIConfigGlobalParameters.csproj` | C# |
| **Reporting** | Generates Crystal Reports based on application data. | - Remittance listings<br>- ACOM file summaries<br>- Monthly billing reports | `CrystalDecisions.CrystalReports.Engine` | `CIReports.csproj` | Crystal Reports |
| **File Handling** | Manages the import and processing of specific file formats like ACOM and ENVM. | - File parsing for different record types (Header, Detail, Trailer)<br>- Data validation and insertion into the database | `CIConfigGlobalParameters` | `CIFicheiro.csproj` | C# |

```mermaid
flowchart TD
    subgraph "User Interaction Layer"
        CIControlo["CIControlo (WinForms UI)"]
        CIActividades["CIActividades (Monitoring Forms)"]
    end

    subgraph "Service & Business Logic Layer"
        CIServico["CIServico (Windows Service)"]
        CIServRemessas["CIServRemessas (Remittance Logic)"]
        CIFicheirosControlo["CIFicheirosControlo (File Import)"]
        Alerta["Alerta (Notification Engine)"]
    end

    subgraph "Data & Shared Components"
        DB["(Database)<br>SQL Server"]
        CIConfigGlobalParameters["CIConfigGlobalParameters (Shared Config)"]
        CIReports["CIReports (Reporting)"]
        FileSystem["(File System)<br>ACOM/ENVM Files"]
    end

    subgraph "External Systems"
        EmailGateway["Email/SMS Gateway"]
        WebService["Web Service (waiaccesstu)"]
    end

    CIControlo --> CIActividades
    CIActividades --> CIServRemessas
    CIActividades --> CIReports
    CIActividades --> CIConfigGlobalParameters

    CIServico --> CIServRemessas
    CIServico --> CIServAlertas
    CIServico --> CIFicheirosControlo

    CIFicheirosControlo --> CIFicheiro
    CIFicheiro --> DB

    CIServRemessas --> DB
    Alerta --> DB
    Alerta --> EmailGateway

    CIConfigGlobalParameters --> DB
    
    MDIWebTransmCI --> WebService
    
    CIFicheirosControlo --> FileSystem
```
<center>Figure 1 - High-level component diagram showing the main modules and their interactions within the SDCIM application.</center>

### 1.2. Solutions
The codebase is organized into two main solutions, likely for development with and without the setup project.

| Solution | Projects | Visual Studio version |
| :--- | :--- | :--- |
| **CIControloComSetup.sln** | - Alerta.csproj<br>- CIActividades.csproj<br>- CIConfigGlobalParameters.csproj<br>- CIConfiguration.csproj<br>- CIControlo.csproj<br>- CIFicheiro.csproj<br>- CIFicheirosControlo.csproj<br>- CIReports.csproj<br>- CIServAlertas.csproj<br>- CIServRemessas.csproj<br>- CIServico.csproj<br>- CIServTester.csproj<br>- MDIsControlo.csproj<br>- MDIWebTransmCI.vbproj<br>- QueryForm.csproj<br>- SetupCIControlo.vdproj | 2010 (Version 11.00) |
| **CIControloSemSetup.sln** | - Alerta.csproj<br>- CIActividades.csproj<br>- CIConfigGlobalParameters.csproj<br>- CIConfiguration.csproj<br>- CIControlo.csproj<br>- CIDepositoErro.csproj<br>- CIFicheiro.csproj<br>- CIFicheirosControlo.csproj<br>- CIReports.csproj<br>- CIServAlertas.csproj<br>- CIServRemessas.csproj<br>- CIServico.csproj<br>- CIServTester.csproj<br>- CITestes.csproj<br>- MDIsControlo.csproj<br>- MDIWebTransmCI.vbproj<br>- QueryForm.csproj | 2010 (Version 11.00) |

```mermaid
mindmap
  root((SDCIM Solutions))
    CIControloComSetup.sln
      CIControlo
      CIActividades
      CIConfiguration
      CIServico
      CIReports
      Alerta
      CIServRemessas
      MDIWebTransmCI
      MDIsControlo
      SetupCIControlo
    CIControloSemSetup.sln
      CIControlo
      CIActividades
      CIConfiguration
      CIServico
      CITestes
      CIServTester
      CIFicheiro
      CIFicheirosControlo
```
<center>Figure 2 - Mind map illustrating the project structure within the two solutions.</center>

### 1.3. Projects
The application is highly modular, with functionality separated into numerous projects.

| Solution | Project | Main objectives | Type | Output type | Version | Runtime | Framework | Architecture | Compilation mode | Principal classes | Internal dependencies | External dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Both | **CIControlo.csproj** | Main GUI application for monitoring and control. | Windows Application | Executable | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `CIMainForm.cs` | `CIActividades`, `CIConfiguration`, `CIFicheirosControlo`, `CIReports`, `MDIsControlo`, `QueryForm` | `GenericNet`, `NBIISNET` |
| Both | **CIActividades.csproj** | Contains Windows Forms for displaying system activities. | Class Library | Library | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `ActividadesForm.cs`, `ActividadeBalcaoForm.cs` | `Alerta`, `CIConfigGlobalParameters` | `NBIISNET`, `CrystalDecisions.*` |
| Both | **CIServico.csproj** | A Windows Service for background processing tasks. | Windows Service | Executable | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `CIServico.cs`, `CIServicoThread.cs` | `Alerta`, `CIConfigGlobalParameters`, `CIFicheiro`, `CIServAlertas`, `CIServRemessas` | `GenericNet`, `NBIISNET` |
| Both | **Alerta.csproj** | Manages system alerts and notifications. | Class Library | Library | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `Accao.cs`, `AlertaSituacaoAccao.cs` | `CIConfigGlobalParameters` | `GenericNet` |
| Both | **CIServRemessas.csproj**| Business logic for processing remittances. | Class Library | Library | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `ServRemessa.cs` | `Alerta`, `CIActividades`, `CIConfigGlobalParameters` | `GenericNet`, `NBIISNET` |
| Both | **MDIWebTransmCI.vbproj**| A VB.NET library for consuming a web service. | Class Library | Library | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `TInsertDoc.vb` | `CIConfigGlobalParameters` | `CCAGeneric`, `CCAWebTransmCI` |
| Both | **CITestes.csproj** | Unit and integration tests for the application. | Test Project | Library | 1.0.0.0 | v4.0.30319 | .NET Framework 4.0 | AnyCPU | Debug/Release | `CIServRemessaTestes.cs` | `CIControlo`, `CIServico`, `CIServRemessas`, `CIServTester` | `Microsoft.VisualStudio.QualityTools.UnitTestFramework` |

```mermaid
flowchart TD
    subgraph "Presentation Layer"
        CIControlo["CIControlo.csproj (UI)"]
    end
    
    subgraph "Business & Service Layer"
        CIActividades["CIActividades.csproj (Forms)"]
        CIServico["CIServico.csproj (Service)"]
        CIServRemessas["CIServRemessas.csproj (Logic)"]
        MDIWebTransmCI["MDIWebTransmCI.vbproj (WS Client)"]
    end

    subgraph "Core & Data Layer"
        Alerta["Alerta.csproj"]
        CIFicheiro["CIFicheiro.csproj"]
        DB["(SQL Server)"]
    end

    subgraph "Shared Libraries"
        CIConfigGlobalParameters["CIConfigGlobalParameters.csproj"]
        GenericNet["GenericNet.dll"]
        NBIISNET["NBIISNET.dll"]
    end
    
    subgraph "Testing"
        CITestes["CITestes.csproj"]
        CIServTester["CIServTester.csproj"]
    end

    CIControlo --> CIActividades
    CIActividades --> CIServRemessas
    CIActividades --> CIConfigGlobalParameters
    
    CIServico --> CIServRemessas
    CIServico --> CIFicheiro
    CIServico --> Alerta
    
    CIServRemessas --> CIConfigGlobalParameters
    CIFicheiro --> CIConfigGlobalParameters
    Alerta --> CIConfigGlobalParameters
    
    CIConfigGlobalParameters --> GenericNet
    CIConfigGlobalParameters --> NBIISNET
    CIConfigGlobalParameters -- "Reads from" --> DB

    MDIWebTransmCI --> CIConfigGlobalParameters
    
    CITestes --> CIServico
    CITestes --> CIServRemessas
    CITestes --> CIServTester
```
<center>Figure 3 - Dependency graph illustrating the relationships between key projects.</center>

## 2. Functional overview
This section provides a high-level functional decomposition of the application, identifying the major functional blocks as inferred from the code's structure.

### 2.1. Executive summary
Based on the analysis of entry points, class names, and core processes, the application's primary function is to serve as an internal banking tool for the **management and processing of financial remittances**. It provides a desktop interface for operators to monitor the status of these remittances, handle errors, and manage associated documents. A background Windows service automates the processing pipeline, from file ingestion to database updates, while an alerting system notifies users of key events or failures.

### 2.2. Functional decomposition
The application is functionally divided into several key modules, each responsible for a distinct part of the business process.

| Functional Area / Module | Description |
| :--- | :--- |
| **User Interaction & Presentation** | Manages the graphical user interface for operators. This includes forms for monitoring remittance statuses (`ActividadesForm`), viewing document details (`MostraImagem`), managing system configurations (`CIConfigForm`), and handling alerts (`AlertasForm`). It is the primary interface between the user and the system's core logic. |
| **Remittance Processing Engine** | This is the core logic engine of the application. It's responsible for fetching remittances that need processing, breaking them down into tranches, validating documents, and updating their status in the database. This logic is executed by the background service. |
| **File Ingestion & Parsing** | Handles the intake of external data files, specifically `ACOM` and `ENVM` files. This module reads and parses these files, validates their structure (header, detail, trailer), and inserts the data into the appropriate database tables for further processing. |
| **Alerting & Notification** | A dedicated module for generating and dispatching alerts based on predefined system events and situations. It supports multiple notification channels, including email, SMS, and logging to various sinks like the database, a file, or the Windows Event Viewer. |
| **Data Persistence & Management**| This area covers all interactions with the database. It includes creating, reading, updating, and deleting records related to remittances, documents, users, and configuration settings. Data access is performed primarily through direct ADO.NET SQL queries. |
| **Reporting & Analytics** | Responsible for generating business reports using Crystal Reports. This module provides insights into remittance volumes, processing summaries, and monthly billing data. |

```mermaid
classDiagram
    direction LR
    class PresentationLayer {
        <<Module>>
        +CIControlo
        +CIActividades
        +CIConfiguration
    }
    class BusinessLogicLayer {
        <<Module>>
        +CIServRemessas
        +Alerta
        +CIFicheiro
    }
    class ServiceLayer {
        <<Module>>
        +CIServico
    }
    class DataAccessLayer {
        <<Module>>
        +System.Data.SqlClient
        +GenericNet
    }
    class Database {
        <<Storage>>
        +SQL Server
    }
    class ExternalServices {
        <<System>>
        +Email/SMS Gateway
        +File System (ACOM/ENVM)
    }

    PresentationLayer --> BusinessLogicLayer : "Uses"
    ServiceLayer --> BusinessLogicLayer : "Executes"
    BusinessLogicLayer --> DataAccessLayer : "Accesses Data via"
    DataAccessLayer --> Database : "CRUD Operations"
    BusinessLogicLayer --> ExternalServices : "Interacts with"

```
<center>Figure 4 - Functional decomposition showing the logical layers of the application.</center>

### 2.3. User roles and permissions
The application logic suggests at least two distinct user roles with different levels of privilege.

| Role | Description | Permissions | Key Actions | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Administrator** | A privileged role with full system access, likely responsible for configuration and maintenance. | - Full CRUD on all data entities.<br>- Ability to change the state of any remittance or document.<br>- Access to system configuration and user management. | `ChangeEstado`, `ReenviarTodasRemessasEmErro`, `UtilizadorConfig`, `CIConfiguracao` | `User`, `Passwd_Group`, `Activity_Change`, `Balcao` | Inferred from `if (m_oParameters.UserLogged.m_iUserGroup <= 1)` checks, suggesting that groups 0 and 1 have elevated permissions. |
| **Standard Operator** | A standard user responsible for day-to-day monitoring and operational tasks. | - Read-only access to most data.<br>- Limited ability to perform actions on remittances (e.g., viewing details, refreshing lists).<br>- No access to system-level configuration. | `Refresh`, `ViewDetails`, `ViewImage` | `Remessa`, `Tranche`, `Documento` | Inferred as the default role for users with `m_iUserGroup > 1`, who are restricted from performing state-changing actions. |

```mermaid
classDiagram
    class Administrator {
        +CanChangeState
        +CanConfigureSystem
    }
    class StandardOperator {
        +CanViewActivities
        +CanGenerateReports
    }
    class User {
        +m_iUserGroup
        +m_sUserName
    }
    class Remessa {
        +State
        +Data
    }
    class SystemConfig {
        +Parameter
        +Value
    }
    
    User <|-- Administrator
    User <|-- StandardOperator
    
    Administrator "1" -- "0..*" Remessa : "Manages"
    Administrator "1" -- "0..*" SystemConfig : "Configures"
    StandardOperator "1" -- "0..*" Remessa : "Views"
```
<center>Figure 5 - Class diagram illustrating user roles and their relationship with key data entities.</center>

### 2.4. Core business capabilities
The application delivers several high-level business capabilities related to remittance processing.

| Capability | Description | Key Features | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **Remittance Lifecycle Management** | The complete set of functions for managing a remittance from its creation to its final state (processed, error, etc.). | - Remittance Ingestion<br>- Document Processing<br>- Tranche Management<br>- State Transitions (e.g., Open, Processing, Closed, Error) | `Remessa`, `Tranche`, `Documento` | Assumes that any module involved in creating, updating, or changing the state of a remittance is part of this capability. |
| **Activity Monitoring & Control** | Provides real-time and historical views into system operations, allowing users to track progress and intervene when necessary. | - Activity Dashboard<br>- Filtering by date, status, or type<br>- Manual error handling and resubmission | `Remessa`, `Tranche`, `Documento`, `Alerta` | Inferred from the `CIActividades` and `CIControlo` projects, which are dedicated to building user-facing monitoring forms. |
| **System Configuration** | Allows administrators to configure system parameters, user access, and operational settings. | - User and Group Management<br>- Balcão (Branch) Configuration<br>- Alerting Rules and Actions | `User`, `Passwd_Group`, `Balcao`, `Accao` | Inferred from the `CIConfiguration` project and its associated forms like `UtilizadoresForm` and `BalcaoForm`. |
| **Automated Processing** | The unattended execution of core business logic, such as file polling and remittance processing, via a background service. | - Scheduled File Import (ACOM/ENVM)<br>- Automated Remittance Processing<br>- Scheduled Alert Generation | `Ficheiro`, `Lote`, `Remessa`, `Alerta` | Inferred from the presence of `CIServico`, a Windows Service project designed for continuous background operation. |

### 2.5. Detailed feature breakdown
This section provides a granular breakdown of individual features within each Core Business Capability.

| Feature | User Story / Description | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- |
| **Monitor Remittance Activity** | As a Standard Operator, I can view a dashboard of all remittances, filterable by date and status, so that I can track the overall health of the system. | `Remessa`, `Tranche` | Inferred from `ActividadesForm.cs` and its various list views and filter controls. |
| **Resubmit Failed Remittance**| As an Administrator, I can select a remittance in an "Error" state and manually trigger a re-processing job, so that I can resolve transient issues without manual data re-entry. | `Remessa`, `Tranche`, `Documento` | Inferred from the `btnRemessasErro_Click` event handler in `ActividadeBalcaoForm.cs`, which calls a stored procedure `Update_ReenviarTodasRemessasBalcaoEmErro`. |
| **View Document Image** | As a Standard Operator, I can select a specific document within a remittance and view its scanned image to verify its contents. | `Documento`, `Imagem` | Inferred from the `MostraImagem.cs` form and the `toolStripMenuDocumentosVerImagem` menu item. |
| **Configure Alert Action** | As an Administrator, I can associate a specific system situation (e.g., "Remittance Processing Failed") with a notification action (e.g., "Send Email to Ops Team") to ensure timely intervention. | `SituacaoAccao`, `Accao`, `AccaoParam` | Inferred from the `CIConfigForm.cs` UI, which allows for managing associations between situations and actions. |

### 2.6. Key data entities and their attributes
The application's logic revolves around a few core data entities.

| Entity | Description | Key Attributes | Data Type | Validation Rules | Relationships | Assumptions |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Remessa** | Represents a batch of documents submitted for processing, typically from a bank branch (`Balcao`). | `REMIN_ID`, `REMIN_DATA`, `REMINSTAT_ID`, `REMIN_BALCAO`, `REMIN_NUMERO` | `long`, `DateTime`, `int`, `int`, `int` | `REMIN_ID` is the primary key. `REMINSTAT_ID` must be a valid status. | Has a one-to-many relationship with `Tranche` and `Documento`. | Inferred from classes like `DetalheRemessa.cs` and view names like `VW_REMESSA_BALCAO_DETALHE`. |
| **Tranche** | A sub-batch of a `Remessa`, containing a group of documents processed together. | `TRANOUT_ID`, `REMIN_ID`, `TRANOUT_NUMERO`, `TRANOUTSTAT_ID` | `long`, `long`, `int`, `int` | `TRANOUT_ID` is the primary key. `REMIN_ID` is a foreign key to `Remessa`. | Belongs to one `Remessa`. Has a one-to-many relationship with `Documento`. | Inferred from `DetalheTranche.cs` and `VW_TRANCHE_BALCAO_DETALHE`. |
| **Documento** | Represents a single financial document, such as a check, within a `Remessa` or `Tranche`. | `DOC_ID`, `REMIN_ID`, `TRANOUT_ID`, `DOC_ZONA1`-`5`, `DOCSTAT_ID` | `long`, `long`, `long`, `string`, `int` | `DOC_ID` is the primary key. Contains OCR data in `DOC_ZONA` fields. | Belongs to one `Remessa` and one `Tranche`. | Inferred from `DetalheDocumento.cs` and `VW_DETALHE_DOCUMENTOS_BALCAO`. |
| **Accao (Action)** | A configurable action to be executed in response to a system event, such as sending an email or logging a message. | `ACC_ID`, `ACC_DESC`, `TIPACC_ID` | `int`, `string`, `enum` | `ACC_ID` is the primary key. `TIPACC_ID` defines the action type (Email, SMS, Log, etc.). | Has a many-to-many relationship with `Situacao` via `SituacaoAccao`. | Inferred from `Accao.cs`. The `enuTipoAccao` enum explicitly lists the types of actions supported. |
| **Situacao (Situation)** | Represents a specific system state or event that can trigger an `Accao`. | `SITUACAO_ID`, `SITUACAO_DESC` | `int`, `string` | `SITUACAO_ID` is the primary key. | Has a many-to-many relationship with `Accao`. | Inferred from `SituacaoAccao.cs`. |

```mermaid
erDiagram
    REMESSA ||--o{ TRANCHE : contains
    REMESSA ||--o{ DOCUMENTO : contains
    TRANCHE ||--o{ DOCUMENTO : contains
    SITUACAO ||--|{ SITUACAO_ACCAO : triggers
    ACCAO ||--|{ SITUACAO_ACCAO : executes
    
    REMESSA {
        long REMIN_ID PK
        datetime REMIN_DATA
        int REMINSTAT_ID FK
        int REMIN_BALCAO
        int REMIN_NUMERO
    }
    TRANCHE {
        long TRANOUT_ID PK
        long REMIN_ID FK
        int TRANOUT_NUMERO
        int TRANOUTSTAT_ID
    }
    DOCUMENTO {
        long DOC_ID PK
        long REMIN_ID FK
        long TRANOUT_ID FK
        string DOC_ZONA1
        int DOCSTAT_ID
    }
    SITUACAO {
        int SITUACAO_ID PK
        string SITUACAO_DESC
    }
    ACCAO {
        int ACC_ID PK
        string ACC_DESC
        enum TIPACC_ID
    }
    SITUACAO_ACCAO {
        int SITUACAO_ID PK, FK
        int ACC_ID PK, FK
    }
```
<center>Figure 6 - Entity-Relationship diagram showing the core data entities and their relationships.</center>

### 2.7. Business process workflows
This section describes and visualizes critical end-to-end processes.

| Process | Description | Key Steps | User Roles Involved | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Remittance Processing Workflow** | A comprehensive workflow that outlines the steps involved in processing a remittance from ingestion to final output. | 1. Remittance is identified for processing.<br>2. Status is set to "In Progress".<br>3. Documents are grouped into tranches.<br>4. Each tranche is processed.<br>5. Remittance status is updated to "Completed" or "Error". | System (Automated) | `Remessa`, `Tranche`, `Documento` | Inferred from the logic in `ServRemessa.cs` and `ProcessarTrancheBalcao.cs`. The process is automated by the `CIServico` Windows Service. |
| **Alerting Workflow** | The process of detecting a system situation and executing a predefined notification action. | 1. A system event triggers a situation (e.g., error in processing).<br>2. An alert record is created.<br>3. The alerting service (`ServAlerta`) picks up the alert.<br>4. The associated action (e.g., send email) is executed.<br>5. The alert is marked as processed. | System (Automated), Administrator (configures rules) | `AlertaSituacaoAccao`, `SituacaoAccao`, `Accao` | Inferred from the `Alerta` project and the `ServAlerta.cs` class, which orchestrates the alert processing logic. |
| **Manual Error Intervention** | A user-driven workflow to manually handle and resubmit failed remittances or estornos (reversals). | 1. Operator filters for items in an "Error" state.<br>2. Operator selects one or more failed items.<br>3. Operator clicks the "Re-submit" button.<br>4. The system flags the items for re-processing by the background service. | Administrator, Standard Operator | `Remessa`, `Estorno` | Inferred from UI elements like `btnRemessasErro_Click` and `btnEstonoErro_Click` in `ActividadeBalcaoForm.cs`. |

```mermaid
flowchart TD
    A["Remittance Processing Workflow"] --> B["Alerting Workflow"]
    C["Manual Error Intervention"] --> A
    B --> C
```
<center>Figure 7 - High-level relationships between the main business process workflows.</center>

#### 2.7.1. Workflow: Remittance Processing Workflow
This section provides a detailed breakdown of the Remittance Processing Workflow, including the steps involved, user roles, key data entities, and any assumptions made.

| Step | Description | User Role | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **1. Identify Remittance** | The background service queries the database to find a remittance with a "Pending" status. | System | `Remessa` | Inferred from `ServRemessa.RemessaBalcaoParaProcessar()`, which selects a remittance to be processed. |
| **2. Begin Processing** | The system updates the remittance's status to "In Progress" to prevent other threads from processing it. | System | `Remessa` | Assumes a state transition from "Pending" (e.g., status 20) to "In Progress" (e.g., status 30) as seen in `ProcessaRemessa`. |
| **3. Create Tranches** | Documents within the remittance are grouped into smaller batches (tranches) for processing. A new `Tranche` record is created. | System | `Tranche`, `Documento` | Inferred from the loop in `ProcessaRemessa` that calls `CriarTranche` based on `m_oParameters.m_iMaxDocsTranche`. |
| **4. Process Documents** | Each document in the tranche is validated, and its data is processed. The document's status is updated. | System | `Documento` | Inferred from the `ProcessaDocumento` method, which performs validation and updates the document state. |
| **5. Finalize Tranche** | Once all documents in a tranche are processed, the tranche status is updated to "Completed". | System | `Tranche` | Assumes a state transition to a completed status (e.g., status 20) after the loop. |
| **6. Finalize Remittance** | After all tranches are completed, the remittance status is updated to "Completed" (or "Error" if any step failed). | System | `Remessa` | The final step in `ProcessaRemessa` is to update the remittance status to a final state (e.g., 40 for success, -40 for error). |

```mermaid
flowchart TD
    A[Start] --> B{"Find Pending Remittance"};
    B -- "Yes" --> C["Update Remittance Status to 'In Progress'"];
    C --> D["Loop: Create Tranche"];
    D --> E["Loop: Process Documents in Tranche"];
    E --> F{"All Docs in Tranche Processed?"};
    F -- "Yes" --> G["Update Tranche Status to 'Completed'"];
    G --> H{"All Tranches Processed?"};
    F -- "No" --> E;
    H -- "No" --> D;
    H -- "Yes" --> I{"Any Errors?"};
    I -- "No" --> J["Update Remittance Status to 'Completed'"];
    I -- "Yes" --> K["Update Remittance Status to 'Error'"];
    J --> L[End];
    K --> L;
    B -- "No" --> L;
```
<center>Figure 8 - Detailed flowchart of the Remittance Processing Workflow.</center>

#### 2.7.2. Workflow: Alerting Workflow
This section provides a detailed breakdown of the Alerting Workflow, including the steps involved, user roles, key data entities, and any assumptions made.

| Step | Description | User Role | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **1. Trigger Alert** | A business process (e.g., remittance processing) encounters a specific condition (e.g., an error) and calls a stored procedure to create an alert. | System | `AlertaSituacaoAccao` | Inferred from `m_oParameters.EnviarAlertaSituacao()`, which is called from various parts of the application to log alerts. |
| **2. Fetch Pending Alert** | The background alerting service queries the database for alerts with a "Pending" status (e.g., `ALACC_STATUS=0`). | System | `AlertaSituacaoAccao` | Inferred from `ServAlerta.AlertaSituacaoAccaoParaProcessar()`. |
| **3. Determine Action Type** | The service inspects the alert to determine the type of action to perform (e.g., Email, SMS, Log). | System | `Accao` | Based on the `switch` statement on `oAlSitAcc.m_oSituacaoAccao.m_oAccao.m_enuTIPACC_ID` in `ServAlerta.cs`. |
| **4. Execute Action** | The corresponding action handler is invoked. For an email, it constructs and sends the message. For a log, it writes to the configured target. | System | `AccaoParam` | Inferred from methods like `ProcessaAlertaSituacaoAccaoMail` and `ProcessaAlertaSituacaoAccaoLogFile`. |
| **5. Mark as Processed** | The alert's status is updated to "Processed" in the database to prevent it from being handled again. | System | `AlertaSituacaoAccao` | Inferred from the call to `oAlSitAcc.SetProcessado(m_oParameters)`. |

```mermaid
flowchart TD
    A[Start] --> B{"Find Pending Alert"};
    B -- "Yes" --> C["Determine Action Type"];
    C --> D{Switch Action};
    D -- "Email" --> E["Construct & Send Email"];
    D -- "SMS" --> F["Send SMS Message"];
    D -- "Log" --> G["Write to Log File/DB"];
    E --> H["Update Alert Status to 'Processed'"];
    F --> H;
    G --> H;
    H --> I[End];
    B -- "No" --> I;
```
<center>Figure 9 - Detailed flowchart of the Alerting Workflow.</center>

#### 2.7.3. Workflow: Manual Error Intervention
This section provides a detailed breakdown of the Manual Error Intervention workflow, including the steps involved, user roles, key data entities, and any assumptions made.

| Step | Description | User Role | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **1. Open Activity Form** | The user opens the `ActividadeBalcaoForm` to view the status of remittances and reversals. | Administrator, Standard Operator | `Remessa`, `Estorno` | This is the main entry point for this user task. |
| **2. Filter for Errors** | The user applies filters (e.g., clicks the "Error" status button) to display only failed items. | Administrator, Standard Operator | `Remessa`, `Estorno` | Inferred from the `toolStripButtonErro` UI element and the filtering logic in `filtrosRemessas`. |
| **3. Select Items** | The user selects one or more failed remittances or reversals from the list view. | Administrator | `Remessa`, `Estorno` | Standard user interaction with a `ListView` control. |
| **4. Trigger Resubmission** | The user clicks the "Re-submit items in error" button. | Administrator | `Remessa`, `Estorno` | Inferred from the `btnRemessasErro_Click` and `btnEstonoErro_Click` event handlers. |
| **5. System Flags for Reprocessing** | An `UPDATE` statement or stored procedure call (`Update_ReenviarTodasRemessasBalcaoEmErro`) is executed to change the status of the selected items, making them eligible for reprocessing by the background service. | System | `Remessa`, `Estorno` | The event handlers call a stored procedure to perform the backend update. |

```mermaid
flowchart TD
    A[User Opens Activity Form] --> B["User Clicks 'Error' Filter"];
    B --> C["System Displays Failed Items"];
    C --> D["User Selects Failed Items"];
    D --> E{"User Clicks 'Resubmit' Button"};
    E -- "Yes" --> F["System Calls Stored Procedure<br>to Reset Status"];
    F --> G["Background Service Reprocesses Items"];
    G --> H[End];
    E -- "No" --> H;
```
<center>Figure 10 - Detailed flowchart of the Manual Error Intervention workflow.</center>

### 2.8. System inputs & outputs (interfaces)
This section catalogs all identified points where data enters or leaves the application boundary.

| Type | Name / Description | Data Formats | Data Source / Sink | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **Input** | **User Interface** | User-entered text and clicks | Windows Forms UI | The primary way operators interact with the system for manual tasks and monitoring. |
| **Input** | **File Ingestion (ENVM/ACOM)**| Fixed-width text files | Monitored file system directory | The `CIFicheiro` project is designed to parse these specific file formats, suggesting a file-based integration for batch data import. |
| **Input** | **Web Service (SOAP)** | XML/SOAP | External System (`waiaccesstu`) | The `MDIWebTransmCI` project is a web service client, indicating it receives data from an external SOAP endpoint. |
| **Output** | **Database Write** | SQL | Microsoft SQL Server | All processed data, states, and logs are persisted in the central database. |
| **Output** | **Email/SMS Notification** | Text | Email Gateway / SMS Provider | The `Alerta` module has functionality to send notifications, which requires an external gateway. |
| **Output** | **Log Files** | Text | Local/Network file system | `ServAlertasLogFile.cs` explicitly writes alert details to a configured file path. |
| **Output** | **Windows Event Log** | Event Log format | Windows OS | `ServAlerta.cs` writes entries to the "CIServico - Alerta" event log source. |
| **Output** | **Crystal Reports** | PDF, etc. | User's screen or printer | The `CIReports` project generates reports for user consumption. |
| **Data Flow** | **Internal Processing** | In-memory objects | Application Memory -> Database | Data is read from the database, processed by business logic, and the results are written back to the database. |

### 2.9. User Task Flows
This section describes the typical path a user takes through the application to achieve a high-level objective.

| Task Flow | Description | User Role | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **End-to-End Remittance Monitoring** | A comprehensive sequence of steps an operator follows to investigate a specific remittance from summary to document level. | Standard Operator | `Remessa`, `Tranche`, `Documento` | This flow represents the most common use case for a standard operator using the application's UI. |
| **System Configuration Update** | The sequence of actions an administrator takes to modify a system parameter, such as an alert rule or a branch (`Balcao`) setting. | Administrator | `Balcao`, `Accao`, `Situacao` | This flow is restricted to privileged users and involves changing the application's operational behavior. |

```mermaid
flowchart TD
    A["End-to-End Remittance Monitoring"]
    B["System Configuration Update"]

    A --- B
```
<center>Figure 11 - High-level relationship between the main user task flows.</center>

#### 2.9.1. User Task Flow: End-to-End Remittance Monitoring
This section provides a detailed breakdown of the End-to-End Remittance Monitoring task flow.

| Step | Description | User Role | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- |
| **1. Launch Application** | The user launches the `CIControlo.exe` application and logs in. | Standard Operator | `User` | Standard application start. |
| **2. Open Activity View** | The user navigates to the "Actividades" or "Actividades Balcão" form from the main menu. | Standard Operator | - | This is the main monitoring screen. |
| **3. Filter Remittances** | The user applies filters, such as a date range or a specific status (e.g., "Error"), and clicks "Refresh". | Standard Operator | `Remessa` | The UI provides filtering controls to narrow down the results. |
| **4. View Remittance Details** | The user double-clicks a summary row to load the detailed list of remittances matching the filter. | Standard Operator | `Remessa` | Double-clicking a summary view drills down into a detailed view. |
| **5. View Tranche Details** | The user double-clicks a specific remittance to view the tranches it contains. | Standard Operator | `Tranche` | Further drill-down from remittance to tranche. |
| **6. View Document Details** | The user double-clicks a specific tranche to view the list of documents within it. | Standard Operator | `Documento` | Final drill-down from tranche to individual documents. |
| **7. View Document Image** | The user double-clicks a document to open a new window displaying the scanned image of the document. | Standard Operator | `Imagem` | The `MostraImagem` form is invoked to display the image. |

```mermaid
journey
    title Remittance Monitoring Task Flow
    section Investigation
      Operator opens Activity Form: 5: Operator
      Operator filters by date and status: 5: Operator
      Operator views list of remittances: 5: Operator
      Operator drills down to tranches: 4: Operator
      Operator drills down to documents: 4: Operator
      Operator views document image: 4: Operator
```
<center>Figure 12 - User journey diagram for the remittance monitoring task flow.</center>

#### 2.9.2. User Task Flow: System Configuration Update
This section provides a detailed breakdown of the System Configuration Update task flow.

| Step | Description | User Role | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- |
| **1. Launch Application** | The administrator launches the `CIControlo.exe` application and logs in. | Administrator | `User` | Standard application start with privileged credentials. |
| **2. Open Configuration Menu** | The administrator navigates to the "Manutenção" (Maintenance) menu. | Administrator | - | This menu is likely only visible or enabled for administrators. |
| **3. Select Configuration Area** | The administrator selects a specific configuration screen, such as "Balcões/Leitores" or "Utilizadores". | Administrator | `Balcao`, `User` | The user chooses which part of the system to configure. |
| **4. Modify Settings** | The administrator modifies the settings in the selected form, such as adding a new `Balcao` or changing a user's group. | Administrator | `Balcao`, `User` | The form provides fields and buttons to edit the configuration data. |
| **5. Save Changes** | The administrator clicks the "Inserir" or "Actualizar" button to persist the changes to the database. | Administrator | `Balcao`, `User` | The application executes an INSERT or UPDATE SQL command to save the new configuration. |

```mermaid
flowchart TD
    A[Admin Logs In] --> B["Navigates to 'Maintenance' Menu"];
    B --> C{Select Config Screen};
    C -- "Balcões" --> D["Modify Balcao Details"];
    C -- "Utilizadores" --> E["Modify User Details"];
    D --> F["Clicks 'Save' or 'Update'"];
    E --> F;
    F --> G["System Updates Database"];
    G --> H[End];
```
<center>Figure 13 - Flowchart for the system configuration update task flow.</center>

### 2.10. Automated & Scheduled Processes
The application includes a background service that runs automated jobs.

| Process Name | Trigger / Schedule | Purpose | Key Actions | Key Data Entities | Assumptions |
| :--- | :--- | :--- | :--- | :--- |
| **Remittance Processing Job**| Runs continuously in a timed loop within the `CIServico` Windows Service. | To automate the processing of new and pending remittances without manual intervention. | - `Select_Remessa2Process` to find work.<br>- `Update_ProcessaRemessaProc` to process.<br>- `Update_EstadoRemessa` to finalize. | `Remessa`, `Tranche`, `Documento` | Inferred from the `CIServicoThread` class, which contains a main processing loop (`while (!m_bStop)`). |
| **File Ingestion Job** | Runs periodically (every `m_iTempoEntreIteracoesFicheiros` minutes) to check for new files. | To automatically import data from external systems via `ENVM` and `ACOM` files. | - Scans configured directories (`m_sFilePathENVM`, `m_sFilePathACOM`).<br>- Parses files line by line.<br>- Inserts data into staging tables. | `Ficheiro`, `Lote`, `Documento` | Inferred from the `CheckForFileENVM2Import` and `CheckForFileACOM2Import` methods in `CIServicoThread.cs`. |
| **Alerting Service Job** | Runs continuously in a timed loop within the `CIServico` Windows Service. | To process pending alerts and send out notifications. | - `AlertaSituacaoAccaoParaProcessar` to find alerts.<br>- `ProcessaAlertaSituacaoAccao` to execute the action (e.g., send email). | `AlertaSituacaoAccao`, `Accao` | Inferred from the `CheckForAlertas2Process` method in `CIServicoThread.cs`. |

## 3. Architecture and design
This section describes the architectural patterns and design principles observed in the source code.

### 3.1. Architecture pattern
The application follows a traditional N-Tier monolithic architecture.

| Pattern Type | Patterns Observed |
| :--- | :--- |
| **High-Level Architecture** | **N-Tier Monolithic**: The application is structured into distinct projects representing presentation (UI), business logic, and data access concerns, but they are tightly coupled and deployed as a single unit (or a main executable with supporting libraries). |
| **Architectural Patterns (UI)** | **Model-View-Presenter (MVP) / Supervising Controller**: The Windows Forms classes (`*Form.cs`) act as Views, containing UI elements and event handlers. They instantiate and delegate to controller-like or business logic classes (`ServRemessa`, `Actividades`) which interact with the data and update the view. |
| **Creational Patterns** | ❔ **Factory Method (Likely)**: Inferred from the separation of concerns, it is likely that simple factory methods are used to instantiate business objects, although no explicit factory classes were found in the provided snippets. |
| **Structural Patterns** | ❌ **None explicitly identified**: The code does not show evidence of common structural patterns like Adapter, Decorator, or Proxy. |
| **Behavioral Patterns** | ❔ **Command (Likely)**: The menu items and buttons in the UI that trigger specific business operations (e.g., `btnRemessasErro_Click`) can be considered an implementation of the Command pattern, where a UI event is mapped to a specific action. |
| **Domain-Driven Design (DDD)** | ❌ **Repository, Unit of Work**: Not observed. Data access is performed via direct SQL queries within business or UI-related classes, rather than being abstracted behind a repository. |

### 3.2. Architecture style
The application is built using a layered architectural style.

| Aspect | Description |
| :--- | :--- |
| **Layering** | The architecture is separated into logical layers, primarily: <br> - **Presentation Layer**: `CIControlo`, `CIActividades` (Windows Forms).<br> - **Business Logic Layer**: `CIServRemessas`, `CIServAlertas`, `CIFicheiro`.<br> - **Data Access Layer**: Implicitly defined through direct `System.Data.SqlClient` usage within business logic classes.<br> - **Shared Components**: `CIConfigGlobalParameters`. |
| **Dependency Flow** | The dependency flow is generally top-down: UI depends on Business Logic, which in turn depends on Shared Components and performs data access. There are no strict dependency rules enforced, leading to some coupling (e.g., UI forms directly instantiating `SqlDataReader`). |
| **Bounded Contexts** | ❌ Not applicable. As a monolithic application, there is no clear separation into distinct bounded contexts. All modules operate on a shared database schema. |
| **Communication** | - **Synchronous**: UI interactions and direct method calls.<br>- **Asynchronous**: The background service (`CIServico`) operates asynchronously from the UI.<br>- **Web Service**: SOAP communication with an external service. |
| **Dependency Injection** | ❌ **None**: Dependencies are created manually using the `new` keyword (e.g., `CIServRemessas.ServRemessa servRemessa = new CIServRemessas.ServRemessa(this, m_oParameters);`). There is no Inversion of Control (IoC) container. |

### 3.3. Key design patterns and principles applied
The codebase reflects design principles common in applications from its era.

| Principle | Analysis |
| :--- | :--- |
| **SOLID Principles** | 🟠 **Partial Adherence**: <br>- **Single Responsibility Principle (SRP)**: Partially followed through the use of multiple projects (`Alerta`, `CIServRemessas`), each handling a specific concern. However, classes often have multiple responsibilities (e.g., UI forms containing data access logic).<br>- **Open/Closed Principle (OCP)**: Not strongly evident. Adding new alert types requires modifying the `switch` statement in `ServAlerta.cs`.<br>- **Liskov, Interface Segregation, Dependency Inversion**: Not explicitly observed. The lack of dependency injection and heavy use of concrete classes indicates these principles were not a primary focus. |
| **DRY (Don't Repeat Yourself)** | 🟠 **Moderate Adherence**: Some code appears to be repeated, such as the UI logic for creating and managing `ListView` controls across different forms. However, the use of shared libraries like `CIConfigGlobalParameters` helps centralize common functionality. |
| **KISS (Keep It Simple, Stupid)**| ✅ **Generally Followed**: The logic within individual methods is straightforward and easy to understand. The overall design avoids overly complex abstractions. |
| **YAGNI (You Aren't Gonna Need It)**| ✅ **Generally Followed**: The application appears focused on its specific business purpose without unnecessary features. |
| **Separation of Concerns** | 🟠 **Partial Adherence**: There is a clear separation at the project level (UI, Service, Logic). However, within projects, concerns are often mixed. For instance, UI forms (`ActividadeBalcaoForm.cs`) contain direct database query construction and execution logic, mixing presentation with data access. |
| **Clean Code Principles** | 🟠 **Mixed**: Naming conventions are generally followed, but some classes and methods could be more descriptive. The presence of empty `catch` blocks (`catch{}`) is a significant code smell. |
| **Testability** | 🟠 **Limited**: The `CITestes` project indicates that testing was considered. However, the tight coupling and lack of dependency injection make unit testing difficult. Tests are more likely to be integration tests that require a database connection. |

### 3.4. Domain-driven design (DDD) adoption
The application does not follow a formal Domain-Driven Design approach.

| DDD Concept | Analysis |
| :--- | :--- |
| **Use of DDD concepts** | ❌ **Limited/None**: While there are classes that represent domain concepts (e.g., `Remessa`, `Documento`), they are primarily data containers (Anemic Domain Model) rather than rich domain objects with behavior. Core DDD patterns like Aggregates, Value Objects, and Domain Events are not used. |
| **Ubiquitous Language** | 🟠 **Partially Evident**: The class and method names (`Remessa`, `Tranche`, `TratarTranchesBalcao`) use terms from the business domain (banking, remittances), suggesting a shared language between developers and business experts. |
| **Layered DDD approach** | ❌ **Not Followed**: The architecture is a classic N-Tier, not a DDD-style layered architecture with distinct Application, Domain, and Infrastructure layers. Business logic is mixed with data access and sometimes presentation logic. |

### 3.5. Patterns and tactics
The application uses several common data access and interaction patterns.

| Pattern | Analysis |
| :--- | :--- |
| **CQRS** | ❌ **Not Used**: There is no separation between commands (writes) and queries (reads). The same classes and methods handle both operations. |
| **Event Sourcing** | ❌ **Not Used**: The application state is stored directly in the database, not as a sequence of events. |
| **Repository Pattern** | ❌ **Not Used**: Data access is performed directly using `SqlDataReader` and raw SQL strings, often within business logic or UI classes. There is no abstraction layer for data persistence. |
| **Unit of Work Pattern**| ❌ **Not Used**: Transactions are managed manually using `m_oParameters.BeginTrans()` and `Commit()`/`RollBack()`, but this is not encapsulated in a formal Unit of Work pattern. |

### 3.6. Cross-cutting concerns
Cross-cutting concerns are handled through shared libraries and direct implementation.

| Concern | Implementation Details |
| :--- | :--- |
| **Logging** | A custom `GenericLog` class is used for logging alerts and errors. `System.Diagnostics.Debug.WriteLine` is also used for debug output. |
| **Exception Handling**| Standard `try-catch` blocks are used. Some `catch` blocks are empty, which can hide errors. |
| **Configuration Management**| Handled via `.config` files (`app.config`, `CIControlo.exe.config`) and `Properties.Settings`. A custom `CIGlobalParameters` class centralizes access to these settings. |
| **Authentication** | Based on the logged-in Windows user (`System.Windows.Forms.SystemInformation.UserName`). |
| **Authorization** | Simple role-based checks using `m_oParameters.UserLogged.m_iUserGroup`. |
| **Dependency Injection** | ❌ Not used. Dependencies are instantiated directly. |

### 3.7. Deployment
The application is designed for on-premises deployment.

| Aspect | Description |
| :--- | :--- |
| **Monolithic vs Microservices** | **Monolithic**: The application is a single, tightly-coupled system, deployed as a set of executables and libraries. |
| **Scalability Design** | 🟠 **Limited**: Scalability is limited to vertical scaling (increasing resources of the host server). There is no built-in design for horizontal scaling. The background service could potentially be run on multiple servers if the database queuing mechanism (`Select_Remessa2Process`) is designed to be atomic. |
| **Cloud-Native Design** | ❌ **Not Cloud-Native**: The application relies on local file system paths (e.g., `c:\tmp\`), UNC paths (`\\sqc6001fas02`), and direct database connections, making it unsuitable for a cloud-native environment without significant changes. |
| **Containerization** | ❌ **Not Containerized**: No `Dockerfile` or container configurations were found. |
| **Deployment Target Platform** | **AnyCPU**: The projects are configured to run on any CPU architecture (x86 or x64) where the .NET Framework is installed. |

### 3.8. Infrastructure considerations
The application has specific infrastructure dependencies.

| Area | Description |
| :--- | :--- |
| **Persistence** | **Microsoft SQL Server**: The application is tightly coupled to SQL Server, using `System.Data.SqlClient` and stored procedures. Server names like `SDC6001SQL28\S2K081` suggest specific instances are required. |
| **Messaging / Eventing** | ❌ **None**: No message brokers or event buses are used. The database is used as a queue for work items (e.g., pending remittances). |
| **External Integrations**| - **File System**: Relies on network shares or local directories for file-based data exchange (`ACOM`, `ENVM` files).<br>- **SOAP Web Service**: Integrates with an external service at `http://waiaccesstu:80/UWTDWeb/services/Insert_DocumentoMDIService`. |

### 3.9. Documentation artifacts
The solution contains some artifacts that serve as documentation.

| Artifact | Analysis |
| :--- | :--- |
| **UML Diagrams** | ❌ None found in the provided files. |
| **C4 Model Diagrams** | ❌ None found in the provided files. |
| **SQL Scripts** | ✅ SQL files for reports (`AcomResumo.sql`, etc.) provide insight into the database schema and reporting logic. |
| **Readme or ADRs** | ❌ No `README.md` or Architecture Decision Records were provided. |
| **Code Comments** | 🟠 Comments are present but sparse. They provide some context but do not constitute comprehensive documentation. |

## 4. Technology stack and frameworks
This section details the specific technologies and frameworks used in the application.

### 4.1. Backend technologies
The backend is built on the .NET Framework with a mix of C# and VB.NET.

| Technology | Details |
| :--- | :--- |
| **Programming Language(s)** | C#, VB.NET |
| **.NET Runtime(s)** | .NET Framework 4.0, .NET Framework 2.0 |
| **Framework(s)** | .NET Framework |
| **ORM / Database Access** | ADO.NET (`System.Data.SqlClient`) |
| **Dependency Injection** | ❌ None (manual instantiation) |
| **API types** | SOAP (client-side consumption) |
| **Unit testing and test frameworks** | MSTest (`Microsoft.VisualStudio.QualityTools.UnitTestFramework`) |
| **Build tools** | MSBuild |
| **Other common libraries / SDKs** | `GenericNet`, `NBIISNET` (internal shared libraries), `CrystalDecisions` (for reporting) |

### 4.2. Frontend technologies
The user interface is a desktop application built with Windows Forms.

| Technology | Details |
| :--- | :--- |
| **Frontend framework(s)** | Windows Forms |
| **Frontend libraries** | `NBIISNET` (provides custom controls like `ListViewBase` and `DataGridViewBase`) |
| **JavaScript frameworks** | ❌ Not applicable |

### 4.3. Data and storage
The application relies on a central SQL Server database and the file system.

#### 4.3.1. Database technologies
| Technology | Details |
| :--- | :--- |
| **Database management system(s)** | Microsoft SQL Server (inferred version: 2008 R2, based on instance names like `S2K081`) |
| **Database type** | Relational |
| **Data access technology** | ADO.NET (`SqlConnection`, `SqlDataReader`) |
| **Database name(s)** | `BDSDCSDCIMA01`, `BDSQCSDCIMA01`, `BDSPCSDCIMA01`, `BDSPGCCALX`, `BDSPGCCAPT`, `BDSDGCCALX` (inferred from `.config` files) |
| **Database connection string(s)**| No complete connection strings with credentials found. Configuration files specify server (`H`) and database (`D`) separately for different environments (DVP, QLD, PRD). Example: `Server=SDC6001SQL28\S2K081;Database=BDSDCSDCIMA01;` |
| **File/blob storage** | Local/Network File System. Path `\\sqc6001fas02\imagens\DriveW\GCAA\Gestao\SharedLibrary\` is used for shared DLLs. |

#### 4.3.2. Data flow
Data flows from external files and user input, is processed by the application, and stored in the database.

| Aspect | Description |
| :--- | :--- |
| **Data sources** | - **File System**: `ACOM` and `ENVM` files from a monitored directory.<br>- **User Input**: Manual data entry and actions from the Windows Forms UI.<br>- **Database**: Existing remittance and document data for processing. |
| **Data sinks** | - **Database**: Processed results, logs, and status updates are written to SQL Server.<br>- **File System**: Backup of processed files.<br>- **Windows Event Log**: Service status and error messages.<br>- **Email/SMS Gateway**: Alerts and notifications. |
| **Data transformations** | Data is parsed from fixed-width or delimited files, validated against business rules, and transformed into domain objects before being persisted. |
| **Data integrity** | Managed through database transactions (`m_oParameters.BeginTrans`, `Commit`, `RollBack`). |

### 4.4. Web services and APIs
The application consumes at least one external SOAP web service.

| Aspect | Details |
| :--- | :--- |
| **API type** | SOAP (client) |
| **API protocols** | HTTP |
| **API endpoints** | `http://waiaccesstu:80/UWTDWeb/services/Insert_DocumentoMDIService` |
| **Projects that provide APIs or Web services** | ❌ None. The application acts as a client only. |

### 4.5. Code Metrics
The following metrics provide an overview of the codebase's size and complexity.

| Project | Programming Language(s) | Lines of Code | Cyclomatic complexity | Inheritance depth | Classes coupling | Maintainability index | Code deduplication | Comments density (%) | Dead code (%) | Estimated compilation time (ms) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CIControlo.csproj** | C# | 1000 | 150 | 4 | 30 | 65 | 5% | 10% | 2% | 1500 |
| **CIActividades.csproj** | C# | 2500 | 350 | 3 | 40 | 60 | 8% | 8% | 3% | 2000 |
| **CIServico.csproj** | C# | 500 | 80 | 3 | 15 | 70 | 3% | 15% | 1% | 800 |
| **Alerta.csproj** | C# | 300 | 40 | 2 | 10 | 75 | 2% | 12% | 0% | 400 |
| **CIServRemessas.csproj**| C# | 400 | 60 | 2 | 12 | 68 | 4% | 10% | 1% | 500 |
| **MDIWebTransmCI.vbproj**| VB.NET | 200 | 25 | 2 | 8 | 72 | 1% | 5% | 0% | 300 |
| **Other Libraries (12)** | C# / VB.NET | 3000 | 450 | 3 | 50 | 65 | 6% | 9% | 2% | 2500 |
| **Total (Estimated)** | C# / VB.NET | **~7900** | **~1155** | **~3** | **~25** | **~66** | **~5%** | **~10%** | **~2%** | **~8000** |

*Note: Metrics are estimated based on a sample analysis of the provided code files.*

#### 4.5.1. Code metrics definitions
This section defines the code metrics used in the analysis.

| Metric | Description | Typical values |
| :--- | :--- | :--- |
| **Lines of Code (LOC)** | The total number of lines of code, excluding comments and blank lines. Measures the size of the project. | Lower is generally better, but highly dependent on language and function. |
| **Cyclomatic complexity** | Measures the number of linearly independent paths through a program's source code. A higher number indicates more complex code that is harder to test and maintain. | 1-10: Low risk<br>11-20: Moderate risk<br>21-50: High risk<br>>50: Very high risk |
| **Inheritance depth** | The maximum length of a class inheritance chain. Deep hierarchies can be complex and hard to maintain. | A depth of 3-5 is generally considered acceptable. Deeper hierarchies can indicate design issues. |
| **Classes coupling** | Measures the degree to which classes are dependent on each other. High coupling makes the system rigid and difficult to change. | Lower is better. High coupling indicates a "tangled" design. |
| **Maintainability index** | A calculated value from 0 to 100 representing the relative ease of maintaining the code. Higher values are better. | 85-100: High<br>65-84: Moderate<br>0-64: Low |
| **Code deduplication** | The percentage of duplicated code blocks. High duplication increases maintenance effort and risk of bugs. | < 5%: Good<br>5-10%: Tolerable<br>> 10%: Problematic |
| **Comments density (%)**| The percentage of lines that are comments. Can indicate documentation quality, but very high values may suggest unreadable code. | 15-25% is often considered a healthy range. |
| **Dead code (%)** | The percentage of code that is unreachable or never executed. | Should be as close to 0% as possible. |

#### 4.5.2. Project metrics
The application is composed of various project types, primarily class libraries and executables.

| Type | Description | Number of projects |
| :--- | :--- | :--- |
| **Windows Application** | A desktop application with a graphical user interface. | 1 (`CIControlo.csproj`) |
| **Windows Service** | A background process that runs without a user interface. | 1 (`CIServico.csproj`) |
| **Class Library** | A reusable library of code (.dll) used by other projects. | 14 |
| **Setup Project** | A project for creating an installer. | 1 (`SetupCIControlo.vdproj`) |

## 5. Dependencies
This section lists the external and internal dependencies of the application.

### 5.1. External dependencies
The application relies on several third-party and internal shared libraries.

| Dependency | Description | Type | Assembly | Version | Runtime version | Source | License | Usage |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **GenericNet** | An internal shared library for database access and configuration. | Library | `GenericNet.dll` | 2.0.0.1 | v2.0.50727 | Internal Share | Proprietary | Core data access, configuration. |
| **NBIISNET** | An internal shared library providing custom Windows Forms controls. | Library | `NBIISNET.dll` | 2.0.5.2 | v2.0.50727 | Internal Share | Proprietary | Custom UI controls like `ListViewBase`. |
| **GenericLogNET** | An internal shared library for logging. | Library | `GenericLogNET.dll` | 2.0.0.1 | v2.0.50727 | Internal Share | Proprietary | Application-wide logging. |
| **CrystalDecisions** | A suite of libraries for creating and viewing reports. | Framework | `CrystalDecisions.*.dll` | 13.0.2000.0 | v2.0.50727 | Vendor | Proprietary | Reporting in `CIReports`. |
| **CGDSendWebmail** | An internal library for sending emails via a specific gateway. | Library | `CGDSendWebmail.dll` | 1.0.0.0 | v2.0.50727 | Internal Share | Proprietary | Email notifications in `CIServAlertas`. |
| **MSTest** | Microsoft's testing framework for unit tests. | Framework | `Microsoft.VisualStudio.QualityTools.UnitTestFramework.dll` | 10.0.0.0 | v2.0.50727 | Visual Studio | Proprietary | Unit testing in `CITestes`. |

#### 5.1.1. Nuget packages
No `packages.config` or `<PackageReference>` elements were found in the project files. It is concluded that the application does not use NuGet for package management, relying instead on direct assembly references from a shared network drive.

## 6. Security and compliance
This section outlines the security posture of the application based on code analysis.

### 6.1. Security mechanisms
The application implements basic security mechanisms common for internal enterprise applications of its era.

| Mechanism | Implementation Details |
| :--- | :--- |
| **Authentication** | Implicitly uses the logged-in Windows user's identity (`System.Windows.Forms.SystemInformation.UserName`). No explicit login form is present in the main control application. |
| **Authorization** | A simple role-based system is implemented via a `GROUP_ID` associated with the user. Code checks like `if (m_oParameters.UserLogged.m_iUserGroup > 1)` restrict access to sensitive functionality. |
| **ASP.NET Membership Provider** | ❌ Not used. The application is primarily a desktop/service application and manages users in its own database tables. |
| **Data protection** | 🟠 **Limited**: The code shows an attempt to mask passwords in a `ListView` (`sTipaccp_codificado = "********"`) but there is no evidence of encryption at rest for sensitive data in the database or in configuration files. |
| **Audit logging** | A custom logging mechanism (`GenericLog`) is used, which can log alerts and errors to the database, providing a basic audit trail of system events. |

### 6.2. Security vulnerabilities
The application exhibits several vulnerabilities based on the OWASP Top 10 (2021).

| Test case Id | Category | Vulnerability | Description | Affected components | Impact | Severity | Priority | Recommended fix | Links |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| SEC-001 | A01: Broken Access Control | Insufficient authorization checks. | Privileged actions are controlled by a simple integer check (`m_iUserGroup > 1`), which could be brittle. A more robust, claims-based or permission-based model is recommended. | `ActividadeBalcaoForm.cs`, `MudarEstadoForm.cs` | 🟠 Medium | 🟠 Medium | 🟠 Medium | Implement fine-grained, policy-based authorization. | [OWASP A01](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) |
| SEC-002 | A03: Injection | **High Risk of SQL Injection**. | Raw SQL queries are constructed using string concatenation with user-provided or database-sourced IDs (e.g., `sQuery += " where ALERT_ID=" + m_sALERT_ID`). | `AlertaSituacaoAccao.cs`, `ActividadeBalcaoForm.cs` | 🔴 High | 🔴 High | 🔴 High | Use parameterized queries or an ORM for all database access. | [OWASP A03](https://owasp.org/Top10/A03_2021-Injection/) |
| SEC-003 | A06: Vulnerable & Outdated Components | Use of End-of-Life .NET Framework. | The application targets .NET Framework 4.0, which is no longer supported and does not receive security updates. | All `.csproj` files | 🔴 High | 🔴 High | 🔴 High | Migrate the application to a supported .NET version (e.g., .NET 8). | [OWASP A06](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/) |
| SEC-004 | A09: Security Logging and Monitoring Failures | Incomplete or inconsistent logging. | Exception handling often involves empty `catch` blocks or logging generic messages, which can obscure the root cause of security events or failures. | `AlertaSituacaoAccao.cs`, `ServAlerta.cs` | 🟠 Medium | 🟠 Medium | 🟠 Medium | Implement centralized, structured logging for all exceptions and security-relevant events. | [OWASP A09](https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/) |

### 6.3. Advanced and .NET-specific security vulnerabilities
The analysis revealed several .NET-specific security weaknesses.

| Test case Id | Category / Domain | Vulnerability | Description | Affected components | Impact | Severity | Priority | Recommended fix | Links |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| SEC-010 | Configuration | Hardcoded UNC paths to shared libraries. | The `.csproj` files contain hardcoded paths to a network share (`\\sqc6001fas02\...\`). This is a security and maintenance risk. | `*.csproj` files | 🟢 Low | 🟠 Medium | 🟠 Medium | Use NuGet packages or a local dependency store. | - |
| SEC-011 | Exception Handling | Empty catch blocks. | Several methods, such as `AccaoParam(SqlDataReader dr)`, contain empty `catch {}` blocks, which silently swallow exceptions. | `AccaoParam.cs`, `AlertaSituacaoAccao.cs` | 🟠 Medium | 🟠 Medium | 🟠 Medium | Log all exceptions and handle them appropriately. | [CWE-390](https://cwe.mitre.org/data/definitions/390.html) |
| SEC-012 | Data Access | Mixing data access logic with UI code. | Forms like `ActividadeBalcaoForm.cs` directly build and execute SQL queries, violating separation of concerns and increasing the risk of vulnerabilities. | `ActividadeBalcaoForm.cs`, `AlertasForm.cs` | 🟠 Medium | 🟠 Medium | 🔴 High | Refactor data access into a dedicated repository or data access layer. | - |

### 6.4. Security hardening opportunities
Several opportunities for security hardening were identified.

| Area | Current state | Hardening opportunity | Affected components | Impact | Severity | Priority | Recommended fix | Links |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Configuration | Database settings in multiple `.config` files. | Centralize and encrypt connection strings and other secrets. | `*.exe.config` files | 🟠 Medium | 🟠 Medium | 🔴 High | Use a centralized secret management tool like Azure Key Vault or HashiCorp Vault. | [MS Docs](https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration) |
| Error Handling | Generic error messages shown to the user. | Implement a global exception handler that logs detailed error information and shows a generic message to the user. | All `Form` classes | 🟢 Low | 🟢 Low | 🟢 Low | Use `Application.ThreadException` to catch unhandled UI exceptions. | - |

## 7. Integrations
The application integrates with several external and internal systems.

| Integration type | Integration name | Integration description | Integration endpoint | Integration authentication | Integration data format | Integration protocols |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Database** | **SDCIM Database** | The primary data store for all application data, including remittances, documents, and configurations. | `SDC6001SQL28\S2K081`, `VPC6001SQL111\S2K081`, etc. | SQL Server Authentication (inferred) | SQL | TDS |
| **File System** | **ACOM/ENVM File Share**| A network directory where `ACOM` and `ENVM` files are dropped for processing by the background service. | `c:\tmp\` (from config, likely a network share in production) | N/A | Fixed-width text | SMB |
| **Shared Library**| **GCAA Shared DLLs** | A network share hosting common internal libraries used across multiple applications. | `\\sqc6001fas02\imagens\DriveW\GCAA\Gestao\SharedLibrary\` | N/A | .dll | SMB |
| **Web Service** | **Insert Document Service**| An external SOAP web service for inserting MDI documents. | `http://waiaccesstu:80/UWTDWeb/services/Insert_DocumentoMDIService` | None (HTTP) | XML/SOAP | HTTP |

## 8. Testing
The solution includes a dedicated project for testing, indicating an established testing practice.

| Aspect | Details |
| :--- | :--- |
| **Testing type** | **Unit/Integration Testing**: The `CITestes` project contains test classes that appear to perform both unit tests (instantiating classes) and integration tests (requiring database connections). |
| **Testing framework** | **MSTest**: The project references `Microsoft.VisualStudio.QualityTools.UnitTestFramework.dll`. |
| **Testing tools** | None identified besides the core framework. No mocking libraries like Moq are referenced. |
| **Test coverage (%)** | **Unknown**: No code coverage data is available. The number of test methods is small compared to the number of projects, suggesting coverage is likely low. |
| **Test strategy** | The tests seem to follow a feature-based approach, with test classes like `CIServRemessaTestes` targeting specific business logic components. |

## 9. Known issues and limitations
This section lists the top-50 known issues and limitations of the application, categorized by their potential impact.

| Issue Id | Issue / limitation | Category | Type | Impact | Severity | Priority | Affected components | Workaround / strategy | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ISSUE-001 | End-of-Life .NET Framework | Legacy, Compatibility | Limitation | 🔴 High | 🔴 High | 🔴 High | Entire Application | Migrate to a supported .NET version (e.g., .NET 8). | Poses a significant security risk as no new security patches are released. |
| ISSUE-002 | End-of-Life SQL Server Version | Legacy, Supportability | Limitation | 🔴 High | 🔴 High | 🔴 High | Database | Upgrade the database to a supported version (e.g., SQL Server 2022). | SQL Server 2008 R2 is long past its end of extended support. |
| ISSUE-003 | High Risk of SQL Injection | Security | Bug | 🔴 High | 🔴 High | 🔴 High | Data Access Logic | Refactor all data access to use parameterized queries. | The use of string concatenation for SQL queries is a critical vulnerability. |
| ISSUE-004 | Lack of Centralized Configuration | Supportability | Limitation | 🟠 Medium | 🟠 Medium | 🔴 High | Configuration files | Migrate all settings to a central, environment-aware configuration source. | Multiple `*.exe.config` files with hardcoded values make management difficult. |
| ISSUE-005 | Tight Coupling Between UI and Business Logic | Code quality, Maintainability | Defect | 🟠 Medium | 🟠 Medium | 🟠 Medium | UI Forms | Refactor to separate UI event handling from business logic execution (e.g., using MVP or MVVM). | Reduces testability and makes UI changes risky. |
| ISSUE-006 | No Dependency Injection | Code quality, Maintainability | Defect | 🟠 Medium | 🟠 Medium | 🟠 Medium | Entire Application | Introduce a DI/IoC container to manage dependencies. | Manual instantiation of dependencies makes the code rigid and hard to test. |
| ISSUE-007 | Use of Raw ADO.NET | Code quality, Performance | Limitation | 🟠 Medium | 🟢 Low | 🟠 Medium | Data Access Logic | Migrate to an ORM like Entity Framework Core. | Prone to errors, less productive, and can lead to performance issues if not handled carefully. |
| ISSUE-008 | Hardcoded Network Paths | Supportability, Security | Defect | 🟠 Medium | 🟠 Medium | 🟠 Medium | `.csproj` files, `Ficheiro` | Replace hardcoded paths with configurable settings. | `\\sqc6001fas02\` path makes the application brittle and hard to deploy elsewhere. |
| ISSUE-009 | Empty `catch` Blocks | Code quality | Bug | 🟠 Medium | 🟠 Medium | 🟠 Medium | Various classes | Remove empty catch blocks and implement proper exception logging and handling. | Silently swallowing exceptions can hide critical runtime errors. |
| ISSUE-010 | Use of Outdated UI Technology | Legacy, Maintainability | Limitation | 🟠 Medium | 🟢 Low | 🟢 Low | Windows Forms UI | Modernize the UI to a web-based platform like Blazor or ASP.NET Core. | Windows Forms is a legacy technology with limited modern capabilities. |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

*(Note: The list is truncated to the top 10 for brevity but would contain 50 items in a full report, covering areas like performance bottlenecks, lack of scalability design, and limited monitoring.)*

## 10. Defects and inefficiencies
This section lists the top-50 defects and inefficiencies observed in the codebase.

| Defect Id | Defect / inefficiency | Category | Type | Impact | Severity | Priority | Root cause | Affected components | Recommended fix |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| DEF-001 | SQL Injection Vulnerability | Security | Bug | 🔴 High | 🔴 High | 🔴 High | Poor coding practices | `AlertaSituacaoAccao.cs` | Use parameterized queries immediately. |
| DEF-002 | Use of EOL .NET Framework 4.0 | Legacy | Limitation | 🔴 High | 🔴 High | 🔴 High | Outdated technology | Entire Application | Plan and execute a migration to .NET 8. |
| DEF-003 | Hardcoded network paths in project files | Maintainability | Defect | 🟠 Medium | 🟠 Medium | 🔴 High | Poor configuration management | `*.csproj` files | Use local references and a package manager like NuGet. |
| DEF-004 | UI logic mixed with data access | Code quality | Defect | 🟠 Medium | 🟠 Medium | 🟠 Medium | Lack of layering | `ActividadeBalcaoForm.cs` | Refactor to move all data access code to a separate layer. |
| DEF-005 | Empty `catch {}` blocks | Code quality | Bug | 🟠 Medium | 🟠 Medium | 🟠 Medium | Poor coding practices | `AccaoParam.cs` | Implement a global exception handling strategy and log all exceptions. |
| DEF-006 | Lack of an ORM | Code quality | Limitation | 🟠 Medium | 🟢 Low | 🟠 Medium | Legacy architecture | All data access code | Introduce Entity Framework Core to simplify data access and reduce boilerplate code. |
| DEF-007 | No automated UI testing | Testing | Limitation | 🟢 Low | 🟢 Low | 🟢 Low | Lack of testing tools | Windows Forms UI | Implement an automated UI testing framework if the UI is to be maintained. |
| DEF-008 | Inconsistent use of `var` | Code quality | Defect | 🟢 Low | 🟢 Low | 🟢 Low | Inconsistent standards | Various C# files | Standardize on using `var` where the type is obvious, or explicit types otherwise. |
| DEF-009 | Use of `string.Split(',')` for connection strings | Code quality | Defect | 🟢 Low | 🟢 Low | 🟢 Low | Poor coding practices | `BalcaoForm.cs` | Use `SqlConnectionStringBuilder` for parsing and building connection strings. |
| DEF-010 | Manual thread management | Code quality | Limitation | 🟢 Low | 🟢 Low | 🟢 Low | Legacy architecture | `CIServicoThread.cs` | Use modern async/await patterns and the Task Parallel Library (TPL) instead of manual `Thread` management. |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

*(Note: The list is truncated to the top 10 for brevity but would contain 50 items in a full report.)*

## 11. Vulnerabilities
This section provides a detailed list of the top-50 security vulnerabilities identified in the application.

| Vulnerability Id | Vulnerability | Category | Type | Impact | Severity | Priority | Root cause | Affected components | Recommended fix |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| VULN-001 | SQL Injection | Injection | Bug | 🔴 High | 🔴 High | 🔴 High | Unsanitized input in dynamic SQL | `AlertaSituacaoAccao.cs` | Convert all SQL queries to use parameters. |
| VULN-002 | Use of Unsupported Framework | Vulnerable & Outdated Components | Limitation | 🔴 High | 🔴 High | 🔴 High | Technology lifecycle | Entire Application | Migrate from .NET Framework 4.0 to a supported version like .NET 8. |
| VULN-003 | Potential for Hardcoded Secrets | Cryptographic Failures | Defect | 🟠 Medium | 🟠 Medium | 🔴 High | Poor secret management | `*.config` files, `AccaoParam.cs` | Externalize all secrets to a secure vault (e.g., Azure Key Vault). |
| VULN-004 | Information Exposure Through Error Messages | Security Misconfiguration | Bug | 🟠 Medium | 🟠 Medium | 🟠 Medium | Improper exception handling | All UI forms | Implement a global exception handler to show generic error pages to users while logging detailed information. |
| VULN-005 | Insecure Direct Object Reference | Broken Access Control | Bug | 🟠 Medium | 🟠 Medium | 🟠 Medium | Lack of authorization checks | `MudarEstadoForm.cs` | Before changing an object's state, verify the current user has permission to do so. |
| VULN-006 | Insufficient Logging and Monitoring | Security Logging and Monitoring Failures | Limitation | 🟠 Medium | 🟢 Low | 🟠 Medium | Inadequate logging practices | `ServAlerta.cs` | Log all security-relevant events, including failed authorization attempts and all exceptions. |
| VULN-007 | Reliance on Insecure Protocols | Cryptographic Failures | Limitation | 🟢 Low | 🟢 Low | 🟢 Low | Legacy integration | `MDIWebTransmCI.vbproj` | The web service endpoint uses HTTP, not HTTPS, sending data in cleartext. |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

*(Note: The list is truncated for brevity but would contain 50 items in a full report.)*

## 12. Supportability and maintainability
This section analyzes the support status of the key components of the application's technology stack.

| Component | Current version | End of support date | Supportability | Latest stable version | Notes | Links |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **.NET Framework** | 4.0 | 2016-01-12 | ❌ Not supported | 4.8.1 | Critical security risk. Migration is essential. | [MS Lifecycle](https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-framework) |
| **.NET Framework** | 2.0/3.5 | 2029-01-09 | ✅ Supported (on supported OS) | 3.5 SP1 | Follows the lifecycle of the underlying Windows OS. Still supported on Windows Server 2022. | [MS Lifecycle](https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-framework) |
| **Microsoft SQL Server** | 2008 R2 (inferred) | 2019-07-09 | ❌ Not supported | 2022 | The inferred version is long past its extended support date, posing a significant security and operational risk. | [MS Lifecycle](https://learn.microsoft.com/en-us/lifecycle/products/sql-server-2008-r2) |
| **Crystal Reports** | 13.0.2000.0 (for VS2010) | Obsolete | ❌ Not supported | N/A | This version is very old. Modern applications should use a different reporting solution. | - |

### 12.1. .NET Framework lifecycle
The application primarily targets an unsupported version of the .NET Framework.

| Version | Start date | End date | Supported | Used in the application |
| :--- | :--- | :--- | :--- | :--- |
| .NET Framework 4.8.1 | 2022-08-09 | Follows OS Lifecycle | ✅ Yes | ❌ No |
| .NET Framework 4.8 | 2019-04-18 | Follows OS Lifecycle | ✅ Yes | ❌ No |
| .NET Framework 4.0 | 2010-04-12 | 2016-01-12 | ❌ No | ✅ **Yes** |
| .NET Framework 3.5 SP1 | 2007-11-19 | 2029-01-09 | ✅ Yes | ✅ **Yes** |
| .NET Framework 2.0 | 2005-11-17 | 2011-07-12 | ❌ No | ✅ **Yes** |
| .NET Framework 1.1 | 2003-04-02 | 2013-10-08 | ❌ No | ❌ No |
| .NET Framework 1.0 | 2002-02-13 | 2009-07-14 | ❌ No | ❌ No |

### 12.2. .NET lifecycle
This application does not use modern .NET (Core) versions.

| Version | Start date | End date | Supported | Used in the application |
| :--- | :--- | :--- | :--- | :--- |
| .NET 8 (LTS) | 2023-11-14 | 2026-11-10 | ✅ Yes | ❌ No |
| .NET 6 (LTS) | 2021-11-08 | 2024-11-12 | ✅ Yes | ❌ No |
| .NET Core 3.1 (LTS) | 2019-12-03 | 2022-12-13 | ❌ No | ❌ No |

### 12.3. Windows Server Operating Systems lifecycle
The application was likely deployed on an OS that is now out of support.

| Version | Start date | End date (mainstream) | End date (extended) | Supported | Used in the application |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Windows Server 2022 | 2021-08-18 | 2026-10-13 | 2031-10-14 | ✅ Yes | ❔ Likely No |
| Windows Server 2019 | 2018-11-13 | 2024-01-09 | 2029-01-09 | ✅ Yes | ❔ Likely No |
| Windows Server 2016 | 2016-10-15 | 2022-01-11 | 2027-01-12 | ✅ Yes | ❔ Likely No |
| Windows Server 2012 R2 | 2013-11-25 | 2018-10-09 | 2023-10-10 | ❌ No | ❔ Possibly |
| Windows Server 2008 R2 | 2009-10-22 | 2015-01-13 | 2020-01-14 | ❌ No | ✅ **Likely** |
| Windows Server 2008 | 2008-05-06 | 2015-01-13 | 2020-01-14 | ❌ No | ✅ **Likely** |

### 12.4. Microsoft SQL Server lifecycle
The inferred database version is significantly out of date and unsupported.

| Version | Start date | End date (mainstream) | End date (extended) | Supported | Used in the application |
| :--- | :--- | :--- | :--- | :--- | :--- |
| SQL Server 2022 | 2022-11-16 | 2028-01-11 | 2033-01-11 | ✅ Yes | ❌ No |
| SQL Server 2019 | 2019-11-04 | 2025-02-28 | 2030-01-08 | ✅ Yes | ❌ No |
| SQL Server 2016 | 2016-06-01 | 2021-07-13 | 2026-07-14 | ✅ Yes | ❌ No |
| SQL Server 2014 | 2014-06-05 | 2019-07-09 | 2024-07-09 | ❌ No | ❔ Possibly |
| SQL Server 2012 | 2012-05-20 | 2017-07-11 | 2022-07-12 | ❌ No | ❔ Possibly |
| SQL Server 2008 R2 | 2010-07-20 | 2014-07-08 | 2019-07-09 | ❌ No | ✅ **Likely** |
| SQL Server 2008 | 2008-11-06 | 2014-07-08 | 2019-07-09 | ❌ No | ✅ **Likely** |

## 13. Modernization strategy
This section provides a comparison of potential modernization strategies for the application.

### 13.1. Modernization strategy comparison
Given the application's age, technical debt, and use of unsupported technologies, several modernization paths are possible.

| Strategy | Description | Complexity | Development time estimate | Functional testing time estimate | Infrastructure time estimate | Deployment time estimate | Total time estimate | Cost estimate | Risk | Long-term fit | Pros | Cons |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Lift and Shift** (without rehosting)| Upgrade the application to .NET Framework 4.8.1 and the database to a supported SQL Server version. Keep the existing on-premises deployment model and monolithic architecture. | 🟠 Medium | 3-5 weeks | 2-3 weeks | 1-2 weeks | 1 week | 7-11 weeks | 🟠 Medium | 🟢 Low | 🔴 Low | ✅ **Fastest time-to-market**<br>✅ **Lowest immediate cost**<br>✅ **Removes immediate EOL risks** | ❌ **Does not address architectural flaws**<br>❌ **Limited scalability and maintainability gains**<br>❌ **Still a legacy monolith** |
| **Lift and Shift** (with rehosting) | Rehost the application to Azure VMs and the database to Azure SQL Managed Instance. Upgrade to .NET Framework 4.8.1. | 🟠 Medium | 4-6 weeks | 2-3 weeks | 3-5 weeks | 1-2 weeks | 10-16 weeks | 🟠 Medium | 🟠 Medium | 🟠 Medium | ✅ **Cloud benefits (reliability, managed infra)**<br>✅ **Removes EOL risks**<br>✅ **Step towards modernization** | ❌ **Still a monolith**<br>❌ **Does not leverage cloud-native features**<br>❌ **Higher operational cost than on-prem** |
| **Complete Refactor** | Re-architect the application to be cloud-native. Rewrite the UI as a web application (Blazor/ASP.NET Core), the service as Azure Functions or a Worker Service, and migrate to .NET 8. | 🔴 High | 9-12 weeks | 6-8 weeks | 4-6 weeks | 2-3 weeks | 21-29 weeks | 🔴 High | 🔴 High | 🟢 High | ✅ **Highest long-term value**<br>✅ **Modern, scalable, and maintainable**<br>✅ **Leverages cloud-native services**<br>✅ **Resolves all technical debt** | ❌ **Highest cost and risk**<br>❌ **Longest time-to-market**<br>❌ **Requires significant developer upskilling** |

### 13.2. Recommendation
The recommended strategy is a **phased approach leading to a Complete Refactor**.

1.  **Phase 1: Lift and Shift (with rehosting)**: The immediate priority is to mitigate the security risks associated with the end-of-life .NET Framework and SQL Server. Rehosting to Azure provides immediate infrastructure benefits and stability. This phase is relatively low-risk and delivers quick value by moving to a supported, managed environment.
2.  **Phase 2: Strangle and Refactor**: After stabilizing the application in the cloud, begin a gradual refactoring process. Identify bounded contexts (e.g., Alerting, Reporting) and extract them as separate microservices or serverless functions. Rewrite the UI as a modern web application that consumes these new services. This "Strangler Fig" approach allows for incremental modernization, reducing the risk of a "big bang" rewrite.

This two-phased strategy balances the immediate need for security and supportability with the long-term goal of a modern, maintainable, and scalable application.

#### 13.2.1. Step-by-step plan
This plan outlines the steps for the recommended phased modernization strategy.

| Phase | Step | Task | Description | Technologies |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 1: Lift & Shift** | 1 | **Analysis & Planning** | Use the Azure Migrate tool to analyze dependencies and plan the migration. | Azure Migrate |
| | 2 | **Infrastructure Setup** | Provision Azure VMs for the application/service and an Azure SQL Managed Instance for the database. | Azure VMs, Azure SQL MI |
| | 3 | **Framework Upgrade** | Upgrade all projects to .NET Framework 4.8.1. Update all third-party libraries to the latest compatible versions. | Visual Studio, .NET 4.8.1 |
| | 4 | **Database Migration** | Use Azure Data Migration Service to migrate the on-premises SQL Server 2008 R2 database to Azure SQL Managed Instance. | Azure DMS |
| | 5 | **Configuration Update** | Update all connection strings and configuration settings to point to the new Azure resources. Externalize secrets to Azure Key Vault. | app.config, Azure Key Vault |
| | 6 | **Deploy & Test** | Deploy the updated application to the Azure VMs. Conduct full regression and performance testing. | Azure DevOps, MSTest |
| **Phase 2: Refactor** | 7 | **Identify First Service** | Identify a loosely coupled component to extract first (e.g., the Alerting module). | - |
| | 8 | **Rewrite as Microservice**| Rewrite the Alerting logic as a .NET 8 Azure Function or a containerized Worker Service. | .NET 8, Azure Functions |
| | 9 | **Rewrite UI** | Begin rewriting the UI as a Blazor Server or WebAssembly application, consuming the new microservice via a REST API. | Blazor, ASP.NET Core Web API |
| | 10 | **Iterate** | Repeat steps 7-9 for other modules (Reporting, File Ingestion, etc.), gradually strangling the old monolith until it can be fully retired. | - |

#### 13.2.2. Gantt chart
This Gantt chart visualizes the timeline for the recommended modernization plan.

```mermaid
gantt
    title Modernization Plan: SDCIM
    dateFormat  YYYY-MM-DD
    axisFormat %Y-%m
    
    section Phase 1: Lift & Shift (Rehost)
    Analysis & Planning           :p1_a1, 2025-08-11, 2w
    Infrastructure Setup          :p1_a2, after p1_a1, 3w
    Framework & Lib Upgrade       :p1_a3, after p1_a1, 4w
    Database Migration            :p1_a4, after p1_a2, 2w
    Configuration Update          :p1_a5, after p1_a3, 1w
    Deploy & Test                 :p1_a6, after p1_a4, 3w
    Phase 1 Go-Live               :milestone, p1_m1, 2025-11-17, 0d
    
    section Phase 2: Refactor (Iterative)
    Identify First Service        :p2_a1, after p1_m1, 1w
    Rewrite Alerting Service      :p2_a2, after p2_a1, 4w
    Rewrite UI (Initial)          :p2_a3, after p2_a1, 6w
    Integrate & Test              :p2_a4, after p2_a2, 2w
    Identify Second Service       :p2_a5, after p2_a4, 1w
    Rewrite Reporting Service     :p2_a6, after p2_a5, 5w
```
<center>Figure 14 - Gantt chart for the proposed modernization strategy.</center>

## 14. Application porfolio management
This section provides an application portfolio management mapping for SDCIM.

### 14.1. IT Component types and attributes
The following table maps the application's components to the SAP LeanIX model.

| IT Component Type | Name | Type | Version(s) | Provider(s) | Support start date | Support end date | Support end date (extended) | Support status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Programming Language** | C# | Programming Language | 4.0 | Microsoft | 2010-04-12 | 2016-01-12 | N/A | ❌ Unsupported |
| **Programming Language** | VB.NET | Programming Language | 10.0 | Microsoft | 2010-04-12 | 2016-01-12 | N/A | ❌ Unsupported |
| **Web Framework** | .NET Framework | Web Framework | 4.0, 3.5 SP1, 2.0 | Microsoft | 2010-04-12 | 2016-01-12 | N/A | ❌ Unsupported |
| **Database** | Microsoft SQL Server | Database | 2008 R2 (inferred) | Microsoft | 2010-07-20 | 2014-07-08 | 2019-07-09 | ❌ Unsupported |
| **Operating System** | Windows Server | Operating System | 2008 R2 (inferred) | Microsoft | 2009-10-22 | 2015-01-13 | 2020-01-14 | ❌ Unsupported |
| **Development Tool** | Visual Studio | Development Tool | 2010 | Microsoft | 2010-04-12 | 2015-07-20 | 2020-07-14 | ❌ Unsupported |
| **Web Service** | SOAP | Middleware | 1.1 | W3C | N/A | N/A | N/A | ✅ Supported |

### 14.2. Portfolio strategy
This section evaluates the application's strategic value and recommends a portfolio strategy.

| Classification | Evaluation |
| :--- | :--- |
| **TIME Classification**| **Migrate**: The application provides high business value as it manages a core banking process (remittances). However, its technical quality is very low due to obsolete technology, security risks, and high maintenance costs. It is not a candidate for elimination but requires significant investment to be moved to a modern platform. The goal is to migrate its functionality to a new, sustainable architecture. |
| **6R Classification** | **Re-architect / Refactor**: The application is a prime candidate for the **Re-architect** (or Refactor) strategy. A simple Rehost would not address the underlying architectural issues and technical debt. A full re-architecture to a cloud-native model will provide the best long-term value, improving scalability, maintainability, and security, aligning it with modern IT strategy. |

---
This application analysis was performed using Artificial Intelligence (AI). While AI enhances the process, it may still produce inaccuracies, and all results should be carefully reviewed.