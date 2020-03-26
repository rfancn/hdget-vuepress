---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- oraclevm
title: OVM3 Inside FAQ - Database
---

#### 0001. How to use mysql client connect to ovm manager mysql db?

    $ mysql -u root -S /u01/app/oracle/mysql/data/mysqld.sock -p
    Enter password: 
    

!!! Note:
!!! please input db password while installing ovm manager,  
!!! usually it is the same as admin password while login to ovm manager Web UI

#### 0002. How many ovm manager owned specific database exist?
There are two ovm manage databse exist in ovs server side:

* appfw (it seems used by weblogic and base framework it depends on)
* ovs   (it is core db ovm manager used)
* performance_schema ( may be performance related)

#### 0003. What's the size of ovm manager db(mysql)?
__Note: Since ovm 3.3.2, the above logic changes to iterate all the files under mysql data dir "/u01/app/oracle/mysql/data/", similar to "du /u01/app/oracle/mysql/data/"__

    $ mysql -u root -S /u01/app/oracle/mysql/data/mysqld.sock -p
    mysql> SELECT table_schema  "DB Name", 
                  ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) "DB Size in MB" 
           FROM   information_schema.tables 
           GROUP  BY table_schema; 




#### 004. What's the code logic to calculate free available space before backuping ovm manager db?
* DiskUsage.java
    ```java
    import java.nio.file.*;
    import java.nio.file.attribute.*;
    import java.io.IOException;
        
    public class DiskUsage {

        static final long K = 1024;
         
        static void printFileStore(FileStore store) throws IOException {
            long total = store.getTotalSpace() / K;
            long used = (store.getTotalSpace() - store.getUnallocatedSpace()) / K;
            long unallocated = store.getUnallocatedSpace() / K;
            long avail = store.getUsableSpace() / K;

            String s = store.toString();
            if (s.length() &gt; 20) {
                System.out.println(s);
                s = "";
            }
            System.out.format("%-20s %12d %12d %12d %12dn", s, total, used, avail, unallocated);
        }

        public static void main(String[] args) throws IOException {
            System.out.format("%-20s %12s %12s %12s %12sn", "Filesystem", "kbytes", "used", "avail", "unallocated");
            if (args.length == 0) {
                FileSystem fs = FileSystems.getDefault();
                for (FileStore store: fs.getFileStores()) {
                    printFileStore(store);
                }
            } else {
                for (String file: args) {
                    FileStore store = Files.getFileStore(Paths.get(file));
                    printFileStore(store);
                }
            }
        }
    }
    ```

* Compile java souce code

        # /u01/app/oracle/java/bin/javac DiskUsage.java

* Run java program

        # /u01/app/oracle/java/bin/java DiskUsage /u01/app/oracle/mysql/dbbackup/

#### 0005. How ovm manager implementation check if there is enough free space before backup mysql db?

* dbSize: get current mysql db size refer to FAQ0003
* fsSpace: available free space on ovm manager server refer to FAQ0004
* if (dbSize * 2) > fsSpace then it think there is enough free space