# User Acess 
This Query Retrieves Users Acess and their last successful execution.

---

```sql
WITH Max_Exe AS
(
    SELECT
        E.UserName,                                     -- Username who executed the report
        E.ReportID,                                     -- ID of the report
        CAST(MAX(E.TimeStart) AS SMALLDATETIME) AS Last_Execution -- Latest execution time for the report
    FROM [dbo].[ExecutionLogStorage] AS E WITH (NOLOCK) -- Query the execution log storage with NOLOCK to avoid locks
	WHERE 1=1
		AND [Status] = 'rsSuccess'
		AND RequestType = 0
    GROUP BY E.UserName, E.ReportID                    -- Group by UserName and ReportID to find max execution time
),

-- CTE to retrieve report catalog information along with its type
Report_Catalog AS 
(
    SELECT
         C.ItemID                                      -- Unique identifier for the report item
        ,C.PolicyID                                    -- Policy ID associated with the report
        ,C.Name AS [Report Name]                       -- Name of the report
        ,C.Path AS [Report Path]                       -- Path where the report is located
        ,CASE (C.[Type])                               -- Determine the report type based on its type code
            WHEN 1  THEN 'Folder'
            WHEN 2  THEN 'Paginated Report'
            WHEN 3  THEN 'File'
            WHEN 4  THEN 'Linked Report'
            WHEN 5  THEN 'Data Source'
            WHEN 6  THEN 'Model'
            WHEN 7  THEN 'Report Part'
            WHEN 8  THEN 'Shared Dataset'
            WHEN 9  THEN 'Unknown'
            WHEN 10 THEN 'Unknown'
            WHEN 11 THEN 'KPI'
            WHEN 12 THEN 'Mobile Report'
            WHEN 13 THEN 'Power BI Report'
            WHEN 14 THEN 'Excel'
            ELSE NULL 
         END AS [Report Type]                          -- Readable report type description
    FROM dbo.Catalog AS C                              -- Query the Catalog table containing report metadata
)
SELECT
     LOWER(U.UserName) AS [UserName]                   
    ,C.[Report Name]                                   
    ,C.[Report Path]                                   
    ,C.[Report Type]                                   
    ,E.Last_Execution                                  
    ,ISNULL(DATEDIFF(DAY, E.Last_Execution, GETDATE()), -1) AS [Date Diff] 
    ,1 AS [Data Type]                                  -- Static flag to distinguish this query block

FROM dbo.PolicyUserRole AS PU                          
    JOIN dbo.Policies AS P                             
        ON PU.PolicyID = P.PolicyID    
    JOIN dbo.Users AS U                                
        ON PU.UserID = U.UserID 
    JOIN dbo.Roles AS R                                
        ON PU.RoleID = R.RoleID    
    JOIN Report_Catalog AS C                           
        ON PU.PolicyID = C.PolicyID
    LEFT JOIN Max_Exe AS E                             
        ON E.ReportID = C.ItemID 
        AND E.UserName = U.UserName

-- UNION combines the above query with additional report execution details 
-- for those users who has acess an item with role but execute by their own username
UNION 

SELECT 
     LOWER(E.UserName) AS [UserName]                   
    ,C.[Report Name]                                   
    ,C.[Report Path]                                   
    ,C.[Report Type]                                   
    ,E.Last_Execution                                  
    ,ISNULL(DATEDIFF(DAY, E.Last_Execution, GETDATE()), -1) AS [Date Diff] 
    ,2 AS [Data Type]                                  -- Static flag to distinguish this query block

FROM Max_Exe AS E                                      
    JOIN Report_Catalog AS C                          
        ON C.ItemID = E.ReportID                       
