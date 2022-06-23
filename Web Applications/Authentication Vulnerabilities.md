#Web 

# Authentication Vulnerabilities

### Password-based
Often times, this kind of authentication is targeted by social engineering attempts (like phishing) and brute-force attacks. Attackers are required to enumerate based on the user account they are trying to compromise by **analyzing publicly available information (internet profiles, usernames, emails and so on**). As users, they can also be negligible regarding their security, such as **using weak guessable passwords**.

Other than users, web applications may expose certain hints/weakness that can help attackers with enumeration. Such as, when enumerating usernames by an automated brute-force, attackers look for subtle differences in the web application (**response time, body, length, headers, etc**) that occur due to logic flaws or misconfigurations. This also applies when attempting to brute-force passwords as well.

About brute-force protection:
- Websites may block IP addresses to stop further attacks from happening. This can be countered with certain steps such as, **using "X-Forwarded-For:" header** to spoof origin IP, trying to **log in to a legitimate (your) account** to see if the ban is lifted, trying to within the minimum number of applicable fail attempts.
- On the other hand, account locking mechanism can be used to enumerate valid accounts/usernames.
- When rate limitations are in place, the attack has to be fine-tuned to not bombard the server with large number of requests per second.
- If data is handled as JSON values, attacker can attempt to send multiple values as an array for a single key/object within a single request.
- Paying attention header values (cookies, tokens, etc) to see if they are weakly encoded and potentially reveal sensitive that can leveraged to gain some sort of access or lead to other exploitation.

### 