# Nagios_Zombie_Plugin

check chrome or any other zombie processes (which exists over 30 minutes) within machines process list including docker!
Recieves arguments $1 = process name to check $2 = Warning level $3= Critical level

Integration instructions:

# Client Side:

You should test the script and see it works locally on the machine first, can be found here:
https://geekpeek.net/nagios-plugin-bash/

After you finalized the script, go to this file within the client:
vi/etc/nagios/nrpe.cfg

And add the path to your command, usually the path convention for customized scripts is this:
/usr/local/nagios/libexec/check_chrome_proc

Add your command bellow the other command inside the bottom of the file, like this:
command[check_chrome_proc]=/usr/local/nagios/libexec/check_chrome_proc /opt/google/chrome 5 20

(This script gets arguments from outside, the warning and critical level, like Nagios convention says)

After updating the nrpe.cfg you should restart the service:
/etc/init.d/nagios-nrpe-server restart

# Server Side:
Now you should update the file:
/Server/etc/objects/commands.cfg

Add your script as command:

Change command_name of check_chrome_proc into your script name.
Change command_line into the correct path of the script.

For instance:
#check chrome zombie processes (exists over 30 minutes) within machines process list
#Receives arguments $1 = process name to check $2 = Warning level $3= Critical level
define command{
	command_name	check_chrome_proc
	command_line	$USER1$/check_chrome_proc /opt/google/chrome 10 30
	}
	
Then go to the machine's file within the server for example: mydtbld0178.cfg

And add the relevant command, for instance:
Service description is the string the nagios UI should display.
check_command check_nrpe!check_chrome_proc - Bolded is the part you should set your script instead.

#Check chrome zombie processes (exists over 30 minutes) within machines process list
define service{
	use generic-service-without-notify
	host_name mydtbld0178.hpeswlab.net
	service_description Open Chrome browsers
	check_command check_nrpe!check_chrome_proc
	max_check_attempts	5
	check_interval	10
	retry_interval	5
	}
	
Finally don't forget to pull from Git and reset the Nagios process.

/*In case you want to add graphic representation, here's how:*/

## Set graphic representation for your Nagios plugin

Only Server side:

If your script output is done correctly, the only thing you need to do is going to this file:
Server/etc/nagiosgraph/map

Add a new role, catching the correct number by regex in Pearl lang:
If you need more guidance here there are instructions at the top of this file usually.
Also if Nagios is not representing the graph, there's something wrong with your regex.
A good way to monitor and understand where the problem is, go by SSH to the machine and run docker logs over the container:
docker logs --tail=100 nagios_nga

An instance for a regex role:

/*Service type: Open Chrome browsers
output: Open Chrome browsers CRITICAL - 430 zombies: /opt/google/chrome processes*/
/output:Open Chrome browsers.*?(\d+).*?zombies:.*?processes/
and push @s, [ 'openChromeBrowsers',
               [ 'openChromeBrowsers', GAUGE, $1 ] ];




# If you have any updates or improvements feel free to send merge requests.
# For any information you can email me @ michael.ido@gmail.com - Enjoy!
