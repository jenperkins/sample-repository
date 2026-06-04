# Content Module Metadata

| Portal Content Module Title | NT M8L4 Troubleshooting with backups |
| ----: | :---- |
| **Module Author(s)** |  |
| **URL \- Jira Ticket** |  |
| **URL \- Content Module** |  |
| **URL \- SCP** |  |
| **Current DEV Deployment** |  |
| **Current Clone Source** |  |
| **Release Clone Source** |  |

# Lesson Metadata

| Description | Troubleshooting failed DMSS components by recognizing failure indicators in logs, Docker states, and SaltStack configurations, then applying systematic diagnostic workflows to restore container services and network visibility. |
| ----: | :---- |
| **Duration** |  |
| **Contains CUI?** | Unselected |

**Begin content after this point.**

# **Add task card \# below. For a new chain, add the prefix “NC” before the task number.**

# NC-1-Introduction

**Course Banner Image**

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Network     │                                                           │
│  Technician  │     Troubleshooting with Backups                          │
│              │                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

*Purple/orange gradient banner with course title*

*![][image1]*

System backup and recovery strategies are applied across critical Distributed Mission Support System (DMSS) components, including pfSense, Security Onion, FreeIPA, Cisco Internetwork Operating System (IOS), and Windows systems. Full, incremental, and configuration-specific backups are performed using platform-specific tools and cross-platform automation such as Ansible. Key services are preserved through systematic backups, including authentication with FreeIPA, log collection with Zeek and Suricata, visibility dashboards with Kibana, and topology modeling with RedSeal.

Diagnosing and recovering failed components rely on recognizing failure indicators through logs, Docker states, and SaltStack configurations. Practical applications include troubleshooting container failures in Security Onion, resolving SaltStack misconfigurations, and recovering from misapplied network settings in Cisco switching environments.

Network technicians (NTs) maintain operational readiness in Department of Defense (DoD) cyber operations by restoring visibility and functionality when misconfiguration, software failure, or mission redeployment disrupts data collection and detection pipelines.

**Estimated time to complete:** X.x hours  
**Round time to the half-hour; omit “.0” for full hours.**

**Module Art**

*Illustration depicting system troubleshooting and recovery operations. The scene includes:*

- *Docker container icons showing service states*  
- *Database backup symbols with restore arrows*  
- *Network switch with port status indicators*  
- *Terminal window displaying log output*  
- *Configuration file icons (YAML, SLS)*  
- *SaltStack pillar hierarchy representation*

# Topics

* Backup strategies for platform recovery  
* System restoration indicators  
* Docker fundamentals and container troubleshooting  
* SaltStack configuration management  
* Security Onion container diagnostics  
* Switch hardware fault identification

# Enabling Learning Objectives

* ELO 8.4.1-P DEMONSTRATE troubleshooting failed system components  
* ELO 8.4.2-P EXECUTE troubleshooting for computer software issues  
* ELO 8.4.3-P EXECUTE troubleshooting for computer hardware issues

# Toolkit

**Security Onion**

**Tool Logo Banner**

Security Onion is a Linux distribution for threat hunting, enterprise security monitoring, and log management. This lesson uses Security Onion utilities including `so-status` for container health monitoring and `so-elasticsearch-restart` for service recovery.

**Docker**

**Tool Logo Banner**

Docker is a containerization platform that packages applications into portable containers. This lesson uses Docker CLI commands including `docker logs` to examine container startup errors and diagnose configuration failures.

**SaltStack**

**Tool Logo Banner**

SaltStack is a configuration management system that manages Security Onion services through states and pillar data. This lesson uses `salt-call pillar.get` to query configuration values and trace container failures to pillar misconfigurations.

**Cisco IOS**

**Tool Logo Banner**

Cisco Internetwork Operating System (IOS) provides the command-line interface for Cisco network devices. This lesson covers diagnostic commands including `show spanning-tree`, `show interfaces`, and `show vlan brief` for identifying switch hardware faults.

# NC \- \# \- Backup Strategies for Platform Recovery

**VMs**:  
**Attachments**:  
**ELOs**: 8.4.1-P DEMONSTRATE troubleshooting failed system components  
8.4.2-P EXECUTE troubleshooting for computer software issues

Backup strategies for platform recovery establish repeatable methods to preserve and restore configurations, data, and systems. Procedures vary by platform, from exporting configurations in pfSense and Cisco IOS to archiving authentication and directory data in FreeIPA to safeguarding event pipelines and dashboards in Security Onion and Kibana. Windows restore points and RedSeal exports extend coverage to host systems and network modeling platforms. Core methods include full, incremental, and differential backups, configuration exports, snapshots, and virtualization images. Cross-platform automation with Ansible ensures consistency by applying inventory snapshots and scripted procedures across nodes. Each method aligns with operational needs such as patching, network reconfiguration, or mission redeployment, providing rapid recovery and sustained visibility.

**Core Backup Types**

Backup types provide different levels of coverage and efficiency depending on operational requirements. Selecting the correct type depends on storage capacity, recovery speed, and the criticality of the preserved data or system.

**Full Backup**  
A full backup captures all system data or configuration files in one operation. It creates a complete copy of the protected system, making recovery easy because only a single dataset is needed. However, it requires a lot of storage and takes longer to process. Full backups often establish baseline protection before deploying more efficient strategies.

**Incremental Backup**  
An incremental backup saves only changed data since the last full backup. It reduces storage needs and backup time, making it ideal for short-interval or frequent backup schedules. Recovery is more complex because it requires the last full backup and every incremental backup made afterward, forming a chain that must be restored.

