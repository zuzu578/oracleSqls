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
