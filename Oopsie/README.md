<h1>⭐ Oopsie – Hack The Box Write-up ⭐</h1>
<img src="https://img.shields.io/badge/HTB-Oopsie-success" alt="HTB"/> <img src="https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen" alt="Difficulty"/> <img src="https://img.shields.io/badge/OS-Linux-blue" alt="OS"/> <img src="https://img.shields.io/badge/Category-Web%20Enumeration%20%7C%20Broken%20Access%20Control%20%7C%20Privilege%20Escalation-red" alt="Category"/>

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
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Very Easy</td>
</tr>
<tr>
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Operating System</td>
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
<td style="background-color: #1e1e1e; color: #ffffff; padding: 8px;">Burp Suite, Web Enumeration, Cookies, Broken Access Control, IDOR, File Upload, PHP Reverse Shell, SSH, SUID, PATH Hijacking, Privilege Escalation</td>
</tr>
</tbody>
</table>

<hr />

<h2>Summary</h2>
<strong>Oopsie</strong> is a Linux machine from Hack The Box focused mainly on <strong>web enumeration, information disclosure, Broken Access Control, and privilege escalation</strong>.

During initial enumeration we discovered a web server on port 80. Although the main page did not directly show a login panel, we used <strong>Burp Suite</strong> to inspect and enumerate the site.

Enumeration allowed us to discover the directory <code>/cdn-cgi/login</code>, where we found a login page. After logging in as the guest user, we analyzed the cookies used by the application.

By modifying values related to the user and access level, we discovered a <strong>Broken Access Control</strong> vulnerability that allowed access to functionality reserved for the administrator.

Next, we enumerated user identifiers until we found the <strong>admin user's Access ID</strong>. By modifying our session to use that identifier, we managed to access the file upload page.

We uploaded a PHP reverse shell to the <code>/uploads</code> directory and obtained initial access to the machine as the web server user.

During system enumeration we found a file containing credentials for the user <code>robert</code>. Using these credentials we managed to log in via SSH.

Finally, we enumerated files belonging to the <code>bugtracker</code> group and found the binary <code>/usr/bin/bugtracker</code>, configured with the <strong>SUID</strong> bit.
The binary called the <code>cat</code> program in an insecure way, allowing us to abuse <strong>PATH Hijacking</strong> to obtain a shell with <code>root</code> privileges.

<hr />

<h2>Key Concepts</h2>
<ul>
 	<li>Web Enumeration</li>
 	<li>Intercepting Proxy</li>
 	<li>Burp Suite</li>
 	<li>Directory Enumeration</li>
 	<li>Cookies</li>
 	<li>Broken Access Control</li>
 	<li>IDOR / User ID Enumeration</li>
 	<li>Privilege Manipulation</li>
 	<li>File Upload</li>
 	<li>PHP Reverse Shell</li>
 	<li>Initial Foothold</li>
 	<li>Credential Disclosure</li>
 	<li>SSH</li>
 	<li>SUID</li>
 	<li>PATH Hijacking</li>
 	<li>Privilege Escalation</li>
</ul>

<hr />

<h2>Attack Flow</h2>
<ol>
 	<li>Port and service enumeration</li>
 	<li>Discovery of HTTP on port 80</li>
 	<li>Access to the web application</li>
 	<li>Configuring Burp Suite as a proxy</li>
 	<li>Spidering and application enumeration</li>
 	<li>Discovery of <code>/cdn-cgi/login</code></li>
 	<li>Login as the guest user</li>
 	<li>Analysis of cookies and session values</li>
 	<li>Modifying the privilege level</li>
 	<li>Access to restricted functionality</li>
 	<li>Enumeration of user identifiers</li>
 	<li>Discovery of the administrator's Access ID</li>
 	<li>Access to the file upload page</li>
 	<li>Uploading a PHP reverse shell</li>
 	<li>Locating the <code>/uploads</code> directory</li>
 	<li>Executing the reverse shell</li>
 	<li>Initial access as the web server user</li>
 	<li>System enumeration</li>
 	<li>Discovery of credentials for the user <code>robert</code></li>
 	<li>Access via SSH</li>
 	<li>Enumeration of files belonging to the <code>bugtracker</code> group</li>
 	<li>Discovery of the SUID binary <code>/usr/bin/bugtracker</code></li>
 	<li>Analysis of how the binary works</li>
 	<li>Identifying <code>cat</code> as an insecurely called executable</li>
 	<li>Exploitation via PATH Hijacking</li>
 	<li>Obtaining a shell as <code>root</code></li>
 	<li>Obtaining the final flag</li>
