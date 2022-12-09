# Perf Test Assignment


## Project Structure

- perf_test_assignment.jmx - main JMeter script to execute
- data.csv - test raw data file used in JMeter script
- octo_perf_execution.pdf - test results report generated from www.octoperf.com
- local_report.zip - test results report generated from a cloud vm
- grafana_influxdb_report.pdf - test results report generated from grafana dashboard

## Tools and tech stack:

| Plugin | Version |
| ------ | ------ |
| JMeter | 5.5.0 |
| Java JDK | 11 |

## Installation and run tests

### Pre-requisites

- Install Oracle JDK 11 or Open JDK 11 or [Amazon Correto JDK](https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/downloads-list.html)

- Download the JMeter 5.5.0 from [here](https://jmeter.apache.org/download_jmeter.cgi) and extract the binaries into a folder

- JMeter plugin manager should be installed by following the steps given [here](https://jmeter-plugins.org/wiki/PluginsManager/)

- Install Custom Thread Groups using plugin JMeter plugin manager

### Run tests

1. Get a clone from the [repository](https://github.com/malithlk00/perf-test-assignment) 

2. Copy the perf_test_assignment.jmx and data.csv files into bin folder of JMeter exracted folder

3. Enable and update following properties in user.properties file located inside bin folder
- jmeter.reportgenerator.overall_granularity=1000
- jmeter.reportgenerator.graph.custom_mm_hit.property.set_granularity=${jmeter.reportgenerator.overall_granularity}

4. Open a comman-prompt inside the bin folder and run follownig command
~~~ssh
jmeter -n -t perf_test_assignment.jmx -l results_all.csv -Jthreads=5 -Jrampup=5 -Jduration=60 -Jthroughput=10 -Jurl=api.tmsandbox.co.nz -e -o reports
~~~


- *results_all.csv* : raw log data file for reporting purpose
- *reports* : folder to save JMeter html dashboard results
- *threads* : parameter for vuser count (default to set as 5)
- *rampup* : ramp up time for users (default to set as 5)
- *duration* : steady state time (default to set as 60 seconds)
- *throughput* : requests per second (default to set as 10)
- *url* : base url of the request without (default to set as api.tmsandbox.co.nz)


## Configure Grafana dashboard:

- Install docker and docker compose
- Clone the [repository](https://github.com/testsmith-io/jmeter-influxdb-grafana-docker) and execute docker-compose.yml file
- Once relavant docker containers are created, start both of them
- In the perf_test_assignment.jmx script, enable backend listner and execute the test
- open url localhost:3000 in browser and navigate to JMeter dashboard to view results



# Test Results Report


## 1. Purpose

The purpose of this performance test is to verify following Functional and NonFunctional requirements and analyse the results.

-----

## 2. Scope

## Functional Requirements

1. API: https://api.tmsandbox.co.nz/v1/Categories/{categoryID}/Details.json?catalogue=false 


2. Test Data: (Category ID: 6327, 6328, 6329, 6330, 6331, 6332, 6333, 6334, 6335, 6336)


3. Test Assertion:

        i. Check for the Response Status
        ii. On a successful response, validate the response with below criteria:
                Parameter Check: Category ID
                Text Check: "CanRelist": true

4. Print following values in a csv file: Category ID, Name, Path, Promotion ID, Price

        Please print all Promotion IDs and respective Prices per Category ID


## Non-Functional Requirements

1. Test should support Vusers (Threads) half the count of Category IDs shared in Test Data
2. The test should ramp up at one VUser (Thread) per second
3. Test should achieve 10 API calls in total for the 1-minute Steady State duration
4. 90 percent of the times the API is expected to perform within 500 ms

## 3. Methodology
### 3.1 Script preparation: 
A single script is created using Jmeter to cover the given scenario. Script can be excuted in non-gui mode.


### 3.2 Test Execution
Response time and the throughput of given transaction is measured by executing the script locally (using a VM) and cloud provider (OctoPerf).


### 3.3 Monitoring
Server resource utilization was not monitored as this is a public accessible API and the server statistics such as CPU Usage, Memory Usage, Disks Input/output, Networks Input/output cannot be accessed.


### 3.4 Reporting
After the execution, excutive summary is added in the readme.md file in github repository (https://github.com/malithlk00/perf-test-assignment)

## 4. Test Environment and Configuration

### 4.1 Load generator configuration
1. Azure Cloud VM

        OS : Windows (Windows 10 Pro)
        Type: Standard B2s (2 vcpus, 4 GiB memory)
        Location: Australia East (Zone 1)
        System Type : 64 bit

2. OctoPerf agent 
        Type: aws ec2
        Location: in Australia (Sydney)


### 4.2 Performance Monitoring

During the test execution, following metrics are monitored
- Response Time
- Throughput

### 4.3 Performance test tool
JMeter 5.5.0 was used to conduct the performance testing.

### 4.4 Test Configuration

| Script | Vusers |Throughput (per/sec) | Ramp up Time (sec)| steady state time (sec) |
| ------ | ------ | --------------------| ------------------| ------------------------|
| Perf_Assignment |   5 | 10            | 5                 |     60                 |

 
## 5. Test Execution Summary and Observations

****Below results are taken from the execution done in Azure Cloud VM****

### 5.1 Verifications
------
#### NFR-01 and NFR-02 :

There are 10 category IDs in data.csv file and half of the count (5) is used as vusers. Ramp up time is also given as 5 so that each vuser is ramped up in one second
![image](https://user-images.githubusercontent.com/4869284/206779227-4f2b779c-101b-4879-b72b-1c0351a67c8d.png)

![image](https://user-images.githubusercontent.com/4869284/206779819-5bc68dc7-1331-472e-8ca8-e47aad63dcb1.png)

-------
#### NFR-03 : 

Throughput is approximately 10 hits per second

![image](https://user-images.githubusercontent.com/4869284/206780064-788e70d3-b2f1-44ae-acf5-7887aeaf4b3d.png)

this was configured by using constant throughput timer
![image](https://user-images.githubusercontent.com/4869284/206780604-a25323f5-b54e-4185-9287-83b0449089cc.png)

------
#### NFR-04	:

As shown in below, 90th percentile response time is below 500ms

![image](https://user-images.githubusercontent.com/4869284/206780398-d2350336-1e6e-4f91-8707-76e8903bc771.png)

*** Above results can be found in local_report.zip ***

### 5.2 Observations

1. All the requests were got successful and received 200 status code. But one request failed due to assertion failure -> Text Check: "CanRelist": true

url: https://api.tmsandbox.co.nz/v1/Categories/6331/Details.json?catalogue=false

![image](https://user-images.githubusercontent.com/4869284/206782470-b84f173c-33f5-4e03-8949-86966c6ebc11.png)

2. Error rate is around 10%

3. As shown in above, both 90th and 95th response percentiles are less than 50ms