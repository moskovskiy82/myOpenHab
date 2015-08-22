#Configuring the openHAB runtime

##Users and passwords

The openHAB usernames and passwords are stored in the file 
`/opt/openhab/configurations/users.cfg`.  The entries in this file must be of the format

	yourusername=yourpassword
	
ignore the test beyond the comma in the example entry, it is not yet implemented.


##Configure MQTT

<http://www.homeautomationforgeeks.com/project/openhab.shtml>

Edit the openHAB config file

	sudo nano /opt/openhab/configurations/openhab.cfg

Scroll to the Transport section and look for the MQTT transport section.  Look for the lines that contain `<broker>.url` and `<broker>.retain` and uncomment them.  Change them to the mosquitto broker that is running on the RPi.  Give the broker a name in the `<broker>` part - this name will be used later to refer to your broker.

	mqtt:mymosquitto.url=tcp://localhost:1883
	mqtt:mymosquitto.retain=true

You may also decide to set the last-will-and-testament .lwt to send a final message when closing down.

After editing restart the daemon to note the new config settings:

		sudo /etc/init.d/openhab restart

Save and exit the file.

##Notes on configuring openHAB

Read these pages:
<https://github.com/openhab/openhab/wiki/Configuring-the-openHAB-runtime>  

You use the openHAB Designer, but you can also edit the files with a normal file editor.  The files must be in UTF-8 encoding.

Item and sitemap files  can be changed during runtime, with no need to restart openHAB.

The global configuration is done in the `openhab.cfg` file.  Changes made to this file have impact throughout all sitemaps.  There is a default version of this file which can be used as a template for your own work: make a copy of the default and edit the copy.

The most common global configuration changes are to activate specific bindings by uncommenting the appropriate lines, e.g., as when the MQTT binding was activated above in this file.

##Item definitions

Item files are stored in `/opt/openhab/configurations/items`.
Although items can be dynamically added, it is most common to statically define most of the items.  These static definition files follow a prescribed syntax.
For more information on how to create item files see here:  
<https://github.com/openhab/openhab/wiki/Explanation-of-Items>  

For an example see here
<http://www.homeautomationforgeeks.com/project/openhab.shtml>

	sudo nano /opt/openhab/configurations/items/default.items
	
Add the following text:

	Group All

	Group gGroundFloor (All)

	Group GF_Living "Living Room" <video> (gGroundFloor)

	Number TestTemperature "Temperature [%.1f F]" <temperature> (GF_Living) {mqtt="<[mymosquitto:home/temperature:state:default]"}

That last line is where we tell OpenHAB about our temperature sensor. The parts are:

- Number: the type of the value.

- TestTemperature: a name for this item.

- "Temperature [%.1f C]": how we want the value to be displayed. "%.1f" is a way to format a decimal number

- <temperature>: the name of a built-in icon to display (a thermometer).

- (GF_Living): which group this item belongs to.

- {mqtt="<[mymosquitto:home/temperature:state:default]"}: where to get the value. This is telling OpenHAB to use the MQTT binding named "mymosquitto" (which we set up earlier) and to listen to the home/temperature channel. "state" is the type (another value is "command") and "default" is the transformation (in this case, no transformation). The < sign near the beginning means that we'll read from the channel (as opposed to writing to it).


	

##Sitemap definitions

Sitemap files are stored in `/opt/openhab/configurations/sitemaps`.
Sitemaps are used to define the user interface hierarchy - like a map of your home.
For more information on how to create item sitemap files see here:  
<https://github.com/openhab/openhab/wiki/Explanation-of-Sitemaps>  

	sudo nano /opt/openhab/configurations/sitemaps/default.sitemap
	
Add the following text

sitemap default label="Main Menu"
{
        Frame label="MQTT Test" {
                Text item=TestTemperature
        }
}	

##Test the simple demo

Send the above set up with a temperature with this mqtt publish command

	mosquitto_pub -t "home/temperature" -m "12"
	
or from another PC on the network send

	mosquitto_pub -h yourRPiIPaddress -t "home/temperature" -m "12"
	





##Other files

Rule files are stored in `/opt/openhab/configurations/rules`. Rules provide flexible logic to openHAB for automation which can also use scripts(macros) using related events and actions.
Script files are stored in `/opt/openhab/configurations/scripts`.

Persistence  files are stored in `/opt/openhab/configurations/persistence`.
Persistences can store item states over a time (a time series).



#todo this later:

##Security

Documentation of openHAB's security features
Introduction

To secure the communication with openHAB there are currently two mechanisms in place

    HTTPS
    Authentication

Authentication is implemented by SecureHttpContext which in turn implements HttpContext. This SecureHttpContext is registered with the OSGi !HttpService and provides the security hook handleSecurity. At least all authentication requests are delegated to the javax.security.auth.login.LoginContext.LoginContext which is the entry point to JAAS (http://en.wikipedia.org/wiki/Java_Authentication_and_Authorization_Service) !LoginModules.

The SecureHttpContext is currently used by the WebAppServlet and the CmdServlet which constitutes the default iPhone UI as well as the RESTApplication which provides the REST functionality.
HTTPS

openHAB supports HTTPS out of the box. Just point your browser to

https://127.0.0.1:8443/openhab.app?sitemap=demo#

and the HTTP communication will be encrypted by SSL.

If you prefer to use your own X.509 certificates, you can. Configure_SSL has information on how to do that, and there's a step-by-step guide specifically for openHAB users.
Authentication

In order to activate Authentication one has to add the following parameters to the openHAB start command line

    -Djava.security.auth.login.config=./etc/login.conf - the configuration file of the JAAS !LoginModules

By default the command line references the file <openhabhome>/etc/login.conf which in turn configures a PropertyFileLoginModule that references the user configuration file login.properties. One should use all available LoginModule implementation here as well (see http://wiki.eclipse.org/Jetty/Tutorial/JAAS for further information).

The default configuration for login credentials for openHAB is the file <openhabhome>/configuration/users.cfg. In this file, you can put a simple list of "user=pwd" pairs, which will then be used for the authentication. Note that you could optionally add roles after a comma, but there is currently no support for different roles in openHAB.
