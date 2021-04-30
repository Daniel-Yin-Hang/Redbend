# Server Environment Overview
## Supported Browser
This version of the Software Management Server UI is designed for **Chrome only**.
## Deliverables
The delivery package contains the following files:
- **final.tar**
- **3rdParty.tgz**
### **final.tar**
One file in the delivery package is named final.tar and contains the following directories:
- **DBUtils:** Database helper scripts
- **Installation:** Ansible installation files
- **Javadoc:** API documentation
- **Monitoring:** System monitoring scripts
- **Project_Deliveries:** Customer-specific predefined data
- **SWM:** Archive (SWM_ALL.tgz) containing the Software Management Server
- **Tomcat:** Archive (apache-tomcat.tgz) containing Apache Tomcat
- **AuthorizationService:** Archive containing the Authorization service
- **DataCollector:** Archive containing the Data Collector service
    - **DataCollector/Installation:** Directory with installation scripts for the Data Collector
- **AdditionalAttributesService:** Archive that contains the Additional Attributes service, which contains:
    - **Installation**
    - **additional_attributes.tgz**
        - **additionalAttributes.tar**
            - **additionalAttributes.jar**
            - **config**
              - **application.properties**
              - **bootstrap.properties**
              - **logback-access.xml**
              - **logback.xml**
- **OperationHistoryService:** Archive containing the Operation History service  
Extract final.tar to a location of your choice on the control node. (For information about control nodes, see
Ansible.) There is no need to manually extract the archive files contained in final.tar. Ansible extracts the files
and performs the installation.
### **The 3rdParty.tgz File**
One file in the delivery package is named 3rdParty.tgz and contains the following third-party software:
- **Ant**
- **Elasticsearch**  
This software is **only** to be installed in hosted environments, when the license is owned by HARMAN.
- **Installation_Libraries_RH7.2**
- **Java**
- **Oracle**
- **Python**
- **Vault**, which is part of Ignite 2.20  
Ignite is the cloud platform that develops, manages, and operates in-vehicle applications and connected
vehicle services.
- **WBXML package**  
Copy 3rdParty.tgz to a subdirectory (3rd-Party) of the directory in which you extracted final.tar.
If all third-party software is installed in all nodes, use the skip-tags in the Ansible-playbook commands
### **Libraries**
The Software Management Server requires a GNU C compiler (gcc) for building binary module dependencies.
Libraries required for compilation are installed automatically by the Ansible installscript; Ansible must have
access to the Red Hat Yum repository. If the repository is not available, the following libraries must be
present on the system:
- libstdc++
- glibc
- zlib
- python-libs
- ntp
### **Linux**
The Software Management Server is installed on Red Hat Enterprise Linux 7.x running on a 64-bit Intel x86.
Most installation steps are performed by the Software Management Server Linux user (non-root); other steps
are performed by the root user and are marked as such.
### **Oracle**
To run the Software Management Server with an Oracle database, you must install the following Oraclerelated software.
- Oracle 19c Standard Edition 2 Release 19.0.0.0.0 – Production Version 19.3.0.0.0, running on Linux x86-64
- During installation, some database operations use Oracle SQL\*Plus. If SQL\*Plus is not installed, or if the
installed version is not the version in the installation package, install SQL\*Plus.
### **PostgreSQL**
To run the Software Management Server with a PostgreSQL database, you must install the following
PostgreSQL-related software:
- PostgreSQL, release 10.x, running on Linux x86-64
- postgresql-contrib RPM
- During installation, some database operations use the PostgreSQL Client (psql).
For high availability you also require:
- Pacemaker
- Corosync

