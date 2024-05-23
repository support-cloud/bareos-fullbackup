###############
Prerequisites:
###############

Step 1. Install ceph client and add client.admin.keyring from the path, /etc/ceph

Step 2. Check the ceph status and it perfectly communicates with the backup node.

Step 3. Download the Openstack openrc file to create a hidden file from the backup node.

Step 4. Install openstack client then check the status (For eg. openstack server list)

#######################################
Setting Up Bareos configuration Files:
#######################################

Step 1. Create a config file FileSet, Job, JobDef and Schedule

FileSet {
Name = "demo"
Include {
Options {
signature = MD5
compression = LZ4
noatime = yes
}
File = [Temporary mount vol path]
}
Ignore FileSet Changes = yes
}

Job {
Name = "demo"
JobDefs = "demo"
FileSet = "demo"
Schedule = "demo"
Storage = Ceph
Messages = Standard
Pool = Incremental
Priority = 10
}

JobDefs {
Name = "demo"
Type = Backup
Level = Incremental
Client = bareos-fd
FileSet = "demo"                     # fileset name
Schedule = "demo"
Storage = Ceph
Messages = Standard
Pool = Incremental
Priority = 10
Client Run Before Job = "/etc/bareos/scripts/backup.sh"      # backup script 
Client Run After Job = "/etc/bareos/scripts/remove.sh"        # backup remove script
Write Bootstrap = "/var/lib/bareos/%c.bsr"
}
  
Schedule {
Name = "demo"
Run = Level=Full 1st mon at 05:30                                        # scheduling time for run jobs
Run = Level=Differential 2nd-5th mon at 05:30
}

Step 2. In order for this script to delete old volumes from disk you need to set 3 directives in your Bareos Pool directive(s)

Recycle = yes                           # Bareos can automatically recycle Volumes
Auto Prune = yes                        # Prune expired volumes
Action on Purge = Truncate              # Delete old backups
Go to path, /etc/bareos/bareos-dir.d/pool

###############################
Storage Daemon Configuration:
###############################

Step 1. Add the storage device configuration file

Device {
Name = CephStorage
Media Type = File
Archive Device = [external storage device mount path or CephFS]
LabelMedia = yes;                   # lets Bareos label unlabeled media
Random Access = yes;
AutomaticMount = yes;               # when device opened, read it
RemovableMedia = no;
AlwaysOpen = no;
Description = "File device. A connecting Director must have the same Name and MediaType."
}


###############################
Customized backup scripts:
###############################

Step 1. Make directory for this location, /etc/bareos

mkdir /etc/bareos/scripts

Backup script : /etc/bareos/scripts/backup.sh 

Remove script: /etc/bareos/scripts/remove.sh

Step 2. Make it owned by root and the group bareos and make it executable

chown root.bareos /etc/bareos/scripts
chmod 770 /etc/bareos/scripts/backup.sh
chmod 770 /etc/bareos/scripts/remove.sh

Step 3.Finally, all configuration are completed then, restart the services for Bareos daemons

systemctl restart bareos-dir bareos-dir bareos-dir apache2
