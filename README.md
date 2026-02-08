# Windows operating system power state forensics

This chapter includes direct quotes from Microsoft documentation.

## Introduction


The Windows operating system includes Windows Advanced Configuration and Power Interface (ACPI) driver (Acpi.sys) which is responsible for power management and Plug and Play (PnP) device enumeration activities and it acts as the interface between the operating system and the ACPI BIOS.

The acpi.sys driver is located in the C:\Windows\System32\drivers\ folder.

![text](/images/acpi.sys_explorer.png)

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/acpi-driver<br>
https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/acpi-bios
<br>
<br>

A power state indicates the level of power consumption — and thus the extent of computing activity — by the system or by a single device. The power manager sets the power state of the system as a whole. Device drivers set the power state of their individual devices.

The ACPI specification defines two sets of discrete power states: system power states and device power states. Each power state has a unique name.

System power states are named Sx, where x is a state number between 0 and 5. Device power states are named D<x>, where <x> is a state number between 0 and 3. The state number is inversely related to power consumption: higher numbered states use less power. States S0 and D0 are the highest-powered, most functional, fully on states. States S5 and D3 are the lowest-powered states and have the longest wake-up latency.

The operating system supports six system power states, referred to as S0 (fully on and operational) through S5 (power off). Each state is characterized by the following:

* Power consumption: how much power does the computer use?

* Software resumption: from what point does the operating system restart?

* Hardware latency: how long does it take to return the computer to the working state?

* System hardware context (such as the content of volatile processor registers, memory caches, and RAM): how much system hardware context is retained? Must the operating system reboot to return to the working state?

State S0 is the working state. States S1, S2, S3, and S4 are sleeping states, in which the computer appears off because of reduced power consumption but retains enough context to return to the working state without restarting the operating system. State S5 is the shutdown or off state.

A system is waking when it is in transition from the shutdown state (S5) or any sleeping state (S1-S4) to the working state (S0), and it is going to sleep when it is in transition from the working state to any sleep state or the shutdown state. The following figure shows the possible system power state transitions.

![text](/images/sysstate.png)

As the previous figure shows, the system cannot enter one sleep state directly from another; it must always enter the working state before entering any sleep state. For example, a system cannot transition from state S2 to S4, nor from state S4 to S2. It must first return to S0, from which it can enter the next sleep state. Because a system in an intermediate sleep state has already lost some operating context, it must return to the working state to restore that context before it can make an additional state transition.

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/power-states<br>
https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/system-power-states
<br>
<br>

So, the Windows operating system includes system power states from S0 to S5. There's one additional system power state which is G3 and it describes Mechanical Off state by the system. The G3 power state can be achieved by switching the Power Supply Unit (PSU) to off state with a physical switch which disables the PSU's standby power. This is normal activity for desktop systems, for example. Here is Microsoft's table describing each system power state:

| Power state   | ACPI state    | Description  |
|:------------- |:------------- |:------------ |
| Working       | S0            | The system is fully usable. Hardware components that aren't in use can save power by entering a lower power state. |
| Sleep (Modern Standby)             | S0 low-power idle             | Some SoC systems support a low-power idle state known as Modern Standby. In this state, the system can very quickly switch from a low-power state to high-power state in response to hardware and network events. Note: SoC systems that support Modern Standby don't use S1-S3.            |
| Sleep             | S1<br>S2<br>S3<br>             | The system appears to be off. The amount of power consumed in states S1-S3 is less than S0 and more than S4. S3 consumes less power than S2, and S2 consumes less power than S1. Systems typically support one of these three states, not all three.<br> In states S1-S3, volatile memory is kept refreshed to maintain the system state. Some components remain powered so the computer can wake from input from the keyboard, LAN, or a USB device.<br> Hybrid sleep, used on desktops, is where a system uses a hibernation file with S1-S3. The hibernation file saves the system state in case the system loses power while in sleep.<br> Note: SoC systems that support Modern Standby don't use S1-S3.            |
| Hibernate             | S4             | The system appears to be off. Power consumption is reduced to the lowest level. The system saves the contents of volatile memory to a hibernation file to preserve system state. Some components remain powered so the computer can wake from input from the keyboard, LAN, or a USB device. The working context can be restored if it's stored on nonvolatile media. Fast startup is where the user is logged off before the hibernation file is created. This allows for a smaller hibernation file, more appropriate for systems with less storage capabilities.            |
| Soft off             | S5             | The system appears to be off. This state is comprised of a full shutdown and boot cycle.            |
| Mechanical off             | G3             | The system is completely off and consumes no power. The system returns to the working state only after a full reboot.            |