</ol>

<hr />

<h2>Connectivity Check</h2>
We start by checking that the target machine is reachable:
<pre>ping 10.129.x.x</pre>

<hr />

<h2>Enumeration</h2>
<h3>Port Scanning</h3>
We use Nmap to identify open ports, available services, and their versions:
<pre>nmap -sCV -q 10.129.x.x</pre>
[caption id="attachment_3031" align="alignnone" width="772"]<img class="size-full wp-image-3031" src="https://diariohacking.com/wp-content/uploads/2026/08/1_port_scanning_with_nmap.png" alt="Port scanning with nmap" width="772" height="316" /> Port scanning with nmap[/caption]

The scan showed two main services:
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
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">22</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">SSH</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">OpenSSH 7.6p1</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">80</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">HTTP</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">Apache 2.4.29</td>
</tr>
</tbody>
</table>
Port 80 indicates that the machine is running an HTTP web server, so we continue enumeration using the browser.

<hr />

<h2>Web Enumeration</h2>
We access the application using the machine's IP address:
<pre>http://10.129.x.x</pre>
[caption id="attachment_3032" align="alignnone" width="766"]<img class="size-full wp-image-3032" src="https://diariohacking.com/wp-content/uploads/2026/08/2_accedemos_al_servicio_web.png" alt="Accessing the web application" width="766" height="424" /> Accessing the web application[/caption]

The main page displayed information about the company's services, including a message
indicating that a login was required to access certain functionality.

However, we didn't initially find a login panel by browsing manually through the page.

[caption id="attachment_3033" align="alignnone" width="787"]<img class="size-full wp-image-3033" src="https://diariohacking.com/wp-content/uploads/2026/08/3_debemos_logearnos.png" alt="Indication that we need to log in to access certain services" width="787" height="327" /> Indication that we need to log in to access certain services[/caption]

The HTB hint tells us we need to do <strong>Spidering</strong> using <strong>Burp Suite</strong>.

<hr />

<h2>Traffic Interception with Burp Suite</h2>
Burp Suite works as an <strong>intercepting proxy</strong> between our browser and
the web server.

The communication can be represented as follows:
<pre>Firefox
   │
   │ HTTP Request
   ▼
Burp Suite
   │
   │ Inspected / Modified Request
   ▼
Web Server
</pre>
We configure Firefox to use Burp's local proxy:
<pre>Manual Proxy</pre>
<pre>HTTP Proxy: 127.0.0.1
Port: 8080</pre>
[caption id="attachment_3035" align="alignnone" width="736"]<img class="size-full wp-image-3035" src="https://diariohacking.com/wp-content/uploads/2026/08/4_configuracion_mozilla.png" alt="Firefox configuration to intercept requests with Burp Suite" width="736" height="625" /> Firefox configuration to intercept requests with Burp Suite[/caption]
<pre></pre>
Next, we browse the application while Burp records the requests made.

This allows us to discover paths and resources that aren't directly visible from the main
page.

<hr />

