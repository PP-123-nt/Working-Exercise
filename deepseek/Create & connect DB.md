## **Create a new empty DB**

1. Open PowerShell terminal
2. run the following command to locate folder for new db
  ```
cd "C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\"
  ```
3. run the following command to create new db name "regional_fbp.db"
  ```
New-Item -ItemType File -Name regional_fbp.db
  ```

<img width="860" height="302" alt="image" src="https://github.com/user-attachments/assets/43f000ad-d47f-4c19-b766-655520249a60" />

## **Connect empty DB in DBGate**

The error is a standard DuckDB safety check: it is refusing to open the file because the file exists, but it is empty (0 bytes).

DuckDB requires a special file header to recognize a file as a valid database. Because the file was created manually as a blank text file, DuckDB sees it and throws an error to prevent accidental data corruption.
<img width="706" height="274" alt="image" src="https://github.com/user-attachments/assets/e0ed078a-0612-43d0-92dd-ea099b73dcaa" />

**The Fix**

You just need to delete that empty file and let DBGate/DuckDB create the proper file from scratch.

Step 1: Delete the empty file
 - Go to your Deepseek folder and delete regional_fbp.db.

Step 2: Connect in DBGate without creating the file first

 - Go back to the connection settings in DBGate

 - Ensure the Database file path is still:
C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\regional_fbp.db

 - Click Connect (or Test).

Because the file no longer exists, DuckDB will now properly initialize a brand new, valid database file at that exact location.
