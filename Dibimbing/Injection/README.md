# Injection Attacks - Training Cheatsheet


# 1. Prerequisites — Kali Linux + Docker

## 1.1 Update Kali Linux

```bash
sudo apt update
sudo apt upgrade -y
```

The full upgrade is optional for the lab, but keeping the package index current is recommended.

## 1.2 Install Docker on Kali Linux

Kali Linux provides Docker Engine through the `docker.io` package.

```bash
sudo apt install -y docker.io
```

Start Docker and enable it at boot:

```bash
sudo systemctl enable docker --now
```

Verify the service:

```bash
sudo systemctl status docker
```

Press `q` to leave the status view.

Check the installed version:

```bash
sudo docker --version
```

Optional functional test:

```bash
sudo docker run --rm hello-world
```

### Important Kali Note

Do not install the package simply named `docker` expecting Docker Engine. On Kali, the package used for Docker Engine is:

```text
docker.io
```

## 1.3 Optional — Use Docker Without `sudo`

```bash
sudo usermod -aG docker $USER
```

Then log out and log back in.

Verify:

```bash
docker ps
```

> Membership in the `docker` group effectively provides high privileges on the host. If you do not want this, continue using `sudo docker ...`.

---

# 2. Run OWASP Juice Shop

OWASP Juice Shop is an intentionally vulnerable web application created for security training.

## 2.1 Pull the Image

```bash
docker pull bkimminich/juice-shop
```

If Docker requires root privileges:

```bash
sudo docker pull bkimminich/juice-shop
```

## 2.2 Start Juice Shop

Recommended local-only setup:

```bash
docker run -d \
  --name juice-shop \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop
```

| Option | Meaning |
|---|---|
| `-d` | Run in the background |
| `--name juice-shop` | Friendly container name |
| `-p 127.0.0.1:3000:3000` | Bind Juice Shop only to the local machine |
| `bkimminich/juice-shop` | Docker image |

Open:

```text
http://127.0.0.1:3000
```

or:

```text
http://localhost:3000
```

## 2.3 Verify the Container

```bash
docker ps
```

View logs:

```bash
docker logs juice-shop
```

Follow logs:

```bash
docker logs -f juice-shop
```

Press `Ctrl+C` to stop following logs.

## 2.4 Stop, Start, and Restart

Stop:

```bash
docker stop juice-shop
```

Start:

```bash
docker start juice-shop
```

Restart:

```bash
docker restart juice-shop
```

## 2.5 Remove the Container

```bash
docker stop juice-shop
docker rm juice-shop
```

List downloaded images:

```bash
docker images
```

Remove the Juice Shop image if required:

```bash
docker rmi bkimminich/juice-shop
```

---

# 3. Docker / Juice Shop Troubleshooting

## Docker Daemon Is Not Running

Typical error:

```text
Cannot connect to the Docker daemon
```

Fix:

```bash
sudo systemctl start docker
sudo systemctl status docker
```

## Permission Denied

Typical error:

```text
permission denied while trying to connect to the Docker daemon socket
```

Quick solution:

```bash
sudo docker ps
```

Or add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Then log out and log back in.

## Port 3000 Is Already Used

Check:

```bash
sudo ss -lntp | grep :3000
```

Use another host port:

```bash
docker run -d \
  --name juice-shop \
  -p 127.0.0.1:3001:3000 \
  bkimminich/juice-shop
```

Open:

```text
http://127.0.0.1:3001
```

## Container Exits Immediately

```bash
docker ps -a
docker logs juice-shop
```

Recreate it if needed:

```bash
docker rm juice-shop

docker run -d \
  --name juice-shop \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop
```

---

# 4. Injection — Core Concept

Injection occurs when an application treats **untrusted user input as part of an instruction** instead of treating it only as data.

```text
User Input
   |
   v
Application
   |
   v
Interpreter
   |-- SQL Database
   |-- NoSQL Database
   |-- Operating System Shell
   `-- Other interpreters
```

The dangerous pattern is:

```text
INSTRUCTION + USER-CONTROLLED INPUT
```

The secure goal is to keep:

```text
INSTRUCTION
```

separate from:

```text
DATA
```

---

# 5. Common Injection Types

| Type | Target | Typical Risk |
|---|---|---|
| SQL Injection | SQL database | Authentication bypass, data access |
| NoSQL Injection | Document / NoSQL database | Query manipulation |
| Command Injection | Operating system shell | OS command execution |
| LDAP Injection | LDAP query | Authentication/query manipulation |
| XPath Injection | XML/XPath engine | Query manipulation |
| Template Injection | Template engine | Expression/code execution |

Training focus:

```text
SQL Injection
NoSQL Injection
Command Injection
```

---

# 6. SQL Injection Cheatsheet

## 6.1 Vulnerable Pattern

Unsafe example:

```javascript
const query =
  "SELECT * FROM users WHERE email = '" +
  email +
  "' AND password = '" +
  password +
  "'";
