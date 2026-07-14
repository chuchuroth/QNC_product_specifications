Document Number: QNC-ORG-001  
Version: Draft 1.0  
Status: Derived companion guide for operations, deployment, and recovery  
Classification: Internal / Project Controlled

0. Purpose and Scope

This guide describes the operational procedures for deployment, configuration, monitoring, fault handling, restart, and Safe Mode operation of the QNC. It applies to baseline deployments as well as approved extensions, in particular FastDDS-based robotics integration, without altering the system boundaries defined in the source documents. The QNC remains not safety-certified; Safe Mode is an operational and fault-management function and does not replace functional safety.  

Procedures for Deployment, Configuration, and Fault Management.

0.1 Operating Principles

The following five core principles are mandatory during operation:  

No partially activated artifacts  

Only validated and compatible profiles, schemas, and QoS artifacts may be activated  

Errors must be isolated by state domain  

Critical errors must transition to Safe Mode within 1 second  

In the event of a failed change, immediately roll back to the last known good state   

1. Configuration Procedures

1.1 Configuration Artifacts and Responsibilities

QNC operational configuration must be divided into separate, version-controlled artifacts. System-wide network, security, and interface settings belong in the System Configuration. Device-specific mapping of commands, parameters, telemetry, errors, and lifecycle sequences belongs in the Device Profile. DDS-specific type definitions and QoS remain outside the Device Profile boundary and are managed via the IDL Type Registry and QoS Profile Catalog. Operation must not mix these boundaries.   

Recommended Artifact Structure

Artifact

Purpose

Owner

Activation Rule

system-config       

Network, security, enabled interfaces, lifecycle policy                 

Operations / Platform          

Only atomic                                          

device-profile      

Device class, protocol binding, commands, parameters, telemetry, fault mapping, lifecycle sequences 

Systems / Device Integration   

Only after schema, integrity, and compatibility check 

dds-idl-registry    

Topic types and versions                                                

Robotics / Middleware          

Only approved types                                  

dds-qos-catalog     

QoS profiles per communication class                                    

Robotics / Middleware / Operations 

Only approved QoS profiles                           

audit-policy        

Logging, retention, change logging                                      

Operations / Security          

Mandatory artifact                                   

The above separation is derived directly from PRS, DPS, ICD, and PSM. 

1.2 Step-by-Step Procedure for Initial Commissioning

Step 1: Check Release Status

Before every installation, confirm that the deployed protocols, bridges, and DDS functions belong to the approved release level. Baseline protocols are IO-Link, Modbus RTU, EtherNet/IP Adapter, and Discrete Digital I/O; FastDDS, MQTT, OPC UA, and other protocols are permitted only in approved extensions.  

Step 2: Establish Hardware and Network Prerequisites

Power supply, cabling, port assignment, IP concept, VLAN rules, and access to northbound and southbound segments must be defined prior to the configuration assignment. For DDS, additionally verify whether multicast is permitted in the target segment or whether a Discovery Server topology is required.   

Step 3: Load System Configuration

Network parameters, TLS/token policy, enabled services, Safe Mode policy, logging/audit settings, and permitted interfaces are prepared as a system artifact. This change must be applied as a single transaction; partial activation is prohibited.  

Step 4: Assign Device Profile

For each field device, assign an approved YAML profile with a matching identity and match policy. The profile must declare exactly one device class, define the southbound binding, and contain all required commands, parameter limits, telemetry, and fault mappings. 

Step 5: Register DDS Artifacts Separately

If FastDDS is enabled, Domain ID, Participant Name, discovery mode, permitted interfaces, IDL types, and QoS profiles are registered separately. DDS artifacts must not be hidden or duplicated inside Device Profiles.  

Step 6: Perform Pre-Activation Validation

Before commit, at minimum schema validation, signature/integrity check, compatibility check, profile-protocol reconciliation, and Safe Mode/fault mapping verification must be completed successfully. Faulty artifacts are discarded; the system remains in the last known good state.  

Step 7: Atomic Activation

