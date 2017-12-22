
## Detailed Explanations

### 1. IMPORTANT Instructions to follow **EVERYTIME BEFORE RUNNING** the program

These paths will be used as examples:
 - `/data/timeline`  <-- Directory that contains the 7 python programs.  Results from program are output into this folder
 - `/data/timeline/extract`   <--- Where all the archive twitter data .tar.gz files have been extracted to.  **_Please manually extract all .tar.gz files into 1 directory_** (since the code goes through all subdirectories in the specified path, folder hierarchy does not matter)

##### 1) Run "clean_database.py" -->  `python clean_database.py`
This will completely remove all tables from the database.  This is required so that consecutive runs do not make data overlap in the tables.  **FAILURE TO DO THIS WILL MAKE DATA OVERLAP**.

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
**IMPORTANT**:  lines are **APPENDED** to final_xxx.csv files, so `final_` and `out_` files must be deleted.  
**IF THESE ARE NOT DELETED, THEN FINAL_XXX.CSV FILES WILL CONTAIN REPEATED INFORMATION**.  

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
   - go to screen
   - press "ctrl a"  then type ":quit"

