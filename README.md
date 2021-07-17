**************************************************************** BEDSIDE MONITOR IoT Project ******************************************************************************

1. IoT Core -- In AWS IoT Core, created the following items
    * Things - BSM_G101, BSM_G102
    * Policy - HealthCarePolicy
    * Rule - BedSideMonitorRule to push the data created to the DynamoDB table "BedSideMonitorData".
    * Things Type - BedSideMonitor
    * Things group - Wing_01
    * Security Certificates are created and activated. 
    * Please refer screenshots folder for the same details which I captured from AWS console.

2. Data Aggregation
    * Created a new table (bsm_agg_data) manually in AWS Dynamodb with attributes
        - partition_id (partition key)
        - from_time (sort key): start time of aggregation in each 1 minute interval within the given time-range
        - to_time: attribute to store end time of aggregation in each 1 minute interval within the given time-range
        - deviceid
        - datatype: sensor data type
        - min_value: aggregated minimum value in each 1 minute interval within the given time-range
        - max_value: aggregated maximum value in each 1 minute interval within the given time-range
        - avg_value: aggregated average value in each 1 minute interval within the given time-range
        - count: number of records processed for aggregation in each 1 minute interval within the given time-range
        - CurrentTime: item creation timestamp
    * "aggregate.py" provides functionality to aggregate the data from the three sensors individually, at the minute level.
    * Creates 60 rows per sensor type in the aggregated table, each being the aggregation of all values within a minute for that sensor data type.
    * main.py will be the main file which takes care of Data Aggregation
    
3. Anomalies & Alerts
    * Created rules config file - config/alerts_config.json
    * A parser is written in "rules.py" for these rules. 
    * Created a new table (bsm_alert) manually in AWS Dynamodb with attributes:
        - partition_id (partition key)
        - time_first_breach (sort key)
        - deviceid, datatype, avg_value, rule, anomaly
    * "rules.py" provides the functionality to run the rule check for a specified time range. It checks all rules on all devices individually within that time period and raise alerts as appropriate. 
    * Alerts are printed on the console, and also saved to the "bsm_alert" table in Dynamodb (check screenshots).

Program Structure
*****************

    * ProjectIoTThing1/BedSideMonitor.py (Device 1)
    * ProjectIoTThing2/BedSideMonitor.py (Device 2)
    * config/alerts_config.json  (rules config file)
    * database.py - Utility class for database connection and methods to read & write data
    * date_utility.py - Utility file with date parsing method.
    * model.py - Model classes to access the corresponding tables in Dynamodb. 
        - Model class "BsmDataModel" for "BedSideMonitorData" table.
        - Model class "BsmAggregateDataModel" for "bsm_agg_data" table.
        - Model class "BsmAlertModel" for "bsm_alert" table.
    * aggregate.py - Has Aggregator class to aggregate the min,max, and average values of the given set of entries withing the specified time range.
    * rules.py - This program provides the functionality to analyse the given set of data and detect anomalies as per the rules config. If detected, alerts are printed on the console and also saved to dynamodb as well.
    * main.py - This is the main program for data aggregation. Execution of this program will process and aggregate the given set of data.

Execution Instructions
**********************

    To get the required output User need to excute 2 programs seperately in the order given below.
        {1} main.py - Execute this program to aggregate the data. This will interally call the aggregate.py program.
            * User inputs 
                        - User need to provide the datetime range in "%Y-%m-%d %H:%M:%S.%f" fromat to fetch the device data from dynamodb table.
                        - Also user need to provide the time interval frequency a which the givn data is aggregated. As per the project requirement it is taken as 60 seconds. However, the user can provide any number in SECOND unit.
                        
            * Steps involved in the program when executed.
                        - Fetch data from "BedSideMonitorData" in dynamodb
                        - Aggregate data (calculate min,max,avg)
                        - Insert aggregated data to "bsm_agg_data" in dynamodb

        {2} rules.py - After the data is aggregated, execute this program to detect anomalies and throw alerts.
            * User inputs 
                        - User need to provide the datetime range in "%Y-%m-%d %H:%M:%S.%f" to fetch the aggregated data from dynamodb.
                       
            * Steps involved in the program when executed.
                        - Fetch data from "bsm_agg_data" in dynamodb
                        - Check for anomalies as per the rules in config/alerts_config.json
                        - It triggers an alert if five continuous aggregated (at the minute level) average values are outside the min/max range.
                        - Prints alert to console
                        - Insert alerts data to "bsm_alert" in dynamodb

ScreenShots
***********
    As required, screenshots of Things created, IoT Rules, Execution Screenshots and DynamoDBtable items have be provided in "screenshots" directory inside the project folder.
