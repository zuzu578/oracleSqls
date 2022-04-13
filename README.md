# subquery
select a.* , (select 어쩌구, 저쩌구 from test z where z.boardId = a.boardId) as contents from 
(select * from board) a 

서브쿼리는 select 절에서도 사용가능 하며 , from 절에서도 사용이가능함 
``` sql
select a.*,(select  z.contents from BoardContent z where z.BoardID = a.BoardID) as contents  from 
(SELECT  
	  a.BoardID
    ,[Subject]  
    ,cast(Contents as nvarchar) Contents  
    , CONVERT(char(10), a.CreateDate, 120) as CreateDate  
    ,isnull(WorkNames ,'없음') WorkNames 
	,AdminNames

   FROM [dbo].[BoardContent] a Join (SELECT CValue, CType FROM Common WHERE CExplain = 'BoardType' AND CValue = 1) b On a.BoardType = b.CValue  --and isnull(a.GubunPart, '') != '구분없음'
    Join  
   ( 
   SELECT DISTINCT a.BoardID,  STUFF  
                   ((SELECT  ',' + WorkName AS [text()]  
                     FROM     WorkPartBoardRelations b Join WorkPart c ON b.WorkPartID = c.WorkPartID  and b.WorkPartID = Case When 635 <> 0 Then c.WorkPartID Else b.WorkPartID End 
					
                     WHERE  b.BoardID = a.BoardID FOR XML PATH('')), 1, 1, '') AS WorkNames  
      FROM     BoardContent a
   ) dd ON a.BoardID = dd.BoardID 
   left outer Join
   ( 
   SELECT DISTINCT a.BoardID,  STUFF  
                   ((SELECT  ',' + AdminName AS [text()]  
                     FROM     BoardAdminRelations b Join Admin c ON b.AdminID = c.AdminID  --and b.WorkPartID = Case When @WorkPartID <> 0 Then @WorkPartID Else b.WorkPartID End
                     WHERE  b.BoardID = a.BoardID FOR XML PATH('')), 1, 1, '') AS AdminNames  
      FROM     BoardContent a   
   ) ee ON a.BoardID = ee.BoardID 
   left outer join
   WorkPartBoardRelations tt ON a.BoardID = tt.BoardID  and tt.WorkPartID = Case When 635 <> 0 Then 635 Else tt.WorkPartID End
   Join WorkPart wp ON tt.WorkPartID = wp.WorkPartID
   Join (Select BoardID, LanguageID From BoardLanguageRelation Group by BoardID, LanguageID) ff On a.BoardID = ff.BoardID and ff.LanguageID = 2
   Join Admin cc ON a.AdminID = cc.AdminID
   WHERE a.BoardType = 1  
   And Case When 0 = 0 and 0 <> '' then isnull(AdminNames,'') + ' ' + isnull(a.AdminName,'') + ' ' + isnull([Subject],'') + ' ' + isnull(cast(Contents as nvarchar(MAX)),'')   
    When 0 = 1 then [Subject]  
    When 0 = 2 Then AdminNames    
    When 0 = 3 Then a.AdminName     
    When 0 = 4 Then cast(Contents as nvarchar)   
    When 0 = 5 Then GubunPart   
    else '1' End  
   like Case When 0 <= 5 and 0 <> '' then '%' + 0 + '%' else '1' end  
   And case when 1 = 6 Then isnull(AuthorYY, '2001-01-02') Else a.CreateDate End >= '2000-01-01' and case when 1 = 6 Then isnull(AuthorYY, '2001-01-02') Else a.CreateDate End  < DATEADD(DD, 1, '2022-04-12')  
   And a.Status = 1  And (1 != 2 OR cc.AdminKind <> 3 OR cc.AdminType <> 2)
   Group by   a.BoardID  
  ,[Subject]  
  ,cast(Contents as nvarchar)   
  ,a.CreateDate  
  ,WorkNames 
  ,AdminNames) a 
 


```


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
  
![wwwwwwwwwwwwwwwwwwwwww](https://user-images.githubusercontent.com/69393030/136308966-7a4212c2-fbd0-40a8-8288-117161174b57.PNG)


# 중복 제거할때(PARTITION BY 사용하여 중복제거 )

		SELECT
		*
		FROM(
		SELECT DISTINCT MIN(p1.person_id) OVER(PARTITION BY p1.person_ID) MIN_PERSONID,
		rownum AS num,
		p1.person_id AS personId,
		p1.title_search,
		p1.kornm AS kornm,
		p1.engnm AS engnm,
		p1.kornm_r AS kornmr,
		SUBSTR(p1.BIRTH_DATE,1,4) AS
		brthYr,
		DECODE(p1.life_yn,'D','Y','N') AS deathYn, SUBSTR(p1.death_date,1,4) AS deathYr,
		p1.debutyear AS debutYr,
		f2.FIELD_NM AS fields,
		p1.FILMOGRAPHY AS filmo,
		p1.PERSONIMG AS Image1,
		CONCAT('https://www.kmdb.or.kr/db/per/', p1.person_id) AS kmdbUrl,
		TO_CHAR(SUBSTR(p3.research_note, 2000, 4000)) AS researchs,
		p5.FIELD_NO AS fieldNo
		FROM
		kmdb.PERSON p1
		left outer JOIN kmdb.FIELD_CODE f2 ON p1.PERSONFIELD = f2.FIELD_NM
		LEFT OUTER JOIN kmdb.PERSON_FIELD p5 ON p1.PERSON_ID = p5.PERSON_ID
		left outer JOIN kmdb.PERSON_RESEARCH p3 ON p1.PERSON_ID = p3.PERSON_ID
		left outer JOIN (SELECT distinct M.TITLE_SEARCH , C.PERSON_ID FROM kmdb.CREDIT_MOVIE C, kmdb.MOVIE_SE M
		WHERE C.MOVIE_ID = M.MOVIE_ID AND C.MOVIE_SEQ = M.MOVIE_SEQ
		) p5 ON p5.person_id = p1.PERSON_ID
		WHERE 1 =1
		)
		WHERE num BETWEEN  1 /**P*/ and  10 /**P*/
