# Simple File Transfer Protocol
- **dva programy** -> server a client (***ipk-simpleftp-server*** a ***ipk-simpleftp-client***)

- postupne vypisovať dotazy na server a to isté pre client

### Dokumentácia k SFTP
https://datatracker.ietf.org/doc/html/rfc913

## Volání programu

### Server
```shell
./ipk-simpleftp-server {-i rozhraní} {-p port} [-u cesta_soubor] [-f cesta_k_adresari]
```

- **-i eth0** -> rozhranie, na ktorom bude server naslouchat, ***argument je volitelný*** (default value 0.0.0.0)
- **-p 115** -> port na ktorom bude server naslouchat pripoijeným klientom, default value 115, **volitelný**
- **-u /tmp/userpass.txt** -> je to absolútna cesta k súboru s databázov užívateľských účtov ktoré majú povolené pripojenie na server
			- struktura súboru je **username:password** na každom riadku s oddelovačom **:**, **:** sa nebude nachádzať v username alebo password
- **-f /tmp/simpleftp/** -> absolútna cesta k pracovnému adresáru klientu/serveru, kam sa budú nahrávať/sťahovať súbory

### Client
```shell
./ipk-simpleftp-client [-h IP] {-p port} [-f cesta k adresari]
```

- **-h 192.168.1.1** -> IPv4 alebo IPv6 adresa serveru
- **-p 115** -> číslo portu, na ktorom beží server. Nepovinný argument, kde sa v prípade jeho neprítomnosti uvažuje hodnota **115**
- **-f /tmp/simpleftp/** -> absolútna cesta k pracovnému aresáru klienta/serveru, kam sa budú nahrávať/stahovať súbory

- po spustení klienta bude užívatel klientské príkazy protokolu Simple FTP zadávat ručne. Odpovedi od serveru bude klientská aplikace tisknout na výstup

## Doporučenie
- v ktorom kolvek momente **CTRL + C**
- pracovný adresar serveru bude rovnaký ako ten v ktorom sa binarka spušťa, a bude rovnaký v ktorom je db s heslami
- Makefile
****
# SFTP

![[Pasted image 20220323153737.png]]

## THE PROTOCOL

SFTP is used by opening a TCP connection to the remote hosts' SFTP
port (115 decimal).  You then send SFTP commands and wait for
replies.  SFTP commands sent to the remote server are always 4 ASCII
letters (of any case) followed by a space, the argument(s), and a
`<NULL>`.  The argument can sometimes be null in which case the command
is just 4 characters followed by `<NULL>`.  Replies from the server are
always a response character followed immediately by an ASCII message
string terminated by a `<NULL>`.  A reply can also be just a response
character and a `<NULL>`.

```XML
<command> : = <cmd> [<SPACE> <args>] <NULL>

<cmd> : =  USER ! ACCT ! PASS ! TYPE ! LIST ! CDIR ! KILL ! NAME ! DONE ! RETR ! STOR

<response> : = <response-code> [<message>] <NULL>

<response-code> : =  + | - |   | !

<message> can contain <CRLF>
```

## Commands
### USER
**USER** *user-id*

- *user-id* to the remote system (*name*)

**Server reply:**
`!<user-id> logged in` -> no password needed

`+User-id valid` -> need to send password

`-Invalid user-id` -> user with given *user-id* does not exist

### ACCT
**ACCT** *account*

- *account* to remote system (name)

**Server reply:**
`! Account valid` -> logged in

`+Account valid` -> send password, next send password

`-Invalid account` -> bad account, try again

### PASS
**PASS** *password*

- *password* is password to your acount on SFTP server

**Server reply:**

`! Logged in` -> password is ok and u can start transfer

`+Send account` -> password is ok but you have not specified account

`-Wrong password` -> bad password, try again
****
**You cannot specify any of the following commands until you receive a '!' response from the remote system.**

### TYPE
**TYPE** { *A | B | C* }

- stored file from source PC is send to server, transmision is stream of given type
- **A - ASCII**
		- ASCII bytes are taken and transmitted over connection and stored in the file in destination system
- **B - binary**
		- 8.bit bytes are taken from source and sent to server
- **C - continuous**
		- bits are taken from the file in the source system continuously, ignoring word boundaries, and sent over the connection packed into 8-bit bytes

**Server reply:**

`+Using { Ascii | Binary | Continuous } mode`

`-Type not valid`

### LIST
**LIST**  { *F | V* } *directory-path*

- if not directory is provided work in current directory
- **F** -> specifies a standart formated listening
		- each file is on new line
- **V** -> verbose directory listening
		- in one line will be **file name, size, protection, last write date, name of last writer**

**Server reply:**
`+directory-path <MKL>`


### CDIR
**CDIR** *new-directory*

- change current directory on the remote host to the argument passed

**Server reply:**

`!Changed working dir to <new-directory>`

`-Can't connect to directory because: (reason)`

`+directory ok, send account/password`

If the server replies with **'+'** you should then send an **ACCT** or **PASS** command.
The server will wait for **ACCT** or **PASS** commands until it returns a **'-'** or **'!'** response.

**Replies to ACCT could be:**

`!Changed working dir to <new-directory>`

`+account ok, send password`

`-invalid account`

**Replies to PASS could be:**

`!Changed working dir to <new-directory>`

`+password ok, send account`

`-invalid password`

### KILL
**KILL** *file-spec*

- this will delete the file from the remote system

**Server reply:**
`+<file-spec> deleted`

`-Not deleted because (reason)`

### NAME
**NAME** *old-file-spec*

- renames the old-file.spec to new-file-spec on the remote system

**Server reply:**

`+File exists`

`-Can't find <old-file-spec>`

**NAME** command is aborted, don't send **TOBE**.

**If you receive a '+' you then send:**
**TOBE** *new-file-spec*

**The server replies with:**
`+<old-file-spec> renamed to <new-file-spec>`

`-File wasn't renamed because (reason)`

### DONE

- tells server that we are done

**Server reply:**
`+(the message may be charge/accounting info)`

and then both systems close the connection

### RETR
**RETR** *file-spec*

- remote system send specified file

**Server reply:**

**-** -> abort **RETR** command and server will wait for another command

`<number-of-bytes-that-will-be-sent> (as ascii digits)`

`-File doesn't exist`

**After successful reply client can use:**

**SEND**
- OK, wait for file

**STOP**
**Server reply:** `+ok, RETR aborted`

- client and server is ready for next command

### STOR
**STOR** { *NEW | OLD | APP* } *file-spec*

- tells remote system to receive the following file and save under that name

**NEW**

- specifies it should create a new generation of the file and **not delete the existing one.**

**Server reply:**

`+File exists, will create new generation of file`

`+File does not exist, will create new file`

`-File exists, but system doesn't support generations`

**OLD**

- specifies it should write over existing file, if any or else create a new file with the specified name

**Server reply:**
`+Will write over old file`

`+Will create new file`

**(OLD should always return a '+')**

**APP**

- specifies that what you send should be appended to the file on the remote site
- if file does not exist, create a new one

**Server reply:**
`+Will append to file`

`+Will create file`

**(APP should always return a '+')**

Then client send:

**SIZE** *number-of-bytes-in-file (as ASCII digits)*

- *number-of-bytes-in-file* 
		- is the exact number of 8-bit bytes you will be sending.

**Server reply:**

`+ok, waiting for file`
****
You then send the file as exactly the number of bytes specified above.
When you are done the remote system should reply:

`+Saved <file-spec>`
`-Couldn't save because (reason)`
****
`-Not enough room, don't send it`
This aborts the **STOR** sequence, the server is waiting for your next command.

****
## EXAMPLE

An example file transfer.  '**S**' is the **sender**, the user process.  '**R**' is the reply from the remote **server**.  Remember all server replies are terminated with **NULL**.  If the reply is more than one line each line ends with a **CRLF**.

**R**: <font color="yellow">(listening for connection)</font>
**S**: <font color="blue">(opens connection to R)</font>
**R**: <font color="yellow">+MIT-XX SFTP Service</font>
**S**: <font color="blue">USER MKL</font>
**R**: <font color="yellow">+MKL ok, send password</font>
**S**: <font color="blue">PASS foo</font>
**R**: <font color="yellow">! MKL logged in</font>
**S**: <font color="blue">LIST F PS: MKL</font>
**R**: <font color="yellow">+PS: MKL</font>
	    <font color="yellow">           Small.File</font>
<font color="yellow">           Large.File</font>
**S**: <font color="blue">LIST V</font>
**R**: <font color="yellow">+PS: **MKL**</font>
    <font color="yellow">           Small.File  1        69(7)  P775240  2-Aug-84 20:08  MKL</font>
    <font color="yellow">           Large.File  100  255999(8)  P770000  9-Dec-84 06:04  MKL</font>
**S**: <font color="blue">RETR SMALL.FILE</font>
**R**:  <font color="yellow">69</font>
**S**: <font color="blue">SEND</font>
**R**: <font color="yellow">This is a small file, the file is sent without a terminating null.</font>
**S**: <font color="blue">DONE</font>
**R**: <font color="yellow">+MIT-XX closing connection</font>
