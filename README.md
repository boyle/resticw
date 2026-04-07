<!-- RESTICW(1) Version 1.0 | RESTICW Usage -->

# NAME

**resticw** - configuration wrapper around restic


# SYNOPSIS

|  **resticw** \<**all**|name\> \<cmd\> \[options\]  
|  **resticw** \<**all**|name\> \<**backup**|**check**\> \[**--quiet**\]  
|  **resticw** \<name\> \<**ncdu**|**redu**>  
|  **resticw** \<name\> \<restic pass-through commands/arguments...\>  
|  **resticw** \<name\> **write** **--exclude-file=**\<list.txt\> **--forget**  
|  **resticw** \<name\> **mount** \</mnt/repo\>  

# DESCRIPTION

A configuration wrapper around **restic**(1).


## OPTIONS

name

: Repository name.

**all**

: Loop over all repositories.
  Call **resticw** without arguments to list all repositories.

**backup**

: Runs **restic backup** using the configuration files to setup configuration arguments.
  Files and directories listed in /**backup** will be included. Files and directories in /**exclude** will be excluded.
  If **backup** does not exist, this repository will be skipped for backup but can still be used for other restic operations
    (e.g. for remote backups stored at this location).
  If the /**hook** script exists, it will be executed before and after the backup with a single argument (**before**, **after**).
  If /**retention** exists, **restic forget** is used to implement the retention policy.
  Repository integrity will be confirmed using **restic check** after all updates are complete.

**check**

: Runs **restic check**.

**ncdu**

: Look for the largest files in the latest snapshot.

**redu**

: Look for the largest files in the whole repository.
  Combines all snapshots.
  Allows marking files/directories into a list which can be used to
  update the exclusions configuration and entirely remove files from the repository. See EXAMPLES.


# EXAMPLES

To permanently remove files from the repository
| **resticw** \<name\> **rewrite** **--exclude-file=**list.txt **--forget**

To mount the repository, for recovering files

| **resticw** \<name\> **mount** \</mnt\>

To check a repository including all the data

| **resticw** \<**all**|name\> **check --read-data**

## Best Practices

 - The classic 3-2-1 rule keeps three copies on two media with one off-site.
 - Check that backups (a) contain the right content, and (b) are still valid. A
   write-only backup isn't worth anyone's time.
 - Data backup to the cloud *must* be encrypted at source. Even then, don't put
   anything in those backups that you don't want your grandma to see.
 - Backup frequently enough that any lost work will not hurt too much. For
   example, backup of working files in the home directory might be hourly while
   more static files could be weekly or monthly.
 - Cloud backups may be bandwidth limited but we still need to apply our best practices.
 - Consider a Retention Policy as both what data needs to be protected (and for how long)
   as well as what data needs to be permanently deleted/removed after a period of time.

## Retention Policy
Retention Policy 24h of hourly backups, 7 daily backups, 4 weekly backups, 12 monthly backups, 75 yearly backups

The /**retention** file format is
```
hourly: 24h
daily: 7d
weekly: 1m
monthly: 1y
yearly: 75y
```

## Automation
Use cron jobs to automate most of this:

 - `/etc/cron.hourly/resticw-backup`:  
   `resticw all backup --quiet`
 - `/etc/cron.d/resticw-check`:  
   `20 04 11 11 * resticw all check --read-data --quiet # Nov 11 @ 04:20 (Rememberance Day)`

# FILES

For a particular backup \<name\>, the configuration files are read from directories:
/**etc**/**resticw**/\<name\> (global) and **$HOME**/**.config**/**resticw**/\<name\> (local).
Local configuration overrides global configuration.

Each \<name\> directory may contain files
  /**repo**, /**passwd**, /**backup**,
  /**exclude**, /**retention** and /**hook**.
The /**repo** and /**passwd** files are mandatory.
The remaining files are optional.
The /**hook** script is an executable.
Any other files in the directory are ignored.

Rules for the ownership and visibility of configuration files is enforced for security. (Or at least reduced finger fudges.)
All files must be chmod go-wx.
The /**passwd** file must be chmod go-rwx.
The owner of the /**passwd** file and $UID must agree.
All files in the directory must have the same owner.
Files in the configuration can be links, in which case the ownership rule applies to
the linked file (see **realpath**(1)).
Repositories without a /**backup** file are skipped during **resticw backup**.

# BUGS
Maybe.

# AUTHOR
Alistair Boyle \<alistair.js.boyle@gmail.com\>

# SEE ALSO
**restic**(1), **ncdu**(1), **redu**(1)
