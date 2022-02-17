# doc-library-LUMN_SERIAL
This repository holds the LumnLibrary which is used to get PAYG days and send a token code to OVES LUMN devices to update PAYG days.

## Implementation in Android Studio

### Steps
    1. Add LumnLibrary.aar (in the Libs folder) to your project. If there is no libs folder you can create one.
        This is done by adding the file in the app/libs folder.

    2. In the module (build.gradle) add the dependency like below
        implementation fileTree(include: ['*.aar'], dir: 'libs')

    3. Sync the project and extend your Activity from the Library.

        ```
            public class TestActivity extends org.oves.lumnlib.LumnLibraryActivity {

                @Override
                protected void onCreate(Bundle savedInstanceState) {
                    super.onCreate(savedInstanceState);
                }
            }
        ```
    This library will offer a simple token entry user interface to input a new token and send to the device to update the PAYG days


# UART Port Settings
Device (DEV): Lumn series product with a 2-wire UART port exposed via connector

Client App (APP): External Unit that connects to DEV via UART port

- Baud Rate 9600bps
- Start Bit 1 bit
- Data Frame 8 bit
- Stop Bit 1 bit

## Data Format
Ascii String enclosed by special escape sequence

APP request / command format <[0-9][A-Z]>;, allowed ASCII character sets delineated by < and >
- Valid command string must start with > and end with <
- Device response format: <[0-9][A-Z]>
- Valid response string must start with < and end with >

# Handshake sequences

## APP side

1) Connect to UART (OTG/USB, RS232 etc.)

2) APP detects DEV broadcast: read from port continuous stream of '<'NEW'>'

3) APP requests attention with a write to part >HAND<

4) APP reads a single \<\OK\>\ followed by silence, confirming DEV in attention mode

5) APP writes an instruction >[Request]<, details dee below

6) APP reads a <[Response]>

7) APP relinquishes attention of DEV via: UART disconnection, 120S inactivity, or command \<\END\>

## DEV side

0) Detect UART (via OTG/USB, RS232 etc.) pull-up signal

1) DEV initiates and repeats <\NEW> and followed by a read.

2) DEV reads and receives attention request string >HAND<

3) DEV sends <\OK>, and goes to attention mode, reads from UART per bus activity interrupt

4) DEV revives a >[Request]<

5) DEV writes a <[Response]> based on the request instruction details below

6) DEV regains broadcast mode with: UART disconnection, 120S inactivity, or command >END<>

Table bellow are the detail format and semantics of the [Request] and [Response], making up the public portion of the instruction set:

| APP Request            | DEV Response                               | Meaning                                                                                          |
|--------------------------------|--------------------------------------|--------------------------------------------------------------------------------------------------|
| >END<                  | null                          | end of session, give up attention                                                                |
| >HAND<                 | < OK>                          | DEV acknowledges APP presence, and goes into attention mode                                         |
| >LCS<                  | < LCS:00000>                   | Accumulated Full-Discharge Cycles (5 Digit Decimal)                                              |
| >LVC<                  | < LVC:A1A2A3A4A5A6A7A8>      | The last valid token expressed in hex, U64 encoded
| >MODE<                 | < MODE:x>                      | Current DEV PAYG Mode: 0=allow negative credit, 1=non-negative credit, 2=reserved                |
| >OCS<                  | < OCS: ENABLED> or < OCS:DISABLED> | Output control state: ["ENABLED"] / ["DISABLED"] (note the leading space in the 8-letter string) |
| >OPID<                 | < OPID:2001704660001A>         | 14 digit Device OEM_ID.   Shorter ID string are buffered with empty spaces                       |
| >OTPSN<                | < OTPSN:00000>                 | Count of past correct code entries, 5-digit decimal                                              |
| >PPID<                 | < PPID:12345678901234567890>   | 20 digit PAYG Operator_ID.   Shorter ID string are buffered with empty spaces                    |
| >PS<                   | < PS:PAYG> or < PS:FREE> | Device PAYG credit credit mode: < PS:PAYG> / < PS:FREE>                                            |
| >RPD<                  | < RPD: 0000D00H00M> | Remaining paid days before device is disabled. Decimal number
|>WOPID:2001704660001A<|< RE:WOK> (OEM ID entry succeed) < RE:FAIL>  (OEM ID entry failed) | Input OEM ID, a 14-digit ASCII string
|>WPPID:12345678901234567890<|< RE:WOK> (PAYG ID entry succeed) < RE:FAIL>  (PAYG ID entry failed) | Input PAYG ID, a 20-digit ASCII string
|>WOTP:012345678901234567890<|< RE:WAIT> (wait for device token processing)  / < RE:FAIL>  (token code entry failed) < RE:xxxx> (token entry succeeded, credit days updated to XXX, an integer value)| Input 21-digit decimal code,successful code entry returns updated credit days?example < RE:0001>?
| >WUPN:+86-13366997615< | < WOK> | Save phone number into device, 20 digit ["+", "-", 0-9]                                          |
| >WUN:XIAOMING<         | < WOK> | Save customer name 20 letter max                                                                 |