<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows/win32/power/system-power-states<<br>
https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/iet-cdt.2013.0137
<br>
<br>

Here's is a diagram provided by Microsoft which describes activity when system power state is changed via Control Panel:

![text](/images/power-comp.png)

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/system-wide-overview-of-power-management
<br>
<br>


Available system power states by the hardware running the operating system can be inspected with powercfg.exe tool with /availablesleepstates parameter:

![text](/images/powercfg_availablesleepstates.png)

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/powercfg-command-line-options
<br>
<br>

Most of the settings can be configured with control panel power & battery settings. Currently the system is set to go to system power state S3 when closing the lid, for example.

![text](/images/controlpanel_powerbattery.png)

What are these Hybrid Sleep and Fast Startup states again?

Hybrid sleep, used on desktops, is where a system uses a hibernation file with S1-S3. The hibernation file saves the system state in case the system loses power while in sleep. Hybrid sleep option can be configured with old school power options control panel item. Currently the test system is configured with disabled option:

![text](/images/controlpanel_allowhybridsleep.png)

Fast startup is a type of shutdown that uses a hibernation file to speed up the subsequent boot. During this type of shutdown, the user is logged off before the hibernation file is created. Fast startup allows for a smaller hibernation file, more appropriate for systems with less storage capabilities. For more info, see Hibernation file types.

When using fast startup, the system appears to the user as though a full shutdown (S5) has occurred, even though the system has actually gone through S4. This includes how the system responds to device wake alarms.

Fast startup logs off user sessions, but the contents of kernel (session 0) are written to hard disk. This enables faster boot.

In Windows, fast startup is the default transition when a system shutdown is requested. A full shutdown (S5) occurs when a system restart is requested or when an application calls a shutdown API.

Similarly, the Fast Startup option can be configured with the old school power options control panel item. Currently the test system is configured with enabled option:

![text](/images/controlpanel_faststartup.png)

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/system-power-states
<br>
<br>

So this laptop test system, running Windows 11 Pro version 10.0.26200.6584 supports the following system power states:

* S0 - Working
* S3 - Sleep
* S3 - Sleep + Hybrid Sleep
* S4 - Hibernate
* S4 - Hibernate + Fast Startup
* S5 - Soft off

And is currently on system power state S0 as the operating system is "running" for the user.

Where are these options configured in the Windows registry? Let's test it. SysInternals Process Monitor tool can be used to identity changed registry keys on a Windows system.

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/sysinternals/downloads/procmon
<br>
<br>

Fast Startup option is configured via "Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Power\HiberbootEnabled" registry key by changing the value between 0 (disabled) and 1 (enabled).

![text](/images/procmon_disablefaststartup.png)
![text](/images/regedit_disablefaststartup.png)

Hybrid Sleep option is configured via two registry keys

* Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\User\PowerSchemes\381b4222-f694-41f0-9685-ff5bb260df2e\238c9fa8-0aad-41ed-83f4-97be242c8f20\94ac6d29-73ce-41a6-809f-6363ba21b47e\ACSettingIndex

* Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\User\PowerSchemes\381b4222-f694-41f0-9685-ff5bb260df2e\238c9fa8-0aad-41ed-83f4-97be242c8f20\94ac6d29-73ce-41a6-809f-6363ba21b47e\ACSettingIndex

By setting the registry key value between 0 (disabled) and 1 (enabled). Alternating Current (AC) describes battery power while Direct Current (DC) describes plugged in power cord in terms of electricity.

![text](/images/procmon_enablehybridsleep.png)
![text](/images/regedit_enablehybridsleep.png)

The GUID of "381b4222-f694-41f0-9685-ff5bb260df2e" stands for Balanced power plan as seen in previous pictures.<br>
The GUID of "238c9fa8-0aad-41ed-83f4-97be242c8f20" stands for Sleep Settings as seen in previous pictures.<br>
The GUID of "94ac6d29-73ce-41a6-809f-6363ba21b47e" stands for Hybrid Sleep as seen in previous pictures.<br>

So in essence the balanced power plan sleep settings were modified. Modifications included hybrid sleep configuration changes for both battery and plugged in power schemes.

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/windows/win32/power/power-policy-settings<br>
https://learn.microsoft.com/en-us/windows-hardware/customize/power-settings/sleep-settings<br>
https://learn.microsoft.com/en-us/windows-hardware/customize/power-settings/sleep-settings-hybrid-sleep
<br>
<br>


