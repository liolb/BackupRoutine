# BackupRoutine 

to help automate backups.

This script can be used to create (regularly) `*.zip`-compressed backups of pre-defined file collections (wildcards are supported).

A configuration file allows to declare "Backup Profiles" and "Backup Destinations". A **Backup Profile** defines a set of files. These files will be backed up as a single compressed (`*.zip`) file by the BackupRoutine and stored at the locations given by the **Backup Destinations**. 

## Getting started

After downloading the code
### Setup the Configuration File 

First copy or rename `sample_config.json` to get `config.json`.
Then, the sections `backup_profile` and `backup_destination` need to be customised to your needs, see sections [[#Backup Profiles]] and [[#Backup Destinations]].

### Running the script

```bash
python3 ./backup.py
```

BackupRoutine runs all active profiles and destinations. The file collections of all active profiles will be separately packed, compressed and backed up to all active destinations. Backups are created as zip files in the form `{backup_profile.id}/archive{backup_datetime}.zip`
After each backup process destinations older backups are deleted as defined in [[#Clean-Up Mechanism]].
For Errors see: [[#Error alerts]].

## Command Line Arguments

BackupRoutine supports the following command line arguments, examples see below.

| Argument | Argument Effect                                                               |
| -------- | ----------------------------------------------------------------------------- |
| `-p`     | specify a Backup Profile                                                      |
| `-d`     | specify Backup Destination(s) (one or several)                                |
| `-t`     | perform a dry run, i.e. run the process without actually creating any backups |

The following example will run only `backup_1` and create backups on all active destinations:

```bash 
python3 ./backup.py -p backup_1
```

The following example will run all active profiles and create backups in `my_backup_vault_1`and `my_backup_vault_2`:

```bash
python3 ./backup.py -d my_backup_vault_1 my_backup_vault_2
```

Using the command line argument  `-t` you can run the backup process without creating backups:

```bash
python3 ./backup.py -t
```

Error logs will be displayed in the terminal if anything in the configuration file is wrong. 
The number of files that will be backed up is displayed.


## Backup Profiles
---
A Backup Profile defines the files of which a backup should be created. 
You can either put a list of all files into `sources`or select files using wildcards and `ignore` patterns.

```json
"backup_profiles": [
    {
        "id": "my_backup",                   // This is the unique name of the backup profile
        
        "active": true,                      // Each profile can be set active or inactive
        
        "source": [                          // A collection of files is defined by a list of search patterns such as
            "*/sample/backup_this_dir/",
            "*/backup_all/*/test_folders/",
            "*/backup_all_textfiles/*.txt"
        ],
        "ignore": [                          // Additionally you can exlude files or directories, as for example
            ".*",                            // ignore all files starting with a dot, ...
            "*/not-this-dir/",
            "temp*"   
        ]
    } 
    // ...you can add as many backup profiles as you wish   
```

## Backup Destinations

A Backup Destination defines where to store the backups and how long to keep them.

```json
"backup_destinations": [
    {
        "id": "my_backup_vault",          // This is the unique name of the backup destination
        
        "active": true,                   // Each destination can be set active or inactive
        
        "days_to_keep": 28,               // For each destination you can define how long the backup should be stored
                                          // If backups should never be deleted: set `days_to_keep = -1
        "directory": "./destination/"     // directory where the backup should be stored
    }
    // ...you can add as many backup destinations as you wish  
```

### Clean-Up Mechanism

Subsequently, after each backup run, a clean-up mechanism will be performed in all declared destinations. For each destination you can define how long the backup should be stored by setting the property `days_to_keep`. All backups older than this date will be deleted. If backups should never be deleted: set `days_to_keep = -1`. 


## Error alerts

The script has no real alert system implemented. In case of an error the user will be "notified" by the error log being opened in the standard editor and displayed to the user.