# Preparing the Server Environment
## Unzipping the final.tar Package
&emsp;&emsp; *tar -xvf final.tar*
## Third-Party Software
The Software Management Solution requires third-party software. HARMAN supplies most third-party
software in the delivery file, 3rdParty.tgz, which must be extracted manually as described in Unzipping the
3rdParty.tgz File. However, Ansible, Red Hat, Oracle, and PostgreSQL are not part of this file; you must install
these separately. The following list specifies the versions that you must use: 
- **Ansible 2.5**
- **Red Hat Enterprise Linux 7.x**
- One of
  - **Oracle database 19c Standard Edition, release 19c Standard Edition 2 Release 19.0.0.0.0 –
Production Version 19.3.0.0.0.** For instructions, see Configuring the Oracle Database.【link】
  - **PostgreSQL Server, release 10.x**

During the installation of the Software Management Server, Ansible unpacks and installs the following thirdparty software and packages, which are included in 3rdParty.tgz:
- **Apache Ant 1.9.2**.Lower versions are not supported
- **Elasticsearch:** elasticsearch-6.8.2.rpm  
This software is **only** to be installed in hosted environments, when the license is owned by HARMAN.
- **Oracle Instant Client 19.6.0.0** (only relevant if using Oracle; not relevant if using PostgreSQL)
- **Python 2.7.8** The Software Management Server does not support versions 2.7.7 and lower and does not
support versions 2.7.9 and higher
- **Vault**, which is part of Ignite
Ignite 2.20 is the cloud platform that develops, manages, and operates in-vehicle applications and
connected vehicle services.
- **WAP Binary XML (WBXML)** package
- **Zulu OpenJDK 11**
Optionally, to ensure high availability for the Oracle database, you can use GoldenGate, additionalsoftware
from Oracle. For more information, see Using GoldenGate for High Availability.【link】
### Unzipping the 3rdParty.tgz File
**3rdParty.tgz** must be manually extracted and placed in a folder for Ansible to unpack.
Create a folder called **3rd-Party** within the extracted **final.tar** folder. Extract the contents of **3rdParty.tgz** into
this folder.  
*tar zxvf 3rdParty.tgz*
## Installing Ansible
Windows is not supported on the control machine
- **To install Ansible:**
  - **On the control node of your system, install pip:**  
*sudo easy_install pip*
  - **After pip is installed, install Ansible:**  
*sudo pip install ansible==2.5*
  - **Install the JMESPath Python library:**  
*sudo pip install jmespath*
  - **If you plan to use password authentication instead of SSH keys in the Ansible inventory.yml file, you must
installsshpass:**    
*sudo yum install sshpass*
# Preparing the Database
## Determining the Size of the Temporay File System(TMPFS)
The database uses the Linux RAM to speed up its operation.At certain times, the linux OS may need to swap the memory out to disk. The Swapping uses the TMPFS. Therefore, the size of TMPFS must be equal to or larger than the total RAM of the system.  
***# cat /etc/fstab***  
&emsp;&emsp;*LABEL=/ / ext3 defaults 1 1*  
&emsp;&emsp;*LABEL=/boot /boot ext3 defaults 1 2*  
&emsp;&emsp;*tmpfs /dev/shm tmpfs **defaults,size=600M** 0 0*  
&emsp;&emsp;*tmpfs /tmp tmpfs **defaults,size=25M** 0 0*  
&emsp;&emsp;*devpts /dev/pts devpts gid=5,mode=620 0 0*  
&emsp;&emsp;*sysfs /sys sysfs defaults 0 0*  
&emsp;&emsp;*proc /proc proc defaults 0 0*  
&emsp;&emsp;*LABEL=SWAP-sda3 swap swap defaults 0 0*  
&emsp;&emsp;*/dev/sda5 swap swap defaults 0 0*  
## Disabling Automatic OS update
Ensure that automatic updates are disabled on the Linux system. Onlu install updates manaually **after** consulting with Customer Support.
## Configuring the Oracle Database
### Oracle Database Credentials
The Software Management Server installation process does not require a SYS to install the Software
Management Server schema, tables, and data. Instead, it uses the SWM_INSTALL user database credentials
and permissions to create schemas, tables, and data.
When installing or upgrading the Software Management Server, use the SWM_INSTALL user. The script to
create the SWM_INSTALL user is provided with the deliverables.  
- **To create a user**  
You must be a DBA with superuser privileges(SYS) to perform this procedure
1. Do one of these  
   - Run *dev_sys_04_create_user.sql*
   - Execute *${swm.dir}/SMA/DB_Utils/ORACLE_PRE_INSTAL*