What kind of events are logged by default by the Windows operating system when power state changes between S0 and other states? How do we investigate when a computer has been in sleep / hibernate / shutdown / etc state based on Windows event logs?

Let's test these power states with the following scenarios:

* Click power off from start menu and re-start the operating system.
* Click sleep from start menu and re-start the operating system. 
* Click hibernate from start menu and re-start the operating system. 
* Create blue screen of death and re-start the operating system.
* Hard shutdown by holding the physical power button for 10 seconds and re-start the operating system.

Hard shutdown can be forced by holding the power button for 10-20 seconds, for example, on Lenovo and Microsoft Surface devices.

<ins>Sources:</ins><br>
https://support.microsoft.com/en-us/surface/force-a-shutdown-and-restart-your-surface-cf13996e-fef0-dbb0-1e94-cdd7ff88b840<br>
https://support.lenovo.com/us/en/troubleshoot/lpt000805
<br>
<br>

Blue screen of death can be programmatically performed with SysInternals NotMyFault tool.

<ins>Sources:</ins><br>
https://learn.microsoft.com/en-us/sysinternals/downloads/notmyfault
<br>
<br>

So we need to test each of these five scenarios with all available system power states. From S0 (Working) to:

* S3 - Sleep
* S3 - Sleep + Hybrid Sleep
* S4 - Hibernate
* S4 - Hibernate + Fast Startup
* S5 - Soft off

And back.

<br>
<br>
<br>

## Test Results

So I performed the below nine tests on the Windows test host:

| Power state change | Test number | Power state initiation |
|--------------------|-----------------|------------------------|
| S0 -> S5 | Test 1 | Shut down from start menu (fast startup disabled) |
| S0 -> S5 | Test 2 | Restart from start menu (fast startup disabled) |
| S0 -> S5 | Test 3 | BSOD |
| S0 -> S5 | Test 4 | Hard shutdown |
| S0 -> S3 | Test 5 | Sleep from start menu |
| S0 -> S3 + Hybrid Sleep | Test 6 | Sleep from start menu |
| S0 -> S4 | Test 7 | Hibernate from start menu |
| S0 -> S4 + Fast Startup | Test 8 | Hibernate from start menu |
| S0 -> S4 + Fast Startup | Test 9 | Shut down from start menu |

### Test 1

#### Events before power state change

| Event time | Event ID |
|------------|----------|
| 2026-01-03T15:35:23.0527356Z | Event ID 1074 |
| 2026-01-03T15:35:30.3957825Z | Event ID 6006 |
| 2026-01-03T15:35:31.3757329Z | Event ID 109 |
| 2026-01-03T15:35:31.6720412Z | Event ID 577 |
| 2026-01-03T15:35:31.8679813Z | Event ID 13 |

Event log entries before power change activity:

<details>
<summary>Event ID 1074</summary>

The Event ID 1074 indicates the directory path of the process which initiated the activity including the user name. The Shutdown Type field indicates that this is was a "power off" action. The initating process changes and can be logged also due to software installations or Windows update installation activity.

![text](/images/Test01-01.png)

```python
 The process C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe (DESKTOP-J1VGUK5) has initiated the power off of computer DESKTOP-J1VGUK5 on behalf of user DESKTOP-J1VGUK5\Test for the following reason: Other (Unplanned)
 Reason Code: 0x0
 Shutdown Type: power off
 Comment: 
```
</details>

<details>
<summary>Event ID 6006</summary>

The Event ID 6006 is not directly related to the power state change activity, but is always included when a power state is changing to S5 in normal conditions.

![text](/images/Test01-02.png)

```python
 The Event log service was stopped. 
```
</details>

<details>
<summary>Event ID 109</summary>

The Event ID 109 also indicates that a shut down was initiated on the host.

![text](/images/Test01-05.png)

```python
 The kernel power manager has initiated a shutdown transition.

 Action: Power Action Shutdown Off 
 Event Code: 0x0 
 Reason: Kernel API 
```
</details>

<details>
<summary>Event ID 577</summary>

![text](/images/Test01-06.png)

```python
 The system has prepared for a system initiated reboot from Active.
```
</details>

<details>
<summary>Event ID 13</summary>

The Event ID 13 is finally logged before the operating system shutdown with the timestamp of the power state change.

![text](/images/Test01-07.png)

```python
 The operating system is shutting down at system time ‎2026‎-‎01‎-‎03T15:35:31.867980300Z.
```
</details>


#### Events after power state change