**Figure: Incremental Backup Schedule**

```
                                                              ┌─────────┐
                                                              │ Friday  │
                                              ┌─────────┐     ├─────────┤
                                              │Thursday │     │Thursday │
                              ┌─────────┐     ├─────────┤     ├─────────┤
                              │Wednesday│     │Wednesday│     │Wednesday│
              ┌─────────┐     ├─────────┤     ├─────────┤     ├─────────┤
              │ Tuesday │     │ Tuesday │     │ Tuesday │     │ Tuesday │
┌─────────┐   ├─────────┤     ├─────────┤     ├─────────┤     ├─────────┤
│ Monday  │   │ Monday  │     │ Monday  │     │ Monday  │     │ Monday  │
├─────────┤   ├─────────┤     ├─────────┤     ├─────────┤     ├─────────┤
│ Sunday  │   │ Sunday  │     │ Sunday  │     │ Sunday  │     │ Sunday  │
│Full Bkup│   │Full Bkup│     │Full Bkup│     │Full Bkup│     │Full Bkup│
└─────────┘   └─────────┘     └─────────┘     └─────────┘     └─────────┘
   Mon           Tue             Wed             Thu             Fri
```

*Each day's incremental backup captures only changes since the previous backup. Recovery requires the full backup plus all subsequent incremental backups in sequence.*

**Differential Backup**  
A differential backup saves all data changed since the last full backup. Recovery is simpler than incremental backup because it requires only the last full and most recent differential backup. Storage use is higher than incremental backups, but less than repeated full backups. Differential backups are useful when recovery speed is more important than minimizing storage.

**Figure: Differential Backup Schedule**

```
                                                              ┌─────────┐
                                                              │ Friday  │
                                                              │    ↑    │
                                              ┌─────────┐     │Thursday │
                                              │Thursday │     │    ↑    │
                              ┌─────────┐     │    ↑    │     │Wednesday│
                              │Wednesday│     │Wednesday│     │    ↑    │
              ┌─────────┐     │    ↑    │     │    ↑    │     │ Tuesday │
              │ Tuesday │     │ Tuesday │     │ Tuesday │     │    ↑    │
┌─────────┐   │    ↑    │     │    ↑    │     │    ↑    │     │ Monday  │
│ Monday  │   │ Monday  │     │ Monday  │     │ Monday  │     │    ↑    │
├─────────┤   ├─────────┤     ├─────────┤     ├─────────┤     ├─────────┤
│ Sunday  │   │ Sunday  │     │ Sunday  │     │ Sunday  │     │ Sunday  │
│Full Bkup│   │Full Bkup│     │Full Bkup│     │Full Bkup│     │Full Bkup│
└─────────┘   └─────────┘     └─────────┘     └─────────┘     └─────────┘
   Mon           Tue             Wed             Thu             Fri
```

*Each differential backup captures ALL changes since the last full backup. Recovery requires only the full backup plus the most recent differential backup.*

**Configuration Export**  
A configuration export generates a file containing only system or application configuration data. Examples include exporting config.xml from pfSense or the running configuration from Cisco IOS. These small exports enable quick restoration of the system state without copying an entire filesystem. Configuration exports are most effective before network or policy changes, allowing a fast rollback to a known working state. It is important to note that these only contain the configuration, not past or current data.

**Snapshot**  
A snapshot captures the state of a VM or application at a specific time. Snapshots are handy for recovery after applying updates or patches because they allow a quick rollback to a stable state if errors happen. While snapshots enable fast recovery, they use storage and are not meant for long-term storage.

**OVA image.**  
An Open Virtualization Archive (OVA) image contains a complete VM along with its configuration settings. OVA images enable the redeployment of entire environments across different hypervisors, supporting portability and the transfer of missions. They are larger and take longer to create than snapshots or configuration exports, but are essential for scenarios that require complete system redeployment or long-term preservation.

**Summary**

Backup types range from complete system copies to lightweight configuration exports and virtual machine packages. The appropriate method depends on recovery objectives, available storage, and operational urgency. A layered approach combining multiple backup types provides reliable data and system availability protection.

# \#-System Restoration Indicators

**VMs**:  
**Attachments**:  
**ELOs**: 8.4.1-P – Demonstrate troubleshooting failed system components

Restoration becomes necessary when standard remediation fails to stabilize operations. Administrators rely on general symptoms and platform-specific log messages to recognize when a system has gone beyond routine troubleshooting. These indicators show when corruption, drift, or persistent service failures require restoration from backup or re-deployment.

**General Signs of Failure**

System failures disrupt visibility, availability, and security across distributed monitoring and security systems (DMSS). While routine issues can often be resolved through configuration changes or service restarts, certain indicators signal deeper problems that require full or partial system restoration. Recognizing these signs helps administrators distinguish between recoverable faults and conditions that threaten data integrity, service stability, or platform reliability.

* **Persistent configuration drift despite remediation**

Systems diverge from the intended baseline, even after corrective steps are taken. For example, firewall rules or routing policies revert after reboot, or updated parameters do not persist across restarts. This suggests that configuration files, management processes, or automation mechanisms are broken, requiring restoration to a known-good state.

* **Daemon or service crash loops, or silent failure** Essential processes repeatedly restart (crash loops) or stop running without clear diagnostic logs. This condition results in partial visibility or service outages, such as a logging pipeline no longer forwarding events. Persistent crash behavior usually points to corruption or dependency failures that cannot be resolved by restarting services alone.  
    