<h2>Discovering the Login Page</h2>
Through enumeration of the site we discover the directory:
<pre>/cdn-cgi/login</pre>
[caption id="attachment_3037" align="alignnone" width="776"]<img class="size-full wp-image-3037" src="https://diariohacking.com/wp-content/uploads/2026/08/5_descubrimiento_url_login_con_burp.png" alt="Browsing the site with Burp intercepting, we discover the login URL" width="776" height="364" /> Browsing the site with Burp intercepting, we discover the login URL[/caption]

Accessing:
<pre>http://10.129.x.x/cdn-cgi/login</pre>
we get the authentication page.

[caption id="attachment_3038" align="alignnone" width="525"]<img class="size-full wp-image-3038" src="https://diariohacking.com/wp-content/uploads/2026/08/6_accedemos_al_panel_login.png" alt="We reach the login panel" width="525" height="600" /> We reach the login panel[/caption]

This is the answer to the question:
<pre>What is the path to the directory on the webserver that returns a login page?</pre>
Answer:
<pre>/cdn-cgi/login</pre>

<hr />

<h2>Access as Guest</h2>
After trying to log in with credentials like admin/admin, I decide to access the application as the guest user, but we see that for the Uploads page we'll need to escalate privileges as admin.

[caption id="attachment_3039" align="alignnone" width="779"]<img class="size-full wp-image-3039" src="https://diariohacking.com/wp-content/uploads/2026/08/7_necesitamos_privilegios_admin_acceder_upload.png" alt="For the uploads page we need administrator privileges" width="779" height="333" /> For the uploads page we need administrator privileges[/caption]

While browsing with Burp, I start analyzing the requests and cookies used by the application.

The cookies store information related to our session and, in this case, also include values related to our access level and user identifier.

This kind of implementation can be dangerous if the server directly trusts values sent by the client.

<hr />

<h2>Broken Access Control</h2>
Using Firefox's dev tools (F12) we see the cookies and discover that certain values could be modified from the browser.

[caption id="attachment_3040" align="alignnone" width="785"]<img class="size-full wp-image-3040" src="https://diariohacking.com/wp-content/uploads/2026/08/8_cookies_de_sesion_dev_tools.png" alt="Session cookies stored - dev tools" width="785" height="243" /> Session cookies stored - dev tools[/caption]

The Hack The Box question was:
<pre>What can be modified in Firefox to get access to the upload page?</pre>
The answer is:
<pre>Cookie</pre>
By modifying the cookie related to our session and access level we managed to access functionality that was normally restricted to the admin user.

This represents a vulnerability of type:
<pre>Broken Access Control</pre>
The server should verify server-side what privileges the user actually has, instead of trusting values that can be modified from the browser.

<hr />

<h2>User Enumeration</h2>
We continue browsing the web application and access the <code>Account</code> section. Looking at the URL, we can see that the application uses
a numeric identifier to determine which user is being queried. We confirm we are the <code>guest</code> user, whose profile appears
associated with the parameter <code>id=2</code>.

[caption id="attachment_3042" align="alignnone" width="775"]<img class="size-full wp-image-3042" src="https://diariohacking.com/wp-content/uploads/2026/08/9_parametro_id_url.png" alt="The ID parameter in the URL" width="775" height="351" /> The ID parameter in the URL[/caption]

This suggests that user profiles might be identified using consecutive numeric values. As a test, we manually modify the <code>id</code> parameter, changing its value from <code>2</code> to <code>1</code>, to check whether it corresponds to a different user.

[caption id="attachment_3043" align="alignnone" width="774"]<img class="size-full wp-image-3043" src="https://diariohacking.com/wp-content/uploads/2026/08/10_cambiamos_valor_parametro_id.png" alt="We change the value of the ID parameter in the URL" width="774" height="357" /> We change the value of the ID parameter in the URL[/caption]

By modifying the identifier we managed to access information belonging to the admin user.

Among the data obtained is the administrator's <strong>Access ID</strong>:
<pre>34322</pre>
This identifier is especially interesting because later we can modify our session cookie to use the administrator's <code>Access ID</code>
and gain access to restricted functionality, such as the file upload page.

