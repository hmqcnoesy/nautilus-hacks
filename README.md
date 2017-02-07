# nautilus-hacks

## Getting lims_user password (#1)

Add a server with `TRACE` preceding the actual Database Type.  

![Server window with TRACE database type](img/01.jpg)

Make sure the "TRACE" part "sticks".  It might not if you create a new server entry, and you'll have to edit the properties. 
You'll know it is good to go when you see "TRACE" in the server list.

![Server list showing TRACE prefix](img/02.jpg)

Select the trace server and login.

![Login window with trace server selected](img/03.jpg)

The Database Trace window appears.  You can clear all the checkboxes and provide a log location for saving.

![Database trace window](img/04.jpg)

Once you click OK in the Database Trace window, you will be logged in and the Nautilus client will be logging.  
Log out immediately and close Nautilus.

Navigate to the location you saved your trace log.

![Windows explorer at log location](img/06.jpg)

Open the log and find the keys to the kingdom.  Good grief.

![Log file showing lims_user password](img/07.jpg)
