# Installation

1. Download csv.exe and execute it on Windows95/98/NT to extract csv.savf.
1. Create save-file on your AS/400.
1. Connect to the AS/400 using any FTP client software.
1. Put the distribution (csv.savf) to the save-file in binary mode.
1. Restore objects from the save-file using RSTOBJ command.

## Sample instruction

Create save-file on the target AS/400:

```
> CRTSAVF FILE(QGPL/CSV)
  File CSV created in library QGPL.
```

Send save-file from Windows95/98/NT/2000 command prompt:

```
E:\>ftp your_as400_hostname
Connected to ????.
220-QTCP AT ????.
220 CONNECTION WILL CLOSE IF IDLE MORE THAN 5 MINUTES.
User (????:(none)): as400_user_id
331 ENTER PASSWORD.
Password:enter_password_for_the_user_id
230 ???? LOGGED ON.
ftp> bi
200 REPRESENTATION TYPE IS BINARY IMAGE.
ftp> put csv.savf qgpl/csv (replace
200 PORT SUBCOMMAND REQUEST SUCCESSFUL.
150 SENDING FILE TO MEMBER CSV IN FILE CSV IN LIBRARY QGPL.
250 FILE TRANSFER COMPLETED SUCCESSFULLY.
287232 bytes sent in 1.10 seconds (260.88 Kbytes/sec)
ftp> quit
221 QUIT SUBCOMMAND RECEIVED.
```

Go back to the 5250 session to restore objects:

```
> DSPSAVF FILE(QGPL/CSV)
> CRTLIB LIB(CSV)
  Library CSV created.
> RSTOBJ OBJ(SOURCE LICENSE) SAVLIB(CSV) DEV(*SAVF) SAVF(QGPL/CSV)
  2 objects restored from CSV to CSV.
> STRREXPRC SRCMBR(MAKE) SRCFILE(CSV/SOURCE)

  Start CRTCSVF compilation.
    You must have authority to exec CRTxxx.

            
       * * * * * * * * * * * * * * * * * * * * * *
            
  Start creating CRTCSVF command...
    creating CL program CRTCSVFCLP...
    result-> 0
    creating main program module CRTCSVFRPG...
    result-> 0
    creating main program CRTCSVFRPG...
    result-> 0
    creating pnlgrp...
    result-> 0
    creating command...
    result-> 0

            
  Compile finished. Confirm error(s) if exists.

            
  Press ENTER to end terminal session.
```

<p></p>
<br>

---

# Usage

