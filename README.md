Functional and Design Specification
PL/SQL Code Coverage Framework

1.	Introduction
The purpose of this document is to outline the design for generating and reporting the PL/SQL code coverage data for the PL/SQL code running in the Oracle database. 
PL/SQL code coverage is a novice idea and there are no commercial or open source tools available in the market which does this.

2.	Design

PL/SQL code coverage framework is built around the in-built profiler tool bundled with RDBMS 10g, known as DBMS_PROFILER. DBMS_PROFILER is in fact a tool to profile the code execution for the sake of performance tuning. This profiler gives out the information regarding the lines of code which are hit and un-hit. We make use of this information to generate the code coverage numbers for the PL/SQL code.

Profiler tables in the repository would contain the information regarding the program unit name, line number and the number of times the particular line got executed.
2.1	PL/SQL code coverage setup

1.	Before running the tests exercising the PL/SQL code, the DBMS_PROFILER should be started for the user session, which connects to the repository to run the PL/SQL code.

2.	Similarly, while logging out of the user session, the DBMS_PROFILER should be flushed and stopped. At this point, the coverage data will be ready in the profiler data tables.

3.	For the JDBC connections executing the PL/SQL code, the DBMS_PROFILER startup is done at the time of giving out the connection from the connection pool and profiler is stopped at the time when the connection is returned to the pool.

4.	The DBMS_PROFILER setup in the JDBC connection pool is protected using system properties to avoid the profiler setup code path going into the production environment.

2.2	Discounting

PL/SQL code coverage framework takes care of the discounting of code patterns that cannot be covered. Discounting can happen at 3 levels.

1.	Common language constructs like BEGIN, END, LOOP, THEN, ELSE etc. which are not executable code lines are discounted. Discounting of these constructs is done at the time of report generation. While computing the total number of lines, these constructs are excluded. The report shows a non-colored line for such discounted code.

2.	Block level discounting can be done in the PLSQL code to exclude the code block from the scope of PL/SQL code coverage. The lines of code should be enclosed in “BEGIN INFEASIBLE” and “END INFEASIBLE” comment lines. While generating the report, the code between these comment lines are excluded and shown as non-colored lines in the report.

3.	For discounting the whole package, the particular package will be registered in an exclusion list which the framework would read while processing the list of packages in the report. The overall summary report will show these packages as discounted.

2.3	Collection of coverage data

1.	At the end of each test suite run, the PL/SQL coverage data would be captured from the plsql_profiler_data and plsql_profiler_units tables. If the PL/SQL engine enhancement is done to dump the data into flat files instead of database tables, then this data will be stored in a raw file format.

2.	At the end of the code coverage farm job, all the raw data files are collected from the testing locations and will be collated into a single file for each package.

3.	The coverage data would contain the information like the number of times the line was hit. This is in fact not required for our purpose since we are interested in knowing only whether the code line is hit atleast once or not. The raw file processing takes care of excluding the irrelevant data and computing the hit/missed lines.

4.	The package level raw data file which consists of the line number and the number of hits fields is then used to generate the code coverage report. The report generation also includes the discounting logic.
2.4	Reporting

1.	Along with generating the package wise report in the HTML format, the data for each package would also be uploaded to the code coverage repository tables. This data will then be used for generating summary reports, drilldown reports and trend charts in the code coverage application running on Oracle Application Express.

2.	The code coverage repository running in an Oracle Application Express application also holds the information regarding the category of PL/SQL packages. When the report is generated, the packages are grouped under relevant categories to show a report which can be drilled down from individual functional areas.

3.	Code line level discounting logic is embedded in the script which does the color coding of the source code report. The source code file is parsed line by line to check the “BEGIN/END INFEASIBLE” comment lines. All the code lines between such comments are ignored while generating the report.

4.	Data is presented in an easily readable format with clear demarcation of covered and uncovered lines of code. Uncovered lines are shown in red and covered lines marked in green. By default, the profiler reports the coverage only for the first line of a SQL statement even if the statement runs into multiple lines. So the report generation tool will mark the remaining lines in yellow color, indicating that those lines are in fact covered. This holds good for any SQL statement, which can run into multiple lines. Some trivial cases are SELECT, INSERT, DELETE and UPDATE statements.
2.5	[Optional for performance] PL/SQL engine enhancement

1.	PL/SQL engine source code is modified to dump the coverage data into flat files instead of repository tables. 
This is an optional performance improvement and is not a mandatory step.

This is an optional step to improve the performance of coverage data dump since I/O to database is more expensive than file system read/write.


3.	Framework workflow






















														Yes												

























4.	User Interface

User interface for the PL/SQL code coverage includes the following:

1.	HTML page where the PL/SQL program unit code would be listed with color coding for covered and uncovered lines.

Screen shots in (1) and (2) depict how the code coverage report UI would look like.

Fig(1) 

<hidden>
Fig(2)
	<hidden>

2.	Pages in EM code coverage application to associate PL/SQL packages with functional area and sub components.


5.	Viability for external use

	This tool can be made available for general use after making some build changes. The setup and the pre-req check scripts need to be bundled to do the necessary setup steps.
