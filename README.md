# TPNETBIOS















        ##    ##  ####### ########      ######   ####    ##      ######
        ###   ##  ##         ##         ##   ##   ##   ##  ##   ##    ##
        ####  ##  ##         ##         ##   ##   ##  ##    ##  ##      
        ## ## ##  #####      ##         ######    ##  ##    ##   ###### 
        ##  ####  ##         ##         ##   ##   ##  ##    ##        ##
        ##   ###  ##         ##         ##   ##   ##   ##  ##    ##   ##
        ##    ##  #######    ##         ######   ####    ##       ##### 



                                  CBIS Net Bios
                             Programmer's Reference







                                 by Tom Thompson

                                   April 1988








                                    CBIS,Inc.
                         5875 Peachtree Industrial Blvd.
                              Bldg. 100  Suite 170
                               Norcross, GA  30092




        Copyright (c) 1988 by CBIS, Inc.  All rights reserved.  No part 
        of this publication may be reproduced without the prior written 
        permission of CBIS, Inc., P.O. Box 921206, Norcross, GA 30092.

        CBIS, Inc. makes no representations or warranties with respect to 
        the contents hereof and specifically disclaims any warranties 
        either expressed or implied of merchantability or fitness for any 
        particular purpose.  CBIS, Inc. reserves the right to change this 
        publication and the software programs to which it relates with no 
        obligation to notify any person or organization of such changes.




                                TABLE OF CONTENTS



                  Introduction . . . . . . . . . . . . . . . . .  1

                  Net Bios . . . . . . . . . . . . . . . . . . .  1

                  Names  . . . . . . . . . . . . . . . . . . . .  2

                  Calling Net Bios . . . . . . . . . . . . . . .  2

             General Commands
                  Reset Command  . . . . . . . . . . . . . . . .  5
                  Cancel Command . . . . . . . . . . . . . . . .  5
                  Adapter Status Command . . . . . . . . . . . .  5
                  Unlink Command . . . . . . . . . . . . . . . .  6

             Name Commands
                  Add Name Command . . . . . . . . . . . . . . .  6
                  Add Group Name Command . . . . . . . . . . . .  6
                  Delete Name Command  . . . . . . . . . . . . .  6

             Session Control Commands
                  Call Command . . . . . . . . . . . . . . . . .  7
                  Listen Command . . . . . . . . . . . . . . . .  7
                  Hang Up Command  . . . . . . . . . . . . . . .  8
                  Session Status Command . . . . . . . . . . . .  8

             Session Data Transfer Command
                  Send Command . . . . . . . . . . . . . . . . .  8
                  Chain Send Command . . . . . . . . . . . . . .  9
                  Receive Command  . . . . . . . . . . . . . . .  9
                  Receive Any Command. . . . . . . . . . . . . .  9

             Datagram Commands
                  Send Datagram Command  . . . . . . . . . . . . 10
                  Receive Datagram Command . . . . . . . . . . . 10
                  Send Broadcast Command . . . . . . . . . . . . 11
                  Receive Broadcast Command  . . . . . . . . . . 11

             Tables
                  1 - Net Bios Command Summary . . . . . . . . . 12
                  2 - Net Bios Control Block (NCB) Format  . . . 13
                  3 - NCB Usage by Command . . . . . . . . . . . 14
                  4 - Net Bios Error Codes . . . . . . . . . . . 15
                  5 - Adapter Status Result Buffer . . . . . . . 17
                  6 - Session Status Result Buffer . . . . . . . 18









                                       i




























































                                      ii



                  ------------
                  Introduction
                  ------------

        This document describes the programming interface for CBIS' Net 
        Bios, which has been implemented for several networks, including 
        ARCNET, EtherNet, StarLan, PC-Office NetBoard, and several 
        internal bus topologies.

        Presented first is a general discussion of Net Bios.  This is 
        followed by assembly language calling procedures.  Then each Net 
        Bios command is described.  Finally, all tables are contained at 
        the end of the document.



                  --------
                  Net Bios
                  --------

        Net Bios provides a hardware-independent interface to a network 
        transport facility.  Two processes, on the same or different 
        machines, use the Net Bios to exchange data.  These processes 
        depend on the Net Bios to perform error checking, sequencing, and 
        routing.

        The services provided by the Net Bios can be broken into five 
        groups.

             1.   General.  Configuration and status.

             2.   Name.  A process or machine is identified by a name.  
                  Multiple names are supported by each machine.  Names 
                  may be added and deleted.

             3.   Session control.  A virtual connection, or session, can 
                  be established between processes or applications.  A 
                  process might establish multiple sessions with remote 
                  processes.  Commands are provided for waiting for a 
                  connection request from another (LISTEN), making a 
                  connection request (CALL), and terminating a connection 
                  (HANG UP).  Throughout this document, a virtual 
                  connection is referred to as a session.

             4.   Session data transfer.  Once a session is established, 
                  the Net Bios maintains tables that allow the process to 
                  specify the remote process for data transfer with a 
                  number, or handle.  Send and receive commands are 
                  provided.  In addition, a "receive any" command allows 
                  a process to receive messages from any of the sessions 
                  it has established.

             5.   Datagram.  Datagrams provide a means to transfer data 
                  between processes without using the session mechanism.  
                  Messages may be sent to a given name, a group name, or 


                                       1



                  to all processes.  These type messages do not provide 
                  the same reliability as session data transfer, since 
                  the sender will not be notified when messages are 
                  undeliverable.



                  -----
                  Names
                  -----

        The Net Bios supports user added names.  A given machine on the 
        network is known (i.e., can be found by other machines) by these 
        names.  Names are 16 characters long and should not contain the 
        asterisk (*) character.  Each Net Bios also has a permanent name, 
        which consists of 10 characters of binary zeros followed by 6 
        more characters which must be unique on the network (typically, 
        this "node number" is in PROM on a LAN card or configurable by 
        DIP switches).  Up to 16 user names may be added to the Net Bios.  
        These are kept in the "local name table".

        Data transfer and session establishment is normally associated 
        with one of the local names.  For example, a datagram may be 
        "sent to" a name. A session request (CALL) would be associated 
        with two names.  The name used by the requester and the name of 
        the process with which a session is desired.



                  ----------------
                  Calling Net Bios
                  ----------------

        Calling the Net Bios is accomplished by building a Net Bios 
        Control Block (NCB), and execute an INT 5Ch with registers ES:BX 
        containing the address of the NCB.  When control is returned, 
        only AX is changed.

        The NCB is 64 bytes long and is shown in Table 2.  The COMMAND 
        field describes the basic function of the command.  All commands 
        are listed in Table 1.  On selected commands, the high order bit 
        of the COMMAND field can be set to indicate the "no-wait" mode.  
        This feature, coupled with the value contained in the POST field, 
        results in three distinct ways that the Net Bios can signal 
        command completion:

             1.   No-wait bit not set.  In this case, the post address is 
                  ignored and control will not be returned from INT 5Ch 
                  until after the command has completed.  The RETCODE 
                  field and the AL register will contain the result of 
                  the operation (see Table 4 for error codes).

             2.   No-wait bit set and POST contains 0:0.  The Net Bios 
                  will (after minimum processing) return immediately, 
                  even though the command may not have completed.  On 


                                       2



                  return, the AL register will contain an "immediate 
                  error" code or 0.  If AL is not 0, then the command 
                  could not be queued (e.g., invalid COMMAND code).  If 
                  AL is 0, then the process should poll the CMD_DONE 
                  field of the NCB, waiting for some value other than 
                  0FFh.  When CMD_DONE is not equal to 0FFh, the command 
                  has completed and all other NCB fields are valid.

             3.   No-wait bit set and POST contains the address of a user 
                  routine.  The Net Bios will return immediately, even 
                  though the command may not have completed.  On return, 
                  the AL register will contain an "immediate error" code 
                  or 0.  If AL is not 0, then the command could not be 
                  queued (e.g., invalid COMMAND code).  If AL is 0, then 
                  the user's POST routine will be called after the 
                  command is completed.  When the POST routine is called, 
                  interrupts are off, the stack is set for an IRET, and 
                  ES:BX contains the address of the completed NCB. 

        Table 1 shows NCB COMMAND field codes, Table 3 shows field usage 
        by command, and Table 4 shows error codes returned in AL and 
        RETCODE.

        Typically, a POST routine can issue another Net Bios command.  
        However, some implementations may return an immediate error 21h 
        (interface busy) on nested interrupt conditions.  If the user is 
        issuing commands from either the POST routine or other hardware 
        routines, then the user must be prepared to queue NCBs that need 
        starting and start them from, say, a timer tick whenever an 
        interface busy error is encountered.  This same logic can also 
        cover the case where the Net Bios is temporarily out of resources 
        (immediate error 09h).  Normally, do not always queue the NCBs 
        for later starting (may dramatically reduce throughput), only do 
        it when the errors 21h or 09h are encountered.

        The user POST routine should be as short as possible and no 
        registers should be changed.  Interrupts may be enabled in the 
        POST routine.

        In some cases, the user's POST routine will be called before 
        control is returned from the INT 5Ch that started the command.
















                                       3



        A typical sequence of Net Bios commands is listed below.

             1. ADD NAME
             2. CALL
             3. SEND
             4. RECEIVE
             5. Repeat 3,4 as needed
             6. HANG UP
             7. DELETE NAME

        Depending on the design of the application, the process doing the 
        above may send a specific message to the other end informing it 
        of the intention to HANG UP.  This would not be necessary if the 
        other end was executing the following sequence.

             1. ADD NAME
             2. LISTEN
             3. RECEIVE
             4. If time-out goto 3
             5. If session lost goto 2
             6. SEND
             7. Goto 3
             


































                                       4



                  -------------
                  Reset Command
                  -------------

        The RESET command clears and configures the Net Bios.  The number 
        of sessions to be supported is supplied in the LSN field (1 to 
        32).  The maximum number of commands (NCBs) outstanding at one 
        time is specified in the NUM field (also 1 to 32).

        When this command is issued, all existing names and sessions are 
        lost, therefore it should be issued before starting any processes 
        that use the Net Bios.

        In some implementations, session and command buffers take space 
        otherwise used for packet buffers.  Therefore, keep these 
        parameters reasonable.



                  --------------
                  Cancel Command
                  --------------

        This command is used to cancel a previous command.  The address 
        of the NCB for the previous command is passed in BUFADR.  No-wait 
        mode is not allowed for the cancel command.  The following 
        commands cannot be canceled:  RESET, CANCEL, ADD NAME, ADD GROUP 
        NAME, DELETE NAME, SESSION STATUS, SEND DATAGRAM and SEND 
        BROADCAST.  SEND and CHAIN SEND may be canceled, but the session 
        will be terminated.




                  ----------------------
                  Adapter Status Command
                  ----------------------

        This command is used to get the status of a Net Bios -- either 
        local or remote.  CALLNAME specifies a name used to search for 
        the appropriate Net Bios.  If the first character of CALLNAME is 
        an asterisk (*), then status is returned for the local Net Bios.

        The status is placed in the buffer addressed by BUFADR.  The 
        format is shown in Table 5.  The maximum number of bytes returned 
        will be 348.  The actual number will be 60 + 18n, where n is the 
        number of names in the name table of the selected Net Bios (the 
        permanent node name is not counted).









                                       5



                  --------------
                  Unlink Command
                  --------------

        This command is used to cancel diskless boot redirection.  
        Typically, a diskless boot ROM makes a connection with a process 
        that supplies a virtual floppy image and redirects INT 13 
        requests for drive A to this network device.  This command 
        cancels that redirection so that the local drive A may be 
        accessed.  Once this command is issued, there is no way to re-
        establish the diskless boot connection.



                  ----------------
                  Add Name Command
                  ----------------

        This command adds a name to the Net Bios local name table.  The 
        name must be unique across the network.  The name, up to 16 
        characters long, is supplied in the NAME field.  If the operation 
        is successful, then the NUM field is used to return a number from 
        1 to 254 associated with the name.



                  ----------------------
                  Add Group Name Command
                  ----------------------

        This command adds a non-unique name to the Net Bios local name 
        table.  The name must not be added as a unique name by any other 
        process or machine.  The name, up to 16 characters long, is 
        supplied in the NAME field.  If the operation is successful, then 
        the NUM field is used to return a number from 1 to 254 associated 
        with the name.

        The main use of group names is to provide a mechanism to 
        "broadcast" messages to a group.  SEND DATAGRAM to a group name 
        will be received by all processes that have issued a RECEIVE 
        DATAGRAM under the group name.



                  -------------------
                  Delete Name Command
                  -------------------

        This command deletes a name from the Net Bios local name table.  
        If no sessions are active under the name to be deleted, the name 
        will be deleted.  If any sessions are active, then the command 
        will not complete until these sessions are terminated.

        Any of the following commands outstanding and associated with the 
        deleted command will be terminated with RETCODE = 17h (name was 


                                       6



        deleted): LISTEN, RECEIVE ANY, RECEIVE DATAGRAM, RECEIVE 
        BROADCAST.



                  ------------
                  Call Command
                  ------------

        This command is used to establish a session with another machine 
        or process.  The remote name is supplied in CALLNAME and the 
        local name is specified in NAME.  In addition, session data 
        transfer time-outs must be specified in RTO and STO.

        A LISTEN command must be outstanding for the called (remote) 
        name.  When the CALL completes successfully, a session number 
        (from 1 to 254) is returned in LSN.  This number is used when 
        sending or receiving session data, or in the HANG UP (session 
        terminate) command.



                  --------------
                  Listen Command
                  --------------

        This command is used to wait for a session to be established.  
        The session will be established when another machine or process 
        executes a CALL to the name associated with the LISTEN command.  
        The NAME field contains the local name under which the session 
        will be established.  The CALLNAME field contains the name of the 
        caller, or an asterisk (*) in the first character to accept a 
        call from any name.  Session data transfer time-outs must be 
        specified in RTO and STO.

        When the LISTEN completes successfully, a session number (from 1 
        to 254) is returned in LSN.  This number is used when sending or 
        receiving session data, or in the HANG UP (session terminate) 
        command.  If an asterisk (*) is used as the CALLNAME, then the 
        caller's name will be returned in CALLNAME upon successful 
        completion of the listen.

        More than one session can be established under the same name, or 
        same pair of names.

        If a CALL is received for a name for which there is a LISTEN with 
        a matching CALLNAME, and there is also a LISTEN for any name 
        (asterisk in first character), then the specific LISTEN will be 
        completed, regardless of the order in which the LISTENs were 
        issued.







                                       7



                  ---------------
                  Hang Up Command
                  ---------------

        This command is used to close or terminate a session.  It may be 
        issued by either the CALLer or LISTENer.  The LSN field contains 
        the session number of the session to be closed.

        If any RECEIVE commands for the session are outstanding on the 
        machine issuing the HANG UP, they are completed with RETCODE = 
        0Ah (session closed).  If SEND commands are outstanding, the HANG 
        UP completion will be delayed until the SEND has completed.

        If the remote end has any RECEIVE or SEND commands in progress 
        when HANG UP is started, they will be terminated with RETCODE = 
        0Ah.  If no remote RECEIVEs or SENDs are terminated with session 
        closed, and if RECEIVE ANY commands are outstanding at the 
        remote, one and only one of the RECEIVE ANYs will be terminated 
        with the session closed error.  If no commands are outstanding at 
        the remote machine when the HANG UP occurs, then the next session 
        command issued by the remote will be terminated RETCODE set to 
        either 08h (invalid session) or 0Ah (session closed).



                  ----------------------
                  Session Status Command
                  ----------------------

        This command is used to obtain status of all sessions associated 
        with a given name.  The NAME field contains the name for which 
        status is desired.  The BUFADR field contains the address of a 
        buffer for the status report and BUFLEN contains the buffer 
        length.  Table 6 shows the format of the status.  BUFLEN must be 
        at least 4.  If the buffer is not large enough to hold the 
        complete report, then it is filled and a RETCODE = 06h (message 
        incomplete is returned).



                  ------------
                  Send Command
                  ------------

        This command sends data to the machine or process with which a 
        session as been established.  The LSN field is set to the session 
        number, and BUFADR and BUFLEN are set to the size and length of 
        the data to be transferred.  The remote session must have issued 
        (or issue before the time-out specified earlier in STO) a RECEIVE 
        or RECEIVE ANY command.

        SENDs are performed in the order in which they are issued.  If an 
        error occurs on the send, then the session is terminated.




                                       8



                  ------------------
                  Chain Send Command
                  ------------------

        The CHAIN SEND command is similar to the SEND command, except the 
        data to be transferred is contained in two separate buffers.  
        CHAIN send concatenates these buffers -- the receiving end will 
        see one message.  The first part of the message is defined by 
        BUFADR and BUFLEN.  The address of the second part is stored in a 
        double word at CALLNAME+2.  The length of the second part is 
        stored in a word at CALLNAME+0.  The total length cannot exceed 
        65535 bytes.



                  ---------------
                  Receive Command
                  ---------------

        This command receives data from the machine or process with which 
        a session as been established.  The LSN field is set to the 
        session number, and BUFADR and BUFLEN are set to the size and 
        maximum length of the receive data buffer.  The remote session 
        must issue a SEND or CHAIN SEND command.  If a send is not issued 
        by the remote within the time specified earlier in RTO, the 
        RECEIVE will complete with a time out error (RETCODE = 05h).   
        Time outs do not cause session termination. 

        RECEIVE commands are completed in the order in which they are 
        issued.  If received data could be used to complete either a 
        RECEIVE or RECEIVE ANY, the RECEIVE is given priority and 
        completed.

        The actual size of the received data is returned in BUFLEN.  If 
        the maximum buffer length was smaller than the length of the 
        data, then the buffer is filled to the maximum and RETCODE = 06h 
        is returned.  The next RECEIVE (or RECEIVE ANY) will be completed 
        with the remainder of the data.



                  -------------------
                  Receive Any Command
                  -------------------

        This command receives data from any session.  The NUM field 
        specifies the name number of allowable sessions, or 0FFh to 
        receive for any session under any name.  The BUFADR and BUFLEN 
        fields are set to the size and maximum length of the receive data 
        buffer.  The remote session must issue a SEND or CHAIN SEND 
        command.  There is no time-out on this command.  The local 
        session number of a completed receive is returned in LSN.

        RECEIVE ANY commands are completed in the order in which they are 
        issued.  If received data could be used to complete either a 


                                       9



        RECEIVE or RECEIVE ANY, the RECEIVE is given priority and 
        completed.

        The actual size of the received data is returned in BUFLEN.  If 
        the maximum buffer length was smaller than the length of the 
        data, then the buffer is filled to the maximum and RETCODE = 06h 
        is returned.  The next RECEIVE ANY (or RECEIVE) will be completed 
        with the remainder of the data.

        Typically, RECEIVE ANY is used by a process that services 
        multiple users.  Session termination by the local or remote 
        process will cause the next pending RECEIVE ANY to complete with 
        RETCODE = 0Ah (session terminated) or 18h (session terminated 
        abnormally).



                  ---------------------
                  Send Datagram Command
                  ---------------------

        This command is used to send a datagram message.  The Net Bios 
        does not necessarily detect whether the message was successfully 
        delivered.  The NUM field associates the datagram with a local 
        name.  The CALLNAME field specifies the destination name (may be 
        a group name).  BUFADR and BUFLEN contain the address and length 
        of the message.  Maximum allowable length is 512 bytes.



                  ------------------------
                  Receive Datagram Command
                  ------------------------

        This command waits for a SEND DATAGRAM to the name associated 
        with the NUM field (may be a group name). An 0FFh in the NUM 
        field is used to receive all datagrams to the machine.  The 
        BUFADR and BUFLEN fields are set to the size and maximum length 
        of the receive data buffer.  There is no time-out on this 
        command.  The name of the sender is returned in CALLNAME.

        The actual size of the received data is returned in BUFLEN.  If 
        the maximum buffer length was smaller than the length of the 
        data, then the buffer is filled to the maximum and RETCODE = 06h 
        is returned.  The remaining data is lost. 












                                       10



                  ----------------------
                  Send Broadcast Command
                  ----------------------

        This command is used to send a broadcast message.  The Net Bios 
        does not necessarily detect whether the message was successfully 
        delivered.  The NUM field associates the datagram with a local 
        name.  BUFADR and BUFLEN contain the address and length 
        of the message.  Maximum allowable length is 512 bytes.

        Each SEND BROADCAST satisfies all outstanding RECEIVE BROADCAST 
        commands on the network.



                  -------------------------
                  Receive Broadcast Command
                  -------------------------

        This command waits for a SEND BROADCAST message.  The NUM field 
        must contain a valid name number.  The BUFADR and BUFLEN fields 
        are set to the size and maximum length of the receive data 
        buffer.  There is no time-out on this command.  The name of the 
        sender is returned in CALLNAME.

        Each SEND BROADCAST satisfies all outstanding RECEIVE BROADCAST 
        commands on the network.

        The actual size of the received data is returned in BUFLEN.  If 
        the maximum buffer length was smaller than the length of the 
        data, then the buffer is filled to the maximum and RETCODE = 06h 
        is returned.  The remaining data is lost. 

























                                       11



                                     Table 1
                            Net Bios Command Summary
        -----------------------------------------------------------------
        | Command | Command Code|No Wait |                              |
        | Name    ----------Hex |Code-----     Description              |
        |---------------------------------------------------------------|
        |                        General Commands                       |
        |---------------------------------------------------------------|
        | RESET            | 32 | -- | Reset Net Bios.                  |
        |------------------+----+----+----------------------------------|
        | CANCEL           | 35 | -- | Cancel a pending command.        |
        |------------------+----+----+----------------------------------|
        | ADAPTER STATUS   | 33 | B3 | Get status of a Net Bios.        |
        |------------------+----+----+----------------------------------|
        | UNLINK           | 70 | -- | Cancel boot redirection.         |
        |---------------------------------------------------------------|
        |                         Name Commands                         |
        |---------------------------------------------------------------|
        | ADD NAME         | 30 | B0 | Add unique name to name table.   |
        |------------------+----+----+----------------------------------|
        | ADD GROUP NAME   | 36 | B6 | Add non-unique name to table.    |
        |------------------+----+----+----------------------------------|
        | DELETE NAME      | 31 | B1 | Delete name from name table.     |
        |---------------------------------------------------------------|
        |                   Session Control Commands                    |
        |---------------------------------------------------------------|
        | CALL             | 10 | 90 | Establish session with another.  |
        |------------------+----+----+----------------------------------|
        | LISTEN           | 11 | 91 | Wait for a CALL from another.    |
        |------------------+----+----+----------------------------------|
        | HANG UP          | 12 | 92 | Close session.                   |
        |------------------+----+----+----------------------------------|
        | SESSION STATUS   | 34 | B4 | Status of sessions under name.   |
        |---------------------------------------------------------------|
        |                Session Data Transfer Commands                 |
        |---------------------------------------------------------------|
        | SEND             | 14 | 94 | Send session data.               |
        |------------------+----+----+----------------------------------|
        | CHAIN SEND       | 17 | 97 | Concatenate and send two buffers.|
        |------------------+----+----+----------------------------------|
        | RECEIVE          | 15 | 95 | Receive session data.            |
        |------------------+----+----+----------------------------------|
        | RECEIVE ANY      | 16 | 96 | Receive data from any session    |
        |                  |    |    | under specified name.            |
        |---------------------------------------------------------------|
        |                       Datagram Commands                       |
        |---------------------------------------------------------------|
        | SEND DATAGRAM    | 20 | A0 | Send data, addressed by name.    |
        |------------------+----+----+----------------------------------|
        | RECEIVE DATAGRAM | 21 | A1 | Receive datagram to name.        |
        |------------------+----+----+----------------------------------|
        | SEND BROADCAST   | 22 | A2 | Send data to all stations.       |
        |------------------+----+----+----------------------------------|
        | RECEIVE BROADCAST| 23 | A3 | Enable receive of next broadcast.|
        -----------------------------------------------------------------


                                       12



                                     Table 2
                       Net Bios Control Block (NCB) Format
        -----------------------------------------------------------------
        |Off-|Size |  Field  |                Field                     |
        |set |Bytes|  Name   |             Description                  |
        |----+-----+---------+------------------------------------------|
        |  0 |  1  | COMMAND | Net Bios command code.  High order bit   |
        | Hex|     |         | set indicates no-wait mode.              |
        |----+-----+---------+------------------------------------------|
        |  1 |  1  | RETCODE | Completion result.  Zero if no error.    |
        |----+-----+---------+------------------------------------------|
        |  2 |  1  | LSN     | Local session number (1 to 254). Returned|
        |    |     |         | by CALL and LISTEN, and supplied for     |
        |    |     |         | SEND and RECEIVE commands.               |
        |----+-----+---------+------------------------------------------|
        |  3 |  1  | NUM     | Name Number (1 to 254). Returned by ADD  |
        |    |     |         | NAME commands, and supplied for RECEIVE  |
        |    |     |         | ANY and datagram commands.               |
        |----+-----+---------+------------------------------------------|
        |  4 |  4  | BUFADR  | Address of message for send and receive. |
        |----+-----+---------+------------------------------------------|
        |  8 |  2  | BUFLEN  | Length of message buffer. For receive    |
        |    |     |         | commands, supply the maximum buffer size |
        |    |     |         | and the actual length is returned.       |
        |----+-----+---------+------------------------------------------|
        | 0A | 16  | CALLNAME| Name from/of remote machine for CALL,    |
        |    | Dec |         | LISTEN and datagrams.  For a CHAIN SEND  |
        |    |     |         | command, the first word specifies the    |
        |    |     |         | length of the second buffer, and the     |
        |    |     |         | next two words specify the address.      |
        |----+-----+---------+------------------------------------------|
        | 1A | 16  | NAME    | Local name. Specifies name to add for ADD|
        |    |     |         | NAME, or name to use for other commands. |
        |----+-----+---------+------------------------------------------|
        | 2A |  1  | RTO     | Receive time-out in .5 second increments.|
        |    |     |         | Supplied on CALL and LISTEN commands.    |
        |----+-----+---------+------------------------------------------|
        | 2B |  1  | STO     | Send time-out in .5 second increments.   |
        |    |     |         | Supplied on CALL and LISTEN commands.    |
        |----+-----+---------+------------------------------------------|
        | 2C |  4  | POST    | Address of user interrupt routine called |
        |    |     |         | when command completes and no-wait mode  |
        |    |     |         | mode was specified in command code. Not  |
        |    |     |         | called if set to 0:0.                    |
        |----+-----+---------+------------------------------------------|
        | 30 |  1  | LANA_NUM| Number of adapter card -- 0 for the first|
        |    |     |         | and 1 for the second, if applicable.     |
        |----+-----+---------+------------------------------------------|
        | 31 |  1  | CMD_DONE| Command completed flag. A value of 0FFH  |
        |    |     |         | indicates the command has not completed. |
        |    |     |         | When the command has completed, CMD_DONE |
        |    |     |         | is set to same value as RETCODE.         |
        |----+-----+---------+------------------------------------------|
        | 32 | 14  | RES     | Used internally by Net Bios.             |
        -----------------------------------------------------------------


                                       13



                                     Table 3
                              NCB Usage by Command
        -----------------------------------------------------------------
        |               | C | R | L | N | B | B | C | N | R | P | L | C |
        |               | O | E | S | U | U | U | A | A | T | O | A | M |
        |               | M | T | N | M | F | F | L | M | O | S | N | D |
        |               | M | C |   |   | A | L | L | E | & | T | A | _ |
        |               | A | O |   |   | D | E | N |   | S |   | _ | D |
        |               | N | D |   |   | R | N | A |   | T |   | N | O |
        |               | D | E |   |   |   |   | M |   | O |   | U | N |
        |               |   |   |   |   |   |   | E |   |   |   | M | E |
        |               |   |   |   |   |   |   |   |   |   |   |   |   |
        |    Command    |   |   |   |   |BUF|BUF|CAL|   |RTO|   |LAN|CMD|
        |      Name     |CMD|RET|LSN|NUM|ADR|LEN|NAM|NAM|STO|PST|NUM|DON|
        |---------------+---+---+---+---+---+---+---+---+---+---+---+---|
        |RESET          | I | O | I | I | - | - | - | - | - | - | I | O |
        |CANCEL         | I | O | - | - | I | - | - | - | - | - | I | O |
        |ADAPTER STATUS | I | O | - | - | I |I/O| I | - | - | I | I | O |
        |UNLINK         | I | O | - | - | - | - | - | - | - | - | I | O |
        |---------------+---+---+---+---+---+---+---+---+---+---+---+---|
        |ADD NAME       | I | O | - | O | - | - | - | I | - | I | I | O |
        |ADD GROUP NAME | I | O | - | O | - | - | - | I | - | I | I | O |
        |DELETE NAME    | I | O | - | - | - | - | - | I | - | I | I | O |
        |---------------+---+---+---+---+---+---+---+---+---+---+---+---|
        |CALL           | I | O | O | - | - | - | I | I | I | I | I | O |
        |LISTEN         | I | O | O | - | - | - |I/O| I | I | I | I | O |
        |HANG UP        | I | O | I | - | - | - | - | - | - | I | I | O |
        |SESSION STATUS | I | O | - | - | I |I/O| - | I | - | I | I | O |
        |---------------+---+---+---+---+---+---+---+---+---+---+---+---|
        |SEND           | I | O | I | - | I | I | - | - | - | I | I | O |
        |CHAIN SEND     | I | O | I | - | I | I | I | - | - | I | I | O |
        |RECEIVE        | I | O | I | - | I |I/O| - | - | - | I | I | O |
        |RECEIVE ANY    | I | O | O | I | I |I/O| - | - | - | I | I | O |
        |---------------+---+---+---+---+---+---+---+---+---+---+---+---|
        |SEND DATAGRAM  | I | O | - | I | I | I | I | - | - | I | I | O |
        |RECV DATAGRAM  | I | O | - | I | I |I/O| O | - | - | I | I | O |
        |SEND BROADCAST | I | O | - | I | I | I | - | - | - | I | I | O |
        |RECV BROADCAST | I | O | - | I | I |I/O| O | - | - | I | I | O |
        -----------------------------------------------------------------

             Legend
                I  =  Field is input (passed to Net Bios)
                O  =  Field is output (returned by Net Bios)
               I/O =  Field is used for both input and output













                                       14



                                     Table 4
                              Net Bios Error Codes
        -----------------------------------------------------------------
        | Code |                   Description                          |
        |------+--------------------------------------------------------|
        |  00  | No error.                                              |
        |------+--------------------------------------------------------|
        |  01  | Illegal buffer length.  A SEND BROADCAST or SEND       |
        |      | DATAGRAM command specified a length greater than 512   |
        |      | bytes, or a status command specified a buffer length   |
        |      | smaller than minimum allowed.                          |
        |------+--------------------------------------------------------|
        |  03  | Invalid command.                                       |
        |------+--------------------------------------------------------|
        |  05  | Time out.  For SEND, RECEIVE, and HANG UP commands, the|
        |      | time-out specified when the session was established has|
        |      | elapsed.  For a CALL or ADAPTER STATUS command, an     |
        |      | internal timer expired.                                |
        |------+--------------------------------------------------------|
        |  06  | Message Incomplete.  The buffer size specified in the  |
        |      | NCB was not large enough to hold the receive data. For |
        |      | RECEIVE or RECEIVE ANY commands, the next command will |
        |      | get the rest of the data.  For other commands, the     |
        |      | remaining data is lost.                                |
        |------+--------------------------------------------------------|
        |  08  | Invalid local session number (LSN).                    |
        |------+--------------------------------------------------------|
        |  09  | Out of resources.  The Net Bios is out of some internal|
        |      | resource, such as buffers.  Delay and reissue the      |
        |      | command.                                               |
        |------+--------------------------------------------------------|
        |  0A  | Session closed.  For a SEND, RECEIVE, RECEIVE ANY, or  |
        |      | HANG UP, this indicates that the session was terminated|
        |      | by the remote computer.                                |
        |------+--------------------------------------------------------|
        |  0B  | Command canceled.  Command execution of the NCB was    |
        |      | aborted by the CANCEL command.                         |
        |------+--------------------------------------------------------|
        |  0D  | Duplicate local name.  An ADD NAME command specified   |
        |      | an existing name.                                      |
        |------+--------------------------------------------------------|
        |  0E  | Name table full.                                       |
        |------+--------------------------------------------------------|
        |  0F  | DELETE NAME completed, but name has active sessions    |
        |      | (name will be deleted when all sessions closed).       |
        |------+--------------------------------------------------------|
        |  11  | Local session table full.                              |
        -----------------------------------------------------------------
                              (Continued Next Page)








                                       15



                               Table 4 (Continued)
                              Net Bios Error Codes
        -----------------------------------------------------------------
        | Code |                   Description                          |
        |------+--------------------------------------------------------|
        |  12  | Remote computer not listening.  On a CALL, the remote  |
        |      | computer was found, but had no outstanding LISTEN for  |
        |      | the CALL.                                              |
        |------+--------------------------------------------------------|
        |  13  | Invalid name number.                                   |
        |------+--------------------------------------------------------|
        |  14  | Name not found.                                        |
        |------+--------------------------------------------------------|
        |  15  | Name not found or "*" or 00h in first byte of remote   |
        |      | name field on a CALL.                                  |
        |------+--------------------------------------------------------|
        |  16  | Name already exists on network.                        |
        |------+--------------------------------------------------------|
        |  17  | Name was deleted.                                      |
        |------+--------------------------------------------------------|
        |  18  | Session terminated abnormally.  Connection with the    |
        |      | remote computer was lost.                              |
        |------+--------------------------------------------------------|
        |  19  | Name conflict.  Two computers using the same name was  |
        |      | detected.                                              |
        |------+--------------------------------------------------------|
        |  21  | Interface busy.  The Net Bios cannot execute because   |
        |      | it was called from an interrupt handler.               |
        |------+--------------------------------------------------------|
        |  22  | Too many commands issued.                              |
        |------+--------------------------------------------------------|
        |  23  | Invalid LAN adapter (LANA) number.                     |
        |------+--------------------------------------------------------|
        |  24  | Command completed before canceled.  Returned in CANCEL |
        |      | NCB when target command completed normally.            |
        |------+--------------------------------------------------------|
        |  26  | Invalid cancel command.  The target NCB could not be   |
        |      | found.                                                 |
        |------+--------------------------------------------------------|
        |40-FE | Hardware error.                                        |
        |------+--------------------------------------------------------|
        |  FF  | Indicates the command has not completed.               |
        -----------------------------------------------------------------














                                       16



                                     Table 5
                          Adapter Status Result Buffer
        -----------------------------------------------------------------
        | Off- | Size  |                  Field                         |
        | set  | Bytes |                Description                     |
        |------+-------+------------------------------------------------|
        |   0  |   6   | Permanent node name.  This six byte identifier |
        |(Hex) | (Dec) | is obtained from either a ROM on the LAN card  |
        |      |       | or from the node address DIP switch.           |
        |------+-------+------------------------------------------------|
        |   6  |   1   | External jumper status.  The high bit of this  |
        |      |       | byte indicates the interrupt number used by    |
        |      |       | the LAN card -- 0 for IRQ 2 and 1 for IRQ 3.   |
        |      |       | The second highest bit indicates the DMA chan- |
        |      |       | nel used by the LAN card -- 0 for channel 1    |
        |      |       | and 1 for channel 3.  This byte is not sup-    |
        |      |       | ported in all implementations.                 |
        |------+-------+------------------------------------------------|
        |   7  |   1   | Power on test result.  Always zero in current  |
        |      |       | versions.                                      |
        |------+-------+------------------------------------------------|
        |   8  |   2   | Software version.  First byte is the major     |
        |      |       | version, second is the minor version number.   |
        |------+-------+------------------------------------------------|
        |  0A  |   2   | Minutes since system started.  When this field |
        |      |       | reaches 0FFFFH, it rolls over to 0.            |
        |------+-------+------------------------------------------------|
        |  0C  |   2   | Number of CRC errors on received packets. (1)  |
        |------+-------+------------------------------------------------|
        |  0E  |   2   | Number of alignment errors. (1)                |
        |------+-------+------------------------------------------------|
        |  10  |   2   | Number of transmit collisions.  (1)            |
        |------+-------+------------------------------------------------|
        |  12  |   2   | Number of aborted transmits.  (1)              |
        |------+-------+------------------------------------------------|
        |  14  |   4   | Number of packets transmitted.  When this      |
        |      |       | field reaches 0FFFFFFFFH, it rolls over to 0.  |
        |------+-------+------------------------------------------------|
        |  18  |   4   | Number of packets received.  When this field   |
        |      |       | reaches 0FFFFFFFFH, it rolls over to 0.        |
        |------+-------+------------------------------------------------|
        |  1C  |   2   | Number of retransmits. (1)                     |
        |------+-------+------------------------------------------------|
        |  1E  |   2   | Number of times receiver was out of buffers.   |
        |      |       | (1)                                            |
        |------+-------+------------------------------------------------|
        |  20  |   8   | Not used -- reserved.                          |
        -----------------------------------------------------------------
                              (Continued Next Page)








                                       17



                               Table 5 (Continued)
                          Adapter Status Result Buffer
        -----------------------------------------------------------------
        | Off- | Size  |                  Field                         |
        | set  | Bytes |                Description                     |
        |------+-------+------------------------------------------------|
        |  28  |   2   | Number of free network command blocks (NCBs).  |
        | (Hex)| (Dec) | (2)                                            |
        |------+-------+------------------------------------------------|
        |  2A  |   2   | Number of NCBs specified in last RESET command.|
        |------+-------+------------------------------------------------|
        |  2C  |   2   | Maximum possible number of NCBs that can be    |
        |      |       | specified in RESET command.                    |
        |------+-------+------------------------------------------------|
        |  2E  |   4   | Not used -- reserved.                          |
        |------+-------+------------------------------------------------|
        |  32  |   2   | Number of pending or active sessions.          |
        |------+-------+------------------------------------------------|
        |  34  |   2   | Number of possible sessions specified in last  |
        |      |       | RESET command.                                 |
        |------+-------+------------------------------------------------|
        |  36  |   2   | Maximum number of possible sessions that can   |
        |      |       | be specified in RESET command.                 |
        |------+-------+------------------------------------------------|
        |  38  |   2   | Maximum packet size supported on the network.  |
        |      |       | Note, this is not related to the maximum       |
        |      |       | session message size, which is 64K bytes.      |
        |------+-------+------------------------------------------------|
        |  3A  |   2   | Number of names in name table.                 |
        |------+-------+------------------------------------------------|
        |  3C  |  18   | First name in name table. (3)                  |
        |      |       |   * 16 bytes - name                            |
        |      |       |   *  1 byte  - name number (2 to 254)          |
        |      |       |   *  1 byte  - name status.  Bit patterns are: |
        |      |       |        *  G----000 - name add in progress      |
        |      |       |        *  G----100 - active name               |
        |      |       |        *  G----101 - delete pending            |
        |      |       |        *  G----110 - improper duplicate name   |
        |      |       |        *  G----111 - duplicate name, delete    |
        |      |       |                      pending                   |
        |------+-------+------------------------------------------------|
        |  4E  | X18   | Addition name table entries as needed.         |
        -----------------------------------------------------------------

        Notes
          1  These fields will not increment further after reaching 
             0FFFFH.
          2  In some implementations, there is no limit on the number of 
             NCBs pending, so an arbitrary large number is used in these 
             fields.
          3  The permanent name (name number 1) does not appear in the 
             name table.





                                       18



                                     Table 6
                          Session Status Result Buffer
        -----------------------------------------------------------------
        | Off- | Size  |                  Field                         |
        | set  | Bytes |                Description                     |
        |------+-------+------------------------------------------------|
        |   0  |   1   | Sessions' name number.                         |
        |------+-------+------------------------------------------------|
        |   1  |   1   | Number of sessions under name.                 |
        |------+-------+------------------------------------------------|
        |   2  |   1   | Number of RECEIVE DATAGRAM and RECEIVE         |
        | (Hex)| (Dec) | BROADCAST commands outstanding.                |
        |------+-------+------------------------------------------------|
        |   3  |   1   | Number of RECEIVE ANY commands outstanding.    |
        |------+-------+------------------------------------------------|
        |  (4) | (36)  | Session status - first session (if at least 1).|
        |   4  |   1   |  * Local session number (LSN).                 |
        |   5  |   1   |  * Session state                               |
        |      |       |     1: LISTEN pending   4: HANG UP pending     |
        |      |       |     2: CALL pending     5: HANG UP complete    |
        |      |       |     3: Active           6: Session aborted     |
        |   6  |  16   |  * Local name (NAME)                           |
        |  16  |  16   |  * Remote name (CALLNAME)                      |
        |  26  |   1   |  * Number of RECEIVE commands outstanding      |
        |  27  |   1   |  * Number of SEND and CHAIN SEND commands      |
        |      |       |    outstanding                                 |
        |------+-------+------------------------------------------------|
        |  28  |  X36  | Session status for additional sessions as      |
        |      |       | needed                                         |
        -----------------------------------------------------------------



























                                       19

