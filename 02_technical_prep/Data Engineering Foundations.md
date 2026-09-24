---
aliases:
  - "data_engineering_fundamentals"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Data Engineering and Modeling/data_engineering_fundamentals.md"
imported: 2026-09-24
---

# Data Engineering Foundations

Review: [[02_technical_prep/system_design/Data Warehousing|Data Warehousing]] · [[02_technical_prep/system_design/Data Processing Systems (Spark)|Data Processing Systems (Spark)]] · [[02_technical_prep/data_modeling/Data Modeling Fundamentals|Data Modeling Fundamentals]]

Concepts already covered in [[02_technical_prep/system_design/Data Warehousing|Data Warehousing]], [[02_technical_prep/system_design/Data Processing Systems (Spark)|Data Processing Systems (Spark)]], [[02_technical_prep/data_modeling/Data Modeling Fundamentals|Data Modeling Fundamentals]]. This note retains the additional examples and procedures.

## Contents

- [[#What is Data Engineering?]]
- [[#Stages of Data Engineering]]
- [[#Role of Data Engineer in Data Science and Analytics]]
- [[#Essential Skills for Success in Senior Data Engineering Roles]]
- [[#Transformation and loading checklist]]

## What is Data Engineering?
- Data engineering is focused on the practical application of data storage, collection, management, and analysis. It involves the design, construction, and maintenance of systems and processes that allow for efficient handling and transformation of data.
  - This includes building **scalable** and reliable data pipelines which transport data from various sources to storage systems and data warehouses, where it can be accessed and used for analysis, and turned into insights.
  - It's especially important for data pipelines to be scalable, since they need to be able handle processing increasing amounts of data quickly, especially with the rapid rise of AI.
- Data can either be structured, semi-structured, or unstructured. Structured data comes in the form of CSV or tabular formats. Semi-structured data comes in the form of XML and JSON. Unstructured data comes in the form of video, images, text, etc.
- Data engineers will start by building a data pipeline to collect data from various sources. Next, they will clean up the data and convert it to a form where it is ready for analysis. The data warehouse is typically the final destination of the data once it has gone through these steps.
- Data engineers work very closely with data scientists to help build the foundation necessary for the data scientist to do their job. This includes:
  - building datasets
  - cleaning up data
  - monitoring data quality
  - building real-time processes
  - scaling to ensure the product can handle large amounts of data.
  - SQL and Python
## Stages of Data Engineering
- The first part of the data engineering flow is gathering business requirements from business personnel or data scientists. Here, engineers ask what kind of data is required and how they plan on using the data. The data engineering project is then planned out based on these requirements.
  - This planning can include how often to collect (batch) data and where to store it.
- The second part of the flow is data discovery and gathering. Here, we look at what kind of data is available and where it should be moved. Within this step, an architecture design is carried out by senior data engineers and solution architects using ETL logic. This design process includes system design, data modeling, etc.
- The third part of the flow involves building the pipeline that will extract the data from its source, such as a database and ingest into the data warehouse.
- The fourth part of the flow involves data transformation, where we filter out data from the database that does not meet the stated requirements.
- The fifth part of the flow involves cleansing and validating the data.
- The sixth part of the flow involves data modeling, where we decide how we want to store the data. Deciding how the data should be stored involves relating dimension and fact tables. In the example of vending machine data, a dimension would be the name of a product being sold, along with the product details and characteristics. A fact would be the quantity of products sold on a daily basis.
- The last part of the flow involves data quality assurance.
## Role of Data Engineer in Data Science and Analytics
- Example of how data engineering relates to data science:
  - There is a retail company that owns 1000s of vending machines spread across various locations. There 10 - 100 items in each vending machine. One of the business requirements for this company is to perform demand forecasting for each item sold in the machines.
  - Demand forecasting involves looking at historical demand data to determine future demand for each product.
  - The vending machine sends data to a transactional database, such as Oracle, each time a product is sold. Once the data is stored in Oracle, it cannot be used for analytics and insights yet. This is because the data stored in the database applies generally to the transactions performed at the vending machine. Not all of the data is applicable to demand forecasting.
  - The data science team uses databricks to perform their analyses. The data engineer comes into the picture here, as they are the ones who are responsible for transporting the required data from the transactional database to databricks.
  - First, the data engineer will extract all of the data from the transactional database. Next, they will transform the transactional data into sales data that can be used for demand forecasting. Finally, they will load the data into databricks. This overall process is call Extract Transform Load (ETL).
## Essential Skills for Success in Senior Data Engineering Roles
- When you hold a senior role in data engineering, your job requires a lot more than just technical expertise. It also requires leadership, innovation, and strategic thinking. Several other skills needed for senior roles include:
  - Strategic Thinking: Aligning technical solutions with business goals to create measurable value.
  - Team Leadership: Managing teams effectively to meet required goals through innovation and collaboration.
  - Stakeholder Management: Presenting technical concepts to non-technical audiences.
  - Delivery Excellence: Ensuring projects are delivered on time and within budget.
  - Technical Expertise: Mastery of modern cloud platforms.
  - Risk Management: Managing risks proactively to avoid project disruptions.
  - Innovation: Developing reuseable accelerators and staying ahead of industry trends.
  - Consumer-Centric Approach: Delivering solutions that exceed client expectations.
  - Project Estimation: Accurately planning timelines and budgets.
  - Continuous Learning: Keeping yourself and your team up to date with emerging tools.

## Transformation and loading checklist

#
#