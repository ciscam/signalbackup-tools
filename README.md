# signalbackup-tools
Tool to work with backup files generated by the Signal android application (https://signal.org/). The tool is provided as-is, there may be bugs. The tool and I are not affiliated with or endorsed by the Signal Foundation.

- [Important Note](#important-note)
- [Requirements](#requirements)
- [Obtaining](#Obtaining)
  - [Compiling](#compiling)
  - [MacOS](#macos)
  - [Windows binary](#windows)
- [Running](#running)
  - [Fixing broken backups](#fix)
  - [Dump unencrypted database to disk](#dump)
  - [Dump media to disk](#dumpmedia)
  - [Export CSV & XML](#export)
  - [Cropping to certain conversations or dates](#crop)
  - [Merging backups](#merge)
  - [Deleting/Replacing attachments](#deleting_attachments)
  - [Advanced options](#advanced)
- [Future plans](#future-plans)
- [Donate](#donate)

### Important Note

Signal is an actively developed application and consequently, the database format changes regularly. Often the changes do not affect the backup file format or the working of this program, but every once in a while a change does break (some of) the functionality of this program. It has happened before and it will happen again. Sometimes I fix it within hours, but when I am short on time, it may take a little longer. As of writing (2 Jan 2021), the new GroupV2-features may interfere with some of the more advanced features of this program and - most importantly - the `--exportxml` option. Any breakage will be dealt with as soon as I have some spare time.

### Requirements

Current stable released versions of
- a c++ compiler (tested with [GCC](https://gcc.gnu.org) 10.1.0 and [Clang](https://clang.llvm.org) 10.0.1, also tested and working with GCC 9.3.0 and 8.4.0)
- [OpenSSL](https://www.openssl.org/) (tested with 1.1.1g) _or_ [Crypto++](https://www.cryptopp.com/) (tested with 8.2.0, known to not compile with 5.6.4, which is still in Ubuntu for some reason). The default is openssl, see below to compile with crypto++ instead.
- [SQLite3](https://www.sqlite.org/) (tested with 3.32.3)

### Obtaining

**<span id="compiling">Compiling</span>**

A script is provided that will compile the binary on Arch and Fedora (and possibly many other distro's). In general just running `g++ -std=c++2a */*.cc *.cc -lcrypto -lsqlite3` should do the trick. When compiling with an old compiler version (gcc 8.x or clang <= 7), also add the `-lstdc++fs` flag and replace `-std=c++2a` with `-std=c++17`. If you have difficulty compling under your OS, I recommend downloading a Fedora Live image, booting of of that and running the following commands in a terminal (in the project directory):

```
 $ sudo dnf install gcc-g++ openssl-devel sqlite-devel which
 $ sh BUILDSCRIPT.sh
```

Alternatively, a Dockerfile has been kindly provided by David J. Meier, and is available at his gitlab page: <https://gitlab.com/splatops/cntn-signalbackup-tools>.

If for any reason you need to compile against cryptopp instead of openssl, run `sh BUILDSCRIPT --config cryptopp`, or manually `g++ -std=c++2a -DUSE_CRYPTOPP */*.cc *.cc -lcryptopp -lsqlite3`.

**<span id="macos">MacOs</span>**

It should be possible to compile and run the program under MacOS as well, for more info see [here](https://github.com/bepaald/signalbackup-tools/issues/9).

**<span id="windows">Windows binary</span>**

For the most recent Windows executable, check [the releases page](https://github.com/bepaald/signalbackup-tools/releases). This executable is a static build, cross compiled from my Arch linux install. It is only minimally tested.

### Running

**<span id="fix">Fixing broken backups</span>**

At the moment it has been used succesfully to fix backups that were corrupted for some reason (see https://github.com/signalapp/Signal-Android/issues/8355, and https://community.signalusers.org/t/tool-to-re-encrypt-signal-backup-optionally-changing-password-or-dropping-bad-frames/6497). If you want to fix a broken backup, run the tool as follows:

```
signalbackups-tools [input] [password] --output [outputfile] (--opassword [newpassword])
```

_NOTE_: if the corruption happens outside of attachment data, which is usually unlikely, chances of recovery are much lower.

If the output password is omitted, the input password is used to encrypt the new backup file. If the 'input' is a directory, it is assumed to contain an unencrypted dump of the backup (as made by this tool) and the input password can be omitted. In this case the output password is required, unless 'output' is also a directory.

If the 'output' is omitted only the scan is done, and the broken message is identified, giving you the option to delete it from the phone. The corrupted attachment data is dumped to file.

<details>
  <summary>Example (click to show)</summary>
  <p>
    
```
[~/programming/signalbackup-tools] $ ./signalbackup-tools CORRUPTEDSIGNALBACKUPS/signal-2019-05-20-05-29-06.backup3 949543593573534240555368549437 --output NEWBACKUPFILE --opassword 949543593573534240555368549437
signalbackup-tools source version 20190926.164320
IV: (hex:) 12 16 72 95 7a 00 68 44 7e cf 7d 20 26 f9 d3 7d (size: 16)
SALT: (hex:) cc 03 85 02 61 97 eb 5b ed 3e 05 00 c4 a8 77 40 28 08 aa 9f e5 a8 00 74 b4 f8 56 aa 24 57 a9 5d (size: 32)
BACKUPKEY: (hex:) 8f ff df 2b 9f 96 73 9a 63 95 0f ea 3f b1 e5 a4 87 12 19 ca 93 31 86 2a 60 3f 41 ef 6d a4 08 44 (size: 32)
CIPHERKEY: (hex:) ce 53 c1 f2 92 4b e3 b8 e1 56 85 61 14 96 82 8b 83 7f 07 21 83 52 1a c2 3f 6b 16 83 3e 33 94 a3 (size: 32)
MACKEY: (hex:) c2 77 af 1e 4b 05 db 62 52 57 af 8a d6 a4 d4 e9 6c 93 53 81 9a e7 6f 12 2c ce 13 8f b3 5e 8d 3a (size: 32)
COUNTER: 303461013
Reading backup file...
FRAME 37636 (071.5%)... 
WARNING: Bad MAC in attachmentdata: theirMac: (hex:) 30 75 bb b3 fb 65 a5 2a 5f b5
                                      ourMac: (hex:) ff f2 37 c1 f0 d4 2c 67 a3 cf 6c 41 55 bd 9c 1d 85 84 0e 66 96 ae 52 4e 90 b5 a3 37 33 3c b4 fc

WARNING: Bad MAC in frame, trying to print frame info:
Frame number: 37637
        Type: ATTACHMENT
         - row id          : 1317 (8 bytes)
         - attachment id   : 1536842122829 (8 bytes)
         - length          : 1516761 (8 bytes)
         - attachment      : (hex:) 47 49 46 38 39 61 e0 01 09 01 f7 00 30 00 ff 00 01 00 02 01 00 05 01 00 05 ... (1516761 bytes total)
Frame is attachment, it belongs to entry in the 'part' table of the database:
 - _id : 1317
 - mid : 1552
 - seq : 0
 - ct : image/gif
 - name : (NULL)
 - chset : (NULL)
 - cd : (NULL)
 - fn : (NULL)
 - cid : (NULL)
 - cl : (NULL)
 - ctt_s : (NULL)
 - ctt_t : (NULL)
 - encrypted : (NULL)
 - pending_push : 0
 - _data : /data/user/0/org.thoughtcrime.securesms/app_parts/part2625620938717109701.mms
 - data_size : 1516761
 - file_name : (NULL)
 - thumbnail : (NULL)
 - aspect_ratio : 2
 - unique_id : 1536842122829
 - digest : (NULL)
 - fast_preflight_id : 5897879359555196456
 - voice_note : 0
 - data_random : (hex:) f7 1e 34 f3 ba 07 34 44 56 04 15 dc 80 88 b7 10 9e c1 18 80 65 c7 7f 60 d9 cc 0f c9 d4 95 ce b4
 - thumbnail_random : (hex:) 14 f7 79 84 e5 a5 68 fe 98 a4 cb db 36 1f 6f c8 ca 3c 57 45 60 e2 d2 f2 f6 ee 42 71 42 7b 8e d7
 - width : 480
 - height : 265
 - quote : 0
 - caption : (NULL)

Which belongs to entry in 'mms' table:
 - _id : 1552
 - thread_id : 1
 - date : 2018-09-13 14:35:22 +0200 (1536842122790)
 - date_received : 2018-09-13 14:35:22 +0200 (1536842122809)
 - msg_box : 10485783
 - read : 1
 - m_id : (NULL)
 - sub : (NULL)
 - sub_cs : (NULL)
 - body : 
 - part_count : 1
 - ct_t : (NULL)
 - ct_l : (NULL)
 - address : +316XXXXXXXX
 - address_device_id : (NULL)
 - exp : (NULL)
 - m_cls : (NULL)
 - m_type : 128
 - v : (NULL)
 - m_size : (NULL)
 - pri : (NULL)
 - rr : (NULL)
 - rpt_a : (NULL)
 - resp_st : (NULL)
 - st : (NULL)
 - tr_id : (NULL)
 - retr_st : (NULL)
 - retr_txt : (NULL)
 - retr_txt_cs : (NULL)
 - read_status : (NULL)
 - ct_cls : (NULL)
 - resp_txt : (NULL)
 - d_tm : (NULL)
 - delivery_receipt_count : 1
 - mismatched_identities : (NULL)
 - network_failures : (NULL)
 - d_rpt : (NULL)
 - subscription_id : -1
 - expires_in : 0
 - expire_started : 0
 - notified : 0
 - read_receipt_count : 0
 - quote_id : 0
 - quote_author : (NULL)
 - quote_body : (NULL)
 - quote_attachment : -1
 - shared_contacts : (NULL)
 - quote_missing : 0
 - unidentified : 0
 - previews : (NULL)
Trying to dump decoded attachment to file 'attachment_1552.bin'
FRAME 37637 (071.6%)... Failed to read next frame (4294967295 bytes at filepos 1611402482)
Starting bruteforcing offset to next valid frame...
Checking offset 802590 bytes
GOT GOOD MAC AT OFFSET 802591 BYTES!
Now let's try and find out how many frames we skipped to get here....
Checking if we skipped 0 frames... nope! :(
Checking if we skipped 1 frames... nope! :(
Checking if we skipped 2 frames... nope! :(
Checking if we skipped 3 frames... YEAH!
Frame number: 37641
        Type: SQLSTATEMENT
         - (statement: "INSERT INTO part VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)" (83 bytes)
         - (uint64 parameter): "1319"
         - (uint64 parameter): "1554"
         - (uint64 parameter): "0"
         - (string parameter): "image/jpeg"
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (uint64 parameter): "0"
         - (string parameter): "/data/user/0/org.thoughtcrime.securesms/app_parts/part7691613523019485618.mms"
         - (uint64 parameter): "133247"
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (uint64 parameter): "1537091993419"
         - (bool parameter)  : "true" (value: "1")
         - (bool parameter)  : "true" (value: "1")
         - (uint64 parameter): "0"
         - (binary parameter): "(hex:) d3 a6 ea 3c 27 90 0f 12 74 71 54 ac 94 92 0f 08 30 04 e0 e1 b3 41 36 37 6d 8a 5d 44 fb 23 6e b5"
         - (bool parameter)  : "true" (value: "1")
         - (uint64 parameter): "720"
         - (uint64 parameter): "1280"
         - (uint64 parameter): "0"
         - (bool parameter)  : "true" (value: "1")

Got frame, breaking
FRAME 39960 (100.0%)... Read entire backup file...
done!
Removing 1 bad frames from database...
Exporting backup to 'NEWBACKUPFILE'
Writing HeaderFrame...
Writing DatabaseVersionFrame...
Writing SqlStatementFrame(s)...
  Dealing with table 'sms'... 32752/32752 entries...done
  Dealing with table 'mms'... 2212/2212 entries...done
  Dealing with table 'part'... 1814/1814 entries...done
  Dealing with table 'thread'... 27/27 entries...done
  Dealing with table 'identities'... 19/19 entries...done
  Dealing with table 'drafts'... 0/0 entries...
  Dealing with table 'push'... 0/0 entries...
  Dealing with table 'groups'... 10/10 entries...done
  Dealing with table 'recipient_preferences'... 63/63 entries...done
  Dealing with table 'group_receipts'... 1195/1195 entries...done
  Dealing with table 'job_spec'... 1/1 entries...done
  Dealing with table 'constraint_spec'... 0/0 entries...
  Dealing with table 'dependency_spec'... 0/0 entries...
Writing SharedPrefFrame(s)...
Writing EndFrame...
Done!
[~/programming/signalbackup-tools] $ 
```

  </p>
</details>

**<span id="dump">Dump unencrypted database to disk</span>**

The program can also dump the unencrypted backup components to a directory, or read the contents of a directory and pack and encrypt it back into a valid backup file. When dumping, make sure the directory to dump to is empty to start with. In theory, the decrypted files could be edited before reencrypting. The tool can be called the same as above, except the output should be a directory:


```
signalbackups-tools [input] [password] --output [outputdirectory]
```

To skip exporting media (like message attachments, avatars and stickers), add the option `--onlydb`.

<details>
  <summary>Example (click to show)</summary>
  <p>
    
```
[~/programming/signalbackup-tools] $ mkdir RAWBACKUP
[~/programming/signalbackup-tools] $ ll RAWBACKUP/
total 0
[~/programming/signalbackup-tools] $ ./signalbackup-tools --output RAWBACKUP/ ~/PHONE/signal-2019-07-14-06-59-26.backup 949543591444534240555456749437
IV: (hex:) 13 3f 94 13 be 5a 6d 1c 97 d0 20 88 4e f8 64 46 (size: 16)
SALT: (hex:) 5e 89 ec d8 f3 99 68 5b 9b a6 8b d8 3b b7 7d 8f e5 6a 2a 03 bb 2c c0 b9 f6 a1 0e bc bf ba 1a 25 (size: 32)
BACKUPKEY: (hex:) 38 4c a3 1c 17 9c f7 9b 27 30 98 bc 13 bf b6 5d 1d 90 df 13 c1 11 79 a4 ef d0 65 75 b9 55 cc 61 (size: 32)
CIPHERKEY: (hex:) 25 15 18 5f ac 06 3f 13 b5 0d c6 eb 8b e0 84 34 13 3f 84 f7 77 9b f6 ec 44 00 cb c0 77 2d 70 1f (size: 32)
MACKEY: (hex:) f3 00 34 77 1f a3 74 74 56 42 5e ad 6b d7 71 bf 40 7f e0 4f df 3a d1 1a 22 79 91 3a 97 73 88 28 (size: 32)
COUNTER: 322933779
Reading backup file...
FRAME 42337 (100.0%)... Read entire backup file...
done!
Writing HeaderFrame...
Writing DatabaseVersionFrame...
Writing Attachments...
Writing Avatars...
Writing SharedPrefFrame(s)...
Writing StickerFrames...
Writing EndFrame...
[~/programming/signalbackup-tools] $ ll RAWBACKUP/
total 2204384
-rw-r--r-- 1 bepaald bepaald    118871 jul 19 15:40 Attachment_1000_1518474349909.bin
-rw-r--r-- 1 bepaald bepaald        16 jul 19 15:40 Attachment_1000_1518474349909.sbf
-rw-r--r-- 1 bepaald bepaald     30017 jul 19 15:40 Attachment_1001_1518475497752.bin
     [...]
-rw-r--r-- 1 bepaald bepaald   9363456 jul 19 15:40 database.sqlite
-rw-r--r-- 1 bepaald bepaald         4 jul 19 15:40 DatabaseVersion.sbf
-rw-r--r-- 1 bepaald bepaald         2 jul 19 15:40 End.sbf
-rw-r--r-- 1 bepaald bepaald        54 jul 19 15:40 Header.sbf
-rw-r--r-- 1 bepaald bepaald        96 jul 19 15:40 SharedPreference_0.sbf
-rw-r--r-- 1 bepaald bepaald        97 jul 19 15:40 SharedPreference_1.sbf
[~/programming/signalbackup-tools] $ ./signalbackup-tools  RAWBACKUP/ --output NEWBACKUPFILE --opassword 949023591444534240555368549425
Exporting backup to 'NEWBACKUPFILE'
Writing HeaderFrame...
Writing DatabaseVersionFrame...
Writing SqlStatementFrame(s)...
  Dealing with table 'sms'... 34595/34595 entries...done
  Dealing with table 'mms'... 2370/2370 entries...done
  Dealing with table 'part'... 1934/1934 entries...done
  Dealing with table 'thread'... 29/29 entries...done
  Dealing with table 'identities'... 21/21 entries...done
  Dealing with table 'drafts'... 0/0 entries...
  Dealing with table 'push'... 0/0 entries...
  Dealing with table 'groups'... 10/10 entries...done
  Dealing with table 'recipient_preferences'... 67/67 entries...done
  Dealing with table 'group_receipts'... 1320/1320 entries...done
  Dealing with table 'job_spec'... 1/1 entries...done
  Dealing with table 'constraint_spec'... 0/0 entries...
  Dealing with table 'dependency_spec'... 0/0 entries...
  Dealing with table 'sticker'... 0/0 entries...
Writing SharedPrefFrame(s)...
Writing EndFrame...
Done!
[~/programming/signalbackup-tools] $ cmp ~/PHONE/signal-2019-07-14-06-59-26.backup NEWBACKUPFILE && echo "Files are identical"
Files are identical
[~/programming/signalbackup-tools] $
```

_NOTE_ The original and new files are not actually guaranteed to be identical, it just so happens that in this case the AvatarFrames are read from the filesystem in the order they appeared in the original.

  </p>
</details>

**<span id="dumpmedia">Dump media to disk</span>**

To only eport all media (like message attachments and avatars) from a backup, run nas follows:

```
signalbackups-tools [input] [password] --dumpmedia [outputdirectory]
```

Where `outputdirectory` is an existing directory.


**<span id="export">Export CSV & XML</span>**

###### Export CSV

To export the tables to a file of comma separated values (CSV), use `--exportcsv [table1]=[filename1],[table2]=[filename2],...`. To get all messages from the database, only the 'sms' and 'mms' tables need to be exported.

###### Export XML

To export to XML file, use `--exportxml [filename]`. _NOTE: Currently this will only export the messages from the sms table, NOT the mms table. This should be exactly the same data the official Signal app used to output when exporting a plaintext backup. All messages with an attachment, as well as all outgoing group messages are in the mms database, and thus are skipped. Exporting the mms table is being worked on. This option should be considered experimental._


**<span id="crop">Cropping to certain conversations or dates</span>**

_NOTE: This feature is experimental (even more so than the others). I test it fairly well myself, but I have no knowledge of it being used by other people. If you use it, please let me know if it works for you._

###### Crop to threads

To crop a backup file to certain threads, run:

```
signalbackup-tools [input] [password] --croptothreads [list-of-threads] --output [output] (--password [newpassword])
```

Where the list of threads are the ids as reported by `signalbackup-tools [input] [password] --listthreads`. The list supports commas for single ids and hyphens for ranges, for example: `--croptothreads 1,2,5-8,10`.

###### Crop to dates

To crop a backup file to certain dates, run:

```
signalbackup-tools [input] [password] --croptodates begindate1,enddate2(,begindate2,enddate2(,...)) --output [output] (--opassword [newpassword])
```

The 'begindate' and 'enddate' must always appear in pairs and can be either in "YYYY-MM-DD hh:mm:ss" format or as a single number of [milliseconds since epoch](https://en.wikipedia.org/wiki/Unix_time). For example, the following commands are equivalent (in my time zone) and both crop the database to the messages between Sept. 18 2019 and Sept 18 2020: `--croptodates "2019-09-18 00:00:00","2020-09-18 00:00:00"` or `--croptodates 1568761200000,1600383600000`.


**<span id="merge">Merging backups</span>**

_NOTE: This feature is experimental (even more so than the others). Previous versions were tested successfully in the wild, but the code has needed an update since the internal database format of Signal changed. The new version, though largely identical to the old, has at this moment in time not been tested as far as I know, apart from some very basic testing by myself._

To merge two backups, the backups must be at compatible database versions. The database version can be found by running `signalbackup-tools [input] [password] --listthreads`. Either both backups need to have database version <= 27, both >= 33, or both in between 27 and 33. If needed, import the backups into Signal and export them again to get them updated and at equal versions. To import all threads from one database into another, run:

```
signalbackup-tools [first_database] [password] --importthreads ALL --source [second_database] --sourcepassword [password] --output [output_file] (--opassword [output password])
```

Always use the backup file with the highest database version as 'first_database' and the older version as source. If not all threads should be imported from the source, a list of thread ids can be supplied (eg `--importthreads 1,2,3,8-16,20`). The thread ids can be determined from the output of `--listthreads`.

If you use this option and read this line, I would really appreciate it if you let me know the results. Either send me a mail (basjetimmer at yahoo-dot-com) or feel free to just open an issue on the tracker for feedback.

**<span id="deleting_attachments">Deleting/Replacing attachments</span>**

_NOTE: This feature is experimental, make sure to always keep a copy of your original backup file. Feedback is appreciated_

###### Deleting attachments

To remove attachments from the database, while keeping the message bodies (for example to shrink the size of the backup) the option `--deleteattachments` can be used. To further specify precisely which attachments are to be deleted, the following options can be added:
- `--onlyinthreads [list-of-threads]`. The list supports commas for single ids and hyphens for ranges, for example: `--onlyinthreads 1,2,5-8,10`. To obtain the number-id of threads use `--listthreads`.
- `--onlyolderthan [date]`/`--onlynewerthan [date]`. Where 'date' supports the same format as the `--croptodates` option ([here](#crop)).
- `--onlylargerthan [size]`. The size is specified in bytes.
- `--onlytype [mime type]`. This argument can be repeated. Only selects attachments which match 'mime type*'. For example `--onlytype image/j` will match both 'image/jpg' and 'image/jpeg'. To delete all image type attachments, simply use `--onlytype image`.

When adding this specifying options, only attachments which match _all_ given options are deleted.

###### Replacing attachments

Replacing attachment is not yet fully implemented, but will allow you to replace the attachments with a placeholder image instead of deleting them.

**<span id="advanced">Advanced options</span>**

The program can run any sql queries on the database in the backup file and save the output. If you know the schema of the database and know what you're doing, feel free to run any query and save the output. Examples:

```
# delete all sms and mms messages from one thread:
signalbackup-tools [input] [password] --runsqlquery "DELETE * FROM sms WHERE thread_id = 1" --runsqlquery "DELETE * FROM mms WHERE thread_id = 1" --output [output] (--opassword [newpassword])
```

```
# list all messages in the sms database where the message body was 'Yes'
$ ./signalbackup-tools [input] [password] --runprettysqlquery "SELECT _id,body,DATETIME(ROUND(date / 1000), 'unixepoch') AS isodate,date FROM sms WHERE body == 'yes' COLLATE NOCASE"
signalbackup-tools source version 20191219.175337
IV: (hex:) 12 16 72 95 7a 00 68 44 7e cf 7d 20 26 f9 d3 7d (size: 16)
SALT: (hex:) cc 03 85 02 61 97 eb 5b ed 3e 05 00 c4 a8 77 40 28 08 aa 9f e5 a8 00 74 b4 f8 56 aa 24 57 a9 5d (size: 32)
BACKUPKEY: (hex:) 8f ff df 2b 9f 96 73 9a 63 95 0f ea 3f b1 e5 a4 87 12 19 ca 93 31 86 2a 60 3f 41 ef 6d a4 08 44 (size: 32)
CIPHERKEY: (hex:) ce 53 c1 f2 92 4b e3 b8 e1 56 85 61 14 96 82 8b 83 7f 07 21 83 52 1a c2 3f 6b 16 83 3e 33 94 a3 (size: 32)
MACKEY: (hex:) c2 77 af 1e 4b 05 db 62 52 57 af 8a d6 a4 d4 e9 6c 93 53 81 9a e7 6f 12 2c ce 13 8f b3 5e 8d 3a (size: 32)
COUNTER: 2907636
Reading backup file...
FRAME 4852 (100.0%)... Read entire backup file...

done!
 * Executing query: SELECT _id,body,DATETIME(ROUND(date / 1000), 'unixepoch') AS isodate,date FROM sms WHERE body == 'yes' COLLATE NOCASE
------------------------------------------------------
| _id   | body | isodate             | date          |
------------------------------------------------------
| 3235  | Yes  | 2017-10-21 17:10:15 | 1508605815286 |
| 9345  | Yes  | 2017-12-18 22:18:36 | 1513635516440 |
| 17125 | Yes  | 2018-02-02 15:46:16 | 1517586376228 |
| 21300 | Yes  | 2018-05-10 21:14:49 | 1525986889325 |
| 26317 | Yes  | 2018-10-25 15:16:58 | 1540480618238 |
| 32433 | Yes  | 2019-05-10 14:22:25 | 1557498145794 |
------------------------------------------------------

# now change a specific message:
[~/programming/signalbackup-tools] $ ./signalbackup-tools [input] [password] --runsqlquery "UPDATE sms SET body = 'No' WHERE _id == 21300" --ouput [output]
```

If you also need to edit the attachments, dump the backup to directory first ([as described above](#dump)) and do whatever you want, but realize when editing the .bin file, it will usually require changes to also be made to the .sbf file and the sql database to end up with a valid database.


## Future plans

- ~~merging existing backups (succesfull tests have been done)~~ _DONE_
- exporting to other formats (~~csv, xml,~~ html) _WIP_
- ~~cropping backup to certain conversations and time spans (succesfully done in testing)~~ _DONE_
- replacing/deleting attachments without changing/deleting messages. For example, replacing with thumbnails or tiny placeholders to save space. _WIP_
- importing databases from the desktop app. I have no experience with that yet. _WIP_

Development will be slow at times.

## Donate

If this tool was helpful to you or you appreciate my work and you can spare it, you might consider donating:

BTC: 17RqHi9XBeUAEShbp2RnbmkCSAU2R94tH4

ETH: 0xc59892f4E54E3e8Ab285f3C41B396d9E15C8B4aA

ZCash: zs1mcr3nsjpewejld9acuqdvhslesck0glmz696wkw2l0mkg926xzegfn2kqhn8fmcz7ld3z4g0w3j

Paypal: [![Paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=U523FZFW3BQBQ&currency_code=USD&source=url)

Donations will help development in that they will put food in my mouth, and I need food to write code :smile:

You might also consider helping out the Signal Foundation here: https://support.signal.org/hc/en-us/articles/360007319831-How-can-I-contribute-to-Signal-