This behavior is related to a vulnerability of type:
<pre>IDOR / Insecure Direct Object Reference</pre>
An <strong>IDOR</strong> vulnerability occurs when an application exposes an internal identifier and allows access to resources belonging to other users
simply by modifying that value, without performing proper authorization checks on the server.

In this case, the application allows enumerating information about other users by directly modifying the <code>id</code> parameter in the URL. This allows us to discover the
admin user's information and obtain their <code>Access ID</code>, which we will later use to escalate our privileges within the application.

<hr />

<h2>Accessing the Upload Page</h2>
With the information gathered during user enumeration, we now know the admin user's <code>Access ID</code>. The next step is to modify the values stored in our session cookie to try to impersonate their privileges.

We open the browser's Developer Tools (<code>DevTools</code>) and modify the cookie values, assigning the administrator's <code>Access ID</code>:
<pre>34322</pre>
We also modify the value corresponding to the user role, changing it to:
<pre>admin</pre>
[caption id="attachment_3046" align="alignnone" width="661"]<img class="size-full wp-image-3046" src="https://diariohacking.com/wp-content/uploads/2026/08/11_modificamos_valor_cookie.png" alt="From dev tools we modify the cookie value" width="661" height="91" /> From dev tools we modify the cookie value[/caption]

This demonstrates that the application trusts values controlled directly by the client to determine the user's privileges, without performing proper server-side validation.

After modifying these values, we managed to access the <code>Upload</code> section, functionality that was originally restricted to users with
administrator privileges.

[caption id="attachment_3047" align="alignnone" width="780"]<img class="size-full wp-image-3047" src="https://diariohacking.com/wp-content/uploads/2026/08/12_conseguimos_acceso_a_uploads.png" alt="After modifying the cookie value we gain access to Upload" width="780" height="469" /> After modifying the cookie value we gain access to Upload[/caption]

This is especially interesting because a poorly protected file upload feature can allow uploading executable files.

In this case, we take advantage of this functionality to upload a PHP file containing a <em>reverse shell</em>, aiming to achieve command execution on the target machine.

<hr />

<h2>Creating the Reverse Shell</h2>
We create a PHP file:
<pre>nano revshell.php</pre>
[caption id="attachment_3051" align="alignnone" width="701"]<img class="size-full wp-image-3051" src="https://diariohacking.com/wp-content/uploads/2026/08/14_creamos_reverse_shell.png" alt="We create the PHP file with the reverse shell" width="701" height="113" /> We create the PHP file with the reverse shell[/caption]

The file contains a reverse shell configured with our Kali VPN IP address and the port we would be listening on.

On our Kali machine we start a listener:
<pre>nc -lvnp 4444</pre>
[caption id="attachment_3050" align="alignnone" width="245"]<img class="size-full wp-image-3050" src="https://diariohacking.com/wp-content/uploads/2026/08/13_iniciamos_listener.png" alt="We start the listener on port 4444" width="245" height="72" /> We start the listener on port 4444[/caption]

The communication can be represented as follows:
<pre>Oopsie
   │
   │ Reverse Connection
   ▼
10.10.14.x:4444
   │
   ▼
Kali
nc -lvnp 4444
</pre>
We upload the PHP file using the upload functionality available to the admin.

<hr />

<h2>Locating the Uploaded File</h2>
After uploading the file, we need to locate the path where the server stored it.

We try the usual directory:
<pre>/uploads</pre>
The server's response indicated that the directory existed.

Therefore, the uploaded file was located at:
<pre>/uploads</pre>
This is the answer to the question:
<pre>On uploading a file, what directory does that file appear in on the server?</pre>
Answer:
<pre>/uploads</pre>

<hr />