2. When prompted, set a password for the new database user.  
The SWM_INSTALL user is created with the relevant credentials and permissions.
### Modifying the Oracle Configuration
Several global parameters must be set to ensure correct operation of Oracle when used by the Software Management Server:
- Create the database with at least 16 redo log files; each must be at least 250MB.
- Create the database using the EM Repository and DB console options.
- Create and run the database using archive mode.
- Set NLS_CHARACTERSET to AL32UTF8.
- Set NLS_NCHAR_CHARACTERSET to AL16UTF16.
- Create the database with at least **2** control files placed on different file systems
- Set the Oracle parameters listed in the following table in the relevant scope;after modifying the parameters, restart Oracle and verify the parameter settings

**Table: Oracle Parameter Settings**

| **Parameter** | **Value** | **Scope** |
|  :----  | :----  |:----  |
| _optim_peek_user_binds | false | Both |
|audit_trail|NONE|spfile|
|commit_logging|BATCH|Both|
|commit_wait|NOWAIT|Both|
|control_management_pack_access|DIAGNOSTIC|Both|
|cursor_sharing|FORCE|Both|
|db_block_size|8192|Both|
|db_recovery_file_dest_size|16G|Both|
|db_writer_processes|16|Spfile|
|dml_locks|8000|Spfile|
|job_queue_processes|1000|Both|
|log_buffer|209715200|Both|
|processes|2000|Spfile|
|shared_pool_reserved_size|260M|Both|
|shared_pool_size|3G|Both|
|sga_max_size|17G to 30G|Spfile|
|sga_target|17G to 30G|Spfile|
|filesystemio_options|NONE|Spfile|
|pga_aggregate_target|3G|Spfile|
|db_16k_cache_size|2G|Both|
- The generic command to set a parameter is:  
*alter system set <parameter_name> = <parameter_value> scope = spfile;*
- **Troubleshooting Oracle Connection Interruptions**  
Troubleshoot Oracle connection interruptions by adding the following parameters to:
*$ORACLE_HOME/network/admin/sqlnet.ora*：
  - **SQLNET.EXPIRE_TIME=10: Make sure to test as required.**
  - **SQLNET.INBOUND_CONNECT_TIMEOUT=120**
- **Restarting Oracle**  
  After modifying Oracle settings, restart Oracle to activate the changes.
- **Verifying Parameter Settings**  
After restarting the Oracle database, use an Oracle client application to connect to the database (as SYS) and run:  
*show parameter <parameter_name>;*  
The displayed value should be the value that you have set.
Repeat this step for all the parameters.  
- **Using Oracle Real Application Cluster (RAC)**  
When using the Oracle RAC database, set the configuration parameters on each node in the cluster before
installing the Software Management Server.
After configuring the nodes, run the verification script (checks_before_install.sql) on each node to
verify that the configuration was implemented successfully:
*${swm.dir}/SMA/DB_Utils/PRE_INSTALL_CHECK/checks_before_install.sql*
### **Required Privileges**
During installation of the Software Management Server, two database schemas are created in Oracle. Both
are granted numerous privileges; all are mandatory for the operation of the Software Management Server.  
Before starting the Software Management Server installation, ensure that the database server supports all
required privileges. If it does not, consult with your Oracle DBA.  
The privileges required are:
- authenticateduser
- connect
- create session
- execute_catalog_role
- resource
- scheduler_admin
- select_catalog_role
- create table
- create view
- create procedure
- create job
- select on sys.dba_pending_transactions
- select on sys.pending_trans$
- select on sys.dba_2pc_pending
- execute on sys.dbms_system
- execute on sys.dbms_lock
### **Removing Unnecessary Privileges**
We recommend that you remove all unnecessary privileges. To do this, run the appropriate SQL code from
SQL\*Plus.
## **Installing and Configuring the PostgreSQL Database**
The following procedure describes the steps to install and configure the PostgreSQL server.  
- **To install and configure PostgreSQL:**
1. **Download the repository RPM file from:**
*https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-redhat10-10-2.noarch.rpm*
2.  **Install the repository RPM file:**
   *rpm -Uvh pgdg-redhat10-10-2.noarch.rpm*