* **Hardware-related issues** Physical components display recurring errors that degrade reliability. Examples include network interfaces entering port flapping cycles, disks generating input/output (IO) errors, or memory faults causing system instability. When hardware issues persist despite diagnostics or reconfiguration, restoration from backup on alternate hardware may be necessary.  
    
* **Corrupted system components** Critical files, services, or structures become unusable, breaking dependent operations. Examples include certificate corruption disrupting authentication, database corruption halting query responses, or dashboards failing to load due to index failures. Restoration is often the only path to returning corrupted components to a stable, functional state.

Recognizing when failures go beyond routine troubleshooting is essential for maintaining operational continuity. Common signs such as configuration drift, repeated service crashes, hardware instability, and component corruption indicate that intervention is needed. Escalating issues immediately helps reduce downtime, protect data integrity, and restore systems to a reliable baseline.

# \#-Platform-Specific Failure Indicators

**VMs**:  
**Attachments**:  
**ELOs**: 8.4.1-P – Demonstrate troubleshooting failed system components

In addition to common failure signs, each platform produces distinct symptoms and log messages indicating the malfunction's severity. These cues show how services, configurations, or hardware perform under stress and are often more accurate than general indicators. For example, corrupted lifecycle management settings in a logging platform suggest a broken pipeline that cannot be fixed through basic troubleshooting, and at the same time, repeated crash records on a network device point to deeper software or hardware issues. Administrators analyze these platform-specific outputs to determine whether the problem is limited to a single service or indicates a wider system disruption that needs restoration from a baseline configuration, snapshot, or full backup. By interpreting these unique signatures, administrators can avoid wasting time on ineffective solutions and restore systems to stable operation more efficiently.

The following chart organizes platform-specific failure indicators alongside the types of validation outputs administrators use to confirm them. It shows how each system reports faults differently, helping to distinguish between isolated service errors and conditions that warrant a full or partial restoration.

**Table: Platform-Specific Failure Indicators**

| Platform | Failure Indicator | Validation Tools |
| :---- | :---- | :---- |
| **Security Onion / ELK** | Missing or broken log pipelines | `journalctl`, `systemctl`, `/var/log/syslog`, Kibana logs |
|  | Corrupted Index Lifecycle Management (ILM) settings | `journalctl`, Elasticsearch logs |
|  | Snapshot errors or full-disk triggers | `df -h`, ILM error logs |
|  | Elasticsearch container not running | `systemctl status elasticsearch`, Docker logs |
|  | Kibana dashboards fail to load | Kibana error logs, browser console |
| **Zeek / Suricata** | Corrupted configs under `/opt/so/saltstack/local/salt/{zeek,suricata}/` | File inspection, Salt logs |
|  | Inject failures from missing tags or invalid SaltStack users | SaltStack logs, `journalctl` |
|  | Containers fail to start | `so-zeek-restart`, `so-suricata-restart`, `so-status` |
| **Kibana** | Visualization or dashboard loading errors | `journalctl -u kibana`, `systemctl status kibana` |
|  | Invalid network bindings | Config file review, `/var/log/syslog` |
|  | Incorrect config keys in `/opt/so/saltstack/local/salt/kibana/` | File inspection, Salt logs |
|  | Inject failures from non-existent keys (e.g., `network.host`) | Config validation, Kibana logs |
| **FreeIPA / FreeRADIUS** | Service errors due to expired certificates | `openssl x509 -enddate`, `journalctl` |
|  | Corrupted identity backend | FreeIPA/FreeRADIUS logs, `systemctl` |
|  | Misconfigured NTP or DNS | `dig`, `ntpq -p`, system logs |
| **pfSense** | Invalid NAT or firewall tables | *Diagnostics \> Logs* (Firewall), console logs |
|  | Corrupted DHCP settings | *Diagnostics \> Logs* (DHCP) |
|  | Stuck boot loops during startup | Console boot logs |
| **Cisco IOS** | Missing `startup-config` forcing setup mode | `show startup-config`, `show version` |
|  | IOS image corruption or checksum mismatch | `verify /md5 flash:IOS_image`, boot logs |
|  | Errors in flash: or crashinfo logs | `dir flash:`, `show logging` |
| **RedSeal** | Failed topology visualizations | RedSeal console logs |
|  | Broken import files | Import logs, system console |
|  | Database corruption blocking analysis | Database integrity checks, RedSeal logs |

**Summary**

Platform-specific indicators offer valuable insight into when normal remediation falls short. Administrators can differentiate between minor service glitches and systemic faults by identifying and confirming unique failure patterns through system outputs. This helps ensure that restoration efforts are necessary, reducing downtime and maintaining operational reliability.

# \#-Validating Configurations Against Baselines

**VMs**:  
**Attachments**:  
**ELOs**: 8.4.2-P – Execute troubleshooting for computer software issues

Maintaining alignment with known good baselines is essential for reliable operations. Over time, changes, updates, or corrupted files can cause systems to drift from their intended state, leading to complex troubleshooting failures. Validation ensures that current configurations match expected service states and that deviations are identified early. Automated tools like Ansible offer consistency across multiple nodes, while manual comparison methods allow administrators to inspect individual files in detail. By reviewing configurations against snapshots, baseline files, and default directories, administrators can quickly determine whether an error results from corruption, drift, or unauthorized modifications.

The following chart summarizes key methods for verifying system states against expected configurations. It includes automated methods, manual techniques, platform-specific issues, and fallback options in a quick-reference format.

**Table: Methods for Validating Configurations Against Baselines**

