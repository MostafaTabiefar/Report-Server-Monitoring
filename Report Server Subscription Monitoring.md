# Report Server Subscription Monitoring
This query retrieves key details about report subscriptions, including their execution performance, associated report metadata, and subscription status.

---
```sql
SELECT
	   S.SubscriptionID																					-- Unique identifier for the subscription
	  ,LOWER(CU.UserName)											AS [Created by]                     -- Username of the person who created/modified the subscription
	  ,S.ModifiedDate												AS [Modified Date]                  -- The date and time the subscription was last modified
	  ,C.[Name]														AS [Report Name]                    -- Name of the report associated with the subscription
      ,[EventType]													AS [Event Type]                     -- Type of subscription event
	  ,S.[LastStatus]												AS [Last Status]                    -- Last execution status of the subscription
	  ,CAST([LastRunTime] AS SMALLDATETIME)							AS [LastRunTime_En]					-- Last execution time, converted to SMALLDATETIME for readability
	  ,ISNULL(A.Average_JobDuration_Seconds, 0)						AS [Average Job Duration Seconds]	-- Average duration of subscription jobs in seconds (default to 0 if NULL)
	  ,CAST(C.ContentSize*1.0/1024/1024 AS SMALLMONEY)				AS [Report Size MB]					-- Size of the report content in megabytes (MB)

FROM [dbo].[Subscriptions]			AS S
LEFT JOIN dbo.[catalog]				AS C
	ON C.ItemID = S.Report_OID
LEFT JOIN dbo.Users					AS CU 
	ON CU.UserID = S.OwnerID     

LEFT JOIN (
    SELECT
         A.SubscriptionID                        
        ,AVG(A.JobDuration) AS Average_JobDuration_Seconds  -- Average job duration in seconds
    FROM
    (
        -- Inner subquery: Calculate the duration of each subscription job
        SELECT
             H.SubscriptionID                       
            ,DATEDIFF(SECOND, H.StartTime, H.EndTime) AS JobDuration
        FROM dbo.SubscriptionHistory AS H                  
    ) AS A                                                 
    WHERE 1=1                                              
    GROUP BY A.SubscriptionID                             
) AS A 
ON A.SubscriptionID = S.SubscriptionID                    -- Join the calculated average job duration back to the main query