```
                    Create CSV format Stream File - Help

      The Create CSV format Stream File (CRTCSVF) command copies an
      externally described database file to a CSV format stream file on
      IFS.  The term CSV stands for Comma Separated Value(s) and used for
      purposes of exchanging data between heterogenous databases.

       O  Read data from database file, convert format to CSV and write to
          stream file directly.  The CSV format file (TOSTMF parameter)
          will be called the STMF for this command.

       O  For string field, trailing blanks are removed. If last byte of
          string field is shift-in (X'0e', used to represent end of DBCS
          character), DBCS trailing blanks are also removed. This is very
          effective when using J/E fields.

       O  For numeric (PACK or ZONE) field, preceeding and trailing '0's
          are supressed. (e.g. '0001.00' -> '1.0') If the field has no
          decimal fraction, period ('.') will not be displayed.

       O  Decimal data errors are ignored and numeric value is set to '0'
          or '0.0' when you specify RCDERR(*IGNORE) or RCDERR(value).

              Note:  Specifying RCDERR(*IGNORE) in conjunction with
              RCDERRMSG(*SECLVL) may generate a lot of low level messages
              in joblog. Specifying RCDERR(*ABORT) or RCDERR(value) is
              recommended when converting large database file.

      Restrictions:

        1.  If the database file contains more than 100 fields, only first
            100 fields are converted.  If the database file contains
            fields with complex definition, less than 100 fields may be
            converted.

        2.  Cannot process database file if record length of the database
            file is greater than 9999.

        3.  Unexpected condition may occur when CSV record (one line)
            exceeds 32767 byte.

                Note:  This restriction applies only to encoding schemes
                (CCSID) which require escape sequences. Usually you won't
                encounter this restriction because maximum record / string
                field length is 32766.

        4.  Only ZONE, PACK, CHAR (except GRAPHIC and VARYING length),
            DATE, TIME, TIMESTAMP fields are converted.  Other field types
            such as BINARY are ignored.

          Note:  Do not precede an entry with an asterisk unless that
          entry is a "special value" that is shown (on the display or in
          the help information) with an asterisk.

  Error messages for CRTCSVF

      *ESCAPE Messages
      CPF9897     Program aborted. &1/&2 records processed, &3 error(s)
                  found.

      *STATUS Messages During the running of the CRTCSVF command, message
      is sent as a status message in every 1,000 records informing the

      interactive user that a copy is occurring.

  File (FILE)

      Specifies the qualified name of the FILE that contains the records
      to be copied. The database file can be a single-format logical,
      physical file.

      This is a required parameter.

      The File Name can be qualified by one of the following library
      values:

      *LIBL
          All libraries in the job's library list are searched until the
          first match is found.

      *CURLIB
          The current library for the job is searched.  If no library is
          specified as the current library for the job, the QGPL library
          is used.

      library-name
          Specify the name of the library to be searched.

      file-name
          Specify the name of the file that contains the records to be
          copied.

  To stream file (TOSTMF)

      Specifies the path name of the output stream file to which data is
      copied. All directories in the path name must exist. New directories
      are not created. If the stream file does not exist, it will be
      created.  For more information on specifying path names, refer to
      Chapter 2 of the CL Reference, SC41-5722.

      This is a required parameter.

  Member name (MBR)

      The File Member Name specifies the FILE member name that is copied.

      The possible values are:

      *FIRST
          The first member (in order of creation date) in the FILE is
          copied.

      member-name
          Specify the name of the FILE member to be copied.

  Overwrite existing stream file (OVRWRT)

      Specifies if a new STMF is created when a STMF of the same name
      already exists in the specified IFS path.

      The possible values are:

      *YES
          A new STMF is created in the specified path. If a STMF of the
          same name already exists in the specified IFS path, CRTCSVF
          deletes it before conversion. Even error occured during
          conversion, original (deleted) stream file will not be
          recovered.

      *NO
          A new STMF is not created if a STMF of the same name already
          exists in the path.  The existing STMF is not replaced, a
          message is displayed, and command aborts.

  Database file CCSID (DBFCCSID)

      Specifies the method of obtaining the database file CCSID (EBCDIC).

      The possible values are:

      *JOB
          The default job  CCSID is used, unless it is 65535.  If the
          default job CCSID is 65535, an error condition is created.

      *FILE
          The database file CCSID is used, unless it is 65535.  If the
          database file CCSID is 65535, default job CCSID is used. If the
          default JOB CCSID is 65535, an error condition is created.

      coded-character-set-identifier
          Specify the database file CCSID.  The values 0, 65534, and 65535
          are not valid.

  Stream file code page (STMFCODPAG)

      Specifies the method of obtaining the stream file code page and the
      CCSID equivalent of the code page that is used for data conversion.

      The possible values are:

      *SYSTEM
          Determine the correct NLV directory (PC ASCII CCSID of selected
          NLV) to be used to locate translation information.

      code-page
          Specify the ASCII code page (CCSID) used. The specified code
          page is associated with the stream file when it is created.

  Additional information (ADDINF)

      Specifies additional information to be written to the STMF.

      The possible values are:

      *NONE
          No additional information will be written.

      *FLDNAM
          Put Field names are inserted in the first row of the STMF.

      *COLHDG
          Column-headings are inserted in the first row of the STMF.

      *BOTH
          Column-headings are inserted in the first row and field names
          are inserted in the second row of the STMF.

  Allow record error (RCDERR)

      Specifies how much record errors can be ignored.

          Note:  Possible record error types are :

          - Character conversion (iconv() function) error.

          - Decimal data error.

          - Double quotation (") in string field replaced by different
          character.

      The possible values are:

      *ABORT
          Record error is not permitted. Program will abort if any record
          error is detected.

      *IGNORE
          Continue processing in case of recoverable error. Record errors
          will be recoverd.

      record-errors
          Specify the maximum number of record error. Program aborts when
          error count exceeds this value. Valid values range from 1
          through 9999.

  Record error message (RCDERRMSG)

      Specifies whether record error messages are sent to joblog or/and
      STMF.  Fatal internal program error or completion message is always
      sent to the program message queue.

      The possible values are:

      *BOTH
          Record error messages are written to the STMF and sent to joblog
          as low level message.

      *STMF
          Record error messages are written to the STMF.

      *SECLVL
          Record error messages are sent to joblog as low level message.

      *NONE
          No record error messages are sent.

  String delimiter replace char (RPLCHR)

      Replaces double quote character (") in string field by specified
      character.

          Note:  If you specify RPLCHR(*DOUBLEQUOTE) or RPLCHR('"'),
          double quotation in string field is not considered to be record
          error.

      The possible values are:

      *DOUBLEQUOTE
          The double quote character (") in string field will be replaced
          by two double quote characters (""). This is equivalent to
          specifying double quote character as character-value.

              Note:  This encoding is same as CA/400 express file transfer
              function.  Microsoft Excel 97 and Lotus 1-2-3 98 convert two
              double quote characters ("") to single double quote
              character (") when they read the CSV file. Other software
              may not interpret this.

      *SPACE
          The double quote character in string field will be replaced by
          space character (X'20').

      'character-value'
          Specify a character to replace double quote character (") in
          string field.

  Field delimiter (FLDDLM)

      Specifies the field delimiter for the record. This value is placed
      between fields.

      The possible values are:

      *COMMA
          The comma (,) character is used as the field delimiter.

      *SPACE
          The space (X'20') character is used as the field delimiter.

      'character-value'
          Specify a character to be used as the field delimiter.

  Debug print out (DEBUG)

      Specifies whether the debug output from the command is printed with
      the job's spooled output.

      The possible values are:

      *NO
          The debug output is disabled.

      *YES
          The debug output is printed with the job's spooled output.
```