<h2>Getting the Reverse Shell</h2>
With the listener ready on Kali, we access the PHP file previously uploaded to the server:
<pre>http://10.129.x.x/uploads/revshell.php</pre>
[caption id="attachment_3052" align="alignnone" width="661"]<img class="size-full wp-image-3052" src="https://diariohacking.com/wp-content/uploads/2026/08/15_visitamos_la_ruta_donde_se_sube_archivo.png" alt="We visit the page where our file was uploaded to" width="661" height="183" /> We visit the page where our file was uploaded to[/caption]

When we visit this URL, the server interprets and executes the PHP file. As a result, the victim machine establishes a connection back to our Kali machine, where we have the listener ready.

We go back to the terminal where we have the listener and confirm we received the connection:

[caption id="attachment_3053" align="alignnone" width="598"]<img class="size-full wp-image-3053" src="https://diariohacking.com/wp-content/uploads/2026/08/16_comprobamos_listener_ha_funciona.png" alt="We check the listener on Kali and see that it worked" width="598" height="147" /> We check the listener on Kali and see that it worked[/caption]

This gives us an initial shell on the victim machine. However, reverse shells obtained this way are usually limited and don't have a fully interactive terminal.

This can cause problems when using certain key combinations, running interactive programs, or working comfortably with the terminal. Because of this, before continuing with enumeration, we're going to try to improve the shell's interactivity.
<h3>Shell Stabilization</h3>
First, we need to stabilize the shell.

The first step consists of creating a pseudo-terminal (PTY) using Python. We run the following command inside the remote shell:
<pre><code class="language-bash">python3 -c 'import pty; pty.spawn("/bin/bash")'</code></pre>
This gives us a more interactive Bash shell over the existing connection.

Next, we temporarily suspend the session using:
<pre><code>Ctrl+Z</code></pre>
This returns us to our local Kali terminal. From there we set the terminal to raw mode and bring the session back to the foreground:
<pre><code class="language-bash">stty raw -echo; fg</code></pre>
Once we're back in the remote shell, we set the terminal type:
<pre><code class="language-bash">export TERM=xterm</code></pre>
If the terminal doesn't respond immediately after running these commands, we can press <code>Enter</code> once or twice.

The full process can be summarized as follows:
<ol>
 	<li>Inside the remote shell, we create a pseudo-terminal:
<pre><code class="language-bash">python3 -c 'import pty; pty.spawn("/bin/bash")'</code></pre>
</li>
 	<li>We suspend the connection with:
<pre><code>Ctrl+Z</code></pre>
</li>
 	<li>Back in our Kali terminal, we run:
<pre><code class="language-bash">stty raw -echo; fg</code></pre>
</li>
 	<li>Once back inside the remote shell, we set the variable:
<pre><code class="language-bash">export TERM=xterm</code></pre>
</li>
</ol>
After this process we have a much more comfortable shell to continue working and interacting with the system.<code></code>

[caption id="attachment_3055" align="alignnone" width="406"]<img class="size-full wp-image-3055" src="https://diariohacking.com/wp-content/uploads/2026/08/17_estabilizamos_la_shell.png" alt="We stabilize the shell" width="406" height="132" /> We stabilize the shell[/caption]

We check our identity:
<pre>whoami</pre>
Expected result:
<pre>www-data</pre>
[caption id="attachment_3056" align="alignnone" width="87"]<img class="size-full wp-image-3056" src="https://diariohacking.com/wp-content/uploads/2026/08/18_comprobamos_nuestra_identidad.png" alt="We check our identity" width="87" height="37" /> We check our identity[/caption]

This confirms that we've achieved our <strong>initial foothold</strong> on the machine. From here we can continue enumerating the system in search of credentials, insecure configurations, or possible privilege escalation vectors.

<hr />

<h2>System Enumeration</h2>
Once inside the machine we continue enumerating the system.

We look for configuration files, credentials, and possible system users that could
allow lateral movement.

During this enumeration we found a file containing a password shared with
the user <code>robert</code>.