```

Normal input:

```text
email    = alice@example.com
password = password123
```

Generated query:

```sql
SELECT * FROM users
WHERE email = 'alice@example.com'
AND password = 'password123';
```

The risk appears when user input can change the SQL syntax itself.

## 6.2 SQL Syntax to Recognize

String delimiters:

```text
'
"
```

Comments vary by database:

```sql
-- comment
# comment
/* comment */
```

Boolean expressions:

```sql
1=1
1=2
'a'='a'
'a'='b'
```

Operators commonly seen while learning SQLi:

```sql
AND
OR
UNION
ORDER BY
```

## 6.3 Basic Lab Payload Patterns

Use only in authorized environments.

Syntax probes:

```text
'
"
```

Boolean examples:

```text
' OR '1'='1
' OR 1=1 --
' AND 1=2 --
```

Comment examples:

```text
' --
' #
```

Classic authentication-bypass concept:

```text
known-user' OR '1'='1' --
```

Do not only memorize the payload. Understand how it changes the query.

## 6.4 SQLi Testing Workflow

```text
1. Identify user-controlled input.
2. Send normal input.
3. Record the baseline response.
4. Introduce a syntax character.
5. Compare the response.
6. Compare TRUE and FALSE conditions.
7. Determine whether the query can be influenced.
8. Stop once the vulnerability is sufficiently demonstrated.
```

Useful comparison:

```text
TRUE condition  -> response A
FALSE condition -> response B
```

Consistent response differences may indicate query manipulation.

## 6.5 SQL Injection Categories

### Error-Based

Potential indicators:

```text
SQL syntax error
unterminated string
database exception
stack trace
```

### Boolean-Based Blind

```text
TRUE condition  -> behavior A
FALSE condition -> behavior B
```

### Time-Based Blind

```text
TRUE condition  -> delayed response
FALSE condition -> normal response
```

Use timing tests carefully to avoid unnecessary load.

### UNION-Based

Concept:

```sql
SELECT column1, column2 FROM table_a
UNION
SELECT column1, column2 FROM table_b;
```

The queries generally require compatible column structures.

---

# 7. Authentication Bypass Concept

Suppose the application builds:

```sql
SELECT *
FROM users
WHERE email = '<EMAIL>'
AND password = '<PASSWORD>';
```

Unsafe concatenation may allow attacker-controlled input to alter the condition.

Conceptually:

```sql
SELECT *
FROM users
WHERE email = 'some-user'
OR 1=1 -- '
AND password = 'anything';
```

The intended password check may no longer control the result.

Behavior depends on:

```text
Database engine
Application query
Comment syntax
Parameter handling
Authentication logic
```

---

# 8. OWASP Juice Shop — Beginner SQLi Practice

> Perform this only against your local Juice Shop container.

Open:

```text
http://127.0.0.1:3000
```

Navigate to:

```text
Account -> Login
```

## Step 1 — Establish Normal Behavior

Try invalid credentials first:

```text
Email:    test@example.com
Password: wrongpassword
```

Observe:

```text
Error message
HTTP request
HTTP response
Status code
Response body
```

## Step 2 — Test a Syntax Character

Try a quote in the controlled lab input:

```text
'
```

Compare the response with the normal baseline.

## Step 3 — Think About the Backend

Ask:

```text
What might the SQL query look like?
Where is my input inserted?
Can my input terminate a string?
Can a comment cause the rest of the query to be ignored?
```

Conceptual payload shapes:

```text
<identifier>' --
```

```text
<identifier>' OR 1=1 --
```

The important part is understanding **why** the resulting query changes.

## Step 4 — Inspect With Burp Suite

Start Burp:

```bash
burpsuite
```

Common browser proxy:

```text
Host: 127.0.0.1
Port: 8080
```

Review:

```text
Proxy -> HTTP history
```

Identify:

```text
HTTP method
URL
Content-Type
Request body
Email parameter
Password parameter
Response status
Response body
```

Send the request to:

```text
Repeater
```

Then modify **one variable at a time**.

---

# 9. NoSQL Injection Cheatsheet

NoSQL databases do not necessarily use SQL syntax.

A conceptual MongoDB-style query:

```javascript
db.users.findOne({
  username: username,
  password: password
})
```

Risk occurs when an application accepts structured attacker-controlled objects and passes them directly into database queries.

## 9.1 Common MongoDB Operators

```text
$ne      Not equal
$eq      Equal
$gt      Greater than
$gte     Greater than or equal
$lt      Less than
$lte     Less than or equal
$in      Match values in a list
$nin     Not in list
$regex   Regular-expression match
$exists  Check whether a field exists
```

Conceptual object:

```json
{
  "username": {
    "$ne": null
  }
}
```

This does not mean every application accepting JSON is vulnerable.

## 9.2 NoSQL Testing Mindset

If an API expects:

```json
{
  "username": "alice"
}
```

ask:

```text
Does the endpoint accept JSON?
Can a string become an object?
Are query operators accepted?
Are data types strictly validated?
Does application behavior change?
```

---

# 10. Command Injection Cheatsheet

Command injection occurs when user-controlled input becomes part of an operating-system shell command.

Unsafe pseudo-code:

```python
os.system("ping -c 1 " + user_input)
```

Normal input:

```text
127.0.0.1
```

Generated command:

```bash
ping -c 1 127.0.0.1
```

## 10.1 Shell Syntax to Recognize

Command separators:

```bash
;
&&
||
```

Pipe:

```bash
|
```

Command substitution:

```bash
$(command)
`command`
```

Redirection:

```bash
>
>>
<
```

## 10.2 Benign Lab Commands

For authorized demonstrations, prefer harmless commands:

```bash
whoami
id
pwd
uname -a
hostname
```

Conceptual input:

```text
127.0.0.1; whoami
```

A vulnerable application might construct:

```bash
ping -c 1 127.0.0.1; whoami
```

The shell can interpret this as two commands.

## 10.3 Command Injection Testing Workflow

```text
1. Identify features likely to call an OS utility.
2. Record normal behavior.
3. Test harmless shell syntax.
4. Use a benign command.
5. Observe output, timing, or response changes.
6. Confirm using the minimum proof needed.
7. Stop before destructive actions.
```

Features worth reviewing:

```text
Ping / network diagnostics
DNS lookup
Traceroute
File conversion
Image processing
Archive handling
PDF generation
Backup utilities
Video/audio processing
Administrative tools
```

These features are not automatically vulnerable.

---

# 11. Burp Suite Quick Workflow

```text
Browser
   |
   v
