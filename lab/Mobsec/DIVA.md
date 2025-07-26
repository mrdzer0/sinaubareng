# Cheatsheet for DIVA Exercise

## Exercise 1: Recon
**Question:**
What is the package name?

**Answer:**
jakhar.aseem.diva

**How to Solve:**
1. doing adb shell `adb shell`
2. type `pm list packages`
3. you will see the `jakhar.aseem.diva`

## Exercise 2: Insecure Logging
**Question:**
What sensitive data do you find in the log?

**Answer:**
The app logs sensitive data such as the credit-card information in Logcat (e.g., during the Insecure Logging challenge).

**How to Solve:**
1. Open DIVA in your emulator.
2. Go to the Insecure Logging feature.
3. Enter any numbers (e.g., `1337`).
4. Click Checkout.
5. In your terminal, run:
   ```sh
   adb logcat | grep diva
   ```
6. Observe the log output that displays the credit-card number in cleartext.

**Part of:** M9 - Insecure Data Storage

## Exercise 3: Hardcoded Info
**Question:**
Can you extract any hardcoded information from the app?

**Answer:**
Yes, hardcoded API keys and secret values are found inside the source code using decompilers like `jadx-gui`, especially in `HardcodeActivity.java`.
the hardcoded is `vendorsecretkey`.

**How to Solve:**
1. Decompile `diva-beta.apk using` using `jadx-gui`.
2. Navigate to `HardcodeActivity.java`.
3. Observe hardcoded credentials or secrets.

**Part of:** 
- M8 - Security Missconfiguration
- M10 - Insufficient Cryptography

## Exercise 4: Insecure Local Storage
**Question:**
Where does the app store sensitive data insecurely on the device?

**Answer:**
The app stores data like usernames and passwords in plaintext in `SharedPreferences` or internal files.

**How to Solve:**
1. Open DIVA in your emulator.
2. Go to "Insecure Data Storage".
3. Enter sample credentials..
4. Use ADB:
```sh
adb shell
cd data/data/jakhar.aseem.diva
cd shared_prefs
cat jakhar.aseem.diva_preferences.xml
```
5. The file contains the stored credentials in plaintext.

**Part of:** M9 - Insecure Data Storage

## Exercise 5: SQLite Database Storage
**Question:**
What is the DB and table name that contains user, password, and credit card?

**Answer:**
Database: `sqli`
Table: `sqliuser`

**How to Solve:**
1. Use ADB:
```sh
adb shell
cd data/data/jakhar.aseem.diva
cd databases
sqlite3 sqli 
.tables
select * from sqliuser;
```
2. Observe records including username, password, and credit card info

**Part of:** M9 - Insecure Data Storage
