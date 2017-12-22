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

### 1. Critical Instructions to follow **everytime before running** the program

These paths will be used as examples:
 - `/data/timeline`  <-- Directory that contains the 7 python programs.  Results from program are output into this folder
 - `/data/timeline/extract`   <--- Where all the archive twitter data .tar.gz files have been extracted to.  **_Please manually extract all .tar.gz files into 1 directory_** (since the code goes through all subdirectories in the specified path, folder hierarchy does not matter)

##### 1) Run "clean_database.py" -->  `python clean_database.py`
This will completely remove all tables from the database.  This is required so that consecutive runs do not make data overlap in the tables.  **Failure to do this will make data overlap**.

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
**If these are not deleted, then final_xxx.csv files will contain repeated information**.  

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

