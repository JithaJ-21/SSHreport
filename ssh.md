## What is SSH?
We can think of SSH as a secret tunnel that lets us talk safely to another computer far away.

It is a secure, encrypted way to remotely access systems using cryptographic keys instead of passwords.

SSH (Secure Shell) is a secure network protocol used to:
*	Log in to a remote system-ssh user@server
*	Run commands on it
*	Transfer files securely- scp localfile.txt user@remote:/path/to/destination/ to copy a file from your computer to the remote server, scp user@remote:/path/to/file.txt ./localfolder/  to copy a file from remote to local, rsync -av file user@server:/path to sync files/folders efficiently, only transferring differences(-a → archive (keep permissions, timestamps, symbolic links, etc.), -v → verbose (see what’s happening)) etc.
*	Forward ports- ssh -L 8080:localhost:80 user@server
*	Run git(github, gitlab..)- git clone git@github.com:user/repo.git
  
…etc.

It replaces insecure methods like Telnet and FTP, which are like sending postcards — anyone can read them whereas SSH is like sending a locked, tamper-proof box — only the person with the key can open it.

Default port: 22

Security: Encryption + authentication
## Why do we need SSH?
#### Without SSH:
*	Passwords travel as plain text 
*	Anyone sniffing the network can steal credentials 
#### With SSH:
*	All data is encrypted
*	Identity of server & client is verified
*	Extremely hard to intercept or tamper with
## How SSH Works?
SSH uses three main ideas:
1.	Encryption – protects data
2.	Authentication – verifies identity
3.	Integrity – ensures data isn’t altered

SSH Connection Flow (Step-by-Step)

1)	***Client → Server Connection***
You run:  ssh user@server_ip
Your computer = SSH client
Remote machine = SSH server

2)	***Server Authentication (Host Verification)***
*	Server sends its public host key
*	Client checks: “Have I seen this server before?” (the public keys of the connected servers are stored in:  ~/.ssh/known_hosts)
If new, the computer asks: Are you sure you want to continue connecting (yes/no)? (the client hasn’t seen the server before, so it asks the client if it’s sure that the server is safe and that it actually wants to connect to that server)
This prevents Man-in-the-Middle (MITM) attacks

3)	***Key Exchange & Encryption Setup***
SSH uses:
*	Asymmetric encryption like RSA/ Ed25519 / ECDSA uses 2 keys- public (which anyone can know) and private (which is secret, only we have it); If data is encrypted with public key, only private key can decrypt it or if encrypted with private key, only public key can decrypt. It’s safer, no need to share the private key. However, it’s slower than symmetric encryption.
*	Symmetric session key (same key is used to lock and unlock the data; Both sender and receiver must have the same key) - faster for data transfer
  
     In SSH, during initial connection, asymmetric encryption is used to:
*	Verify the server
*	Exchange a secure session key
     and symmetric encryption is used to use that session key to quickly encrypt all commands/files.
 	
After this:
*	All communication is encrypted
*	Even SSH passwords are encrypted

4)	***User Authentication***
Two main methods:

A. **Password Authentication**
*	Server asks for password
*	Password is encrypted
*	Less secure

B. **SSH Key Authentication** (BEST PRACTICE)
*	Uses public-private key pair
*	No password sent over network

**FULL END-TO-END EXAMPLE** (Story Form)
Situation:
•	My  laptop → Client
•	My friend’s Linux PC → Server
•	Username on server = jitha
•	Server IP = 192.168.1.10
Note: all ssh commands are typed in the Terminal
________________________________________
1️⃣ Generate SSH keys (On your laptop):
Syntax: ssh-keygen -t ed25519      
* if we don’t mention -t ed25519, the system chooses a default, usually RSA(older) which creates bigger keys, it still works but it’s not the best practice. ed25519 is recommended as it’s modern, fast and very secure; it creates smaller keys
* In case of symmetric keys, bigger keys will be exponentially harder to crack and smaller ones will be easier to brute-force. In case of asymmetric keys, bigger keys will be stronger but slower. However, ed25519 is smaller in bits but extremely strong, because it uses a better math algorithm than old RSA.

Using the syntax, keys are created locally. 

Private key will be stored in ~/.ssh/id_ed25519 and public key key in ~/.ssh/id_ed25519.pub
________________________________________
2️⃣ Copy public key to remote computer :
Syntax: ssh-copy-id <user>@<remote_ip_or_hostname>
Ex: ssh-copy-id jitha@192.168.1.10
Public key goes to server; adds the public key to: ~/.ssh/authorized_keys
________________________________________
3️⃣ Login: 
Syntax: ssh <user>@<remote_ip_or_hostname>
Ex: ssh jitha@192.168.1.10
 How SSH Key Authentication Works
•	Client proves it owns the private key
•	Server checks it against the public key
•	If they match → access granted 
Private key never leaves our system; Login happens WITHOUT password
________________________________________
WHERE everything lives 
Item	Location
Private key	Client:  ~/.ssh/id_ed25519
Public key	Client:  ~/.ssh/id_ed25519.pub
Authorized public keys	Server:  ~/.ssh/authorized_keys
Known servers	Client:  ~/.ssh/known_hosts
All these keys are present in the ssh directory 	
Generate keys locally → copy public key to server → login using private key.
What Are SSH Keys?
SSH keys are cryptographic keys used to authenticate securely. They come in pairs:
Key Type	Purpose
Private Key	Stays with YOU (secret)
Public Key	Shared with the server
How Are SSH Keys Secured?
1. File Permissions
•	Private key: 600
•	.ssh directory: 700
Linux has 3 types of users:
1.	Owner (you)
2.	Group
3.	Others (everyone else)
And 3 permissions:
•	Read (r) = 4
•	Write (w) = 2
•	Execute (x) = 1
The numbers are just addition of these
* chmod is a command used to set who can read, write, or use a file or folder in Linux.
 .ssh directory → 700