Configuration changes are activated exclusively in an atomic manner. During the switch, no mixed state of old and new configuration may be observable. If any part of the activation fails, an automatic rollback occurs.  

Step 8: Initialization and Device Start

After successful activation, the QNC lifecycle sequence runs from BOOTING through INITIALIZING to READY and ACTIVE. On the profile side, the initialization_sequence and, if applicable, the enable_sequence are executed.  

Step 9: Functional Verification

At the conclusion of initial commissioning, telemetry, fault/event channel, command acceptance, status messages, Safe Mode inhibit, and audit logging must be verified. For DDS deployments, discovery, QoS, and topic compatibility checks are added.  

1.3 Parameter Definitions and Recommended Configuration Domains

System-Wide

Network configuration: static or DHCP, VLAN if required  

Security: TLS 1.2+, token-based authentication, RBAC  

Audit/Logging: JSON logs, rotation, retention  

Lifecycle Policy: recovery mode manual/auto, Service Mode permitted/not permitted  

Safe Mode Policy: actuation lock mandatory, DDS optional subscribe-only   

Profile-Related

device_identity  

protocol_binding with timeouts/retry classes  

capabilities  

commands  

parameters with min/max, type, unit, scaling  

telemetry  

faults including severity, latching, safe_mode_required  

lifecycle_sequences including recover_sequence and shutdown_sequence 

DDS-Related

domain_id  

participant_name  

discovery_mode (SIMPLE or DISCOVERY_SERVER)  

allowed_interface_list  

ignore_nonapproved_peers  

lease_duration_ms  

announcement_period_ms  

Topic type version and assigned QoS profile 

1.4 Validation and Verification

Validation Step

Objective

Result on Failure

Schema validation            

Structure correct                      

Block activation                      

Signature/hash verification  

Integrity ensured                      

Reject artifact, log security fault   

Compatibility check          

Firmware, profile, DDS version match   

Deny activation                       

Parameter range check        

No impermissible setpoints             

Reject command or profile             

Safe Mode check              

Critical faults correctly mapped       

Block release                         

Communication check          

Link/session/discovery stable          

Remain in READY or DEGRADED       

Audit check                  

Change traceable                       

Stop rollout                          

These checks are derived from the governance and conformance rules required in PRS/DPS/ICD/PSM. 

2. DDS Domain Configuration Guidance

2.1 DDS Architecture Principle

DDS is an approved extension function in the QNC and is operated by its own DDS Participant Service. Domain configuration is managed independently of Device Profiles. DDS errors must be detected and isolated as DDS Domain Faults; they must not propagate to southbound services.  

2.2 Recommended Domain ID Strategy

The source documents require a controlled Domain ID assignment but do not specify a concrete numbering policy. The following operational standard is therefore recommended:

Range

Purpose

Rule

0–31       

Laboratory / development         

Not for production                                

32–95      

Single cell / standalone         

One cell = one domain                             

96–159     

Production line / area           

One line = one domain                             

160–191    

Integration / acceptance test    

Time-limited                                      

192–223    

DR / recovery / fallback         

Activate only under controlled conditions         

224–232    

Reserved for central middleware services 

Platform approval only                            

233–255    

Locked / special cases           

Do not use without architecture approval          

This partitioning is a derived operational policy for scaling, isolation, and avoidance of unplanned collisions; it remains compatible with the document set because Domain IDs are treated as governed configuration and not as part of the device profile.  

2.3 Selecting Discovery Mode

SIMPLE Discovery is the default for small, flat network segments with multicast support and a limited number of participants. DISCOVERY_SERVER should be used for segmented, larger, or cross-network deployments, especially when multicast is restricted or stricter participant control is required. Baseline is SIMPLE; Discovery Server is considered an advanced deployment option.  

2.4 Participant Configuration

Each participant receives a stable name, an explicit Domain ID, a defined discovery policy, and a restricted network view. The use of allowed_interface_list and ignoring non-approved peers is recommended to reduce discovery noise and allow only approved communication paths. 

2.5 QoS Standards

QoS profiles are assigned normatively per data class:

QoS Profile

Use Case

QOS_CMD_STRICT    

