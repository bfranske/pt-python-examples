# Introduction
The Python scripts on this page demonstrate the ability to connect to and work with the Cisco Packet Tracer software over the PTMP protocol as an "ExApp".

These are only very basic scripts and have not been thoroughly checked for issues, errors, or omissions and are designed to show a very basic example of ExApp communication with Packet Tracer over PTMP.

The scripts were principally developed using the "[Cisco Packet Tracer Packet Tracer Messaging Protocol (PTMP) Specification Document](https://tutorials.ptnetacad.net/help/default/files/CiscoPacketTracerPTMPSpecification.pdf)" and the "Packet Tracer 5.0 Inter Process Communication (IPC) Specifications Document" which are available online elsewhere as PDF files. The Cisco [Packet Tracer built-in help pages for IPC](https://tutorials.ptnetacad.net/help/default/index.htm) are also of some assistance (the PTMP specification document, but not the IPC specification document, is linked to from the built-in help pages as well). The IPC Specification document is difficult to find, critical information from it has been included below. Lastly, you will need the [Packet Tracer Extensions API documentation](https://tutorials.ptnetacad.net/help/default/IpcAPI/class_i_p_c.html) to know what IPC API calls are available for you to make. This is included in the IPC section of the Packet Tracer help files as well.

However, it is important to note that even with all of the original source documents in hand you will find there are errors and omissions in them which will prevent you from making actual IPC calls over the PTMP protocol. Specifically, the original documents lack details on how to actually format the IPC call strings using the function names in the Packet Tracer Extensions API documentation and all options for the XML manifest which must be imported into Packet Tracer to register the ExApp and how to encrypt that manifest which is required before importing. Hopefully this repository will demystify at least some of those things for you (and maybe more over time).

# Example Script Usage

1. Start by reviewing the ptmp-test.xml file. This metafile sets security permissions for the ExApp within Packet Tracer as well as sets a password (key) the ExApp will need to use to authenticate itself to Packet Tracer. You may want to change the key, if you do be sure to change the corresponding key value in the scripts before you run them.
2. Next, you will need to encrypt the XML file using the meta executable program which can be located in your Packet Tracer installation bin directory. It includes built-in help, you do not need an integrity file or certificate bundle, they are optional. You should output the encrypted file as a .pta file
3. Open Packet Tracer and under the Extensions -> IPC -> Configure Apps menu add the encrypted .pta file you just created. You should be able to verify the App decription, settings, and security options on this window as well.
4. You should now be able to run the python3 example code ptmp-negot-textenc.py which uses text encoding over PTMP and ptmp-negot-binenc.py which uses binary encoding. I suggest starting with text encoding as it is marginally easier to understand what is going on than with the binary encoding. As the PTMP specification document states "For backwards compatibility with some development platforms that do not support binary, it is necessary to support text encoding. However, binary encoding allows for efficient conversion and shorter messages, thus it is also necessary to support binary encoding." So you may want to eventually move on to binary encoding.
5. In addition to performing negotiation and authentication these sample Python files execute two basic IPC calls, one of which takes no arguments and returns the Packet Tracer version number, the other of which sends an IPC Log messgage (as an argument) to Packet Tracer which can be viewed in Packet Tracer in the Extensions -> IPC -> Log window.

# Metafile Information

The metafile information is required to register your ExApp with Packet Tracer and use it. In order to import the metadata file into Packet Tracer the file must be converted/encrypted into a .pta file instead of a .xml file. This is done with the "meta" executable program which is included in your Packet Tracer installation directory in the bin subdirectory. The meta utility includes built-in help, you do not need an integrity file or certificate bundle, they are optional and explained in the help.

## Metadata Fields

| **Field ID**      | **Explanation**                                                                                                                                                                                                                                                                    |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PT_VERSION        | Unknown, maybe the minimum version of Packet Tracer which is supported by the ExApp?                                                                                                                                                                                               |
| IPC_VERSION       | Unknown, set as 1.0, likely to allow for future IPC protocol versions that have not happened to date, may need to match PTMP version number in PTMP negotiation string                                                                                                             |
| NAME              | ExApp name, displayed in Packet Tracer ExApp configuration dialog                                                                                                                                                                                                                  |
| VERSION           | ExApp version, displayed in Packet Tracer ExApp configuration dialog                                                                                                                                                                                                               |
| ID                | Unique identifier used as a username when the ExApp connects over PTMP and authenticates as well as to identify which ExApp Packet Tracer is communicating with. Recommended to be a hierarchical naming pattern such as net.netacad.cisco.autoIP                                  |
| DESCRIPTION       | ExApp description, displayed in Packet Tracer ExApp configuration dialog                                                                                                                                                                                                           |
| AUTHOR            | ExApp author name, displayed in Packet Tracer ExApp configuration dialog                                                                                                                                                                                                           |
| CONTACT           | ExApp author contact information, displayed in Packet Tracer ExApp configuration dialog                                                                                                                                                                                            |
| EXECUTABLE_PATH   | Path to the ExApp executable file. It seems it doesn't matter what this is set to as long as the ExApp is started independently of Packet Tracer, but is likely how Packet Tracer launches the ExApp if it does so either on startup or if a file indicates an ExApp should start. |
| KEY               | Password or key used during ExApp PTMP authentication phase                                                                                                                                                                                                                        |
| SECURITY_SETTINGS | List of enabled security privileges, see possible privileges and descriptions in table below                                                                                                                                                                                       |
| LOADING           | ExApp default loading method, see possible methods and descriptions in table below                                                                                                                                                                                                 |
| SAVING            | Unknown, likely related to ability of ExApp to save data into open Packet Tracer files?                                                                                                                                                                                            |
| INSTANCES         | Packet Tracer will not allow more than this many instances of the ExApp to connect to a single Packet Tracer instance                                                                                                                                                              |

## Security Privileges
The XML metafile contains a number of security setting privileges which can be included (allowed) or not for the ExApp. This table describes available security privileges:

| **Security Privilege Name** | **Description**                                                |
|-----------------------------|----------------------------------------------------------------|
| GET_NETWORK_INFO            | Get information and configuration on current network           |
| CHANGE_NETWORK_INFO         | Change information and configuration on current network        |
| SIMULATION_MODE             | Switching between Realtime and Simulation modes                |
| MISC_GUI                    | Examine device tables                                          |
| FILE                        | File new, open, save, save as                                  |
| CHANGE_PREFERENCES          | Change sound and display preferences                           |
| CHANGE_GUI                  | Add and remove buttons to GUI                                  |
| ACTIVITY_WIZARD             | Use Activity Wizard to author or edit activity files           |
| MULTIUSER                   | Makes multi-user connections                                   |
| IPC                         | Launch external applications, disconnect external applications |
| APPLICATION                 | Exit application, change UUID                                  |

## ExApp Loading Options
This XML metafile option controls how Packet Tracer and/or the ExApp get started:

| **Loading Option** | **Description**                                                                                                                      |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| ON_STARTUP         | When PT launches, it will automatically launch this ExApp                                                                            |
| ON_DEMAND          | PT launches the ExApp when a file indicates loading it, another external application launches it, or the ExApp is started separately |
| DISABLED           | PT will not accept connections from this ExApp                                                                                       |

# Useful Specification Information

## PTMP Encoding Formats

Note that the encoding for various information depends on whether you have negotiated text mode or binary encoding when establishing your PTMP connection.

| **Name**     | **Binary Encoding**                                    | **Text Encoding (all terminated by \0)**                   |
|--------------|--------------------------------------------------------|------------------------------------------------------------|
| byte         | An 8-bit signed value                                  | A signed value between -128 to 127                         |
| bool         | An 8-bit value -- 1 (true) or 0 (false)                | "true" or "false"                                          |
| short        | A 16-bit signed number                                 | A signed number between -2^15 to 2^15 - 1                  |
| int          | A 32-bit signed number                                 | A signed number between -2^31 to 2^31 - 1                  |
| long         | A 64-bit signed number                                 | A signed number between -2^63 to 2^63 - 1                  |
| float        | A single precision 32-bit                              | A decimal number                                           |
| double       | A double precision 64-bit                              | A decimal number                                           |
| string       | Variable-length Unicode characters terminated by \0    | Variable-length Unicode characters terminated by \0        |
| Qstring      | Variable-length Qt Unicode characters terminated by \0 | Variable-length Qt Unicode characters terminated by \0     |
| IP Address   | A 32-bit value                                         | An IP address in the x.x.x.x format                        |
| IPv6 Address | A 128-bit value                                        | An IPv6 address in the x:x:x:x:x:x:x:x format              |
| MAC Address  | A 48-bit value                                         | A MAC address in the xxxx.xxxx.xxxx format                 |
| uuid         | A 128-bit value                                        | A UUID in the {HHHHHHHH-HHHHHHHH-HHHH-HHHHHHHHHHHH} format |

When using binary encoding there are no terminations or separations between items, with text encoding a null character \0 is used to terminate each item.

## IPC Data Types

These type values are used when sending IPC messages to identify the type of each parameter/argument being sent.

| **Name**     | **Type Value** | **Encoding**                                         |
|--------------|----------------|------------------------------------------------------|
| void         | 0              | N/A                                                  |
| byte         | 1              | Same as PTMP                                         |
| bool         | 2              | Same as PTMP                                         |
| short        | 3              | Same as PTMP                                         |
| int          | 4              | Same as PTMP                                         |
| long         | 5              | Same as PTMP                                         |
| float        | 6              | Same as PTMP                                         |
| double       | 7              | Same as PTMP                                         |
| string       | 8              | Same as PTMP                                         |
| Qstring      | 9              | Same as PTMP                                         |
| IP Address   | 10             | Same as PTMP                                         |
| IPv6 Address | 11             | Same as PTMP                                         |
| MAC Address  | 12             | Same as PTMP                                         |
| uuid         | 13             | Same as PTMP                                         |
| pair         | 14             | \<first type>\<first value>\<second type>\<second value> |
| vector       | 15             | \<type>\<vectorsize>\<element1>\<element2>...            |
| data         | 16             | \<custom data type in string>\0\<value>                |

<!---
the backslashes in front of angle bracket values in the above table are strictly for getting the output to render properly in Markdown
they are not part of the encoding for IPC calls, however the \0 IS part of the encoding
-->

## PTMP Message Types

Each PTMP message has a type assigned to it, these type IDs are used to identify the message type

| **Type ID** | **Message**              |
|-------------|--------------------------|
| 0           | Negotiation request      |
| 1           | Negotiation response     |
| 2           | Authentication request   |
| 3           | Authentication challenge |
| 4           | Authentication response  |
| 5           | Authentication status    |
| 6           | Keep-alive               |
| 7           | Disconnect               |
| 8           | Communication            |
| 100-199     | IPC messages             |
| 200-299     | Multi-user messages      |

## IPC Message Call Format

Note that these are made *within* an appropriate IPC LTV PTMP message

### IPC Requests
* Call ID (int): an ID to differentiate between IPC calls
* Multiple calls of the following format:
    * Call name (string): name of the call
        * Note that the name of the call is **not** in the format of "ipc.appWindow().getVersion()" the ipc. portion should be left off and instead of dots separating the parts either "\0 0 \0" (in the case of text encoding) or "\0\0" (in the case of binary encoding) is used. As usual the string should also be terminated with a \0
        * Text encoding example "appWindow\0 0 \0getVersion\0"
        * Binary encoding example "appWindow\0\0getVersion\0"
    * Multiple arguments of the following format (optional, can omit if no arguments/parameters are required for a call):
        * Type (byte): type of the argument
        * Value (variable based on type): value of the argument
    * End call value (byte): void type value (for text encoding this is another "\0 0 \0", for binary encoding just a byte of 0)

### IPC Responses
* Call ID (int): same ID as the call
* Return type (byte): type of the return value
* Return value (variable based on type): return value of the IPC call

### IPC Error Handling
* Call ID (int): same ID as the call
* Error in class (string): name of the class which the error occurred
* Error name (string): error message

## IPC Events
The Packet Tracer Event Dispatcher may also send PTMP messages about events such as file opened, file saved, device created, device deleted, simulation mode entered, device power change, port IP address changed, etc.

### Event message format
* Event ID (int): an ID to differentiate between events
* Class name (string): the class that generated the event
* Object UUID (uuid): the UUID of the object
* Event name (string): the event name
* Multiple info of the following format:
    * Type (byte): type of the info
    * Value (variable based on type): value of the info
* End event value (byte): void type value

### Event subscription message format
* Class name (string): the class that generated the event
* Object UUID (uuid): the UUID of the object; if the UUID is null, then it will subscribe to all instances of the same class
* Event name (string): the event name
* Subscribe (bool): true = subscribe, false = unsubscribe