---
layout: post
title:  "Simulate a datamart to improve dashboard performance - SharePoint, Python and PowerBI"
date:   2021-12-17 13:46:00 +0100
categories: blog
---


<h2>Context</h2>
Last year, I designed a new PowerBI (PBI) report template that could be used to follow up the progress of each of the projects the organization is implementing at a given moment. Every project would count on an Excel file to introduce relevant information about the project indicators and an PBI report that would visualize the data and produce additional quantitative insights. 

<img src="https://user-images.githubusercontent.com/34288246/149135915-052a509d-b88e-425e-a33e-d326aeae9f03.jpg" alt="report_hint" width="200"/> <img src="https://user-images.githubusercontent.com/34288246/149134648-3f8472e9-923b-4d60-a8ad-83cb1fd91924.jpg" alt="report_hint" width="200"/>

The performance of the reports on refresh has been, though, a permanent issue, reaching up to 10 minutes per refresh in the oldest computers of the organization. After full adoption of the tool in all of our country offices, we introduce today a new data pipeline that reduces the refresh time to a range from 8 to 16 seconds. 

<figure>
    <a href="https://user-images.githubusercontent.com/34288246/149135915-052a509d-b88e-425e-a33e-d326aeae9f03.jpg" ><img src="https://user-images.githubusercontent.com/34288246/149135915-052a509d-b88e-425e-a33e-d326aeae9f03.jpg" alt="report_hint" width="200"/>
    <figcaption>Detail of one of the report's sections.</figcaption>
</figure>



<h2>Specific problems</h2>
Performance problems were due, among other reasons, to the file type of the involved data sources (_.xlsx_ files had to be used, since the organization do not count yet on a cloud-based SQL database) and how PBI deals with it. But also to the over-exploitation of the same data sources in repeated queries within the same report, and the high number of items present in the SharePoint sites (ShP) where the data sources were present (which meant that PowerQuery needed a longer time to find the file). 
One additional issue I wanted to overcome was the fact that the set of computers in the organization was fairly old. That meant that the operations performed by PowerQuery in the PBI reports locally in those computers supposed additional refresh time. 

<h2>Proposed solution</h2>
The solution, released today, consists in a Python script that: 
* reduces the workload of the less efficient PBI and the slow, old computers, outsourcing most of the ETL processes that feed the PBI reports (handling more efficiently with _.xlsx_ files, avoiding ShP site size issues, and shortcoming data quality problems in the original data sources),

          [script of one of the python functions]

* outputs one ready-to-use _.csv_ file for each different table present in the report´s model (this prevents multiple calls to the same file during refresh),

          [screenshot of the data model]

* runs every few minutes in a separated computer through the Windows Task Scheduler; this way, every update in the original _.xlsx_ data source is made available to the reports via the _.csv_ files output by the script in no time.

<h2>Conclusion</h2>
Working with this datamart-like infrastructure presents several advantages: 
* it is built in open source software, saving money to the organization in BPI´s Premium and Pro functionalities related to the use of Microsoft´s _dataflows_;
* as a consequence, and unlike PowerQuery or DAX operations performed within the PowerBI report, the resulting _datamart_ can be used as a base for aggregations of the data from the individual projects that will allow for performance analysis of entire Country Offices or even the whole Organization.
* both Python's ETL capacities and performance are significantly superior to PBI's, reducing data refresh to a few seconds; 
* this infrasstructure also provides with the flexibility needed to plan for further scaling of the system;
* the script loops across all the current projects in every country office, checking for data integrity and overcoming those issues every few minutes -what results in more stable reports.
* in general, more efficient reports that refresh quick and reduces the errors introduced by the users in the original data source contribute to further adoption of the tools we have introduced for the monitoring of the organization´s projects.