The file was:
<pre>db.php</pre>
This is the answer to the question:
<pre>What is the file that contains the password that is shared with the robert user?</pre>
Answer:
<pre>db.php</pre>
Inside the file we found credentials that could be reused to log in as the user
<code>robert</code>.

<hr />

<h2>Access as Robert</h2>
We use the credentials found to connect via SSH:
<pre>ssh robert@10.129.x.x</pre>
After authenticating we get a stable shell as:
<pre>robert@oopsie:~$</pre>
This represents lateral movement from the web server user to a local
system user.

<hr />

<h2>Privilege Enumeration</h2>
The next goal is to check whether the user <code>robert</code> can run programs
with elevated privileges.

The HTB question gives us a hint:
<pre>What executable is run with the option "-group bugtracker"
to identify all files owned by the bugtracker group?
</pre>
The answer is:
<pre>find</pre>
We can use:
<pre>find / -group bugtracker 2&gt;/dev/null</pre>
This command searches the system for files belonging to the group:
<pre>bugtracker</pre>
During this enumeration we found:
<pre>/usr/bin/bugtracker</pre>

<hr />

<h2>Identifying the SUID Binary</h2>
We check the binary's permissions:
<pre>ls -la /usr/bin/bugtracker</pre>
The result shows a permission similar to:
<pre>-rwsr-xr--</pre>
The letter:
<pre>s</pre>
indicates that the binary has the following bit set:
<pre>SUID</pre>
SUID stands for:
<pre>Set owner User ID</pre>
When an executable has the SUID bit enabled, it can run using the privileges of the
file's owner.

In this case, the owner is:
<pre>root</pre>
Therefore, regardless of which user runs the binary:
<pre>/usr/bin/bugtracker</pre>
the program will use the privileges of:
<pre>root</pre>

<hr />

<h2>Analyzing Bugtracker</h2>
After locating the binary <code>/usr/bin/bugtracker</code>, the next step is to analyze how it works to determine whether it can be used as a privilege escalation vector.

We start by running it:
<pre>/usr/bin/bugtracker</pre>
[caption id="attachment_3065" align="alignnone" width="334"]<img class="size-full wp-image-3065" src="https://diariohacking.com/wp-content/uploads/2026/08/26_ejecutamos_el_binario.png" alt="We run the binary file" width="334" height="138" /> We run the binary file[/caption]

The program asks for a <strong>Bug ID</strong> and, after entering an identifier, tries to display the information corresponding to that report.

As a first test, we analyzed whether the value entered could be vulnerable to a possible
<strong>command injection</strong>. This type of vulnerability can occur when an application directly uses user input inside a system command without performing proper validation or sanitization.

To check this, we entered an identifier followed by a second command using the <code>;</code> character, which allows chaining instructions in certain shell contexts.

The intent was to check whether the content entered as the <code>Bug ID</code> was being interpreted directly by the system and whether, due to the binary's special permissions, the additional command was executed with elevated privileges.

However, the result did not give us a shell or root access. We were still running commands with the privileges of the user <code>robert</code>, so this test indicated that the binary was not vulnerable to direct command injection via this method.

[caption id="attachment_3066" align="alignnone" width="778"]<img class="size-full wp-image-3066" src="https://diariohacking.com/wp-content/uploads/2026/08/27_comprobamos_si_es_vulnerable_a_inyeccion_de_comandos.png" alt="We run tests to check whether it's vulnerable to command injection" width="778" height="268" /> We run tests to check whether it's vulnerable to command injection[/caption]

This leads us to analyze in more detail how <code>bugtracker</code> works internally.

During the analysis we discovered that the binary uses the executable:
<pre>cat</pre>
the system needs to consult the environment variable:
<pre>PATH</pre>
to determine which executable to use.

This is especially interesting because, if we can place an executable we control in a directory and make that directory appear before the usual paths within <code>PATH</code>, the system could run our program instead of the legitimate binary <code>/bin/cat</code>.

