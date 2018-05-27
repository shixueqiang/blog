---
title: DHT网络中计算torrent资源热度的一种方法
id: 123
categories:
  - 未分类
date: 2017-10-08 00:18:44
tags:
---

DHT协议中有四种查询操作：
主要负责通过UDP与外部节点交互，封装4种基本操作的请求以及响应。

ping：检查一个节点是否“存活”

find_node：向一个节点发送查找节点的请求

get_peers：向一个节点发送查找资源的请求

announce_peer：向一个节点发送自己已经开始下载某个资源的通知

具体的协议可参考：[http://www.bittorrent.org/beps/bep_0005.html](http://www.bittorrent.org/beps/bep_0005.html)

那么当收到一个announce_peer请求时，则表示资源被下载，热度可以+1，下面是我实现的mysql更新资源热度的存储过程：
<pre class="html">DELIMITER $$
&nbsp;
USE `torrent`$$
&nbsp;
DROP PROCEDURE IF EXISTS `updatehots_pro`$$
&nbsp;
CREATE DEFINER=`shixq`@`%` PROCEDURE `updatehots_pro`(IN tId VARCHAR(64))
BEGIN
  DECLARE t_error INTEGER DEFAULT 0;
  DECLARE t_id VARCHAR(64) DEFAULT NULL;  
  DECLARE t_hots JSON DEFAULT NULL;
  DECLARE hotsLength INTEGER DEFAULT 0;
  DECLARE mTime DATETIME;
  DECLARE mHot INTEGER DEFAULT 0; 
  DECLARE mNow VARCHAR(20);
  DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET t_error=1;    
  START TRANSACTION;    
  SELECT t.id,t.hots INTO t_id,t_hots FROM t_torrent t WHERE t.id = tId;  
  IF t_id IS NOT NULL THEN
    SET mNow = DATE_FORMAT(NOW(),'%Y-%m-%d'); 
    IF t_hots IS NOT NULL THEN
      SET mTime = STR_TO_DATE(REPLACE(JSON_EXTRACT(t_hots,'$[0].time'),'"',''),'%Y-%m-%d');
      IF STR_TO_DATE(mNow,'%Y-%m-%d') = mTime THEN /* 和第一个时间相同就在第一个基础上+1*/
      SET mHot = JSON_EXTRACT(t_hots,'$[0].hot');
      UPDATE t_torrent t SET t.hot = mHot + 1, t.hots = JSON_REPLACE(t_hots,'$[0].hot',mHot + 1) WHERE t.id = tId;
      ELSEIF STR_TO_DATE(mNow,'%Y-%m-%d') &gt; mTime THEN /* 在头部插入*/
        SET hotsLength = CEIL((LENGTH(t_hots) - LENGTH(REPLACE(t_hots,'},',''))) / 2 + 1);
        IF hotsLength = 14 THEN /* 只保留最近两周热度*/
          SET t_hots = JSON_REMOVE(t_hots,'$[13]');
        END IF;
        IF 'OBJECT' = JSON_TYPE(t_hots) THEN
          SET t_hots = JSON_ARRAY(JSON_OBJECT("time",mNow,"hot",1),t_hots);
        ELSEIF 'ARRAY' = JSON_TYPE(t_hots) THEN
        SET t_hots = JSON_MERGE(JSON_OBJECT("time",mNow,"hot",1),t_hots);
        END IF;
      UPDATE t_torrent t SET t.hot = 1, t.hots = t_hots WHERE t.id = tId;
      END IF;
    ELSE
      UPDATE t_torrent t SET t.hot = 1, t.hots = JSON_OBJECT("time",mNow,"hot",1) WHERE t.id = tId;
    END IF;  
  END IF;
  IF t_error = 1 THEN 
    ROLLBACK;    
  ELSE    
    COMMIT;    
  END IF;           
  SELECT t_error,mNow,mTime;
END$$
&nbsp;
DELIMITER ;</pre>
&nbsp;