| Category | Method | Purpose | Examples / Notes |
| :---- | :---- | :---- | :---- |
| **Automated Validation** | Playbooks / `ansible-pull` | Confirm compliance of `.yml`, `.conf`, and container environment files with baselines | Enforce consistency across nodes |
|  | Baseline enforcement | Keep service states and parameters aligned with defined standards | Useful for distributed systems |
| **Manual Validation** | File comparison | Compare contents, timestamps, permissions | `diff`, `cat`, `ls -lt`, `stat` |
|  | Snapshot cross-referencing | Review earlier `.sls`, `.conf`, `.yml` versions for known-good states | Use to detect drift or corruption |
| **Platform Baseline Issues** | Zeek | Invalid tag entries in `init.sls` block service loading | Check `/opt/so/saltstack/local/salt/zeek/init.sls` |
|  | Suricata | References non-existent users in `init.sls` stopping container startup | Check `/opt/so/saltstack/local/salt/suricata/init.sls` |
|  | Kibana | Invalid network references in `init.sls` halting dashboards | Check `/opt/so/saltstack/local/salt/kibana/init.sls` |
| **Container Validation** | Status checks | Confirm expected services are running | Review process and service logs |
| **Fallback** | Default file restoration | Overwrite broken local customizations with known-good defaults | Use `/opt/so/saltstack/default/salt/...` |

Comparing current system states with known good configurations to detect drift, corruption, or unauthorized changes ensures systems stay aligned with expected baselines. Automated methods offer consistency across environments, while manual checks enable targeted inspection of individual files. Platform-specific issues highlight where errors frequently occur, and fallback options maintain reliable recovery paths. Together, these practices sustain stability and reliability in systems.

# \#-Docker Fundamentals

**VMs**:  
**Attachments**:  
**ELOs**: 8.4.2-P – Execute troubleshooting for computer software issues

Docker is a containerization platform that packages applications and their dependencies into lightweight, portable units called containers. Containers run independently of the host operating system and enable multiple services to operate in isolation on the same system. This design simplifies deployment, enhances scalability, and reduces conflicts between applications. Understanding Docker’s concepts and lifecycle is crucial for identifying failures and applying troubleshooting steps in general IT environments or specialized systems such as distributed monitoring and security platforms.

**Docker Lifecycle**

Docker organizes applications into containers containing the software, dependencies, and configuration required to run a service. By isolating these elements, Docker reduces the risk of conflicts between applications on the same host and enables faster deployment.

The container lifecycle follows a predictable sequence:

* **Creation:** A container is instantiated from an image but has not yet started.  
* **Start:** The container is launched and transitions into an active state.  
* **Running:** The container executes its intended service or application.  
* **Stopped:** The container halts execution but retains its configuration and data.  
* **Removed:** The container and its state are deleted from the system.

Understanding these stages allows administrators to trace where a failure occurs, such as a container that never transitions from “start” to “running” due to misconfiguration or resource constraints.

**Container Failure**

Containerized services depend on well-structured configurations, stable system resources, and trusted images. When one of these elements fails, containers either refuse to start, terminate prematurely, or behave inconsistently. Recognizing the main causes of failure helps in faster isolation and resolution.

* **Bad configuration files:** Service containers depend on external configuration files for settings like network bindings, log paths, and memory allocations. If these files have errors, incorrect syntax, or reference unavailable resources, containers may fail during startup. Misaligned parameters can lead to repeated restarts or poor performance, even if a container starts. Version mismatches between configuration files and container images can cause subtle failures that are hard to detect without proper validation.  
* **Resource exhaustion:** Containers share host system resources. If memory, CPU, or storage limits are reached, containerized services may stop responding, restart in a crash loop, or fail scheduled health checks. This is common when multiple high-demand services run simultaneously, such as log ingestion pipelines and intrusion detection. Resource exhaustion can also happen gradually due to memory leaks, persistent log growth, or unmonitored background tasks.  
* **Image corruption:** Images form the foundation for services by storing binaries, dependencies, and baseline configurations. They can prevent containers from starting if an image download is incomplete, the registry source is unreliable, or the storage medium is damaged or corrupted. Inconsistent image versions across hosts also cause irregular behavior, making diagnosis more difficult. Verification methods like checksum validation and maintaining a trusted local registry help reduce the risk of corruption.

**Summary:**

Container failure stems from faulty configuration files, insufficient resources, or corrupt images. Each cause exhibits unique diagnostic patterns: configuration errors appear in logs during startup, resource shortages cause services to stall or loop, and corrupted images block proper initialization. Solving these issues through validation, monitoring, and integrity checks restores service continuity.

# \#-**Docker Commands for Container Inspection**

**VMs**:  
**Attachments**:  
**ELOs**: 8.4.2-P – Execute troubleshooting for computer software issues

Docker’s CLI commands provide direct access to the state and behavior of containers. These commands support inspecting running services, verifying configurations, and detecting failures. When multiple containerized services run simultaneously, CLI tools are essential for checking service status, ensuring alignment with baseline settings, and diagnosing performance or configuration issues.

The chart below organizes key Docker CLI commands by purpose, use case, and diagnostic focus to guide systematic troubleshooting:

**Table: Docker CLI Commands for Container Inspection**

| Command | Purpose | Use Case | Diagnostic Focus |
| :---- | :---- | :---- | :---- |
| `docker ps` | Lists running containers | Confirm service activity and uptime | Service availability |
| `docker ps -a` | Lists all containers, including stopped | Identify failures or repeated exits | Startup or crash loops |
| `docker logs <container>` | Displays container log output | Detect misconfigurations or startup errors | Configuration and runtime errors |
| `docker inspect <container>` | Shows detailed metadata | Verify configuration, networking, and resource settings | Misconfigurations or mismatched parameters |
| `docker stats` | Monitors live resource usage | Detect CPU, memory, or I/O exhaustion | Resource exhaustion |
| `docker top <container>` | Lists running processes inside a container | Confirm service execution within container | Process validation |
| `docker events` | Streams real-time Docker events | Track container creation, restarts, or failures | Lifecycle tracking |
| `docker exec -it <container> /bin/bash` | Opens interactive shell | Perform deeper manual inspection or adjustments | In-depth diagnosis |

