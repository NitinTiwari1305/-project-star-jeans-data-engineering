<h1 align="center">ETL Building for an E-commerce Jeans Company</h1>

<p align="center">A Data Engineering Project</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/66283452/209023968-1effb930-5759-4694-89d6-8cecc3ee5db1.png" width="550" />
</p>

*Obs 1: The company and business problem are both fictitious, although the data is real.*

*Obs 2: Scraping the H&M website is allowed according to H&M's robots.txt file.*

*The in-depth Python code explanation is available in [this](https://github.com/NitinTiwari1305/-project-star-jeans-data-engineering/blob/main/star-jeans.ipynb) Jupyter Notebook.*

# 1. **Star Jeans and Business Problem**
<p align="justify"> Michael, Franklin, and Trevor, after several successful businesses, are starting a new company called Star Jeans. For now, their plan is to enter the USA fashion market through an E-commerce platform. The initial idea is to sell one product for a specific audience, which is <b>male jeans</b>. Their goal is to keep prices low and slowly increase them as they acquire new clients. However, this market already has strong competitors, such as H&M for instance. In addition to that, the three businessmen aren't familiar with this segment in particular. Therefore, in order to better understand how this market works, they hired a Data Engineering consultant to gather competitor market intelligence regarding H&M. They require tracking the following attributes about H&M male jeans: </p>

- Product Name
- Product Type
- Product Fit
- Product Color
- Product Composition
- Product Price

# 2. **Solution Plan**
## 2.1. How was the problem solved?

<p align="justify"> We gathered information on H&M male jeans by designing and implementing an end-to-end automated ETL pipeline, divided into the following sequential components: </p>

- <b> Understanding the Business Problem</b>: Defining tracking boundaries, setting up structural table models, and sketching out the processing pipeline architecture.

- <b> Extraction </b>: Extracting catalog paths (`product_id` and `product_type`) from the central catalog showroom canvas (Job 01), before parsing deep nested inventory features from individual landing pages into a tabular dataset (Job 02). More details available in <a href="#3-extraction">Section 3</a>.

- <b> Transformation </b>: Data Cleaning, structural string mapping, schema modeling, and raw data type normalization (Job 03). More details available in <a href="#4-transformation">Section 4</a>.

- <b> Loading </b>: Automating relational database ingestion into a persistent cloud database infrastructure instance (Job 04). More details available in <a href="#5-loading">Section 5</a>.

- <b> Streamlit App </b>: Connecting data endpoints to a managed presentation tier (Job 05) and rendering interactive user controls for business analytics visualization (Job 06). More details available in <a href="#6-streamlit-app">Section 6</a>.

<p align="justify">The full engineering system blueprint is documented in the structural <a href="https://docs.google.com/spreadsheets/d/1ipHa7oxNVYF1zpFfDz5yG63RP0GvpRLFOzBozBHIdRA/edit?usp=sharing">ETL Documentation Tracker</a>, and the high-level process framework is mapped below: </p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/66283452/208748904-f7ada2f7-8ced-4bbd-a473-85102fab9c5e.png"/>
</p>

<p align="justify"> All operational jobs execute in strict sequential order. Data pipeline components (Jobs 01-04) are orchestrated using <a href="https://github.com/NitinTiwari1305/-project-star-jeans-data-engineering/blob/main/star-jeans-etl/webscraping-hm.py">this extraction service script</a>, while the user presentation layer (Jobs 05-06) is powered by <a href="https://github.com/NitinTiwari1305/-project-star-jeans-data-engineering/blob/main/star-jeans-etl/streamlit-app/star-jeans-app.py">this dashboard application script</a>. The data acquisition layer executes automatically via Windows Task Scheduler on a weekly loop, seamlessly propagating record updates straight to the presentation layer without demanding manual administrative handling.</p>

## 2.2. Tools and techniques used:
- [Python 3.10.8](https://www.python.org/downloads/release/python-3108/), [Pandas](https://pandas.pydata.org/) and [Beautiful Soup](https://beautiful-soup-4.readthedocs.io/en/latest/).
- [SQL](https://www.w3schools.com/sql/) and [PostgresSQL](https://www.postgresql.org/).
- [Jupyter Notebook](https://jupyter.org/) and [VSCode](https://code.visualstudio.com/).
- [Web Scraping Architecture](https://towardsdatascience.com/a-step-by-step-guide-to-web-scraping-in-python-5c4d9cef76e8).
- [ETL Automated Pipelines](https://www.keboola.com/blog/etl-process-overview) and [OS-level Task Task Orchestration](https://www.windowscentral.com/how-create-automated-task-using-task-scheduler-windows-10).
- [Streamlit Framework](https://streamlit.io/).
- [Git](https://git-scm.com/) and [Github](https://github.com/).

# 3. **Extraction**
<p align="justify"> Data extraction is performed by programmatically crawling H&M male jeans web assets using Python and the Beautiful Soup parser framework. This orchestration layer splits tasks into two independent jobs: </p>

- <p align="justify"> <b> Job 01 (Catalog Scrape)</b>: Aggregates foundational web endpoints and raw root product identifiers directly from the overview catalog display layout, capturing critical high-level categories (`product_type`) that are completely missing from internal item landing pages. Isolates product identifiers into uniform `style_id` (leading 7 digits) and `color_id` (trailing 3 digits) sequences to simplify relational database keys later. </p>

- <p align="justify"> <b> Job 02 (Deep Page Scrape)</b>: Loops through the derived endpoint arrays to parse deep structural features, including descriptive catalog names, specific structural cut metrics (`product_fit`), textile data, and pricing matrices. Injects a running `scraping_datetime` timestamp marker row during each transaction pass to track systemic pricing shifts over time. </p>

# 4. **Transformation**
<p align="justify"> The incoming unstructured arrays are channeled through an automated data cleansing workflow (<b>Job 03</b>). Structural variables are standardized to lower snake_case format across item titles, fits, and pricing matrices to ensure clean querying interfaces. </p>

<p align="justify"> The core structural complexity requires normalizing the nested text string expressions inside the `product_composition` variable into separate, independent feature matrices tracking material volume percentages: cotton, spandex, polyester, elastomultiester, lyocell, and rayon. Duplicate data frames are systematically dropped, low-variance tracking variables are pruned, and the table structures are aligned to this structural production schema model: </p>

<div align="center">

| **Database Field** | **Data Typing & Definition** |
|:--------------------:|----------------|
|      product_id     | 10-digit primary composite key alphanumeric vector (style_id + color_id) |
|      style_id       | 7-digit index string tracking base structural product geometry | 
|      color_id       | 3-digit tracking index detailing item color variation |
|      product_name   | Cleansed item model moniker text string |
|      product_type   | Categorical product taxonomy type group string |
|      product_color  | Cleaned color variation text string |
|      product_fit    | Cut metric geometry classification (e.g., slim_fit, skinny, loose_fit) |
|      cotton         | Numeric continuous percentage tracking cotton content ratio |
|      spandex        | Numeric continuous percentage tracking spandex content ratio |
|      polyester      | Numeric continuous percentage tracking polyester content ratio |
|    elastomultiester | Numeric continuous percentage tracking elastomultiester ratio |
|       lyocell       | Numeric continuous percentage tracking lyocell content ratio |
|       rayon         | Numeric continuous percentage tracking rayon content ratio |
|      product_price  | Continuous float variable mapping item price |
|   scraping_datetime | Structural ISO date-time tracker logging pipeline capture loop |

</div>

# 5. **Loading**
<p align="justify">Following text transformation operations, data records are marshaled into a persistent PostgreSQL relational engine using Python's SQLAlchemy database engine layer. Database hosting relies on a remote relational cloud platform powered by <a href="https://neon.tech/">Neon.tech</a> (<b>Job 04</b>). </p>
  
<p align="justify"><i>CRITICAL SCHEMA POLICY: Incoming structural records are appended incrementally into the target architecture rather than overwriting historical layers. This data history tracking enables analytics engines to compute precise rolling price elasticities and map competitor promotional trends over continuous historical windows.</i> </p>

# 6. **Streamlit App**
<p align="justify"> Streamlit serves as the presentation tier, exposing the relational cloud data points via a clean filtering UI. The front-end layout communicates directly with the cloud database layers across two distinct operations: </p>
  
- <p align="justify"> <b> Job 05 (Database Connection)</b>: Establishes a secure pool connection linking the cloud PostgreSQL instance directly to the Streamlit execution stack. </p>  
- <p align="justify"> <b> Job 06 (Presentation Mapping)</b>: Exposes the data tables inside an interactive UI matrix, supported by dynamic search parameters, material thresholds, and product category filtering nodes. </p>   

<div align="center">

|          **Click below to access the App** |
|:------------------------:|
|          [![Streamlit App](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=Streamlit&logoColor=white)](https://star-jeans.streamlit.app/)
</div>


# 7. **Conclusion**
In this project the main objective was accomplished:
 <p align="justify"> <b> We successfully built an automated, production-grade ETL infrastructure pipeline that extracts unstructured e-commerce data from primary market competitors on a routine loop, runs text-parsing transformations, and updates a central cloud PostgreSQL database. By routing database layers straight to an interactive Streamlit frontend web container, company stakeholders can track market conditions, analyze structural material layouts, and map retail price strategies in the USA male apparel sector. </b> </p>
  
# 8. **Next Steps**
<p align="justify"> Production optimization roadmap items include:
- Migrating local scheduler loops from local task scripts to a fully decoupled workflow engine like <b>Apache Airflow</b> to handle complex dependency mapping and eliminate local system availability dependencies.
- Integrating proxies into the extraction modules to maximize scraping robustness against access rate restrictions.
</p>

# Contact

- nitintiwari1305@gmail.com
- [![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/nitintiwari1305/)
