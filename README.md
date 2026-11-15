# Healthcare IT Help Desk Ticketing Lab (osTicket & Docker)

## Project Overview
This project is a self-hosted, containerized **Healthcare IT Help Desk Ticketing System** deployed locally using **Docker Compose**, **osTicket**, and **MariaDB**. The objective of this lab was to simulate a real-world hospital/clinical Tier 1 IT Support environment—from standing up the underlying server infrastructure and configuring administrative routing rules to triaging, troubleshooting, and resolving urgent clinical support tickets.


## Key Skills Demonstrated
* **Containerization & Infrastructure:** Docker, Docker Compose, MariaDB, Linux CLI, YAML configuration, local port mapping.
  
* **IT Service Management (ITSM):** Ticket lifecycle management, priority triage, SLA policy enforcement, department routing, and documentation best practices.
  
* **Healthcare IT Support:** Epic Hyperspace / Citrix session troubleshooting, Workstation on Wheels (WOW) hardware diagnostics, Imprivata Single Sign-On (SSO) badge enrollment, and urgent clinical communication.

---

## Part 1: Terminal Setup & Architecture Deployment

Rather than installing a web server, PHP engine, and database directly onto the host operating system, I built an isolated multi-container architecture from the command line  using **Docker Compose**:

### Step 1:
- Creating the Lab Directory & Writing `docker-compose.yml` via `nano`
* **Action:** Opened the macOS Terminal, created the `DOCKER__LAB` workspace directory using `mkdir` command, launched the `nano` text editor to author the `docker-compose.yml` configuration file:

```bash
mkdir DOCKER__LAB
cd DOCKER__LAB
nano docker-compose.yml
```
---

### Step 2:
- **Database Container (`osticket-db`):** Configured **MariaDB 10.11** on a persistent Docker volume (`db_data`) to store all ticket histories, user records, department configurations, and internal troubleshooting logs.
* **Application Container (`osticket-app`):** Configured **osTicket** on an Alpine Linux image powered by **Nginx** and **PHP-FPM**, mapped to `localhost:8080` on the host machine (`platform: linux/amd64` for Apple Silicon compatibility) and linked to the database over an isolated Docker bridge network.

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: osticket-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: RootPassword123!
      MYSQL_DATABASE: osticket
      MYSQL_USER: osticket_user
      MYSQL_PASSWORD: OsticketPassword123!
    volumes:
      - db_data:/var/lib/mysql

  osticket:
    image: devinsolutions/osticket:latest
    platform: linux/amd64
    container_name: osticket-app
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: osticket
      MYSQL_USER: osticket_user
      MYSQL_PASSWORD: OsticketPassword123!
      ADMIN_FIRSTNAME: Admin
      ADMIN_LASTNAME: User
      ADMIN_EMAIL: admin@homelab.local
      ADMIN_USERNAME: ostadmin
      ADMIN_PASSWORD: AdminPassword123!
      INSTALL_SECRET: MySecretKey123!
    volumes:
      - osticket_data:/var/www/html

volumes:
  db_data:
  osticket_data:
```
<img width="1469" height="599" alt="s2_nano" src="https://github.com/user-attachments/assets/6678c38d-ba81-4233-a9cd-04e8c0cfdd6b" />

---

## Part 2: Initializing & Verifying the Containers via Terminal

### Step 1: 
* **Action:** Saved and exited `nano` (`Ctrl + O`, `Enter`, `Ctrl + X`), then ran the Docker Compose startup command in detached mode (`-d`) to pull the images, initialize the database, and launch both containers:
```bash
docker compose up -d
docker ps
```

### Step 2:

* **Verification:** Confirmed both `osticket-db` and `osticket-app` were active and listening on port `8080`:

<img width="609" height="124" alt="s6 2_docker" src="https://github.com/user-attachments/assets/df594008-cefa-4b8e-b85e-b6343a2b0a45" />

---

## Part 3: Step-by-Step Post-Login Configuration

### Step 1: 
- After initializing the containers and logging into the Staff Control Panel (`http://localhost:8080/scp/`), I configured the ticketing system from a blank installation into a structured healthcare support desk:

<img width="1466" height="880" alt="s4_osTicket" src="https://github.com/user-attachments/assets/da2e72bf-662f-4146-8b87-6e08f5c40df4" />

* **Verification:** Confirmed admin log in was successful and interface is functional.

<img width="1468" height="878" alt="s5_confirmation" src="https://github.com/user-attachments/assets/966129be-5346-48db-b616-7497fc2d84a9" />

--- 

## Part 4: Creating Clinical Departments

### Step 1: Creating Clinical Departments
To mirror how hospital IT teams separate Tier 1 triage from specialized clinical engineers, I navigated to **Admin Panel > Agents > Departments** and created dedicated routing groups:
* **Clinical Applications (EHR):** Handles Epic Hyperspace, Citrix XenApp, and clinical documentation software issues.
* **Clinical Desktop & Hardware:** Handles Workstations on Wheels (WOWs), monitors, barcode scanners, and Zebra label printers.
* **Identity & Access Management (IAM):** Handles Active Directory password resets, account lockouts, and Imprivata OneSign badge enrollments.

<img width="731" height="486" alt="s5 2_departments" src="https://github.com/user-attachments/assets/45603e25-9995-4c88-b1d8-8be5ea54604f" />

### Step 2: Configuring SLA (Service Level Agreement) Plans
Because downtime in a clinical setting directly impacts patient care, I navigated to **Admin Panel > Manage > SLA Plans** to establish response targets based on clinical urgency:
* **Critical Clinical SLA (1-hour Grace Period):** Applied to exam room, Emergency Department, and ICU workstation outages where patient care is actively delayed.
* **Standard Clinical SLA (4-Hour Grace Period):** Applied to routine hardware requests, non-urgent printer issues, and general software questions.

<img width="728" height="321" alt="s5 3_SLA" src="https://github.com/user-attachments/assets/2323b36a-01a5-4bbf-abb4-f2fcc07dd7ae" />

### Step 3: Building Healthcare Help Topics & Automated Routing
To ensure end-users could easily categorize their issues from the self-service portal (`http://localhost:8080`), I navigated to **Admin Panel > Manage > Help Topics** and built custom intake categories mapped directly to their respective departments and SLA plans:
* `EHR / Epic Hyperspace Access` → Auto-routed to **Clinical Applications** (High Priority)
* `Clinical Hardware / Workstation on Wheels (WOW)` → Auto-routed to **Clinical Desktop & Hardware** (Medium Priority)
* `Imprivata Badge & Single Sign-On` → Auto-routed to **Identity & Access Management** (High Priority)
* `Medical Printers & Specimen Labelers` → Auto-routed to **Clinical Desktop & Hardware** (Medium Priority)

<img width="730" height="513" alt="s9 3_helptopics" src="https://github.com/user-attachments/assets/42aeac10-1ef0-48e4-93e2-36d235eb76a1" />

<img width="727" height="512" alt="s9 4_helptopics" src="https://github.com/user-attachments/assets/652baeb6-f859-4ef8-b0bd-adf7c40c6d64" />

---

## Part 5: Simulated Healthcare Support Tickets & Resolutions

### Step 1: 
After completing the backend configuration, I switched to the end-user web portal (`http://localhost:8080`) to submit three realistic clinical incidents.

<img width="735" height="820" alt="s10 2_ticket" src="https://github.com/user-attachments/assets/d798a1c3-4f1e-4aa2-aabb-81891654c7b4" />

* **Confirmation the ticket was submitted successfully**
<img width="725" height="821" alt="s10 3_confirmation" src="https://github.com/user-attachments/assets/86aee851-8ef9-4e58-9324-5edcbe4448c3" />

### Step 2: 
- Returned to the Agent Queue to triage, document internal root-cause notes, communicate with the clinician, and close each ticket.

<img width="717" height="352" alt="s11_open_ticket" src="https://github.com/user-attachments/assets/16ece768-96a7-4378-9c65-876477f5351e" />

---