3.  **Install Postgres:**  
*yum install postgresql10*  
*yum install postgresql10-server*  
*yum install postgresql10-contrib -y*
4.  **Run:**  
*curl https://install.citusdata.com/community/rpm.sh | sudo bash*  
*yum install -y pg_cron_10*
5.  **Switch to the** postgres **user:**  
*su - postgres*
6.  **Initialize the database:**  
*/usr/pgsql-10/bin/initdb*
7.  **As a root user, start the database:**  
*service postgresql-10 start*
8.  **In postgresql.conf, add:**  
*shared_preload_libraries = 'pg_cron'*
*cron.database_name = 'postgres'*
9.  **As a root user, restart the database:**  
*service postgresql-10 restart*
10. **Create the following extensions:**  
*psql -U postgres -w -c 'CREATE EXTENSION \"pg_cron\";'*  
*psql -U postgres -w -c 'CREATE EXTENSION \"uuid-ossp\";'*  

**Table: PostgreSQL Parameter Settings**  
|**Name**|**Value**|
|  :----  | :----  |
listen_addresses| '*'|
|port |5432|
|max_connections| 2000|
|authentication_timeout| 1min|
|shared_buffers| 16384MB|
|work_mem| 64MB|
|dynamic_shared_memory_type| posix|
|shared_preload_libraries| 'pg_cron'|
|cron.database_name| 'postgres'|
|wal_level| hot_standby|
|synchronous_commit| off|
|max_wal_sender| 3|
|wal_keep_segments| 32|
|hot_standby| on|
|max_standby_archive_delay| -1|
|max_standby_streaming_delay| -1|
|wal_receiver_status_interval| 2s|
|hot_standby_feedback| on|
|log_destination| 'stderr'|
|logging_collector| on|
|log_directory| 'log'|
|log_filename| 'postgresql-%a.log'|
|log_truncate_on_rotation| on|
|log_rotation_age| 1d|
|log_rotation_size| 0|
|log_line_prefix| '%m [%p]'|
|log_timezone| Country name|
|autovacuum| on|
|log_autovacuum_min_duration| -1|
|autovacuum_max_workers| 4|
|autovacuum_naptime| 1min|
|autovacuum_vacuum_threshold| 50|
|autovacuum_analyze_threshold| 50|
|autovacuum_vacuum_scale_factor| 0.2|
|autovacuum_analyze_scale_factor| 0.1|
|autovacuum_freeze_max_age| 200000000|
|autovacuum_multixact_freeze_max_age| 400000000|
|autovacuum_vacuum_cost_delay| 20ms|
|autovacuum_vacuum_cost_limit| -1|
|datestyle| 'iso, mdy'|
|timezone| Country name|
|lc_messages| 'en_US.UTF-8'|
|lc_monetary| 'en_US.UTF-8'|
|lc_numeric| 'en_US.UTF-8'|
|lc_time| 'en_US.UTF-8'|
|default_text_search_config| 'pg_catalog.english'|
|restart_after_crash| off|
|shared_preload_libraries| 'pg_cron'|
|cron.database_name| 'postgres'|

**To install PostgreSQL on a single server for a cluster:**
- **Run** *./standalone.sh*