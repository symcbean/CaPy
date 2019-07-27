#CaPy#

CaPy is a simple command line tool for accessing the CyberArk REST API.

This was developed against CyberArk 10.3 and relies on access to some
undocumented interfaces.

It can retrieve passwords (including dual-auth), approve requests, 
retrieve the request queue and the server health. The man page for
the agent is shown below.

It requires Python 2.7 and python-requests

## Installation ##
Copy CaPy (the python program), CaPy.1 and CaPy.5 to suitable locations
and the default ini file to /etc/CaPy/CaPy.ini
e.g. 

    cp CaPy /usr/local/bin
    gzip CaPy.1 && cp CaPy.1.gz /usr/local/man/man1/
    gzip CaPy.5 && cp CaPy.5.gz /usr/local/man/man5/
    gzip CaPyscript.5 && cp CaPyscript.5.gz /usr/local/man/man5
    mkdir /etc/CaPy && cp CaPy.ini /etc/CaPy
   
Read the 2 man pages then edit CaPy.ini to match your environment.
Optionally you can add a password to use for the client as described
in the man page (`man 5 CaPy`) for the ini file.
 
# capy(1) - a simple agent for interacting with CyberArk

Version 0.1, July 2019

```
CaPy  -h

CaPy [-v] [-c ALTCONFIG] -p  SECTIONNAME

CaPy [-v] [-c ALTCONFIG] -t SECTIONNAME

CaPy [-v] [-i CAUSER] [-c ALTCONFIG] -e  SECTIONNAME

CaPy [-v] [-i CAUSER] [-c ALTCONFIG] [-r REASON] [-w MINUTES] [-q] -g  SAFE/NAME   SECTIONNAME

CaPy [-v] [-i CAUSER] [-c ALTCONFIG] -k  SAFE/NAME  SECTIONNAME

CaPy [-v] [-i CAUSER] [-c ALTCONFIG] -l SECTIONNAME
```


# Description

Interact with CyberArk REST services to retrieve target accounts / cluster health information. Optional integration with email and syslog for auditing / support. 
Any outcome other than the requested action will result in a non-zero exit status.

Stored credentials used by the agent are rotated automatically according to the rules specified in the ini file.

It is expected that the command will usually be invoked within a shell script and on a sudo island. 


# Options



* **-c**, **--config** **ALTCONFIG**  
  Read config from file ALTCONFIG rather than default /etc/opt/CaPy.ini   
  This requires AllowUserFiles=True in the default ini file and uses the same section name for both default and custom config file.
  
* **-e**  
  Retrieve the cluster health report (returned in JSON format to stdout)
  
* **-t**  
  Rotate the password for the PvwaUser configured in the ini file and update the local cleartext copy
  
* **-g** **TARGET**  
  Retrieve the password for the account and print to stdout. TARGET can be specified using the following formats:  
...
SAFENAME/USERNAME
SAFENAME/USERNAME@HOSTNAME
SAFENAME/USERNAME@HOSTNAME,DBNAME
USERNAME
USERNAME@HOSTNAME
USERNAME@HOSTNAME,DBNAME
... 
* **-i** **CAUSER**  
  Log in to PVWA as CAUSER rather instead of using details from config file. User will be prompted for password via stdin. This will
  Suppress the automatic password rotation unless -t is explicitly specified.
  
  
* **-k** **TARGET**  
  Check in an exclusive account
  
* **-l**  
  List authorization requests which the log on account can approve. Output is in JSON format.
  
* **-q**  
  Quit (exit normally) after creating a dual authorized request and do not wait for request to be approved.
  
* **-r**, **--reason** **REASON**  
  Create a dual authorized request supplying the given REASON. The REST API will be polled at increasing intervals (initially 10 seconds) for approval. This option must be accompanied by -g SAFE/NAME and -w WAITTIME. In the absence of -q, the script will wait  for the request raised to be approved then retrieve the account. The script will ignore any previously created requests.  
  The timeframe for the request will be derived from the current time and the wait time. An additional 3 minutes is added to the start/end of the window to allow for clocks being out of sync.
  
* **-x**, **--x-script** **SCRIPTFILE**  
  Read a customer set of HTTP requests from SCRIPTFILE and apply. This requires AllowUserFiles=True in the ini file. 
  
* **-v**, **--verbose**  
  Enable verbose output (to stderr)
  
* **-w**, **--waittime** **WAITTIME**  
  Allow up to WAITTIME minutes for dual authorizied requests and/or for the account to be checked in (if checked out)
  
* **SECTIONNAME**  
  Read config (see below) from named section in config file
  

# Examples


* **CaPy** **-g** **P-UNX-SSH/user01** **default**  
  Retrieve password for the account
  **user01**
  from safe
  **P-UNX-SSH**
  using the vault configuration in the section named "default" from the config file.
  The request will fail if the account is not immediately available.
  
* **CaPy** **-w** **2** **-g** **user02** **default**  
  Retrieve password for the account
  **user01**
  from any available safe. Allow up to 2 minutes (approx) for a locked account to be released. The request will fail if more than one record is found for "user02"
  
* **CaPy** **-r** **'Server** **restarted'** **-w** **30** **-g** **P-APP-MYSQL/dbuser02@db.example.com,contentdb** **default**  
  Create a dual-authorization request and retrieve the password for the account
  **dbuser02**
  Associated with the database
  **contentdb**
  and the host
  **db.example.com**
  from the safe named
  **P-APP-MYSQL**
  
* **CaPy** **-k** **dbuser02,contentdb** **default**  
  Check in the exclusive account associated with database contentdb. No stdout output generated. Note that if the account is not checked out or is not exclusive, the script will
  exit with a status of 0
  
* **CaPy** **-t** **-i** **CAADMUSER** **default**  
  Log in to CyberArk as CAADMUSER (CaPy will prompt for password) and change the password for the user defined in the [default] section of the ini file. i.e. Resynchromize the password.
  

# Files

*/etc/opt/CaPy.ini*
: The default configuration file. See
  **CaPy.ini**(5)
  for further details   


*Custom scripts*
: See
  **CaPyscript**(5)
  for details of customer scripts