chmod 700 ~/.ssh
What 700 means:
Who	Permission	Meaning
Owner	7 (4+2+1)	Can read, write, enter
Group	0	No access
Others	0	No access
👉 Effect:
•	Only YOU can enter .ssh
•	No one else can even see file names
•	Prevents others from browsing or copying keys
________________________________________
🔑 Private key file → 600
chmod 600 ~/.ssh/id_ed25519
What 600 means:
Who	Permission	Meaning
Owner	6 (4+2)	Can read & write
Group	0	No access
Others	0	No access
👉 Effect:
•	Only YOU can read the private key
•	No one else on the system can copy or use it
⚠️ SSH refuses to use a private key if permissions are too open
(this is intentional security)
________________________________________
🔑 2. Passphrase (Optional but Recommended)- it’s like a password to open (decrypt) the private key file.
To add passphrase, use:
ssh-keygen -p
to change passphrase (protect key even more), use:
ssh-keygen -p -f ~/.ssh/id_ed25519

Even if key is stolen → attacker needs passphrase
When you run:
ssh-keygen -p
You are:
•	Adding (or changing) a passphrase
•	This encrypts your private key file itself
________________________________________
🔐 Without passphrase (danger case), the private key file will be in plain form
If attacker gets your private key file:
•	They can use it immediately
•	No extra protection
________________________________________
🔐 With passphrase (safe case)
Private key file is stored as:
Encrypted private key
To use it:
1.	SSH asks for passphrase
2.	Passphrase decrypts private key in memory(RAM)
3.	Key is used →login→ then memory is cleared
👉 If attacker steals the file:
•	They still need the passphrase
•	File alone is useless
________________________________________
🧠 3. SSH Agent (Key Manager)
eval "$(ssh-agent -s)" - Starts the agent and sets environment variables for your shell → ready to use 

ssh-add ~/.ssh/id_ed25519 – to add key
•	Holds decrypted keys in memory
•	You type passphrase once
•	Safer than storing plain keys

Session = active login time (your terminal login)
SSH treats each command/program(ssh, scp, git push etc) as a separate process, even in the same terminal/session
Within the same session, with passphrase+no agent, passphrase is asked while starting each command/program and once we exit the program, the RAM is cleared. On the other hand, What exactly is ssh-agent tied to?
When ssh-agent runs, it creates a private socket like this:
/tmp/ssh-XXXXXX/agent.YYYY
And sets an environment variable:
SSH_AUTH_SOCK
⚠️ This socket:
•	Belongs to your user
•	Has file permissions
•	Is accessible only inside your session
A socket is a private pipe for communication b/w programs
An environment variable is a dynamic, named value that stores configuration data like file paths or API keys, accessible by the OS and running applications to influence their behaviour and settings without code changes. 
Who can use your ssh-agent?
✅ Can use it:
•	YOU
•	Programs you start
•	SSH commands from your terminal
•	Forwarded sessions you explicitly allow
❌ CANNOT use it:
•	Other users on the same machine
•	Someone logged in as another account
•	Someone on the network
•	Someone after you log out
why attacker can’t use your agent
Socket has permissions like:
srw------- jitha jitha
Meaning:
•	•  
•	s = socket
•	rw------- = read & write for owner only, nobody else
•	•  jitha (first one) → owner of the file
•	This is the user who can read/write the socket
•	•  jitha (second one) → group of the file
•	Every user in this group would also have access if permissions allowed, but here rw------- only allows owner
•	Only jitha user can read/write
•	No one else ❌
When you logout:
•	Session ends
•	Environment variables gone
•	ssh-agent process stops
•	Socket disappears
•	RAM cleared
No key left anywhere ❌
Why attacker can’t “come later”
For attacker to use your agent, they must:
•	Be logged in as you
•	OR control your session
At that point:
System already trusts them as YOU
SSH agent does NOT authenticate users.
The operating system does.
If attacker can use your agent:
👉 They are already authenticated as YOU
SSH agent doesn’t weaken security — it assumes OS security.
Because if attacker can:
•	Run commands as you
•	Access your session
•	Access your RAM
Then they can already:
•	Read browser cookies
•	Steal passwords
•	Install malware
•	Do FAR worse than SSH
Why SSH agent is still safe in real world
🔐 Protection layers:
•	Agent socket permissions
•	User isolation
•	Session isolation
•	Optional time limits Example:
ssh-add -t 30m
After 30 minutes:
•	Agent forgets key
•	Even YOU need passphrase again
________________________________________
🏢 4. Hardware Security (Advanced) - Instead of storing your private key on disk or in RAM, the key is stored inside a physical device.
Examples-
•	YubiKey
•	TPM
•	Smart cards
Private key never leaves hardware

## Logout completely and securely
*	Close remote SSH session:
**exit**  OR  **logout**
*	Stop ssh-agent (optional, extra safe):
**ssh-agent -k**

## Common SSH Attacks & Defenses
|Attack|	Defense|
|-----|-----|
|Brute-force login|	Disable passwords|
|MITM|	known_hosts check|
|Key theft|	Passphrase + permissions|
|Weak keys|	Use Ed25519|
	








