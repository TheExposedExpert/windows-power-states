# Windows operating system power state forensics

This chapter includes direct quotes from Microsoft documentation.


The Windows operating system includes Windows Advanced Configuration and Power Interface (ACPI) driver (Acpi.sys) which is responsible for power management and Plug and Play (PnP) device enumeration activities and it acts as the interface between the operating system and the ACPI BIOS.

The acpi.sys driver is located in the C:\Windows\System32\drivers\ folder.

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

So, the Windows operating system includes system power states from S0 to S5. There's one additional system power state which is G3 and it describes Mechanical Off state by the system. The G3 power state can be achieved by switching the Power Supply Unit (PSU) to off state with a physical switch which disables the PSU's standby power. This is normal activity for desktop systems, for example. Here is Microsoft's table describing system power each state:

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

Similarly, the Fast Startup option can be configured with the old school power options control panel item. Currently the test system is configured with enabled option:

![text](/images/controlpanel_faststartup.png)

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

![text](/images/controlpanel_faststartup.png)
![text](/images/controlpanel_faststartup.png)

Hybrid Sleep option is configured via two registry keys

* Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\User\PowerSchemes\381b4222-f694-41f0-9685-ff5bb260df2e\238c9fa8-0aad-41ed-83f4-97be242c8f20\94ac6d29-73ce-41a6-809f-6363ba21b47e\ACSettingIndex

* Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\User\PowerSchemes\381b4222-f694-41f0-9685-ff5bb260df2e\238c9fa8-0aad-41ed-83f4-97be242c8f20\94ac6d29-73ce-41a6-809f-6363ba21b47e\ACSettingIndex

By setting the registry key value between 0 (disabled) and 1 (enabled). Alternating Current (AC) describes battery power while Direct Current (DC) describes plugged in power cord in terms of electricity.

![text](/images/controlpanel_faststartup.png)
![text](/images/controlpanel_faststartup.png)

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


