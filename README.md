# test
```=
# 第七章_子查詢(重要)

#1.由一個具體的應用引入子查詢
#需求: 誰的工資比Abel的高

#方式1:
SELECT last_name,salary
FROM employees
WHERE last_name = 'Abel';

SELECT last_name,salary
FROM employees
WHERE salary > 11000;

#方式2:自連接方式
SELECT e2.last_name,e2.salary
FROM employees e1,employees e2
WHERE e2.`salary` > e1.`salary` #多表的連接條件
AND e1.`last_name` = 'Abel';

#方式3: 子查詢
SELECT last_name,salary
FROM employees
WHERE salary > (
		SELECT salary
		FROM employees
		WHERE last_name = 'Abel'
		);

#2. 稱謂的規範: 外查詢(或主查詢)、內查詢(子查詢)

/*
子查询（内查询）在主查询之前一次执行完成。
子查询的结果被主查询（外查询）使用 。
-注意事项
 -子查询要包含在括号内
 -将子查询放在比较条件的右侧
 -单行操作符对应单行子查询，多行操作符对应多行子查询
*/

/*
3. 子查詢的分類
角度1: 從內查詢返回的結果條目數
單行子查詢 vs 多行子查詢

角度2: 內查詢是否被執行多次
	相關子查詢 vs 不相關子查詢

比如: 相關子查詢的需求:查詢工資大於本部門平均工資的員工信息
     不相關子查詢的需求:查詢工資大於本公司平均工資的員工信息
*/

#子查詢的編寫技巧跟思路: 1.從裡往外寫 2.從外往裡寫

#4.單行子查詢
#4.1 單行操作符: = != > >= < <=
#题目：查询工资大于149号员工工资的员工的信息

SELECT employee_id,last_name,salary
FROM employees
WHERE salary >(
	       SELECT salary
	       FROM employees
               WHERE employee_id = 149
);

#下列子查詢內的內容先寫
SELECT salary
FROM employees
WHERE employee_id = 149;

#题目：返回job_id与141号员工相同，salary比143号员工多的员工姓名，job_id和工资

SELECT last_name,job_id,salary
FROM employees
WHERE job_id = (
		SELECT job_id
		FROM employees
		WHERE employee_id = 141
		)
AND salary > (
	SELECT job_id
	FROM employees
	WHERE employee_id = 143		
	);

#题目：返回公司工资最少的员工的last_name,job_id和salary

SELECT last_name,job_id,salary
FROM employees
WHERE salary = (
	SELECT MIN(salary)
	FROM employees
	);

#题目：查询与141号员工的manager_id和department_id相同的其他员工的employee_id，
#manager_id，department_id

#方式一:
SELECT employee_id,manager_id,department_id
FROM employees
WHERE manager_id = (
		SELECT manager_id
		FROM employees
		WHERE employee_id = 141
)
AND department_id = (
		SELECT department_id
		FROM employees
		WHERE employee_id = 141
)
AND employee_id <>141;

#方式二: 了解
SELECT employee_id,manager_id,department_id
FROM employees
WHERE (manager_id,department_id)= (
				SELECT manager_id,department_id
				FROM employees
				WHERE employee_id = 141

)
AND employee_id <>141;

#HAVING 中的子查询
#题目：查询最低工资大于110号部门最低工资的部门id和其最低工资
SELECT department_id,MIN(salary)
FROM employees
WHERE department_id IS NOT NULL
GROUP BY department_id
HAVING MIN(salary) > (
			SELECT MIN(salary)
			FROM employees
			WHERE department_id = 110
			); 
			
#CASE中的子查询
#题目：显式员工的employee_id,last_name和location。其中，
#若员工department_id与location_id为1800#的department_id相同，
#则location为’Canada’，其余则为’USA’。
SELECT employee_id,last_name,(CASE department_id WHEN (SELECT department_id FROM departments WHERE location_id = 1800) THEN 'Canada'
						ELSE 'USA' END)"location"
FROM employees;

#4.2 子查询中的空值问题
SELECT last_name, job_id
FROM employees
WHERE job_id =
		(SELECT job_id
		FROM employees
		WHERE last_name = 'Haas');
		
#4.3 非法使用子查询
#錯誤情況:Subquery returns more than 1 row
SELECT employee_id, last_name
FROM employees
WHERE salary =
		(SELECT MIN(salary)
		FROM employees
		GROUP BY department_id);

#5.多行子查詢
#5.1 多行子查詢的操作符: IN ANY ALL SOME(等同ANY)

#5.2 舉例:
#IN:
SELECT employee_id, last_name
FROM employees
WHERE salary IN
		(SELECT MIN(salary)
		FROM employees
		GROUP BY department_id);

#ANY: 
#题目：返回其它job_id中比job_id为‘IT_PROG’部门任一工资低的员工的员工号、
#姓名、job_id 以及salary
SELECT employee_id,last_name,job_id,salary
FROM employees
WHERE job_id <> 'IT_PROG'
AND salary < ANY (
		SELECT salary
		FROM employees
		WHERE job_id = 'IT_PROG'
		);

#题目：返回其它job_id中比job_id为‘IT_PROG’部门所有工资低的员工的员工号、
#姓名、job_id 以及salary
SELECT employee_id,last_name,job_id,salary
FROM employees
WHERE job_id <> 'IT_PROG'
AND salary < ALL (
		SELECT salary
		FROM employees
		WHERE job_id = 'IT_PROG'
		);

#题目：查询平均工资最低的部门id
#MySQL中聚合函數是不能嵌套的
#方式一:
SELECT department_id
FROM employees
GROUP BY department_id
HAVING AVG(salary) = (
SELECT MIN(avg_sal)
FROM(
			SELECT AVG(salary) avg_sal
			FROM employees
			GROUP BY department_id
			) t_dept_avg_sal
		);

#方式二:
SELECT department_id
FROM employees
GROUP BY department_id
HAVING AVG(salary) <= ALL (

		SELECT AVG(salary) avg_sal
		FROM employees
		GROUP BY department_id
		);

#5.3 空值問題
SELECT last_name
FROM employees
WHERE employee_id NOT IN (
			SELECT manager_id
			FROM employees
			#where manager_id is not null
			);

#6.相關子查詢
#回顧：查询员工中工资大于公司平均工资的员工的last_name,salary和其department_id
#6.1
SELECT last_name,salary,department_id
FROM employees
WHERE salary >(
		SELECT AVG(salary)
		FROM employees
		);

#题目：查询员工中工资大于本部门平均工资的员工的last_name,salary和其department_id
#方式一: 使用相關子查詢
SELECT last_name,salary,department_id
FROM employees e1
WHERE salary >(
		SELECT AVG(salary)
		FROM employees e2
		WHERE department_id = e1.`department_id`
		);

#方式二:在FROM中聲明子查詢
#from型的子查询：子查询是作为from的一部分，子查询要用()引起来，并且要给这个子查询取别
#名， 把它当成一张“临时的虚拟的表”来使用。
SELECT e.last_name,e.salary,e.department_id
FROM employees e,(
			SELECT department_id,AVG(salary)avg_sal
			FROM employees
			GROUP BY department_id) t_dept_avg_sal
WHERE e.department_id = t_dept_avg_sal.department_id
AND e.salary > t_dept_avg_sal.avg_sal;

#题目：查询员工的id,salary,按照department_name 排序
SELECT employee_id,salary
FROM employees e
ORDER BY (
		SELECT department_name
		FROM departments d
		WHERE e.`department_id` = d.`department_id`
		)ASC;

#結論: 在查詢SELECT中，除了GROUP BY 和 LIMIT之外，其他位置都可以聲明子查詢!
/*
SELECT ...,...,...(存在聚合函數)
FROM ...(LEFT/RIGHT)JOIN...ON 多表的連接條件
(LEFT/RIGHT)JOIN ... ON...
WHERE 多表的連接條件 AND 不包含聚合函數的過濾條件
GROUP BY...,...
HAVING 包含聚合函數的過濾條件
ORDER BY...,...(ASC / DESC)
LIMIT ...,...
*/

#题目：若employees表中employee_id与job_history表中employee_id相同的数目不小于2，
#输出这些相同id的员工的employee_id,last_name和其job_id
SELECT *
FROM job_history;

SELECT employee_id,last_name,job_id
FROM employees e
WHERE 2 <= (
		SELECT COUNT(*)
		FROM job_history j
		WHERE e.`employee_id` = j.`employee_id`
		);

#6.2 EXISTS 與 NOT EXISTS关键字

#题目：查询公司管理者的employee_id，last_name，job_id，department_id信息
#方式一: 自連接
SELECT DISTINCT employee_id,last_name,job_id,department_id
FROM employees emp JOIN employees mgr
ON emp.`manager_id` = mgr.`employee_id`;

#方式二: 子查詢
SELECT employee_id,last_name,job_id,department_id
FROM employees
WHERE employee_id IN (
			SELECT DISTINCT manager_id
			FROM employees
			);

#方式三: 使用EXISTS
SELECT employee_id,last_name,job_id,department_id
FROM employees e1
WHERE EXISTS(
SELECT *
FROM employees e2
WHERE e1.`employee_id` = e2.`employee_id`
);

#題目: 查询公司管理者的employee_id，last_name，job_id，department_id信息

#方式一:
SELECT d.department_id,d.department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL;

#方式二:
SELECT department_id,department_name
FROM departments d
WHERE NOT EXISTS(
		SELECT * 
		FROM employees e
		WHERE d.`department_id` = e.`department_id`
		);

#问题：以上两种方式有好坏之分吗？
#解答：自连接方式好！
```
