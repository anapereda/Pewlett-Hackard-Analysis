-- Creating tables for PH-EmployeeDB
CREATE TABLE departments (
     dept_no VARCHAR NOT NULL,
     dept_name VARCHAR(40) NOT NULL,
     PRIMARY KEY (dept_no),
     UNIQUE (dept_name)
);

CREATE TABLE employees(
	emp_no INT NOT NULL,
	bith_date DATE NOT NULL, 
	first_name VARCHAR NOT NULL, 
	last_name VARCHAR NOT NULL,
	gender VARCHAR NOT NULL,
	hire_date DATE NOT NULL,
	PRIMARY KEY (emp_no)
);

DROP TABLE employees CASCADE;

CREATE TABLE dept_manager (
dept_no VARCHAR(4) NOT NULL,
	emp_no INT NOT NULL,
	from_date DATE NOT NULL,
	to_date DATE NOT NULL,
	FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
	PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE salaries (
  emp_no INT NOT NULL,
  salary INT NOT NULL,
  from_date DATE NOT NULL,
  to_date DATE NOT NULL,
  FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
  PRIMARY KEY (emp_no)
);

CREATE TABLE dept_employee (
	emp_no INT NOT NULL, 
	dept_no VARCHAR NOT NULL, 
	from_date DATE NOT NULL,
	to_date DATE NOT NULL,
	FOREIGN KEY (emp_no) REFERENCES employees(emp_no),
	FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
	PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE titles(
	emp_no INT NOT NULL, 
	title VARCHAR (50) NOT NULL,
	from_date DATE NOT NULL,
	to_date DATE,
	FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	PRIMARY KEY (emp_no, title, from_date)
);	

DROP TABLE departments CASCADE;
DROP TABLE dept_employee CASCADE;


SELECT * FROM departments;

COPY departments from '/Users/anitapereda/Public/departments.csv' DELIMITER ',' CSV HEADER;

COPY employees from '/Users/anitapereda/Public/employees.csv' DELIMITER ',' CSV HEADER;

COPY dept_employee from '/Users/anitapereda/Public/dept_emp.csv' DELIMITER ',' CSV HEADER;


COPY dept_manager from '/Users/anitapereda/Public/dept_manager.csv' DELIMITER ',' CSV HEADER;
COPY salaries from '/Users/anitapereda/Public/salaries.csv' DELIMITER ',' CSV HEADER;
COPY titles from '/Users/anitapereda/Public/titles.csv' DELIMITER ',' CSV HEADER;

SELECT * FROM departments;

-- Data analysis 

SELECT first_name, last_name
FROM employees
WHERE bith_date BETWEEN '1952-01-01' AND '1955-12-31';

SELECT first_name, last_name
FROM employees
WHERE bith_date BETWEEN '1952-01-01' AND '1952-12-31';

SELECT first_name, last_name
FROM employees
WHERE bith_date BETWEEN '1953-01-01' AND '1953-12-31';

SELECT first_name, last_name
FROM employees
WHERE bith_date BETWEEN '1954-01-01' AND '1954-12-31';

SELECT first_name, last_name
FROM employees
WHERE bith_date BETWEEN '1955-01-01' AND '1955-12-31';

--Narrow the search for retirement eligibility
--count the number of rows inthe query
SELECT COUNT(first_name)
FROM employees
WHERE (bith_date BETWEEN '1952-01-01' AND '1955-12-31')
AND (hire_date BETWEEN '1985-01-01' AND '1988-12-31');

--save your new search into a table
SELECT emp_no, first_name, last_name
INTO retirement_info
FROM employees
WHERE (bith_date BETWEEN '1952-01-01' AND '1955-12-31')
AND (hire_date BETWEEN '1985-01-01' AND '1988-12-31');

DROP TABLE retirement_info;

SELECT * FROM retirement_info;
 
COPY retirement_info TO '/Users/anitapereda/Public/retirement_info.csv' DELIMITER ',' CSV HEADER;

--Joins

--inner join; joining departments and dept_manager tables
SELECT departments.dept_name,
     dept_manager.emp_no,
     dept_manager.from_date,
     dept_manager.to_date
FROM departments
INNER JOIN dept_manager
ON departments.dept_no = dept_manager.dept_no;
--this is not the best option because we do not have the specifics of the people who are retiring

SELECT retirement_info.emp_no, retirement_info.first_name, retirement_info.last_name, dept_employee.to_date
FROM retirement_info 
LEFT JOIN dept_employee ON retirement_info.emp_no= dept_employee.emp_no;

SELECT ri.emp_no,
	ri.first_name,
	ri.last_name,
	de.to_date
INTO current_emp
FROM retirement_info as ri
LEFT JOIN dept_employee as de 
ON ri.emp_no = de.emp_no
WHERE de.to_date = ('9999-01-01');

SELECT * FROM current_emp;

--Employee count by department number
SELECT COUNT(ce.emp_no), de.dept_no
INTO emp_by_dept
FROM current_emp as ce
LEFT JOIN dept_employee as de
ON ce.emp_no = de.emp_no
GROUP BY de.dept_no
ORDER BY de.dept_no;


COPY current_emp TO '/Users/anitapereda/Public/current_emp.csv' DELIMITER ',' CSV HEADER;

SELECT e.emp_no, e.first_name, e.last_name, e.gender, s.salary
INTO emp_info
FROM employees as e
INNER JOIN salaries as s
ON(e.emp_no= s.emp_no)
INNER JOIN dept_employee as de
ON (e.emp_no = de.emp_no)
WHERE (e.bith_date BETWEEN '1952-01-01' AND '1955-12-31')
     AND (e.hire_date BETWEEN '1985-01-01' AND '1988-12-31')
	 AND (de.to_date ='9999-01-01')

SELECT * FROM emp_info;

-- List of managers per department
SELECT  dm.dept_no,
        d.dept_name,
        dm.emp_no,
        ce.last_name,
        ce.first_name,
        dm.from_date,
        dm.to_date
INTO manager_info
FROM dept_manager AS dm
    INNER JOIN departments AS d
        ON (dm.dept_no = d.dept_no)
    INNER JOIN current_emp AS ce
        ON (dm.emp_no = ce.emp_no);

SELECT * FROM dept_info

SELECT ce.emp_no,
	ce.first_name,
	ce.last_name,
	d.dept_name
INTO dept_info
FROM current_emp as ce
INNER JOIN dept_employee AS de
ON (ce.emp_no = de.emp_no)
INNER JOIN departments AS d
ON (de.dept_no = d.dept_no);

SELECT * FROM departments;

SELECT ri.emp_no, ri.first_name, ri.last_name, de.dept_no
INTO retirement_sales
FROM retirement_info as ri
INNER JOIN dept_employee AS de
ON (ri.emp_no = de.emp_no)
WHERE (de.dept_no = 'd007')

SELECT ri.emp_no, ri.first_name, ri.last_name, de.dept_no
INTO retirement_sales_dev
FROM retirement_info as ri
INNER JOIN dept_employee AS de
ON (ri.emp_no = de.emp_no)
WHERE de.dept_no  IN ('d007', 'd005')

SELECT * FROM retirement_info;

--challenge
--Create the table 
SELECT * FROM retirement_info;

SELECT COUNT(emp_no)
FROM current_emp

SELECT cu.emp_no, cu.first_name, cu.last_name, 
	ti.from_date, ti.title, ti.to_date, s.salary
INTO dup_retiring_title
FROM current_emp as cu
INNER JOIN titles AS ti
ON (cu.emp_no = ti.emp_no)
INNER JOIN salaries as s
ON (cu.emp_no = s.emp_no);

DROP TABLE dup_retiring_title;

SELECT * FROM dup_retiring_title;
--DELIVERABLE 1
-- Partition the data to show only the most recent title per employee
SELECT emp_no, first_name, last_name, from_date, title, salary, to_date
INTO retirement_title
FROM
	(SELECT emp_no, first_name, last_name, 
	 from_date, title, to_date, salary, ROW_NUMBER() OVER
	(PARTITION BY (emp_no)
	ORDER BY to_date DESC) rn
	FROM dup_retiring_title) tmp WHERE rn = 1 
	ORDER BY emp_no;

DROP TABLE retirement_title; 

SELECT * FROM retirement_title;



SELECT COUNT(rt.first_name), rt.title
INTO employees_title_count
FROM retirement_title as rt
GROUP BY rt.title
ORDER BY count;

SELECT * FROM employees_title_count;
DROP TABLE employees_title_count;
--DELIVERABLE 2

SELECT e.emp_no, e.first_name, e.last_name, e.bith_date,
	ti.title, ti.from_date, ti.to_date
INTO mentorship
FROM employees as e
INNER JOIN titles as ti
ON (e.emp_no = ti.emp_no)
WHERE (e.bith_date BETWEEN '1965-01-01' AND '1965-12-31');

DROP TABLE mentorship;

SELECT * FROM mentorship;

--Partition data

SELECT emp_no, first_name, last_name, bith_date, from_date, to_date, title
INTO mentorship_elegibility
FROM
	(SELECT emp_no, first_name, last_name, bith_date, from_date, to_date, title, ROW_NUMBER() OVER
	(PARTITION BY (emp_no)
	ORDER BY to_date DESC) rn
	FROM mentorship) tmp WHERE rn = 1 
	ORDER BY emp_no;

SELECT count(first_name)
FROM mentorship_elegibility;