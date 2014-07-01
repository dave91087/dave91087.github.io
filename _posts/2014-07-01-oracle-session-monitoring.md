---
layout: post
title: "Oracle Session Monitoring"
description: ""
category: Oracle
tags: [oracle, tech, sessions]
---
{% include JB/setup %}
In the course of our day to day work, we frequently need to monitor the various sessions running on the DB. Below are some queries I find useful.

I generally save them as easy to run scripts in one folder, for example ~/SQL/sess.sql so that I can run @sess to quickly access my sessions script.

## Active Sessions

{% highlight sql %}
set lines 250
column sid format 999
column process format a13
column status format a10
column user format a20
column schema format a20
column "OS User" format a20
column program format a17
column "Optimiser Cost" format 99999999999
column "SQL Text" format a121

SELECT  s.sid,
        s.PROCESS,
        s.status "Status",
        substr(s.username, 0, 20) "User",
        substr(s.schemaname, 0, 20) "Schema",
        s.osuser "OS User",
        substr(s.program, 0, 30) "Program",
        a.OPTIMIZER_COST "Optimiser Cost",
        Substr(a.sql_text,1,120) "SQL Text"
FROM    v$session s,
        v$sqlarea a
WHERE       s.sql_hash_value = a.hash_value (+)
AND         s.sql_address = a.address (+)
AND NOT    (s.SCHEMANAME = 'SYS' AND s.osuser = 'oracle')
AND         status = 'ACTIVE'
ORDER BY  substr(s.program, 0, 30)
        , username
        , schemaname
        , s.osuser;
{% endhighlight %}

## Current Blocked Sessions

{% highlight sql %}
   SELECT    s1.username
          || '@'
          || s1.machine
          || ' ( SID='
          || s1.sid
          || ' )  is blocking '
          || s2.username
          || '@'
          || s2.machine
          || ' ( SID='
          || s2.sid
          || ' ) '
             AS blocking_status
     FROM v$lock l1,
          v$session s1,
          v$lock l2,
          v$session s2
    WHERE     s1.sid = l1.sid
          AND s2.sid = l2.sid
          AND l1.BLOCK = 1
          AND l2.request > 0
          AND l1.id1 = l2.id1
          AND l2.id2 = l2.id2;
{% endhighlight %}

## Transactions
{% highlight sql %}
-- Formatting Columns
set lines 270
column start_time format a20
column sid format 999
column serial# format 999999
column username format a20
column status format a10
column schemaname format a20
column osuser format a17
column process format a13
column program format a17
column module format a20
column logon format a10
column lockwait format a20
column t_status format a10
column machine format a28
column used_ublk format 99999999

--Query
select rpad(t.start_time, 20) as start_time,
       s.sid,
       s.serial#,
       rpad(s.username,20),
       s.status,
       s.schemaname,
       rpad(s.osuser,20),
       s.process,
       substr(s.program, 0, 30) "Program",
       rpad(s.module, 20),
       to_char(s.logon_time,'DD/MON/YY HH24:MI:SS') logon_time,
       t.status as t_status,
       t.used_ublk,
       s.lockwait
from v$transaction t, v$session s
where s.saddr = t.ses_addr
order by s.status desc, t.status, start_time;
{% endhighlight %}
