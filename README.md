# Sweetly Virtual Audio Driver

A kernel-level virtual audio driver for Windows providing two distinct virtual audio cables (Session A and Session B) with loopback capabilities. This driver allows applications to route audio between endpoints (Speaker -> Microphone) with low latency.

## Architecture

This driver modifies the Microsoft Virtual Audio Device (MSVAD) sample to include a circular buffer loopback mechanism.

- **Render Stream (Speaker)**: Audio written to the render endpoint is copied into a shared circular buffer.
- **Capture Stream (Microphone)**: Audio read from the capture endpoint is pulled from the same circular buffer.

This allows any audio played to the "Sweetly Session Output" device to be immediately available on the "Sweetly Session Input" device.

## Devices

The driver installs the following devices:

1.  **Sweetly Session A**
    *   Output: "Connect to Sweetly Session A Output"
    *   Input: "Connect to 3rd Party App Input"
2.  **Sweetly Session B**
    *   Output: "Connect to 3rd Party Output"
    *   Input: "Connect to Sweetly Session B Input"

## Build Instructions

### Prerequisites
- Visual Studio 2022
- Windows Driver Kit (WDK) 10

### Building
1. Open the `x64 Native Tools Command Prompt for VS 2022`.
2. Navigate to the project root.
3. Run the build command:

```cmd
msbuild "simple\vadsimpl.vcxproj" /p:Configuration="Win8 Release" /p:Platform=x64 /t:Rebuild /p:TargetName=vadsimpl
msbuild "simple\vadsimpl.vcxproj" /p:Configuration="Win8 Release" /p:Platform=x64 /t:Rebuild /p:TargetName=vadsimpl_b
```

## Installation

### Self-Signing (Development)
Since this is a kernel driver, it must be signed. For development usage, create a self-signed certificate:

1. Generate a certificate and add it to the store (one-time setup).
2. Sign the binaries:

```cmd
signtool sign /fd sha256 /td sha256 /s PrivateCertStore /n "SweetlyLocalTest" /a /tr http://timestamp.digicert.com "simple\x64\Win8Release\vadsimpl.sys"
signtool sign /fd sha256 /td sha256 /s PrivateCertStore /n "SweetlyLocalTest" /a /tr http://timestamp.digicert.com "simple\x64\Win8Release\vadsimpl_b.sys"
```

### Installing the Driver

1. Add the driver package to the driver store:
```cmd
pnputil /add-driver "msvad.inf" /install
```

2. Create the virtual devices using `devcon.exe` (requires WDK tools):
```cmd
devcon install "msvad.inf" Root\SWLYSessionOutput
devcon install "msvad.inf" Root\SWLYSessionInput
devcon install "msvad.inf" Root\SWLYSessionOutputB
devcon install "msvad.inf" Root\SWLYSessionInputB
```

