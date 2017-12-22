# Documentation
A Database for Twitter Content and Network Analysis

--------------------------------------
1. IMPORTANT Instructions BEFORE running program
2. IMPORTANT Settings BEFORE running program
3. Steps in Running the Program
4. Other Notes/Debug
--------------------------------------

## Summary:
1. BEFORE running or RE-RUNNING the program, make sure to:  
    i) Empty database using `python clean_database.py`  
    ii) Clean up data directory using `python clean_directory.py <path/to/data>`  
    iii) Remove all `final_xxx.csv` and `out_xxx.csv` and `deleted_duplicates.txt` files  
    iv) Recommended to run this program in 'screen' mode.  

2. Essential Postgresql Database settings to have BEFORE running the programs:  
    i) The path to program directory must allow execution by 'others': drwxrwxrwx <- the rightmost x needs to be there.  
    ii) Make sure postgresql database version is at least 9.5+  
    iii) Make sure psql database is pointing to harddrive, and not temp folder  
    iv) Make sure "maintenance_work_mem" is 1 GB.  

3. Main Execution:  
 - To run the main program: Requires `akmaster.py` and `akparse.py`  
      `python akmaster.py <path/to/data>`  
       This will do all the extracting, parsing, sorting, removing duplicates, and creating/filling database tables  
       *akparse.py is run automatically inside akmaster.py  

 - To run the juicy analysis after data in database:  
     `sudo python akpost.py <parameters>`  
     `sudo python akpsql.py <parameters>`  
     *sudo solves problem with matplotlib library on garnet server.  

 - Some helpful programs to use to debug:
    - `python akfind.py <path/to/data>`

 - Overall 7 python files:
    - akmaster.py
    - akparse.py
    - akpsql.py
    - akpost.py
    - clean_database.py
    - clean_directory.py
    - akfind.py

4. Other Notes
    - What Python libraries does this program need?
    - What is ErrorCount.txt for?
    - Error Messages?
    
--------------------------------------

## Detailed In-Depth Explanations

### 1. Instructions to follow **everytime before executing** the main program

The following two paths will be used as examples:
 - `/data/timeline`  <-- Directory that contains the 7 python programs.  Results from program are output into this folder
 - `/data/timeline/extract`   <--- Where all the archive twitter data .tar.gz files have been extracted to.  **_Please manually extract all .tar.gz files into 1 directory_** (since the code goes through all subdirectories in the specified path, folder hierarchy does not matter)

##### 1) Run "clean_database.py" -->  `python clean_database.py`
This will completely remove all tables from the database.  This is required so that consecutive runs do not make data overlap in the tables.  **Note: Failure to do this will make data overlap**.

To test that this ran correctly:  
   - Go onto psql commandline and type `\d+`
   - You should get: `No relations found.`

##### 2) Run "clean_directory.py <path/to/directory/to/data>" --> `python clean_directory.py /data/timeline/extract`
NOTE: the parameter is the path to the data- where all the tar.gz files have been extracted to.  
This path should lead to a huge hierarchy of folders, eventually showing a bunch of .bz2 files.  
This will remove all folders and jsons that were created from the programs: akmaster.py, akparse.py  

To test that this ran correctly:  
   - You should see no folders and jsons of the .bz2 files.  You should only see .bz2 files

##### 3) REMOVE ALL final_xxx.csv and out_xxx.csv files AND `deleted_duplicates.txt` in your program directory
**Important**:  lines are **appended** to final_xxx.csv files, so `final_` and `out_` files must be deleted.  
**Note: If these are not deleted, then final_xxx.csv files will contain repeated information**.  

To test that this ran correctly:
   - Type `ls` on current directory where all the programs are (/data/timeline).  You should only see python programs.

##### 4) ***OPTIONAL***
It is recommend to run this program using "screen" so that it can run in the background.  These programs may take a very long time.
`screen`  
<start program>  
To leave screen: hold three in this order: `ctrl a d`  
To get back to screen:  `screen -r`  
if you have more than one screen:  `screen -r <id>`  
To terminate screen:    
   1) go to screen  
   2) press "ctrl a"  then type ":quit"  

--------------------------------------

### 2. Settings that must be set before executing the main program

##### 1) The path to program directory can be executed by others: drwxrwxrwx <- the rightmost x needs to be there.  
If is not done, then there will be an error when using COPY command:  
`psycopg2.Programming Error: could not open file "/data/timeline/final_xx.csv" for reading: Permission denied`  
- NOTE: If you google this, people will say this is due to running COPY instead of psycopg2's copy_from function.   I have tested implementing copy_from, and it causes random errors (ie: psycopg2.DataError: date/time field value out of range: 0")  

HOW TO FIX:  
	if /data/timeline is my program directory, check with ls -ld command on every folder to the path:  
	-> `ls -ld /`  
	-> `ls -ld /data`  
	-> `ls -ld /data/timeline`  
All 3 of these commands should show "d___ ___ __x"  <-- rightmost x needed  
	If not, use these commands:  
	-> `sudo chmod 777 /`  
	-> `sudo chmod 777 /data`  
	-> `sudo chmod 777 /data/timeline`  

##### 1.5) (When created) All final_xxx.csv files should be rwxrwxrwx because they are used to COPY into the database

##### 2) Postgresql Database Version 9.5 or higher (For faster GIN Index Creation)  
 - To check version on linux commandline:  `psql --version`  
```
kima5@garnet:/data/timeline$ psql --version  
psql (PostgreSQL) 9.5.6  
```

##### 3) Postgresql is pointing to HARD DRIVE instead of tmp space.  
To look at all tmp and harddrive on server: `df -h`  
To check Postgres data: `df /var/lib/postgresql`  
   - Make sure this command shows the harddrive folder  
If not, then the program will force the tmp memory to be full, and program will hang forever.  

##### 4) Postgresql Settings are optimized for Bulk Insertion (all the COPY commands)  
"maintenance_work_mem:  Build time for a GIN index is very sensitive to the maintenance_work_mem setting; it doesn't pay to skimp on work memory during index creation."  
Source: https://www.postgresql.org/docs/9.5/static/gin-tips.html  

 - To check:  Go to psql interactive terminal:  `psql`  
Inside psql terminal: `Show maintenance_work_mem;`  
```
 maintenance_work_mem   
----------------------  
 1GB  
(1 row)  
```
I recommend 1GB.  Default setting is 16MB.  

To change this value:  
i) Look up where "postgresql.conf" file is:  
    `locate postgresql.conf`  
ii) Must be *sudo* user to edit this file:  
     `sudo vi /etc/postgresql/9.5/main/postgresql.conf`  
iii) Change the value: (You can keep the default commented, but make sure new value is uncommented)  
```
#maintenance_work_mem = 16MB            # min 1MB  
maintenance_work_mem = 1GB              # min 1MB  
```

iv) After editing this postgresql.conf file, *must* run these commands:  
    Go to psql interactive terminal: `psql`  
    Run command:  `SELECT pg_reload_conf();`  
source: http://www.heatware.net/databases/postgresql-reload-config-without-restarting/  
- Note: Database does not need to be restarted.  

To check if everything worked:  
    Go to psql interactive terminal: `psql`  
    run command: `Show maintenance_work_mem;`  
    Your result should be your new value.  
