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
&nbsp;&nbsp;DECLARE t_error INTEGER DEFAULT 0;
&nbsp;&nbsp;DECLARE t_id VARCHAR(64) DEFAULT NULL;  
&nbsp;&nbsp;DECLARE t_hots JSON DEFAULT NULL;
&nbsp;&nbsp;DECLARE hotsLength INTEGER DEFAULT 0;
&nbsp;&nbsp;DECLARE mTime DATETIME;
&nbsp;&nbsp;DECLARE mHot INTEGER DEFAULT 0; 
&nbsp;&nbsp;DECLARE mNow VARCHAR(20);
&nbsp;&nbsp;DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET t_error=1;    
&nbsp;&nbsp;START TRANSACTION;    
&nbsp;&nbsp;SELECT t.id,t.hots INTO t_id,t_hots FROM t_torrent t WHERE t.id = tId;  
&nbsp;&nbsp;IF t_id IS NOT NULL THEN
&nbsp;&nbsp;&nbsp;&nbsp;SET mNow = DATE_FORMAT(NOW(),'%Y-%m-%d'); 
&nbsp;&nbsp;&nbsp;&nbsp;IF t_hots IS NOT NULL THEN
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET mTime = STR_TO_DATE(REPLACE(JSON_EXTRACT(t_hots,'$[0].time'),'"',''),'%Y-%m-%d');
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IF STR_TO_DATE(mNow,'%Y-%m-%d') = mTime THEN /* 和第一个时间相同就在第一个基础上+1*/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET mHot = JSON_EXTRACT(t_hots,'$[0].hot');
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UPDATE t_torrent t SET t.hot = mHot + 1, t.hots = JSON_REPLACE(t_hots,'$[0].hot',mHot + 1) WHERE t.id = tId;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ELSEIF STR_TO_DATE(mNow,'%Y-%m-%d') &gt; mTime THEN /* 在头部插入*/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET hotsLength = CEIL((LENGTH(t_hots) - LENGTH(REPLACE(t_hots,'},',''))) / 2 + 1);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IF hotsLength = 14 THEN /* 只保留最近两周热度*/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET t_hots = JSON_REMOVE(t_hots,'$[13]');
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;END IF;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IF 'OBJECT' = JSON_TYPE(t_hots) THEN
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;SET t_hots = JSON_ARRAY(JSON_OBJECT("time",mNow,"hot",1),t_hots);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ELSEIF 'ARRAY' = JSON_TYPE(t_hots) THEN
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET t_hots = JSON_MERGE(JSON_OBJECT("time",mNow,"hot",1),t_hots);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;END IF;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UPDATE t_torrent t SET t.hot = 1, t.hots = t_hots WHERE t.id = tId;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;END IF;
&nbsp;&nbsp;&nbsp;&nbsp;ELSE
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UPDATE t_torrent t SET t.hot = 1, t.hots = JSON_OBJECT("time",mNow,"hot",1) WHERE t.id = tId;
&nbsp;&nbsp;&nbsp;&nbsp;END IF;  
&nbsp;&nbsp;END IF;
&nbsp;&nbsp;IF t_error = 1 THEN 
&nbsp;&nbsp;&nbsp;&nbsp;ROLLBACK;    
&nbsp;&nbsp;ELSE    
&nbsp;&nbsp;&nbsp;&nbsp;COMMIT;    
&nbsp;&nbsp;END IF;           
&nbsp;&nbsp;SELECT t_error,mNow,mTime;
END$$
&nbsp;
DELIMITER ;</pre>
&nbsp;