This type of technique is known as:
<pre>PATH Hijacking</pre>
This vulnerability is especially relevant in this case due to the permissions with which <code>bugtracker</code> runs. If the binary has the <strong>SUID</strong> bit and runs with the privileges of the file's owner, any program called insecurely could end up running with those same privileges.

Therefore, after ruling out direct command injection, the next step will be to take advantage of the insecure way in which the <code>cat</code> executable is resolved.

<hr />

<h2>PATH Hijacking</h2>
Linux looks for executables following the order defined in the variable:
<pre>PATH</pre>
We can check it using:
<pre>echo $PATH</pre>
If we manage to place a directory we control at the beginning of PATH, we can
make the system run our own file before the legitimate executable.

In this case, the goal is to create an executable called:
<pre>cat</pre>
The binary <code>bugtracker</code> will try to run <code>cat</code>, but will find our
malicious file first.

<hr />

<h2>Exploitation via PATH Hijacking</h2>
Once we identified that <code>bugtracker</code> uses the <code>cat</code> command
without specifying its absolute path, we can take advantage of this behavior to carry out
a <strong>PATH Hijacking</strong> attack.

The first step is to create our own executable called <code>cat</code>.
We create it inside the <code>/tmp</code> directory and make it run a Bash shell:
<pre><code class="language-bash">echo '/bin/bash' &gt; /tmp/cat
chmod +x /tmp/cat</code></pre>
Next, we modify the <code>PATH</code> environment variable so that the
<code>/tmp</code> directory is checked before the system's usual paths:
<pre><code class="language-bash">export PATH=/tmp:$PATH</code></pre>
We can confirm that our executable takes priority using:
<pre><code class="language-bash">which cat</code></pre>
The result should show:
<pre>/tmp/cat</pre>
This confirms that, when the <code>cat</code> command is invoked, the system will find
our file in <code>/tmp</code> first.

With <code>PATH</code> modified, we run the binary again:
<pre>/usr/bin/bugtracker</pre>
When the program asks for a <strong>Bug ID</strong>, we enter any value,
for example:
<pre>1</pre>
This time, when processing the request, the binary runs our fake <code>cat</code>,
which gives us a new shell.

We check the privileges obtained:
<pre>whoami</pre>
Result:
<pre>root</pre>
[caption id="attachment_3067" align="alignnone" width="431"]<img class="size-full wp-image-3067" src="https://diariohacking.com/wp-content/uploads/2026/08/28_escalada_privilegios_a_root.png" alt="Privilege escalation to root" width="431" height="324" /> Privilege escalation to root[/caption]

We can also confirm it via the prompt:
<pre>root@oopsie:~#</pre>
This confirms that the privilege escalation was successful.

<hr />

<h2>Getting the Root Flag</h2>
Once we have a shell as root, we go to the directory:
<pre>cd /root</pre>
We list its contents:
<pre>ls</pre>
Finally we read the flag:
<pre>cat root.txt</pre>
[caption id="attachment_3068" align="alignnone" width="553"]<img class="size-full wp-image-3068" src="https://diariohacking.com/wp-content/uploads/2026/08/29_obtenemos_flag_root.png" alt="We get the root flag" width="553" height="259" /> We get the root flag[/caption]

✅ <strong>Root flag obtained successfully.</strong>

Oopsie machine completed

[caption id="attachment_3076" align="alignnone" width="764"]<img class="size-full wp-image-3076" src="https://diariohacking.com/wp-content/uploads/2026/08/30_maquina_htb_oopsie_resuelta.png" alt="Hack the Box Oopsie machine completed" width="764" height="584" /> Hack the Box Oopsie machine completed[/caption]

