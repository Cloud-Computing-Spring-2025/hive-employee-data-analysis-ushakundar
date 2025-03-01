# HadoopHiveHue
HadoopHiveHue
Hadoop , Hive, Hue setup pseudo distributed environment using docker compose

## Prerequisites
Ensure you have access to GitHub Codespaces open in visual studio code and the repository containing the following files:

docker-compose.yml

hadoop_hive.env

hue-overrides.ini

input_dataset/employees.csv

input_dataset/departments.csv

input_dataset_generator.py

## Steps to Setup Environment

Launch GitHub Codespaces: Once you open the Github repository. Select the code drop down menu and create a new codespace. Then open the codespace in Visual Studio code.

### Run Docker Containers:
```bash
docker-compose up -d
```
### Run the python file after pip install faker as follows:
```bash
pip install faker
```
### run the python dataset generator file to generate the csv files.
```bash
python input_dataset/input_dataset_generator.py
```
### move the generated input files to HDFS resourcemanager by using the commands:
```bash
docker cp employees.csv resourcemanager:/
docker cp departments.csv resourcemanager:/
```

### Create HDFS directories to store the input file from resourcemanager using the commands:

    hdfs dfs -mkdir -p /user/hive/warehouse/employees
    hdfs dfs -mkdir -p /user/hive/warehouse/departments
### Move the input files from resourcemanager to the newly created directories

    hdfs dfs -put input_dataset/employees.csv /user/hive/warehouse/employees/
    hdfs dfs -put input_dataset/departments.csv /user/hive/warehouse/departments/

## Open and login the Hue web terminal from the PORTS tab in visual studio coded or localhost://8888

## Execute the following Hive queries:
#### a.Create the department table
```sql
 Create table command to create the departments table:
     CREATE TABLE departments (
         dept_id INT,
         department_name STRING,
         location STRING
     )
     ROW FORMAT DELIMITED
     FIELDS TERMINATED BY ','
     STORED AS TEXTFILE;
```
#### b. Load departments.csv into the department table:
```sql
LOAD DATA INPATH '/user/hive/warehouse/department/departments.csv' INTO TABLE departments;
```
#### c. Create table command to create the departments table:
```sql
CREATE TABLE temp_employees (
         emp_id INT,
         name STRING,
         age INT,
         job_role STRING,
         salary DOUBLE,
         project STRING,
         join_date STRING,
         department STRING
     )
     ROW FORMAT DELIMITED
     FIELDS TERMINATED BY ','
     STORED AS TEXTFILE;
```
#### d. Load employees.csv into the temp_employees table:
```sql
LOAD DATA INPATH '/user/hive/warehouse/employees/employees.csv' INTO TABLE temp_employees;
```
#### e. create a partition table and load the data into the newly created partitoned table:
```sql
CREATE TABLE employees (
         emp_id INT,
         name STRING,
         age INT,
         job_role STRING,
         salary DOUBLE,
         project STRING,
         join_date STRING
     )
     PARTITIONED BY (department STRING)
     ROW FORMAT DELIMITED
     FIELDS TERMINATED BY ','
     STORED AS PARQUET;
```
#### The command to load data is as follows:
```sql
SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT INTO TABLE employees PARTITION (department)
SELECT emp_id, name, age, job_role, salary, project, join_date, department FROM temp_employees;
```
#### f. Retrieve all employees who joined after 2015.
```sql
SELECT * FROM employees WHERE year(join_date) > 2015;
```
output is in file named: output1.csv

#### g.Find the average salary of employees in each department.
```sql
     SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department;
```
output is in file named: output2.csv

#### h. Identify employees working on the 'Alpha' project.
```sql
SELECT * FROM employees WHERE project = 'Alpha';
```
output is in file named: output3.csv

#### i. Count the number of employees in each job role.
```sql
SELECT job_role, COUNT(*) AS count FROM employees GROUP BY job_role;
```
output is in file named: output4.csv

#### j. Retrieve employees whose salary is above the average salary of their department.
```sql
SELECT e.* FROM employees e JOIN ( SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department) d ON e.department = d. department WHERE e.salary d.avg_salary;
```
output is in file named: output5.csv

#### k. Find the department with the highest number of employees.
```sql
SELECT department, COUNT(*) AS emp_count FROM employees GROUP BY department ORDER BY emp_count DESC LIMIT 1;
```
output is in file named: output6.csv

#### l. Check for employees with null values in any column and exclude them from analysis.
```sql
SELECT * FROM employees WHERE emp_id IS NOT NULL AND name IS NOT NULL AND age IS NOT NULL and job_role IS NOT NULL and salary IS NOT NULL and project is NOT NULL and join_date IS NOT NULL and department is NOT NULL;
```
output is in file named: output7.csv

#### m. Join the employees and departments tables to display employee details along with department locations.
```sql
SELECT e.*, d.location FROM employees e JOIN departments d ON e.department = d.department_name WHERE e.emp_id IN(SELECT emp_id FROM employees WHERE emp_id IS NOT NULL AND name IS NOT NULL AND age IS NOT NULL and job_role IS NOT NULL and salary IS NOT NULL and project is NOT NULL and join_date IS NOT NULL and department is NOT NULL);
```
output is in file named: output8.csv

#### n. Rank employees within each department based on salary.
```sql
SELECT emp_id, name, salary, department, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank FROM employees where emp_id IN(SELECT emp_id FROM employees WHERE emp_id IS NOT NULL AND name IS NOT NULL AND age IS NOT NULL and job_role IS NOT NULL and salary IS NOT NULL and project is NOT NULL and join_date IS NOT NULL and department is NOT NULL);
```
output is in file named: output9.csv

#### o. Find the top 3 highest-paid employees in each department.
```sql
SELECT * FROM ( SELECT emp_id, name, salary, department, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank FROM employees) ranked WHERE rank <= 3 AND emp_id IN(SELECT emp_id FROM employees WHERE emp_id IS NOT NULL AND name IS NOT NULL AND age IS NOT NULL and job_role IS NOT NULL and salary IS NOT NULL and project is NOT NULL and join_date IS NOT NULL and department is NOT NULL);
```
output is in file named: output10.csv
###
After running the queries you can download the outputs as csv files and import them into out repositories on HUE web terminal.

Push the changes into the Github classroom.
