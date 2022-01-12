---
layout: post
title:  "Simulate a datamart to improve dashboard performance - SharePoint, Python and PowerBI"
date:   2021-12-17 13:46:00 +0100
categories: blog
---


<h2>Context</h2>
Last year, I designed a new PowerBI (PBI) report template that could be used to follow up the progress of each of the projects the organization is implementing at a given moment. Every project would count on an Excel file to introduce relevant information about the project indicators and an PBI report that would visualize the data and produce additional quantitative insights. 
<p></p>
<a href="https://user-images.githubusercontent.com/34288246/149135915-052a509d-b88e-425e-a33e-d326aeae9f03.jpg"><img src="https://user-images.githubusercontent.com/34288246/149135915-052a509d-b88e-425e-a33e-d326aeae9f03.jpg" alt="report_hint" width="400"> 
<a href="https://user-images.githubusercontent.com/34288246/149134648-3f8472e9-923b-4d60-a8ad-83cb1fd91924.jpg"><img src="https://user-images.githubusercontent.com/34288246/149134648-3f8472e9-923b-4d60-a8ad-83cb1fd91924.jpg" alt="report_hint" width="400">

The performance of the reports on refresh has been, though, a permanent issue, reaching up to 10 minutes per refresh in the oldest computers of the organization. After full adoption of the tool in all of our country offices, we introduce today a new data pipeline that reduces the refresh time to a range from 8 to 16 seconds.

<h2>Specific problems</h2>
<p></p>
Performance problems were due, among other reasons, to the file type of the involved data sources (_.xlsx_ files had to be used, since the organization do not count yet on a cloud-based SQL database) and how PBI deals with it. But also to the over-exploitation of the same data sources in repeated queries within the same report, and the high number of items present in the SharePoint sites (ShP) where the data sources were present (which meant that PowerQuery needed a longer time to find the file). 
<p></p>
One additional issue I wanted to overcome was the fact that the set of computers in the organization was fairly old. That meant that the operations performed by PowerQuery in the PBI reports locally in those computers supposed additional refresh time. 
<p></p>
<h2>Proposed solution</h2>
The solution, released today, consists in a Python script that:

* reduces the workload of the less efficient PBI and the slow, old computers, outsourcing most of the ETL processes that feed the PBI reports (handling more efficiently with _.xlsx_ files, avoiding ShP site size issues, and shortcoming data quality problems in the original data sources),

          #14. Steps to upload the resulting CSV files to SharePoint:
          def upload_file_to_sharepoint(source_folder,file_name):
      
          #a. Reads the file in OneDrive:
          path=source_folder+file_name
          with open(path, 'rb') as content_file:
          file_content = content_file.read()

          #b. Connecting to the desired folder in the tennant:
          target_url="/sites/Group-HPRS/Sdilene%20dokumenty/Data_loads-Do_not_modify/individual_inditracks"
          target_folder = ctx2.web.get_folder_by_server_relative_url(target_url)

          #c. Upload the file to SharePoint
          name = os.path.basename(path)
          target_file = target_folder.upload_file(name, file_content).execute_query()    

          #15. Run program and create individual CSVs for each project:
          print(current_inditracks[['Country', 'Project_code']])
          ctx2=autenticate_in_sharepoint(url_hprs,username,password) #function n.3, for connecting to the central SharePoint site where the final CSVs will be uploaded.

          for index, row in df_shp_sites.iterrows():
              shp_country = row['country']
              url = row['site']
              try:
                  ctx=autenticate_in_sharepoint(url,username,password) #function n.4, for dowloading tables from each Country SharePoint

                  for index, row in current_inditracks.iterrows():
                      project_code= row['Project_code']
                      inditrack_country=row['Country']
                      relative_url=row['IndiTrack_Relative_url']

                      if shp_country ==inditrack_country:

                          try:

                              Outcomes,Activities, Indicators, Milestones = download_inditrack(ctx, project_code, relative_url) #function n.4

                              #14a. Create csv for DIM_outcome_progress: 
                              DIM_outcome_progress =dim_outc_progress(Outcomes, project_code, timestamp) #function n.5

                              #14b. Create csv for outc_expected_time:
                              outc_expected_time = outc_exp_time(DIM_outcome_progress, Indicators) #function n.6

                              #14c. Create csv for outc_reported_time:
                              outc_reported_time = outc_rep_time(DIM_outcome_progress, Indicators) #function n.7

                              #14d. Create csv for outcome_progress_top:
                              outcome_progress_top = outc_time_top(DIM_outcome_progress, outc_reported_time) #function n.8

                              #14e. Create csv for DIM_activities_progress:
                              DIM_activities_progress = dim_act_progress (Activities, project_code, timestamp) #function n.9

                              #14f. Create csv for act_expected_time:
                              act_expected_time = act_exp_time(DIM_activities_progress, Indicators) #function n.10

                              #14g. Create csv for act_reported_time:
                              act_reported_time = act_rep_time(DIM_activities_progress, Indicators) #function n.11

                              #14h. Create csv for activity_progress_top:
                              activity_progress_top = act_time_top(DIM_activities_progress, act_reported_time) #function n.12

                              #14i. Create csv for milestones:
                              milestones= milestones_df(Milestones,project_code, timestamp) #function n.13

                              print ("")
                              print('Iteration for ' + project_code + ' has been sucessful')        

                          except:       
                              print ("")
                              print('Iteration for ' + project_code + ' has been cancelled')


              except:
                  print("The Golem has failed to connect to " + shp_country + "'s SharePoint site")
          

* outputs one ready-to-use _.csv_ file for each different table present in the report´s model (this prevents multiple calls to the same file during refresh),

![Report_Sample_Datamarts_3](https://user-images.githubusercontent.com/34288246/149145893-12aad80b-63da-4511-aae6-9873257b4f49.jpg)


* runs every few minutes in a separated computer through the Windows Task Scheduler; this way, every update in the original _.xlsx_ data source is made available to the reports via the _.csv_ files output by the script in no time.

<h2>Conclusion</h2>
Working with this datamart-like infrastructure presents several advantages: 
* it is built in open source software, saving money to the organization in BPI´s Premium and Pro functionalities related to the use of Microsoft´s _dataflows_;
* as a consequence, and unlike PowerQuery or DAX operations performed within the PowerBI report, the resulting _datamart_ can be used as a base for aggregations of the data from the individual projects that will allow for performance analysis of entire Country Offices or even the whole Organization.
* both Python's ETL capacities and performance are significantly superior to PBI's, reducing data refresh to a few seconds; 
* this infrasstructure also provides with the flexibility needed to plan for further scaling of the system;
* the script loops across all the current projects in every country office, checking for data integrity and overcoming those issues every few minutes -what results in more stable reports.
* in general, more efficient reports that refresh quick and reduces the errors introduced by the users in the original data source contribute to further adoption of the tools we have introduced for the monitoring of the organization´s projects.