**Summary**

CLI commands provide a structured set of tools for assessing container health. By combining log inspection, resource monitoring, and configuration verification, administrators isolate failure causes and confirm that containerized services operate as intended.

# \#-Knowledge Check

**VMs**:  
**Attachments**:  
**ELO**: 	8.4.1-P – Demonstrate troubleshooting failed system components  
8.4.2-P – Execute troubleshooting for computer software issues

## Question:

# What is a common indicator that a system restoration may be required in Security Onion or ELK? (Multiple Choice)

**Choose directive based on question type:**  
Multiple choice (retries \- 0\) \- (Select the correct answer.)  
Selection (retries \- half of the total number of answer options, rounded down) \- (Select all that apply.)  
Short answer (retries \- 2\) \- (*Specify directive.*)

**Bold correct answer(s):**  
**ANSWER 1: Corrupted ILM settings**

ANSWER 2: Misplaced log settings

ANSWER 3: Broken file settings

ANSWER 4: Expired user tokens

**List correct answer number(s)**:

# NC-\#-SaltStack and Container Troubleshooting

**VMs**: **Attachments**: **ELOs**: ELO 8.4.1-P DEMONSTRATE troubleshooting failed system components ELO 8.4.2-P EXECUTE troubleshooting for computer software issues

Security Onion uses SaltStack to manage containerized services. Understanding this relationship is essential for effective troubleshooting when containers fail to start. SaltStack acts as the configuration management layer between administrator-defined settings and the Docker containers that run Security Onion services.

## SaltStack Architecture in Security Onion

SaltStack manages Security Onion through two primary mechanisms: **states** and **pillar data**. States define what services run and how they are configured. Pillar data provides the configuration values that states use when deploying containers.

*Table 8.4-01: SaltStack Components for Container Management*

| Component | Location | Purpose |
| :---- | :---- | :---- |
| States | `/opt/so/saltstack/default/salt/` | Define service configurations and container deployments |
| Local Overrides | `/opt/so/saltstack/local/salt/` | Site-specific customizations |
| Pillar Data | `/opt/so/saltstack/local/pillar/` | Configuration values (memory, ports, features) |
| Minion Pillar | `/opt/so/saltstack/local/pillar/minions/` | Node-specific configuration overrides |

## Pillar Data as Configuration Source

Pillar data contains the configuration values that control how containers are deployed. These values include memory allocations, network ports, feature flags, and service-specific settings. When pillar values are invalid or misconfigured, the affected container fails to start. Security Onion reads pillar data each time a container is created or restarted, making pillar files the authoritative source for container configuration.

## Container Lifecycle and Salt

When a Security Onion container fails to start, Salt handles the failure by cleaning up the failed container. Security Onion provides many utility scripts in the path with the "so-" prefix including restart commands like `so-kibana-restart`. Running a restart command causes Salt to read the current pillar configuration, remove any existing failed container, recreate the container with the pillar values, and start the new container. This process makes restart commands useful for troubleshooting because they force Salt to recreate the container and capture fresh logs in the process.

## Log Locations for Troubleshooting

When diagnosing container failures, administrators check multiple log sources. Docker logs capture container startup errors and application messages. The Salt minion log records state application errors. The `so-status` command provides a container health overview, and `salt-call pillar.get` queries current configuration values.

*Table 8.4-02: Log Sources for Container Troubleshooting*

| Log Source | Command | Information Provided |
| :---- | :---- | :---- |
| Docker logs | `docker logs <container>` | Container startup errors, application messages |
| Salt minion log | `/opt/so/log/salt/minion` | Salt state application errors |
| so-status | `so-status` | Container health overview |
| Pillar values | `salt-call pillar.get <path>` | Current configuration values |

## Troubleshooting Workflow

When a container shows `missing` status in `so-status`, a systematic workflow identifies the root cause. The administrator first checks `so-status` to identify which container is failing. Attempting a restart using the appropriate `so-<service>-restart` command recreates the container and captures logs. While the restart runs, checking docker logs reveals startup errors. If the error indicates a configuration problem, querying the pillar values confirms whether the configuration is valid. After fixing any pillar misconfiguration, the administrator completes or reruns the restart to verify the fix. This workflow ensures that errors are captured in docker logs and that configuration issues are traced back to their pillar source.

---

# NC-\#-Diagnose Security Onion Container Status

**VMs**: cptoperations **Attachments**: **ELOs**: ELO 8.4.1-P DEMONSTRATE troubleshooting failed system components ELO 8.4.2-P EXECUTE troubleshooting for computer software issues

Security Onion uses containerized services managed by SaltStack. When containers fail to start, so-status provides immediate visibility into service health. The diagnostic workflow uses container restarts, docker logs, and pillar checks to identify the root cause.

The following systems are used in this workflow:

* **cptoperations** (*199.63.64.17*) \- Console (workstation)  
* **onion-m** (*199.63.64.10*) \- SSH from cptoperations

## Workflow

1\. Log in to **cptoperations** using the username and password that follows:

`trainee` `CyberTraining1!`

2\. Open a terminal and connect to **onion-m** by entering the command that follows:

`ssh trainee@199.63.64.10`

3\. Enter the password when prompted:

`CyberTraining1!`

4\. Check container status by entering the command that follows:

`sudo so-status`

5\. Enter the password when prompted for sudo authentication:

`CyberTraining1!`

The output displays the status of all Security Onion containers. If it shows  `⌛ System appears to be starting. No highstate has completed since the system was restarted`, wait 5 to 10 more minutes for the system to start.

Notice that `so-elasticsearch` shows `missing` status, indicating the container cannot start. When a container shows `missing`, it means the container failed to start and Salt cleaned up the failed container.

Depending on how long the system has been starting, the `so-elastalert` container may also be missing.

```
                       Security Onion Status
                         Container | Status  |             Details
-----------------------------------+---------+---------------------
                 so-dockerregistry | running |           Up 3 days
                     so-elastalert | running |           Up 3 days
                  so-elastic-fleet | running |           Up 3 days
 so-elastic-fleet-package-registry | running | Up 3 days (healthy)
                  so-elasticsearch | missing |
                       so-idstools | running |           Up 3 days
                       so-influxdb | running | Up 3 days (healthy)
                         so-kibana | running |           Up 3 days
                         so-kratos | running |           Up 3 days
                       so-logstash | running |           Up 3 days
                          so-nginx | running | Up 3 days (healthy)
                          so-redis | running |           Up 3 days
                      so-sensoroni | running |           Up 3 days
                            so-soc | running |           Up 3 days
                       so-telegraf | running |           Up 3 days

 ! Check container logs and /opt/so/log for more details.
```

6\. When a container is missing, attempting a restart will recreate it and capture logs. Begin the Elasticsearch restart by entering the command that follows:

`sudo so-elasticsearch-restart`

This command takes approximately 5 minutes to complete. While it runs, open a second terminal to check the container logs.

7\. Open a second terminal on **cptoperations** and connect to **onion-m** by entering the command that follows:

`ssh trainee@199.63.64.10`

8\. Enter the password when prompted:

`CyberTraining1!`

9\. In the second terminal, check the Elasticsearch container logs by entering the command that follows:

`sudo docker logs so-elasticsearch 2>&1 | tail -20`

10\. Enter the password when prompted for sudo authentication:

`CyberTraining1!`

If the output shows `No such container: so-elasticsearch`, wait 30-60 seconds for Salt to recreate the container, then retry the command.

The output displays the Elasticsearch startup error:

```
Exception in thread "main" java.lang.RuntimeException: starting java failed with [1]
output:
Error occurred during initialization of VM
Too small maximum heap
error:
```

The error message `Too small maximum heap` reveals the problem. The JVM heap size value is invalid.

11\. The error indicates a configuration problem. Security Onion stores container configuration in SaltStack pillar data. Check the pillar value by entering the command that follows:

`sudo salt-call pillar.get elasticsearch:esheap`

The output displays the current esheap value:

```
local:
    4096
```

The `esheap` pillar value `4096` is missing the required unit suffix. Without the suffix, `4096` is interpreted as 4096 bytes instead of megabytes, which is far too small for the JVM heap. Valid heap sizes require a unit such as `m` for megabytes or `g` for gigabytes (for example, `4096m` or `4g`). This invalid value is passed to the JVM, causing Elasticsearch to fail on startup.

---

# \#-Resolve Elasticsearch Container Failure

**VMs**: cptoperations **Attachments**: **ELOs**: ELO 8.4.2-P EXECUTE troubleshooting for computer software issues

The error `Too small maximum heap` indicates a JVM configuration problem. Security Onion stores container configuration in SaltStack pillar data. The pillar check confirmed the esheap value is `4096` without the required unit suffix. Students will fix the configuration and verify the service starts successfully.

The following systems are used in this workflow:

* **cptoperations** (*199.63.64.17*) \- Console (workstation)  
* **onion-m** (*199.63.64.10*) \- SSH from cptoperations

## Workflow

1\. If not already connected, open a terminal on **cptoperations** and connect to **onion-m** by entering the command that follows:

`ssh trainee@199.63.64.10`

2\. Enter the password when prompted:

`CyberTraining1!`

3\. The pillar configuration is stored in `/opt/so/saltstack/local/pillar/minions/`. Fix the missing unit suffix by entering the command that follows:

`sudo sed -i "s/esheap: '4096'/esheap: '4096m'/" /opt/so/saltstack/local/pillar/minions/onion-m_managersearch.sls`

4\. Enter the password when prompted for sudo authentication:

`CyberTraining1!`

This command adds the `m` suffix for megabytes to the heap value.

5\. Verify the fix was applied by entering the command that follows:

`sudo salt-call pillar.get elasticsearch:esheap`

The output should now show the correct value with the unit suffix:

```
local:
    4096m
```

6\. The restart from the diagnostic step may have completed. If Elasticsearch is still not running, restart it by entering the command that follows:

`sudo so-elasticsearch-restart`

This command may take 1-2 minutes to complete as Elasticsearch initializes.

7\. Verify Elasticsearch is running by entering the command that follows:

`sudo so-status | grep elasticsearch`

The output shows `so-elasticsearch` with `running` status:

```
                  so-elasticsearch | running |        Up 1 minute
```

Depending on the earlier deployment state, the so-elastalert container may still be missing. If so, also run  
`sudo so-elastalert-restart --force`

`so-status` shows `✔ This onion is ready to make your adversaries cry!`

---

# \#-Verify Security Onion Web Interface

**VMs**: cptoperations **Attachments**: **ELOs**: ELO 8.4.2-P EXECUTE troubleshooting for computer software issues

