Smart Toll Plaza Management & GIS Command Center

Python 3.9+Framework: StreamlitAI: FastALPR | YOLOv9GIS: Plotly MapboxLicense: MIT

An enterprise-grade, full-stack Python application designed to simulate and manage the National Highways Authority of India (NHAI) transit and toll infrastructure.

The platform integrates an AI-powered Automatic Number Plate Recognition (ANPR) engine, a high-performance Pan-India Geographical Information System (GIS) matrix, dynamic congestion pricing, and financial reconciliation auditing.

SYSTEM ARCHITECTURE & KEY MODULES

smart-toll-plaza-system/

├── app.py│ Central IAM Authentication & Landing Portal

├── database.py│ CloudSQLAdapter ORM, Dynamic Pricing & Logic

├── TOLL\_PLAZA\_LIST @26 may 2026.csv│ Pan-India 2,000+ Plaza Geospatial Dataset

├── cloud\_mock\_db.json│ Persistent Mock Database (Cloud SQL Ready)

├── requirements.txt│ Project Dependencies

└── pages/

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ├── 1_📸_Live_Camera.py    │   Hardware Stream, FastALPR & NETC FASTag Gateway  ├── 2_🗺️_GIS_Command_Center.py    │   Pan-India Infrastructure Mapping & Spatial Filters  └── 3_📊_Analytics.py        Financial Telemetry, Security Audit & Shift Closure   `

CORE FEATURES

1.  📸 AI-POWERED ANPR & PAYMENT PROCESSING
    

File: 1\_📸\_Live\_Camera.py

• Computer Vision & OCREmploys fast\_alpr using YOLO-v9 license plate detection and MobileViT OCR for optical extraction.

• Intelligent Regex FilteringEnforces Indian license plate syntax validation across:

*   Standard plates: OD02AB1234
    
*   Bharat Series: 21BH1234AA
    
*   Diplomatic / UN plates: 77CD12, UN4567
    

• NETC FASTag Gateway SimulationSimulates bank network API handshakes with latency, balance authorization, and decline handlers.

• Security & VAHAN InterceptionCross-references incoming plates against blacklisted stolen/defaulter records to trigger law enforcement alerts.

• Automatic VIP / Diplomatic ExemptionRecognizes diplomatic identifiers such as CD, CC, and UN and logs verified zero-cost passages.

1.  🗺️ PAN-INDIA GIS INFRASTRUCTURE MATRIX
    

File: 2\_🗺️\_GIS\_Command\_Center.py

• 2,000+ Operational Node VisualizationUses Plotly Mapbox with carto-positron to plot the national highway grid across all states.

• Payload Compression EnginePre-compiles HTML hover data through vectorized Pandas operations to reduce browser rendering latency.

• Zero-Toll Region AwarenessDynamically alerts operators when querying states or UTs with zero operational toll plazas:

*   Tripura
    
*   Mizoram
    
*   Manipur
    
*   Nagaland
    
*   Andaman & Nicobar Islands
    
*   Lakshadweep
    
*   Arunachal Pradesh
    
*   Sikkim
    

• Persistent Interactive ProfilesCaptures on-click node events to isolate selected infrastructure metadata cards without resetting the viewport.

1.  💰 DYNAMIC PRICING & FINANCIAL OVERSIGHT
    

Files: database.py & 3\_📊\_Analytics.py

• Congestion / Surge PricingAutomatically detects peak traffic windows:

*   08:00–10:00
    
*   17:00–20:00
    

A 15% tariff multiplier is applied during these periods.

• Operator Shift ReconciliationEnables physical cash drawer auditing against system receipts, logging positive overage or negative shrinkage variances.

• Visual TelemetryVisualizes real-time revenue breakdowns by vehicle class and payment method adoption, including FASTag and Cash.

1.  🔐 ROLE-BASED ACCESS CONTROL (RBAC)
    

Regional Director:

• Full administrative clearance• Pan-India GIS maps• Nationwide analytics• Security audit tables• Infrastructure monitoring• Financial telemetry

Plaza Operator:

• Localized terminal access• Live Camera ANPR• FASTag transaction processing• Shift Closure reconciliation

TECH STACK

Frontend / Framework:

• Streamlit• Plotly Express• Plotly Graph Objects

Machine Learning / Computer Vision:

• FastALPR• YOLOv9• MobileViT OCR• OpenCV• NumPy

Data Processing & Geospatial:

• Pandas• Regular Expressions (re)• Plotly Mapbox

Database / Backend:

• Python• JSON-based ORM• CloudSQLAdapter• Supabase / PostgreSQL migration-ready architecture

INSTALLATION & SETUP

1.  CLONE THE REPOSITORY
    

git clone [https://github.com/](https://github.com/)/smart-toll-plaza-system.git

cd smart-toll-plaza-system

1.  CREATE A VIRTUAL ENVIRONMENT
    

Windows:

python -m venv venv

venv\\Scripts\\activate

macOS / Linux:

python3 -m venv venv

source venv/bin/activate

1.  INSTALL DEPENDENCIES
    

pip install streamlit pandas numpy plotly opencv-python-headless fast-alpr

Or:

pip install -r requirements.txt

1.  ENSURE DATASET PLACEMENT
    

Verify that:

TOLL\_PLAZA\_LIST @26 may 2026.csv

is located in the project root directory alongside app.py.

1.  LAUNCH THE APPLICATION
    

streamlit run app.py

DEMO ACCESS CREDENTIALS

NOTE: These credentials are intended only for local/demo environments. Do not use real credentials in source code or public repositories.

Role: Regional Director

Enterprise Email:[director@nhai.gov](mailto:director@nhai.gov)

Authentication Token:admin88

Node Access:All Nodes (Pan-India)

Role: Plaza Operator

Enterprise Email:[op1@manguli.nhai](mailto:op1@manguli.nhai)

Authentication Token:toll2026

Node Access:Manguli Toll Plaza (Cuttack)

APPLICATION WORKFLOW

User Login↓RBAC Validation↓Role Selection↓Regional Director / Plaza Operator↓Regional Director:GIS + Analytics + Security

Plaza Operator:Live ANPR + FASTag + Shift Closure↓Vehicle Detection↓ANPR / OCR↓License Plate Validation↓Vehicle Classification↓Blacklist Check↓VIP / Diplomatic Check↓FASTag Authorization↓Dynamic Toll Calculation↓Transaction Recording↓Audit & Reconciliation

TOLL TRANSACTION PROCESSING

1.  Vehicle enters the toll lane.
    
2.  Camera captures the vehicle image.
    
3.  ANPR detects the license plate.
    
4.  OCR extracts the registration number.
    
5.  Regex validation verifies the plate format.
    
6.  Vehicle classification is determined.
    
7.  Blacklist / security records are checked.
    
8.  VIP or diplomatic exemption is evaluated.
    
9.  FASTag balance authorization is simulated.
    
10.  Dynamic pricing is calculated.
    
11.  Transaction is recorded.
    
12.  Audit trail is generated.
    

DYNAMIC PRICING LOGIC

Peak Hours:

Morning Peak:08:00 – 10:00

Evening Peak:17:00 – 20:00

During peak hours:

Final Toll = Base Toll × 1.15

Outside peak hours:

Final Toll = Base Toll

SECURITY & DATA INTEGRITY

Immutable Audit Trails:

The application records:

• Manual overrides• VIP exemptions• FASTag wallet declines• Blacklisted vehicle interceptions• Law enforcement alerts• Financial reconciliation events

Each audit event receives a unique identifier similar to:

AUD-XXXXXXXX

Defensive Data Parsing:

During dataset loading:

• Coordinates are validated.• Numeric attributes are coerced.• Invalid spreadsheet values are handled defensively.• Malformed records are prevented from crashing the application.

ANALYTICS DASHBOARD

The analytics module provides:

• Total revenue• Cash collection• FASTag collection• Revenue by vehicle class• Transaction counts• Shift variance• Blacklisted vehicle interceptions• Failed transactions• Manual overrides• VIP exemptions• Security events• Vehicle throughput• Payment-method distribution

GIS COMMAND CENTER

The GIS module provides:

• Interactive toll plaza mapping• State / UT filtering• Toll plaza search• Infrastructure metadata• Geographic coordinates• Operational node visualization• Selected-plaza information cards

EXAMPLE PLATE FORMATS

OD02AB1234

MH12CD5678

DL01AB1234

21BH1234AA

77CD12

UN4567

These examples are illustrative and should not be interpreted as validation of every possible Indian registration format.

FUTURE ENHANCEMENTS

☐ Real-time CCTV camera integration

☐ Production-grade ANPR deployment

☐ Real NETC / FASTag API integration

☐ PostgreSQL / Supabase migration

☐ Cloud deployment

☐ Real-time WebSocket telemetry

☐ Advanced vehicle classification

☐ Multi-lane toll monitoring

☐ SMS / Email security alerts

☐ Mobile operator dashboard

☐ Predictive traffic analytics

☐ AI-based anomaly detection

☐ Automated incident reporting

☐ Multi-factor authentication

☐ Encrypted audit logs

DEPLOYMENT

Typical deployment workflow:

GitHub Repository↓Python Environment↓Install requirements.txt↓Configure Database↓Configure Secrets↓Start Streamlit↓Smart Toll Plaza Dashboard

For production deployment, sensitive credentials and API keys should be stored using environment variables or the platform's secrets-management system.

DISCLAIMER

This project is a software simulation / prototype intended for educational, research, demonstration, and hackathon purposes.

It does not represent an official NHAI system and should not be connected to real toll infrastructure, FASTag banking systems, government databases, or law-enforcement systems without appropriate authorization, security controls, and regulatory compliance.

CONTRIBUTING

Contributions, suggestions, and improvements are welcome.

Suggested workflow:

git clone [https://github.com/](https://github.com/)/smart-toll-plaza-system.git

cd smart-toll-plaza-system

git checkout -b feature/new-feature

git add .

git commit -m "Add new feature"

git push origin feature/new-feature

Then open a Pull Request on GitHub.

LICENSE

Distributed under the MIT License.

See the LICENSE file for more information.

SUPPORT

If you find this project useful, consider giving the repository a ⭐ on GitHub.

Smart Toll Plaza Management & GIS Command Center

AI • GIS • ANPR • FASTag • Analytics • Security