&nbsp;

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
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">With what kind of tool can intercept web traffic?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>Proxy</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">2</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What is the path to the directory on the webserver that returns a login page?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>/cdn-cgi/login</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">3</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What can be modified in Firefox to get access to the upload page?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>Cookie</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">4</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What is the access ID of the admin user?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>34322</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">5</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">On uploading a file, what directory does that file appear in on the server?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>/uploads</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">6</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What is the file that contains the password that is shared with the robert user?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>db.php</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">7</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What executable is run with the option "-group bugtracker" to identify all files owned by the bugtracker group?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>find</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">8</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">Regardless of which user starts running the bugtracker executable, what user's privileges will it use to run?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>root</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">9</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What SUID stands for?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>Set owner User ID</strong></td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">10</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;">What is the name of the executable being called in an insecure manner?</td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px;"><strong>cat</strong></td>
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
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Web Enumeration</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Burp Suite, Firefox, Spidering</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Web Exploitation</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Cookie Manipulation, IDOR, File Upload</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Initial Access</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">PHP Reverse Shell, Netcat</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Lateral Movement</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Credential Enumeration, SSH</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Privilege Escalation</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">SUID, PATH Hijacking, Bash</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Post-Exploitation</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Linux Commands, Filesystem Enumeration</td>
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
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Information Disclosure</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Enumerating the application reveals paths, identifiers, and functionality that should not be exposed.</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Broken Access Control</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The application allows modifying values related to the session and privileges from the client.</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>IDOR</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">User identifiers can be enumerated and used to access other users' resources.</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Unrestricted File Upload</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The upload functionality allows uploading a PHP file that can later be executed on the server.</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>Hardcoded Credentials</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">Credentials stored in configuration files allow access to another system user.</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>SUID Misconfiguration</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The bugtracker binary runs with root privileges.</td>
</tr>
<tr>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;"><strong>PATH Hijacking</strong></td>
<td style="background: #2b2b2b; color: #c9d1d9; padding: 10px; border: 1px solid #30363d;">The binary calls cat without using an absolute path, allowing the executable to be replaced by manipulating PATH.</td>
</tr>
</tbody>
</table>

<hr />

<h2>Key Takeaways</h2>
<ul>
 	<li>Web enumeration can reveal functionality that isn't visible from the main page.</li>
 	<li>A proxy like Burp Suite allows intercepting, inspecting, and modifying HTTP requests.</li>
 	<li>Applications should never trust privilege values sent directly by the client.</li>
 	<li>User identifiers must be properly validated to prevent IDOR vulnerabilities.</li>
 	<li>File upload functionality must strictly validate the type and content of files.</li>
 	<li>Credentials should not be stored directly in files accessible from compromised users.</li>
 	<li>SUID binaries should be carefully reviewed because they can run with elevated privileges.</li>
 	<li>Privileged programs must use absolute paths to run other binaries.</li>
 	<li>The PATH variable can be manipulated to hijack program execution if proper controls aren't implemented.</li>
 	<li>Several seemingly small vulnerabilities can be chained together to achieve full system compromise.</li>
</ul>

<hr />

<h2>Conclusion</h2>
Oopsie shows how a chain of relatively simple vulnerabilities can end up leading to
the complete compromise of a machine.

The attack starts with enumeration of a web application and discovery of a hidden
login page. Afterward, cookie manipulation and identifier enumeration
allow escalation from a guest user to gaining access to admin functionality.

Access to the file upload functionality allows obtaining an initial shell on the server
via a PHP reverse shell.

Next, system enumeration reveals reused credentials that allow logging in
as the user <code>robert</code> via SSH.

Finally, an insecurely configured SUID binary allows abusing the
<code>PATH</code> variable and executing code with root privileges.

<strong>Result: full machine compromise via Web Enumeration → Broken Access Control → File Upload → Credential Reuse → SUID PATH Hijacking.</strong>

<hr />

<h2>⚠️ Legal Disclaimer</h2>
This content is <strong>strictly for educational purposes</strong>.

All testing was carried out in a controlled
<strong>Hack The Box</strong> lab environment.

Do not use these techniques against systems, applications, or networks without explicit authorization.

&nbsp;