Command request/response                      

QOS_STATUS_FAST   

Lifecycle and status messages                 

QOS_TELEM_STREAM  

High-rate telemetry                           

QOS_TELEM_CRITICAL

Critical health data                          

QOS_FAULT_EVENT   

Faults/warnings, relevant for late joiners    

QOS_CONFIG_AUDIT  

Configuration and audit events                

QOS_DIAG_BULK     

Diagnostics/tracing                           

Offer/request mismatches must not be silently accepted but recorded as structured DDS faults. 

2.6 Network and Interoperability Rules

DDS topics must be organized by function (e.g., Command Ingress, Status Egress, Fault Event, Diagnostics). Normalized data should be produced once and published multiple times rather than recalculated per consumer. Only validated IDL versions and approved QoS combinations may be active per domain.  

2.7 Safe Mode and DDS

In Safe Mode, DDS may remain active at most in subscribe-only behavior if this is approved. Any actuation published or consumed via DDS must be blocked in this state. Status, fault, and health traffic may remain active to enable diagnosis and coordinated recovery.   

DDS Reference Diagram

          +-----------------------------+
          |   Northbound Edge Services  |
          | REST / WS / Logs / Bridges  |
          +-------------+---------------+
                        |
          +-------------v---------------+
          |   Integration & Control     |
          | Command Broker / Fault Mgr  |
          | Lifecycle / State Aggregate |
          +-------------+---------------+
                        |
      +-----------------+------------------+
      |                                    |
+-----v----------------+      +------------v-----------+
| Profile & Semantic   |      | DDS Participant Service|
| Normalization        |      | Domain / Discovery /   |
| Profiles / Mapping   |      | QoS / IDL Validation   |
+-----+----------------+      +------------+-----------+
      |                                       
+-----v-------------------------------+
| Physical & Protocol Adaptation      |
| IO-Link / Modbus / EIP / DIO        |
+-------------------------------------+

The separation corresponds directly to the four-layer architecture and the separate governance of profiles and DDS artifacts.   

3. Deployment Mode Selection Criteria

3.1 Available Deployment Modes

Mode

Description

Suitable for

Standalone Device Gateway   

Single QNC with baseline protocols and standard northbound interfaces 

Single cell, commissioning, smaller machines      

Distributed Edge Node       

Multiple protocol services, optional bridges             

Line operation, segmentation, load distribution   

Edge + DDS Peer             

QNC actively participates in robotics DDS domain         

ROS 2 / FastDDS integration                       

Cloud-Connected Edge        

Forwarding to higher levels via approved services        

Telemetry, fleet oversight, remote diagnostics    

Advanced Multi-Network DDS  

Larger environments with Discovery Server                

Segmented plants, multiple networks               

These modes are derived from PSM and PRS.  

3.2 Selection Criteria

Standalone Device Gateway

Preferred when low complexity, local control, few devices, and low middleware dependency are priorities. This mode minimizes fault interfaces and simplifies recovery. 

Distributed Edge Node

Suitable when protocol diversity, segmentation, higher availability of sub-functions, and organizational separation of device domains are required. Fault isolation is better here, but configuration management is more demanding.  

Edge + DDS Peer

Select only when robotics workloads require genuine DDS participation (i.e., topic-based data exchange, standardized IDL types, and coordinated QoS profiles). The operational benefit is interoperability; the cost is higher discovery, QoS, and versioning effort.  

Cloud-Connected Edge

Appropriate for observability, historization, central diagnostics, and operational data analysis. This mode should not be prioritized for hard local real-time decisions. 

Advanced Multi-Network DDS

Deploy only with architecture approval when domain segmentation, Discovery Server control, and larger numbers of participants are mandatory. 

3.3 Trade-off Matrix

Criterion

Standalone

Distributed

Edge + DDS Peer

Cloud-Connected

Advanced DDS

Operational effort         

low        

medium      

high            

medium          

very high    

Scalability                

low-medium 

high        

high            

high            

very high    

Fault isolation            

medium     

high        

high with good DDS governance 

medium     

high         

Resource consumption       

low        

