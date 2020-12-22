## Hive 中的数据划分

### **Databases**



- **Databases**: Namespaces function to avoid naming conflicts for tables, views, partitions, columns, and so on.  Databases can also be used to enforce security for a user or group of users.

- Tables

  : Homogeneous units of data which have the same schema. An example of a table could be page_views table, where each row could comprise of the following columns (schema):

  - `timestamp`—which is of INT type that corresponds to a UNIX timestamp of when the page was viewed.
  - `userid` —which is of BIGINT type that identifies the user who viewed the page.
  - `page_url—`which is of STRING type that captures the location of the page.
  - `referer_url—`which is of STRING that captures the location of the page from where the user arrived at the current page.
  - `IP—`which is of STRING type that captures the IP address from where the page request was made.

- **Partitions**: Each Table can have one or more partition Keys which determines how the data is stored. Partitions—apart from being storage units—also allow the user to efficiently identify the rows that satisfy a specified criteria; for example, a date_partition of type STRING and country_partition of type STRING. Each unique value of the partition keys defines a partition of the Table. For example, all "US" data from "2009-12-23" is a partition of the page_views table. Therefore, if you run analysis on only the "US" data for 2009-12-23, you can run that query only on the relevant partition of the table, thereby speeding up the analysis significantly. Note however, that just because a partition is named 2009-12-23 does not mean that it contains all or only data from that date; partitions are named after dates for convenience; it is the user's job to guarantee the relationship between partition name and data content! Partition columns are virtual columns, they are not part of the data itself but are derived on load.

- **Buckets** (or **Clusters**): Data in each partition may in turn be divided into Buckets based on the value of a hash function of some column of the Table. For example the page_views table may be bucketed by userid, which is one of the columns, other than the partitions columns, of the page_view table. These can be used to efficiently sample the data.