| Event time | Event ID |
|------------|----------|
| 2026-01-03T15:37:18.9517820Z | Event ID 12 |
| 2026-01-03T15:37:18.9528746Z | Event ID 20 |
| 2026-01-03T15:37:29.1776946Z | Event ID 6005 |
| 2026-01-03T15:37:29.1779612Z | Event ID 6013 |

Event log entries after power change activity:

<details>
<summary>Event ID 12</summary>

The Event ID 12 is first logged after operating system has been powered on with the timestamp of the power state change.

![text](/images/Test01-08.png)

```python
 The operating system started at system time ‎2026‎-‎01‎-‎03T15:37:18.500000000Z.
```
</details>

<details>
<summary>Event ID 20</summary>

The Event ID 20 indicates that the power change operation for both shutdown and bootup was successful.

![text](/images/Test01-09.png)

```python
 The last shutdown's success status was true. The last boot's success status was true.
```
</details>

<details>
<summary>Event ID 6005</summary>

The Event ID 6005 indicates that the event log service has been started. Similar to Event ID 6006 this is not directly related to power change activity, but the 6005 event is always included when operating system returns to S0 (Working) state from S5 (Soft off) state.

![text](/images/Test01-03.png)

```python
 The Event log service was started.
```
</details>

<details>
<summary>Event ID 6013</summary>

The Event ID 6013 is logged periodically (I believe once a day by default) by the operating system, but it is also logged after an operating system has returned to S0 (Working) state from S5 (Soft off) state. In this case the uptime in seconds value indicates a short uptime as the host has just started up.

![text](/images/Test01-04.png)

```python
 The system uptime is 10 seconds.
```
</details>

<br>

### Test 2

Test 2 has exactly the same event IDs logged as Test 1, but events include mention of operating system restart instead of power off.

#### Events before power state change

| Event time | Event ID |
|------------|----------|
| 2026-01-03T16:00:27.6372575Z | Event ID 1074 |
| 2026-01-03T16:00:30.4897288Z | Event ID 6006 |
| 2026-01-03T16:00:35.4032567Z | Event ID 109 |
| 2026-01-03T16:00:35.8332933Z | Event ID 577 |
| 2026-01-03T16:00:36.0076589Z | Event ID 13 |

Event log entries before power change activity:

<details>
<summary>Event ID 1074</summary>

The Event ID 1074 indicates the directory path of the process which initiated the activity including the user name. The Shutdown Type field indicates that this is was a "restart" action. 

![text](/images/Test02-01.png)

```python
The process C:\Windows\SystemApps\Microsoft.Windows.StartMenuExperienceHost_cw5n1h2txyewy\StartMenuExperienceHost.exe (DESKTOP-J1VGUK5) has initiated the restart of computer DESKTOP-J1VGUK5 on behalf of user DESKTOP-J1VGUK5\Test for the following reason: Other (Unplanned)
 Reason Code: 0x0
 Shutdown Type: restart
 Comment: 
```

The initating process changes and can be logged also due to software installations or Windows update installation activity. For example, TrustedInstaller.exe can be source for the activity during operating system update installations:

![text](/images/Test02-03.png)

```python
The process C:\WINDOWS\servicing\TrustedInstaller.exe (DESKTOP-J1VGUK5) has initiated the restart of computer DESKTOP-J1VGUK5 on behalf of user NT AUTHORITY\SYSTEM for the following reason: Operating System: Upgrade (Planned)
 Reason Code: 0x80020003
 Shutdown Type: restart
 Comment: 
```

And OptionalFeatures.exe can be source for the activity when built-in operating system features are added to the host and the feature installation requires a restart to complete it:

![text](/images/Test02-04.png)

```python
The process OptionalFeatures.exe has initiated the restart of computer DESKTOP-J1VGUK5 on behalf of user DESKTOP-J1VGUK5\Test for the following reason: No title for this reason could be found
 Reason Code: 0x80020000
 Shutdown Type: restart
 Comment: 
```

</details>

<details>
<summary>Event ID 109</summary>

The Event ID 109 also indicates that a reboot was initiated on the host.

![text](/images/Test02-01.png)

```python
The kernel power manager has initiated a shutdown transition.

Action: Power Action Reboot 
Event Code: 0x0 
Reason: Kernel API
```
</details>

#### Events after power state change

| Event time | Event ID |
|------------|----------|
| 2026-01-03T16:00:44.9476897Z | Event ID 12 |
| 2026-01-03T16:00:44.9488803Z | Event ID 20 |
| 2026-01-03T16:00:53.4769290Z | Event ID 6005 |
| 2026-01-03T16:00:53.4772313Z | Event ID 6013 |

Event log entries after power change activity:


