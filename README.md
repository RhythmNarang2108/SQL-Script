
CREATE TABLE Projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100),
    start_date DATE,
    end_date DATE,
    budget DECIMAL(10,2)
);

INSERT INTO Projects VALUES
(1, 'AI Chatbot', '2025-01-01', '2025-06-30', 50000),
(2, 'E-commerce Platform', '2025-02-15', '2025-08-15', 75000),
(3, 'Healthcare App', '2025-03-01', '2025-09-01', 60000),
(4, 'Smart Home System', '2025-04-01', '2025-10-01', 90000),
(5, 'Finance Tracker', '2025-05-01', '2025-11-01', 45000);

CREATE TABLE Tasks (
    task_id INT PRIMARY KEY,
    task_name VARCHAR(100),
    project_id INT,
    assigned_to INT,
    due_date DATE,
    is_completed TINYINT 
);

INSERT INTO Tasks(task_id, task_name, project_id, assigned_to, due_date, is_completed) VALUES
(1, 'Design UI', 1, 101, '2025-04-01',1),
(2, 'Build API', 1, 102, '2025-04-15', 0),
(3, 'Database Setup', 2, 103, '2025-05-01', 1),
(4, 'Testing', 2, 101, '2025-05-15', 0),
(5, 'Deployment', 3, 104, '2025-06-01', 1),
(6, 'Security Audit', 3, 102, '2025-06-15', 0),
(7, 'Voice Integration', 4, 105, '2025-07-01', 0),
(8, 'Budget Planning', 4, 101, '2025-07-15', 1),
(9, 'User Feedback', 5, 103, '2025-08-01', 0),
(10, 'Analytics Setup', 5, 104, '2025-08-15', 1);

CREATE TABLE Teams (
    member_id INT PRIMARY KEY,
    member_name VARCHAR(100),
    role VARCHAR(50)
);

INSERT INTO Teams VALUES
(101, 'Alice', 'Team Lead'),
(102, 'Bob', 'Developer'),
(103, 'Charlie', 'Team Lead'),
(104, 'Diana', 'QA Engineer'),
(105, 'Eve', 'Developer');

CREATE TABLE Model_Training (
    training_id INT PRIMARY KEY,
    project_id INT,
    model_name VARCHAR(100),
    accuracy DECIMAL(5,2),
    training_date DATE
);

INSERT INTO Model_Training VALUES
(1, 1, 'GPT-3', 92.5, '2025-03-01'),
(2, 1, 'BERT', 89.0, '2025-03-15'),
(3, 2, 'ResNet', 85.0, '2025-04-01'),
(4, 3, 'XGBoost', 88.5, '2025-05-01'),
(5, 4, 'YOLOv5', 91.0, '2025-06-01');

CREATE TABLE Data_Sets (
    dataset_id INT PRIMARY KEY,
    project_id INT,
    dataset_name VARCHAR(100),
    size_gb DECIMAL(5,2),
    last_updated DATE
);

INSERT INTO Data_Sets VALUES
(1, 1, 'Chat Logs', 12.5, '2025-08-10'),
(2, 2, 'Product Catalog', 8.0, '2025-08-01'),
(3, 3, 'Medical Records', 15.0, '2025-08-05'),
(4, 4, 'Sensor Data', 20.0, '2025-07-30'),
(5, 5, 'Transaction History', 9.5, '2025-08-02');

WITH TaskSummary AS (
  SELECT project_id,
         COUNT(*) AS total_tasks,
         SUM(CASE WHEN is_completed = 1 THEN 1 ELSE 0 END) AS completed_tasks
  FROM Tasks
  GROUP BY project_id
)
SELECT p.project_name, ts.total_tasks, ts.completed_tasks
FROM Projects p
JOIN TaskSummary ts ON p.project_id = ts.project_id;

SELECT member_id, member_name, task_count
FROM (
  SELECT t.assigned_to AS member_id,
         tm.member_name,
         COUNT(*) AS task_count,
         RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
  FROM Tasks t
  JOIN Teams tm ON t.assigned_to = tm.member_id
  GROUP BY t.assigned_to, tm.member_name
) ranked
WHERE rnk <= 2;

SELECT t.*
FROM Tasks t
WHERE t.due_date < (
  SELECT AVG(due_date)
  FROM Tasks
  WHERE project_id = t.project_id
);

SELECT *
FROM Projects
WHERE budget = (
  SELECT MAX(budget) FROM Projects
);

SELECT p.project_name,
       ROUND(100.0 * SUM(CASE WHEN t.is_completed = TRUE THEN 1 ELSE 0 END) / COUNT(*), 2) AS completion_percentage
FROM Projects p
JOIN Tasks t ON p.project_id = t.project_id
GROUP BY p.project_name;

SELECT t.task_name, t.assigned_to, tm.member_name,
       COUNT(*) OVER (PARTITION BY t.assigned_to) AS task_count
FROM Tasks t
JOIN Teams tm ON t.assigned_to = tm.member_id
ORDER BY t.assigned_to;

SELECT t.*
FROM Tasks t
JOIN Teams tm ON t.assigned_to = tm.member_id
WHERE tm.role = 'Team Lead'
  AND t.is_completed = 0
  AND t.due_date <= DATEADD(DAY, 15, GETDATE());

SELECT *FROM Projects p 
WHERE NOT EXISTS (
  SELECT 1 FROM Tasks t WHERE t.project_id = p.project_id
);

SELECT project_id, model_name, accuracy
FROM (
  SELECT *,
         RANK() OVER (PARTITION BY project_id ORDER BY accuracy DESC) AS rnk
  FROM Model_Training
) ranked
WHERE rnk = 1;

SELECT DISTINCT p.*
FROM Projects p
JOIN Data_Sets ds ON p.project_id = ds.project_id
WHERE ds.size_gb > 10
  AND ds.last_updated >= DATEADD(DAY, -30, GETDATE());
