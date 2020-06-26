# Pewlett-Hackard-Analysis

We were tasked to create a relational data base with the employee CSV files; the main objective was to improve data handling. Moreover, the company Pewett Hackward company is about to experience a big wave of retirements; thus, we are anticipating the number of future hires and creating a mentorship program from special individuals who meet retirement eligibility but are willing to work less time and help training the incoming employees.  The initial data provided included six CSV documents with employee, department and manager information; departments, dept_emp, dept_manager, salaries and titles.

The first step was to create a physical ERD (Entity Relational Diagram) to highlight the tables and their relationship to each other. The main primary primary and foreign keys were employee and department number that allowed to build relationships between the different files. 
![ERD](https://github.com/anapereda/Pewlett-Hackard-Analysis/blob/master/EmployeeDB.png)

We used pgAdmin 4 and SQL as the main technological tool to build the data base called ‘PH-Employee DB’. The first step was to upload the data into pgAdmin 4 and find employees eligible for retirement; those who born between 1952 and 1955 and hired between 1985 and 1988 and created the retirement_info CSV.  With this information we could join to other tables to get other information like salaries and titles; we used inner joins to the retirement_info CSV. 
The main challenge was to obtained the most recent data (title) for those employees who changed position throughout their career within the company. The challenge was overcome by partitioning the data and ordering by date; the latest title the data that we took and used. This is the code that we used for the second technical deliverable;   
	
	SELECT emp_no, first_name, last_name, 		bith_date, from_date, to_date, title
	INTO mentorship_elegibility
	FROM
		(SELECT emp_no, first_name, 			last_name, bith_date, from_date, 			to_date, title, ROW_NUMBER() 		OVER
	(PARTITION BY (emp_no)
	ORDER BY to_date DESC) rn
	FROM mentorship) tmp WHERE rn = 1 
	ORDER BY emp_no;

Using SQL queries, we found a total of 33,118 employees eligible to retire distributed in 7 different employee titles; the break out per title is included in the employees_title_by count.csv file; the two largest categories are senior staff and senior engineer with 15,599 and 14,735 respectively. The retirement_title.CSV contains the list of current employees who were born between Jan. 1952 and Dec. 31 1955; it includes employee number, firs name, last name, date of hire, title and salary. The last table is mentorship_elibility.csv; it has 1914 current employees born between 1965-01-01 and 1965-12-31. The next steps I would recommend is to start contacting those who are legible for retirement or the mentorship program to look form the programs; in terms of data the product has to be updated using similar technology. I think it would be useful to archive past and current employees in different tables; so it is easier to access current information. 