medium      

medium-high     

medium          

high         

Recovery complexity        

low        

medium      

high            

medium          

high         

Robotics interoperability  

low        

medium      

very high       

low             

very high    

The matrix is a derived selection aid based on the documented modes and architecture principles.  

4. Fault Recovery Procedures

4.1 Error Classes

The QNC separates errors by origin and state domain: Lifecycle, Connection, Protocol, Profile, Device Class, Vendor-Specific; additionally, there are configuration, security, and DDS Domain faults. This separation is operationally mandatory because recovery must always begin in a domain-specific manner.  

4.2 General Recovery Schema

Every recovery process follows this sequence:

Detect error  

Classify error and determine severity  

Lock actuation if critical  

Isolate domain  

Diagnose root cause  

Execute limited recovery action  

Verify state and telemetry  

Explicitly return to normal operation  

Complete audit entry and post-event review  

4.3 Scenario-Based Procedures

A. Communication Loss to Field Device

Detection: Link/protocol state changes, telemetry stops, heartbeat or polling fails. Critical communication errors must be detected and logged promptly.  

Diagnosis:  

Check cabling, port, power supply  

Check protocol-specific session state  

Verify whether the active profile is still valid  

Check whether the device reports a vendor fault  

Recovery:  

Start limited retry/reconnect sequence  

If present in the profile, execute recover_sequence  

If actuation cannot be safely continued: force Safe Mode  

After reconnection, stabilize status/health before releasing actuation  

B. Profile Error or Profile Incompatibility

Detection: Schema error, signature error, incompatible firmware/protocol version, or active profile invalidation. 

Recovery:  

Immediately abort new activation  

Revert to the last valid profile  

Re-initialize associated sessions  

If error was detected during active control: latch Safe Mode and require explicit recovery step  

C. Misconfiguration / Failed Update

Detection: Activation error, validation error, service/firmware update fails or violates compatibility.  

Recovery:  

Perform atomic rollback to the last known good state  

Generate audit and fault entry  

Re-check configuration state  

Keep system in READY until functional test is completed 

D. DDS Discovery Loss / QoS Mismatch / IDL Incompatibility

Detection: Participant not active, discovery unstable, offered/requested mismatch, schema version incompatible.  

Recovery:  

Isolate DDS domain; do not disturb southbound operation  

Check participant configuration, Domain ID, discovery mode, and QoS assignment  

Allow affected endpoints to re-join  

On repeated failure, fall back to defined minimal function (e.g., status/fault publication only)  

Only re-enable full DDS function after successful re-validation 

E. Unauthorized Command / Parameter Outside Limits

Detection: Command Broker or profile validation rejects the request.  

Recovery:  

Reject the individual command; do not change system state  

Return error cause to the calling instance  

Classify repeated violations as configuration or integration issue  

No automatic Safe Mode transition unless the profile or fault mapping explicitly requires it 

4.4 Restart Strategies

Restart Type

Use Case

Effect

Service Restart           

Single service disturbed              

Least disruption                            

Protocol Adapter Restart  

Southbound session defective          

Only affected device class/instance         

Participant Restart       

Local DDS fault                       

DDS re-join, southbound remains active      

Controlled System Restart 

Persistence/integrity problem         

Full lifecycle restart                      

Rollback + Restart        

After failed update                   

Return to last valid version                

This order supports the principle of “smallest possible recovery measure first.”  

4.5 State Restoration

After restart, configuration and persistent fault/audit context must be restored. Only once profiles, protocol bindings, and optional DDS artifacts have been validated may the system transition from READY to ACTIVE.  

5. Safe Mode Operations

5.1 Trigger Criteria

Safe Mode (also called Operational Inhibit Mode) is triggered by critical communication errors, integrity/configuration errors, profile invalidation during active control, certain device- or sequence-related fault conditions, and selected DDS Domain faults if configured as critical. On the profile side, this is defined via fault mappings and safe_mode_required.   

5.2 Behavior in Safe Mode