With Elasticsearch now running, the Security Onion Console (SOC) and Kibana dashboards can connect to the cluster. Verifying web interface access confirms that the troubleshooting workflow restored full functionality.

The following systems are used in this workflow:

* **cptoperations** (*199.63.64.17*) \- Console (workstation)  
* **onion-m** (*199.63.64.10*) \- Web interface from cptoperations

## Workflow

1\. On **cptoperations**, open Firefox and navigate to the Security Onion Console by entering the URL that follows:

`https://199.63.64.10`

2\. Accept the certificate warning by selecting **Advanced** and then **Accept the Risk and Continue**.

The certificate warning appears because Security Onion uses a self-signed certificate.

3\. Log in to the Security Onion Console using the username and password that follows:

`trainee@dmss.lan` `CyberTraining1!`

The SOC dashboard loads, displaying the main navigation menu.

4\. Select **Kibana** to open the Kibana interface.

Kibana loads successfully, confirming that the Elasticsearch backend is operational. The dashboards display without errors, indicating that the pillar fix restored full functionality.

---

# \#-Knowledge Check

**VMs**: cptoperations **Attachments**: **Retries**: 2 **ELO**: 8.4.1-P – Demonstrate troubleshooting failed system components

## Question:

# Which onion-m service failed to start? (Short Answer)

(Enter the service name)

**Answer**: elasticsearch

# \#-Knowledge Check

**VMs**: cptoperations **Attachments**: **Retries**: 2 **ELO**: 8.4.2-P – Execute troubleshooting for computer software issues

## Question:

# What was the correct configuration setting to enable Elasticsearch to start? (Short Answer)

(Enter the correct esheap value)

**Answer**: 4096m

# \#-Identify Switch Hardware Faults

**VMs**:  
**Attachments**:  
**ELOs**: ELO 8.4.3-P – Execute troubleshooting for computer hardware issues

Switch-level misconfigurations at layers two and three directly affect VLAN communication, STP stability, and overall network visibility in DMSS environments. These problems usually emerge as intermittent connectivity failures, routing inconsistencies, or degraded performance across segments. Expanding each issue helps illustrate how minor errors in configuration create operational impacts:

* **Misconfigured STP priorities can cause issues:** STP chooses which switch acts as the root bridge, forming the logical center of the layer two network. If priorities are not set intentionally, the election defaults to the lowest bridge ID. This can lead to an edge or access-layer switch becoming the root, resulting in inefficient traffic paths, too many blocked ports, and slower convergence during topology changes. Ongoing instability at the STP root can also cause broadcast storms and widespread network disruptions.  
  **Figure: STP Root Bridge Election and Port Roles**

```
                         ┌─────────────────────┐
                         │    ROOT BRIDGE      │
                         │ Priority 4096       │
                         │   (configured)      │
                         │  ┌──────┬──────┐    │
                         │  │  DP  │  DP  │    │
                         └──┴──┬───┴───┬──┴────┘
                               │       │
                    ┌──────────┘       └──────────┐
                    │                             │
             ┌──────┴──────┐               ┌──────┴──────┐
             │     RP      │               │     RP      │
             ├─────────────┤               ├─────────────┤
             │             │               │             │
             │  ┌────┬────┐│               │┌────┬────┐  │
             │  │BLK │ DP ││───────────────││DP  │    │  │
             │  └────┴────┘│               │└────┴────┘  │
             │  Priority   │               │  Priority   │
             │   32768     │               │   32768     │
             │  (default)  │               │  (default)  │
             └─────────────┘               └─────────────┘

    LEGEND:  DP = Designated Port    RP = Root Port    BLK = Blocked Port
```

*STP elects the switch with the lowest bridge ID as the Root Bridge. The root bridge has all Designated Ports (DP). Non-root switches have one Root Port (RP) pointing toward the root. Redundant paths are blocked (BLK) to prevent loops.*

* **VLAN inconsistencies across trunks:** Trunk ports carry multiple VLANs across inter-switch links. If VLAN lists or tagging configurations differ between trunk endpoints, some VLANs fail to pass traffic while others succeed. This selective visibility causes incomplete monitoring, broken inter-VLAN communication, and missing data in security platforms. In DMSS, mismatched VLAN trunks can cause one sensor to see only partial traffic flows, leading to gaps in analysis.  
* **Duplex mismatches cause collisions or retransmissions:** A duplex mismatch happens when one end of a link is set to half duplex and the other to full duplex. This leads to repeated collisions, late collision errors, and CRC failures that reduce throughput. These mismatches also cause unnecessary retransmissions, wasting bandwidth and creating blind spots in monitoring. Even a single mismatched interface can disrupt the bundle and decrease stability in port channels.  
  **Figure: Duplex Mismatch Scenarios**

```
    Manual Settings, Autonegotiation Disabled
    ─────────────────────────────────────────

       ┌───┐           ┌───┐           ┌───┐
       │ 1 │           │ 2 │           │ 3 │
       │10/│           │10/│           │10/│
       │100│           │100│           │100│
       └─┬─┘           └─┬─┘           └─┬─┘
         │               │               │
    Settings:       Settings:       Settings:
    100 Full        1000 Full       10 Half
         │               │               │
         │               │               │
    ┌────┴───────────────┴───────────────┴────┐
    │  F0/1          F0/2            F0/3     │
    │                                         │
    │  Result:       Result:         Result:  │
    │  100 Half      1000 Full       10 Half  │
    │                                         │
    │  Autonegotiation Enabled, 10/100/1000   │
    └─────────────────────────────────────────┘

    When one side disables autonegotiation and the other enables it:
    - Speed is detected correctly
    - Duplex defaults to HALF on the autonegotiating side
    - Result: Duplex MISMATCH causing collisions and CRC errors
```

