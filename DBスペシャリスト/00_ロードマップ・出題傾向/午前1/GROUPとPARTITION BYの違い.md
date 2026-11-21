
SQLにおける `GROUP BY` と `PARTITION BY` の決定的な違いは、「集約して行数が減るか、行数が維持されるか」という点にあります。
それぞれの特徴と違いを分かりやすく整理しました。
### 1. 比較のまとめ

| **項目**      | **GROUP BY (集約)**              | **PARTITION BY (ウィンドウ関数)**                          |
| ----------- | ------------------------------ | --------------------------------------------------- |
| **行数の変化**   | **減る**（グループごとに1行にまとめられる）       | **維持される**（元の行数はそのまま残る）                              |
| **主な目的**    | グループごとの集計（合計、平均、最大など）          | 行ごとの計算（順位付け、累計、グループ内での比較など）                         |
| **一緒に使う機能** | 集合関数（`SUM`, `COUNT`, `AVG` など） | ウィンドウ関数（`ROW_NUMBER`, `RANK`, `SUM() OVER(...)` など） |

### 2. それぞれのイメージと具体例

従業員テーブル（`employees`）を例に考えてみます。
#### ① `GROUP BY`（データをまとめる）

部署（`department`）ごとにグループ化し、部署ごとの「人数」や「給与の合計」を求めたいときに使います。**結果の行数は部署の数（グループの数）に圧縮されます。**

SQL

```
-- 部署ごとの人数と給与合計を取得（行数は部署数になる）
SELECT 
    department,
    COUNT(*) AS employee_count,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

#### ② `PARTITION BY`（計算の単位を区切る）

元の明細データを1行も崩さずに、「各従業員のデータ」に「所属部署の合計給与」や「部署内での給与順位」を横並びで付加したいときに使います。**結果の行数は元の全従業員分のまま変わりません。**

SQL

```
-- 各従業員の情報のほか、所属部署ごとの給与合計と、部署内での給与順位を同時に表示
SELECT 
    name,
    department,
    salary,
    SUM(salary) OVER (PARTITION BY department) AS dept_total_salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_salary_rank
FROM employees;
```

### まとめ

- データを「まとめて要約したい」なら **`GROUP BY`**   
- 詳細な明細行を残したまま、「特定のグループ単位で順位や累計を計算したい」なら **`PARTITION BY`**

