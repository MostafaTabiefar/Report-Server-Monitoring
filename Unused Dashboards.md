```sql
DECLARE @Last_Visit INT = 30

;WITH Last_Exe AS
(
SELECT
	C.Name AS [Report Name],
    MAX(TimeStart) AS Max_Exe
FROM [PBIReportServer].dbo.ExecutionLogStorage EL WITH(NOLOCK)
	LEFT JOIN [PBIReportServer].dbo.Catalog C WITH(NOLOCK) 
		ON (EL.ReportID = C.ItemID)
WHERE 1=1
	AND [Status] = 'rsSuccess'
	AND RequestType = 0
	AND ReportAction = 13

GROUP BY C.Name
HAVING DATEDIFF(DAY, MAX(TimeStart), GETDATE()) > @Last_Visit
), Reports AS
(
SELECT 
      [Path]																	AS [Report Path]
      ,[Name]																	AS [Report Name]
	  ,[ParentID]																AS [FolderID]
      ,U1.UserName																AS [Report Created By]
	  ,U2.UserName																AS [Report Modified By]
      ,[CreationDate]															AS [Report Created Date]
FROM [PBIReportServer].[dbo].[Catalog]	AS C
	JOIN [dbo].[Users]		AS U1	
		ON C.CreatedByID = U1.UserID
	JOIN [dbo].[Users]		AS U2
		ON U2.UserID = C.ModifiedByID
WHERE 1=1
	AND [Type] = 13
), Folders AS
(
SELECT 
	C.ItemID						AS FolderID
	,C.Name							AS [Folder Name]
FROM [PBIReportServer].[dbo].Catalog						AS C
WHERE 1=1
	AND TYPE = 1
	AND [Path] IS NOT NULL
)
SELECT 
	 E.[Report Name]
	,F.[Folder Name]
	,R.[Report Path]
	,R.[Report Created By] AS [Report Created By]
	,CAST(R.[Report Created Date] AS smalldatetime) AS [Report Create Date]
	,R.[Report Modified By]
	,CAST(E.max_exe AS smalldatetime) AS [Last Execution Date]
	,DATEDIFF(DAY, E.Max_Exe, GETDATE())	AS [LastVisit DateDiff]
FROM Last_Exe AS E
		 JOIN Reports AS R ON R.[Report Name] = E.[Report Name]
	LEFT JOIN Folders AS F ON F.FolderID = R.FolderID
