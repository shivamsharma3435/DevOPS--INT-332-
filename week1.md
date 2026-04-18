DevOps Basics + Virtualization
## INTRODUCTION TO DEVOPS
Definition:
- DevOps is a combination of Development (Dev) and Operations (Ops) practices that aims to automate and integrate the processes of software development and IT operations.

In simple words : DevOps is a methodology that ensures faster, reliable, and continuous delivery of software.

# Objectives of DevOps:
- Faster software delivery
- Continuous integration & deployment (CI/CD)
- Improved collaboration between teams
- Automation of manual processes
- High-quality and reliable systems

# Why DevOps is Needed?
Traditional Approach Problems:

Before DevOps:
- Developers write code → handover to operations
- Operations deploy and manage
Issues:
- Communication gap
- Delayed deployments
- Frequent errors
- Difficult debugging

DevOps Solution:
DevOps introduces ->
- Continuous collaboration
- Automation tools
- Monitoring systems

Result:
- Faster releases
- Reduced failures
- Better productivity

## VIRTUALIZATION
- Virtualization is the process of creating multiple virtual machines (VMs) on a single physical machine using a software called a hypervisor.

Key Concept:
- One physical server → multiple virtual servers (VMs)

Each VM has:
- Its own OS
- Its own applications
- Its own resources

# Hypervisor
A hypervisor is a software that:
- Creates and manages virtual machines
- Allocates hardware resources to each VM

Types of Hypervisor:
1. Type 1 (Bare Metal)
Runs directly on hardware
Example: VMware ESXi
2. Type 2
Runs on top of OS
Example: VirtualBox

## Architecture of Virtualization
Structure:

Physical Server → Hypervisor → Multiple VMs → Apps

# Features of Virtual Machines

Each VM:
- Fully isolated
- Has separate OS
- Works like real computer

# Advantages of Virtualization
1. Better Resource Utilization
Multiple VMs share one system
2. Strong Isolation
If one VM fails → others unaffected
3. Multiple OS Support
Run Windows + Linux together
4. Cost Saving
Less hardware required
5. Disaster Recovery
Easy backup and restore
6. Scalability
Add/remove VMs easily

# Disadvantages of Virtualization
These are VERY IMPORTANT (often asked in exams)

1. Heavy Resource Usage
Each VM requires full OS
2. Slow Boot Time
OS takes time to start
3. Inefficient Scalability
Not suitable for microservices
4. Limited Portability
Moving VMs is complex
5. High Maintenance
Manage multiple OS
6. Increased Cost
Licensing + infrastructure cost
4. WHY VIRTUALIZATION WAS NOT ENOUGH
Problem Summary:

Even after virtualization:
- Systems were still heavy
- Startup time was slow
- Resource usage was high
- Need for Better Solution:
This led to:
Containerization (next topic in Week 2)

## COMPARISON: TRADITIONAL vs VIRTUALIZATION
Feature	         Traditional	    Virtualization
Apps             per server 1	    Multiple
Resource usage	 Poor	            Better
Cost	         High	            Reduced
Isolation	     N/A	            Strong
Flexibility	     Low	            High

