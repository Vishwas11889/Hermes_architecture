Hermes Corporate Deployment Architecture

1. Overview

The solution consists of two separate codebases:

1. UI Application — independently developed and packaged as "UI.exe"
2. Hermes Agent Runtime — a modified version of the upstream "NousResearch/hermes-agent" repository, packaged and distributed as a controlled Hermes backend/runtime.

The UI and Hermes runtime communicate through a defined local interface. The Hermes runtime is responsible for agent execution, skills, tools, providers, and Outlook automation.

---

2. High-Level Architecture

┌─────────────────────────────────────────────────────────────┐
│                    Corporate Windows Machine                 │
│                                                             │
│  ┌───────────────────────┐                                  │
│  │       UI.exe          │                                  │
│  │                       │                                  │
│  │  Separate Codebase    │                                  │
│  │  - User Interface     │                                  │
│  │  - Configuration      │                                  │
│  │  - User Interaction   │                                  │
│  └───────────┬───────────┘                                  │
│              │                                              │
│              │ Local IPC / HTTP / CLI / Process Interface   │
│              ▼                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                  Hermes Backend                         │ │
│  │                                                        │ │
│  │                    Hermes.exe                          │ │
│  │                                                        │ │
│  │  Modified Hermes Agent Codebase                       │ │
│  │  ├── Agent                                             │ │
│  │  ├── Tools                                             │ │
│  │  ├── Skills                                            │ │
│  │  ├── Providers                                         │ │
│  │  ├── Outlook Integration                               │ │
│  │  └── Agent Runtime                                     │ │
│  └────────────────────────┬───────────────────────────────┘ │
│                           │                                  │
│                           ▼                                  │
│                 ┌─────────────────────┐                     │
│                 │ Python Runtime       │                     │
│                 │ & Dependencies      │                     │
│                 └─────────────────────┘                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘

---

3. Repository Architecture

The upstream Hermes repository is used as the base implementation.

NousResearch/hermes-agent
        │
        ▼
      Fork
        │
        ▼
Corporate Hermes Repository
        │
        ├── Modified Hermes Source
        ├── Agent
        ├── Tools
        ├── Skills
        ├── Providers
        ├── Configuration
        └── Runtime Dependencies

The UI application is maintained independently.

Corporate Workspace
│
├── ui/
│   ├── UI source code
│   └── UI.exe
│
└── hermes/
    ├── Modified Hermes source
    ├── Skills
    ├── Tools
    ├── Providers
    └── Runtime

---

4. Application Separation

The UI and Hermes backend are intentionally maintained as separate applications.

UI Application

Responsibilities:

- User interface
- User interaction
- Configuration UI
- Agent invocation
- Displaying agent responses
- Displaying task status
- Starting/stopping Hermes backend when required

The UI does not contain the Hermes agent source code.

---

Hermes Backend

Responsibilities:

- Agent execution
- LLM interaction
- Tool execution
- Skill execution
- Outlook automation
- Scheduled tasks
- Agent configuration
- Runtime management

The Hermes backend does not depend on the UI for its core functionality.

---

5. Runtime Communication

The UI communicates with Hermes through a controlled local interface.

Possible interfaces include:

UI.exe
   │
   ├── HTTP
   │
   ├── Local IPC
   │
   ├── CLI invocation
   │
   └── Local process communication

The selected interface should be standardized and documented.

For example:

UI.exe
   │
   │ HTTP / localhost
   ▼
Hermes.exe
   │
   ▼
Hermes Agent

The Hermes backend should expose only the interfaces required by the UI.

---

6. Hermes Packaging Strategy

The modified Hermes source should be packaged independently from the UI.

Recommended initial approach

Use a controlled Python runtime containing:

Hermes Runtime
│
├── Python Runtime
├── Python Dependencies
├── Hermes Source
├── Skills
├── Tools
├── Providers
├── Configuration
└── Required Native Dependencies

The UI launches or communicates with this runtime through "Hermes.exe" or the designated Hermes launcher.

---

7. Packaging Options

Option A — PyInstaller

Modified Hermes Source
        │
        ▼
    PyInstaller
        │
        ▼
     Hermes.exe

Advantages:

- Python does not need to be installed separately
- Can produce a self-contained executable
- Easier distribution

Considerations:

- Dynamic imports must be handled
- Runtime-loaded skills/files must be included
- Native dependencies must be verified
- Some Hermes functionality may require additional PyInstaller configuration

PyInstaller should therefore be introduced only after validating the Hermes runtime entry point and dependencies.

---

Option B — Bundled Python Runtime

Hermes Package
│
├── python.exe
├── Python libraries
├── site-packages
├── Hermes source
├── Skills
├── Tools
└── Configuration

The launcher starts the Hermes Python runtime.

This approach keeps the Hermes runtime closer to its native Python execution model and can simplify troubleshooting and maintenance.

For the initial corporate implementation, this approach can be preferred if PyInstaller introduces compatibility issues.

---

