# Installation

1. Download 'smtp???.exe' and execute it on Windows95/98/NT/2000/XP to extract 'smtp???.savf'.
1. Create save-file on your AS/400.
1. Connect to the AS/400 using any FTP client software.
1. Put the distribution (smtp???.savf) to the save-file in binary mode.
1. Create/restore object to library 'SMTP' from the save-file using RSTOBJ command.

## Sample instruction

Create save-file on the target AS/400:

```
> CRTSAVF FILE(QGPL/SMTP)
  File SMTP created in library QGPL.
```

Send save-file from Windows95/98/NT/2000 command prompt:

```
C:\>ftp your_as400_hostname
Connected to ????.
220-QTCP AT ????.
220 CONNECTION WILL CLOSE IF IDLE MORE THAN 5 MINUTES.
User (????:(none)): as400_user_id
331 ENTER PASSWORD.
Password:enter_password_for_the_user_id
230 ???? LOGGED ON.
ftp> bi
200 REPRESENTATION TYPE IS BINARY IMAGE.
ftp> put smtp???.savf qgpl/smtp
200 PORT SUBCOMMAND REQUEST SUCCESSFUL.
150 SENDING FILE TO MEMBER SMTP IN FILE SMTP IN LIBRARY QGPL.
250 FILE TRANSFER COMPLETED SUCCESSFULLY.
ftp: 225984 bytes sent in 34.44Seconds 6.56Kbytes/sec.
ftp> quit
221 QUIT SUBCOMMAND RECEIVED.
```

Go back to the 5250 session to restore objects:

```
> CRTLIB LIB(SMTP) TEXT('ILE-RPG SMTP client')
  Library SMTP created.
> RSTOBJ OBJ(*ALL) SAVLIB(SMTP) DEV(*SAVF) SAVF(QGPL/SMTP)
  5 objects restored from SMTP to SMTP.
> DLTF FILE(QGPL/SMTP)
  Object SMTP in QGPL type *FILE deleted.
> CHGCURLIB CURLIB(SMTP)
  Current library changed to SMTP.
> STRREXPRC SRCMBR(MAKE)
Start SNDM compilation.
 You must have authority to exec CRTxxx.
 Confirm that you have IBM supplied library QSYSINC.
 This procedure changes job's CURLIB before compilation.
* * * * * * * * * * * * * * * * * * * * * *

Change current library to "SMTP"...
  result-> 0

Start creating modules for SNDM...
  creating module SNDM...
  result-> 0
  creating module CDATE...
  result-> 0
  creating module FD_...
  result-> 0
  creating module GETERRINFO...
  result-> 0
  creating module GETFILENAM...
  result-> 0
...........(SNIP).............
  creating module PREPAREUS...
  result-> 0
  creating module SNDPM...
  result-> 0
  Creating main program (SNDM)...
  result-> 0

Start creating command ...
  creating pnlgrp...
  result-> 0
  creating command...
  result-> 0

Compile finished. Confirm error(s) if exists.
```

<p></p>
<br>

---

# Usage