*A duplex mismatch occurs when endpoints negotiate different duplex settings. This causes late collisions, CRC errors, and significant performance degradation.*

* **Mismatched encapsulation settings:** VLAN traffic on trunk ports requires a consistent encapsulation method, usually IEEE 802.1Q (dot1q). If one side uses dot1q and the other uses Inter-Switch Link (ISL) or has no explicit configuration, VLAN tags may not be interpreted correctly. This can cause dropped frames, orphaned VLANs, or non-functional trunk links. Encapsulation mismatches are often overlooked but are essential for maintaining uniform VLAN propagation across distributed monitoring systems.

**Summary**

Switch-level hardware faults often originate from Layer 2 and Layer 3 configuration errors. Misconfigured STP priorities destabilize network topologies by selecting incorrect root bridges, while VLAN inconsistencies across trunks disrupt inter-switch communication. Duplex mismatches cause CRC errors, collisions, and retransmissions that diminish performance, and mismatched encapsulation settings hinder proper VLAN tagging. Recognizing these issues offers a solid basis for diagnosing and fixing hardware-related faults in DMSS switching environments.

# \#-Cisco Switch Diagnostic Commands

**VMs**:  
**Attachments**:  
**ELOs**: ELO 8.4.3-P – Execute troubleshooting for computer hardware issues

Troubleshooting hardware faults in switching environments depends on reliable diagnostic visibility. Cisco switches include commands that reveal how VLANs, STP, interfaces, and port channels are configured and performing. These commands allow administrators to confirm baseline alignment, detect mismatches, and identify root causes of degraded performance.

The chart below organizes the key commands, their purpose, everyday use cases, and the types of faults they help diagnose:

**Table: Cisco Switch Diagnostic Commands**

| Command | Purpose | Use Case | Diagnostic Focus |
| :---- | :---- | :---- | :---- |
| `show spanning-tree` | Displays STP status, priorities, and root bridge | Verify correct root bridge and confirm convergence | STP loops, root misplacement, slow reconvergence |
| `show interfaces` | Shows interface details including speed, duplex, and errors | Detect collisions, CRC errors, or duplex mismatches | Duplex mismatch, physical layer faults |
| `show vlan brief` | Lists VLANs and associated ports | Confirm VLAN presence and consistency across trunks | VLAN mismatches, missing VLAN assignments |
| `show etherchannel summary` | Summarizes port-channel configuration and status | Validate consistency of bundled links | Port-channel misconfiguration, inconsistent member interfaces |

**Summary** Cisco diagnostic commands offer structured ways to verify switch health. Each command targets a specific area, such as STP, VLANs, interfaces, or port channels, helping administrators identify faults and ensure network behavior aligns with planned configurations.

# \#-Knowledge Check

**VMs**: **Attachments**: **Retries**: 0 **ELO**: 8.4.3-P – Execute troubleshooting for computer hardware issues

## Question:

# A network administrator observes late collisions and CRC errors on a switch interface. Which condition is most likely causing these symptoms? (Multiple Choice)

(Select the correct answer.)

ANSWER 1: Misconfigured STP priority

**ANSWER 2: Duplex mismatch**

ANSWER 3: VLAN tagging inconsistency

ANSWER 4: Trunk encapsulation mismatch

**List correct answer number(s)**: 2

---

# \#-Knowledge Check

**VMs**: **Attachments**: **Retries**: 0 **ELO**: 8.4.3-P – Execute troubleshooting for computer hardware issues

## Question:

# Which Cisco IOS command displays the current root bridge and port roles for Spanning Tree Protocol? (Multiple Choice)

(Select the correct answer.)

ANSWER 1: show interfaces

ANSWER 2: show vlan brief

**ANSWER 3: show spanning-tree**

ANSWER 4: show etherchannel summary

**List correct answer number(s)**: 3

---

# NC \- \# \- Conclusion

**Course Banner Image**

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Network     │                                                           │
│  Technician  │     Troubleshooting with Backups                          │
│              │                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

*Purple/orange gradient banner with course title*

Effective troubleshooting of DMSS components combines systematic diagnostic workflows with targeted configuration corrections. Container failures in Security Onion are resolved by checking service status, examining Docker logs for startup errors, tracing misconfigurations to SaltStack pillar data, and verifying fixes through restart commands. Switch-level faults such as STP misconfigurations, VLAN inconsistencies, and duplex mismatches are identified through Cisco IOS diagnostic commands and corrected by aligning configurations with operational baselines. Together, these methods restore visibility, maintain operational readiness, and ensure reliable data collection across distributed monitoring and security platforms.

**Module Art**

*Illustration depicting system troubleshooting and recovery operations. The scene includes:*

- *Docker container icons showing service states*  
- *Database backup symbols with restore arrows*  
- *Network switch with port status indicators*  
- *Terminal window displaying log output*  
- *Configuration file icons (YAML, SLS)*  
- *SaltStack pillar hierarchy representation*

# Assessment Questions (As Reference)

(For NT, use the [**P-ELO Assessment Index**](https://docs.google.com/spreadsheets/d/1m7VtvbKf0sGDjno012a3ub_EzMKzgXB6khjlwkIC4NQ/edit?gid=920500347#gid=920500347) to find and copy in the assessment questions for the ELOs listed here from the PTs and Capstones addressing each ELO. Refer to these questions while developing the lesson.)

# PT\#/CAP-M\#.Q.\#

**ELOs:** x.x.x P…  


[image1]: <data:image/png;base64,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>
