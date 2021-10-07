# oracleSqls
프로젝트하면서 썻었던 sql 복습 




	SELECT * FROM
		((SELECT DISTINCT 
			TB.COL_NAME AS title
			,a.COL_NAME AS colName
			,a.COL_ID AS COL_ID 
			,c.COL_DEPTH
			,a.ADJUST_DATE AS adjustDate
			,a.RECOMMEND_YN AS recommendYn
		
			FROM 
			KMDB.KMDB_COLLECTIONS a 
			LEFT OUTER JOIN (SELECT COL_ID, COL_DEPTH, UPPER_COL_ID FROM KMDB.KMDB_COLLECTIONS_UPPER) c ON a.COL_ID = c.COL_ID
			LEFT OUTER JOIN(SELECT COL_ID, COL_NAME FROM KMDB.KMDB_COLLECTIONS) TB ON TB.COL_ID = c.UPPER_COL_ID
			)A
			LEFT OUTER JOIN(SELECT 
			   DISTINCT 
			   D.COL_ID

			   ,COUNT(TA.COL_ID) OVER(PARTITION BY TB.TYPE_CLSS) AS DATA_COUNT
			   FROM KMDB.KMDB_COLLECTIONS_REL D 
			   LEFT OUTER JOIN(SELECT DISTINCT COL_ID, COL_NAME FROM KMDB.KMDB_COLLECTIONS) TA ON TA.COL_ID = D.COL_ID  
			   LEFT OUTER JOIN(SELECT DISTINCT COL_ID, REL_ID, TYPE_CLSS FROM KMDB.KMDB_COLLECTIONS_REL) TB ON TB.REL_ID = TA.COL_ID) F ON  A.COL_ID=F.COL_ID)
		WHERE 
			recommendYn = '1'
		AND 
			COL_DEPTH = 2
		AND 
			ROWNUM BETWEEN 1 AND 4
		ORDER BY adjustDate DESC;
    
    
    => 각 행마다 컬럼 갯수를 세줌 
    
    ![wwwwwwwwwwwwwwwwwwwwww](https://user-images.githubusercontent.com/69393030/136308909-f3b020bc-62d1-474b-9333-7d05bad9b4e7.PNG)
