# SQL改写

例子
``` SQL
SELECT CAST('COLOBJ' || to_char(rownum + 10000000) AS VARCHAR2(32)) fed_priv_id,
       CAST(xx.tab_id AS VARCHAR2(32)) tab_id,
       CAST(yy.TENANT_ID AS VARCHAR2(64)) tenant_id,
       CAST(xx.FIELD_NAME AS VARCHAR2(64)) field_name,
       CAST(xx.sensitive_level AS VARCHAR2(1)) field_sens_lvl,
       CAST(xx.POLICYID AS VARCHAR(16)) mask_rule,
       CAST(NULL AS VARCHAR(16)) mask_type,
       CAST('1' AS VARCHAR(20)) isaccess
FROM
  (SELECT a.MOBJ_Id,
          a.tab_id,
          a.FIELD_NAME,
          a.policyid AS POLICYID,
          sensitive_level
   FROM
     (SELECT MOBJ_ID,
             tab_id,
             FIELD_NAME,
             POLICYID,
             sensitive_level
      FROM
        (SELECT a.MOBJ_ID,
                a.TAB_ID,
                a.FIELD_NAME,
                b.POLICYID,
                b.VERSEQ,
                b.sensitive_level,
                row_number() over(PARTITION BY a.TAB_ID, a.FIELD_NAME
                                  ORDER BY b.VERSEQ DESC nulls LAST) rn
         FROM MDS.DACP_META_TAB_FIELD a,
              MDS.COLUMN_VAL b
         WHERE a.MOBJ_ID = b.COL_XMLID
           AND a.TAB_ID = b.XMLID)
      WHERE rn = 1) a) xx,
       (SELECT DISTINCT tenant_id FROM MDS.DACP_META_TENANT) yy;
```