
## Helpfull commands. These commands are part of ssh toolkit

- scp
- sftp
- ftp : This is not secure command, shares user details in plain text.

## scp
- Example
	- scp test.txt nilesh@linuxserver:/tmp/test.txt


## sftp
- Example
	- sftp nilesh@linuxserver
		- connets to the server
	- After connection
		- ls : list remote server files
		- lls : list local machine files
	- Append "l" to all commands to run it on local machine
	- put : will transer local to remote server
	- 
## Windows support
- On windows these tools will come with putty, but its name is different
	- scp = pscp
	- sftp = psftp