### Ticket #1: Epic Hyperspace Failing to Launch via Citrix
* **Submitted By:** Dr. Aris Thorne (Emergency Department)
* **Help Topic:** `EHR / Epic Hyperspace Access`
* **Priority:** High
* **Reported Issue:** *"I'm trying to open Epic Hyperspace on Exam Room 4's computer, and it keeps hanging on the Citrix Receiver loading screen before saying 'Cannot connect to the Citrix XenApp server.' I have patients waiting."*

**Troubleshooting & Root Cause (Internal Note):**
- Verified workstation `ED-RM04-PC` had active LAN connectivity via ICMP ping test to the local gateway.
- Inspected Windows Task Manager and identified hung Citrix Workspace background processes (`concentr.exe` and `wfica32.exe`) caused by an improper session disconnect.
- Terminated the hung Citrix processes, cleared the local Citrix Receiver cache, and reset the user's orphaned session on the Citrix StoreFront server.

<img width="716" height="832" alt="DrThorne_log" src="https://github.com/user-attachments/assets/45159893-264d-4810-9916-a1e301de0056" />

**Resolution & User Communication:** Informed Dr. Thorne that his hung Citrix session was cleared and verified that Epic Hyperspace launched cleanly on Exam Room 4's workstation before closing the ticket.

<img width="717" height="833" alt="DrThorne_Email" src="https://github.com/user-attachments/assets/8b9b2073-0fbe-4908-baa6-9062b6877ab7" />

---

### Ticket #2: Clinical Monitor Randomly Shutting Off During Charting
* **Submitted By:** Maya Lin, RN (3rd Floor Med-Surg)
* **Help Topic:** `Clinical Hardware / Workstation on Wheels (WOW)`
* **Priority:** Medium
* **Reported Issue:** *"The monitor on our rolling computer cart (WOW-3B) keeps going black randomly for a few seconds while we're charting meds, then turns back on. Sometimes it won't wake up unless we reboot."*
  
**Troubleshooting & Root Cause (Internal Note):**
- Performed a physical inspection of the rolling cart mount and discovered a loose DisplayPort cable connection caused by repeated cart movement.
- Checked Windows Power & Sleep settings and disabled PCI Express Link State Power Management and USB selective suspend to prevent aggressive power-saving dropouts while running on cart battery power.
- Reseated the DisplayPort cable securely into the locking port and verified stable battery inverter voltage.
  
<img width="735" height="768" alt="MayaLin_log" src="https://github.com/user-attachments/assets/57b48bec-3ef7-4cdf-902f-c744a75b7636" />

**Resolution & User Communication:** Explained to Maya that the loose video cable was secured and power-saving display timeouts were adjusted, confirming normal display stability during bedside charting.

<img width="725" height="770" alt="MayaLin_Email" src="https://github.com/user-attachments/assets/2eb90d2d-7473-4e56-a5b4-ae161618416a" />


---

### Ticket #3: Imprivata Badge Tap Not Logging Nurse Into ICU Workstation
* **Submitted By:** Carlos Rivera, RN (Intensive Care Unit)
* **Help Topic:** `Imprivata Badge & Single Sign-On`
* **Priority:** High
* **Reported Issue:** *"I got a replacement hospital ID badge from Security this morning, and now when I tap my badge on the workstation reader in ICU Pod B, nothing happens—it won't auto-login."*

**Troubleshooting & Root Cause (Internal Note):**
- Verified that the USB proximity badge reader on `ICU-POD-B02` had a solid green power indicator and registered a beep upon badge tap, ruling out hardware failure.
- Checked the user's profile in the Imprivata OneSign Admin Console and confirmed the new replacement badge's RFID serial number had not yet been bound to Active Directory account `crivera`.
- Triggered the self-service badge enrollment prompt at the workstation and walked the nurse through authenticating with his AD credentials to link the new badge.

<img width="728" height="768" alt="Priyapatel_log" src="https://github.com/user-attachments/assets/7cf000ef-4271-4d34-a61b-cbf4a2699e26" />


**Resolution & User Communication:** Confirmed with Carlos that his replacement badge was successfully enrolled in Imprivata OneSign and tested tap-and-go authentication on the ICU workstation.

<img width="736" height="773" alt="PriyaPatel_email" src="https://github.com/user-attachments/assets/5a60660c-2208-4319-b272-e8d0c44b0cc1" />

---