In Safe Mode, new actuation commands are mandatorily rejected. Status, health, fault, and diagnostic queries remain permitted; DDS may, if approved, continue in subscribe-only mode. Safe Mode is a risk-reduction function but not a certified safety function.   

5.3 Entry Procedure

Latch critical fault  

Immediately block new actuation  

Publish fault/event and current system mode  

Isolate affected domain  

Keep diagnostic paths open  

Optionally place affected device or DDS sub-function into degraded mode  

5.4 Operation in Safe Mode

While in Safe Mode: no automatic return to productive actuation without explicit recovery action; operators and support work in a diagnostic-oriented manner; every action must be logged; if a physical cause is possible, a field check must be performed before software recovery.  

5.5 Safe Return from Safe Mode

Return to normal operation occurs only when  

the root cause has been eliminated,  

configuration/profile/session have been re-validated,  

health and status are stable,  

an explicit recovery action has been triggered,  

a short functional test has passed.  

Safe Mode Decision Table

Condition

Action

Critical communication loss            

Activate Safe Mode                          

Non-critical single command faulty     

Reject command, no Safe Mode                

Profile integrity violated             

Stop activation or activate Safe Mode if active 

DDS QoS mismatch only on extension path

Isolate DDS, continue southbound operation  

Manually confirmed restoration successful 

Exit Safe Mode                            

This table summarizes the fault and inhibit principles described in the sources for operational use. 

6. Operational Best Practices

6.1 Monitoring

Monitoring along all state domains is recommended: Lifecycle, Connection, Protocol, Profile, Device Class, Vendor-Specific, and DDS Domain if the extension is active. Critical metrics include connection state, fault rate, recovery counter, restart frequency, discovery stability, command reject rate, and audit events.  

6.2 Logging and Alerting

Logs should be structured in JSON, filterable by severity, and operated with rotation/retention. Alerting is required at minimum for critical faults, Safe Mode entries, rollbacks, signature errors, profile invalidation, DDS discovery loss, and repeated command rejections.  

6.3 Maintenance

Maintenance work should preferably be performed in SERVICE_MODE or READY, not during active actuation. Before every maintenance activity, back up the current configuration, profile states, and fault history. After maintenance, a shortened functional and Safe Mode test is mandatory.  

6.4 Update and Rollback Strategy

Every update must be executed as a controlled change with pre-validation, atomic activation, observation window, and defined rollback threshold. Firmware and configuration updates must be signed/verified; in case of errors, immediately revert to the last known good state.  

Recommended Change Process

Approve change  

Validate artifacts  

Open maintenance window  

Activate atomically  

Observe health/fault/telemetry  

Perform smoke test  

Either close change or trigger rollback 

6.5 Operational Minimum Checklists

Daily Operation

Review fault overview  

Check Safe Mode events  

Verify communication and discovery stability  

Review audit and security events 

Weekly Routine

Check log rotation and retention  

Evaluate recovery counters and restart trends  

Verify profile/firmware compatibilities  

Compare DDS topic/QoS catalog against target  

Before Production Release

Complete configuration validation  

Safe Mode functional test  

Rollback test  

Rejoin/reconnect test  

Full audit evidence  

7. Operational State Model

BOOTING
  -> INITIALIZING
      -> READY
          -> ACTIVE
          -> SERVICE_MODE
          -> DEGRADED
          -> SAFE_MODE / FAULT_LOCKED

This state model reflects the operating states and inhibit concepts described in the ICD and PSM.  

8. Limits of This Guide

The source documents clearly define architecture, states, governance, and recovery principles; however, they do not consistently contain concrete REST endpoints, CLI commands, or bit-exact protocol frames. This guide therefore deliberately formulates system- and interface-neutral operating procedures that should be supplemented in your project-specific runbooks with the actual API names, service IDs, topic names, and device templates.  

9. Executive Summary

The robust operational strategy for the QNC is: strictly separate approved artifacts, activate every change atomically, cleanly isolate state domains, immediately switch to Operational Inhibit/Safe Mode on critical errors, perform recovery explicitly and verifiably, and always be able to roll back to the last known good state. It is precisely this combination that creates the required deployment safety and runtime stability. 
