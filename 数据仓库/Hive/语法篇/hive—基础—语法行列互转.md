列转行的操作语句
select chn,sum(nvl(demo['active'],0)) as active,sum(nvl(demo['new'],0)) as new from
(select chn,map(status,value) as demo from (select chn,status,count(idfa) value from ods.ods_channel_detail_d group by chn,status)p) a
group by chn;