Burp Proxy
   |
   v
Target Application
```

Useful Burp areas:

```text
Proxy          -> capture traffic
HTTP history   -> review requests
Repeater       -> replay and modify requests
Decoder        -> encode/decode values
Comparer       -> compare responses
```

Recommended workflow:

```text
1. Capture a normal request.
2. Send it to Repeater.
3. Establish a baseline.
4. Change one parameter.
5. Send the request.
6. Compare the result.
7. Repeat carefully.
```

Golden rule:

```text
Change one variable at a time.
```

---

# 12. Common Injection Indicators

Possible indicators:

```text
500 Internal Server Error
Different HTTP response
Database error
Unexpected login success
Different number of records
Unusual response delay
Command output
Stack trace
Unexpected application behavior
```

Remember:

```text
An error alone does not prove injection.
```

Look for a repeatable relationship between your input and application behavior.

---

# 13. Potential Impact

## SQL / NoSQL Injection

```text
Authentication bypass
Unauthorized data access
Sensitive-data exposure
Data modification
Data deletion
Privilege escalation
Administrative account access
```

## Command Injection

```text
Execute OS commands
Read application/server files
Modify application files
Access environment secrets
Reach internal services
Compromise the application server
```

Actual impact depends on context and must be demonstrated safely.

---

# 14. Root Cause

The common root problem is:

```text
Untrusted DATA becomes trusted INSTRUCTION.
```

Unsafe patterns:

```text
SQL syntax + user input
Shell command + user input
Database operator + user input
```

Secure design keeps:

```text
CODE / INSTRUCTION
```

separate from:

```text
DATA
```

---

# 15. SQL Injection Mitigation

## 15.1 Prepared Statements / Parameterized Queries

Preferred concept:

```javascript
const result = await db.query(
  "SELECT * FROM users WHERE email = $1",
  [email]
);
```

Think:

```text
SQL instruction = fixed
User input      = data
```

Avoid:

```javascript
const query =
  "SELECT * FROM users WHERE email = '" +
  email +
  "'";
```

---

# 16. ORM / ODM

Examples:

```text
Sequelize
Prisma
Hibernate
Entity Framework
Django ORM
SQLAlchemy
Mongoose
```

Important:

```text
ORM != automatic protection from every injection issue
```

Risk can return when using:

```text
Raw SQL
Dynamic query fragments
Unsafe filters
Direct object/operator injection
```

Use safe parameterization even when raw queries are necessary.

---

# 17. Input Validation

Input validation should be used as **defense in depth**.

For an integer ID:

```text
Expected: digits only
Example: 12345
```

For a UUID:

```text
Validate against UUID format.
```

For sort direction:

```text
Allow only:
ASC
DESC
```

Prefer **allowlists** for known sets of valid values.

Input validation should complement parameterization, not replace it.

---

# 18. Command Injection Mitigation

Best principle:

```text
Do not invoke a shell when you do not need one.
```

Instead of:

```python
os.system("ping -c 1 " + host)
```

prefer APIs that pass command arguments separately:

```python
subprocess.run(
    ["ping", "-c", "1", host],
    shell=False
)
```

Additional controls:

```text
Strict allowlists
Input type/format validation
Avoid shell=True
Least privilege
Restricted file-system permissions
Restricted network access
Container/sandbox isolation where appropriate
```

---

# 19. NoSQL Injection Mitigation

```text
Validate request schemas
Enforce expected data types
Reject unexpected objects
Reject untrusted query operators
Use safe ODM/query APIs
Avoid passing request bodies directly into queries
Apply least privilege
```

Unsafe conceptual pattern:

```javascript
db.users.find(req.body)
```

Safer concept:

```javascript
const username = validateString(req.body.username);

