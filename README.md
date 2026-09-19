### Documentation is included in the Documentation folder ###


### REFrameWork Template ###
**Robotic Enterprise Framework**

* Built on top of *Transactional Business Process* template
* Uses *State Machine* layout for the phases of automation project
* Offers high level logging, exception handling and recovery
* Keeps external settings in *Config.xlsx* file and Orchestrator assets
* Pulls credentials from Orchestrator assets and *Windows Credential Manager*
* Gets transaction data from Orchestrator queue and updates back status
* Takes screenshots in case of system exceptions


### How It Works ###

1. **INITIALIZE PROCESS**
 + ./Framework/*InitiAllSettings* - Load configuration data from Config.xlsx file and from assets
 + ./Framework/*GetAppCredential* - Retrieve credentials from Orchestrator assets or local Windows Credential Manager
 + ./Framework/*InitiAllApplications* - Open and login to applications used throughout the process

2. **GET TRANSACTION DATA**
 + ./Framework/*GetTransactionData* - Fetches transactions from an Orchestrator queue defined by Config("OrchestratorQueueName") or any other configured data source

3. **PROCESS TRANSACTION**
 + *Process* - Process trasaction and invoke other workflows related to the process being automated 
 + ./Framework/*SetTransactionStatus* - Updates the status of the processed transaction (Orchestrator transactions by default): Success, Business Rule Exception or System Exception

4. **END PROCESS**
 + ./Framework/*CloseAllApplications* - Logs out and closes applications used throughout the process


### For New Project ###

1. Check the Config.xlsx file and add/customize any required fields and values
2. Implement InitiAllApplications.xaml and CloseAllApplicatoins.xaml workflows, linking them in the Config.xlsx fields
3. Implement GetTransactionData.xaml and SetTransactionStatus.xaml according to the transaction type being used (Orchestrator queues by default)
4. Implement Process.xaml workflow and invoke other workflows related to the process being automated



This repository contains the **Performer** component of an enterprise-style Bank Reconciliation automation built with UiPath.
 
The solution follows a **Dispatcher / Performer architecture** using:
 
- UiPath Studio
- UiPath Orchestrator Queues
- Robotic Enterprise Framework (REFramework)
- Transaction-based processing
- Business and System Exception handling
- Retry and recovery mechanisms
- Logging and auditability
- Automated testing
 
---
 
## Solution Architecture
 
The complete Bank Reconciliation solution consists of two UiPath processes:
 
```text
Bank Statement / Transaction Source
                |
                v
+--------------------------------+
| Bank Reconciliation Dispatcher |
|                                |
| Read input data                |
| Validate transactions          |
| Prepare queue items            |
| Add items to Orchestrator      |
+---------------+----------------+
                |
                v
        UiPath Orchestrator
          BR_RECON_QUEUE
                |
                v
+--------------------------------+
| Bank Reconciliation Performer  |
|                                |
| REFramework                    |
| Get Queue Item                 |
| Process Transaction            |
| Reconcile Transaction          |
| Handle Exceptions              |
| Set Transaction Status         |
+---------------+----------------+
                |
                v
       Reconciliation Result
Performer Responsibility
The Performer consumes transaction items created by the Dispatcher and stored in the UiPath Orchestrator queue.
The Performer is responsible for processing each transaction independently and updating its final status.
The project is implemented using the UiPath Robotic Enterprise Framework (REFramework) to provide reliability, exception handling, retries, logging and recovery.
REFramework Architecture
The Performer follows the standard REFramework state-machine pattern:
START
  |
  v
INIT
  |
  |-- Load Config.xlsx
  |-- Load Orchestrator Assets
  |-- Initialize applications/resources
  |
  v
GET TRANSACTION DATA
  |
  |-- Retrieve next Queue Item
  |-- No more transactions?
  |       |
  |       +---- YES ----> END PROCESS
  |
  v
PROCESS TRANSACTION
  |
  |-- Validate transaction
  |-- Perform reconciliation
  |-- Apply matching rules
  |-- Generate transaction result
  |
  +-----------------------------+
  |                             |
SUCCESS                  EXCEPTION
  |                             |
  v                             v
Set Successful          Business Exception
Status                  or System Exception
  |                             |
  +-------------+---------------+
                |
                v
       GET NEXT TRANSACTION
                |
                v
           END PROCESS
Orchestrator Queue
The Performer consumes transactions from the following UiPath Orchestrator queue:
BR_RECON_QUEUE
Each queue item represents one reconciliation transaction.
Queue-based processing provides:
Transaction-level isolation
Centralized monitoring
Retry support
Status tracking
Scalability
Improved auditability
Better failure recovery
Dispatcher / Performer Pattern
The project uses a two-process architecture.
Dispatcher
The Dispatcher:
Reads transaction input
        |
        v
Validates input
        |
        v
Transforms data
        |
        v
Creates queue items
        |
        v
BR_RECON_QUEUE
The Dispatcher does not perform the full reconciliation.
Performer
The Performer:
BR_RECON_QUEUE
        |
        v
Get Queue Item
        |
        v
Validate Transaction
        |
        v
Perform Reconciliation
        |
        v
Set Transaction Status
This separation makes the automation easier to scale, recover and maintain.
REFramework States
1. Init
The Init state prepares the automation environment.
Typical responsibilities include:
Load configuration
Load Orchestrator assets
Initialize applications
Initialize connections
Validate required resources
Prepare logging
Relevant workflows may include:
Framework/InitAllSettings.xaml
Framework/InitAllApplications.xaml
Framework/KillAllProcesses.xaml
2. Get Transaction Data
This state retrieves the next transaction from the Orchestrator queue.
The Performer continues requesting queue items until no new transaction is available.
Example flow:
Get Transaction Item
        |
        v
Transaction Found?
   /          \
YES           NO
|              |
v              v
Process        End
Transaction    Process
3. Process Transaction
This is where the main Bank Reconciliation business logic is executed.
The processing workflow can include:
Read queue transaction data
Validate required fields
Retrieve corresponding internal record
Apply matching logic
Compare transaction values
Determine reconciliation result
Prepare output/result data
The core processing workflow is:
Process.xaml
4. End Process
The End Process state performs cleanup before the automation terminates.
Examples include:
Close applications
Release resources
Close connections
Write final logs
Perform cleanup
Exception Handling
The Performer differentiates between two major exception categories.
Business Exception
A Business Exception occurs when the system is functioning correctly but the transaction violates a business rule.
Examples:
Missing transaction reference
Invalid transaction amount
Unsupported transaction type
Transaction already processed
Transaction cannot be matched
Required business data is missing
Business Exceptions are normally not retried automatically because repeating the same transaction will not correct the underlying business problem.
System Exception
A System Exception occurs because of a technical or environmental problem.
Examples:
Application unavailable
Network interruption
Orchestrator communication failure
Database connection failure
File temporarily unavailable
Unexpected application crash
Timeout
System Exceptions may be retried according to the REFramework and Orchestrator retry configuration.
Retry Strategy
The Performer supports transaction retry for recoverable technical failures.
Example:
Transaction
    |
    v
Process
    |
    v
System Exception
    |
    v
Retry Allowed?
   /      \
YES       NO
|          |
v          v
Retry     Failed
Business Exceptions are generally not retried automatically.
System Exceptions may be retried based on the configured retry policy.
Transaction Status
Each queue transaction can finish with one of several statuses.
Successful
 
Failed
   |
   +-- Business Exception
   |
   +-- System Exception
The SetTransactionStatus workflow is responsible for updating the transaction status and recording relevant exception information.
Disaster Recovery
The Dispatcher / Performer architecture provides strong recovery capabilities.
If the robot stops unexpectedly:
Completed transactions
        |
        +---- Remain completed
 
Pending transactions
        |
        +---- Remain available in queue
 
System-failed transactions
        |
        +---- Can be retried
 
New robot session
        |
        +---- Continues remaining work
This means the entire reconciliation batch does not need to restart after a failure.
The queue acts as a persistent transaction store.
Project Structure
Example project structure:
BankReconciliation_Performer/
│
├── Main.xaml
├── Main.xaml.json
├── project.json
├── project.uiproj
├── README.md
├── LICENSE
├── .gitignore
│
├── Data/
│   └── Config.xlsx
│
├── Framework/
│   ├── InitAllSettings.xaml
│   ├── InitAllApplications.xaml
│   ├── KillAllProcesses.xaml
│   ├── GetTransactionData.xaml
│   ├── SetTransactionStatus.xaml
│   ├── RetryCurrentTransaction.xaml
│   ├── TakeScreenshot.xaml
│   └── Process.xaml
│
└── Tests/
    ├── MainTestCase.xaml
    ├── ProcessTestCase.xaml
    ├── GetTransactionDataTest.xaml
    ├── InitAllSettingsTest.xaml
    ├── InitAllApplicationsTest.xaml
    └── Tests.xlsx
Exact file names may differ depending on the REFramework version.
Configuration
Runtime configuration is stored primarily in:
Data/Config.xlsx
The configuration can contain references to:
Queue names
Application paths
Retry settings
Timeout values
Business configuration
Orchestrator Asset names
Environment-specific settings
Sensitive values should not be stored directly in Config.xlsx.
Security
Credentials and sensitive information should never be hard-coded into workflows or committed to source control.
Sensitive information should instead be stored using:
UiPath Orchestrator Credential Assets
UiPath Orchestrator Assets
Environment variables
External secret-management solutions
Examples of information that should not be committed:
Passwords
API keys
Authentication tokens
Production banking information
Customer account information
Real transaction data
Private certificates
Testing
The project contains REFramework test workflows that can be used to validate individual components.
Examples include:
GetTransactionDataTest
InitAllApplicationsTest
InitAllSettingsTest
MainTestCase
ProcessTestCase
WorkflowTestCase
Testing individual workflows helps identify failures before deploying the automation to production.
Running the Project on Another Computer
Clone the repository:
git clone https://github.com/YOUR_USERNAME/uipath-bank-reconciliation-performer.git
Open the cloned project in UiPath Studio.
UiPath Studio should restore the activity-package dependencies defined in the project.
Connect the UiPath Robot or Studio environment to the correct Orchestrator tenant.
Ensure that the following queue exists:
BR_RECON_QUEUE
Also configure any required Orchestrator Assets referenced by Config.xlsx.
Then run:
Main.xaml
Required Environment
The project requires:
UiPath Studio Desktop
UiPath Robot
UiPath Orchestrator
Git
Access to BR_RECON_QUEUE
Required Orchestrator Assets
Required UiPath activity packages
Related Repository
The transaction Dispatcher is maintained separately.
uipath-bank-reconciliation-dispatcher
The Dispatcher reads the transaction source and sends transactions into:
BR_RECON_QUEUE
The Performer then consumes and processes those transactions.
GitHub link:
https://github.com/YOUR_USERNAME/uipath-bank-reconciliation-dispatcher
Replace YOUR_USERNAME with the correct GitHub username.
End-to-End Processing
Source Transactions
       |
       v
Dispatcher
       |
       v
Validate Data
       |
       v
Create Queue Items
       |
       v
BR_RECON_QUEUE
       |
       v
REFramework Performer
       |
       v
Get Transaction
       |
       v
Perform Reconciliation
       |
       +-------------------------+
       |                         |
       v                         v
    Success                  Exception
                                 |
                         +-------+-------+
                         |               |
                         v               v
                    Business         System
                    Exception       Exception
                                         |
                                         v
                                       Retry
       |
       v
Set Queue Status
       |
       v
Next Transaction
       |
       v
Final