```
                      Send SMTP Mail (SNDM) - Help

    The Send SMTP Mail (SNDM) command allows you to send a Internet
    compliant mail from AS/400 command line.

    Some of the specific functions that can be performed by the SNDM
    command include the following:

     o  US-ASCII, Latin-1 (German, French, etc.) or Japanese can be used
        as mail header and mail body text.

     o  SNDM command talks with SMTP (mail) server directly by TCP/IP
        socket interface. You should enable TCP/IP and configure
        domain/host name on the AS/400 before running SNDM command.
        Also you should configure your AS/400 to use DNS (Domain Name
        System) server or 'hosts' file for SNDM command to resolve IP
        address from TCP/IP host name.

     o  In default, SNDM command assumes the same AS/400 running the
        command as SMTP server. In this case, you should configure the
        AS/400 as mail (SMTP) server.  If you already have SMTP server
        running on other host, you can use it by specifying the server
        at SMTPHOST parameter.

     o  System date/time is referred when generating date filed in mail
        header.  Make sure that QDATE, QTIME, QUTCOFFSET system values
        are set correctly.

     o  SNDM command creates temporary stream file in directory
        specified at TMPDIR parameter.  The path name will be:
        '/(tmpdir)/SNDM_(job number)-(user name)-(job
        name)_(date).(time).TXT' For example:
        '/tmp/SNDM_90605-TCPIP-QPADEV0006_1999-01-20-17.33.19.215.TXT'

        The temporary file reamins if you specify DEBUG(*YES).  Even you
        specify DEBUG(*NO), the temporary file might not be removerd by
        SNDM command automaically if SNDM command ends abnormally.  Use
        WRKLNK command or any other IFS-accessible function to delete
        the file when you don't need the temporary file.

     o  See low level message or joblog when error occured.  SNDM
        command creates spooled file QPRINT which records all activities
        in the command when you specify DEBUG(*YES).

     o  Wait time (time out interval) depends on the socket
        communication phase.  Usually, SNDM command times out in 1 to 3
        minutes when SMTP server doesn't respond.  If you are sending
        small mail but SNDM doesn't finish for several minutes, cancel
        the command manually.

     o  Refer to RFC 821, 822, 1468, 2045, 2046, 2047, 2048, 2049

    Restrictions:

      1.  Mail body text (physical file member specified at FILE
          parameter) and attachment files (stream file(s) specified at
          ATTACHMENT parameter) can be sent. Attachment files cannot be
          objects in QSYS.LIB file system.

      2.  Maximum number of recipients ('to' and 'cc' and 'bcc') is 30
          due to the restiriction of AS/400 CPP (maximum length of
          command string = 6000 bytes).  Maximum field length of mail
          addr, description, subject are 64 bytes respectively.

      3.  'Content-type' of mail body will be set to 'text/plain'.
          Other text formats such as HTML or RTF are not supported.  For
          attachment files, 'Contents-type' will be set to
          'application/octet-stream'.

      4.  For (safe) US-ASCII characters, no encoding will take place.
          For unsafe ASCII and 8bit ISO-8859-1 characters, Q-encoding
          will be used to encode mail header and quoted-printable
          encoding will be used for mail body.  For Japanese characters,
          B-encoding will be used to encode mail header. Body text will
          be converted to ISO-2022-JP.  For attachment files, Base64
          encoding is always used.  Other MIME encodings such as nesting
          are not supported.

      5.  EBCDIC Japanese CCSIDs (5026/5035/1390/1399) will be converted
          to ISO-2022-JP.  You cannot send a mail by other Japanese
          codes such as Shift-JIS(x-sjis) or EUC.  SBCS katakana will be
          replaced by DBCS katakana.  Non-JIS defined DBCS characters
          will be replace by thick '=' character or SNDM terminates in
          error depending on NONJISDBCS parameter.

        Note:  Do not precede an entry with an asterisk unless that
        entry is a "special value" that is shown (on the display or in
        the help information) with an asterisk.

Error messages for SNDM

    *ESCAPE Messages
    CPF9897     Command failed.
    CPF9898     Mail sent, non-JIS character(s) replaced

Sender (FROM)

    Specifies sender.

    This is a required parameter.

    Mail Address
        Specify the internet mail address of a person or organization.
        Up to 64 characters can be specified.  Quote by
        single-quotateion (') to enter lower case character.  DBCS
        characters are not allowed.

    Description
        Specify mail user information such as sender name or
        organization name.  Japanese (DBCS) and special characters are
        allowed.

Recipient (To/Cc/Bcc) (TO)

    Specifies recipients. Maximum number of recipients is 30.

    This is a required parameter.

    Mail Address
        Specify the internet mail address of a person or organization.
        Up to 64 characters can be specified.  Quote by
        single-quotateion (') to enter lower case character.  DBCS
        characters are not allowed.

    Description
        Specify mail user information such as sender name or
        organization name.  Japanese (DBCS) and special characters are
        allowed.

    The possible Recipient Type values are:

    *TO
        The user is the primary recipient of the mail.

    *CC
        The user is receiving a copy of the mail sent to the primary
        recipient.  However, this copy recipient is not identified on
        the distribution as a receiver on the distribution.

    *BCC
        The user is receiving a copy of the mail sent to the primary
        recipient.  However, this copy recipient is not identified on
        the distribution as a receiver on the distribution.

    You can enter multiple values for this parameter. If you are on an
    entry display and you need additional entry fields to enter these
    multiple values, type a plus sign (+) in the entry field opposite
    the phrase "+ for more" and press the Enter key.

Mail body file (FILE)

    Specify database file and library name which contains mail body
    text.  Database file should be source physical file with record
    length 92 or data physical file with record length 80.

    This is a required parameter.

    The possible library values are:

    *LIBL
        All libraries in the user and system portions of the job's
        library list are searched until the first match is found.

    *CURLIB
        The current library for the job is used to locate the database
        file or device file.  If no library is specified as the current
        library for the job, the QGPL library is used.

    library-name
        Specify the name of the library to be searched.

    file-name
        Specify the name of the database or device file that contains
        the mail body text.

Member (MBR)

    Specifies the database file member name which contains the mail body
    text to be sent.

    The possible values are:

    *FIRST
        The first member in the database file is used as mail body text.

    member-name
        Specify the name of the database file member.

Subject (SUBJECT)

    Specify mail subjet (title). This is a optional parameter but it is
    recommended to specify mail subject.  A maximum of 64 characters can
    be specified and any characters are allowed.

Attachment (ATTACHMENT)

    Specifies the name of the stream file that will be sent as
    attachment file.  A maximum of 5 path names can be specified. DBCS
    characters are not allowed.  For more information on specifying path
    names, refer to Chapter 2 of the CL Reference, SC41-5722.

    You can enter multiple values for this parameter. If you are on an
    entry display and you need additional entry fields to enter these
    multiple values, type a plus sign (+) in the entry field opposite
    the phrase "+ for more" and press the Enter key.

Reply-to (REPLYTO)

    Specify reply mail address. If this parameter is specified, most
    mail clients will send reply to this mail address, not to sender.

    Mail Address
        Specify the internet mail address of a person or organization.
        Up to 64 characters can be specified.  Quote by
        single-quotateion (') to enter lower case character.  DBCS
        characters are not allowed.

    Description
        Specify mail user information such as sender name or
        organization name.  Japanese (DBCS) and special characters are
        allowed.

Mail (SMTP) server name (SMTPHOST)

    Specify mail (SMTP) server with which SNDM command will communicate.

    The possible values are:

    *LOCALHOST
        Assume the AS/400 running SNDM command as SMTP server. The
        AS/400 must be configured and running as SMTP server.

    host-name
        Specify mail (SMTP) server TCP/IP host name.

Non-JIS character action (NONJISDBCS)

    Specify how SNDM command should act when it detects undefined JIS
    character in mail header or mail body text.  This applies only to
    Japanese (CCSID 5026/5035/1390/1399) environment.

    The possible values are:

    *ABORT
        Command aborts, mail not sent.

    *REPLACE
        Replace undefined JIS character(s) to thick DBCS '=' and send
        mail.

Message header CCSID (HDRCCSID)

    Specifies the method of obtaining the job CCSID to encode mail
    header.

    The possible values are:

    *DFTJOBCCSID
        The default job CCSID is used.

    coded-character-set-identifier
        Specify valid EBCDIC CCSID.

Datebase file CCSID (DBFCCSID)

    Specifies the method of obtaining the mail body text file CCSID.

    The possible values are:

    *DFTJOBCCSID
        The default job CCSID is used.

    *FILE
        The database file CCSID is used, unless it is 65535.

    coded-character-set-identifier
        Specify valid EBCDIC CCSID.

Work directory (TMPDIR)

    Specify directory in which the command creates temporary file.  Root
    ('/') directory cannot be specified.

    The possible values are:

    '/TMP'
        Directory '/TMP' is used as working directory.

    directory-name
        Specifies the path name of the directory being used.

Debug print out (DEBUG)

    Specify if you want QPRINT spooled file to examine how the command
    works.  The purpose of this parameter is problem determination and
    you should use DEBUG(*NO) in normal operation to avoid performance
    degradation.

    The possible values are:

    *NO
        No spooled file is generated.

    *YES
        Record activity to spooled file, QPRINT.
 ```