8. Recommended Corporate Architecture

                     Corporate Windows Machine
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
        ┌───────────┐                  ┌─────────────┐
        │   UI.exe  │                  │ Hermes.exe  │
        │           │                  │             │
        │ UI Layer  │◄────────────────►│ Agent Layer │
        └───────────┘       IPC         └──────┬──────┘
                                               │
                                               ▼
                                      ┌─────────────────┐
                                      │ Hermes Runtime  │
                                      │                 │
                                      │ Python          │
                                      │ Dependencies    │
                                      │ Agent           │
                                      │ Tools           │
                                      │ Skills          │
                                      │ Providers       │
                                      └────────┬────────┘
                                               │
                            ┌──────────────────┼──────────────────┐
                            │                  │                  │
                            ▼                  ▼                  ▼
                         Outlook             LLMs              Other
                         COM/API           Providers           Tools

---

9. Build Pipeline

The corporate build process should follow a controlled pipeline.

Upstream Hermes
       │
       ▼
Corporate Fork
       │
       ▼
Corporate Modifications
       │
       ▼
Code Review
       │
       ▼
Dependency Locking
       │
       ▼
Dependency / CVE Scan
       │
       ▼
SAST
       │
       ▼
Secret Scan
       │
       ▼
Malware Scan
       │
       ▼
Build Hermes Package
       │
       ▼
Generate SBOM
       │
       ▼
Hash Build Artifacts
       │
       ▼
Security Approval
       │
       ▼
Corporate Distribution

---

10. Security Boundaries

The packaged Hermes runtime must be treated as a controlled application.

The following must not be embedded in the executable/package:

API Keys
Passwords
OAuth Client Secrets
Access Tokens
Private Certificates
Corporate Credentials
Production Secrets

Secrets should be supplied through an approved enterprise configuration or secret-management mechanism.

---

11. Immutable Application Runtime

Users should not directly modify the bundled Hermes runtime.

The expected lifecycle is:

Source Modification
       │
       ▼
Code Review
       │
       ▼
Security Scan
       │
       ▼
Build
       │
       ▼
New Hermes Version
       │
       ▼
New Artifact Hash
       │
       ▼
Security Approval
       │
       ▼
Deployment

If Hermes behavior needs to change, a new approved build should be generated rather than modifying files inside the installed runtime.

---

12. Suggested Installation Structure

A possible corporate installation layout:

C:\Program Files\Company\Hermes\
│
├── Hermes.exe
├── runtime\
│   ├── Python\
│   ├── Libraries\
│   └── Dependencies\
│
├── skills\
├── tools\
├── providers\
├── config\
└── manifest\
    ├── VERSION
    ├── SBOM
    └── HASHES

User-specific runtime data should remain outside the installation directory:

%LOCALAPPDATA%\Company\Hermes\
│
├── logs\
├── sessions\
├── cache\
├── user-config\
└── workspace\

---

13. Update Strategy

Automatic unrestricted updates should not be enabled in the corporate environment.

Recommended flow:

New Hermes Source
       │
       ▼
Corporate Review
       │
       ▼
Security Validation
       │
       ▼
Build
       │
       ▼
Versioned Artifact
       │
       ▼
Security Approval
       │
       ▼
Corporate Deployment

Example:

Hermes v1.0
Hermes v1.1
Hermes v1.2

Each version should have its own build metadata and artifact hash.

---

14. Two-Use-Case Support

The same Hermes backend can support both required Outlook use cases.

                         Hermes Backend
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
        Normal Outlook Agent       Scheduled Outlook Agent
                 │                         │
                 │                         │
          User-triggered             Cron/Scheduler
                 │                         │
                 └────────────┬────────────┘
                              ▼
                       Outlook Integration

The Outlook functionality should be implemented as reusable Hermes capabilities/skills rather than creating two completely separate Hermes runtimes.

---

15. Final Target Architecture

┌─────────────────────────────────────────────────────────────┐
│                    Corporate Windows Host                   │
│                                                             │
│  ┌───────────────────┐       ┌───────────────────────────┐ │
│  │      UI.exe       │       │       Hermes.exe          │ │
│  │                   │       │                           │ │
│  │ Separate          │◄─────►│ Modified Hermes           │ │
│  │ Codebase          │ IPC   │ Backend                   │ │
│  └───────────────────┘       └─────────────┬─────────────┘ │
│                                            │               │
│                                            ▼               │
│                               ┌─────────────────────────┐  │
│                               │ Hermes Runtime           │  │
│                               │                         │  │
│                               │ Python                  │  │
│                               │ Agent                   │  │
│                               │ Skills                  │  │
│                               │ Tools                   │  │
│                               │ Providers               │  │
│                               │ Outlook COM/API         │  │
│                               └────────────┬────────────┘  │
│                                            │               │
│                           ┌────────────────┼────────────┐  │
│                           ▼                ▼            ▼  │
│                        Outlook            LLMs        Tools│
│                                                             │
└─────────────────────────────────────────────────────────────┘

Design Principles

1. UI and Hermes remain separate codebases.
2. Hermes is built from the organization's controlled fork.
3. The Hermes runtime is packaged independently of the UI.
4. The runtime should not be user-editable after deployment.
5. Secrets must never be bundled into the EXE.
6. Dependencies must be pinned and scanned.
7. Every production build should have a version and artifact hash.
8. Updates should go through the corporate build/security process.
9. The same Hermes runtime should support both interactive and scheduled Outlook use cases.
10. PyInstaller should be used only after validating the actual Hermes runtime entry point and dynamic dependencies.