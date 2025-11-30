# **POC – Java Bug Analysis**

## **Authors**

| Author         | Created on | Version | Last updated by | Last Edited On | Level        | Reviewer |
| -------------- | ---------- | ------- | --------------- | -------------- | ------------ | -------- |
| Syed Rehan Ali | 2025-11-26 | 1.0     | Syed Rehan Ali  | 2025-11-27     | Pre Reviewer |Komal          |
| Syed Rehan Ali | 2025-11-26 |         | Syed Rehan Ali  |                | L0 Reviewer  |          |
| Syed Rehan Ali | 2025-11-26 |         | Syed Rehan Ali  |                | L1 Reviewer  |          |
| Syed Rehan Ali | 2025-11-26 |         | Syed Rehan Ali  |                | L2 Reviewer  |          |

---

## **Table of Contents**

1. [Overview](#1-overview)
2. [Repository Structure](#2-repository-structure)
3. [Prerequisites](#3-prerequisites)
4. [POC Execution Steps](#4-poc-execution-steps)
   4.1 [Optional: NVD API Key for Faster OWASP Scans](#41-optional-nvd-api-key-for-faster-owasp-scans)
5. [Troubleshooting](#5-troubleshooting)
6. [Recommendation](#6-recommendation)
7. [Conclusion](#7-conclusion)
8. [Sequence for Screenshots](#8-sequence-for-screenshots)
9. [Contact Information](#9-contact-information)
10. [References](#10-references)

---

### **1. Overview** <a name="1-overview"></a>

* Automated bug detection using **SpotBugs & SonarQube**
* Test coverage analysis using **JaCoCo**
* Security vulnerability detection with **OWASP Dependency Check**
* Reporting & tracking of defects via **HTML and dashboard reports**
* Dockerized execution ensures compatibility across Windows, WSL, and Linux

---

### **2. Repository Structure** <a name="2-repository-structure"></a>

| File/Folder | Purpose                   |
| ----------- | ------------------------- |
| /src        | Java source code          |
| /target     | Build output              |
| pom.xml     | Maven build configuration |

---

### **3. Prerequisites** <a name="3-prerequisites"></a>

| Tool / Requirement                 | Installation / Link                                                                                                                                   |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Java JDK 17+                       | [Adoptium](https://adoptium.net)                                                                                                                      |
| Maven 3.9+                         | [Download](https://maven.apache.org/download.cgi)                                                                                                     |
| SonarQube Server                   | [Docs](https://docs.sonarqube.org/latest/setup/install-server/)                                                                                       |
| SonarScanner CLI (Docker or local) | Docker: `sonarsource/sonar-scanner-cli` <br>Local: [Docs](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/) |
| SpotBugs Maven Plugin              | Add to `pom.xml`                                                                                                                                      |
| JaCoCo Maven Plugin                | Add to `pom.xml`                                                                                                                                      |
| OWASP Dependency Check CLI / Maven | Docker: `owasp/dependency-check:latest` <br>Maven: `org.owasp:dependency-check-maven`                                                                 |
| Browser for HTML reports           | Any modern browser                                                                                                                                    |

---

### **4. POC Execution Steps** <a name="4-poc-execution-steps"></a>

| Step | Task Performed             | Description / Output / Notes                                                                                                                                                                                                                                                                    |
| ---- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Clone Repository           | Retrieve the Java project repository                                                                                                                                                                                                                                                            |
| 2    | Navigate to Project Folder | Enter the project directory                                                                                                                                                                                                                                                                     |
| 3    | Clean & Compile Project    | Build the project and ensure compilation success (`BUILD SUCCESS`)                                                                                                                                                                                                                              |
| 4    | Run Unit Tests             | Execute tests to verify functionality (`Tests run: 10, Failures: 0`)                                                                                                                                                                                                                            |
| 5    | Generate JaCoCo Report     | Produce test coverage report in HTML format <br>**WSL / Windows:** `explorer.exe "$(wslpath -w target/site/jacoco/index.html)"` <br>**Linux GUI:** `xdg-open target/site/jacoco/index.html`                                                                                                     |
| 6    | Run SpotBugs Analysis      | Identify static code bugs and code smells                                                                                                                                                                                                                                                       |
| 7    | Run SonarQube Analysis     | Docker CLI: <br>`docker run --rm -e SONAR_HOST_URL="http://host.docker.internal:9000" -e SONAR_TOKEN="sqa_61202acee7c120547d21a05695fb6896419a55e4" -v "$(pwd)":/usr/src sonarsource/sonar-scanner-cli -Dsonar.projectKey=java-checks -Dsonar.sources=src -Dsonar.java.binaries=target/classes` |
| 8    | Run OWASP Dependency Check | Docker CLI: <br>`docker run --rm -v "$(pwd)":/src -v "$(pwd)/owasp-report:/report" owasp/dependency-check:latest --project "salary-api" --scan /src --format "HTML" --out /report` <br>HTML output: `explorer.exe "$(wslpath -w owasp-report/dependency-check-report.html)"`                    |
| 9    | Review Reports             | Open JaCoCo, SpotBugs, SonarQube, OWASP HTML reports for coverage, vulnerabilities, and bugs                                                                                                                                                                                                    |
| 10   | Fix Bugs & Repeat          | Apply fixes and repeat analysis until no critical issues remain                                                                                                                                                                                                                                 |

---

### **4.1 Optional: NVD API Key for Faster OWASP Scans** <a name="41-optional-nvd-api-key-for-faster-owasp-scans"></a>

**Why:** Without an NVD API key, OWASP Dependency-Check downloads the entire NVD feed for each run, which can take a long time (your logs show progress of 319,612 records). Using an API key allows incremental updates and faster scans.

**Steps to use the NVD API key:**

1. **Obtain NVD API Key:**

   * Register at [NVD API Key Request](https://nvd.nist.gov/developers/request-an-api-key)
   * Copy the API key provided.

2. **Run OWASP scan with API key (Docker example):**

   ```bash
   docker run --rm \
     -v "$(pwd)":/src \
     -v "$(pwd)/owasp-report:/report" \
     -e NVD_API_KEY="YOUR_NVD_API_KEY" \
     owasp/dependency-check:latest \
     --project "salary-api" \
     --scan /src \
     --format "HTML" \
     --out /report
   ```

3. **Optional: Cache NVD DB to avoid repeated downloads without API key:**

   ```bash
   docker run --rm \
     -v "$(pwd)":/src \
     -v "$(pwd)/owasp-db:/usr/share/dependency-check/data" \
     -v "$(pwd)/owasp-report:/report" \
     owasp/dependency-check:latest \
     --project "salary-api" \
     --scan /src \
     --format "HTML" \
     --out /report
   ```

---

### **5. Troubleshooting** <a name="5-troubleshooting"></a>

| Issue                           | Cause                                     | Solution                                                                                   |
| ------------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------ |
| SonarScanner cannot connect     | Using `localhost` inside Docker container | Use `host.docker.internal` on WSL: `http://host.docker.internal:9000`                      |
| Docker container fails to start | Port conflict / existing container        | Remove existing container: `docker rm -f sonarqube`                                        |
| HTML report cannot be opened    | No GUI browser                            | Open HTML report in browser                                                                |
| Maven build or test fails       | Wrong JDK / missing dependencies          | Verify JDK 17+, run `mvn clean install`                                                    |
| OWASP Dependency Check slow     | Full NVD feed downloaded each run         | Optional: Use `NVD_API_KEY` environment variable for faster incremental updates            |
| OWASP Dependency Check outdated | No internet / old DB                      | Ensure connectivity, add `--updateonly` option if needed or mount local cache (`owasp-db`) |
| SonarScanner CLI not found      | Not installed or PATH missing             | Use Docker version or add local binary to PATH                                             |

---

### **6. Recommendation** <a name="6-recommendation"></a>

* Maintain high **test coverage (80–90%)**
* Integrate **static analysis and security scans** in CI/CD pipelines
* Use **SonarQube Quality Gates** to track code quality
* Keep OWASP Dependency-Check database up-to-date or use NVD API key for faster scans

---

### **7. Conclusion** <a name="7-conclusion"></a>

This POC validates a **robust Java Bug Analysis workflow**:

* Automated builds & compilation
* Static code analysis (SpotBugs, PMD, SonarQube)
* Test coverage reporting (JaCoCo)
* Security scanning for vulnerable dependencies (OWASP Dependency Check)
* HTML and dashboard reports for developer review
* Dockerized setup ensures consistent results across platforms
* Optional NVD API key improves scan performance

---

### **8. Sequence for Screenshots** <a name="8-sequence-for-screenshots"></a>

| Step | Screenshot / Notes                             |
| ---- | ---------------------------------------------- |
| 1    | Git clone repository                           |
| 2    | Navigate project folder                        |
| 3    | Maven clean & compile output (`BUILD SUCCESS`) |
| 4    | Maven test results                             |
| 5    | JaCoCo report opened in browser                |
| 6    | SpotBugs analysis summary                      |
| 7    | SonarQube dashboard screenshot                 |
| 8    | OWASP Dependency Check HTML report screenshot  |
| 9    | Bug review process                             |
| 10   | Fix applied & re-run steps                     |

**Screenshots you provided:**

<img width="1911" height="461" alt="image" src="https://github.com/user-attachments/assets/b94ff6f4-6a63-43b6-af44-2a49a5f4b3bc" />  
<img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/09de3819-6980-4950-9e1d-777903cec289" />  

<img width="1912" height="939" alt="image" src="https://github.com/user-attachments/assets/a60d41b6-bcc4-40fe-8132-7303d3beda06" />  
 
<img width="1915" height="592" alt="image" src="https://github.com/user-attachments/assets/1120503d-5701-43ec-90e6-61859bb120ef" />  
<img width="1906" height="835" alt="image" src="https://github.com/user-attachments/assets/06a5d1ed-76e8-46f8-9e88-eebc838535cf" />  
<img width="1918" height="835" alt="image" src="https://github.com/user-attachments/assets/5ecbdc1f-f548-49c3-9c17-e482b6db01d4" />  
<img width="1918" height="552" alt="image" src="https://github.com/user-attachments/assets/dc958a03-e10d-4ba4-bf00-99c77fb00cef" />
<img width="1914" height="998" alt="image" src="https://github.com/user-attachments/assets/99d8f980-a9fd-408b-af02-f3add81b4e81" />
<img width="1919" height="937" alt="image" src="https://github.com/user-attachments/assets/cfa8446b-1a01-4b02-aa55-4e81311d2bb0" />
<img width="1917" height="937" alt="image" src="https://github.com/user-attachments/assets/35d5a09e-5493-4c45-87f1-6f7637495fc4" />
<img width="1919" height="944" alt="image" src="https://github.com/user-attachments/assets/9c4a9838-2d4d-46df-9a89-14aeb27ec1bc" />
<img width="1912" height="949" alt="image" src="https://github.com/user-attachments/assets/e2fb070f-d46b-421b-bf38-6d01e75551e5" />

---

### **9. Contact Information** <a name="9-contact-information"></a>

| Name           | Email                                                                                 |
| -------------- | ------------------------------------------------------------------------------------- |
| Syed Rehan Ali | [syed.rehan.ali.snaatak@mygurukulum.co](mailto:syed.rehan.ali.snaatak@mygurukulum.co) |

---

### **10. References** <a name="10-references"></a>

| Resource               | Link                                                       |
| ---------------------- | ---------------------------------------------------------- |
| SonarQube              | [https://www.sonarsource.com](https://www.sonarsource.com) |
| SpotBugs               | [https://spotbugs.github.io](https://spotbugs.github.io)   |
| JaCoCo                 | [https://www.jacoco.org](https://www.jacoco.org)           |
| OWASP Dependency Check | [https://owasp.org](https://owasp.org)                     |
| Maven                  | [https://maven.apache.org](https://maven.apache.org)       |
| Docker                 | [https://docs.docker.com](https://docs.docker.com)         |