db.users.find({
  username: username
});
```

---

# 20. Defense in Depth

Combine controls:

```text
Parameterized queries
        +
Strict input validation
        +
Safe ORM / API usage
        +
Least privilege
        +
Secure error handling
        +
Logging and monitoring
        +
Security testing
```

Do not rely on only one control.

---

# 21. Injection Testing Mini Checklist

## Before Testing

```text
[ ] I have authorization.
[ ] I know the permitted scope.
[ ] I understand the target feature.
[ ] I captured a normal request.
```

## During Testing

```text
[ ] Change one parameter at a time.
[ ] Start with harmless input.
[ ] Compare baseline and modified responses.
[ ] Avoid destructive payloads.
[ ] Record request/response evidence.
```

## After Confirmation

```text
[ ] Stop at sufficient proof.
[ ] Document the affected parameter.
[ ] Explain the root cause.
[ ] Explain realistic impact.
[ ] Provide remediation.
[ ] Retest after remediation.
```

---

# 22. Quick Reference

## SQL Injection

Look at:

```text
Login forms
Search fields
Filters
IDs
Sorting
API query parameters
JSON fields passed into SQL queries
```

Question:

```text
Can my input change the SQL query structure?
```

Primary control:

```text
Prepared / parameterized statements
```

## NoSQL Injection

Look at:

```text
JSON APIs
MongoDB-backed applications
Dynamic filters
Search functionality
Authentication APIs
```

Question:

```text
Can my input become a database query operator?
```

Primary controls:

```text
Schema validation
Strict type validation
Safe query construction
```

## Command Injection

Look at:

```text
Ping
DNS lookup
System utilities
File processing
Archive handling
Converters
Administration tools
```

Question:

```text
Can my input become part of an OS command?
```

Primary controls:

```text
Avoid shell execution
Pass arguments separately
Use strict allowlists
```

---

# 23. Key Takeaway

```text
Injection is not about memorizing payloads.

Injection is about understanding the boundary between:

DATA
and
INSTRUCTIONS
```

A good tester asks:

```text
Where does my input go?
Which interpreter receives it?
How is the input inserted?
Can it change the intended instruction?
What is the smallest safe proof?
```

A good developer asks:

```text
Can I avoid the interpreter?
Can I parameterize the value?
Can I strictly validate the input?
Can I reduce privileges?
Can I safely handle errors?
```

---

# 24. Lab Command Quick Reference

```bash
# Update package index
sudo apt update

# Install Docker
sudo apt install -y docker.io

# Enable and start Docker
sudo systemctl enable docker --now

# Check Docker
sudo docker --version
sudo docker ps

# Pull Juice Shop
sudo docker pull bkimminich/juice-shop

# Run Juice Shop locally
sudo docker run -d \
  --name juice-shop \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop

# Check containers
sudo docker ps
sudo docker ps -a

# Logs
sudo docker logs juice-shop
sudo docker logs -f juice-shop

# Stop / start / restart
sudo docker stop juice-shop
sudo docker start juice-shop
sudo docker restart juice-shop

# Remove
sudo docker stop juice-shop
sudo docker rm juice-shop
```

Access the lab:

```text
http://127.0.0.1:3000
```

---

# 25. Official References

- Kali Linux Documentation — Installing Docker on Kali Linux  
  https://www.kali.org/docs/containers/installing-docker-on-kali/
- OWASP Developer Guide — OWASP Juice Shop  
  https://devguide.owasp.org/en/07-training-education/01-vulnerable-apps/01-juice-shop/
- OWASP Juice Shop Companion Guide — Running OWASP Juice Shop  
  https://pwning.owasp-juice.shop/companion-guide/local/part1/running.html
- OWASP Cheat Sheet Series — SQL Injection Prevention  
  https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — OS Command Injection Defense  
  https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- OWASP Cheat Sheet Series — NoSQL Security  
  https://cheatsheetseries.owasp.org/cheatsheets/NoSQL_Security_Cheat_Sheet.html
